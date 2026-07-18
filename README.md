
After the 2026-27 Budget's changes removing the 50% CGT discount and cross-asset loss offsetting, we test the impact across tax brackets and entity structures.

## Static Assumptions

Static assumptions across all scenarios:

- 7% capital gain p.a.
- 2% dividend income p.a.
- 10 years (sensitivity: 5/10/15/20/30 years)
- 2.5% CPI p.a. (for inflation-indexed cost base)
- Stock-level annual volatility: lognormal distribution, median 22% (clipped 8-60%)
- Pairwise correlation: single-factor model, rho = 0.35
- Portfolio: 20 equally-weighted stocks ($5,000 each on a $100,000 portfolio)
- Annual turnover: beta(2,6) distribution (mean 25%), capped at 50% of stocks; full liquidation at year 10
- Monte Carlo: 10,000 simulation runs (1,000 for sensitivity sweeps)
- Capital gains and dividends are reinvested
- Returns are lognormal (standard in quantitative finance)
- **Dividends: 80% fully franked at 30%.** Company pays 0% effective on franked dividends (franking credit offsets corporate tax). Individual pays top-up at `(mr - 0.30) / 0.70`, with excess credits refunded if mr < 30%. Unfranked portion (20%) taxed at full rate.
- **Pty Ltd: going concern by default.** The primary Pty Ltd scenario does not distribute. Franking credits accumulate but are never settled. The `PTY_DISTRIBUTE` flag controls this; Pty Ltd (Dist.) is a separate scenario that tests full liquidation with franking settlement at year 10.

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

- CGT: cost base indexed to CPI, real gain taxed at max(marginal_rate, 30%)
- Dividends: franked treatment (as above)
- Nominal losses offset real gains across assets (existing law unchanged). Stocks gaining nominally but underperforming CPI are neither a gain nor a loss -- effectively stranded.
- Source: [Bills Digest No. 67, 2025-26](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067)

### 3: Pty Ltd (Going Concern)

- Corporate tax: 30% on realised nominal capital gains and unfranked dividends
- Franked dividends: 0% effective corporate tax (franking credits offset). Credits received on portfolio dividends accumulate in the franking account.
- CGT paid by the company also generates franking credits.
- Cross-asset loss offsetting within the company (all nominal losses offset nominal gains)
- **No distribution.** The company is a perpetual accumulation structure. Franking credits accumulate but are never settled against individual tax.

### 4: Pty Ltd (Dist.)

- Identical accumulation to Pty Ltd (Going Concern) during years 0-9.
- **At year 10: full liquidation.** Initial capital returned tax-free. Profits distributed as franked dividends. Individual settles at marginal rate with franking credits attached.
- Franking refunds apply when mr < 30% (ATO pays the individual). Top-up tax applies when mr > 30%.

## Modelling Approach

Monte Carlo simulation with annual turnover and correlated returns. Without cross-asset offsetting (Post-Budget), winners are taxed every year while loser losses are stranded. This compounds: after-tax reinvestment is smaller each year, shrinking the compounding base.

1. Generate 10,000 random 20-stock portfolios:
   - Per-stock volatility: lognormal distribution (median 22%, clipped 8-60%)
   - Pairwise correlation: single-factor model, rho = 0.35
   - Returns: lognormal, mean nominal capital gain 7% p.a.
2. Each year: apply price returns and dividends, randomly sell 0-50% of stocks (beta(2,6) turnover distribution, mean 25%)
3. Realize CGT on sold stocks per scenario rules; stocks underperforming CPI are stranded (no gain, no loss)
4. Reinvest after-tax proceeds, reset cost bases (with CPI cost-year tracking per position)
5. Year 10: liquidate all remaining positions
6. Pty Ltd (Going Concern): accumulate indefinitely at 30% corporate rate, no franking settlement
7. Pty Ltd (Dist.): franking settlement at year 10 (return of capital + franked distribution)

## Results (10 Years, $100k Initial, 10,000 Sims)

| Archetype | Pre-Budget | Post-Budget | Pty Ltd | Pty (Dist) |
| --------- | ---------- | ----------- | ------- | ---------- |
| 1: 2%     | $254,565   | $221,636    | $198,689 | $238,400 |
| 2: 18%    | $233,015   | $212,073    | $198,689 | $215,708 |
| 3: 32%    | $215,292   | $201,936    | $198,689 | $195,852 |
| 4: 47%    | $197,419   | $178,933    | $198,689 | $174,579 |

