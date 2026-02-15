# Chapter 35: Default, Recovery, and Credit Events — Economic vs Contractual Reality

---

## Introduction

When a credit analyst says "XYZ Corp has defaulted," what exactly does that mean? The answer is less obvious than it first appears—and getting it wrong can be extraordinarily expensive.

Consider this scenario: a trader holds $50 million notional of CDS protection on a distressed issuer. The reference entity's bonds have collapsed from 95 to 35, equity has cratered, and the financial press is full of restructuring speculation. The trader expects a windfall from the CDS protection. But weeks pass, and no payment arrives. The contract hasn't triggered because no **credit event**—the precise contractual definition that activates CDS protection—has yet occurred. Economic distress is not the same as a contractual trigger.

This distinction between *economic* distress (what markets price continuously) and a *contractual* credit event (the legal trigger for CDS settlement) lies at the heart of credit derivatives. A CDS can remain "dormant" while spreads widen and bonds trade at distressed prices if the legally defined trigger has not occurred (or has not been noticed/settled under the contract process).

Prerequisites: [Chapter 02 — Time Value, Discount Factors, and Replication](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 05 — Fixed Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md), [Chapter 08 — Spreads 101](chapters/chapter_08_spreads_101.md)

Follow-on: [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md), [Chapter 37 — Cash Credit: Risky Bonds, Spreads, and CS01](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md), [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 39 — CDS Credit Events and Settlement](chapters/chapter_39_cds_credit_events_settlement.md), [Chapter 40 — The CDS Auction Process](chapters/chapter_40_cds_auction_process.md)

## Learning Objectives
- Distinguish economic distress from a contractual credit event (and explain why CDS may not pay immediately when a name is distressed).
- Define recovery rate and loss-given-default (LGD), and translate post-default bond prices into \(R\) and \(LGD\).
- Translate a CDS quote into the default-contingent cashflows: protection payment and accrued premium on a concrete timeline.
- Explain and compute the physical-settlement delivery option from multiple deliverable prices.
- Define and interpret desk risk scalars: jump-to-default and recovery-rate DV01 (bump object, bump size, units, sign).

## Setup (This Chapter)
- Perspective: cashflows are written from the **protection buyer** perspective unless stated otherwise (positive = cash received).
- Units: bond prices \(P\) are quoted as price per 100 face; CDS notional \(N\) is currency.
- Recovery and LGD: \(R \in [0,1]\), \(LGD := 1-R\). If the bond/auction final price is \(P\) per 100, use \(R=P/100\).
- Spreads: \(S\) is an annual rate in **decimal** (e.g., 200 bp = 0.02). One basis point is \(1\text{ bp}=10^{-4}\).
- Premium accrual examples use ACT/360 with simple accrual: accrued premium \(\approx N \cdot S \cdot \alpha\).

1. **Economic Default vs Contractual Credit Event** — Why spreads can blow out while CDS contracts sit dormant
2. **Par vs Distressed Trading Conventions** — How the market quotes instruments changes as credit deteriorates
3. **Recovery Rates and Loss Given Default** — How much you get back, and why it's not a fixed number
4. **Credit Risk Premium Decomposition** — Why market spreads exceed actuarial expectations
5. **Credit Event Taxonomy** — The ISDA definitions that determine when protection pays
6. **Structural Model Intuition** — The Merton model preview connecting equity and credit

Understanding these concepts is essential before tackling CDS mechanics (Chapter 38), CDS pricing (Chapter 41), survival curve construction (Chapter 36), or credit relative value (Chapter 44). Every formula in those chapters builds on the definitions established here.

---

## 35.1 Economic Default vs Contractual Credit Event

### The Distinction That Costs Money

Traders often use "default" loosely in conversation, but CDS contracts settle only when a contractually defined **credit event** occurs. That distinction hides a crucial difference:

**Economic default** is a continuous deterioration visible in market prices. Spreads widen, equity falls, bond prices collapse, funding dries up. This is what traders watch on their screens.

**Contractual credit event** is a discrete boundary crossing defined by legal documentation. Your CDS protection pays only when a defined credit event occurs, the notice/evidence conditions are satisfied, and the settlement steps complete.

The mismatch creates **timing risk** for hedgers. A bond-plus-CDS hedge can show substantial P&L volatility during the "gray zone" where economic distress is obvious but no contractual trigger has occurred.

> **Analogy: The Fire Alarm**
>
> *   **Economic Default (Smoke)**: The room is filled with smoke. Everyone *knows* there is a fire. The price of the house (bonds) has collapsed.
> *   **Credit Event (Sprinklers)**: The sprinklers (CDS Payout) only turn on if someone pulls the specific red handle on the wall.
> *   **The Risk**: You might die of smoke inhalation (lose money on the bond) while standing right next to the sprinkler system, just because the handle hasn't been pulled yet (no legal "Failure to Pay").

