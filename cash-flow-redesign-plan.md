# Cash Flow Redesign — Implementation Plan

Consolidates the notes in `suplus updates.md` and `allocate the surplus updates.md`
into a single spec. Written 2026-09-02.

## Goal

Today every number is blended into one annual average, which hides the question
that actually matters: **do my regular paychecks cover my recurring bills on
their own, without reserving bonus money for the mortgage or groceries?**

The redesign separates steady money from lumpy money everywhere — income,
deductions, bills, allocation, and the amortization engine — and adds a
12-month calendar so timing problems become visible.

Headline metric: **bonus dollars reserved for monthly bills → drive to $0.**

---

## 1. New tab: Cash Flow

A new tab holding every input. Sits between Home Buying and Surplus, so data
still flows forward: Home Buying produces the mortgage payment → Cash Flow
consumes it as a bill → Surplus consumes Cash Flow's totals.

New tab order:

    Savings Goals → Home Buying → Cash Flow → Surplus → House Payoff → Retirement → Kids' Future

### 1a. Income section

| Field | Notes |
| --- | --- |
| Gross regular pay (monthly $) | moved from Surplus |
| Gross bonus ($) | moved from Surplus |
| Bonus frequency | monthly / quarterly / annually — moved from Surplus |
| Bonus months | which months bonus lands; drives the calendar |

### 1b. Payroll deductions section

Two tax-rate fields, then five **fixed** rows. Nothing free-form here — the
Retirement tab reads the retirement rate by name, and the user asked to keep
clutter down.

    Tax rate    regular [ 20% ]    bonus [ 22% ]

    Deduction              Amount   Treatment   Applies to
    Benefits premiums       $180    pre-tax     regular
    HSA                     $  0    pre-tax     regular
    Retirement (traditional)   7%   pre-tax     both
    Retirement (Roth)          0%   post-tax    both
    Tithe                     10%   post-tax    both

- All benefit premiums consolidate into **one** line (user request).
- **HSA stays separate** from premiums: it is savings the user still owns, not
  money spent. Folding it into premiums would overstate cost of living.
- Every row has an **applies to: regular / bonus / both** selector. This
  generalises the "does 401k come out of bonus checks" question — the user sets
  it to match their actual payroll rather than the app guessing.
- Separate regular/bonus tax rates, since supplemental wages are commonly
  withheld at a flat 22%. Default both to the same value so it is a no-op.
- **Tithe base is gross** (confirmed; matches current index.html:1512).

#### Order of operations — fixes an existing bug

Retirement is currently subtracted as if it were a bill (index.html:1511, inside
`annualDeductions`) but tax was already computed at index.html:1497 on income
that still included it. **Pre-tax retirement therefore gets zero tax benefit
today.** Correct order:

    Gross
      − pre-tax deductions    (HSA, benefits premiums, traditional retirement)
    = Taxable
      − tax                   (taxable × rate)
      − post-tax deductions   (Roth retirement, tithe)
    = Net take-home           → available for bills

Expect the surplus number to rise once this lands. That is the bug being
corrected, not a new assumption.

### 1c. Budget section

Replaces the "Budget categories" box on the Surplus tab entirely. Each bill
gains three attributes beyond today's name + amount:

| Column | Values |
| --- | --- |
| Amount | as entered, in its own cadence — a $1,400 semi-annual premium is entered as 1400, not 233 |
| Cadence | monthly / quarterly / semi-annual / annual |
| Funded by | regular / bonus |
| Due month(s) | drives the calendar |

Housing is a **pinned, non-removable** first row (monthly, regular-funded) that
keeps today's auto-fill from Home Buying, the manual override, and the "use
calculated value" button.

### 1d. 12-month running balance

Twelve months across, running balance down, two lines plotted:

- **regular-only** — regular pay against regular-funded bills
- **combined** — reality, with bonus included

The gap between them is the headline metric: dollars of bonus the user is forced
to hold back to cover ordinary bills. Also surface the minimum monthly float and
any month where the regular-only line dips negative.

---

## 2. Surplus tab becomes a pure dashboard

No bill editing, no pay inputs. Three output boxes plus the allocation controls.

### 2a. Income box (all monthly)

|  | Regular only | Bonus-adjusted |
| --- | --- | --- |
| Gross | 7,083 | 7,250 |
| Net | 5,522 | 5,656 |

"Bonus-adjusted" amortises the bonus to a monthly average. **Net = after tax,
pre-tax and post-tax deductions** under the corrected order of operations above.

### 2b. Monthly budget box

Keeps today's breakdown — tithe, taxes, budgeted categories, retirement,
housing — but reads its numbers from Cash Flow.

