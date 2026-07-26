# introduction
I want this tab to be used to decide how I allocate my kids savings budget. By that I mean what type of financial account vehicles. I like the 529 plan for tax benefits and just a brokerage account for more flexibility.

I want this to follow the import/export functions as every other page.
## summary and modifications
I want to first be shown how much I have saved annually towards the kids future in total. This should be the figure inputted in the bottom portion of the surplus page.

I want a date picker to choose when this will begin. (default to today)

Then I want an optional section to increase the amount annually. This would be the new annual contributions amount. I'd like to add rows for any year. Please use years (2027,2028,etc).
# input children
I then want to be able to add each of my children. I want to input child's name, date of birth.

Then I want to be able to add a 529 plan and/or a regular brokerage account.

### 529 option
inputs:
- amount to allocate in dollars. (I want to be shown a percentage of total kids savings)
- rate of return
- age child will go to college/trade school (default age 18)

output:
- total amount returned. also include money earned and contributed breakdown.
- amount available each of the 4 years.

### brokerage
inputs:
- amount to allocate in dollars. (I want to be shown a percentage of total kids savings)
- rate of return
- age of child withdraw (default age 18)

output:
- total amount returned. also include money earned and contributed breakdown.

## ad-hoc easy buttons

I want an "easy button" for both the brokerage and 529 plans. These would be checkboxes that are defaulted to unchecked. The idea would be to calculate the necessary inputs in order to have all the kids 529 plans to be even. And one button for all the brokerage accounts to be even. Would this be possible?

# decisions (2026-07-26)

Answers to open questions before implementation:

- **Allocation scaling:** Each child's account is set as a % of the total kids-future pool, not a fixed dollar amount. As the total grows in later years (per the year-by-year increase table), every account's dollar contribution grows proportionally with it.
- **529 withdrawal model:** Once a child hits college age, contributions to that account stop, but the balance keeps earning returns through the 4 college years. The 4 annual withdrawal amounts are solved so they're equal and fully deplete the account by the end of year 4 (not a simple balance/4 split).
- **Easy button (auto-balance):** "Even" means equal *ending balance* across children's accounts of the same type at their respective ages — not equal dollar contributions. Since kids have different time horizons (based on DOB), the required contribution per child will differ so their projected balances come out equal. Solved via future-value-of-a-growing-annuity math: back-solve each child's needed contribution for a shared target balance, choosing the target so the contributions sum to the available pool for that account type.
- **Compounding cadence (assumed default):** Monthly compounding, matching how the rest of the app (HYSA, retirement, mortgage) already compounds — flag if annual is preferred instead.