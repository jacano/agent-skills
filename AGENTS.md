# AGENTS.md — agent-skills

One directory per skill: a `SKILL.md` (the procedure and the failure it prevents) and the
scripts that carry it out. No packages; state lives in the system temp directory, so a clone
anywhere works. [`README.md`](README.md) lists the skills.

## Picking one up

Read its `SKILL.md` in full before running anything, run its scripts from the root of this
repository, and report what happened — including the parts it says were never tested.

## Conventions

- **English**, everywhere: code, comments, commits, documentation. Reply to the person in
  their language.
- **A skill earns its place** by the failure it prevents; what a tool's own documentation
  covers is not a skill.
- **Keep the code in the skill's directory, with nothing to install**: a skill that needs a
  package is a skill that fails on a machine with no network.
- **Say what was tested, when and where, and what was not.**
- **Commits** are Conventional Commits: imperative, lowercase, no trailing period, under 72
  characters (`feat(chrome-cdp): …`).
