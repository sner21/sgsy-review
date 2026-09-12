# sgsy-review

A [Hermes Agent](https://hermes-agent.nousresearch.com/docs) skill: **pre-commit code verification**.

Review pipeline before code lands — a static security scan, a self-review checklist, and an
independent reviewer subagent that returns a pass/fail verdict.

**Core principle:** no agent should verify its own work. Fresh context finds what you miss.

## What it does

1. **Get the diff** — `git diff --cached`, falling back to `git diff` / `git diff HEAD~1 HEAD`; splits diffs over 15k chars by file.
2. **Static security scan** — greps added lines only for hardcoded secrets, shell injection, `eval()`/`exec()`, `pickle.loads()`, and SQL string-formatting.
3. **Self-review checklist** — secrets, input validation, parameterized queries, path traversal, error handling.
4. **Independent reviewer subagent** — `delegate_task` with the diff and scan results only, no shared context with the implementer. Fail-closed: unparseable output = fail, and any security concern or logic error forces `passed: false`.
5. **Verdict** — combined report. It does **not** commit and does **not** auto-fix; the verdict is the deliverable.

## Install

Hermes loads skills from its skills directory. Drop this folder in as
`<hermes>/skills/software-development/sgsy-review/`:

```bash
git clone https://github.com/sner21/sgsy-review.git \
  "$HOME/hermes/skills/software-development/sgsy-review"
```

On Windows (this repo's origin machine) the skills root is `H:\hermes\skills`, so:

```bash
git clone https://github.com/sner21/sgsy-review.git "H:/hermes/skills/software-development/sgsy-review"
```

Then invoke it with `sgsy-review`, or just ask for a review before committing — the skill
triggers on `commit`, `push`, `ship`, `done`, `verify`, `review before merge`.

## When to use

- After a feature or bug fix, before `git commit` / `git push`
- After any task with 2+ file edits in a git repo
- As the quality gate after each task in subagent-driven development

**Skip for:** documentation-only changes, pure config tweaks, or when the user says
"skip verification".

## Relation to other skills

- **`github`** reviews *other people's* PRs on GitHub with inline comments. This skill verifies
  *your* changes before they land.
- **`test-driven-development`** — this pipeline does not run tests or linters; pair the two when
  you want those gates as well.

## Credits

Adapted from [obra/superpowers](https://github.com/obra/superpowers) and MorAlekss.

## License

MIT — see [LICENSE](LICENSE).
