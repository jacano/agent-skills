# AGENTS.md — agent-skills

Working notes for an agent that lands in this repository. Read this first.

## What this is

One directory per skill. Each holds a `SKILL.md` — the procedure, the failure it avoids
and what was verified — and the scripts that carry it out. The scripts have no
dependencies and keep their state in the system temp directory, so they run from a clone
in any location.

`README.md` lists the skills; do not repeat that list here.

The sibling repositories are [`dsh-config`](https://github.com/jacano/dsh-config) (the
Harness configuration) and [`model-bench`](https://github.com/jacano/model-bench) (the
benchmark that chooses its models). A skill here may call into either; say so in the
skill rather than copying their content.

## Picking one up

1. Read the skill's `SKILL.md` in full before running anything. These procedures exist
   because the obvious version of the task fails silently, and the failure is described
   there.
2. Run its scripts from the root of this repository, in the paths the `SKILL.md` shows.
3. Report what actually happened, including the parts it says were never verified here.
   A skill that was exercised on one platform says so, and saying so is the skill
   working.

## Conventions

- **English.** Code, comments, commits and documentation here are written in English.
  Reply to the person in whatever language they are using.
- **A skill earns its place** by the failure it prevents. If the task can be done from
  the documentation of a tool, it is not a skill.
- **Keep the code in the skill's own directory**, and keep it dependency-free: a skill
  that needs an install is a skill that fails on a machine with no network.
- **State what was verified, when, on which platform**, and what was not. A procedure
  that claims more than it tested is the thing this repository is against.
- **Commits** are Conventional Commits, imperative, lowercase, no trailing period, under
  72 characters: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, optionally scoped —
  `feat(chrome-cdp): …`.
- **Never commit a key or a token.** Nothing here should need one.
