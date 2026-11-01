# 07a — TND money mechanics (Tunisian dinar) for a France ↔ Tunisia persona

> Research date: **2026-11-01**. Dimension: how the Tunisian dinar works as money, the official rates the
> app can read, and the exchange-control facts that shape what a user with ties to France and Tunisia
> spends and moves. Grades: **[Verified: url]** means the page or file was opened in this session.
> **[Inferred: basis]** and **[Unverified: …]** are as defined in 00-factcheck.md.
>
> **Exchange-control and customs figures change by circular.** Each one below names the text it comes
> from and that text's date. Even where it is marked Verified, **check before relying on it**. The app
> should treat these as reference notes it can update, never as hard-coded rules.
>
> **Neutrality.** The gap between official rates and the rate a user actually gets (cards, transfers,
> cash) is recorded here only as a **budgeting fact the app must model**. This document gives no
> guidance on where or how to exchange money.

---

## 0. Summary for the data model (read this first)

| # | Finding | Consequence for the app | Grade |
|---|---|---|---|
| 1 | TND is ISO 4217 numeric **788**, **3 minor units** (1 dinar = 1 000 millimes) | `amount_minor` stores **millimes**. The exponent comes from a currency table, never a hard-coded `/100` | [Verified: SIX list-one.xml, Pblshd 2026-09-17] |
| 2 | There are 7 three-decimal currencies, 17 zero-decimal, 2 four-decimal (CLF, UYW) and 13 with "N.A." (metals, SDR, test codes) | The exponent is a per-currency column with values 0, 2, 3 or 4. "N.A." codes are rejected as account currencies | [Verified: same file, parsed] |
| 3 | ISO, CLDR 46 and Flutter `intl` 0.20.3 **disagree** on the digits for several currencies (e.g. IQD: ISO 3, CLDR 0, intl 0; HUF/IDR/COP/PKR: ISO 2, intl 0) | Storage uses the **ISO** exponent. Display and cash rounding come from a separate table. Do not take the exponent from `NumberFormat` | [Verified: cldr-core 46.0.0 currencyData.json; dart-lang/i18n number_symbols_data.dart] |
| 4 | `intl` has **no `fr_TN` or `ar_TN` locale**, and its simple symbol for TND is **"din"**. CLDR `fr-TN` uses **"DT"** and `ar-TN` uses **"د.ت.‏"** | Ship our own symbol and format override for TND (and per locale) | [Verified: intl constants.dart and number_symbols_data.dart; CLDR fr-TN/ar-TN] |
| 5 | The BCT publishes daily **interbank mean rates (4 dp)**, **manual-exchange (banknote) mean rates (3 dp)** and 3- and 6-month **forward** EUR and USD rates. There is no API. There is an HTML page, a date-keyed archive and an "xls" that is really an HTML table | `FxRate` needs `rate_kind`, `unit`, `published_date` (the table's own date) and `fetched_at` | [Verified: bct.gov.tn cours.jsp, cours_archiv.jsp, cours_archiv_xls.jsp] |
| 6 | Several rates are quoted **per 10, 100 or 1 000 units** (JPY per 1 000, DKK/NOK per 100, SAR/AED/MAD/QAR/SEK/CHF/DZD/MRU per 10) | Store `unit` and divide it out. Getting this wrong makes a yen amount 1 000× too large | [Verified: cours.jsp, 29/10/2026 table] |
| 7 | Asking the archive for a non-business day returns the **previous** business day's table, labelled with **that** day's date (30/10 and 31/10/2026 both returned "Journée du 29/10/2026") | Key a rate by the date printed in the table, never by the date requested. Carry the rate forward across gaps and record that it was carried | [Verified: POST cours_archiv.jsp, input=2026-10-30 and 2026-10-31] |
| 8 | BCT terms: data may be downloaded **"à des fins exclusivement personnelles"**. Copying it to another site or network, or deriving commercial products from it, needs BCT's prior agreement | Fetching on the device for the user's own view is the defensible default. Bundling BCT rates, re-serving them from our server, or charging for them requires BCT permission | [Verified: bct.gov.tn documents/termes.pdf, created 2016-12-23] |
| 9 | The ECB reference rates **do not include TND** (the list has 29 currencies) | EUR↔TND comes from the BCT (TND per 1 EUR), from the card network's rate, or from the user. ECB alone is not enough for this persona | [Verified: ECB eurofxref-daily.xml, 2026-10-30] |
| 10 | Import and export of dinar **banknotes and coins is prohibited** (except under bilateral agreements, e.g. with Libya) | A TND cash wallet is purely an in-Tunisia account. The app never models a "carry TND to France" transfer as an ordinary transfer | [Verified: bct.gov.tn page.jsp?id=67, citing décret 77-608 as amended by décret 2007-394 — page undated] check before relying on it |
| 11 | Same-day spread between the interbank mean and the manual-exchange mean: EUR 3.3565 vs 3.329 (−0.82 %), USD 3.0138 vs 2.988 (−0.86 %) | Rates differ by channel. A transaction must record the rate actually used (`fx_rate`) **and** which kind of rate it was | [Verified: cours.jsp, Journée du 29/10/2026; arithmetic by this doc] |
| 12 | French cards outside the euro area: the average bank fee is **2.5 % + €0.30 per payment** and **2.4 % + €3.10 per withdrawal**, *on top of* the Visa or Mastercard rate (Panorabanques, June 2025) | Model the fee as its own line ("FX & fees" category), separate from the conversion | [Verified: lafinancepourtous.com 2025-07-10] |
| 13 | 06-roadmap.md's "lean daily driver" leaves out multi-currency (06-roadmap.md line 198) | For this persona multi-currency is a **must** from the first daily-driver release. See §8 | [Verified: 06-roadmap.md read this session] |

URLs for every Verified item in this table are listed in § Sources.

---

## 1. ISO 4217: the TND code, number and minor units

Source: the official ISO 4217 maintenance agency list (SIX), `list-one.xml`, root attribute
`Pblshd="2026-09-17"`. I downloaded and parsed it this session.
[Verified: https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml]

```xml
<CcyNtry>
  <CtryNm>TUNISIA</CtryNm>
  <CcyNm>Tunisian Dinar</CcyNm>
  <Ccy>TND</Ccy>
  <CcyNbr>788</CcyNbr>
  <CcyMnrUnts>3</CcyMnrUnts>
</CcyNtry>
```

- **Alpha code** `TND`, **numeric** `788`, **minor units** `3`. One dinar is 1 000 **millimes**. A price of
  "14,500 DT" is fourteen dinars five hundred millimes, so `amount_minor = 14500`.
  [Verified: per above; "millime" as the name of the subunit — Unverified: prior knowledge, not stated in list-one.xml]
- **EUR** is `978` with 2 minor units [Verified: same file].
- CLDR records TND as Tunisia's currency `_from: 1958-11-01` [Verified: https://cdn.jsdelivr.net/npm/cldr-core@46.0.0/supplemental/currencyData.json].

### 1.1 All non-2-decimal ISO currencies (so the model is general)

Parsed from the same file (unique codes; the same code can appear for several countries):

| Minor units | Count | Codes |
|---|---|---|
| **0** | 17 | BIF, CLP, DJF, GNF, ISK, JPY, KMF, KRW, PYG, RWF, UGX, UYI, VND, VUV, XAF, XOF, XPF |
| **3** | 7 | BHD, IQD, JOD, KWD, LYD, OMR, **TND** |
| **4** | 2 | CLF, UYW |
| **N.A.** | 13 | XAG, XAU, XBA, XBB, XBC, XBD, XDR, XPD, XPT, XSU, XTS, XUA, XXX |
| 2 | 139 | everything else (EUR, USD, GBP, MAD, DZD, …) |

[Verified: list-one.xml, Pblshd 2026-09-17, parsed with Python ElementTree]

XOF and XAF are 0-decimal and XPF (French Pacific territories) is 0-decimal. These matter for a
French-linked user who travels in francophone Africa or overseas France. [Verified: per above]

### 1.2 ISO is not what the formatters use

ISO says how many decimals the currency *has*. CLDR and Flutter's `intl` say how many to *show*.
They disagree:

| Code | ISO minor units | CLDR 46 `_digits` (cash) | Flutter `intl` 0.20.3 `currencyFractionDigits` |
|---|---|---|---|
| TND | 3 | 3 | 3 |
| IQD | **3** | **0** | **0** |
| HUF | 2 | 2 (cash 0) | **0** |
| IDR | 2 | 2 (cash 0) | **0** |
| COP | 2 | 2 (cash 0) | **0** |
| PKR | 2 | 2 (cash 0) | **0** |
| AFN, ALL, IRR, KPW, LAK, LBP, MGA, MMK, RSD, SOS, SYP, YER | 2 | 0 | 0 |
| CZK, SEK, NOK, TWD, TZS, UZS, MNT, MUR, AMD, CRC, GYD | 2 | 2 (cash 0) | 2 |

[Verified: CLDR https://cdn.jsdelivr.net/npm/cldr-core@46.0.0/supplemental/currencyData.json ;
intl https://raw.githubusercontent.com/dart-lang/i18n/main/pkgs/intl/lib/number_symbols_data.dart (pub.dev
latest 0.20.3, published 2026-06-25); cross-compared with list-one.xml by a script]

