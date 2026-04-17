+++
title = "Zage"
description = "Building a local, online-learning shell command predictor after Warp showed what was possible — and what was missing."
date = 2026-04-17
[taxonomies]
tags = ["rust", "shells", "machine-learning", "privacy"]
+++

# Zage

I used Warp for a while. Their smart autocomplete worked well — the kind of feature that disappears into your workflow until you try a terminal without it and keep reaching for a key that does nothing. It predicted what I'd type next based on what I'd typed before. Saved keystrokes. Added up.

They weren't doing online learning. The suggestions came from a fixed model that didn't adapt to how I actually worked. That's a reasonable product decision — training models is expensive and doing it on-device is harder than doing it server-side. But it meant the autocomplete was pattern matching against a static distribution of shell commands, not against me. Useful, but capped.

The suggestions also required sending shell history to a third-party endpoint. Command lines accumulate secrets: database URLs, API tokens, SSH targets, internal service names. I don't have a problem with Warp specifically — it's a solid terminal. I just don't like not being in control of what an AI sees, especially when it's someone else's API endpoint.

So I got curious whether I could build it myself.

---

[Zage](https://github.com/casualjim/zage) is a Rust CLI that predicts your next shell command. It runs locally, learns as you use it, and hooks into zsh and bash through the suggestion mechanisms those shells already support. Roughly 23,000 lines of Rust, which is not what I planned to write when I started.

## How it works

The core is an online embedding model that learns continuously from your shell history. Context — workspace, directory, recent commands, exit status — gets embedded into a vector space, and candidate commands get embedded the same way, then scored by how close they are. The model trains continuously — every command you run updates the weights, on your machine, in your terminal. No round-trip to a server.

Storage is libsql locally. If you want to sync across machines, Turso works as a backend with client-side encryption. Your data, your keys.

The model doesn't memorize commands verbatim. Tokens get broken into subword pieces through hashing, which means it can generalize from `git push origin main` to `git push origin feature/xyz` without having seen the second form. Sampled softmax keeps training tractable as the vocabulary grows. A replay buffer prevents catastrophic forgetting — the patterns from projects you haven't touched in weeks don't get steamrolled by whatever you've been living in since. An adaptive blend score mixes recent frequency with model confidence, so suggestions are grounded in what you actually do rather than what the model alone thinks you might do.

It learns per-repository. In a Rust project, it suggests cargo commands. In a Go project, go commands. In ~/tmp running one-offs, it falls back to general habits. The context switch is automatic.

## What happened

I started on a weekend. Frequency counting, basically. Then I wanted generalization. Then per-context learning. Then proper embeddings. Then the embeddings needed to train online without being slow. Then replay buffers because the model forgot old patterns. Then blend scoring because the model was overconfident about weird suggestions.

Each step was small. Each step was necessary. The result is something I use every day and that genuinely gets better the more I use it. Everything runs locally. My shell history stays on my machine.

[Zage](https://github.com/casualjim/zage) is MIT licensed.
