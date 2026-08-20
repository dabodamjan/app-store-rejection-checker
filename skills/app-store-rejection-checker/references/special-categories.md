# Special categories: extra requirements beyond the base guidelines

Apply the matching section when Step 2 classifies the app into one of these. Each
carries requirements that are near-certain rejections when missed. Guideline numbers
verified against the live guidelines on 2026-08-20; quotes for 5.4 taken verbatim from
the live text. Always re-read the matching live section during the audit; these are
summaries.

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

- Apps that could provide inaccurate data or be used for diagnosing or treating
  patients get extra scrutiny; accuracy claims need disclosed data and methodology,
  and unvalidated claims get the app rejected.
- Explicitly not permitted: apps claiming to take x-rays or measure blood pressure,
  body temperature, blood glucose, or blood oxygen using only device sensors.
- Apps should remind users to check with a doctor before medical decisions.
- If the app has regulatory clearance (for example FDA), submit a link to that
  documentation with the app.
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

## User-generated content (1.2)

Required, all four: a method for filtering objectionable material; a mechanism to
report offensive content with timely responses; the ability to block abusive users;
published contact information. Apps used primarily for objectionable content
(pornography, objectification, harassment, anonymous-chat abuse patterns) can be
removed outright regardless of moderation tooling. Creator-content apps additionally
need an age-restriction mechanism when content can exceed the app's rating (1.2.1,
4.7.5 for embedded software).

Static signals: chat, comments, profiles, or upload features in code with no report,
block, or moderation surface anywhere.

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
