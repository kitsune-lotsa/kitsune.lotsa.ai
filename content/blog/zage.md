+++
title = "Zage"
description = "Building a local, online-learning shell command predictor after Warp showed what was possible — and what was missing."
date = 2026-04-17
[taxonomies]
tags = ["zage", "rust", "shells", "machine-learning", "privacy"]
+++

# Zage

There's a thing that happens when you use a tool that predicts what you'll type next in a terminal. You stop noticing it. Then you switch to a different terminal and you sit there staring at the blinking cursor, and it feels like something is missing, like someone took the carpet out of a hallway and now every footstep echoes. You didn't appreciate the carpet until it was gone.

That was Warp, for me. Their autocomplete suggested what I'd type next based on what I'd typed before, and it was good. Ghost text appearing as you type. Keystrokes saved. It adds up — not in any dramatic way, just in the way that small conveniences always do, invisibly, until you lose them.

But it wasn't learning. Not really. The suggestions came from a fixed model trained on a distribution of shell commands, not on me. That's a reasonable product decision — training models on-device is harder than doing it server-side, and server-side means you need to send data somewhere, and command lines accumulate secrets the way desks accumulate paper clips. Database URLs, API tokens, SSH targets, internal service names, passwords occasionally typed directly into the wrong prompt. The kind of things you don't want leaving your machine.

There was also the question of what those third-party endpoints could see. I don't mind AI, but I'd like to know when I'm being watched by it, and what it sees. The UI wasn't for me either, but that was secondary. What stuck was the feeling that something useful was being held back by a constraint that didn't have to exist.

So I started building one myself.

---

I didn't set out to write 23,000 lines of Rust. I set out to see if it was possible, and then I couldn't stop.

The first weekend in May 2025 I put together the foundation — n-grams, Markov chains, context from directory and hostname and exit status, shell integration for bash and zsh. The basics. A few weeks later I added a neural model and a socket server for embedding requests. Then I put it down for seven months.

Seven months. That's a long time to ignore something you were excited about on a Saturday afternoon. But that's how it goes — other things come up, the enthusiasm doesn't disappear, it just goes dormant under whatever is paying the rent and demanding attention. The repo sat there.

I picked it back up in January. I came in with a GBDT reranker and sqlite-vec for embeddings and a daemon server and all the scaffolding you'd expect from someone trying to solve the problem with enough complexity to feel serious about it. On January 11th I tried building a bi-encoder using Burn, a Rust ML framework. On January 12th I removed it. One day. On the 12th I also added an online model with subword hashing and context embeddings. On the 13th I killed the entire reranker pipeline. The online model replaced everything it did.

The bi-encoder lasted a day. The reranker lasted two. What survived was the simplest thing: train on every command, score by similarity, blend with frequency, done. Sometimes the thing you need was there from the start and you just had to stop bolting more things onto it.

---

The way it works now is this. There's an embedding model that runs locally and trains continuously — every command you run updates the weights, on your machine, in your terminal. Context like your working directory, recent commands, exit status, which repository you're in — all of that gets embedded into a vector space. Candidate commands get embedded the same way, then scored by proximity. The model learns per-repository. In a Rust project it suggests cargo commands. In a Go project, go commands. In some random directory where you're running one-offs, it falls back to general habits. The context switch is automatic.

The model doesn't memorize commands verbatim. Tokens get broken into subword pieces through hashing, which is a way of saying it can generalize from `git push origin main` to `git push origin feature/xyz` without having seen the second form before. There's a replay buffer to prevent catastrophic forgetting — so the patterns from projects you haven't touched in weeks don't get steamrolled by whatever you've been living in since. And an adaptive blend score mixes recent frequency with model confidence, keeping suggestions grounded in what you actually do rather than what the model alone thinks you might do.

Everything lives in libsql locally. If you want to sync across machines, Turso works as a backend with client-side encryption, which means your data stays yours even when it moves.

---

I think there's something about building a tool for yourself that's different from building one for a user. When you build for a user you're solving a problem you imagine they have. When you build for yourself you're solving a problem you have, and you know immediately when the solution is wrong because you feel it every time you open a terminal. The bi-encoder was wrong. I felt it in a day. The reranker was wrong. I felt it in two. What was right was the thing I should have trusted from the beginning — simple, online, adapting to me, not pretending to be smarter than it needed to be.

The terminal is a strange place. It's where most of us spend most of our working day, and yet the tools for navigating it haven't changed much in decades. Autocomplete that actually learns who you are feels like it should have existed a long time ago. It just required someone stubborn enough to build it locally, and foolish enough to think 23,000 lines of Rust was a reasonable answer to a weekend project.

It wasn't a weekend project. Not really. It was a weekend project that turned into seven months of thinking about it, followed by a week of throwing away everything that was unnecessary, followed by a thing that works the way I always wanted Warp to work.

Sometimes the best thing you can build is the thing that should already exist.
