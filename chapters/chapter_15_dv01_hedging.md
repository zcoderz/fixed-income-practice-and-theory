# Chapter 15: DV01 Hedging — Hedge Ratios, Risk Weights, and Practical Caveats

---

## Introduction

You have calculated your portfolio's DV01. It shows $500,000 per basis point—meaning a mere 10 bp move in rates produces a $5 million P&L swing. The natural question becomes: *how do you neutralize that exposure?*

The answer seems straightforward: find an offsetting instrument and trade it in the right size. But "the right size" depends on a formula that encodes a critical assumption—that rates move together in a specific way. When that assumption fails, so does the hedge. A trader who is "DV01 flat" can still lose substantial money if the curve twists, if basis relationships shift, or if large moves reveal convexity mismatches.

This concept—matching first-order sensitivities to achieve (local) neutrality—is closely related to **immunization** and **duration matching** from traditional fixed-income portfolio management. What began as “match duration to reduce small parallel-shift exposure” has evolved into DV01 and key-rate frameworks used on modern trading desks. The principle is unchanged: specify *what rates you are bumping*, compute sensitivities under that bump, and size the hedge so the first-order PV change is near zero.

Prerequisites: [Chapter 11 — DV01/PV01 — Definitions, Computation, and “What’s Being Bumped”](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12 — Duration — Macaulay, Modified, and the Connection to DV01](chapters/chapter_12_duration_macaulay_modified_dv01.md), [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md)  
Follow-on: [Chapter 16 — Curve Hedging Beyond DV01 — Twists, Butterflies, and PCA](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md), [Chapter 23 — Treasury Futures — CTD, Conversion Factors, Delivery Options, and the Link to Repo](chapters/chapter_23_treasury_futures.md), [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md)

## Learning Objectives
- Translate “DV01” from a quote into a position-level $/bp risk number with correct units.
- Size a DV01 hedge across bonds, swaps (PV01), and futures (CTD + conversion factor), with explicit sign conventions.
- Explain what “DV01-neutral” means (and does *not* mean), including twist, convexity, basis, and spread failure modes.
- Use simple risk weights (regression/min-variance intuition) when a 1:1 DV01 match is not variance-minimizing.
- Diagnose common hedge breaks by checking bump object, units, and the “same-curve” assumption.

This chapter develops the mechanics and limitations of DV01 hedging. We begin with the fundamental hedge ratio and apply it across bonds, swaps, and futures. We then confront the uncomfortable truth: DV01 neutrality is a first-order, local approximation that breaks down in multiple ways. Understanding *when* and *why* DV01 hedges fail is as important as knowing how to construct them.

In this chapter, we cover:

1.  **The fundamental hedge ratio** — deriving the core formula for neutralizing linear price risk
2.  **Instrument-specific mechanics** — how to hedge with swaps (using PV01) and futures (using CTD conversion factors), including the critical CTD switching dynamics
3.  **Risk-weighted hedging** — optimizing hedges for variance minimization rather than simple DV01 matching, and recognizing when historical betas break down
4.  **Failure modes** — why "DV01 flat" portfolios still lose money (twist, convexity, basis, spread risk)
5.  **Credit spread hedging** — using CDS to hedge the spread component that rate hedges cannot address
6.  **The Hedger's Hierarchy** — a practitioner framework for sequencing hedge decisions
7.  **Practical diagnostics** — how to verify hedges and attribute residuals

---

## 15.1 DV01 Revisited: Definition and Units

In the **yield-based** setting (yield-to-maturity \(y\) is the rate factor), a standard definition of DV01 is:

$$\boxed{DV01 = -\frac{1}{10{,}000}\frac{dP(y)}{dy}\;\approx\;-\frac{\Delta P}{10{,}000\,\Delta y}}$$

The negative sign is a convention so that DV01 is **positive** for most fixed-income securities (prices fall as yields rise).

**Book convention (sign + bump size).** In this book, DV01 is defined as the PV change for a **1bp parallel decrease** in the relevant rate factor:

$$\boxed{DV01 := PV(\text{rates down }1\text{bp}) - PV(\text{base})}$$

Here “1bp” means \(10^{-4}\) in rate units. The word “rates” means the **bump object** you choose (a bond yield, a curve node, a forward-rate bucket, etc.). Under this convention, a long fixed-rate bond typically has **positive** DV01: if rates fall by 1bp, PV increases.

For a single fixed-rate bond (priced per 100 of face), yield-based DV01 can be written in duration form as:

$$\boxed{DV01_{\text{per 100}} = \frac{P \times D_{\text{mod}}}{10{,}000}}$$

where \(P\) is the bond price (quoted per 100 of face) and \(D_{\text{mod}}\) is modified duration (in years). In practice, when \(D_{\text{mod}}\) comes from a risk report or an approximation, treat this as a local approximation.

### 15.1.1 Unit Conventions and Traps

DV01 is typically quoted "per 100 face value per 1 bp." This means that for a bond with DV01 = 0.08, a 1 bp yield increase causes the price to fall by approximately $0.08 per $100 of face value. To convert to position-level dollar exposure ($\text{DV01}^{\$}$), one must scale by the position size:

$$\text{DV01}^{\$} = F \times \frac{\text{DV01 (per 100)}}{100}$$

where $F$ is the face value of the position.

**Example:** A $10 million face position in a bond with DV01 = 0.08 per 100:

$$\text{DV01}^{\$} = 10{,}000{,}000 \times \frac{0.08}{100} = \$8{,}000 \text{ per bp}$$

The factor-of-100 unit conversion is a notorious source of errors. Traders who confuse "per 100 face" with "per $1 face" will miscalculate hedge sizes by a factor of 100—a catastrophic mistake in practice.

Bond quotes are typically **clean** (excluding accrued interest), while the **settlement cash amount** uses the dirty price:

$$\text{Settlement Cash} = (P_{\text{clean}}+AI)\times \frac{F}{100}$$

Accrued interest $AI$ is (approximately) rate-insensitive, so DV01 is essentially the same whether you compute it from clean or dirty price—but settlement cash is not.

> **Pitfall — “What is being bumped?” mismatch:** DV01 numbers are only comparable if the **bump object**, **bump size**, **units**, and **sign convention** match.
> **Why it matters:** A hedge ratio computed from mismatched DV01 definitions can be wrong even when the arithmetic is “correct”.
> **Quick check:** Write down (i) what curve/yield is bumped, (ii) $1\text{bp}=10^{-4}$, (iii) whether DV01 is “rates down” or “rates up”, and (iv) whether the number is per 100, per 1mm, or per contract.

