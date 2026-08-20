# Rejection taxonomy: what actually gets apps rejected, ranked

Prioritization for the audit report: spend attention where rejections actually
happen. Compiled 2026-08-20 from Apple's transparency report, Apple Developer Forum
threads, and developer-community postmortems; confidence is labeled per cluster.
Guideline numbers verified against the live guidelines the same day.

**The ranking is an editorial heuristic, not measured frequency.** Apple publishes
only the section-level counts below; per-guideline frequency cannot be derived from
them, so the cluster order reflects a judgment call synthesizing those counts with
recurring developer reports. Treat it as a sensible default ordering, never as data
to cite.

Contents:

1. [The only primary-source numbers](#the-only-primary-source-numbers)
2. [Ranked clusters](#ranked-clusters)
3. [Frequency claims to distrust](#frequency-claims-to-distrust)

## The only primary-source numbers

Apple's App Store Transparency Report (2025 edition,
apple.com/legal/app-store/transparency/2025/): 9,100,620 submissions reviewed,
2,093,244 rejected (about 23%). Rejections by guideline category (a submission can be
counted in several):

| Category | Count | Share of rejections |
|---|---|---|
| Performance (section 2) | 1,354,418 | 64.7% |
| Legal (section 5) | 495,673 | 23.7% |
| Design (section 4) | 415,532 | 19.9% |
| Business (section 3) | 283,820 | 13.6% |
| Safety (section 1) | 151,159 | 7.2% |

The ordering (Performance far first, then Legal, Design, Business, Safety) is stable
across report editions and is the reliable signal; exact totals vary by edition. Apple
publishes no finer-grained "top rejection reasons" document; every listicle claiming
per-guideline percentages is unsourced.

## Ranked clusters

Ranking synthesizes Apple's category data with recurring developer reports. "First
submission" vs "update" marks when the trap usually springs.

### 1. Completeness, crashes, broken demo access (2.1): high confidence

Ranked first because Performance (section 2) dominates Apple's section-level counts
and 2.1 is that section's most-reported member in developer threads: a judgment
call, since Apple publishes no per-guideline numbers. Live text: "We will reject
incomplete app bundles and binaries that crash or exhibit obvious technical
problems." Recurring forms:

- Crashes or obvious bugs on the reviewer's device, core features unreachable
- Placeholder content still in the build
- Broken or missing demo credentials for login-gated flows: a persistent multi-year
  pattern in Apple Developer Forum threads (2FA codes the reviewer cannot receive,
  credentials that fail outside the developer's environment, staging servers down
  during review)

Hits first submissions and updates alike. Largely runtime territory, but the demo
account and review notes surface is fully auditable before submission.

### 2. Spam, duplicates, template output (4.3): high confidence, escalating

The clearest 2024-2026 escalation. The June 2026 4.3(b) tightening names categories
where Apple will not accept new submissions unless they offer a "meaningfully different
or improved experience": dating, flashlight, sound effects, wallpaper, simple timers,
fortune telling. Generic AI chatbots, image generators, and summarizers, and
white-label or template output, are the community-reported hot zone. Mostly a
first-submission trap. Semantic judgment, not static.

### 3. Account creation without account deletion (5.1.1(v)): high confidence

Mechanically simple, frequently missed, and re-triggered when an update adds accounts.
Common in generated and scaffolded codebases where auth is wired without a delete
flow. Statically auditable (static-checks.md section 7).

### 4. Minimum functionality, web wrappers (4.2): high confidence

Recurring rejection sentence: the app is "not sufficiently different from a mobile
browsing experience." Hits website wrappers and thin API-chat wrappers on first
submission. Static fingerprint plus semantic judgment.

### 5. Tracking and data-sharing consent (5.1.2): medium-high confidence

ATT violations (tracking before or despite the prompt; Firebase or ad SDKs configured
with IDFA and no ATT flow) recur in forum reports; often an update trap when an SDK is
added later. The November 2025 third-party-AI disclosure wording is the newest surface
in this family and will grow as enforcement catches up.

### 6. Payments and subscription disclosure (3.1.x): medium-high confidence

Digital goods sold outside IAP (3.1.1), missing restore purchases, and paywalls
lacking required subscription information (3.1.2) all produce steady rejections;
community forums for subscription SDKs carry a constant stream of 3.1.2 threads.
External-link rules (3.1.1(a)) are jurisdiction-dependent and in flux; always
re-checked live.

### 7. Login services (4.8): medium confidence

Third-party social login without a privacy-protecting alternative. Real and recurring
in forum threads; no frequency data. First-submission trap. Statically detectable.

### 8. Metadata accuracy (2.3): medium confidence

Descriptions claiming features the app lacks, screenshots not showing the app in use,
hidden or undocumented features (2.3.1), undisclosed IAP in metadata (2.3.2), keyword
stuffing and trademark use (2.3.7). "Metadata Rejected" outcomes resolve without a new
binary, so they are cheaper, but they still stall releases.

### 9. UGC moderation and age rating (1.2, 2.3.6): medium confidence

UGC apps missing the required moderation set (filtering, reporting with timely
response, user blocking, published contact info). Tightening pressure from the
2025-2026 age-rating overhaul, which added mandatory questionnaire answers (deadline
January 31, 2026) including AI chatbot impact on sensitive content.

### 10. Long tail worth one look each: lower frequency

- **IP infringement (5.2.1, 4.1)**: borrowed icons, brands, screenshots, or names;
  copycat submissions.
- **Export compliance**: missing `ITSAppUsesNonExemptEncryption` answer; a speed bump
  more than a rejection.
- **Special categories**: kids, medical, gambling, VPN, crypto and finance carry their
  own hard requirements; see special-categories.md. Low volume overall, near-certain
  rejection when requirements are missed.

## Frequency claims to distrust

Do not repeat these in reports; they circulate widely without primary sources:

- Any "guideline X accounts for N% of rejections" figure (for example "4.3 is 28%"):
  no primary source exists, and Apple's own category data contradicts the popular ones.
- "N% of appeals succeed" and "AI false rejections rose N% in 2025": attributed only
  to unnamed forum reports.
- Fixed review-time promises; see metadata-and-review-notes.md for what Apple actually
  publishes about timing.
