# Chapter 43: Risks in CDS and Hedging Strategies

---

## Introduction

You've sold CDS protection on a single-name corporate. What exactly are you exposed to? The obvious answer—credit spread movements—captures only part of the picture. A protection seller faces a constellation of risks: the continuous mark-to-market swings as spreads move (CS01), the discrete catastrophe if default actually occurs (jump-to-default), the uncertainty about what recovery rate will materialize at auction, and the subtle curve-shape exposures that arise because a 5-year CDS loads on default probabilities across the entire term structure.

The 2008 financial crisis demonstrated painfully that CS01 alone misses critical exposures. Traders who hedged spread risk but ignored jump-to-default found themselves facing unlimited losses when counterparties defaulted. The Lehman bankruptcy showed that even "hedged" books could sustain massive P&L shocks from recovery uncertainty—the auction final price of 8.625 cents on the dollar was far below most recovery assumptions. These events transformed how desks think about CDS risk: from a single CS01 number to a full taxonomy encompassing spread, default, recovery, curve, and basis dimensions.

O'Kane provides a comprehensive framework in his treatment of CDS risk management, covering "sensitivities to changes in the credit spread curve, changes in the Libor rate curve, changes in the recovery rate, the passage of time, and the risk of a default event." This chapter develops that complete framework systematically. We begin with CS01 and its close relative RPV01 (Section 43.1), then examine jump-to-default risk and O'Kane's Value on Default decomposition (Section 43.2). Section 43.3 addresses recovery risk—both the sensitivity to assumed recovery and the uncertainty around auction outcomes. Section 43.4 covers curve-shape risk and the "credit key-rate" approach analogous to rates bucket exposures from Chapter 14. Section 43.5 introduces spread convexity (gamma), often neglected but material for large moves. Section 43.6 discusses basis risk when hedging CDS with bonds or indices. Finally, Section 43.7 provides the risk management framework: limits, stress testing, counterparty risk, and wrong-way risk considerations.

Throughout, we maintain the connections established in earlier chapters: CS01 for credit is analogous to DV01 for rates (Chapter 11), curve-shape risk parallels key-rate exposures (Chapter 14), and hedging logic follows the same notional-matching principles (Chapter 15). We also preview where these concepts lead—Chapter 44 on CDS relative value and Chapter 47 on index hedging build directly on the risk measures developed here.

---

## Conventions and Notation

**Position sign:** Unless explicitly stated otherwise:
- **Protection buyer** (long protection) has positive value when credit worsens (spreads widen / default becomes more likely)
- **Protection seller** (short protection) has the opposite sign

**Rates and units:**
- 1 bp $= 10^{-4}$ in decimal spread
- Spreads $S$ are written in decimal per year unless explicitly shown in bp

**Valuation time:** $t$. Maturity $T$. Premium payment dates $t_1, \ldots, t_n$ with year fractions $\Delta_i$.

**Discount factor:** $Z(t,u)$ (risk-free discounting)

**Survival probability:** $Q(t,u) = P(\tau > u \mid \mathcal{F}_t)$ under the pricing measure

**Recovery:** $R \in [0,1]$, fraction of par recovered at default for the reference obligation

**Notional:** $N$ dollars

**Contract spread:** $S_0$ (running coupon in the CDS contract)

**Current par spread:** $S(t,T)$, the market running spread that makes a new CDS have zero PV at time $t$

**Risky annuity / RPV01:** $\text{RPV01}(t,T)$, the premium-leg annuity-like PV object

**Value on default:** $\text{VOD}$, the value conditional on default occurring "now"

**CS01 / Credit DV01:** first-order PV sensitivity to a spread bump; sign conventions vary by desk

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time |
| $T$ | CDS maturity |
| $t_i$ | Premium payment dates; $\Delta_i$: accrual year fraction |
| $N$ | Notional (USD) |
| $R$ | Recovery assumption |
| $Z(t,u)$ | Risk-free discount factor |
| $Q(t,u)$ | Survival probability to $u$ |
| $S_0$ | Contractual running spread |
| $S(t,T)$ | Current par spread for maturity $T$ |
| $V(t)$ | PV (USD) of the CDS position |
| $\text{RPV01}(t,T)$ | Risky annuity linking spread changes to PV |
| $\text{VOD}$ | Value on default |

---

## 43.1 Spread Risk: CS01 and RPV01

### 43.1.1 The Central Role of RPV01

Just as duration links yield changes to bond price changes (Chapter 12), RPV01 links spread changes to CDS value changes. O'Kane provides the foundational representation in his Chapter 6:

$$\boxed{V(t) = \bigl(S(t,T) - S_0\bigr) \cdot \text{RPV01}(t,T)}$$

This elegant formula says the mark-to-market of an existing CDS equals the spread differential times the risky annuity. When you buy protection at $S_0 = 100$ bp and spreads widen to $S(t,T) = 150$ bp, you've locked in 50 bp of "excess" protection—the present value of receiving that 50 bp edge over the contract's life is exactly $50\text{bp} \times \text{RPV01}$.

O'Kane defines RPV01 as "the time $t$ present value of a credit risky \$1 annuity which matures at time $T$." The rigorous integral representation is:

$$\text{RPV01}(t,T) = \int_t^T Z(t,s) \, Q(t,s) \, ds$$

For practical computation, O'Kane provides the payment-date approximation including accrued-at-default:

$$\boxed{\text{RPV01}(t,T) = \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(t,t_n) \bigl(Q(t,t_{n-1}) + Q(t,t_n)\bigr)}$$

The trapezoidal weighting $(Q(t,t_{n-1}) + Q(t,t_n))/2$ captures the approximation that default, if it occurs during period $n$, happens mid-period on average. This accounts for the accrued premium that would be paid at default.

**Unit check:** $Z$ (unitless) $\times$ $Q$ (unitless) $\times$ $\Delta$ (years) gives years. Multiplying by spread (per year) yields a unitless PV factor per unit notional.

**Sanity check:** RPV01 is smaller for shorter maturities (fewer premium payments) and for wider spreads (lower survival probability reduces expected premium receipts).

### 43.1.2 CS01 / Credit DV01: Definition and Sign Conventions

CS01 measures the dollar change in PV for a 1 bp change in spreads. O'Kane defines Credit DV01 precisely in Section 8.3.2:

$$\boxed{\text{Credit DV01} = -\bigl(V(S + 1\text{bp}) - V(S)\bigr)}$$

O'Kane explains the sign convention: "The negative sign is used to ensure that the Credit DV01 of a short protection position will be positive." This matches how rates DV01 is typically reported positive for long bond positions—the short-protection position is economically similar to being long a risky bond.

Equivalently, using the derivative form:

$$\text{Credit DV01} \simeq -\frac{\partial V}{\partial S} \times 1\text{bp}$$

**The RPV01 link:** O'Kane shows that for small spread changes, combining the MTM formula with first-order approximation:

$$\Delta V \approx \text{RPV01}(t,T) \times \Delta S$$

Thus for a protection buyer with notional $N$:

$$\text{CS01}_{\text{buyer}} \approx N \times \text{RPV01}(t,T) \times 10^{-4}$$

O'Kane notes two important observations:
- "When $S_t = S_0$, the Credit DV01 equals the RPV01 of the contract times 1bp"
- "When $S_t$ deviates from $S_0$, this is no longer true. The Credit DV01 increases or decreases depending on the value of $(S_0 - S_t)$"

This parallels the rates DV01 formula from Chapter 11, where DV01 $\approx$ Duration $\times$ Price $\times$ 0.0001.

### 43.1.3 What Gets Bumped? The Critical Specification

A CS01 number is meaningless without specifying what is bumped. O'Kane emphasizes this operational detail—different bump methods produce different sensitivities:

