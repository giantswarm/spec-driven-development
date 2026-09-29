# Setting up your plans repo

Ten minutes, most of it renaming. Work through it top to bottom — or paste this file to
your agent and ask it to do the lot.

## 1. Create the repo

Fork this repo on GitHub (or clone it and push to a fresh repo). Name it after
the team that owns it — `<team>-plans` works well.

Note: a *fork* of a private repo can never be made public. If your plans repo might go
public one day, create it from the template rather than forking.

## 2. Check the symlinks survived

```sh
ls -l .claude/skills/
```

Each entry should be a symlink into `../../.agents/skills/`. If your platform flattened
them into copies (Windows without developer mode, or a zip download), recreate them:

```sh
for s in .agents/skills/*/; do
  n=$(basename "$s")
  ln -sfn "../../.agents/skills/$n" ".claude/skills/$n"
  ln -sfn "../../.agents/skills/$n" ".cursor/skills/$n"
done
```

Delete `.cursor/` if nobody on the team uses Cursor, and `.claude/` if nobody uses Claude
Code. `.agents/` is the source of truth either way.

## 3. Make it yours

- **`README.md`** — replace the first paragraph with what your team plans here.
- **`CHANGELOG.md`** — clear it and start your own history.
- **`CODEOWNERS`** — create one if your org uses them: `* @your-org/your-team`.
- **`context/glossary.md`** — seed it with three or four terms your team argues about
  most. `/grill-me` grows it from there.
- **`example-plan/`** — read it once, then delete it (and the "Example plan terms" section of `context/glossary.md`). It is there to show the shape.

## 4. Decide how issues get published

`/to-issues` is optional and GitHub-shaped. If your team tracks work elsewhere — a project
board, Linear, Jira — `.agents/skills/to-issues/SKILL.md` is the one file to rewrite.
Nothing else in the workflow depends on how issues are created. If your team works straight
from the spec, delete the skill and the step disappears.

## 5. Check your agent picks the skills up

Open the repo in your agent and run `/plan`. If the skill isn't found:

- **Claude Code** reads `.claude/skills/` — check the symlinks resolve.
- **Codex and skills.sh-based agents** read `.agents/skills/` directly.
- **Cursor** reads `.cursor/skills/`.

## 6. Protect `main`

Require a pull request. Reviewing the plan is the whole point; a plan that lands without
review defeats it.

## 7. Optional: publish the plan websites

Every plan renders to a self-contained `index.html`. Turning on GitHub Pages for the repo
gives reviewers a browsable URL per plan instead of a raw-HTML download. The pages need no
build step.
