
# Structuring Scenarios for Investment

After the 2026-27 Budget's changes removing the 50% CGT discount and cross-asset loss offsetting, we test the impact across tax brackets and entity structures.

## Static Assumptions

Static assumptions across all scenarios:

- 7% capital gain p.a.
- 2% dividend income p.a.
- 10 years
- 2.5% CPI p.a. (for inflation-indexed cost base)
- Stock-level annual volatility: lognormal distribution, median 22% (clipped 8-60%)
- Pairwise correlation: single-factor model, ρ = 0.35
- Portfolio: 20 equally-weighted stocks ($5,000 each on a $100,000 portfolio)
- Annual turnover: beta(2,6) distribution (mean 25%), capped at 50% of stocks; full liquidation at year 10
- Monte Carlo: 10,000 simulation runs
- Capital gains and dividends are reinvested
- Returns are lognormal (standard in quantitative finance)

Source for budget changes: [Budget 2026-27: Tax Reform](https://budget.gov.au/content/04-tax-reform.htm)

## Archetypes

The 4 archetypes are based on the current resident tax rates 2025–26 from the [ATO](https://www.ato.gov.au/tax-rates-and-codes/tax-rates-australian-residents). Effective rates include the 2% Medicare levy.

| Taxable income      | Tax on this income                         | Effective rate |
| ------------------- | ------------------------------------------ | -------------- |
| $0 – $18,200        | Nil                                        | 2%             |
| $18,201 – $45,000   | 16c for each $1 over $18,200               | 18%            |
| $45,001 – $135,000  | $4,288 plus 30c for each $1 over $45,000   | 32%            |
| $135,001 – $190,000 | $31,288 plus 37c for each $1 over $135,000 | 39%            |
| $190,001 and over   | $51,638 plus 45c for each $1 over $190,000 | 47%            |

To capture each bracket, we place archetypes at the top of the income bracket.

- 1: 18,200 — 2%
- 2: 45,000 — 18%
- 3: 135,000 — 32%
- 4: 190,000 — 47%

## Scenarios

### 1: Pre-Budget (Individual)

- CGT: 50% discount, taxed at marginal rate
- Dividends: taxed at marginal rate
- Cross-asset loss offsetting (gains and losses netted before CGT)

### 2: Post-Budget (Individual)

- CGT: cost base indexed to CPI, real gain taxed at max(marginal_rate, 30%)
- Dividends: taxed at marginal rate
- Nominal losses offset real gains across assets (existing law unchanged). Stocks gaining nominally but underperforming CPI are neither a gain nor a loss — effectively stranded.
- Source: [Bills Digest No. 67, 2025-26](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067)

### 3: Post-Budget Pty Ltd

- Corporate tax: 30% on both dividends and net capital gains
- Cross-asset loss offsetting within the company
- Franking credits on distribution; individual settles up at marginal rate

## Modelling Approach

Monte Carlo simulation with annual turnover and correlated returns. Without cross-asset offsetting, winners are taxed every year while loser losses are stranded. This compounds: after-tax reinvestment is smaller each year, shrinking the compounding base.

1. Generate 10,000 random 20-stock portfolios:
   - Per-stock volatility: lognormal distribution (median 22%, clipped 8-60%)
   - Pairwise correlation: single-factor model, ρ = 0.35
   - Returns: lognormal, mean nominal capital gain 7% p.a.
2. Each year: apply price returns and dividends, randomly sell 0-50% of stocks (beta(2,6) turnover distribution, mean 25%)
3. Realize CGT on sold stocks per scenario rules; stocks underperforming CPI are stranded (no gain, no loss)
4. Reinvest after-tax proceeds, reset cost bases
5. Year 10: liquidate all remaining positions
6. Pty Ltd: accumulate franking credits, distribute at year 10 with return-of-capital treatment

This exposes the hidden cost: Post-Budget's inflation indexing partially compensates for the loss of the 50% CGT discount, but Pre-Budget still dominates. Pty Ltd suffers from full distribution at year 10 — the franking top-up brings the total rate to the individual's marginal rate. Retaining earnings indefinitely would keep the rate at 30%.

## Results

Mean after-tax wealth after 10 years ($100k initial, 10,000 simulations):

| Archetype | Pre-Budget | Post-Budget | Pty Ltd |
| --------- | ---------- | ----------- | ------- |
| 1: 2%     | $238,358   | $208,975    | $225,043 |
| 2: 18%    | $220,347   | $202,169    | $204,501 |
| 3: 32%    | $205,403   | $194,487    | $186,527 |
| 4: 47%    | $190,202   | $174,920    | $167,269 |

### Key Findings

- **Pre-Budget wins across all brackets.** The 50% CGT discount with cross-asset offsetting is irreplaceable. At 2%, the gap is $25k; at 47%, $9k.
- **Pty Ltd vs Post-Budget depends on bracket.** At 2-18%, Pty Ltd beats Post-Budget (franking refunds recover the 30% corporate tax; cross-asset offsetting helps). At 32-47%, Post-Budget beats Pty Ltd (inflation indexing on cost base outweighs cross-asset offsetting over 10 years).
- **Lower brackets are worst-hit by the 30% floor.** The 2% bracket sees effective rates jump from ~0% to 30% on every gain. At 18%, same jump. This is the most unfair aspect of the reforms.
- **The quarantine of losses is real but secondary.** With ~5.3 losers per portfolio (27% of stocks), the penalty from quarantined losses adds ~$2-5k drag vs a hypothetical world with full offsetting. The 30% floor and loss of the 50% discount are the primary drivers.

## Open Questions

- **Cross-asset loss offsetting confirmed.** Per the [Bills Digest No. 67](https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/bd/bd2526/26bd067): nominal losses offset real gains across assets. Stocks gaining nominally but underperforming CPI are neither a gain nor a loss — effectively stranded.
- **Pty Ltd model assumes full distribution at year 10.** The franking mechanism brings the total tax to the individual's marginal rate. Retaining earnings in the company permanently keeps the rate at 30%. A "never distribute" scenario would make Pty Ltd the preferred structure for all brackets.
- **Fixed marginal rates.** The model assumes constant rates. In reality, dividends and realized gains push investors into higher brackets, increasing the Post-Budget penalty for mid-bracket investors.