> **Desk Reality:** Rates risk is often communicated as “I’m long $X$ DV01”, meaning PV increases by about $X$ for a 1bp rally under the desk’s bump design.
> **Common break:** Mixing “per 100 face”, “per $1mm notional”, and “per contract” without normalization.
> **What to check:** Convert everything to **position-level** $/bp before sizing the hedge.

---

## 15.2 The DV01 Hedge Ratio

### 15.2.1 Derivation

Consider a position in bond A with face value $F_A$ and per-100 DV01 of $\text{DV01}_A$. We wish to hedge this position using bond B with per-100 DV01 of $\text{DV01}_B$. For the combined portfolio to be DV01-neutral, the total dollar sensitivity must be zero:

$$\text{DV01}_A^{\$} + \text{DV01}_B^{\$} = 0$$

Substituting the position-level DV01 expressions:

$$F_A \times \frac{\text{DV01}_A}{100} + F_B \times \frac{\text{DV01}_B}{100} = 0$$

Solving for the hedge face value $F_B$ gives the one-instrument DV01 hedge ratio:

$$\boxed{F_{B}=\frac{-F_{A} \times \text{DV01}_{A}}{\text{DV01}_{B}}}$$

The negative sign indicates that if we are long bond A (positive $F_A$), we typically short bond B (negative $F_B$) to offset rate exposure—assuming both DV01s are positive. If one DV01 is negative (possible for some option-like structures), the same algebra applies but the “hedge” can involve long/long or short/short positions.

### 15.2.2 What "DV01-Neutral" Actually Means

A portfolio is “DV01-neutral” if its first-order PV change is approximately zero under the **specified bump design**. In practice, that usually means “neutral to small parallel shifts in a chosen curve/yield.”

It does *not* protect against:
- **Curve twists:** When short-term and long-term rates move differently.
- **Large moves:** Where convexity dominates the linear DV01 approximation.
- **Basis divergence:** When the hedge and position are driven by different curves (e.g., Treasury vs. Swap).
- **Spread risk:** When credit spreads widen while risk-free rates remain static.

### 15.2.3 The P&L Formula for Imperfect Hedges

Let $\Delta bp_A$ and $\Delta bp_B$ be the realized changes (in **bp**) of the chosen rate factors for A and B. Using the book convention that DV01 is “rates down 1bp” (so PV change for a +1bp move is approximately $-\text{DV01}\times 1$), the hedged portfolio’s first-order P&L is:

$$\Delta PV \approx -\text{DV01}_A^{\$}\,\Delta bp_A - \text{DV01}_B^{\$}\,\Delta bp_B$$

If the hedge is DV01-neutral (i.e., $\text{DV01}_A^{\$}+\text{DV01}_B^{\$}=0$), then:

$$\Delta PV \approx -\text{DV01}_A^{\$}\,(\Delta bp_A-\Delta bp_B)$$

So the hedge works best when $\Delta bp_A\approx \Delta bp_B$ under the chosen bump object; any divergence shows up as residual P&L.

### 15.2.4 Worked Example: Bond-Bond Hedge

**Scenario:** A market maker sells $100 million face value of a call option and must hedge with the underlying bond.

Assume the following per-100 DV01s at the current rate level:
- **Option:** DV01 = 0.0369 per 100 at 5% rates
- **Bond (5s of Feb 15, 2011):** DV01 = 0.0779 per 100 at 5% rates

**Step 1: Compute Required Hedge**

Applying the hedge ratio formula:
$$F_B = 100{,}000{,}000 \times \frac{0.0369}{0.0779} = \$47{,}370{,}000$$

**Step 2: Verify Hedge**

The dollar DV01 of the bond hedge equals:
$$\$47{,}370{,}000 \times \frac{0.0779}{100} = \$36{,}901 \text{ per bp}$$

This matches the option's dollar DV01 of $\$100{,}000{,}000 \times 0.0369/100 = \$36{,}900$ per bp. The market maker is now DV01-neutral for small rate changes.

This hedge is **local**. As rates move, each instrument’s DV01 changes (especially for option-like instruments), so the hedge ratio must be monitored and rebalanced.

### 15.2.5 Worked Example: The Basis Trap in a Stress Scenario

> **Practitioner Note (Stress Scenario):** This example illustrates what happens when the “same yield change” assumption fails across different curves (rates vs. credit spreads).

**Scenario (hypothetical):** A trader held:
- Long $50mm corporate bonds (BBB-rated, 10-year, DV01 = $42,000 per bp)
- Short $50mm Treasury notes (10-year, DV01 = $42,000 per bp)
- Net DV01: Zero

**Stress move (illustrative):**
- Treasury yields fall 50 bp (a rally)
- Corporate spreads widen 200 bp
- Net corporate yields rise 150 bp (spread widening outweighs the Treasury rally)

**P&L Calculation:**
- Treasury hedge: Lost ~$42,000 × 50 = $2,100,000 (rates fell, short position lost)
- Corporate bond: Lost ~$42,000 × 150 = $6,300,000 (yields rose, long position lost)
- **Total Loss: $8,400,000**

Despite being “DV01 neutral,” the trader lost money on *both* legs. In this scenario, the Treasury hedge offsets **rates** but does not offset **spreads**; if rates rally while spreads widen, the rates hedge can add losses while the credit position also loses. (Also note: DV01-based P&L scaling is a local approximation; for 50–150bp moves, convexity and nonlinearity start to matter.)

**Lesson:** A DV01-neutral hedge is only neutral to **parallel moves in the same curve**. When hedging across curves (Treasury vs. corporate), you need a spread hedge (CDS or index) in addition to the rate hedge.

---

## 15.3 Hedging with Swaps and Futures

### 15.3.1 Swap PV01 and Bond-Swap Hedges

In the swap market, a common risk measure is **PV01** (often just called “risk”): the PV change for a 1bp change in the swap’s fixed rate (holding the discounting setup fixed).

For a plain-vanilla swap:

$$\boxed{\text{PV01}_{\text{swap}} = N \times \sum_{i} \tau_i P(0, t_i) \times 0.0001}$$

where $N$ is the notional amount, $\tau_i$ is the accrual fraction for period $i$, and $P(0, t_i)$ is the discount factor. The sum $A = \sum_i \tau_i P(0, t_i)$ is the **annuity factor**.