**Rule for the model** [Inferred: from the divergence above]:
- `currency.storage_exponent` comes from **ISO**. It sets `amount_minor`, and it does not change for an existing row.
- `currency.display_digits` and `currency.cash_rounding` come from **CLDR**, separately. Showing an IQD
  amount without decimals is fine. Storing it without them would lose the 3rd decimal.
- Never derive the exponent from `NumberFormat.currency(...).decimalDigits`. That would silently store
  HUF in whole forints while the bank statement shows fillér.

If the ISO exponent ever changes (it has for some currencies in the past), a stored amount needs a
migration keyed on `(currency, exponent_version)`. [Unverified: prior knowledge of past ISO changes; no
amendment checked this session]

---

## 2. How TND amounts are written

| Context | Form | Grade |
|---|---|---|
| CLDR `fr-TN` symbol | **DT**; pattern `#,##0.00 ¤`; decimal **","**, group U+202F (narrow no-break space). TND digits = 3 override the pattern's 2, so 14 500 millimes print as `14,500 DT` and 1 234 500 millimes as `1 234,500 DT` | [Verified: cldr-numbers-full@46.0.0 main/fr-TN] |
| CLDR `ar-TN` symbol | **د.ت.‏** (ends with an RLM); pattern `‏#,##0.00 ¤` (starts with an RLM); latin digits; decimal **","**, group **"."** | [Verified: cldr-numbers-full@46.0.0 main/ar-TN] |
| CLDR plain `fr` | no TND symbol at all (display name only), so the formatter falls back to **TND** | [Verified: cldr-numbers-full@46.0.0 main/fr] |
| CLDR plain `ar` | symbol د.ت.‏ but decimal **"."** and group **","**, the opposite of `ar-TN` | [Verified: main/ar] |
| Flutter `intl` | **no** `fr_TN` or `ar_TN` entry (it has `fr`, `fr_CH`, `ar`, `ar_DZ` …). `simpleCurrencySymbol('TND')` = **"din"** (DZD and KWD also give "din") | [Verified: dart-lang/i18n number_symbols_data.dart and src/intl/constants.dart, fetched this session] |
| BCT tables | 4 decimals with a decimal comma (`3,3565`), interbank; 3 decimals (`3,329`), manual exchange | [Verified: bct.gov.tn cours.jsp] |
| Receipts, menus, signs | "14,500 DT", "14.500 DT", "14,5 DT", "14 DT 500", "TND 14.500", "14,500 د.ت" all occur | [Unverified: prior knowledge — check against real receipts during OCR work] |

