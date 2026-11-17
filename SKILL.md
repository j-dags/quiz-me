---
name: quiz-me
description: Turns a code change (diff, commit, branch, PR) or a whole package, repo, or folder into a short interactive HTML page — the 3-4 ideas worth knowing, then a quiz whose answers link back to the passage that explains them. Use when the user runs /quiz-me or asks to be quizzed on a change or a codebase. For plain terminal explanations of a diff (summary, expert review map), use explain-diff instead.
metadata:
  author: j-dags (based on chrisgraffagnino/explain-diff)
  version: 0.2.0
---

# Quiz Me

Teach the few things worth knowing about some code, then check that they stuck. The output is one HTML page: about a 5-minute read, then a quiz.

Two rules drive everything:

- **Selectivity.** A bigger target does not get a longer page; it gets a pickier choice of what to teach.
- **Altitude.** Start with the big picture (what it is for, what a person sees) and go down to how the code works only when the higher levels are covered and the detail is genuinely needed.

## Step 0: Help

If the arguments are exactly `help`, `--help`, `-h`, or `?`, print this and stop:

```
quiz-me — learn some code, then get quizzed on it

Usage: /quiz-me [target]

Writes /tmp/YYYY-MM-DD-quiz-me-<slug>.html: the 3-4 key ideas (~5 min
read), then a 3-5 question quiz. After answering, each question links to
the passage that explains it.

Target (optional, defaults to your working diff):
  <empty>         Branch diff vs upstream/main, plus uncommitted changes
  <PR number>     e.g. 4821, fetched via `gh pr diff`
  <git range>     e.g. main..HEAD (use <sha>~1..<sha> for one commit)
  <commit>        e.g. HEAD~1, or a commit SHA
  <branch>        compared against the repo's default branch
  <path>          limit a diff to a file or directory
  this package    explain what the project IS, not what changed
  (also: this repo / this project / this codebase / a folder with no diff)

For a terminal summary or an expert review map of a diff, use /explain-diff.
```

## Step 1: Pick the framing

Choose one, and say which in the terminal reply if it was not obvious.

- **Change framing**: there is a real diff to explain (a PR, range, commit, branch, or uncommitted changes in a git repo). The question is "what changed, and why?"
- **Overview framing**: the user asks about "this package / repo / project / codebase / folder", the target is not a git repo, or the path has no diff. The question is "what is this?" Do not invent a "before" (such as a framework starter or scaffold) to diff against.

If a git repo has no diff and no explicit target, say so and use overview framing.

## Step 2: Gather the material

**Change framing:**

- No target: `git diff @{upstream}...HEAD` (fall back to `git diff main...HEAD`, then `git diff HEAD~1`). If there are uncommitted changes, or the range diff is empty, also include `git diff HEAD`.
- PR number: `gh pr view <n>` for title/description, `gh pr diff <n>` for the diff.
- Range / commit / branch: `git diff <range>`; a branch is diffed against the default branch. Read the commit messages (`git log`) for stated intent.
- Path: restrict the commands above to that path.

**Overview framing:**

- Read the README, package manifest (`package.json`, `pyproject.toml`, etc.), and entry points first. Skip generated files, lockfiles, `node_modules`, build output, and untouched framework boilerplate.
- Read the project's own source files, prioritizing whatever a user of the thing sees or touches.
- If the user's agent memory or project notes describe the project's purpose, use them, and mark it as stated intent.

## Step 3: Understand

1. Change framing: read every hunk, then the enclosing function or component; behavior often lives in the unchanged lines around a change. Read stated intent separately from what you infer.
2. Overview framing: work out what the thing is for, who uses it, what they see or do, and how the code is laid out.
3. Look at callers and tests only as far as the key ideas need.
4. Change framing only: skim `references/analysis-lenses.md` and apply the lenses that obviously fire (especially removed behavior and cross-file impact). You need at most 3 watch-outs; stop when you have the real ones.

Scale effort to the target: a 20-line change or a 3-file project needs minutes.

## Step 4: Choose the key ideas, top down

Pick **3 key ideas, 4 at most**; fewer if the target is small. Do not pad.

Candidates come in four levels. Fill from the top; move down a level only when everything important above it is covered and a slot is left.

| Level | Overview framing | Change framing |
|---|---|---|
| 1. Purpose | What it is and what it's for; why it exists | Why the change was made; the problem it solves |
| 2. What a person sees | What a user or visitor sees and can do | User-, API-, or data-visible behavior, before → after |
| 3. Where things live | How the code is organized; which file to open to change what | Which parts of the system the change touches and how they connect |
| 4. How it works | An implementation mechanism | A mechanism the change relies on, or a design choice where an obvious alternative existed |