**Sign convention (payer vs receiver).** To hedge a long bond (positive DV01 under the “rates down” convention), you need a hedge instrument with **negative** DV01: it should lose money when rates fall and gain money when rates rise. A **payer** swap (pay fixed, receive floating) typically has that property.

**Worked Example (template): Sizing a payer-swap hedge**

**Example Title**: Hedge a cash-bond DV01 with a payer swap (PV01 sizing)

**Context**
- You are long a cash bond and want to neutralize first-order rates risk using an IRS, sizing by DV01/PV01.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-17
- Settlement date (bond): 2026-02-19
- Swap effective date: 2026-02-19
- Fixed-leg payment dates (assume annual): 2027-02-19, 2028-02-19, 2029-02-19, 2030-02-19, 2031-02-19
- Accrual start/end: each period runs from the previous payment date to the next (so \(\tau_i \approx 1\) in this toy setup)

**Inputs**
- Bond position:
  - Face value \(F=\$50{,}000{,}000\)
  - Clean price \(P_{\text{clean}}=99.20\) per 100
  - Accrued interest at settlement \(AI=0.80\) per 100
  - Modified duration (given) \(D_{\text{mod}}=4.60\)
- Swap (5Y payer IRS):
  - Discount factors to payment dates (toy): \(P(0,t_i)=\{0.95,0.91,0.87,0.83,0.79\}\)
  - Accrual fractions (toy): \(\tau_i=1\)

**Outputs (What You Produce)**
- Settlement cash (bond): \((P_{\text{clean}}+AI)\times F/100\)
- Position DV01 (bond): currency per 1bp
- PV01 per \(\$1\)mm notional (swap): currency per 1bp per \(\$1\)mm
- Hedge notional \(N\) (swap): \(\$ \) notional to pay fixed

**Step-by-step**
1. Translate quote to settlement cash:
   - Dirty price \(P_{\text{dirty}}=P_{\text{clean}}+AI=100.00\).
   - Settlement cash \(=100.00\times \$50{,}000{,}000/100=\$50{,}000{,}000\).
2. Compute bond DV01 (yield-based approximation, per 100):
   - \(DV01_{\text{per 100}}\approx P_{\text{clean}}\times D_{\text{mod}}/10{,}000 = 99.20\times 4.60/10{,}000=0.0456\) (dollars per 100 per bp).
   - Position DV01 \(=F\times DV01_{\text{per 100}}/100 = \$50{,}000{,}000\times 0.0456/100=\$22{,}800\) per bp.
3. Compute swap PV01 per \(\$1\)mm:
   - Annuity \(A=\sum_i \tau_i P(0,t_i)=0.95+0.91+0.87+0.83+0.79=4.35\).
   - PV01 per \(\$1\)mm \(= \$1{,}000{,}000\times A\times 10^{-4}=\$435\) per bp.
4. Size hedge notional:
   - \(N \approx \$22{,}800 / (\$435 \text{ per bp per } \$1\text{mm}) \approx 52.4\text{mm}\).
   - Action: pay fixed on \(\$52.5\)mm notional (rounded).

**Cashflows (table)**
The PV01 is the PV of the incremental fixed-leg cashflows from a 1bp change in fixed rate:

| Date | Cashflow | Explanation |
|---|---|---|
| 2027-02-19 | \(N\times \tau_1 \times 10^{-4}\) | extra fixed paid for a +1bp fixed-rate bump |
| 2028-02-19 | \(N\times \tau_2 \times 10^{-4}\) | same |
| 2029-02-19 | \(N\times \tau_3 \times 10^{-4}\) | same |
| 2030-02-19 | \(N\times \tau_4 \times 10^{-4}\) | same |
| 2031-02-19 | \(N\times \tau_5 \times 10^{-4}\) | same |

**P&L / Risk Interpretation**
- If rates rise, the bond PV falls (negative P&L) while the payer swap tends to gain (positive P&L), so the hedge offsets first-order rate moves.
- If the bond is priced off a curve/spread that does not move one-for-one with the swap curve, the residual is **basis/spread risk**.

**Sanity Checks**
- Units check: bond DV01 is per 100 face; swap PV01 is per \(\$1\)mm notional. Convert both to position-level \(\$/\text{bp}\).
- Sign check: long bond has positive DV01 under “rates down”; a payer swap should contribute negative DV01 to offset it.

### 15.3.2 Futures Hedge Ratio Basics

Futures hedging introduces an extra layer of complexity: the **Conversion Factor (CF)** and the **Cheapest-to-Deliver (CTD)** bond. Since Treasury futures track a virtual "6% coupon bond" (or similar standard), physical bonds are deliverable through a conversion factor system.

A common **duration-based hedge ratio** for futures is:

$$\boxed{N^* = \frac{P \times D_P}{V_F \times D_F}}$$

where $P$ is the portfolio value, $D_P$ is its duration, $V_F$ is the futures contract value, and $D_F$ is the duration of the bond underlying the futures (typically the CTD).

In practice, a common desk approximation is to use the DV01 of the CTD bond directly:

$$\boxed{\text{DV01}_{\text{fut}} \approx \frac{\text{DV01}_{\text{CTD}}}{\text{CF}}}$$

This implies that one futures contract acts like $1/\text{CF}$ units of the CTD bond in terms of duration exposure.

**Worked Example:** Hedge a $25,000 position DV01 with Treasury futures.

*   **CTD Stats:** DV01 = 0.0750 per 100; CF = 0.90.
*   **Contract Size:** $100,000 face.
*   **CTD Risk per Contract:** $100{,}000 \times (0.0750/100) = \$75.00$.
*   **Futures Risk per Contract:** $75.00 / 0.90 = \$83.33$.
*   **Hedge Size:** $25{,}000 / 83.33 \approx 300$ contracts (Short).

### 15.3.3 CTD Switching and the Quality Option

The CTD bond can change as the rate environment changes. This creates a discontinuity in the hedge relationship that simple linear formulas cannot capture.

**Why Conversion Factors Are Imperfect:** Conversion factors are calculated at a notional yield (often 6% for U.S. Treasury bond futures). They adjust for coupon differences most cleanly at that yield; away from it, deliverables are no longer economically equivalent, and CTD can switch as yields move.

The key insight involves the **price ratio-yield relationship**. For each deliverable bond $i$, define the price ratio:

$$\text{Price Ratio}_i = \frac{P_i}{\text{CF}_i}$$