### 2c. Surplus box

Annual / quarterly / monthly as today, plus the new line:

    Regular monthly surplus = net regular − regular-funded deductions
                              − regular-funded bills (monthly-normalised)

    Bonus annual leftover   = net bonus/yr − bonus-funded deductions
                              − bonus-funded bills (annualised)

**Invariant to preserve:** `regular × 12 + bonus leftover` must reconcile to the
total annual surplus. The reorg decomposes the bottom line; it does not move it
(except for the pre-tax bug fix above).

---

## 3. Allocate the surplus — two parallel blocks

Split into two blocks, each with the same four destinations.

    ┌─ From regular pay (monthly) ─────── surplus: $420 /mo ─┐
    │  Mode:  [ % ]  [ $ / month ]                           │
    │    Discretionary    40%  →  $168 /mo      $2,016 /yr   │
    │    Savings          20%  →  $ 84 /mo      $1,008 /yr   │
    │    Kids' future     10%  →  $ 42 /mo      $  504 /yr   │
    │    House payoff     30%  →  $126 /mo      $1,512 /yr   │
    │                          └ remainder, auto-set         │
    └────────────────────────────────────────────────────────┘
    ┌─ From bonus (annual) ──────────── leftover: $4,800 /yr ┐
    │  Mode:  [ % ]  [ $ / year ]                            │
    │    Discretionary    25%  →  $1,200 /yr                 │
    │    Savings          25%  →  $1,200 /yr                 │
    │    Kids' future     25%  →  $1,200 /yr                 │
    │    House payoff     25%  →  $1,200 /yr                 │
    │                          └ remainder, auto-set         │
    └────────────────────────────────────────────────────────┘
    ┌─ Combined (annual) ────────────────────────────────────┐
    │  Discretionary $3,216    Savings       $2,208          │
    │  Kids' future  $1,704    House payoff  $2,712          │
    └────────────────────────────────────────────────────────┘

- Regular block shows both `/mo` and `/yr` — the user thinks monthly, everything
  downstream is annual.
- **Each block gets its own %/$ toggle.** Dollars read naturally on a small
  monthly figure, percentages on a lumpy annual one.
- House payoff is the auto-remainder in both blocks (consistent with today).

### The combined row is the integration seam

Two tabs reach into these numbers today:

- `computePayoff()` reads `payoffAnnual` — index.html:1549
- Kids' Future reads `kidsFutureAnnual` — index.html:1731, 1756

**If the combined row keeps publishing the same four annual totals, both tabs
keep working untouched.** This is what makes the change contained.

Note: `savingsAnnual` and `vacationAnnual` are display-only today — nothing
consumes them, and Savings Goals has its own separate contribution inputs.
Wiring savings allocation into that tab is a real gap but is **out of scope**.

---

## 4. Amortization engine — monthly vs. lump extras

`buildAmortization` currently applies each year's extra as a **single lump on
day one of the year** (index.html:1409-1411, before the monthly loop). Correct
for bonus money, wrong for the regular block: $126/month would be modelled as
$1,512 every January, overstating interest saved.

Since the whole point of this redesign is separating steady from lumpy money, it
should not be flattened at the payoff engine. Fix: pass `monthlyExtra` separately
and apply it inside the existing month loop (~5 lines).

---

## 5. Migration

Existing `localStorage` data must survive. One-time upgrade on load:

| Existing | Becomes |
| --- | --- |
| `categories: [{id, name, amount}]` | + `cadence:"monthly"`, `fundedBy:"regular"`, `dueMonths:[]` |
| `healthInsuranceMonthly: 180` | benefits premiums line, pre-tax, regular |
| `hsaMonthly` | HSA line, pre-tax, regular |
| `retirementPct: 7` | retirement (traditional), pre-tax, both |
| `titheRegular` / `titheBonus` | tithe rate 10%, applies-to derived from the two flags |
| `taxRate` | both regular and bonus tax rate |
| `vacationDollar` etc. (annual) | **bonus** allocation block (same denomination) |
| — | regular allocation block defaults to 100% discretionary (float) |
| `housingOverride` | pinned Housing bill row |

The user should re-check both allocation blocks after first load.

---

## 6. Build order

1. State model + migration, verified against existing saved data
2. `buildAmortization` monthly-extra support
3. Cash Flow tab: markup, nav button, tab reorder
4. `computeCashFlow()` — income, deductions, net, bill normalisation
5. Rewrite `computeSurplus()` to consume Cash Flow; hold the reconciliation invariant
6. Surplus tab: three boxes + two-block allocation
7. 12-month running balance view
8. Wire print/export to include the new tab; update README