Level 4 ideas earn a slot only if a reader would get something important wrong without them. Implementation details such as animation technique, styling, design tokens, dark mode, accessibility fallbacks, config flags, and framework boilerplate are almost always level 4. Mention them in one line in the map, or not at all, unless the target is specifically about them.

Check: if someone asked a colleague "what is this?" (overview) or "what did this change do?" (change), would the colleague's answer cover your key ideas? If your ideas would be the third or fourth thing the colleague mentions, go up a level.

## Step 5: Write the quiz before the page

For each key idea, write 1-2 questions; 3-5 questions total.

- 4 options each. Wrong options are plausible misreadings, never jokes or trivia.
- Medium difficulty: answerable by someone who read the key idea, not by guessing from the wording.
- For every option, one or two sentences on why it is right or wrong.
- Each question names its **source passage**: the key-idea paragraph or code block that answers it.
- Questions follow the altitude of the key ideas: "what is this site for?" or "where would you change the contact link?" rather than CSS trivia.

Then write the page so every question is answerable from its source passage. Anything on the page that no question and no key idea depends on is a candidate to cut.

## Step 6: Write the page

| Section | Content | Budget |
|---|---|---|
| Title | Overview: what the thing is, plainly ("A placeholder personal website"). Change: the effect of the change ("Cache keys become tenant-scoped"), not its mechanics | 1 line |
| What and why | Overview: what it is, who it's for, its current status. Change: the observable outcome and its stated or inferred intent | 2-3 sentences |
| Map | Overview: the files or folders that matter, one line each (what lives there). Change: one line per theme, **theme** (`files`) what changed | 2-6 lines |
| Key ideas | One subsection per key idea: the idea in plain words, a concrete example, and a real code snippet (a few lines, with `file:line`) only when the code makes the idea clearer | ~150 words each, excluding code |
| Watch out for | Change framing only: `file:line`: what could break, under what conditions. "None" is valid. Omit this section in overview framing unless something is clearly broken | max 3 one-liners |
| Quiz | The questions from Step 5 | 3-5 questions |

**Total prose budget: about 1,000 words**, excluding code blocks and the quiz. It does not grow with the size of the target.

Writing rules:

- Plain, short sentences. No preamble, no "in this walkthrough we will", no recap at the end.
- Define a term only where a key idea needs it, in one clause, in place. No general framework background.
- Quote real code; every claim must survive the reader opening the file at the cited line.
- Distinguish "the README / PR says" from "this appears to".
- Never present a suspicion as a confirmed bug. If a watch-out looks serious, suggest a dedicated code-review pass.
- If there was more going on than fits, add one final line under the map: "Not covered: …".

## Step 7: HTML page

Write a single self-contained HTML file (inline CSS and vanilla JS, no external resources) to `/tmp/YYYY-MM-DD-quiz-me-<slug>.html`, using today's date and a short kebab-case slug of the title. Desktop only; no responsive layout needed.

- **Layout:** sticky left-sidebar table of contents linking to each section, highlighting the section in view. Content in a readable column (~760px).
- **Code:** `<pre>` blocks. If you style a `div` as code instead, it must have `white-space: pre` or `pre-wrap`.
- **Diagram:** optional, at most one, and only if it makes a key idea clearer than prose (a file map for overview framing often qualifies). Build it from styled HTML boxes, with concrete example data on any arrows. Never ASCII art.
- **Quiz** (at the end of the page):
  - Clicking an option locks the question and marks the chosen and correct options with text labels ("Your answer", "Correct answer") as well as color, then shows the explanation for the chosen option, plus why the correct one is right if it was wrong.
  - Only after answering, show a link like "See where this is explained: Key idea 2". Never before, or the quiz becomes a lookup.
  - The link scrolls to the exact source passage (its own `id`), not the section heading, and highlights it for about 2 seconds, then fades.
  - After following a link, a floating "Back to question N" button in the bottom-right returns to that question (answer still showing), then hides.
- **Before saving**, check: every quiz source link points at an `id` that exists; every code block preserves whitespace; prose is within budget (count words outside `<pre>` and the quiz; if well over 1,000, cut before saving).

## Step 8: Terminal reply

Do **not** print the explanation in the terminal. The page is the deliverable. Reply with only:

1. The one-line title.
2. A scope note if needed (e.g. "Not a git repo, so this is an overview of the project rather than a change").
3. The page as a clickable `file:///tmp/...html` link, as the **last line**.

If writing the file fails, say so and print the page content as markdown instead (quiz answers in `<details>` blocks).
