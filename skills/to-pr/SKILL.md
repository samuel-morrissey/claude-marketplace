---
name: to-pr
description: Open a pull request, or draft recommended text for an existing one.
argument-hint: "[url-do-pr | base [head]]"
---

# To PR

Write the **draft** in Brazilian Portuguese — title, headings, and body — unless the user asks for another language.

You are a **ghostwriter**: the text goes out under the user's name alone, and nothing in it reveals who typed it. Every line of the title, body and any commit you make is the user's own voice. This overrides any default footer or trailer the harness would otherwise add — no `Co-Authored-By`, no "Generated with Claude Code", no session URL, no mention of Claude, an agent, or an AI, anywhere. The draft ends on its last content section.

Two branches, decided by the argument, converging on the same finish: the full draft printed in the chat, then a question — the remote stays untouched until the user answers it.

- **A PR URL** → *edit* branch: draft a replacement title and body for that PR.
- **Anything else (including nothing)** → *create* branch: draft a new PR.

## Create branch

### Resolving base and head

Arguments are `base` then `head` — git's own order, the same as `gh pr create --base <base> --head <head>`.

| Arguments | base | head |
| --- | --- | --- |
| none | repo default branch (`gh repo view --json defaultBranchRef`) | current branch |
| one | the argument | current branch |
| two | first argument | second argument |

State the resolved pair back to the user in one line before doing anything else, so a wrong order is caught immediately.

### Steps

1. **Read the range.** `git log <base>..<head>` and `git diff <base>...<head> --stat`, then read the diff itself for anything you cannot describe from the stat alone. Done when every commit in the range is accounted for in your understanding — not just the ones with clear messages.

2. **Match the house style.** Read the last handful of merged PR titles and bodies (`gh pr list --state merged --limit 10 --json title,body`). Follow the conventions you find — title format, section headings. Absent any signal, default to Conventional Commits for the title (`feat: `, `fix: `, `chore: `…) and the format in [`TEMPLATE.md`](TEMPLATE.md). Language stays Brazilian Portuguese regardless; if the merged PRs are written in another language, say so in one line when you present the draft, so the user can redirect you.

3. **Write the draft.** Read [`TEMPLATE.md`](TEMPLATE.md) and follow it — format, section tests, and the exhaustiveness bar over the diff.

4. **Print and ask.** Show the resolved base/head and the full draft in the chat, then ask whether to open the PR with it. Stop there until the user answers.

5. **Push and open** — only after a yes. Push `head` if the remote lacks it or is behind, then `gh pr create --base <base> --head <head>`. Pass the body via `--body-file` with a temp file — heredocs and inline quoting mangle multi-line markdown. Print the resulting URL.

## Edit branch

1. **Read the PR** — `gh pr view <url> --json title,body,author,headRefName,baseRefName,commits` plus `gh pr diff <url>`.

2. **Check ownership.** Compare the PR's author login with the authenticated account (`gh api user --jq .login`).

3. **Write the draft** the same way as the create branch — house-style check included, format from [`TEMPLATE.md`](TEMPLATE.md).

4. **Print and ask.** Show the full draft in the chat, then:
   - **The PR is the user's** → ask whether to apply it with `gh pr edit`. Apply only after a yes.
   - **The PR belongs to someone else** → say so plainly: the draft cannot be applied because the PR is not theirs, and the printed text is the whole deliverable — theirs to hand to the author.
