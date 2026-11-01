# 02 — Bank Sync Feasibility (EU + France)

> Research dimension: open banking / bank sync for an expense manager + reminder + tracker.
> Research date: 2026-11-01. Audience: a solo developer building for personal use first,
> with an architecture that could become a commercial product later.
>
> **Grading.** `[Verified: url]` = page opened in this research session and the claim read there.
> `[Inferred: basis]` = follows from evidence (including search-result snippets that were
> not opened) but not confirmed directly. `[Unverified: from prior knowledge]` = recalled, not
> checked. Pricing and policy change often, so re-check every load-bearing number before you
> sign anything.

---

## 0. TL;DR verdict

| Phase | Recommended bank-data strategy | Feasibility |
|---|---|---|
| **Personal (you, your own accounts)** | **Enable Banking "restricted production"** (free, limited to accounts you link and whitelist yourself), plus **CSV/OFX/QIF file import** as a fallback that always works, plus **smart capture** (Android notification parsing, receipt OCR, manual quick-add). | **HIGH.** Live PSD2 data for your own accounts costs almost nothing. Enable Banking's review of unlicensed apps is the one gate. |
| **Product (paying users)** | **A commercial contract under a licensed provider's licence.** Candidates: Enable Banking (quote), Powens or Bridge (French ACPR payment institutions, strong FR coverage), Salt Edge Partner Program, Yapily/Tink/TrueLayer (higher entry cost). Do **not** get your own AISP licence at launch. | **MEDIUM.** Technically easy. The constraint is money and contracts: no provider publishes a self-serve price, and secondary sources suggest entry costs in the low hundreds of €/month plus per-connection fees. KYB on a company is required. |
| **Regulatory future** | FiDA is not adopted. PSD3/PSR are politically agreed but not yet in force. | Don't build on either. Design an adapter layer so the provider can be swapped. |

**Architecture advice that holds whichever path you pick:** put every data source behind one interface (`TransactionSource`: aggregator, file import, notification parser, OCR, manual). Normalise to one internal transaction model, with status `pending|booked`, raw description, cleaned merchant, MCC, category and source. Deduplicate across sources. That way the bank-sync decision stays a two-way door. [Inferred: design judgment from the provider volatility documented below, e.g. GoCardless closing its free tier]

---

## 1. Regulatory baseline (what you are allowed to do, and how often users re-consent)

### 1.1 PSD2 AIS: who may call bank APIs

- Under PSD2, only a registered/licensed **AISP** (or a payment institution with AIS permission) may access bank (ASPSP) interfaces. It needs an eIDAS QWAC/QSealC certificate. An app developer without a licence accesses data **through** a licensed provider, either as its customer (the provider is the regulated AISP that faces the user) or as its registered **agent**. [Unverified: from prior knowledge, standard PSD2 structure]
- For a **personal tool reading only your own accounts**, the provider's terms usually cover it (e.g. Enable Banking restricted mode, below). For a **product** you need a commercial contract with the provider, and your company goes through KYB. [Inferred: Enable Banking docs say production apps are "pending" until contractual formalities are cleared or own accounts are whitelisted, see §2.2]
- Getting your own AISP registration (in France, from the ACPR) means professional indemnity insurance, security policies and incident reporting. It is not proportionate for a solo developer at launch. [Unverified: from prior knowledge]

### 1.2 Consent renewal: 90 days became 180 days (2023 RTS change)