At the notional yield, the conversion-factor system is designed so deliverables have roughly comparable economics. Away from that yield, relative value shifts:

- **Above 6% yield:** The high-duration bond becomes CTD (its price falls fastest, making it cheapest relative to its conversion factor credit)
- **Below 6% yield:** The low-duration bond becomes CTD (its price rises slowest)

**The CTD Switch Threshold:** Consider two deliverable bonds:
- Bond A: 4.75s of November 15, 2008 (low coupon, low duration)
- Bond B: 5s of August 15, 2011 (higher coupon, higher duration)

At exactly 6%, both have zero cost of delivery. Above 6%, Bond B (higher duration) is CTD. Below 6%, Bond A (lower duration) is CTD. The switch occurs at a specific yield threshold.

> **Desk Reality: The CTD Switch Trap**
>
> **The Problem:** Your hedge ratio depends on the CTD's duration. When the CTD switches, your hedge becomes mismatched *overnight*—without any trading.
>
> **Example:** You're short 300 futures contracts hedging a $25,000 DV01 position. The CTD is the 5s (DV01 = 0.0750, CF = 0.90), giving futures DV01 = $83.33 per contract.
>
> **Overnight:** Yields fall 15 bp, crossing the CTD switch threshold. The new CTD is the 4.75s (DV01 = 0.0680, CF = 0.88).
>
> **New futures DV01 (per contract):** \(0.0680/0.88=0.07727\) per 100 \(\Rightarrow \$100{,}000\times(0.07727/100)=\$77.27\) per bp.
>
> **Your hedge DV01 is now:** \(-300\times \$77.27=-\$23{,}181\) per bp
>
> **Your position is still:** $25,000 DV01 (long)
>
> **Mismatch:** You're now $1,819 DV01 under-hedged. A further 10 bp sell-off (yields +10bp) costs you about $18,190 in first-order P&L.
>
> **The fix:** Monitor the CTD and rebalance around switch thresholds. The short’s ability to choose which bond to deliver is an embedded option-like feature (often called the **quality option**).

### 15.3.4 Tailing the Hedge

Because futures are marked-to-market daily, gains and losses are realized through daily cashflows (variation margin). This means a futures hedge is closer to a **sequence of short-horizon hedges** than to a single forward-style hedge.

In practice, desks often apply a **tailed hedge**: scale the hedge by an approximate discounting factor. Under a simple-interest approximation with ACT/360, a common rule-of-thumb is:

$$N^*_{\text{tailed}} \approx N^* \times \left(1-r\,\frac{d}{360}\right)$$

where \(r\) is a relevant short rate and \(d\) is the number of days to hedge maturity.

**Example:** For a 6-month hedge at 5% rates:
$$\text{Tailing factor} \approx 1-0.05\times \frac{180}{360}=0.975$$

A 300-contract hedge becomes 293 contracts after tailing. For short horizons the difference is often small; for longer horizons it can be noticeable.

### 15.3.5 Stacking vs. Stripping

When hedging long-dated exposures with futures, traders face a choice:

**Stacking:** Concentrate the hedge in the nearest liquid contract and roll forward.
- *Pro:* Maximum liquidity, tight bid-ask
- *Con:* Roll risk (cost of rolling), curve mismatch risk

**Stripping:** Match hedge tenors to exposure tenors using multiple contracts.
- *Pro:* Better tenor match, reduced roll risk
- *Con:* Less liquidity in back months, more contracts to manage

In practice, hedgers often blend the two: use liquid front contracts for execution efficiency while adding some tenor matching when roll risk or curve-shape mismatch dominates.

---

## 15.4 Risk-Weighted Hedging and Variance Minimization

### 15.4.1 Beyond Simple DV01 Neutrality

The standard DV01 hedge ratio ($h=\text{DV01}_{\text{target}}/\text{DV01}_{\text{hedge}}$) assumes the target and hedge rate factors move one-for-one. In reality, they are imperfectly correlated and have different volatilities, so a 1:1 DV01 match does not necessarily minimize variance.

A common approach is **regression-based hedging**. If you assume the relationship between target and hedge changes is approximately linear, you can write:

$$\Delta y_{\text{target}} = \alpha + \beta \Delta y_{\text{hedge}} + \epsilon$$

This is the same structure used in minimum-variance futures hedging: regress changes in the exposure on changes in the hedge, and interpret the **slope** as the hedge ratio.

In this minimum-variance view, the optimal hedge ratio is the **slope** of the best-fit regression line. If your P&L is approximately linear in yield changes via DV01, a practical sizing rule is: take the simple DV01 hedge ratio and scale it by \(\beta\):

$$\boxed{h^* = \beta \times \frac{\text{DV01}_{\text{target}}}{\text{DV01}_{\text{hedge}}}}$$

### 15.4.2 The Regression Beta Decomposition

The regression coefficient can be expressed as correlation times a volatility ratio:

$$\boxed{\beta = \frac{\rho \sigma_{\text{target}}}{\sigma_{\text{hedge}}}}$$

where \(\rho\) is correlation and \(\sigma\) denotes the standard deviation of the corresponding changes (in the same units, e.g., bp). A **volatility-weighted** hedge uses only the volatility ratio; the **regression-based** hedge multiplies that ratio by \(\rho\) to reflect imperfect correlation.

> **Desk Reality:** Regression hedges are only as good as the window and regime used to estimate $\beta$.
> **Common break:** Betas drift (policy shifts, liquidity events, supply/demand) and the hedge becomes systematically over- or under-sized.
> **What to check:** Re-estimate betas on multiple windows; stress the hedge under plausible alternative $\rho$ and volatility ratios.

### 15.4.3 Two-Bond Hedges and Risk Weights

With more than one hedge instrument, regress the target change on multiple hedge changes:

$$\Delta y_{20} = \alpha + \beta_{10} \Delta y_{10} + \beta_{30} \Delta y_{30} + \epsilon$$

The coefficients $\beta_{10},\beta_{30}$ act like **risk weights**: they describe how the target yield tends to move when the hedge yields move. A practical sizing heuristic is:
- compute the target position DV01 (in $/bp$),
- allocate that DV01 across hedges using the regression weights,
- convert each allocated DV01 into a hedge notional using each hedge instrument’s DV01 per unit.

### 15.4.4 Worked Example: Regression-Based Hedge

**Setup:** Hedge a $10 million face position in 20-year bonds using 30-year bonds.

