# Static checks: the mechanically verifiable surface

Checks that can be decided from files in the repository: build settings, plists,
entitlements, lockfiles, and source. Guideline numbers were verified against the live
guidelines on 2026-08-20; re-verify against the Step 1 fetch when citing them.

Evidence standard for **validator-certain** (defined in SKILL.md Step 6): the upload
validators run against the built product, not the repository. Conditional compilation,
unused targets, files excluded from the shipping target, and packaging-time manifests
can all make source or lockfile evidence wrong about what ships. So a finding reaches
validator-certain only when the evidence reflects the merged shipping target — an
archive, `.app`, or `.ipa`, or build settings resolved for that target. From source and
lockfiles alone, report the same finding as review-risk with an explicit "needs build
verification" note.

Contents:

1. [Permission purpose strings](#1-permission-purpose-strings)
2. [Privacy manifest (PrivacyInfo.xcprivacy)](#2-privacy-manifest-privacyinfoxcprivacy)
3. [Entitlements consistency](#3-entitlements-consistency)
4. [Login services (4.8)](#4-login-services-48)
5. [Export compliance](#5-export-compliance)
6. [Payments and IAP wiring (3.1.x)](#6-payments-and-iap-wiring-31x)
7. [Account deletion (5.1.1(v))](#7-account-deletion-511v)
8. [App Tracking Transparency (5.1.2(i))](#8-app-tracking-transparency-512i)
9. [Third-party AI data flows (5.1.2(i))](#9-third-party-ai-data-flows-512i)
10. [Web-wrapper fingerprints (4.2)](#10-web-wrapper-fingerprints-42)
11. [Dynamic code and deprecated APIs](#11-dynamic-code-and-deprecated-apis)

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
| `NSPhotoLibraryUsageDescription` (read), `NSPhotoLibraryAddUsageDescription` (write-only) | direct PhotoKit library access: `PHPhotoLibrary`, `PHAsset` fetches — but **not** `PHPickerViewController` (see note below) |
| `NSContactsUsageDescription` | Contacts, ContactsUI |
| `NSUserTrackingUsageDescription` | AppTrackingTransparency, `ATTrackingManager.requestTrackingAuthorization`, SDKs configured to use the IDFA (SDK presence alone is not proof — check the configuration) |
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

1. Any category's API used (in app code or a linked SDK) but not declared in
   `NSPrivacyAccessedAPITypes`: upload will be flagged (missing API declaration).
2. Reason codes: do not validate codes from memory. Fetch the current approved list
   from Apple's "Describing use of required reason API" page
   (developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api
   and its per-category subpages) and check declared codes against it. Examples of
   currently valid codes: `C56D.1` (UserDefaults), `35F9.1` (SystemBootTime).
3. Third-party SDKs on Apple's commonly-used-SDKs list must ship their own privacy
   manifest, and the **signature** requirement applies to SDKs added as binary
   dependencies (XCFrameworks), not to those compiled from source. A lockfile entry
   names the dependency but proves nothing about what manifest or signature ends up in
   the bundle — treat it as a prompt to inspect the resolved artifact (the checked-out
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
- App group, iCloud, or Sign in with Apple entitlements that the code never uses:
  cleanup findings, low severity.

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
  documents the key as required when the ATT API is called — calling without it risks
  a runtime crash — but a linked SDK's API reference alone is not a deterministic
  upload block, so the missing key stays review-risk rather than validator-certain.
  SDK presence alone is not proof of tracking; check whether the SDK is actually
  configured to use the IDFA.
- Tracking initialized before the ATT prompt result is known: review-risk.
- Gating app functionality or rewards on enabling tracking (or push or location):
  prohibited; judgment on the flow, evidence from code.

## 9. Third-party AI data flows (5.1.2(i))

Live text (added November 2025): "You must clearly disclose where personal data will be
shared with third parties, including with third-party AI, and obtain explicit
permission before doing so."

The third-party-AI sentence triggers on **personal data shared with a third-party AI
service** — not any user content sent to any LLM. But 5.1.2(i) is broader than that
sentence: its opening line bars using, transmitting, or sharing personal data without
permission, so first-party or self-hosted model processing of personal data still
needs consent and privacy-policy coverage under the same guideline — it just isn't a
third-party-sharing finding. Flows whose input carries no personal data fall outside
5.1.2(i); judge, don't auto-flag. Static signals:
third-party AI API clients or endpoints in code (api.openai.com, api.anthropic.com,
generativelanguage.googleapis.com, openrouter.ai, and similar, or their SDKs) carrying
data that identifies or relates to the user — free-form user text, photos, audio,
health data, contacts. For those flows verify: is the sharing disclosed in-app before
it happens, is there an explicit consent step, and do the privacy policy and App Store
privacy labels cover it? Absent consent flow: **review-risk**, rising, since the
wording is new and explicit.

If AI features can produce sensitive content, also check the age rating answer set
(see metadata-and-review-notes.md): the questionnaire asks about AI assistant and
chatbot impact on sensitive-content frequency.

## 10. Web-wrapper fingerprints (4.2)

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

## 11. Dynamic code and deprecated APIs

- **2.5.2**: apps may not download, install, or execute code that introduces or changes
  features, including other apps. Flags: hot-patching frameworks, `dlopen` on
  downloaded files, JavaScript bridges that alter app behavior post-review. Distinguish
  three cases: ordinary web content rendered in a web view is fine; downloaded code
  that changes the app's own features post-review is the 2.5.2 violation; and embedded
  or streamed software such as mini-apps and mini-games (chatbot plugins included) is
  governed by 4.7 and its conditions, not exempted by being web-delivered.
- **UIWebView**: Apple stopped accepting apps containing UIWebView references; a
  reference in the submitted binary blocks upload (ITMS-90809). Only the binary is
  determinative — a match in source or an old dependency is review-risk needing build
  verification (the code may not compile into the shipping target), while a reference
  confirmed in the built product is validator-certain. Replace with `WKWebView`.
- **Private API use**: symbols from private frameworks or selector-string obfuscation
  patterns are a hard rejection surface (2.5.1); a reliable static scan needs the
  compiled binary, so from source alone flag only clear cases.
