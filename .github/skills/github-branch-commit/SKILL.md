---
name: github-branch-commit
description: Prepares a GitHub issue branch and commits changes with approval
---

You are an expert release engineer. Prepare a safe, reviewable Git branch and commit workflow for the user's GitHub issue.

## Issue discovery and verification

- Before asking the user for an issue URL, read the current branch with `git branch --show-current` and identify the current repository with authenticated `gh` CLI commands.
- Check whether that exact branch is linked to an issue in the current repository. Use the GitHub GraphQL API through `gh api graphql` to inspect issues' `linkedBranches` and match the linked branch ref name exactly. Paginate through the repository's issues and each `linkedBranches` connection as needed; do not infer a link from the branch name.
- The GraphQL query should request `repository.issues(first: 100, after: $cursor, states: [OPEN, CLOSED])`, including each issue's `number`, `title`, `url`, and `linkedBranches(first: 100) { nodes { ref { name } } pageInfo { hasNextPage endCursor } }`, plus the issues connection's `pageInfo`. Continue pagination until the exact branch is found or all issues have been checked.
- If exactly one issue links to the current branch, use it without asking the user for a URL. Verify it with `gh issue view <issue-number> --json title,number,url,state`.
- If no issue links to the current branch, ask the user for a valid GitHub issue URL, then verify it with `gh issue view <issue-url> --json title,number,url,state`.
- If multiple issues link to the current branch, ask the user which issue to use. Verify the selected issue with `gh issue view`.
- If branch lookup fails or cannot be completed because of an authentication, network, or API error, report the failure and ask the user for the issue URL; do not treat an incomplete lookup as proof that no issue is linked.
- If `gh` confirms the issue does not exist, stop and ask the user to create the issue first. Do not create it on the user's behalf.
- Read the issue title, category, and ticket number from the issue. Do not rely on a category or number supplied separately by the user.
- Supported categories are `Feature`, `Task`, `Bugfix`, and `Enhancement`.

## Workflow

1. Discover the issue linked to the current branch, or ask for an issue URL if none is linked, following the discovery and verification rules above.
2. Read the GitHub issue number and the issue title, category, and ticket number from the verified issue. The GitHub issue number used with `gh issue develop` is distinct from the category ticket number in the title.
3. Inspect the current repository state with read-only commands: `git status`, `git branch --show-current`, and the relevant staged and unstaged diffs.
4. Identify which changed files belong to the issue. Do not include unrelated files, generated files, secrets, or environment files.
5. If the user explicitly requested only a branch, continue when the working tree is clean even if there are no changes to commit. Skip commit proposals and commit approval; the explicit request authorizes creating the issue-linked branch. If the working tree is not clean, preserve its changes and ask before checking out another branch. If the user requested commits and there are no relevant changes, stop and tell the user that there is nothing to commit.
6. List branches already linked to the issue with `gh issue develop --list <github-issue-number>`. If the command fails, stop and report the failure. Do not reuse a linked branch name. If the current branch is linked to the issue, continue on it; otherwise propose a new branch.
7. For a new branch proposal, use this format:
   `<category>-<number>-<short-kebab-case-description>`

   Examples:
   - `task-0-create-button`
   - `feature-0-create-hero-section`
   - `bugfix-0-disable-button`
   - `enhancement-0-update-background`

8. Propose one or more present-tense commit messages. Each message must:
   - Start with `[Category-Number]`.
   - Describe the change inclusively and concisely.
   - Be no longer than 50 characters, including the prefix.
9. Use multiple commits only when the changes are logically independent. Explain the proposed grouping and list the files for each commit.
10. Show the user the issue reference, current branch, changed-file summary, the branch to use (current branch if linked, otherwise the proposed new branch name), commit breakdown, commit messages, and checks to be run.
11. Ask explicitly: `Do you approve creating this branch and these commit(s)?`
12. For requests that include commits, do not create a branch, stage files, or commit until the user gives clear approval. For branch-only requests, the explicit request authorizes creating the branch, but never stage or commit files.
13. After approval, run the relevant existing tests, lint, typecheck, and formatting checks when available. Also run `git diff --check`.
14. Report check results. If a check fails, stop and ask whether the user wants to fix it before committing.
15. After approval and successful checks, if the current branch is linked to the issue, keep using it and do not create or switch branches. Otherwise, create and check out the approved branch using `gh issue develop <github-issue-number> --name <approved-branch-name> --checkout`. This associates the branch with the GitHub issue. Do not substitute `git switch -c`; if `gh issue develop` fails, stop and report the error without creating an unlinked branch. If the requested name already exists remotely but is not linked, do not retry with that name or overwrite it; ask the user to link it through GitHub or approve a different new branch name.
16. Stage only the approved files and create only the approved commit(s) on the issue-linked branch.
17. Show the resulting branch and commit summary, and confirm the branch is linked to the issue. Do not push the branch or configure upstream tracking.

## Fast branch-only checks

For an explicit branch-only request, run these checks in order and avoid reading full diffs when the tree is clean:

1. `git branch --show-current`
2. Check exact current-branch linkage with authenticated `gh api graphql`; if linked, verify with `gh issue view <issue-number> --json title,number,url,state`. If no match, ask the user for a URL and verify it with `gh issue view <issue-url> --json title,number,url,state`.
3. `git status --short --branch`
4. `gh issue develop --list <github-issue-number>`
5. If the current branch is linked, leave it checked out. Otherwise, after confirming a clean tree, create and check out the approved issue-linked branch with `gh issue develop <github-issue-number> --name <branch-name> --checkout`.
6. Verify with `git status --short --branch && git branch --show-current`.

In the final response, list the commands actually run and report the issue, resulting branch, and verification result. Do not run tests or inspect diffs for a branch-only operation that changes no project files.

## Naming rules

- Branch names must be lowercase kebab case, related to the issue, and no longer than 50 characters.
- Commit messages must be present tense and no longer than 50 characters.
- Shorten the description when either limit would be exceeded; never silently violate the limit.

## Safety rules

- Never run `git reset`, `git rebase`, `git push`, `git clean`, `git stash`, `git commit --amend`, force commands, or commands that discard, rewrite, or silently move the user's work.
- Never overwrite, delete, or revert user changes.
- Never commit secrets, credentials, `.env` files, or unrelated changes.
- Create a remote branch only through the approved `gh issue develop` command so it is linked to the verified issue; never push a branch or configure upstream tracking.
- Never bypass hooks with `--no-verify`.
- If the working tree contains changes that cannot be safely separated, stop and ask the user how to proceed.
