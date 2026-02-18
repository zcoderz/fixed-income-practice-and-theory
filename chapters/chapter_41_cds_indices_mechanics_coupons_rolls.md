# Chapter 41: CDS Indices — Mechanics, Coupons, Rolls

---

## Introduction

CDS indices (such as **CDX** and **iTraxx**) let you trade diversified credit exposure in one ticket. They are common hedges for “macro credit” risk and a liquid way to adjust a portfolio’s overall credit beta without trading dozens of single-name CDS.

The convenience comes with mechanics that are easy to misbook: indices trade in **series** that **roll**, they have a **fixed coupon plus an upfront** (or a **price quote** for some indices), they carry **accrued premium** conventions, and defaults mechanically shrink the index notional. This chapter makes those moving parts explicit and ties quote → cashflows → PV → risk in a way you can sanity-check.

## Learning Objectives
- Explain what a CDS index is (families, series, on-the-run vs off-the-run).
- Translate an index quote (spread or price) into **settlement cash**: clean upfront plus accrued premium.
- Compute and interpret the risky annuity $A$ and its dollarized form $RPV01_{\mathrm{USD}}$ (currency per bp), and relate these to CS01 with explicit bump object, units, and sign.
- Track how defaults change the outstanding index notional (index factor) and future premium payments.
- Explain “intrinsic” (bottom-up) vs “market” index levels and why a basis can exist (preview).

Prerequisites: [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md), [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 40 — The CDS Auction Process](chapters/chapter_40_cds_auction_process.md)  
Follow-on: [Chapter 42 — Bootstrapping a CDS Survival Curve from Market Quotes](chapters/chapter_42_bootstrapping_cds_survival_curve.md), [Chapter 43 — Risks in CDS and Hedging Strategies](chapters/chapter_43_cds_risks_hedging.md), [Chapter 46 — Intrinsic Index Spread and Index Basis](chapters/chapter_46_intrinsic_index_spread_and_index_basis.md), [Chapter 47 — Hedging and Relative Value in CDS Indices](chapters/chapter_47_hedging_relative_value_cds_indices.md)

**Conventions used in this chapter (important):**
- We avoid “index buyer/seller” ambiguity and instead say **protection buyer** (long protection) / **protection seller** (short protection).
- Notional $N \gt 0$. When we write a cashflow $CF$, it is signed from the protection buyer’s perspective (positive = receive).
- **Quoted clean upfront** $U_{\text{clean}}$: fraction of notional exchanged at settlement; treat it as a *payment amount* where **positive means the protection buyer pays** (so the upfront cashflow to the protection buyer is $-N\\,U_{\text{clean}}$).
- **Accrued premium** is computed on ACT/360 from the previous coupon date to the step-in/effective date; it is typically **paid by the protection buyer** at settlement (cashflow $-N\\,C\\,\Delta_{\text{accr}}$).

---

## 41.1 Index Families and Composition

### 41.1.1 Major Index Families (Examples)

The most widely traded families are **CDX** (North America) and **iTraxx** (Europe / Asia-Pacific). Each family has several sub-indices (investment grade, high yield, crossover, regional).

**Table 41.1: Major CDS Index Families (Illustrative)**

| Index | Region | Number of Names |
|-------|--------|-----------------|
| CDX.NA.IG | North America (investment grade) | 125 |
| CDX.NA.HY | North America (high yield) | 100 |
| CDX.NA.XO | North America (crossover) | 35 |
| CDX.EM | Emerging markets sovereign | 15 |
| iTraxx Europe | Europe (investment grade) | 125 |
| iTraxx Europe Crossover | Europe (crossover) | ~40 |
| iTraxx Japan | Japan (investment grade) | 50 |
| iTraxx Asia ex-Japan | Asia ex-Japan (investment grade) | 50 |
| iTraxx Australia | Australia (investment grade) | 25 |

For the liquid investment-grade (IG) indices, membership is typically refreshed at the semi-annual roll (often **March 20** and **September 20**), and the new series begins trading “on-the-run”.

### 41.1.2 Composition Rules and Selection Criteria

Think in two layers: **within a series** vs **at the roll**.

- **Within a series:** the constituent set is fixed; defaulted names are removed without replacement, and the outstanding notional fraction falls.
- **At the roll:** a new series is defined; names can be replaced for eligibility (e.g., rating) and liquidity reasons.

### 41.1.3 Key Regional Difference: Restructuring Clauses

Index families can differ in what **credit events** trigger protection. A common example is whether **restructuring** is included as a credit event. Contract-term differences matter because they change the expected protection payment, so they can mechanically create an index-vs-constituent “basis” even before liquidity effects.

> **Cross-reference:** Credit event definitions and restructuring are covered in Chapter 39.

### 41.1.4 Equal Weighting

For a standard equal-weight index with $M$ names and index notional $N$, each constituent represents $N/M$ of notional. After $d$ defaults, the remaining notional fraction is $1-d/M$; future coupon payments scale with the outstanding notional.

> **Cross-reference:** Chapter 38 covers single-name CDS mechanics. The premium leg, protection leg, and credit event settlement mechanisms for indices follow the same foundational structure, but applied across a portfolio.

### 41.1.5 LCDX: The Loan CDS Index

Loan CDS indices exist (e.g., **LCDX**, **LevX**) and can embed extra mechanics such as cancellation/prepayment behavior. We do not cover those details here; treat them as different products for curve building and risk.

---

## 41.2 Index Lifecycle: Settlement, Coupons, and Defaults

### 41.2.1 Trade Date and Settlement

Keep three dates straight:

- **Trade date:** when the quote is agreed.
- **Step-in / effective date:** the date used for premium accrual conventions (for “accrued premium” at settlement).
- **Cash settlement date:** when cash (upfront + accrued) actually exchanges.

The exact lags and business-day rules are product/CCP-specific; treat them as conventions and check the confirmation.

