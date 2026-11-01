# 03 — Technology Stack Research

> Dimension: TECHNOLOGY STACK for an expense manager + reminder + tracker app.
> Research date: 2026-11-01. Mobile-first, open to web; personal use first, built to become a commercial product.
>
> **Grading key** — every claim carries one:
> - **[Verified: url]** — the source was opened in this research session (WebFetch, or a registry API queried with `curl`).
> - **[Inferred: basis]** — follows from verified evidence, or comes from a search-result summary that was not opened directly.
> - **[Unverified: prior knowledge]** — recalled; there is a checkable fact, but it was not checked here.
> - **[Speculative]** — design judgement; no checkable fact.

---

## 0. TL;DR

| Decision | Choice | Grade |
|---|---|---|
| **Primary UI stack** | **Flutter 3.47.x (Dart 3.13)** with Riverpod, Drift, fl_chart, flutter_local_notifications, home_widget, google_mlkit_text_recognition | Versions [Verified: pub.dev API + Flutter release feed, see §2]; choice [Speculative] |
| **Runner-up** | **React Native 0.88 + Expo SDK 58** (expo-sqlite / op-sqlite, expo-notifications, expo-widgets, PowerSync RN) | Versions [Verified: npm registry + expo.dev/changelog/sdk-58] |
| **Local data** | SQLite through **Drift**, encrypted with **SQLite3 Multiple Ciphers** (`sqlite3` 3.x hook `source: sqlite3mc`) | [Verified: sqlite3.dart UPGRADING_TO_V3.md] |
| **Sync (phase 2+)** | **PowerSync** (self-hosted Open Edition or Cloud) to **Postgres** (Supabase, or self-hosted) | [Verified: powersync.com/pricing] |
| **Backend** | Phase 1: **none** (on-device only). Phase 2: **Supabase** (Auth + Postgres + Edge Functions) or a small self-hosted Go/Node API next to Postgres + PowerSync | [Speculative] |
| **AI** | On-device rules + merchant dictionary first; **Claude Haiku 5.5** through your own backend proxy for categorisation and natural-language entry ($0.10 / $0.50 per MTok); on-device LLMs (Apple Foundation Models, Gemini Nano) as an optional privacy tier | Prices [Verified: platform.claude.com pricing] |

**Why Flutter wins for this developer** [Speculative, on verified inputs]: it is the only option where one codebase gives Android + iOS + a usable web build *and* every hard requirement already has a maintained, recently published package (Drift 2.35.3, PowerSync 2.5.0, flutter_local_notifications 22.3.3, home_widget 0.10.0, ML Kit text recognition 0.17.1, fl_chart 1.3.0 — all published between Aug and Oct 2026) [Verified: pub.dev API], the toolchain is already installed, and Dart's typed, single-language model is the fastest path for a solo developer. React Native + Expo is the runner-up and would win if the web build mattered more than the mobile build.

