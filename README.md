
After the 2026-27 Budget's changes removing the 50% CGT discount and cross-asset loss offsetting, we test the impact across tax brackets and entity structures.

## Static Assumptions

- 6% capital gain p.a., 3% dividend yield p.a. (9% total return -- rounded from VAS 10yr: 5.2% CG, 4.16% yield)
- 10 years (sensitivity: 5/10/15/20/30 years)
- 2.5% CPI p.a. (for inflation-indexed cost base)
- Stock-level annual volatility: lognormal distribution, median 22% (clipped 8-60%)
- Pairwise correlation: single-factor model, rho = 0.35
- Portfolio: 20 equally-weighted stocks ($5,000 each on a $100,000 portfolio)
- Annual turnover: beta(2,6) distribution (mean 25%), capped at 50% of stocks; full liquidation at year 10
- Monte Carlo: 10,000 simulation runs (1,000 for sensitivity sweeps)
- Capital gains and dividends are reinvested
- Returns are lognormal
- **Dividends: 80% fully franked at 30%.** Company pays 0% effective on franked dividends (franking credit offsets corporate tax). Individual pays top-up at `(mr - 0.30) / 0.70`, with excess credits refunded if mr < 30%. Unfranked portion (20%) taxed at full rate.
- **Post-Budget CGT floor: 32%** (30% statutory + 2% Medicare levy). Minimum rate on real gains applies to all brackets.
- **Pty Ltd: 30% corporate accumulation + multi-year retirement distribution.** Profits are drawn down over multiple retirement years, keeping total taxable income under $135k. Effective individual rate on distributions: 32% (30% bracket + 2% Medicare). Franking credits offset, leaving a ~2% net top-up.

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

- 1: 18,200 -- 2%
- 2: 45,000 -- 18%
- 3: 135,000 -- 32%
- 4: 190,000 -- 47%

## Scenarios

### 1: Pre-Budget (Individual)

- CGT: 50% discount, taxed at marginal rate
- Dividends: franked treatment (top-up at `(mr - 0.30)/0.70`; refund if mr < 30%)
- Cross-asset loss offsetting (gains and losses netted before CGT)

### 2: Post-Budget (Individual)

- CGT: cost base indexed to CPI, real gain taxed at max(marginal_rate, 32%)
- Dividends: franked treatment (as above)
- Nominal losses offset real gains across assets (existing law unchanged). Stocks gaining nominally but underperforming CPI are neither a gain nor a loss -- effectively stranded.
- Source: [Bills Digest No. 67, 2025-26](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067)

### 3: Pty Ltd

- Corporate tax: 30% on realised nominal capital gains and unfranked dividends
- Franked dividends: 0% effective corporate tax (franking credits offset). Credits received accumulate in the franking account.
- Cross-asset loss offsetting within the company (all nominal losses offset nominal gains)
- **At retirement: multi-year distribution.** Profits drawn down over multiple years, keeping total taxable income under $135k. Effective individual rate: 32% (30% bracket + 2% Medicare). Franking credits offset most of this, leaving a ~2% net top-up (0.32 - 0.30).

## Modelling Approach

Monte Carlo simulation with annual turnover and correlated returns.

1. Generate 10,000 random 20-stock portfolios:
   - Per-stock volatility: lognormal distribution (median 22%, clipped 8-60%)
   - Pairwise correlation: single-factor model, rho = 0.35
   - Returns: lognormal, mean nominal capital gain 6% p.a.
2. Each year: apply price returns and dividends, randomly sell 0-50% of stocks (beta(2,6) turnover distribution, mean 25%)
3. Realize CGT on sold stocks per scenario rules; stocks underperforming CPI are stranded
4. Reinvest after-tax proceeds, reset cost bases (with CPI cost-year tracking per position)
5. Year 10: liquidate all remaining positions
6. Pty Ltd: accumulate at 30% corporate rate, then distribute at 32% effective rate with franking settlement

## Results (10 Years, $100k Initial, 10,000 Sims)

| Archetype | Pre-Budget | Post-Budget | Pty Ltd |
| --------- | ---------- | ----------- | ------- |
| 1: 2%     | $263,426   | $234,177    | $199,706 |
| 2: 18%    | $237,620   | $219,469    | $199,706 |
| 3: 32%    | $216,706   | $205,475    | $199,706 |
| 4: 47%    | $195,918   | $180,472    | $199,706 |

### Key Findings

- **Pre-Budget wins at lower brackets.** The 50% CGT discount at 2-18% creates near-zero effective CGT rates. Combined with franking refunds on dividends, Pre-Budget dominates at brackets below the corporate rate.
- **Pty Ltd wins at 47% ($199,706 vs $195,918).** The 30% corporate CGT rate with full loss offsetting and 0% tax on franked dividends overcomes Pre-Budget's 50% discount when the individual's marginal rate is high. Gap at 47%: Pty +$3.8k (1.9%).
- **Post-Budget is worst at every bracket.** The 32% CGT floor (30% + Medicare), CPI-indexed gains with quarantined real losses, and intertemporal loss-carry rebound effects make Post-Budget the least favourable structure at all income levels.
- **Pty Ltd is largely bracket-agnostic.** The 30% corporate rate applies regardless of the shareholder's bracket. The distribution top-up at 32% adds ~2% above the corporate rate. Result: $199,706 for all brackets -- your tax rate doesn't rise with income.

## 30-Year Comparison

At longer horizons, the compounding advantage of the 30% corporate rate becomes decisive.

| Archetype | Pre-Budget | Post-Budget | Pty Ltd |
| --------- | ---------- | ----------- | ------- |
| 1: 2%     | $1,580,608 | $1,102,828  | $787,933 |
| 2: 18%    | $1,228,372 | $965,280    | $787,933 |
| 3: 32%    | $979,937   | $835,817    | $787,933 |
| 4: 47%    | $764,542   | $597,279    | $787,933 |

At 30 years, Pty Ltd dominates the 47% bracket ($787,933 vs Pre-Budget $764,542) and is competitive at 32%. Pre-Budget still dominates lower brackets.

## Sensitivity: Time Horizon

Ranking stability across 5, 10, 15, 20, and 30-year horizons. Pty Ltd advantage over Pre-Budget at 47% grows from 0% (5yr) to ~3% (30yr). Post-Budget falls further behind at every horizon. Full table and chart in notebook Section 5.

## Footnotes

- **Multi-year distribution assumption.** Pty Ltd profits are assumed to be drawn down over multiple retirement years, keeping total taxable income under $135k. This produces an effective individual rate of 32% (30% bracket + 2% Medicare). A lump-sum distribution at the individual's full working-year marginal rate would produce worse results (see notebook history for sensitivity testing).
- **VAS returns.** Capital gain and dividend yield are rounded from Vanguard Australian Shares Index (VAS) 10-year performance to mid-2026: 9.36% total return (5.20% CG + 4.16% yield). We use 9% total (6% CG + 3% yield) as a conservative round.
- **Bills Digest interpretation confirmed.** Nominal losses offset real gains across assets. "No cross-asset offsetting" interpretation was rejected. Source: [Bills Digest No. 67, 2025-26](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067).
- **Fixed marginal rates during accumulation.** The model uses constant rates during the 10-year accumulation phase. In reality, dividends and realised gains push investors into higher brackets, increasing the Post-Budget penalty for mid-bracket investors.
