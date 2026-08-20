# App Store Connect metadata, review notes, and process mechanics

The store-listing and submission-form surface: what reviewers cross-check against the
binary, and how the review process behaves when something is rejected. Guideline
numbers verified against the live guidelines on 2026-08-20.

Contents:

1. [Metadata checks (2.3.x)](#metadata-checks-23x)
2. [Review notes checklist (2.3.1)](#review-notes-checklist-231)
3. [Demo account checklist (2.1)](#demo-account-checklist-21)
4. [Metadata Rejected vs binary rejection](#metadata-rejected-vs-binary-rejection)
5. [Review timing, expedited review, appeals](#review-timing-expedited-review-appeals)

## Metadata checks (2.3.x)

- **Accuracy (2.3, 2.3.1)**: the description, screenshots, and previews must match what
  the binary does. Cross-check every feature claim in the description against the code
  inventory from Step 2; flag claims with no implementation ("AI-powered" with no model
  or API call in the code, "community" with no UGC surface). Hidden, dormant, or
  undocumented features in the build are an explicit 2.3.1 violation.
- **IAP disclosure (2.3.2)**: if items, levels, or subscriptions require additional
  purchases, the description, screenshots, and previews must say so.
- **Screenshots (2.3.3)**: must show the app in use, not splash screens, login walls,
  or pure marketing art.
- **Age rating (2.3.6)**: answers must match the app's actual content and features.
  The questionnaire was overhauled in 2025 (see recent-changes.md): tiers are now 4+,
  9+, 13+, 16+, 18+ (displayed on OS 26 and later; older OS versions show the old
  tiers), answers were due by January 31, 2026, and the questionnaire asks how AI
  assistant and chatbot features affect the frequency of sensitive content. When
  checking answers, remember Messaging and Chat, User-Generated Content, and Social
  Media are separate questionnaire capabilities — chat without a UGC declaration is
  not automatically a mismatch. Real mismatch flags: a capability clearly present in
  the code but not declared (a social feed with no Social Media answer), or ad SDKs
  with no matching content descriptors.
- **Name and keywords (2.3.7)**: app name at most 30 characters; no pricing or
  irrelevant phrases in name, subtitle, or keywords. On trademarks, the rule targets
  **unauthorized** or discovery-gaming use — other developers' trademarks or popular
  app names used without permission or with no relevant content in the app — not
  every trademark reference (naming a service the app legitimately integrates with
  is not itself a violation).
- **All-audiences metadata (2.3.8)**: icons, screenshots, and previews must be
  appropriate for a 4+ audience even when the app is rated higher; "for kids" wording
  is reserved for the Kids Category.
- **Rights to metadata materials (2.3.9, 5.2.1)**: no borrowed screenshots, brands, or
  celebrity imagery without permission.

## Review notes checklist (2.3.1)

Live text: "All new features, functionality, and product changes must be described with
specificity in the Notes for Review section." Good notes cover everything a reviewer
cannot infer from the binary:

- Demo credentials (below), including for each account tier if roles differ
- Setup steps to reach core functionality quickly
- Region-locked or geo-dependent content, and how to see it from the review location
- Required hardware or peripherals, and what happens without them
- Server-driven or remote-config behavior, explained rather than discovered
- For medical apps with regulatory clearance: the link to that documentation (1.4.1)
- For VPN or licensed categories: license information where a territory requires it

## Demo account checklist (2.1)

A broken login for the reviewer is functionally a completeness rejection. Verify:

- A demo account exists for every login-gated flow, is filled with realistic content
  (not a blank state), and its credentials are in App Review Information
- Credentials tested on a clean device shortly before submission
- 2FA or OTP flows: the code or bypass is supplied in the notes; a code sent to a
  phone the reviewer does not have is a rejection generator
- The backing server and staging environment stay up for the whole review window
- If credentials legally cannot be provided, a demo mode needs prior arrangement with
  Apple; do not assume it silently substitutes

## Metadata Rejected vs binary rejection

- **Metadata Rejected** concerns only the store listing (text, images, review
  information). Fix the flagged fields and resubmit; the same binary is reused, no new
  upload needed.
- A general **Rejected** verdict does not by itself require a new build — the
  remediation determines that. Rejections resolved by fixing metadata, App Review
  Information, demo access, or a backend issue can be resubmitted on the same binary;
  a new upload is required when the fix is in the binary itself (crashes, missing
  features, code-level violations) or when the upload was refused as **Invalid
  Binary**.
- Either way the submission lands in an "Unresolved Issues" state: remove the rejected
  item to let the rest proceed, or fix and resubmit, and reply in the same App Store
  Connect message thread.

## Review timing, expedited review, appeals

- Apple's published figure (developer.apple.com/distribute/app-review/): "90% of
  submissions are reviewed in less than 24 hours." Treat it as a target, not a
  promise: 2026 community reports show wide variance (multi-day waits, especially for
  Mac apps and new accounts). Never present a guaranteed turnaround.
- **Expedited review** can be requested (developer.apple.com/contact/app-store/,
  expedite topic) for a critical bug fix in the live version (with repro steps) or a
  genuinely time-sensitive event. Routine releases do not qualify, and repeated
  misuse gets future requests denied.
- **On rejection**: reply in the Resolution Center first; a phone call can be
  requested. An App Review Board appeal must argue specific compliance and is limited
  to one appeal per rejected submission. If unrelated new issues surface during a
  bug-fix review, Apple may offer to let the current submission through and defer the
  new issues to the next one; that offer arrives in the message thread.