- The EBA's final report (5 April 2022) introduced a **mandatory** SCA exemption when the customer accesses account information **through an AISP**, and extended the SCA renewal period **from every 90 days to every 180 days**. [Verified: https://www.eba.europa.eu/eba-publishes-final-report-amendment-its-technical-standards-exemption-strong-customer]
- The amending act is **Commission Delegated Regulation (EU) 2022/2360**, applicable from **25 July 2023** (with a transitional carve-out for banks that already applied the old Art. 10 exemption). [Inferred: multiple search snippets quoting Art. 3 of 2022/2360 (lawplayer.com, financialinstitutionsnews.com, finreg360.com); EUR-Lex not opened]
- A bank developer portal confirms the 25.07.2023 go-live of the 90→180-day extension. [Verified: https://developer.unicredit.eu/newsroom/291-implementation-according-to-eu-reg-2022-2360-go-live-on-25-07-2023]
- **In practice the limit is set per bank.** Enable Banking's API exposes `maximum_consent_validity` per ASPSP. The Nordea example is 15 552 000 s, which is **180 days**, and `valid_until` cannot exceed now + that maximum. Some banks still return shorter maxima. [Verified: https://enablebanking.com/docs/api/reference/]
- **Product implication:** build a **re-consent reminder flow**: a notification about 7 days before `valid_until`, plus deep-link reconnection. This fits naturally in a "reminder" app. [Inferred: design judgment]

### 1.3 PSD3 / PSR (the PSD2 successor)

- Secondary sources report a political agreement on **27 Nov 2025** and compromise texts published around **23–24 Apr 2026**. Formal adoption and Official Journal publication were expected in H2 2026. Key AIS changes: a mandatory **permission dashboard** at banks (users can see and revoke TPP access) and SCA "on first access and every 180 days thereafter". [Inferred: search snippets from openbankingtracker.com/banq.ai PSD3 guides, mondaq, chambers.com; official texts not opened]
- Application would follow a transition period after publication (typically 18+ months), so PSD3/PSR will not change your 2026–2027 build. [Inferred: usual EU application lag; exact period not verified]

### 1.4 FiDA (Financial Data Access Regulation, the "open finance" layer beyond payment accounts)

- In Feb 2025 the Commission reportedly considered **withdrawing** FiDA. Its 2025 work programme then kept it among **pending** proposals rather than withdrawing it. [Inferred: search snippets from vixio.com, thepaypers.com, journaldunet.com]
- 2026: sources conflict. Some say trilogues were **paused since early 2026** over whether gatekeeper platforms (Amazon, Apple, Google, Meta) can be data users. One says the Council presidency circulated a non-paper on 6 Apr 2026. **No source found confirms a provisional agreement.** [Inferred: search snippets from fiskil.com, konsentus.com, sebastienrousseau.com (2026-07-31), financexmagazine.com; the European Parliament Legislative Train page returned 404]
- **Verdict:** FiDA is not law and won't apply before 2027–2028 even in the best case. It would extend data access to savings, investments, insurance and pensions, which matters for a future "net-worth" feature, not for MVP. [Inferred: from the above]

---

## 2. Aggregator-by-aggregator assessment

### 2.0 Comparison table

| Provider | Status (Nov 2026) | Free / entry pricing | Individual can sign up? | FR coverage | Regulatory model for you | Data enrichment |
|---|---|---|---|---|---|---|
| **GoCardless Bank Account Data** (ex-Nordigen) | **Closed to new accounts since July 2025.** Existing users keep working. | The old free tier (Nordigen: up to ~50 req/day, 90-day history) is gone for newcomers. No public price list. | **No** (new signups stopped) | Was broad EU incl. FR [Unverified: from prior knowledge] | n/a | Basic |
| **Enable Banking** (Finland) | Active; 2 500–2 700+ banks in ~30 countries | **Free "restricted production"** for own linked accounts. Commercial use priced by quote ("Get a Quote" since Mar–Apr 2026). | **Yes**, for personal restricted mode (subject to review) | Included in 30 EU countries [Inferred: homepage country count; FR bank list not opened] | Registered AISP (FIN-FSA). Product use needs a production agreement. | MCC on transactions; categories not documented |
| **Powens** (ex-Budget Insight, FR) | Active; 1 800+ banks in Europe | Quote-only (platform fee + usage). Free sandbox account. | Sandbox yes; production via sales | **Strong, French native** [Inferred: French ACPR institution, FR/ES/MX presence] | ACPR-authorised payment institution (CIB 16948) | Categorisation offered [Unverified: from prior knowledge] |
| **Bridge** (by Bankin', FR) | Active | Quote / demo only | Sales-led, business-targeted | **Strong, French native** [Inferred: ACPR-regulated, FR focus] | ACPR-authorised payment institution (entity Perspecteev) | Categorisation is Bankin's core competence [Unverified: from prior knowledge] |
| **Tink** (Visa) | Active | "Standard" and "Enterprise" plans, prices not public; prices shown are "applicable exclusively to existing customers" | Sales-led | Good EU incl. FR [Unverified: from prior knowledge] | Licensed (Sweden) [Unverified] | Enrichment/categorisation products [Unverified] |
| **TrueLayer** | Active | Pricing page reportedly 404s. Starter historically ~£150–300/mo (secondary). Free sandbox. | Sandbox yes; production via sales | EU coverage exists; FR depth not confirmed | Licensed (UK FCA, EEA entity in Ireland) [Inferred: search snippets] | Basic + classification [Unverified] |
| **Salt Edge** | Active; 5 000+ banks, 50+ countries, 2 500+ EU banks | Usage-based "from $500/month" (secondary snippet). Basic trial of 100 live connections reported. | Sales-led; "Get API keys" | Included in EU coverage | **AISP licensed under PSD2; Partner Program lets unlicensed businesses use its licence** | **Categorisation, merchant identification, insights** |
| **Yapily** | Active | Entry production ~£200–500/mo (secondary). Free sandbox. | Sales-led | UK+EU focus [Inferred: secondary] | Licensed [Unverified] | Limited [Unverified] |
| **Plaid (EU)** | Active; claims >95% bank coverage in 20 EU countries | Free "Launch" tier and custom Scale/Enterprise (secondary, Oct 2026). Free sandbox. | Self-serve sandbox; production review | FR announced 2019 (BNP Paribas, SG, La Banque Postale partnerships) | Licensed [Unverified] | Strong in US; EU enrichment unclear |
| **open-banking.io** (indie reseller) | Active | **€3/mo first account + €1/mo per extra account**, VAT incl., no card to sign up | **Yes**, individuals and businesses | Not stated | **Names no licence or regulator on its homepage.** It likely resells a licensed AISP. | Not stated |

Sources and grades for each cell are in the subsections.

### 2.1 GoCardless Bank Account Data (ex-Nordigen)

- "From July 2025 onwards, GoCardless has stopped accepting new Bank Account Data accounts … if you are an existing user, your account should continue to work." [Verified: https://actualbudget.org/docs/advanced/bank-sync/gocardless/] (this is a secondary source: Actual Budget's docs)
- The fetched `gocardless.com/bank-account-data/` URL served **Pay by Bank** marketing content (UK bank count, sign-up for payments), not a Bank Account Data product page. This is consistent with the product no longer being sold to newcomers. [Verified: https://gocardless.com/bank-account-data/ (observed content)]
- A 2026 indie comparison (written by an author working on a competing service) says the 10-connection free tier was removed during the GoCardless consolidation and that there is no public price list. [Verified: https://dev.to/johnfrandsen/the-cheapest-open-banking-apis-for-small-businesses-and-indie-builders-in-2026-5cab (secondary, conflicted author)]
- **Verdict: do not plan on GoCardless BAD.** It is the cautionary tale behind the "provider adapter" rule.

### 2.2 Enable Banking

- Registered AISP regulated by the Finnish FSA (FIN-FSA). Covers 2 700+ banks (2 500+ in places) in 30 European countries. No public pricing; "Get a Quote" onboarding (March 2026 changelog). [Verified: https://enablebanking.com/]
- Production apps "at first appear as pending". They activate once contractual formalities are cleared **or after you whitelist your own accounts** (linked accounts). [Verified: https://enablebanking.com/docs/api/reference/]
- Third-party integrations (Firefly III, Actual Budget) document the personal path: register a **Production** app, link your accounts in the Enable Banking portal, and only those accounts are returned. Actual Budget marks the integration experimental. [Verified: https://actualbudget.org/docs/advanced/bank-sync/enable-banking] (Firefly III page 403'd; its content is known only from search snippets: [Inferred])
- "Free restricted mode … access your own bank accounts without a paid subscription". Use of the Control Panel/API is free under the published ToS for sandbox and restricted production. [Inferred: search snippets from docs.firefly-iii.org and rfp.wiki]
- An Oct 2024 changelog reportedly says all production apps are reviewed and approved unless a licensed company registered them. Expect a manual review even for the personal path. [Inferred: search snippet from enablebanking.com blog changelog-october-2024]
- Data: transactions carry `status` (e.g. `BOOK`) and `merchant_category_code`. Consent validity is per ASPSP via `maximum_consent_validity`. The PDNG (pending) status value was not confirmed on the page read; it is a standard Berlin Group/ISO status and availability depends on the bank. [Verified for BOOK/MCC/consent: https://enablebanking.com/docs/api/reference/; pending: Unverified: from prior knowledge]
- **Verdict: best personal-phase choice**, and a credible product-phase candidate once you have a company and a quote.

### 2.3 Powens (ex-Budget Insight)

- "Powens SAS is a payment institution registered in France … authorized and regulated by the ACPR under CIB number 16948". Covers 1 800+ banks across Europe. Sandbox and "Create a free Account" available. No pricing on the site. [Verified: https://www.powens.com/]
- Pricing model: hybrid platform fee + usage (API calls / services), quote-based. [Inferred: search snippets from thepaypers.com, softwareadvice]
- **Verdict:** a top candidate for a **France-first product**, because FR coverage and FR banks' quirks are its home turf. Expect a sales process and a minimum commitment. [Inferred]

### 2.4 Bridge (powered by Bankin')

- Bridge is an "établissement de paiement agréé et contrôlé par l'ACPR". [Verified: https://bridgeapi.io/en/regulation-and-compliance]
- Its pricing page has no prices, only "Réserver une démo". It targets businesses (accounting software, e-commerce, fintech, real estate). [Verified: https://bridgeapi.io/pricing]
- The licensed entity is **Perspecteev** (ACPR 16918P), and the Bankin' consumer app acts as its registered agent (REGAFI 381188). [Inferred: search snippets from bridgeapi.io legal notice and Bankin' CGU]
- Note: Bankin' is also a **direct consumer competitor** (budgeting app), so a dependency on Bridge has a strategic angle. [Inferred: design judgment]

### 2.5 Tink (Visa)

- The pricing page lists Standard and Enterprise with no public numbers: "The prices listed on this page are applicable exclusively to our existing customers. New prospects … should contact our sales team." [Verified: https://tink.com/pricing/]
- Secondary: hard to get below a four-figure monthly commitment, and indie builders are often redirected to partners. [Verified: dev.to article above (secondary)]
- **Verdict:** a product-phase option only at scale.

### 2.6 TrueLayer

- No public price. A free sandbox exists; production needs a commercial conversation; historical starter ~£150–300/mo; `truelayer.com/pricing` 404s (Aug 2026). [Verified: dev.to article above (secondary)]
- **Verdict:** UK-strong. Check FR coverage before considering it.

### 2.7 Salt Edge

- "AISP licensed under PSD2". 5 000+ banks worldwide, 50+ countries, 2 500+ EU banks. **Partner Program**: businesses with "No PSD2 licence" can access EU bank data, with Salt Edge handling compliance; the Lending page says "even without having your own PSD2 licence and eIDAS certificates". Enrichment: categorisation, merchant identification, financial insights. [Verified: https://saltedge.com/products/spectre]
- Pricing: "usage-based fees starting from $500 per month" and a Basic free trial of 100 live connections (limited period). [Inferred: search snippet from saltedge.com and fitgap; the opened page showed no pricing]
- **Verdict:** the clearest documented **"use our licence" model** with built-in enrichment. Pricing is probably too high for the early product.

### 2.8 Yapily

- Entry production ~£200–500/mo depending on coverage; free sandbox; production via conversation. [Verified: dev.to article above (secondary)]. The primary Yapily site was not opened. [Unverified]

### 2.9 Plaid (EU)

- Claims >95% bank coverage across 20 countries, with a European Bank Coverage Explorer. Expanded to France, Spain and Ireland in 2019 with French bank partnerships. [Inferred: search snippets from plaid.com/en-eu/global, plaid.com/plaid-in-france-spain-and-ireland]
- Pricing: a free "Launch" tier, with Scale/Enterprise custom (third-party snapshot, Oct 2026). [Inferred: search snippet from toolradar.com]
- **Verdict:** a viable option for international expansion. US data-residency concerns are a GDPR talking point. [Inferred]

### 2.10 Indie / low-cost resellers

- **open-banking.io**: €3/mo + €1/mo per extra account, individuals welcome, PSD2-based, but **the homepage names no licence or regulator**. [Verified: https://open-banking.io/]. The only comparison found was written by someone who works on it (conflict of interest). **Fine for a toy; do due diligence (REGAFI / EBA register, underlying AISP) before anything with users.** [Inferred]
- **SimpleFIN Bridge**: $1.50/mo or $15/yr. [Verified: https://beta-bridge.simplefin.org/]. It is a read-only "RSS for financial information" protocol. [Verified: https://www.simplefin.org/]. Its institution coverage is **US-focused** (via a US aggregator); no EU/FR coverage is documented. [Unverified: from prior knowledge] → **Not usable for French banks.**

---

## 3. Data fields: what you actually get

| Field | PSD2 AIS reality | Notes for the app |
|---|---|---|
| Booked transactions | Always | 90+ days of history on first consent is common, then incremental. Some banks give more. [Unverified: from prior knowledge] |
| Pending transactions | **Bank-dependent.** Many FR banks expose card "à venir" (pending/deferred-debit) operations inconsistently. | Model `pending` explicitly and reconcile when booked (amount/date may change). [Inferred: Berlin Group status model; FR specifics Unverified] |
| Merchant name | Raw "libellé" (e.g. `CB CARREFOUR 12/10`); no clean merchant from the bank | You need your own **merchant-cleaning** step (regex rules + an ML/LLM fallback) or provider enrichment (Salt Edge, Bridge, Powens, Tink). [Verified for Salt Edge enrichment offer: saltedge.com/products/spectre; rest Inferred] |
| MCC | Sometimes (`merchant_category_code` in Enable Banking's model) | Good feature for auto-categorisation. [Verified: enablebanking.com/docs/api/reference/] |
| Category | **Not part of PSD2.** Comes from the provider's enrichment or from you | Build your own categoriser; it is a differentiator anyway. [Inferred] |
| Balance | Yes (booked/available, bank-dependent) | |
| Card deferred-debit ("débit différé") | Often a separate card account or a monthly lump sum | Important in France: link the card account or split the monthly debit. [Unverified: from prior knowledge] |

---

## 4. Non-aggregator data paths (always available, zero regulation)

### 4.1 File import: CSV / OFX / QIF / CAMT.053

- French banks commonly offer **CSV, OFX ("Money"), QIF ("Quicken") and PDF** downloads from online banking:
  - Boursobank: CSV export of a chosen period ("Exporter en Format CSV"), plus QIF and OFX from account transactions. [Verified: https://www.boursobank.com/aide-en-ligne/banque/mes-releves-et-documents-bancaires/question/comment-et-sous-quels-formats-puis-je-exporter-mes-mouvements-de-compte-et-de-carte-bancaire-26780]
  - Caisse d'Épargne / Banque Populaire (pro FAQ): csv, qif, ofx or pdf. [Inferred: search snippets from caisse-epargne.fr and banquepopulaire.fr FAQ pages]
  - BNP Paribas: PDF/CSV/OFX/QIF, ~90-day window for current operations. [Inferred: third-party guide snippet]
  - Others (e.g. Crédit Agricole, Société Générale): formats not confirmed. [Unverified]
- **CAMT.053** (ISO 20022 statement) is mostly offered to business/corporate accounts via EBICS. It is not a typical consumer export. [Unverified: from prior knowledge]
- **Implementation:** an OFX parser (SGML/XML variants), a QIF parser, and a **CSV mapper with saved per-bank profiles** (delimiter `;`, decimal comma, date `dd/mm/yyyy`, encoding often Windows-1252 or ISO-8859-1 in FR exports). Add duplicate detection via a hash of date+amount+normalised label, plus a FITID when it exists. [Inferred: format conventions; FR encoding detail Unverified]
- **Verdict:** a must-have in both phases. Zero cost, zero compliance, works for every bank, and it is the fallback when consent expires.

### 4.2 PDF statement parsing

- PDF is universal, so a later-phase "drop your relevé PDF" feature makes sense: text extraction + per-bank templates, with an LLM fallback. Medium effort, and layouts change. [Inferred: design judgment]

---

## 5. Smart capture (no bank API needed)

### 5.1 Android: NotificationListenerService on banking-app notifications

- **Technically feasible.** An app declares a service with `BIND_NOTIFICATION_LISTENER_SERVICE` and the user grants notification access in system settings. Every posted notification (package, title, text) then reaches the app, which can parse e.g. "Paiement de 12,50 € chez X" from bank or neobank apps that push real-time card alerts. [Inferred: search snippet citing developer.android.com NotificationListenerService reference; standard Android API]
- **Google Play policy: uncertain.** No Play policy text on notification access for finance apps was found in this session. Practitioner sources say it needs a genuine core-function justification, and a finance app's SMS auto-import was reportedly rejected under the SMS/Call-Log restricted-permission policy. [Inferred: search snippets from ptkd.com and dev.to; Unverified as policy]
- **Personal phase:** no issue. Sideload or use an internal/closed track. **Product phase:** budget for Play review risk. Keep the feature opt-in, process on-device only, filter to an allow-list of bank packages, and state the disclosure clearly. Several existing budgeting apps do this. [Unverified: from prior knowledge]
- **Quality:** an excellent "real-time" signal (the transaction appears seconds after payment) that complements the aggregator's delayed booked data. Notification formats change with app updates, so use per-app regex templates with a fallback to an LLM/heuristic parser and user confirmation. [Inferred]
- **SMS parsing:** less relevant in France (most banks push app notifications rather than SMS), and Play's SMS policy is strict. [Inferred: snippet above + Unverified]

### 5.2 iOS: Shortcuts "Transaction" automation and FinanceKit

- iOS has **no notification-listener equivalent**. Third-party apps cannot read other apps' notifications. [Unverified: from prior knowledge; well-known platform constraint]
- **Shortcuts personal automation "Transaction"** fires when an Apple Pay / Wallet card is tapped. It passes the merchant and amount to a shortcut, which can call your app's **App Intent** to log the expense. Apps such as TravelSpend and YNAB-community shortcuts use it. Reported flakiness: empty merchant / amount 0.0 occasionally (Apple DTS forum thread), broken variable mapping in some iOS versions, failure if Wallet has Mobile Data off, breakage after card changes. [Inferred: search snippets from developer.apple.com/forums/thread/797233, help.travel-spend.com, reddit r/shortcuts and r/ynab]
- The user must set up the automation manually (the app can provide a shared shortcut + guide). It only covers Apple Pay taps, not physical-card or online payments. [Inferred]
- **Apple FinanceKit** (read Wallet transactions/balances) is available **only in the US (Apple Card/Cash/Savings, iOS 17.4+) and the UK (open-banking, iOS 18.4+)**. It requires an entitlement per bundle ID and a Finance-category app distributed in the US/UK App Store. **Not available in France/EU.** [Verified: https://developer.apple.com/financekit]

### 5.3 Receipt OCR

- **On-device:** Google ML Kit Text Recognition v2 runs on Android and iOS and supports Latin script (plus Chinese, Devanagari, Japanese, Korean). [Verified: https://developers.google.com/ml-kit/vision/text-recognition/v2]. It is free and on-device. [Unverified: from prior knowledge; the page did not state it]. Apple's Vision framework (VNRecognizeTextRequest) is the iOS-native alternative. [Unverified: from prior knowledge]. Flutter plugins exist for ML Kit. [Unverified]
- **Cloud receipt APIs (structured fields such as total, date, merchant, VAT):** Mindee's pricing page lists Starter 44/mo and Pro 116/mo (annual options), Enterprise custom, with a 14-day free trial; 1 page = 1 credit; overage from $0.05. [Verified: https://www.mindee.com/pricing]. Older Mindee pages mention 250 free receipts/month. That conflicts with the current pricing page, so the free tier is **not** currently confirmed. [Inferred: search snippet from mindee.com/fr/product/receipt-ocr-api]. Alternatives: Veryfi, Taggun, Google Document AI, Azure Document Intelligence. [Unverified: pricing not checked]
- **LLM vision extraction** (send the receipt image to a multimodal model and ask for JSON fields) is a strong cheap option at personal scale. Weigh GDPR and data-processing terms in the product phase. [Inferred: design judgment]
- **Recommended:** on-device OCR first (free, private) + a rule-based field extractor (total = the largest amount near "TOTAL"/"TTC", date regex, merchant = first lines), then an optional cloud/LLM fallback. [Inferred]

### 5.4 Email receipt parsing

- Paths: **Gmail API** with OAuth (read-only scope with a query such as `category:purchases` or sender filters), **IMAP** for other providers, or a **forwarding address** (user forwards receipts to `receipts@yourapp`) using an inbound-mail webhook (e.g. Postmark/Mailgun/SES inbound). [Unverified: from prior knowledge]
- **Caution:** Gmail read scopes are **restricted scopes**. For a public app they require Google OAuth verification plus an annual third-party security assessment (CASA), which is costly. The forwarding-address model avoids this entirely. [Unverified: from prior knowledge; re-check Google's restricted-scope policy before the product phase]
- Structured data: many e-commerce emails embed schema.org `Order`/`Invoice` markup, and the rest can be parsed with templates or an LLM. [Unverified: from prior knowledge]
- **Personal phase:** an IMAP poll of your own mailbox with an app password, or a forwarding rule. **Product phase:** the forwarding address first; Gmail OAuth only once there is revenue.

### 5.5 Subscriptions and reminders from data

- Recurring-payment detection (same merchant, similar amount, regular period) over aggregator or imported data feeds the **reminder** feature (upcoming bills, trial endings, re-consent before `valid_until`). No extra data source is needed. [Inferred: design judgment]

---

## 6. Feasibility verdict

### Personal phase (you, own accounts, 0–6 months)

| Source | Effort (solo, expert) | Cost | Risk |
|---|---|---|---|
| CSV/OFX/QIF import with per-bank profiles | 3–5 days | €0 | Low |
| Enable Banking restricted production (own whitelisted accounts) | 4–7 days (JWT app auth, consent flow, sync job, re-consent reminder) | €0 (free tier) [Inferred: snippets] | Medium: app review; per-bank quirks; the free policy could change (cf. GoCardless) |
| Android notification listener (allow-listed bank apps) | 3–5 days + per-bank templates | €0 | Low for personal use (no Play review needed if sideloaded) |
| Receipt OCR on-device + field extractor | 4–6 days | €0 | Low |
| iOS Shortcuts Transaction intent | 1–2 days | €0 | Flaky per sources |
| Email receipts via forwarding/IMAP | 3–5 days | €0–few € | Low |

Effort figures are estimates. [Speculative: design judgment for an expert solo developer, assuming the core app already exists]

**Verdict: feasible, high confidence.** A "near-real-time + reconciled" pipeline (notification capture for instant entries, Enable Banking for authoritative booked data, reconciliation merging them) is genuinely innovative compared with typical FR budgeting apps, and costs nothing. [Inferred]

### Product phase (paying users, company)

1. **Create a company** (KYB prerequisite for any production aggregator contract). [Inferred: Enable docs "contractual formalities"; dev.to "signed contract + KYB" snippet]
2. **Shortlist and get quotes**: Enable Banking (EU-wide, engineering-friendly), Powens and Bridge (France-first, ACPR), and the Salt Edge Partner Program (explicit "use our licence" model + enrichment). Tink/TrueLayer/Yapily/Plaid if you expand beyond France. [Inferred]
3. **Budget assumption:** entry costs reported by secondary sources run **~£150–500/mo minimum** (TrueLayer, Yapily), **"from $500/mo"** (Salt Edge), four-figure (Tink). Enable/Powens/Bridge are quote-only. Assume **€200–600/mo + per-connected-account fees** until real quotes arrive. Price the premium tier so roughly 50–150 paying users cover it. [Inferred: secondary sources; Speculative for the break-even count]
4. **Offer free tiers without bank sync** (file import + notification capture + OCR), and gate live bank sync behind premium. This controls aggregator cost per user. [Speculative: business design]
5. **Play Store:** prepare a justification for notification-listener access, or ship it as a separate opt-in module or a non-Play build. **App Store:** Shortcuts integration only; FinanceKit is not available in FR. [Inferred]
6. **Compliance:** GDPR (financial data is sensitive in practice; do a DPIA; EU hosting), the provider's agent/customer terms, and consent-dashboard sync once PSD3/PSR apply. [Inferred]
7. **Don't seek your own AISP licence** until scale and revenue justify it. [Inferred]

**Verdict: feasible, medium confidence.** The blocker is commercial (quotes, minimums, KYB), not technical.

---

## 7. Open questions to settle before the product phase

- Exact Enable Banking commercial quote, and whether its restricted mode stays free. [Unverified]
- Powens and Bridge minimum commitments and per-connection pricing. [Unverified]
- Pending-transaction availability for the top FR banks via each provider. [Unverified]
- Current Google Play policy wording on notification access for finance apps. [Unverified]
- open-banking.io's underlying licensed AISP. [Unverified]
- PSD3/PSR Official Journal publication date, and FiDA trilogue outcome. [Inferred status only]

---

## 8. Sources

**Opened in this session (Verified):**
- Actual Budget — GoCardless bank sync docs: https://actualbudget.org/docs/advanced/bank-sync/gocardless/
- Actual Budget — Enable Banking bank sync docs: https://actualbudget.org/docs/advanced/bank-sync/enable-banking
- GoCardless page (served Pay by Bank content): https://gocardless.com/bank-account-data/
- Enable Banking homepage: https://enablebanking.com/
- Enable Banking API reference: https://enablebanking.com/docs/api/reference/
- Powens homepage: https://www.powens.com/
- Bridge pricing: https://bridgeapi.io/pricing
- Bridge regulation & compliance: https://bridgeapi.io/en/regulation-and-compliance
- Tink pricing: https://tink.com/pricing/
- Salt Edge Account Information: https://saltedge.com/products/spectre
- open-banking.io: https://open-banking.io/
- SimpleFIN: https://www.simplefin.org/ and https://beta-bridge.simplefin.org/
- dev.to (secondary, author works on open-banking.io), "The Cheapest Open Banking APIs … 2026": https://dev.to/johnfrandsen/the-cheapest-open-banking-apis-for-small-businesses-and-indie-builders-in-2026-5cab
- EBA final report on RTS amendment (90→180 days): https://www.eba.europa.eu/eba-publishes-final-report-amendment-its-technical-standards-exemption-strong-customer
- UniCredit developer portal, 2022/2360 go-live 25.07.2023: https://developer.unicredit.eu/newsroom/291-implementation-according-to-eu-reg-2022-2360-go-live-on-25-07-2023
- Apple FinanceKit: https://developer.apple.com/financekit
- Google ML Kit Text Recognition v2: https://developers.google.com/ml-kit/vision/text-recognition/v2
- Boursobank export formats FAQ: https://www.boursobank.com/aide-en-ligne/banque/mes-releves-et-documents-bancaires/question/comment-et-sous-quels-formats-puis-je-exporter-mes-mouvements-de-compte-et-de-carte-bancaire-26780
- Mindee pricing: https://www.mindee.com/pricing

**Seen only as search-result snippets (Inferred):**
- Firefly III Enable Banking tutorial (403 on fetch): https://docs.firefly-iii.org/tutorials/data-importer/eb/
- Enable Banking changelog Oct 2024: https://enablebanking.com/blog/2024/11/06/changelog-october-2024
- Regulation 2022/2360 text/summaries: https://lawplayer.com/eu/act/32022R2360, https://www.financialinstitutionsnews.com/2022/12/05/sca-amendments-to-apply-from-25-july-2023/
- PSD3/PSR status: https://www.openbankingtracker.com/guides/psd3-psr-readiness, https://www.mondaq.com/fintech/1780650/
- FiDA status: https://www.vixio.com/insights/pc-fida-lives-see-another-day-european-commission-backs-away, https://www.fiskil.com/open-finance-tracker/standard/fida, https://sebastienrousseau.com/2026-07-31-fida-open-finance-data-sharing-schemes-banks-2026/, https://www.konsentus.com/fida-readiness-series-how-fida-transforms-data-access/
- Bridge / Perspecteev licensing: https://bridgeapi.io/en/legal-notice/, https://web.bankin.com/cgu/mars+26/CGU+Bankin'+mars+2026.pdf
- Plaid EU: https://plaid.com/en-eu/global/, https://plaid.com/plaid-in-france-spain-and-ireland/, https://toolradar.com/compare/plaid-vs-truelayer
- Salt Edge pricing snippets: https://us.fitgap.com/products/saltedge-open-banking-gateway
- Powens pricing model: https://thepaypers.com/payments/company-information/powens
- iOS Shortcuts Transaction automation: https://developer.apple.com/forums/thread/797233, https://help.travel-spend.com/shortcuts--automation/ignQHsp85RQDsig2QwVcdX/set-up-apple-pay-automation/7tL8XfjBceg4D7mQeiSK2V
- Android notification-listener policy discussion: https://ptkd.com/journal/android-notification-listener-service-security, https://developer.android.com/reference/android/service/notification/NotificationListenerService
- French bank exports: https://www.caisse-epargne.fr/faq-entreprises/comptes-et-documents/consultation-des-comptes/pro-comment-telecharger-les-comptes-depuis-le-service-de-banque-en-ligne-de-mon-entite/, https://invoicedataextraction.com/blog/convertir-releve-bnp-paribas-pdf-excel-pennylane-cegid-ebp
- Mindee receipt page (250 free/month claim): https://mindee.com/fr/product/receipt-ocr-api
