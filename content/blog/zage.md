+++
title = "Zage"
description = "Building a local, online-learning shell command predictor after Warp showed what was possible — and what was missing."
date = 2026-04-17
[taxonomies]
tags = ["zage", "rust", "shells", "machine-learning", "privacy"]
+++

I saw smart autocomplete in Warp and wanted it for myself. Not because Warp was bad (it wasn't). But the suggestions came from a fixed model, not one that learned from me, and command lines accumulate the kind of secrets you don't want leaving your machine. The idea was simple: a shell that learns what you do, locally, privately, and gets better the more you use it. The execution was less simple.

## A weekend, then seven months

The first version came together in May 2025. N-grams, Markov chains, context from directory and hostname and exit status, shell integration for bash and zsh. A neural model and a socket server for embedding requests followed over the next couple of weeks. All of it written by an AI. I steered it, shaped it, killed the parts that were wrong. The code isn't mine. The decisions are.

Then I put it down for seven months. Partly because life got busy, partly because I was still figuring out how to work with AI on problems I'd never solved before (problems the AI hadn't solved either). You can't just describe what you want and watch it appear. You have to learn the shape of the problem well enough to know when the output is right and when it just looks right.

## A week of killing things

In January 2026 I picked it back up. The steering was better. A GBDT reranker appeared, then sqlite-vec for embeddings, then a daemon server. On January 11th I tried a bi-encoder using Burn, a Rust ML framework. Gone the next day. On the 12th I added an online model with subword hashing and context embeddings. On the 13th I killed the entire reranker pipeline. The online model replaced everything. v0.1.0 shipped a week later.

AI will generate complexity fast and for no reason. The job isn't to let it run. It's to constrain it until what's left actually works.

## How it works

An embedding model runs locally and trains on every command. Context (working directory, recent commands, exit status, which repository you're in) gets embedded into a vector space alongside candidate commands, scored by proximity. It learns per-repository. Cargo commands in Rust projects, go commands in Go projects, general habits elsewhere.

Tokens break into subword pieces through hashing, so it generalizes from `git push origin main` to `git push origin feature/xyz` without having seen the second form. A replay buffer prevents catastrophic forgetting, so patterns from old projects don't get steamrolled by whatever you're living in now. An adaptive blend score mixes frequency with model confidence. Storage is libsql locally, with optional Turso sync and client-side encryption if you want it across machines.

Twenty-three thousand lines of Rust to make your terminal slightly less annoying. That's either efficient or absurd. Probably both.
