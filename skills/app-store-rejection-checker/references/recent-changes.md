# Recent changes: rejection surfaces added or reshaped 2024-2026

The moving parts. Each entry: what changed, since when, and what to check. Verified
against the live guidelines and the linked Apple pages on 2026-10-17. During an audit,
Step 1's news check covers anything newer than this file.

Contents:

1. [October 16, 2026 revision: hidden features, creative assets, UGC remediation](#october-16-2026-revision-hidden-features-creative-assets-ugc-remediation)
2. [2026 guideline revisions before October](#2026-guideline-revisions-before-october)
3. [Privacy manifests and required-reason APIs (May 2024)](#privacy-manifests-and-required-reason-apis-may-2024)
4. [Third-party AI disclosure in 5.1.2(i) (November 2025)](#third-party-ai-disclosure-in-512i-november-2025)
5. [Age rating overhaul (2025, deadline January 2026)](#age-rating-overhaul-2025-deadline-january-2026)
6. [4.3(b) saturated-category tightening (June 2026)](#43b-saturated-category-tightening-june-2026)
7. [EU DMA terms (August-October 2026)](#eu-dma-terms-august-october-2026)
8. [External purchase links in the US (in flux)](#external-purchase-links-in-the-us-in-flux)
9. [Login services relaxation (January 2024)](#login-services-relaxation-january-2024)
10. [Account deletion (since June 2022, still a top trap)](#account-deletion-since-june-2022-still-a-top-trap)

## October 16, 2026 revision: hidden features, creative assets, UGC remediation

Apple revised the guidelines on October 16, 2026 (changelog:
developer.apple.com/news/?id=92th7e5i; companion post on the intent:
developer.apple.com/news/?id=vvzx6cul). The page footer now reads "Last Updated:
October 16, 2026"; the previous revision was June 8, 2026. What moved, verified
against the live text and an archived copy of the previous text:

- **2.5.19 (new) takes over hidden, dormant, and undocumented features from 2.3.1.**
  Live text: "Don't include any hidden, dormant, or undocumented features in your app;
  your app's functionality should be clear to end users and App Review. Attempting to
  hide undocumented features in your app binary, such as through code obfuscation or
  other methods, is grounds for removal of your app from the App Store and termination
  of your developer account." The first sentence was lifted out of 2.3.1(a) unchanged;
  the second sentence is new and names obfuscation and account termination. Apple's
  companion post frames the rule as the "longstanding prohibition on apps
  misrepresenting what they do during the App Review process and then changing after
  approval," with "adding features or functionality that weren't included in the
  version of the app that was submitted" as the example, and removal from the
  Developer Program as the consequence. Cite **2.5.19**, not 2.3.1, for hidden-feature
  findings from now on. What the text does not say: it does not name remote
  configuration, feature flags, or A/B testing, in either direction. Data-driven
  changes to copy, layout, or pricing presentation of a paywall whose code shipped in
  the reviewed binary are not what the sentence describes; features that ship dark
  and switch on after approval are. Check procedure: static-checks.md section 13.
- **2.3.1(a) keeps the review-notes and misleading-marketing rules.** It still
  requires all new features and product changes to be "described with specificity in
  the Notes for Review section" and "accessible for review", and still makes
  misleading marketing grounds for removal and termination. The hidden-feature
  sentence is the only thing that left.
- **2.3.14 (new): creative assets.** The App Store now accepts promotional assets
  beyond app-in-use screenshots and previews (product page header, search results;
  announced October 5, 2026, with an Asset Library in App Store Connect). Live text:
  creative assets "must pertain to your app's content and functionality or
  overarching brand, follow standard metadata guidelines, and be appropriate for all
  audiences" and "should not include inappropriate content, references to other apps
  with an intent to mislead, unverifiable product claims, or specific or discounted
  pricing information." Apple's detailed rules live at
  developer.apple.com/app-store/asset-best-practices/. Audit angle:
  metadata-and-review-notes.md.
- **1.2 widens scope and adds a remediation-after-notice clause.** The opening now
  covers "apps that enable users to create, generate, upload, share, or otherwise
  contribute content", naming "social media, user-generated content platforms, and AI
  content creation apps", and lists "nonconsensual intimate imagery" among the harms
  (the companion post says this includes imagery generated with AI tools). New
  sentence: "Developers who fail to implement appropriate safeguards or fail to
  remediate identified violations after notice may be considered in violation of the
  Developer Code of Conduct." The four required mechanisms (filtering, reporting with
  timely responses, blocking, published contact information) are unchanged. An AI
  image or text generator with no moderation set is now squarely a 1.2 app. Audit
  angle: special-categories.md, user-generated content.
- **3.1.1(a): links go to "a permitted destination".** The separate Music Streaming
  Services Entitlements bullet was folded into the StoreKit External Purchase Link
  bullet; links now point to "a permitted destination" rather than "the developer's
  website", "approved destinations may vary by region", and the entitlements apply
  "only in specific storefronts of the App Store, and only on certain platforms in
  each of those storefronts." The live text remains the only safe source for external
  link mechanics (see the US section below).
- **Section 5 intro:** apps facilitating human trafficking or child exploitation
  "will be removed from the App Store and blocked from alternative distribution", in
  addition to authorities being notified.
- **Unannounced edits worth knowing:** 2.3.3 (screenshots), 4.1(c) (copycats), and
  5.1.1(ii) (purpose strings) each gained a "Learn more" link to App Review help
  pages (developer.apple.com/help/app-review/); the 2.3.3 help page states App Review
  approves only submissions "where more than half of screenshots show the app in
  use". 3.1.3(f) was reworded to say free stand-alone companion apps to paid
  web-based VoIP, cloud storage, email, or web hosting tools do not need IAP. "e.g."
  became "such as" or "for example" across the document, so quote wording from the
  live fetch rather than from older copies. "Before You Submit" gained "Provide
  required documentation for apps and features that require regulatory approval."

Context Apple gave with the revision: in the previous year it "prevented over 150K
apps with safety issues from reaching customers and terminated over 193K developer
accounts for engaging in fraud and abuse", tracing violations to actors "working to
conceal harmful content that only becomes active once an app is approved." That is
the enforcement posture behind 2.5.19: an app whose behavior changes after approval
is treated as fraud, not as a metadata nit.

Separate but adjacent, from the October 5, 2026 iPhone Duo announcement: "Starting
April 2027, any apps or games submitted will need to include screenshots for iPhone
Duo." Not a guideline change, but a submission blocker from that date.

## 2026 guideline revisions before October

Two earlier 2026 revisions, both verified from Apple's news posts:

- **February 6, 2026** (developer.apple.com/news/?id=d75yllv4): "apps with random or
  anonymous chat are subject to the 1.2 User-Generated Content guideline." A
  random-chat or anonymous-chat feature needs the full 1.2 moderation set.
- **June 8, 2026** (developer.apple.com/news/?id=a233fmpw): the 4.3(b) tightening
  covered in its own section below, plus: the Introduction's kid and teen safety
  guidance was revised; 1.2 gained a paragraph on developer responsibility for
  violating content (the October revision extended it with the remediation clause
  above); 4.3(a) and 4.3(b) gained examples; and 4.5.3 now says Live Activities may
  not be used "to spam, phish, or send unsolicited messages". The same post revised
  the Developer Program License Agreement (Foundation Models framework terms, a new
  AI and machine learning subsection 3.3.11, identity-verification duties in 3.1 and
  14.8); the agreement is behind sign-in and not audited by this skill.

## Privacy manifests and required-reason APIs (May 2024)

Since May 1, 2024, App Store Connect requires approved reasons declared in a privacy
manifest for a defined set of APIs (file timestamps, UserDefaults, system boot time,
disk space, active keyboards), applying transitively to third-party SDKs; SDKs on
Apple's commonly-used list must ship their own privacy manifests, and signatures are
additionally required when a listed SDK is added as a binary dependency. Missing
declarations block the upload. Source: developer.apple.com/news/?id=3d8a9yyh. Check procedure:
static-checks.md section 2.

## Third-party AI disclosure in 5.1.2(i) (November 2025)

The live 5.1.2(i) text now reads: "You must clearly disclose where personal data will
be shared with third parties, including with third-party AI, and obtain explicit
permission before doing so." The scope is personal data shared with third-party AI
services: an app sending personal data to an external LLM or ML API needs a
disclosed, consented data flow plus matching privacy labels and policy. First-party
processing and non-personal input are not automatic triggers, but the line deserves a
look in any AI-integrated app. New wording, enforcement still maturing. Check
procedure: static-checks.md section 10.

## Age rating overhaul (2025, deadline January 2026)

Rating tiers changed from 4+/9+/12+/17+ to 4+, 9+, 13+, 16+, 18+. The new tiers
surface on OS 26 and later (iOS/iPadOS/macOS/tvOS/visionOS/watchOS 26); devices on
older OS versions still display the old 4+/9+/12+/17+ tiers. A new mandatory
questionnaire (in-app controls, capabilities, medical and wellness topics, violent
themes) had to be answered by January 31, 2026 to keep submitting updates. The
questionnaire explicitly requires factoring in how "AI assistants and chatbot
functionality" affect the frequency of sensitive content. Note when auditing answers:
Messaging and Chat, User-Generated Content, and Social Media are three **separate**
capabilities in the questionnaire (direct user-to-user communication; broad
distribution of user-created content; redistribution or amplification through a
social feed); a chat feature without a UGC declaration is not automatically a
mismatch. Sources: developer.apple.com/news/?id=ks775ehf and
developer.apple.com/help/app-store-connect/reference/age-ratings/. Audit angle:
rating answers vs actual features (metadata-and-review-notes.md).

## 4.3(b) saturated-category tightening (June 2026)

4.3(b) now rejects apps "indistinguishable from what's already widely available" and
names categories closed to me-too submissions: dating, flashlight, sound effects,
wallpaper, simple timers, fortune telling, unless the app offers a "meaningfully
different or improved experience." The same text calls out drinking games, Kama Sutra,
fart, and burp apps as low-effort. What the June 8, 2026 revision actually added: "We
may remove these apps from the App Store going forward if they are not updated,
improved, or do not attract customers." That turns 4.3(b) from a submission-time
filter into an ongoing removal ground, so already-shipped apps in these categories are
exposed, not just new submissions. Community reports say generic AI chatbot, image
generator, and summarizer apps are treated the same way in practice. Audit angle: the
differentiation judgment in Step 4.

## EU DMA terms (August-October 2026)

Apple's DMA terms were updated August 18, 2026, fully effective October 1, 2026
(developer.apple.com/support/dma-and-apps-in-the-eu/). Points that create rejection or
compliance surfaces for EU-distributed apps:

- External purchase links carry a Store Services Commission (15% standard, 10% for
  Small Business Program and certain partner programs) on sales within 7 days of the
  link tap; a 5% Core Technology Commission applies to sales of digital goods and
  services in apps distributed via alternative marketplaces or Web Distribution. The
  EUR 10M global / EUR 1M lifetime EU waiver is narrow: it applies only to qualifying
  small **marketplace operators**, and only to the fees their own marketplace app
  charges (download price or subscription to access the marketplace), not to
  alternative-distribution sales generally.
- Kids Category apps cannot offer out-of-app purchase links, and purchase flows using
  alternative payment must sit behind a parental gate; for under-13 users (age varies
  by storefront), out-of-app offers are not permitted at all.
- Apps distributed via alternative marketplaces or web distribution must pass
  notarization (accuracy, functionality, safety, security, privacy review), a distinct
  rejection surface from App Review.
- From October 1, 2026, a legal entity or establishment in the EU is no longer
  required to operate an alternative app marketplace or use Web Distribution
  (eligibility criteria still apply: financial-stability or scale bars per the
  linked page). This is scoped to those two distribution channels, not a general
  App Store change.

Rates and mechanics here change with litigation and regulation; re-check the linked
page whenever an EU app links out for purchases.

## External purchase links in the US (in flux)

US storefront rules for linking to external purchases changed repeatedly through
2025 (court orders, then a December 2025 appellate reversal allowing Apple to charge
commission on external-link purchases). Do not state current US mechanics from any
cached source, this file included: read the live 3.1.1(a) text and Apple's StoreKit
external purchase entitlement documentation at audit time.

## Login services relaxation (January 2024)

4.8 no longer mandates Sign in with Apple specifically. Apps using third-party or
social login for the primary account must offer another login service that limits data
collection to name and email, allows keeping the email private, and does not collect
app interactions for advertising without consent; five exemption classes exist (see
static-checks.md section 4). Findings should say "privacy-protecting login option
required", not "Sign in with Apple required".

## Account deletion (since June 2022, still a top trap)

In force since June 30, 2022, and still among the most-hit requirements because
scaffolded and generated codebases wire signup without deletion: if the app supports
account creation, it must offer in-app account deletion, full deletion rather than
deactivation, not region-restricted, and not routed through support except for highly
regulated industries. Source:
developer.apple.com/support/offering-account-deletion-in-your-app/. Check procedure:
static-checks.md section 7.
