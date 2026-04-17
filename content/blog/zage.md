+++
title = "Zage"
description = "Building a local, online-learning shell command predictor after Warp showed what was possible — and what was missing."
date = 2026-04-17
[taxonomies]
tags = ["zage", "rust", "shells", "machine-learning", "privacy"]
+++

# Zage

I used Warp for a while. Their smart autocomplete worked well — ghost text suggestions that appear as you type, the kind of thing you stop noticing until you switch terminals and find yourself staring at a blank line expecting something to be there. It predicted what I'd type next based on what I'd typed before. Saved keystrokes. Added up.

They weren't doing online learning. The suggestions came from a fixed model that didn't adapt to how I actually worked. That's a reasonable product decision — training models is expensive and doing it on-device is harder than doing it server-side. But it meant the autocomplete was pattern matching against a static distribution of shell commands, not against me. Useful, but capped.

The suggestions required sending shell history to a third-party endpoint. Command lines accumulate secrets: database URLs, API tokens, SSH targets, internal service names. Ultimately there was too much AI in Warp without knowing what those third-party endpoints could see. I don't mind AI — I just want to know when I'm being watched by it, and what it sees and sends. The UI wasn't for me either, but that was secondary.

So I got curious whether I could build it myself.

---

[Zage](https://github.com/casualjim/zage) is a Rust CLI that predicts your next shell command. It runs locally, learns as you use it, and hooks into zsh and bash through the suggestion mechanisms those shells already support. Roughly 23,000 lines of Rust, which is not what I planned to write when I started.

## How it works

The core is an online embedding model that learns continuously from your shell history. Context — workspace, directory, recent commands, exit status — gets embedded into a vector space, and candidate commands get embedded the same way, then scored by how close they are. The model trains continuously — every command you run updates the weights, on your machine, in your terminal. No round-trip to a server.

Storage is libsql locally. If you want to sync across machines, Turso works as a backend with client-side encryption. Your data, your keys.

The model doesn't memorize commands verbatim. Tokens get broken into subword pieces through hashing, which means it can generalize from `git push origin main` to `git push origin feature/xyz` without having seen the second form. Sampled softmax keeps training tractable as the vocabulary grows. A replay buffer prevents catastrophic forgetting — the patterns from projects you haven't touched in weeks don't get steamrolled by whatever you've been living in since. An adaptive blend score mixes recent frequency with model confidence, so suggestions are grounded in what you actually do rather than what the model alone thinks you might do.

It learns per-repository. In a Rust project, it suggests cargo commands. In a Go project, go commands. In ~/tmp running one-offs, it falls back to general habits. The context switch is automatic.

## What happened

I built the whole foundation in a weekend — n-gram model, Markov chains, directory and hostname and exit status context, sequence detection with SQL scoring, shell integration for bash and zsh. All of it in two days at the start of May 2025. Then I spent a few weeks on a neural model foundation with pretrained embeddings and a socket server for embedding requests, and then I put it down for seven months.

Seven months is a long time to ignore something you were excited about on a Saturday. But that's what happens — other things come up, the enthusiasm doesn't disappear, it just goes dormant underneath whatever is paying the rent. The repo sat there.

I picked it back up in January. Came in hot with a hybrid reranker and a GBDT implementation and a daemon server and sqlite-vec for command embeddings. On January 11th I tried building a bi-encoder using Burn, a Rust ML framework. On January 12th I removed it. It lasted one day. On January 12th I also added online model tokenization with subword hashing and context embeddings. On January 13th I killed the entire reranker pipeline — the online model replaced everything it did. By the 14th the online prediction epic was closed as "always-on." A week later I tagged v0.1.0.

The story isn't gradual improvement. The story is building something, abandoning it, coming back, throwing away the complicated parts, and landing on something simpler that actually worked. The bi-encoder lasted a day. The reranker lasted two. What survived was the online model — train on every command, score by similarity, blend with frequency, done. Sometimes the thing you need was there from the start and you just had to stop bolting things onto it.

https://github.com/casualjim/zage