### 35.1.1 Par vs Distressed Trading Conventions

As credit quality deteriorates, the language of quoting shifts.

**Par-ish trading (near par)**

When bonds trade near par, the market naturally focuses on **yield and spread**: "What spread am I earning for bearing credit risk over this horizon?"

**Distressed trading (far below par)**

When bonds trade far below par, the question becomes more like: "What fraction of par will I recover, and when?" Quoting shifts toward **price/recovery language** ("cents on the dollar").

> **Desk Reality: The Distressed Quote Shift**
>
> A trader calling for a quote on a stressed name will hear very different language:
> - *Par trading*: "I'm bid at 250 over, offered at 245"
> - *Distressed trading*: "I'm bid at 35 cents, offered at 37"
>
> This shift is practical: for deeply distressed credits, yield-to-maturity is mathematically definable but often a poor summary of the economics because the dominant uncertainty is *default timing and recovery*.

**CDS quoting: fixed coupon + upfront (bond-like quoting)**

To facilitate trading, CDS (and especially CDS indices) are often standardized with a fixed running coupon and a recovery assumption. The market still quotes a spread (analogous to a bond yield), and an **upfront** amount is exchanged at trade time to reflect the PV difference between the quoted spread and the fixed coupon for the remaining life of the contract. After the trade, the protection buyer pays the fixed coupon on surviving notional.

In this chapter, treat "the CDS quote" as a small bundle you must keep straight: running coupon, any upfront, and the settlement recovery convention.

> **Pitfall — CDS “spread” vs “upfront”:** A CDS can be quoted as a running spread *or* as fixed-coupon + upfront. Mixing these quote objects is a common source of 100× errors.
> **Quick check:** spread is a *rate* (bp per year); upfront is *currency* (points × notional).

### 35.1.2 “Default” Is Not One Definition

Rating agencies, legal documentation, and market participants can use the word *default* differently. CDS contracts therefore focus on a specific set of contractually defined **credit events**. A common simplifying description is that credit events include *failure to pay*, *bankruptcy*, and (depending on the contract) *restructuring*.

The practical implication is basis and timing risk: a name can look economically "defaulted" (prices and spreads) while no contractual trigger has occurred (or while the settlement process is still in flight).

### 35.1.3 Historical vs Risk-Neutral (Preview)

Historical default frequencies are useful for intuition and stress testing. Pricing and hedging, however, use **risk-neutral** quantities implied by market prices (Chapter 36). In practice, hazard rates inferred from credit spreads (given an assumed recovery) can differ materially from hazard rates inferred from historical default frequencies, reflecting both risk premia and modeling choices.

---

## 35.2 Recovery Rates: What You Get Back

### 35.2.1 Defining Recovery

A common operational definition in credit risk is:

- **Bond recovery rate:** the recovery rate for a bond is normally defined as the price at which it trades about 30 days after default as a percent of its face value.

If a defaulted bond trades at price \(P\) per 100 of face value, define

$$\boxed{R := \frac{P}{100}}, \qquad \boxed{LGD := 1-R}$$

For **CDS cash settlement**, the same arithmetic is used, but \(P\) is the contractual settlement price (often an auction final price) expressed per 100 of par.

**Recovery vocabulary (three related objects)**

| Term | What it is | Where it shows up |
|---|---|---|
| Post-default bond price \(P\) | observed trading level after default | cash bonds (“cents on the dollar”) |
| Credit-derivatives recovery price | settlement price used for CDS payoff | CDS cash settlement |
| Workout recovery | ultimate value delivered through bankruptcy/reorg | realized over months/years |

> **Pitfall — Recovery convention mismatch:** “Recovery” is not a single universal object (bond price shortly after default vs auction final price vs ultimate workout). Always name which one you are using.

### 35.2.2 Recovery Rate Statistics by Seniority

Recovery depends strongly on where an instrument sits in the capital structure. As one empirical snapshot, average recoveries on US corporate bonds (as % of face value) vary by class:

| Class | Average recovery (% of face) |
|---|---:|
| Senior secured bond | 52.2 |
| Senior unsecured bond | 37.2 |
| Senior subordinated bond | 31.0 |
| Subordinated bond | 31.4 |
| Junior subordinated bond | 24.7 |

Two practical takeaways that show up repeatedly in credit work:
- **Seniority matters** (senior tends to recover more than subordinated).
- **Recovery distributions are broad**, and strict capital-structure “waterfalls” need not hold exactly in realized outcomes.

Different datasets and conventions produce different “typical recovery” summaries. The most relevant recovery value for the credit derivatives market is the expected recovery rate for senior unsecured bonds. This is equal to 34.89%. The median value is 42.27%. Treat these as rough summaries of a wide distribution, not constants.

### 35.2.3 Absolute Priority Rule and Violations

