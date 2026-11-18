# Email Sharing Rules: Which Rule Wins

> **What this is:** the exact order Day AI uses to decide whether an email is shared with the workspace or kept private, and what that means for Gmail label inclusion and exclusion rules. Read it before writing the recommended setup in `workspace/PRIVACY.md` or the per-user steps in `rollouts/preflight/ENABLEMENT/`.

## The rules a user can create

Each user sets their own email sharing rules in their sharing settings. Their agent can open that screen with `open_email_sharing_rules`. Every rule has two parts:

- **What it matches:** a **Gmail label**, an **email address**, a **domain**, or **all email** (the global share toggle).
- **What it does:** **Shared** ("Share with workspace") or **Private** ("Keep private").

An **inclusion** is a Shared rule and an **exclusion** is a Private rule. The settings screen shows each as a badge on the rule.

If the workspace has **default rules** turned on, an Owner or Admin can also add workspace-wide Private rules for an address or domain on the Members page. Defaults can only keep mail private, never share it.

## The order, per email

Day AI decides each email separately, using the rules of the person whose mailbox it came from. The first step that has a matching rule decides, and later steps aren't consulted.

1. **Gmail labels.** If the email carries any label that has a rule, only the label rules count, and address, domain, default and all-email rules are ignored.
   - If **any** matching label rule is Private, the email is **Private**.
   - Otherwise (every matching label rule is Shared), the email is **Shared**.
2. **Email address rules** for someone on the email.
3. **Workspace default address rules** (Private).
4. **Domain rules** for someone on the email. A domain rule matches the company's root domain (so `acme.com` also covers `mail.acme.com`). It never matches personal domains like gmail.com or outlook.com.
5. **Workspace default domain rules** (Private).
6. **The all-email rule** (the global share toggle).
7. **No rule matches:** the email is **Private**.

Steps 2 to 6 are checked for each person on the email (sender and recipients). For each person, only the most specific matching step counts, so an address rule overrides a domain rule for that person. If the deciding rule for **any** person is Private, the email is **Private**.

**At any step, Private beats Shared.**

## What this means for Gmail label rules

- **An exclusion label always wins against an inclusion label.** An email labelled both `Sales` (Shared) and `Investors [CONFIDENTIAL]` (Private) stays private. This holds for any number of labels: one Private label is enough.
- **An inclusion label overrides address, domain and default exclusions.** This is the one to warn people about. If `counsel@lawfirm.com` has a Private address rule, or `lawfirm.com` is excluded by a workspace default, an email to them that the user labels `Clients` (Shared) is **shared**. The label decides; the exclusion is never checked.
- **To keep something private no matter how it's labelled, use a Private label rule** (for example a `[CONFIDENTIAL]` label). An address or domain exclusion only protects email that carries no labelled rule.
- **Unlabelled email falls through to address and domain rules.** A user who only has label inclusions and no all-email rule shares only the labelled email; everything else is private by default.
- **Labels match exactly.** Each rule matches one label. A rule on a parent label doesn't cover its sub-labels, so add a rule for each one. Only labels the user created can be picked; built-in Gmail labels like Inbox or Starred can't.
- **Only the mailbox owner's labels count.** When a teammate is on the same email, Day AI also checks the teammate's own address, domain and all-email rules, but not their labels.
- **Labelling later works.** Adding or removing a label in Gmail re-syncs that thread, so the decision follows the label as it is now.

## Threads and teammates

- **One private email makes the whole thread private** on thread-level views such as the thread summary. If any email in a thread is Private, the summary isn't shared.
- **Teammates can make a thread private.** If a teammate who has their own sharing rules is on an email, their rules must also allow sharing. If their deciding rule is Private, the email is Private for everyone.
- **Rule changes don't reach search right away.** A change applies right away to thread summaries. Email search and the timeline catch up when that mailbox's email is re-processed.

## Worked examples

Assume the user has Shared label rules for `Sales` and `Clients`, Private label rules for `Investors [CONFIDENTIAL]` and `Recruiting [CONFIDENTIAL]`, a Private domain rule for `hr-vendor.com`, and the global share toggle on.

| Email | Deciding step | Result |
|-------|---------------|--------|
| Labelled `Sales` | 1: labels | Shared |
| Labelled `Sales` and `Investors [CONFIDENTIAL]` | 1: labels, one is Private | **Private** |
| No labels, from `jane@hr-vendor.com` | 4: domain | Private |
| Labelled `Clients`, from `jane@hr-vendor.com` | 1: labels (the domain rule is skipped) | **Shared** |
| No labels, from `bob@prospect.com` | 6: all-email toggle | Shared |
| Same as above, with the global toggle off | 7: nothing matches | Private |

## Recommended setup patterns

- **Confidential categories:** create Gmail labels such as `Legal [CONFIDENTIAL]` and `Investors [CONFIDENTIAL]` with Private rules. They win against any inclusion label.
- **Sensitive people or vendors:** use address or domain exclusions (or workspace defaults), and tell users not to put Shared labels on that mail, because a Shared label overrides those exclusions. If a vendor must never be shared, apply a Private label to its mail with a Gmail filter.
- **Label-only sharing:** turn the global toggle off and add Shared label rules for the categories that should be shared. Anything unlabelled stays private.
