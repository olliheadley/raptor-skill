# The deletion audit

The legwork engine for [Step 1](SKILL.md). Read-only. The goal is a proven list of what to delete and what to combine — with evidence, not vibes.

## Fan out

Split the surface into areas (one per subsystem: routing, providers, tools, git, persistence, config, docs, dependencies). Give each a **read-only** Explore agent, in parallel. Each agent reads its area thoroughly and returns candidates — it does not edit.

Tell every agent the order that matters: **prove non-use before recommending deletion.** A wrong "safe-delete" on externally-called code breaks production.

Each agent also applies the **deletion test** to its area — flagging shallow modules (interface as complex as their body) and leaky seams as `combine`/`shallow-module` candidates, so the audit surfaces where to *deepen*, not only where to cut.

## Candidate schema

Each candidate an agent returns:

| Field | Meaning |
|---|---|
| `item` | the concrete thing — file, function, endpoint, dependency, feature |
| `action` | `delete` · `combine` · `question-requirement` |
| `type` | `dead-code` · `disabled-feature` · `vestigial` · `legacy-shim` · `unused-dependency` · `orphan-endpoint` · `stale-doc` · `duplicate-to-combine` · `shallow-module` · `leaky-seam` |
| `confidence` | `safe-delete` · `needs-owner-signoff` · `investigate` |
| `evidence` | file:line **plus proof of non-use** (see checklist) |
| `loc_impact` | approximate lines removed or collapsed |
| `owner_question` | for `needs-owner-signoff`: the exact question to put to the human owner |

## Proof-of-non-use checklist

A candidate is only `safe-delete` when you can show one of these:

- **No importers** — grep the whole tree; the only hit is the symbol's own definition.
- **No callers** — the function/endpoint is defined but never invoked.
- **Disabled flag** — commented out of a registry, behind a false flag, or marked "temporarily disabled / not implemented".
- **Dead branch** — a code path the surrounding logic can never reach (e.g. a default the caller always overrides).
- **Not in the external consumer's allowlist** — for API surface, the actual reverse-proxy / frontend / client repo never references it. *This is the one you cannot skip — read the consumer.*

Anything on a used surface where you cannot produce this proof is `needs-owner-signoff`, with a sharp `owner_question` — never `safe-delete`.

## Tally

Aggregate the candidates and sort by confidence and `loc_impact`:

- **Safe-delete total** — schedule directly into the first deletion PR.
- **Needs-owner-signoff** — collect the `owner_question`s and put them to the owner in one batch. Deleting API surface is behavior-changing until the owner confirms non-use.
- **Combine** — the survivors for [Step 3](SKILL.md); each names the N duplicate sites (or the shallow modules) and the single **deep module** they collapse into.

The headline number — "≈N LOC deletable, of M total" — is what makes the Raptor 1 → Raptor 3 case concrete. If the safe-delete pile is tiny, either the codebase is already lean or the audit wasn't aggressive enough; re-read for disabled features and vestigial paths before concluding it's lean.