The **Absolute Priority Rule (APR)** is the legal principle that senior creditors must be paid in full before junior creditors receive anything. In theory, the waterfall proceeds as:

1. Secured creditors (up to collateral value)
2. Senior unsecured creditors
3. Subordinated creditors
4. Preferred equity
5. Common equity

Empirically, the **Absolute Priority Rule** is not always obeyed in the U.S.; in some cases, holders of subordinated debt can recover more than holders of more senior debt. Several mechanism-level forces can push outcomes away from a strict waterfall:

**Negotiated Settlements:** Bankruptcy is expensive and time-consuming. Senior creditors may accept less than full recovery to avoid protracted litigation, especially when junior creditors threaten to delay proceedings.

**Option Value of Equity:** In Chapter 11 reorganizations, equity holders retain some claims on the restructured entity. The option value of potential recovery gives them bargaining power.

**Timing and Liquidity:** Senior creditors may prefer quick, certain recovery over uncertain claims that could take years to resolve.

> **Desk Reality: APR Violations in Practice**
>
> A strict waterfall is a useful *starting point*, but realized recoveries by seniority can violate it. If you are mapping prices ↔ hazard ↔ recovery (or hedging with CDS), make sure your model assumptions match the recovery concept and deliverable obligations you are actually using.

### 35.2.4 Recovery vs Default Rate Correlation

Recovery rates are significantly negatively correlated with default rates. One way to remember this is the “doubly bad” intuition: bad macro regimes bring both *more* defaults and *worse* prices on defaulted paper.

For example, when the default rate on non-investment-grade bonds in a year is \(1\%\), the recovery rate tends to be relatively high (about \(55\%\) on average); when this default rate is \(10\%\), the recovery rate tends to be relatively low (about \(30\%\) on average).

Mechanism-level intuition (not a law): when many names default, distressed assets can flood the market at the same time, and prices can be pushed down by limited risk-bearing capacity and liquidity.

### 35.2.5 Credit Derivatives Recovery vs Workout Recovery

Credit derivatives and cash bonds can use the word *recovery* for different measurement windows:

- In the **credit-derivatives market**, the recovery price is the price of some defined reference obligation determined within 72 days of the default event.
- In **cash-bond “workout”**, the ultimate value received can take years and can include mixtures of cash, new debt, and equity.

> **Desk Reality: CDS vs Workout Recovery Divergence**
>
> Consider a hypothetical company that defaults with a CDS settlement price of 25 per 100. Over the next few years, the bankruptcy process unfolds and bondholders ultimately receive 45 per 100 in a mix of new debt, equity, and cash.
>
> The CDS protection buyer received $(1 - 0.25) = 75\%$ of notional at the auction. The bondholder who held through workout received only $(1 - 0.45) = 55\%$ loss. The CDS overcompensated by 20 points.
>
> Conversely, if new negative information emerges post-default (fraud, environmental liabilities), workout recovery could be lower than the auction price.

For CDS pricing, the credit derivatives recovery price is the relevant concept.

---

## 35.3 Credit Risk Premium: Real-World vs Risk-Neutral

A traded credit spread is not a pure “expected loss” number. Default probabilities inferred from market spreads are **risk-neutral** (pricing) quantities, while default probabilities measured from historical data are **real-world** (physical) quantities. The difference between them is one way credit risk premia show up in prices.

### 35.3.1 Anchor: Two Default-Probability Objects

- **Real-world (physical)** default probabilities / hazards: estimated from realized defaults; useful for scenario analysis and stress testing.
- **Risk-neutral** default probabilities / hazards: implied by market prices; used to value cashflows via risk-neutral valuation.

### 35.3.2 Expand: Why Spreads Can Exceed Actuarial Expected Loss

Several effects push market spreads above a simple actuarial expected-loss calculation:

1. **Systematic credit risk:** default rates vary over the business cycle; defaults cluster rather than arriving as independent events.
2. **Liquidity and funding premia:** corporate credit can be harder to finance and to trade than government bonds.
3. **Taxes and other non-default components:** for some high-quality bonds, non-default components can explain a meaningful share of observed spreads.
4. **Recovery uncertainty (and cyclicality):** recoveries tend to be lower in high-default environments, amplifying tail losses.

### 35.3.3 Check: A Quick Expected-Loss Back-of-the-Envelope (Use With Care)

In a toy “flat hazard” setting, an actuarial expected-loss rate is roughly

$$S_{EL} \approx h \cdot (1-R) = h \cdot LGD,$$

where \(h\) is a hazard rate (per year) and \(R\) is recovery.

**Toy numbers (hypothetical):** if \(h=2\%\) per year and \(R=40\%\) (\(LGD=60\%\)), then \(S_{EL} \approx 0.02 \times 0.60 = 0.012 = 120\) bp.

