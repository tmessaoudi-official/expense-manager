# 04 — Product & Feature Design

> **Superseded in part (2026-11-01):** for multi-currency, France ⇄ Tunisia and the revised hours, `07-cross-border.md` wins where it conflicts with this file.

> Research dimension: **product and feature design** for an expense manager + reminder + tracker.
> Research date: 2026-11-01. Audience: personal use first, built so it can become a commercial product later.
> Bank sync is out of scope here (see the bank-sync / open-banking dimension). Every feature below is
> designed to work **without** a bank feed, and to get better when one is added.
>
> **Grading key.** `[Verified: <url>]` means the source was opened in this session. `[Inferred: <basis>]`
> means it follows from observed evidence. `[Unverified: from prior knowledge]` means it is recalled and was
> not checked. `[Speculative]` marks a design judgment with no checkable fact behind it. Most of this document
> is design, so most of it is `[Speculative]` by nature. The few policy and market facts the design depends on
> are graded on their own.

---

## 1. Product thesis

Most expense apps are **ledgers that look backwards**: they tell you where the money went. The gap is an app
that **looks forward and acts for you**:

1. **Capture costs about 2 seconds**, so the data is complete without a bank feed.
2. It answers one question at a glance: **"How much can I spend today without hurting anything I've committed to?"**
3. It **remembers money obligations for you**: bills, renewals, trials ending, warranties expiring,
   receipts you must keep for tax, debts within the household.
4. It **acts**: it drafts the cancellation, pre-fills the expense report, settles the household balance.

[Speculative] The positioning is "a personal CFO that lives in your lock screen", not "a spreadsheet with charts".

### Positioning anchors (market price points, short)

