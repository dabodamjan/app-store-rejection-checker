![app-store-rejection-checker](assets/banner.png)

# app-store-rejection-checker

A Claude Code skill that audits your iOS or iPadOS app before App Store
submission and tells you what is likely to get rejected, with the guideline number,
the evidence it found in your repo, a concrete fix, and an honest confidence level.

It is not a linter with a fixed rule list. The skill starts every audit by fetching
the current App Store Review Guidelines from Apple's site, then combines two kinds of
analysis:

- **Static checks**: purpose strings vs the APIs you actually link, privacy manifest
  and required-reason declarations, entitlements, export compliance, IAP and restore
  wiring, account deletion, App Tracking Transparency, web-wrapper fingerprints.
- **Semantic review**: Claude reads your actual code and your actual store copy and
  judges the things mechanical tools cannot: does your description match what the app
  does, is your app differentiated enough for a saturated category, are your review
  notes and demo account good enough for a reviewer to reach your core flow, does your
  AI feature disclose and get consent before sharing personal data with a third-party
  AI service.

Because the live guidelines are fetched at run time, the skill greatly reduces
guideline drift: the main guidelines text is always current, and the audit checks
Apple's news feed for recent announcements (a headline scan, not exhaustive — some
auxiliary policy pages change without an announcement, so the skill fetches those
pages when a finding depends on them). The bundled reference files carry the
rejection landscape: the only frequency data Apple publishes, plus a
confidence-labeled ranking of rejection clusters; the live text is always the
authority on conflict.

## Install

The skill lives in `skills/app-store-rejection-checker/`. Three ways to use it:

**Personal (all your projects):**

```bash
git clone https://github.com/dabodamjan/app-store-rejection-checker.git
mkdir -p ~/.claude/skills
cp -R app-store-rejection-checker/skills/app-store-rejection-checker ~/.claude/skills/
```

**Project (one repo):**

```bash
git clone https://github.com/dabodamjan/app-store-rejection-checker.git
mkdir -p .claude/skills
cp -R app-store-rejection-checker/skills/app-store-rejection-checker .claude/skills/
```

**Plugin:** the `skills/` layout is plugin-compatible; if you package this repo as a
Claude Code plugin, the skill loads namespaced as `<plugin-name>:app-store-rejection-checker`.

Then, in a session inside your app repo, either invoke it directly:

```
/app-store-rejection-checker
```

or just ask. Naming the skill in the request makes triggering reliable:

```
Check my app for App Store rejection risks before I submit using app-store-rejection-checker.
```

The audit covers more if you also paste your App Store Connect metadata (description,
keywords, age rating, review notes, demo account) when asked; the store listing is a
real rejection surface, not decoration.

## Use with other agents

The skill is written in the open [Agent Skills format](https://agentskills.io): a
folder with a `SKILL.md`, using only fields from the spec in its frontmatter (one of
them, `allowed-tools`, the spec marks experimental, so support may vary between
agents). It is not tied to Claude Code. Cursor, Codex CLI, and Gemini CLI support
the format natively (agentskills.io lists the full set of adopters); copy
`skills/app-store-rejection-checker/` into whatever directory your agent reads skills
from. The instructions are portable, subject to each agent's tool support — steps
that fetch Apple's live pages need whatever fetch or browser tool your agent
exposes, and tool names and permissions differ between agents.

For an agent without native skill support, point it at
`skills/app-store-rejection-checker/SKILL.md` and ask it to follow the procedure. The skill is
instructions plus reference files, not code, so any agent that can read files and
fetch the live guidelines can run the audit.

## What it checks

| Area | Examples |
|---|---|
| Completeness signals (2.1) | demo account and review notes quality, placeholder content |
| Metadata accuracy (2.3.x) | description vs actual code, screenshots, name and keyword rules, age rating answers |
| Payments (3.1.x) | digital goods outside IAP, restore purchases, subscription disclosure, loot box odds |
| Minimum functionality (4.2) | web-wrapper fingerprints, thin AI-wrapper patterns |
| Differentiation (4.3) | saturated-category exposure, template output signals |
| Login services (4.8) | social login without a privacy-protecting alternative |
| Privacy (5.1.x) | purpose strings, privacy manifest, account deletion, ATT, third-party AI data sharing |
| Special categories | kids, medical, gambling, VPN, crypto, finance, UGC obligations |
| Recent rule changes | 2024-2026 surfaces: privacy manifests, AI disclosure, age rating overhaul, EU DMA |

Every guideline number in the bundled references was verified against the live
guidelines text on 2026-08-20, and each run re-fetches the main guidelines and quotes
finding-relevant wording from that live text. Auxiliary policy surfaces (account
deletion, EU DMA terms, age ratings, required-reason APIs) are fetched when a finding
depends on them rather than re-verified wholesale on every run.

## Example finding

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

## Honest limits

- The skill sees your repository and the metadata you share, plus the live guidelines.
  It cannot verify runtime behavior: crashes, broken links, restore-purchase flows,
  OAuth round trips, server-driven content, or anything a reviewer only sees on a
  device. The report lists these as unchecked rather than pretending.
- Findings are risk assessments with stated confidence, not verdicts. App Review is
  run by people; no tool can guarantee approval, and this one does not claim to.
- This project is not affiliated with or endorsed by Apple. App Store, Xcode, and
  related marks belong to Apple Inc.

## Prior art and positioning

This space is not empty, and the existing tools are good at what they do:

- [fastlane precheck](https://docs.fastlane.tools/actions/check_app_store_metadata/)
  runs 10 text-pattern rules over App Store Connect metadata (curse words,
  placeholder text, broken URLs, mentions of other platforms).
- [greenlight](https://github.com/RevylAI/greenlight) is the strongest mechanical
  checker I found: an offline Go CLI covering private API use, privacy manifests,
  payment-evasion patterns, deprecated APIs, and more, plus a Revyl-backed tier that
  runs account-deletion, restore-purchase, and Sign in with Apple flows on cloud
  simulators.
- [berkayturk/appstore-precheck](https://github.com/berkayturk/appstore-precheck)
  already combines static checks with LLM analysis and publishes an eval harness.
  [cruisediary/apple-app-review-skills](https://github.com/cruisediary/apple-app-review-skills)
  covers the review surface with a large set of prompt-driven skills and agents.
  [dpearson2699/swift-ios-skills](https://github.com/dpearson2699/swift-ios-skills)
  ships a broader iOS skill set including an app-store-review skill (note: under the
  PolyForm Perimeter license, which is source-available rather than a permissive
  open-source license).

What this skill does differently: it fetches the live guidelines at run time instead
of freezing a rule list, it does semantic review of your real code and store copy
rather than pattern matching alone, its reference data sticks to the only frequency
numbers Apple publishes plus a confidence-labeled ranking instead of recycled listicle
statistics, and it is MIT licensed.

## Who made this

I am [Damjan Dabo](https://dabo.dev), an indie iOS developer in Croatia. I ship my own
apps on the App Store ([Itemlist](https://getitemlist.app), [QRGenie](https://qrgenie.app),
[BarcodeCraft](https://barcodecraft.com)) and have been submitting to App Review for
eight-plus years. This skill distills what those submissions taught me into something
reusable, built and reviewed with Claude Code.

Found a check that is wrong or missing? Issues and PRs are welcome. If this saved you
a rejection, a star helps others find it. I am also on
[X](https://x.com/DamjanDabo).

## License

MIT. See [LICENSE](LICENSE).
