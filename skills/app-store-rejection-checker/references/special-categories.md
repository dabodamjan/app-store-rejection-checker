# Special categories: extra requirements beyond the base guidelines

Apply the matching section when Step 2 classifies the app into one of these. Each
carries requirements that are near-certain rejections when missed. Guideline numbers
verified against the live guidelines on 2026-11-05 (text dated October 16, 2026);
quotes for 1.4.1, 5.1.1(ix), 5.1.3, and 5.4 taken verbatim from the live text. Always
re-read the matching live section during the audit; these are summaries.

Contents:

1. [Kids Category (1.3)](#kids-category-13)
2. [Medical and health (1.4.1, 5.1.3)](#medical-and-health-141-513)
3. [User-generated content (1.2)](#user-generated-content-12)
4. [Gambling, contests, real-money gaming (5.3)](#gambling-contests-real-money-gaming-53)
5. [VPN apps (5.4)](#vpn-apps-54)
6. [Cryptocurrency (3.1.5)](#cryptocurrency-315)
7. [Financial trading and money management (3.2.1(viii))](#financial-trading-and-money-management-321viii)
8. [Dating (no dedicated section)](#dating-no-dedicated-section)

## Kids Category (1.3)

- No links out of the app, purchasing opportunities, or other distractions for kids
  unless behind a parental gate in a designated area.
- No third-party analytics or third-party advertising, with narrow carve-outs:
  analytics only if the service collects no IDFA and nothing identifiable about
  children; contextual ads only from providers with publicly documented practices and
  human review of ad creatives for age appropriateness.
- Compliance with children's privacy law (COPPA, GDPR-K and equivalents) is on the
  developer.
- Once in the Kids Category, the obligations stick even if the category is later
  deselected.
- EU note: Kids Category apps cannot offer out-of-app purchase links under the DMA
  terms; see recent-changes.md.

Static signals to cross-check: any ad or analytics SDK in the lockfile of a Kids
Category app; `SKAdNetwork` identifiers; outbound links without a parental gate.

## Medical and health (1.4.1, 5.1.3)

Scope is wider than apps that call themselves medical: diet, nutrition, allergen,
symptom, sleep, and supplement apps are read against 1.4.1 when they tell a user what
is safe, healthy, or recommended. Live text: "Medical apps that could provide
inaccurate data or information, or that could be used for diagnosing or treating
patients may be reviewed with greater scrutiny." Then: "Apps must clearly disclose
data and methodology to support accuracy claims relating to health measurements, and
if the level of accuracy or methodology cannot be validated, we will reject your app",
and "Apps should remind users to check with a doctor in addition to using the app and
before making medical decisions."

- Explicitly not permitted: apps claiming to take x-rays or measure blood pressure,
  body temperature, blood glucose, or blood oxygen using only device sensors.
- **Sources and method visible in the app** (review-risk for any app giving dietary or
  health guidance): the methodology sentence is enforced as "show your sources".
  Developers report 1.4.1 rejections of diet and nutrition apps with the text "We are
  still unable to locate links to medical sources in the app"
  (developer.apple.com/forums/thread/760773, July 2024) and for missing "appropriate
  citations for health and medical recommendations"
  (developer.apple.com/forums/thread/767340, October 2024); in both threads a
  disclaimer alone did not clear it. Static signal: a bundled or fetched reference
  dataset (ingredients, foods, dosages, thresholds) with a source field that no view
  ever renders, or guidance copy with no sources screen reachable from the result or
  from Settings. Fix: a sources-and-method screen linked from where the verdict is
  shown, naming the institutions or publications behind the data.
- **Verdict wording as an accuracy claim** (review-risk): binary labels such as
  "Safe" and "Dangerous", "Healthy" and "Unhealthy", applied to foods, products, or
  readings are accuracy claims the app usually cannot validate. Prefer wording that
  states what the data says ("high in X", "flagged", "not in our data") and keep the
  word "safe" out of the app name's neighbors (subtitle, screenshots) when it is in
  the name. Related: an engine that defaults unknown or unparseable input to the
  favorable verdict is both the 1.4.1 "inaccurate information" risk and a product
  bug; the honest output for unknown input is "could not check".
- **Doctor reminder** (review-risk when absent): the text says "check with a doctor",
  so a disclaimer that names only a dietitian, coach, or pharmacist is incomplete.
  The reminder belongs where decisions are made (the result screen) and in onboarding
  and the store description, not only in Settings.
- If the app has regulatory clearance (for example FDA), submit a link to that
  documentation with the app; "Before You Submit" also asks for "required
  documentation for apps and features that require regulatory approval".
- **Regulated medical device declaration** (App Store Connect, since March 26, 2026;
  developer.apple.com/news/?id=nyqbfz1y): for apps distributed in the EEA, UK, or US
  whose "primary or secondary category is Health & Fitness or Medical", or that are
  "marked as containing frequent references to Medical or Treatment Information in
  the Age Rating questionnaire", a regulated-medical-device status is required before
  a new app can be distributed there, and existing apps must declare by early 2027.
  The help page (developer.apple.com/help/app-store-connect/manage-app-information/declare-regulated-medical-device-status/)
  says "If your app doesn't provide any health or medical features, or access health
  data to support non-health features, you should select No"; a Yes needs regulatory
  details. The answer shows on the product page. Audit angle: if Step 2 found health
  features and the category or age rating answer trips the trigger, ask whether the
  declaration is answered; a missing one blocks distribution in those regions rather
  than failing review.
- **Age rating answers for health content** (2.3.6): the questionnaire's "Health or
  Wellness Topics" covers "self-care or lifestyle recommendations" including "calorie
  tracking, dieting advice, or exercise recommendations" and rates 9+; "Medical or
  Treatment Information" covers "diagnoses or guidance around the management of
  medical conditions", Infrequent rating 13+ and Frequent 16+ (Apple's age ratings
  definitions page, fetched 2026-11-05). An app that names a medical condition and
  helps manage it with "None" on the medical answer is a mismatch; "Frequent" is also
  what trips the device declaration above. Judgment-call on the honest level, flag
  only clear mismatches.
- **Account type (5.1.1(ix))**: healthcare is on the live list of "highly regulated
  fields" whose apps "should be submitted by a legal entity that provides the
  services, and not by an individual developer." Not readable from the repo; ask
  (metadata-and-review-notes.md, developer identity).
- Health and fitness data has its own privacy rules (5.1.3(i)): data gathered in the
  health, fitness, and medical-research context (HealthKit, Clinical Health Records,
  Motion and Fitness, MovementDisorder APIs, health research) may not be used or
  disclosed to third parties for advertising, marketing, or other use-based data
  mining. Uses for improving health management or for health research are permitted,
  and only with permission. A separate carve-out allows using the data to provide a
  benefit directly to the user (for example a reduced insurance premium) only when
  the app is submitted by the entity providing the benefit and the data is not
  shared with a third party. The specific health data collected from the device must
  be disclosed. Also: no writing false data into HealthKit, and no storing personal
  health information in iCloud (5.1.3(ii)).
- **Health data in iCloud (5.1.3(ii))**, the static form: CloudKit (`CKContainer`,
  `NSPersistentCloudKitContainer`, iCloud key-value store, or an iCloud-synced
  document directory) carrying fields that describe the user's body or condition:
  symptoms, diagnoses, medications, readings, allergies framed as medical. That is
  review-risk; the text says apps "may not store personal health information in
  iCloud" with no carve-out for the private database. Food preferences, shopping
  lists, and product notes are not health information; the line is whether a record
  says something about the person's health. If the app syncs such fields, the fix is
  a non-iCloud store or dropping the fields, not a privacy-policy sentence.

## User-generated content (1.2)

Scope, per the October 16, 2026 text: "Apps that enable users to create, generate,
upload, share, or otherwise contribute content", which "includes social media,
user-generated content platforms, and AI content creation apps." Random or anonymous
chat was explicitly pulled under 1.2 on February 6, 2026. So an AI image, video, or
text generator is a 1.2 app even with no social feed.

Required, all four: a method for filtering objectionable material; a mechanism to
report offensive content with timely responses; the ability to block abusive users;
published contact information. The harms the text names now include "nonconsensual
intimate imagery" (Apple's announcement: including imagery generated with AI tools)
alongside IP infringement and anonymous bullying. Apps used primarily for
objectionable content (pornography, objectification, harassment, anonymous-chat abuse
patterns) can be removed outright regardless of moderation tooling. Creator-content
apps additionally need an age-restriction mechanism when content can exceed the app's
rating (1.2.1, 4.7.5 for embedded software).

Enforcement: the sentence added on October 16, 2026 reads "Developers who fail to
implement appropriate safeguards or fail to remediate identified violations after
notice may be considered in violation of the Developer Code of Conduct." The
surrounding text, in place since June 2026, says Apple will ask for removal plus a
plan to improve compliance, may pull the app until improvements are shown, and that
"Egregious or repeated behavior is grounds for immediate removal of your app from
the App Store, and from the Apple Developer Program." A missing moderation set in a
generator app is therefore an account-level risk, not only a rejection.

Static signals: chat, comments, profiles, upload, or generation features in code with
no report, block, or moderation surface anywhere; an AI generation endpoint (OpenAI,
Replicate, Stability, or similar, or an on-device generator) with no output
filtering (no safety classifier call, no blocklist, no Sensitive Content Analysis
framework use) and no report path for generated content.

## Gambling, contests, real-money gaming (5.3)

- Sweepstakes and contests must be sponsored by the developer, with official rules
  shown in the app and Apple not named as sponsor (5.3.1, 5.3.2).
- IAP may not be used to buy credit or currency for real-money gaming (5.3.3).
- Real-money gaming and lotteries need licensing and permissions in every location
  where the app is available, must be geo-restricted to those locations, and must be
  free on the App Store (5.3.4).
- If gambling-adjacent functionality arrives as embedded or streamed software (HTML5
  mini-games and similar), read the live 4.7 text during the audit; embedded software
  carries its own restrictions.
- Verify the age rating against the current questionnaire rather than assuming a
  fixed floor.

## VPN apps (5.4)

From the live 5.4 text: VPN services must use the `NEVPNManager` API and "may only be
offered by developers enrolled as an organization." The app must clearly declare what
user data is collected and how it is used, on a screen shown before any purchase or
use. VPN apps "may not sell, use, or disclose to third parties any data for any
purpose, and must commit to this in their privacy policy." Where a territory requires
a VPN license, the license information goes in the App Review Notes field.
Non-compliance is grounds for removal from the store and the developer program.

Static signals: `NetworkExtension` entitlement and API use; individual (not
organization) developer account is a submission-blocking mismatch the developer must
resolve outside the code.

## Cryptocurrency (3.1.5)

- Wallets: only from developers enrolled as an organization.
- Mining: not on device; cloud-based only.
- Exchanges: only on approved exchanges, only in regions where licensed.
- ICOs, crypto futures, and securities-like trading: only from established banks,
  securities firms, futures commission merchants, or other approved financial
  institutions.
- No offering cryptocurrency for completing tasks (downloads, referrals, social posts).

## Financial trading and money management (3.2.1(viii))

Apps for financial trading, investing, or money management should be submitted by the
financial institution performing the services, with required licensing and permissions
in every region of availability. A third-party developer shipping a trading front-end
under their own account is a structural rejection risk no code change fixes.

## Dating (no dedicated section)

No dedicated guideline; dating apps combine 1.2 UGC obligations with 4.3(b), which
names dating among the saturated categories where new submissions need a "meaningfully
different or improved experience." Expect both surfaces to be probed on first
submission. Regional age-verification law increasingly applies; that is legal exposure
beyond guideline text.
