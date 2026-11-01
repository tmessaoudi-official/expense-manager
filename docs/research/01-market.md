# 01 — Market & Competition + Innovation Gaps

> Research date: 2026-11-01. Scope: personal expense / budget / bill-reminder apps, global and France/EU.
> Grading: **[Verified: url]** = source opened in this research session; **[Inferred: basis]**; **[Unverified: prior knowledge]** = recalled, not checked today.
> Note on sources: several third-party pages returned by search are affiliate or AI-generated comparison sites. Where only those exist, the claim is graded Inferred or Unverified, not Verified.

---

## 1. Executive summary

1. **The market is crowded at the top, thin in the middle.** US "premium aggregator" apps (YNAB, Monarch, Copilot, Rocket Money) cost roughly **$9–15/month or ~$100–110/year** and focus on North America. [Verified: ynab.com/pricing, monarch.com/pricing, copilot.money/pricing, help.rocketmoney.com]
2. **The 2026 trend is agentic AI.** Incumbents now sell AI that *acts*, not only one that answers. Rocket Money launched **Rowan**, an agent reached by SMS, inside a **$15/month Premium+** tier (25 Aug 2026; built with Anthropic). Copilot ships a "money assistant" that asks before it acts. Revolut launched **AIR** (UK, April 2026). ChatGPT added personal-finance features (May 2026; US rollout to Free/Go users from Oct 2026). [Verified: help.rocketmoney.com; Inferred: multiple press reports on Rowan, AIR and ChatGPT, listed in sources; the OpenAI page itself returned 403]
3. **France and the EU are under-served by "premium" budgeting.** French apps are either freemium aggregators (Bankin', Linxo), which have moved toward B2B data and payments, or wealth trackers that treat budgeting as a side feature (Finary). Bankin' Plus starts at **€3.33/month**. Finary only includes budgeting from its **€149.99/year "Plus"** tier. [Verified: bankin.com/fr/pluspro.html, finary.com/fr/pricing, linxo.com]
4. **Open-banking access for indie developers became harder in the EU.** GoCardless Bank Account Data, the free-tier aggregator many indie and open-source apps relied on, **no longer accepts new accounts**. Actual Budget now points European users to **Enable Banking**. [Verified: actualbudget.org/docs/advanced/bank-sync/] One secondary summary dates the closure to July 2025. [Inferred: search summary of Actual's GoCardless doc page]
5. **Open-source and local-first is a real, loved niche.** Actual Budget (MIT, ~29.9k★) and Firefly III (AGPL-3.0, ~25.1k★) both have strong followings. Their weak points are mobile UX and bank sync that requires a self-hosted server. Cashew (Flutter, GPL-3.0, ~4.8k★) shows that a polished, manual-first mobile app can earn a large audience. [Verified: github.com/actualbudget/actual, github.com/firefly-iii/firefly-iii, github.com/jameskokoska/Cashew]
6. **Mint's shutdown (23 March 2024) is the cautionary tale.** A free, ad- or affiliate-funded app was folded into its owner's other product (Credit Karma), and the budgeting features users depended on were dropped. This left millions of "Mint refugees" and led to the paid-subscription wave. [Inferred: consistent across NerdWallet, Engadget and WalletHub results; Intuit's own release not opened]

**Biggest gaps found, detailed in §5:**
- Reminders and bills that act *before* money moves: due dates, notice periods, the French *résiliation en 3 clics* right.
- Trustworthy local-first privacy combined with an *optional* AI layer.
- A forward-looking "safe to spend until payday" forecast built from detected recurrences.
- France-specific objects that no global app models: Sécurité sociale and mutuelle reimbursements, meal-voucher cards, tax and CAF schedules.
- Graceful degradation when bank sync breaks.

---

## 2. Competitor matrix: global

| App | Platforms | Price (current) | Bank sync | Standout | Weaknesses / complaints | Grade |
|---|---|---|---|---|---|---|
| **YNAB** | iOS, Android, Web, watchOS | $14.99/mo or $109/yr; 34-day trial, no card required | Direct import for select banks in US, CA, UK **and EU**; file import elsewhere | Zero-based "give every dollar a job" method with a strong community and education | Price; steep learning curve; prices in USD only [Unverified: prior knowledge of reviews] | [Verified: ynab.com/pricing] |
| **Monarch** (rebranded domain monarch.com) | Web, iOS, Android | Core $14.99/mo or $99.99/yr; Plus tier exists (price not shown); first week free | Aggregators not named on the pricing page | Household sharing; Plus tier adds forecasting and investments; tax exports; will preparation via partner | Sync reliability complaints on Reddit (users report re-linking) [Inferred: Reddit mirrors dated Jan 2026]; US-centric | [Verified: monarch.com/pricing] |
| **Copilot Money** | iPhone, iPad, Mac, Web; **no native Android** | $9/mo or $109/yr; one plan, "no AI credits" | 16,000+ banks (US) | Polished design; "money assistant" that asks before acting; six personalities plus a "Roast me" mode; Amazon and Venmo enrichment; households "coming this fall" | Apple-first: Android users use the web app [Inferred: multiple comparison sites; no native Play listing found]; US only | [Verified: copilot.money/pricing] |
| **Rocket Money** | iOS, Android, Web | Free; Premium is pay-what-you-choose on a sliding scale; **Premium+ $15/mo** includes Rowan AI agent, bill negotiation and a high-yield savings account | US aggregators | Subscription detection and cancellation; bill negotiation (35–60% success fee on first-year savings, or included in Premium+); **Rowan** SMS agent (Aug 2026) | Fee surprises on negotiation [Inferred: reviewer reports]; US only | [Verified: help.rocketmoney.com] |
| **Lunch Money** | Web-first; limited mobile | About $10/mo or ~$100/yr pay-what-you-want (third-party reports; official page returned 403) | Plaid (US/CA) plus others [Unverified] | Multi-currency, rules, API, developer-friendly; suits digital nomads | Mobile app limited [Inferred: fincomparelab review title]; small team | [Inferred: costbench and getfinny 2026 reviews] |
| **Emma** (UK) | Android, iOS, web beta for paying users | Basic £0; Plus £4.99/mo (£41.99/yr); Pro £9.99/mo (£83.99/yr); Ultimate £14.99/mo (£124.99/yr) | UK open banking; FCA-regulated | Subscription tracking, cashback, credit score, rent reporting, savings/ISA and investing ("super-app"); AI integrations "coming soon" | UK-centric; budgeting gated behind Plus; feature creep toward financial products | [Verified: emma-app.com/plans/compare-emma-plans] |
| **Spendee** | iOS, Android (web "soon" for budgets) | Free (1 wallet, 1 budget); Plus $1.99/mo ($14.99/yr); Premium $5.99/mo ($35.99/yr); 7-day trial | Bank sync on all plans (list of supported banks) | Shared wallets; low price | Older product; shallow forecasting [Unverified] | [Verified: spendee.com/pricing] |
| **Wallet by BudgetBakers** (CZ) | iOS, Android, Web | Free trial; prices not on homepage | "15K+ bank connections"; also sells its own **AISP banking API** | 14M+ downloads, 50 languages; ISO 27001; **MCP AI assistant integration and REST API**; ShareCost for splitting | Pricing opaque; reviewers report sync breakage [Unverified] | [Verified: budgetbakers.com] |
| **Money Lover** (VN) | iOS, Android | Free download; premium not listed | "Automatic" entry claimed; bank sync not explicit | Bill and recurring reminders before due dates; travel mode with FX; debts and loans | Ads and upsell [Unverified] | [Verified: moneylover.me] |
| **Splitwise** | iOS, Android, Web | Free tier limited to about 4 new expenses/day with ads; Pro about $4.99/mo or $39.99/yr (store price varies by country) | None | De-facto standard for shared expenses | Free tier capped since about late 2023, a strong source of user resentment | [Inferred: App Store listing quoted in search results; getfinny 2026 pricing] |
| **Revolut** (neo-bank built-in) | iOS, Android | Included with account | Own account only, plus some external accounts | Category budgets and analytics; **AIR** AI assistant (UK, April 2026) for insights, subscriptions, card freeze | Covers only one bank's slice of spending (vendor estimate 40–70% for multi-account users) [Inferred: vendor guide] | [Verified: help.revolut.com is cited in search; Inferred for AIR via crowdfundinsider] |
| **ChatGPT personal finance** | ChatGPT apps | Rolling out to US Free/Go users from Oct 2026 | US bank connection [Unverified: provider not confirmed] | General-purpose LLM with account context, a horizontal threat | US-only; trust and accuracy concerns | [Inferred: search results; openai.com page returned 403] |
| **BudgetGPT** (startup) | iOS | — | — | Chat-first budgeting (launched 21 Jan 2026) | Press-release claims only | [Inferred: GlobeNewswire release in search results] |
| **Cleo** | iOS, Android | — | US | "Autopilot" (Feb 2026) executes pre-approved actions | US; aimed at young users and cash advances | [Inferred: search summary] |
| **Quicken** | Desktop, mobile | — | US | "Quicken AI" chat (summer 2026) | Legacy UX | [Inferred: search summary] |

---

## 3. Competitor matrix: France / EU

| App | Model | Price | Notes | Grade |
|---|---|---|---|---|
| **Bankin'** (Paris) | Freemium aggregator, iOS, Android and Chrome extension | Plus from **€3.33/mo**, Pro from **€8.33/mo** | Plus: custom categories, unlimited export, link a refund to an expense, transaction checking, notifications by theme, budget history, month-over-month comparison. Pro: business accounts. Also sells B2B data and cashback | [Verified: bankin.com/fr/pluspro.html, bankin.com] |
| **Linxo** | Historic budget app, now **Linxo Lab** (free); company focused on B2B "Hub Data" aggregation and payments | Linxo Lab is free | 1,700+ bank connectors; payment institution licensed by the ACPR; ISO 27001. A Crédit Agricole logo appears on the site, but ownership is not stated there | [Verified: linxo.com]; ownership by Crédit Agricole [Unverified: prior knowledge] |
| **Finary** | Wealth tracker, with budgeting as an add-on | Free (2 syncs); Lite €54.99/yr; **Plus €149.99/yr** (budget, fee scanner, family mode); Pro €349.99/yr | Budgeting not in Free or Lite; revenue also from crypto fees (0.99–1.49%) | [Verified: finary.com/fr/pricing] |
| **Emma / YNAB / Wallet / Spendee** | Foreign apps with EU sync | see §2 | YNAB lists EU direct import; Emma is UK-centric | [Verified: per §2] |
| **Actual Budget** (OSS) | Local-first; EU sync via **Enable Banking** (GoCardless closed to new accounts) | Free, self-host or PikaPods | Bank credentials sit on the self-hosted server, outside E2E encryption [Inferred: search summary of Actual docs] | [Verified: actualbudget.org, actualbudget.org/docs/advanced/bank-sync/] |
| **Firefly III** (OSS) | Self-hosted PHP double-entry bookkeeping | Free | Rules, recurring transactions, piggy banks, REST API, Docker; no official mobile app mentioned | [Verified: github.com/firefly-iii/firefly-iii] |

**Market reading for France.** The local leaders came out of the 2010s PSD2 aggregation wave. Their monetisation drifted to B2B data, cashback and financial products, which left the consumer UX stagnant. There is no French app that plays the role Copilot or YNAB play in the US: a premium, design-led, method-driven budgeting app. [Inferred: from the offers verified above; no French "premium method" app found]

---

## 4. Open-source and local-first niche (relevant for architecture)

| Project | Stack | License | Stars | Bank sync | Lesson |
|---|---|---|---|---|---|
| Actual Budget | Node/TS, Electron, web; local-first sync with optional E2E | MIT | ~29.9k | Enable Banking (EU), SimpleFIN (NA), Akahu (NZ), Pluggy (BR); GoCardless legacy | Local-first plus optional sync is credible and popular. Mobile is web-based, not native | [Verified: github + docs] |
| Firefly III | PHP (Laravel-style) | AGPL-3.0 | ~25.1k | Separate data importer | Power users accept self-hosting; UX is "accountant-grade" | [Verified: github] |
| Cashew | **Flutter** + Drift (SQLite) + Firebase | GPL-3.0 | ~4.8k | None (CSV and Google Sheets import, Drive backup) | A manual-first **Flutter** app with subscriptions, upcoming payments, debts and reminders can be loved without bank sync. Highly relevant to this developer's toolchain | [Verified: github.com/jameskokoska/Cashew] |
| Buckwheat | Android | GPL-3.0-only | ~790 | None | "Daily allowance" UX (money ÷ days until next income) is a simple, sticky idea | [Inferred: F-Droid and AlternativeTo via search] |

---

## 5. Shutdowns and failures, and what to learn

| Event | What happened | Lesson | Grade |
|---|---|---|---|
| **Mint → Credit Karma** (announced Nov 2023, shut 23 Mar 2024) | Intuit consolidated Mint into Credit Karma. Budgets and custom categories were not carried over, and data was deleted after shutdown | A free app paid for by affiliate revenue makes the user the product. Provide **export and data ownership** from day one; a sustainable paid model builds trust | [Inferred: NerdWallet, Engadget, WalletHub results agree] |
| **GoCardless Bank Account Data** stops new signups | The free EU AIS tier that indie apps used is closed to new accounts | **Do not build the core value on one aggregator**: abstract the provider and keep manual or smart input first-class | [Verified: actualbudget.org/docs/advanced/bank-sync/] |
| **Splitwise free-tier cap** (~late 2023) | Daily expense limit introduced on the free tier | Changing a free tier after the fact drives users to look for alternatives, an opening for fair pricing | [Inferred: search results] |
| **FiDA (EU open finance)** | Reported withdrawn in Feb 2025, then kept as pending. Status in late 2026 not confirmed | Do not count on open *finance* (insurance, pensions) data rights soon | [Verified: journaldunet.com article dated 11 Feb 2025; Inferred: later reprieve per Vixio, finAPI and Linklaters results] |
| **PSD3 / PSR** | Provisional agreement 27 Nov 2025; ECON approval reported 5 May 2026; includes a **permission dashboard** at banks and bans on obstacles to data access | Open banking in the EU gets *more* reliable over time, so bank sync is a good later-phase add-on | [Inferred: Hogan Lovells and other law-firm results; final texts not opened] |

---

## 6. Gaps and genuinely innovative angles

Grades here are mostly **[Speculative]** (design judgment) unless marked otherwise.

### 6.1 Reminders that act before money moves (the "reminder" half nobody owns)
- Current apps *track* bills after they post. Money Lover and Emma Plus send bill reminders, and Rocket Money lists upcoming bills. Very few model the **contract lifecycle**: renewal date, notice period, price-increase date, end of trial, warranty expiry. [Inferred: feature lists verified above]
- **French lever:** since 1 June 2023, professionals in France must offer online termination "in three clicks" (decree no. 2023-417). An app that detects a subscription, warns "trial ends in 3 days / notice period starts", and **deep-links to the merchant's termination page** works within the law, with no negotiation fees. [Inferred: legal blogs (Deloitte Avocats, Que Choisir) in search results; Légifrance not opened]
- Calendar-native reminders (ICS export, OS notifications, widgets) instead of in-app-only reminders.

### 6.2 Recurrence engine plus "safe-to-spend" forecasting
- Detect recurring items from any input source (bank, CSV, receipts, manual entry). Project the cash curve to the next payday and show a **daily allowance** (Buckwheat's idea, fed by a proper recurrence model). Copilot and Monarch Plus offer forecasting at $9–15/month and only in the US. [Verified for pricing; Speculative for differentiation]

### 6.3 Local-first privacy with an optional, bounded AI layer
- Market proof that people want this: Actual (~29.9k★) and Firefly (~25.1k★). Copilot advertises "no training external AI on your data". [Verified]
- Angle: an **on-device ledger (SQLite) with E2E sync**. AI is opt-in, sees only redacted or aggregated data, and runs in **propose-then-confirm** mode, the same pattern Copilot ("asks before acting") and Rowan ("human oversight where required") use. This fits GDPR expectations in the EU. [Speculative]

### 6.4 Smart input without bank sync (removes the aggregator dependency)
- Receipt and invoice OCR, PDF bank-statement import, e-mail or forwarded-invoice parsing, share-sheet capture of payment notifications, and Android notification listening (on-device). Cashew shows manual-first can win; smart capture closes the friction gap. [Inferred: Cashew's traction plus the GoCardless closure]
- Apple **FinanceKit** is available only in the US (Apple Card, Cash, Savings) and the UK (open-banking data, iOS 18.4+), and needs an entitlement, so it is **not an option for France** today. [Inferred: Apple developer page as summarised in search results; page not opened directly]

### 6.5 France-specific objects (a moat global apps will not build)
- **Health:** link a medical expense to its *Assurance Maladie* reimbursement plus the *mutuelle* top-up, and track what is still pending. Bankin' already offers "link a refund to an expense", which validates the need. [Verified: bankin pluspro page; extension is Speculative]
- Meal-voucher cards (titres-restaurant) as a separate restricted "wallet". Income tax withheld at source and *taxe foncière* / *taxe d'habitation* schedules. CAF benefits. Regulated savings accounts (Livret A, LEP) as goals. French bank-statement CSV and OFX dialects. [Speculative; product objects from prior knowledge, Unverified]

### 6.6 Shared and household money done fairly
- Splitwise's capped free tier, plus Copilot announcing households only for fall 2026, shows that shared budgets are still unsolved at a fair price. Combine splitting with budgets: "who owes whom" feeds directly into each person's safe-to-spend figure. [Verified: Copilot pricing page; Inferred: Splitwise]

### 6.7 Resilience UX for broken sync
- Users' top complaint across aggregator apps is silent partial syncs and re-linking. [Inferred: Reddit mirrors and comparison blogs, low quality] Design for it: each account shows a freshness indicator, a reconciliation prompt ("balance differs by €12.40, add missing?"), and falls back to statement import. [Speculative]

### 6.8 Agent interfaces, built carefully
- BudgetBakers already exposes an **MCP** integration and Rowan works over SMS: "your finances as a tool for an AI agent" is arriving. A *local* MCP server over the user's own ledger (privacy-preserving) is a credible differentiator for a developer audience. [Verified: budgetbakers.com mentions MCP; Speculative for differentiation]

---

## 7. Pricing benchmarks (for later commercialisation)

| Segment | Price point | Examples |
|---|---|---|
| Freemium FR/EU aggregator | €3–8/mo | Bankin' Plus €3.33, Pro €8.33 [Verified] |
| Low-cost global | $15–36/yr | Spendee Plus $14.99/yr, Premium $35.99/yr [Verified] |
| UK super-app | £42–125/yr | Emma Plus to Ultimate [Verified] |
| US premium budgeting | $99–109/yr | YNAB $109, Monarch $99.99, Copilot $109 [Verified] |
| AI agent tier | $15/mo | Rocket Money Premium+ (Rowan) [Verified] |
| Wealth plus budgeting FR | €149.99/yr | Finary Plus [Verified] |

Implication: a France-first premium app could plausibly sit at **€3–6/month or €30–50/year**, below US premium and above freemium aggregators, with the AI or bank-sync tiers priced separately to cover aggregator and model costs. [Speculative]

---

## 8. Positioning hypothesis

> **"The money app that tells you *before*."** It is local-first and private by default. Bills, renewals and trials come with reminders and legal-termination shortcuts. A recurrence-driven safe-to-spend forecast is the main screen. Smart capture works without a bank, and bank sync is an optional add-on. It understands French money (Sécu and mutuelle, meal vouchers, tax and CAF). AI proposes and you confirm.

[Speculative — design judgment from the gaps above]

---

## 9. Open questions / to verify next
- Lunch Money official pricing (page returned 403). [Unverified]
- Monarch Plus price, and Copilot's Android plans. [Unverified]
- BudgetBakers Wallet and Money Lover current premium prices (not on their homepages). [Unverified]
- FiDA's status in late 2026 and the final PSD3/PSR application dates. [Unverified]
- Enable Banking and other EU aggregators (Powens, Bridge, Tink, Yapily) pricing for indie developers. This belongs to the bank-sync research dimension.
- Exact scope of the *résiliation en 3 clics* rule, checked on Légifrance (Code de la consommation).

---

## Sources

**Opened in this session (Verified):**
- https://www.ynab.com/pricing
- https://www.monarch.com/pricing (redirected from monarchmoney.com/pricing)
- https://www.copilot.money/pricing
- https://help.rocketmoney.com/en/articles/2217739-how-much-does-rocket-money-cost
- https://emma-app.com/ and https://emma-app.com/plans/compare-emma-plans
- https://www.spendee.com/pricing
- https://budgetbakers.com/
- https://moneylover.me/
- https://bankin.com/ and https://bankin.com/fr/pluspro.html
- https://www.linxo.com/
- https://finary.com/fr/pricing
- https://actualbudget.org/ and https://actualbudget.org/docs/advanced/bank-sync/
- https://github.com/actualbudget/actual
- https://github.com/firefly-iii/firefly-iii
- https://github.com/jameskokoska/Cashew
- https://cashewapp.web.app/
- https://www.journaldunet.com/business/1539123-open-banking-en-peril-bruxelles-abandonne-fida-sous-la-pression-des-lobbys-financiers/

**Seen via search results only (Inferred):**
- Rocket Money Rowan: https://thepaypers.com/fintech/news/rocket-money-launches-rowan-ai-agent-for-personal-finance , https://claude.com/customers/rocket-money , https://www.rocketmoney.com/learn/personal-finance/what-is-rowan
- Rocket Money negotiation FAQ: https://help.rocketmoney.com/en/articles/6676442-bill-negotiations-faq
- BudgetGPT: https://www.globenewswire.com/news-release/2026/01/21/3223241/0/en/budgetgpt-launches-ai-native-budgeting-app-that-lets-users-chat-directly-with-their-money.html
- ChatGPT finance: https://openai.com/index/personal-finance-chatgpt/ (returned 403)
- Revolut AIR: https://www.crowdfundinsider.com/2026/04/272459-digital-bank-revolut-introduces-ai-tools-to-streamline-money-management/ ; Revolut budgeting help: https://help.revolut.com/help/app-features/budgeting-and-analytics/how-can-i-control-my-spending-with-the-budgeting-feature
- Mint shutdown: https://www.nerdwallet.com/article/finance/mint-app-closing-what-it-means-how-to-pick-a-new-budget-service , https://www.engadget.com/intuit-is-closing-down-mint-its-popular-free-budget-tracking-app-054145229.html , https://wallethub.com/edu/b/what-happened-to-mint/151868
- Splitwise: https://apps.apple.com/us/app/splitwise/id458023433 , https://getfinny.app/blog/splitwise-pricing-2026
- Lunch Money: https://costbench.com/software/personal-finance/lunch-money/ , https://getfinny.app/blog/lunch-money-review-2026
- Copilot Android: https://feedback.copilot.money/feature-requests/p/copilot-android-app
- Apple FinanceKit: https://developer.apple.com/financekit
- FiDA: https://www.vixio.com/insights/pc-fida-lives-see-another-day-european-commission-backs-away , https://www.finapi.io/en/fida-regulation-status-pending/
- PSD3/PSR: https://www.hoganlovells.com/en/publications/final-texts-for-psd3-and-psr-awaited-as-european-parliament-and-council-of-eu-announce-provisional
- Résiliation 3 clics: https://blog.avocats.deloitte.fr/la-resiliation-en-3-clics-nouvel-outil-au-service-des-consommateurs/ , https://www.quechoisir.org/actualite-contrats-d-assurance-la-resiliation-en-3-clics-entre-en-vigueur-n108126
- Buckwheat: https://apt.izzysoft.de/fdroid/index/apk/com.danilkinkin.buckwheat?repo=main
- Sync complaints (low quality): https://envelopebudgeting.com/articles/budgeting-app-syncing-problem