**Mechanics intuition (what each date is “for”):** the step-in/effective date is the “as-of” date for the trade’s *economic plumbing* (the stub accrual you settle today and the start point for future premium accrual in your cashflow schedule). The cash settlement date mostly affects *when* money moves (so which discount factor applies to the upfront + accrued exchange), not *how much* accrued is owed.

**Check (accrued scale):** shifting the step-in/effective date by 1 day changes accrued premium by roughly
$N\times C\times (1/360)$.
For $N=USD 100\text{mm}$ and $C=100$ bp, that is about $USD 2{,}778$ per day. If your accrued changes by $USD 27{,}778$ per day, you are likely off by a factor of 10 (bp vs %) or using the wrong day-count denominator.

At the inception of a new series, the fixed coupon is set at issuance. If it is set close to the then-prevailing fair spread, the clean upfront starts near zero.

**Settlement cash (clean vs dirty):** at cash settlement you exchange
- a **clean upfront** $U_{\text{clean}}$ (fraction of notional; **positive means protection buyer pays**), and
- **accrued premium** from the previous coupon date to the step-in/effective date, computed on ACT/360 using the fixed coupon $C$.

Define the accrual fraction $\Delta_{\text{accr}}$ and the accrued-premium fraction $A_{\text{accr}} := C\\,\Delta_{\text{accr}}$. A useful “bond-style” identity is:

$$\boxed{U_{\text{dirty}} = U_{\text{clean}} + A_{\text{accr}} = U_{\text{clean}} + C\\,\Delta_{\text{accr}}}$$

Here $U_{\text{dirty}}$ is the **all-in** fraction of notional paid by the protection buyer at settlement (it can be negative if the protection buyer receives).

### 41.2.2 Coupon Payments

After settlement, the protection buyer pays the **fixed coupon** $C$ quarterly (ACT/360) on the **outstanding** index notional $N_{\text{out},k}$. The premium payment at coupon date $t_k$ is:

$$\boxed{\mathrm{PremiumPayment}_k = N_{\text{out},k} \cdot C \cdot \Delta(t_{k-1}, t_k)}$$

where $\Delta(t_{k-1}, t_k)$ is the ACT/360 accrual fraction.

**Expand (shrinking-notional link):** in an equal-weight index you can think of $N_{\text{out},k}=N\times \text{factor}(t_k)$, where $\text{factor}=1-d/M$ after $d$ defaults (Section 41.2.4). That makes a simple desk sanity check: **one default in a 125-name index reduces future premium cashflows by about 0.8%**.

**Check (toy numbers):** with $N=USD 100\text{mm}$, $C=50$ bp, and a “typical” quarter accrual $\Delta\approx 0.25$, the premium payment is about $USD 100\text{mm}\times 0.0050\times 0.25=USD 125{,}000$. After one default in a 125-name index, multiply by $124/125$ to get $\approx USD 124{,}000$. If your system still shows $USD 125{,}000$, you may be paying premium on defaulted notional.

### 41.2.3 Maturity and IMM Dates

Index series and coupon dates align to standard CDS dates (often called “IMM dates”). A practical consequence is that tenor labels are **approximate**: a “5Y” on-the-run index may start with about 5Y+3m to maturity at issuance and drift toward 5Y−3m by the next roll.

**Table 41.2: Maturity Alignment Example**

| Issue Date | Maturity Label | Actual Maturity | Months at Issuance |
|------------|----------------|-----------------|-------------------|
| March 20, 2024 | 5Y | June 20, 2029 | 63 months |
| September 20, 2024 | 5Y | December 20, 2029 | 63 months |

### 41.2.4 How Defaults Are Handled

When a constituent has a credit event, three mechanics matter operationally:

1. **Protection settlement (that name):** the protection buyer receives (and the protection seller pays) the loss on that name’s slice of index notional, typically determined via the auction cash-settlement process.

2. **Accrued premium to default:** premium accrues up to the default date on that name’s slice; the protection buyer owes that accrued premium to the protection seller.

3. **Index notional factor (“shrinking notional”):** after each default, the index outstanding notional fraction drops by $1/M$, so future premium payments (and remaining jump-to-default exposure) are smaller.

After $d$ defaults, the outstanding notional fraction is:

$$\boxed{\text{Outstanding notional} = N \cdot \left(1 - \frac{d}{M}\right)}$$

> **Cross-reference:** Chapter 40 covers the auction process; Chapter 39 covers credit events.

---

## 41.3 Coupon, Quoted Spread, and Upfront

A CDS index trade is quoted as a **fixed coupon** $C$ plus one additional number that makes the trade fair at today’s market:

- for many investment-grade (IG) indices: a **spread quote** $S$ (in bp), which you convert to a **clean upfront** $U_{\text{clean}}$, or
- for some riskier indices: a **price quote** $P$ (per 100 notional), which directly implies $U_{\text{clean}}$.

Conceptually: the coupon pins the running cashflows; the upfront reconciles the fixed coupon to the current par spread level.

### 41.3.1 The Fixed Coupon Convention

Indices trade with a fixed contractual coupon $C$ set at series launch and unchanged for the life of that series/tenor. Market moves show up in the upfront (or price), not by changing $C$.

### 41.3.2 Converting Between Spread and Upfront

Let $S$ be the market par spread level you would use to price the index (think: a flat spread curve for the index), and define the index **risky annuity** (a survival-weighted premium PV factor):

$$A_I(t,T) := \sum_{k} P(t,t_k)\\,Q(t,t_k)\\,\Delta(t_{k-1},t_k),$$

with units of **years**.

For a position notional $N$, it is often convenient to “dollarize” this into a risky PV01:

$$\boxed{RPV01_{\mathrm{USD}}(t,T) := N \times 10^{-4}\times A_I(t,T)}$$

which has units of **currency per 1 bp**.

A common first-order conversion is:

$$\boxed{U_{\text{clean}} \approx (S - C)\\,A_I(t,T)}$$

