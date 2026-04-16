# Technical Memo — Hedging Mortgage Negative Convexity

> **Technical deep-dive companion to the overview memo.**
> Corporate Treasury · Interest Rate Portfolio Management · April 2026

← [Back to overview](./README.md)

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Prepayment Models — Quantifying the Embedded Option](#prepayment-models--quantifying-the-embedded-option)
  - [The PSA / CPR benchmark](#the-psa--cpr-benchmark)
  - [Rate-responsive prepayment models](#rate-responsive-prepayment-models)
  - [Stochastic vs. deterministic prepayment](#stochastic-vs-deterministic-prepayment)
  - [Data sources for calibration](#data-sources-for-calibration)
- [The Hedge — Receiver Swaptions as Convexity Protection](#the-hedge--receiver-swaptions-as-convexity-protection)
  - [Why receiver swaptions?](#why-receiver-swaptions)
  - [Matching the hedge to the mortgage book](#matching-the-hedge-to-the-mortgage-book)
  - [Key parameters to determine](#key-parameters-to-determine)
  - [The cost dimension — vol regime matters](#the-cost-dimension--vol-regime-matters)
- [Open Questions and Research Directions](#open-questions-and-research-directions)
- [References & Data Sources](#references--data-sources)

---

## Executive Summary

Retail banks originating fixed-rate mortgages face a structural and often underappreciated form of interest rate risk: **negative convexity**. Unlike a standard bond, a fixed-rate mortgage loan does not behave symmetrically to rate movements. When rates fall, borrowers refinance early, returning capital to the bank at the worst possible time — just as reinvestment yields have declined. When rates rise, borrowers hold on to their low-rate loans, extending the bank's asset duration precisely when funding costs are rising.

This memorandum examines the mechanics of negative convexity in a retail mortgage book, the prepayment models used to quantify it, and the case for using receiver swaptions as a structural hedge. We do not propose a specific model to build; rather, we map out the research landscape, the key data sources, and the strategic questions a Corporate Treasury team must answer before sizing and executing a swaption overlay.

> **Key thesis.** *A portfolio of receiver swaptions — giving the bank the right to receive fixed and pay floating — can partially offset the negative convexity embedded in a fixed-rate mortgage book, at a cost that must be weighed against the volatility reduction and regulatory capital benefit it provides.*

---

## Prepayment Models — Quantifying the Embedded Option

### The PSA / CPR benchmark

The standard market benchmark for prepayment speed is the **Public Securities Association (PSA) model**, which defines a **Conditional Prepayment Rate (CPR)** — the annualised percentage of the outstanding principal expected to prepay each year. The PSA model assumes CPR ramps linearly by 0.2% CPR per month, starting at 0.2% CPR in month 1 and reaching 6% CPR at month 30, then stays flat at 6% CPR (100 PSA). Faster prepayment pools are expressed as multiples: 200 PSA means twice the CPR at each point.

The CPR is converted to a **Single Monthly Mortality (SMM)** rate — the proportion of the remaining balance prepaid each month — via:

$$\text{SMM} = 1 - (1 - \text{CPR})^{1/12}$$

The PSA model is a useful benchmark but is purely mechanical — it does not respond to the interest rate environment. For hedging purposes, a **rate-sensitive prepayment model** is essential.

### Rate-responsive prepayment models

Industry-standard prepayment models decompose prepayment speed into several multiplicative components, each capturing a different driver:

- **Refinancing incentive (RI):** the primary driver. Modelled as a function of the ratio of the contract rate to the current market mortgage rate. A common formulation uses the arctangent function to produce an S-shaped response — slow at low incentive, fast at high incentive, plateauing at the maximum prepayment speed:

$$\text{RI}(t) = 0.2406 - 0.1389 \times \arctan\!\left(5.952 \times \left(1.089 - \frac{C}{R(t)}\right)\right)$$

where $C$ is the contract coupon rate and $R(t)$ is the current market mortgage rate.

- **Seasoning multiplier:** prepayment is low at origination and ramps up over the first 2–3 years as borrowers settle and become eligible to refinance.
- **Seasonality:** prepayment peaks in spring/summer (housing market activity) and troughs in winter.
- **Burnout:** pools with a history of low prepayment have fewer refinancing-eligible borrowers remaining; they are less sensitive to rate incentives.

The full CPR is the product of these components:

$$\text{CPR}(t) = \text{RI}(t) \times \text{Seasoning}(t) \times \text{Seasonality}(t) \times \text{Burnout}(t)$$

### Stochastic vs. deterministic prepayment

A **deterministic** prepayment model maps a rate scenario to a single prepayment speed. A **stochastic** model acknowledges that borrower behaviour is not fully determined by the rate incentive — there is genuine uncertainty in how many borrowers will actually refinance at a given rate level. Recent academic literature (Perotti et al. 2024, arXiv:2410.21110) models this uncertainty as a non-hedgeable risk factor, leading to an **incomplete market framework** where perfect replication of the prepayment option is impossible. The practical implication is that any swaption hedge will leave **residual basis risk**.

### Data sources for calibration

Building a credible prepayment model for a French retail mortgage book requires the following data:

| Data source                         | Content                                                                 | Use                                              |
| ----------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------ |
| Internal loan tape                  | Origination date, coupon, LTV, region, borrower type, remaining balance | Calibrate RI, seasoning, burnout                 |
| Banque de France statistics         | Aggregate mortgage production, average rates, duration                  | Macro-level validation                           |
| ECB / EBA loan-level data           | Harmonised European mortgage tape (where available)                     | Benchmarking prepayment speed                    |
| Bloomberg / Refinitiv               | Current EUR mortgage rates, swap rates, vol surface                     | Rate incentive calculation                       |
| INSEE housing data                  | French housing transactions, property prices                            | Seasonality and burnout calibration              |
| SIFMA / ASF historical CPR          | Observed prepayment speeds for comparable pools                         | PSA benchmark comparison                         |

---

## The Hedge — Receiver Swaptions as Convexity Protection

### Why receiver swaptions?

A **receiver swaption** gives its holder the right — but not the obligation — to enter into a swap as the fixed-rate receiver (and floating-rate payer) at a predetermined strike rate and on a predetermined future date. This instrument is the natural hedge for mortgage negative convexity because:

- **When rates fall and borrowers prepay:** the bank receives the returned principal but faces lower reinvestment yields. The receiver swaption is now in-the-money — the bank can exercise and receive the higher pre-agreed fixed rate, compensating for the lost mortgage interest income.
- **When rates rise and borrowers stay:** the swaption expires worthless, but the bank continues to receive mortgage coupons above the new market rate. The cost is the premium paid.

> **Intuition.** *The bank bought the right to receive a fixed rate it defined before rates moved. When clients repay early (in a falling-rate world), the bank is not left empty-handed — it enters the swaption and keeps receiving the agreed fixed rate on the hedged notional.*

### Matching the hedge to the mortgage book

The structural challenge is that the notional of the mortgage book is not fixed — it amortises according to the scheduled repayment plan and the stochastic prepayment. The hedge must therefore be **dynamic** or constructed as a **replicating portfolio** of swaptions across different maturities. The literature identifies two main approaches:

- **Static replication (portfolio of European swaptions):** a grid of swaptions with start dates $T_i$ and tenors $T_j$ is calibrated so that their combined notionals replicate the expected amortisation schedule under various rate scenarios. Casamassima et al. (2022) and Perotti et al. (2024) develop this approach formally, solving a linear system $A\mathbf{w} = \mathbf{b}$ to find the optimal swaption notionals $w_{i,j}$.
- **Dynamic delta-gamma hedging:** continuously rebalance a portfolio of IRSs and swaptions to match the Delta (DV01) and Gamma (convexity) of the mortgage book. More expensive in terms of transaction costs but handles path-dependent prepayment more accurately.

### Key parameters to determine

Before executing any swaption hedge, Corporate Treasury must answer the following questions through quantitative analysis:

| Question                          | Why it matters                                                                                                                                         |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| What strike to use?               | At-the-money (ATM) strikes are most liquid but most expensive. Out-of-the-money (OTM) receiver swaptions cost less but protect only when rates fall below the strike. |
| What maturity / expiry?           | The expiry should match the peak prepayment risk horizon — typically 2–7Y for French mortgages. A ladder of expiries reduces concentration risk.       |
| What underlying swap tenor?       | Should match the residual maturity of the mortgage being hedged. A 3Y expiry into a 7Y swap (3×7 swaption) is common for mid-life mortgages.           |
| Bermudan vs. European?            | Bermudan swaptions (exercisable on multiple dates) better match the continuous prepayment option but are less liquid and harder to price.              |
| What notional coverage?           | Full convexity hedging is rarely optimal due to cost. A partial hedge targeting the worst 2–3 IRRBB scenarios is more practical.                       |
| How to account for basis risk?    | The swaption references a swap rate; the prepayment driver is the retail mortgage rate. The spread between them (mortgage–swap spread) introduces basis risk. |

### The cost dimension — vol regime matters

Swaption premiums are driven by **implied volatility** — the market's expectation of future interest rate movements. The cost of building a swaption hedge is therefore highly path-dependent: buying protection in a low-vol environment (as was the case in 2019–2021) is cheap; buying after a vol spike (as in 2022–2023 following the ECB hiking cycle) is expensive.

The relevant vol surface is the **EUR swaption vol cube**, which gives implied normal (Bachelier) volatility as a function of expiry, swap tenor, and strike. Key reference points for a French mortgage book:

- **2Y5Y receiver swaption** (2-year expiry, 5-year underlying): captures near-term refinancing risk for recently originated mortgages.
- **3Y7Y receiver swaption:** mid-book hedge for seasoned loans at peak prepayment sensitivity.
- **5Y10Y receiver swaption:** long-dated coverage, primarily for IRRBB EVE protection.

In normal vol regimes (implied normal vol ~60–80 bp for 5Y5Y EUR), the cost of a 10% OTM receiver swaption ladder covering 10% of the mortgage book notional runs at roughly 30–60 bp upfront — a meaningful but manageable cost relative to the NII and EVE stabilisation it provides.

---

## Open Questions and Research Directions

This memo does not resolve the full hedging problem — it maps the landscape. The following questions remain open and would require further quantitative work:

1. **Calibration of a French mortgage prepayment model:** how sensitive are French borrowers to rate incentives vs. other drivers (LTV, employment, housing mobility)? The OTS-style arctangent model may need recalibration to French data.
2. **Optimal strike selection:** what is the cost–benefit frontier of different strike levels given the bank's capital position and IRRBB tolerance?
3. **Bermudan vs. European swaption:** is the liquidity premium for Bermudan instruments justified given the bank's ability to delta-hedge dynamically?
4. **Hedge accounting (IFRS 9):** can the swaption hedge qualify for cash flow hedge accounting, eliminating P&L volatility from the fair value of the options?
5. **Interaction with existing IRS overlay:** how does the swaption convexity hedge interact with the existing fixed-rate bond and IRS book? Joint optimisation of the full ALM hedge portfolio is required.
6. **Vol regime timing:** given that vol is a key cost driver, is there an optimal approach to building the hedge incrementally rather than in one execution?

---

## References & Data Sources

**Regulation**

- BCBS 368 — *Interest Rate Risk in the Banking Book*, April 2016
- EBA/GL/2022/14 — *Guidelines on IRRBB and CSRBB*, October 2022

**Academic & industry research**

- Casamassima et al. (2022) — *Pricing and Hedging Prepayment Risk in a Mortgage Portfolio*, [arXiv:2109.14977](https://arxiv.org/abs/2109.14977)
- Perotti et al. (2024) — *Modeling and Replication of the Prepayment Option of Mortgages including Behavioral Uncertainty*, [arXiv:2410.21110](https://arxiv.org/abs/2410.21110)
- Perotti et al. (2025) — *Pricing and Hedging the Prepayment Option under Stochastic Housing Market Activity*, [arXiv:2507.08641](https://arxiv.org/abs/2507.08641)
- Malkhozov, Mueller, Vedolin, Venter — *Mortgage Hedging in Fixed Income Markets*, TSE Working Paper
- Hanson (2014) — *Mortgage Convexity*, Journal of Financial Economics
- BIS Working Paper 532 — *Mortgage Risk and the Yield Curve*
- DWS Research Institute — *Convexity and Prepayment Risk*, April 2024

**Data sources**

- Banque de France — Statistics on French mortgage production and rates ([stat.banque-france.fr](https://stat.banque-france.fr))
- ECB Statistical Data Warehouse — Euro area MFI interest rates and loan volumes
- Bloomberg EUR Swaption Vol Cube — `SWVC` function, normal (Bachelier) implied vols

---

← [Back to overview](./README.md)

<sub>*for discussion purposes only.*</sub>
