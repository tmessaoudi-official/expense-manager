# 07 — France ↔ Tunisia and multi-currency (synthesis)

> **Where this file conflicts with 04-product.md or 06-roadmap.md, this file wins.**
>
> Synthesis of 07a-tnd-mechanics.md, 07b-tunisia-capture.md and 07c-multicurrency-design.md, with the
> **2026-11-01 fact-check verdicts applied**. Those verdicts override the three research files (see §0).
> Research date: 2026-11-01.
>
> **Persona.** A user with ties to France and Tunisia. They spend in EUR and TND, move money between
> accounts in the two currencies, and may later live or travel in other countries. The app must track,
> budget and help optimise spending across all of them.
>
> **Grading.** `[Verified: url (fact-check)]` = re-opened at the primary source during the 2026-11-01
> fact-check pass, the same convention 06-roadmap uses. `[Verified: url]` = opened by the research pass
> that produced 07a/b/c today and not contradicted by the fact-check. `[Inferred: basis]` = follows from
> verified evidence or from search summaries that were not opened. `[Unverified: …]` = recalled or not
> reachable. `[Speculative]` = a design or estimation judgment. **Every hour figure is [Speculative].**
> A grade that cites a §0 row (`[Verified: C3]`) or a sibling file points to the URL recorded there.
> This synthesis step made no new web fetches. Its grades rest on the same-day fact-check pass and on
> the 07a/b/c research passes.
>
> **Exchange-control and customs figures change by circular.** Only a dated official text earns
> `[Verified]` here. Everything else carries **"check before relying on it"**. The app stores such figures
> as editable, dated reference notes, never as hard-coded rules.
>
> **Neutrality.** The gap between an official rate and the rate a user actually obtains (card, transfer,
> cash) appears here only as a **budgeting fact the app must model**. This document gives no guidance on
> where or how to exchange money, and it does not name, rank or recommend any exchange or transfer channel.

---

## 0. Fact-check corrections applied (they override 07a/07b/07c)