Suppose you estimate (per 100 face):
- **20-year DV01:** 0.118428
- **30-year DV01:** 0.142940
- **Regression beta:** 1.057 (20-year yield moves about 1.057× 30-year yield, in-sample)

**Simple DV01 Hedge:**
$$F_{30} = -10{,}000{,}000 \times \frac{0.118428}{0.142940} = -\$8{,}286{,}000$$

**Regression-Adjusted Hedge:**
$$F_{30} = -10{,}000{,}000 \times 1.057 \times \frac{0.118428}{0.142940} = -\$8{,}758{,}000$$

The regression approach requires shorting an additional $472,000 face of 30-year bonds because historical data shows 20-year yields are slightly more volatile than 30-year yields.

**Residual risk:** Even with regression sizing, the residual $\epsilon$ is not zero. You are reducing variance, not eliminating risk.

### 15.4.5 When Betas Break: Regime Change Risk

Historical betas can become misleading when market dynamics shift. Reasons include policy regime changes, liquidity events, and shifts in the relative volatility of different curve points.

> **Practitioner Note (Regime Change):**
>
> Regression hedging is not “set and forget.” If the volatility/correlation regime changes, betas estimated on one sample can become actively harmful.
>
> Illustrative example: you size a hedge using a historical beta of 1.2, but the realized beta shifts to 0.8. You are now over-hedged; instead of reducing variance, the hedge can amplify P&L swings until you recalibrate.

**Diagnostic: Sensitivity Analysis**

Before relying on a regression hedge, stress-test the parameters:

| Parameter Change | Impact on Hedge |
|------------------|-----------------|
| Beta 1.05 → 0.90 | Hedge size drops ~14% |
| Correlation 0.99 → 0.90 | Beta drops ~9% (if vol ratio unchanged) |
| Vol ratio 1.1 → 0.9 | Beta drops ~18% |

If small parameter changes materially affect your hedge, you have **parameter risk** that must be monitored.

---

## 15.5 When DV01 Hedges Fail

DV01 hedging is a first-order local approximation. It works best for small, specified shifts in the chosen bump object. The primary failure modes are:

### 15.5.1 Curve Twist (Non-Parallel Moves)

DV01 aggregates risk into a single number, masking exposure to curve shape. A portfolio long 30-year bonds and short 2-year bonds could have zero net DV01. However, if the curve steepens (30y yields rise, 2y yields fall), both positions lose money.

This is the motivation for **key-rate DV01** and bucketed hedging (Chapter 14): instead of one scalar, you hedge exposure to *shape* changes by neutralizing multiple curve points.

### 15.5.2 Convexity Mismatch

DV01 is linear. Convexity is quadratic. The Taylor series expansion shows the P&L impact:

$$\frac{\Delta P}{P} \approx -D \Delta y + \frac{1}{2} C (\Delta y)^2$$

Option-like instruments (callables, mortgages, IO/PO strips) can have DV01 and convexity that change rapidly with the level of rates. A DV01-neutral hedge can therefore drift quickly and require frequent rebalancing.

### 15.5.3 Basis Risk

Basis risk arises when the hedge instrument and the hedged position are driven by different curves. While they may be correlated, they are not identical. Common disconnects include:

*   **Treasury vs. Swap:** Hedging a corporate bond (priced off swaps) with Treasuries exposes you to swap spread changes.
*   **Discount vs. Projection:** In multi-curve frameworks, a hedge might match the projection curve risk (e.g., term SOFR) but leave discount curve risk (OIS) open.
*   **On-the-Run vs. Off-the-Run:** Liquidity premiums can cause On-the-Run Treasuries to yield significantly less than Off-the-Run counterparts. A hedge across this boundary carries "liquidity risk" disguised as basis risk.

In short: a DV01 hedge is only “clean” if the hedge and the position are driven by the **same curve** under the same bump definition.

### 15.5.4 Spread and Credit Risk

Rates hedges do not neutralize **credit spread risk**. Define CS01 in the same style as DV01: the PV change for a 1bp tightening of a specified spread object (so it is typically positive for a long corporate bond):

$$\boxed{CS01 := PV(\text{spreads down }1\text{bp}) - PV(\text{base})}$$

As with DV01, you must state:
- **bump object** (OAS, Z-spread, asset-swap spread, etc.),
- **bump size** (1bp $=10^{-4}$),
- **units** (currency per 1bp, per 100 or per notional),
- **sign convention**.

For intuition, many toy calculations treat a bond yield as “risk-free curve + additive spread” and bump the spread like an additive yield shift. Under that simplifying assumption, CS01 magnitude is often similar to DV01 magnitude for fixed cashflows—but in real pricing setups (curve choices, spread definitions, optionality) they can differ materially.

### 15.5.5 Hedging Credit Spread Risk with CDS

To hedge spread risk, you need an instrument that *gains when spreads widen*. A standard tool is a **credit default swap (CDS)**: buying protection has positive PV exposure to spread widening.

For hedging, it is useful to work with **risky PV01** (often written RPV01):
- \(RPV01\) is the present value of paying **1bp per year** of running premium until maturity or default (under discounting and survival).
- It is typically quoted as a positive number “per \(\$1\)mm notional per 1bp”.

If you define bond CS01 as \(CS01 := PV(\text{spreads down }1\text{bp})-PV(\text{base})\), then (to first order) a protection buyer has \(CS01_{\text{CDS}} \approx -RPV01\). A simple sizing rule is therefore:

$$\boxed{N_{\text{CDS}} \approx \frac{CS01_{\text{bond}}}{RPV01_{\text{CDS}}}}$$

This section is only a preview; full CDS mechanics, pricing, and risk measures appear in Chapters 38–43.

---

## 15.6 Cross-Currency Hedging: A Preview

When hedging a non-USD bond with USD instruments, you introduce **FX risk** on top of rate risk. This creates a multi-dimensional hedging problem.

**The Risk Layer Cake:**

| Risk Layer | Exposure | Hedge Instrument |
|------------|----------|------------------|
| Foreign rate risk | EUR bond price moves with EUR rates | EUR IRS or Bund futures |
| Domestic rate risk | USD valuation of EUR flows | USD IRS or Treasury futures |
| FX spot risk | EUR/USD exchange rate | FX forward or spot |
| Cross-currency basis | EUR-USD basis spread | Cross-currency swap |

**The Basic Workflow:**

1. **Hedge foreign rate risk:** Pay fixed on EUR swap matching the bond's EUR DV01
2. **Hedge FX exposure:** Sell EUR forward to convert future EUR cash flows to USD
3. **Monitor basis:** Cross-currency basis creates a "cost" of hedging that varies over time

