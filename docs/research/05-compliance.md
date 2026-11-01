# 05 — Security, Privacy, Compliance & Store Policy

> Research dimension 5 of the expense-manager feasibility study. Researched 2026-11-01.
> Every claim carries a grade: **[Verified: url]** = the source was opened in this session;
> **[Inferred: basis]** = deduced from verified material or from secondary sources read only as search summaries;
> **[Unverified: prior knowledge]** = recalled, not checked this session. This is engineering research, **not legal advice**.

---

## 0. TL;DR — what changes the plan

| # | Finding | Impact | Grade |
|---|---|---|---|
| 1 | Apple 5.1.1(ix): apps in "banking and financial services" "should be submitted by a legal entity that provides the services, and not by an individual developer"; 3.2.1(viii): apps for "money management should be submitted by the financial institution performing such services". | **Biggest store risk for a solo dev on an individual Apple account.** Wording targets regulated services, and many indie budgeting apps exist, but a reviewer can invoke it. Position the app as a *personal expense tracker / organiser* (no advice, no payments, no account aggregation on iOS v1) and plan an **organization account (D-U-N-S)** before commercialisation. | [Verified: https://developer.apple.com/app-store/review/guidelines/] for the text; interpretation [Inferred: guideline wording + existence of indie budgeting apps such as those found below] |
| 2 | Google Play: SMS reading for expense tracking is allowed **only** via the "SMS-based money management" exception (READ_SMS/RECEIVE_SMS…), which needs a Permissions Declaration + review, and exceptions are framed as temporary and only when "no alternative method" exists. | SMS parsing is a **gated, revocable** feature — never the core of v1. | [Verified: https://support.google.com/googleplay/android-developer/answer/10208820] |
| 3 | Notification-listener parsing (Android) has **no dedicated Play declaration** found in this session; the user must grant it manually in system settings. Android 15 hides *2FA/OTP* notification content from untrusted listeners. | Notification listener is the most realistic "auto-capture" path on Android; bank *transaction* notifications are likely unaffected by the OTP redaction. | No-declaration: [Verified: absence across two targeted searches this session]; OTP redaction scope: [Inferred: Android Authority reports, read only as search summaries]; transaction notifs unaffected: [Inferred] |
| 4 | iOS: no third-party app can read other apps' notifications. The workable auto-capture is the **Shortcuts "Transaction" automation** (iOS 17+, Apple Pay/Wallet taps) feeding an App Intent. **FinanceKit** needs an **organization** account + managed entitlement and covers **US and UK only** — not France/EU. | iOS auto-capture = Apple Pay taps via Shortcuts only; plan an `AddExpense` App Intent. | Shortcuts trigger: [Inferred: multiple vendor guides + Apple Developer Forums threads, search summaries]; FinanceKit US/UK-only + entitlement: [Verified: https://developer.apple.com/financekit]; organization-account requirement: [Inferred: Apple docs as summarised by search] |
| 5 | GoCardless Bank Account Data (ex-Nordigen, the classic free EU AIS API): "New signups for Bank Account Data are currently disabled." | The cheap open-banking path most indie EU finance apps used is **closed**. Bank sync = paid aggregator (Tink, Salt Edge, Powens, Enable Banking…) or postpone. | [Verified: https://bankaccountdata.gocardless.com/new-signups-disabled] |
| 6 | A personal expense tracker is **not on the CNIL's mandatory-DPIA list** (14 types), but financial data is generally treated as "highly personal"; doing a lightweight DPIA anyway is cheap and recommended. | Write a 3–5 page DPIA before any cloud sync / commercial launch. | List: [Verified: https://www.cnil.fr/sites/default/files/atoms/files/liste-traitements-avec-aipd-requise-v2.pdf]; financial data as highly personal: [Inferred: Fieldfisher national-lists compilation, search summary] |
| 7 | **Local-first + E2EE sync** removes most GDPR burden (the operator never sees plaintext) and is the strongest product differentiator vs. aggregator apps. | Architecture recommendation. | [Inferred: GDPR art. 25/32 logic — Unverified article text this session] |

---

## 1. Threat model (STRIDE-lite, for a local-first expense app)

| Asset | Threat | Likely actor | Mitigation | Grade |
|---|---|---|---|---|
| Transaction DB on device | Device theft / unlocked phone snooping | Opportunistic thief, partner/family | Encrypted DB (SQLite3MultipleCiphers / SQLCipher-compatible) with key in Android Keystore / iOS Keychain; app lock (biometric + PIN fallback); auto-lock on background; screenshot blocking on sensitive screens (`FLAG_SECURE`) | [Unverified: prior knowledge] |
| Cloud backup / sync blob | Server breach, rogue operator, subpoena | Attacker, insider | **E2EE**: encrypt client-side (XChaCha20-Poly1305 / AES-GCM), key derived from passphrase (Argon2id) or device-held key; server stores ciphertext only | [Unverified: prior knowledge] |
| Bank aggregator tokens | Token theft → full read access to accounts | Attacker | Keep tokens server-side (aggregator requires a backend), short-lived, encrypted at rest with KMS; never ship aggregator secret in the app | [Unverified: prior knowledge] |
| SMS / notification text | Over-collection (OTP, personal SMS) | The app itself (compliance risk) | Parse on-device only, whitelist sender IDs/package names, discard non-matching text immediately, never upload raw text (Play Spyware policy explicitly bans budgeting apps exfiltrating non-financial SMS) | [Verified: https://support.google.com/googleplay/android-developer/answer/10208820] |
| Receipt photos / OCR | Leakage of addresses, card digits | Third-party OCR processor | On-device OCR (ML Kit / Apple Vision) by default; cloud LLM extraction opt-in and disclosed as processor | [Unverified: prior knowledge] |
| Reminders / push | Lock-screen exposure of amounts | Bystander | Private notification content by default (`VISIBILITY_PRIVATE`), user opt-in for amounts | [Unverified: prior knowledge] |
| App binary | Reverse engineering, repackaging with malware | Fraudster | R8/obfuscation, Play Integrity / App Attest for the sync API, certificate pinning optional | [Unverified: prior knowledge] |
| Account | Credential stuffing | Attacker | Passkeys / magic link, rate limiting, no password storage where possible | [Unverified: prior knowledge] |

**Baseline standard:** use **OWASP MASVS** as the checklist — control groups STORAGE, CRYPTO, AUTH, NETWORK, PLATFORM, CODE, RESILIENCE, PRIVACY. [Verified: https://mas.owasp.org/MASVS/] Target MASVS L1 + the PRIVACY group for v1; L2 when bank sync ships. [Inferred: L2 is meant for apps handling sensitive financial data — Unverified wording this session]

---

## 2. Encryption at rest, E2EE and app lock

### 2.1 Encrypted local DB (Flutter stack)

| Component | Status (2026-11-01) | Grade |
|---|---|---|
| `sqlcipher_flutter_libs` | **Discontinued** — "0.7.0+eol … no longer does anything"; replacement: `package:sqlite3` **3.x** | [Verified: https://pub.dev/packages/sqlcipher_flutter_libs] |
| `package:sqlite3` 3.x encryption | 3.8.0 (current): "On all native platforms, you can also use SQLite3MultipleCiphers or SQLCipher instead of SQLite to encrypt databases"; selected via build-hook options; web uses a separate `sqlite3mc.wasm` + `pragma key` | [Verified: https://pub.dev/packages/sqlite3] |
| `drift` ORM on top of sqlite3 | De-facto Flutter SQL ORM, works with encrypted sqlite | [Unverified: prior knowledge] |
| `flutter_secure_storage` | Stores the DB key in Keychain / Keystore-backed storage | [Unverified: prior knowledge] |

**Rule 9 (deprecation) note:** do **not** start with `sqlcipher_flutter_libs`; any tutorial using it is stale. [Verified: pub.dev EOL notice above]

### 2.2 Biometric app lock

- `local_auth` 3.0.2 (publisher flutter.dev) supports `biometricOnly` and `persistAcrossBackgrounding` (formerly sticky auth). Its API returns a **boolean** — it does not bind authentication to a key. [Verified: https://pub.dev/packages/local_auth]; boolean result [Inferred: examples assign to `bool`]
- A boolean gate is fine for a UI lock but is bypassable on a rooted device. For real protection, bind the DB key to biometrics: Android `BiometricPrompt` + `CryptoObject` with a Keystore key (`setUserAuthenticationRequired`), which **requires `BIOMETRIC_STRONG`**. [Inferred: Credential Manager reference states CryptoObject requires BIOMETRIC_STRONG, read via search summary of developer.android.com]; iOS equivalent: Keychain item with `SecAccessControl` `.biometryCurrentSet`. [Unverified: prior knowledge]
- Recommendation: v1 = `local_auth` gate + Keystore/Keychain-held DB key; v2 = key-binding via a small platform channel. [Speculative design judgment]

### 2.3 E2EE sync options

| Option | How | Pros | Cons | Grade |
|---|---|---|---|---|
| A. Encrypted backup file to user's own cloud (iCloud Drive / Google Drive / file export) | Export encrypted DB snapshot | No backend, zero GDPR processor footprint for you | No real-time multi-device, conflicts | [Unverified: prior knowledge] |
| B. Own sync server storing ciphertext ops (CRDT/op-log, e.g. Automerge/Yjs-style or custom) | Client encrypts each op; server is a dumb relay | Multi-device, server never sees data → strong marketing claim | Key recovery UX (lost passphrase = lost data), more engineering | [Unverified: prior knowledge] |
| C. Managed backend (Supabase/Firebase) with plaintext + RLS | Classic SaaS | Fastest to build, server-side analytics | You become full GDPR controller of financial data; breach impact high | [Unverified: prior knowledge] |

Recommended: **local-first (no account needed) → option A → option B** for the paid tier. [Speculative]

---

## 3. GDPR obligations for a personal-finance app (France/EU)

| Obligation | What it means here | Grade |
|---|---|---|
| Lawful basis | Contract (art. 6(1)(b)) for core features; consent for analytics, marketing, optional AI/cloud OCR | [Unverified: prior knowledge of GDPR art. 6] |
| Data minimisation (art. 5(1)(c)) | Parse SMS/notifications on-device, keep only amount/merchant/date; no contacts, no location by default | [Unverified: prior knowledge] |
| Privacy by design/default (art. 25) | Local-first, E2EE, opt-in telemetry | [Unverified: prior knowledge] |
| Security (art. 32) | Encryption at rest + in transit, access control, breach procedure | [Unverified: prior knowledge] |
| DPIA (art. 35) | Not on CNIL's mandatory list (see below). WP248: two of nine criteria usually ⇒ DPIA; "data of a highly personal nature" is one criterion and financial data is commonly placed there. Bank aggregation + categorisation + "insights" can hit a second (evaluation/scoring or combining datasets). → **do a short DPIA before bank sync or cloud sync** | Threshold quote [Inferred: WP248 quoted in secondary sources, search summary]; recommendation [Inferred] |
| Records of processing (art. 30) | Keep a simple register even as a micro-business (the <250 exemption does not apply to non-occasional processing) | [Unverified: prior knowledge] |
| Processors (art. 28) | DPA with every processor: hosting, email, crash reporting (Sentry/Crashlytics), aggregator, payment/subscription (RevenueCat) | [Unverified: prior knowledge] |
| International transfers | EU-US Data Privacy Framework is in force (General Court upheld it 3 Sept 2025); appeal C-703/25 P pending at the CJEU — status at 2026-11-01 **unconfirmed** (conflicting reports). Prefer EU-hosted processors to avoid the question entirely. | [Inferred: WilmerHale / Mondaq / StreamLex search summaries]; final status [Unverified] |
| Data subject rights | Export (portability — CSV/JSON export is also a feature), deletion, access | [Unverified: prior knowledge] |
| ePrivacy / cookies (web app) | CNIL consent rules for non-essential trackers | [Unverified: prior knowledge] |

### 3.1 CNIL mandatory-DPIA list (14 types) — checked in full

Health data in health establishments; genetic data of vulnerable persons; HR profiling; constant employee monitoring; social/health alert systems; professional whistleblowing; health data warehouses; **profiling that can lead to exclusion from a contract** (e.g. credit scoring, payment-fraud detection); mutualised contractual-breach files; **profiling using external data sources** (data brokers); biometric identification of vulnerable persons; social housing; social/medico-social support; large-scale location data. [Verified: https://www.cnil.fr/sites/default/files/atoms/files/liste-traitements-avec-aipd-requise-v2.pdf]

→ A personal expense tracker is **not listed** [Verified: same]. It **would** approach the list if it ever did credit-worthiness scoring, fraud scoring, or enriched profiles from third-party data. **Avoid those features** (also avoids being a regulated service). [Inferred]

### 3.2 EU hosting

Choose an EU-headquartered host for any backend (e.g. Scaleway, OVHcloud, Hetzner, Clever Cloud) to avoid transfer analysis and US CLOUD Act exposure; for crash/analytics prefer self-hosted (GlitchTip/Sentry self-hosted, Plausible/Umami). [Unverified: prior knowledge — pricing covered by the infrastructure dimension]

### 3.3 Open banking regulatory note (if bank sync is added)

- Providing consolidated account information to users is a regulated **Account Information Service** under PSD2. Apps typically avoid licensing by operating as an **agent** of a licensed AISP or by consuming a licensed provider's service. [Inferred: Plaid/TrueLayer guidance, search summary]
- TrueLayer's article (UK/FCA framing): an agent "cannot provide or purport to provide account information services in its own right", must disclose it provides the AISP's service, consent is collected by the AISP, and "AISPs must also register their agents" with the regulator. [Verified: https://truelayer.com/blog/data-chain-agents/] — **UK rule; the French analogue (ACPR register, PSD2 agent regime) was not checked this session.** [Unverified]
- PSD3/PSR is in legislative progress and may change the regime. [Unverified: prior knowledge]

---

## 4. Google Play policy (current)

| Topic | Rule | Grade |
|---|---|---|
| SMS / Call Log | Allowed only as default SMS/Phone/Assistant handler, **or** an approved exception. Exception "**SMS-based money management** — apps that track and manage budget": READ_SMS, RECEIVE_SMS, RECEIVE_MMS, RECEIVE_WAP_PUSH (no SEND/WRITE). Exceptions apply only when "there's currently no alternative method". Spyware section: "personal loans or budgeting apps may not exfiltrate or share non-financial or personal SMS history". | [Verified: https://support.google.com/googleplay/android-developer/answer/10208820] |
| SMS relevance in France | French/EU banks mostly signal card payments via their own app push notifications, not SMS (SMS mostly used for OTP/3-D Secure). So the SMS exception has low value in the EU vs. India-style markets. | [Unverified: prior knowledge] |
| Notification Listener | User must enable it in Settings > Notification access; app declares a service guarded by `BIND_NOTIFICATION_LISTENER_SERVICE`. No dedicated Play declaration form found this session (searches of support.google.com returned none). Still subject to general User Data / Restricted-permission rules (prominent disclosure, core feature, functional if declined). | Mechanism [Unverified: prior knowledge — the docs page fetched was truncated]; no declaration found [Verified: two targeted searches, no Play page]; general rule [Inferred: support.google.com/…/answer/16585319 search summary] |
| Android 15 sensitive notifications | 2FA-code notifications are redacted ("sensitive notification content hidden") for listeners without `RECEIVE_SENSITIVE_NOTIFICATIONS` (role/signature only). | [Inferred: Android Authority reports via search summary] |
| Financial features declaration | "Any app that contains any financial features must complete the Financial features declaration form within Play Console." Policy scope: products "related to the management or investment of money". No France/EU-specific section. | [Verified: https://support.google.com/googleplay/android-developer/answer/9876821] |
| Data safety form | Mandatory for every app on closed/open/production tracks (internal-only exempt), even if no data is collected. Relevant types: *Purchase history*, *Other financial info*, *User payment info*. Asks about encryption in transit and deletion mechanism. | [Verified: https://support.google.com/googleplay/android-developer/answer/10787469] |
| Account deletion | If users can create an account: in-app deletion path **and** a web link for deletion requests (works after uninstall). Offline-only accounts are out of scope. | [Verified: https://support.google.com/googleplay/android-developer/answer/13327111] |
| Registration cost | **US$25 one-time**; identity verification (ID, card in legal name); personal accounts must verify device access via Play Console app. | [Verified: https://support.google.com/googleplay/android-developer/answer/6112435] |
| Testing requirement (personal accounts created after 13 Nov 2023) | **≥12 testers opted in continuously for 14 days** in a closed test before applying for production. (The current page says 12; a 20-tester figure is not what it says now.) | [Verified: https://support.google.com/googleplay/android-developer/answer/14151465] |
| Android developer verification (sideloading) | Go-live 30 Sep 2026 in Indonesia, Singapore, Thailand (Brazil per press); global in 2027. Full-distribution ADC account $25; free limited account (≤20 devices). Matters if distributing APKs outside Play (F-Droid/direct). | [Verified: https://support.google.com/android-developer-console/answer/16561738] |
| Service fees | Auto-renewing subscriptions: 15% in remaining markets. New tier listed for EEA together with UK/US/AU/JP (EEA rollout 30 Jun 2026): subscriptions **10% + 5% billing fee** on first $1M; other new-install transactions 20% + 5%. Alternative billing / external offers allowed in EEA "subject to program requirements" (fees not listed on that page). | [Verified: https://support.google.com/googleplay/android-developer/answer/112622] |
| DSA trader status (Play) | Google's process not found this session. | [Unverified] |

---

## 5. Apple App Store

| Topic | Rule | Grade |
|---|---|---|
| 5.1.1(i) privacy policy | Link in App Store Connect and in-app; explain consent revocation & deletion | [Verified: https://developer.apple.com/app-store/review/guidelines/] |
| 5.1.1(ii) consent | Paid functionality must not depend on granting data access; easy consent withdrawal | [Verified: same] |
| 5.1.1(v) account deletion | If account creation exists, in-app account deletion required | [Verified: same] |
| **5.1.1(ix) regulated fields** | "banking and financial services … should be submitted by a legal entity that provides the services, and not by an individual developer" | [Verified: same] |
| **3.2.1(viii)** | "Apps used for financial trading, investing, or money management should be submitted by the financial institution performing such services and must have necessary licensing" | [Verified: same] |
| Interpretation | "Money management" most plausibly means managing funds (neobank, robo-advisor), not logging one's own expenses; many indie expense trackers ship on individual accounts. Still a **reviewer-discretion risk**, higher once bank aggregation is added. | [Inferred: wording + market observation; Unverified: no Apple statement on budgeting apps found] |
| 3.1.1 IAP | Unlocking features/subscriptions must use IAP; restore mechanism required; no licence keys | [Verified: same] |
| 3.1.1(a) / US | External purchase link entitlements per storefront; US storefront exempt from the anti-steering ban | [Verified: same] |
| 3.1.2 subscriptions | ≥7 days period, must work on all user's devices | [Verified: same] |
| Privacy nutrition labels | Required in App Store Connect; must match actual collection incl. SDKs | [Unverified: prior knowledge — page not opened] |
| Developer Program fee | **99 USD / membership year** (local currency at enrolment). Individual: legal name, 2FA. Organization: D-U-N-S, legal entity (no DBA), domain email, public website. | [Verified: https://developer.apple.com/programs/enroll/] |
| Small Business Program | 15% commission if ≤1M USD proceeds; 10% for subscriptions after first year and for EU-alternative-terms developers in the program | [Verified: https://developer.apple.com/app-store/small-business-program/] |
| EU DMA terms | New EU terms in **Attachment 14** (DPLA updated 18 Aug 2026, effective **1 Oct 2026**). IAP commission 26% std / 15% small-biz & subs after year 1; alternative in-app PSP 20% / 10%; out-of-app link "store services commission" 15% / 10% on sales within 7 days; CTC 5% for alternative-marketplace/web distribution. Page **also** says the new terms "eliminate the Initial Acquisition Fee and Store Services Fee" — **internally inconsistent; read the agreement before choosing.** Must keep the chosen payment option 12 months; monthly transaction reports for alternative PSP. | [Verified: https://developer.apple.com/support/dma-and-apps-in-the-eu/] |
| DSA trader status | Monetised apps = trader; address, phone, email shown on EU product page; non-declared apps removed from EU storefronts since 17 Feb 2025. Individuals' address becomes public → privacy reason to publish via a company. | [Inferred: MacRumors/9to5Mac/TechRepublic search summaries] |
| Reading bank notifications | Not possible for third-party iOS apps (no notification-listener API). | [Unverified: prior knowledge — consistent with no such API in iOS SDK] |
| Shortcuts "Transaction" automation | iOS 17+ trigger when tapping a Wallet card/pass; provides merchant, amount, card name to an automation → app's App Intent. User must create the automation (or import a shared Shortcut). Reported unreliable/timeouts; Wallet mobile-data setting can break it. | [Inferred: vendor guides (WalletPal, TravelSpend) + Apple Developer Forums threads 746889 / 765516, search summary] |
| FinanceKit | Managed entitlement requested by Account Holder; app in Finance category; distributed in **US or UK** App Store; US = Apple Card/Cash/Savings (iOS 17.4+), UK = open-banking institutions (iOS 18.4+). Organization account required. **No France/EU.** | Regions/requirements [Verified: https://developer.apple.com/financekit]; org-account requirement [Inferred: Apple docs as summarised by search] |

---

## 6. Monetisation models and their compliance impact

| Model | Compliance impact | Recommended? | Grade |
|---|---|---|---|
| Free, local-only, donations | Minimal: no account, no processor; still needs privacy policy + Data safety/labels | Personal phase | [Inferred] |
| One-time "Pro" unlock (IAP) | IAP mandatory on both stores; no subscription obligations | Good early | [Verified: Apple 3.1.1; Play fee page] |
| Subscription (sync, bank sync, AI) | IAP/Play Billing; 15% (Apple SBP / Play subs, non-EEA); EEA Play 10%+5%; must work on all devices (Apple 3.1.2); account deletion must explain subscription cancellation (Play) | **Yes for cloud features** — costs recur, so revenue should | [Verified: pages above] |
| EU alternative PSP (Stripe/Paddle) in-app | Apple 20%/10% still due + monthly reports + 12-month lock-in; saves ~5–6 pts vs IAP minus PSP fees (~1.5–3%) | **Not worth it at indie scale** | [Inferred: arithmetic on verified rates; PSP fees Unverified] |
| Web checkout (web app / out-of-app link) | Apple out-of-app link 15%/10% commission within 7 days (if that line stands); a pure web app sold on the web pays no store fee | Good for the web/PWA tier | [Verified: Apple DMA page; inconsistency noted] |
| Ads | Brings ad SDK data sharing, consent (TCF/CMP), weakens privacy positioning | **Avoid** | [Inferred] |
| Selling/“insights” on aggregated data | Profiling → likely DPIA, possibly CNIL list item; reputational risk | **Never** | [Verified: CNIL list for profiling items] |
| Affiliate offers (cheaper bank/insurance) | May drift into intermediary-in-banking/insurance (IOBSP/IAS in France) regulation | Avoid unless advised | [Unverified: prior knowledge of French intermediary regimes] |

Use **RevenueCat** (or equivalent) to unify IAP/Play Billing entitlements — it is a processor → DPA needed. [Unverified: prior knowledge]

---

## 7. Open-source licence for a public repo that may become a product

| Licence | Effect | Fit | Grade |
|---|---|---|---|
| MIT / Apache-2.0 | Anyone can fork and ship a competing closed app | Weak protection; max adoption | [Unverified: prior knowledge] |
| AGPL-3.0 | Strong copyleft incl. network use; deters closed SaaS forks. App Store distribution conflict historically raised for GPL (VLC pulled from App Store 2011 after a copyright holder's complaint, returned after relicensing) — **the sole copyright holder is not bound by their own licence**, so you can still ship to stores; conflict appears only once outside contributions arrive → requires a **CLA** or DCO + relicensing grant | Good if you want "open but no free-riding", with a CLA | VLC history [Inferred: Engadget/TechSpot coverage via search summary]; copyright-holder point [Unverified: prior knowledge] |
| FSL-1.1 (Functional Source License) | "You can do anything with FSL software except undermine its producer"; each version becomes Apache-2.0 or MIT **after two years** | **Best fit** for "public now, product later" — not OSI open source, but source-available with a guaranteed open future | [Verified: https://fsl.software/] |
| BSL 1.1 | Similar delayed-open model, configurable change date (≤4 years) and additional use grant | Fine, more knobs | [Unverified: prior knowledge] |
| Open-core | Core (FSL/AGPL) public; sync server / bank-sync connectors private | Common commercial pattern | [Speculative] |

**Recommendation:** FSL-1.1-ALv2 for the app + server, or AGPL-3.0 + CLA if OSI "open source" status matters; keep secrets, aggregator keys and signing configs out of the repo; add `SECURITY.md`. [Speculative judgment, licence facts as graded above]

**Public-repo hygiene (privacy):** no real transactions in fixtures, synthetic data only; no personal bank names in parsers' test data beyond public sender formats; secret scanning (gitleaks) in CI. [Speculative]

---

## 8. One-time and recurring compliance costs

| Item | Cost | Grade |
|---|---|---|
| Google Play developer registration | US$25 once | [Verified: https://support.google.com/googleplay/android-developer/answer/6112435] |
| Apple Developer Program | US$99 / year | [Verified: https://developer.apple.com/programs/enroll/] |
| D-U-N-S number (org accounts) | Free (up to ~28 days) | [Verified: https://support.google.com/android-developer-console/answer/16561738] |
| 12 closed testers × 14 days | Time cost (~2–3 weeks calendar); recruiting friends/communities | [Verified: https://support.google.com/googleplay/android-developer/answer/14151465] |
| Company for org accounts (e.g. French micro-entreprise / SASU) | Micro-entreprise registration is free; SASU has formation + accounting costs | [Unverified: prior knowledge] |
| Privacy policy + terms + DPIA | DIY with CNIL templates (0 €) or lawyer review (several hundred to a few thousand €) | [Unverified: prior knowledge] |
| Pen-test before bank-sync launch | Optional; MASVS self-assessment first | [Speculative] |

---

## 9. Recommended compliance roadmap (aligned to product phases)

1. **Phase 0 — personal MVP (local-only):** encrypted DB (`sqlite3` 3.x), Keystore/Keychain key, `local_auth` lock, no account, no network except optional FX rates. Privacy policy even if collecting nothing. Android: notification-listener capture behind prominent disclosure; **no SMS permission**. iOS: Shortcuts Transaction → App Intent. [Speculative design]
2. **Phase 1 — store beta:** Play personal account + 12 testers/14 days; Data safety form ("no data collected" if true); Financial features declaration; Apple individual account, position as *expense tracker/organizer*, clear "no financial services" metadata. [Verified rules above; positioning Speculative]
3. **Phase 2 — sync + subscription:** E2EE sync on EU host; DPA register; short DPIA; account deletion in-app + web link; IAP/Play Billing subs via RevenueCat; **switch to organization accounts (D-U-N-S)** — also hides home address on EU App Store pages. [Inferred]
4. **Phase 3 — bank sync (optional):** licensed aggregator (not GoCardless — closed); agent/consumer model per aggregator terms; check ACPR implications; MASVS L2; reconsider Apple 3.2.1(viii) exposure. [Inferred]

---

## Sources (opened this session unless marked)

- Google Play — Use of SMS or Call Log permission groups: https://support.google.com/googleplay/android-developer/answer/10208820
- Google Play — App testing requirements for new personal developer accounts: https://support.google.com/googleplay/android-developer/answer/14151465
- Google Play — Financial Services policy: https://support.google.com/googleplay/android-developer/answer/9876821
- Google Play — Data safety section: https://support.google.com/googleplay/android-developer/answer/10787469
- Google Play — Account deletion requirements: https://support.google.com/googleplay/android-developer/answer/13327111
- Google Play — Developer registration: https://support.google.com/googleplay/android-developer/answer/6112435
- Google Play — Service fees: https://support.google.com/googleplay/android-developer/answer/112622
- Android Developer Console — Understanding Android developer verification: https://support.google.com/android-developer-console/answer/16561738
- Apple — App Review Guidelines: https://developer.apple.com/app-store/review/guidelines/
- Apple — Enrollment: https://developer.apple.com/programs/enroll/
- Apple — App Store Small Business Program: https://developer.apple.com/app-store/small-business-program/
- Apple — DMA and apps in the EU: https://developer.apple.com/support/dma-and-apps-in-the-eu/
- Apple — Get started with FinanceKit: https://developer.apple.com/financekit
- CNIL — Liste des traitements AIPD requise (PDF): https://www.cnil.fr/sites/default/files/atoms/files/liste-traitements-avec-aipd-requise-v2.pdf
- OWASP MASVS: https://mas.owasp.org/MASVS/
- pub.dev — sqlcipher_flutter_libs (EOL): https://pub.dev/packages/sqlcipher_flutter_libs
- pub.dev — sqlite3: https://pub.dev/packages/sqlite3
- pub.dev — local_auth: https://pub.dev/packages/local_auth
- GoCardless — Bank Account Data signups disabled: https://bankaccountdata.gocardless.com/new-signups-disabled
- TrueLayer — Customers and the data chain: agents: https://truelayer.com/blog/data-chain-agents/
- Functional Source License: https://fsl.software/
- *Search-summary only (not opened):* Android Authority on Android 15 OTP redaction (https://androidauthority.com/android-15-two-factor-authentication-codes-3492585); Apple Developer Forums threads 746889 and 765516 (Shortcuts Transaction trigger); WalletPal / TravelSpend Shortcuts guides; MacRumors & TechRepublic on DSA trader status; WilmerHale / StreamLex on the EU-US DPF appeal; Fieldfisher national DPIA lists; Engadget/TechSpot on VLC & GPL (2011); Plaid AISP blog.