| Product | Model | Price point observed | Grade |
|---|---|---|---|
| YNAB | Zero-based ("give every dollar a job") | $14.99/mo or $109/yr, USD only; 34-day trial; sharing with others (the page says both "up to six people" and "up to 5 favorites") | [Verified: https://www.ynab.com/pricing] |
| Monarch | Aggregator + budgets; forecasting in the "Plus" tier | $14.99/mo or $99.99/yr | [Verified: https://www.monarch.com/pricing] |
| Rocket Money | Subscription tracking + cancellation concierge (Premium) + bill negotiation (success fee) | Premium is pay-what-you-want, about $7–14/mo per third-party guides | [Inferred: several 2026 third-party guides agree; vendor page not opened] |
| Splitwise | Shared expenses; Pro removes a daily expense cap, adds receipt scan, itemization, currency conversion | price not on the page; free-tier cap number not stated | [Verified: https://www.splitwise.com/pro] |
| Tricount (bunq) | Free group splitting, 150+ currencies, bunq card link | free; former Premium removed | [Inferred: third-party listings and bunq help pages in search results] |

**Takeaway** [Inferred: the table above]: the paid tier in this market sits around **$100/yr**. No single
product combines **budgeting + reminders/subscriptions + household settle-up + receipt/warranty vault + EU/French tax**.
That combination is the opening.

---

## 2. Complete feature set

Each area lists the core features first, then the "smart" layer.

### 2.1 Capture

| Feature | Description | Notes |
|---|---|---|
| **1-tap quick add** | Floating button → amount keypad first (the amount is the only required field) → category chips ranked by context (time, place, recent use) → done. | Target: under 3 s from unlock to saved [Speculative]. |
| **Widget / lock-screen / shortcut add** | Add from a home-screen widget, an iOS Shortcut / Android quick-settings tile, or a share-sheet target. | See §2.13 for platform limits. |
| **Natural-language & voice** | "Lunch 14,50 with Sam, split half" → amount, currency, category, split, counterparty. Works in FR/EN/AR, with decimal comma and `€` before or after. | Parse on-device first (rules + small model); fall back to the cloud only with consent [Speculative]. |
| **Receipt scan (OCR)** | Photo → merchant, date, total, VAT lines, line items. The receipt is attached to the transaction and filed in the vault. | On-device OCR (ML Kit / Apple Vision) [Unverified: from prior knowledge] keeps the receipt image private. |
| **Import** | CSV/OFX/QIF/CAMT.053 import with a mapping wizard and saved import profiles; e-mail-forward address for e-receipts and invoices. | Gives a "bank-feed-lite" path without an aggregator [Speculative]. |
| **Recurring detection** | Detects series (same merchant ± amount tolerance, regular period) and proposes "make this a bill/subscription". | Same engine powers price-hike detection (§2.4). |
| **Duplicate guard** | Same amount + date ± 1 day + similar merchant → "looks like a duplicate". | Essential once import and manual entry are mixed. |
| **Offline-first** | Every capture works offline; syncs later. | Basis of the local-first architecture [Speculative]. |

### 2.2 Categorization

- Two-level tree (group → category), editable, with icon and colour; sensible FR/EN/AR defaults.
- **Rules engine** (user-visible, editable): `IF merchant contains "X" AND amount < 20 THEN category = Café, tag = #work`.
  Rules are created **from corrections**: re-categorizing twice proposes a rule [Speculative].
- **Tags** cut across categories (`#vacation-2027`, `#frais-reels`, `#client-A`).
- **Splits**: one transaction → several categories (supermarket: groceries + household + pharmacy).
- **Merchant normalization**: `CB CARREFOUR 1234 PARIS` → "Carrefour".
- Confidence display: auto-categorized items show a subtle "auto" marker until confirmed.

### 2.3 Budgets — two modes, one model

| Mode | Who it suits | Mechanics |
|---|---|---|
| **Envelope / zero-based** | Users who want control and are motivated by "every euro has a job" (YNAB-style) | Income goes into "To assign"; the user funds envelopes; spending draws them down; overspending must be covered by moving money from another envelope. Rollover per envelope. |
| **Category limits** (soft caps) | Most people, and the default | A monthly limit per category, a progress bar, and alerts at 80% and 100%. No assignment ritual. |
| **"No budget" mode** | Trackers who only want awareness | Safe-to-spend + trends only. |

Design decision [Speculative]: **one data model** (`Budget` with `mode` + `BudgetLine`) behind all three, so a
user can move from limits to envelopes without migrating data. Periods: monthly by default; weekly, per pay
cycle, or custom (people paid on the 25th or every two weeks).

### 2.4 Bills, subscriptions & reminders

| Feature | Description |
|---|---|
| **Bill calendar** | Every upcoming obligation on a calendar and a timeline: rent, energy, phone, insurance, loan payments, taxes, annual fees. |
| **Smart reminders** | Lead time per bill (e.g. 3 days before), a "paid?" one-tap confirm, a snooze, escalation if unpaid on the due date. Quiet hours respected. |
| **Subscription tracker** | Name, price, billing period, next renewal, payment method, a "last used" self-report, a cost per year and per day shown side by side. |
| **Price-hike detection** | The recurring series engine compares each new occurrence with the series baseline → "Your streaming plan went from €13,49 to €15,99 (+18.5%, +€30/yr)". |
| **Free-trial-ending alert** | When the user logs a trial: an end date and the price after the trial; alerts at D-3 and D-1 with "keep / cancel" buttons. |
| **Annual-renewal radar** | Annual subscriptions and insurance renewals surface **45 days ahead**, when cancelling or switching is still possible. |
| **Cancellation helper** | Per-merchant cancel card: the deep link to the online cancellation page, a pre-filled letter or e-mail, notice period, and a reminder to check the next statement. |
| **Non-money reminders tied to money** | Warranty expiry, end of return window, car inspection, passport renewal fee, tax deadlines. |

**French/EU angle for the cancellation helper**: since 1 June 2023, consumer contracts concluded online in France must
offer a free, permanent, direct and easy-to-find online cancellation function ("résiliation en trois clics").
It does not change notice periods or commitment terms. [Inferred: several secondary sources in search results agree
on the date and content, citing decree no. 2023-417 and law no. 2022-1158; the official text was not opened.]
Product consequence [Speculative]: the helper's main job in France is **finding the cancel link and timing it**
(before the commitment period renews), not writing registered letters. A registered-letter template stays as the fallback.

### 2.5 Cash-flow forecast & "safe-to-spend"

- **Forecast** = current balances + scheduled income + bills, subscriptions and planned transfers + goal
  contributions, projected day by day for 30, 60 and 90 days. A "low point" marker shows the lowest projected
  balance and its date.
- **Safe-to-spend (STS)**, the headline number:

  ```
  STS(today) = available balance
             − sum(committed outflows until next income)      # bills, subscriptions, loan payments
             − goal contributions due before next income
             − safety buffer (user-set, e.g. €100 or 5% of income)
             + expected income only if marked "guaranteed"
  STS per day = STS(today) / days until next income
  ```

  Shown as "€38 today · €412 until payday". The number changes colour (calm, careful, stop). [Speculative]
- **What-if sandbox**: "If I buy this €600 laptop on Friday, what happens?" → the forecast is redrawn and the low point and goal delays are shown.
- **Variable spending modelled** from the trailing 3-month median per category, so the forecast does not assume zero spending on groceries.

### 2.6 Goals

- Target amount + date → a required monthly contribution, automatically put into the forecast.
- Goal types: savings (holiday, emergency fund), **debt payoff** (snowball/avalanche ordering), **"sinking funds"**
  for known irregular costs (car insurance, Christmas, a school year), and **spending-reduction goals** ("−20% on takeaways").
- Progress visual + "on track / 2 weeks behind" status; linked to an account or virtual (just a tracked amount).

### 2.7 Multi-currency

- Each account has a currency; each transaction stores the **original amount + currency** and the
  **converted amount in the account currency**, plus the rate and its source and date.
- Home currency for reports; daily reference rates (e.g. ECB reference rates [Unverified: from prior knowledge])
  with a manual override when the card's actual rate is known.
- Travel mode: an auto-detected trip (foreign currency transactions) → a trip budget, per-day spend in both currencies.
- Fees and FX spread tracked as their own category (shows the real cost of paying abroad).

### 2.8 Shared / household expenses & settle-up

- **Household**: members, roles (owner, member, viewer), shared accounts and shared budgets, and a private
  space per member (private transactions are never visible to the household).
- **Split rules**: equal, by percentage (e.g. proportional to income, 60/40), by shares, by exact amounts,
  or per-category defaults ("rent 50/50, groceries 60/40").
- **Running balance** between members; **settle-up** with the fewest transfers (debt simplification across a group).
- **Settle-up payment**: generate a SEPA payment QR code (EPC QR) or a payment link so the debtor's banking app
  can pay it. [Unverified: from prior knowledge, the EPC QR standard and European wallet links; verify per provider before building]
- **Trips / events** (temporary groups with guests who do not need an account, via a link), similar to Tricount/Splitwise.
- **Import from Splitwise/Tricount** exports to lower switching cost [Speculative].

### 2.9 Receipt & warranty vault

- Every receipt/invoice image or PDF is stored, linked to its transaction, and OCR text is searchable
  ("the receipt for the drill").
- **Warranty tracking**: for any purchase above a threshold or in an "equipment" category, the app proposes
  a warranty record. The default expiry is **2 years from delivery** (the French legal conformity guarantee, for
  new, used or refurbished goods bought from a professional), plus any extra commercial warranty the user enters.
  A claim requires a proof of purchase (delivery slip, invoice or receipt), which is exactly what the vault keeps.
  [Verified: https://www.service-public.gouv.fr/particuliers/vosdroits/F11094]
  - Nuance: the defect is presumed to have existed at delivery for 24 months on new goods and 12 months on used
    goods; the page is inconsistent about refurbished goods. [Verified: same page] The app should show
    "presumption until" as a second date, not hide it.
- **Return window** reminder (user-set per merchant; the 14-day withdrawal right for distance purchases is a likely
  default [Unverified: from prior knowledge]).
- **Retention policy per receipt**: "keep until" date driven by its purpose (warranty end, tax retention, forever).
  The vault never deletes on its own; it proposes clean-ups.
- Export a "claim pack": receipt + warranty card + photos as a single PDF.

### 2.10 Tax & deductible tagging (French context first, pluggable per country)

| Feature | Basis |
|---|---|
| **Frais réels tracker** for salaried users: tag expenses as professional (commuting, vehicle, meals, business travel, work clothing, training, equipment, telework, dual residence) | The categories are those listed by the administration. [Verified: https://www.service-public.gouv.fr/particuliers/vosdroits/F1989] |
| **"Frais réels vs 10%" comparator**: a running total of tagged costs against the automatic 10% deduction (minimum €509, maximum €14 555 per household member, as stated on the page) → "Opting for frais réels would deduct €X more this year" | [Verified: F1989 above]; amounts change yearly, so they are stored as **dated parameters**, never hard-coded. |
| **Kilometric log**: trips (date, purpose, km, vehicle fiscal horsepower) → the deduction via the official kilometric scale (three distance bands, by fiscal horsepower) | Scale structure and the freeze of the scale for recent years: [Inferred: search-result summaries of impots.gouv and BOFiP material; exact rates not opened]. Store the scale as a yearly parameter table; the electric-vehicle +20% uplift is on F1989 [Verified]. |
| **Proof retention**: receipts tagged for frais réels get "keep until" = end of the 3rd year, because they are not attached to the return but must be kept for 3 years | [Verified: F1989]; the app computes the date conservatively (from the filing year) [Speculative]. |
| **Annual export**: the attached note listing nature and amount of each expense (what the return asks for) as PDF/CSV | [Verified: F1989 says to list the nature and amount of each expense in an attached note] |
| **Notes de frais** (employee expense reports): tag "to be reimbursed", group into a report, export PDF + receipts, track "submitted / reimbursed", reconcile with the reimbursement when it arrives | [Speculative] Note F1989 says employer reimbursements must be added to taxable pay when opting for frais réels, so the two features must share data. [Verified: F1989] |
| **Telework allowance** tracking | The page gives exemption limits for 2024 income only; treat them as a yearly parameter to check, not as current values. [Verified: F1989] |
| Self-employed / micro-entrepreneur mode (v2): revenue ledger, VAT-free threshold watch, URSSAF declaration reminders | [Speculative] — rules not researched here. |

Disclaimer requirement [Speculative]: the app gives **estimates and organisation**, not tax advice; every computed
figure links to its parameter table and source.

### 2.11 Insights & anomaly detection

- Month in review: three plain-language insights at most ("Groceries +22% vs your 3-month median, mostly 4 visits to X").
- **Anomalies**: an unusually large amount for a merchant, a new merchant with a recurring pattern
  (a forgotten trial?), a double charge, a bill that did not arrive when expected, a category drifting up 3 months in a row.
- **"Cost of habit"**: a recurring small spend projected to a year ("€4,20 × 18/month = €907/yr").
- Explanations are always clickable down to the transactions behind them (no black-box numbers) [Speculative].

### 2.12 Gamification that is not childish

[Speculative] Design rule: **reward the behaviours that matter, never the act of opening the app.**

- **Streaks of "clean days"** (every transaction categorized, no overdue bill), not login streaks.
- **"Money days bought back"**: cancelling a €12/mo subscription = "+€144/yr, 1.2 days of your average spending".
- Quarterly **"financial health check"** score made of explainable parts (buffer in months, share of fixed costs,
  goals on track, subscriptions reviewed), each with one concrete next action.
- Calm visual tone: no confetti for spending; a quiet check mark and a sentence.
- Opt-out of all of it in one switch.

### 2.13 Widgets, lock screen, wearables

| Surface | What to show | Platform notes |
|---|---|---|
| iOS Home Screen widget | STS today, next bill, quick-add buttons | Interactive widget buttons (App Intents) since iOS 17 [Unverified: from prior knowledge] |
| iOS Lock Screen | accessoryInline "€38 safe today"; accessoryCircular budget gauge; accessoryRectangular next 2 bills | WidgetKit accessory families `accessoryCircular / Rectangular / Inline` [Unverified: from prior knowledge — the Apple page fetched in this session returned no body] |
| iOS Live Activity | A trip/day budget countdown while travelling | [Unverified: from prior knowledge] |
| Android Home Screen | STS, quick-add, bills list (Glance / RemoteViews) | [Unverified: from prior knowledge] |
| Android lock screen | Same as home widget when allowed | Lock-screen widgets were announced for phones via AOSP after Android 16 (QPR1), and on phones the first implementation was tied to the screensaver while charging/docked. **So a glanceable lock-screen STS is not reliable on Android phones**; plan on a persistent notification or quick-settings tile instead. [Inferred: 2025 press coverage of the Android Developers Blog announcement; shipped behaviour not confirmed] |
| Wear OS / watchOS (v2) | STS complication, quick voice add | [Speculative] |
| Flutter bridge | `home_widget`-style plugin with native widget code per platform | [Unverified: from prior knowledge — the widget UI is native on both platforms whatever the app framework] |

### 2.14 Accessibility

- WCAG 2.2 AA as the target for the app and the web client [Unverified: from prior knowledge; the EU accessibility act may also apply to a commercial product — check in the legal dimension].
- Dynamic type to the largest sizes without truncating amounts; amounts never conveyed by colour only (sign + icon + text).
- Screen-reader labels that read money properly ("minus fourteen euros fifty", not "minus one four comma five zero").
- Reduced motion; haptics optional; one-handed reach (primary actions at the bottom).
- **Cognitive accessibility**: plain-language mode, no jargon ("money left for this month" instead of "available to budget").

### 2.15 Internationalization (FR / EN / AR)

- ICU message format with plurals and gender; French as a first-class language, not a translation.
- **Arabic = RTL**: mirrored layouts, but **numbers and amounts stay LTR** inside RTL text (bidi isolation);
  choice of Western Arabic (0-9) or Eastern Arabic (٠-٩) digits per user.
- Locale-correct money formatting: `14,50 €` (fr-FR), `€14.50` (en-IE), `‏14٫50 €` (ar) via CLDR data.
  [Unverified: from prior knowledge — exact CLDR output must be checked in tests]
- Calendars: Gregorian by default; Hijri date display as an option (useful for Ramadan/Eid budgeting) [Speculative].
- Week start (Monday in FR), date formats, decimal separators all locale-driven; the NL parser must accept both `,` and `.`.

### 2.16 Trust, privacy, data ownership (feature-level)

- Local-first storage, end-to-end encrypted sync as the design target, app lock (biometric/PIN), a "hide amounts" privacy mode for use in public.
- Full export (CSV + JSON + receipts ZIP) at any time; account deletion that really deletes.
- No selling of transaction data; any cloud AI is opt-in per feature. [Speculative — and a GDPR matter for the legal dimension]

---

## 3. Signature "wow" features (the differentiators)

1. **Safe-to-Spend everywhere.** One number, computed from obligations and goals rather than from the
   bank balance, on the widget, the lock screen (iOS), a persistent notification (Android) and the watch.
   It turns a budgeting app into a glance. Few apps show forward-looking spendable money; most show a balance or
   a budget bar. [Speculative]
2. **Obligation radar ("nothing renews behind your back").** Bills, subscriptions, trials, annual renewals,
   price hikes, warranty and return windows in **one timeline**, each with an action button (pay, cancel,
   claim, keep). It merges what Rocket Money does for subscriptions with what no tracker does for warranties. [Speculative]
3. **Say it, snap it, forget it — capture in 2 seconds, offline, in three languages.** Voice/NL capture that
   understands "14,50", "split with Sam", Arabic or French, plus on-device receipt OCR. Without a bank feed,
   capture speed decides whether the data is complete. [Speculative]
4. **Proof vault with legal clocks.** Every receipt carries the dates the law attaches to it: warranty end
   (2-year legal conformity guarantee) and tax-proof retention (3 years for frais réels). "Show me what I can still
   claim" becomes a query. [Verified basis: F11094 and F1989 pages above; the feature itself is Speculative]
5. **Frais-réels autopilot.** The app keeps the "frais réels vs 10%" comparison live all year, logs kilometres,
   and exports the note the tax return asks for, plus notes de frais for employers. This is a concrete French
   pain point that US-built apps ignore. [Inferred: none of the US tools in §1 is priced or described for French tax]
6. **Household money without awkward conversations.** Fair-share rules (by income), private spaces, automatic
   balances, and settle-up by SEPA QR code. It combines a budget app with a Tricount-style group in one ledger,
   so shared spending also counts against personal budgets. [Speculative]
7. **What-if sandbox.** Drag a planned purchase onto the forecast and see the low point, the safe-to-spend change
   and the delay to each goal before buying. [Speculative]
8. **Explainable insights only.** Every insight, score and alert can be opened to the exact transactions and the
   parameter (with its source) behind it. Trust is the product in personal finance. [Speculative]

---

## 4. Prioritisation — MoSCoW mapped to releases

MVP = personal daily driver (manual + import, single user, one device + backup). v1 = public beta / first
commercial version. v2 = differentiation at scale. Each feature appears once.

| # | Feature | MoSCoW | Release | Why there |
|---|---|---|---|---|
| 1 | Accounts (cash, current, savings, card) with opening balances | Must | MVP | Foundation |
| 2 | 1-tap quick add, offline | Must | MVP | Data completeness without a bank feed |
| 3 | Categories (2 levels) + tags + splits | Must | MVP | Foundation |
| 4 | Transfers between accounts | Must | MVP | Balances wrong without it |
| 5 | CSV/OFX import with saved mapping profiles + duplicate guard | Must | MVP | Bank-feed-lite path |
| 6 | Rules engine (manual rules + "make rule from correction") | Must | MVP | Makes import tolerable |
| 7 | Category-limit budgets with alerts | Must | MVP | Default budgeting mode |
| 8 | Bills & subscriptions with due dates + local reminders | Must | MVP | The "reminder" in the brief |
| 9 | Free-trial-ending alerts | Must | MVP | Cheap, high value |
| 10 | Safe-to-spend number (basic formula) | Must | MVP | Signature feature #1, core version |
| 11 | Receipt photo attach (no OCR yet) | Must | MVP | Starts the vault early |
| 12 | Multi-currency per account + manual rate | Must | MVP | EU travel reality |
| 13 | App lock, local encryption, full export | Must | MVP | Trust baseline |
| 14 | FR + EN UI | Must | MVP | Personal use language |
| 15 | Home-screen widget (STS + quick add) | Should | MVP | Glanceability is the hook |
| 16 | Recurring detection → propose bill/subscription | Should | v1 | Needs data history |
| 17 | Price-hike detection | Should | v1 | Built on #16 |
| 18 | Cash-flow forecast 30/60/90 days + low point | Must | v1 | Makes STS credible |
| 19 | Envelope / zero-based budget mode | Should | v1 | Power users; same model |
| 20 | Goals (savings, sinking funds) in the forecast | Should | v1 | |
| 21 | On-device receipt OCR (merchant, date, total) | Should | v1 | |
| 22 | Warranty & return-window records with legal defaults | Should | v1 | Signature #4 |
| 23 | Frais-réels tagging + 10% comparator + yearly parameters | Should | v1 | Signature #5 core |
| 24 | Kilometric log + scale tables | Should | v1 | |
| 25 | Sync across devices (E2E-encrypted target) + web client (read-mostly) | Must | v1 | Commercial requirement |
| 26 | Household: members, shared budgets, split rules, balances, settle-up | Should | v1 | Signature #6 |
| 27 | Natural-language text capture (FR/EN) | Should | v1 | |
| 28 | Lock-screen widgets (iOS) + persistent notification/tile (Android) | Should | v1 | |
| 29 | Cancellation helper cards (deep link, template, timing) | Should | v1 | Obligation radar actions |
| 30 | Insights: month in review + anomaly rules (double charge, unusual amount) | Should | v1 | |
| 31 | Accessibility AA pass + screen-reader money labels | Must | v1 | Before any public release |
| 32 | Arabic + RTL + digit choice | Should | v1 | Brief asks FR/EN/AR |
| 33 | Bank aggregation (open banking) | Could | v2 | Decided in the bank-sync dimension |
| 34 | Voice capture (on-device speech) in FR/EN/AR | Could | v2 | |
| 35 | Receipt line items + VAT lines | Could | v2 | |
| 36 | Notes de frais reports (submit/reimbursed tracking, PDF pack) | Could | v2 | Could move to v1 if commercial B2B2C demand appears |
| 37 | What-if sandbox | Could | v2 | Signature #7 |
| 38 | Debt payoff planner (snowball/avalanche) | Could | v2 | |
| 39 | Trips/events with guest links, Splitwise/Tricount import | Could | v2 | |
| 40 | SEPA QR / payment-link settle-up | Could | v2 | Needs provider verification |
| 41 | E-mail-forward inbox for e-receipts and invoices | Could | v2 | |
| 42 | Health-check score + non-childish gamification | Could | v2 | |
| 43 | Travel mode (auto trip budget, live activity) | Could | v2 | |
| 44 | Wear OS / watchOS complications | Could | v2 | |
| 45 | Hijri calendar display | Could | v2 | |
| 46 | Self-employed (micro-entrepreneur) mode | Won't (now) | later | Separate rule set; research first |
| 47 | Investment portfolio tracking | Won't (now) | later | Different product; dilutes focus |
| 48 | Bill negotiation service | Won't | — | Service business, not software [Speculative] |
| 49 | Credit scoring / lending offers | Won't | — | Regulatory weight and trust conflict [Speculative] |

---

## 5. Key UX flows

### Flow A — 1-tap capture (target under 3 s)
1. Tap widget "+" (or the app's floating button) → keypad opens with the cursor on the amount.
2. Type `14,50` → category chips appear, ranked by time of day, location (if allowed) and recent use.
3. Tap "Lunch" → saved, haptic tick, a toast with "Undo · Add details · Split".
4. Optional: "Add details" → account, note, tag, photo. Default account = last used for this category.

### Flow B — Receipt scan → warranty vault
1. Camera → edge detection → OCR → "Electronics store · 12 Oct · €249,00 · Electronics" (the delivery date defaults to the receipt date and can be changed, because the legal clock runs from delivery).
2. Matches an existing imported transaction (amount ± date) → "Attach to this transaction?" or creates one.
3. Category is "equipment" and the amount is over the threshold → "Track warranty? Legal guarantee until 12 Oct 2028" + optional commercial warranty years.
4. Vault shows the item with two clocks: warranty end and keep-until. Reminder 30 days before the warranty ends.

### Flow C — Subscription price hike → cancel
1. A new occurrence of the "Streaming" series is 18.5% above baseline → notification: "Streaming went up €2,50/mo (+€30/yr)".
2. Open → history chart of the price, cost per year, "last used" prompt.
3. Actions: **Keep** (accepts the new baseline) · **Cancel** → cancel card: the online cancel link, notice period,
   the best date to cancel before the next renewal, "remind me to check the next statement".
4. After cancelling, the series is marked "ended"; the forecast and STS update; "+€192/yr bought back".

### Flow D — Free trial
1. Quick add "trial" (or detected €0/€1 new merchant) → asks end date + price after trial.
2. D-3 and D-1 notifications with "Keep / Cancel" buttons; Cancel opens Flow C step 3.

### Flow E — Household settle-up
1. A shared expense is added by either member with the category's default split (e.g. groceries 60/40).
2. The household tab shows "Sam owes you €83,20" with the transactions behind it.
3. "Settle up" → choose a payment method → a SEPA QR / payment link (v2) or "mark as settled" → a `Settlement` is recorded and the balances zero out.

### Flow F — Monthly close (5 minutes, once a month)
1. Notification on the 1st: "Close October — 7 items to review".
2. Swipe-review uncategorized and auto-categorized items (right = confirm, left = change).
3. Budget recap, one insight, goals progress, subscriptions renewing next month.
4. Envelope mode only: assign the new month's income.

### Flow G — Frais-réels year-end
1. In January a card appears: "Your tagged professional costs: €X; the 10% deduction: €Y → frais réels would save about €Z of taxable income".
2. Review missing receipts (tagged expenses without proof are highlighted).
3. Export the note (nature + amount per expense) and the receipts ZIP; set keep-until dates.

---

## 6. Domain & data model

Principles [Speculative]:
- **Money** = integer minor units (`amount_minor` BIGINT) + ISO 4217 `currency` (CHAR(3)); never floats.
- IDs are **UUIDv7** (sortable, generated offline on the device).
- Every syncable row has `household_id` or `owner_user_id`, `created_at`, `updated_at`, `deleted_at` (soft
  delete for sync tombstones), and a `version` / HLC timestamp for conflict resolution.
- Rules and regulatory numbers live in **dated parameter tables**, never in code.

### 6.1 Entities

| Entity | Key fields | Notes |
|---|---|---|
| **User** | id, locale, home_currency, digits_style, week_start | |
| **Household** | id, name, home_currency | A single user still has a household of one (simplifies sharing later) |
| **HouseholdMember** | household_id, user_id \| guest_name, role (owner/member/viewer/guest), default_share | Guests for trips without accounts |
| **Account** | id, household_id, owner_member_id (null = shared), name, type (cash/current/savings/card/loan/wallet), currency, opening_balance_minor, is_on_budget, institution_ref (nullable, for later bank sync), archived | |
| **Transaction** | id, account_id, date, booked_at, amount_minor (signed), currency, original_amount_minor, original_currency, fx_rate, fx_source, payee_id, description_raw, note, status (pending/cleared/reconciled), source (manual/import/ocr/voice/bank), import_hash, recurring_series_id, transfer_peer_id, visibility (private/shared), is_reimbursable | Transfers = two linked transactions |
| **TransactionSplit** | transaction_id, category_id, amount_minor, memo | Sum of splits = transaction amount (DB check) |
| **Category** | id, household_id, parent_id, name_i18n, icon, colour, kind (expense/income/transfer), is_tax_relevant | Two levels |
| **Tag** / **TransactionTag** | id, name / transaction_id, tag_id | |
| **Payee** (Merchant) | id, display_name, normalization_patterns[], default_category_id, cancel_url, cancel_notes | Feeds the cancellation helper |
| **Rule** | id, household_id, priority, conditions (JSON: field/op/value list), actions (JSON: set category/tag/payee/split/visibility), origin (user/learned), enabled, hit_count | Evaluated on import and capture; ordered |
| **Budget** | id, household_id, mode (limits/envelope/none), period (monthly/weekly/pay_cycle/custom), start_day, currency | |
| **BudgetLine** | budget_id, category_id, period_start, limit_or_assigned_minor, rollover_policy | Envelope "available" is computed, not stored |
| **RecurringSeries** | id, kind (bill/subscription/income/transfer/goal_contribution), payee_id, account_id, category_id, expected_amount_minor, amount_tolerance, rrule (RFC 5545), next_due, status (active/trial/paused/ended), trial_ends_on, price_after_trial_minor, notice_period_days, commitment_ends_on, baseline_amount_minor | One table for bills **and** subscriptions; the `kind` decides UI |
| **Occurrence** | series_id, due_on, expected_minor, matched_transaction_id, state (upcoming/paid/missed/skipped), price_change_pct | Price-hike detection reads this |
| **Reminder** | id, target_type (occurrence/warranty/goal/tax_deadline/custom), target_id, fire_at, lead_time, channel (local push/e-mail), state (scheduled/fired/snoozed/done), quiet_hours | Scheduled on-device; mirrored server-side for multi-device |
| **Receipt** (Attachment) | id, transaction_id (nullable), file_ref, mime, sha256, ocr_text, ocr_fields (JSON), purpose[] (proof/warranty/tax/expense_report), keep_until | `keep_until` = max of purpose-driven dates |
| **Warranty** | id, receipt_id, item_name, purchased_on, delivered_on, legal_until, presumption_until, commercial_until, return_until, claim_status | Defaults from the legal parameter table |
| **Goal** | id, household_id, kind (savings/sinking_fund/debt_payoff/reduce_spending), target_minor, currency, target_date, linked_account_id, strategy, status | Required contribution is computed → RecurringSeries(kind=goal_contribution) |
| **SharedExpense** | transaction_id, payer_member_id, split_method (equal/percent/shares/exact) | |
| **ShareAllocation** | shared_expense_id, member_id, amount_minor | Sum = transaction amount |
| **Settlement** | id, household_id, from_member_id, to_member_id, amount_minor, currency, method (qr/link/cash/mark), settled_at | Balances = allocations − payments − settlements |
| **TaxProfile** | user_id, country, regime (salaried_10pct/frais_reels/…), vehicle_fiscal_hp, is_electric | |
| **TaxTag** / **TripLog** | transaction_id, tax_category, year / date, purpose, km, vehicle | Mileage log |
| **ExpenseReport** | id, user_id, title, status (draft/submitted/reimbursed), submitted_on, reimbursed_transaction_id | Notes de frais |
| **ParameterTable** | key (e.g. `fr.ir.abattement10.min`), valid_from, valid_to, value, source_url | Yearly tax scales, legal durations |
| **FxRate** | base, quote, date, rate, source | |
| **Insight** | id, kind, period, payload, explained_by (transaction ids), dismissed | |

### 6.2 Relationships (simplified)

```
Household 1─* HouseholdMember
Household 1─* Account 1─* Transaction 1─* TransactionSplit *─1 Category
Transaction *─* Tag
Transaction *─1 Payee            Payee 1─* RecurringSeries 1─* Occurrence 0..1─1 Transaction
Rule (household-scoped) ──applies to──> Transaction
Budget 1─* BudgetLine *─1 Category
Transaction 1─* Receipt 0..1─1 Warranty
Goal 1─0..1 RecurringSeries(kind=goal_contribution)
Transaction 0..1─1 SharedExpense 1─* ShareAllocation *─1 HouseholdMember
Household 1─* Settlement
Reminder ──polymorphic──> Occurrence | Warranty | Goal | custom
ParameterTable ──read by──> STS engine, tax comparator, warranty defaults
```

### 6.3 Derived values (computed, never stored as truth)

| Value | Derived from |
|---|---|
| Account balance | opening balance + transactions |
| Envelope available | assigned − spent + rollover |
| Household balance per member | allocations − amounts paid − settlements |
| Safe-to-spend | balances, upcoming Occurrences, goal contributions, buffer, ParameterTable (none needed today) |
| Forecast series | balances + Occurrences + modelled variable spending |
| Receipt keep_until | max(warranty legal/commercial end, tax retention end, user choice) |

---

## 7. Open design questions (for the decision phase)

1. Envelope mode in MVP or v1? Recommended: v1 — category limits cover most people; the model already supports both. [Speculative]
2. Household in v1 implies sync + accounts + conflict resolution in v1; that is the largest engineering cost in this list. [Inferred: features 25–26 depend on multi-user sync]
3. How much AI: rules + on-device models first, cloud LLM only opt-in for NL parsing edge cases and insights wording. [Speculative]
4. Android lock-screen gap: accept a persistent notification / quick-settings tile as the Android equivalent. [Inferred: §2.13]
5. Tax features as a paid "France pack" in a commercial version? [Speculative]

---

## Sources

### Opened in this session (Verified)
- YNAB pricing — https://www.ynab.com/pricing
- Monarch pricing — https://www.monarch.com/pricing
- Splitwise Pro — https://www.splitwise.com/pro
- Service-public, frais réels (salaried) — https://www.service-public.gouv.fr/particuliers/vosdroits/F1989
- Service-public, garantie légale de conformité — https://www.service-public.gouv.fr/particuliers/vosdroits/F11094

### Search-result summaries only (Inferred — page not opened)
- Rocket Money pricing and cancellation (third-party guides): https://www.fincomparelab.com/guides/rocket-money-pricing/ , https://getfinny.app/blog/rocket-money-pricing-2026
- Tricount features and the end of Premium: https://www.tricount.com/en/features , https://tricount.com/en/what-happened-with-premium , https://help.tricount.com/articles/add-bunq-payments-as-expenses
- French "résiliation en trois clics" (decree 2023-417, from 1 June 2023): https://www.blogdumoderateur.com/resilier-contrat-en-ligne-juin-2023/ , https://blog.avocats.deloitte.fr/la-resiliation-en-3-clics-nouvel-outil-au-service-des-consommateurs/ , https://www.inc-conso.fr/node/15413
- Kilometric scale freeze and structure: https://www.legifiscal.fr/actualites-fiscales/4094-ir-revalorisation-2025-bareme-indemnites-kilometriques.html?pdf=1 , https://www.moneyvox.fr/impot/actualites/108318/impots-2026-le-bareme-kilometrique-est-gele-pour-la-3eme-annee-consecutive
- Android lock-screen widgets after Android 16 QPR1 (press coverage): https://9to5google.com/2025/03/06/android-phone-lockscreen-widgets/ , https://tomsguide.com/phones/android-phones/googles-bringing-lock-screen-widgets-back-to-android-phones-this-summer-heres-what-we-know

### Fetched with no usable body
- Apple WidgetKit accessory widgets — https://developer.apple.com/documentation/widgetkit/creating-accessory-widgets-and-watch-complications (the page body was not returned; the related claims are graded Unverified)
