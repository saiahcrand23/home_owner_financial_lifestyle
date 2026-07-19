# background
I am getting ready to buy my first house and I want my wife and I to be on the same page if we do so. So that means I want all our budget categories, house price, retirement, and surplus to be agreed upon. So I want to build an application that calculates all of these things. I have been using a google sheet but I'd like something easier to use.

Reference material (not read by the app, just context for decisions below):
- Prior draft + Claude UI feedback: `home-lifestyle_finances-updates.md`
- Existing google sheet with real numbers and a more developed housing calculator: `Finances/Projections/House/Budget Graphs.gsheet`
- Household context (income, debt, 401k, family, relocation goals): `Background.gdoc`

# instructions
I am wanting to build a calculator that yields the monthly housing costs, down payment, etc given things like house price, mortgage term, etc. Also I want build a calculator to understand how much surplus I will have annually and quarterly (annual divided by 4) after inputting budget categories and pay. I want to calculate what I'd have at retirement given the retirement budget I provide in the previous calculator and things like retirement age and return rate. Finally a calculation for when the house is payed off.

I want these values to be remembered the next time I open the application. I have used an index.html file in the past that worked very well. This application will be considerable larger though.

Data flow: home buying → housing cost feeds surplus calculator → surplus feeds both house payoff and retirement calculators. Fields that auto-fill from an earlier calculator (housing cost, retirement $) stay editable afterward — autofill is a convenience, not a lock. One field flows backward: the home buying calculator's affordability check needs gross monthly income, which isn't entered until the surplus calculator — so that one output only populates once the surplus calculator's pay inputs are filled in.

# resolved decisions
These were open questions after the first planning pass; resolving them here so the build doesn't stall on ambiguity.

- **Sync between two people:** single `index.html`, `localStorage` autosave, plus an Export/Import JSON button so my wife and I can pass a snapshot of all inputs between devices. No backend.
- **Units:** HOA fee is monthly. Property tax rate / PMI rate / annual maintenance % are all annual rates applied against home value (or loan balance, for PMI), converted to a monthly dollar amount for display. Needs/wants are flat monthly dollar amounts, not percentages.
- **House payoff extra funds:** applied as one **annual lump sum** at the start of the year (both the surplus-to-payoff split and any optional manually-added yearly amounts) — not spread monthly alongside the regular mortgage payment.
- **Retirement return rate:** **nominal**, not inflation-adjusted. Ending balance is future dollars, not today's purchasing power.
- **Retirement contribution basis:** the retirement % in the surplus calculator applies to **gross regular pay + bonus pay combined** (monthly-equivalent), not take-home/post-tithe pay.
- **Irregular annual expenses** (car registration, gifts, annual insurance premiums, subscriptions, etc.): folded into **needs** for v1 rather than adding a 5th budget category. Revisit if needs starts feeling like a junk drawer.
- **Mortgage type:** fixed-rate only, term is a dropdown (15 / 20 / 30 years). No ARM support.

## home buying calculator
I want this calculator to just use some basic inputs and calculate the rest. The calculations that I want are:
- housing costs which include:
	- P&I
	- property tax
	- home insurance
	- HOA fees
	- PMI
	- maintenance reserve
- cash needed at closing
- an affordability check against income

### inputs
I need to place the following in the inputs:
- home price
- down payment percentage (please display the dollar amount immediately after the percentage is provided)
- interest rate
- mortgage length (dropdown: 15 / 20 / 30 years, fixed-rate only)
- property tax rate (annual, % of home value)
- annual home insurance amount
- HOA fees (monthly)
- PMI rate (annual, % of loan amount — only applies if down payment < 20%)
- Annual Maintenance (% of home value)
- closing cost inputs, matching the level of detail already in the Budget Graphs sheet: loan origination fee %, discount points %, appraisal fee, home inspection, title insurance & search, attorney/escrow/closing, recording fees, transfer/excise tax %, prepaid property tax (months), prepaid homeowners insurance, prepaid interest (days), other closing costs
- optional upfront/move-in costs: moving costs, immediate repairs, furniture/appliances, home warranty, utility deposits, other

### outputs
- monthly housing cost breakdown (P&I, property tax, insurance, HOA, PMI, maintenance)
- total cash to close = down payment + closing costs + other upfront costs
- affordability check (25% rule): housing cost as % of gross monthly income, surplus/shortfall vs. the 25% threshold, pass/fail status; this will only be calculated once the income is inputted in the next calculator
- amortization schedule (internal, surfaced as an optional table/chart): year-by-year beginning balance, principal paid, interest paid, ending balance — this feeds the house payoff calculator
- PMI end date: computed from the amortization schedule as the year loan-to-value crosses ~80% (only relevant if down payment < 20%); PMI cost drops to $0 in the monthly breakdown from that year forward rather than being charged for the life of the loan

