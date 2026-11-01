# 07c — Multi-country / multi-currency design (EUR + TND first, N currencies later)

> Research date: 2026-11-01. Scope: a user with ties to France and Tunisia who spends in EUR and TND, moves
> money between the two, and may later live or travel in other countries. The app must **track, budget and
> help optimise** spending across currencies. This file extends 04-product §2.7 (multi-currency) and §6 (data
> model), and roadmap item #12 in 06-roadmap.
>
> Grades: **[Verified: url]** = opened in this session · **[Inferred: basis]** · **[Unverified: prior
> knowledge]** · **[Speculative]** = design judgement. Regulatory and customs figures are graded strictly:
> only an official page opened this session, with its date, earns [Verified].
>
> Neutrality: the gap between an official rate and the rate a user actually obtains is recorded below only as a
> **budgeting fact the app must model**. Nothing here is guidance on where or how to exchange money.

---

## 0. Summary

1. **The ECB does not publish a TND rate.** Its euro reference rates cover 29 currencies, and TND is not one of
   them. [Verified: https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html]
   The "ECB daily rates" plan in 04 §2.7 therefore cannot cover the core use case on its own.
2. **The Banque Centrale de Tunisie (BCT) publishes the authoritative TND rates**, and the open-source
   **Frankfurter v2** API already ingests them as provider `BCT` (no key, no quota, MIT code). [Verified:
   https://api.frankfurter.dev/v2/providers; https://www.bct.gov.tn/bct/siteprod/cours.jsp]
3. **Frankfurter's default "blended" EUR/TND is not the BCT figure.** For 2026-10-29 the blend gave 3.3594
   and `providers=BCT` gave 3.3565, which is BCT's own published interbank average for that day.
   [Verified: https://api.frankfurter.dev/v2/rates?base=EUR&quotes=TND&from=2026-10-26&to=2026-10-31 and
   …&providers=BCT; matched against cours.jsp]. **Design rule: pin the provider per currency pair. Never label
   a blend as "the official rate".**
4. **TND has 3 decimal places** (millimes), like BHD, KWD, OMR, LYD, IQD and JOD. JPY has 0, and CLF and UYW
   have 4. [Verified: ISO 4217 List One, published 2026-09-17,
   https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml].
   `amount_minor BIGINT` still works, but each currency needs an **exponent table** and the app needs an
   explicit **rounding policy** for converting between 2 and 3 decimals.
5. **EU fee transparency covers EU currencies only.** CBPR2 (Art. 3a) requires card issuers to express
   conversion charges as a mark-up over the ECB reference rate and to send a message for payments in a
   **"Union currency"**. [Verified: https://www.legislation.gov.uk/eur/2019/518/body/data.html]. A card
   payment in TND falls outside that regime, and there is no ECB TND rate to measure a mark-up against. The app
   has to **compute the spread itself**, against a reference rate it names (BCT).
6. **Even official sources publish more than one rate for the same day.** On 29/10/2026 BCT published EUR at
   3.3565 TND (interbank average) and 3.329 TND (average rate for *change manuel* and travellers' cheques).
   [Verified: cours.jsp, "Journée du 29/10/2026"]. So "the rate" is not one number. Each conversion must record
   **which kind of rate** it used (reference, realised or manual), along with any fee.
7. **Good peer apps do not convert silently.** Firefly III lets a user-entered `foreign_amount` win over any
   computed rate. Lunch Money converts each transaction at the rate of its own date. YNAB gives up and keeps
   one plan per currency. Firefly also shows the trap to avoid: when it has no rate, it assumes 1:1. [Verified:
   Firefly III docs on GitHub, see §3].

---

## 1. FX rate sources — coverage, limits, licence

### 1.1 Comparison table

| Source | TND? | Free tier / limits | Licence & terms that matter for an app | Grade |
|---|---|---|---|---|
| **ECB euro reference rates** | **No** (29 currencies, RUB suspended) | Free; about 16:00 CET on each TARGET working day | "published for information purposes only"; "Using the rates for transaction purposes is strongly discouraged." Reuse is free if the information is reproduced accurately, the ECB is cited, and any modification (e.g. a derived cross rate) is stated explicitly. | [Verified: ECB rates page above; https://www.ecb.europa.eu/services/disclaimer/html/index.en.html] |
| **BCT — cours moyens** (bct.gov.tn) | **Yes, TND is the base.** Interbank table: 20 currencies. Change-manuel table: 17. Forwards: EUR and USD only. | Free HTML page, daily, published per working day ("Journée du …"). **Units vary per currency: 1, 10, 100 or 1000** (JPY per 1000, CHF/SEK/SAR per 10, DKK/NOK per 100). No buy/sell columns. | No terms of use shown on the page. The "Mentions légales" content was not retrieved. Frankfurter lists `terms_url: null` for BCT. | [Verified: https://www.bct.gov.tn/bct/siteprod/cours.jsp, 29/10/2026]; reuse terms [Unverified: legal notice not opened] |
| **Frankfurter v2** (frankfurter.dev) | **Yes**, from BCT, Bank Al-Maghrib, Bank of Algeria, IMF, BIS, INFOREURO, NBP, HMRC and others (106 providers, 228 currencies) | "There are no quotas" (abuse throttling only); daily; history back to 1948; v1 deprecated but "remains available indefinitely"; self-hostable with Docker | Code, docs and images are MIT. Data is "public data, shared for information", "come as is, with no warranty"; "If a provider publishes terms of use, they apply." Blended rates are "Frankfurter's own calculation and free to use." | [Verified: https://frankfurter.dev/, https://frankfurter.dev/license/, /v2/providers, /v2/currencies] |
| **ExchangeRate-API open endpoint** (open.er-api.com) | **Yes** (EUR→TND 3.357753, updated Sun 01 Nov 2026 00:02 UTC) | No key; refreshes every 24 h; HTTP 429 when rate limited, lifted after 20 min; "request once every hour and never get rate limited"; `time_eol` field announces deprecation | Docs page: **attribution link required** ("Rates By Exchange Rate API"), caching allowed, **"not allowed to re-distribute it"**. Terms page: "not in any product or service that offers programmatic or automatic access to exchange rate data"; caching "for customer end-use only"; free plans "can be discontinued … without notice". The Terms page itself states no attribution requirement, so the two pages differ. Keep both URLs. | [Verified: https://www.exchangerate-api.com/docs/free, https://www.exchangerate-api.com/terms, https://open.er-api.com/v6/latest/EUR] |
| **exchangerate.host** | ~168 currencies (TND not checked one by one) | **100 requests/month** free; paid from $14.99/month | Now an **APILayer** product (Vienna). Free-tier commercial use and HTTPS are not stated. | [Verified: https://exchangerate.host/]; TND [Unverified] |
| **Open Exchange Rates** | **Yes** (listed in currencies.json) | Free: 1,000 req/month, hourly, **USD base only**; Developer $12/month | Free-plan commercial use not stated on the signup page | [Verified: https://openexchangerates.org/signup, https://openexchangerates.org/api/currencies.json] |
| **Fixer** | [Unverified: not checked] | Free: **100 calls/month**, hourly, **"Non-Commercial Use"**; Basic $14.99/month | Free tier rules out a published app | [Verified: https://fixer.io/product] |
| **currencyapi** | [Unverified: 171 fiat currencies claimed; TND not checked] | Free: 300 req/month, daily, **"private use only"**; Small $9.99/month | Free tier rules out a published app | [Verified: https://currencyapi.com/pricing/] |
| **CurrencyBeacon** | [Unverified: not checked] | Free: **5,000 req/month, hourly**; Basic $10/month | "HTTPS + commercial use" is listed as a shared feature, but the page does not confirm it for the Free plan | [Verified: https://currencybeacon.com/pricing] |
| **Wise rates API** (`GET /v1/rates?source=&target=`) | TND exists as a Wise payout currency on several Wise country pages | The endpoint exists. Wise's own example authenticates with partner client credentials. Whether a **personal token** may call it is **not confirmed by official docs** (only a third-party Raycast extension does so). | Built for Wise customers and partners, not as a public rate feed | [Inferred: search summary of docs.wise.com affiliate guide; https://www.raycast.com/jorgetoh/wise-lens]; personal-token access [Unverified] |
| **Other central banks via Frankfurter** (Bank Al-Maghrib, Bank of Algeria, IMF "representative rate", EC INFOREURO monthly accounting rate, HMRC monthly) | TND in all of these provider lists | Through Frankfurter at no cost | Each one's own terms apply. INFOREURO and HMRC are *monthly accounting* rates, which suit an "accounting rate" mode better than daily spending. | [Verified: /v2/providers listing] |

### 1.2 What the sources imply [Speculative unless graded]

- **Primary for EUR↔TND: BCT, read through Frankfurter with `providers=BCT`.** It is the authoritative
  source for the dinar, needs no key and has no quota. It is published per Tunisian working day, so it has gaps
  on weekends and holidays (Frankfurter reported `publishes_missed: 1` for BCT, and the latest BCT date on
  2026-11-01 was 2026-10-29). [Verified: /v2/providers]
- **Primary for EU and major currencies: ECB, through Frankfurter `providers=ECB`.** A probe with
  `providers=ECB` and `quotes=TND` returned `[]`, as expected, while the positive control `providers=ECB&quotes=USD` returned 1.1172 for 2026-10-30. [Verified: probes this session]
- **Fallback for currencies BCT and ECB do not cover: the Frankfurter blend, labelled "blended (indicative)".**
  ExchangeRate-API is a second fallback only for **on-device, end-use** fetches. Its terms forbid
  redistribution, so **a self-hosted rate proxy, or rate tables bundled into the app or a sync payload, must
  not carry ExchangeRate-API data.** [Inferred: ERA terms quoted above]
- **Rule out free tiers that forbid commercial or app use** (Fixer, currencyapi), and any that are too small
  for daily refreshes across many users (exchangerate.host at 100 calls/month) unless the app runs a single
  server-side fetcher. [Inferred: plan limits above]
- **Attribution in the UI:** a line in the "Rates" footer, e.g. "Rates: Banque Centrale de Tunisie, ECB (via
  Frankfurter)", plus the ERA link whenever ERA data is shown. This satisfies the ECB's cite-the-source
  condition and ERA's docs. Derived cross rates (TND→USD through EUR) must say they are derived, per the ECB's
  "modifications must be stated explicitly" condition. [Verified: ECB disclaimer; ERA docs]

### 1.3 Caching and offline strategy [Speculative]

| Concern | Design |
|---|---|
| Local-first | An `fx_rate` table on the device holds (date, base, quote, rate, provider, rate_kind, fetched_at). Every conversion **reads the local table only**. The network fills the table and is never consulted at conversion time. |
| Refresh cadence | One fetch per day per pinned provider, after its publication time (ECB about 16:00 CET; BCT on Tunisian working days). Frankfurter accepts many quotes in one request, so a single call per provider is enough. |
| Missing day (weekend, holiday, offline) | Use the **latest prior published date** and show its age ("rate of 29/10, 3 days old"). Lunch Money also uses the "closest day". [Verified: https://support.lunchmoney.app/settings/multicurrency.md]. **Never assume 1:1** (the Firefly III default, see §3). With no rate at all, mark the amount **unconverted**, leave it out of home-currency totals with a visible badge, and queue it for re-conversion. |
| History | Backfill the rates on the dates of imported transactions with a time-series query (`from`/`to`), cached forever. A published reference rate for a past date does not change. |
| First run offline | Ship a small seed of recent monthly rates for the user's chosen currencies, clearly dated, and replace it on the first sync. ERA data must not be in the seed (redistribution). |
| Server role (later sync phase) | One server fetcher pulls ECB and BCT through Frankfurter (or self-hosted Frankfurter, MIT) and syncs the rate rows to devices, so each device does not hit public APIs. This works only with sources that allow redistribution (central-bank data under its own terms). |
| Provider death | Store `provider` on every rate row, keep a swappable adapter, and watch the ERA `time_eol` field. Frankfurter v1 → v2 is a precedent: v1 is deprecated, but its maintainers say it stays available. |

---

## 2. Budgeting facts the app must model (rates are not one number)

| Fact | Evidence | Model consequence [Speculative] |
|---|---|---|
| The ECB says its rates are for information, not transactions | [Verified: ECB page] | A reference rate is a **yardstick**, never "what you paid". |
| BCT published two EUR averages on the same day: 3.3565 interbank and 3.329 for change manuel / travellers' cheques | [Verified: cours.jsp 29/10/2026] | `rate_kind` ∈ {reference_interbank, reference_cash, realised, manual}. Reports state which kind was used. |
| Frankfurter's blend differs from the BCT figure by about 0.09 % on the same day (3.3594 vs 3.3565) | [Verified: probes above] | A blend is "indicative". Pin the provider per pair. |
| EU card rules: the mark-up over the ECB rate must be disclosed, and issuers must send a message for payments in a **Union currency** | [Verified: legislation.gov.uk copy of Reg. 2019/518 Art. 3a; its rules now sit in the codified Reg. (EU) 2021/1230, Inferred from the EUR-Lex record in search results, EUR-Lex page did not load] | TND payments get no regulated mark-up message. The app derives the cost itself: realised amount compared with the reference rate on the booking date, plus any explicit fee lines. |
| The bank's converted amount often posts days after the purchase | [Verified: YNAB guide advises reconciling because rates "depend on when the vendor or your bank processes the transaction", https://support.ynab.com/en_us/using-multiple-currencies-in-ynab-a-guide-SyBF6PHno.md] | A transaction holds an **estimated** conversion at entry, replaced by the **realised** amount when the statement or import arrives. The difference goes to "FX cost" (see §4.4). |

---

## 3. How good multi-currency finance apps do it

| App | Home/reporting currency | Account currency | Conversion rule | Budgets | Transfers between own accounts in different currencies | Grade |
|---|---|---|---|---|---|---|
| **YNAB** | One currency per plan | Every account in a plan should use the plan's currency; "YNAB doesn't convert currencies within a plan" | None, the user converts manually | One plan per currency | Outflow to a "Currency Transfer" category in plan A, inflow in plan B "factoring in the exchange rate and any fees" | [Verified: support.ynab.com guide above] |
| **Lunch Money** | "Primary currency" for "all totals and summaries", can be changed any time | Per transaction | Daily fetch of more than 160 currencies; **historic rate of the transaction's own day**; closest day if missing | Shown in the primary currency | (not covered by the page) | [Verified: https://support.lunchmoney.app/settings/multicurrency.md]; TND not listed on that page |
| **Actual Budget** | n/a | n/a | "currency agnostic and does not support multi-currency"; an experimental rule-template workaround stores the original amount and rate in notes | n/a | n/a | [Verified: https://actualbudget.org/docs/budgeting/multi-currency] |
| **Firefly III** | One "primary currency" per financial administration, and conversion goes **only** to it | "Asset accounts have one main currency" | Optional downloaded rates (built-in currencies only, about weekly, no history) or user-set rates. A user `foreign_amount` **overrides** the computed value ("assumes that you know better"). **"If Firefly III has no exchange rate data … the exchange rate is 1."** Up to 8 decimals. | (not covered) | "you must also indicate the amount transferred in both currencies" | [Verified: https://raw.githubusercontent.com/firefly-iii/docs/main/docs/docs/how-to/firefly-iii/features/currencies.md and …/explanation/financial-concepts/exchange-rates.md, …/currencies.md] |
| **Tricount** | One main currency per group | n/a | Expenses in other currencies converted at a "market rate … updated automatically on a daily basis" (v3.7+); custom rate not documented | n/a (sharing app) | Settles in the group currency | [Unverified: search snippets of tricount.com; the official page returned 404 when opened] |
| **Wallet (BudgetBakers)** | Home-currency overview | One currency per account (multi-currency is account-based) | Automatic rates | — | — | [Unverified: third-party reviews only] |
| **Spendee** | Main currency, switchable | One currency per wallet | Daily rate updates; totals reportedly **recalculated at current rates** rather than fixed at entry | — | — | [Unverified: third-party reviews only] |

**Lessons [Speculative]:**
- Take Firefly's **"user's realised amount wins"** rule and Lunch Money's **"rate of the transaction date"**
  rule. Do not take Firefly's **1:1 fallback** or Spendee's **revalue history at today's rate** (the reported
  past spending would change every day).
- Do not make YNAB's one-budget-per-currency model the default for a two-currency life. It hides the combined
  picture this persona wants. Offer it as an option: a budget can be single-currency (§4.5).
- Treat currency conversion between own accounts as a first-class **transfer with two legs**, as Firefly and
  YNAB do, not as two unrelated transactions.

---

## 4. Design for this app

### 4.1 Currencies and exponents

| Element | Design | Grade |
|---|---|---|
| `Currency(code CHAR(3), numeric, exponent SMALLINT, symbol, name_i18n, source_version)` seeded from ISO 4217 List One | EUR = 2, **TND = 3**, JPY = 0, KWD/BHD/OMR = 3, CLF = 4 | Exponents [Verified: ISO List One 2026-09-17]; table [Speculative] |
| `amount_minor BIGINT` stays (04 §6) | 1 TND = 1000 minor units. BIGINT gives about 9.2×10^18, ample even at exponent 4. | [Inferred: arithmetic] |
| Display | The exponent drives formatting: `41,956 TND` in fr_TN with 3 decimals. Arabic and Latin digits per `digits_style` (04 §6 User). The symbol "د.ت" is Frankfurter's metadata for TND, but ISO does not standardise symbols, so the code is the safe fallback. | [Verified: /v2/currencies symbol]; rest [Speculative] |
| Cash rounding | Some countries round cash payments to a coin step. Store an optional per-country `cash_rounding_step` parameter, not hard-coded. | [Unverified: no TND cash rounding rule checked] |

### 4.2 Rounding policy [Speculative]

- Convert with **decimal or rational arithmetic on the integer minor units**, never with floats:
  `target_minor = round(source_minor × rate × 10^(e_target − e_source))`.
- **Half-even (banker's rounding)** at the target exponent, applied **once per conversion**, never per
  intermediate step. Example: 12,50 EUR × 3.3565 = 41.95625 TND → 41.956 TND. TND → EUR loses the third
  decimal: 41.956 × 0.29793 (BCT TND→EUR on 29/10) = 12.49995 → 12.50 EUR.
- **Never recompute a stored realised amount from a rate.** The realised amount is the fact and the rate is
  derived from it (`realised_rate = target_minor / source_minor`, stored at high precision).
- Splits and shares in a foreign currency use the largest-remainder method, so the sum of allocations equals
  the total exactly in the transaction's own currency (04 §6 already has the sum-equals-total DB check).
  Conversion to home currency happens after splitting.

### 4.3 Three currency roles

| Role | Where stored | Meaning |
|---|---|---|
| **Transaction currency** | `Transaction.original_currency` + `original_amount_minor` | What the merchant charged (TND at a Tunis shop, USD for a SaaS subscription) |
| **Account currency** | `Account.currency` → `Transaction.amount_minor` + `currency` | What moved on the account (a EUR card paying in TND posts a EUR amount) |
| **Reporting (home) currency** | `User.home_currency` / `Household.home_currency`, **switchable for a view** | Totals, charts, net worth. Conversion happens at **report time from cached rates** and is **never written back** to the transactions. |

The persona needs to switch the **reporting currency per view** ("show my month in TND", "show my year in EUR")
without touching stored data. Firefly III cannot do this: it converts only to its single primary currency.
[Verified: Firefly exchange-rates doc]. This is a differentiator. [Speculative]

### 4.4 Converting between own accounts: realised and unrealised FX [Speculative]

**Transfer EUR account → TND account:**

```
Leg A (EUR account):  -500,00 EUR                     transfer_peer_id = B
Leg B (TND account): +1 650,000 TND (amount actually received)   transfer_peer_id = A
Fee line (optional): -x EUR or -y TND, category "Fees & FX cost", linked to the transfer
Stored on the transfer: realised_rate = 3.30, reference_rate = BCT 3.3565 (date d), rate_kind=realised
FX cost (computed)  : (reference_rate × 500 − 1 650) TND, converted to the reporting currency
```

*(Synthetic numbers that show the shape of the record. They are not a quote of any provider.)*

- **Realised FX cost** = (amount the reference rate would have given − amount actually received) + explicit
  fees. It is computed and shown as a **virtual expense** in an "FX cost" view. Leg B's budget category is
  "transfer", so the cost does not double-count. It can optionally be materialised as a split for users who
  want it in their category budget.
- **Unrealised FX** = the change in value of foreign-currency **balances** between two report dates when they
  are revalued at reference rates. It is **display only** ("your TND cash is worth 1.2 % less in EUR than a
  month ago"). It is never a transaction and never budget spend.
- **Card payment in a foreign currency** (EUR card, TND merchant): `original_amount_minor` in TND,
  `amount_minor` in EUR from the statement. The implied rate compared with the reference rate on the booking
  date gives the card's FX cost, and the bank's own "commission" line, when there is one, is linked as a fee.
  An entry made at purchase time is **estimated** (`fx_status = estimated`) until the import or statement
  replaces it (`fx_status = realised`). The difference goes to FX cost, not back into the category.
- **Cash withdrawal abroad** = a transfer from the card account to a TND **cash account** (cash is an account,
  04 §6 Account.type). Cash spending is then recorded in TND with no conversion, and the conversion cost is
  captured once, at the withdrawal.

### 4.5 Budgets across currencies [Speculative]

- **Each budget line has one currency** (`Budget.currency` already exists in 04 §6). The persona can run
  "France monthly" in EUR and "Tunisia monthly" in TND.
- **TND spending charged to a EUR budget** (a EUR card in Tunisia) counts at the **account amount** (EUR, what
  left the account), not at a reference-converted figure, so the budget matches the bank.
- **Envelope mode across currencies:** money is assigned in the envelope's currency. Moving money between a
  EUR envelope and a TND envelope is an explicit conversion event (the YNAB "Currency Transfer" idea, inside
  one plan) at a rate the user enters or accepts.
- **Combined view:** a "Life budget" rolls up all lines into the chosen reporting currency at **period-average
  reference rates** (or period-end rates, configurable) and labels the rate basis.
- **Trip budgets** (04 §2.7 travel mode, roadmap #43) become a special case of a currency- and
  country-scoped budget.

### 4.6 Recurring items charged in a foreign currency [Speculative]

- `RecurringSeries.charge_currency` + `expected_amount_minor` in **that** currency (a TND phone plan, a USD
  app subscription paid from a EUR card).
- The forecast converts at the latest reference rate and shows a **band** ("≈ 9,80–10,10 EUR") built from the
  observed realised-vs-reference gap of past occurrences.
- Price-hike detection (04 `Occurrence.price_change_pct`) compares amounts **in the charge currency**, so a
  movement in the exchange rate does not raise a false "price increase" alert. A separate "FX drift" note
  covers the currency effect.

### 4.7 Country rule packs: France rules must not apply to Tunisian spending [Speculative]

- Add **`jurisdiction` (ISO 3166-1 alpha-2)** to `Account` (default = institution country) and an optional
  per-transaction override (a French card used in Tunisia: the account is FR, the place of spending is TN).
- `ParameterTable` keys are already country-prefixed (`fr.ir.abattement10.min`, 04 §6). A **RulePack**
  (`country`, `version`, `valid_from`, `rules[]`) binds features such as frais réels, notes de frais, warranty
  durations and proof-retention dates to that country.
- **Binding rule:** a tax or legal feature evaluates only transactions whose **tax jurisdiction** matches its
  pack, i.e. the jurisdiction of the taxpayer profile (`TaxProfile.country`) for deductible tagging, and the
  place of purchase for consumer-law features like the warranty presumption. Tunisian spending never feeds
  the French 10 %-vs-frais-réels comparator unless the user explicitly tags it as a professional expense under
  their French profile.
- Tunisia pack v1 is **empty by design** (no tax features), so it is a placeholder and no un-researched rules
  get invented. Its rows come from official texts later, each with a `source_url` and date.

### 4.8 Cross-border cash and exchange-control facts as dated parameters (reminders, never advice)

| Fact | Status | How the app may use it [Speculative] |
|---|---|---|
| Carrying cash of **≥ 10,000 EUR** (or the equivalent in other currencies) into or out of the EU must be declared to customs. Bank transfers and money-transfer companies are not covered. Legal basis: Regulation (EU) 2018/1672. National basis given as Art. **L.152-12** CMF on the English page, while a search snippet of a French page cited **L.152-1**: the discrepancy is recorded, not resolved. | [Verified: https://www.douane.gouv.fr/en/fiche/cash-reporting-obligation, "Updated on 04/05/2026"] | Optional: when a trip is planned and the user's cash-account balances in the trip currencies exceed a parameter, show an informational reminder linking to the customs page. Stored as `eu.cash_declaration.threshold`, `valid_from`, `source_url`. |
| Tunisian rules on the **travel allowance** (*allocation touristique*), import or export of dinar banknotes, convertible-dinar or foreign-currency accounts for people with ties abroad, and Tunisian cash-declaration thresholds | **[Unverified — check before relying on it.]** No BCT circular, bct.gov.tn regulation page or douane.gov.tn text was opened in this session. Search results show these rules are set and changed **by BCT circular** (e.g. circulars 2025-05 and 2025-16 on the pilgrimage allowance, according to Tunisian press), so any figure goes stale fast. | No figure is shipped. The pack ships with **empty parameter rows** that point to the official sources, filled only after the official text has been read and dated. The app may let a user record an *allowance used this year* as a manual **budget envelope**, without the app asserting the legal amount. |
| Some accounts may be **restricted in what they can convert or transfer** (e.g. non-convertible balances) | [Unverified — check before relying on it.] | Model only as an account flag `convertible: yes/no/unknown`, set by the user. It drives which accounts the "move money" flow offers. No rule is asserted. |

---

## 5. Data-model deltas to 04-product §6 [Speculative]

| Entity | Change |
|---|---|
| **Currency** (new) | `code, numeric, exponent, symbol, name_i18n, iso_list_version` |
| **FxRate** (new) | `date, base, quote, rate (DECIMAL 18,10), provider (BCT/ECB/FRANKFURTER_BLEND/ERA/USER), rate_kind (reference_interbank/reference_cash/indicative/user), unit (per 1/10/100/1000 as published), fetched_at`; unique (date, base, quote, provider, rate_kind) |
| **Transaction** | add `fx_status (none/estimated/realised)`, `fx_rate_kind`, `reference_rate_id` (FK FxRate, the yardstick), `fee_minor` + `fee_currency` (or a linked fee split), `jurisdiction` override. The existing `fx_rate` and `fx_source` keep the **realised/implied** rate. |
| **Transfer** (between own accounts) | Today: two transactions linked by `transfer_peer_id`. Add on both legs (or a small `Conversion` row): `realised_rate`, `reference_rate_id`, `fee_txn_id`. FX cost is computed, not stored. |
| **Account** | add `jurisdiction`, `convertible (yes/no/unknown)`; `currency` already exists |
| **Budget / BudgetLine** | `Budget.currency` exists. Add `scope_jurisdiction` (optional) and an `envelope_conversion` event type for cross-currency envelope moves. |
| **RecurringSeries** | add `charge_currency`; `expected_amount_minor` is held in that currency |
| **User** | `home_currency` exists. Add `reporting_currency_default` and `secondary_currencies[]` (for showing amounts side by side). |
| **ParameterTable / RulePack** | add a `country` column (or enforce the key prefix) + `RulePack(country, version, valid_from, source_url)` |
| **Settlement / ShareAllocation** | `Settlement.currency` exists. Allow settlement in a different currency from the expense, with a stored rate, as Tricount's group-currency model does. |

---

## 6. "Optimisation" levers for a two-currency life [Speculative product ideas]

All of these show the **user's own data** back to them. None recommends a provider, venue or channel.

1. **"True cost" line on every conversion:** the realised rate against the reference rate of the day, shown
   as an amount and a %. Fees and spread become a visible, budgetable category ("FX cost this year: …").
2. **Compare the user's own channels:** a table of *the user's own* past conversions grouped by the method
   they recorded, with the median effective cost per method. The app ships **no preset method labels**: the
   user names their own (e.g. "card payment", "account transfer"), and the app ranks nothing outside the
   user's history.
3. **Fixed vs proportional fees:** from the recorded fee lines, show how the effective cost % changes with
   amount ("your recorded transfers under 100 EUR cost 4.1 % on average, over 500 EUR 1.3 %"), so the user can
   see the effect of batching.
4. **Timing awareness (descriptive only):** a chart of the reference rate over the user's conversion dates, and
   the gap between the purchase-time estimate and the posted amount. No prediction and no "convert now"
   signals. Exchange-rate forecasting is out of scope.
5. **Dual-currency safe-to-spend:** the forecast (04 §2.5) per currency pocket, so a TND shortfall does not
   hide behind a healthy EUR balance, plus a combined figure at the reference rate.
6. **Recurring charges in a foreign currency:** a list of subscriptions billed in a currency other than their
   paying account's, with this year's accumulated FX cost per item.
7. **Trip and stay summaries:** per-country spending in both currencies, cost per day, FX cost share.
8. **Reconciliation nudges:** "3 TND payments are still estimated; import your statement to finalise their
   EUR cost."

---

## 7. Impact on 06-roadmap [Speculative]

- Item #12 "Multi-currency per account + manual rate" (MVP, 10/16/28 in the O/L/P columns) assumed EU travel.
  For a **TND-first** persona, the MVP must also include: the exponent table and rounding policy (#4.1–4.2),
  the local FxRate cache with BCT and ECB adapters through Frankfurter, estimated vs realised conversion with
  FX cost (#4.4), and cross-currency transfers. A rough re-estimate is **about 1.5–2× the original
  (≈ 18/28/45)**. Report-currency switching, unrealised revaluation, the recurring FX band and the "own
  channels" table can wait for v1.x.
- The **RulePack jurisdiction binding (§4.7)** belongs to Phase 0 data-model v1 (06 row "Domain model v1"),
  because adding a jurisdiction column after French rules have tagged data means a migration and risks
  mis-tagged history. It is a one-way door, so decide it early.
- Test fixtures should cover a 3-decimal currency (TND), a 0-decimal one (JPY) and a missing-rate day from
  the start. A suite that tests only EUR would not exercise the exponent and rounding logic.

---

## 8. Open questions / to verify

| # | Question | Why it matters |
|---|---|---|
| Q1 | BCT site "Mentions légales": reuse terms for the cours pages | Needed before redistributing BCT rates from our own server (§1.3) |
| Q2 | Wise `/v1/rates` with a personal token: allowed and stable? | Optional "realised-rate benchmark" import; [Unverified] |
| Q3 | Tunisian exchange-control texts (allocation, account types, cash declaration) | Pack content; **check before relying on it** |
| Q4 | Codified CBPR text (Reg. (EU) 2021/1230) article numbering | Citing the current law rather than the amending act |
| Q5 | Which TND payment channels in Tunisia export statements (CSV/OFX) that the importer can read | Without imports, TND realised amounts depend on manual entry (cross-ref 02-bank-sync) |

---

## Sources (opened in this session, 2026-11-01)

- ECB euro reference rates — https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html
- ECB disclaimer and copyright — https://www.ecb.europa.eu/services/disclaimer/html/index.en.html
- BCT cours moyens (Journée du 29/10/2026) — https://www.bct.gov.tn/bct/siteprod/cours.jsp
- Frankfurter home / licence — https://frankfurter.dev/ · https://frankfurter.dev/license/
- Frankfurter API probes — https://api.frankfurter.dev/v2/providers · /v2/currencies · /v2/rates?base=EUR&quotes=TND (blend, `providers=BCT`, `providers=ECB`, 2026-10-26..31)
- ExchangeRate-API open access docs / terms / live response — https://www.exchangerate-api.com/docs/free · https://www.exchangerate-api.com/terms · https://open.er-api.com/v6/latest/EUR
- exchangerate.host — https://exchangerate.host/
- Open Exchange Rates — https://openexchangerates.org/signup · https://openexchangerates.org/api/currencies.json
- Fixer — https://fixer.io/product
- currencyapi — https://currencyapi.com/pricing/
- CurrencyBeacon — https://currencybeacon.com/pricing
- ISO 4217 List One (published 2026-09-17) — https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xml
- Regulation (EU) 2019/518 as adopted (Art. 3a/3b) — https://www.legislation.gov.uk/eur/2019/518/body/data.html
- French customs, cash reporting obligation (updated 04/05/2026) — https://www.douane.gouv.fr/en/fiche/cash-reporting-obligation
- YNAB, Using multiple currencies — https://support.ynab.com/en_us/using-multiple-currencies-in-ynab-a-guide-SyBF6PHno.md
- Lunch Money, Multicurrency — https://support.lunchmoney.app/settings/multicurrency.md
- Actual Budget, Multi-currency — https://actualbudget.org/docs/budgeting/multi-currency
- Firefly III docs (GitHub raw) — https://raw.githubusercontent.com/firefly-iii/docs/main/docs/docs/how-to/firefly-iii/features/currencies.md · …/explanation/financial-concepts/exchange-rates.md · …/explanation/financial-concepts/currencies.md

Search-result only (not opened, graded [Inferred]/[Unverified] where used): docs.wise.com affiliate guide;
raycast.com/jorgetoh/wise-lens; tricount.com help snippets; BudgetBakers Wallet and Spendee third-party reviews;
EUR-Lex record of Reg. 2019/518 repeal → 2021/1230; Tunisian press on BCT circulars 2025-05 / 2025-16.