**The comma trap** [Inferred: from the CLDR separators above plus the 3-decimal exponent]. In French "14,500"
means fourteen and a half. In an English-style receipt "14,500" means fourteen thousand five hundred. For
TND the three digits after the separator are a valid millime amount, so the string cannot be read
correctly without knowing the currency and the locale. OCR, voice entry and notification parsers must:
1. resolve the currency first;
2. for a 3-decimal currency, read a single separator followed by exactly 3 digits as the **decimal**
   separator when the locale is `fr-TN` or `ar-TN`, or when the source is a Tunisian merchant;
3. show the parsed amount back to the user ("14,500 DT = 14 dinars 500 millimes") before saving, whenever
   the reading is ambiguous.

**What to build** [Inferred: from the intl gaps above]:
- A small `CurrencyFormatOverrides` table keyed by `(locale, currency)`: symbol, position, decimal and
  group separators. Fill it from CLDR `fr-TN` and `ar-TN` for TND, and use it in place of `intl` for the
  symbol and separators.
- Arabic UI (04-product §2.15): amounts stay LTR. Keep the RLM characters CLDR puts in the pattern, or
  bidi will move the "د.ت." symbol to the wrong side of the number.

---

## 3. BCT official rates (Banque Centrale de Tunisie)

### 3.1 What is published

