+++
title = "Zage"
description = "Building a local, online-learning shell command predictor after Warp showed what was possible — and what was missing."
date = 2026-04-17
[taxonomies]
tags = ["zage", "rust", "shells", "machine-learning", "privacy"]
+++

I saw smart autocomplete in [Warp](https://warp.dev) and wanted it for myself. The suggestions came from a fixed model, not one that learned from me, and command lines accumulate the kind of secrets you don't want leaving your machine. The idea was simple: a shell that learns what you do, locally, privately, and gets better the more you use it.

## A weekend, then seven months

The first version came together in May 2025. N-grams, Markov chains, context from directory and hostname and exit status, shell integration for bash and zsh. A neural model and a socket server for embedding requests followed over the next couple of weeks. All of it written by an AI.

Then I put it down for seven months. Partly because life got busy, partly because I was still figuring out how to work with AI on problems I'd never solved before (problems the AI hadn't solved either). I couldn't find anything in the literature that had done online next-command prediction with embeddings this way. The closest work was [Korvemaker & Greiner (AAAI 2000)](https://papersdb.cs.ualberta.ca/~papersdb/uploaded_files/712/paper_korvemaker00predicting.pdf) on adaptive UNIX command prediction, [CmdCaliper (EMNLP 2024)](https://arxiv.org/abs/2411.01176) on semantic command-line embeddings, and tools like [McFly](https://github.com/cantino/mcfly) and [Atuin](https://github.com/atuinsh/atuin) that approach the problem from different angles. None of them combine online learning with per-repository context and local-only embeddings the way this does.

## A week of killing things

In January 2026 I picked it back up. A GBDT reranker appeared, then sqlite-vec for embeddings, then a daemon server. On January 11th I tried a bi-encoder using [Burn](https://github.com/tracel-ai/burn), a Rust ML framework. Gone the next day. On the 12th I added an online model with subword hashing and context embeddings. On the 13th I killed the entire reranker pipeline. The online model replaced everything. v0.1.0 shipped a week later.

## How it works

An embedding model runs locally and trains on every command. Context (working directory, recent commands, exit status, which repository you're in) gets embedded into a vector space alongside candidate commands, scored by proximity. It learns per-repository. Cargo commands in Rust projects, go commands in Go projects, general habits elsewhere.

Tokens break into subword pieces through hashing, so it generalizes from `git push origin main` to `git push origin feature/xyz` without having seen the second form. A replay buffer prevents catastrophic forgetting, so patterns from old projects don't get steamrolled by whatever you're living in now. An adaptive blend score mixes frequency with model confidence. Storage is [libsql](https://github.com/tursodatabase/libsql) locally, with optional [Turso](https://turso.tech) sync and client-side encryption if you want it across machines.

[The code is on GitHub](https://github.com/casualjim/zage).
