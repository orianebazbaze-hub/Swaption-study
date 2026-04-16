# Hedging Mortgage Negative Convexity with Swaptions

> **A Research Memorandum — ALM & Interest Rate Risk**
>  Interest Rate Portfolio Management · 

---

## At a Glance

- **Problem.** Fixed-rate retail mortgages embed a prepayment option that gives the bank negative convexity. When rates fall, borrowers refinance en masse and the bank reinvests at lower yields; when rates rise, borrowers stay and the bank is locked into below-market coupons. The downside dominates — this generates asymmetric EVE losses in IRRBB downward-rate scenarios, independent of any directional view on rates.

- **Hedge.** A laddered portfolio of EUR receiver swaptions (2Y5Y / 3Y7Y / 5Y10Y, modestly out-of-the-money) sized as a static replication of the expected mortgage amortisation schedule under stochastic prepayment. Receiver swaptions move in-the-money precisely when the bank needs it most — when rates fall and prepayments accelerate.

- **Cost.** In a normal EUR vol regime, a partial overlay covering ~10% of book notional runs at roughly 30–60 bp upfront — against meaningfully lower EVE sensitivity in parallel-down and steepener scenarios and improved IRRBB SOT headroom. The cost is highly vol-regime dependent: building the hedge after a vol spike is materially more expensive.

- **Scope.** This memo is a research landscape, not a model build. It covers the mechanics of mortgage convexity, the prepayment-modelling toolkit (PSA benchmark → rate-responsive CPR), the swaption hedge design space (static replication vs. dynamic delta-gamma, strike / expiry / tenor choices), and the open calibration questions specific to a French retail book.

> 📄 **Full technical analysis:** [`TECHNICAL_MEMO.md`](./TECHNICAL_MEMO.md) — prepayment models, hedge construction, swaption cost analysis, and open research questions.

---

## The Problem — Negative Convexity in a Mortgage Book

### What is negative convexity?

For a standard bond, duration falls when rates rise and rises when rates fall — this is **positive convexity**, and it works in the investor's favour. A fixed-rate mortgage behaves differently because the borrower holds an embedded prepayment option. When rates fall significantly, the rational borrower refinances: the bank receives the outstanding principal back but must reinvest it at the new, lower rates. This creates negative convexity:

- **Rates fall:** borrowers prepay → bank's asset duration shortens → bank must reinvest at lower yields
- **Rates rise:** borrowers hold on → bank's asset duration extends → bank is locked into below-market fixed rates

The asymmetry is the core problem. The bank is effectively **short a call option on the bond** — equivalently, short a put on interest rates, since bond prices rise as rates fall. This embedded option has a negative value to the bank and must be properly accounted for in ALM.

| Rate scenario       | Borrower behaviour                              | Bank impact                                                    |
| ------------------- | ----------------------------------------------- | -------------------------------------------------------------- |
| Rates fall −100 bp  | Refinancing surge — early repayment of capital  | Duration shortens; reinvests at lower rates; loss of NII       |
| Rates rise +100 bp  | Borrowers stay; no prepayment incentive         | Duration extends; funding cost rises; NII squeeze              |
| Rates stable        | Normal amortisation schedule                    | Expected cash flows; hedging cost is the option premium        |

### Why it matters for Corporate Treasury

Under IRRBB (BCBS 368 / EBA GL 2022/14), banks must measure and stress-test both **Economic Value of Equity (EVE)** and **Net Interest Income (NII)** sensitivity. A mortgage book with unhedged negative convexity will show asymmetric EVE losses: the downward rate scenarios (steepener, parallel down) generate outsized negative EVE, because the bank cannot benefit from the embedded optionality the way a rational investor would. This can push the bank close to the −15% Tier 1 EVE limit without any genuine interest rate directional bet.

Beyond regulation, negative convexity is a P&L risk: in a falling-rate environment, NII compression from prepayments occurs simultaneously with mark-to-market losses on fixed-rate funding positions, creating a double-hit on income and capital.

---

## Conclusion & Recommendation

Negative convexity embedded in fixed-rate retail mortgages represents one of the most structurally persistent forms of interest rate risk on a bank's balance sheet. It is asymmetric, path-dependent, and only partially captured by standard duration/DV01 metrics. Receiver swaptions provide the most natural and liquid hedge because they directly offset the economic loss the bank suffers when rates fall and borrowers prepay en masse.

The practical complexity lies not in the concept but in the **calibration**: choosing the right prepayment model, the right swaption parameters, and the right hedging ratio requires a combination of internal loan-level data, market vol data, and a clear view of the bank's risk appetite and regulatory constraints. This memo proposes that work begin on the calibration of a rate-responsive CPR model using the bank's internal loan tape, followed by a scenario analysis of swaption hedge effectiveness under the six standard IRRBB stress scenarios.

---

## Repository Contents

| File | Purpose |
| ---- | ------- |
| [`README.md`](./README.md) | Overview — the pitch, the problem, and the recommendation |
| [`TECHNICAL_MEMO.md`](./TECHNICAL_MEMO.md) | Full technical analysis: prepayment models, hedge construction, cost analysis, open questions |

---

<sub>*Confidential — for discussion purposes only.*</sub>
