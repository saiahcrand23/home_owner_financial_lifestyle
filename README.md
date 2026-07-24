# Financial Goal Projections

A single-page, offline-capable calculator for planning a home purchase, monthly budget, house payoff strategy, retirement savings, and short-term savings goals — all in one place, with every number feeding the next.

**Live app:** https://saiahcrand23.github.io/home_owner_financial_lifestyle/

## Why this exists

Most budgeting tools either need an account and a server, or are a spreadsheet that's hard to hand off to someone else to review. This is neither: it's one `index.html` file with no backend, no build step, and no external dependencies. Everything runs in your browser, your numbers stay on your device, and you can hand a snapshot of your plan to someone else (a spouse, a partner, yourself on another computer) as a single JSON file.

## Calculators

The app is organized into five tabs. Data flows forward — numbers you enter in an earlier tab automatically feed the calculators after it — but every auto-filled field stays editable if you want to override it.

1. **Savings Goals** — Sequence short-term savings priorities (paying off loans, building an emergency fund, saving for a house down payment, or just a general high-yield savings account) against your actual contribution schedule (paychecks, bonuses, one-time deposits). Produces a month-by-month timeline showing when each goal is hit.
2. **Home Buying** — Enter a home price, down payment, mortgage terms, and closing costs to get a full monthly housing cost breakdown (principal & interest, property tax, insurance, HOA, PMI, maintenance), total cash needed at closing, and an amortization schedule. PMI automatically drops off once equity crosses 20%.
3. **Surplus** — Enter take-home pay, tithe, and custom budget categories (add as many as you need) to see your monthly/quarterly/annual surplus after expenses. Housing cost auto-fills from the Home Buying tab. Split the surplus between discretionary spending and extra house payoff by percentage or dollar amount.
4. **House Payoff** — Reuses the mortgage amortization engine from Home Buying and layers in your recurring payoff allocation plus any one-off extra payments by year, showing the revised payoff date and total interest saved.
5. **Retirement** — Projects your retirement balance from current age to retirement age given a contribution rate, employer match, and expected return. Supports one-time contribution changes at specific future ages, an inflation-adjusted ("today's dollars") view alongside the nominal projection, and a balance-over-time chart.

## Data & privacy

- **Nothing leaves your browser.** All calculations run client-side; there's no server, analytics, or network call of any kind.
- **Autosave:** every input is saved to `localStorage` as you type, so your numbers are there the next time you open the app on the same browser.
- **Export / Import:** use the "Export data" button to download your full set of inputs as a JSON file, and "Import data" to load one back in — on the same computer or a different one. This is how two people (e.g. spouses) can pass a plan back and forth without a shared account.
- **Print / Save PDF:** generates a clean, readable report of all five calculators (not just the tab you're on) suitable for printing or saving as a PDF.
- **Reset all:** clears everything back to defaults. There's a confirmation prompt since this can't be undone (unless you have an export saved).

## Running it locally

No install, no server, no dependencies:

```bash
open index.html
```

or just double-click the file. It works the same offline as it does online.

## Deployment

The `main` branch is published via GitHub Pages at the live URL above. Any push to `main` triggers a Pages rebuild automatically.

## Tech

Vanilla HTML/CSS/JavaScript in a single file — no frameworks, no build tooling, no package manager. State lives in one JS object that every calculator reads from and writes to, persisted to `localStorage` and serializable to/from JSON for export/import.
