# Finance App: Project Plan

An Australian-focused personal finance and investing app. It combines education, calculators and portfolio tools (Pearler-style) with an AI Q&A assistant. ETFs and long-term investing come first, then superannuation, high-interest savings accounts (HISAs), interest rates and global banks.

---

## Overview

- **Core focus:** personal finance and long-term ETF investing.
- **Wider coverage:** super, HISAs, the RBA cash rate and global central banks, and mortgage vs offset vs invest decisions.
- **AI assistant:** answers finance questions from trusted sources and can run the app's calculators.
- **Difficulty:** medium to hard, mainly because the scope is broad. Build it in phases.

### Important legal limit

In Australia, **personal financial advice requires an Australian Financial Services Licence (AFSL)**. Design the app as **general information and tools only**:

- Show a **general advice warning** throughout, including in every AI answer.
- Never tell a user to buy, sell or hold a specific product ("you should buy VAS").
- Frame Q&A as education: explain concepts, trade-offs and how the calculators work.
- Seek proper legal advice before any public or commercial launch.

This decision affects the whole design, so it comes first.

---

## Phase 1: Core investing tools (the Pearler part)

### 1. Calculators (start here: cheap to build and useful immediately)
- **Compound growth / FIRE calculator:** starting balance, regular contributions, expected return, fees and inflation, with the target date or FIRE number.
- **Fee impact calculator:** the cost of different management fees (MERs) over 10–40 years (for example 0.07% vs 0.50% vs 1.5%).
- **Dollar-cost averaging (DCA) planner:** brokerage cost per trade, best buy frequency for a given brokerage, and target-allocation rebalancing ("buy $X of A and $Y of B to get back on target").
- **Tax tools:** capital gains tax (CGT) estimate with the 50% discount after 12 months, and franking credit gross-up.

### 2. Portfolio tracker
- Enter trades manually or import a broker CSV (CommSec, Pearler, Stake, SelfWealth, Vanguard Personal Investor).
- Holdings, cost base, returns (time-weighted and money-weighted), dividends and distributions, franking credits.
- Asset allocation by holding, region, sector and asset class.
- CGT report by financial year, with parcel matching (FIFO or a specific parcel).

**Real auto-investing**, which Pearler offers, needs a broker partnership and its own licensing. Leave it out of scope and focus on tracking and planning.

### 3. ETF explorer
- Key facts per ETF: MER, fund size, distribution yield and frequency, domicile, index tracked, number of holdings.
- **Overlap tool:** for example how much VGS and IVV overlap, or A200 and VAS.
- Comparison view and simple screener filters (region, asset class, MER, ethical screen).
- **Data:** issuer websites and product disclosure statements (Vanguard, BetaShares, iShares, VanEck, SPDR), ASX data, and a market-data API for prices (for example EOD Historical Data or Twelve Data; check licensing terms).

## Phase 2: Wider coverage

### 4. Superannuation
- Fund fee and performance comparison using APRA data and the YourSuper comparison tool data.
- Contribution calculators: salary sacrifice, concessional and non-concessional caps, carry-forward unused cap amounts, the government co-contribution, spouse contributions.
- "Super vs outside super" comparison for extra savings (tax rate vs access age).

### 5. HISAs and savings
- Comparison table of rates and conditions (deposit requirements, transaction requirements, intro vs ongoing rates, balance caps).
- "Effective rate" logic: what you actually earn if you don't meet the bonus conditions.
- Rates change often, so plan for scheduled scraping or manual updates, and show a "last checked" date on every rate.

### 6. Interest rates and macro
- **RBA cash rate**, decision history and next meeting date (RBA statistical tables, F1 series).
- **Global central bank rates** (Fed, ECB, BoE, BoJ, RBNZ, and others) and inflation data from the FRED API.
- **Mortgage vs offset vs invest calculator:** compare putting spare cash in an offset account against investing it, given the interest rate, tax rate and expected return.

## Phase 3: AI Q&A assistant

- **Retrieval over trusted, curated content (RAG):** ASIC Moneysmart, ATO, APRA and RBA pages, plus the app's own written guides. Answers should cite their sources.
- **Tool use:** the assistant can call the app's calculators. For example, "how much would I have at 60 if I invest $500 a month?" runs the real compound calculator instead of the model doing the maths.
- **Guardrails:** the general advice warning goes in the system prompt and the UI; the assistant refuses personal product recommendations and redirects to education; current rates and figures always come from the app's data, not the model's memory.
- **Evaluation:** a test set of finance questions with expected answers, checked for accuracy, citations and staying within the general-advice limit.

## Stack and repository layout

- **Web app:** Next.js (TypeScript) with Tailwind.
- **Backend and auth:** Supabase (Postgres, auth, row-level security for each user's portfolio).
- **Data jobs:** Python scripts on a schedule (GitHub Actions or Supabase cron) for ETF data, rates and macro series.
- **AI:** the Claude API with tool use, plus pgvector in Supabase for retrieval.
- **Mobile:** start as a responsive web app or PWA; consider React Native later.

```
finance-app/
  apps/web/                  # Next.js app
    app/calculators/
    app/portfolio/
    app/etfs/
    app/super/
    app/savings/
    app/rates/
    app/ask/                 # AI assistant
  packages/finance-core/     # pure TypeScript calculation library (fully unit tested)
  data-jobs/                 # Python: ETF data, HISA rates, RBA/FRED series
  supabase/migrations/
  content/guides/            # written education content (also used for retrieval)
  .github/workflows/
```

Keep all financial maths in `packages/finance-core` with thorough unit tests. The UI and the AI assistant both call the same tested functions.

## Milestones

1. **Calculators** (compound/FIRE, fee impact, DCA, CGT) with unit tests.
2. **Portfolio tracker** with manual entry, then CSV import.
3. **ETF explorer** and overlap tool.
4. **User accounts** and saved portfolios (Supabase).
5. **Super tools.**
6. **HISA comparison and rates/macro dashboard.**
7. **AI Q&A assistant** with retrieval, calculator tools and an evaluation set.
8. **Polish:** PWA, notifications (for example "RBA decision today"), dark mode.

## Risks and notes

- **Regulation:** stay within general information; get legal advice before launching publicly.
- **Data licensing:** check terms for price data and for scraping issuer and bank sites.
- **Stale data:** show "as at" dates everywhere; never let the AI state rates from memory.
- **Calculation errors:** they cost user trust, so unit tests on the finance core are essential.
- **Security and privacy:** portfolio data is sensitive; use row-level security, no unnecessary personal data, and encryption at rest.