> **Practitioner Note:** Full treatment of cross-currency hedging appears in Chapters 29-31. The key insight here is that DV01-neutral in one currency does *not* make you DV01-neutral from the perspective of another currency's P&L—you must account for both the foreign rate sensitivity and the exchange rate sensitivity of your foreign-currency position.

---

## 15.7 The Hedger's Hierarchy

Practitioners often think of hedging as a **sequence of decisions**, each removing a layer of risk while leaving residuals that may or may not be worth hedging further.

> **Practitioner Note (Synthesis):** This hierarchy is a practical way to sequence hedge decisions: remove the biggest first-order risk first, then decide which residuals are worth paying to hedge.

**Level 1: Neutralize DV01 (Parallel Shift Protection)**
- **Risk addressed:** First-order rate sensitivity
- **Instruments:** Treasuries, IRS, futures
- **Residual:** Curve twist, convexity, basis, spread

**Level 2: Neutralize Key Rates (Twist Protection)**
- **Risk addressed:** Non-parallel curve movements (steepening/flattening)
- **Instruments:** Maturity-matched hedges (2y, 5y, 10y, 30y)
- **Residual:** Curvature, convexity, basis, spread

**Level 3: Neutralize Convexity (Gamma Protection)**
- **Risk addressed:** Large move risk, callable/putable optionality
- **Instruments:** Options, swaptions, convexity matching
- **Residual:** Basis, spread, vega

**Level 4: Neutralize Spread Risk (CS01 Protection)**
- **Risk addressed:** Credit spread movements
- **Instruments:** CDS, credit indices
- **Residual:** Basis risk (bond-CDS basis)

**Level 5: Manage Basis Risk (Imperfect Correlation Residual)**
- **Risk addressed:** Tracking error between hedge and position
- **Instruments:** Tighter instrument matching, regression adjustment
- **Residual:** Model risk, liquidity risk

**Level 6: Accept Tail Risk**
- **Reality:** Some risks cannot be economically hedged
- **Approach:** Position limits, stress testing, scenario analysis

> **Desk Reality: Where Most Desks Stop**
>
> The cost-benefit of hedging diminishes at each level:
>
> | Level | Typical Coverage | Why Desks Stop |
> |-------|------------------|----------------|
> | 1-2 | Almost universal | Core risk management |
> | 3 | Selective (option desks, mortgage) | Convexity hedges are expensive |
> | 4 | Credit desks; some multi-asset | CDS has bid-ask and counterparty risk |
> | 5-6 | Rarely fully hedged | Cost exceeds benefit |
>
> **The practical rule:** Hedge the risks you're not being paid to take. A rates trader hedges rates (Level 1-2) but may accept spread risk. A credit trader hedges spread (Level 4) but may accept some basis risk.

---

## 15.8 Additional Worked Examples

### Example A: DV01-Neutral but Twist Loss

**Setup:**
*   **Portfolio:** Long 2y (Risk = \$2k) and Long 10y (Risk = \$8k). Total Risk = \$10k.
*   **Hedge:** Short a 5y bond (Risk = \$8k per unit).
*   **Hedge Ratio:** Need \$10k total protection. $10/8 = 1.25$ units.
*   **Net Position:** Flat DV01 ($10k - 1.25 \times 8k = 0$).

