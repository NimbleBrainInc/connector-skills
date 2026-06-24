---
name: linkedin
description: >-
  How to use the LinkedIn connector's tools — reading your profile/network,
  publishing posts and URL shares, commenting, fetching post content, pulling
  organization page and share analytics, and uploading images. Use when the user
  works with LinkedIn, posting to LinkedIn, a LinkedIn share or article, comments,
  reactions, followers, a company/organization page, or LinkedIn post stats.
---

You are using the **LinkedIn** connector (via Composio). Its tools act as the
connected LinkedIn account. Apply this guidance whenever you touch LinkedIn.

## Everything is a URN

- LinkedIn addresses objects by **URN**, not plain ids or @handles:
  `urn:li:person:XXXX`, `urn:li:organization:XXXX`, `urn:li:ugcPost:XXXX` /
  `urn:li:share:XXXX`, `urn:li:comment:(...)`, `urn:li:image:XXXX`. Tools expect
  the full URN — passing a bare numeric id or a profile URL will fail.
- You almost never know a URN up front. **Resolve it first:** call the
  get-my-info tool to obtain the authenticated **author URN** before posting, and
  use the person/company lookups to turn a name into a URN before reading.

## Posting

- A post needs an **author** URN and **commentary** (the text body, ≤ ~3000
  chars). Author can be the **person** (post as the user) or an **organization**
  URN (post as a company page) — confirm which identity the user means; posting to
  the wrong one is embarrassing and not undoable.
- **visibility** (PUBLIC vs CONNECTIONS) and **lifecycleState** default to
  PUBLISHED/PUBLIC — i.e. it goes live immediately to the chosen audience. There
  is no draft state on the API; treat every create call as "publish now."
- Sharing a link/article is a **separate tool** from a plain text post — use the
  article/URL-share tool when there's a URL to attach, not a text post with the
  link pasted in the body.
- **Images are multi-step:** initialize an upload to get an upload URL + image
  URN, push the bytes to that URL (out of band), then reference the returned
  **image URN** in the post. Don't expect a single "post with image file" call.

## Reading and engaging

- Fetch a post's content by its **ugcPost/share URN**; list reactions on a
  share/post/comment URN. Commenting needs the **actor** URN (who comments), the
  **target** URN (what's being commented on), and the message; a reply also needs
  the parent comment URN.

## Analytics (organization only)

- Org page stats, share stats, and network/follower size are **organization-scoped
  and require admin access** to that page — they take an organization URN, not a
  person. There is no equivalent for personal-profile analytics here. Page/share
  stats support lifetime or time-bounded (DAY/MONTH) windows.

## Care

- **Publishing, commenting, and deleting are public and irreversible.** Show the
  exact post/comment text and the chosen author identity for confirmation before
  creating; never auto-publish on a vague request. Delete only with an explicit,
  specific instruction.
- Match the user's voice and the platform's norms; don't pad posts with hashtags
  or emoji unless asked.
- **Rate limits are real.** The shared connector app has strict quotas — batch
  reads, page through results instead of pulling everything, and don't retry a
  write in a tight loop. If a write returns an upgrade/version or scope error,
  report it rather than retrying; member-posting and org-admin scopes can't
  coexist on one connection, so some accounts can post *or* read org stats, not
  both.