**Where native code is unavoidable**: the Android notification listener needs a native Kotlin service in every cross-platform stack [Inferred: no framework abstracts it; Flutter plugin exists but is a thin bridge]. Home-screen widgets need native UI in Flutter — `home_widget` "does not allow writing Widgets with Flutter itself" [Verified: https://pub.dev/packages/home_widget] — and in KMP on iOS; **Expo SDK 58 is the exception**: `expo-widgets` runs widgets in JS on a dedicated Hermes runtime on iOS and (new) Android [Verified: https://expo.dev/changelog/sdk-58]. That is a real point for Expo, but it does not flip the pick: two small native widgets (Glance + WidgetKit) are ~1–2 weeks of work, while Flutter's advantages (toolchain already installed, Drift + PowerSync maturity, single-language Dart velocity) apply to the whole product [Speculative].

---

## 1. Requirements that drive the stack

| # | Requirement | Why it matters for an expense manager | Hardest part |
|---|---|---|---|
| R1 | iOS + Android from one codebase; web as a bonus | personal use on one phone now, commercial reach later | web parity |
| R2 | Offline-first | expenses are entered in shops, metro, abroad; no network = no lost entry | sync and conflict rules |
| R3 | Local reminders (bills, subscriptions, budget alerts) | the "reminder" pillar; must fire with the app closed | OS background limits, exact alarms |
| R4 | Home-screen widgets (month spend, quick add) | the fastest entry path is one tap from the home screen | native widget code |
| R5 | Android notification listener (parse bank/card push notifications) | automatic capture without open banking | OS redaction, Play review, user trust |
| R6 | On-device OCR for receipts | receipt photo → amount, date, merchant without uploading images | parsing quality, not OCR itself |
| R7 | Charts | tracking pillar | performance on large histories |
| R8 | AI categorisation + natural-language entry ("lunch 14.50 yesterday with Sam") | differentiator | privacy, cost, latency |
| R9 | Encryption at rest | financial data; GDPR expectation | key management |
| R10 | Path to multi-device + sharing (household budgets) | commercial feature | sync engine choice |

---

## 2. Current versions (checked 2026-11-01)

### 2.1 Frameworks

| Framework | Current stable | Release date | Grade |
|---|---|---|---|
| Flutter | **3.47.9** (Dart **3.13.7**) | 2026-10-29 | [Verified: https://storage.googleapis.com/flutter_infra_release/releases/releases_linux.json] |
| Flutter next | 3.50, scheduled November 2026 | — | [Inferred: search snippet citing the docs.flutter.dev archive schedule; the fetched page's tables did not render]; "current stable" may move within weeks of this document |
| Expo SDK | **58** (npm `expo` 58.0.22) | 2026-10-15 | [Verified: https://expo.dev/changelog + npm registry] |
| React Native | **0.88.1** (Expo SDK 58 moved RN 0.86 → 0.88) | — | [Verified: npm registry; https://expo.dev/changelog/sdk-58] |
| Kotlin | **2.4.21** | 2026-10-08 | [Verified: GitHub API JetBrains/kotlin releases/latest] |
| Compose Multiplatform | **1.12.2** (iOS stable since 1.8.0, May 2025) | 2026-10-15 | [Verified: GitHub API JetBrains/compose-multiplatform]; iOS-stable date [Inferred: search snippet citing JetBrains blog, May 2025] |
| Capacitor | **8.6.0** (`@capacitor/core`) | — | [Verified: npm registry] |
| Tauri | **2.12.4** (`@tauri-apps/cli`) | — | [Verified: npm registry] |

### 2.2 Flutter packages relevant to this app

| Package | Version | Published | Role | Grade |
|---|---|---|---|---|
| drift | 2.35.3 | 2026-10-23 | typed SQLite ORM, reactive streams, migrations; Android/iOS/macOS/Windows/Linux/web | [Verified: pub.dev API; https://pub.dev/packages/drift] |
| sqlite3 | 3.8.0 | 2026-10-23 | native SQLite bundled through build hooks; encryption via `source: sqlcipher` or `source: sqlite3mc` | [Verified: pub.dev API; sqlite3.dart UPGRADING_TO_V3.md] |
| sqlcipher_flutter_libs | 0.7.0**+eol** | 2026-02-15 | **END OF LIFE — do not use**; replaced by sqlite3 3.x hook config | [Verified: https://pub.dev/packages/sqlcipher_flutter_libs] |
| sqflite | 2.4.5 | 2026-10-21 | simpler SQLite plugin (no web, no codegen) | [Verified: pub.dev API] |
| powersync | 2.5.0 | 2026-10-19 | sync SDK | [Verified: pub.dev API] |
| flutter_local_notifications | 22.3.3 | 2026-10-25 | scheduled local notifications | [Verified: pub.dev API] |
| workmanager | 0.10.10 | 2026-09-07 | Android WorkManager / iOS BGTaskScheduler bridge | [Verified: pub.dev API] |
| background_fetch | 1.7.0 | 2026-05-25 | alternative periodic background runner | [Verified: pub.dev API] |
| home_widget | 0.10.0 | 2026-09-17 | data bridge to native widgets; renders Flutter widgets to images; interactive widgets calling Dart; Android + iOS | [Verified: https://pub.dev/packages/home_widget] |
| google_mlkit_text_recognition | 0.17.1 | 2026-08-17 | ML Kit OCR bridge | [Verified: pub.dev API] |
| notification_listener_service | 1.0.0 | 2026-06-13 | Android NotificationListenerService bridge | [Verified: pub.dev API] |
| fl_chart | 1.3.0 | 2026-10-21 | charts | [Verified: pub.dev API] |
| flutter_riverpod | 3.4.3 | 2026-09-03 | state management | [Verified: pub.dev API] |
| supabase_flutter | 2.18.3 | 2026-10-12 | Supabase client | [Verified: pub.dev API] |
| pocketbase | 0.25.3 | 2026-11-01 | PocketBase client | [Verified: pub.dev API] |
| appwrite | 28.2.0 | 2026-10-28 | Appwrite client | [Verified: pub.dev API] |
| firebase_core | 5.1.0 | 2026-10-23 | Firebase | [Verified: pub.dev API] |

None of the packages above is marked discontinued on pub.dev except `sqlcipher_flutter_libs` (EOL suffix) [Verified: pub.dev API `isDiscontinued` false for all; version string `0.7.0+eol`].

### 2.3 React Native / JS packages

| Package | Version | Published | Grade |
|---|---|---|---|
| expo-sqlite | 58.0.15 | 2026-10-31 | [Verified: npm registry] — note `useLibSQL` config property deprecated in SDK 58 [Verified: expo.dev/changelog/sdk-58] |
| @op-engineering/op-sqlite | 19.0.3 | 2026-10-29 | [Verified: npm registry] |
| expo-notifications | 58.1.7 | 2026-10-28 | [Verified: npm registry]; SDK 58 adds Android `delivery: 'alarmClock'` (uses `AlarmManager.setAlarmClock()`) [Verified: expo.dev/changelog/sdk-58] |
| expo-background-task / expo-task-manager | 58.0.12 / 58.0.13 | 2026-10-31 | [Verified: npm registry] |
| @powersync/react-native | 2.3.3 | 2026-10-19 | [Verified: npm registry] |
| victory-native / @shopify/react-native-skia | 43.0.1 / 2.14.0 | Oct 2026 | [Verified: npm registry] |
| react-native-mmkv | 4.3.2 | 2026-06-22 | [Verified: npm registry] |

---

## 3. Framework comparison

Scores are 1–5 for **this** app and **this** developer (solo, expert in bash/Docker/Linux, Flutter/Node/PHP/Android SDK already installed). [Speculative] unless a cell cites a source.

| Criterion | Flutter | React Native + Expo | Kotlin/Compose Multiplatform | Native Swift + Kotlin | PWA / web (+Capacitor/Tauri) |
|---|---|---|---|---|---|
| Solo-dev velocity | **5** — one language, hot reload, toolchain already installed | 4 — fast, but JS/TS dependency churn and native module upgrades each SDK | 3 — great on Android, iOS needs Xcode + Swift interop for platform APIs | 1 — two full apps | 4 for the UI, 2 once native features are needed |
| iOS + Android reach | 5 | 5 | 4 — iOS stable since CMP 1.8.0 [Inferred: search snippet citing JetBrains blog] | 5 | 3 (Capacitor wraps it) |
| Web reach | 3 — works (Drift supports web [Verified: pub.dev/packages/drift]), canvas-rendered, weak SEO, larger bundle [Unverified: prior knowledge] | **4** — react-native-web / Expo Router give a real DOM web app [Unverified: prior knowledge] | 2 — Compose for Web is newer [Unverified: prior knowledge] | 0 | **5** |
| Home-screen widgets | 3 — native widget UI required; home_widget bridges data, renders Flutter→image, interactive callbacks [Verified: pub.dev/packages/home_widget] | 4 — `expo-widgets` now on iOS (stable since SDK 56) **and Android** (SDK 58), JS on a dedicated Hermes runtime [Verified: expo.dev/sdk/56; expo.dev/changelog/sdk-58] | 4 — Glance on Android natively; iOS still SwiftUI [Unverified: prior knowledge] | 5 | 0–1 (no widgets for PWA) [Unverified: prior knowledge] |
| Background tasks + local notifications | 4 — flutter_local_notifications + workmanager, mature [Verified versions: pub.dev] | 4 — expo-notifications (alarmClock delivery), expo-background-task [Verified: npm + SDK 58 changelog] | 4 | 5 | 1–2 — web push needs a server; no reliable local scheduling on iOS PWA [Unverified: prior knowledge] |
| Android notification listener | 3 — plugin exists (`notification_listener_service` 1.0.0) but you will own a Kotlin service in practice | 3 — needs a custom native module | **5** — it is just Android code | 5 | 0 |
| On-device OCR | 4 — google_mlkit_text_recognition 0.17.1 | 4 — community ML Kit / Vision modules [Unverified: prior knowledge] | 4 | 5 (ML Kit + Apple Vision directly) | 2 (Tesseract.js / WASM) |
| Charts | 4 — fl_chart 1.3.0 | 4 — victory-native on Skia | 3 | 4 (Swift Charts / Vico) | 5 (any JS lib) |
| Offline-first data | **5** — Drift (reactive SQL) + PowerSync Flutter SDK | 5 — op-sqlite / expo-sqlite + PowerSync RN | 4 — SQLDelight / Room KMP [Unverified: prior knowledge] | 4 | 3 — OPFS SQLite WASM works but storage can be evicted [Unverified: prior knowledge] |
| Maturity in 2026 | 5 — 3.47.x patch train weekly (3.47.6 → 3.47.9 in October) [Verified: Flutter release feed] | 5 — SDK cadence ~2 per half-year (56 May, 57 June, 58 Oct) [Verified: expo.dev/changelog, expo.dev/sdk/56] | 4 | 5 | 5 web / 3 Tauri-mobile |
| **Weighted fit** | **~4.3** | ~4.2 | ~3.6 | ~3.4 (velocity kills it) | ~2.5 |

### 3.1 Notes per option

**Flutter** — Strengths: single Dart codebase, very strong offline data story (Drift + PowerSync both first-party-quality and updated in Oct 2026 [Verified: pub.dev API]), the developer already has the toolchain. Weaknesses: web is a canvas app, fine for a logged-in dashboard, poor for a marketing site (use a separate static site) [Unverified: prior knowledge]; native widget/listener code in Kotlin + Swift is unavoidable. Watch: Flutter 3.50 is reportedly due in November 2026 [Inferred: search snippet of docs.flutter.dev/release/archive]; pin the SDK with FVM.

**React Native + Expo** — Strengths: best web story among the mobile frameworks, EAS cloud builds (no Mac needed for iOS builds) [Unverified: prior knowledge], widgets now in JS on both platforms [Verified: expo.dev/changelog/sdk-58]. Weaknesses: SDK upgrades every few months carry breaking changes (e.g. SDK 58 changed foreground notification default behaviour and deprecated several APIs) [Verified: expo.dev/changelog/sdk-58]; native module work for the notification listener.

**Kotlin / Compose Multiplatform** — Best if Android were the only serious target: the notification listener, Glance widgets and WorkManager are native. iOS widgets, Vision OCR and notifications still need Swift. Web is the weak link [Unverified: prior knowledge].

**Native Swift + Kotlin** — Best quality ceiling, worst velocity: two codebases for a solo developer. Not recommended before product–market fit [Speculative].

**PWA / Capacitor / Tauri** — A PWA cannot provide widgets or a notification listener, and local scheduled notifications are unreliable on iOS [Unverified: prior knowledge]. Capacitor 8.6.0 [Verified: npm] can wrap a web app with native plugins, but every hard requirement becomes a plugin you write. Tauri 2 mobile works, but its own team said mobile was "overpromised" and not all plugins were ported [Inferred: search snippet of v2.tauri.app/blog/tauri-2-0-0-release-candidate]. Use the web only as a **companion** (dashboard, export, admin), not as the primary app.

---

## 4. Platform constraints that affect the design

### 4.1 Android notification listener (automatic capture)

- Access is granted by the user in Settings → Notification access; no runtime dialog; the service declares `BIND_NOTIFICATION_LISTENER_SERVICE` [Unverified: prior knowledge; the reference page was fetched but its class description was not reached].
- **Android 15 OTP redaction**: "Android will stop untrusted apps that implement a `NotificationListenerService` from reading unredacted content from notifications where an OTP has been detected. Trusted apps such as companion device manager associations are exempt." [Verified: https://developer.android.com/about/versions/15/behavior-changes-all]. Card-payment notifications normally carry an amount, not an OTP, so this mostly affects 3-D Secure codes — but OEM classifier false positives could redact a payment notification [Inferred: from the quoted rule; untested].
- The full sensitive-content permission (`RECEIVE_SENSITIVE_NOTIFICATIONS`) is signature|role-level, not obtainable by an ordinary app [Inferred: search summary of AOSP commits].
- **Restricted settings** (Android 13+): sideloaded apps cannot enable notification access until the user taps "Allow restricted settings" in App info [Unverified: prior knowledge]. Matters for F-Droid/APK distribution.
- **Google Play**: no primary source found in this session on a Play declaration specific to `NotificationListenerService`; the general **Financial features declaration** applies to every app on Play [Inferred: search snippet of support.google.com/googleplay/android-developer/answer/13849271]. **Check Play Console policy for notification access before the first submission.**
- **iOS has no equivalent** — no third-party app can read other apps' notifications [Unverified: prior knowledge]. iOS auto-capture paths are: Shortcuts automations on Wallet transactions, App Intents, share sheets, and open banking [Unverified: prior knowledge].

### 4.2 Reminders and background work

- Local scheduled notifications do not need a server and survive the app being killed; on Android, exact alarms need the alarm permission and `setAlarmClock()` is the most reliable path (Expo SDK 58 exposes exactly this) [Verified for Expo: expo.dev/changelog/sdk-58; Android permission detail Unverified: prior knowledge].
- Schedule reminders **from the local database** (recompute the next N reminders on every app open, sync, and boot) instead of relying on periodic background tasks, which iOS throttles heavily [Unverified: prior knowledge on iOS BGTaskScheduler throttling; design is Speculative].

### 4.3 On-device OCR

- **Google ML Kit Text Recognition v2**: supports Latin, Chinese, Devanagari, Japanese, Korean; Android and iOS guides exist; no deprecation notice on the page [Verified: https://developers.google.com/ml-kit/vision/text-recognition/v2]. On-device processing and free-of-charge use were **not stated on that page** [Unverified: prior knowledge says ML Kit text recognition runs on-device at no cost].
- **Apple Vision** (`VNRecognizeTextRequest` and newer document APIs) is the native iOS alternative [Unverified: prior knowledge].
- OCR is the easy half; **receipt parsing** (total vs subtotal vs VAT lines, FR/EU number formats `14,50 €`) is the hard half. Plan: OCR on-device → regex/heuristics for amount/date → optional LLM pass for merchant + category on the extracted *text* only (never the image) [Speculative].

### 4.4 On-device LLMs

- **Apple Foundation Models framework** (iOS/iPadOS/macOS 26): ~3B-parameter on-device model, guided generation into Swift types, tool calling, offline, no inference cost; requires an Apple Intelligence-capable device with it enabled [Inferred: search snippets citing MacRumors / 9to5Mac, not Apple docs].
- **ML Kit GenAI / Prompt API (Gemini Nano via AICore)**: limited device list (Pixel 9/10, Galaxy S26, some Xiaomi/OPPO/vivo/OnePlus/Honor); inference only while the app is the top foreground app; `ErrorCode.BUSY` on bursts [Inferred: search snippet of https://developers.google.com/ml-kit/genai].
- Consequence: on-device LLMs are an **optional enhancement tier** for supported devices, not the baseline — most users will not have them, and Gemini Nano cannot run from a background notification-listener path [Inferred: foreground-only rule above].

---

## 5. Data layer

### 5.1 Local-first options

| Option | Status (2026-11) | Fit | Grade |
|---|---|---|---|
| **Drift** (Flutter) | 2.35.3, Oct 2026; reactive streams, migrations, all platforms incl. web | **Primary** — relational, typed, testable | [Verified: pub.dev] |
| sqflite | 2.4.5, Oct 2026 | simpler, no web, no codegen | [Verified: pub.dev] |
| **SQLite encryption** | `sqlite3` 3.x: hook `source: sqlcipher` or `source: sqlite3mc` (SQLite3MC also works on web); `sqlcipher_flutter_libs` "no longer functional" | use **sqlite3mc**; key in Keychain/Keystore via flutter_secure_storage [key storage: Unverified: prior knowledge] | [Verified: sqlite3.dart UPGRADING_TO_V3.md] |
| **PowerSync** | Flutter 2.5.0, RN 2.3.3 (Oct 2026); Postgres/MongoDB/MySQL/SQL Server backends; self-hostable Open Edition | **Sync layer of choice** | [Verified: pub.dev, npm, powersync.com/pricing] |
| ElectricSQL | `@electric-sql/client` 1.5.28 (Sep 2026); read-path "shape" sync from Postgres; JS-first | the Dart `electricsql` package is **0.8.2, last published 2024-07-24** (pre-rewrite, stale); writes go through your API | [Verified: npm + pub.dev API] |
| Rocicorp Zero | `@rocicorp/zero` 1.11.3 (Oct 2026); 1.0 shipped 2026 per InfoQ; TypeScript, Postgres → SQLite replica in zero-cache | good for an RN/web stack, not Flutter | version [Verified: npm]; 1.0 [Inferred: search snippet of infoq.com/news/2026/06/zero-version-1] |
| Replicache | superseded in practice by Zero (same vendor) | avoid for new work | [Unverified: prior knowledge] |
| InstantDB | `@instantdb/react-native` 1.0.67 (Aug 2026); hosted triple store | JS only, vendor lock-in | [Verified: npm] |
| **Triplit** | last npm publish **2025-07-31**; team acqui-hired by Supabase Aug 2025, now community-maintained | **avoid** — stale | publish date [Verified: npm]; acquisition [Inferred: search snippet] |
| Automerge / Yjs | 3.5.0 / 13.6.33 (Sep 2026) | CRDT documents; overkill for ledger rows, useful for shared notes | [Verified: npm] |
| **cr-sqlite** | last release **v0.16.3, 2024-01-17** (repo pushed Aug 2026, not archived) | **avoid** — no release in ~2 years | [Verified: GitHub API] |

**Why not CRDTs for the ledger** [Speculative]: expenses are append-mostly rows with a clear owner; last-writer-wins per field plus server-side validation (PowerSync's model: local writes go to an upload queue, your backend applies them) is simpler and auditable. A financial ledger wants the server to be the arbiter, not a merge function.

### 5.2 Sync service pricing

| Service | Free | Paid entry | Grade |
|---|---|---|---|
| **PowerSync Cloud** | $0: 2 GB synced/mo, 500 MB hosted, 50 peak concurrent clients; deactivated after inactivity (page says both 1 week and 2 weeks) | **Pro from $49/mo**: 30 GB synced, 10 GB hosted, 1,000 concurrent clients, then $1/GB and $30 per 1,000 clients; Team from $599/mo | [Verified: https://www.powersync.com/pricing] |
| **PowerSync Open Edition (self-hosted)** | free, source-available; all client SDKs, Sync Streams partial sync | Enterprise self-hosted custom | [Verified: https://www.powersync.com/pricing] |

Self-hosting the Open Edition in Docker next to Postgres fits this developer's skill set and costs only the VPS [Inferred: Open Edition availability + developer profile]. "Source-available" is not OSI open source — read the licence before commercial use [Unverified: licence text not opened].

### 5.3 Backend options

| Backend | Status / price | Fit | Grade |
|---|---|---|---|
| **Supabase** | Free: 500 MB DB, 50k MAU, 5 GB egress, **paused after 1 week inactivity**, 2 active projects. **Pro from $25/mo**: 8 GB disk/project, 100k MAU, 250 GB egress, $10 compute credit, never paused | Postgres + Auth + Storage + Edge Functions; first-class PowerSync pairing; self-hostable | [Verified: https://supabase.com/pricing]; EU regions not stated on the page [Unverified: prior knowledge says EU regions exist] |
| PocketBase | **v0.40.6** (2026-10-23) — still **pre-1.0** | single Go binary, SQLite; great for personal phase; no Postgres → no PowerSync | [Verified: GitHub API] |
| Appwrite | Flutter SDK 28.2.0 (Oct 2026) | BaaS with Flutter SDK; no native PowerSync source | [Verified: pub.dev]; server version tag seen as 2.6.2 on GitHub — not interpreted [Inferred] |
| Firebase | firebase_core 5.1.0 | Firestore offline cache is document-based, not relational; analytics on spending is awkward; US-company data residency questions | version [Verified: pub.dev]; fit [Unverified: prior knowledge] |
| **Custom Go / Node / PHP API** | — | Thin API for: LLM proxy, open-banking webhooks, PowerSync upload handler, billing | [Speculative] |

**Recommendation** [Speculative]: Postgres is the system of record (Supabase-hosted for speed, or self-hosted on an EU VPS for sovereignty). A small custom service (Go or Node) holds the secrets: LLM API key, open-banking aggregator credentials, PowerSync write handler. Both Supabase and PowerSync can be self-hosted, so there is no one-way door [Verified: Open Edition on powersync.com/pricing; Supabase self-hosting Unverified: prior knowledge].

---

## 6. AI: categorisation and natural-language entry

### 6.1 Claude API prices (per million tokens)

| Model | Input | Output | Cache hit | Batch (−50%) in/out | Grade |
|---|---|---|---|---|---|
| **Claude Haiku 5.5** (prompt ≤ 100k tokens) | **$0.10** | **$0.50** | $0.01 | $0.05 / $0.25 | [Verified: https://platform.claude.com/docs/en/about-claude/pricing] |
| Claude Haiku 5.5 (prompt > 100k) | $0.50 | $2.50 | $0.05 | $0.25 / $1.25 | [Verified: same] |
| **Claude Sonnet 5.5** | **$2** | **$10** | $0.10 | $1 / $5 | [Verified: same] |
| **Claude Opus 5.5** | **$4** | **$20** | $0.20 | $2 / $10 | [Verified: same] |

US-only inference (`inference_geo: "us"`) costs 1.1×; global routing is standard price [Verified: pricing + data-residency pages].

### 6.2 Cost model for this app

Assume one categorisation call ≈ 400 input tokens (system prompt with category list + merchant string + few-shot examples) and 50 output tokens (JSON) [Speculative assumptions].

- Haiku 5.5: 400 × $0.10/1M + 50 × $0.50/1M = **$0.000065 per transaction**. A heavy user with 300 transactions/month ≈ **$0.02/month**; with prompt caching on the static prefix and the Batch API for overnight re-categorisation, lower still [Inferred: arithmetic on verified prices].
- Sonnet 5.5 on the same call: $0.0013 per transaction (20× Haiku) — reserve it for monthly "insights" summaries, not per-transaction work [Inferred: arithmetic].
- Natural-language entry ("taxi 23€ yesterday, split with Léa") ≈ similar size → same order of cost [Speculative].

Conclusion: **LLM cost is negligible; privacy and latency are the real constraints** [Inferred].

### 6.3 Privacy and residency findings (load-bearing for an EU product)

- Default API retention: "For Anthropic API users, we automatically delete inputs and outputs on our backend within 30 days of receipt or generation" [Verified: https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data].
- "Retained data is never used for model training without your express permission" [Verified: https://platform.claude.com/docs/en/manage-claude/api-and-data-retention].
- Zero data retention is available by arrangement, but **not** for the Covered Models (Fable 5 / 5.1, Mythos 5 / 5.1); Haiku/Sonnet/Opus 5.5 are not in the Covered list [Verified: same page].
- **No EU inference or storage option on the first-party API**: "Only `us` and `global` are available" for inference geo; "Only `us` is currently available" for workspace geo [Verified: https://platform.claude.com/docs/en/manage-claude/data-residency]. An EU product sending personal financial data must therefore treat this as a **third-country transfer** under GDPR (DPA + SCCs, disclosed in the privacy policy) [Unverified: legal interpretation, not legal advice]. Partner platforms (Bedrock, Google Cloud Google-managed) set region by endpoint and may offer EU regions [Inferred: data-residency page says region is determined by endpoint there; EU availability not checked].

### 6.4 Design that follows from these findings [Speculative]

1. **Tier 0 — deterministic, on-device (default, always on)**: merchant normalisation + user's own past choices + a rule table ("SNCF" → Transport). Covers most repeat transactions after a few weeks of use.
2. **Tier 1 — on-device LLM where available** (Apple Foundation Models / Gemini Nano): opt-in, nothing leaves the device.
3. **Tier 2 — cloud LLM (Claude Haiku 5.5) via your backend proxy**, opt-in, sending **only** the minimal string (merchant text, amount bucket, locale) — never account numbers, IBANs, names or receipt images. The API key never ships in the app; the proxy rate-limits per user.
4. Every AI result is a **suggestion with confidence**; the user's correction feeds Tier 0 (learn locally).

---

## 7. Recommended stack

### 7.1 Primary: Flutter + Drift + PowerSync/Postgres + Claude Haiku via proxy

| Layer | Choice | Version (2026-11-01) |
|---|---|---|
| UI | Flutter, Material 3 + Cupertino adaptive widgets | 3.47.9 / Dart 3.13.7 [Verified] |
| State | Riverpod | flutter_riverpod 3.4.3 [Verified] |
| Local DB | Drift on `sqlite3` 3.x with `source: sqlite3mc` encryption | drift 2.35.3, sqlite3 3.8.0 [Verified] |
| Notifications | flutter_local_notifications | 22.3.3 [Verified] |
| Background | workmanager (Android periodic jobs, iOS best-effort) | 0.10.10 [Verified] |
| Widgets | home_widget + native Glance (Android) / WidgetKit SwiftUI (iOS) | 0.10.0 [Verified] |
| Auto-capture (Android) | own Kotlin NotificationListenerService via platform channel (or notification_listener_service 1.0.0) | [Verified version] |
| OCR | google_mlkit_text_recognition (Android/iOS) | 0.17.1 [Verified] |
| Charts | fl_chart | 1.3.0 [Verified] |
| Sync (phase 2) | PowerSync Flutter SDK → PowerSync service (self-hosted Open Edition or Cloud) → Postgres | powersync 2.5.0 [Verified] |
| Backend (phase 2) | Supabase (Auth, Postgres, Storage) + small Go/Node service for LLM proxy and upload handler | supabase_flutter 2.18.3 [Verified] |
| AI | Claude Haiku 5.5 (per-transaction), Sonnet 5.5 (monthly insights), Batch API for bulk | prices [Verified] |
| CI/CD | Docker-based Linux CI for Android + tests; macOS runner (or a cloud Mac service) only for iOS builds/signing | [Speculative] |

**Integration caveat to verify in a spike**: PowerSync's Flutter SDK ships its own SQLite layer; confirm (a) Drift interop — a `drift_sqlite_async` package exists (0.3.1, published 2026-06-04) [Verified: pub.dev API], which per prior knowledge is the PowerSync↔Drift bridge [Unverified: package page not read], and (b) that encryption with SQLite3MC works together with PowerSync before committing. This is the single highest technical risk in the stack — spike it in week 1.

### 7.2 Runner-up: React Native 0.88 + Expo SDK 58

Choose it if **web becomes a first-class surface** (the same React code serving a real DOM dashboard) or if a team of JS developers joins. Stack: Expo Router, op-sqlite or expo-sqlite, PowerSync RN 2.3.3, expo-notifications (`alarmClock` delivery), expo-widgets (iOS + Android), victory-native/Skia, EAS Build [versions Verified: npm; features Verified: expo.dev/changelog/sdk-58]. Cost: faster SDK churn and more JS dependency maintenance [Speculative].

### 7.3 Rejected (for now)

- **Native ×2** — velocity [Speculative].
- **KMP/CMP** — strong Android, but iOS platform features still need Swift and web is weak; revisit if the app becomes Android-first [Speculative].
- **PWA-only / Tauri mobile** — no widgets, no notification listener, weak iOS local notifications [Unverified: prior knowledge; Tauri mobile maturity Inferred: Tauri RC blog snippet].
- **Triplit, cr-sqlite** — stale releases [Verified: npm / GitHub API dates].
- **sqlcipher_flutter_libs** — EOL [Verified: pub.dev].

---

## 8. Reference architecture

### 8.1 Components

```
┌──────────────────────────── Mobile app (Flutter) ────────────────────────────┐
│ UI (screens, quick-add, charts)  ←→  Riverpod providers                       │
│        │                                                                       │
│ Domain: Ledger, Budgets, Recurring/Reminders, Categoriser (Tier 0 rules)       │
│        │                                                                       │
│ Drift repository  ──►  SQLite (SQLite3MC-encrypted; key in Keychain/Keystore) │
│        │                         ▲            ▲             ▲                  │
│ Reminder scheduler ─► local notifications     │             │                  │
│ Capture adapters:  Android NotificationListener (Kotlin) ── parse ─┘          │
│                    Receipt camera → ML Kit OCR → parser ───────────┘          │
│                    Natural-language box → (on-device LLM | proxy)              │
│ Widget bridge (home_widget) → Glance / WidgetKit (read-only snapshot + quick-add)
│ PowerSync client (phase 2): local writes → upload queue                        │
└────────────────────────────────────┬──────────────────────────────────────────┘
                                     │ HTTPS (JWT)
┌────────────────────────────────────▼──────────────────── Backend (phase 2+) ──┐
│ PowerSync service (sync streams: per-user + per-household buckets)            │
│ API service (Go/Node): upload handler (validate → write Postgres),             │
│                        LLM proxy (minimise → Claude Haiku 5.5, rate limit),    │
│                        open-banking connector (phase 3, webhooks), billing      │
│ Postgres (Supabase or self-hosted, EU region) — system of record, RLS          │
│ Object storage (optional, encrypted receipt images — opt-in only)              │
└───────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Data flows

1. **Manual / NL entry**: UI → Drift insert (instant, offline) → Tier 0 categoriser → optional LLM suggestion (async; updates the row when it returns) → reminders/budgets recomputed → widget snapshot refreshed.
2. **Android notification capture**: NotificationListener (Kotlin) → allow-listed package names only → regex parser per bank template → *pending* transaction in Drift → user confirms in one tap (never auto-commit money data silently).
3. **Receipt**: camera → ML Kit OCR on device → amount/date heuristics → optional LLM on the **text** → pending transaction; image kept locally unless the user opts into cloud backup.
4. **Reminders**: on every app start, sync, boot and data change, recompute the next N occurrences of each recurring item and (re)schedule local notifications; no server needed.
5. **Sync (phase 2)**: Drift/PowerSync local write → upload queue → API upload handler validates and writes Postgres → PowerSync streams the change to the user's other devices / household members.
6. **Open banking (phase 3)**: aggregator webhook → API → Postgres → PowerSync → device; categorised by the same pipeline. (Aggregator choice is covered in the bank-sync research document.)

### 8.3 Non-functional choices [Speculative]

- Money as **integer minor units + ISO 4217 currency code**, never floats; FX rates stored per transaction date.
- IDs are client-generated UUIDv7 (sortable, offline-safe).
- Every table has `updated_at`, `deleted_at` (soft delete) for sync.
- Export (CSV/JSON) from day 1 — data portability is both a GDPR right and a trust feature.

---

## 9. Deprecation check of everything recommended

| Item | Status | Grade |
|---|---|---|
| Flutter 3.47 | current stable [Verified: release feed]; 3.50 reportedly due Nov 2026 [Inferred: search snippet] — not a deprecation, plan an upgrade | mixed, see cells |
| Drift, sqlite3 3.x, powersync, flutter_local_notifications, workmanager, home_widget, google_mlkit_text_recognition, fl_chart, flutter_riverpod, supabase_flutter | all published Aug–Oct 2026, none discontinued | [Verified: pub.dev API] |
| sqlcipher_flutter_libs / sqlite3_flutter_libs | **obsolete with sqlite3 3.x — remove** | [Verified: sqlite3.dart UPGRADING_TO_V3.md] |
| ML Kit Text Recognition v2 | no deprecation notice (page last updated 2024-07-10) | [Verified] |
| Expo SDK 58 deprecations (runner-up) | `useLibSQL` (expo-sqlite), `File.md5`, `withLayoutContext`, RN `ImageBackground`, several `@expo/ui` props | [Verified: expo.dev/changelog/sdk-58] |
| Claude Haiku/Sonnet/Opus 5.5 | current; older Sonnet 4.5 marked deprecated, Opus 4 / Sonnet 4 / Haiku 3.5 retired on first-party | [Verified: pricing page] |
| Triplit, cr-sqlite | stale — avoid | [Verified: dates] |
| PocketBase | pre-1.0 (v0.40.6) — breaking changes possible | [Verified: GitHub API] |

---

## 10. Effort indicators for the stack (feeds the roadmap) [Speculative]

| Work item | Rough effort (solo, expert) |
|---|---|
| Week-1 spike: Drift + SQLite3MC + PowerSync interop on Android + iOS | 3–5 days |
| Core ledger, categories, budgets, charts (offline only) | 3–4 weeks |
| Reminders + recurring engine + local notifications | 1–2 weeks |
| Home widgets (Android Glance + iOS WidgetKit) | 1–2 weeks |
| Android notification listener + 3–5 bank parsers | 1–2 weeks |
| Receipt OCR + parser (FR/EU formats) | 1–2 weeks |
| LLM proxy + NL entry + categorisation tiers | 1–2 weeks |
| Sync + auth + households (PowerSync + Supabase) | 3–4 weeks |
| iOS build/signing pipeline, store submissions | 1 week + review time |

---

## Sources

Opened in this session (Verified):

- Flutter release feed: https://storage.googleapis.com/flutter_infra_release/releases/releases_linux.json
- Flutter SDK archive / schedule: https://docs.flutter.dev/release/archive
- Expo SDK 56: https://expo.dev/sdk/56
- Expo changelog: https://expo.dev/changelog
- Expo SDK 58: https://expo.dev/changelog/sdk-58
- npm registry (expo, react-native, @capacitor/core, @tauri-apps/cli, @powersync/react-native, @electric-sql/client, @rocicorp/zero, @instantdb/react-native, @triplit/client, @automerge/automerge, yjs, expo-sqlite, expo-notifications, expo-background-task, expo-task-manager, @op-engineering/op-sqlite, react-native-mmkv, victory-native, @shopify/react-native-skia): https://registry.npmjs.org/
- pub.dev API (drift, sqflite, sqlite3, sqlcipher_flutter_libs, powersync, home_widget, flutter_local_notifications, workmanager, background_fetch, google_mlkit_text_recognition, notification_listener_service, fl_chart, flutter_riverpod, supabase_flutter, pocketbase, appwrite, firebase_core): https://pub.dev/api/packages/
- https://pub.dev/packages/home_widget
- https://pub.dev/packages/drift
- https://pub.dev/packages/sqlcipher_flutter_libs
- https://github.com/simolus3/sqlite3.dart/blob/main/UPGRADING_TO_V3.md
- GitHub API releases: JetBrains/kotlin, JetBrains/compose-multiplatform, pocketbase/pocketbase, appwrite/appwrite, vlcn-io/cr-sqlite — https://api.github.com/repos/
- PowerSync pricing: https://www.powersync.com/pricing
- Supabase pricing: https://supabase.com/pricing
- Claude pricing: https://platform.claude.com/docs/en/about-claude/pricing
- Claude API and data retention: https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
- Claude data residency: https://platform.claude.com/docs/en/manage-claude/data-residency
- Commercial retention article: https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data
- Android 15 behaviour changes: https://developer.android.com/about/versions/15/behavior-changes-all
- NotificationListenerService reference (partial read): https://developer.android.com/reference/android/service/notification/NotificationListenerService
- ML Kit Text Recognition v2: https://developers.google.com/ml-kit/vision/text-recognition/v2

Seen only as search-result summaries (Inferred):

- Compose Multiplatform 1.8.0 iOS stable: https://blog.jetbrains.com/kotlin/2025/05/compose-multiplatform-1-8-0-released-compose-multiplatform-for-ios-is-stable-and-production-ready/
- Zero 1.0: https://infoq.com/news/2026/06/zero-version-1
- Triplit acqui-hire: https://johnny-sh.pages.dev/blog/choosing-a-sync-engine-in-2026
- Tauri 2.0 RC (mobile caveats): https://v2.tauri.app/blog/tauri-2-0-0-release-candidate/
- Apple Foundation Models: https://www.macrumors.com/2025/06/09/foundation-models-framework/ , https://9to5mac.com/2025/09/29/apple-intelligence-ios-26-models-developers/
- ML Kit GenAI / Prompt API: https://developers.google.com/ml-kit/genai , https://developer.android.com/blog/posts/ml-kit-s-prompt-api-unlock-custom-on-device-gemini-nano-experiences
- AOSP sensitive-notification permission: https://gerrit.omnirom.org/plugins/gitiles/android_frameworks_base/+/9752502c0e863c613e22690a3969ede161f7814e%5E%21
- Google Play Financial features declaration: https://support.google.com/googleplay/android-developer/answer/13849271
