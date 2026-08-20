---
name: app-store-rejection-checker
description: Audits an iOS or iPadOS Xcode project and its App Store Connect metadata against the current App Store Review Guidelines and reports likely rejection risks, each tied to a guideline number with evidence, a fix, and a confidence level. Use before an App Store submission, or when the user mentions App Store review, app rejection, review guidelines, rejection risk, submission preflight, or asks whether their app will pass review.
license: MIT
compatibility: Needs network access to fetch the live App Store Review Guidelines. Designed for repositories containing an Xcode project or Swift package.
metadata:
  author: dabodamjan
  homepage: https://github.com/dabodamjan/app-store-rejection-checker
allowed-tools: Read Grep Glob Bash WebFetch WebSearch
---

# App Store review preflight

Audit the repository and the developer's App Store Connect metadata the way an App Review
reviewer would: read the actual code, the actual configuration, and the actual store copy,
then report what is likely to be rejected and why.

Non-negotiable output rules:

1. Every finding cites a specific guideline number (or names the automated upload
   validation) and states the evidence found in this repository or metadata.
2. Every finding carries a confidence level (defined in Step 6). Never present a
   judgment call as a certainty.
3. Never promise approval, estimate approval odds as a percentage, or invent statistics.
   Passing this audit reduces known risks; it does not guarantee anything.
4. The live guidelines text fetched in Step 1 is the authority. The bundled reference
   files are a map of the rejection landscape (verified 2026-08-20); on any conflict,
   the live text wins, and the report should note the discrepancy. When the live fetch
   truncates before a section, a finding may quote wording recovered via the Step 1
   web-search fallback instead, labeled as a secondary source.

## Step 1: fetch the live guidelines

Do this before running any checklist.

Fetch https://developer.apple.com/app-store/review/guidelines/ with WebFetch. The page is
long and fetch results may truncate near the end (section 5.4 and later). Fetch results
are typically cached per URL, so fetching again retrieves the same snapshot; what a
targeted prompt changes is which part of that snapshot gets extracted. Use targeted
prompts to pull each area out of the snapshot:

1. Sections 1 and 2 (Safety, Performance): ask for the subsection structure and any
   wording relevant to the app under audit.
2. Sections 3 and 4 (Business, Design): same.
3. Section 5 (Legal): same. If the result cuts off before 5.4, re-prompting will not
   recover text beyond the truncation point. Fall back to a web search for the
   specific subsection wording, label it a secondary source in the report, and note
   that the tail of the live text could not be confirmed this run.

Then check https://developer.apple.com/news/ for review-related announcements newer than
the reference files (they were last verified against the live text on 2026-08-20). Pay
attention to anything touching payments, privacy, age ratings, or AI, since those moved
repeatedly in 2024-2026; [references/recent-changes.md](references/recent-changes.md)
lists the changes already covered. When a headline plausibly applies to the app under
audit, fetch the announcement and assess its applicability like any other check;
headlines that do not apply get one line in the verification note, nothing more. This
news check is a headline scan, not exhaustive:
Apple also updates auxiliary policy pages that the news feed does not always announce.
When a finding depends on one of these surfaces, fetch the relevant page too:

- Account deletion: developer.apple.com/support/offering-account-deletion-in-your-app/
- EU DMA terms: developer.apple.com/support/dma-and-apps-in-the-eu/
- Age ratings: developer.apple.com/help/app-store-connect/reference/age-ratings/
- Required-reason APIs:
  developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api

Pages under developer.apple.com/documentation/ are rendered client-side; a plain fetch
often returns an empty shell. When that happens, fall back to a web search for the
specific fact, corroborated by a primary source (an Apple page that does fetch, or an
Apple staff answer in the developer forums), and label it a secondary source. If no
corroboration is found, say the fact could not be verified this run instead of citing
memory.

While auditing, whenever a finding rests on exact guideline wording, quote the wording
from the live fetch, not from memory and not from the reference files. The one
exception is a section the live fetch could not confirm because of truncation: there,
quote the search-derived wording and label it a secondary source in the finding.