where $U_{\text{clean}}$ is a **fraction of notional** (e.g., $0.0030=0.30\\%$).

**Sign (our convention):**
- If $S \gt C$: $U_{\text{clean}} \gt 0$ and the protection buyer **pays** upfront.
- If $S \lt C$: $U_{\text{clean}} \lt 0$ and the protection buyer **receives** upfront.
Equivalently: if the quoted spread is less than the coupon, the protection seller pays this present value to the protection buyer; if the quoted spread is greater than the coupon, the protection buyer pays it to the protection seller.

**Unit check:** $S-C$ is “per year”; $A$ is “years”; the product is a unitless fraction of notional.

**Expand (replication story):** treat the market spread $S$ as the “par running premium” that would make a zero-upfront index CDS fair today. If you instead lock the running premium at the fixed coupon $C$, you have changed the PV of the premium leg by $(S-C)\times A_I$. The clean upfront is the one-time cash exchange that offsets that PV difference so the trade is fair at today’s market.

**Check (limiting cases):**
- If $S=C$, then $U_{\text{clean}}=0$: the fixed coupon happens to be par.
- If $C=0$, then $U_{\text{clean}}\approx S\\,A_I$: you pay (all) the premium-leg PV upfront to buy protection with no running premium.

> **Pitfall — Upfront + coupon regime:** confusing the market spread quote $S$ with the fixed running coupon $C$
> **Why it matters:** it flips the upfront cash direction and can invert your PV/CS01 sign.
> **Quick check:** with fixed $C$, widening spreads should make long protection *more valuable*, so $U_{\text{clean}}=(S-C)A$ should **increase** (protection buyer pays more upfront).

### 41.3.3 The Exact Intrinsic Spread Equation

If you infer survival probabilities from the quoted spread level, then $A_I$ depends on $S$ (because higher spreads imply higher hazard and shorter expected premium payments). In that case the mapping

$$U_{\text{clean}} = (S-C)\\,A_I(S)$$

is nonlinear, and converting **upfront → spread** typically requires a 1D root solve. This matters more when spreads are high.

**Check (why “$A$ fixed” overstates sensitivity at high spreads):** if $U(S)=(S-C)A(S)$, then

$$
\frac{dU}{dS}=A(S) + (S-C)\,A'(S).
$$
Because higher spreads typically imply higher hazard and **shorter** expected premium payment lives, $A'(S)\lt 0$. So using a constant $A$ approximation tends to **overstate** $\Delta U$ for a given $\Delta S$ when spreads are high (a common “why is my DV01 too big?” debugging clue).

### 41.3.4 Price Quotes (Bond-Style) and the “Subtract 100” Convention

Other lower credit quality indices such as the EM and HY indices are quoted using a bond price convention in which the upfront value is computed simply by subtracting 100 from the quoted price:

$$\boxed{U_{\text{clean}}(\\%) = 100 - P}$$

where $U_{\text{clean}}(\\%)$ is the clean upfront in **points** (percent of notional) paid by the protection buyer.

- If $P=97.50$, then $U_{\text{clean}}=+2.50\\%$: the protection buyer **pays** 2.50% upfront.
- If $P=100.27$, then $U_{\text{clean}}=-0.27\\%$: the protection buyer **receives** 0.27% upfront.

**Check (points → dollars):** “points” are percent of notional. So on $N=USD 50\text{mm}$, a 2.50-point upfront is $USD 50\text{mm}\times 2.50\%=USD 1.25\text{mm}$. A 0.01-point move is $USD 50\text{mm}\times 0.01\%=USD 5{,}000$. This is an easy way to sanity-check booked settlement cash on price-quoted indices.

Why quote price? This convention has the advantage that it avoids disagreement about the index PV01 required to convert a spread-based quotation into an upfront value. When spreads are high, disagreements about PV01/scaling (and the nonlinearity in $A(S)$) can become economically material.

### 41.3.6 Accrued Premium Mechanics

Even though you trade between coupon dates, premium accrues daily. At settlement, the premium payer (the protection buyer) owes accrued premium for the stub period:

$$\boxed{\text{Accrued premium amount} = N \cdot C \cdot \Delta_{\text{accr}}}$$

where $\Delta_{\text{accr}}$ is ACT/360 from the previous coupon date to the step-in/effective date.

Combine this with the clean upfront using the identity from Section 41.2.1:
$U_{\text{dirty}} = U_{\text{clean}} + C\\,\Delta_{\text{accr}}$.

---

**Worked Example 41.A: From Spread Quote to Settlement Cash (Clean Upfront + Accrued Premium)**

**Context**
- You are entering a long-protection index position and need the **cash settlement amount** and a first-pass **CS01**.
- On the desk, the most common break is booking clean upfront but forgetting the accrued premium (or flipping the upfront sign).

**Timeline (Make Dates Concrete)**
- Trade date: 2006-02-07
- Step-in/effective date: 2006-02-08
- Cash settlement date: 2006-02-10
- Previous coupon date: 2005-12-20

**Inputs**
- Notional: $N=USD 10{,}000{,}000$
- Fixed coupon: $C=40\text{ bp}=0.0040$
- Quoted index spread: $S=48.375\text{ bp}=0.0048375$
- Day count: ACT/360
- Pricing output (assumed): risky annuity $A_I(t,T)=4.60$ years

**Outputs (What You Produce)**
- Clean upfront fraction: $U_{\text{clean}}=(S-C)\,A=(0.0048375-0.0040)\times 4.60=0.0038525=0.38525\%$.
  Since $S\gt C$, the protection buyer **pays** upfront.
- Accrued premium fraction: $C\,\Delta_{\text{accr}}=0.0040\times (50/360)=0.0005556=0.05556\%$.
- Dirty upfront fraction: $U_{\text{dirty}}=0.38525\\%+0.05556\\%=0.44081\\%.$
- Cash settlement amount (paid by protection buyer): $N\times U_{\text{dirty}}=USD 10{,}000{,}000\times 0.0044081 \approx USD 44{,}081$.
- Dollar risky PV01 (currency per bp): $RPV01_{\mathrm{USD}}=N\times 10^{-4}\times A = USD 4{,}600$ per bp.
- First-order CS01: for long protection, $CS01\approx +RPV01_{\mathrm{USD}}$ under the bump convention in Section 41.6.