**Method 1: Par-spread bump (market risk view)**
- Bump the market par CDS spread $S(t,T_i)$ by +1 bp at the relevant maturity
- Rebuild the survival curve using the bootstrapping machinery from Chapter 42
- Reprice and compute $\Delta V$

This is the standard approach for trading desks measuring exposure to observable market quotes.

**Method 2: Parallel curve shift (O'Kane's Credit DV01)**
- Bump the entire CDS spread curve by +1 bp across all tenors
- Rebuild and reprice

O'Kane describes this as "the change in the value of the CDS position for a one-basis point parallel increase in the CDS curve." This gives a single aggregate number useful for stress testing but misses curve-shape exposures.

**Method 3: Hazard-node bump (model risk view)**
- Hold the spread curve fixed
- Bump the hazard rate $\lambda_k$ in a specific time bucket
- Recompute survival probabilities and reprice

This produces a "hazard DV01" useful for understanding model sensitivities but not directly observable in the market.

**Practical warning:** Mixing CS01 numbers computed under different conventions across systems is a common source of reconciliation breaks.

### 43.1.4 Analytical Approximation for Credit DV01

O'Kane provides an analytical expression showing the exact relationship between Credit DV01 and RPV01:

$$\frac{\text{Credit DV01}}{1\text{bp}} = \text{RPV01}(t,T) + \frac{\Delta(S_0 - S_t)}{(1-R)} \sum_{n=1}^{N} \tau_n \exp\left(-(r_t + S_t(1-R)^{-1})\tau_n\right)$$

The second term captures how the sensitivity changes when the contract is away from par ($S_t \neq S_0$). For a par contract, the Credit DV01 simplifies to RPV01 times 1bp.

---

## 43.2 Jump-to-Default Risk: The Binary vs. Continuous Distinction

### 43.2.1 CS01 vs JTD: Two Fundamentally Different Risks

One of the most critical distinctions in CDS risk management is understanding that **spread risk (CS01) and jump-to-default risk (JTD) are fundamentally different exposures**—and a hedge that neutralizes one may leave you fully exposed to the other.

| Risk Type | Nature | What It Measures | Hedgability |
|-----------|--------|------------------|-------------|
| **CS01 (Spread Risk)** | Continuous | Value change for 1bp spread widening | Readily hedgable with other CDS |
| **JTD (Jump-to-Default)** | Binary/Discrete | Value change if name defaults overnight | Hard to hedge; requires protection |

**CS01** captures the daily mark-to-market volatility from spread movements. If spreads widen 10bp, your P&L is approximately $10 \times \text{CS01}$. This is smooth, continuous risk that accumulates gradually and can be hedged by taking offsetting spread exposure in other CDS or indices.

**JTD** captures the catastrophic, discrete loss that occurs if the reference entity defaults. This is not a function of how spreads move—it's the binary outcome: either default happens (massive P&L shock) or it doesn't (no JTD realization). Unlike spread risk, JTD cannot be hedged incrementally; you either have protection or you don't.

### 43.2.2 The Fundamental Conflict: CS01 Hedge ≠ JTD Hedge

> **The Conflict:** A hedge that perfectly neutralizes CS01 may leave you catastrophically exposed to JTD—or vice versa—if recovery assumptions are wrong or if the hedging instrument doesn't match the reference entity.

Consider this critical scenario:

**Position:** Short protection on \$10mm notional, 5-year CDS, spread at 200bp
**Hedge:** Long protection on a correlated name or index to neutralize CS01

If you hedge CS01 to zero using an index or different single-name:
- **Spread moves:** Net P&L ≈ 0 (hedged)
- **Reference entity defaults:** You owe $(1-R) \times \$10\text{mm} = \$6\text{mm}$ (at 40% recovery)
- **Your hedge:** Pays nothing—it was on a different name/index that didn't default

**This is the essence of "basis risk at default"**—your spread hedge may work perfectly for daily P&L but provide zero protection against the binary default event.

> **Desk Reality: The Friday Night Downgrade**
>
> It's Friday afternoon. You run a credit book with short protection on Name A. You're CS01-flat against the index—you've carefully matched your spread exposure. You go home.
>
> Over the weekend, Name A files for bankruptcy.
>
> **Monday morning P&L:**
> - Your short protection position: Loss of \$6mm (at 40% recovery)
> - Your index hedge: Small gain (index widened 5bp on the news, giving you maybe \$50k)
> - **Net P&L: -\$5.95mm**
>
> Your "hedged" book just lost nearly \$6 million because **CS01 hedging doesn't address JTD risk**. The index gave you spread protection, not default protection on that specific name.
>
> **The lesson:** CS01 and JTD are different animals. Know which one you're exposed to, and which one you've actually hedged.

### 43.2.3 The Discontinuous Nature of Default

While spread movements produce continuous P&L, default is a discontinuous event that can overwhelm all other risk factors. A protection seller might have hedged CS01 to zero yet still face catastrophic loss upon default. This is the essence of jump-to-default (JTD) risk.

> **Analogy: The Dynamite Shack**
>
> *   **Spread Risk (CS01)**: The risk that the wind blows harder (market panic). The shack shakes, and its value changes. You can hedge this by bracing the walls.
> *   **Jump-to-Default (JTD)**: The risk that the dynamite inside the shack explodes.
> *   **The Lesson**: Bracing the walls (hedging CS01) does nothing if the shack explodes. You need a specific hedge for the explosion (buying protection on that specific name).

O'Kane makes this concrete: "A short protection position with a notional of \$10 million and with a contractual spread of 200 bp with a risky PV01 of 4.0 (roughly five years to maturity) will lose a maximum of approximately \$10m × 200bp × 4.0 = \$800,000 if the spread tightens to zero and the protection buyer defaults." But the default of the *reference entity* can cause a loss of \$6 million (assuming 40% recovery)—nearly ten times larger.



### 43.2.4 O'Kane's VOD Decomposition

O'Kane defines Value on Default as "the change in the value of the CDS contract on an immediate default." He breaks this into three components:

1. **The CDS position cancels and has a mark-to-market value of zero.** "If $V(t)$ is the value of the position immediately before default, we lose $V(t)$. If $V(t) < 0$ then this is a gain. The change is therefore $0 - V(t) = -V(t)$."

2. **Protection payment or receipt.** "If we are a protection buyer, we receive par in return for delivery of a defaulted asset which is a gain of $(1-R)$. For a protection seller, it is a loss of $(1-R)$."

3. **Accrued premium settlement.** "The protection buyer pays coupon accrued to the protection seller for the fraction of the CDS premium which has accrued since the previous payment date."

The VOD formula is therefore:

$$\boxed{\text{VOD} = \begin{cases} -V(t) - (1-R) + \Delta_0 S_0 & \text{for a protection seller} \\ -V(t) + (1-R) - \Delta_0 S_0 & \text{for a protection buyer} \end{cases}}$$

where $\Delta_0 S_0$ is the premium accrued at default.

### 43.2.5 VOD as a Measure of Surprise

O'Kane provides crucial intuition: "If default follows a gradual but significant widening in credit spreads, then we should find that the VOD is small. This is because just before default, the value of a long protection contract should converge to $V(t) = (1-R) - \Delta S_0$."

Substituting this into the VOD formula for a protection buyer:
$$\text{VOD} = -[(1-R) - \Delta S_0] + (1-R) - \Delta_0 S_0 \approx 0$$

O'Kane concludes: "As a result, the VOD is in some ways a measure of the 'unexpected shock' of the default."

### 43.2.6 JTD as the P&L Jump

A practical JTD measure reports the change in portfolio value upon default:

$$\boxed{\text{JTD} = \text{VOD} - V(t)}$$

This captures not just the default cash flows but the loss of current mark-to-market. For a par contract where $V(t) = 0$, for the protection buyer:

$$\text{JTD} = (1-R) - \Delta_0 S_0 \approx (1-R)$$

