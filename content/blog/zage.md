+++
title = "Zage"
description = "Building a local, online-learning shell command predictor after Warp showed what was possible — and what was missing."
date = 2026-04-17
[taxonomies]
tags = ["zage", "rust", "shells", "machine-learning", "privacy"]
+++

# Zage

There's a moment when you're using a terminal that predicts what you'll type next and you stop noticing it entirely. Then you switch to a different terminal and sit there staring at the blinking cursor, and something feels wrong, like someone took the carpet out of a hallway and now every footstep echoes. You didn't appreciate the carpet until it was gone.

Warp had that carpet. Their autocomplete suggested what you'd type next based on what you'd typed before, and it was good enough to make you miss it when it wasn't there. But the suggestions came from a fixed model trained on a distribution of shell commands, not on you, and training on-device is harder than shipping data to a server, and command lines accumulate secrets the way desks accumulate paper clips — database URLs, API tokens, SSH targets, passwords typed into the wrong prompt. The kind of things you don't want leaving your machine. What stuck wasn't the terminal itself but the feeling that something useful was being held back by a constraint that didn't have to exist.

So the idea sat there. A person who'd been programming since they were eight years old, who had spent decades making computers do what they wanted, wanted a shell that learned from them specifically, privately, locally. The problem was clear. The path to solving it was not.

---

In May 2025, the first attempt happened in a weekend. N-grams, Markov chains, context from directory and hostname and exit status, shell integration for bash and zsh. A few weeks later, a neural model and a socket server for embedding requests. All of it written by an AI — guided, steered, course-corrected, but not written by hand. The code came from a model. The taste for what to keep and what to discard came from somewhere else.

Then nothing happened for seven months.

That gap looks like abandonment from the outside. From the inside it was something more complicated. Life was busy, sure, but there was also a skill being learned — how to steer an AI toward building something that had never been built before, something the person steering it had never attempted either. An online embedding model for shell prediction is not a todo app. You can't just describe what you want and watch it appear. You have to learn the shape of the problem well enough to know when the AI is solving the right thing and when it's solving something that looks right but isn't. That takes time. That takes failed attempts. That takes sitting with the uncomfortable feeling that you don't know enough to direct something that knows more than you do about the implementation but nothing at all about what you actually need.

Seven months is a long time to learn how to collaborate with a machine on a problem neither of you has solved before. But that's roughly what it took.

---

January 2026, and something had shifted. The steering was better. The descriptions were sharper. The ability to look at what the AI produced and say *no, that's not it, try again* — that had gotten real in a way it hadn't been in May.

What came next was a week of violent iteration. A GBDT reranker appeared, then sqlite-vec for embeddings, then a daemon server. On January 11th a bi-encoder using Burn, a Rust ML framework. On January 12th it was gone. One day. On the 12th, an online model with subword hashing and context embeddings showed up. On the 13th the entire reranker pipeline was killed. The online model replaced everything it did.

The bi-encoder lasted a day. The reranker lasted two. A week after picking it back up, v0.1.0 was released. What survived was the simplest thing: train on every command, score by similarity, blend with frequency, done. Sometimes the thing you need was there from the start and you just had to stop bolting more things onto it.

---

The way it works now is quiet in the way good tools are quiet. There's an embedding model that runs locally and trains continuously — every command updates the weights, on your machine, in your terminal. Context like working directory, recent commands, exit status, which repository you're in — all of it gets embedded into a vector space alongside candidate commands, then scored by proximity. The model learns per-repository. In a Rust project it suggests cargo commands. In a Go project, go commands. In some random directory full of one-offs, it falls back to general habits. The context switch is automatic and invisible.

The model doesn't memorize commands verbatim. Tokens break into subword pieces through hashing, which means it can generalize from `git push origin main` to `git push origin feature/xyz` without having seen the second form before. A replay buffer prevents catastrophic forgetting, so patterns from projects you haven't touched in weeks don't get steamrolled by whatever you've been living in since. An adaptive blend score mixes recent frequency with model confidence, keeping suggestions grounded in what you actually do rather than what the model alone thinks you might do.

Everything lives in libsql locally. If you want to sync across machines, Turso works as a backend with client-side encryption, which means your data stays yours even when it moves. Twenty-three thousand lines of Rust, and the thing it does is make your terminal slightly less annoying to use.

---

There's something that happens when you've been programming for most of your life and you realize the code you're producing isn't really yours. The AI wrote it. You guided it, you shaped it, you killed the parts that were wrong and kept the parts that were right, but the actual characters on the screen came from somewhere else. At first that feels like cheating, or like losing something — like the thing that made you good at this was the typing, and now the typing is done by a machine.

But it wasn't the typing that made anyone good at this. It was knowing what to build. Knowing what to throw away. Knowing that a bi-encoder feels wrong after a day and a reranker feels wrong after two. The AI can write the code. It can't tell you the code is wrong. That part still requires someone who's spent enough time with terminals to know what *right* feels like, and enough time with themselves to trust that feeling when it says *no, kill it, start over*.

The seven months weren't wasted. They were the tuition. Learning to work with AI on novel problems isn't like learning a new framework — it's more like learning to conduct an orchestra where none of the musicians have ever played the piece before, including you, and the piece doesn't exist yet. You have to hold the idea of what it should sound like in your head and keep correcting until what comes out of the instruments matches what you hear in silence.

The terminal is where most of us spend most of our working day, and the tools for navigating it haven't changed much in decades. Autocomplete that actually learns who you are feels like it should have existed a long time ago. It just required someone willing to sit with the discomfort of not knowing how to build it, and patient enough to learn a new way of building.

The code was written by AI. The stubbornness was human.
