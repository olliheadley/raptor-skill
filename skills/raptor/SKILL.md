---
name: raptor
description: "Refactor by subtraction, using the SpaceX 5-step algorithm (question requirements, delete, simplify, accelerate, automate) to take a codebase from Raptor 1 to Raptor 3, meaning fewer and deeper modules with only the seams that earn their place. Use when simplifying architecture, removing dead code or overhead, cutting bloat, reducing complexity, or when the goal is to delete before adding. Improves architecture by removing, not adding."
license: MIT
metadata:
  author: olliheadley
  version: '1.0.0'
---

# Raptor

Picture the three engines side by side. **Raptor 1** is a rat's nest of external plumbing, sensors, and wires. **Raptor 3** makes the same thrust with a smooth outside — because every part that could be deleted or integrated *was*. *The best part is no part. What isn't there can't break, and can't explode.*

This skill refactors code by **subtraction**, using the 5-step algorithm. The end state is not merely fewer lines — it is **better architecture, reached by removal**: fewer, **deeper modules**, every remaining **seam** earning its place, the whole thing more testable and easier for a human or an agent to navigate. Subtraction is the method; improved architecture is the result. *If a step leaves the code smaller but not clearer, you deleted lines, not complexity.*

Run the steps **in order** — the order is the whole point:

> The most common error of a smart engineer is to **optimize a thing that should not exist.** Steps 3–5 (simplify, accelerate, automate) are wasted — or worse — on a part that step 2 would have deleted. The moment you catch yourself designing an abstraction, adding a helper, or writing a test before you have deleted, you have skipped ahead. Go back to step 1.

## The lens you cut with

A small vocabulary so "improve the architecture" stays checkable, not vibes (full set: `/codebase-design`):

- **Deep module** — a narrow interface over substantial hidden implementation. The shape you are cutting toward.
- **Shallow module** — interface nearly as complex as what it hides; it earns its keep only by existing. A prime target for deletion or absorption.
- **Seam** — a boundary you can test or swap across. Every seam is a cost: keep the ones that carry weight, delete the ones that merely chop the code into pieces.
- **The deletion test** — your primary instrument in Steps 1–2. For anything you suspect is shallow: would deleting it *concentrate* complexity somewhere sensible, or just *move* it around? "Concentrates" means delete or absorb it.
- **Locality** — related logic (and its bugs) lives together. A pure function extracted only for testability, far from where it is called, is shallow. *The interface is the test surface* — if a thing is hard to test through its real interface, that is the smell, not a missing unit test.

## Step 1 — Make the requirements less dumb

Every requirement is guilty until proven necessary. *"Your requirements are definitely dumb, it does not matter who gave them to you"* — and the ones from a smart person (including the user, including yourself five minutes ago) are the most dangerous, because you question them least.

- **Inventory the surface, and find the friction.** Fan out read-only Explore agents in parallel to list what exists — modules, endpoints, features, flags, dependencies, duplicated concepts — and, applying the deletion test as you go, note the architectural friction: where understanding one concept means bouncing between many small modules, where a module's interface is as complex as its body, where extractions exist only for testability with no locality, and where tightly-coupled modules leak across their seams. These are the parts most likely to be dumb requirements. See [AUDIT.md](AUDIT.md) for the fan-out recipe and the candidate schema.
- **Give every part an owner.** Each requirement must be accountable to a *named person*, not a department — someone you can ask "why does this exist?" A part nobody owns is the first to go.
- **Verify against reality, not the repo.** Anything on a public surface — an API, an exported interface, a config key — may be used by a caller you can't see. *Unreferenced in this repo ≠ unused.* Read the actual consumer (the other repo, the frontend, the reverse-proxy allowlist) before you believe a thing is dead.
- **Question the requirement even when the user gave it.** If the evidence says a requested part is redundant or dumb, don't silently build it and don't silently skip it — bring the evidence back and let the owner re-decide. That is this step working as intended, not insubordination.