For a contract already in-the-money (e.g., protection buyer when spreads have widened), the JTD loss is smaller—you were already being compensated through positive MTM.

**Why JTD matters more than CS01 for distressed names:** When spreads are at 2000 bp, a 1 bp move is noise. But default probability is elevated, making JTD the dominant risk. Risk limits should scale JTD exposure with default probability.

---

## 43.3 Recovery Risk

### 43.3.1 Recovery Sensitivity

O'Kane notes that "the biggest uncertainty in the VOD is the value of the recovery payment on default." He distinguishes two types of recovery sensitivity:

**Ongoing MTM sensitivity:** Holding survival probabilities fixed:

$$V_{\text{buyer}}(t) = (1-R) A(t) - S_0 \cdot \text{RPV01}(t,T)$$

where $A(t) = \int_t^T Z(t,s)(-dQ(t,s))$ is the "default annuity." Taking the derivative:

$$\boxed{\frac{\partial V_{\text{buyer}}}{\partial R} = -A(t)}$$

**Interpretation:** Protection buyer loses value when assumed recovery increases—a higher recovery means smaller protection payment.

**Rec01 / Recovery Rate DV01:** O'Kane states that practitioners "calculate the change in the mark-to-market value for an absolute change of 1% in the recovery rate. This is known as the recovery rate DV01":

$$\text{Rec01} = \frac{\partial V}{\partial R} \times 0.01$$

### 43.3.2 The Calibration Interaction

The analysis above holds survival fixed, but in practice changing recovery affects implied hazard rates. Under the reduced-form framework (Chapter 36), spread, hazard, and recovery are linked via the credit triangle:

$$S \approx \lambda(1-R)$$

If you recalibrate to fixed market spreads after changing recovery, hazard rates must adjust. This creates a more complex sensitivity where:
- Higher assumed $R$ → higher implied $\lambda$ (to match same spread)
- Protection leg PV changes from both $R$ and $Q$ movements

O'Kane illustrates: "If we widen the market spread curve to 500 bp, the sensitivity changes. The recovery rate sensitivity is now 0.0473 and the change in the value of the position for the same recovery rate reduction is -\$23,670, a much larger fall than when the curve was at 120 bp."

This interaction means recovery risk cannot be separated from model calibration choices.

### 43.3.3 Auction Uncertainty and Real-World Recovery

Market practice uses auctions to determine settlement price after credit events (see Chapter 40). The auction final price $FP$ determines:

$$\text{Protection payout} = (1 - FP) \times N$$

Recovery uncertainty is substantial:
- **Lehman Brothers (2008):** Auction price 8.625%
- **Washington Mutual (2008):** Auction price 57%
- **General Motors (2009):** Auction price 12.5%

For risk management, scenario analysis over plausible recovery ranges is essential. If you model recovery at 40% but auction settles at 20%, the protection payout is 80% vs. 60% of notional—a 33% increase in settlement value.

---

## 43.4 Curve-Shape Risk and Credit Key-Rates

### 43.4.1 Reading the Credit Curve: What Shape Tells You

Before diving into curve risk mechanics, it's essential to understand what credit curve shapes signal about market expectations. Unlike rates curves, credit curves carry direct information about perceived default timing.

**Upward Sloping Curve (Normal):**
- Short-term spreads < Long-term spreads
- **Signal:** "Business as usual"—the market expects the company to survive the near term but assigns some probability to distress over the longer horizon
- **Example:** 1Y spread = 80bp, 5Y spread = 150bp, 10Y spread = 200bp
- **Intuition:** If you survive the next year (likely), you still face cumulative default risk over 10 years

**Flat Curve:**
- Spreads roughly constant across tenors
- **Signal:** Market is uncertain about timing—default risk is perceived as relatively constant per period
- **Example:** 1Y = 150bp, 5Y = 160bp, 10Y = 155bp

**Inverted Curve (Front End Higher):**
- Short-term spreads > Long-term spreads
- **Signal:** Imminent distress—the market fears near-term default
- **Example:** 1Y spread = 500bp, 5Y spread = 350bp, 10Y spread = 280bp
- **Intuition:** If you survive the crisis (unlikely), spreads should normalize; the back end prices in "survival scenario"

O'Kane illustrates this with an example of "a steeply inverted curve implying a distressed credit" where front-end spreads are dramatically elevated relative to the back end.

> **Desk Reality: The Curve as a Survival Bet**
>
> When a credit curve inverts, the front end is pricing high probability of near-term default. The back end is cheaper because:
> 1. If default happens in Year 1, you never reach Year 5—so 5Y protection has less expected payout
> 2. If the company survives, spreads likely normalize—so 5Y protection will be worth less
>
> **Inverted curve = Market says "they probably won't make it."**
> **Upward sloping curve = Market says "if they survive the short term, there's still risk ahead."**

### 43.4.2 Curve Trades: Flatteners and Steepeners

Understanding curve shape allows you to express views on credit deterioration or recovery without taking a directional spread bet.

**The Credit Curve Steepener (Betting on Survival):**
- **Trade:** Buy short-dated protection, sell long-dated protection (DV01-neutral)
- **View:** The company survives the near term; curve normalizes (steepens or un-inverts)
- **Example:** Buy 1Y protection, sell 5Y protection (weighted to match CS01)
- **P&L:** Profits if short-end spreads tighten relative to long-end

**The Credit Curve Flattener (Betting on Distress):**
- **Trade:** Sell short-dated protection, buy long-dated protection (DV01-neutral)
- **View:** Credit deteriorates; curve inverts or flattens as near-term default risk rises
- **Example:** Sell 1Y protection, buy 5Y protection
- **P&L:** Profits if short-end spreads widen relative to long-end