| # | Topic | Research file said | Position used here | Grade |
|---|---|---|---|---|
| C1 | CLDR version | 07a pins CLDR 46.0.0, with HUF/IDR/COP at 2 digits ("cash 0"), and illustrates it with a HUF "fillér" example | The current CLDR is **49.0.0**. There **HUF, IDR and COP have `_digits 0`** (HUF cash rounding 5). PKR is unchanged (2, cash 0). The ISO-vs-display gap is **wider** than 07a said, which strengthens the "store the ISO exponent" rule. The fillér example is dropped | [Verified: https://registry.npmjs.org/cldr-core/latest → 49.0.0; cldr-core@49.0.0 supplemental/currencyData.json (fact-check)] |
| C2 | `intl` "din" symbol | "DZD and KWD also give din" | `simpleCurrencySymbol` is **"din"** for TND **and** for MKD, BHD, DZD, IQD, JOD, KWD, LYD and RSD. Five of these are 3-decimal dinars, so a "din" display could stand for any of them | [Verified: pub.dev intl 0.20.3 (2026-06-25), constants.dart (fact-check)] |
| C3 | Reg. (EU) 2019/518 | 07a: Unverified. 07c: "EU fee transparency covers EU currencies only" | **Art. 3a(1)** (charges stated as a percentage mark-up over the latest ECB euro reference rates, at an ATM or point of sale) is **not** limited to Union currencies. **Art. 3a(5)** (the electronic message the issuer sends after a payment) applies only to payment orders "denominated in any Union currency". So a TND payment does **not** trigger the 3a(5) message. How issuers apply 3a(1) to TND is unclear, because the ECB publishes no TND rate to measure a mark-up against. The codified Reg. (EU) 2021/1230 was not opened (EUR-Lex returned no body) | 3a(1)/3a(5) text [Verified: https://www.legislation.gov.uk/eur/2019/518/body/data.html (fact-check)]; issuer practice for TND [Unverified: no issuer disclosure or EBA/Commission guidance checked]; 2021/1230 numbering [Unverified] |
| C4 | French cash declaration | 07c: "into or out of the EU"; L.152-12 vs L.152-1 unresolved | **≥ €10,000** in cash when **entering or leaving France, including movements within the EU** (national rule, Art. L.152-12 CMF), on top of the EU external-border rule (Reg. (EU) 2018/1672). Sanctions under L.152-4, up to 50 % of the amount. Transfers through banks or money-transfer companies are not subject to the declaration. L.152-12 is the intra-EU basis, which resolves the discrepancy | [Verified: https://www.douane.gouv.fr/en/fiche/cash-reporting-obligation, "Updated on 04/05/2026" (fact-check)] |
| C5 | 07b TL;DR #3 | "EUR↔TND movements happen only through banks or money-transfer operators" | **Refuted, and dropped.** Circular 2026-10 also allows payment institutions to pay out funds coming from abroad (capped at the equivalent of 20,000 TND per operation, "dûment justifiées"). The app models **any** currency conversion between the user's own accounts as a two-leg transfer, and it does not name or rank channels | [Verified: https://businessnews.com.tn/wp-content/uploads/2026/09/yA1IYG-Cir_2026_10_fr.pdf, Circulaire n° 2026-10 of 25 Sept 2026, art. 1–2 (fact-check; press-hosted copy, not checked against bct.gov.tn)] |
| C6 | Circular 2026-10 figures | 07b listed them next to "exchange control" | Wallet tiers (1,500 / 5,000 / 20,000 TND), the 3,000 TND cash-transfer cap and the 20,000 TND cap on funds from abroad are **payment-institution operating limits**. They are **not** personal exchange-control allowances, and the app must not present them as such | [Verified: same PDF, art. 2 and 16 (fact-check)] |
| C7 | Travel allowance and dinar banknote ban | 07a §0 row 10 and §4.3: Verified | **6,000 DT/yr (3,000 DT for children under 10), no carry-over, and the ban on importing or exporting dinar notes and coins** come only from an **undated** BCT summary page whose newest citation is from 2016. The circulars themselves (2007-04, 2009-21, 2014-16) were not opened | [Verified: undated BCT page https://www.bct.gov.tn/bct/siteprod/page.jsp?id=67; circular not opened] **check before relying on it** |
| C8 | 5,000 DT re-export and reconversion | 07a §4.4 cites the undated page | These figures are verified against a **dated** text: circular 94-13, consolidated 2008 ("modifié par circulaire n°2007-13 du 25/04/2007"), Articles 7–9. Its boundary is **"supérieur ou égal à 5.000 DT"**, while the summary page says "supérieur". The two texts disagree on a value of exactly 5,000 | [Verified: https://www.bct.gov.tn/bct/siteprod/documents/cir9413.pdf, PDF dated 2008-04-28 (fact-check)] **check before relying on it** |
| C9 | 20,000 DT customs declaration | 07a: "contradicts each other and the 5,000 DT figure" | Most likely **a separate rule**: a customs declaration threshold reported "in force since March 2016", distinct from the BCT re-export and reconversion document threshold. It is probably current, but douane.gov.tn was unreachable (curl exit 7 on https and http) | [Unverified: press only — lapresse.tn 2026-04-28 and 2026-06-19, webdo.tn, allafrica 2026-05-29] **check before relying on it** |
| C10 | Advice-like lines | 07a §5.1 (a line contrasting card types by fee level), §4.4 (a reason for keeping exchange slips), §5.3 (provider-level transfer quotes from search snippets); 07c §6.3 (a line that read as a nudge to change amounts) | All four are **dropped**. Card fees: "fees vary by issuer; the app records the fee actually charged". Exchange slip: the receipt vault accepts it as a **document type**, with no statement of why. Transfer cost: only the World Bank RPW corridor average, and only once it has been opened (it returned HTTP 403). Batching: "shows how the effective cost varies with amount in the user's own history" | [Verified: fact-check read of 07a and 07c] |
| C11 | Flouci (07b §3.2) | "Verified" across the board | Verified: ~500,000 downloads, > 140,000 accounts, Shopify / WooCommerce / PrestaShop / Converty plugins, "facilitateur de paiement autorisé par la BCT", and a business API (QR, pre-auth, webhooks) with no documented end-user history API. **Inferred**: the Webank account link and BTE as the card BIN sponsor (their roles are unclear). **Unverified**: "virtual Visa" | [Verified: https://www.leconomistemaghrebin.com/2026/06/08/paiement-digital-flouci-premiere-tunisie/ , https://docs.flouci.com (fact-check)]; Webank/BTE [Inferred: snippets]; virtual Visa [Unverified] |
| C12 | Sensor Tower (07b TL;DR #9) | Wallet top; "the leader peaks at $16/week" | **YNAB is #1** (about $7/week average). The **$16 peak is Wallet's** (mid-October). Then Splitwise (peak $13), Mobills ($3–4/week), Money Manager (~$2/week). The data is Q4 2023, three years old: weak evidence of a tiny local market | [Verified: sensortower.com Q4 2023 TN budgeting blog post (fact-check)] |
| C13 | Cash in circulation | "about 30.2 bn TND, +16 %": Inferred | **30.217 bn TND at 31 Aug 2026** is Verified via a press relay of the BCT balance sheet. The **≈ +16 % year on year** stays Inferred (29.970 bn on 19 Aug 2026 against 25.864 bn a year earlier ≈ +15.9 %). The link to the cheque law comes from press only | [Verified: https://lapresse.tn/2026/09/16/les-chiffres-cles-du-bilan-de-la-bct-a-fin-aout-2026/ (fact-check)]; growth [Inferred: tunisienumerique snippet] |
| C14 | ML Kit scripts | Latin, Han, Devanagari, Japanese, Korean | Same, **plus a Cyrillic** script model. Still **no Arabic** | [Verified: https://developers.google.com/ml-kit/vision/text-recognition/v2/languages (fact-check)] |
| C15 | Stamp duty notation | "TND 1.500 / 2.000" | PwC's "TND 1,500 / 2,000" uses the millimes convention: **1.5 and 2 dinars**, and invoices under 50 TND stay at 1 TND. The legal basis is Loi n° 2025-17 du 12 déc. 2025 (LF 2026). The JORT was not opened | [Verified (secondary): https://taxsummaries.pwc.com/tunisia/corporate/other-taxes, last reviewed 29 June 2026 (fact-check)]; LF text [Unverified] |
| C16 | Frankfurter coverage | "106 providers, 228 currencies" | 106 providers. "228 currencies" is the homepage claim, but **`/v2/currencies` lists 170**. TND is in **17** provider lists: BAM, BCBO, BCT, BDI, BIS, BOA, CBKKW, CBO, CBU, HMRC, IMF, INFOREURO, JPC, LB, NBP, NBU, UST | [Verified: https://frankfurter.dev/ , https://api.frankfurter.dev/v2/providers , /v2/currencies (fact-check)] |
| C17 | "0.29793 (BCT TND→EUR)" (07c §4.2) | Presented as a BCT rate | **Derived** as 1 / 3.3565. The BCT publishes only TND per foreign unit. The rounding arithmetic in that example is correct | [Verified: cours.jsp EUR 3,3565 (fact-check); inverse = arithmetic] |
| C18 | ExchangeRate-API terms | "forbids redistribution and programmatic access products" | Confirmed, and the Terms page also says the free and paid plans are "suitable for either commercial or personal use". The "programmatic access" clause targets products that pass rate data on to others. Whether an app that only displays converted amounts on the device counts as "end-use" is **not settled by the text** | terms [Verified: https://www.exchangerate-api.com/terms (fact-check)]; end-use reading [Inferred] |
| C19 | Card-fee averages | 2.5 % + €0.30 per payment; 2.4 % + €3.10 per withdrawal | Confirmed, labelled as **June 2025 averages** (Panorabanques, published 10 July 2025, about 16 months old) | [Verified: https://www.lafinancepourtous.com/2025/07/10/carte-bancaire-combien-coute-un-paiement-ou-un-retrait-despeces-a-letranger/ (fact-check)] |

---

## 1. TL;DR — what changes for the product

| # | Change | Why | Grade |
|---|---|---|---|
| 1 | **Multi-currency moves into the lean daily driver.** The old 360 h lean cut (06 §2.4) dropped it, but a user who spends in EUR and TND cannot use a single-currency ledger. Revised lean: **≈ 280 / 475 / 820 h** (§9) | Persona requirement | [Speculative] |
| 2 | **Currency exponent becomes data, not code.** TND has 3 minor units (millimes). A seeded `Currency` table drives storage for every ISO currency (0, 2, 3 or 4 decimals) and is generated from the ISO list, not typed in by hand | Hard-coding `/100` silently corrupts every TND amount by a factor of 10 | [Verified: https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml, Pblshd 2026-09-17 (fact-check)] |
| 3 | **The ECB cannot be the only rate source.** It publishes no TND rate. EUR↔TND comes from the **BCT**, read on the device through **Frankfurter v2 with `providers=BCT` pinned**. The Frankfurter blend is not labelled "official" | ECB list: 29 currencies, no TND. Blend 3.3594 vs BCT 3.3565 on 29/10/2026 | [Verified: ECB eurofxref-daily.xml 2026-10-30; api.frankfurter.dev (fact-check)] |
| 4 | **Rates must not be redistributed from a project server for now.** BCT terms allow downloads "à des fins exclusivement personnelles". Re-serving or deriving commercial products needs prior BCT agreement. ExchangeRate-API forbids redistribution | Licence constraint on the v2 sync design | [Verified: https://www.bct.gov.tn/bct/siteprod/documents/termes.pdf (PDF created 2016-12-23) (fact-check)] |
| 5 | **A conversion between own accounts is one two-leg transfer** with the realised rate, any fee, and a computed **FX cost against a named reference rate**. EU rules give no per-payment conversion message for TND, so the app computes this itself | C3 | [Verified: 2019/518 Art. 3a(5) (fact-check)]; design [Speculative] |
| 6 | **Budgets live in their own currency.** A combined "life" view converts at a declared planning rate. Safe-to-spend is computed **per currency pocket**, so a TND shortfall cannot hide behind a healthy EUR balance | Daily spot conversion makes a TND budget "overspend" whenever the rate moves | [Inferred: 07a §6, 07c §4.5] |
| 7 | **Price-hike detection compares in the series' own currency**, so a rate move never fires a false "price increase" alert | 07c §4.6 | [Speculative] |
| 8 | **Country rule packs.** French tax and legal features (frais réels, the 10 % comparator, warranty presumptions, résiliation) evaluate only FR-scoped spending. The Tunisia pack ships empty by design | One-way door: a `jurisdiction` column added after French rules have tagged data needs a migration | [Speculative] |
| 9 | **Tunisian capture has no API path.** Manual entry plus cash accounts, Android notification and SMS parsing, PDF statement import and Latin-script OCR. Monthly statement reconciliation is the ground truth | No open-banking regulation and no API provision in circular 2026-10 | [Verified (secondary): https://www.fiskil.com/open-finance-tracker/tunisia (fact-check)]; circular grep [Verified (fact-check)] |
| 10 | **Exchange-control and customs figures ship as empty, dated reference notes**, filled only from dated official texts, with "check before relying on it" | They change by circular; several are only on an undated page | §7 |

---

## 2. Currency facts

| Fact | Value | App consequence | Grade |
|---|---|---|---|
| TND ISO code / numeric / minor units | **TND / 788 / 3** (1 dinar = 1,000 millimes) | `amount_minor` stores millimes: "14,500 DT" → `14500` | [Verified: SIX list-one.xml, Pblshd 2026-09-17 (fact-check)]; "millime" as the subunit name [Unverified: prior knowledge] |
| EUR | 978 / 2 | — | [Verified: same file] |
| Exponent counts (unique codes) | 0-decimal: **17**; 2-decimal: **139**; 3-decimal: **7**; 4-decimal: **2** (CLF, UYW); "N.A.": **13** (metals, SDR, test codes) | The exponent column takes 0, 2, 3 or 4. "N.A." codes are rejected as account currencies | [Verified: same file, parsed (fact-check)] |
| CLDR `fr-TN` | Symbol **"DT"**, decimal `,`, group **U+202F**, pattern `#,##0.00 ¤` with **U+00A0** before the symbol; TND digits 3 override the pattern's 2 → `14,500 DT`, `1 234,500 DT` | Format override for (fr-TN, TND) | [Verified: cldr-numbers-full@46.0.0 main/fr-TN, still "DT" in 49.0.0 (fact-check)] |
| CLDR `ar-TN` | Symbol **"د.ت.‏"** (ends with RLM), decimal `,`, group `.`, pattern starts with U+200F, Latin digits by default | Keep the RLM marks, or bidi moves the symbol to the wrong side | [Verified: cldr-numbers-full@46.0.0 main/ar-TN (fact-check)] |
| CLDR plain `ar` | Same symbol, but decimal `.` and group `,`: the **opposite** of ar-TN | Never fall back from ar-TN to ar for separators | [Verified: 07a, main/ar] |
| Flutter `intl` 0.20.3 | **No `fr_TN` or `ar_TN`** locale. `simpleCurrencySymbol('TND')` = "din", shared with 8 other codes (C2). `currencyFractionDigits` = 0 for IQD, HUF, IDR, COP, PKR | Do not take the storage exponent or the TND symbol from `intl` | [Verified: pub.dev intl 0.20.3 (fact-check)] |
| ISO vs display digits | IQD: ISO 3, CLDR 0, intl 0. HUF/IDR/COP: ISO 2, CLDR 49 0, intl 0. PKR: ISO 2, CLDR 2 (cash 0), intl 0 | **Storage uses ISO. Display uses CLDR 49.** The two are separate columns | [Verified: C1, C2] |
| Rate source for EUR↔TND | **BCT** "cours moyens" (TND per `unit` of foreign currency): interbank 4 dp, 20 currencies; change manuel 3 dp, 17 currencies; 3- and 6-month forwards for EUR and USD. 29/10/2026: EUR 3,3565 interbank, 3,329 change manuel; USD 3,0138 / 2,988 | `FxRate.rate_kind`, `unit`, `published_date` | [Verified: https://www.bct.gov.tn/bct/siteprod/cours.jsp, "Journée du 29/10/2026" (fact-check)] |
| BCT units | JPY per **1000**; DKK, NOK per **100**; DZD, SAR, MAD, SEK, CHF, AED, MRU, QAR per **10** | Divide by `unit`. Getting it wrong makes a JPY amount 1,000× too large | [Verified: same (fact-check)] |
| BCT access | No API. HTML page (latin-1), POST archive `cours_archiv.jsp`, an "xls" that is really an HTML table; `cours_xls.jsp` → 404. A non-business-day request returns the previous business day's table, with **that** day's date | Key a rate by the table's own date. Carry forward across gaps and flag it | [Verified: fact-check re-fetch]; "no API" [Inferred: absence only] |
| Same-day spread | Interbank vs change-manuel mean: EUR **−0.82 %**, USD **−0.86 %** | "The rate" is not one number. Each conversion records which kind it used | [Verified: per above; arithmetic] |
| ECB | 29 currencies on 2026-10-30, no TND, MAD or DZD. "Using the rates for transaction purposes is strongly discouraged" | ECB stays the source for EU and major currencies | [Verified: ECB eurofxref-daily.xml and reference-rates page (fact-check)] |
| Frankfurter v2 | Keyless, no quota, MIT code. `providers=BCT` returns BCT's figure; the default blend does not. BCT `terms_url` is null there | Pin a provider per pair | [Verified: C16; frankfurter.dev/license (fact-check)] |
| French card used outside the euro area | June 2025 averages: **2.5 % + €0.30 per payment**, **2.4 % + €3.10 per withdrawal**, on top of the card network rate on the **processing** date | Fee as its own line; `fx_rate_date` can differ from the purchase date | [Verified: C19] |

---

## 3. Data-model fixes (against 04-product §6)

### 3.a Seeded `Currency` table (all ISO currencies; every non-2-decimal one listed)

The seed is **generated at build time** from SIX `list-one.xml` (exponent, numeric) and from
`cldr-core` + `cldr-numbers-full` **pinned at 49.0.0** (display digits, cash rounding, symbols per
locale). A test fails if the generated exponents differ from the committed table, so a changed ISO list
becomes a reviewed diff rather than a silent change. [Speculative: design]

Symbol, symbol placement and decimal/grouping separators are **locale properties, not currency
properties**. The seed therefore has two tables:

| Table | Columns |
|---|---|
| **Currency** (new) | `code` CHAR(3) PK, `numeric` CHAR(3), `storage_exponent` SMALLINT (ISO; 0/2/3/4), `display_digits` SMALLINT (CLDR 49), `cash_digits`, `cash_rounding` (CLDR 49), `default_symbol` (CLDR root / `en`), `name_i18n`, `iso_list_version` ("2026-09-17"), `cldr_version` ("49.0.0"), `accountable` BOOL (false for "N.A." codes) |
| **CurrencyFormatOverride** (new) | `locale`, `currency`, `symbol`, `symbol_position` (before/after), `pattern`, `decimal_sep`, `group_sep`, `bidi_marks` — generated from CLDR for every (supported UI locale × seeded currency) pair, plus hand-added rows only where CLDR and `intl` disagree in a way that matters (TND in fr-TN and ar-TN first) |

Seed contents by exponent:

| `storage_exponent` | Codes | Grade |
|---|---|---|
| **0** (17) | BIF, CLP, DJF, GNF, ISK, JPY, KMF, KRW, PYG, RWF, UGX, UYI, VND, VUV, XAF, XOF, XPF | [Verified: list-one.xml 2026-09-17 (fact-check)] |
| **3** (7) | BHD, IQD, JOD, KWD, LYD, OMR, **TND** | [Verified: same; CcyMnrUnts=3 set is exactly these 7 (fact-check)] |
| **4** (2) | CLF, UYW | [Verified: same (fact-check)] |
| **2** (139) | everything else (EUR, USD, GBP, MAD, DZD, …) | [Verified: same] |
| **N.A.** (13) | XAG, XAU, XBA, XBB, XBC, XBD, XDR, XPD, XPT, XSU, XTS, XUA, XXX → `accountable = false` | [Verified: same] |

Rows whose display differs from storage, or whose `intl` defaults are unusable (these get explicit test cases):

| Code | storage (ISO) | display (CLDR 49) | `intl` 0.20.3 digits | `intl` simple symbol | Override needed | Grade |
|---|---|---|---|---|---|---|
| TND | 3 | 3 | 3 | "din" | fr-TN "DT" / ar-TN "د.ت.‏" with CLDR separators | [Verified: C1, C2, §2] |
| IQD | 3 | 0 | 0 | "din" | Show 0, **store 3** | [Verified: C1, C2] |
| BHD, JOD, KWD, LYD | 3 | 3 | 3 [Inferred: no divergence reported] | "din" | Symbol from CLDR so dinars are distinguishable | symbol [Verified: C2] |
| OMR | 3 | 3 | 3 [Inferred] | [Unverified] | Symbol from CLDR | [Verified: exponent] |
| HUF, IDR, COP | 2 | **0** | 0 | — | Show 0, **store 2** | [Verified: C1] |
| PKR | 2 | 2 (cash 0) | 0 | — | Store 2, display per CLDR | [Verified: C1, C2] |
| MKD, DZD, RSD | 2 | per CLDR | per intl | "din" | Symbol from CLDR | [Verified: C2] |

Symbols for the other codes come from the CLDR generator and are **not hand-verified** in this research.
[Inferred: generated data, pinned version]

### 3.b Currency on `RecurringSeries` and `Occurrence` (FX-neutral price-hike alerts)

- `RecurringSeries.charge_currency` (the currency the merchant bills in) plus `expected_amount_minor`
  and `baseline_amount_minor` **in that currency**. The paying account's currency can differ (a USD
  subscription paid from a EUR card; a TND phone plan paid from a TND account).
- `Occurrence.charged_amount_minor` + `charged_currency` (= the series' `charge_currency`), and separately
  `account_amount_minor` + `account_currency` from the matched transaction.
- **Price-hike rule:** `price_change_pct` is computed on `charged_amount_minor` against the baseline,
  **only in `charge_currency`**. A change in `account_amount_minor` with an unchanged charged amount is
  recorded as `fx_effect_pct` and shown as an "FX drift" note, never as a price increase.
- If the merchant charge currency is unknown (the statement only shows the EUR amount), the series is
  flagged `charge_currency_unknown`, and price-hike detection is suppressed until the user sets it, with a
  visible reason. [Speculative: design, from 07c §4.6]

### 3.c Exponent-aware amount parsing (receipts, notifications, quick add, import)

Order of operations for every parser (quick add, NL entry, notification/SMS templates, OCR post-processor,
CSV/PDF import): [Inferred: from the ISO exponent and the CLDR separators, §2]

1. **Resolve the currency first**: from the account, the sender/package template, a token (`DT`, `TND`,
   `د.ت`, `millimes`, `مليم`, `€`, `EUR`), or, last, the user's selected current country. Never from the
   phone locale alone.
2. **Then the separators**, using the currency's exponent and the source's locale. For an exponent-3
   currency, a single separator followed by exactly 3 trailing digits is the **decimal** separator in a
   fr-TN / ar-TN / Tunisian-merchant context: `14,500 DT`, `14.500 DT` and `14 DT 500` all mean
   `14500` millimes. For EUR the same `14,500` shape in an English-style source is a thousands group.
3. Accept an explicit **millimes** unit (`500 millimes` → `500`) and never multiply by 1,000 twice.
4. Arabizi words (`9ahwa 2.500`) are tokenised so that a digit inside a word is a letter, not an amount.
   [Speculative]
5. When the reading is ambiguous, **echo the parse before saving**: "14,500 DT = 14 dinars 500 millimes".
6. Parsers return `(amount_minor, currency, confidence)`. They never return a float.

Test corpus (Phase 0 money suite): `1.500`, `1,500 DT`, `1 500,000`, `15 DT 500`, `د.ت 12,350`,
`14,500` with (TND, fr-TN) → 14500 and with (USD, en-US) → 1450000. Real-receipt strings are added after
the sample-collection step (§6). Receipt strings beyond these shapes are [Unverified: prior knowledge —
collect samples].

### 3.d Conversion between own accounts = one transfer with realised rate and visible cost

```
Leg A (EUR account):  −500,00 EUR                       transfer_peer_id = B
Leg B (TND account):  +1 650,000 TND (amount actually received)
Fee (optional):       −x EUR or −y TND, category "FX & fees", linked to the transfer
Conversion row:       realised_rate = 3.300000 (= 1 650,000 / 500,00, derived, stored at high precision)
                      reference_rate_id → FxRate(BCT, interbank, EUR/TND, published 29/10/2026, 3.3565)
FX cost (computed):   reference amount − received = 1 678,250 − 1 650,000 = 28,250 TND
                      (+ fee), shown converted to the reporting currency
```
*Synthetic numbers that only show the shape of the record. They are not a quote from any provider.*

- The **realised amounts are facts**; the rate is derived from them and never used to recompute them.
- The FX cost is **computed**, not stored, and shown in an "FX & fees" view. Leg B's category is
  "transfer", so the cost is not double-counted; the user may materialise it as a split.
- A card payment in a foreign currency is `fx_status = estimated` at entry and `realised` when the
  statement or import arrives. The difference goes to FX cost, not back into the category.
- An ATM withdrawal abroad is a transfer from the card account to a **cash account** in that currency. Cash
  spending is then recorded with no conversion, and the conversion cost is captured once.
- If no rate exists for a date, the amount is `unconverted`: left out of home-currency totals with a visible
  badge, queued for re-conversion. **Never 1:1** (the Firefly III fallback). [Verified: Firefly III
  exchange-rates doc (fact-check)]

### 3.e Rounding rules per exponent

- Conversion on integer minor units with decimal arithmetic:
  `target_minor = round_half_even(source_minor × rate × 10^(e_target − e_source))`, **once per
  conversion**, never per intermediate step. [Speculative: standard practice]
- Example: 12,50 EUR × 3.3565 = 41.95625 TND → **41.956 TND** (`41956`). Back: 41.956 × 0.29793 (**derived**
  from BCT 3.3565, C17) = 12.49995 → **12.50 EUR**. [Verified: arithmetic]
- Booked amounts take the statement's value, never a recomputed one.
- Splits and shares use the **largest-remainder** method in the transaction's own currency: 10,000 DT in
  three → 3,334 + 3,333 + 3,333 DT. The existing "sum of splits = total" DB check works at any exponent.
  Conversion to the home currency happens after splitting.
- Display rounding uses `display_digits`; cash rounding uses `cash_rounding` and applies only to a cash
  account's suggested amounts, never to stored values.
- If ISO ever changes an exponent, existing rows migrate under a `(currency, iso_list_version)` key.
  [Unverified: past ISO changes not checked]

### Updated entity rows (replace or extend 04 §6.1)

| Entity | Key fields (new or changed in **bold**) | Notes |
|---|---|---|
| **Currency** (new) | **code, numeric, storage_exponent, display_digits, cash_digits, cash_rounding, default_symbol, name_i18n, iso_list_version, cldr_version, accountable** | §3.a; generated seed |
| **CurrencyFormatOverride** (new) | **locale, currency, symbol, symbol_position, pattern, decimal_sep, group_sep, bidi_marks** | §3.a |
| **User** | id, locale, home_currency, **reporting_currency_default, secondary_currencies[]**, digits_style, week_start, **fx_residency[] (optional, user-declared per jurisdiction)** | `fx_residency` only selects which reference notes to show. It never blocks or advises |
| **Account** | id, household_id, owner_member_id, name, type (cash/current/savings/card/loan/wallet), currency → **FK Currency**, opening_balance_minor, is_on_budget, institution_ref, **jurisdiction (ISO 3166-1), convertibility (convertible / non_convertible / blocked / unknown; user-set)**, archived | A non-convertible balance is "local only" money (§5) |
| **Transaction** | id, account_id, date, booked_at, amount_minor, currency, original_amount_minor, original_currency, fx_rate (realised/implied), fx_source (statement/network/bct_interbank/bct_cash/ecb/derived/user), **fx_status (none/estimated/realised), fx_rate_date, reference_rate_id → FxRate, fee_minor, fee_currency (or fee_txn_id), jurisdiction_override, balance_after_minor, external_ref**, payee_id, …, source (manual/import/ocr/voice/bank/**notification/sms/pdf_statement**), import_hash, recurring_series_id, transfer_peer_id | `balance_after_minor` and `external_ref` come from the post-transaction message content that circular 2026-10 art. 38(b) mandates (amount, fees, balance, reference) |
| **Conversion** (new, 1 per cross-currency transfer) | **id, leg_a_txn_id, leg_b_txn_id, realised_rate (DECIMAL 18,10), reference_rate_id, fee_txn_id, method_label (user-defined, free text)** | FX cost computed, not stored. `method_label` has no presets (§8) |
| **FxRate** | **date, published_date, base, quote, rate (DECIMAL 18,10), unit (1/10/100/1000), provider (BCT/ECB/FRANKFURTER_BLEND/ERA/USER), rate_kind (reference_interbank/reference_cash/forward_3m/forward_6m/indicative/derived/user), fetched_at, carried_forward**; unique (date, base, quote, provider, rate_kind) | Device-local. Not bundled, not re-served (§4) |
| **Budget** | id, household_id, mode, period, start_day, currency, **fx_policy (per_line_currency / account_amount / planning_rate), planning_rate_id, scope_jurisdiction** | §5 |
| **BudgetLine** | budget_id, category_id, period_start, limit_or_assigned_minor (in the line's currency), rollover_policy, **currency** | One currency per line |
| **EnvelopeConversion** (new event) | **from_line, to_line, from_minor, to_minor, rate, rate_kind** | Moves between a EUR and a TND envelope |
| **RecurringSeries** | …, **charge_currency**, expected_amount_minor and baseline_amount_minor **in charge_currency**, **charge_currency_unknown**, **funding_note** | §3.b |
| **Occurrence** | series_id, due_on, expected_minor, **charged_amount_minor, charged_currency, account_amount_minor, account_currency**, matched_transaction_id, state, price_change_pct (**charge currency only**), **fx_effect_pct** | §3.b |
| **Receipt** | …, `ocr_fields` adds **stamp_duty_minor, vat_lines[]**; `purpose[]` adds **exchange_slip** as a plain document type | No rationale is attached to the document type (C10) |
| **ParameterTable** | key, **country**, valid_from, valid_to, value, **unit**, source_url, **source_date, last_checked** | `tn.*`, `eu.*`, `fr.*` keys |
| **RulePack** (new) | **country, version, valid_from, source_url, rules[]** | §5; TN v1 empty |
| **ReferenceNote** (new) | **jurisdiction, topic, value (nullable), unit, source_text, source_url, source_date, grade, last_checked** | §7; shipped empty or "check before relying on it" |
| **Settlement** | …, currency, **may differ from the expense currency, with a stored rate** | 07c §5 |

---

## 4. Rate source recommendation

| Pair / need | Source | Licence terms for in-app use | Grade |
|---|---|---|---|
| **EUR↔TND (and other currencies vs TND)** | **BCT** "cours moyens", fetched **on the device** through Frankfurter `providers=BCT` (or the BCT page directly as a fallback adapter) | BCT: downloads for "fins exclusivement personnelles"; copying to another site or network, or deriving commercial products, needs prior BCT agreement; cite the site as the source. On-device fetch for the user's own budgeting, with "Source: BCT, cours moyen interbancaire du dd/mm/yyyy" and a link, is the defensible default. **Bundling BCT rates, re-serving them from a sync server, or putting them behind a paid tier needs BCT permission first** (contact bct@bct.gov.tn). This is not legal advice (see 05-compliance) | terms [Verified: termes.pdf (fact-check)]; consequences [Inferred] |
| **EUR↔EU and major currencies** | **ECB** through Frankfurter `providers=ECB` | Free reuse if reproduced accurately, the ECB cited, and any modification (such as a derived cross rate) stated explicitly. "For information purposes"; not for transactions | [Verified: https://www.ecb.europa.eu/services/disclaimer/html/index.en.html (fact-check)] |
| **Currencies neither covers** | Frankfurter blend, labelled **"blended (indicative)"** | Frankfurter code MIT; data "public data, shared for information", no warranty; provider terms apply; "Blended rates are Frankfurter's own calculation and free to use … not an official rate from any institution" | [Verified: https://frankfurter.dev/license/ (fact-check)] |
| Second fallback, on-device only | ExchangeRate-API open endpoint | Attribution link required; caching allowed "for customer end-use only"; no redistribution; free plans can be discontinued without notice; commercial use allowed on the free plan. **Never** in a seed, a server proxy or a sync payload | [Verified: C18]; end-use reading [Inferred] |
| Ruled out | Fixer free ("Non-Commercial Use"), currencyapi free ("private use only"), exchangerate.host free (100 req/month) | Terms or quota incompatible with a published app | [Verified: fixer.io/product, currencyapi.com/pricing, exchangerate.host (fact-check)] |
| **The rate the user actually got** | The statement, the card network line, or the amount received | This is the fact; reference rates are only the yardstick | [Inferred: 07a §5] |

**Offline caching** [Speculative: design, 07c §1.3]
- Every conversion reads the **local `FxRate` table only**. The network fills the table and is never
  consulted at conversion time.
- One fetch per day per pinned provider, after publication (ECB about 16:00 CET; BCT per Tunisian working day).
- Missing day: use the latest prior **published** date, show its age ("rate of 29/10, 3 days old"), set
  `carried_forward`. No rate at all: `unconverted` badge, excluded from totals, queued.
- History: backfill rates for imported transaction dates with a time-series query, cached forever.
- First run offline: the user enters a rate, or the amount stays unconverted. No bundled BCT or ERA seed.
  An ECB-only dated seed is allowed under the ECB terms [Inferred].
- Provider death: `provider` on every row, swappable adapters (Frankfurter, direct BCT page, direct ECB XML),
  and a watch on ERA's `time_eol` field.
- Server role in v2: a server fetcher may sync **ECB** rows. BCT rows stay device-fetched until BCT reuse
  terms are settled (open question Q1 in 07c).

**Manual override**
- Any transaction or conversion accepts a user rate or, better, the realised target amount
  (`fx_source = user`). A user value always wins over a computed one (Firefly III's
  `foreign_amount` rule). [Verified: Firefly III exchange-rates doc (fact-check)]
- A budget's planning rate is a user-declared `FxRate(provider = USER, rate_kind = user)`.

---

## 5. Home currency and two-country budgeting

### 5.1 Three currency roles

| Role | Stored in | Meaning |
|---|---|---|
| Transaction (merchant) currency | `original_currency`, `original_amount_minor` | What the merchant charged |
| Account currency | `Account.currency` → `amount_minor` | What moved on the account |
| Reporting currency | `User.reporting_currency_default`, **switchable per view** | Totals, charts, net worth, converted at report time from cached rates and **never written back** |

Switching the reporting currency per view ("my month in TND", "my year in EUR") without touching stored
data is something Firefly III cannot do: it converts only to its single primary currency. [Verified:
Firefly III exchange-rates doc (fact-check)]; differentiator [Speculative]

### 5.2 Budgets per currency vs converted [Speculative unless graded]

- **Default `fx_policy = per_line_currency`.** Each budget line is in one currency: "France monthly" in
  EUR, "Tunisia monthly" in TND.
- **TND spending on a EUR card counts at the account amount** (what left the EUR account), so the budget
  matches the bank (`fx_policy = account_amount` for that line).
- **Combined "life" view** rolls up all lines into the reporting currency at a **declared planning rate**
  (default: the period-average reference rate, or a user rate), and labels the basis. Converting daily at
  spot would make a TND budget "overspend" whenever the rate moves: noise, not behaviour. [Inferred: 07a §6]
- Moving money between a EUR envelope and a TND envelope is an explicit `EnvelopeConversion` event.
- YNAB's one-plan-per-currency model is an **option**, not the default: it hides the combined picture.
  [Verified: support.ynab.com multi-currency guide (07c)]
- Trip budgets (04 #43) are a special case of a currency- and country-scoped budget.

### 5.3 Safe-to-spend across EUR and TND

Computed **per currency pocket** `c`:

```
STS_c = Σ spendable balances of on-budget accounts in c
      − upcoming Occurrences charged to accounts in c before the next income in c
      − goal contributions funded from c
      − buffer_c
```

- A **non-convertible or blocked** TND balance (`Account.convertibility`) counts only in the TND pocket,
  shown as "TND (local only)". It never feeds a EUR safe-to-spend figure or a EUR goal. [Inferred: 07a §4.2]
- The widget shows the pocket for the user's **current country**, chosen manually or from the last
  transaction's jurisdiction, never from location tracking (06 §0, CNIL geolocation note).
- A **shortfall in any pocket is flagged** even when the combined figure is positive.
- The combined figure converts pockets at the planning rate, and is labelled "combined, at <rate basis>".
  Covering a TND shortfall from a EUR surplus is shown as "needs a conversion", priced at the user's own
  median realised cost from their history. The app does not suggest how or where. [Speculative]

### 5.4 Per-country rule packs [Speculative: design, 07c §4.7]

- `Account.jurisdiction` defaults to the institution's country. A per-transaction `jurisdiction_override`
  covers the case of a French card used in Tunisia (account FR, place of spending TN).
- **Binding rule:** a tax or legal feature evaluates only transactions whose jurisdiction matches its pack.
  For deductible tagging that is the taxpayer profile (`TaxProfile.country`); for consumer-law features
  (warranty presumption, return windows, résiliation timing) it is the place of purchase.
- So **French features apply only to FR-scoped spending**: frais réels and the 10 % comparator, km log,
  notes de frais, warranty clocks, "résiliation 3 clics" helpers. Tunisian spending never feeds the French
  comparator unless the user explicitly tags it as a professional expense under the French profile.
- The **Tunisia pack v1 is empty by design**: no tax features, no invented rules. Its rows (VAT 19/13/7 %,
  stamp duty, any reference notes) arrive later from official texts, each with `source_url` and date.
  [Verified (secondary) for VAT and stamp duty values: C15]
- The `jurisdiction` column is a **one-way door**: it belongs in Phase 0 domain model v1, before any French
  rule has tagged data.

---

## 6. Tunisia capture plan (no PSD2)

| Rank | Channel | Works? | Limits | Grade |
|---|---|---|---|---|
| 1 | **Manual quick add + cash accounts** (millime chips 0.500 / 1 / 2 / 5 …) | Yes, for everything, including the large cash share (30.217 bn TND in circulation at 31 Aug 2026) | Discipline-dependent | cash figure [Verified: C13]; chips [Speculative] |
| 2 | **Android notification listener** on allow-listed bank and wallet apps and on the default SMS app (which shows bank SMS alerts without `READ_SMS`) | Yes, for new alerts | Android 15+ **redacts OTP-detected notifications** for all apps, and Tunisian mobile payments are validated by SMS codes, so the OTP message is invisible; parse the post-transaction confirmation (art. 38(b) content). Threshold alerts (UIB MESSAG'EK: ≥ 500 TND, 2015 data) miss small payments. Play review risk as in 06 R4 | A15 [Verified: https://developer.android.com/about/versions/15/behavior-changes-all (fact-check)]; UIB [Verified: https://www.leconomistemaghrebin.com/2015/12/21/uib-service-gratuit-information-sms/ (fact-check)]; TUNPAY / SMS codes [Verified: webdo.tn 2026-05-06 (fact-check), press]; whether confirmations quoting a reference are also redacted [Unverified: device test needed] |
| 3 | `READ_SMS` back-fill | Possible | Play's SMS exception may be temporary and requires "no alternative"; 06 R4 says never use SMS permissions. **Not planned** | [Inferred: Play policy per 00-factcheck] |
| 4 | **PDF statement import** (text-layer parser per bank) | Likely | No format is mandated (art. 38(c) "par tout moyen approprié"). Statement downloads are mentioned for some banks, in an unstated format. Scanned PDFs need OCR. Layouts change | art. 38(c) [Verified: circular PDF (fact-check)]; downloads [Inferred: snippets] |
| 5 | **Receipt OCR** with ML Kit | Latin-script receipts | **No Arabic** in ML Kit. Stamp duty is a tax component of the total, not a line. Three VAT rates | [Verified: C14, C15] |
| 6 | CSV/OFX import | Only if a bank offers it | **No Tunisian bank export found** (absence of evidence, not evidence of absence) | [Unverified: two searches, no bank FAQ opened] |
| — | Open banking / aggregator | **No** | No regulation, no API provision in circular 2026-10. Payment institutions are TND-only and domestic (art. 1). Flouci's API is business-oriented with no end-user history | [Verified (secondary): Fiskil (fact-check); circular (fact-check); C11] |
| — | iOS automatic capture | **No** | No listener, no SMS access, **no Apple Pay in Tunisia**, so the Shortcuts "Transaction" trigger cannot fire for Tunisian cards. iOS = manual, OCR, PDF import, share sheet | [Verified: https://support.apple.com/en-us/102775 (fact-check)] |

**Design rules** [Inferred: from the coverage limits above]
- The **monthly statement reconciliation** is the ground truth. Notifications give speed, the duplicate
  guard merges the two (using `external_ref` and `balance_after_minor` where present), and cash closes the gap.
- Show **feed coverage** per account (the share of the statement total the live feed caught), so the user
  knows when to reconcile.
- **ATM withdrawal = transfer bank → cash account**, never an expense. Wallet top-ups are transfers too.
  A periodic "count your cash" prompt books unexplained differences to an "untracked cash" line.
- Parse only **allow-listed senders and packages**, never act on links in SMS. [Unverified: prior knowledge]
- Capture templates per (country, sender/package) ship as **downloadable data**, not code.
- The account → currency mapping comes from the sender/app, never from the phone locale (dual-SIM users).
- **Start with a sample-collection step**: redacted screenshots and PDFs from the user's own apps, kept out
  of the public repo, before any Tunisian template or PDF parser is written. No sample of any Tunisian
  bank's SMS, push or statement layout was found. [Unverified]
- Arabic-script receipts: manual entry, or a cloud OCR with explicit consent. A **20-receipt Tunisian
  sample benchmark** decides whether Tesseract `ara` is worth adding. [Speculative]
- Utility bills (STEG, SONEDE SMS bills) can seed bill reminders; periodicity not verified. [Inferred: snippets]

---

## 7. Exchange-control and customs facts — check before relying on it

These are shown in the app, if at all, as **ReferenceNote rows with their source text and date**, never as
rules. The app does not block, warn about legality or advise on any of them. Where `fx_residency` is
declared, it only selects which notes are shown.

| Topic | Fact | Source and date | Grade |
|---|---|---|---|
| Dinar convertibility | Convertible for **current** transactions (IMF Art. VIII accepted 6 Jan 1993). Capital movements and flows abroad go through the BCT or authorised intermediaries | page.jsp?id=67 (undated; cites loi 76-18, décret 77-608) | [Verified: undated BCT page] **check before relying on it** |
| Dinar as unit of account between residents | "Le dinar doit être à la fois une monnaie de compte et une monnaie de règlement" | BCT FAQ, Jan 2019 (PDF created 2019-02-05), Q3 | [Verified: https://www.bct.gov.tn/bct/siteprod/documents/Document_20190211_fr_002.pdf (fact-check)] **check before relying on it** |
| Non-resident Tunisian | Domiciled abroad **more than 2 years AND** centre of activity abroad | Avis de change n°3 of 5 Oct 1982, via page.jsp?id=67 and the Jan 2019 FAQ | [Verified: FAQ Jan 2019 (dated) and undated page (fact-check)] **check before relying on it** |
| Account types | Comptes étrangers en devises ou en dinars convertibles (non-residents), comptes intérieurs, PPR accounts (circ. 2017-04 of 23 June 2017: funds "ne peuvent, en aucun cas, alimenter" accounts abroad), allocation-touristique convertible-dinar account, comptes spéciaux, prestataires de services, comptes capital (Avis n°5) | page.jsp?id=67 (undated) + FAQ Jan 2019 | [Verified: undated page; PPR via dated FAQ] **check before relying on it**. App use: only the user-set `convertibility` flag |
| Resident travel allowance | **6,000 DT/yr; 3,000 DT for children under 10; no carry-over** | Undated BCT summary page, newest citation 2016 (circ. 2007-04, 2009-21, 2014-16 not opened) | [Verified: undated BCT page; circular not opened] **check before relying on it** |
| Dinar notes and coins | Import and export **prohibited** (except under bilateral agreements, e.g. with the Central Bank of Libya, 18 Feb 2007) | Undated BCT page citing décret 77-608 as amended by 2007-394 | [Verified: undated BCT page; décret not opened] **check before relying on it**. App consequence: a TND cash account is local to Tunisia; a leftover balance waits for the next stay and is never modelled as a transfer to France |
| Non-resident re-export / reconversion | Documentary thresholds at **5,000 DT**; circular 94-13 says **"≥ 5,000 DT"**, the summary page says "supérieur" | Circ. 94-13 of 7 Sep 1994, consolidated 2008, as amended by 2007-13 of 25 Apr 2007, Articles 7–9 | [Verified: https://www.bct.gov.tn/bct/siteprod/documents/cir9413.pdf, PDF dated 2008-04-28 (fact-check)] **check before relying on it** |
| Tunisian customs declaration | **20,000 DT** reported for a customs currency declaration, "in force since March 2016", a separate rule from the 5,000 DT BCT documents. Reports that Tunisians resident abroad have no threshold, and a "€6,000" figure, also circulate | Press 2026 (lapresse.tn 2026-04-28 and 2026-06-19, webdo.tn, allafrica 2026-05-29); douane.gov.tn unreachable | [Unverified] **check before relying on it** |
| French / EU cash declaration | **≥ €10,000** in cash entering or leaving France, **including within the EU** (L.152-12 CMF) and at the EU external border (Reg. 2018/1672). Bank and money-transfer transfers are not covered | douane.gouv.fr, updated 04/05/2026 | [Verified: C4] **check before relying on it** |
| Payment-institution caps (not allowances) | Wallet balances 1,500 / 5,000 / 20,000 TND; cash transfers ≤ 3,000 TND per operation; funds from abroad ≤ 20,000 TND equivalent per operation | Circular 2026-10 of 25 Sept 2026, art. 2 and 16 | [Verified: press-hosted PDF (fact-check); not checked against bct.gov.tn or the 28 Sept corrected version] **check before relying on it** |
| EU card conversion transparency | Art. 3a(1) mark-up over the ECB rate (not limited to Union currencies); Art. 3a(5) post-payment message only for Union currencies, so none for TND | Reg. 2019/518 as adopted | [Verified: C3]; codified 2021/1230 [Unverified] |

**What the app may do with these** [Speculative]: an optional, user-created "annual allowance used" envelope
in TND whose limit the user types; an informational link to the customs page when a planned trip's
cash-account balances exceed a user-visible `eu.cash_declaration.threshold` parameter. No figure is
asserted as current law, and every displayed figure shows its source and date.

---

## 8. Optimisation levers for a two-currency life [Speculative]

All of these show the user's **own** data back to them. None names, ranks or recommends a provider, venue
or channel, and none predicts rates.

1. **True cost on every conversion and foreign card payment**: realised vs the named reference rate, as an
   amount and a %, plus fees; an "FX & fees" budgetable category ("FX cost this year: …").
2. **The user's own methods compared**: past conversions grouped by the `method_label` the user typed, with
   the median effective cost per label. No preset labels ship.
3. **Cost vs amount**: shows how the effective cost varies with amount in the user's own recorded history.
4. **Descriptive timing**: the reference rate over the user's conversion dates, and the estimate-vs-posted gap.
   No "convert now" signal.
5. **Per-pocket safe-to-spend** (§5.3), so a TND shortfall is visible.
6. **Foreign-currency subscriptions list**: series whose charge currency differs from the paying account's,
   with the year's accumulated FX cost per item.
7. **Stay summaries**: per-country spend in both currencies, cost per day, FX-cost share.
8. **Reconciliation nudges**: "3 TND payments are still estimated; import your statement to finalise them."

---

## 9. Roadmap impact

### 9.1 Items and hours (focused hours, O / L / P) [Speculative]

This breakdown **supersedes** both 06's item #12 "Multi-currency per account + manual rate" (10/16/28),
which is removed, and 07c §7's single re-estimate of ≈ 18/28/45. Do not add either of them to the figures below.

| ID | Item | Phase | O | L | P |
|---|---|---|---|---|---|
| P0-a | Generated `Currency` + `CurrencyFormatOverride` seed (ISO list + CLDR 49 pinned), rounding helpers, money-type suite extended (TND, JPY/XOF, CLF, IQD display-vs-storage, unit division sabotage, carry-forward) | Phase 0 | 6 | 10 | 16 |
| P0-b | `jurisdiction` columns + `RulePack` binding in domain model v1 | Phase 0 | 2 | 4 | 8 |
| M-1 | Per-account currency; cross-currency transfer (two legs, `Conversion`, fee line, computed FX cost); estimated → realised | MVP | 14 | 22 | 36 |
| M-2 | Local `FxRate` cache + Frankfurter adapter pinned to BCT and ECB, carry-forward, unconverted badge, manual override | MVP | 8 | 14 | 24 |
| M-3 | Exponent-aware amount parsing in quick add, notification templates and CSV import, plus the string corpus | MVP | 6 | 10 | 18 |
| M-4 | Per-currency budget lines, `fx_policy`, combined view at a planning rate, per-pocket STS with `convertibility` | MVP | 8 | 14 | 24 |
| M-5 | `charge_currency` on RecurringSeries/Occurrence, FX-neutral comparison and `fx_effect_pct` | MVP | 2 | 4 | 8 |
| M-6 | Cash-account flows: ATM withdrawal as transfer, "count your cash", millime chips | MVP | 4 | 8 | 14 |
| M-7 | Sample collection + Tunisian notification/SMS templates for the user's own apps | MVP | 6 | 12 | 24 |
| V-1 | PDF statement import (text-layer, per bank, from collected samples) | v1 | 16 | 28 | 48 |
| V-2 | Reporting-currency switch per view, unrealised revaluation display, recurring FX band, own-methods table | v1 | 12 | 20 | 34 |
| V-3 | ReferenceNote + `tn.*` / `eu.*` parameter rows with source and date, editable without a release | v1 | 2 | 4 | 8 |
| V-4 | 20-receipt Tunisian OCR benchmark; stamp duty and 3 VAT rates in the receipt parser | v1 | 4 | 8 | 12 |

### 9.2 Reconciliation with 06 §2.3

| Phase | Old O / L / P | Change | New O / L / P |
|---|---|---|---|
| Phase 0 | 80 / 130 / 214 | + P0-a + P0-b (no hardening line in Phase 0) = +8 / +14 / +24 | **88 / 144 / 238** |
| MVP | 258 / 427 / 723 | − #12 (10/16/28) + M-1..M-7 (48/84/148) = +38 / +68 / +120; hardening at 15/20/25 % on that = +6 / +14 / +30 | **302 / 509 / 873** |
| v1 | 301 / 518 / 880 | + V-1..V-4 (34/60/102); hardening at 15/20/25 % = +5 / +12 / +26 | **340 / 590 / 1 008** |
| v2, launch | unchanged | BCT server redistribution only after BCT agreement (no hours) | unchanged |
| **Total** | 990 / 1 680 / 2 880 | +91 / +168 / +302 | **≈ 1 080 / 1 850 / 3 180** |

At the likely figure the daily driver moves from ~week 16 to **~week 19 full-time** (≈ 653 h ÷ 35), or from
~week 56 to **~week 65 side-project**. [Speculative: arithmetic on the table]

### 9.3 Revised lean option (keeps multi-currency)

06 §2.4's lean daily driver was **≈ 360 h likely** (Phase 0 at 110 h without the iOS half of the spike,
plus ~250 h of MVP work), with no multi-currency, so #12 was never in it and nothing is subtracted.

| Part | L | Basis |
|---|---|---|
| Old lean | 360 | 06 §2.4 |
| + P0-a, P0-b | +14 | §9.1 |
| + M-1..M-7 | +84 | §9.1 |
| + hardening 20 % on the MVP part (84) | +17 | 06 MVP hardening rate |
| **Revised lean (likely)** | **≈ 475 h** | ≈ **13.6 weeks full-time** or ≈ **48 weeks side-project** |

**Revised lean O / L / P ≈ 280 / 475 / 820 h.** 06 gives the old lean only as an L figure, so its O and P
are derived here from the MVP's own O/L (0.60) and P/L (1.69) ratios: ≈ 217 and ≈ 609. Additions: O 8 + 48
+ 7 hardening = 63; P 24 + 148 + 37 hardening = 209. [Speculative: derived]

Leaner still: drop M-2 (manual rates only, no BCT/ECB fetch): about −17 h L, **≈ 458 h**, at the cost of the
reference rate that FX cost (§3.d) is measured against. [Speculative]

What moves into the MVP that 06 left out of the lean cut: per-account currency, the exponent table,
cross-currency transfers with FX cost, per-currency budgets and per-pocket STS, FX-neutral recurrence, cash
flows, and Tunisian notification templates. What stays in v1: PDF statement import, reporting-currency
switch, revaluation, reference notes, Arabic-OCR decision.

### 9.4 New risks

| # | Risk | Likelihood / impact | Mitigation | Grade |
|---|---|---|---|---|
| R11 | **Exponent / rounding data corruption**: an amount stored at the wrong exponent (TND as cents, IQD at intl's 0, HUF/IDR/COP at CLDR 49's 0), a missed `unit` division (JPY ×1000), or repeated rounding. Silent and permanent in a money app | Medium / **High** | Exponent only from the generated `Currency` table; no `/100` anywhere (lint rule); decimal arithmetic on integers; round once, half-even; property tests that convert across 0/2/3/4 exponents and back; sabotage tests that drop the `unit` division or swap ISO for display digits and must go red; migration keyed on `iso_list_version` | [Speculative] |
| R12 | **Rate-source terms block server distribution** (BCT personal-use only; ERA no redistribution) and break the v2 sync design | Medium / Medium | Device-side fetch by default; `provider` on every row; server syncs ECB rows only; request BCT agreement before v2; swappable adapters | terms [Verified: C18, termes.pdf]; impact [Inferred] |
| R13 | **Stale or wrong regulatory figures** shown as current (allowance, thresholds, payment-institution caps read as allowances) | Medium / High (trust, possible harm) | ReferenceNote only, with source text, date and grade; empty rows by default; no figure from an undated page shown without that label; never a block or advice | [Inferred: C6–C9] |
| R14 | **FX moves fire false price-hike alerts** or make TND budgets "overspend" | High (without the fix) / Medium | Compare in `charge_currency`; `fx_effect_pct` note; budgets in line currency with a planning rate | [Speculative] |
| R15 | **Incomplete Tunisian feeds** (OTP redaction, ≥ 500 TND alert thresholds, cash) make the ledger look complete when it is not | High / Medium | Feed-coverage indicator; statement reconciliation as ground truth; cash count prompt; "untracked cash" line | [Inferred: §6] |
| R16 | **"14,500" parse ambiguity** (14.5 TND vs 14,500 units) in OCR, notifications and quick add | High / High | Currency first, then separators; echo the parse when ambiguous; confidence score; corpus tests in both readings | [Inferred: §3.c] |
| R17 | **Unconverted amounts silently treated as 1:1** | Low with the rule / High | `unconverted` state excluded from totals with a badge; a test that a missing rate never yields rate 1 | [Verified: Firefly III fallback exists (fact-check)]; mitigation [Speculative] |
| R18 | **Jurisdiction added late**, so French rules have already tagged Tunisian spending | Medium / Medium | `jurisdiction` in Phase 0 model v1 (P0-b) | [Speculative] |

---

## 10. What remains unverified

- **Exchange control:** the travel-allowance circulars (2007-04, 2009-21, 2014-16) and décret 2007-394 were
  not opened; the allowance and the banknote ban rest on an undated page. The **20,000 DT customs threshold**
  (douane.gov.tn unreachable). Circular 2026-10 against bct.gov.tn and its 28 Sept corrected version.
  **Check before relying on any of them.**
- **Transfer cost**: the World Bank Remittance Prices Worldwide FR→TN corridor (HTTP 403). No figure is used.
- **BCT reuse terms for rates specifically** ("Mentions légales"; Frankfurter's `terms_url` is null). They
  block server-side redistribution until settled.
- How issuers apply the Art. 3a(1) mark-up rule to TND payments; the codified Reg. 2021/1230 numbering.
- Whether ERA's "end-use" caching covers an app displaying converted amounts (Inferred only).
- Any Tunisian bank's CSV/OFX export; any real SMS, push or PDF statement layout; whether Android 15 also
  redacts confirmation messages that carry a reference number.
- Flouci's Webank and BTE roles and its "virtual Visa"; whether D17 or Flouci send push notifications.
- Arabic OCR quality (Tesseract `ara`, Apple Vision language list); real Tunisian receipt formats.
- LF 2026 (Loi n° 2025-17) in the JORT; STEG and SONEDE billing periodicity.
- Whether the current UIB alert threshold is still 500 TND (2015 source).
- CLDR symbols for currencies other than TND (they come from the pinned generator, not hand-checked).
- Every hour figure in §9 (Speculative; recalibrate with logged hours after Phase 0 and the MVP).

---

## Sources

**Re-opened in the 2026-11-01 fact-check pass (relied on here):**
- ISO 4217 List One (SIX), Pblshd 2026-09-17 — https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml
- CLDR — https://registry.npmjs.org/cldr-core/latest (49.0.0); cldr-core@49.0.0 supplemental/currencyData.json; cldr-numbers-full@46.0.0 and @49.0.0 main/fr-TN, main/ar-TN (via cdn.jsdelivr.net)
- Flutter `intl` 0.20.3 — https://pub.dev/api/packages/intl ; dart-lang/i18n number_symbols_data.dart and src/intl/constants.dart
- BCT daily rates — https://www.bct.gov.tn/bct/siteprod/cours.jsp (Journée du 29/10/2026); archive https://www.bct.gov.tn/bct/siteprod/cours_archiv.jsp ; https://www.bct.gov.tn/bct/siteprod/cours_archiv_xls.jsp?select=2026-10-01
- BCT terms of use (PDF created 2016-12-23) — https://www.bct.gov.tn/bct/siteprod/documents/termes.pdf
- BCT exchange-regulation summary (undated; newest citation 2016) — https://www.bct.gov.tn/bct/siteprod/page.jsp?id=67
- BCT FAQ on exchange regulation, Jan 2019 — https://www.bct.gov.tn/bct/siteprod/documents/Document_20190211_fr_002.pdf
- BCT circular 94-13, consolidated 2008 — https://www.bct.gov.tn/bct/siteprod/documents/cir9413.pdf
- BCT circular 2026-10 (press-hosted) — https://businessnews.com.tn/wp-content/uploads/2026/09/yA1IYG-Cir_2026_10_fr.pdf
- ECB — https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml ; https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html ; https://www.ecb.europa.eu/services/disclaimer/html/index.en.html
- Frankfurter — https://frankfurter.dev/ ; https://frankfurter.dev/license/ ; https://api.frankfurter.dev/v2/providers ; /v2/currencies ; /v2/rates?base=EUR&quotes=TND (blend, providers=BCT, providers=ECB)
- ExchangeRate-API — https://www.exchangerate-api.com/terms ; https://www.exchangerate-api.com/docs/free ; https://open.er-api.com/v6/latest/EUR
- Fixer, currencyapi, exchangerate.host, Open Exchange Rates, CurrencyBeacon pricing pages — https://fixer.io/product ; https://currencyapi.com/pricing/ ; https://exchangerate.host/ ; https://openexchangerates.org/signup ; https://currencybeacon.com/pricing
- Reg. (EU) 2019/518 as adopted — https://www.legislation.gov.uk/eur/2019/518/body/data.html ; https://www.legislation.gov.uk/eur/2019/518/article/1/adopted
- French customs, cash reporting (updated 04/05/2026) — https://www.douane.gouv.fr/en/fiche/cash-reporting-obligation
- La finance pour tous, card fees abroad (2025-07-10) — https://www.lafinancepourtous.com/2025/07/10/carte-bancaire-combien-coute-un-paiement-ou-un-retrait-despeces-a-letranger/
- Firefly III exchange rates — https://raw.githubusercontent.com/firefly-iii/docs/main/docs/docs/explanation/financial-concepts/exchange-rates.md
- Fiskil open-finance tracker, Tunisia — https://www.fiskil.com/open-finance-tracker/tunisia
- Apple Pay countries — https://support.apple.com/en-us/102775
- Android 15 behaviour changes — https://developer.android.com/about/versions/15/behavior-changes-all
- ML Kit text recognition v2 languages — https://developers.google.com/ml-kit/vision/text-recognition/v2/languages
- PwC Tunisia other taxes (last reviewed 29 June 2026) — https://taxsummaries.pwc.com/tunisia/corporate/other-taxes
- TUNPAY (webdo.tn, 2026-05-06) — https://www.webdo.tn/fr/actualite/national/paiement-mobile-la-bct-lance-le-label-tunpay-pour-unifier-lecosysteme/397433/
- UIB MESSAG'EK (2015) — https://www.leconomistemaghrebin.com/2015/12/21/uib-service-gratuit-information-sms/
- BCT balance sheet end Aug 2026 (press) — https://lapresse.tn/2026/09/16/les-chiffres-cles-du-bilan-de-la-bct-a-fin-aout-2026/
- Flouci — https://www.leconomistemaghrebin.com/2026/06/08/paiement-digital-flouci-premiere-tunisie/ ; https://docs.flouci.com ; https://apps.apple.com/us/app/id1486786024
- Sensor Tower Q4 2023 TN budgeting — https://sensortower.com/blog/2023-q4-unified-top-5-personal%20finance%20budgeting%20and%20planning-revenue-tn-63e363f0e1714cfff1979caa

**Opened by the 07c research pass (not contradicted):**
- YNAB multi-currency guide — https://support.ynab.com/en_us/using-multiple-currencies-in-ynab-a-guide-SyBF6PHno.md
- Lunch Money multicurrency — https://support.lunchmoney.app/settings/multicurrency.md
- Actual Budget multi-currency — https://actualbudget.org/docs/budgeting/multi-currency

**Search summaries only (Inferred / Unverified):** Tunisian press on the 20,000 DT customs declaration
(lapresse.tn 2026-04-28 and 2026-06-19, webdo.tn, allafrica 2026-05-29); tunisienumerique (cash growth);
webmanagercenter 2024-11-29 (Flouci card BIN sponsor); banq.ai / openbankingtracker (0 TN aggregators);
STEG / SONEDE SMS bills; Tesseract `ara` threads.

**Not reachable:** douane.gov.tn (curl exit 7); World Bank RPW FR→TN corridor (HTTP 403); EUR-Lex
Reg. 2021/1230 (no body); BCT SDMX portal (HTTP 500).

**Upstream files:** 04-product.md (§2.7, §4, §6), 06-roadmap.md, 07a-tnd-mechanics.md,
07b-tunisia-capture.md, 07c-multicurrency-design.md (same directory).
