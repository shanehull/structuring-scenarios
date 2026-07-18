
# Structuring Scenarios for Investment

After the 2026-27 Budget's changes removing the 50% CGT discount and cross-asset loss offsetting, we test the impact across tax brackets and entity structures.

## Static Assumptions

Static assumptions across all scenarios:

- 7% capital gain p.a.
- 2% dividend income p.a.
- 10 years (sensitivity: 5/10/15/20/30 years)
- 2.5% CPI p.a. (for inflation-indexed cost base)
- Stock-level annual volatility: lognormal distribution, median 22% (clipped 8-60%)
- Pairwise correlation: single-factor model, r = 0.35
- Portfolio: 20 equally-weighted stocks ($5,000 each on a $100,000 portfolio)
- Annual turnover: beta(2,6) distribution (mean 25%), capped at 50% of stocks; full liquidation at year 10
- Monte Carlo: 10,000 simulation runs
- Capital gains and dividends are reinvested
- Returns are lognormal (standard in quantitative finance)
- **Dividends: 80% fully franked at 30%.** Company pays 0% effective on franked dividends (franking credit offsets corporate tax). Individual pays top-up at `(mr - 0.30) / 0.70`, with excess credits refunded if mr < 30%.
- **Pty Ltd: going concern.** The company does not distribute at year 10. It is a perpetual accumulation structure paying 30% corporate tax only. The `PTY_DISTRIBUTE` flag can be toggled to test liquidation scenarios.