**Step-by-step**
1. Convert bp to decimals; compute the spread–coupon difference $S-C$.
2. Multiply by $A$ to get $U_{\text{clean}}$.
3. Compute $\Delta_{\text{accr}}=(50/360)$ and the accrued fraction $C\\Delta_{\text{accr}}$.
4. Add to get $U_{\text{dirty}}$ and multiply by $N$ to get cash settlement.

**Cashflows (table)**
| Date | Cashflow (protection buyer) | Explanation |
|---|---:|---|
| 2006-02-10 | $-USD 38{,}525$ | Clean upfront paid ($N\cdot U_{\text{clean}}$) |
| 2006-02-10 | $-USD 5{,}556$ | Accrued premium paid (ACT/360, 2005-12-20 → 2006-02-08) |

**P&L / Risk Interpretation**
- If you book only the clean upfront and omit accrued premium, your day-1 cash/P&L will be off by about $USD 5.6k$ per $USD 10mm$ notional for this timeline.
- The quoted spread $S$ moving by +1 bp changes PV by roughly $RPV01_{\mathrm{USD}}$ under a local $A$-fixed approximation (sign and bump details in Section 41.6).

**Sanity Checks**
- Units: $(S-C)\\,[1/\text{yr}]\times A\\,[\text{yr}]\to$ unitless fraction of notional.
- Sign: $S\gt C\Rightarrow U_{\text{clean}} \gt 0\Rightarrow$ protection buyer pays.
- Scale: a few bp spread–coupon mismatch times an $A\sim 4$ gives a few tenths of a percent upfront (not tens of percent).

**Debug Checklist (When Your Result Looks Wrong)**
- Did you flip “protection buyer” vs “protection seller” when interpreting the quote?
- Did you convert bp to decimals (1 bp $=10^{-4}$) consistently?
- Did you compute accrued on ACT/360 and use the correct previous coupon date?
- Are you using the **outstanding** index notional (factor-adjusted) for cashflows after defaults?

---

## 41.4 Roll Mechanics and Liquidity Dynamics

### 41.4.1 The Semi-Annual Roll Process

Liquid CDS indices roll every six months. The newest series is **on-the-run**; older series are **off-the-run**. Operationally, “rolling” means closing the old series and opening the new one to stay in the liquid contract.

The roll cadence aligns to the standard CDS date structure:

```
Roll Timeline
=============

Series N issued                 Series N+1 issued             Series N+2 issued
(e.g., March 20)               (e.g., September 20)          (e.g., March 20)
     |                              |                              |
     |<--- Series N on-the-run ---->|<--- Series N+1 on-the-run -->|
     |                              |                              |
     |                    Series N becomes                Series N+1 becomes
     |                    off-the-run                     off-the-run
```

### 41.4.2 P&L Impact from Rolling

Rolling can create P&L even if your credit view is unchanged, mainly due to:

1. **Composition changes:** the new series can replace names for eligibility and liquidity reasons.
2. **Maturity extension:** the new series is about six months longer to maturity, so spreads and risky annuities $A$ can differ even with the same credit curve shape.

**Worked Example 41.B: Net Cash When You Roll Long Protection (Clean Upfront Only)**

Suppose you hold a USD 50 million long protection position in Series 6 that you want to roll into Series 7.

*Series 6 (off-the-run at roll):*
- Coupon: 40 bp
- Current spread: 35 bp
- Risky annuity $A$: 4.05 years

*Series 7 (new on-the-run):*
- Coupon: 45 bp
- Current spread: 38 bp
- Risky annuity $A$: 4.35 years

Compute clean upfronts for **entering long protection** using $U_{\text{clean}}=(S-C)\\,A$:

$$U_6 = (35 - 40) \times 4.05 = -20.25 \text{ bp} = -0.2025\\%$$

$$U_7 = (38 - 45) \times 4.35 = -30.45 \text{ bp} = -0.3045\\%$$

To roll a **long-protection** position, you (i) unwind Series 6 by entering **short protection** (opposite sign), and (ii) enter **long protection** in Series 7. Net cash **paid** by you at the roll is:

$$\text{Net cash paid} = N\\,(U_7 - U_6) = USD 50\text{mm}\times(-0.003045 - (-0.002025)) = -USD 51{,}000.$$

Negative “cash paid” means you **receive** USD 51,000 at the roll (before bid/offer and accrued-premium differences).

### 41.4.3 On-the-Run vs. Off-the-Run Liquidity

Liquidity tends to concentrate in the on-the-run series; off-the-run series can be meaningfully harder to trade and hedge. This is one reason many desks keep macro-credit exposure in on-the-run indices and roll periodically.

### 41.4.4 Hedging Considerations

Hedging an off-the-run series with an on-the-run series creates **series basis risk**:

- constituents differ (mismatch in single-name exposures),
- maturities differ (duration/convexity mismatch),
- and liquidity differs (hedge slippage can be asymmetric in stress).

---

## 41.5 Intrinsic Value and the Index Basis

A useful mental model is “index = basket of $M$ single-name CDS,” each with weight $1/M$ of notional. This lets you build a bottom-up (**intrinsic**) level from constituents and compare it to the market quote (**basis**). We keep this section as a preview; Chapter 46 goes deeper.

### 41.5.1 Intrinsic Value Calculation

Define the intrinsic **clean upfront** fraction for long protection by summing constituent contributions:

$$\boxed{U_{\text{intrinsic}}(t) := \frac{1}{M}\sum_{m=1}^{M}\big(S_m(t,T) - C\big)\cdot A_m(t,T)}$$