## Step 2: inventory the project

Build a picture of what the app actually is before judging it:

- Locate the Xcode project: `*.xcodeproj`, `*.xcworkspace`, `Package.swift`.
- Dependencies: `Podfile.lock`, `Package.resolved`, `Cartfile.resolved`, vendored SDKs.
- Configuration: every `Info.plist` (plus `INFOPLIST_KEY_*` entries inside
  `project.pbxproj`), `*.entitlements`, `PrivacyInfo.xcprivacy`, `*.storekit`.
- Store metadata in the repo, if present: `fastlane/metadata/`, marketing copy, release
  notes.
- Read enough source to know what the app does: app entry point, main views or view
  controllers, networking layer, any payment or auth code.

Classify the app: does it have accounts, payments, subscriptions, UGC, chat or AI
features, web content, kids targeting, health/medical/finance/gambling/VPN exposure?
The classification decides which checks below are load-bearing.

If the project also builds for other platforms (macOS, watchOS, visionOS, tvOS
targets, or shared entitlements carrying their keys), state in the report that only
the iOS/iPadOS surface is audited and the other platforms are out of scope.

If App Store Connect metadata is not in the repo, ask the user to paste the app name,
subtitle, description, keywords, age rating, screenshots description, and the App Review
Information fields (notes and demo account). The metadata checks are real rejection
surfaces; skipping them silently would understate risk. If the user declines, or there
is no user to ask (a non-interactive or CI run), mark those checks "not audited" in the
report and continue. Never invent metadata to audit.

## Step 3: static checks

Work through [references/static-checks.md](references/static-checks.md). It covers the
mechanically verifiable surface:

- Permission purpose strings vs the APIs and SDKs actually linked (upload validation
  rejects mismatches).
- Privacy manifest (`PrivacyInfo.xcprivacy`): presence, required-reason API
  declarations, tracking declarations. Enforced at upload since May 1, 2024.
- Entitlements consistency (push, Sign in with Apple, app groups).
- Login services (4.8): third-party social login without a privacy-protecting
  alternative.
- Export compliance (`ITSAppUsesNonExemptEncryption`).
- Payments wiring: IAP vs external payment SDKs for digital goods, restore purchases,
  subscription disclosure, loot box odds.
- Account deletion path when account creation exists.
- App Tracking Transparency wiring vs tracking SDKs.
- Analytics consent: analytics SDKs initialized collection-on with no consent flow.
- Third-party AI data flows: personal data shared with third-party LLM or ML APIs
  without disclosure and consent wiring.
- Web-wrapper fingerprints (guideline 4.2).
- Dynamic code loading, deprecated API red flags.

These checks read build settings, plists, lockfiles, and source. Prefer evidence from
files over inference; quote file paths and keys in findings.

## Step 4: semantic review

This is the part a linter cannot do. Read the code and the copy, then judge:

- **2.3 metadata accuracy**: does the store description claim anything the code does not
  implement? Do screenshots (as described) show the app in use, or splash/marketing art?
  Are there hidden, dormant, or undocumented features in the code (2.3.1), including
  debug switches, remote feature flags that change behavior after review, or endpoints
  that serve different content by region or date?
- **4.2 minimum functionality / 4.3 differentiation**: for thin apps, wrappers, and apps
  in categories Apple names as saturated (dating, flashlight, sound effects, wallpaper,
  simple timers, fortune telling), form a view on what this app does that existing apps
  do not, and say plainly if the answer is thin. This is a judgment call; label it as
  one.
- **AI and data flows (5.1.2(i))**: if the app shares **personal data** with a
  third-party AI service (an external LLM or ML API), verify the app discloses this
  and obtains explicit permission before doing so, and that the privacy labels and
  privacy policy cover it. First-party or on-device processing, and flows that carry
  no personal data, are not automatic 5.1.2(i) triggers, but judge whether the
  content sent could contain personal data in practice, and say which side of the
  line the app falls on.