If the observed market spread is 200 bp, the “extra” 80 bp is not automatically a mistake: it can reflect risk premia, liquidity/tax effects, and model/convention choices.

> **Pitfall — “\(h \approx S/LGD\)” without assumptions:** The “credit triangle” heuristic comes from strong simplifications (flat hazard, simplified discounting, recovery convention, etc.). Use it for intuition and sanity checks, not as an identity.

---

## 35.4 Credit Events: The Contractual Trigger

A CDS pays only after a contractually defined **credit event**. A practical high-level classification is:

- **Hard credit events:** post-event debt tends to trade at (roughly) one distressed price.
- **Soft credit event:** **restructuring**; the debt can continue trading with a term structure of prices.

### 35.4.1 Anchor: Hard vs Soft

A hard credit event is one that "would cause all the debt of the reference entity to become immediately due and payable and hence trade at the same price."

Restructuring is the only soft credit event. Following a restructuring event, debt can continue trading with a term structure of prices. Short dated assets will tend to trade at higher prices than long dated assets, and assets with higher coupons will trade with a higher price than those with lower coupons.

### 35.4.2 Expand: What Counts as a “Credit Event” (High Level)

In simplified terms, credit events are usually defined around:

- failure to make a payment as it becomes due,
- bankruptcy, and
- restructuring of debt.

Restructuring is sometimes excluded in North American credit default swap contracts.

For this chapter, the important point is the operational consequence: economic distress can be extreme while the contractual trigger has not yet occurred, and the settlement price used for CDS can differ from a “workout” notion of recovery.

**Check (limiting case):** If no contractual credit event occurs before the CDS matures, the protection leg pays **0**—even if the issuer traded at distressed prices for months. CDS protection pays on the *contract trigger*, not on “how bad things look on the screen.”

### 35.4.3 Check: Why Soft Events Create a Delivery Problem

After a hard event, many deliverable obligations collapse to similar prices, so “deliver any bond for par” is close to “pay cash at one recovery price.”

After a restructuring, obligations can trade at different prices (term structure + coupon effects). That price dispersion is exactly what makes the **delivery option** (Section 35.5.5) valuable.

### 35.4.4 Restructuring Clauses (Preview)

Because restructuring creates heterogeneous deliverable prices, contracts can modify deliverability rules *after a restructuring* to reduce the value of the delivery option. One important example is the introduction of a **Modified Restructuring (Mod-Re)** clause in May 2001 in the U.S., which restricted the maturity of deliverable obligations following a restructuring credit event.

This is one source of regional/vintage differences in CDS contract behavior; details are deferred to Chapter 39.

---

## 35.5 CDS Cashflows, PV, and Risk (Bird’s-Eye View)

This section is the “spreadsheet recipe” view of a CDS: translate the quote into cashflows, write the PV equation, and name the risk scalars.

### 35.5.1 Quote → Cashflow Objects

A single-name CDS trade is specified (conceptually) by:

- Notional \(N\).
- A running spread \(S\) (or a fixed coupon plus an upfront amount, as discussed in Section 35.1.1).
- Payment dates \(\{T_i\}\) and accrual fractions \(\alpha_i\) (payments are often quarterly in arrears, and contracts often use standard maturity dates).
- A contractual **credit event** definition and maturity date.
- A settlement convention (cash vs physical) and a settlement recovery \(R\) (often expressed as “price per 100” divided by 100).

### 35.5.2 Anchor: Premium Leg and Protection Leg

From the protection buyer perspective (positive = cash received):

- **Protection leg (default-contingent):**
  $$\boxed{\Pi_{\text{prot}} = N(1-R) = N \cdot LGD.}$$
  **Checks:** \(0 \le \Pi_{\text{prot}} \le N\); higher recovery \(R\) means a smaller protection payment.

- **Premium leg (paid while the name survives):** periodic payments \(-N \cdot S \cdot \alpha_i\) on scheduled dates, and because payments are in arrears, a final **accrued premium** is typically due if default happens between payment dates:
  $$\boxed{\Pi_{\text{accrued prem}} \approx N \cdot S \cdot \alpha(T_{\text{prev}}, \tau).}$$

> **Pitfall — CDS premium accrual on default:** The regular payments from the buyer of protection to the seller of protection cease when there is a credit event. However, because these payments are made in arrears, a final accrual payment by the buyer is usually required (the stub from the last payment date to the credit event date). Forgetting it is a common settlement/P&L error.

### 35.5.3 PV Equation (Preview)

The mark-to-market value is the present value of expected (risk-neutral) cashflows:

$$\boxed{V_{\text{CDS}} = PV(\text{protection leg}) - PV(\text{premium leg}).}$$

This chapter focuses on translating quotes into cashflows and risk intuition; the full “hazard curve + discounting” machinery appears in Chapters 36 and 41–42.

### 35.5.4 Cash vs Physical Settlement (and the Auction Recovery)

