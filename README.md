![app-store-rejection-checker](assets/banner.png)

# app-store-rejection-checker

A Claude Code skill that audits your iOS or iPadOS app before App Store submission
and tells you what is likely to get rejected: the guideline number (or the upload
validation it trips), the evidence it found in your repo, a concrete fix, and an
honest confidence level.

- **Live guidelines, not a frozen rule list.** Every audit starts by fetching the
  current App Store Review Guidelines from Apple's site.
- **Static checks** for the mechanical surface: purpose strings vs the APIs you
  actually link, privacy manifest and required-reason declarations, entitlements,
  export compliance, IAP and restore wiring, the paywall disclosures Apple's
  subscription rules require (the offer and the price after it, billed amount most
  prominent, terms and privacy links in the app), the free-trial toggle that
  subscription vendors report rejected since January 2026, price comparisons the
  developer cannot back, account deletion, App
  Tracking Transparency, ratings-prompt rules, web-wrapper fingerprints, and the
  hidden-feature patterns behind Apple's October 2026 rule 2.5.19 (reviewer
  detection, features shipped dark behind a switch, dormant code, obfuscated entry
  points).
- **Semantic review** for what mechanical tools cannot judge: does your description
  match what the app does, is it differentiated enough for a saturated category,
  will your review notes and demo account get a reviewer to your core flow, will the
  reviewer's fresh sandbox account see the same paywall your users see, does your AI
  feature get consent before sharing personal data with a third-party AI service.
- **Account-level exposure called out.** Findings whose guideline or license clause
  names developer-account termination (concealed features, false prices,
  subscription bait-and-switch, review manipulation) are ranked above everything
  else and labeled as such when the evidence matches the clause's condition, because
  an appeal rarely restores a terminated account.