Source for budget changes: [Budget 2026-27: Tax Reform](https://budget.gov.au/content/04-tax-reform.htm)

## Archetypes

The 4 archetypes are based on the current resident tax rates 2025-26 from the [ATO](https://www.ato.gov.au/tax-rates-and-codes/tax-rates-australian-residents). Effective rates include the 2% Medicare levy.

| Taxable income      | Tax on this income                         | Effective rate |
| ------------------- | ------------------------------------------ | -------------- |
| $0 - $18,200        | Nil                                        | 2%             |
| $18,201 - $45,000   | 16c for each $1 over $18,200               | 18%            |
| $45,001 - $135,000  | $4,288 plus 30c for each $1 over $45,000   | 32%            |
| $135,001 - $190,000 | $31,288 plus 37c for each $1 over $135,000 | 39%            |
| $190,001 and over   | $51,638 plus 45c for each $1 over $190,000 | 47%            |

To capture each bracket, we place archetypes at the top of the income bracket.

- 1: 18,200 - 2%
- 2: 45,000 - 18%
- 3: 135,000 - 32%
- 4: 190,000 - 47%

## Scenarios

### 1: Pre-Budget (Individual)

- CGT: 50% discount, taxed at marginal rate
- Dividends: franked treatment (top-up at `(mr - 0.30)/0.70`; refund if mr < 30%)
- Cross-asset loss offsetting (gains and losses netted before CGT)

### 2: Post-Budget (Individual)

- CGT: cost base indexed to CPI, real gain taxed at max(marginal_rate, 30%)
- Dividends: franked treatment (as above)
- Nominal losses offset real gains across assets (existing law unchanged). Stocks gaining nominally but underperforming CPI are neither a gain nor a loss -- effectively stranded.
- Source: [Bills Digest No. 67, 2025-26](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067)

### 3: Post-Budget Pty Ltd (Going Concern)

- Corporate tax: 30% on realised nominal capital gains and unfranked dividends
- Franked dividends: 0% effective corporate tax (franking credits offset)
- Cross-asset loss offsetting within the company (all nominal losses offset nominal gains)
- **No distribution at year 10.** The company is a perpetual accumulation vehicle. Franking credits accumulate but are never settled against individual tax.
- Setting `PTY_DISTRIBUTE=True` tests the full-liquidation case with franking settlement.

## Modelling Approach

Monte Carlo simulation with annual turnover and correlated returns. Without cross-asset offsetting (Post-Budget), winners are taxed every year while loser losses are stranded. This compounds: after-tax reinvestment is smaller each year, shrinking the compounding base.

1. Generate 10,000 random 20-stock portfolios:
   - Per-stock volatility: lognormal distribution (median 22%, clipped 8-60%)
   - Pairwise correlation: single-factor model, p = 0.35
   - Returns: lognormal, mean nominal capital gain 7% p.a.
2. Each year: apply price returns and dividends, randomly sell 0-50% of stocks (beta(2,6) turnover distribution, mean 25%)
3. Realize CGT on sold stocks per scenario rules; stocks underperforming CPI are stranded (no gain, no loss)
4. Reinvest after-tax proceeds, reset cost bases
5. Year 10: liquidate all remaining positions
6. Pty Ltd: going concern -- no distribution, no franking settlement. Accumulate indefinitely at 30% corporate rate.

## Results (10 Years, $100k Initial, 10,000 Sims)

| Archetype | Pre-Budget | Post-Budget | Pty Ltd |
| --------- | ---------- | ----------- | ------- |
| 1: 2%     | $254,565   | $221,636    | $198,689 |
| 2: 18%    | $233,015   | $212,073    | $198,689 |
| 3: 32%    | $215,292   | $201,936    | $198,689 |
| 4: 47%    | $197,419   | $178,933    | $198,689 |

### Effective CGT Rates (median CGT / nominal gain)

| Archetype | Pre-Budget | Post-Budget | Pty Ltd |
| --------- | ---------- | ----------- | ------- |
| 1: 2%     | 1%         | 20%         | 31%     |
| 2: 18%    | 7%         | 21%         | 31%     |
| 3: 32%    | 14%        | 25%         | 31%     |
| 4: 47%    | 24%        | 45%         | 31%     |

### Key Findings

- **Pre-Budget wins at lower brackets, Pty Ltd wins at 47%.** The franking dividend treatment and 30% corporate rate with full loss offsetting overcome Pre-Budget's 50% CGT discount at the top bracket. Gap at 47%: Pty +$1.3k above Pre (0.7%).
- **Post-Budget is worst at every bracket.** At 47%, Post-Budget trails Pty Ltd by $20k (10%). Effective CGT rates above the stated 47% are possible due to intertemporal loss-carry rebounding -- a stock that generates a nominal loss offset in year N and then recovers in year N+1 faces CGT on the full recovery with no shelter.
- **Post-Budget's 30% floor punishes low brackets.** At 2%, effective CGT jumps from ~0% (Pre) to 20% (Post). At 18%, from 7% to 21%. This is the most regressive element of the reform.
- **Pty Ltd is bracket-agnostic.** The corporate rate is 30% regardless of individual bracket. Results are identical across all brackets ($198,689). This is a key structural advantage: your tax rate doesn't rise with income.

## Sensitivity Analysis

### Distribution vs Going Concern

At year 10, what happens if the company liquidates with franking settlement?

| Archetype | GoingConc  | Distrib    | Delta    | Drag%  |
| --------- | ---------- | ---------- | -------- | ------ |
| 1: 2%     | $198,689   | $238,137   | +$39,448 | +39.4% |
| 2: 18%    | $198,689   | $215,488   | +$16,799 | +16.8% |
| 3: 32%    | $198,689   | $195,670   | -$3,019  | -3.0%  |
| 4: 47%    | $198,689   | $174,437   | -$24,252 | -24.3% |

- **At 2% and 18% (mr < 30%): distribution INCREASES wealth.** Excess franking credits are refunded. The ATO pays you.
- **At 32% (mr ~ 30%): near neutral.** Slight drag from top-up tax on unfranked component and CGT timing.
- **At 47% (mr > 30%): distribution destroys 24% of wealth.** Franking top-up brings the total tax rate to 47% on all accumulated profits.

### Time Horizon (47% Bracket Only)

| Horizon | Pre-Budget | Post-Budget | Pty Ltd | Pty/Pre Gap |
| ------- | ---------- | ----------- | ------- | ----------- |
| 5yr     | $140k      | $132k       | $140k   | 0.0%        |
| 10yr    | $199k      | $180k       | $200k   | +0.5%       |
| 15yr    | $281k      | $245k       | $284k   | +1.1%       |
| 20yr    | $395k      | $331k       | $402k   | +1.8%       |
| 30yr    | $765k      | $597k       | $788k   | +3.0%       |

- **Pty Ltd advantage grows with horizon.** At 30 years, Pty Ltd leads Pre-Budget by 3.0% ($23k). The 30% corporate rate compounds more effectively than the 50% CGT discount with 47% dividend tax.
- **Post-Budget falls further behind with time.** At 30 years, Post-Budget trails Pty by 32%. The compounding penalty of 47% tax + quarantined losses is severe.
- **At lower brackets, Pre-Budget dominance is persistent.** The 50% CGT discount at low marginal rates is unbeatable.

### Distribution Drag by Horizon (47% Bracket)

| Horizon | Drag% |
| ------- | ----- |
| 5yr     | 7.2%  |
| 10yr    | 12.3% |
| 15yr    | 15.8% |
| 20yr    | 18.3% |
| 30yr    | 21.2% |

Distribution drag increases with horizon: more years of accumulated profits to settle at marginal rate.

## Open Questions

- **End-of-life distribution.** The going-concern assumption works for accumulation, but death triggers a deemed disposal. A company in perpetuity requires a plan for intergenerational transfer (bucket company? testamentary trust?).
- **Dividend franking rate.** 80% franked at 30% is a reasonable ASX assumption but varies by portfolio composition. International stocks (unfranked) would worsen all scenarios proportionally.
- **Fixed marginal rates.** The model assumes constant rates. In reality, dividends and realised gains push investors into higher brackets, increasing the Post-Budget penalty for mid-bracket investors.
- **Cost-year CPI tracking.** The model tracks per-stock purchase years for CPI indexation. Short holding periods (from turnover) severely reduce the CPI benefit. The formula `(1 + CPI)^(yr - cost_year)` correctly captures this: a stock held 1 year gets only 1 year of CPI.
- **Bills Digest interpretation confirmed.** Nominal losses offset real gains across assets. "No cross-asset offsetting" interpretation was rejected. Source: [Bills Digest No. 67, 2025-26](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067).
