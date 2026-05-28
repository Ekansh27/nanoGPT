# TKK-003 — Merge Conflict Walkthrough

This doc captures how I deliberately created a merge conflict, resolved it, and committed the fix.

## Setup

The target file was `README.md`, the same file modified in TKK-002. Line 9 originally read:

```
**TKK-002: change**
```

## Step 1 — Two branches off master, both editing the same line

From a clean `master`:

```bash
# Branch A
git checkout master
git checkout -b tkk-003-conflict-a
# edit line 9 → "**TKK-003 branch A:** this line was edited on the conflict-a branch."
git commit -am "TKK-003: edit README on branch A"

# Branch B
git checkout master
git checkout -b tkk-003-conflict-b
# edit line 9 → "**TKK-003 branch B:** this line was edited on the conflict-b branch — totally different content."
git commit -am "TKK-003: edit README on branch B"
```

Both branches now have one commit ahead of `master`, and both touch the exact same line of the same file.

## Step 2 — Trigger the conflict

```bash
git checkout tkk-003-conflict-a
git merge tkk-003-conflict-b
```

Git's output:

```
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

`README.md` was rewritten with conflict markers around the disputed lines:

```
<<<<<<< HEAD
**TKK-003 branch A:** this line was edited on the conflict-a branch.
=======
**TKK-003 branch B:** this line was edited on the conflict-b branch — totally different content.
>>>>>>> tkk-003-conflict-b
```

- `<<<<<<< HEAD` → version on the current branch (branch A).
- `=======` → divider.
- `>>>>>>> tkk-003-conflict-b` → version coming from the branch being merged in.

Git is saying: "I don't know which of these you want — pick one, blend them, or write something new."

## Step 3 — Resolve manually

I opened `README.md`, deleted the markers, and replaced the disputed lines with a single combined line:

```
**TKK-003:** this line had conflicting edits from branches A and B — resolved by keeping both ideas combined into a single note.
```

The three options at every conflict:

1. Keep "ours" (the HEAD/branch-A version).
2. Keep "theirs" (the incoming/branch-B version).
3. Write something new that combines or supersedes both. ← used here.

## Step 4 — Stage and commit the resolution

```bash
git add README.md
git commit -m "TKK-003: resolve merge conflict in README.md"
```

`git add` tells Git "this file is resolved, move on." Once every conflicted file is staged, `git commit` finalizes the merge — Git auto-generates a merge commit (two parents: branch A's tip and branch B's tip).

## Step 5 — Verify the merge graph

```bash
git log --oneline --graph -5
```

Output:

```
*   ec7ddc4 TKK-003: resolve merge conflict in README.md
|\
| * e111a7c TKK-003: edit README on branch B
* | cc13c91 TKK-003: edit README on branch A
|/
*   f36f1cd Merge pull request #2 from Ekansh27/tkk-002-modify-file
```

The "Y" shape confirms two branches diverged from a common base and were joined by a merge commit with two parents.

## Lessons

- A conflict only happens when two branches edit the **same lines** of the same file. Edits to different lines (or different files) merge automatically.
- Conflict markers are just plain text Git writes into the file — nothing magical. Resolving means editing the file like any other text edit.
- `git merge --abort` is the escape hatch — it backs out cleanly if the conflict is too messy to resolve on the spot.
- VSCode shows inline "Accept Current / Accept Incoming / Accept Both" buttons over each conflict, which is faster than editing markers by hand.
