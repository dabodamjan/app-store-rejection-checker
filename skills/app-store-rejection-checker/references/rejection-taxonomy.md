# Rejection taxonomy: what actually gets apps rejected, ranked

Prioritization for the audit report: spend attention where rejections actually
happen. Compiled 2026-08-20 from Apple's transparency report, Apple Developer Forum
threads, and developer-community postmortems; confidence is labeled per cluster.
Guideline numbers re-verified against the live guidelines on 2026-11-05 (text dated
October 16, 2026).

**The ranking is an editorial heuristic, not measured frequency.** Apple publishes
only the section-level counts below; per-guideline frequency cannot be derived from
them, so the cluster order reflects a judgment call synthesizing those counts with
recurring developer reports. Treat it as a sensible default ordering, never as data
to cite.

Contents:

1. [The only primary-source numbers](#the-only-primary-source-numbers)
2. [Ranked clusters](#ranked-clusters)
3. [Account-level exposure](#account-level-exposure)
4. [Frequency claims to distrust](#frequency-claims-to-distrust)

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

The same report's account-level figures (fetched 2026-11-05): 193,035 developer
accounts terminated, 192,702 of them under the license agreement's fraud clause
(3.2(f) in the report's numbering); 10,127 termination appeals; 499 restorations,
with a footnote that most appealed terminations were for fraud. Apple's May 20, 2026
newsroom post adds that App Review uses AI to "analyze app similarity, and flag
potentially problematic changes in app updates" and that it removed nearly 59,000 apps
approved as simple games or utilities that then changed for fraud ("bait-and-switch").
Those are the only published numbers on terminations; nothing says how many
rejections or submissions precede one.

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
community forums for subscription SDKs carry a constant stream of 3.1.2 threads. The
3.1.2 rejection text has been stable since at least 2019 (trial length and post-trial
price not stated, billed amount not the most prominent element); what changed in 2026
is enforcement reach: free-trial toggles rejected since mid-January per subscription
vendors, and paywalls that passed on earlier submissions failing on later ones.
Reviewer-sandbox purchase failures reported as 2.1(b) belong to the same submission
and are covered in metadata-and-review-notes.md. External-link rules (3.1.1(a)) are
jurisdiction-dependent and in flux; always re-checked live. Check procedure:
static-checks.md section 6.

### 7. Login services (4.8): medium confidence

Third-party social login without a privacy-protecting alternative. Real and recurring
in forum threads; no frequency data. First-submission trap. Statically detectable.

### 8. Metadata accuracy (2.3): medium confidence

Descriptions claiming features the app lacks, screenshots not showing the app in use,
undisclosed IAP in metadata (2.3.2), keyword stuffing and trademark use (2.3.7), and,
since October 2026, creative assets that carry pricing or unverifiable claims
(2.3.14). "Metadata Rejected" outcomes resolve without a new binary, so they are
cheaper, but they still stall releases.

Hidden, dormant, or undocumented features used to sit in this cluster under 2.3.1.
Since October 16, 2026 they are 2.5.19, whose text names code obfuscation and
account termination, and Apple's companion announcement ties the rule to
"bait-and-switch" apps and cites 193K developer accounts terminated for fraud and
abuse in the previous year. Treat a 2.5.19 finding as account-level exposure, not a
metadata nit: rank it by severity in the report even though its frequency is
unknown. Static signals: static-checks.md section 13.

### 9. UGC moderation and age rating (1.2, 2.3.6): medium confidence

UGC apps missing the required moderation set (filtering, reporting with timely
response, user blocking, published contact info). Tightening pressure from the
2025-2026 age-rating overhaul, which added mandatory questionnaire answers (deadline
January 31, 2026) including AI chatbot impact on sensitive content. The 1.2 scope
widened twice in 2026: random or anonymous chat was pulled in on February 6, and the
October 16 revision explicitly covers "AI content creation apps" and nonconsensual
intimate imagery, with a remediate-after-notice clause that escalates to the
Developer Code of Conduct. AI generators with no moderation set are the new exposure.

### 10. Long tail worth one look each: lower frequency

- **IP infringement (5.2.1, 4.1)**: borrowed icons, brands, screenshots, or names;
  copycat submissions.
- **Export compliance**: missing `ITSAppUsesNonExemptEncryption` answer; a speed bump
  more than a rejection.
- **Special categories**: kids, medical, gambling, VPN, crypto and finance carry their
  own hard requirements; see special-categories.md. Low volume overall, near-certain
  rejection when requirements are missed.

## Account-level exposure

Severity ranking in the report (SKILL.md Step 6) puts a finding that carries
termination or Program-removal language above others at the same confidence, whatever
its frequency. Text that names that consequence, all on the live page dated
October 16, 2026, plus the license agreement as fetched 2026-11-05:

- Introduction: trying "to trick the review process", stealing data, copying, or
  manipulating ratings or discovery gets the developer "expelled from the Apple
  Developer Program".
- 2.5.19: hiding undocumented features "through code obfuscation or other methods".
- 2.3.1(a): misleading marketing and "promoting a false price, whether within or
  outside of the App Store"; 2.3.1(b): "Egregious or repeated behavior".
- 3.1.2(a): tricking users "into purchasing a subscription under false pretenses" or
  "bait-and-switch and scam practices".
- 4.3(b): "Repeated submissions of this kind".
- 1.2: egregious or repeated failure to moderate; failing to remediate after notice
  is a Developer Code of Conduct matter.
- 5.4: VPN apps that do not comply.
- 5.6: "Repeated manipulative or misleading behavior or other fraudulent conduct";
  5.6.4 counts "excessive customer reports ... and excessive refund requests" against
  the developer.
- Developer Program License Agreement: section 3.2 item 6 (acts that interfere with
  the App Store, "bait-and-switch pricing, consumer misrepresentation, deceptive
  business practices"); section 3.2 item 5(3) (code that would "deceive regarding the
  identity, purpose, or behavior of the product, code, or program, including through
  post-review modifications that would cause harm to others"); section 11.2 item 7,
  termination "effective immediately upon notice" for "hiding or trying to hide
  functionality from Apple's review, falsifying consumer reviews for Your Application,
  engaging in payment fraud".

Which audit findings map onto that list: reviewer detection and dormant or obfuscated
code (static-checks.md section 13), false or tricky prices and subscription
bait-and-switch (section 6), review manipulation (section 14), missing UGC moderation
in a generator app (special-categories.md), and anything in the review notes or
metadata that is not true of the submitted build (metadata-and-review-notes.md). The
pattern-of-submissions side (repeated rejections, replacement apps, template
portfolios) is not visible from a repository; the report can only name it when the
developer says the app has been rejected before. Reported 2026 cases and the advice
that follows: metadata-and-review-notes.md, repeated rejections section.

## Frequency claims to distrust

Do not repeat these in reports; they circulate widely without primary sources:

- Any "guideline X accounts for N% of rejections" figure (for example "4.3 is 28%"):
  no primary source exists, and Apple's own category data contradicts the popular ones.
- "N% of appeals succeed" and "AI false rejections rose N% in 2025": attributed only
  to unnamed forum reports.
- Fixed review-time promises; see metadata-and-review-notes.md for what Apple actually
  publishes about timing.
