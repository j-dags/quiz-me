# quiz-me

A [`SKILL.md`](https://agentskills.io) agent skill that turns code into a short interactive HTML page: the 3-4 ideas worth knowing, then a quiz whose answers link back to the passage that explains them.

Point it at a change (diff, PR, commit, branch) or at a whole project ("this package") for an overview. It picks what to teach big picture first (what the code is for, what a person sees) and only goes down to implementation detail when it's actually needed. A bigger target gets a pickier selection, not a longer page.

## Usage

```
/quiz-me [target]
```

| Target | Meaning |
| --- | --- |
| *(empty)* | Branch diff vs upstream/main, plus uncommitted changes |
| `<PR number>` | e.g. `4821`, fetched via `gh pr diff` |
| `<git range>` | e.g. `main..HEAD` (use `<sha>~1..<sha>` for one commit) |
| `<commit>` | e.g. `HEAD~1`, or a commit SHA |
| `<branch>` | compared against the repo's default branch |
| `<path>` | limit a diff to a file or directory |
| `this package` | explain what the project *is*, not what changed (also: this repo / this project / this codebase) |

Output goes to `/tmp/YYYY-MM-DD-quiz-me-<slug>.html` (about a 5-minute read plus a 3-5 question quiz). The terminal shows only the link.

## Install

Clone the repo and symlink it into your skills folder:

```sh
git clone https://github.com/j-dags/quiz-me.git
ln -s "$PWD/quiz-me" ~/.claude/skills/quiz-me
```

## Credits

Based on [explain-diff](https://github.com/Chris-Graffagnino/explain-diff) by Chris Graffagnino; `references/analysis-lenses.md` comes from that project. MIT licensed.