On `https://www.bct.gov.tn/bct/siteprod/cours.jsp`, fetched 2026-11-01 (a Sunday). The newest table was
**"Journée du 29/10/2026"** (Thursday). [Verified: https://www.bct.gov.tn/bct/siteprod/cours.jsp]

| Table | What it is | Precision | Example 29/10/2026 |
|---|---|---|---|
| **Cours moyens … marché des changes interbancaire (opérations en compte et au comptant)** | Mean interbank rate, TND per `unit` of foreign currency | 4 dp | EUR 1 = 3,3565; USD 1 = 3,0138; GBP 1 = 3,9933; JPY **1000** = 18,9938; MAD **10** = 3,0418; DZD **10** = 0,2215 |
| **Cours moyens … pour le change manuel et les chèques de voyage** | Mean rate for banknotes and traveller's cheques | 3 dp | EUR 1 = 3,329; USD 1 = 2,988 |
| **Cours moyens à terme** | Forward mean rates, EUR and USD only | 4 dp | 3 months: USD 3,0385, EUR 3,4029; 6 months: USD 3,0693, EUR 3,4461 |

Currencies quoted (interbank, 20): DZD, SAR, CAD, DKK, USD, GBP, JPY, MAD, NOK, SEK, CHF, KWD, AED,
EUR, LYD, MRU, BHD, QAR, CNY, OMR. The manual-exchange table lists 17 (no DZD, MAD or MRU).
[Verified: cours.jsp, per above]

**What the BCT itself says about these rates** (exchange-regulation page): the dinar's rate *"est
déterminé librement sur le marché des changes créé depuis le 1er mars 1994"*, and the BCT *"publie, à
titre indicatif, au plus tard le lendemain, le cours de change interbancaire des devises et des billets
de banque"*. [Verified: https://www.bct.gov.tn/bct/siteprod/page.jsp?id=67 — page undated]

So the rates are **indicative means, published no later than the next day**. They are not rates the BCT
gives customers. Press reports say they update around 16:00 Tunis time. [Unverified: press only —
tunisienumerique.com 2026-09-30 search snippet, not opened]

### 3.2 Access mechanics (there is no API)

| Mechanism | Detail | Grade |
|---|---|---|
| Current table | `GET https://www.bct.gov.tn/bct/siteprod/cours.jsp`, HTML, **Windows-1252 / Latin-1** encoded | [Verified: fetched, decoded as latin-1] |
| Archive by date | `POST https://www.bct.gov.tn/bct/siteprod/cours_archiv.jsp`, body `input=YYYY-MM-DD&langue=` (the call the page's own jQuery makes). Returns an HTML fragment | [Verified: input=2026-10-01 returned "Journée du 01/10/2026", EUR 3,3740] |
| Non-business day | `input=2026-10-30` and `input=2026-10-31` both returned the **29/10/2026** table. 30/10/2026 (a Friday) had no table of its own when fetched | [Verified: per above; why 30/10 is missing is not known — holiday or publication lag] |
| "XLS" export | `GET cours_archiv_xls.jsp?select=YYYY-MM-DD&langue=` (also `cours_archiv_xls1.jsp`). Served as `application/xls`, but the body is a **UTF-8 HTML table** with columns: date, numeric currency code, name, alpha code, unit, value | [Verified: select=2026-10-01, 7 246 bytes, first row "01/10/2026 012 DINAR ALGERIEN DZD 10 0,2219"] |
| Current "xls" link `cours_xls.jsp?langue=` | **404** | [Verified: HTTP 404 this session] |
| SDMX portal `gosdmx.bct.gov.tn/gosdmx/pnrd` | **HTTP 500** this session, so it could not be assessed as a rates source | [Verified: HTTP 500] |
| CSV or JSON API | None found | [Inferred: no link on cours.jsp, index.jsp or the sitemap plan_site3.jsp] |

Parsing contract [Inferred: from the files above]: decode as latin-1 (page) or UTF-8 (xls); read the
decimal comma; divide by `Unité`; key the rate by the date printed in the table; and treat a changed
layout as an error (stop and show "rate unavailable"), never as a zero.

### 3.3 Terms of reuse — a design constraint

BCT "Termes et conditions d'utilisation du site web" (PDF created 2016-12-23):
- Users *"peuvent imprimer, télécharger des informations ou données du site à des fins exclusivement
  personnelles"*.
- They may not *"recopier tout ou partie du site sur un autre site ou un réseau interne"*, nor *"vendre ou
  … distribuer le contenu du site ou d'en dériver des produits à des fins commerciales sans accord
  préalable de la BCT"*. Contact: bct@bct.gov.tn.
- Data quoted as a citation must be reproduced exactly, with the site named as the source.
[Verified: https://www.bct.gov.tn/bct/siteprod/documents/termes.pdf]

Consequences [Inferred: from the terms above; this is not legal advice — see 05-compliance]:
- **Defensible:** the app on the user's device fetches the BCT table for that user's own budgeting,
  shows "Source: BCT, cours moyen interbancaire du 29/10/2026", and links to bct.gov.tn.
- **Needs BCT agreement before shipping:** bundling BCT rates in the app, re-serving them from a project
  backend or sync server, or putting them behind a paid tier.
- **Fallbacks:** the rate on the user's own card or bank statement (the best source for that transaction
  anyway, see §5), a rate the user types in, or a commercial FX API under its own licence (out of scope here).

### 3.4 ECB does not cover TND

The ECB daily reference file for 2026-10-30 lists 29 currencies: USD, JPY, CZK, DKK, GBP, HUF, PLN, RON,
SEK, CHF, ISK, NOK, TRY, AUD, BRL, CAD, CNY, HKD, IDR, ILS, INR, KRW, MXN, MYR, NZD, PHP, SGD, THB,
ZAR. **TND, MAD and DZD are not on it.**
[Verified: https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml]

04-product §2.7 plans to use "daily reference rates (e.g. ECB)". For this persona that needs a second
source, or a cross rate. A cross through USD (for example BCT TND/USD with ECB EUR/USD) is a derived
estimate, and should be stored with `rate_kind = derived`. [Inferred]

---

## 4. Exchange control: what "non-convertible" means for this persona

The dinar is convertible for **current** transactions: Tunisia accepted IMF Article VIII on
6 January 1993. Capital movements, and all flows between Tunisia and abroad, go through the BCT or
**authorised intermediaries (IA)**. Residents must repatriate foreign-currency income and, with
exceptions, sell it on the exchange market.
[Verified: https://www.bct.gov.tn/bct/siteprod/page.jsp?id=67 (sections "Régime de change" and "Principes
de base"; page undated, cites loi 76-18 of 21 Jan 1976 and décret 77-608 of 27 Jul 1977)] — check before relying on it.

Between residents, an obligation in a currency other than the dinar is prohibited unless the BCT grants
a waiver. *"Le dinar doit être à la fois une monnaie de compte et une monnaie de règlement."*
[Verified: BCT FAQ "Questions fréquemment posées sur la Réglementation des Changes – Janvier 2019",
https://www.bct.gov.tn/bct/siteprod/documents/Document_20190211_fr_002.pdf, Q3] — check before relying on it.

### 4.1 Resident or non-resident ("Tunisien résident à l'étranger", TRE)

A Tunisian individual is a **non-resident** when both hold: domiciled abroad for **more than two years**,
**and** having the **centre of activity** (where income is earned) abroad (Avis de change n°3 of
5 October 1982). Such a person *"peu[t] bénéficier du statut de résident pour effectuer un certain nombre
d'opérations en Tunisie (emprunts en dinar, ouverture de comptes intérieurs, acquisition de biens et droits
immobiliers …)"* and becomes a resident again on returning for good.
[Verified: page.jsp?id=67 "Notion de résidence"; and FAQ Jan 2019 Q1] — check before relying on it.

**App consequence** [Inferred]: the rules that apply depend on a user status the app cannot work out.
Store an optional, user-declared `fx_residency` per jurisdiction ("resident of TN", "non-resident TN
national", "foreign national") and use it **only** to choose which reference notes to show. Never use it
to block or advise anything.

### 4.2 Account types the app should be able to represent

| Account (BCT name) | Who | Key rule | App mapping | Grade |
|---|---|---|---|---|
| **Comptes étrangers en devises ou en dinars convertibles** | Any non-resident, natural or legal person, any nationality (so this covers a TRE) | *"crédités et débités librement en devises"* | Account with `currency` EUR (or another foreign currency) **or** TND with `convertibility = convertible` | [Verified: page.jsp?id=67] |
| **Comptes intérieurs** (ordinary TND accounts) | Residents; non-resident Tunisians may open them (see 4.1) | Ordinary dinar account | `currency = TND`, `convertibility = non_convertible` | [Verified: page.jsp?id=67, 4.1 quote] |
| **Comptes PPR** (Personnes Physiques Résidentes) in foreign currency or convertible dinars | Resident individuals, for the income listed in circ. 2017-04 of 23 June 2017 | Funds *"ne peuvent, en aucun cas, alimenter des comptes à ouvrir à l'étranger"* | Foreign-currency or convertible-TND account, flagged "restricted outflow" | [Verified: FAQ Jan 2019 Q4] |
| **Compte "allocation touristique en dinar convertible"** | Residents with unused allocation after a trip | Holds the unused part of the travel allowance (circ. 2007-15 of 26 Apr 2007) | Convertible-TND sub-account | [Verified: page.jsp?id=67] |
| **Comptes spéciaux en devises ou en dinars convertibles** | Includes Tunisians *moving their habitual residence from abroad to Tunisia* | Credited with income from assets lawfully held abroad | Foreign-currency or convertible-TND account | [Verified: page.jsp?id=67, citing avis 21 Apr 1987, circ. 87-37, 2007-19] |
| **Comptes "prestataires de services"** | Residents paid in foreign currency by non-residents abroad (e.g. freelancers) | Open at an IA | Foreign-currency or convertible-TND account | [Verified: page.jsp?id=67, circ. 2006-14] |
| **Comptes capital** | Non-residents' dinar assets that carry no transfer guarantee | Opening needs prior BCT authorisation **for non-resident Tunisians and their spouses** | TND, `convertibility = blocked` | [Verified: page.jsp?id=67, Avis de change n°5 of 5 Oct 1982] |

All page.jsp?id=67 rows: page undated; check before relying on it.

**Model addition** [Inferred: from the table]: `Account.convertibility ∈ {convertible, non_convertible,
blocked, n/a}`, shown for TND accounts only. This matters for **budgeting**: money in a
non-convertible TND account is spendable only in Tunisia, so a "safe-to-spend" figure or a goal in EUR
should not silently count it as EUR. Show it as "TND (local only)".

### 4.3 Travel allowance (allocation touristique) for residents

| Item | Rule | Source text and date | Grade |
|---|---|---|---|
| Who | Resident individuals, Tunisian or foreign | circ. 2007-04 (9 Feb 2007), amended by 2009-21 (13 Nov 2009) and 2014-16 (10 Dec 2014) | [Verified: page.jsp?id=67 — page undated; latest cited amendment 10 Dec 2014] **check before relying on it** |
| Amount | **6 000 DT per year**; **3 000 DT per year** for children under 10. Foreign residents who transfer savings on salary: 3 000 DT, and 1 500 DT for children under 10 | same | same |
| Carry-over | Unused allowance **cannot** be carried over to the next year | same | same |
| Form | Cash, cheques, or a **nominative international payment card reserved for the allowance**. That card *"ne peut être utilisée à partir de la Tunisie que pour des réservations d'hôtels à l'étranger"* | same | same |
| Still in force? | In 2023 the BCT said a rise was "not on the agenda" and that the last change dated from 2009. A Dec 2025 circular (reported as 2025-16) charges 2026 pilgrimage allowances to the **2026** transfer rights | press: businessnews.com.tn 2023-06-14; managers.tn and businessnews.com.tn 2025-12 | [Unverified: press, search snippets only] |
| Reform | A draft new exchange code has circulated. A 2024 evaluation proposes setting the ceiling in EUR (e.g. €3 000) | tunisienumerique.com PDF, 2024-03 | [Unverified: search snippet only] |

**App consequence** [Inferred]: for a user who declares TN residency, an optional **"annual allowance
tracker"** fits naturally: a calendar-year envelope in TND, filled by spending tagged as drawn on the
allowance. It must be an editable figure with its source and date shown. Do not hard-code 6 000.

### 4.4 Carrying money in and out of Tunisia

| Rule | Detail | Grade |
|---|---|---|
| **Dinar banknotes and coins** | *"L'importation et l'exportation du dinar tunisien en billets ou en pièces sont interdites"*, except under BCT agreements with foreign counterparts (one exists with the Central Bank of Libya, 18 Feb 2007, circ. 2007-06) | [Verified: page.jsp?id=67, citing décret 77-608 as amended by décret 2007-394 of 26 Feb 2007 — page undated] **check before relying on it** |
| **Foreign currency brought in by non-residents** | Free, no limit on amount. It must be **declared to customs on entry** if it will be paid into a foreign-currency or convertible-dinar account, or if the traveller expects to **re-export more than the equivalent of 5 000 DT** | [Verified: page.jsp?id=67, citing avis de change JORT n°10 of 3 Feb 2006 as amended by JORT n°18 of 2 Mar 2007, and circ. 94-13 of 7 Sep 1994 as amended by 2007-13 of 25 Apr 2007] **check before relying on it** |
| **Re-export of unused foreign currency (non-residents)** | Below the equivalent of 5 000 DT: no document needed. At or above 5 000 DT, if physically imported: the customs-stamped import declaration | same | same |
| **Converting dinars back (non-residents)** | No limit on amount. Below 5 000 DT: the exchange slip (*bordereau d'échange*) from the original exchange. Above 5 000 DT: the slip **plus** the customs declaration | same; circ. 94-13 (PDF opened: Articles 6–8) | [Verified: page.jsp?id=67 and https://www.bct.gov.tn/bct/siteprod/documents/cir9413.pdf] **check before relying on it** |
| **Habitual residents abroad** | *"sont autorisés à garder par devers elles les devises qu'elles ont régulièrement importées pour faire face à leurs dépenses ordinaires pendant leur séjour en Tunisie"* | [Verified: page.jsp?id=67 "Principes de base" — page undated] **check before relying on it** |
| **Customs declaration threshold reported in 2026 press** | "20 000 TND or more must be declared on entry or exit" (La Presse, 2026-01-31); "TRE have no threshold" (customs spokesman, reported 2026-05-29); "declare above €6 000" (April 2026 article). **These contradict each other and the 5 000 DT figure above.** They may be different rules (a customs declaration requirement versus the BCT documents needed to re-export or convert back) | [Unverified: press search snippets; douane.gov.tn refused connections on ports 443 and 80 this session] **check before relying on it** |
| **Online declaration** | Customs reportedly offers remote currency declaration through a "Smart Traveler" app or douane.gov.tn | [Unverified: press, lapresse.tn 2025-07-06 snippet] |

**App consequence** [Inferred]:
- A **cash TND wallet** is local to Tunisia. When the user leaves, the leftover is a balance that waits
  for the next trip, not an amount moved to France. Budget it as "held in TND".
- Keep an **"exchange slip" attachment** on a cash-exchange transaction (photo of the *bordereau*). It is
  a document the user may need later, which the receipt vault (04-product §2.9) can already store.
  The app gives no advice on it; it is simply a document type.

---

## 5. Card payments, transfers and the real rate (budgeting facts)

The rate a user pays is **not** the BCT mean. The app has to record what was actually charged.

### 5.1 A French (euro) card used in Tunisia

| Fact | Detail | Grade |
|---|---|---|
| Conversion base | The rate *"en vigueur lors du traitement de la transaction par la société émettrice de la carte (Visa ou MasterCard)"*, so the network rate on the processing date, which can be a different day from the purchase | [Verified: https://www.lafinancepourtous.com/2025/07/10/carte-bancaire-combien-coute-un-paiement-ou-un-retrait-despeces-a-letranger/] |
| Bank fees outside the euro area (averages) | **Payments: 2.5 % + €0.30 per transaction. Withdrawals: 2.4 % + €3.10.** About €42.60 per €1 000 spent (5 × €100 withdrawals + 2 × €250 payments), Panorabanques, June 2025 vs June 2024 | [Verified: same page, dated 2025-07-10] |
| Spread across banks | Traditional banks typically charge 1.5–3 % plus a fixed fee. Some online and neobank cards charge nothing (often with a monthly cap). 2026 examples: 2.70 %, 2.25 % (from 1 July 2026), 2.20 % at regional banks | [Unverified: press summaries — moneyvox.fr 2026, selectra.info — not opened] |
| Dynamic currency conversion (DCC) | A terminal or ATM may offer to charge in EUR at its own rate. The acquirer's rate then replaces the network rate. The difference is not itemised as a fee, so the app can see it only by comparing against a reference rate | [Unverified: prior knowledge; selectra.info article on DCC seen in search, not opened] |
| ATM-side fees in Tunisia | A Tunisian ATM may add its own fee on a foreign card, separate from the French bank's fee | [Unverified: prior knowledge — check before relying on it] |
| EU transparency rules | Reg. (EU) 2019/518 (amending 924/2009) requires card issuers to state conversion charges as a mark-up over the ECB rate and to send a message after the payment. My understanding is that this covers **Union currencies** only, so a TND payment would get no such message | [Unverified: EUR-Lex returned empty bodies (HTTP 202) this session] |

### 5.2 A Tunisian card abroad

For residents, the card that works abroad is the **nominative international card reserved for the travel
allowance**, and it is capped by that allowance (§4.3). From Tunisia it can be used only for hotel
bookings abroad. [Verified: page.jsp?id=67] — check before relying on it.

A practical consequence for budgeting [Inferred]: foreign online subscriptions (streaming, software)
paid from a Tunisian resident account count against the allowance or need another channel. The app
should let a **RecurringSeries** carry a `funding_note` and an "allowance-consuming" tag. It should not
judge or advise.

### 5.3 Moving money France → Tunisia

| Fact | Detail | Grade |
|---|---|---|
| Cost measure | The World Bank Remittance Prices Worldwide database measures total cost (fee + FX margin) per corridor, including FR→TN. It was **not reachable** this session (HTTP 403). Search snippets show provider quotes of about 2.8–4.5 % total cost on €140 and €345 sends, and an older collection period (Oct–Nov 2024) | [Unverified: https://remittanceprices.worldbank.org/corridor/FR/TN returned 403; figures from search snippets] |
| Two cost parts | The fee is visible. The FX margin is hidden in the rate. Only comparing the rate received with a reference rate (BCT interbank on the same day) shows it | [Inferred: definition of total cost in RPW methodology — Unverified, page not opened] |
| Arrives where | Into a foreign-currency or convertible-dinar account (non-residents), or converted to TND into a local account | [Verified: account types §4.2] |

### 5.4 Official rate vs obtained rate — the gap the app must model

On 29/10/2026 the BCT's **interbank** mean was EUR 3,3565 TND and its **banknote** mean was EUR 3,329 TND,
a gap of **−0.82 %** (USD −0.86 %). Counter buy and sell rates are spread on either side of the banknote
mean, card networks apply their own rate, and banks add a percentage. [Verified: BCT figures from cours.jsp;
the rest is Inferred from §5.1]

**Model it, don't advise it.** For each cross-currency transaction, store:

| Field | Meaning |
|---|---|
| `original_amount_minor`, `original_currency` | What the merchant charged, e.g. 14500 TND (= 14,500 DT) |
| `amount_minor`, `currency` | What hit the account, e.g. 449 EUR (= €4.49; an illustrative figure, not a measured one) |
| `fx_rate` (decimal string, ≥ 6 significant digits) | The rate actually applied, derived as amount / original if the statement does not give it |
| `fx_source` ∈ {`statement`, `network`, `bct_interbank`, `bct_manual`, `ecb`, `derived`, `user`} | Where the rate came from |
| `fx_rate_date` | The date the rate applies to (the processing date for cards, which can differ from `date`) |
| `fee_minor` + `fee_currency` (or a linked fee transaction) | The fixed and percentage fees, booked to the **"FX & fees"** category (04-product §2.7) |
| `reference_rate_id` → FxRate | The BCT or ECB rate for that day, used only to show "you paid X % over the reference" |

"X % over reference" is then a fact the user sees in Insights ("FX and card fees this month: €12.40,
2.9 % of foreign spend"). It is an **optimisation signal**, not a recommendation of a provider.
[Inferred: from 04-product §2.7 and §2.11]

---

## 6. Proposed data-model deltas (against 04-product §6)

The existing model already has `amount_minor` BIGINT + ISO `currency`, `Transaction.original_amount_minor /
original_currency / fx_rate / fx_source`, and `FxRate(base, quote, date, rate, source)`. [Verified: 04-product.md §6.1, read this session]

| Entity | Add | Why |
|---|---|---|
| **Currency** (new reference table) | `code`, `numeric`, `storage_exponent` (ISO), `display_digits`, `cash_rounding` (CLDR), `iso_list_version` (e.g. "2026-09-17") | §1.2 divergences; IQD and HUF would otherwise be stored wrong |
| **CurrencyFormatOverride** (new) | `locale`, `currency`, `symbol`, `pattern`, `decimal_sep`, `group_sep` | intl lacks fr_TN and ar_TN and prints "din" (§2) |
| **FxRate** | `rate_kind` ∈ {interbank_mean, banknote_mean, forward_3m, forward_6m, network, statement, derived, user}; `unit` (1/10/100/1000); `published_date` (the table's own date); `fetched_at`; `carried_forward` bool | BCT's three tables, the per-10/100/1000 quotes, archive gaps (§3) |
| **Account** | `convertibility` (TND only); `jurisdiction` (country) | Non-convertible TND is "local only" money (§4.2) |
| **Transaction** | `fx_rate_date`, `fee_minor`/`fee_currency` or a linked fee row, `reference_rate_id` | Obtained rate vs reference (§5.4) |
| **User** | optional `fx_residency` per jurisdiction | Picks which reference notes to show (§4.1), never blocks |
| **ReferenceNote** (new, optional) | `jurisdiction`, `topic` (allowance, cash-in/out, declaration), `value`, `unit`, `source_text`, `source_date`, `last_checked` | Shows regulatory figures **with their date**, updatable without a release |
| **Budget** | already has `currency`. Add `fx_policy` ∈ {spot_at_txn, fixed_planning_rate, per_line_currency} | A TND envelope next to EUR envelopes: decide whether the budget converts, uses a planning rate, or keeps lines in their own currency |

Multi-currency budgeting [Inferred]: a user who spends in both EUR and TND needs **per-currency
envelopes** (a "Tunisia trip" or "Tunisia household" budget in TND) **plus** a consolidated view in the
home currency at a **declared planning rate**. Converting daily at a moving rate makes a TND budget
"overspend" whenever the rate moves, which is noise, not behaviour.

Money arithmetic [Inferred: standard practice, consistent with 04-product §6]: never use floats. Convert
with a decimal rate, round once, with the rounding mode stored (half-even for display, the statement's
value for booked amounts), and keep the unrounded rate. Splits in TND must add up to the millime.

---

## 7. Phase-0 money-type test cases to add (06-roadmap exit criterion (b))

[Inferred: from the findings above]
1. `amount_minor = 14500`, `TND` formats as `14,500 DT` (fr-TN override), not `din 14.500` or `14,50 TND`; `1234500` formats as `1 234,500 DT`.
2. Parse "14,500" with currency TND and locale fr-TN → 14500 millimes. With currency USD and locale en-US → 1450000 cents.
3. JPY rate from BCT `1000 = 18,9938` → 0.0189938 TND per JPY. A sabotage check that drops the `unit` division must fail.
4. An IQD amount keeps 3 stored decimals while showing 0.
5. Archive request for a Saturday returns Thursday's table → `published_date = Thursday`, `carried_forward = true` for Friday and Saturday.
6. A TND split of 10,000 DT three ways adds up exactly (3,334 + 3,333 + 3,333).
7. A 0-decimal (XOF) and a 4-decimal (CLF) account round-trip with no loss.

---

## 8. Impact on the roadmap

- 06-roadmap's "lean daily driver" leaves out multi-currency. [Verified: 06-roadmap.md line 198] For a
  France ↔ Tunisia persona the minimum is: per-account currency, 3-decimal TND, the fr-TN/ar-TN format
  override, the manual rate plus rate source, and the "FX & fees" category. The BCT fetch can follow,
  once its reuse terms are settled. [Inferred]
- Feature #12 "Multi-currency per account + manual rate" (Must, MVP) stays. Its estimate (10/16/28 h)
  probably does not include the currency table, the format overrides or the BCT parser. [Inferred: the
  estimate's line in 06-roadmap gives no breakdown]

---

## Sources

### Opened in this session (Verified)
- ISO 4217 list one (SIX), Pblshd 2026-09-17: https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml
- BCT daily rates: https://www.bct.gov.tn/bct/siteprod/cours.jsp (table "Journée du 29/10/2026", fetched 2026-11-01)
- BCT rate archive (POST, `input=2026-10-01`, `2026-10-30`, `2026-10-31`): https://www.bct.gov.tn/bct/siteprod/cours_archiv.jsp
- BCT archive "xls" export: https://www.bct.gov.tn/bct/siteprod/cours_archiv_xls.jsp?select=2026-10-01&langue=
- BCT terms of use (PDF created 2016-12-23): https://www.bct.gov.tn/bct/siteprod/documents/termes.pdf
- BCT exchange-regulation summary (undated page; citations up to 2016): https://www.bct.gov.tn/bct/siteprod/page.jsp?id=67
- BCT FAQ on exchange regulation, January 2019 (PDF created 2019-02-05): https://www.bct.gov.tn/bct/siteprod/documents/Document_20190211_fr_002.pdf
- BCT circular 94-13 (travellers' currency import, reconversion, re-export): https://www.bct.gov.tn/bct/siteprod/documents/cir9413.pdf
- ECB daily reference rates (2026-10-30): https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml
- CLDR 46.0.0 currency data: https://cdn.jsdelivr.net/npm/cldr-core@46.0.0/supplemental/currencyData.json
- CLDR 46.0.0 numbers/currencies for fr-TN, ar-TN, fr, ar: https://cdn.jsdelivr.net/npm/cldr-numbers-full@46.0.0/main/{fr-TN,ar-TN,fr,ar}/{currencies,numbers}.json
- Flutter/Dart `intl` (pub.dev 0.20.3, 2026-06-25): https://pub.dev/api/packages/intl ; https://raw.githubusercontent.com/dart-lang/i18n/main/pkgs/intl/lib/number_symbols_data.dart ; https://raw.githubusercontent.com/dart-lang/i18n/main/pkgs/intl/lib/src/intl/constants.dart
- La finance pour tous, card fees abroad (2025-07-10): https://www.lafinancepourtous.com/2025/07/10/carte-bancaire-combien-coute-un-paiement-ou-un-retrait-despeces-a-letranger/
- Project files: docs/research/04-product.md §2.7 and §6; docs/research/06-roadmap.md

### Search-result summaries only (Unverified — page not opened)
- Travel allowance unchanged since 2009, BCT 2023 statement: https://businessnews.com.tn/2023/06/14/article-1198407/1198407/
- Pilgrimage 2026 allowance circular: https://managers.tn/2025/12/09/hajj-2026-voici-les-nouvelles-regles-concernant-lallocation-touristique/ ; https://businessnews.com.tn/2025/12/08/la-bct-autorise-le-versement-des-allocations-touristiques-pour-le-pelerinage-2026/1377402/
- Draft exchange-code evaluation (2024): https://www.tunisienumerique.com/wp-content/uploads/2024/03/Evaluation-du-Projet-de-Code-des-changes.pdf
- Currency declaration thresholds (2025–2026 press): https://www.lapresse.tn/2026/01/31/touristes-etrangers-ce-qui-change-dans-les-bureaux-de-change-tunisiens/ ; https://fr.allafrica.com/stories/202605290593.html ; https://www.tunisienumerique.com/tunisiens-residents-a-letanger-devises-a-lentree-en-tunisie-ce-que-la-douane-autorise-et-ce-quelle-exige/ ; https://www.lapresse.tn/2025/07/06/une-appli-pour-declarer-vos-devises-avant-de-quitter-la-tunisie/
- BCT rate publication time (press): https://www.tunisienumerique.com/taux-de-change-tunisie-2026-09-30/
- French bank card fees 2026 (press): https://www.moneyvox.fr/carte-bancaire/actualites/108840/credit-agricole-de-la-touraine-et-du-poitou-ce-que-coute-un-paiement-ou-un-retrait-a-etranger-en-2026 ; https://selectra.info/finance/actualites/marche/carte-bancaire-etranger-payer-en-euros-conversion-dynamique

### Fetched with no usable body
- douane.gov.tn: connection refused (ports 443 and 80)
- EUR-Lex Reg. (EU) 2019/518 (HTML and PDF): HTTP 202, empty body
- World Bank RPW FR→TN corridor: HTTP 403
- BCT SDMX portal gosdmx.bct.gov.tn: HTTP 500
- BCT `cours_xls.jsp` (current-day export): HTTP 404