**Shock:** 2y yields +10bp, 10y yields -5bp (Curve flattens/twists). 5y yields unchanged.
*   **2y P&L:** $-\$2{,}000 \times (+10) = -\$20{,}000$
*   **10y P&L:** $-\$8{,}000 \times (-5) = +\$40{,}000$
*   **Hedge P&L:** $0$ (since 5y didn't move)
*   **Net P&L:** $+\$20{,}000$

**Conclusion:** The portfolio made \$20,000 purely from the curve twist, despite being DV01 hedged. Note that the result could easily have been a loss if the twist went the other way.

### Example B: Convexity Mismatch Under Large Move

**Setup:**
*   **Long:** High Convexity bond (C=120)
*   **Short:** Low Convexity bond (C=60)
*   **Constraint:** Both have Duration=8 and Price=100.
*   **DV01:** Matched (0.08 per 100).

**Shock:** Yields rise +100 bp (1.0%).
*   **Long Price Change:** $-8(0.01) + 0.5(120)(0.01)^2 = -0.08 + 0.006 = -0.074$
*   **Short Price Change:** $-8(0.01) + 0.5(60)(0.01)^2 = -0.08 + 0.003 = -0.077$
*   **Net P&L:** Long loses 7.4 pts, Short gains 7.7 pts. **Net Profit +0.3 pts.**

**Shock:** Yields fall -100 bp (-1.0%).
*   **Long Price Change:** $-8(-0.01) + 0.5(120)(-0.01)^2 = +0.08 + 0.006 = +0.086$
*   **Short Price Change:** $-8(-0.01) + 0.5(60)(-0.01)^2 = +0.08 + 0.003 = +0.083$
*   **Net P&L:** Long gains 8.6 pts, Short loses 8.3 pts. **Net Profit +0.3 pts.**

**Conclusion:** Being "long convexity" (Long C=120, Short C=60) generates profit in large moves *regardless of direction*.

### Example C: CTD Switch Impact

**Setup:**
- Hedge: Short 300 TY futures contracts
- Example CTD candidate A: DV01 = 0.0780, CF = 0.92
- Example CTD candidate B: DV01 = 0.0710, CF = 0.88

**Hedge effectiveness before the CTD switch:**
- Futures DV01 = 0.0780 / 0.92 = 0.0848 per 100
- Per contract: $100,000 × 0.0848/100 = $84.78
- Total hedge: 300 × $84.78 = $25,434 DV01

**After CTD switch (yields fall below threshold):**
- New CTD: Candidate B
- Futures DV01 = 0.0710 / 0.88 = 0.0807 per 100
- Per contract: $100,000 × 0.0807/100 = $80.68
- Total hedge: 300 × $80.68 = $24,205 DV01

**Impact:** Hedge shrinks by $1,229 DV01 (~5%) without any trading. If the position being hedged is still $25,434 DV01, you're now under-hedged by $1,229 per bp.

---

## 15.9 Practical Notes

### Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Sign Convention Confusion** | State your convention explicitly. This book uses DV01 > 0 for long bonds. Ensure your hedge (short bond or payer swap) has negative DV01. |
| **Unit Mistakes** | "Per 100 face" vs "per $1 face" is a factor of 100. Always check if the DV01 is ~0.08 (per 100) or ~0.0008 (per 1). |
| **Mixing Methodologies** | Bumping par rates vs. shifting zero curves can produce slightly different DV01s. Stick to one consistency (e.g., "bump-and-rebuild"). |
| **Ignoring Curve Shape** | DV01-neutral is not risk-free. Always check key-rate exposures (Chapter 14) for non-parallel risk. |
| **Stale Regression Betas** | Using a beta from the wrong regime. Recalibrate or revert to simple DV01 matching during transitions. |
| **CTD Switch Blindness** | Not monitoring yield levels relative to CTD thresholds. Futures DV01 can change discontinuously. |
| **Liquidity Mismatch** | Hedging an illiquid position with a liquid instrument creates "gap risk"—you may be hedged on paper but unable to monetize the position's gain if the hedge moves against you. |

### Verification Tests

Practitioners should run specific "stress tests" to validate hedges:
1.  **Parallel Shift:** Verify Net P&L $\approx 0$ for a small (+1 bp) parallel bump.
2.  **Twist Scenario:** Apply a steepening (+10 bp long end, -10 bp short end) to reveal bucket mismatches.
3.  **Large Move:** Apply a +/- 50 bp shock to verify convexity exposure isn't catastrophic.
4.  **Basis Shock:** Move the hedge curve (e.g., Swap) by +5 bp while holding the asset curve (e.g., Treasury) constant to check basis sensitivity.
5.  **CTD Monitor:** Check yield vs. CTD switch thresholds; stress-test around those levels.
6.  **Liquidity Stress:** Can you unwind both legs of the hedge in a crisis? At what cost?

---

## Summary

1. **DV01** is a first-order PV sensitivity per 1bp under a *specified bump design*. This book uses \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\).
2. Always convert DV01 into **position-level \(\$/\text{bp}\)** and normalize units (per 100 face vs per \(\$1\)mm vs per contract).
3. The one-instrument DV01 hedge ratio is \(F_{B}=-F_{A}\,(\text{DV01}_A/\text{DV01}_B)\) when DV01s are defined on the same bump object.
4. “DV01-neutral” mainly means neutral to **small parallel shifts**; twists require key-rate/bucket hedging (Chapter 14).
5. Swaps are commonly hedged with **PV01**: \(PV01=N\sum_i \tau_i P(0,t_i)\times 10^{-4}\); payer vs receiver determines sign.
6. Futures hedging requires **CTD + conversion factors**; futures DV01 can jump when CTD switches; tailing adjusts for daily settlement.
7. Regression/min-variance sizing scales the DV01 hedge by a **beta**: \(h^*=\beta\,(\text{DV01}_{\text{target}}/\text{DV01}_{\text{hedge}})\), with \(\beta=\rho\sigma_{\text{target}}/\sigma_{\text{hedge}}\).
8. Key failure modes: convexity mismatch, basis (different curves), and spread risk (CS01).
9. A CDS can hedge spread risk; a common sizing rule is \(N_{\text{CDS}}\approx CS01_{\text{bond}}/RPV01_{\text{CDS}}\).
10. Desk hygiene: write down bump object, bump size, units, and sign before trusting “DV01-flat”.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| **DV01** | \(PV(\text{rates down }1\text{bp})-PV(\text{base})\) for a stated bump object | Makes sign/units explicit and comparable across positions. |
| **Hedge Ratio** | \(F_B = -F_A (\text{DV01}_A/\text{DV01}_B)\) | Basic sizing rule for a one-instrument DV01 hedge. |
| **Immunization** | Constructing a portfolio to reduce sensitivity to a specified rate shift | Explains why DV01 hedges are local and model-dependent. |
| **PV01** | Swap PV change per 1bp in fixed rate (annuity \(\times\) notional \(\times 10^{-4}\)) | Core sizing metric for bond–swap hedges. |
| **CTD** | Cheapest-to-deliver bond for futures | Determines futures DV01; can switch as yields move. |
| **Conversion Factor** | Futures delivery adjustment for coupon differences | Makes deliverables comparable at a notional yield; imperfect away from it. |
| **Risk Weight** | Regression coefficient \(\beta\) | Scales hedges for relative volatility/correlation. |
| **Basis Risk** | Hedge and position do not move one-for-one | Residual P&L even if DV01 sums to zero. |
| **CS01** | \(PV(\text{spreads down }1\text{bp})-PV(\text{base})\) for a stated spread object | Rates hedges do not neutralize spread risk. |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(DV01\) | \(PV(\text{rates down }1\text{bp})-PV(\text{base})\) | currency per 1bp; bump object must be stated |
| \(DV01^{\$}\) | Position-level DV01 | currency per 1bp (whole position) |
| \(D_{\text{mod}}\) | Modified duration | years; used in \(DV01_{\text{per 100}}\approx P D_{\text{mod}}/10{,}000\) |
| \(F\) | Face value / notional | currency |
| \(PV01\) | Swap PV sensitivity to 1bp in fixed rate | currency per 1bp; often quoted per \(\$1\)mm notional |
| \(CF\) | Conversion factor | unitless |
| \(CTD\) | Cheapest-to-deliver | bond/issue identifier |
| \(\beta\) | Regression slope (risk weight) | unitless |
| \(\rho\) | Correlation | unitless |
| \(\sigma\) | Volatility of rate-factor changes | bp per time unit (state horizon) |
| \(CS01\) | \(PV(\text{spreads down }1\text{bp})-PV(\text{base})\) | currency per 1bp; spread object must be stated |

---

## Flashcards

