# 06 — Feasibility, Effort Estimate & Roadmap (synthesis)

> **Superseded in part (2026-11-01):** for multi-currency, France ⇄ Tunisia and the revised hours, `07-cross-border.md` wins where it conflicts with this file.

> Research dimension 6: the synthesis of 01-market, 02-bank-sync, 03-tech-stack, 04-product and 05-compliance,
> **with the 2026-11-01 fact-check corrections applied**. Where a correction conflicts with a research file,
> the correction wins (see §0).
> Research date: 2026-11-01. Audience: a solo expert developer (bash/Docker/Linux) who is **new to Flutter/Dart
> in production**, building for personal use first and designing for a commercial product later.
>
> **Grading.** `[Verified: url]` = opened in this session (this file's session, or the fact-check pass that
> re-opened it today). `[Inferred: basis]` = follows from verified evidence or from search summaries that were not opened.
> `[Unverified: prior knowledge]` = recalled, not checked. `[Speculative]` = design or estimation judgment with
> no checkable fact behind it. **Every effort figure in this document is [Speculative]**: it is an estimate,
> built bottom-up from the work-item sizes in 02/03 and the feature list in 04.

---

## 0. Corrections applied (they override the research files)

| Topic | Research file said | Corrected position used here | Grade |
|---|---|---|---|
| French "premium method" app | none exists (01 §3) | Tiny entrants exist: **Finzee** (envelope / "reste à vivre", French localisation, manual entry, 7 App Store ratings, v1.0 Dec 2025) and Plan & Multiply (free envelope app). **No scaled brand.** Finzee's "reste à vivre" is direct prior art for safe-to-spend, so STS alone is **not** a differentiator. It has to be backed by the recurrence engine and the contract lifecycle | [Verified: https://apps.apple.com/app/id6743862740 (fact-check)]; Plan & Multiply [Inferred] |
| Copilot price | $9/mo or $109/yr | $9/mo is below 12 × $109/yr ÷ 12, so it is most likely an annual-equivalent or promotional price | [Verified: https://www.copilot.money/pricing (fact-check)] |
| Monarch | $14.99/mo, forecasting in Plus | $14.99 / $99.99 is most likely the **Core** price. The Plus price and the Plus feature split are not shown | [Verified: https://www.monarch.com/pricing (fact-check)] |
| Rocket Money Rowan | included in Premium+ $15/mo | Included, but launched 25 Aug 2026 as a **limited rollout, primary user only**, with no general-availability date | price [Verified: help.rocketmoney.com]; rollout [Inferred] |
| Résiliation "3 clics" | from 1 June 2023 | Art. L215-1-1 C. conso + décret 2023-417 (D215-1 to D215-3) took effect **1 June 2023**, with compliance due **1 Sept 2023**. It covers **consumer contracts concluded electronically** (or where online subscription is offered) and sets **no click count**. Notice periods and commitment terms still apply. Fines are up to €15k (natural person) / €75k (legal person) | [Verified: https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000047615001 (fact-check)] |
| GoCardless Bank Account Data | closed since July 2025 | Closed to new signups ("currently disabled", no reopening date). The July 2025 date rests on Actual Budget's docs. **Do not cite** `gocardless.com/bank-account-data/announcement`, which is about a legacy product discontinued in Dec 2023 | [Verified: https://bankaccountdata.gocardless.com/new-signups-disabled] |
| Enable Banking "free" | free restricted production | The mechanism is verified (linked accounts only, until an agreement is signed). "Free" is **Inferred**. Allowed uses are **evaluation or individual non-commercial** use only, the number of linked accounts may be capped, and API consent is still needed | [Verified: https://enablebanking.com/docs/api/linked-accounts/] |
| Consent validity | 180 days | 180 days is the **SCA re-authentication ceiling** (Reg. 2022/2360, from 25 Jul 2023). Each ASPSP can set a shorter `maximum_consent_validity`, so read it per bank | [Verified: https://enablebanking.com/docs/api/reference/] |
| "No provider publishes self-serve prices" | 02 TL;DR | Applies to the **licensed majors** only. open-banking.io (€3/mo + €1/account) and SimpleFIN ($1.50/mo, US only) do publish prices | [Verified: 02 §2.10] |
| Aggregator entry cost €200–600/mo | Inferred | Still Inferred, **lower confidence**: it rests on secondary sources, one of them from a conflicted author | [Inferred] |
| FiDA | not before 2027–2028 | Not adopted (no deal as of May 2026). With a phased rollout of about 4 years, the first obligations come **no earlier than 2028** and the full scope around **2030+** | [Verified: https://www.projectivegroup.com/fida-back-in-motion-from-ambitious-to-workable/ (fact-check)] |
| PSD3/PSR | adoption expected H2 2026 | **Not in the Official Journal** as of the latest sources (Apr–Jun 2026). Application is expected around **Q2/Q3 2028** | [Inferred: Matheson / banq.ai / Worldline via search (fact-check)] |
| Android notification listener on Play | "allowed" / "no declaration [Verified]" | **Unverified.** No Play text either allows or forbids it, and no dedicated declaration form was found. Treat it as a **Play review risk** under the User Data / core-functionality rules. The Financial features declaration is mandatory for **every** app on Play | declaration [Verified: https://support.google.com/googleplay/android-developer/answer/13849271]; listener [Unverified] |
| Android 15 OTP redaction | targets API 35 | Applies to **all apps** on Android 15, whatever their targetSdk | [Verified: https://developer.android.com/about/versions/15/behavior-changes-all] |
| FinanceKit | US/UK, organization account required | US/UK only, per-bundle entitlement, Finance category. The **"organization account required" claim is dropped**: the page does not say it. Either way FinanceKit is irrelevant for France | [Verified: https://developer.apple.com/financekit] |
| Apple 5.1.1(ix) / 3.2.1(viii) | reviewer-discretion risk | Text verified. Apple's own FinanceKit page calls **budgeting** a "financial management tool", which slightly **raises** the risk for an individual account. An organization (D-U-N-S) account before commercial launch is better supported than before | [Verified: https://developer.apple.com/app-store/review/guidelines/] |
| Apple EU DMA page "inconsistent" | 05 §5 | Not inconsistent. The old "Store Services Fee" was eliminated and replaced by the new "store services commission" tiers: 26/15 IAP, 20/10 alternative PSP, 15/10 link-out, 5% CTC | [Inferred: DMA page wording (fact-check)] |
| CNIL DPIA list | not listed | Confirmed not listed. Two items matter: **large-scale geolocation from a mobile app**, and **ad personalisation from external data**. Both argue for no location by default and no ads | [Verified: CNIL PDF (fact-check)] |
| Claude data retention | 30 days | 30-day default. Exceptions: data flagged by trust & safety is kept up to **2 years**, and data can be kept longer for legal or Usage Policy reasons. There is **no EU-pinned** inference option (only `us`/`global`) | [Verified: platform.claude.com data-residency + api-and-data-retention pages (fact-check)] |
| PowerSync encryption | assumed | PowerSync Flutter 2.x officially documents sqlite3mc/sqlcipher encryption via the same sqlite3 hook Drift uses. `drift_sqlite_async` is published by powersync.com. Each pair works, but **no source confirms all three together**, so the week-1 spike stays. `powersync_sqlcipher` is obsolete for 2.x | [Verified: docs.powersync.com data-encryption, pub.dev drift_sqlite_async (fact-check)] |
| Expo widgets on Android | score 4 | Android support is **new in SDK 58** and has no stability label, which weakens Expo's widget advantage | [Verified: https://expo.dev/changelog/sdk-58] |
| iOS accessory widgets | iOS 17 | The accessory families date from **iOS 16**. Interactive widgets date from iOS 17 | [Unverified: prior knowledge (Apple page rendered empty)] |
| Android lock-screen widgets | screensaver-only on phones | The trigger is **chosen by each OEM** (charging or docking are examples), and apps can opt out. Still unreliable, so the persistent-notification / quick-settings-tile fallback stands | [Verified: https://android-developers.googleblog.com/2025/03/widgets-on-lock-screen-faq.html (fact-check)] |
| Flutter 3.50 | due Nov 2026 | Beta 3.50.0-0.3.pre published 2026-10-29. The stable date is unconfirmed. Pin the SDK with FVM | [Verified: Flutter release feed (fact-check)] |
| Retired Claude models | 03 §9 | Add **Opus 4.1** to the retired list | [Verified: platform.claude.com pricing (fact-check)] |

---

## 1. Feasibility verdict per area

| # | Area | Verdict | Why | Grade |
|---|---|---|---|---|
| 1 | **Core ledger + budgets + bills/reminders + safe-to-spend** (offline, single user) | 🟢 **GREEN** | Mature, recently published packages cover every requirement: Drift 2.35.3, flutter_local_notifications 22.3.3, fl_chart 1.3.0, Riverpod 3.4.3. Reminders are scheduled on-device from the DB and need no server. Cashew (Flutter + Drift, ~4.8k★) shows a manual-first app can be loved without bank sync | packages [Verified: pub.dev API (03)]; Cashew [Verified: github.com/jameskokoska/Cashew] |
| 2 | **Encryption at rest + app lock** | 🟢 GREEN | `sqlite3` 3.x with `source: sqlite3mc` is the supported path; `sqlcipher_flutter_libs` is EOL. `local_auth` covers the UI lock | [Verified: sqlite3.dart UPGRADING_TO_V3.md; pub.dev sqlite3, local_auth] |
| 3 | **File import** (CSV/OFX/QIF, FR dialects) | 🟢 GREEN | No regulation, no cost, works for every bank, and is the fallback when consent expires. French banks commonly export CSV/OFX/QIF | [Verified: Boursobank FAQ (02)]; others [Inferred] |
| 4 | **Receipt OCR** | 🟢 GREEN for OCR, 🟠 AMBER for parsing | ML Kit Text Recognition v2 supports Latin script on Android and iOS. The hard part is parsing FR receipts (TTC vs HT, decimal comma) | [Verified: developers.google.com/ml-kit/vision/text-recognition/v2]; parsing difficulty [Speculative] |
| 5 | **Android auto-capture** (notification listener) | 🟢 personal / 🟠 **AMBER** product | Technically standard. Play policy is **unverified**, Android 15 redacts OTP-detected notifications for all apps, OEM classifiers may produce false positives, and bank-app formats change. Personal sideload or internal track: no issue | redaction [Verified: developer.android.com A15]; Play [Unverified] |
| 6 | **iOS auto-capture** | 🔴 **RED** for automatic capture, 🟠 AMBER via Shortcuts | iOS apps cannot read other apps' notifications. FinanceKit is US/UK only. The only path is the Shortcuts "Transaction" automation (Apple Pay taps only), which is reported flaky | FinanceKit [Verified: developer.apple.com/financekit]; Shortcuts [Inferred: forum threads via search] |
| 7 | **AI layer** (categorisation, NL entry, insights) | 🟢 cost / 🟠 **AMBER** privacy | Haiku 5.5 costs about $0.000065 per categorisation. The real constraint is GDPR: there is no EU-pinned inference, so a cloud call is a third-country transfer. Mitigation: on-device rules first, an opt-in proxy that sends the minimal string only | price [Verified: platform.claude.com pricing]; cost [Inferred: arithmetic]; residency [Verified: data-residency page] |
| 8 | **Bank sync — personal** | 🟢 GREEN | Enable Banking restricted mode on your own linked accounts, for non-commercial individual use. Expect a manual app review | [Verified: enablebanking.com/docs/api/linked-accounts/]; "free" [Inferred] |
| 9 | **Bank sync — product** | 🟠 **AMBER** | The blocker is commercial, not technical: company + KYB, quote-only majors, entry costs estimated at €200–600/mo (low confidence), and a GoCardless-style closure risk | [Inferred: 02 §2, §6] |
| 10 | **Multi-device sync + household** | 🟠 **AMBER** | The largest engineering cost. The Drift + PowerSync + sqlite3mc triple is unconfirmed (each pair is documented), and E2EE conflicts with the server acting as arbiter (§8, decision 3) | [Verified pairs: PowerSync docs, pub.dev]; triple [Inferred] |
| 11 | **Web client** | 🟠 AMBER | Flutter web is canvas-rendered: fine for a logged-in companion dashboard, poor for a marketing site (use a static site for that) | [Unverified: prior knowledge (03)] |
| 12 | **Store publication** | 🟠 **AMBER** | Play: 12 testers for 14 days (new personal accounts), the Financial features declaration, the Data safety form. Apple: 5.1.1(ix)/3.2.1(viii) can be invoked against an individual account, and DSA trader status makes the address public. Both are solved by an organization account | [Verified: Play answer/14151465, answer/13849271; Apple guidelines]; DSA [Inferred] |
| 13 | **GDPR** | 🟢 GREEN (local-first) → 🟠 AMBER (cloud) | Not on CNIL's mandatory-DPIA list. Local-only removes most of the burden. Cloud sync, AI and bank sync bring a short DPIA, DPAs, EU hosting and records of processing | [Verified: CNIL list PDF]; rest [Inferred/Unverified: 05 §3] |
| 14 | **Market / differentiation** | 🟠 AMBER | The gap is real (no scaled premium method app in France; incumbents drifted to B2B). But STS has prior art (Finzee, Buckwheat), the market is crowded, and distribution is the hard part. The defensible moat is the **French obligation layer**: contract lifecycle + résiliation timing, warranty clocks, frais réels | gap [Verified: 01 §3 pricing pages]; prior art [Verified: Finzee listing]; moat [Speculative] |
| 15 | **Monetisation** | 🟠 AMBER | French users anchor on freemium (Bankin' Plus from €3.33/mo). A €30–50/yr premium sits between freemium and US premium ($99–109/yr). The price is unvalidated | prices [Verified: bankin.com, ynab.com, monarch.com]; price point [Speculative] |
| 16 | **Regulatory future** (FiDA, PSD3/PSR) | ⚪ NEUTRAL | Neither changes a 2026–2028 build. FiDA ≥2028 (realistically 2030+), PSD3 around 2028. Build adapters, not bets | [Inferred: fact-check §0] |

**Overall verdict: FEASIBLE.** [Inferred: 9 green-or-green/amber areas, no red area on the critical path; the one red, iOS auto-capture, is a missing feature, not a blocker]. The personal daily driver is low-risk. The commercial product is feasible but carries business, store-policy and sync-architecture risk rather than technical risk.

---

## 2. Effort estimate (solo expert developer, new to Flutter/Dart)

### 2.1 Assumptions

| # | Assumption | Grade |
|---|---|---|
| A1 | Stack = Flutter 3.47/3.50 + Drift + Riverpod; PowerSync/Postgres from v2; Claude Haiku 5.5 via a proxy from v1 (the 03 recommendation) | [Verified: versions in 03]; choice [Speculative] |
| A2 | The developer knows software engineering, Docker, Linux, CI and backends well, but **Dart/Flutter, SwiftUI (WidgetKit) and Kotlin (Glance, NotificationListener) are new or rusty**. Ramp-up is budgeted explicitly in Phase 0, and a native-code premium is built into widget and listener items | [Speculative] |
| A3 | "Focused hours" = hands-on-keyboard engineering, testing and writing. Meetings don't exist. Admin (company, D-U-N-S, contracts) is counted as **calendar waits**, not hours | [Speculative] |
| A4 | Full-time = 35 focused h/wk; side-project = 10 focused h/wk (the task's paces). Real weeks lose time to context switching, which hurts the side-project pace most: each 10-h week loses ~1 h to "where was I" | [Speculative] |
| A5 | Scope per phase = the MoSCoW table in 04 §4, re-cut so that **sync + household + bank sync are v2** (per the roadmap structure requested), and "Could/Won't" items are out unless listed | [Speculative] |
| A6 | Estimates include tests (unit + Drift migration tests + golden tests on key screens) and a hardening buffer per phase (15–30%) | [Speculative] |
| A7 | iOS builds use a cloud Mac (Codemagic free tier: **500 macOS M2 min/month**, then $0.095/min) until a Mac is needed for SwiftUI widget work | [Verified: https://codemagic.io/pricing/] |
| A8 | No AI-assistance discount is applied in the base figures; §2.5 discusses it separately | [Speculative] |

### 2.2 Work breakdown (focused hours — optimistic / likely / pessimistic)

#### Phase 0 — Foundations

| Item | O | L | P | Basis |
|---|---|---|---|---|
| Dart/Flutter ramp-up (language, Riverpod, Drift codegen, platform channels) | 20 | 40 | 70 | A2 |
| Repo, FVM pin, lint, Docker-based Linux CI (Android build + tests), gitleaks, licence + SECURITY.md | 8 | 14 | 24 | 05 §7 hygiene |
| **Spike: Drift + sqlite3mc + PowerSync on Android + iOS** (go/no-go for the stack) | 24 | 32 | 48 | 03 §10: "3–5 days" |
| Domain model v1 (money as minor units, UUIDv7, soft delete, dated ParameterTable), schema, migration tests | 16 | 24 | 40 | 04 §6 |
| App shell, navigation, theming, i18n scaffold (FR/EN, ICU) | 12 | 20 | 32 | 04 §2.15 |
| **Phase 0 total** | **80** | **130** | **214** | |

#### MVP — personal daily driver (Android first)

| Item (04 §4 #) | O | L | P |
|---|---|---|---|
| Accounts, transactions, transfers, 2-level categories, tags, splits (#1, 3, 4) | 40 | 60 | 90 |
| 1-tap quick add, offline (#2) | 12 | 20 | 32 |
| CSV/OFX/QIF import + saved per-bank profiles + duplicate guard (#5) — 02 §6: 3–5 days | 24 | 36 | 60 |
| Rules engine + "rule from correction" (#6) | 16 | 28 | 44 |
| Category-limit budgets + 80/100% alerts (#7) | 16 | 24 | 40 |
| Recurring series (RRULE) + bills/subscriptions + on-device reminder scheduler + trial alerts (#8, 9) — 03 §10: 1–2 wk | 32 | 50 | 80 |
| Safe-to-spend, basic formula (#10) | 8 | 14 | 24 |
| Receipt photo attach (#11) | 6 | 10 | 16 |
| Multi-currency per account + manual rate (#12) | 10 | 16 | 28 |
| App lock, encryption on, full export, encrypted backup file to own cloud (#13; 05 §2.3 option A) | 16 | 26 | 40 |
| Android home widget (STS + quick add) in Glance (#15) | 12 | 20 | 36 |
| Basic charts / month view | 12 | 20 | 32 |
| Android notification-listener capture + 3–5 bank-app parsers, pending→confirm (02 §5.1; sideload or internal track) | 20 | 32 | 56 |
| Hardening, dogfood fixes (+15 / 20 / 25%) | 34 | 71 | 145 |
| **MVP total** | **258** | **427** | **723** |

#### v1 — smart features, iOS parity, still single-user

| Item (04 §4 #) | O | L | P |
|---|---|---|---|
| Recurring detection + price-hike detection (#16, 17) | 20 | 32 | 52 |
| Cash-flow forecast 30/60/90 + low point + variable-spend model (#18) | 24 | 40 | 64 |
| Envelope / zero-based mode on the same model (#19) | 16 | 28 | 44 |
| Goals + sinking funds in the forecast (#20) | 12 | 20 | 32 |
| On-device OCR + FR receipt field parser (#21) — 03 §10: 1–2 wk | 28 | 44 | 72 |
| Warranty / return-window vault with legal parameter table (#22) | 12 | 20 | 32 |
| Frais réels tagging + 10% comparator + km log + yearly scales + export note (#23, 24) | 24 | 40 | 64 |
| First backend: LLM proxy (rate limit, minimisation) + NL entry FR/EN + AI categorisation tiers (#27) — 03 §10: 1–2 wk | 28 | 44 | 72 |
| iOS home + lock-screen widgets (WidgetKit) + Android persistent notification / QS tile (#28) | 20 | 32 | 52 |
| Cancellation helper cards (cancel URL, notice period, timing) (#29) | 10 | 16 | 28 |
| Insights: month in review + anomaly rules (#30) | 16 | 28 | 44 |
| Accessibility AA pass + screen-reader money labels (#31) | 16 | 24 | 40 |
| Arabic + RTL + digit choice (#32) | 16 | 28 | 48 |
| iOS signing/build pipeline + Shortcuts "Transaction" → App Intent | 20 | 36 | 60 |
| Hardening (+15 / 20 / 25%) | 39 | 86 | 176 |
| **v1 total** | **301** | **518** | **880** |

#### v2 — sync, household, bank sync, web companion

| Item | O | L | P |
|---|---|---|---|
| Backend: Postgres + self-hosted PowerSync + auth + upload handler + RLS/row rules; multi-device sync (#25) | 60 | 100 | 170 |
| Household: members, roles, private spaces, split rules, balances, min-transfer settle-up, invites (#26) | 50 | 80 | 130 |
| Web companion (Flutter web, read-mostly dashboard + export) (#25) | 30 | 50 | 90 |
| `TransactionSource` adapter + Enable Banking (JWT app auth, consent, sync job, re-consent reminder) + cross-source reconciliation/dedup (notification vs booked) — 02 §6: 4–7 days for the connector alone | 40 | 64 | 110 |
| Merchant cleaning for bank libellés | 8 | 16 | 28 |
| Ops: EU VPS, Docker Compose, backups + restore drill, monitoring (GlitchTip/self-hosted) | 16 | 28 | 48 |
| Hardening (+20 / 25 / 30%) | 41 | 85 | 173 |
| **v2 total** | **245** | **423** | **749** |
| *Option: true E2EE sync instead of server arbiter (key management, recovery UX, encrypted fields)* | *+40* | *+80* | *+160* |

#### Product launch — store, monetisation, compliance

| Item | O | L | P |
|---|---|---|---|
| Billing: RevenueCat + IAP/Play Billing, paywall, entitlement gating (bank sync / sync / AI as premium) | 20 | 32 | 56 |
| In-app account deletion + web deletion link (Play + Apple rule) | 6 | 10 | 16 |
| Privacy policy, terms, short DPIA, records of processing, processor DPAs | 16 | 28 | 48 |
| Store listings, Data safety, privacy labels, Financial features declaration, screenshots, static marketing site | 20 | 32 | 52 |
| MASVS L1 self-assessment + hardening (Play Integrity / App Attest for the API, FLAG_SECURE, private notifications) | 16 | 28 | 48 |
| Onboarding, empty states, in-app help | 12 | 20 | 32 |
| Crash reporting + privacy-friendly analytics (self-hosted) | 6 | 10 | 16 |
| Review-rejection buffer (Play listener justification, Apple 5.1.1(ix) back-and-forth) | 8 | 20 | 48 |
| **Launch total** | **104** | **180** | **316** |

### 2.3 Totals and calendar weeks

| Phase | Focused hours O / L / P | Full-time (35 h/wk) weeks O / L / P | Side-project (10 h/wk) weeks O / L / P |
|---|---|---|---|
| Phase 0 Foundations | 80 / 130 / 214 | 2.3 / 3.7 / 6.1 | 8 / 13 / 21 |
| MVP (daily driver) | 258 / 427 / 723 | 7.4 / 12.2 / 20.7 | 26 / 43 / 72 |
| v1 (smart features) | 301 / 518 / 880 | 8.6 / 14.8 / 25.1 | 30 / 52 / 88 |
| v2 (sync + household + bank) | 245 / 423 / 749 | 7.0 / 12.1 / 21.4 | 25 / 42 / 75 |
| Product launch | 104 / 180 / 316 | 3.0 / 5.1 / 9.0 | 10 / 18 / 32 |
| **Total** | **≈ 990 / 1 680 / 2 880** | **≈ 28 / 48 / 82 wk** (≈ 6.5 / 11 / 19 months) | **≈ 99 / 168 / 288 wk** (≈ 2 / 3.2 / 5.5 years) |
| + E2EE option | +40 / +80 / +160 | +1 / +2.3 / +4.6 | +4 / +8 / +16 |

Cumulative **likely** milestones, full-time: foundations end of week ~4; **daily driver ~week 16 (≈ 4 months)**; v1 ~week 31; v2 ~week 43; launch ~week 48 plus calendar waits. Side-project: **daily driver ~week 56 (≈ 13 months)**; launch ~3+ years. [Speculative: sums of the tables above]

**Calendar waits that hours do not cover** (they can overlap with development if started early):

| Wait | Duration | Grade |
|---|---|---|
| Play closed test: ≥12 testers opted in for 14 consecutive days (new personal accounts) | ≥ 2 weeks + recruiting | [Verified: https://support.google.com/googleplay/android-developer/answer/14151465] |
| D-U-N-S number for organization accounts | free, up to ~28 days | [Verified: 05 §8, support.google.com/android-developer-console/answer/16561738] |
| Company creation (micro-entreprise or SASU) | days to weeks | [Unverified: prior knowledge] |
| Enable Banking production app review (even the personal path) | unknown; manual | [Inferred: 02 §2.2] |
| Commercial aggregator quote + contract + KYB | weeks to months | [Inferred: 02 §6] |
| App Store / Play review per submission | days; longer on rejection | [Unverified: prior knowledge] |

### 2.4 Lean path: what a ruthless cut looks like

If the side-project pace is the reality, the full plan is about 3 years. A **lean daily driver** (Android only, no widget, no multi-currency, no rules UI beyond auto-learned rules, notification capture plus CSV import, bills + reminders + basic STS) is roughly **Phase 0 (110 h, because the iOS half of the spike is deferred) + ~250 h ≈ 360 h likely**. That is **≈ 10 weeks full-time or ≈ 36 weeks side-project**. The trade-off: iOS, and therefore commercial reach, is deferred, and the stack decision is validated on Android only. [Speculative]

### 2.5 Effect of AI-assisted coding

| Evidence | What it says | Grade |
|---|---|---|
| METR RCT (early-2025 tools, 16 experienced OSS developers, 246 tasks on their own mature repos) | Developers were **19% slower** with AI, while believing they were ~20% faster. METR itself says the result may not generalise to unfamiliar codebases or less mature projects, and now labels it out of date | [Verified: https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/] |
| METR follow-up (Feb 2026, late-2025 tools) | Point estimates lean toward a **speedup** (returning devs ~18%, new devs ~4%), but the confidence intervals include zero and METR flags selection bias | [Inferred: secondary blogs summarising METR; primary post not opened] |

How it applies here [Speculative]:

- This project is the setting METR said it might **not** cover: **greenfield, small, an unfamiliar language (Dart), and boilerplate-heavy** (CRUD screens, Drift tables, parsers, i18n strings, tests, platform-channel glue). That is where assistants plausibly help most.
- It does **not** help with calendar waits (testers, D-U-N-S, reviews, sales), product judgment, or the hardest engineering: sync semantics, reconciliation, OEM background quirks, store negotiations.
- Planning factor: apply **×0.75–0.85 to the coding-heavy items** (≈ 60% of hours) and ×1.0 to the rest, which gives **≈ 10–15% fewer total hours** (likely total ≈ 1 450–1 520 h instead of 1 680). Treat any larger "felt" gain with suspicion, given METR's 40-point perception gap. **Measure it**: log the hours per item against this table and recalibrate after the MVP.
- Biggest practical value for a Flutter newcomer: cutting the Phase 0 ramp-up and the Swift/Kotlin native-glue learning curve, provided every generated piece of money-handling code is covered by tests (integer minor units, rounding, FX).

---

## 3. Phased roadmap

```
Weeks (full-time, likely)  0    4         16             31          43    48
                           |P0--|---MVP---|-----v1------|----v2-----|Launch|
Side-project equivalent    0   13        56            108         150   168
Parallel calendar tracks:          [12 testers x 14 d]   [company + D-U-N-S] [aggregator quotes]
```

### Phase 0 — Foundations (L ≈ 130 h; FT ~4 wk / SP ~13 wk)

- **Milestones:** M0.1 repo + CI green on Android; M0.2 **stack spike verdict** (Drift + sqlite3mc + PowerSync, Android + iOS via cloud Mac); M0.3 schema v1 with migration tests; M0.4 app shell in FR/EN.
- **Exit criteria:** (a) the spike shows an encrypted DB that PowerSync can sync and Drift can query, on both OSes, **or** a documented fallback (e.g. Drift + sqlite3mc now, own sync later; or switch to Expo); (b) a money-type test suite (minor units, currency, rounding) passes; (c) a "kill switch" mutation test proves the migration tests notice a schema change.
- **Risks:** interop failure (R3); Flutter learning curve underestimated (R2); no Mac for iOS (R9).

### MVP — personal daily driver (L ≈ 427 h; FT ~12 wk / SP ~43 wk)

- **Scope:** 04 §4 #1–15 plus Android notification capture. Android first, sideloaded or internal track; iOS compiles but is not polished.
- **Milestones:** M1.1 ledger + quick add usable; M1.2 CSV/OFX import of your own bank exports; M1.3 bills + reminders firing with the app killed; M1.4 STS on the widget; M1.5 notification capture with your own bank app(s).
- **Exit criteria (daily-driver test):** **30 consecutive days** of real use where (a) every expense is captured within 24 h, (b) no reminder is missed (verified against a log), (c) the backup restores on a clean device, (d) no data-loss bug is open. [Speculative]
- **Risks:** scope creep (R1); OEM battery killers delaying reminders (R7); notification formats changing.

### v1 — smart features (L ≈ 518 h; FT ~15 wk / SP ~52 wk)

- **Scope:** recurrence detection, price hikes, forecast, envelope mode, goals, OCR, warranty vault, frais réels + km log, LLM proxy + NL entry, iOS parity (widgets, lock screen, Shortcuts intent), cancellation helper, insights, a11y AA, Arabic RTL.
- **Milestones:** M2.1 forecast + low point; M2.2 obligation radar (bills, trials, renewals, warranties in one timeline with actions; this is the real differentiator, given the Finzee prior art); M2.3 OCR receipt → vault; M2.4 frais-réels comparator with dated parameter tables sourced to service-public; M2.5 iOS TestFlight build; M2.6 **Play closed test with 12 testers for 14 days started** (it can run during v1 and doubles as user research).
- **Exit criteria:** (a) 12+ external testers active for 14 days with crash-free sessions ≥ 99% [Speculative target]; (b) categorisation accuracy ≥ 90% after a few weeks on your own data, measured; (c) the LLM proxy sends only minimal fields (an audit test asserts that no IBAN, name or image leaves the device); (d) a11y AA checklist passed.
- **Risks:** OCR parsing quality on FR receipts; the LLM third-country transfer (R8); Apple review of the beta (R5); the Play listener policy (R4).

### v2 — sync, household, bank sync (L ≈ 423 h, +80 h for E2EE; FT ~12 wk / SP ~42 wk)

- **Scope:** Postgres + PowerSync (EU host), auth, multi-device, household (shared budgets, private spaces, settle-up), web companion, `TransactionSource` adapter + Enable Banking restricted (your own accounts) + reconciliation.
- **Milestones:** M3.1 two devices in sync, conflict tests green; M3.2 household of 2 with private spaces and a settle-up; M3.3 Enable Banking pulls booked transactions and reconciles them with notification-captured pendings; M3.4 restore-from-backup drill on the server.
- **Exit criteria:** (a) offline edits on two devices converge with no lost rows in a scripted chaos test; (b) the household balance equals allocations − payments − settlements under property-based tests; (c) consent expiry triggers the re-consent reminder ≥ 7 days ahead; (d) the DPIA draft is written.
- **Risks:** the sync-architecture choice (R6); aggregator terms changing (R5→R6 list); server ops burden.

### Product launch — store, monetisation, compliance (L ≈ 180 h; FT ~5 wk + waits / SP ~18 wk)

- **Prerequisites started earlier, in parallel:** company creation, D-U-N-S, organization accounts on Apple (99 USD/yr) and Play (US$25 one-time); aggregator quotes (Enable Banking, Powens, Bridge, Salt Edge Partner Program).
- **Milestones:** M4.1 paywall + entitlements via RevenueCat; M4.2 privacy policy, terms, DPIA, records of processing, DPAs signed; M4.3 store listings + Data safety + Financial features declaration; M4.4 production release FR first; M4.5 commercial aggregator contract signed (or bank sync stays personal-only, and the free tier ships without it).
- **Exit criteria:** both stores approved; account deletion works in-app and via the web link; MASVS L1 self-assessment documented; the unit economics sheet shows the bank-sync tier covers the aggregator minimum at the projected paying-user count.
- **Risks:** Apple 5.1.1(ix)/3.2.1(viii) (R5); aggregator minimums (R6); weak conversion in a freemium-anchored market (R10).

---

## 4. Running cost table

Assumptions: prices excl. VAT unless stated. The "1k-user product" = 1 000 monthly active users, ~10% paying, ~300 with bank sync enabled. [Speculative assumptions]

| Item | Personal phase | 1k-user product | Grade |
|---|---|---|---|
| Apple Developer Program | 99 USD/yr, needed only for distribution / TestFlight; free on-device provisioning has short-lived profiles | 99 USD/yr (organization account) | fee [Verified: https://developer.apple.com/programs/enroll/]; free-provisioning detail [Unverified] |
| Google Play registration | US$25 one-time (optional while sideloading) | US$25 one-time | [Verified: https://support.google.com/googleplay/android-developer/answer/6112435] |
| Store commission on subscriptions | — | Apple SBP 15% (EU IAP reduced tier 15%); Play EEA subscriptions 10% + 5% billing fee on the first $1M | [Verified: Apple SBP + DMA pages; Play answer/112622 (05, fact-check)] |
| Hosting (Postgres + PowerSync Open Edition + API + monitoring) | €0 in MVP (no backend). From v1 the LLM proxy runs on an existing box or a Hetzner CPX02 (1 vCPU, 1 GB) at **€6.49/mo** | 2 × Hetzner CPX22 (2 vCPU, 4 GB, EU) at **€19.99/mo** each ≈ €40/mo + backup storage. Managed alternative: Supabase Pro **$25/mo** + PowerSync Pro **from $49/mo** ≈ $74/mo | Hetzner [Verified: https://www.hetzner.com/cloud/regular-performance/ — page showed "currently unavailable" on Create buttons]; Supabase/PowerSync [Verified: 03 §5 pricing pages] |
| PowerSync licence | Open Edition self-host: free, source-available (licence text not read) | same, or Pro from $49 | [Verified: powersync.com/pricing]; licence terms [Unverified] |
| Bank aggregator | Enable Banking restricted mode: €0 (evaluation / individual non-commercial) | **€200–600/mo + per-connection fees** until real quotes arrive (low confidence) | personal [Inferred: terms snippet]; product [Inferred: secondary sources, 02 §6] |
| LLM API (Claude Haiku 5.5 categorisation + Sonnet 5.5 monthly insight) | ~$0.02/mo for a heavy user (300 tx) | Upper bound **~$0.05 per active user/mo**, so ≈ **$50/mo** at 1k MAU (categorisation ≈ $0.02 + one monthly Sonnet insight of 5k in / 1k out ≈ $0.02); lower with caching, the Batch API and on-device Tier 0 rules | prices [Verified: platform.claude.com pricing]; usage [Inferred: arithmetic on 03 §6.2 assumptions] |
| Subscription infrastructure (RevenueCat) | — | Free up to **$2,500 monthly tracked revenue**, then 1% of tracked revenue (the page does not say whether 1% applies to the full amount or only the excess) | [Verified: https://www.revenuecat.com/pricing/] |
| iOS CI (cloud Mac) | Codemagic free: 500 macOS M2 min/month | Pay-as-you-go **$0.095/min** (M2) → ~$10–30/mo for a weekly release train | free/rate [Verified: https://codemagic.io/pricing/]; monthly figure [Inferred] |
| Domain | ~€10–20/yr | ~€10–20/yr (+ transactional e-mail a few €/mo) | [Unverified: prior knowledge] |
| Crash + analytics | self-hosted on the same VPS: €0 | €0 marginal (GlitchTip / Plausible self-hosted) | [Unverified: prior knowledge (05 §3.2)] |
| Legal / company | €0 (DIY privacy policy) | Lawyer review: several hundred to a few thousand € one-off; company accounting if SASU | [Unverified: prior knowledge (05 §8)] |
| **Monthly total** | **≈ €0–15/mo** (plus 99 USD/yr if iOS distribution) | **≈ €100–200/mo without bank sync; ≈ €350–850/mo with bank sync** | [Inferred: sums of rows] |

**Break-even sketch** [Speculative]: at €4/mo (≈ €3.40 net after a 15% store fee), covering €350–850/mo takes **≈ 100–250 paying users**, consistent with 02 §6's "50–150 paying users cover the aggregator" for its lower aggregator band. Gate bank sync behind premium so free users cost only hosting + LLM cents.

---

## 5. Top 10 risks and mitigations

| # | Risk | Likelihood / impact | Mitigation | Grade |
|---|---|---|---|---|
| R1 | **Scope creep / never shipping**: 49 features in 04, and "genius, complete" invites more | High / High | Hard phase gates with exit criteria (§3); the MVP is the 30-day daily-driver test, not a feature count; keep a "v-later" list; the lean path (§2.4) as fallback | [Speculative] |
| R2 | **Side-project pace stretches the plan to 3+ years**; motivation decays | High (at 10 h/wk) / High | Lean path first; ship to yourself early; time-box each phase at 1.3× the likely figure, then cut scope rather than extend | [Inferred: §2.3 arithmetic] |
| R3 | **Drift + sqlite3mc + PowerSync triple fails** (each pair is documented, the triple is not) | Medium / High | Phase 0 spike on both OSes as a go/no-go; fallback (a) Drift + sqlite3mc with your own sync later, (b) PowerSync's own SQLite without Drift, (c) Expo + op-sqlite + PowerSync RN | [Verified pairs: PowerSync docs, pub.dev; triple Inferred] |
| R4 | **Google Play rejects or limits the notification-listener capture** | Medium / Medium | Opt-in, on-device only, allow-listed bank packages, prominent disclosure, works when declined; ship it as a separate flavour or a non-Play build if rejected; re-check Play Console help before submission; **never use SMS permissions** | listener policy [Unverified]; SMS rule [Verified: Play answer/10208820] |
| R5 | **Apple 5.1.1(ix) / 3.2.1(viii) invoked against an individual account**, plus the DSA trader address becoming public | Medium / High | Position as an expense organiser (no advice, payments or aggregation in iOS v1); create a company + D-U-N-S and move to organization accounts **before** commercial launch | [Verified: guidelines text]; interpretation [Inferred] |
| R6 | **Aggregator access or cost changes** (the GoCardless precedent; Enable Banking's free mode is personal-only and could change) | Medium / High (for the bank-sync tier) | `TransactionSource` adapter; file import + capture stay first-class; bank sync premium-only; quotes from ≥ 3 providers (Enable Banking, Powens, Bridge, Salt Edge Partner); note that Bridge belongs to a direct competitor (Bankin') | [Verified: GoCardless closed; Enable Banking linked-accounts docs] |
| R7 | **Reminders fail to fire** (OEM battery killers, exact-alarm permission, iOS background throttling); the "reminder" pillar loses trust | Medium / High | Schedule from the DB on every start, sync, boot and edit; use `setAlarmClock()`-style exact alarms where permitted; an in-app "missed reminders" audit log; a reliability test matrix on 3–4 OEMs | [Unverified: prior knowledge (03 §4.2)] |
| R8 | **Privacy / GDPR exposure from cloud AI and sync** (no EU-pinned Claude inference; T&S retention up to 2 years; plaintext financial data on the server) | Medium / High | Tier 0 on-device rules by default; AI opt-in per feature; a minimisation proxy with an audit test; EU-hosted backend; DPA + SCC disclosure; DPIA before v2; consider Bedrock/Vertex EU endpoints (region availability not checked) | [Verified: data-residency + retention pages]; EU partner regions [Inferred] |
| R9 | **iOS development friction**: no local Mac, native SwiftUI widgets, Kotlin Glance/listener code, all new skills | Medium / Medium | Codemagic free 500 min for builds; defer iOS widgets to v1; budget a Mac (rented or used) before the WidgetKit work; keep native code thin (data snapshot + intents only) | [Verified: codemagic.io/pricing]; Mac need [Inferred] |
| R10 | **Weak differentiation / distribution**: STS has prior art (Finzee, Buckwheat); French users anchor on €3.33/mo freemium | Medium / High (commercial only) | Lead with the **French obligation layer** (contract lifecycle + résiliation timing, warranty clocks, frais réels), which global apps won't build; use the 12-tester closed test as user research; validate willingness to pay before building v2 | [Verified: Finzee listing, Bankin' pricing]; strategy [Speculative] |

Also tracked (not in the top 10): data-loss bugs in a money app (mitigation: migration tests, restore drills, export from day 1); Flutter web limits; FiDA/PSD3 timing (no dependency, so no exposure).

---

## 6. The 5 decisions to make next

| # | Decision | Options | Recommendation | Grade |
|---|---|---|---|---|
| D1 | **UI stack** | Flutter · React Native + Expo SDK 58 · KMP/CMP · native ×2 · PWA | **Flutter 3.47 (pin with FVM, move to 3.50 when it is stable)**, **conditional on the Phase 0 spike**. Fallback to Expo if the Drift/PowerSync/sqlite3mc triple fails or the web becomes first-class. Reasons: toolchain already installed; one language; the strongest offline data story; Expo's Android widgets are brand new | [Verified: versions + Expo SDK 58 changelog]; choice [Speculative] |
| D2 | **MVP platform** | Android-first · iOS-first · both from day 1 | **Android-first** for the daily driver: the notification listener exists only there, no Mac is needed, and sideloading avoids store review during dogfooding. iOS parity in v1 | [Inferred: 03 §4.1, 02 §5.2] |
| D3 | **Sync architecture** (decide before v2; a costly door to reverse) | (a) server-arbiter PowerSync + Postgres on an EU host; (b) E2EE op-log relay; (c) encrypted backup files only | **MVP: (c)** (an encrypted backup to your own cloud). **v2: (a)** self-hosted in the EU, because household balances need server-side validation and it is ~80 h cheaper, with encryption at rest and field-level client encryption for notes and receipts. Keep (b) as a paid "vault" option if the privacy positioning demands it. The data model (UUIDv7, soft delete, owner/household IDs) stays compatible with all three | [Speculative; costs from §2.2] |
| D4 | **Bank sync: when and with whom** | none · personal Enable Banking · commercial aggregator at launch | **Personal Enable Banking in v2** (or earlier, once the MVP is stable, for your own use only). For the product: **premium-gated**, contracted only after the company exists, with quotes from Enable Banking, Powens, Bridge and the Salt Edge Partner Program. Never make it the core value | [Verified: Enable Banking linked-accounts docs; GoCardless closed]; strategy [Inferred: 02 §6] |
| D5 | **Licence + legal form** | MIT/Apache · AGPL + CLA · FSL-1.1 · closed; micro-entreprise vs SASU, and when | **FSL-1.1-ALv2 now** for the public repo: source-available, each version turns Apache-2.0 after 2 years, and it blocks free-riding competitors. Synthetic fixtures only, gitleaks in CI. **Create the company at the start of the launch phase** (D-U-N-S ≤ 28 days, org accounts, KYB for the aggregator, a hidden home address on EU store pages) | FSL [Verified: https://fsl.software/]; D-U-N-S [Verified: 05 §8]; legal form [Unverified: prior knowledge] |

---

## 7. What this synthesis did not verify

- The effort figures are bottom-up estimates [Speculative] and should be recalibrated with logged hours after Phase 0 and the MVP.
- Commercial aggregator pricing (only quotes settle it), the PowerSync Open Edition licence text, Google Play's stance on notification access, and Bedrock/Vertex EU-region availability for Claude models are unverified.
- METR's 2026 follow-up numbers come from secondary summaries; the primary post was not opened.
- The Hetzner prices were read today, but the page showed the products as "currently unavailable" to order. Re-check them before relying on them, or price Scaleway/OVHcloud as alternatives (not checked).

---

## Sources

**Opened in this session (synthesis pass):**
- Hetzner Cloud: https://www.hetzner.com/cloud , https://www.hetzner.com/cloud/regular-performance/
- RevenueCat pricing: https://www.revenuecat.com/pricing/
- Codemagic pricing: https://codemagic.io/pricing/
- METR early-2025 study: https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/

**Opened in the 2026-11-01 fact-check pass (relied on here):**
- https://apps.apple.com/app/id6743862740 (Finzee)
- https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000047615001
- https://bankaccountdata.gocardless.com/new-signups-disabled
- https://enablebanking.com/docs/api/linked-accounts/ , https://enablebanking.com/docs/api/reference/
- https://developer.apple.com/financekit , https://developer.apple.com/app-store/review/guidelines/ , https://developer.apple.com/support/dma-and-apps-in-the-eu/ , https://developer.apple.com/programs/enroll/ , https://developer.apple.com/app-store/small-business-program/
- https://support.google.com/googleplay/android-developer/answer/14151465 , /answer/13849271 , /answer/10208820 , /answer/112622 , /answer/6112435
- https://developer.android.com/about/versions/15/behavior-changes-all
- https://android-developers.googleblog.com/2025/03/widgets-on-lock-screen-faq.html
- https://platform.claude.com/docs/en/about-claude/pricing , https://platform.claude.com/docs/en/manage-claude/data-residency , https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
- https://www.powersync.com/pricing , https://docs.powersync.com/client-sdks/advanced/data-encryption.md , https://pub.dev/packages/drift_sqlite_async
- https://supabase.com/pricing
- https://expo.dev/changelog/sdk-58
- https://storage.googleapis.com/flutter_infra_release/releases/releases_linux.json
- https://www.projectivegroup.com/fida-back-in-motion-from-ambitious-to-workable/
- https://www.cnil.fr/sites/default/files/atoms/files/liste-traitements-avec-aipd-requise-v2.pdf
- https://fsl.software/
- https://www.ynab.com/pricing , https://www.monarch.com/pricing , https://www.copilot.money/pricing , https://bankin.com/fr/pluspro.html

**Search summaries only (Inferred):**
- METR Feb 2026 follow-up, via https://blog.codercops.com/blog/ai-coding-productivity-2x-not-10x-metr-2026 and https://dev.to/vibehackers/ai-coding-tools-and-productivity-what-the-controlled-evidence-shows-1emf
- PSD3/PSR timing (Matheson, banq.ai, Worldline), via the fact-check pass

**Upstream research files:** 01-market.md, 02-bank-sync.md, 03-tech-stack.md, 04-product.md, 05-compliance.md (same directory).
