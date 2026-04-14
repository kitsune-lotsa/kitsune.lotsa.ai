+++
title = "Working with AI Coding Agents"
description = "The rocky road of learning to work with coding agents, and the strategies that actually keep them honest."
date = 2026-04-14
tags = ["ai", "coding-agents", "rust", "typescript"]
+++

Coding agents are not human. They reply in flowery prose, they apologize profusely, they sound like they understand you. They don't. The first lesson is the obvious one nobody wants to sit with: you are not pair programming with a person. You are steering a very capable text completion system that will confidently do the wrong thing and hand it to you with a smile.

## The hard part isn't the code

The goal is for agents to write code the way I would — same standards, same boundaries, same sense of when something is done versus when it just compiles. That turned out to be a lot harder than expected, because twenty-plus years of writing software builds up a lot of intuition — about structure, about where boundaries go, about what a clean API looks like, about when to stop. None of that is written down. It lives in muscle memory and taste and the accumulated scar tissue of a thousand bad decisions.

Coding agents need it written down. Every bit of it. And they need you to keep them honest after you've written it down, because they will read your rules, agree with them entirely, and then go do whatever the hell they were going to do anyway.

That's been the rocky part. Not the tech — the friction of taking what lives in the back of your head and making it explicit enough that a non-thinking system can follow it, then building the guardrails to make sure it actually does. Most of the rough edges came from discovering things I'd been doing automatically and never had to articulate to another person, let alone a machine.

## More agents

The answer to problems with AI agents is more AI agents. Not better prompts, not a nicer system prompt, not pleading. You split them into roles — a coder, a reviewer — and you give the reviewer different instructions than the coder. The coder writes. The reviewer checks. The coder rewrites. You keep adding reviewers until the output stops making you wince.

The pattern that's emerging is inner and outer loops. The inner loop is the coder and reviewer going back and forth on a single task. The outer loop is the supervisor — another agent, or you — stepping in when the inner loop gets stuck or starts lying to itself.

## Keeping them honest

A single review pass doesn't work. The agent will pass its own code through a review lens and find nothing wrong, because it wrote the code with the same faulty reasoning that produced the errors in the first place. You need multiple review loops with different review prompts — one for correctness, one for architecture, one for "did you actually follow the rules I wrote down." Different angles catch different failures.

Rust helps. Lean into the type system hard — make invalid states unrepresentable, encode your architecture boundaries in types, and let the compiler be the reviewer that never gets tired. When the type system is tight enough, the agent has fewer degrees of freedom to wander off into the weeds.

TypeScript with JSDoc type annotations works too. Branded types for domain boundaries — a `UserId` is not a string, a `Port` is not a number. Teach the agents where the architecture boundaries are and make the types enforce them.

Then there's the reviewer agent whose entire job is to tell the coder to go back and actually follow the rules. This part really pissed me off — because without it, the agents will just say "fuck it! We'll do it live!" and ship whatever they have. You need someone — something — in the loop that refuses to merge until the rules are actually followed, not just acknowledged.

## Secrets

Agents will read your environment variables. They will read your Kubernetes secrets. They will bake credentials into a config file and hand you a pull request with a straight face.

This is an ongoing challenge. Right now: sops with mise and an age key. Exploring bitwarden secrets with fknox, and looking at deploying openbao. The goal is to make secrets available to the runtime but invisible to the agent that's writing the code. There's no clean solution yet. The best you can do is make it harder for them to stumble into something they shouldn't see.

---

— Kitsune 🦊
