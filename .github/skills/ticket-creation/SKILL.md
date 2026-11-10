---
name: ticket-creation
description: "Use when the user wants to draft or create a GitHub issue, ticket, feature, task, bugfix, or enhancement for the current repository."
---

## When to use the skill

- When the user wants to create a GitHub issue for the current repository.

## GitHub integration

- Use the available GitHub issue-listing and issue-creation tool when one is provided.
- If no GitHub integration is available, use the authenticated `gh` CLI when it is available.
- Confirm that the target repository is the current repository before listing or creating issues.
- Never ask the user to provide a personal access token in chat.
- If neither a GitHub integration nor an authenticated `gh` CLI is available, draft the issue and tell the user that manual creation is required.

## Core Process

### Phase 1. Identifying the issue

1. Ask the user to select one category:
   - `Feature`
   - `Task`
   - `Bugfix`
   - `Enhancement`
2. Ask the user what the ticket is for. If it is vague or overly scoped, ask more questions.
   Example:
   - User: Create a button (Wrong)
   - User: Create a button in the hero section that will link to the merchant's website. (Correct)
3. For tech specs, possible implementation and test cases, create recommendation for the user. Ask the user if this is valid or not.
4. For references, ask the user for the links or files for the references.

### Phase 2. Preparing github issue

1. Inspect the existing GitHub issues for similar cases. If a ticket addresses the same or a nearly identical issue, report it to the user and ask whether to continue. If continuing, find the highest number already used for that category. Use the next number as the ticket number. Numbering starts at `0` when the category has no existing tickets. Treat this number as a naming convention, not a guaranteed unique identifier when multiple users create tickets concurrently.
2. Draft the issue title in this exact format:
   `[Category-Number]: Short descriptive title`

   Use the category with its first letter capitalized, for example: `[Task-0]: Create a button`.

3. Draft the issue body with exactly these sections, in this order:

   ## General description and requirements

   ## Tech Specs

   ## Possible implementation

   ## Test cases

   ## References

4. Show the complete title and body to the user for review. Do not create the GitHub issue yet.
5. Ask explicitly: `Do you approve creating this GitHub issue?`

### Phase 3. Create GitHub issue

1. Once approved, verify that the final title and body have not changed.
2. Create the issue in the current repository using the available GitHub integration or authenticated `gh` CLI.
3. Report the created issue URL and number.
4. If creation fails, do not retry blindly. Report the error and leave the approved draft unchanged.

## Requirements

- Keep the title concise and specific.
- Use the selected category consistently in the title and numbering lookup.
- Do not invent technical details. Mark unknown information as `TBD` or ask the user for it.
- Preserve the five required body sections even when a section has no information yet.
- Never create, edit, or submit a GitHub issue before the user has reviewed the full draft and explicitly approved it. Treat anything other than an explicit approval as a refusal or request for changes, and revise the draft instead.
