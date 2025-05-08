---
layout: post
title:  "OO Interfaces, Types & Contracts — Why They Still Matter in 2025"
date:   2025-05-06 01:00:00 -0300
categories: [ oop, ai, braindump ]
tags: [en-US]
permalink: /posts/oo-interfaces-types-and-contracts-why-they-still-matter-in-2025
---

**Hey folks — long time no post!**  
After a fun detour at *unbounded.dev*, I’m reviving my personal blog. Expect fast, low‑friction braindumps: ideas captured before they calcify, with minimal “production polish.” Today’s dump is on a workhorse topic I keep circling back to—object‑oriented interfaces, static types, and why they still matter (especially now that we’re all pairing with AI agents).

---

### Interfaces as Living Contracts

An interface is a **formal promise**: any class that implements it *will* expose these methods and *will* respect these types.  
A static type checker becomes your automatic referee, blocking a call to `sendEmail()` on something that only guarantees `sendSMS()`.

In practice, that means:

* **Fewer “undefined‑method” surprises** sneaking into prod.  
* **Docs that don’t rot.** Change a signature and the build fails until every caller updates.  
* **Peace of mind** that, at minimum, the *shape* of your objects is right before you hit `run`.

### How Interfaces Shape the Way We Think

When contracts are explicit, your brain naturally separates **what** a module does from **how** it does it:

* Cleaner, more modular architectures.  
* Teams building in parallel as long as everyone targets the same interface.  
* Fewer “integration Fridays” spent untangling mismatched method calls.

It’s design‑by‑contract baked straight into the language—no extra slide deck required.

### Interfaces + AI: A Natural Pair

Large‑language‑model coders crave structure. Hand an agent an interface like:

```text
Implement Persistence<T> with:
  save(item: T): void
  load(id: string): T
```

… and ambiguity disappears; if it drifts, the compiler screams.
Research on type‑constrained decoding shows that feeding LLMs interface info slashes compile‑time errors and boosts functional correctness—formal contracts help machines as much as humans.

### The Cost Spectrum

There’s an ongoing skirmish between “just trust the developer” camps (think classic C) and “prove everything up front” camps (hello, Rust).
Rust’s borrow‑checker eliminates whole classes of memory bugs, but compile times are noticeably slower than C and the learning curve is steeper.

Does that overhead always pay off? Not necessarily. On fast‑moving or “vibe‑coding” projects, minimal typing—just enough interfaces and value objects to catch foot‑guns—can be a sweet spot. You keep iteration speed high while still avoiding the most face‑palm errors.

Static types aren’t an all‑or‑nothing proposition; they’re a dial you can turn based on risk, performance targets, and team expertise. Strike the balance that makes sense for your codebase instead of fighting holy wars.

### Reality Checks

* Up‑front cost. More declarations; quick prototypes can feel slower.
* False sense of safety. Type‑correct code can still be logically wrong—testing stays mandatory.
* Over‑engineering risk. An interface for every micro‑concept leads to abstraction soup. Follow the Interface Segregation Principle: split only when a real client needs the smaller surface.

### Wrap‑Up

Interfaces and strong static types aren’t magic bullets, but they remain one of the cheapest ways to buy predictability—and now, to boost the productivity of your AI coding sidekicks. Use them freely, stay pragmatic, and I’ll see you in the next brain‑dump!