- **Portable.** Written in the open Agent Skills format; Cursor, Codex CLI, and
  Gemini CLI run it too. See [Use with other agents](#use-with-other-agents).

## Install

Paste this into Claude Code or Codex inside your app repo:

```
Install the skill from https://github.com/dabodamjan/app-store-rejection-checker and check this app for App Store rejection risks.
```

The agent fetches the repo and takes it from there. The audit covers more if you
paste your App Store Connect metadata (description, keywords, age rating, review
notes, demo account) when asked; the store listing is a real rejection surface,
not decoration.

**Manual install**, if you want the skill permanently in place:

```bash
git clone https://github.com/dabodamjan/app-store-rejection-checker.git
mkdir -p ~/.claude/skills
cp -R app-store-rejection-checker/skills/app-store-rejection-checker ~/.claude/skills/
```

Then run `/app-store-rejection-checker` in a Claude Code session inside your app
repo. Use `.claude/skills/` instead to install for a single project; the `skills/`
layout is also plugin-compatible.

## Example finding

Every finding in the report looks like this, headed by the guideline number it
cites (or the upload validation it trips):

```
### [5.1.1(v)] Account creation without in-app account deletion
Confidence: review-risk
Evidence: FirebaseAuth in Podfile.lock; SignUpView.swift implements registration;
no account deletion UI or deleteUser call anywhere in Sources/.
Why it matters: "If your app supports account creation, you must also offer account
deletion within the app" (live guideline text). Deactivation-only flows are
explicitly insufficient, and support-email routes are only acceptable in highly
regulated industries. Scaffolded codebases often wire up auth without a delete flow.
Fix: add a delete-account action in settings that removes the Firebase user and all
associated backend records, then re-run this audit.
```

## What it checks

| Area | Examples |
|---|---|
| Completeness signals (2.1) | demo account and review notes quality, placeholder content, in-app purchases the reviewer's sandbox can actually load |
| Metadata accuracy (2.3.x) | description vs actual code, screenshots, creative assets, prices shown in metadata, category fit, name and keyword rules, age rating answers |
| Hidden features (2.5.19) | reviewer detection, features shipped dark behind a switch, dormant code, obfuscated entry points, review-notes coverage of flags and experiments |
| Payments (3.1.x) | digital goods outside IAP, restore purchases, trial and renewal disclosure, price prominence, trial toggles, false or tricky prices, loot box odds |
| Minimum functionality (4.2) | web-wrapper fingerprints, thin AI-wrapper patterns |
| Differentiation (4.3) | saturated-category exposure, template output signals |
| Login services (4.8) | social login without a privacy-protecting alternative |
| Privacy (5.1.x) | purpose strings, privacy manifest, account deletion, ATT, third-party AI data sharing and what it means for privacy labels |
| Developer conduct (5.6) | custom or gated ratings prompts, developer identity and support contact, what the termination clauses actually say |
| Special categories | kids, medical and health (sources shown in-app, verdict wording, doctor reminder, medical-device declaration, health data in iCloud), gambling, VPN, crypto, finance, UGC obligations (since October 2026 also AI content generators) |
| Recent rule changes | 2024-2026 surfaces: privacy manifests, AI disclosure, age rating overhaul, EU DMA, the October 2026 hidden-feature and creative-asset rules, the October 2026 license agreement, 2026 enforcement shifts (trial toggles, limited-history requests) |

Guideline numbers in the bundled references were verified against the live text on
2026-11-05 (Apple's text dated October 16, 2026). Each run re-fetches
the main guidelines (the authority on any conflict) and quotes finding-relevant
wording from them, scans Apple's news feed headlines, and fetches auxiliary policy
pages (account deletion, EU DMA terms, age ratings, required-reason APIs, the
subscriptions page, the license agreement, and the App Store Connect and App Review
help pages the references quote) when a finding depends on them.

## Honest limits

- The skill sees your repository, the metadata you share, and the live guidelines.
  It cannot verify runtime behavior (crashes, broken links, restore-purchase flows,
  OAuth round trips, server-driven content); the report lists those as unchecked
  rather than pretending.
- Findings are risk assessments with stated confidence, not verdicts. App Review is
  run by people; no tool can guarantee approval, and this one does not claim to.
- Not affiliated with or endorsed by Apple. App Store, Xcode, and related marks
  belong to Apple Inc.

## Use with other agents

The skill is a `SKILL.md` plus reference files, no code, in the open
[Agent Skills format](https://agentskills.io). Cursor, Codex CLI, and Gemini CLI
support the format natively (agentskills.io lists all adopters): copy
`skills/app-store-rejection-checker/` into whatever directory your agent reads
skills from. Live-page fetches need whatever fetch or browser tool your agent
exposes, and the `allowed-tools` frontmatter field is spec-experimental, so
support may vary. Agents without native skill support can be pointed at the
`SKILL.md` and asked to follow it.

## Prior art

Good tools already exist here: [fastlane precheck](https://docs.fastlane.tools/actions/check_app_store_metadata/)
(text-pattern rules over App Store Connect metadata),
[greenlight](https://github.com/RevylAI/greenlight) (the strongest mechanical
checker I found: an offline Go CLI plus a Revyl-backed tier that runs key flows on
cloud simulators), [berkayturk/appstore-precheck](https://github.com/berkayturk/appstore-precheck)
(static checks plus LLM analysis with an eval harness), and the skill sets
[cruisediary/apple-app-review-skills](https://github.com/cruisediary/apple-app-review-skills)
and [dpearson2699/swift-ios-skills](https://github.com/dpearson2699/swift-ios-skills)
(the latter under the source-available PolyForm Perimeter license).

What this one does differently: live guidelines at run time instead of a frozen
rule list, semantic review of your real code and store copy rather than pattern
matching alone, reference data limited to the only frequency numbers Apple
publishes plus a confidence-labeled ranking, and an MIT license.

## Who made this

I am [Damjan Dabo](https://dabo.dev), an indie iOS developer in Croatia. I have
been shipping my own apps on the App Store since 2022 ([Itemlist](https://getitemlist.app),
[QRGenie](https://qrgenie.app), [BarcodeCraft](https://barcodecraft.com)); this
skill distills what those submissions taught me, built and reviewed with Claude
Code and Codex. I write about building
iOS apps and working with AI tooling at [dabo.dev](https://dabo.dev), and I am on
[LinkedIn](https://www.linkedin.com/in/damjan-dabo/) and
[X](https://x.com/DamjanDabo).

Found a check that is wrong or missing? Issues and PRs are welcome. If this saved
you a rejection, a star helps others find it.

## License

MIT. See [LICENSE](LICENSE).