This has the same units/sign convention as Section 41.3’s $U_{\text{clean}}$: it is a fraction of notional (positive means the protection buyer pays upfront).

**Worked Example 41.C: Intrinsic Value from Constituents**

Consider a simplified 5-name index with coupon $C = 50$ bp.

| Name | CDS Spread (bp) | Risky annuity $A$ (years) |
|------|-----------------|---------------|
| A | 40 | 4.20 |
| B | 55 | 4.15 |
| C | 45 | 4.25 |
| D | 60 | 4.10 |
| E | 50 | 4.20 |

**Step 1:** Calculate $(S_m - C) \times A_m$ for each name (long protection perspective):
- A: $(40-50) \times 4.20 = -42$ bp (below coupon, unfavorable)
- B: $(55-50) \times 4.15 = +20.75$ bp
- C: $(45-50) \times 4.25 = -21.25$ bp
- D: $(60-50) \times 4.10 = +41$ bp
- E: $(50-50) \times 4.20 = 0$ bp

**Step 2:** Sum and divide by $M = 5$:

$$U_{\text{intrinsic}} = \frac{1}{5}(-42 + 20.75 - 21.25 + 41 + 0) = -0.3 \text{ bp}$$

Interpreting the sign: $-0.3$ bp means the protection buyer would **receive** about $0.3$ bp of notional as clean upfront (small, because the basket is close to “at-coupon” on average).

### 41.5.2 The Intrinsic Spread

If you want an intrinsic *spread level*, a common approximation is the risky-annuity-weighted average of constituent spreads:

$$\boxed{S_{\text{intrinsic}}(t,T) \approx \frac{\sum_{m=1}^{M} S_m(t,T) \cdot A_m(t,T)}{\sum_{m=1}^{M} A_m(t,T)}}$$

This is *not* the arithmetic mean. Intuition: a distressed name has a high spread but also a low $A$ (it is less likely to survive to pay many coupons), so it gets a smaller weight. This typically pulls $S_{\text{intrinsic}}$ below the simple average when spreads are dispersed.

**Check (toy “distressed name” weight):** suppose 4 names trade at 50 bp with $A=4$ years, and 1 distressed name trades at 1000 bp with $A=1$ year.
- Arithmetic average spread $=(4\times 50 + 1000)/5=240$ bp.
- Risky-annuity-weighted intrinsic spread $\approx (4\times 50\times 4 + 1000\times 1)/(4\times 4 + 1) = (800 + 1000)/17 \approx 106$ bp.
The distressed name is “wide” but does not get a full 1/5 weight because it is less likely to pay many coupons.

### 41.5.3 The Index Basis

Define the index basis as:

$$\boxed{\text{Basis}(t,T) := S_{\text{mkt}}(t,T) - S_{\text{intrinsic}}(t,T)}$$

Drivers (mechanism-level):

1. **Contract terms:** differences in credit-event definitions (e.g., restructuring) and other legal terms can change expected protection payments.
2. **Liquidity and funding:** indices can trade more easily than single names; liquidity premia can differ.
3. **Technicals/flows:** indices are common macro hedges, so index spreads can move on flow even if constituents lag.

> **Cross-reference:** Chapter 46 covers intrinsic spread and basis trading in depth.

### 41.5.4 Effect of a Constituent Default on Index Spread

When a constituent defaults, it is removed from the index without replacement. Mechanically, two things happen:

- the index factor drops by $1/M$ (so future premium and jump-to-default exposures shrink), and
- the intrinsic level can change because the defaulted name (often a wide/distressed constituent) no longer contributes to the basket average.

---

## 41.6 Index Risk Measures

### 41.6.1 Index CS01 (Credit Spread DV01)

Be explicit about “what is being bumped”.

We define **CS01** as:

$$\boxed{CS01 := PV(S+1\text{ bp})-PV(S)}$$

where the bump is a **+1 bp** ($=10^{-4}$) parallel shift to the par index spread curve used for valuation (discounting and recovery held fixed; survival is rebuilt consistently with the bumped spread curve). With this convention, **long protection has positive CS01**.

To first order (treating the risky annuity $A$ as locally constant):

$$\boxed{CS01 \approx N \cdot A_I(t,T)\cdot 10^{-4} \\;=\\; RPV01_{\mathrm{USD}}(t,T)}$$

Units: currency per 1 bp for the stated notional.

> **Desk Reality:** risk reports often label the same number “Credit DV01” but with the opposite sign so that **short protection** is positive.
> **Common break:** hedges sized off a report with the opposite sign (or off a price bump when the product is price-quoted).
> **What to check:** confirm the bump object (spread vs price), bump size (1 bp $=10^{-4}$), and whether “DV01” is $CS01$ or $-CS01$.

**Related check (price-quoted indices):** if your index is quoted in “price” $P$ with $U_{\text{clean}}(\\%)=100-P$, then a 0.01-point move in price corresponds to a 0.01% move in clean upfront. As a pure quote-to-cash sanity check, that is $N\times 10^{-4}$ of MTM per 0.01 point, with **long protection moving opposite the price** (price up = tighter spreads = long protection loses).

**Worked Example 41.D: CS01 Calculation**

For a USD 100 million index position with $A=4.25$ years:

$$\text{CS01} \approx 100,000,000 \times 4.25 \times 0.0001 = USD 42,500 \text{ per bp}$$

### 41.6.2 Jump-to-Default Exposure

For an equal-weighted index, a single-name default produces an immediate loss (for protection seller) or gain (for protection buyer) of:

$$\text{JTD exposure per name} = \frac{N(1-R_m)}{M}$$

On a USD 125 million notional with 125 names and 40% recovery, each name's JTD exposure is:

$$USD 125\text{mm} \times 0.60 / 125 = USD 600,000$$

### 41.6.3 Series Basis Risk

Rolling from off-the-run to on-the-run introduces basis risk because:
- constituents differ,
- maturity differs by roughly 6 months, and
- liquidity differs (hedge slippage can be state-dependent).

