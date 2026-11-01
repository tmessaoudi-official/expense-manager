# 07b — Capture & market in Tunisia (no PSD2)

> Dimension: how a user with ties to France and Tunisia gets **Tunisian (TND) transactions** into a
> local-first Flutter expense app (Android first), and what the Tunisian market, documents and languages
> change for capture, OCR and parsing.
> Research date: **2026-11-01**. Grades: `[Verified: url]` = opened this session; `[Inferred: basis]`
> (a WebSearch summary counts as Inferred, never as Verified); `[Unverified: …]` = prior knowledge or not reachable.
> Sibling files: 07c (multi-currency design: exponent table, FX, rounding). Exchange-control and customs
> rules belong there or to a dedicated file. This file only flags them, as Unverified.

---

## TL;DR — what changes for Tunisia

| # | Finding | Consequence for the app | Grade |
|---|---|---|---|
| 1 | **No open-banking or API path exists, and none is scheduled.** The BCT is "exploring" open banking (an evaluation in 2023). Fiskil tags the status "Planned". The new payment-institution circular (n° 2026-10) contains no account-information or API provision. | Tunisian capture = **manual + OCR + PDF/statement import + Android notification/SMS parsing**. No aggregator can be bought, whatever the budget. | [Verified: https://www.fiskil.com/open-finance-tracker/tunisia ; circular text, see §2] |
| 2 | **TND has 3 decimals** (ISO 4217 minor units = 3; 1 dinar = 1000 millimes). Only 7 ISO currencies have exponent 3 (BHD, IQD, JOD, KWD, LYD, OMR, TND). | Each currency needs its own exponent (07c). Parsers must read `12.500` / `12,500` in a TND context as **12 dinars 500 millimes**, which is the opposite of the EUR reading. | [Verified: ISO 4217 List One, `Pblshd="2026-09-17"`, https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml] |
| 3 | **Wallets and payment institutions are TND-only and domestic**: "Les services de paiement sont fournis exclusivement en dinar tunisien et à l'intérieur du territoire tunisien" (circular 2026-10, art. 1). | A wallet account in the app is always TND. EUR↔TND movements happen only through banks or money-transfer operators and must be modelled as a **transfer between two accounts in different currencies**. | [Verified: circular 2026-10 PDF, see §2] |
| 4 | **Apple Pay is not offered in Tunisia** (France is). | The iOS Shortcuts "Transaction" automation in 02 §5.2 does **not** work for Tunisian cards. On iOS, Tunisian capture is manual, OCR or file import only. | [Verified: https://support.apple.com/en-us/102775] |
| 5 | **SMS matters more than in France.** Banks run SMS alert services (UIB since 2015, STB since 2004), utilities send bills by SMS, and mobile payments are validated by **SMS codes** through Tunisie Monétique. | On Android, SMS/notification parsing is a core Tunisian feed. However, OTP-bearing messages are **redacted** for notification listeners on Android 15+, and SMS alerts can have amount thresholds (UIB: ≥ 500 TND), so the feed is **incomplete**. | [Verified: UIB, TUNPAY and Android sources in §3–§4] |
| 6 | **Receipts carry a fiscal stamp line.** The stamp duty is TND 1 per invoice. Since LF 2026, invoices from large commercial areas pay **TND 1.500** (50–100 TND invoices) or **TND 2.000** (> 100 TND invoices). VAT is 19 % / 13 % / 7 %. | OCR must treat `droit de timbre` / `timbre fiscal` as a **tax component of the total**, not as a line item, and must accept three VAT rates. | [Verified: https://taxsummaries.pwc.com/tunisia/corporate/other-taxes — last reviewed 29 Jun 2026; secondary source, JORT not opened] |
| 7 | **On-device OCR has no Arabic in ML Kit.** Text Recognition v2 supports Chinese, Devanagari, Japanese, Korean and Latin scripts only, and Arabic is in none of its language tables. | French/Latin receipts work with ML Kit. Arabic-script receipts need another engine (Tesseract `ara`/`Arabic`, or a cloud OCR with consent), and that needs a sample test. | [Verified: https://developers.google.com/ml-kit/vision/text-recognition/v2 and …/v2/languages] |
| 8 | **The economy is cash-heavy and getting more so**: notes and coins in circulation are about **30.2 bn TND** (end Aug 2026), up about 16 % year on year, a rise linked to the new cheque law. | **Cash accounts and cash envelopes are first-class.** ATM-withdrawal capture plus a later "where did the cash go" reconciliation is a core Tunisian flow. | [Inferred: WebSearch summaries of BCT balance-sheet coverage — https://fr.allafrica.com/stories/202609160562.html , https://www.tunisienumerique.com/tunisie-le-volume-de-billets-et-monnaies-en-circulation-frole-un-record-de-30-milliards-de-dinars/ (fetch timed out)] |
| 9 | **Local personal-finance market ≈ nil.** Sensor Tower's Q4 2023 top-5 Tunisian budgeting apps are all global (Wallet, Splitwise, YNAB, Mobills, Money Manager), and the leader peaks at **$16/week** in revenue. Banks offer only basic budgeting (an Amen Bank 2016 web service with threshold SMS alerts; NEO BTE). | No local incumbent to beat. The persona is served by **global apps that ignore TND specifics** (3 decimals, timbre, SMS), and that gap is the differentiator. Willingness to pay locally is low. | [Verified: https://sensortower.com/blog/2023-q4-unified-top-5-personal%20finance%20budgeting%20and%20planning-revenue-tn-63e363f0e1714cfff1979caa ; https://businessnews.com.tn/2016/03/16/article-1136262/1136262/] |

---

## 1. Capture channels ranked for Tunisian transactions

| Rank | Channel | Coverage | Effort | Risk | Grade |
|---|---|---|---|---|---|
| 1 | **Manual quick-add + cash account** | Everything, including the large cash share | Already in MVP (04 §2.1) | None | [Inferred: cash data in TL;DR #8] |
| 2 | **Android notification listener** on allow-listed bank and wallet apps **and on the SMS app** (which receives bank SMS alerts) | Card payments and transfers in real time, where the bank pushes or texts alerts. OTP messages are redacted. | Per-bank templates; same engine as France | Play review risk (02 §5.1, 00-factcheck) | [Inferred: Android behaviour per §4; no Tunisian sample collected] |
| 3 | **SMS read via `READ_SMS` / `RECEIVE_SMS`** (Play "SMS-based money management" exception) | Back-fills SMS history on install | Declaration + review | Play may grant only a **temporary** exception and requires "no alternative method". The case is more defensible in Tunisia (SMS is the alert channel) but still a review risk. | [Inferred: Play policy confirmed in 00-factcheck (support.google.com/…/10208820, not re-opened here)] |
| 4 | **PDF statement import** (text-layer PDF parser per bank) | Monthly reconciliation of everything booked | Per-bank layout parsers; scanned PDFs need OCR | Layouts change | [Inferred: statement downloads are documented for Attijari Real Time and Webank — WebSearch snippets; no format sample] |
| 5 | **Receipt OCR** | Shop purchases with a printed ticket | ML Kit for Latin script; Arabic needs another engine | Arabic digits and script (see §5) | [Verified: ML Kit pages] |
| 6 | **CSV/OFX import** | Only if a bank offers it | Generic mapping wizard already planned | **No Tunisian bank found that offers CSV or OFX** | [Unverified: not found in any source; check each bank's web banking] |
| — | **Open banking / aggregator API** | — | — | Does not exist | [Verified: Fiskil; circular 2026-10] |

**Design rule:** because no Tunisian feed is complete, the **monthly statement reconciliation** (rank 4)
is the ground truth. Notifications and SMS (ranks 2–3) give speed, the duplicate guard (04 §2.1) merges
the two, and cash closes the gap. [Inferred: from the coverage limits above]

---

## 2. Regulation & open banking (BCT)

- **Open banking:** status "Planned". "Tunisia is exploring open banking regulation through the Banque
  Centrale de Tunisie", with a 2023 evaluation named as the only initiative. No regulation is named.
  [Verified: https://www.fiskil.com/open-finance-tracker/tunisia]
  Banq.ai lists 34 Tunisian banks with **0 sandboxes and 0 API aggregators**. [Inferred: WebSearch snippet, https://www.banq.ai/country/tunisia]
- **Circular n° 2026-10 (25 Sept 2026)**, "Règles régissant l'activité et le fonctionnement des
  établissements de paiement". It repeals circular 2018-16, takes effect three months after publication,
  runs to 46 pages and had a corrected version on 28 Sept 2026 (numbering only).
  [Verified: text extracted with `pdftotext` from https://businessnews.com.tn/wp-content/uploads/2026/09/yA1IYG-Cir_2026_10_fr.pdf — hosted by the press, not on bct.gov.tn; summary at https://managers.tn/2026/09/26/nouvelle-circulaire-de-la-bct-voici-ce-qui-change-pour-les-etablissements-de-paiement/ ; the correction date comes from a WebSearch snippet]
  - **Art. 1:** services "exclusivement en dinar tunisien et à l'intérieur du territoire tunisien". [Verified: same PDF]
  - **Art. 16 (payment-account tiers):** level 1 balance ≤ 1 500 TND (individuals only); level 2 ≤ 5 000 TND with cash withdrawals ≤ 3 000 TND/day; level 3 ≤ 20 000 TND with cash withdrawals ≤ 10 000 TND/day. [Verified: same PDF, dated 25 Sept 2026 — these are payment-institution caps and can change by a later circular]
  - **Art. 2:** cash-based transfers ≤ 3 000 TND per operation. Funds from abroad ≤ the equivalent of 20 000 TND per operation, "dûment justifiées". [Verified: same PDF, 25 Sept 2026]
  - **Art. 38(b):** after each transaction the client must be told its outcome, amount, **fees and taxes,
    new balance and reference number**. This is the content a wallet notification or SMS should carry,
    which makes it a useful parsing target (amount + fee + balance + ref). [Verified: same PDF]
  - **Art. 38(c):** a statement of operations is provided "par tout moyen approprié, … selon les modalités prévues dans la convention de compte". **No format is mandated**, so there is no CSV guarantee. [Verified: same PDF]
  - **No provision on APIs, data portability or third-party account access** was found in the text. [Verified: grep of the extracted text for `API`, `interop`, `historique`, `relev` — only art. 38(c)]
- **Mobile payment:** in May 2026 the BCT created a single national label, **TUNPAY**, with Société
  Monétique Tunisie. Mobile payments "repose[nt] toujours sur … des transactions validées par codes de
  confirmation SMS via Tunisie Monétique". 2025: 8.4 M operations, 1 769 MTND.
  [Verified: https://www.webdo.tn/fr/actualite/national/paiement-mobile-la-bct-lance-le-label-tunpay-pour-unifier-lecosysteme/397433/ (6 May 2026)]
- **Instant payment:** in Sept 2026 a BCT delegation studied Brazil's PIX. This is a study, not a launch. [Inferred: WebSearch snippet, https://www.webdo.tn/fr/actualite/national/paiement-instantane-la-bct-regarde-du-cote-du-modele-bresilien-pix/404153/]
- **H1 2026 payment figures** (BCT bulletin "Paiements en chiffres"): 6 151 k bank cards, 3 427 ATMs,
  45.6 k POS terminals, **218 886 mobile wallets** (+5.5 %), 5 M mobile transactions worth 914.2 MTND.
  [Verified: https://managers.tn/2026/08/14/paiements-en-tunisie-les-usages-numeriques-poursuivent-leur-progression-au-premier-semestre-2026/ — press relay of the BCT bulletin]

**Exchange control / customs** (travel allowance, the international technology card, convertible-dinar
or foreign-currency accounts for non-residents, cash-declaration thresholds): **not researched here, and
no bct.gov.tn or douane.gov.tn page was opened.** The app must be able to model these as **parameters**
(ParameterTable rows with a source URL and validity dates). No value is given here.
[Unverified: check before relying on it]

---

## 3. Banks, wallets and their signals

### 3.1 Banks (mobile apps, alerts, statements)

| Bank | App / channel | Alerts | Statement / export | Grade |
|---|---|---|---|---|
| **BIAT** | MyBIAT (Play, App Store, AppGallery), > 370 k active retail users when MyBIAT Corporate launched | Push-notification management (aggregator listing) | Not documented | [Inferred: WebSearch snippets — kapitalis.com 2022 launch, mwm.ai listing; mybiat.tn refused the connection] |
| **Attijari bank (TN)** | Attijari UP (new, June 2025), Attijari Mobile Tunisie (older), **Webank** (digital) | Webank: push notifications | Attijari Real Time and Webank: **downloadable statements** (format not stated; PDF assumed) | [Inferred: WebSearch snippets — kapitalis.com 2025-06-03, appgoblin/mwm listings] |
| **UIB** | UIB Mobile | **MESSAG'EK**: free SMS for cash movements / transfers **≥ 500 TND**, credits and debits, deposit and savings accounts (since 15 Dec 2015) | Not documented | [Verified: https://www.leconomistemaghrebin.com/2015/12/21/uib-service-gratuit-information-sms/ — 2015, may have changed] |
| **STB** | STB app; STBNET, STB Direct, **STB SMS** (SMS banking since 2004) | SMS banking | Transaction history in the app, by year (Play listing) | [Inferred: WebSearch snippets — webmanagercenter 2008, managers.tn 2020, appwisp listing] |
| **BH Bank** | "BH BANK" app | Not documented | Not documented | [Verified: listed in https://managers.tn/2020/07/20/vous-partez-en-vacances-grace-a-ces-applications-votre-banque-vous-suivra-partout (2020)] |
| **Amen Bank** | Amen Mobile ("suivi des opérations"); @mennet web banking had a **budget service with threshold SMS alerts** (2016) | Threshold SMS (2016) | Not documented | [Verified: same managers.tn 2020 roundup; businessnews.com.tn 2016] |
| **La Poste Tunisienne** | e-Dinar accounts + **D17** app (balance, transfers, bills, Masterpass QR merchant payments, motorway tolls since Dec 2025) | PIN per payment | Not documented | [Inferred: WebSearch snippets — apps.apple.com DigiPostBank D17, managers.tn 2026-03-10, idaraty.tn] |
| Others | The 2020 roundup counts **18 m-banking apps** (BNA, ATB, BT, UBCI, QNB, Zitouna, BTK "multilingual, multi-currency", BTE …) | — | "historique des transactions" and a statement of operations, described generally | [Verified: managers.tn 2020 roundup] |

**Gap (stated plainly):** no Tunisian bank was confirmed to offer **CSV, OFX, QIF or CAMT** export.
PDF statements are the only export seen, and even that is from snippets. No current **sample** of a bank
SMS, push notification or statement layout was found for any bank. The personal phase should therefore
start with a **sample-collection step** (redacted screenshots and PDFs from the user's own apps, kept
out of the public repo) before any template is written. [Unverified: absence of evidence, not evidence of absence]

### 3.2 Wallets & payment apps

| App | What it is | Relevance to capture | Grade |
|---|---|---|---|
| **Flouci** | Wallet linked to a Webank account, QR merchant payments, bill pay, top-ups, physical and virtual Visa cards; ~500 k downloads, > 140 k accounts (June 2026); Shopify, WooCommerce and PrestaShop plugins; a BCT-authorised payment facilitator | Push notifications likely (not verified). The **API is merchant-only** (QR, pre-auth, webhooks), with no end-user transaction-history API. | [Verified: https://www.leconomistemaghrebin.com/2026/06/08/paiement-digital-flouci-premiere-tunisie/ ; https://docs.flouci.com] |
| **D17 / e-Dinar** (La Poste) | Wallet on e-Dinar virtual accounts; QR (Masterpass), transfers, bills, tolls | Notification and SMS formats unknown | [Inferred: WebSearch snippets, see 3.1] |
| **Sobflous** | Wallet since 2012–13: transfers by e-mail or QR, bill pay, top-ups, transfer to bank account; applied for a payment-institution licence in 2020 | Licence status unconfirmed | [Inferred: WebSearch snippets — apps.apple.com id1514434055, managers.tn 2020] |
| **Utility bills** | STEG (electricity and gas): SMS with amount and due date to about 60 % of customers (2020), e-mail bills since 2021, USSD `*127#` payment. SONEDE (water): SMS bill service launched July 2025. | **Bill reminders** (04 §2.4) can be seeded from these SMS. Periodicity was not verified. | [Inferred: WebSearch snippets — leconomistemaghrebin 2020, webmanagercenter 2021, allafrica 2025-07 (403 on fetch)] |

---

## 4. Android & iOS capture — what changes vs France

- **OTP redaction (Android 15+, all apps):** "Android will stop untrusted apps that implement a
  NotificationListenerService from reading unredacted content from notifications where an OTP has been
  detected." [Verified: https://developer.android.com/about/versions/15/behavior-changes-all]
  Tunisian mobile and online payments are validated by **SMS codes**, so the OTP SMS itself is
  invisible to the listener. The **post-transaction** confirmation (art. 38(b): amount, fees, balance, ref)
  is the message to parse. Whether the system's OTP detector also redacts confirmation messages that
  quote a reference number is **untested**. [Inferred: from the Android text + circular; device test required]
- **SMS via the SMS app's notification:** a listener allow-listed to the default SMS app sees
  bank-alert SMS without `READ_SMS`. This avoids the SMS-permission declaration but not the
  notification-access review risk, and it sees only new messages, not history. [Inferred: standard Android behaviour; Play stance Unverified per 00-factcheck]
- **Incomplete feeds:** threshold-based alerts (UIB ≥ 500 TND) miss small payments, which are the bulk of
  daily spending. The app should show **"feed coverage"** per account (how much of the statement total the
  live feed caught) so the user knows to reconcile. [Inferred: UIB threshold, §3.1]
- **Sender identity:** bank SMS come from alphanumeric sender IDs, which are not verified. Parse only
  allow-listed senders and never act on links in SMS (phishing). [Unverified: prior knowledge]
- **iOS:** no listener, no SMS access, **no Apple Pay in Tunisia**, so the Shortcuts "Transaction"
  trigger cannot fire for Tunisian cards. Tunisian capture on iOS = manual, OCR, PDF import and share-sheet.
  [Verified: https://support.apple.com/en-us/102775]
- **Dual-SIM / roaming:** a user with ties to both countries may carry a French and a Tunisian SIM.
  SMS alerts arrive on the SIM registered with each bank. Nothing changes for parsing, but the account →
  currency mapping must come from the **sender/app**, not from the phone's locale. [Speculative]

---

## 5. Receipts, invoices & amount formats

### 5.1 Fiscal content of a Tunisian receipt/invoice

| Element | Value | Parser rule | Grade |
|---|---|---|---|
| VAT rates | 19 % standard; 13 % and 7 % reduced (LF 2026 moved solar-panel imports 7 → 19 %) | Accept {19, 13, 7} plus 0/exempt. A ParameterTable row per rate, dated. | [Verified: PwC page above, reviewed 29 Jun 2026] |
| Stamp duty (droit de timbre) | **TND 1.000** per invoice (up from 0.600 on 1 Jan 2023); large commercial areas under LF 2026: **TND 1.500** (invoice 50–100 TND), **TND 2.000** (> 100 TND); export invoices exempt | Keep it in the total, tag it as tax, never as a product line. Rendered as `1,500 DT` it means **1.5 dinars**. | [Verified: PwC page above — secondary; JORT text not opened] |
| E-invoicing (El Fatoora, Tunisie TradeNet) | Extended to VAT-liable **service providers from 1 Jan 2026** (LF 2026); B2C transmission rules differ; bank statements excluded | Consumers will mostly still get **paper or PDF**. A future import path for structured e-invoices (TEIF XML) is possible but not for MVP. | [Inferred: WebSearch snippets — tuniscope.com 2026-01-23, lapresse.tn, noqta.tn; no official text opened] |
| Mandatory mentions | Tax ID (*matricule fiscal*) of both parties, net amount, VAT rate and amount | Useful anchors for OCR field detection on invoices (not on till tickets) | [Inferred: WebSearch snippet from a secondary source; Code de la TVA not opened] |

### 5.2 Amount formats (the main parsing trap)

CLDR output for 1 234,5 in each locale, checked with Node ICU 78.3 / CLDR 48.0
(`Intl.NumberFormat`) [Verified: run locally this session]:

| Locale | TND | EUR |
|---|---|---|
| fr-TN | `1 234,500 DT` | `1 234,50 €` |
| ar-TN | `‏1.234,500 د.ت.‏` (Latin digits, `.` grouping, `,` decimal) | `‏1.234,50 €` |
| fr-FR | `1 234,500 TND` | `1 234,50 €` |

Rules the NL parser and OCR post-processor need [Inferred: ISO exponent + CLDR output + till-ticket conventions, Unverified on real tickets]:
1. **Decide the currency first** (from the account, sender, `DT` / `TND` / `د.ت` / `millimes` / `مليم`, or the
   user's current country), then the separators.
2. In **TND**, a separator followed by exactly 3 digits at the end of the amount is the **decimal**:
   `12.500`, `12,500` and `12 500 millimes` all mean 12.5 TND. In **EUR** the same shape is a thousands group.
3. Prices are often said, and sometimes printed, in **millimes** (`500 millimes`, or `500` on a price tag).
   Accept a millimes unit and do not multiply by 1000 twice. [Inferred: WebSearch snippet, ou-et-quand.net — "it's usual … day to day millimes"]
4. Round only at display. Store `amount_minor` in millimes (exponent 3). EUR↔TND conversion rounding is 07c's policy.
5. Tests: a table of real strings (`1.500`, `1,500 DT`, `1 500,000`, `15 DT 500`, `د.ت 12,350`) in both
   currencies, in the money-type suite required by 06 Phase 0.

### 5.3 OCR engine choice

- **ML Kit** (planned in 04 §2.1): Latin script only for our purposes, **no Arabic**. [Verified: ML Kit v2 pages]
  Most supermarket and fuel tickets are printed in French/Latin script [Unverified: prior knowledge — collect samples].
- **Apple Vision:** the page body did not load, so its Arabic support is **not confirmed**. Check at runtime
  with `supportedRecognitionLanguages()`. [Unverified]
- **Tesseract** `ara` (tessdata_best / tessdata_fast): forum reports say `ara` holds Western digits while
  the separate `Arabic` script model holds Eastern Arabic digits, and that fine-tuning on numerals can
  backfire. [Inferred: WebSearch snippets — github.com/tesseract-ocr/tessdoc Data-Files.md, groups.google.com tesseract-ocr threads]
- **Recommendation:** ML Kit first (it covers the Latin majority). An Arabic receipt falls back to
  manual entry, or to a cloud OCR with explicit consent (05 compliance). Run a **20-receipt Tunisian
  sample benchmark** before choosing Tesseract. [Speculative]

---

## 6. Languages

- **Three registers:** French (banks, receipts, most bank-app UIs), Modern Standard Arabic (official
  documents, some SMS), and **Tunisian Arabic (Derja)**, often typed in Latin script with digits for
  letters ("Arabizi": 3 = ع, 7 = ح, 9 = ق). [Unverified: prior knowledge]
- **Consequence for voice and NL quick-add** (04 §2.1): the parser must accept French, MSA and Arabizi
  category words. Arabizi digits must not be read as amounts: `9ahwa 2.500` = coffee 2.5 TND.
  This needs a token rule (a digit inside a word = a letter). [Speculative]
- **UI:** the planned FR / EN / AR with RTL (04 §2.15) is sufficient. CLDR's ar-TN uses **Latin digits**
  by default (table 5.2), so Eastern-Arabic digits should stay a user option, not the ar-TN default.
  [Verified: CLDR output above]

---

## 7. Cash-heavy usage — product implications

- Notes and coins in circulation are about **30.2 bn TND (31 Aug 2026)**, up about 16 % on the year. Press attributes
  the rise to fewer and smaller cheques after the new cheque law, plus seasonal withdrawals.
  [Inferred: WebSearch snippets — allafrica.com 2026-09-16, tunisienumerique.com, lapresse.tn 2026-05-08]
- The features that follow [Speculative unless noted]:
  1. **ATM withdrawal = transfer bank → cash wallet**, never an expense. Category budgets would otherwise double-count it.
  2. **Cash envelopes** with a periodic "count your cash" prompt; the unexplained difference goes to an "untracked cash" line, so the budget stays honest.
  3. **Fast cash entry** in millimes with common amounts (0.500, 1, 2, 5 …) as chips.
  4. **Wallet top-ups** (cash-in at an agent) are transfers too; the circular's 3 000 TND cash-transfer cap and the wallet tiers are optional validation hints. [Verified for the values: circular 2026-10, art. 2 and 16]
- **Rate gap (budgeting fact only):** a user moving money between EUR and TND sees a difference between a
  reference rate and the rate their card, transfer or cash exchange actually gave. The app stores the
  **rate actually obtained** (`fx_rate`, `fx_source` = card / transfer / cash / manual) next to a
  reference rate, and reports the difference as a cost. 07c owns the method. [Inferred: 04 §2.7 data model]

---

## 8. Data-model deltas (to hand to 04 §6 / 07c)

| Change | Why | Grade |
|---|---|---|
| Currency exponent table (TND = 3) driving `amount_minor` | ISO minor units | [Verified: ISO List One] |
| `Transaction.source` adds `sms`, `notification`, `pdf_statement` (today: manual/import/ocr/voice/bank) | Tunisian feeds | [Inferred] |
| `Transaction.fee_minor`, `balance_after_minor`, `external_ref` (optional) | Art. 38(b) messages carry fee, balance and reference; the duplicate guard and reconciliation use them | [Verified: circular art. 38(b)] |
| `Receipt.ocr_fields` keys `stamp_duty_minor`, `vat_lines[]` | Timbre fiscal and 3 VAT rates | [Verified: PwC] |
| `Account.feed_coverage` (computed) | Threshold SMS / redacted OTP give incomplete feeds | [Inferred] |
| ParameterTable `tn.*` keys (VAT rates, stamp duty, wallet tiers, any exchange-control limits), each with `source_url` and validity dates | Rules change by circular / finance law | [Inferred] |
| Capture templates per (country, sender/package) as **downloadable data**, not code | Bank formats change; FR and TN templates share one engine | [Speculative] |

---

## 9. What this file did not verify

- Any CSV/OFX/QIF export at a Tunisian bank; actual SMS, push and PDF formats for any bank or wallet.
- mybiat.tn (connection refused), the SONEDE article (HTTP 403), the cash-circulation article (timeout), and the Apple Vision language list (page body empty).
- The official text of LF 2026 (JORT), Code de la TVA, El Fatoora/TTN rules, and bct.gov.tn itself (the circular came from a press-hosted copy of the PDF).
- Everything exchange-control and customs. **Check before relying on it.**
- STEG/SONEDE billing periodicity; Sobflous licence status; whether D17 or Flouci send push notifications.

---

## Sources

Opened this session (Verified):
- Circular BCT n° 2026-10 (25 Sept 2026), PDF: https://businessnews.com.tn/wp-content/uploads/2026/09/yA1IYG-Cir_2026_10_fr.pdf
- https://managers.tn/2026/09/26/nouvelle-circulaire-de-la-bct-voici-ce-qui-change-pour-les-etablissements-de-paiement/
- https://www.fiskil.com/open-finance-tracker/tunisia
- https://taxsummaries.pwc.com/tunisia/corporate/other-taxes (last reviewed 29 Jun 2026)
- https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml (published 2026-09-17)
- https://www.webdo.tn/fr/actualite/national/paiement-mobile-la-bct-lance-le-label-tunpay-pour-unifier-lecosysteme/397433/
- https://managers.tn/2026/08/14/paiements-en-tunisie-les-usages-numeriques-poursuivent-leur-progression-au-premier-semestre-2026/
- https://www.leconomistemaghrebin.com/2015/12/21/uib-service-gratuit-information-sms/
- https://managers.tn/2020/07/20/vous-partez-en-vacances-grace-a-ces-applications-votre-banque-vous-suivra-partout
- https://www.leconomistemaghrebin.com/2026/06/08/paiement-digital-flouci-premiere-tunisie/
- https://docs.flouci.com
- https://developer.android.com/about/versions/15/behavior-changes-all
- https://support.apple.com/en-us/102775
- https://developers.google.com/ml-kit/vision/text-recognition/v2 and https://developers.google.com/ml-kit/vision/text-recognition/v2/languages
- https://sensortower.com/blog/2023-q4-unified-top-5-personal%20finance%20budgeting%20and%20planning-revenue-tn-63e363f0e1714cfff1979caa
- https://businessnews.com.tn/2016/03/16/article-1136262/1136262/
- Local check: Node `Intl.NumberFormat`, ICU 78.3 / CLDR 48.0

Search snippets only (Inferred):
- https://www.banq.ai/country/tunisia · https://www.webdo.tn/fr/actualite/national/paiement-instantane-la-bct-regarde-du-cote-du-modele-bresilien-pix/404153/
- https://fr.allafrica.com/stories/202609160562.html · https://www.tunisienumerique.com/tunisie-le-volume-de-billets-et-monnaies-en-circulation-frole-un-record-de-30-milliards-de-dinars/ · https://www.lapresse.tn/2026/05/08/forte-hausse-des-billets-en-circulation-plus-de-28-milliards-de-dinars-en-tunisie/
- https://kapitalis.com/tunisie/2022/03/02/la-biat-lance-sa-nouvelle-offre-digitale-mybiat/ · https://mwm.ai/apps/mybiat/1516557814
- https://kapitalis.com/tunisie/2025/06/03/lancement-de-la-nouvelle-application-mobile-attijari-up/ · https://appgoblin.info/apps/tn.com.attijarirealtime.mobile · https://mwm.ai/apps/webank/1436966767
- https://www.webmanagercenter.com/2008/07/10/44552/la-stb-et-tunisiana-lancent-la-recharge-gsm/ · https://appwisp.com/app/android/com.pixelstrade.stb
- https://apps.apple.com/us/app/digipostbank-d17/id1475640303 · https://managers.tn/2026/03/10/les-tunisiens-peuvent-desormais-recharger-leur-badge-de-telepeage-a-distance/ · https://idaraty.tn/fr/procedures/obtention-de-la-carte-d17-la-poste
- https://apps.apple.com/us/app/id1514434055?l=en-us · https://managers.tn/2020/07/27/la-premiere-fintech-tunisienne-se-transforme-en-etablissement-de-paiement/
- https://www.leconomistemaghrebin.com/2020/06/23/steg-nouvelles-technologies-faciliter-paiement-factures/ · https://www.webmanagercenter.com/2021/01/21/462366/hichem-anane-presente-le-nouveau-format-de-la-facture-de-la-steg/ · https://fr.allafrica.com/stories/202507280622.html
- https://www.tuniscope.com/article/423074/business/economie/facturation-electronique-132422 · https://www.noqta.tn/en/blog/facturation-electronique-tunisie-guide-2026 · https://www.lapresse.tn/?p=4306360
- https://github.com/tesseract-ocr/tessdoc/blob/master/Data-Files.md · https://groups.google.com/g/tesseract-ocr/c/7GRGV9NjFow/m/BcCWt56KBQAJ
- https://apps.apple.com/mx/app/neo-bte/id6746444685 · https://www.ou-et-quand.net/monnaie/tunisie/
