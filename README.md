# Financial Goal Projections

A single-page, offline-capable calculator for planning a home purchase, monthly budget, house payoff strategy, retirement savings, and short-term savings goals — all in one place, with every number feeding the next.

**Live app:** https://saiahcrand23.github.io/home_owner_financial_lifestyle/

## Why this exists

Most budgeting tools either need an account and a server, or are a spreadsheet that's hard to hand off to someone else to review. This is neither: it's one `index.html` file with no backend, no build step, and no external dependencies. Everything runs in your browser, your numbers stay on your device, and you can hand a snapshot of your plan to someone else (a spouse, a partner, yourself on another computer) as a single JSON file.

## Calculators

The app is organized into seven tabs. Data flows forward — numbers you enter in an earlier tab automatically feed the calculators after it — but every auto-filled field stays editable if you want to override it.

1. **Savings Goals** — Sequence short-term savings priorities (paying off loans, building an emergency fund, saving for a house down payment, or just a general high-yield savings account) against your actual contribution schedule (paychecks, bonuses, one-time deposits). Produces a month-by-month timeline showing when each goal is hit.
2. **Home Buying** — Enter a home price, down payment, mortgage terms, and closing costs to get a full monthly housing cost breakdown (principal & interest, property tax, insurance, HOA, PMI, maintenance), total cash needed at closing, and an amortization schedule. PMI automatically drops off once equity crosses 20%.
3. **Cash Flow** — The input surface for everything income- and bill-related. Gross regular and bonus pay (including which months the bonus lands), a payroll deductions section (tax rates, benefits premiums, HSA, traditional and Roth retirement, tithe — each with a pre/post-tax treatment and a regular/bonus/both applicability), and the budget itself. Every bill is entered at its own cadence (monthly, quarterly, semi-annual, annual), assigned a funding source (regular pay or bonus), and given its due month(s). Produces a 12-month running balance with two lines — regular-pay-only and combined — whose gap is the headline number: **how many bonus dollars are tied up covering ordinary monthly bills**. The goal is to drive that to $0.
4. **Surplus** — A pure dashboard over the Cash Flow inputs. Three boxes: Income (gross and net, regular-only vs. bonus-adjusted), Monthly Budget (housing, bills, retirement, benefits, tithe, taxes), and Surplus (annual/quarterly/monthly, plus the regular-vs-bonus split). Allocation is split into two parallel blocks — regular pay allocated monthly, bonus allocated annually — each dividing across discretionary, savings, kids' future, and house payoff, with house payoff as the auto-set remainder.
5. **House Payoff** — Reuses the mortgage amortization engine from Home Buying and layers in your payoff allocation plus any one-off extra payments by year, showing the revised payoff date and total interest saved. The regular-pay share is applied as a monthly extra payment and the bonus share as a start-of-year lump, so steady and lumpy money are credited with the interest they actually save.
6. **Retirement** — Projects your retirement balance from current age to retirement age given a contribution rate, employer match, and expected return. Supports one-time contribution changes at specific future ages, an inflation-adjusted ("today's dollars") view alongside the nominal projection, and a balance-over-time chart.
7. **Kids' Future** — Projects savings set aside for children from the kids' future allocation, with per-child accounts and scheduled contribution increases by year.

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

Payroll math runs in the order real payroll does: gross, minus pre-tax deductions, equals taxable; then tax; then post-tax deductions. (Earlier versions subtracted retirement *after* computing tax on the full gross, so pre-tax contributions received no tax benefit.)

Vanilla HTML/CSS/JavaScript in a single file — no frameworks, no build tooling, no package manager. State lives in one JS object that every calculator reads from and writes to, persisted to `localStorage` and serializable to/from JSON for export/import.