**Common Curve Trade Structures:**
| Trade | Structure | View | Risk |
|-------|-----------|------|------|
| **1s5s Steepener** | Long 1Y prot, Short 5Y prot | Survival, curve normalizes | Default before 1Y (you're short the back) |
| **1s5s Flattener** | Short 1Y prot, Long 5Y prot | Distress, curve inverts | Recovery/spread tightening |
| **2s10s Steepener** | Long 2Y prot, Short 10Y prot | Medium-term survival | Long-dated spread widening |
| **Front-End Roll** | Long 6M, Short 1Y | Extreme near-term view | Curve steepening |

> **Practitioner Note: "Flipping the Curve"**
>
> When traders say they're "betting the curve flips," they mean buying a steepener on a credit with an inverted curve. The thesis: either the company defaults (and the steepener loses less than outright short protection) or it survives and the curve un-inverts (steepener profits).
>
> This is a convex trade: limited loss if default occurs (since both legs pay off in default, with partial offset), but large gain if the company survives and curve normalizes.

### 43.4.3 The Limitation of Parallel CS01

A 5-year CDS is sensitive to the entire term structure of default probabilities, not just "the 5-year spread." Parallel CS01 misses curve-shape risk—the exposure to non-parallel movements like steepening or flattening.

This parallels the rates world: just as key-rate DV01s (Chapter 14) decompose interest rate sensitivity across the curve, "bucketed CS01" or "credit key-rates" decompose credit spread sensitivity.

### 43.4.4 O'Kane's Curve Hedging Framework

O'Kane describes computing hedge ratios by bumping each spread point on the CDS curve while holding other spreads fixed. In Section 8.5, he shows that for on-market CDS (where $V_i = 0$ at inception):

$$\frac{\partial V_i}{\partial S_i} = -\text{RPV01}(t,T_i)$$

O'Kane notes: "We therefore do not need to do any bumping, we can just calculate the risky PV01."

For off-diagonal sensitivities within a bootstrap framework:

$$\frac{\partial V_j}{\partial S_i} = 0 \quad \text{for } i \neq j$$

This locality property—that bumping one spread point doesn't affect the sensitivity of on-market CDS at other maturities—is a consequence of the bootstrap construction.

**CDS equivalent notional** for hedging: To hedge exposure to maturity $T_i$, compute:

$$\boxed{N_i^{\text{hedge}} = \left(-\frac{\partial V}{\partial S_i}\right) \left(\frac{\partial V_i}{\partial S_i}\right)^{-1}}$$

O'Kane explains: "These hedges are known as the CDS equivalent notionals. They tell us how much of the on-market CDS we have to do in order to immunise our CDS position against small movements in the CDS spread curve."

### 43.4.5 Hazard-Bucket Sensitivities

An alternative decomposition bumps hazard rates in time buckets rather than spread quotes. This "model risk" view reveals exposure to the shape of the hazard term structure:

| Bucket | Bump | Measures |
|--------|------|----------|
| $\lambda_{0-1y}$ | +1bp hazard | Near-term default exposure |
| $\lambda_{1-3y}$ | +1bp hazard | Medium-term exposure |
| $\lambda_{3-5y}$ | +1bp hazard | Long-end exposure |

A short-maturity CDS loads only on early buckets; a long CDS has exposure across the curve. Single-maturity hedges leave residual curve-twist risk.

### 43.4.6 Curve Hedge Construction

**Goal:** Hedge a 5y CDS using 3y and 7y instruments to neutralize both parallel and slope risk.

**Method:**
1. Compute bucket sensitivities $\partial V/\partial S_{3y}$ and $\partial V/\partial S_{7y}$ for the 5y position
2. Set up the linear system using O'Kane's framework:
   $$N_3 \cdot \frac{\partial V_3}{\partial S_{3y}} + N_7 \cdot \frac{\partial V_7}{\partial S_{3y}} = -\frac{\partial V}{\partial S_{3y}}$$
   $$N_3 \cdot \frac{\partial V_3}{\partial S_{7y}} + N_7 \cdot \frac{\partial V_7}{\partial S_{7y}} = -\frac{\partial V}{\partial S_{7y}}$$
3. Solve for hedge notionals $N_3, N_7$

This is directly analogous to key-rate hedging in rates (Chapter 15).

O'Kane provides extensive numerical examples showing how equivalent hedge notionals vary with contract maturity and how "the hedges are mostly confined to the maturity points adjacent to the maturity of the CDS contract, i.e., they are local."

---

## 43.5 Spread Convexity (Gamma)

### 43.5.1 Why Linear Approximations Fail for Large Moves

The linear relationship $\Delta V \approx \text{CS01} \times \Delta S$ breaks down for large spread moves. O'Kane addresses this in Section 8.3.3 on spread convexity.

**The source of convexity:** RPV01 itself depends on spreads through survival probabilities. When spreads widen substantially:
- Survival probability drops
- RPV01 decreases (fewer expected premium payments)
- The relationship between $\Delta V$ and $\Delta S$ becomes nonlinear

### 43.5.2 O'Kane's Spread Gamma Formula

O'Kane defines spread convexity (gamma) as "the second-order sensitivity to changes in the market CDS spread." For a short protection position:

$$\frac{\partial^2 V(t)}{\partial S_t^2} = \frac{2\Delta}{(1-R)} \sum_{n=1}^{N} \tau_n \exp\left(-(r_t + S_t(1-R)^{-1})\tau_n\right) + \text{second term}$$

O'Kane notes: "For a new short protection CDS position, $S_t = S_0$, and the second term is zero and the spread convexity is just the first term which is positive."

**Sign convention from O'Kane's Table 8.3:**
- Short protection: **Positive** spread convexity
- Long protection: **Negative** spread convexity

### 43.5.3 Second-Order P&L Approximation

Including the second derivative:

$$\Delta V \approx \frac{\partial V}{\partial S} dS + \frac{1}{2} \frac{\partial^2 V}{\partial S^2} (dS)^2$$

O'Kane provides a numerical example: "We see that 98.8% of the change in the value of the position can be explained by the first-order spread sensitivity."

He concludes: "This example demonstrates that, in general, the effect of spread convexity is small. In addition, when a short protection position is duration hedged by buying protection, the effect of convexity will be reduced as the convexity of the short protection position which is positive will be reduced by the convexity of the long protection position which is negative."

### 43.5.4 Practical Implications

**For large spread moves (>50 bp):** Linear CS01 significantly misestimates P&L. A protection buyer might expect \$500k gain from a 100 bp widening based on CS01, but realize only \$450k due to gamma drag.

**For distressed names:** When spreads are already at 1000+ bp, gamma effects dominate. Small CS01 doesn't mean small risk—the position can still move dramatically on further distress.

**Gamma hedging:** Unlike rates (where you can trade swaptions for gamma), credit gamma is harder to hedge directly. Awareness and scenario analysis are the primary tools.

---

## 43.6 Basis Risk: CDS vs. Bonds vs. Indices

### 43.6.1 Cash-CDS Basis

O'Kane defines the CDS-cash basis as:

$$\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread}$$

He notes this "definition is not explicit in that it does not define which bond Libor spread should be used. For a fixed rate bond, the natural choice is the asset swap spread of a bond trading close to par."

O'Kane divides basis drivers into **fundamental factors** and **market factors**:

**Fundamental Factors (O'Kane Section 5.6.1):**
1. **Funding:** "Default swaps are unfunded transactions and bonds are funded. For the same spread, CDS are favoured by investors who have funding costs above Libor while bonds are favoured by those who fund below."
2. **Delivery option:** "In a CDS contract, the protection buyer is able to choose which physical asset out of a basket of obligations to deliver following a credit event."
3. **Technical default:** "The standard credit events may be viewed as being broader than those which constitute default on a bond."
4. **Loss on default:** "The protection payment on a CDS following a credit event is a fraction $(1-R)$ of the face value of the contract. The default risk on a bond purchased at a full price of $P$ is to a loss of $P-R$."
5. **Premium accrued at default:** CDS pays accrued premium; bonds lose accrued coupon.

**Market Factors (O'Kane Section 5.6.2):**
1. **Relative liquidity:** "The liquidity of the cash and CDS markets is different across the term structure."
2. **Synthetic CDO technical short:** CDO issuance drives CDS spreads tighter.
3. **New issuance/loans:** Corporate issuance widens CDS.
4. **Convertible issuance:** Convertible arbitrage drives protection demand.
5. **Demand for protection:** "It is much easier to go short a credit by buying protection in the CDS market than by shorting a cash bond."
6. **Funding risk:** "Since the CDS is unfunded, it effectively locks in a funding rate of Libor flat until maturity."

### 43.6.2 Basis Risk in Bond-CDS Hedges

Hedging a bond with CDS removes default loss but not basis risk:

**Scenario:** Own \$10mm corporate bond; buy \$10mm CDS protection.
- If default: Bond losses offset by CDS protection payment
- If no default, basis widens: Bond spread +80bp, CDS +40bp
  - Bond MTM loss: significant
  - CDS MTM gain: only partially offsetting
  - Net P&L: **negative** despite "being hedged"

This basis risk was painfully evident in 2008 when basis traded through historical ranges.

### 43.6.3 Index Basis and Proxy Hedging

Using a CDS index to hedge single-name exposure introduces:

**Systematic vs. idiosyncratic risk:** Index hedges capture market-wide moves but leave single-name specific risk.

**Index basis:** The index trades at a spread different from its theoretical fair value (intrinsic spread from Chapter 46). This "index basis" can move independently.

**Constituent mismatch:** Your single name may not be in the index, or may have different weight/recovery assumptions.

**Chapter 47 cross-reference:** Index hedging mechanics and residual risk are covered in detail.

---

## 43.7 Risk Management Framework

### 43.7.1 Risk Limits Structure

A comprehensive CDS risk limit framework includes:

| Limit Type | Metric | Rationale |
|------------|--------|-----------|
| CS01 | $/bp per name, sector, total | Controls spread exposure |
| JTD | $ per name, concentration | Controls default loss |
| Recovery scenarios | PV change at R±10% | Controls recovery uncertainty |
| Curve exposure | Bucket CS01 limits | Controls term structure risk |
| Basis | Cash-CDS mismatch limits | Controls basis divergence |
| Concentration | Notional per issuer | Prevents single-name blow-up |

**Name-level JTD limits are critical:** A book can be CS01-flat but have massive JTD if concentrated in a few names.

### 43.7.2 Portfolio-Level Hedging

O'Kane describes the practical workflow for managing a CDS book in Section 8.7:

1. "We group the CDS positions by reference entity"
2. "For each reference entity, we bump the CDS spreads at the standard hedge maturity points"
3. "For each bump, we calculate the change in the value of the CDS positions"
4. "We use these value changes to generate the equivalent CDS notionals"
5. "We repeat until all of the reference entities have been hedged"

He notes: "One of the advantages of trading a large book is that some of the higher-order risks such as the spread gamma may be largely offset when aggregated over a large number of positions. However, this is far from certain and care should certainly be taken to monitor the higher-order risks to ensure that these risks are not compounding."

### 43.7.3 Stress Testing Scenarios

O'Kane emphasizes that first-order hedging is "only appropriate for fairly small market moves." He recommends: "determining the effect on the book of large movements or shocks in market prices and default. This is best captured by generating scenarios in which we stress the market inputs and reprice the book in each scenario."

**Standard stress scenarios for CDS books:**

**Spread scenarios:**
- Parallel +100bp / +500bp / -50bp
- Curve steepening: short-end -25bp, long-end +75bp
- Curve flattening: short-end +50bp, long-end -25bp
- Single-name blowout: specific issuer +500bp

**Default scenarios:**
- Largest single-name default at assumed recovery
- Largest single-name default at recovery = 10%
- Multiple correlated defaults (sector stress)

**Recovery scenarios:**
- All recovery assumptions +10% / -10%
- Distressed names: recovery at 5% vs. 40% assumption

**Basis scenarios:**
- Cash-CDS basis widens 50bp
- Index basis inverts

O'Kane specifically recommends: "The VOD of the book should be calculated per trade and per reference credit."

### 43.7.4 Counterparty Risk in CDS

O'Kane provides a detailed treatment of counterparty risk in Section 8.8. He identifies a "fundamental asymmetry in the counterparty exposure of the parties to a CDS contract."

**Protection Seller's Counterparty Risk:**
"The scenario in which the protection seller loses due to counterparty risk is one in which the protection buyer stops making the promised payments of premium." The loss is capped by the spread times RPV01—typically less than 10% of notional.

Two mitigating factors:
1. "A tightening of spreads is usually a gradual process and so the changes in contract value could be captured through a collateral posting process"
2. "Default correlation is generally positive, i.e. it is generally unlikely that one credit is tightening while at the same time the counterparty defaults"

**Protection Buyer's Counterparty Risk:**
O'Kane describes the worst case: "The reference credit experiences a credit event, the protection buyer triggers the protection on the CDS, the protection seller fails to deliver the par amount." On \$10 million notional with 40% recovery, the loss is \$6 million.

He concludes: "The protection buyer is therefore the counterparty who has the most counterparty risk."

### 43.7.5 Wrong-Way Risk

Wrong-way risk occurs when credit exposure increases precisely when the counterparty's creditworthiness deteriorates. In CDS:

**Classic example:** Bank A buys protection from Bank B on Corporate C. If Bank B and Corporate C are correlated (same country, same sector), then when Corporate C defaults:
- Bank A needs to collect protection payment
- But Bank B may also be in distress
- Counterparty exposure is highest when counterparty credit is worst

O'Kane notes: "Default correlation is generally positive"—this correlation is the essence of wrong-way risk.

**Mitigation strategies:**
- **Choose counterparties carefully:** Avoid buying protection from entities correlated with reference entity
- **Use collateral/margin:** Daily margin calls limit counterparty exposure buildup
- **Central clearing:** Reduces bilateral counterparty risk
- **Collateral posting:** O'Kane notes this "requires the counterparties to agree to periodically deposit securities equal in value to the CDS mark-to-market"

This is a specific case of the correlation structures discussed in CVA (Chapter 34).

### 43.7.6 P&L Attribution and Risk Reporting

A daily P&L explain for CDS should decompose:

$$\Delta V = \underbrace{\text{CS01} \cdot \Delta S_{\parallel}}_{\text{parallel spread}} + \underbrace{\sum_i \text{CS01}_i \cdot \Delta S_i}_{\text{curve shape}} + \underbrace{\text{Rec01} \cdot \Delta R}_{\text{recovery}} + \underbrace{\mathbf{1}_{\text{default}} \cdot \text{JTD}}_{\text{event}} + \underbrace{\Theta}_{\text{time}} + \underbrace{\text{residual}}_{\text{unexplained}}$$

**Residual drivers:**
- Spread convexity (gamma)
- Interpolation/model changes
- Numerical precision
- Basis effects if multi-instrument book

A well-functioning risk system should have residual < 5% of total P&L on typical days.

> **Deep Dive: Theta (The Cost of Insurance)**
>
> If markets don't move (spreads unchanged, no default), does your P&L stay flat? **No.**
>
> *   **Protection Buyer**: You pay premium every day. You *lose* Theta.
> *   **Protection Seller**: You collect premium every day. You *earn* Theta.
> *   **The Trade-off**: Buying protection is like buying a put option. You bleed cash (Theta) waiting for the crash. If the crash (widening/default) doesn't happen, you slowly bleed to death.

---

## 43.8 Worked Examples

### Shared Conventions

| Convention | Value |
|------------|-------|
| Valuation time | $t = 0$ |
| Notional | $N = 10{,}000{,}000$ |
| Recovery | $R = 40\%$ unless stated |
| Risk-free rate | Flat $r = 3\%$ continuous |
| Premium schedule | Annual payments |

**Toy survival curve (piecewise hazard):**

$$\lambda(t) = \begin{cases} 1.5\% & 0 \leq t \leq 1 \\ 2.0\% & 1 < t \leq 3 \\ 2.5\% & 3 < t \leq 5 \end{cases}$$

### Example 43.1: Computing RPV01 and Par Spread

**Given:** 5-year CDS with shared conventions.

**Step 1: Discount factors**

| Year | $Z_i = e^{-0.03i}$ |
|------|-------------------|
| 1 | 0.9704 |
| 2 | 0.9418 |
| 3 | 0.9139 |
| 4 | 0.8869 |
| 5 | 0.8607 |

**Step 2: Survival probabilities**

| Year | $Q_i$ |
|------|-------|
| 1 | $e^{-0.015} = 0.9851$ |
| 2 | $0.9851 \times e^{-0.02} = 0.9656$ |
| 3 | $0.9851 \times e^{-0.04} = 0.9465$ |
| 4 | $0.9465 \times e^{-0.025} = 0.9231$ |
| 5 | $0.9465 \times e^{-0.05} = 0.9003$ |

**Step 3: Protection leg PV**

Default probabilities: $d_i = Q_{i-1} - Q_i$

$$\text{PV}_{\text{prot}} = (1-R) \sum_{i=1}^5 Z_i \cdot d_i = 0.6 \times 0.0906 = 0.0544$$

**Step 4: RPV01 (using O'Kane's trapezoidal approximation)**

$$\text{RPV01} = \frac{1}{2}\sum_{i=1}^5 Z_i (Q_{i-1} + Q_i) = 4.3694$$

**Step 5: Par spread**

$$S_{5y} = \frac{\text{PV}_{\text{prot}}}{\text{RPV01}} = \frac{0.0544}{4.3694} = 0.01245 = 124.5 \text{ bp}$$

### Example 43.2: CS01 Calculation

**Using Example 43.1 setup with contract at par ($S_0 = 124.5$ bp).**

**CS01 via RPV01 approximation (O'Kane's method):**

$$\text{CS01} = N \times \text{RPV01} \times 10^{-4} = 10{,}000{,}000 \times 4.3694 \times 0.0001 = \$4{,}369/\text{bp}$$

**Verification via finite difference:**

If spreads widen 1bp to 125.5bp:
$$V_+ = (0.012545 - 0.01245) \times 4.3694 \times N = 0.0001 \times 4.3694 \times 10{,}000{,}000 = \$4{,}369$$

As O'Kane notes, "When $S_t = S_0$, the Credit DV01 equals the RPV01 of the contract times 1bp."

### Example 43.3: VOD and JTD

**Given:** Contract spread $S_0 = 124.5$ bp, $R = 40\%$, default occurs $\Delta_0 = 0.25$ years after last premium.

**VOD for protection buyer per O'Kane's formula:**

$$\text{VOD} = -V(t) + (1-R) - \Delta_0 S_0$$

For a par contract ($V(t) = 0$):

$$\text{VOD} = (1 - 0.40) - 0.25 \times 0.01245 = 0.60 - 0.00311 = 0.5969$$

**Dollar VOD:**

$$\text{VOD} \times N = 0.5969 \times 10{,}000{,}000 = \$5{,}968{,}875$$

**For protection seller:**

$$\text{VOD}_{\text{seller}} = -\$5{,}968{,}875$$

This is the seller's loss upon immediate default—nearly \$6mm on a \$10mm notional position. As O'Kane notes, this is a measure of "unexpected shock" since a gradual widening would have built up positive MTM offsetting the default loss.

### Example 43.4: Recovery Sensitivity

**Holding survival fixed, reprice at R = 20%, 40%, 60%.**

**From Example 43.1:** $\sum Z_i d_i = 0.0906$ (the "default annuity")

| Recovery | $(1-R)$ | $\text{PV}_{\text{prot}}$ | $V_{\text{buyer}}$ |
|----------|---------|---------------------------|---------------------|
| 20% | 0.80 | 0.0725 | +\$181,273 |
| 40% | 0.60 | 0.0544 | \$0 (par) |
| 60% | 0.40 | 0.0362 | -\$181,273 |

**Rec01:**

$$\text{Rec01} = \frac{-181{,}273 - 181{,}273}{40\%} \times 1\% = -\$9{,}064 \text{ per +1% recovery}$$

### Example 43.5: Curve-Shape Exposure

**Hazard bucket sensitivities for 5y par CDS:**

| Bucket | $\Delta\lambda$ = +10bp | $\Delta V$ |
|--------|-------------------------|------------|
| 0-1y | +0.001 | +\$5,759 |
| 1-3y | +0.001 | +\$10,668 |
| 3-5y | +0.001 | +\$9,632 |

**Per bp hazard (divide by 10):**

| Bucket | \$/bp hazard |
|--------|--------------|
| 0-1y | \$576 |
| 1-3y | \$1,067 |
| 3-5y | \$963 |

**Insight:** The 5y CDS loads across all buckets, with largest exposure in the 1-3y segment where the hazard bump has the most duration impact.

### Example 43.6: Credit Curve Steepener Trade

**Setup:** A distressed credit with an inverted curve. You believe the company will survive the near-term crisis.

**Initial Curve:**
| Tenor | Spread (bp) | RPV01 |
|-------|-------------|-------|
| 1Y | 800 | 0.95 |
| 5Y | 500 | 3.80 |

**Trade Construction (DV01-neutral 1s5s steepener):**

**Step 1: Determine hedge ratio**
- 1Y CS01 per $1mm = $1,000,000 × 0.95 × 0.0001 = $95/bp
- 5Y CS01 per $1mm = $1,000,000 × 3.80 × 0.0001 = $380/bp
- Ratio: $380/$95 = 4.0

**Step 2: Structure the trade**
- Buy $40mm 1Y protection (long front end)
- Sell $10mm 5Y protection (short back end)

**Step 3: Verify DV01 neutrality**
- 1Y leg CS01: +$40mm × $95/bp per $1mm = +$3,800/bp
- 5Y leg CS01: -$10mm × $380/bp per $1mm = -$3,800/bp
- **Net parallel CS01: $0**

**Scenario A: Survival and Curve Normalization**
*Curve un-inverts:* 1Y → 200bp (-600bp), 5Y → 250bp (-250bp)

| Leg | Notional | Spread Move | P&L |
|-----|----------|-------------|-----|
| 1Y long prot | $40mm | -600bp | -$2,280,000 |
| 5Y short prot | $10mm | -250bp | +$950,000 |
| **Net** | | | **-$1,330,000** |

Wait—this is a loss? Yes, but compare to outright short protection:
- If you had sold $10mm 5Y protection outright: P&L = +$950,000
- But you also would have been short JTD risk of $6mm

**The steepener loses less on survival but also had less JTD exposure.** Let's examine default:

**Scenario B: Default at 40% Recovery**

| Leg | Notional | Protection Payment | Net |
|-----|----------|-------------------|-----|
| 1Y long prot | $40mm | +$24mm (you receive) | +$24mm |
| 5Y short prot | $10mm | -$6mm (you pay) | -$6mm |
| **Net** | | | **+$18mm** |

**Key Insight:** The steepener is **long JTD** because you have more protection bought than sold. This is a convex trade:
- If company survives: Lose on curve normalization, but gain on carry (you receive net spread)
- If company defaults: Large gain from net long protection

**Actual Steepener P&L Profile:**
| Outcome | Steepener | Outright Short 5Y Prot |
|---------|-----------|------------------------|
| Survival + curve normalizes | -$1.33mm | +$0.95mm |
| Default (R=40%) | +$18mm | -$6mm |

The steepener is betting on survival with a safety net—if wrong about survival, you still win big on default.

### Example 43.7: CS01 Hedge vs JTD Exposure

**Goal:** Hedge 5y protection buyer with 3y short protection.

**RPV01s:** $\text{RPV01}_{5y} = 4.37$, $\text{RPV01}_{3y} = 2.76$

**Hedge notional (parallel CS01 neutral, using O'Kane's equivalent notional formula):**

$$N_{3y} = 10\text{mm} \times \frac{4.37}{2.76} = 15.83\text{mm}$$

**Scenario: Curve steepening (5y +5bp, 3y -5bp)**

| Position | $\Delta S$ | $\Delta V$ |
|----------|-----------|------------|
| 5y long prot (\$10mm) | +5bp | +\$21,847 |
| 3y short prot (\$15.83mm) | -5bp | +\$21,847 |
| **Net** | | **+\$43,694** |

The hedge was CS01-neutral to parallel moves but has **positive P&L on steepening**—residual curve risk. As O'Kane emphasizes, this approach only immunizes against "small independent movements" at individual maturity points, not curve twists.

### Example 43.8: Full Risk Report

**Position:** 5y protection buyer, \$10mm notional, contract at par.

| Risk Measure | Value |
|--------------|-------|
| **CS01 (parallel)** | +\$4,369/bp |
| **CS01 (0-1y bucket)** | +\$576/bp hazard |
| **CS01 (1-3y bucket)** | +\$1,067/bp hazard |
| **CS01 (3-5y bucket)** | +\$963/bp hazard |
| **JTD (R=40%)** | +\$5,969k |
| **JTD (R=20%)** | +\$7,969k |
| **Rec01** | -\$9,064 per +1% R |

**Stress scenarios:**

| Scenario | P&L |
|----------|-----|
| Spread +100bp parallel | +\$437k |
| Spread +500bp parallel | +\$1,850k (convexity reduces gain) |
| Default at R=40% | +\$5,969k |
| Default at R=20% | +\$7,969k |
| Recovery +10% (no default) | -\$91k |

### Example 43.9: Bond-CDS Hedge Basis Risk

**Setup:** Own \$10mm corporate bond at price 102, buy CDS protection.

**JTD hedge sizing:**
- Bond loss at default (R=40%): $(102-40)/100 \times 10\text{mm} = \$6.2\text{mm}$
- CDS notional needed: $6.2\text{mm} / 0.6 = \$10.33\text{mm}$

**Scenario: Basis widens (bond spread +80bp, CDS +40bp)**

| Position | Sensitivity | Move | P&L |
|----------|-------------|------|-----|
| Bond | Duration 4.5 | +80bp | -\$367k |
| CDS | RPV01 4.37 | +40bp | +\$181k |
| **Net** | | | **-\$186k** |

Despite being "default hedged," the basis move creates significant loss. This illustrates O'Kane's point that basis factors—funding, liquidity, deliverables—drive divergence between cash and CDS markets.

### Example 43.10: Index Proxy Hedge

**Setup:**
- Single-name 5y CDS: long protection \$10mm, CS01 = \$4,369/bp
- Index 5y CDS: CS01 = \$3,800/bp per \$10mm notional
- Assume beta = 0.8 (single-name moves 0.8x index)

**Hedge notional:**

$$N_{\text{idx}} = 10\text{mm} \times \frac{4{,}369 \times 0.8}{3{,}800} = 9.2\text{mm}$$

**Scenario A: Systematic move (index +20bp, single-name +16bp)**
- Single-name P&L: +\$69,904
- Index hedge P&L: -\$69,904
- Net: \$0

**Scenario B: Idiosyncratic (index +10bp, single-name +40bp)**
- Single-name P&L: +\$174,760
- Index hedge P&L: -\$34,952
- Net: **+\$139,808** (unhedged idiosyncratic gain)

### Example 43.11: P&L Attribution

**Position:** 5y par CDS, protection buyer, \$10mm.

**Day's market moves:**
- 5y spread: +15bp
- 3y spread: +5bp (curve steepening)
- Recovery assumption unchanged
- No default

**Attribution:**

| Component | Calculation | P&L |
|-----------|-------------|-----|
| Parallel spread | \$4,369 × 15bp | +\$65,535 |
| Curve shape | (from bucket sensitivities) | +\$8,200 |
| Recovery | 0 | \$0 |
| Carry/theta | (small) | +\$500 |
| **Total explained** | | +\$74,235 |
| **Actual P&L** | (from system) | +\$75,000 |
| **Residual** | | +\$765 (1.0%) |

Residual within acceptable tolerance.

---

## Summary

CDS risk management requires a multi-dimensional framework that extends far beyond simple spread sensitivity:

1. **CS01 vs JTD are fundamentally different risks:** CS01 is continuous (daily spread volatility), JTD is binary (default event). A CS01 hedge does NOT protect against JTD—the "Friday Night Downgrade" scenario illustrates how a perfectly hedged book can lose millions on a single-name default.

2. **CS01/Credit DV01** measures first-order spread exposure; O'Kane emphasizes always specifying bump method (parallel, bucket, hazard)

3. **RPV01** links spread changes to value changes via O'Kane's fundamental MTM formula: $V = (S - S_0) \times \text{RPV01}$

4. **Jump-to-default (JTD)** captures discontinuous default risk; VOD measures the "unexpected shock" component

5. **Credit curve shape signals market expectations:** Inverted curves (front-end > back-end) signal imminent distress; upward sloping curves indicate "business as usual"

6. **Curve trades (flatteners/steepeners)** allow expression of views on credit survival without directional spread bets. A 1s5s steepener bets on survival while maintaining a long-JTD safety net.

7. **Recovery risk** affects both VOD (default payout) and ongoing MTM through calibration; O'Kane shows sensitivity increases dramatically for wide-spread names

8. **Curve-shape risk** requires bucket exposures—O'Kane's equivalent notional hedges are "local" but don't capture curve twists

9. **Spread convexity** matters for large moves; O'Kane shows it's typically small (98.8% explained by first-order) but accumulates

10. **Basis risk** persists in all cross-instrument hedges; O'Kane catalogues fundamental and market drivers

11. **Risk limits** should cover CS01, JTD, recovery scenarios, curve buckets, and concentrations

12. **Stress testing** must include spread, default, and recovery shocks; O'Kane recommends "calculating VOD per trade and per reference credit"

13. **Counterparty and wrong-way risk** require careful attention; O'Kane notes "the protection buyer is the counterparty who has the most counterparty risk"

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CS01 vs JTD | Spread risk (continuous) vs default risk (binary) | Different hedges needed; CS01 hedge ≠ JTD hedge |
| CS01 | PV change per 1bp spread move | Primary day-to-day risk metric |
| RPV01 | Risky premium-leg annuity | Links spread to value; duration analog |
| JTD/VOD | Value change on default | Discontinuous risk; dominates for distressed |
| Inverted credit curve | Front-end spread > back-end spread | Signals imminent distress |
| Credit curve steepener | Long front, short back (DV01-neutral) | Bets on survival; long JTD |
| Credit curve flattener | Short front, long back (DV01-neutral) | Bets on distress; short JTD |
| Rec01 | PV change per 1% recovery | Often overlooked; material uncertainty |
| Bucket CS01 | Sensitivity to curve points | Captures term structure risk |
| Spread gamma | Second derivative to spread | Matters for large moves |
| Cash-CDS basis | Bond spread vs CDS spread gap | Residual in bond-CDS hedges |
| CDS equivalent notional | Hedge notional to neutralize bucket | O'Kane's key hedging output |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is RPV01? | The risky premium-leg annuity: present value of receiving 1bp of spread until maturity or default |
| 2 | Write O'Kane's CDS MTM formula | $V(t) = (S(t,T) - S_0) \times \text{RPV01}(t,T)$ |
| 3 | What is CS01? | PV change for a 1bp change in specified credit spread input |
| 4 | What is the key difference between CS01 and JTD risk? | CS01 is continuous (daily spread volatility); JTD is binary (default event)—a CS01 hedge does NOT protect against JTD |
| 5 | Why must bump method be specified for CS01? | Different bumps (par spread vs hazard) produce different sensitivities |
| 6 | What is O'Kane's VOD formula for protection buyer? | $\text{VOD} = -V(t) + (1-R) - \Delta_0 S_0$ |
| 7 | How does accrued premium affect VOD? | Reduces buyer's net receipt by $\Delta_0 S_0 N$ |
| 8 | What does an inverted credit curve signal? | Imminent distress—market fears near-term default (front-end spreads > back-end) |
| 9 | What does an upward-sloping credit curve signal? | "Business as usual"—company expected to survive near-term but faces cumulative long-term risk |
| 10 | What is a credit curve steepener? | Long front-end protection, short back-end protection (DV01-neutral); bets on survival |
| 11 | What is a credit curve flattener? | Short front-end protection, long back-end protection (DV01-neutral); bets on distress |
| 12 | If spreads widen, what happens to protection buyer's PV? | Increases (positive CS01) |
| 13 | What is Rec01? | PV change per +1% recovery assumption |
| 14 | Why is recovery sensitivity complex? | Changing R affects implied hazard if spreads held fixed (credit triangle) |
| 15 | What is curve-shape credit risk? | Sensitivity to non-parallel spread movements |
| 16 | What is O'Kane's CDS equivalent notional formula? | $N_i = (-\partial V/\partial S_i) / (\partial V_i/\partial S_i)$ |
| 17 | What is cash-CDS basis per O'Kane? | CDS spread minus bond Libor spread |
| 18 | Name two fundamental basis drivers (O'Kane) | Funding and delivery option (also: technical default, loss-on-default, accrued at default) |
| 19 | Why can bond+CDS hedge leave residual P&L? | Basis moves independently due to fundamental and market factors |
| 20 | What is spread convexity? | Second derivative of V to S; nonlinearity for large moves |
| 21 | For protection buyer, is spread gamma positive or negative? | Negative (gains decelerate as spreads widen; per O'Kane Table 8.3) |
| 22 | What is JTD in a P&L explain? | PV jump from immediate default scenario |
| 23 | Why does JTD matter more than CS01 for distressed names? | Default probability is high; CS01 captures small moves only |
| 24 | Who has more counterparty risk in a CDS per O'Kane? | The protection buyer |
| 25 | What is "The Friday Night Downgrade" scenario? | CS01-hedged book loses massively on weekend default because CS01 hedge (e.g., index) doesn't cover JTD on specific name |

---

## Mini Problem Set

1. A 5y CDS has RPV01 = 4.2 years. Notional \$20mm. Compute CS01 for protection buyer.

   *Sketch:* $\text{CS01} = 20{,}000{,}000 \times 4.2 \times 10^{-4} = \$8{,}400/\text{bp}$

2. Using O'Kane's VOD formula, compute VOD for buyer with $R = 35\%$, $S_0 = 150$bp, $\Delta_0 = 0.25$, $N = \$5$mm, $V(t) = 0$.

   *Sketch:* $\text{VOD} = (1 - 0.35 - 0.25 \times 0.015) \times 5{,}000{,}000 = (0.65 - 0.00375) \times 5\text{mm} = \$3.23\text{mm}$

3. If Rec01 = -\$12,000 per +1%, what is PV change for recovery falling from 40% to 30%?

   *Sketch:* $\Delta R = -10\%$; $\Delta V = -12{,}000 \times (-10) = +\$120{,}000$

4. Explain why a 5y CDS has exposure to 1y hazard rates.

   *Sketch:* 5y survival requires surviving years 1-5; early hazard affects all later survival probabilities. The 5y RPV01 includes premium payments weighted by survival to each date.

5. A bond trades at 95, recovery assumed 40. Size CDS notional for JTD hedge.

   *Sketch:* Loss = $(95-40)/100 \times N_{\text{bond}} = 0.55N$; CDS payout = $0.6N_{\text{CDS}}$; $N_{\text{CDS}} = 0.55/0.6 \times N_{\text{bond}} = 0.917N_{\text{bond}}$

6. Spreads widen 50bp. Initial CS01 = \$5,000/bp. Why might actual P&L be less than \$250,000?

   *Sketch:* Negative gamma for protection buyer—as spreads widen, RPV01 compresses (lower survival), reducing subsequent sensitivity.

7. Set up the system to hedge a 5y exposure using 3y and 7y CDS to neutralize parallel and slope risk, using O'Kane's equivalent notional framework.

8. Bond spread widens 60bp, CDS widens 30bp. Bond duration 5, notional \$10mm. CDS RPV01 = 4.5, notional \$10mm. Compute net P&L.

9. Per O'Kane, why might an index hedge increase idiosyncratic risk while reducing systematic risk?

10. Describe a stress scenario where a CS01-neutral book has significant P&L.

11. A distressed credit has 1Y spread = 600bp (RPV01 = 0.92) and 5Y spread = 400bp (RPV01 = 3.6). Structure a DV01-neutral 1s5s steepener on $10mm notional. What is the net JTD exposure at 40% recovery?

    *Sketch:* 1Y CS01 per $1mm = $92/bp; 5Y CS01 per $1mm = $360/bp; Ratio = 3.91. Trade: Buy $39.1mm 1Y prot, Sell $10mm 5Y prot. Net JTD at R=40%: +$39.1mm × 0.6 - $10mm × 0.6 = +$17.5mm (long JTD).

12. Explain why "The Friday Night Downgrade" scenario results in a loss for a CS01-hedged book.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| CDS MTM formula $V = (S-S_0) \times \text{RPV01}$ | O'Kane Ch 6 |
| RPV01 definition as "credit risky \$1 annuity" | O'Kane Ch 6 |
| RPV01 trapezoidal approximation formula | O'Kane Eq 6.4 |
| Credit DV01 definition with sign convention | O'Kane Section 8.3.2 |
| "Credit DV01 equals RPV01 times 1bp" when at par | O'Kane Section 8.3.2 |
| VOD formula and three-component decomposition | O'Kane Section 8.3.7 |
| VOD as "measure of unexpected shock" | O'Kane Section 8.3.7 |
| Recovery Rate DV01 definition | O'Kane Section 8.3.6 |
| CDS equivalent notional formula | O'Kane Eq 8.7 |
| Spread convexity formula and sign | O'Kane Section 8.3.3 |
| "98.8% explained by first-order sensitivity" | O'Kane Section 8.3.3 example |
| "Steeply inverted curve implying distressed credit" | O'Kane Ch 7 (curve interpolation) |
| CDS-cash basis definition | O'Kane Section 5.6 |
| Fundamental and market basis factors | O'Kane Sections 5.6.1-5.6.2 |
| "Relative value trading" of basis | O'Kane Section 5.6 |
| "Distressed credit" switches to upfront format | O'Kane Ch 6 |
| Portfolio-level hedging workflow | O'Kane Section 8.7 |
| Counterparty risk asymmetry | O'Kane Section 8.8 |
| "Protection buyer has the most counterparty risk" | O'Kane Section 8.8.3 |
| Auction settlement mechanism | O'Kane Ch 5, Hull Ch 25 |
| CDS spread cannot be negative | O'Kane Section 5.6.1 |

### (B) Claude-Extended Content (Priority 2)

| Content | Context |
|---------|---------|
| CS01 vs JTD distinction as "continuous vs binary" | Extended from O'Kane's separate treatments; pedagogical framing |
| "The Friday Night Downgrade" scenario | Practitioner scenario illustrating CS01/JTD mismatch; not directly from O'Kane |
| Credit curve shape interpretation (inverted = distress signal) | Extended from O'Kane's inverted curve example; standard practitioner knowledge |
| Curve trade structures (1s5s steepener/flattener) | Standard practitioner terminology; follows from O'Kane's curve risk treatment |
| "Betting on survival with a safety net" (steepener characterization) | Pedagogical framing based on the mathematics of curve trades |

### (C) Reasoned Inference (Derived from A or B)

- Risk taxonomy organization follows from CDS payoff structure (premium vs protection legs) and O'Kane's systematic treatment
- CS01 ambiguity follows from observation that different bump methods produce different survival perturbations (O'Kane discusses par-spread and parallel bumps)
- P&L attribution formula follows standard Taylor expansion applied to CDS PV function, consistent with O'Kane's first and second-order derivative analysis
- Gamma sign (negative for buyer) follows from RPV01 decreasing with spread, which O'Kane demonstrates via the convexity formula
- JTD importance for distressed names follows from VOD formula—when spreads are high, CS01 moves are small relative to default loss
- Steepener trade being "long JTD" follows from net notional: more protection bought than sold, so net positive payment upon default
- Inverted curve interpretation follows from hazard rate extraction: high front-end spreads → high near-term default probability

### (D) Flagged Uncertainties

- Exact ISDA standard-coupon regime details and specific desk CS01 conventions not fully specified in sources; industry practice may vary
- Recovery swap mechanics not explicitly covered in O'Kane; dedicated instruments for hedging recovery risk are limited
- Wrong-way risk quantification requires correlation modeling beyond the scope of O'Kane's CDS chapters; Hull's CVA treatment provides conceptual framework
- Specific historical auction prices (Lehman 8.625%, etc.) are widely reported but not directly verified from attached sources
- Curve trade terminology (1s5s, 2s10s) is standard practitioner language but not explicitly defined in O'Kane; conventions may vary by desk
