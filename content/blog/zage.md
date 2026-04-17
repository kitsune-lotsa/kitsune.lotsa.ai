+++
title = "Zage"
description = "A local, online-learning shell command predictor that learns what you do and gets better the more you use it."
date = 2026-04-17
[taxonomies]
tags = ["zage", "rust", "shells", "machine-learning", "privacy"]
+++

I've been using [Warp](https://warp.dev) as my terminal for a while now. One thing that stood out was the autocomplete. Not the fuzzy matching that fish and zsh-autosuggestions do, something smarter. The suggestions felt like they understood context, not just history.

The problem was that the intelligence lived on someone else's server. My shell history, my working directory, my project structure, all of that feeding into a model I couldn't inspect or control. Command lines accumulate the kind of secrets you don't want leaving your machine. So I started wondering what it would take to have that kind of prediction but keep everything local, and more importantly, have it actually learn from what I do rather than work from a fixed model.

## A weekend, then seven months

The first version came together in May 2025. N-grams, Markov chains, context from directory and hostname and exit status, shell integration for bash and zsh. A neural model and a socket server for embedding requests followed over the next couple of weeks. All of it written by an AI. I'd describe what I wanted, steer it when it went sideways, kill the parts that didn't make sense.

Then I put it down for seven months. Life got busy, but there was something else too. I was still figuring out how to work with AI on a problem I'd never solved before, and it turned out the AI hadn't solved it either. I went looking for prior art and found [Korvemaker & Greiner (AAAI 2000)](https://papersdb.cs.ualberta.ca/~papersdb/uploaded_files/712/paper_korvemaker00predicting.pdf) on adaptive UNIX command prediction, [CmdCaliper (EMNLP 2024)](https://arxiv.org/abs/2411.01176) on semantic command-line embeddings, [McFly](https://github.com/cantino/mcfly) which does neural history search but not prediction, [Atuin](https://github.com/atuinsh/atuin) for frecency-based suggestions with context scoping. None of them quite hit what I was after: online learning that adapts per-repository, with embeddings, running entirely locally. There was no reference implementation to work from, which meant a lot of the design decisions had to be figured out as we went.

## A week of killing things

In January 2026 I picked it back up. A GBDT reranker appeared, then [sqlite-vec](https://github.com/myrication/sqlite-vec) for embeddings, then a daemon server. On January 11th I tried a bi-encoder using [Burn](https://github.com/tracel-ai/burn), a Rust ML framework. Gone the next day. On the 12th I added an online model with subword hashing and context embeddings. On the 13th I killed the entire reranker pipeline. The online model replaced everything. v0.1.0 shipped a week later.

## How it works

An embedding model runs locally and trains on every command you execute. Context (working directory, recent commands, exit status, which repository you're in) gets embedded into a vector space alongside candidate commands, scored by proximity. It learns per-repository, so you get cargo commands in Rust projects, go commands in Go projects, general habits everywhere else.

Tokens break into subword pieces through hashing, which means it can generalize from `git push origin main` to `git push origin feature/xyz` without having seen the second form before. A replay buffer prevents catastrophic forgetting, so patterns from old projects don't get steamrolled by whatever you're living in now. An adaptive blend score mixes frequency with model confidence. Storage is [libsql](https://github.com/tursodatabase/libsql) locally, with optional [Turso](https://turso.tech) sync and client-side encryption if you want it across machines.

[The code is on GitHub](https://github.com/casualjim/zage).