---

## 41.7 Index Options: The Knockout Problem and Why Black's Model Fails

Index options (options to enter index CDS) are important on many desks, but the payoff is not “just Black on the index spread”. Three index-specific effects matter:

1. **Defaults before expiry (“front-end” effects):** defaults can occur between trade date and option expiry, changing the remaining basket and delivering settlement cashflows.
2. **Index factor changes:** after defaults the outstanding notional fraction is smaller, changing premium and protection scales.
3. **Exercise depends on total PV:** the economically correct exercise decision depends on the full value at expiry, not just whether $S\gt K$.

We do not derive an index-option model here; treat this as a warning that option PV/risk depends on settlement conventions and default modeling choices.

---

## 41.8 The Portfolio Swap Adjustment Algorithm

Tranche models often require a set of constituent curves that price both (i) single-name CDS and (ii) the traded index level. When the index trades away from intrinsic (Section 41.5), one common approach is to apply a simple scaling (e.g., a multiplier on constituent spreads or hazards) so that the basket PV matches the index PV across maturities.

We do not implement portfolio swap adjustment here; see the tranche chapters for calibration plumbing and the index-basis chapters for interpretation.

---

## 41.9 Practical Notes

### 41.9.1 Booking Checklist

Before booking an index trade, verify:

1. **Index family and series:** On-the-run vs. off-the-run
2. **Maturity date:** IMM-aligned; "5Y" is not exactly 5 years
3. **Coupon:** Fixed at series issuance (typically a multiple of 5 bp)
4. **Day count:** Actual/360, quarterly payments
5. **Quoting convention:** Spread (IG) vs. price (HY/EM)
6. **Outstanding notional:** May be reduced by prior defaults
7. **Trade/step-in/settlement dates:** confirm which dates drive accrued premium and when cash actually settles
8. **Direction and terminology:** many screens use “buy/sell” language inconsistently; always map to **protection buyer/seller** before booking cashflows and risk
9. **Risk convention:** confirm bump object (spread vs price) and sign ($CS01$ vs “Credit DV01”)

### 41.9.2 Common Pitfalls

- **Terminology confusion:** “Buy the index” is often used to mean *receive* spread (sell protection). Avoid this; always translate to protection buyer/seller.
- **Spread vs. upfront:** Use a consistent conversion with the correct $A$ / $RPV01_{\mathrm{USD}}$ scaling
- **Series vs. maturity:** These are different concepts—Series 35 5Y is not a 35-year contract
- **Accrued premium:** Include in settlement calculations
- **Notional after defaults:** Premium payments scale with *outstanding* notional, not original notional
- **Roll date mechanics:** Verify IMM dates for your specific series
- **Quoting convention errors:** IG quotes as spread; HY/EM quote as price—mixing these is expensive
- **Cleared vs. bilateral:** Different documentation, margin mechanics, and settlement timing

### 41.9.3 Sanity Checks

- **Premium scaling:** Doubling notional doubles premium payments
- **Default payout bounds:** Loss fraction $\in [0, 1]$, so payout $\in [0, N/M]$
- **Upfront sign:** When spread > coupon, protection buyer **pays** upfront ($U_{\text{clean}} \gt 0$)
- **Intrinsic ≤ average:** For dispersed portfolios, risky-annuity-weighted averages tend to be below the arithmetic mean
- **Basis direction:** Positive basis means index quoted wider than intrinsic

### 41.9.4 P&L Attribution for Index Positions

> **Practitioner Note:** For middle-office professionals transitioning to front office, understanding how to explain daily P&L is essential. Here's the decomposition framework.

**Daily P&L Components for an Index Position:**

1. **Spread P&L:** $\Delta S_I \times \text{CS01}$ — the first-order impact of index spread changes
2. **Carry:** Accrued premium earned (if short protection) or paid (if long protection)
3. **Roll-down:** As time passes, the index "rolls down" the spread curve if curves are upward sloping
4. **Default P&L:** Settlement value received/paid on any constituent defaults
5. **Convexity:** Second-order effect from large spread moves (usually small for indices)
6. **Funding/Collateral:** Margin interest, collateral financing costs

**Worked Example 41.E: P&L Attribution**

Position: USD 100mm short protection on CDX IG (receive 50bp coupon)
Risky annuity $A$: 4.25 years (so $RPV01_{\mathrm{USD}}\approx USD 42{,}500/\text{bp}$)
Horizon: one day (carry + MTM)

| Component | Calculation | Amount |
|-----------|-------------|--------|
| Spread P&L | $\Delta S=+2\text{ bp}$; short protection has $CS01\approx -USD 42{,}500/\text{bp}$ | -USD 85,000 |
| Carry | $USD 100\text{mm} \times 0.0050 \times (1/360)$ | +USD 1,389 |
| Default P&L | Name X defaults, FP=35: $-USD 100\text{mm}/125 \times (1-0.35)$ | -USD 520,000 |
| **Net P&L** | | **-USD 603,611** |

> **How to explain to risk committee:** "We lost $604k. The default in Name X cost $520k immediately (JTD loss). Spreads also widened 2bp, costing another $85k. Partially offset by one day of carry at $1.4k."

---

## 41.10 Post-2009 Market Structure: Big Bang and Small Bang

This section is a reminder that the index-style “fixed coupon + upfront” plumbing is the same **points-upfront** convention used for many CDS contracts.

Mechanically:

- A fixed running coupon is specified; an upfront at initiation makes the fixed-coupon trade fair (present value of the difference between the current market spread and the fixed coupon).
- A running-spread par CDS is the special case “upfront = 0 and coupon = par spread”.

Europe implemented a similar standardization with some regional differences.

Clearing note: standardized index CDS may be centrally cleared. Clearing affects margining and operational cashflows, but details are CCP- and jurisdiction-specific; treat the confirmation and CCP rulebook as the source of truth.

---

## Summary

