# Raptor

A Claude skill for refactoring by subtraction. It takes the way SpaceX simplifies a rocket engine and points it at code.

If you have seen the photo of Raptor 1, 2 and 3 lined up next to each other, you already get the idea. Raptor 1 is buried in wires and pipes. Raptor 3 is almost smooth, because every part that could be removed or folded into another part was. The rule behind it is short: the best part is no part. What is not there cannot break.

Most refactoring advice tells you to add structure. This skill tells you to take it away first, and only add a little back for the small amount that survives.

## Install

```bash
npx skills add olliheadley/raptor-skill
```

That drops the skill into your agent's skills folder. In Claude Code you then run it by name:

```
/raptor
```

## What it does

Raptor runs Elon Musk's five step algorithm, in order, because the order is the whole point.

1. **Question the requirements.** Every requirement is guilty until proven necessary, and the ones handed to you by a smart person are the most dangerous, because you question them least. Give each part an owner you can actually ask "why does this exist". For anything on a public surface, read the real caller before you call it dead. Unreferenced in your repo does not mean unused.
2. **Delete.** Dead code, disabled features, half finished paths, unused dependencies, duplicates of things that already work somewhere else. If you are not adding at least ten percent of it back later, you did not delete enough.
3. **Simplify and combine what is left.** Collapse the same job done in three places into one deep module: a small interface with the real work hidden behind it. This is the step that turns a smaller codebase into a better one.
4. **Accelerate.** Speed the loop up, but only now. If you are digging your grave, do not dig faster.
5. **Automate.** Last. Add tests for what remains, not for the thing you were about to delete.

The whole idea in one line: delete before you optimize. The most common mistake a good engineer makes is polishing something that should not exist in the first place.

## Why the order matters

Steps three, four and five are wasted on a part that step two would have removed. If you notice yourself designing an abstraction or writing a test before you have deleted anything, you jumped ahead. Go back to step one.

## What is in here

- `skills/raptor/SKILL.md` is the skill itself.
- `skills/raptor/AUDIT.md` is the read-only audit recipe it uses in step one: how to fan out across a codebase, prove a thing is actually unused, and sort what you find by how sure you are.

## Where it came from

The five steps are Musk's, from the interview he gave Everyday Astronaut during a tour of Starbase. The tiny commit discipline in the planning step is Martin Fowler's: make each step small enough that the program still runs after it. The deep versus shallow module language comes from John Ousterhout's A Philosophy of Software Design.

## License

MIT. Use it however you like.