### Effective CGT Rates (median CGT / nominal gain)

| Archetype | Pre-Budget | Post-Budget | Pty Ltd | Pty (Dist) |
| --------- | ---------- | ----------- | ------- | ---------- |
| 1: 2%     | 1%         | 20%         | 31%     | 31%        |
| 2: 18%    | 7%         | 21%         | 31%     | 31%        |
| 3: 32%    | 14%        | 25%         | 31%     | 31%        |
| 4: 47%    | 24%        | 45%         | 31%     | 31%        |

Note: Pty Ltd and Pty (Dist) have identical CGT and nominal gains during accumulation. The effective CGT rate is the same for both. The difference in terminal wealth comes from the franking settlement, not the CGT rate.

### Key Findings

- **Pre-Budget wins at lower brackets, Pty Ltd edges ahead at 47%.** The combination of 30% corporate CGT rate, 0% tax on franked dividends, and full loss offsetting overcomes Pre-Budget's 50% CGT discount at the top bracket. Pty Ltd leads Pre-Budget by $1.3k (0.7%) at 47%.
- **Post-Budget is worst at every bracket.** At 47%, Post-Budget trails Pty Ltd by $19.8k (10%). Effective CGT rates above the stated 47% are possible due to intertemporal loss-carry rebounding: a stock that generates a nominal loss offset in year N and then recovers in year N+1 faces CGT on the full recovery with no shelter.
- **Post-Budget's 30% floor punishes low brackets.** At 2%, effective CGT jumps from ~1% (Pre) to 20% (Post). At 18%, from 7% to 21%. This is the most regressive element of the reform.
- **Pty Ltd is bracket-agnostic (going concern).** The corporate rate is 30% regardless of who owns the company. Result is identical across all brackets: $198,689. This is a key structural advantage: your tax rate doesn't rise with income.
- **Distribution creates a bracket-dependent outcome.** At 2% and 18% (mr < 30%), the franking refund at year 10 makes distribution beneficial: Pty (Dist) beats going concern by significant margins. At 32% (mr ≈ 30%), near neutral. At 47% (mr > 30%), distribution destroys 12.1% of wealth via the franking top-up.

## Sensitivity: Time Horizon

Terminal wealth across 5, 10, 15, 20, and 30-year horizons. All values in $k (1,000 simulation runs per horizon).

See notebook Section 5 for the full table and chart. Key observations:

- **Ranking is stable across horizons.** Pty Ltd and Pre-Budget are close at all horizons for the 47% bracket. Pty Ltd advantage grows modestly from 0% at 5yr to ~3% at 30yr.
- **Post-Budget gap widens with time.** At 47%, Post-Budget trails Pty Ltd by -5% (5yr) growing to -24% (30yr). The compounding penalty of 47% CGT + quarantined losses is severe over long horizons.
- **At lower brackets, Pre-Budget dominates at all horizons.** The 50% CGT discount at low marginal rates is unmatched.

## Open Questions

- **End-of-life distribution.** The going-concern assumption works for accumulation, but death triggers a deemed disposal for both individuals (CGT event) and companies (shares valued at market). Intergenerational transfer strategies (bucket company, testamentary trust) are outside scope.
- **Dividend franking rate.** 80% franked at 30% is a reasonable ASX assumption but varies by portfolio composition. International stocks (unfranked) would worsen all scenarios proportionally. A sensitivity on FRANKING_PCT would quantify this.
- **Fixed marginal rates.** The model assumes constant rates. In reality, dividends and realised gains push investors into higher brackets, increasing the Post-Budget penalty for mid-bracket investors. A dynamic bracket model would show larger Post-Budget losses.
- **Cost-year CPI tracking.** Per-stock purchase years track CPI indexation precisely. Short holding periods (from 25% annual turnover) severely reduce the CPI benefit. A stock held 1 year gets only 1 year of CPI adjustment.
- **Bills Digest interpretation confirmed.** Nominal losses offset real gains across assets. "No cross-asset offsetting" interpretation was rejected. Source: [Bills Digest No. 67, 2025-26](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067).