- **Physical settlement (conceptual):** the protection buyer can deliver eligible obligations (bonds/loans) and receive par in cash. The contract allows a *basket* of deliverables rather than a single bond.
- **Cash settlement (common in practice):** an ISDA-organized auction is used to determine the mid-market value of the **cheapest deliverable** bond several days after the credit event. The auction price (per 100 of face value) defines \(R\) via \(R=P/100\).

**Check (hard-event “one price” limit):** If, after a credit event, all deliverable obligations trade at essentially the same cash price \(P\) per 100, then cash and physical settlement are economically equivalent: physical settlement delivers a bond worth \(P\) and receives par, implying a payoff of \(N(1-P/100)=N(1-R)\), which is exactly the cash-settlement formula.

### 35.5.5 The Delivery Option

The ability to choose among deliverable obligations is a **delivery option**. It only has value when different deliverables trade at different prices following a credit event (which is why it is most naturally associated with a soft credit event like restructuring).

For a protection buyer hedging a particular security, the delivery-option value after a credit event is (conceptually) the difference between the value of the security they hold and the value of the cheapest deliverable.

**Check (toy example, hypothetical):** suppose you are hedging a bond quoted at 43 per 100, but after a restructuring another deliverable is quoted at 37. You can (i) sell the 43 bond, (ii) buy the 37 deliverable, and (iii) deliver the cheapest bond into physical settlement. The delivery option is worth about \(43-37=6\) points per 100 (i.e., \(6\%\) of notional).

**Check (non-negativity and dollars):** For the protection buyer, the delivery option value is non-negative: you can always choose to deliver the cheapest eligible obligation available. In the toy example, “6 points” is \(6\%\) of notional, so on \(N=\$100\text{mm}\) the delivery-option value is about \(\$6\text{mm}\).

### 35.5.6 Risk Scalars (Bump Object, Size, Units, Sign)

Even before you can do full CDS valuation, you should be able to define (and sanity-check) two desk risk scalars.

**Jump-to-default (JTD)** (cashflow-level intuition)
- **Bump object:** default time \(\tau\) / credit-event indicator.
- **Bump size:** “jump from survive to default now” (a discrete event, not a 1bp bump).
- **Units:** currency.
- **Sign (protection buyer):** usually positive (default creates the protection payment).
- **Cash approximation:** \(JTD \approx N(1-R) - \text{accrued premium}\) (ignoring discounting and any settlement lag).

**Recovery-rate DV01** (recovery sensitivity)
- **Bump object:** recovery rate \(R\) used for settlement/valuation.
- **Bump size:** absolute \(1\%\) change in recovery, \(\Delta R = 0.01\) (not “1% relative”).
- **Units:** currency per 1% recovery.
- **Sign convention (this chapter):** \(\text{RecDV01} := PV(R \downarrow 1\%) - PV(\text{base})\).
- **Check (cashflow-level):** for the pure default payoff \(N(1-R)\), an absolute 1% decrease in recovery increases the payoff by \(0.01\,N\).

> **Desk Reality:** In mark-to-market risk, “recovery DV01” is often computed by bumping the assumed recovery and re-implying the hazard/survival curve so that market spreads remain matched. The resulting sign can depend on the position’s moneyness; keep the “cashflow-level” check above as your sanity anchor.

### 35.5.7 Worked Example — Cash-Settled CDS Default Cashflows (with Accrued Premium)

**Example Title**: Cash-settled CDS: protection payment + accrued premium on real dates

**Context**
- You are long protection and want to translate “spread + recovery” into the cash you will receive/pay if a credit event occurs.
- This is a desk-critical check: it is the first thing you reconcile against settlement statements.

**Timeline (Make Dates Concrete)**
- Trade date: 2020-03-20
- Premium payment dates (quarterly, in arrears): 2020-06-20, 2020-09-20, 2020-12-20, …, 2023-03-20, …
- Credit event / default date: 2023-05-20
- Cash settlement date: 2023-05-25 (hypothetical “several days later” date for the cashflow table)

**Inputs**
- Notional: \(N=\$100{,}000{,}000\)
- Running spread: \(S=90\text{ bp}=0.009\)
- Day count for accrual (example convention): ACT/360
- Auction final price: \(P=35\) per 100 \(\Rightarrow R=0.35\), \(LGD=0.65\)

**Outputs (What You Produce)**
- Protection payment: \(N(1-R)\)
- Accrued premium due at default: \(N \cdot S \cdot \alpha(2023\text{-}03\text{-}20, 2023\text{-}05\text{-}20)\)
- Net default cashflow (buyer): protection payment minus accrued premium
- Risk scalars (cash approximations): JTD and RecDV01