| # | Question | Answer |
|---|---|---|
| 1 | What is DV01 under this book’s convention? | \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\) for a stated bump object. |
| 2 | How do you convert DV01 “per 100” to position risk? | Multiply by (Position Face / 100). |
| 3 | What is the one-instrument DV01 hedge ratio? | \(F_B = -F_A (\text{DV01}_A/\text{DV01}_B)\), using consistent bump object and units. |
| 4 | If you own a bond (positive DV01), do you buy or sell futures to hedge? | Sell futures (short position gains when rates rise). |
| 5 | What does a payer swap do in a hedge? | It tends to gain when rates rise (negative DV01 under “rates down”), offsetting a long bond’s rate risk. |
| 6 | Why might a hedge ratio differ from the simple DV01 ratio? | Because regression/min-variance sizing multiplies the DV01 ratio by a beta. |
| 7 | What is immunization (in one sentence)? | Designing a portfolio to reduce sensitivity to a specified rate shift (often a parallel shift). |
| 8 | What is basis risk? | The risk that the hedge and the position do not move one-for-one. |
| 9 | Does a DV01-neutral portfolio have zero risk? | No: twist, convexity, basis, and spreads can still drive P&L. |
| 10 | What is the CTD in futures hedging? | The cheapest-to-deliver bond that determines the contract’s effective duration/DV01. |
| 11 | What is CS01? | PV sensitivity to a 1bp tightening of a stated spread object. |
| 12 | Why does convexity matter for large moves? | DV01 is linear; convexity adds a quadratic term that dominates for large \(|\Delta y|\). |
| 13 | What is a twist scenario? | Different tenors move by different amounts (or opposite directions). |
| 14 | What does an \(R^2\) in a hedge regression tell you? | Fraction of target variance explained by the hedge factor(s), in-sample. |
| 15 | What is the main unit trap in DV01 calculations? | Confusing per-100, per-\(\$1\)mm, and per-contract quoting. |
| 16 | What is the regression beta decomposition? | \(\beta = \rho \,(\sigma_{\text{target}}/\sigma_{\text{hedge}})\). |
| 17 | Why can historical betas become unreliable? | Regime changes can alter correlations and relative volatilities across the curve. |
| 18 | What happens to futures DV01 when the CTD switches? | It can jump discontinuously as DV01/CF changes. |
| 19 | How do you hedge credit spread risk on a corporate bond? | Use a spread hedge (e.g., buy CDS protection) sized by CS01 vs RPV01. |
| 20 | What is tailing the hedge? | Adjusting futures hedge size for daily settlement (time value of variation margin). |

---

## Mini Problem Set

1. (Compute) A bond has DV01 = 0.072 per 100 and position face \(F=\$15\)mm. Compute position DV01 in \(\$/\text{bp}\).
2. (Compute) Long \(F_A=\$15\)mm of Bond A (DV01 = 0.072 per 100). Hedge with Bond B (DV01 = 0.060 per 100). Compute hedge face \(F_B\).
3. (Compute) DV01 is quoted as 0.085 per 100. A trader mistakenly treats it as 0.085 per \(\$1\) face. By what factor is risk overstated?
4. (Compute) Portfolio value \(P=\$120\)mm, duration \(D_P=6.5\). Futures contract value \(V_F=\$110{,}000\), CTD duration \(D_F=7.8\). Estimate contracts using the duration-based hedge ratio.
5. (Compute) 5Y annual swap with discount factors 0.99, 0.97, 0.95, 0.93, 0.91 and \(\tau_i=1\). Compute PV01 per \(\$1\)mm notional.
6. (Concept) A DV01-neutral hedge loses money when 2Y yields +20bp and 10Y yields -10bp. What went wrong?
7. (Concept) A corporate bond is DV01-hedged with Treasuries. Credit spreads widen 30bp with no rate change. Which risk explains the loss?
8. (Compute) A regression shows \(\beta = 1.15\) and \(\rho = 0.97\). If \(\sigma_{\text{target}} = 5.5\) bp/day, what is \(\sigma_{\text{hedge}}\)?
9. (Concept) Why can a DV01-neutral hedge between a callable and a noncallable bond become unstable as yields change?
10. (Desk) Give two reasons Treasury futures DV01 per contract can change even without rolling.
11. (Compute) Two CTD candidates have \((DV01,CF)=(0.075,0.90)\) and \((0.068,0.88)\) (DV01 per 100). Compute the implied futures DV01 per contract under each CTD assumption for a \(\$100{,}000\) contract size.
12. (Desk) You hedge a \(\$50\)mm corporate bond with Treasuries and expect spreads to widen. Design a two-instrument hedge using Treasury shorts and CDS protection that leaves you flat on rates but long \(\$10{,}000\) of CS01 exposure (state your CS01 sign convention).

### Solution Sketches (Selected)
1. \(DV01^{\$}=\$15{,}000{,}000\times(0.072/100)=\$10{,}800\) per bp.
2. \(F_B=-\$15\text{mm}\times(0.072/0.060)=-\$18\text{mm}\) (short).
3. Factor \(=100\).
4. \(N^*=(120{,}000{,}000\times 6.5)/(110{,}000\times 7.8)\approx 909\) contracts.
5. Annuity \(A=0.99+0.97+0.95+0.93+0.91=4.75\). PV01 per \(\$1\)mm \(=\$1{,}000{,}000\times 4.75\times 10^{-4}=\$475\) per bp.
6. Curve twist / key-rate mismatch: DV01-neutral does not imply twist-neutral.
7. CS01 / spread risk: Treasuries hedge rates, not credit spreads.
8. \(\sigma_{\text{hedge}}=\rho\,\sigma_{\text{target}}/\beta = 0.97\times 5.5/1.15\approx 4.64\) bp/day.
11. Futures DV01 per 100: \(0.075/0.90=0.08333\), \(0.068/0.88=0.07727\). Per contract: \(\$100{,}000\times(0.08333/100)=\$83.33\) and \(\$100{,}000\times(0.07727/100)=\$77.27\) per bp.

---

## References
- (Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, “YIELD-BASED DV01”; “DV01 hedge ratio (equation 5.7)”; “CHAPTER 8 Regression-Based Hedging”; “MULTI-FACTOR EXPOSURES AND RISK MANAGEMENT” (risk weights); “IMPERFECTION OF CONVERSION FACTORS AND THE DELIVERY OPTION AT EXPIRATION”; “TAILS: A CLOSER LOOK AT HEDGING WITH FUTURES”)
- (Neftci, *Principles of Financial Engineering*, “Dollar duration DV01”; “Immunization”)
- (Hull, *Options, Futures, and Other Derivatives*, “Calculating the Minimum Variance Hedge Ratio”; “Impact of Daily Settlement”; “Duration-Based Hedge Ratio / Treasury futures CTD discussion”)
- (Hull, *Risk Management and Financial Institutions*, “Interest Rate Deltas in Practice”)
- (Elton et al., *Modern Portfolio Theory and Investment Analysis*, “Treasury Bond Futures”; “Profits and Losses from Futures Contracts” (mark-to-market cashflows))
- (*Modeling Single Name and Multi Name Credit Derivatives*, “The Bond DV01”; “Building the Libor Discount Curve (PV01 definition)”)
