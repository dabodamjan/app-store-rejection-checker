# Static checks: the mechanically verifiable surface

Checks that can be decided from files in the repository: build settings, plists,
entitlements, lockfiles, and source. Guideline numbers were verified against the live
guidelines on 2026-11-05 (text dated October 16, 2026); re-verify against the Step 1
fetch when citing them.

Evidence standard for **validator-certain** (defined in SKILL.md Step 6): the upload
validators run against the built product, not the repository. Conditional compilation,
unused targets, files excluded from the shipping target, and packaging-time manifests
can all make source or lockfile evidence wrong about what ships. So a finding reaches
validator-certain only when the evidence reflects the merged shipping target: an
archive, `.app`, or `.ipa`, or build settings resolved for that target. From source and
lockfiles alone, report the same finding with the marked sub-form from SKILL.md Step 6:
`review-risk (validator-blocking if shipped as-is; needs build verification)`. The
sub-form is the review-risk level with a qualifier, not a fourth confidence level.

Contents:

1. [Permission purpose strings](#1-permission-purpose-strings)
2. [Privacy manifest (PrivacyInfo.xcprivacy)](#2-privacy-manifest-privacyinfoxcprivacy)
3. [Entitlements consistency](#3-entitlements-consistency)
4. [Login services (4.8)](#4-login-services-48)
5. [Export compliance](#5-export-compliance)
6. [Payments and IAP wiring (3.1.x)](#6-payments-and-iap-wiring-31x)
7. [Account deletion (5.1.1(v))](#7-account-deletion-511v)
8. [App Tracking Transparency (5.1.2(i))](#8-app-tracking-transparency-512i)
9. [Analytics consent (5.1.1(ii))](#9-analytics-consent-511ii)
10. [Third-party AI data flows (5.1.2(i))](#10-third-party-ai-data-flows-512i)
11. [Web-wrapper fingerprints (4.2)](#11-web-wrapper-fingerprints-42)
12. [Dynamic code and deprecated APIs](#12-dynamic-code-and-deprecated-apis)
13. [Hidden, dormant, and undocumented features (2.5.19)](#13-hidden-dormant-and-undocumented-features-2519)
14. [Ratings prompts (5.6.1, 3.2.2(x))](#14-ratings-prompts-561-322x)

## 1. Permission purpose strings

Missing purpose strings are rejected by Apple's automated upload validation ("Missing
purpose string" / ITMS-90683). The validation fires even when the triggering API is
reached only through a linked third-party SDK, not the app's own code, so check
dependencies, not just source. A confirmed miss is **validator-certain** only when the
trigger is confirmed in the shipping target (see the evidence standard above); from
source or lockfile evidence alone, report it as review-risk needing build
verification.

Method: list linked frameworks and pods (`Podfile.lock`, `Package.resolved`, project
build phases), grep source for trigger APIs, then confirm the merged Info.plist (the
`Info.plist` files plus any `INFOPLIST_KEY_*` build settings in `project.pbxproj`)
contains a **present and non-empty** matching key. An empty or boilerplate string
("This app needs camera access") passes validation but is a review-risk under 5.1.1:
the string must explain the actual use.

| Info.plist key | Trigger APIs / frameworks |
|---|---|
| `NSCameraUsageDescription` | `AVCaptureDevice`, `UIImagePickerController` (camera source), AVFoundation capture |
| `NSMicrophoneUsageDescription` | `AVAudioSession` recording, `AVCaptureDevice` audio |
| `NSLocationWhenInUseUsageDescription`, `NSLocationAlwaysAndWhenInUseUsageDescription` | CoreLocation, `CLLocationManager` |
| `NSPhotoLibraryUsageDescription` (read), `NSPhotoLibraryAddUsageDescription` (write-only) | direct PhotoKit library access: `PHPhotoLibrary`, `PHAsset` fetches, but **not** `PHPickerViewController` (see note below) |
| `NSContactsUsageDescription` | Contacts, ContactsUI |
| `NSUserTrackingUsageDescription` | AppTrackingTransparency, `ATTrackingManager.requestTrackingAuthorization`, SDKs configured to use the IDFA (SDK presence alone is not proof; check the configuration) |
| `NSHealthShareUsageDescription`, `NSHealthUpdateUsageDescription` | HealthKit, `HKHealthStore` |
| `NSBluetoothAlwaysUsageDescription` | CoreBluetooth, `CBCentralManager`, `CBPeripheralManager` |
| `NSFaceIDUsageDescription` | LocalAuthentication with Face ID biometry |
| `NSSpeechRecognitionUsageDescription` | Speech framework, `SFSpeechRecognizer` |
| `NSCalendarsFullAccessUsageDescription`, `NSRemindersFullAccessUsageDescription` | EventKit |

The table is the common core, not the full list; if the code uses a permission-gated
API not listed here, check Apple's documentation for the matching key.

Picker exemption: `PHPickerViewController` (and the SwiftUI `PhotosPicker`) runs out of
process and requires **no** photo-library purpose string or permission. An app whose
only photo access goes through the picker should not be flagged for a missing
`NSPhotoLibraryUsageDescription`; only direct PhotoKit library access triggers it.
Similarly, `UIImagePickerController` with the photo-library source no longer requires
the key, but its camera source still requires `NSCameraUsageDescription`.

## 2. Privacy manifest (PrivacyInfo.xcprivacy)

Enforced at upload since **May 1, 2024**: apps using "required reason" APIs must declare
an approved reason in a privacy manifest, or App Store Connect refuses the upload
(source: developer.apple.com/news/?id=3d8a9yyh). Findings here are **validator-certain**
only when the API use is confirmed in the shipping target (evidence standard at the top
of this file); use in source or a lockfile dependency alone is review-risk needing
build verification, since the code may not be compiled into the submitted binary.

Manifest structure (root plist keys):

- `NSPrivacyTracking` (bool) and `NSPrivacyTrackingDomains` (array; must be non-empty
  when tracking is true)
- `NSPrivacyCollectedDataTypes` (array of dicts: type, linked, tracking, purposes)
- `NSPrivacyAccessedAPITypes` (array of dicts: category + reason codes)

Required-reason API categories and their triggers:

| Category | Triggered by |
|---|---|
| `NSPrivacyAccessedAPICategoryFileTimestamp` | file creation/modification date APIs on `FileManager`/`NSURL`, `stat` family |
| `NSPrivacyAccessedAPICategoryUserDefaults` | `UserDefaults` / `NSUserDefaults` |
| `NSPrivacyAccessedAPICategorySystemBootTime` | `ProcessInfo.systemUptime`, `mach_absolute_time` |
| `NSPrivacyAccessedAPICategoryDiskSpace` | volume capacity keys, `systemFreeSize`, `statfs` |
| `NSPrivacyAccessedAPICategoryActiveKeyboards` | `UITextInputMode.activeInputModes` |

Checks:

1. Any category's API used but not declared in `NSPrivacyAccessedAPITypes`: upload
   will be flagged (missing API declaration). Scan the app's own source and any
   dependency source actually present in the repo (vendored SDKs, checked-in package
   sources). Linked SDKs usually cannot be scanned from a bare repository: SPM
   checkouts live in DerivedData, and CocoaPods sources may not be committed. For
   those, the check falls to Xcode's archive upload validation; say so in the
   report's "Not checked" section rather than implying the SDK half was scanned.
2. Reason codes: do not validate codes from memory. Fetch the current approved list
   from Apple's "Describing use of required reason API" page
   (developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api
   and its per-category subpages) and check declared codes against it. Examples of
   currently valid codes: `C56D.1` (UserDefaults), `35F9.1` (SystemBootTime). These
   pages are rendered client-side, so a plain fetch may return an empty shell. When
   it does, fall back to a web search for the code, corroborated by a primary source
   (an Apple page or an Apple staff forum answer); if no corroboration is found,
   state in the report that declared reason codes were not validated this run.
3. Third-party SDKs on Apple's commonly-used-SDKs list must ship their own privacy
   manifest, and the **signature** requirement applies to SDKs added as binary
   dependencies (XCFrameworks), not to those compiled from source. A lockfile entry
   names the dependency but proves nothing about what manifest or signature ends up in
   the bundle; treat it as a prompt to inspect the resolved artifact (the checked-out
   package or downloaded XCFramework, or the built app), not as evidence in itself.
   The list is linked from the same Apple page.
4. `NSPrivacyTracking` true with empty `NSPrivacyTrackingDomains`, or tracking SDKs
   present with `NSPrivacyTracking` false: inconsistency worth flagging (review-risk).

An app with no manifest at all and no required-reason API use is not automatically
rejected; flag it as a gap only when triggers exist.

## 3. Entitlements consistency

Read `*.entitlements` and, when a built product is available,
`codesign -d --entitlements :- <App>.app`.

- `aps-environment` present but no push registration in code
  (`registerForRemoteNotifications`, `UNUserNotificationCenter`), or push code present
  with no entitlement: the latter is flagged by upload validation ("Missing Push
  Notification Entitlement").
- Entitlements that the code never justifies: cleanup findings, low severity. Common
  examples, not an exhaustive list: app groups, iCloud, Sign in with Apple, `siri`
  (justified by SiriKit or App Intents adoption), App Attest. Apply the same
  present-vs-used comparison to other developer-selected capability keys, but not to
  signing-generated ones: code signing injects entitlements no source ever references
  (`application-identifier`, `com.apple.developer.team-identifier`, `get-task-allow`,
  `beta-reports-active`), and flagging those is noise.
- Shared entitlements files in multi-platform projects can carry keys for another
  platform (macOS sandbox keys are the common case). Attribute keys to the right
  target before flagging them, and remember other platforms are out of scope for
  this audit (SKILL.md Step 2).

## 4. Login services (4.8)

If the app uses a third-party or social login service (Facebook Login, Google Sign-In,
Log in with X, Sign In with LinkedIn, Login with Amazon, WeChat Login) to set up or
authenticate the user's **primary** account, guideline 4.8 requires offering another
login service that: limits data collection to name and email, lets users keep their
email private, and does not collect app interactions for advertising without consent.
Sign in with Apple satisfies this but since January 2024 is not the only option; word
findings as "needs a privacy-protecting login option", not "needs Sign in with Apple".

Static signal: social-login SDK in the lockfile plus login UI in code, with no
`com.apple.developer.applesignin` entitlement and no `AuthenticationServices` /
`ASAuthorizationAppleIDProvider` usage and no other qualifying service. That is a
**review-risk** finding. 4.8 lists exemptions (own-account-system-only apps, education
and enterprise apps, government or industry-backed ID, clients for a specific
third-party service); check them before flagging.

## 5. Export compliance

`ITSAppUsesNonExemptEncryption` (Info.plist):

- Absent: App Store Connect asks the export compliance question on every submission.
  A speed bump, not a rejection; flag as low severity.
- Do not infer export status from framework names. CryptoKit, CommonCrypto, and
  HTTPS/TLS are OS-provided cryptography, which is generally exempt; their presence
  does not establish non-exempt encryption, so never flag `false` as a
  misdeclaration on that basis alone. The signal worth flagging is **proprietary or
  bundled** cryptography (a custom algorithm implementation, bundled OpenSSL or
  similar): that is what the developer must answer Apple's export compliance
  questionnaire about. When in doubt, point the developer at the questionnaire and
  the OS-provided vs proprietary distinction rather than asserting a status.

## 6. Payments and IAP wiring (3.1.x)

- **Digital goods sold outside IAP (3.1.1)**: payment SDKs (Stripe, PayPal, Braintree)
  or checkout URLs wired to unlock digital content, features, or subscriptions is the
  classic hard rejection. Physical goods and services consumed outside the app are the
  legitimate use of those SDKs; classify what is actually being sold before flagging.
  3.1.1 also bans unlock mechanisms like license keys, QR codes, or cryptocurrency for
  digital content.
- **External purchase links (3.1.1(a))**: rules for linking out to other purchase
  methods changed repeatedly through 2024-2026 by storefront (US litigation, EU DMA)
  and depend on entitlements and commission terms that shift. Do not assert current
  mechanics from these references: if the app links out to purchase digital goods,
  read the live 3.1.1(a) text and Apple's StoreKit external purchase documentation at
  audit time, and flag the surface as high-attention.
- **Restore purchases (3.1.1)**: non-consumable IAP or subscriptions present (StoreKit
  in code, products configured) with no restore mechanism in the UI: review-risk.
- **Subscription disclosure (3.1.2(c))**: before asking users to subscribe, the app
  must clearly say what the user gets for the price, and the paywall must carry the
  disclosures in the checklist below. Check the paywall screen content, not only the
  product configuration.
- **Loot boxes (3.1.1)**: randomized paid items require disclosing the odds of each
  item type before purchase.

### Paywall disclosure checklist (3.1.2(c), Schedule 2, 2.3.1(a), 5.6)

Live 3.1.2(c) text: "Before asking a customer to subscribe, you should clearly describe
what the user will get for the price. ... Ensure you clearly communicate the
requirements described in Schedule 2 of the Apple Developer Program License
Agreement." The binding detail lives on two other public Apple pages, both fetched
2026-11-05:

- Schedule 2, section 3.8(b) of the Developer Program License Agreement
  (developer.apple.com/support/terms/apple-developer-program-license-agreement/):
  "clearly and conspicuously disclose" the title of the subscription, the "Length of
  subscription", and the "Price of subscription, and price per unit if appropriate";
  "Links to Your Privacy Policy and Terms of Use must be accessible within Your
  Licensed Application."
- Apple's auto-renewable subscriptions page (developer.apple.com/app-store/subscriptions/):
  the sign-up screen must include the "Subscription name and duration, and the content
  or services provided during the subscription period", the "Full renewal price, shown
  clearly and prominently, and localized in available currencies", and "A way for
  current subscribers to sign in or restore purchases". On layout: "In the purchase
  flow, the amount that will be billed must be the most prominent pricing element in
  the layout", and breakdown prices "should be displayed in a subordinate position and
  size to the annual price." On trials: "In the purchase flow for a free trial, clearly
  indicate how long the free trial lasts and the price billed once the free trial is
  over."

Read the paywall view code and its string resources, then check each item. Rejections
in this family quote the subscriptions page back almost word for word; a developer
thread from December 2019 (developer.apple.com/forums/thread/127394) carries the
standard text: the app "offers a free trial or introductory period but does not make
it clear that a payment will be automatically initiated for the next subscription
period."

- **Offer and post-offer price disclosure** (review-risk): when a product carries an
  introductory offer, the screen must state what the offer is and what follows it.
  For a free trial the subscriptions page wants the trial length and "the price
  billed once the free trial is over"; for paid introductory pricing (the page's
  "pay as you go" and "pay up front" types) the discounted price and its period, and
  the standard renewal price after it. Judge the whole presentation, not one
  element: a trial line and the renewal price that sit together and read as one
  statement pass; a trial badge in one corner and a bare price elsewhere, with
  nothing joining them, is the shape the 2019 rejection text above describes. Prefer
  copy rendered from the product's offer data; a hard-coded sentence is not a
  finding on its own, but it goes stale the moment the offer changes in App Store
  Connect, so note it.
- **Billed amount most prominent** (review-risk): a per-month or per-day breakdown of
  an annual price rendered larger than the annual amount, a price with no period
  attached, or a primary button that sells the trial ("Try for free", "Continue with
  free trial") while the billed amount sits small or elsewhere. The amount Apple will
  charge, with its period, is the largest price on the screen.
- **Trial copy that depends on an eligibility check** (judgment-call; an inference,
  so give the test): showing trial copy only to customers who are eligible is
  correct, and StoreKit's `isEligibleForIntroOffer` returns a plain Bool, so the
  StoreKit path has no unknown state to mishandle. The risk sits in the code around
  it: a subscription SDK's eligibility call that can return unknown or fail, a
  cached result from another account, or a product fetch that errors, when every
  path other than "eligible" falls through to copy that shows no offer and no
  disclosure. Then the reviewer's fresh sandbox account can see a screen with no
  trial while Apple's purchase sheet offers one: the mismatch the rejection text
  above describes. Flag it only when the code shows that path (an error or unknown
  branch that silently renders the no-offer paywall), never for the eligibility
  check itself. Whether a reviewer hits it cannot be read from the repository, so
  report it with the test: run the paywall on a fresh sandbox tester and compare
  what the screen says with what the purchase sheet shows. Fix: keep the
  eligibility-based presentation and give the unknown and error paths an honest
  fallback that still carries the disclosure ("29.99 a year, renews automatically;
  includes a 7-day free trial if you have not had one").
- **Trial toggle** (review-risk, reported developer experience): a switch on the
  purchase screen that adds or removes a free trial. Apple published no rule, but
  subscription vendors report 3.1.2 rejections since mid-January 2026 quoting "The
  purchase screen includes a toggle to add or remove a free trial from the
  subscription purchase" (RevenueCat, revenuecat.com/blog/growth/rip-toggle-paywall,
  February 9, 2026, updated April 23, 2026; Adapty,
  adapty.io/blog/your-toggle-paywall-is-about-to-get-rejected/, February 11, 2026).
  Flag the control itself; the fix is separate plans with the trial attached to one of
  them.
- **Terms and privacy links** (review-risk): Schedule 2 requires both to be
  "accessible within Your Licensed Application"; links only on the website or in the
  store listing do not satisfy it. The agreement does not say where in the app. A
  link from the paywall itself is the recommended placement, since that is the
  screen the reviewer judges, not a requirement.
- **False or tricky prices** (cite 2.3.1(a) and 5.6): 2.3.1(a) makes "promoting a
  false price, whether within or outside of the App Store" grounds for removal "and
  termination of your developer account"; 5.6 says apps should never "trick them
  into making unwanted purchases ... raise prices in a tricky manner". The
  repository cannot show that a price was never charged: a strikethrough may be last
  quarter's real price, a first-launch countdown may back a genuine introductory
  offer, and "save 40%" usually compares the annual plan with twelve months of the
  monthly one, which the subscriptions page allows in a subordinate position. So the
  signals below are questions for the developer, reported as **judgment-call** until
  answered: a strikethrough or "was" price that matches no configured product price
  (ask for the price history); a permanent discount badge (ask what the higher price
  is and when it is in force); "today only" or countdown copy driven by a constant,
  a first-launch timestamp, or a remote flag (ask which configured offer it
  reflects); price text that does not come from the product's own price data. A
  comparison the developer cannot back with a configured price or offer becomes
  **review-risk** and account-level, with 3.1.2(a) cited as below. "Outside of the
  App Store" means screenshots, ads, and creative assets count too
  (metadata-and-review-notes.md). One developer report ties an account termination
  for "misleading auto-renewable subscription practices" to a paywall whose owner
  later found a reference-price strikethrough and a hard-to-dismiss win-back screen
  (developer.apple.com/forums/thread/839800, July 2026; the developer's own
  attribution, Apple named no screen).
- **Scam and bait-and-switch language in 3.1.2(a)**: "Apps that attempt to trick users
  into purchasing a subscription under false pretenses or engage in bait-and-switch and
  scam practices ... will be removed from the App Store and you may be removed from the
  Apple Developer Program." Cite it with 5.6 on a false-price finding the developer
  could not back; it is what makes that finding account-level rather than a paywall
  nit.

Calibration: a hard paywall with no free tier is not a finding; nothing in the live
3.1.x text requires a free tier. A close control that is hard to find is a 5.6
"trick" signal only in combination with the pricing patterns above; on its own it is
a usability note, not a guideline finding.

## 7. Account deletion (5.1.1(v))

Live text: "If your app supports account creation, you must also offer account deletion
within the app." In force since June 30, 2022. Deactivation-only is explicitly not
enough; deletion cannot be limited by region; routing through a support call or email
is only acceptable for apps in highly regulated industries (5.1.1(ix)). Source:
developer.apple.com/support/offering-account-deletion-in-your-app/.

Static signal: auth SDK or signup flow present (Firebase Auth, Supabase, Cognito,
custom register endpoints) with no deletion code path (search for delete-account UI,
`deleteUser`, account-deletion endpoints). Account creation with no deletion path is a
**review-risk** finding with high real-world frequency; scaffolded auth without a
delete flow is a known failure mode of generated codebases.

## 8. App Tracking Transparency (5.1.2(i))

Live text: "You must receive explicit permission from users via the App Tracking
Transparency APIs to track their activity."

Static signals:

- Ad, attribution, or analytics SDKs configured to use the IDFA with no
  `ATTrackingManager.requestTrackingAuthorization` call and no
  `NSUserTrackingUsageDescription`: review-risk needing build verification. Apple
  documents the key as required when the ATT API is called, but a linked SDK's API
  reference alone is not a deterministic upload block, so the missing key stays
  review-risk rather than validator-certain.
  SDK presence alone is not proof of tracking; check whether the SDK is actually
  configured to use the IDFA.
- Tracking initialized before the ATT prompt result is known: review-risk.
- Gating app functionality or rewards on enabling tracking (or push or location):
  prohibited; judgment on the flow, evidence from code.

## 9. Analytics consent (5.1.1(ii))

Live text: "Apps that collect user or usage data must secure user consent for the
collection, even if such data is considered to be anonymous at the time of or
immediately following collection." The same subsection requires "an easily accessible
and understandable way to withdraw consent," and then carves out a lawful-basis
exception: "Apps that collect data for a legitimate interest without consent by
relying on the terms of the European Union's General Data Protection Regulation
("GDPR") or similar statute must comply with all terms of that law."

Distinct from section 8: ATT governs cross-app tracking via the IDFA; 5.1.1(ii)
covers plain analytics collection with no IDFA involved, which is why an app can
pass every ATT check and still fail here. The common indie-app shape is an analytics
SDK that starts collecting at launch with no consent step. Static signals, each
verifiable from source:

- An analytics SDK initialized with collection on by default (Firebase Analytics,
  Mixpanel, Amplitude, PostHog, or similar), and
- no consent gate on the init or send path (no stored consent flag checked before
  initializing the SDK or sending events), and
- no consent surface in onboarding or first-run UI.

Calibration: first check for a declared lawful basis. The exception quoted above
conditions on complying "with all terms of that law", and this audit cannot
adjudicate GDPR compliance. So a declaration of legitimate interest per GDPR or a
similar statute (typically stated in the privacy policy) moves the finding out of
missing-consent territory but never earns a silent pass: still note the declared
basis in the report, state that its GDPR compliance was not and cannot be verified
here, and flag any inconsistency between the declared basis and the actual data
flows (for example, a policy declaring legitimate-interest analytics while the SDK
sends identifying or sensitive events). Absent a declared basis: collection on by default with a
Settings opt-out is extremely common in shipping apps, and Apple rarely enforces
this sentence against anonymous first-party analytics; enforcement attention goes to
ATT and IDFA tracking (section 8). So all three signals together land at
**judgment-call** by default: the guideline text is real and the finding is worth
reporting, but a default review-risk rating here would flag most real apps. Raise to
review-risk only with aggravating factors: the analytics carry sensitive or
identifying data, the privacy labels or policy claim no collection happens, or the
UI misrepresents the actual behavior to the user.

That misrepresentation signal needs verification before it can be claimed. A
settings toggle that reads as off while the SDK collects is a misreport only if the
stored default the toggle actually reads is false. Before concluding that, check
`register(defaults:)` calls (often in an analytics manager's initializer), defaulted
property wrappers such as `@AppStorage` with a default value, and SDK-side defaults.
A `bool(forKey:)` read that falls back to false is not evidence on its own;
registered defaults commonly set the key to true, in which case the toggle agrees
with the SDK. Also check withdrawal, scoped to the basis of collection: where the
app collects on the strength of consent, the withdraw-consent sentence binds
regardless of the finding's rating, and a consent or opt-out surface with no way to
turn collection off afterwards is incomplete under the same subsection. Under a
declared no-consent lawful basis there is no consent to withdraw; the equivalent
check is whether the app honors the opt-out or objection mechanism its own privacy
policy describes.

## 10. Third-party AI data flows (5.1.2(i))

Live text (added November 2025): "You must clearly disclose where personal data will be
shared with third parties, including with third-party AI, and obtain explicit
permission before doing so."

The third-party-AI sentence triggers on **personal data shared with a third-party AI
service**, not any user content sent to any LLM. But 5.1.2(i) is broader than that
sentence: its opening line bars using, transmitting, or sharing personal data without
permission, so first-party or self-hosted model processing of personal data still
needs consent and privacy-policy coverage under the same guideline; it just isn't a
third-party-sharing finding. Flows whose input carries no personal data fall outside
5.1.2(i); judge, don't auto-flag. Static signals:
third-party AI API clients or endpoints in code (api.openai.com, api.anthropic.com,
generativelanguage.googleapis.com, openrouter.ai, and similar, or their SDKs) carrying
data that identifies or relates to the user: free-form user text, photos, audio,
health data, contacts. For those flows verify: is the sharing disclosed in-app before
it happens, is there an explicit consent step, and do the privacy policy and App Store
privacy labels cover it? Absent consent flow: **review-risk**, rising, since the
wording is new and explicit.

If AI features can produce sensitive content, also check the age rating answer set
(see metadata-and-review-notes.md): the questionnaire asks about AI assistant and
chatbot impact on sensitive-content frequency.

Privacy labels follow the provider's retention, not the app's intent. Apple's privacy
details page (developer.apple.com/app-store/app-privacy-details/) defines "collect" as
"transmitting data off the device in a way that allows you and/or your third-party
partners to access it for a period longer than what is necessary to service the
transmitted request in real time." An AI provider that retains inputs under its
default terms (common, for abuse monitoring, for days or longer) makes the photos,
audio, or text the app sends "collected" data. So a "Data Not Collected" label, or a
label missing the sent data type, next to a third-party AI call is a 2.3 accuracy
finding at **judgment-call**: the audit cannot see which retention terms the developer
has with the provider. Ask, and point at the provider's retention page and any
zero-retention option; if zero retention is confirmed in writing, the label can stay
narrow.

## 11. Web-wrapper fingerprints (4.2)

Guideline 4.2: the app should include "features, content, and UI that elevate it beyond
a repackaged website." The recurring rejection sentence developers report is that the
app "is not sufficiently different from a mobile browsing experience."

Static fingerprint, each element adds weight:

- A single `WKWebView` filling the root view controller or root SwiftUI view
- App content loaded from one remote URL; navigation happens inside the web view
- No native navigation (tab bar, navigation stack), no push handling, no offline or
  connection-error handling, no meaningful use of any native framework beyond WebKit

A full fingerprint is a strong **review-risk**; report which native capabilities exist
and which are missing so the fix is actionable. The judgment side (is the native layer
substantial enough) belongs to the semantic pass.

## 12. Dynamic code and deprecated APIs

- **2.5.2**: apps may not download, install, or execute code that introduces or changes
  features, including other apps. Flags: hot-patching frameworks, `dlopen` on
  downloaded files, JavaScript bridges that alter app behavior post-review. Distinguish
  three cases: ordinary web content rendered in a web view is fine; downloaded code
  that changes the app's own features post-review is the 2.5.2 violation; and embedded
  or streamed software such as mini-apps and mini-games (chatbot plugins included) is
  governed by 4.7 and its conditions, not exempted by being web-delivered.
- **UIWebView**: Apple stopped accepting apps containing UIWebView references; a
  reference in the submitted binary blocks upload (ITMS-90809). Only the binary is
  determinative: a match in source or an old dependency is review-risk needing build
  verification (the code may not compile into the shipping target), while a reference
  confirmed in the built product is validator-certain. Replace with `WKWebView`.
- **Private API use**: symbols from private frameworks or selector-string obfuscation
  patterns are a hard rejection surface (2.5.1); a reliable static scan needs the
  compiled binary, so from source alone flag only clear cases.

## 13. Hidden, dormant, and undocumented features (2.5.19)

Live text (new guideline, October 16, 2026; the first sentence moved here from
2.3.1(a), the second is new): "Don't include any hidden, dormant, or undocumented
features in your app; your app's functionality should be clear to end users and App
Review. Attempting to hide undocumented features in your app binary, such as through
code obfuscation or other methods, is grounds for removal of your app from the App
Store and termination of your developer account." Apple's announcement frames it as
the rule against apps that misrepresent what they do during review and change after
approval ("bait-and-switch"). 2.3.1(a) still requires every new feature to be
"described with specificity in the Notes for Review" and "accessible for review";
the Introduction still says trying "to trick the review process" gets the developer
expelled. Cite 2.5.19 for hidden-feature findings and 2.3.1(a) for the missing
disclosure.

What the text does not say: it names neither remote configuration, nor feature
flags, nor A/B testing, in either direction. The test it sets is whether
functionality is "clear to end users and App Review" and whether anything is hidden
"in your app binary". So the audit question is not "does this app use remote config"
but "is there functionality in the binary the reviewer cannot see or was not told
about, and is any of it concealed on purpose". Remote changes to data (paywall copy,
price display, layout order) for code that shipped in the reviewed binary read as
outside the sentence; a screen, tab, or capability that exists only after a
server-side switch reads as inside it. That reading is an inference from the text plus Apple's "features
that weren't included in the version submitted" framing, not a quoted rule; say so
when a finding depends on it.

Static signals, each from source, lockfiles, or build settings:

- **Reviewer detection** (review-risk): code that changes behavior when it believes
  App Review is looking. Account-level exposure: it reads as one of 2.5.19's "other
  methods", and the Introduction names trying "to trick the review process" as
  grounds for expulsion. Grep for identifiers like
  `isReview`, `reviewMode`, `appReview`, `isAppleReviewer`, `underReview`; Apple IP
  ranges or ASN strings (`17.0.0.0/8`); geo checks against Cupertino; date gates that
  unlock a feature after a planned review window; and StoreKit environment checks
  (`appStoreReceiptURL` ending in `sandboxReceipt`, `AppTransaction.environment`)
  whose result reaches view or feature branches rather than receipt validation or
  server selection. Sandbox detection for purchase validation is legitimate; the
  finding is the data flow into UI. The most common real shape is a paywall that
  hides a plan, price, or purchase option when the receipt is a sandbox receipt, so
  the reviewer sees a different paywall from users; 2.1(b) separately requires IAPs
  to be "visible to the reviewer". Beyond the guidelines, the Developer Program
  License Agreement's termination section (11.2, as fetched 2026-11-05) lists
  "hiding or trying to hide functionality from Apple's review" in item 7; the
  section's lead-in makes termination "effective immediately upon notice from
  Apple" for every condition it lists, and item 7, unlike ordinary breaches under
  item 1, builds in no 30-day cure period. Cite the agreement clause in the
  finding; it is what turns reviewer detection from a rejection into account
  exposure.
- **Dormant code** (the text's own word): user-facing functionality compiled into
  the shipping target with no path to reach it. The rule is about screens and
  behaviors, not symbols: a dead helper function or an unreferenced type is not a
  feature, and the linker may strip it anyway. Two shapes, rated differently:
  - *Test and override switches readable in Release* (review-risk): mock purchases,
    screenshot mode, entitlement overrides, slow-network or region toggles read from
    `UserDefaults`, launch arguments, or the environment without `#if DEBUG`. A user
    or reviewer can trigger them, they change what the app does, and they are what
    the Apple engineer's reply under hidden entry points says a submitted app should
    not contain.
  - *Unreachable screens and flows* (judgment-call): a view, tab, or flow whose
    presentation or navigation call is commented out or never made while the
    implementation stays in the target; in-development features with the UI
    disconnected. From source alone the audit cannot tell whether the code survives
    the build, so name the check that settles it: archive the shipping target and
    look for the feature's type names or user-facing strings in the built
    executable (`strings` or `nm` on the archived binary). If they ship, delete the
    code or finish and document the feature; `#if DEBUG` is the halfway house.

  Users not being able to reach it is not a defense; the rule is about what is in
  the binary. Neither shape earns the account-level label on its own: 2.5.19's
  termination sentence is about attempting to hide, so the label needs a
  concealment signal (obfuscation, reviewer detection, a switch that unhides the
  feature after approval). One developer report: a pending termination notice was
  withdrawn after a second appeal self-reported leftover unreleased code
  (experimental features with the UI commented out but still compiled) in an
  unreleased app's binary; that app was then rejected under 2.3.1 (Hidden Features,
  the rule's home before October 2026), and its resubmission with the code removed
  was still awaiting review when the thread was posted
  (developer.apple.com/forums/thread/818150, February 2026; the developer's own
  attribution, not confirmed by Apple in the thread).
- **Dark-shipped features** (review-risk): a remote flag (Firebase Remote Config,
  LaunchDarkly, Statsig, GrowthBook, ConfigCat, Optimizely, custom JSON) or a local
  default gating a whole screen, tab, or capability with the default off, and no
  entry in the review notes explaining how to reach it. Read the flag defaults
  (`setDefaults`, bundled defaults plist, `@AppStorage` defaults) and trace which
  flags feed navigation or feature registration. A flag that only reorders
  onboarding or swaps paywall copy is not this finding. If review notes were not
  supplied, report the flag inventory as a review-notes to-do (judgment-call), not
  as a hidden-feature finding.
- **Hidden entry points** (review-risk): debug or developer menus reachable in
  Release builds (shake handlers in `motionEnded`, secret tap-count gestures,
  long-press easter eggs) and URL-scheme or deep-link handlers that flip feature
  flags. `#if DEBUG` around the surface is the fix; its absence is the signal. The
  closest Apple statement is an Apple DTS engineer's forum reply (December 2025,
  developer.apple.com/forums/thread/810549, a secondary source): submitted apps
  "should not contain hidden features or debugging options that alter the intended
  user experience for the general public". Role-gated admin screens (`isAdmin`, role
  checks) are a different case: a production admin area is legitimate, but the
  reviewer needs demo credentials for that role, since Apple's help page on
  completing a review asks for credentials for each account type, administrative
  accounts included; a review-notes description alone does not substitute. Flag
  missing per-role credentials (review-risk, 2.1(a); demo account checklist in
  metadata-and-review-notes.md), not the screen.
- **Obfuscated feature code** (review-risk; the one method the text names): selectors and
  class names assembled from decoded strings (`NSClassFromString`,
  `NSSelectorFromString`, `performSelector` with runtime-built names), base64 or XOR
  blobs decoded into symbol names, `dlopen`/`dlsym` on bundled or downloaded paths,
  and obfuscator build phases. Ordinary string encryption of API keys is not feature
  hiding; the finding needs a feature behind the obfuscation. Overlaps section 12
  (2.5.1, 2.5.2); cite both where both apply.
- **Post-review payment swaps** (review-risk; cite 3.1.1 as well): a remote toggle
  that replaces IAP with a web checkout or external SDK after approval, or a hard
  paywall that can be switched on remotely after a freemium submission. Signals:
  network-sourced checkout URLs opened from paywall code, or a payment SDK reached
  from paywall code behind a remote toggle. Calibrate by storefront: 3.1.1(a) says
  the entitlements "are not required" for links in United States storefront apps,
  so a US-only checkout link shown next to IAP is not this finding; outside the US
  the external-purchase CTA needs the entitlement (section 6), and the 2.5.19 side
  is the toggle that changes the purchase path after approval.
- **Undisclosed experiments** (judgment-call): paywall or onboarding variants
  configured in a paywall SDK (RevenueCat, Superwall, Adapty) or an experimentation
  SDK that the default flow never shows. The text does not name experiments, and a
  variant the reviewer never sees could still be read as undocumented; each variant
  the reviewer may land on must itself pass 3.1.2 and 2.1(b), and 2.3.1(a) wants it
  described. The fix is a review-notes line naming the variants and what
  each shows, plus confirming every variant is reachable in the submitted build.
- **Regional or account gating** (judgment-call; review-risk when the gated branch
  is a whole screen or capability the reviewer cannot reach): feature branches on
  `Storefront.current`, `SKStorefront.countryCode`, `Locale.current.region`, or the
  signed-in user's plan, with no review-notes explanation. Apple's help page on
  completing a review
  (developer.apple.com/help/app-review/before-submitting-for-review/complete-review)
  asks for regional differences to be described or ruled out.

Calibration: remote-config SDK presence alone is not a finding; list it in the
report's "Not checked" section (anything driven by remote configuration is outside
what a repository audit can see). Reviewer detection and obfuscated feature code are
the two patterns to report at the top of the findings regardless of frequency: the
live text names obfuscation as grounds for termination, reviewer detection falls
under its "or other methods", and the license agreement names hiding functionality
from review outright. Dormant code with no concealment signal is an ordinary 2.5.19
finding, not account-level.

## 14. Ratings prompts (5.6.1, 3.2.2(x))

Live text, 5.6.1: "Use the provided API to prompt users to review your app; this
functionality allows customers to provide an App Store rating and review without the
inconvenience of leaving your app, and we will disallow custom review prompts."
3.2.2(x): "Apps must not force users to rate the app, review the app, download other
apps, or other store-related actions in order to access functionality, content, or
use of the app." Section 3's introduction adds that attempts to "inflate your chart
rankings with paid, incentivized, filtered, or fake feedback" may end in expulsion
from the Developer Program, and 5.6.3 bars "Manipulating any element of the App Store
customer experience such as charts, search, reviews, or referrals".

Static signals, each from source:

- **Custom prompt UI** (review-risk): a hand-built "rate us" alert, sheet, or star
  control shown proactively, or a prompt that opens the store's write-review URL
  (`itms-apps://` with `action=write-review`), with no system request
  (`SKStoreReviewController.requestReview`, `AppStore.requestReview`, SwiftUI's
  `requestReview` environment action). A Settings row that opens the store's review
  page on the user's own tap is not a prompt and not this finding. Note for the fix:
  the system prompt may show nothing and is rate-limited by the OS, so no flow may
  depend on it appearing.
- **Review gating** (review-risk): a satisfaction question ("Enjoying the app?") whose
  positive branch calls the system prompt and whose negative branch opens a feedback
  form or does nothing. That is the "filtered" feedback the Section 3 introduction
  names. The pattern that stays inside the rule is triggering the system prompt on an
  in-app signal (a completed task, a streak) with no question deciding who sees it.
- **Access or rewards gated on rating** (review-risk; 3.2.2(x)): an unlock, currency,
  or feature conditioned on tapping a rate action, or a flow that cannot be dismissed
  without one.

The circulating citation "5.6.3" for the prompt rule is wrong; the prompt sentence is
in 5.6.1, and 5.6.3 is Discovery Fraud. Cite 5.6.1 for custom prompts, the Section 3
introduction plus 5.6.3 for filtering, and 3.2.2(x) for forced actions.