**Step-by-step**
1. Translate the auction price into recovery: \(R=P/100=0.35\).
2. Compute the protection payment: \(N(1-R)=100{,}000{,}000\times 0.65=\$65{,}000{,}000\).
3. Compute accrued premium from the last payment date to default: there are 61 days from 2023-03-20 to 2023-05-20, so \(\alpha=61/360\).
4. Accrued premium \(=100{,}000{,}000 \times 0.009 \times (61/360)=\$152{,}500\).
5. Net default cashflow to the protection buyer (ignoring discounting and settlement lag): \(65{,}000{,}000-152{,}500=\$64{,}847{,}500\).
6. Cashflow-level RecDV01 check: an absolute 1% decrease in recovery changes the protection payment by \(+0.01N=+\$1{,}000{,}000\).

**Cashflows (table)**
| Date | Cashflow (protection buyer) | Explanation |
|---|---:|---|
| 2023-05-25 | \(+\$65{,}000{,}000\) | Cash-settled protection payment \(N(1-R)\) |
| 2023-05-25 | \(-\$152{,}500\) | Accrued premium from 2023-03-20 to 2023-05-20 (ACT/360) |

**P&L / Risk Interpretation**
- The protection payment is the “big number”; the accrued premium is a small but real settlement item.
- If you are hedging a bond, the hedge is not “free carry”: you paid premiums up to the event, and the cash arrives after a process lag.

**Sanity Checks**
- Units check: \(N\) (currency) × \(S\) (per year) × \(\alpha\) (years) = currency.
- Sign check: protection buyer receives \(+N(1-R)\) and pays \(-\)premium.
- Bounds: if \(R=1\), protection payment is 0; if \(R=0\), it is \(N\).

**Debug Checklist (When Your Result Looks Wrong)**
- Are you using “price per 100” consistently when converting \(P\) to \(R\)?
- Did you use the correct last premium date (the previous standard date) and the contract day count?
- Are you mixing “running spread” with “fixed coupon + upfront” quoting?

---

## 35.6 Structural Model Intuition: The Merton Preview

Reduced-form (hazard-rate) models are the workhorse for CDS valuation (Chapter 36), but structural models are a useful mental model: default happens when the firm’s assets are insufficient to repay debt.

### 35.6.1 Anchor: Equity Is a Call on Firm Assets

Let \(V_T\) be the firm’s asset value at time \(T\), and let \(D\) be the debt repayment due at \(T\). A simple default rule is:

- If \(V_T < D\), default is rational and equity is worth 0.
- If \(V_T > D\), the firm repays debt and equity holders receive \(V_T-D\).

So the equity payoff is:

$$\boxed{E_T=\max(V_T-D,0).}$$

That is, equity behaves like a call option on the firm’s assets with strike \(D\).

### 35.6.2 Expand: Option-Pricing Link (Preview)

Under additional modeling assumptions (e.g., a diffusion model for \(V_t\) with volatility \(\sigma_V\) and a constant risk-free rate \(r\)), one can relate today’s equity value \(E_0\) to today’s asset value \(V_0\) via a Black–Scholes–Merton style expression:

$$E_{0}=V_{0}\Phi(d_{1})-D e^{-r T}\Phi(d_{2}),$$

where

$$d_{1}=\\frac{\\ln(V_{0}/D)+(r+\\sigma_{V}^{2}/2)T}{\\sigma_{V}\\sqrt{T}}, \\qquad d_{2}=d_{1}-\\sigma_{V}\\sqrt{T}.$$

The debt value in this simple setup is \(V_0-E_0\).

This is not how CDS curves are built in practice, but it explains why equity and credit move together: both are claims on the same underlying asset value.

**Check (distance-to-default intuition):** In this setup, default occurs when \(V_T<D\). The term \(d_2\) is often interpreted as a “distance to default”: larger \(V_0\) (all else equal) increases \(d_2\) and makes default less likely; higher asset volatility \(\sigma_V\) tends to reduce \(d_2\) and make default more likely even while it increases the option value of equity. This is one reason equity volatility and credit spreads often co-move.

### 35.6.3 Check: Limiting Cases (Intuition)

- If \(V_0\) is far above \(D\), equity is deep in-the-money and default is unlikely over short horizons.
- If \(V_0\) is near \(D\), equity is option-like: small changes in \(V_0\) (or in volatility) can move default risk meaningfully.

---

## Summary

1. **Economic distress** (what markets price continuously) is not the same as a **contractual credit event** (the legal trigger for CDS settlement). This creates timing and basis risk in the “gray zone.”

2. **Recovery** is not one universal object. Be explicit about whether you mean a post-default bond price, a CDS auction settlement price, or an ultimate workout recovery.

3. A practical recovery convention is: if the relevant post-event price is \(P\) per 100 of face, then \(R=P/100\) and \(LGD=1-R\).

4. Recoveries vary by **seniority** and can be lower in high-default environments (“doubly bad”): high default years tend to coincide with low recovery years.

