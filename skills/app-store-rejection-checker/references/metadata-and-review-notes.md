# App Store Connect metadata, review notes, and process mechanics

The store-listing and submission-form surface: what reviewers cross-check against the
binary, and how the review process behaves when something is rejected. Guideline
numbers verified against the live guidelines on 2026-11-05 (text dated October 16,
2026).

Contents:

1. [Metadata checks (2.3.x)](#metadata-checks-23x)
2. [Developer identity and contact (1.5, 5.6.2)](#developer-identity-and-contact-15-562)
3. [Review notes checklist (2.3.1)](#review-notes-checklist-231)
4. [Demo account checklist (2.1)](#demo-account-checklist-21)
5. [In-app purchase readiness (2.1(b))](#in-app-purchase-readiness-21b)
6. [Metadata Rejected vs binary rejection](#metadata-rejected-vs-binary-rejection)
7. [Review timing, expedited review, appeals](#review-timing-expedited-review-appeals)
8. [Repeated rejections and account exposure](#repeated-rejections-and-account-exposure)

## Metadata checks (2.3.x)

- **Accuracy (2.3, 2.3.1)**: the description, screenshots, and previews must match what
  the binary does. Cross-check every feature claim in the description against the code
  inventory from Step 2; flag claims with no implementation ("AI-powered" with no model
  or API call in the code, "community" with no UGC surface). Hidden, dormant, or
  undocumented features in the build are a 2.5.19 violation since October 2026
  (previously 2.3.1), with account termination named in the text; see
  static-checks.md section 13. 2.3.1(a) itself still makes misleading marketing,
  "whether within or outside of the App Store", grounds for removal.
- **IAP disclosure (2.3.2)**: if items, levels, or subscriptions require additional
  purchases, the description, screenshots, and previews must say so.
- **Prices in metadata (2.3.1(a), 2.3.7, 2.3.14)**: two different rules, and a
  matching configured price cures only the first. Accuracy: 2.3.1(a) makes
  "promoting a false price, whether within or outside of the App Store" grounds for
  removal and account termination, so a price or discount shown in a screenshot or
  preview must match a price or offer actually configured. One that appears nowhere
  in the product configuration is a question for the developer (judgment-call until
  answered); it becomes a review-risk finding, with the account-level label, only
  when the developer confirms it was never in force or no offer exists behind it.
  Prices merely stale because storefront pricing changed are a judgment-call.
  Placement: 2.3.7 says not to "pack any of your metadata with
  trademarked terms, popular app names, pricing information, or other irrelevant
  phrases just to game the system" and that "app names, subtitles, screenshots, and
  previews should not include prices, terms, or descriptions that are not specific
  to the metadata type"; 2.3.14 bars "specific or discounted pricing information" in
  creative assets outright. A price in the name, subtitle, or keywords, or a
  discount in a creative asset, is a review-risk finding whether or not the number
  is right. The in-app side is in static-checks.md section 6.
- **Category (2.3.5)**: "Select the most appropriate category for your app ... If
  you're way off base, we may change the category for you." Apple's category
  definitions page (developer.apple.com/app-store/categories/) adds that "Choosing
  categories that are not appropriate for your app is against the App Review
  Guidelines." Judgment-call, with one hard edge: a Health & Fitness or Medical
  primary or secondary category triggers the regulated-medical-device declaration
  (special-categories.md), so an app whose code is a diet or symptom tool but whose
  category avoids both needs an honest fit elsewhere (Apple lists "dietary & food
  allergy" under Food & Drink, for example), not a category chosen to dodge the form.
- **Screenshots (2.3.3)**: must show the app in use, not splash screens, login walls,
  or pure marketing art. Apple's 2.3.3 help page (linked from the guideline since
  October 2026) says App Review approves only submissions "where more than half of
  screenshots show the app in use" per device set; its best practices also ask that
  screenshots be updated with each version to match the current interface. Starting
  April 2027, submissions must also include iPhone Duo screenshots (Apple news,
  October 5, 2026).
- **Creative assets (2.3.14, since October 16, 2026)**: promotional assets for the
  product page header and search results (Asset Library in App Store Connect) are a
  separate surface from screenshots. Live text: they "must pertain to your app's
  content and functionality or overarching brand, follow standard metadata
  guidelines, and be appropriate for all audiences" and "should not include
  inappropriate content, references to other apps with an intent to mislead,
  unverifiable product claims, or specific or discounted pricing information."
  Apple's asset best practices page adds: no website URLs or copyright symbols, no
  Apple-designated recognitions (Editor's Choice, App of the Day, Apple Design Award),
  no logos or references to other platforms or marketplaces, and a 4+ rating for
  every asset regardless of the app's rating. Flag assets that show prices or
  discounts, award or ranking claims the developer cannot verify, competitor or
  other-platform names, or content above 4+. Rules:
  developer.apple.com/app-store/asset-best-practices/. If the developer confirms no
  creative assets are configured, say so in one line; if they were simply not
  supplied, mark 2.3.14 "not audited", since assets can exist in App Store Connect
  without being in the repo.
- **Age rating (2.3.6)**: answers must match the app's actual content and features.
  The questionnaire was overhauled in 2025 (see recent-changes.md): tiers are now 4+,
  9+, 13+, 16+, 18+ (displayed on OS 26 and later; older OS versions show the old
  tiers), answers were due by January 31, 2026, and the questionnaire asks how AI
  assistant and chatbot features affect the frequency of sensitive content. When
  checking answers, remember Messaging and Chat, User-Generated Content, and Social
  Media are separate questionnaire capabilities; chat without a UGC declaration is
  not automatically a mismatch. Real mismatch flags: a capability clearly present in
  the code but not declared (a social feed with no Social Media answer), or ad SDKs
  with no matching content descriptors.
- **Name and keywords (2.3.7)**: app name at most 30 characters; no pricing or
  irrelevant phrases in name, subtitle, or keywords. On trademarks, the rule targets
  **unauthorized** or discovery-gaming use (other developers' trademarks or popular
  app names used without permission or with no relevant content in the app), not
  every trademark reference (naming a service the app legitimately integrates with
  is not itself a violation).
- **All-audiences metadata (2.3.8)**: icons, screenshots, and previews must be
  appropriate for a 4+ audience even when the app is rated higher; "for kids" wording
  is reserved for the Kids Category.
- **Rights to metadata materials (2.3.9, 5.2.1)**: no borrowed screenshots, brands, or
  celebrity imagery without permission.

## Developer identity and contact (1.5, 5.6.2)

Live text, 1.5: "Make sure your app and its Support URL include an easy way to contact
you ... Failure to include accurate and up-to-date contact information not only
frustrates customers, but may violate the law in some countries or regions." 5.6.2:
"Your representation of yourself, your business, and your offerings on the App Store
or alternative distribution must be accurate. The information you provide must be
truthful, relevant, and up-to-date". 5.6 as a whole carries account termination.

Checks, each on the metadata the developer supplied:

- The support URL and privacy policy URL resolve, belong to the developer, and are not
  placeholder or template pages (an unfilled site template, an empty policy, a
  "coming soon" page). 2.1(a) separately rejects "empty websites, and other temporary
  content". Review-risk when a URL is dead or empty; the audit can fetch the URLs if
  network is available, and says so when it did.
- The support page gives a working contact route (email, form, or similar), not only
  marketing copy.
- The seller name, legal entity, and support contact match the app's actual
  publisher, and the privacy policy names the real data recipients (static-checks.md
  sections 9 and 10 for the AI and analytics cases). Judgment-call; the audit cannot
  verify legal identity, only consistency between what the policy says and what the
  code does.
- 5.1.1(ix): apps in "highly regulated fields (such as banking and financial services,
  healthcare, gambling, legal cannabis use, air travel and crypto exchanges) or that
  require sensitive user information should be submitted by a legal entity that
  provides the services, and not by an individual developer." The account type is not
  in the repo; when Step 2 puts the app in one of these fields, ask, and flag an
  individual enrollment as a structural review-risk no code change fixes.

## Review notes checklist (2.3.1)

Live text: "All new features, functionality, and product changes must be described with
specificity in the Notes for Review section." Good notes cover everything a reviewer
cannot infer from the binary:

- Demo credentials (below), including for each account tier if roles differ
- Setup steps to reach core functionality quickly
- Region-locked or geo-dependent content, and how to see it from the review location
- Required hardware or peripherals, and what happens without them
- Server-driven or remote-config behavior, explained rather than discovered: name
  the flags and experiments the reviewer may hit, what each variant shows, and
  confirm every variant is reachable in the submitted build (2.5.19 wants
  functionality "clear to end users and App Review"; a variant the reviewer cannot
  reach reads as undocumented)
- For medical apps with regulatory clearance: the link to that documentation (1.4.1)
- For VPN or licensed categories: license information where a territory requires it
- Every product, plan, price, flag, and experiment the notes name must exist in the
  submitted build and configuration. Notes promising a plan that is not configured,
  or a demo video recorded on an older version, are a 2.3.1(a) accuracy finding
  (features must be "described with specificity"); report them as that. Only a
  pattern of such mismatches reads as the conduct Apple's termination notices
  describe; the notice wording is quoted in the account exposure section below.
  Cross-check the notes against the product list and the paywall code.
- Expect a "limited App Review history" information request on a new account. Apple's
  review status help page (developer.apple.com/help/app-review/after-submitting-for-review/review-status/)
  lists "Limited review history: Apps with little or no prior App Review history"
  among the reasons reviews take longer, and separately says that when "App Review
  requested more information and there are no other issues to resolve" the developer
  can reply in App Store Connect instead of resubmitting; it does not tie the two
  together. Developers do: a forum post quotes a message asking for additional
  information, citing no guideline, which says the app "has been submitted by a
  developer account that has a limited App Review history"
  (developer.apple.com/forums/thread/849346, message dated September 30, 2026), and
  a GitHub issue (github.com/sharneng/spliit2go/issues/184, opened October 5, 2026)
  shows the same request under the label "Guideline 2.1 - Information Needed - New
  App Submission" with its itemized list: a screen recording on a physical device running the
  latest OS from launch through the main flow (registration, account deletion,
  content reporting and blocking), the app's purpose and audience, setup steps and
  credentials, the external services it relies on, regional differences, and
  regulatory documents where relevant. Those are reported requests, not guideline
  text; preparing the recording and answers before the first submission costs
  little and shortens the loop if the request comes.

## Demo account checklist (2.1)

A broken login for the reviewer is functionally a completeness rejection. Verify:

- A demo account exists for every login-gated flow, is filled with realistic content
  (not a blank state), and its credentials are in App Review Information
- Credentials tested on a clean device shortly before submission
- 2FA or OTP flows: the code or bypass is supplied in the notes; a code sent to a
  phone the reviewer does not have is a rejection generator
- The backing server and staging environment stay up for the whole review window
- If credentials legally cannot be provided, a demo mode needs prior arrangement with
  Apple; do not assume it silently substitutes

## In-app purchase readiness (2.1(b))

Live text: "If you offer in-app purchases in your app, make sure they are complete,
up-to-date, visible to the reviewer and functional. If any configured in-app purchase
items cannot be found or reviewed in your app, explain the reason in your review
notes." Apple's help page on configuring in-app purchases for review
(developer.apple.com/help/app-review/before-submitting-for-review/configure-in-app-purchases,
fetched 2026-11-05) sets the environment: "App Review evaluates your In-App
Purchases in the same sandbox environment you use for testing", and "In-App
Purchases do not need prior approval from App Review to function in review." The
same page makes one account-side item a hard condition: "The Account Holder must
accept the Paid Apps Agreement in the Business section of App Store Connect, then
confirm the agreement is in effect", because "Paid In-App Purchases won't function
in review until this agreement is active." It also asks that storefront or device
restrictions on products be stated "in the Notes field in the App Review
Information section", and that the products be submitted "once they're in the Ready
for Review state". The recurring failure shape is a paywall that cannot load its
products or a purchase that errors, reported back as a 2.1(b) rejection ("unable to
complete purchase") even though the same build works for the developer. RevenueCat's
rejection guide: "Often times, purchases will work fine in Sandbox
and Testflight, but App Store Review will not be able to fetch products"
(revenuecat.com/docs/test-and-launch/app-store-rejections, undated, a vendor
source).

What the audit can check from the repo and the supplied metadata:

- **Paid Apps Agreement active.** Not in the repo; ask, and report the answer as
  developer-supplied. Apple's sandbox technote TN3186 ("Troubleshooting In-App
  Purchases availability in the sandbox", under
  developer.apple.com/documentation/technotes/, fetched 2026-11-05) says the
  agreement "is in effect if App Store Connect shows an Active status in its
  Agreements section", lists an unsigned, superseded, or expired agreement among the
  reasons products do not load, and adds that banking and tax information must also
  show Active. An expired membership needs the agreement accepted again.
- **Products attached to the submission.** App Store Connect help
  (developer.apple.com/help/app-store-connect/manage-submissions-to-app-review/submit-an-in-app-purchase,
  fetched 2026-11-05): "The first consumable, non-consumable, auto-renewable
  subscription, and non-renewing subscription In-App Purchase of each type must be
  submitted with a new app version." The same page: "After the first item of a given
  type is approved, you can submit additional items of that type without a new app
  version, as long as your app has at least one approved app version at the time of
  submission." So the attach-to-version check applies to the first product of each
  type, and to every product while the app has no approved version yet; a first-time
  product type that is not added to the version submission is invisible to the
  reviewer. A later product of a type that already has an approved product can be
  submitted on its own, without a version, and is fine as long as it is in a
  submitted or approved state before review. For the products the check covers, ask
  whether the product is attached to the version submission and shows Ready for
  Review, with a price and localizations set (TN3186 names a missing price or
  localization as a reason a product does not load). App Store Connect's status
  reference
  (developer.apple.com/help/app-store-connect/reference/in-app-purchases-and-subscriptions/in-app-purchase-statuses,
  fetched 2026-11-05) defines Approved as "Apple has approved your In-App Purchase",
  and Waiting for Review, In Review, and Accepted as states of a submission already
  sent; a product in any of these is fine, however it was submitted. Prepare for
  Submission ("you haven't yet submitted it for review") on a product the paywall
  needs is the finding. If the developer cannot say, list it under "Not checked".
- **Product identifiers match.** The identifiers in code or in a `.storekit`
  configuration must match the ones configured in App Store Connect, and the bundle
  identifier must match the app record; a mismatch loads nothing. TN3186's remaining
  causes are identifier and signing causes: a bundle ID "disabled for the In-App
  Purchase capability in Certificates, Identifiers & Profiles", a wildcard App ID,
  an app "signed with an invalid certificate or provisioning profile", and an
  inactive Apple Developer Program account (the membership item under the agreement
  bullet above). In-app purchase is a capability on the explicit App ID, not a key in
  the app's entitlements file; Apple DTS (developer.apple.com/forums/thread/801832,
  September 2025) says the capability "appears enabled by default for an explicit
  App ID and disabled for a wildcard App ID" and that the
  `com.apple.developer.in-app-purchase` entitlement "doesn't exist", so a build whose
  entitlements carry that key should drop it. What to compare: the code's identifier
  list and the target's bundle identifier against what the developer supplies, an
  explicit (non-wildcard) App ID registered with the capability, and a signing
  certificate and provisioning profile that are valid for it.
- **Error handling the reviewer can read.** A generic "Something went wrong" on
  purchase failure gives the reviewer nothing to put in the rejection and the
  developer nothing to diagnose. Not a guideline finding; recommend surfacing the
  StoreKit error description in the alert and logging the product fetch result.
- **Trial eligibility and sandbox state.** See the eligibility item in
  static-checks.md section 6: what the reviewer's fresh sandbox account shows on the
  paywall can differ from what the developer's account shows.

Production availability after approval is a different question, and TN3188
("Troubleshooting In-App Purchases availability in the App Store", same location)
is its pointer, not a review check: it asks the developer to confirm each product's
status is Approved once Apple has reviewed it, to update the product information on
"Developer Action Needed" and to reselect countries or regions on "Developer Removed
from Sale", to confirm storefront availability, and to allow time after approval
before the products appear everywhere. The HTML pages of both technotes render
client-side and return an empty shell to a plain fetch; the full text is at the
`.json` endpoints under developer.apple.com/tutorials/data/documentation/technotes/
(tn3186-troubleshooting-in-app-purchases-availability-in-the-sandbox.json and
tn3188-troubleshooting-in-app-purchases-availability-in-the-app-store.json), which
is where the TN3186 quotes above come from. Point the developer at the technotes by
name and number. Runtime purchase behavior stays in the report's "Not checked" list
regardless.

## Metadata Rejected vs binary rejection

- **Metadata Rejected** concerns only the store listing (text, images, review
  information). Fix the flagged fields and resubmit; the same binary is reused, no new
  upload needed.
- A general **Rejected** verdict does not by itself require a new build; the
  remediation determines that. Rejections resolved by fixing metadata, App Review
  Information, demo access, or a backend issue can be resubmitted on the same binary;
  a new upload is required when the fix is in the binary itself (crashes, missing
  features, code-level violations) or when the upload was refused as **Invalid
  Binary**.
- Either way the submission lands in an "Unresolved Issues" state: remove the rejected
  item to let the rest proceed, or fix and resubmit, and reply in the same App Store
  Connect message thread.

## Review timing, expedited review, appeals

- Apple's published figure (developer.apple.com/distribute/app-review/): "90% of
  submissions are reviewed in less than 24 hours." Treat it as a target, not a
  promise: 2026 community reports show wide variance (multi-day waits, especially for
  Mac apps and new accounts). Never present a guaranteed turnaround.
- **Expedited review** can be requested (developer.apple.com/contact/app-store/,
  expedite topic) for a critical bug fix in the live version (with repro steps) or a
  genuinely time-sensitive event. Routine releases do not qualify, and repeated
  misuse gets future requests denied.
- **On rejection**: reply in the Resolution Center first; a phone call can be
  requested. Apple's appeal page
  (developer.apple.com/help/app-review/after-submitting-for-review/appeal-to-the-app-review-board)
  says to reply to the message in App Store Connect before appealing, to "Cite the
  specific guideline or terms your app was rejected or removed under, and explain
  point by point how your app meets it", to submit "one appeal per rejected or removed
  app" covering every cited violation at once, and not to appeal "if you've addressed
  the issues App Review identified and resubmitted the app for review." Its model
  appeal says the demo account and setup steps "are included in the App Review
  Information section of this submission, along with screenshots showing the feature
  working as described"; a screen recording of the flow is the practical equivalent. If unrelated new issues surface
  during a bug-fix review, Apple may offer to let the current submission through and
  defer the new issues to the next one; that offer arrives in the message thread.

## Repeated rejections and account exposure

The guidelines treat the pattern of submissions, not only each one. Live text in
"After You Submit": "if your app is repeatedly rejected for the same guideline
violation or you've attempted to manipulate the review process, review of your app
will take longer to complete." 2.3.1(b): "Egregious or repeated behavior is grounds
for removal from the Apple Developer Program." 4.3(b): "Repeated submissions of this
kind may lead to removal from the Apple Developer Program." 5.6: "Repeated
manipulative or misleading behavior or other fraudulent conduct will lead to your
removal from the Apple Developer Program." Apple's 2025 transparency report
(apple.com/legal/app-store/transparency/2025/) counts 193,035 developer accounts
terminated, 10,127 termination appeals, and 499 restorations; its footnote says most
appealed terminations were for fraud. Apple publishes no number of rejections that
triggers account action; the text is qualitative ("repeated", "egregious").

Developers report 2026 termination or pending-termination notices after: a new app
rejected four or five times on an account fifteen years old
(developer.apple.com/forums/thread/841277, August 2026); repeated rejections followed
by submitting a rebuilt replacement app (developer.apple.com/forums/thread/831303,
notice dated May 29, 2026); a second 4.3(a) rejection arriving with an account removal
warning (developer.apple.com/forums/thread/847013, September 2026); 66 apps removed
at once under 4.3 for sharing a similar "binary, metadata, and/or concept"
(developer.apple.com/forums/thread/845150, September 2026). Each is the developer's
own account of events. The first two say the notice cited section 3.2(f) of the
license agreement (the conduct clause; recent-changes.md); the other two were 4.3
actions. None quotes its notice in full, and one names the app. Two earlier threads
do paste the notice text, which says the conduct "can include, but are not limited
to, inaccurate metadata describing your app or service" and "providing misleading
responses in App Store Connect" (developer.apple.com/forums/thread/804267, October
2025; developer.apple.com/forums/thread/767063, October 2024, where the notice
referred to one dated binary submission). Treat all of them as the shape of the
risk, not as thresholds.

What this means for the audit's advice, when the developer says the app has already
been rejected:

- Fix every cited item before resubmitting, and reply in the thread first to ask for
  all outstanding reasons. Serial rejections for different reasons are the common
  loop; a second rejection for the same reason is the one Apple's text names.
- Do not resubmit an unchanged binary with only the build number bumped; if the
  rejection was a reviewer-environment failure (sandbox purchase, server down), say so
  in the notes with evidence instead.
- Do not delete the app record and recreate it under a new bundle identifier, and do
  not submit a rebuilt variant to escape a rejection history; the replacement-app case
  above is exactly that.
- A rejection whose own text names Program removal deserves a stop and a reply, not
  a quick fix-and-resubmit: 4.3(b) ("Repeated submissions of this kind"), 2.3.1(b),
  3.1.2(a), 5.6. An ordinary first rejection under 4.3(a) or 1.1 carries no such
  language; fix what was cited, use the reply thread to confirm nothing else is
  outstanding, and resubmit once. The full list of clauses with that language is in
  rejection-taxonomy.md, account-level exposure.
