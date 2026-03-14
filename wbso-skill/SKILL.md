name: wbso-skill
description: Use when you need a fully AI-driven WBSO bookkeeping workflow for one assignee: collect inputs, inspect GitHub PRs with gh, inspect and update Linear issues with the `linear` CLI, present a dry run for review, then apply the approved changes and output paste-ready daily booking rows.
---

# WBSO Booking

This skill is instruction-only. Do not use helper scripts. Do not create code or tests as part of the bookkeeping workflow unless the user explicitly asks for automation.

## Core tools

- Use `gh` for GitHub.
- Use `linear` for Linear.
- Prefer JSON output where possible.
- The AI should do the analysis, reasoning, dry run summary, and final apply steps itself.

## Repo-specific rules

Use the default workflow unless a repository is called out here.

### MatchWornShirt/monorepo

- Sync the concrete TB label between Linear and the GitHub PR.
- If a linked Linear issue already has a concrete TB label, Linear is the source of truth.
- When linking a PR to a Linear issue, the PR body should use a `references ISSUE-123` line.

### 360ERP/mws-ftg

- Do not add GitHub WBSO / TB labels.
- Still reconcile every PR against Linear in `OPS` or `BAC`.
- If the user only asks for ticket linking for this repo, do not add or change TB labels in Linear unless they explicitly ask for that as well.
- If a PR needs a ticket link, put the full Linear issue URL in the PR body instead of a `references ISSUE-123` line.
- If the PR title contains a key like `[mws-1747]`, treat that as a strong hint for the matching `OPS-1747` or `BAC-545` issue and verify it before creating anything new.

## Run pattern

Follow this exact sequence:

1. Ask for any missing run inputs:
   - start date
   - end date
   - repositories
   - GitHub assignee
   - Linear assignee name if different
   - Linear team
   - optional Linear project
   - expected total hours or per-milestone targets
   - holidays, PTO, and any other blocked dates with no work logged
   - optional other excluded dates
2. Verify access:
   - `gh auth status`
   - `linear --version`
3. Collect PR data from GitHub.
4. Reconcile each PR against Linear.
5. Infer missing WBSO milestones when needed.
6. Produce a dry run for user review.
7. Wait for user corrections or approval.
8. Apply approved Linear changes with `linear`.
9. Output final paste-ready booking rows.

## Execution checklist

Use this order during a real run:

1. collect the date range, repositories, assignee, Linear team/project, expected hours, and holidays / PTO / blocked dates
2. verify `gh` and `linear` access
3. gather PRs and inspect linked Linear issues
4. produce a dry run with milestone proposals, duplicate notes, and booking output
5. resolve duplicate candidates before creating any new Linear issues
6. sync linked issues and PR labels where the final mapping is already clear
7. create only the remaining missing Linear issues
8. backfill PR bodies to the final approved Linear issue reference format for that repo
9. generate the final booking rows with holidays and other excluded dates left blank

## GitHub collection

For each repository, fetch merged PRs for the assignee in the requested range.

Use:

```bash
gh pr list --repo OWNER/REPO --state merged --author ASSIGNEE \
  --search 'merged:YYYY-MM-DD..YYYY-MM-DD author:ASSIGNEE' \
  --limit 200 \
  --json number,title,body,url,headRefName,mergedAt,labels,author
```

For each PR, fetch changed files:

```bash
gh pr view PR_NUMBER --repo OWNER/REPO --json files
```

## Linear reconciliation

Detect the Linear issue key:

- First from the PR description/body
- Otherwise from the branch name
- If both exist and disagree, trust the PR description

Inspect linked issues with:

```bash
linear issue view ISSUE-123 --json
```

Use these rules:

- If a linked Linear issue already has a concrete WBSO label, that Linear WBSO label is the source of truth and GitHub should be synced to match it.
- If the linked Linear issue has no concrete WBSO label, use the GitHub PR WBSO label if present.
- If neither side has a concrete WBSO label, infer the best milestone from:
  - PR title
  - PR body
  - branch name
  - changed files
  - linked Linear issue title/description/labels if available
- If the inference is ambiguous, do not apply automatically. Present it in the dry run and ask the user to decide.
- Before creating a new Linear issue, search likely duplicates in both `BAC` and `OPS`.
- If an existing issue is clearly the same workstream, reuse it instead of creating a duplicate ticket.
- One Linear issue may legitimately correspond to multiple PRs; in that case sync all relevant PRs to that existing issue.
- For repositories that use title hints like `[mws-1747]`, prefer reusing the matching existing issue if it exists and the PR content is consistent with that issue.

## Label handling