5. Default probabilities inferred from traded spreads are **risk-neutral**; default probabilities measured from history are **real-world**. The gap reflects risk premia and other spread components (liquidity, taxes, model conventions).

6. Credit events can be usefully grouped into **hard** vs **soft**: hard events collapse debt toward one distressed price; **restructuring** is the soft event that can leave a term structure of post-event prices.

7. A CDS quote maps to two cashflow legs: the **premium leg** (periodic spread payments while the name survives, plus accrued premium on default) and the **protection leg** (a default-contingent payment \(N(1-R)\)).

8. Settlement conventions matter. Cash settlement uses an auction-based recovery price; physical settlement embeds a **delivery option** when deliverables trade at different prices.

9. Two desk risk scalars you should define cleanly are **jump-to-default (JTD)** and **recovery-rate DV01**: state bump object, bump size, units, and sign convention.

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Economic distress | Deterioration visible in prices/spreads | Can move long before any contractual trigger |
| Credit event | Contractual trigger for CDS settlement | Determines if/when protection pays |
| Hard credit event | Event that collapses obligations toward one distressed price | Supports “one recovery” intuition |
| Soft credit event (restructuring) | Event after which obligations can keep trading with term structure | Creates delivery/settlement complexity |
| Recovery rate \(R\) | Relevant post-event price per 100 divided by 100 | Scales the protection payment |
| Loss given default \(LGD\) | \(1-R\) | Fraction of notional lost/paid on default |
| Premium leg | Periodic premium payments while the name survives | Ongoing cost of protection |
| Accrued premium on default | Stub premium from last pay date to default date | Must be included in settlement cashflows |
| Protection leg | Default-contingent payment \(N(1-R)\) | Main hedge payoff |
| Cash settlement | Settlement using an auction-based recovery price | Defines \(R\) for cash payoff |
| Physical settlement | Deliver eligible obligations for par | Introduces a delivery option |
| Delivery option | Right to choose which deliverable to deliver | Can be material after restructuring |
| Real-world vs risk-neutral | Historical vs market-implied default probabilities | Use risk-neutral for PV, real-world for scenarios |
| Jump-to-default (JTD) | PV/cash impact of “default now” | Discrete risk not captured by small spread bumps |
| Recovery-rate DV01 | PV change for a 1% absolute recovery change | Sensitivity to recovery assumptions |
| Merton model (structural) | Equity as an option on firm assets | Intuition linking equity and credit

---

## Notation

| Symbol | Definition |
|--------|------------|
| \(N\) | CDS notional (currency) |
| \(P\) | Relevant post-event price per 100 of face (bond price or auction price) |
| \(R\) | Recovery rate \(=P/100\) (unitless) |
| \(LGD\) | Loss given default \(=1-R\) (unitless) |
| \(S\) | Running CDS spread (per year, decimal) |
| \(T_i\) | Premium payment dates |
| \(\alpha(t_1,t_2)\) | Year fraction between dates (ACT/360 in examples) |
| \(\tau\) | Default/credit-event time or date |
| \(h(t)\) | Hazard rate (per year) |
| \(V_{\text{CDS}}\) | CDS mark-to-market value (currency) |
| \(\text{JTD}\) | Jump-to-default (currency) |
| \(\text{RecDV01}\) | PV change for \(R \downarrow 1\%\) (currency per 1% recovery) |
| \(V_0, V_T\) | Firm asset value at time 0 and \(T\) (Merton) |
| \(D\) | Debt repayment due at \(T\) (Merton) |
| \(E_0, E_T\) | Equity value at time 0 and \(T\) (Merton) |
| \(r\) | Risk-free rate (per year) |
| \(\sigma_V\) | Asset volatility (1/\(\sqrt{\text{year}}\)) |
| \(\Phi(\cdot)\) | Standard normal CDF |
| \(d_1, d_2\) | Black–Scholes terms (Merton) |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Economic distress vs credit event — what’s the difference? | Distress is market deterioration; a credit event is the contractual trigger for CDS settlement |
| 2 | Name three common credit-event types (simplified) | Failure to pay, bankruptcy, restructuring (sometimes excluded by contract) |
| 3 | Hard vs soft credit event — key difference? | Hard events collapse obligations toward one distressed price; restructuring (soft) can leave a term structure of prices |
| 4 | Name the only soft credit event in this taxonomy | Restructuring |
| 5 | Define recovery rate \(R\) in this chapter | \(R=P/100\) where \(P\) is the relevant post-event price per 100 of face (bond price or auction price) |
| 6 | Define \(LGD\) | \(LGD=1-R\) |
| 7 | Cash-settled protection payment to the protection buyer | \(N(1-R)\) |
| 8 | Premium-leg periodic payment (sign) | Protection buyer pays \(-N\cdot S \cdot \alpha\) |
| 9 | Why is accrued premium due on default? | Premiums are usually paid in arrears, so the stub from last payment date to default is owed |
| 10 | In cash settlement, what determines the recovery price? | An auction-based price for the cheapest deliverable (converted to \(R=P/100\)) |
| 11 | What is physical settlement? | Protection buyer can deliver eligible obligations for par and choose which deliverable to deliver |
| 12 | What is the delivery option? | The right to choose among deliverables; valuable when deliverables trade at different prices |
| 13 | When is the delivery option most likely to matter? | After restructuring (soft event) when prices across obligations can differ |
| 14 | Real-world vs risk-neutral default probabilities | Real-world: estimated from history; risk-neutral: implied by traded spreads and used for PV |
| 15 | Define jump-to-default (JTD) | PV/cash impact of a “default now” scenario (a discrete jump, not a 1bp bump) |
| 16 | Define recovery-rate DV01 (state bump clearly) | PV change for an absolute 1% recovery move; e.g., \(PV(R\downarrow 1\%) - PV(\text{base})\) |
| 17 | What does “APR not always obeyed” mean? | In some cases, subordinated debt can recover more than more senior debt |
| 18 | Merton model equity payoff | \(E_T=\max(V_T-D,0)\): equity is a call on firm assets with strike equal to debt repayment |
| 19 | Why do equity and credit co-move in a structural model? | Both are claims on the same underlying asset value \(V_t\) |

