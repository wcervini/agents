---
description: Group changes into semantic commits and push
---

Group all currents changes into meaningful semantic commits and push the current branch

Optional context for commit messages: `$ARGUMENTS`

## Rules:

- First inspect the full repository state;
  - `git status --short`
  - `git diff --stat`
  - `git diff`
  - `git log --oneline -10`
- Identify related files groups by intent: feature, fix, refactos, tests, docs, chore, release or config
- Create multiple commits where there are independent changes. Do not mix unrelated changes into the same commit.
- If `$ARGUMENTS` is not empty, use it as context to adjust commit messages, but not force tha text if it dos not accuratelly describe the changes.
- Use clear semantic, concise commit messages tha follow the repos recent style.
- Before commiting, check for sensivity o suspicious files (`.env`, tokens, credentials, secrets). If any appear, stop and ask
- Include nre modified and deleted files that belong to each group.
- Do no revert existing changes.
- Do not use `no-verify`
- Do not force push.

## Workflow:

1. Show the proposed commit plan whith the files included in each commit
2. If the grouping is clear, continue. If there is real ambiguity, ask before commiting.
3. For each group:
   - Add only the files for thar group with `git add <files>`
   - Create the commit with semmantic messages
4. Once all commit have been created, run:
   `git push`
5. When finished, summarize the commits created and and the commits was pushed.:w