- Use the exact concrete TB label names that exist in GitHub and Linear, not just the shortened milestone names.
- In Linear, do not assign the parent/group label `WBSO`; only assign the concrete child TB label.
- Preserve existing non-WBSO labels on Linear issues, for example `Bug`.
- When syncing from Linear to GitHub, mirror the concrete TB label only.

## WBSO milestones

Use exactly these milestone names:

- `00 - Bucket`
- `13 - Identity and Access Management (IAM)`
- `18 - Products and Product Listing Split`
- `19 - Scalability and Performance: IaC`
- `22 - Mix-bid function`
- `23 - Frame the Game configurator / FTG.com relaunch`
- `24a - Fabricks Service - logging`
- `24b - Fabricks Service - automation of steps`
- `25 - Middleware for search rank boosting`
- `26 - Buy now function`
- `27 - Search and Filtering`
- `28 - Personalized recommendations`
- `29 - Real time sales tax - Global marketplace`
- `30 - Multiple storefronts & data isolation`
- `31 - 360 image processing`
- `32 - Easy bidding`
- `33 - Operational Insights`
- `34 - Realtime reservation mechanism`
- `35 - LLM-driven data access`

## Dry-run output

The AI should present a concise review table or bullet list per PR with:

- repo and PR number
- PR title
- detected linked Linear issue or missing-link status
- detected or inferred WBSO milestone
- source of truth for that milestone: `linear`, `github`, or `inferred`
- rationale for inferred milestones
- proposed action:
  - reuse existing linked issue
  - update linked issue tag/assignment/state
  - create new issue
- explicit review markers for anything ambiguous

Also include:

- milestone totals
- the final day-by-day booking rows in paste-ready format
- any holidays / PTO / blocked dates that were excluded from booking output

## Apply commands

Only run these after the user approves the dry run.

### If no linked Linear issue exists

Create the issue:

```bash
linear issue create \
  --title "PR TITLE" \
  --team TEAM \
  --assignee "ASSIGNEE NAME" \
  --state Done \
  --label "WBSO MILESTONE" \
  --project "PROJECT NAME" \
  --description-file /tmp/wbso-issue-description.md
```

Skip `--project` if no Linear project was provided.

Use a markdown description file that reads like a normal issue description derived from the PR itself:

- summarize the change in plain product/engineering language
- optionally include a short implementation bullet list based on the PR title, body, and changed files
- include the GitHub PR URL only if linking back is useful for normal team workflow
- do not mention AI, WBSO bookkeeping, or that the ticket was created retroactively

If you need to adjust the issue after creation, use:

```bash
linear issue update ISSUE-123 --assignee "ASSIGNEE NAME"
linear issue update ISSUE-123 --state Done
linear issue update ISSUE-123 --label "WBSO MILESTONE"
linear issue update ISSUE-123 --project "PROJECT NAME"
```

Skip the `--project` update if no Linear project was provided.

### If a linked Linear issue already exists

Update it:

```bash
linear issue update ISSUE-123 --assignee "ASSIGNEE NAME"
linear issue update ISSUE-123 --state Done
linear issue update ISSUE-123 --label "WBSO MILESTONE"
```

Before applying, confirm the exact flags on the local version if needed:

```bash
linear issue create --help
linear issue update --help
linear issue comment add --help
```

## PR backfill rules

- When a PR is reused against an existing Linear issue or a new ticket is created, update the PR body so it points to the final Linear issue.
- For `MatchWornShirt/monorepo`, use `references ISSUE-123`.
- For `360ERP/mws-ftg`, use the full Linear issue URL.
- Handle templates that start with extra lines such as `...` before the `references ...` line.
- If the PR still contains an outdated ticket reference, replace it with the approved final issue reference.
- Preserve the rest of the PR body whenever possible.

## Booking rules

Generate one output line per calendar day in order from start date to end date.

- Working day format: `hours<TAB>milestone`
- Weekend / Dutch public holiday / excluded date: blank line

Keep the allocation believable:

- average about 20 hours per week
- never more than 8 hours in one day
- no weekend bookings
- no Dutch public holiday bookings
- place hours near the dates of relevant PR activity
- keep totals close to the expected target
- if multiple milestones are active in the same period, spread them naturally

## AI behavior expectations

- The AI should reason from the live PR and Linear data, not from a precomputed script output.
- The AI should show its proposed actions before mutating Linear.
- The AI should let the user correct milestone mappings, ticket choices, and booking distribution before apply.
- The AI should only execute the approved `linear` commands after confirmation.
- When creating tickets, the AI should write descriptions that look like normal teammate-written Linear issues rather than administrative backfills.
- Before generating booking rows, the AI should explicitly ask for holidays, PTO, and other no-work dates so those days stay blank.
- The AI should start small when possible: test one PR, let the user inspect it, then continue with larger batches.