---

## Mini Problem Set

**1. Protection payment.** A CDS has notional \(\$25{,}000{,}000\) and recovery \(R=0.35\). Compute the cash-settled protection payment.

**2. Accrued premium.** Notional \(\$50{,}000{,}000\), running spread 150 bp, 40 days since the last premium date on ACT/360. Compute accrued premium due on default.

**3. Delivery option.** Two deliverables are quoted at 45 and 38 per 100 after a restructuring. Notional \(\$10{,}000{,}000\). Compute the delivery-option value (in dollars).

**4. JTD (cash approximation).** You are a protection buyer with notional \(\$100{,}000{,}000\), recovery \(R=0.35\), running spread 200 bp, and 30 days of premium accrued (ACT/360). Approximate \(JTD \approx N(1-R)-N S \alpha\).

**5. Recovery DV01 (cash check).** For a cash-settled CDS with notional \(\$100{,}000{,}000\), what is the change in the protection payment for an absolute 1% decrease in recovery?

**6. Concept (soft vs hard).** Why is restructuring “soft,” and why does that make settlement/delivery more complicated?

**7. Concept (risk-neutral vs real-world).** Give two reasons why market-implied (risk-neutral) default probabilities can differ from historical default frequencies.

**8. Concept (recovery object).** Name two reasons the CDS settlement recovery price can differ from ultimate workout recovery.

**9. Structural intuition.** In the Merton payoff \(E_T=\max(V_T-D,0)\), compute equity payoff when (a) \(V_T=80\), \(D=100\) and (b) \(V_T=130\), \(D=100\).

### Solution Sketches (Selected)

**1.** \(25{,}000{,}000\times(1-0.35)=\$16{,}250{,}000\).

**2.** \(\alpha=40/360\). Accrued \(=50{,}000{,}000\times 0.015\times (40/360)=\$83{,}333\) (rounded).

**3.** Difference \(=7\) points per 100 \(\Rightarrow 0.07\times 10{,}000{,}000=\$700{,}000\).

**7.** Two drivers: (i) default rates vary over time (systematic risk/default clustering), and (ii) spreads can include non-default components such as liquidity premia and taxes.

---

## References

- O’Kane, Dominic. *Modelling Single-name and Multi-name Credit Derivatives*. Sections 3.2.2–3.2.3 (recovery definitions/statistics; APR note), 5.4.1 (credit events: hard vs soft), 5.4.3 (delivery option), 5.4.4 (restructuring clause; Mod-Re), 8.3.6 (recovery-rate sensitivity / recovery DV01).
- Hull, John C. *Options, Futures, and Other Derivatives*. Business Snapshot 25.1 / Section 25.1 (CDS payment conventions, credit event definition, cash settlement and accrued premium), “Real-World vs. Risk-Neutral Probabilities” (risk-neutral vs historical default probabilities).
- Hull, John C. *Risk Management and Financial Institutions*. Section 19.3 (recovery definition, seniority recoveries, recovery–default correlation), “The Use of Fixed Coupons” (fixed coupon + upfront quoting).
- Damiano, Andrea. *Counterparty Credit Risk, Collateral and Funding*. Section 3.1.4 (formal CDS premium/protection legs and accrued premium).
- Elton, Edwin J., Martin J. Gruber, Stephen J. Brown, and William N. Goetzmann. *Modern Portfolio Theory and Investment Analysis*. Corporate bond spread discussion (default vs taxes vs systematic risk components).