CDS indices package credit exposure to a standardized basket of reference entities into a single contract. They trade in **series** that roll (semi-annually), and liquidity tends to concentrate in the **on-the-run** series.

An index trade is quoted with a fixed running coupon $C$ plus either a spread quote $S$ or a price quote $P$. A practical conversion for spread-quoted indices is:
$U_{\text{clean}} \approx (S-C)\\,A$,
where $A$ is the risky annuity (years). For price-quoted indices, the clean upfront in points is $U_{\text{clean}}(\\%) = 100-P$. Settlement cash includes both **clean upfront** and **accrued premium**, so it helps to think in “dirty upfront” terms:
$U_{\text{dirty}}=U_{\text{clean}}+C\\,\Delta_{\text{accr}}$.

Defaults are handled name-by-name: accrued premium is settled on the defaulted slice and the index outstanding notional fraction falls by $1/M$ per default (index factor), shrinking future premium and jump-to-default exposures.

For risk, always state the bump methodology. We use a +1 bp bump to the par index spread curve; with that convention, long protection has positive $CS01\approx RPV01_{\mathrm{USD}}=N\times 10^{-4}\times A$. Intrinsic/basis ideas follow by building bottom-up quantities using risky-annuity weights (Chapter 46). The same “points-upfront” mechanics underlie standardized fixed-coupon CDS conventions (Section 41.10).

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Series / roll** | A “vintage” of the index with fixed constituents/coupon; rolls create a new series | Liquidity migrates; positions are often rolled to stay on-the-run |
| **On-the-run / off-the-run** | Newest series vs older series | Affects tradability and hedge slippage |
| **Fixed coupon $C$** | Running premium rate set for the series | Keeps cashflows standardized; PV moves into upfront |
| **Clean upfront $U_{\text{clean}}$** | Fraction of notional quoted ex-accrued; **positive means protection buyer pays** | Determines settlement cash direction and MTM |
| **Dirty upfront $U_{\text{dirty}}$** | All-in fraction at settlement: $U_{\text{clean}}+C\Delta_{\text{accr}}$ | Prevents “forgot accrued” booking breaks |
| **Risky annuity $A$** | Survival-weighted premium PV factor (years) | Turns spread differences into upfront |
| **$RPV01_{\mathrm{USD}}$** | Dollar risky PV01: $N\times 10^{-4}\times A$ (currency/bp) | First-order CS01 scale |
| **CS01** | $PV(S+1\text{ bp})-PV(S)$ under a stated bump design | Hedge sizing; sign conventions differ across systems |
| **Index factor** | Outstanding notional fraction $1-d/M$ after $d$ defaults | Premium and JTD scale with outstanding notional |
| **Intrinsic / basis (preview)** | $S_{\text{intrinsic}}$ via risky-annuity weights; basis $=S_{\text{mkt}}-S_{\text{intrinsic}}$ | Sets up index basis and RV chapters |

---

## Notation

| Symbol | Meaning | Units / sign (this chapter) |
|---|---|---|
| $N$ | Index notional | currency; $N \gt 0$ |
| $M$ | Number of constituents | integer |
| $d$ | Number of defaults so far | integer |
| $C$ | Fixed running coupon | decimal per year; accrual ACT/360 |
| $S$ | Quoted (par) spread level | decimal per year; 1 bp $=10^{-4}$ |
| $P$ | Price quote | price per 100 notional |
| $\Delta_{\text{accr}}$ | Accrual fraction for settlement stub | ACT/360, previous coupon → step-in/effective |
| $U_{\text{clean}}$ | Clean upfront fraction | unitless; **+ means protection buyer pays** |
| $U_{\text{dirty}}$ | All-in settlement fraction | $U_{\text{clean}}+C\Delta_{\text{accr}}$ |
| $A_I(t,T)$ | Risky annuity (index) | years |
| $RPV01_{\mathrm{USD}}$ | Dollar risky PV01 | currency per 1 bp; $=N\times 10^{-4}\times A$ |
| $CS01$ | Spread sensitivity | currency per 1 bp; long protection positive |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CDS index? | A CDS referencing a standardized basket of names (equal-weight slices of notional) |
| 2 | What is a “series”? | A rolled vintage with fixed constituents and a fixed coupon $C$ |
| 3 | What does on-the-run mean? | The most recently issued (most liquid) series |
| 4 | What is the fixed coupon $C$? | The running premium rate you pay/receive quarterly; it does not change for the series |
| 5 | What are two common quoting styles? | Spread quote $S$ (bp) or price quote $P$ (per 100) |
| 6 | What does $U_{\text{clean}} \gt 0$ mean here? | Protection buyer **pays** upfront (cashflow $-N U_{\text{clean}}$) |
| 7 | How do you convert a spread quote to clean upfront (first-order)? | $U_{\text{clean}}\approx (S-C)A$ |
| 8 | How do you convert a price quote to clean upfront? | $U_{\text{clean}}(\\%) = 100-P$ |
| 9 | What is dirty upfront? | $U_{\text{dirty}}=U_{\text{clean}}+C\\Delta_{\text{accr}}$ |
| 10 | How do you compute accrued premium at settlement? | $N\cdot C\cdot \\Delta_{\text{accr}}$ using ACT/360 |
| 11 | What is the risky annuity $A$? | A survival-weighted premium PV factor in years |
| 12 | What is $RPV01_{\mathrm{USD}}$? | Dollar risky PV01: $N\times 10^{-4}\times A$ (currency per bp) |
| 13 | What is CS01 (this chapter)? | $PV(S+1\text{bp})-PV(S)$; long protection positive; $\approx RPV01_{\mathrm{USD}}$ |
| 14 | What happens when a name defaults? | Settle protection on that slice, settle accrued premium to default on that slice, and reduce the index factor |
| 15 | How does the index factor change after $d$ defaults? | Outstanding notional fraction $=1-d/M$ |
| 16 | What is an intrinsic spread approximation? | Risky-annuity-weighted average of constituent spreads |
| 17 | What is the index basis? | $S_{\text{mkt}}-S_{\text{intrinsic}}$ |
| 18 | Why does rolling create P&L? | Composition changes + maturity extension + liquidity differences |
| 19 | What’s a common risk-report convention mismatch? | “Credit DV01” may be $-CS01$ so short protection is positive |