**Done when** every part on the surface either has an owner and a live reason to exist, or is named on the delete list — and every externally-facing item's real callers have been checked, not assumed.

## Step 2 — Delete the part or process

*"If you're not adding things back at least 10% of the time, you're not deleting enough."* Start lean; add back only what you are forced to. You can make an "in case" argument for anything — refuse it.

Delete: dead code, disabled or half-built features, vestigial paths, legacy shims, unused dependencies, orphaned endpoints, shallow modules that fail the deletion test, and any duplicate of something that already works elsewhere.

- **Two confidence tiers.** *Safe-delete* = provably unreachable (no callers, dead branch, disabled flag) — just do it. *Needs-owner-signoff* = anything on a used surface — confirm external non-use (Step 1) before cutting.
- **Preserve observed behavior.** Deleting never-reached code changes nothing observable — that is your license to cut without ceremony. Anything that would change a *used* path's behavior is not a deletion; it goes to the backlog or the owner, never done silently.

**Done when** each deletion is proven safe (build clean, no callers, consumer checked), the removed lines are counted — and you have added *something* back. If you added back nothing, you did not delete enough: go again.

## Step 3 — Simplify and combine into deep modules

*Now*, and only for what survived — this is where you "optimize a thing that should exist," and where the leaner codebase becomes a *better-architected* one.

Combine toward **depth**: several shallow modules implementing one responsibility collapse into a single **deep module** — one home, a narrow interface, the complexity hidden inside. Ask of every survivor — *"this is used by X, and Y also needs it; can they share one deep part?"* Three git modules become one. Four HTTP clients become one. A type defined three times becomes one **single source of truth**.

Judge the result by the deletion test in reverse: the new module should *concentrate* complexity behind a simple interface and be testable through that interface. If combining only produced a bigger shallow module, stop — you moved complexity, you did not hide it.

**Done when** every survivor concept has exactly one home, no responsibility is implemented twice, and each combined module is deep (simple interface, real implementation) rather than a merged blob.

## Step 4 — Accelerate

Only after 1–3. Speed up the loop — but *"if you're digging your grave, don't dig faster."* A faster cycle aimed at a part you should have deleted just reaches the wrong place sooner. Keep this minimal; usually the existing dev loop is enough once the surface is small.

## Step 5 — Automate

Last. Add tests or CI for what *remains* — an add-back justified by the risk of the Step-3 merges, not a goal in itself. Automating a process before deleting it is the Tesla battery-mat mistake: pouring effort into streamlining a thing that turned out not to be needed at all. Test each survivor *through its own interface* — the deep modules from Step 3 make this cheap, which is the point.

## Produce the plan

The output is a plan of the **tiniest commits**, each leaving the codebase working (Fowler), ordered so deletion lands before combination and each step is one reviewable PR. Compose the sibling skills instead of restating them:

- **Interview, scope, and file it** — run `/request-refactor-plan` for the interview discipline, the tiny-commit breakdown, and the issue template. These five Raptor steps are the plan's spine and its commit order.
- **Present candidates when the surface is large** — a short ranked list (`strong` / `worth-exploring` / `speculative`), each with the files, the friction, and the deeper shape it collapses into, so the user can pick before you cut.
- **Pressure-test a direction** — `/grilling` before committing to a combine that touches a load-bearing survivor; `/codebase-design` (design-it-twice) when a survivor's new interface is worth exploring twice.
- **Backlog the deferrals** — every behavior-changing find and every deferred owner decision from Steps 1–2 goes to the backlog with file:line and a proposed fix, so nothing is lost and nothing is done silently.

## Verify nothing broke

Behavior-preserving is a claim you must be able to *show*. Capture a golden-path baseline before the first cut; after each PR, diff against it (build clean, tests green, same outputs on the paths that matter). "I'm confident it still works" is not verification — a passing diff is. Better architecture is checkable too: each deep module you built should be testable through its interface, and a newcomer — human or agent — should reach any concept in fewer hops than before.
