---
name: github
description: >-
  How to use the GitHub connector's tools — finding repositories, reading repos,
  READMEs, branches, and commits, and creating and managing issues, pull requests,
  reviews, comments, labels, assignees, and file contents. Use when the user works
  with GitHub, repos, issues, PRs, pull requests, code reviews, branches, commits,
  or wants to open an issue or PR.
---

You are using the **GitHub** connector (via Composio). Its tools act on the
connected user's GitHub account. Apply this guidance whenever you touch GitHub.

## Almost every call needs owner + repo

- Most tools take an **`owner`** (account or org) and **`repo`** (name, no `.git`)
  pair to target a repository — confirm both before acting. Issue/PR/review tools
  additionally take the **issue or PR number** (`issue_number` / `pull_number`).
- Don't guess these. If the user gives a `owner/repo` shorthand or a GitHub URL,
  split it into `owner` and `repo`. If you only have a fuzzy name, **find first**.

## Find before you act

- Use the **find-repositories** tool to resolve a vague reference into a concrete
  `owner/repo`, and **find-pull-requests** to locate PRs. These use **GitHub search
  query syntax**, not plain keywords — qualifiers like `language:`, `stars:>100`,
  `org:`, `user:`, `pushed:>YYYY-MM-DD`, `in:name,description,readme` for repos, and
  `repo:owner/name`, `state:open`, `author:`, `label:`, `is:pr` for PR/issue search.
- Issue vs PR search are **separate** in GitHub: a search must carry `is:issue` or
  `is:pull-request` — a query with neither returns nothing useful (the API rejects
  the mix). Don't expect one query to span both.
- Search has hard limits: queries over ~256 chars (excluding qualifiers) or with
  more than five `AND`/`OR`/`NOT` operators fail validation. Keep them tight.

## Reading

- To understand a repo, you typically chain: get-repository (metadata, default
  branch), get-readme (overview), get-branch / get-commit for state. The README and
  file-content reads may return **base64-encoded** content — decode before showing.

## Writing — issues, PRs, comments

- **Create-issue** needs `owner`, `repo`, `title` (body/labels/assignees optional).
  Adding labels or assignees to an existing issue is a **separate** call against its
  number — labels/assignees must already exist on the repo or org.
- **Create-pull-request** needs the `head` branch (source) and `base` branch
  (target) — both must already exist on the repo; the tool does not create branches.
  Order matters: head → base, not the reverse.
- **Create-a-review** lets you `APPROVE`, `REQUEST_CHANGES`, or `COMMENT` on a PR —
  this is a real, visible review event, not a draft. Comment on an issue/PR via the
  issue-comment tool (PRs are issues for commenting purposes).

## Editing files (read-modify-write)

- Create-or-update-file-contents writes **one file per call**. Content must be a
  **base64-encoded** string, and you pass a `message` (commit message) and usually a
  `branch`. To **update** an existing file you must supply its current blob **`sha`**
  (read the file first to get it); omitting `sha` only works for a brand-new file and
  otherwise fails with a conflict. This is not a git push — it's a single committed
  change.

## Care

- Creating issues, PRs, reviews, and comments is **public and immediately visible**
  to the repo's watchers and notifies people. Confirm the target `owner/repo`,
  the content, and the audience before posting; never open issues/PRs in bulk or
  against the wrong repo.
- Submitting a review (especially `APPROVE` / `REQUEST_CHANGES`) carries weight on
  someone else's PR — only do it when the user clearly intends to review.
- Writes commit to a branch and change history; prefer a feature branch over
  committing straight to `main`/`master` unless the user says otherwise.
- Permissions are scoped to the connection: a "not found" repo may simply be private
  or outside the connector's access rather than nonexistent — say so instead of
  inventing it. Mind rate limits on large scans; page and summarize rather than
  fetching everything.