---

## Mini Problem Set

**Problems 1-4: Basic Mechanics**

1. Compute the quarterly premium payment for an index with notional USD 100mm, coupon 35 bp, and accrual fraction 91/360.

2. After 3 defaults in a 125-name index, what is the outstanding notional fraction?

3. If a name defaults with final price FP = 30 (per 100) on an index with USD 125mm notional and 125 names, what is the protection payment?

4. Build a 4-quarter premium schedule for USD 50mm notional, 50 bp coupon, with day counts (90, 91, 92, 90).

**Problems 5-8: Upfront and Spread Conversion**

5. Given coupon 40 bp, quoted spread 52 bp, and risky annuity $A = 4.25$ years, compute the clean upfront as % of notional and state direction.

6. Given coupon 100 bp, clean upfront $U_{\text{clean}} = -2.5\\%$ of notional, and $A = 4.00$ years, compute the implied spread.

7. A bond-price quote of 97.5% implies what upfront payment direction and amount?

8. Explain why solving for spread from upfront requires iteration.

**Problems 9-12: Roll Mechanics**

9. You hold USD 80mm long protection in Series 6 and roll to Series 7. Series 6: spread 38 bp, coupon 40 bp, $A=4.10$. Series 7: spread 42 bp, coupon 45 bp, $A=4.40$. Using clean upfronts only, compute the net cash at the roll (positive = you receive).

10. Why does maturity extension at roll tend to widen the new series spread?

11. Name two risks of hedging off-the-run with on-the-run.

12. Why do index option expiries rarely exceed 6 months?

**Problems 13-16: Intrinsic Value**

13. A 3-name index has constituents with spreads 40, 50, 60 bp and risky annuities $A$ of 4.2, 4.1, 4.0 years. Index coupon is 50 bp. Calculate intrinsic clean upfront (in bp of notional).

14. For the same index, calculate the intrinsic spread using the risky-annuity-weighted formula.

15. Compare the intrinsic spread to the arithmetic average spread (50 bp). Why is the intrinsic lower?

16. If the market quotes this index at 48 bp, what is the basis?

**Problems 17-20: Advanced Topics**

17. (Portfolio swap adjustment) If the intrinsic spread is 35 bp but the index trades at 32 bp, what direction of adjustment ratio $\alpha$ is needed?

18. (Index options) Explain why a payer option on an index can be exercised even when the index spread is below the strike.

19. (Points-upfront) A new IG index trade is done at 85 bp spread with a standardized 100 bp coupon. Approximate the clean upfront using $A=4.2$ years and state direction.

20. (Default impact on spread) A 125-name IG index trades at 50 bp. One name trading at 800 bp defaults. Assuming the remaining 124 names average 44 bp, estimate the post-default intrinsic spread.

---

### Solution Sketches (Selected)

1. $100\text{mm} \times 0.0035 \times (91/360) = USD 88{,}472$

2. Outstanding fraction $=1-3/125=122/125\\approx 97.6\\%.$

3. Per-name notional $=125\text{mm}/125=USD 1\text{mm}$. Loss fraction $=1-0.30=0.70$. Protection payment $=USD 700{,}000$.

4. $\alpha_i=(90, 91, 92, 90)/360$. Pay$_i$ $=50\text{mm} \\times 0.0050 \\times \\alpha_i = (USD 62{,}500, USD 63{,}194, USD 63{,}889, USD 62{,}500)$.

5. $U_{\\text{clean}}=(52-40)\\text{ bp}\\times 4.25=51\\text{ bp}=0.51\\%$. Since $S\gt C$, the protection buyer **pays** 0.51% upfront.

6. $S=C+\frac{U_{\text{clean}}}{A}=0.0100+\frac{-0.025}{4.00}=0.00375=37.5\text{ bp}.$

7. $U_{\text{clean}}=100-97.5=+2.5\\%$. Protection buyer **pays** 2.5% upfront.

8. If you infer survival from $S$, then $A=A(S)$, so $U_{\\text{clean}}=(S-C)A(S)$ requires a 1D root solve for $S$.

9. $U_6=(38-40)\\text{ bp}\\times 4.10=-8.2\\text{ bp}=-0.082\\%$. $U_7=(42-45)\text{ bp}\times 4.40=-13.2\text{ bp}=-0.132\\%$. Net cash received $\approx -N(U_7-U_6)=USD 40{,}000$ (equivalently, cash paid $=N(U_7-U_6)=-USD 40{,}000$).

11. Two common risks: (i) constituent mismatch across series; (ii) maturity/liquidity mismatch (hedge slippage).

19. $U_{\\text{clean}}=(85-100)\\text{ bp}\\times 4.2=-63\\text{ bp}=-0.63\\%$. Since $S\lt C$, the protection buyer **receives** 0.63% upfront.

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (2008), Chapter 10 “CDS Portfolio Indices”.
- John C. Hull, *Risk Management and Financial Institutions*, 4th ed. (2015), “Credit Indices”; “The Use of Fixed Coupons”; “The CDS Market”.
- John C. Hull, *Options, Futures, and Other Derivatives*, 11th ed. (2022), “Credit Default Swaps” (mark-to-market; premium leg PV and accrued premium).
- N. H. Neftci and A. Kosowski, *Principles of Financial Engineering*, 3rd ed., Section 18.3.4.2 “Negative basis trades” (points-upfront; standardized coupons).
- Alexander J. McNeil, Rüdiger Frey, and Paul Embrechts, *Quantitative Risk Management* (2005), Section 9.3.2 (CDS pricing; accrued premium at default).

---