## surplus calculator
That surplus will be used on vacations/fun and to pay off the house. I would like the total surplus to be displayed (monthly, quarterly, and annual). Then I would like a radio-type button that allows you to use a percentage or hard amounts to divide funds between vacations and home pay off. I'd like the amounts immediately displayed if the percentage option is used.

I need to first calculate the take home pay. So there will be an input for gross and also monthly HSA contributions, effective tax rate, and monthly health insurance. This would not include retirement since this is still being decided upon. Also remember my bonus is payed out quarterly along with one of my pay checks, so HSA and health incurance should not be take out of the bonus individually. But the effective tax rate of course will.

Deduction order: tax (computed after pre-tax HSA/health insurance are backed out), then HSA, then health insurance, then retirement, then tithe last — tithe's dollar amount is always 10% of the *original* gross for that income stream, not of whatever's left after the other deductions:
```
taxable_regular      = gross_regular - HSA - healthIns
net_regular_pretithe = taxable_regular - (taxable_regular × taxRate)

taxable_bonus        = gross_bonus                        // no HSA/health insurance on bonus
net_bonus_pretithe   = taxable_bonus - (taxable_bonus × taxRate)

annual_pretithe   = (net_regular_pretithe × 12) + (net_bonus_pretithe × 4)
annual_deductions = (needs + wants + housing + retirement$) × 12
tithe_annual      = (tithe_regular_checked ? gross_regular × 12 × 0.10 : 0)
                  + (tithe_bonus_checked   ? gross_bonus × 4 × 0.10   : 0)

annual surplus = annual_pretithe − annual_deductions − tithe_annual
```
`quarterly = annual ÷ 4`, `monthly = annual ÷ 12`. Retirement $ is derived from the retirement % applied to (gross monthly regular pay + gross bonus pay ÷ 3, i.e. bonus averaged monthly) — gross, per the resolved contribution basis above.

### inputs
I need a place to input the following:
- gross regular pay monthly (checkbox: tithe 10% of this gross amount, subtracted last in the deduction order above — after tax, HSA, health insurance, and retirement)
- gross bonus pay quarterly (checkbox: tithe 10% of this gross amount, subtracted last in the deduction order above — after tax and retirement; bonus has no HSA/health insurance deduction)
- effective tax rate
- monthly HSA contributions
- monthly health insurance contributions
- budget categories:
	- needs
	- wants
	- the housing costs calculated in the previous calculator. (I'd like this to be auto filled from the previous calculator but I would like to be able to edit it if I want to)
	- retirement (percentage input, display the resulting dollar amount immediately after the percentage is provided against regular pay and bonus)

### outputs
- total surplus (monthly, quarterly, annual)

### 2nd input section
I need to then choose how much I want allocated to vacation/fun (like to call discretionary) and home pay off ammount. This would be ideally done through a radio button that I can choose to use a percentage or hard amount.

- percentage or dollar amount radio button
- percentage or dollar amount input section

## house payoff calculator
This calculator will be ready to use once the surplus is calculated from the previous calculator. I would like another section to optionally provide annual allocations each year (add a row by year basically if I want to). All extra funds — both the surplus-to-payoff split and any manually-added yearly amounts — are applied annually at the beginning of the year, on top of the regular monthly mortgage payments.

The calculator reuses the amortization engine from the home buying calculator (starting balance, rate, term from those inputs) and layers in the extra annual principal payments to recompute a new, earlier payoff date. Because extra principal changes the loan-to-value trajectory, PMI drop-off is recalculated here too, so this calculator and the home buying calculator never disagree on when PMI ends.

### inputs
- I need to provide annual amounts any year I would optionally choose (add a row per year), in addition to the recurring surplus-to-payoff amount carried over from the surplus calculator

### outputs
- revised payoff year (vs. original mortgage term)
- total interest saved vs. the original amortization schedule
- year-by-year table: extra payment applied, remaining balance, PMI status

## retirement calculator
This calculator will be ready to use once the surplus inputs are provided. I just want to know how much I will have at the retirement age and how much was earned/contributed.

Monthly contribution = retirement $ from the surplus calculator + employer match (from the settings entered there), assumed flat for all remaining years (no growth with future raises — reasonable simplification for v1). Growth is compounded monthly at the nominal annual return rate.

### inputs
- current age
- starting balance
- retirement age
- return rate (nominal, annual %)
- employer 401k match settings (match rate %, cap as % of pay) — feeds the retirement calculator's contribution total, not the surplus math

### outputs
- projected balance at retirement age
- breakdown of total contributed vs. total earned (growth)
