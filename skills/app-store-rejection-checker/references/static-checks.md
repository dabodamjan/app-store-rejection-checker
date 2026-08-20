# Static checks: the mechanically verifiable surface

Checks that can be decided from files in the repository: build settings, plists,
entitlements, lockfiles, and source. Guideline numbers were verified against the live
guidelines on 2026-08-20; re-verify against the Step 1 fetch when citing them.

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
- **Subscription disclosure (3.1.2)**: before asking users to subscribe, the app must
  clearly say what the user gets for the price; rejections in this family also cite
  missing subscription length, price, and functional privacy policy and terms-of-use
  links in the app and metadata (the binding wording comes from 3.1.2 plus Schedule 2
  of the Developer Program License Agreement). Check the paywall screen content.
- **Loot boxes (3.1.1)**: randomized paid items require disclosing the odds of each
  item type before purchase.

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
means an app that declares it collects analytics under legitimate interest per GDPR
or a similar statute (typically stated in its privacy policy) is not violating the
consent sentence at all; with such a declaration, do not report consentless
collection as a finding, beyond a note if the declaration looks inconsistent with
the actual data flows. Absent a declared basis: collection on by default with a
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
with the SDK. Also check withdrawal regardless of rating: the withdraw-consent
sentence still binds, and a consent or opt-out surface with no way to turn
collection off afterwards is incomplete under the same subsection.

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