- **Review notes and demo account quality**: judge them against the checklist in
  [references/metadata-and-review-notes.md](references/metadata-and-review-notes.md).
  Missing or broken demo access commonly blocks review of every login-gated flow, and
  fixing it costs nothing but preparation.
- **Special categories**: if Step 2 classified the app into any high-scrutiny category,
  apply [references/special-categories.md](references/special-categories.md) and
  re-read the matching live guideline section from Step 1.

Prioritize findings using [references/rejection-taxonomy.md](references/rejection-taxonomy.md),
an editorial ranking of rejection clusters (built from Apple's section-level counts
plus recurring developer reports) with per-cluster confidence labels.

## Step 5: metadata and submission readiness

Work through [references/metadata-and-review-notes.md](references/metadata-and-review-notes.md)
for the App Store Connect surface: name and keyword rules (2.3.7), screenshot rules
(2.3.3), IAP disclosure in metadata (2.3.2), age rating (2.3.6 and the 2025-2026
questionnaire overhaul), review notes (2.3.1), and demo credentials. It also explains
what "Metadata Rejected" means mechanically and what to expect from review timing.

## Step 6: report

Produce the report as markdown. Structure:

1. **Summary**: app classification, overall risk picture in two or three sentences, the
   top three findings.
2. **Findings**, ordered by severity then confidence. Severity means likely rejection
   impact: how likely the finding, shipped as-is, is to block approval or force a
   resubmission. It is not the taxonomy file's cluster ranking, which reflects
   frequency; a rare special-category miss outranks a frequent metadata nit. Each
   finding:

   ```
   ### [guideline number] short title
   Confidence: validator-certain | review-risk | judgment-call
   Evidence: what was found, with file paths or metadata fields
   Why it matters: one or two sentences, citing the live guideline text (or, for a
   section the live fetch truncated, the labeled secondary source per rule 4)
   Fix: the concrete change to make
   ```

   Confidence levels mean exactly this:
   - **validator-certain**: Apple's automated upload validation or App Store Connect
     will block this deterministically (for example a missing purpose string for a
     linked API, or a missing required-reason declaration), **and** the evidence
     reflects what actually ships: an archive, `.app`, or `.ipa`, or target-resolved
     build settings for the shipping target. Source or lockfile evidence alone never
     reaches this level: conditional compilation, unused targets, files excluded
     from the shipping target, and packaging-time manifests can all make it false.
     In a pre-submission repo audit with no built product, expect this level to go
     unused; that is correct, not a gap.
   - **review-risk**: a pattern human reviewers reject frequently, backed by the
     taxonomy file or the live guideline text; not mechanical, but well evidenced.
     When the issue would be mechanically blocked by upload validation if shipped
     as-is but the evidence is source or lockfile only, mark the sub-form explicitly:
     `review-risk (validator-blocking if shipped as-is; needs build verification)`.
     This keeps a genuinely missing purpose string triageable apart from soft
     judgment-adjacent patterns in the same bucket.
   - **judgment-call**: a semantic assessment (differentiation, metadata accuracy,
     copy quality) where a reasonable reviewer could go either way.

   The `Confidence:` line uses only these three levels. The marked sub-form above is
   the review-risk level with a parenthetical qualifier, not a fourth level. Never
   carry the taxonomy file's per-cluster labels (high, medium-high, medium
   confidence) into a finding.

3. **Not checked**: list what this audit cannot see. At minimum: runtime crashes,
   broken links, restore-purchase behavior, live server content, OAuth flows, and
   anything driven by remote configuration. Never imply these were verified.
4. **Verification note**: state that guideline citations were checked against the live
   guidelines during this run, and list any section the live fetch could not confirm.
   For each such section, say whether any finding quotes search-derived wording
   instead, labeled as a secondary source; that labeled quote is acceptable only for
   the truncated sections, per rule 4.

Do not pad the report. A clean area gets one line ("no findings in X"), not a section.
