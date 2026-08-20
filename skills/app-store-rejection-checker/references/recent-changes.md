# Recent changes: rejection surfaces added or reshaped 2024-2026

The moving parts. Each entry: what changed, since when, and what to check. Verified
against the live guidelines and the linked Apple pages on 2026-08-20. During an audit,
Step 1's news check covers anything newer than this file.

Contents:

1. [Privacy manifests and required-reason APIs (May 2024)](#privacy-manifests-and-required-reason-apis-may-2024)
2. [Third-party AI disclosure in 5.1.2(i) (November 2025)](#third-party-ai-disclosure-in-512i-november-2025)
3. [Age rating overhaul (2025, deadline January 2026)](#age-rating-overhaul-2025-deadline-january-2026)
4. [4.3(b) saturated-category tightening (June 2026)](#43b-saturated-category-tightening-june-2026)
5. [EU DMA terms (August-October 2026)](#eu-dma-terms-august-october-2026)
6. [External purchase links in the US (in flux)](#external-purchase-links-in-the-us-in-flux)
7. [Login services relaxation (January 2024)](#login-services-relaxation-january-2024)
8. [Account deletion (since June 2022, still a top trap)](#account-deletion-since-june-2022-still-a-top-trap)

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
