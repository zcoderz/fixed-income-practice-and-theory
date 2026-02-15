# Chapter 43: Risks in CDS and Hedging Strategies

---

## Introduction

You've sold CDS protection on a single-name corporate. What exactly are you exposed to? The obvious answer—credit spread movements—captures only part of the picture. A protection seller faces a constellation of risks: continuous mark-to-market swings as spreads move (CS01), the discrete default event (jump-to-default), uncertainty about recovery at settlement, and curve-shape exposures because a 5-year CDS loads on default risk across many horizons.

A CS01-neutral book can still take large losses when **reference entities** default, because the P&L jump at default is not captured by small-spread hedges. Even when default does not occur, recovery assumptions and cash-vs-CDS basis can move independently of “the par spread”, creating residual P&L in hedged books. The goal of this chapter is to turn “CDS risk” from a single number into a practical taxonomy: spread risk, default jump risk, recovery risk, curve-shape risk, convexity, and basis/hedge mismatch risk.

We begin with CS01 and its close relative RPV01 (Section 43.1), then examine jump-to-default risk and the value-on-default (VOD) decomposition (Section 43.2). Section 43.3 addresses recovery risk—both the sensitivity to assumed recovery and the uncertainty around settlement outcomes. Section 43.4 covers curve-shape risk and “credit key-rate” ideas analogous to rates bucket exposures (Chapter 14). Section 43.5 introduces spread convexity (gamma). Section 43.6 discusses basis risk when hedging CDS with bonds or indices. Finally, Section 43.7 pulls this together into a risk-management checklist: limits, stress testing, counterparty risk, and wrong-way risk considerations.

Prerequisites: [Chapter 11 — DV01/PV01 Definitions and Computation](chapters/chapter_11_dv01_pv01_definitions_computation.md); [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md); [Chapter 15 — DV01 Hedging](chapters/chapter_15_dv01_hedging.md); [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md); [Chapter 42 — Bootstrapping a CDS Survival Curve](chapters/chapter_42_bootstrapping_cds_survival_curve.md)

Follow-on: [Chapter 44 — CDS Relative Value Trading Frameworks](chapters/chapter_44_cds_relative_value_trading_frameworks.md); [Chapter 47 — Hedging and Relative Value in CDS Indices](chapters/chapter_47_hedging_relative_value_cds_indices.md)

---

## Learning Objectives
- Translate a CDS quote and position into cashflows, PV, and a set of risk measures.
- Define and compute/interpret `CS01`, `RPV01`, `VOD`/jump-to-default, and recovery sensitivity with explicit units and sign.
- Explain why “CS01-hedged” does not mean “default-hedged”, and identify the residual risks (recovery and basis).
- Use curve/bucket risk (credit key-rate style) to size hedges and interpret a P&L explain.

## Setup: Signs, Units, and Risk Definitions

**PV sign:** $PV>0$ means the position is an asset to the holder.

**Positions (economic exposure):**
- **Protection buyer** (long protection) benefits from **spread widening** and from the **default payout**.
- **Protection seller** (short protection) has the opposite exposure.

**Units:**
- $1\text{ bp}=10^{-4}$.
- Spreads $S$ are annualized (per year). “200 bp” means $S=0.02$.

**Timeline (simplified):**
- Premium payment dates $t_1,\ldots,t_n$ with accrual fractions $\Delta_i$ (ACT/360 unless stated otherwise).
- Default time $\tau$. If default occurs between $t_{i-1}$ and $t_i$, premium accrues up to $\tau$ and the contract terminates.

**Core objects:**
- $Z(t,u)$: discount factor (from the chosen discount curve).
- $Q(t,u)=\mathbb{P}(\tau>u\mid \mathcal{F}_t)$: survival probability under the pricing measure.
- $R\in[0,1]$: recovery rate (fraction of par); $LGD:=1-R$.
- $N$: notional in currency units.
- $S_0$: contractual running spread on an existing CDS.
- $S(t,T)$: current par spread for maturity $T$ (new-trade spread that gives zero PV at $t$).

**Risk measures (chapter conventions; always state bump design on a real desk):**
- **RPV01** $RPV01(t,T)$: risky PV01 / risky annuity, defined so the premium-leg PV of a running-spread CDS is $PV_{\text{prem}}=N\cdot S_0\cdot RPV01(t,T)$. Units: years.
- **CS01**: $CS01 := PV(S+1\text{bp})-PV(S)$ where the bump applies to the market par spread curve used to rebuild the survival curve (discounting and recovery held fixed). Units: currency per 1bp for the stated notional. With this convention, long protection has **positive** CS01.
- **Credit DV01 (common alternate sign)**: some systems report $-CS01$ so that **short protection** has a positive “DV01”. Do not mix conventions across systems.
- **VOD (value-on-default)**: PV jump under an immediate-default scenario including (i) loss of pre-default MTM, (ii) protection payment $LGD\cdot N$, and (iii) accrued premium.
- **Rec01**: $Rec01 := PV(R+1\%)-PV(R)$. Units: currency per +1% absolute recovery.

---

## 43.1 Spread Risk: CS01 and RPV01

### 43.1.1 The Central Role of RPV01

Just as duration links yield changes to bond price changes (Chapter 12), `RPV01` links spread changes to CDS value changes. A useful identity for a running-spread CDS is:

$$\boxed{V(t) = \bigl(S(t,T) - S_0\bigr) \cdot \text{RPV01}(t,T)}$$

This is the PV of a **protection buyer** (long protection) when $S(t,T)$ and `RPV01` are computed from the same pricing inputs. A protection seller has the opposite PV sign.

**Derivation (one line):** write the PV per unit notional as $V(t)=PV_{\text{prot}}(t,T)-S_0\cdot \text{RPV01}(t,T)$. Define the par spread $S(t,T)$ by setting $0=PV_{\text{prot}}(t,T)-S(t,T)\cdot \text{RPV01}(t,T)$, so $PV_{\text{prot}}(t,T)=S(t,T)\cdot \text{RPV01}(t,T)$. Substituting gives the boxed identity.

**Definition (premium-leg PV):** `RPV01` is defined so that the remaining premium leg PV of a running-spread CDS is

$$PV_{\text{prem}}(t,T) = N \cdot S_0 \cdot \text{RPV01}(t,T).$$

In discrete time, `RPV01` has two parts: scheduled premium payments that occur only if the name survives to each payment date, plus the expected accrued premium paid if default happens between premium dates:

$$
\text{RPV01}(t,T)=\sum_{n=1}^{N} \Delta_n Z(t,t_n) Q(t,t_n)
\;+\;\sum_{n=1}^{N} \int_{t_{n-1}}^{t_n} \Delta(t_{n-1},s)\, Z(t,s)\,(-dQ(t,s)).
$$

Here $\Delta(t_{n-1},s)$ is the year-fraction accrued from $t_{n-1}$ up to $s$ (so the integrand measures “accrued premium if default occurs at $s$”).

A common approximation replaces each accrued-premium integral by a midpoint/trapezoidal rule, yielding:

$$\boxed{\text{RPV01}(t,T) \approx \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(t,t_n) \bigl(Q(t,t_{n-1}) + Q(t,t_n)\bigr)}$$

The trapezoidal weighting $(Q(t,t_{n-1}) + Q(t,t_n))/2$ captures the approximation that default, if it occurs during period $n$, happens mid-period on average. This accounts for the accrued premium that would be paid at default.

**Unit check:** $Z$ (unitless) $\times$ $Q$ (unitless) $\times$ $\Delta$ (years) gives years. Multiplying by spread (per year) yields a unitless PV factor per unit notional.

**Sanity check:** RPV01 is smaller for shorter maturities (fewer premium payments) and for wider spreads (lower survival probability reduces expected premium receipts).

### 43.1.2 CS01 / Credit DV01: Definition and Sign Conventions

CS01 measures the PV change for a +1 bp move in the market spread quote(s), under a specified bump design.

**Chapter definition (CS01):** Let $PV(S)$ denote the CDS PV when the market **par spread curve** is $S(\cdot)$ and the survival curve is calibrated to those quotes. Define:

$$\boxed{\text{CS01} := PV(S+1\text{bp})-PV(S).}$$

The bump is a +1 bp parallel shift unless we explicitly say “bucket CS01”. Unless stated otherwise, discounting and recovery are held fixed under the bump; only the calibrated survival curve is rebuilt.

With this convention:
- Units are currency per 1 bp for the stated notional.
- A protection buyer has **positive** CS01 (spread widening increases PV).

**Common alternate report (“Credit DV01”):** Many systems flip the sign so that **short protection** has a positive “DV01”. One common definition is:

$$\boxed{\text{Credit DV01} := -\bigl(PV(S+1\text{bp})-PV(S)\bigr)=-\text{CS01}.}$$

Always check the definition before sizing hedges.

**Near-par approximation (RPV01 scaling):** If the contract is near par ($S_0 \approx S(t,T)$) and you ignore the fact that `RPV01` itself changes under the bump, then the MTM identity implies:

$$\Delta PV \approx N \cdot \text{RPV01}(t,T) \cdot \Delta S,$$

so

$$\text{CS01} \approx N \cdot \text{RPV01}(t,T) \cdot 10^{-4}.$$

When the contract is away from par ($S_0\neq S(t,T)$), the exact CS01 differs from the simple $N\cdot RPV01\cdot 1\text{bp}$ scaling because the calibrated survival curve (and therefore `RPV01`) moves under the bump.

This parallels the rates DV01 formula from Chapter 11, where DV01 $\approx$ Duration $\times$ Price $\times$ 0.0001.

### 43.1.3 What Gets Bumped? The Critical Specification

A CS01 number is meaningless without specifying what is bumped—different bump methods produce different sensitivities:

**Method 1: Par-spread bump (market risk view)**
- Bump the market par CDS spread $S(t,T_i)$ by +1 bp at the relevant maturity
- Rebuild the survival curve using the bootstrapping machinery from Chapter 42
- Reprice and compute $\Delta V$

This is the standard approach for trading desks measuring exposure to observable market quotes.

**Method 2: Parallel curve shift (aggregate CS01 / “Credit DV01”)**
- Bump the entire CDS spread curve by +1 bp across all tenors
- Rebuild and reprice

This gives a single aggregate number useful for stress testing but misses curve-shape exposures.

**Method 3: Hazard-node bump (model risk view)**
- Hold the spread curve fixed
- Bump the hazard rate $\lambda_k$ in a specific time bucket
- Recompute survival probabilities and reprice

This produces a "hazard DV01" useful for understanding model sensitivities but not directly observable in the market.

> **Pitfall — “What is being bumped?”**
> **Why it matters:** Two “CS01” numbers can differ materially if one system bumps par spreads and rebuilds survival, while another holds the survival curve fixed (or bumps hazard nodes). Hedge ratios built from mismatched definitions can be wrong.
> **Quick check:** When reconciling two CS01s, ask: (i) what curve was bumped (par spreads vs hazard), (ii) was the survival curve rebuilt, and (iii) were discounting and recovery held fixed?

### 43.1.4 Why CS01 Deviates from $N \cdot RPV01 \cdot 1\text{bp}$

The scaling

$$\text{CS01} \approx N \cdot \text{RPV01}(t,T) \cdot 10^{-4}$$

is a useful **near-par** back-of-the-envelope check, but it is not an identity under the chapter’s CS01 definition (par-spread bump with survival-curve rebuild). Two effects create a wedge:

1. **Curve rebuild:** bumping market par spreads changes the calibrated survival probabilities $Q(t,\cdot)$, which changes both the protection leg PV and `RPV01`.
2. **Moneyness:** when $S_0 \neq S(t,T)$, the PV is not “just” $S-S_0$ times a fixed annuity; the annuity itself moves with the curve.

**Check:** In distressed limits, `RPV01` can become very small (few expected premium payments), so CS01 can look small even though `VOD`/JTD remains on the order of $(1-R)N$. This is why CS01-hedged does not imply default-hedged.

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

> **Desk Reality:** It is common to be “CS01-flat” using an index or proxy hedge while still carrying large single-name default exposure.
> **Common break:** Treating a proxy spread hedge as if it were default protection on the specific reference entity.
> **What to check:** Run a single-name default scenario and compare your name-level `VOD`/JTD to any default payoff your hedge would actually deliver (often close to zero for proxies).

### 43.2.3 The Discontinuous Nature of Default

While spread movements produce continuous P&L, default is a discontinuous event that can overwhelm all other risk factors. A protection seller might have hedged CS01 to zero yet still face a large loss upon default. This is the essence of jump-to-default (JTD) risk.

Using the MTM identity $V(t) = \bigl(S(t,T) - S_0\bigr)\cdot \text{RPV01}(t,T)$, a long-protection position's MTM loss from spread tightening is **bounded** because spreads cannot go below zero. If $S(t,T) \to 0$, then $V(t) \to -S_0\cdot \text{RPV01}\times N$.

**Example:** With $N=\$10\text{mm}$, $S_0 = 200$ bp, and $\text{RPV01} \approx 4.0$, the maximum MTM loss from spread tightening is about:
$$10{,}000{,}000 \times 0.02 \times 4.0 = \$800{,}000$$

By contrast, for a **protection seller**, a reference-entity default creates a discrete loss of roughly $(1-R)N$ (e.g., $\$6\text{mm}$ at 40\% recovery). This scale difference is why a CS01 hedge can look “tight” in daily P&L yet fail catastrophically on default.



### 43.2.4 Value on Default (VOD): Decomposition

In this chapter, define `VOD` as the **change in PV caused by default** under an *immediate default* scenario (pre-default MTM to post-default settlement), not the post-default value itself. For a single-name CDS, you can decompose the default jump into three pieces:

1. **The contract terminates:** right after default, there are no remaining CDS cashflows, so the post-default MTM is $0$. This contributes $-V(t)$.
2. **Protection settlement:** the protection buyer receives $LGD=1-R$ per unit notional (the seller pays it).
3. **Accrued premium:** the protection buyer pays premium accrued from the last premium date up to default. If $\Delta_0$ is the accrual year fraction since the last payment date, this is $\Delta_0 S_0$ per unit notional.

Putting this together (per unit notional):

$$\boxed{\text{VOD} = \begin{cases} -V(t) - (1-R) + \Delta_0 S_0 & \text{protection seller} \\ -V(t) + (1-R) - \Delta_0 S_0 & \text{protection buyer.} \end{cases}}$$

### 43.2.5 Why VOD Can Be “Small” Right Before Default

VOD is a *jump* relative to the pre-default MTM $V(t)$. In a stylized picture where the market has already priced a near-certain default (spreads have widened a lot before the event), the pre-default MTM of long protection can get close to the default payoff net of accrued premium. In that limit, the incremental jump at default can be small because the MTM already reflected most of the loss-given-default cashflow.

### 43.2.6 JTD (Jump-to-Default) Risk and Reporting Conventions

In this chapter, **jump-to-default (JTD) risk** means exposure to the PV jump at default. With the definitions above, the immediate-default P&L jump for a CDS position is exactly `VOD`.

For a par CDS ($V(t)=0$), a protection buyer has:

$$\text{JTD}=\text{VOD}=(1-R)-\Delta_0 S_0 \approx (1-R)$$

and the seller has the opposite sign.

**Why JTD matters more than CS01 for distressed names:** When spreads are at 2000 bp, a 1 bp move is noise. But default probability is elevated, making JTD the dominant risk. Risk limits should scale JTD exposure with default probability.

---

## 43.3 Recovery Risk

### 43.3.1 Recovery Sensitivity

Recovery enters CDS risk in two distinct ways:
1. **Default-contingent payout:** the settlement cashflow at default scales with $LGD=1-R$, so recovery has first-order impact on `VOD`/JTD.
2. **Calibration interaction:** if you re-calibrate the survival curve to keep market spreads fixed after changing $R$, then $Q(t,\cdot)$ changes too and affects both legs.

To isolate the mechanics, first hold survival probabilities fixed. For a protection buyer:

$$V_{\text{buyer}}(t) = (1-R) A(t) - S_0 \cdot \text{RPV01}(t,T)$$

where
$$A(t) := \int_t^T Z(t,s)(-dQ(t,s))$$
is the PV of a unit-$LGD$ protection leg (sometimes called a “default annuity”). Taking the derivative:

$$\boxed{\frac{\partial V_{\text{buyer}}}{\partial R} = -A(t)}$$

**Interpretation:** Protection buyer loses value when assumed recovery increases—a higher recovery means smaller protection payment.

**Rec01 / Recovery Rate DV01 (chapter definition):** PV change for an absolute +1\% change in recovery:

$$\text{Rec01} = \frac{\partial V}{\partial R} \times 0.01$$

**Check (units + toy magnitude):** $A(t)$ is a PV *per unit notional* (unitless fraction, like “cents per dollar of notional”). If $A(t)=0.08$ and $N=\$10\text{mm}$, then $\partial V_{\text{buyer}}/\partial R = -0.08\times \$10\text{mm}=-\$800{,}000$ per absolute $+1.00$ change in $R$. So $Rec01 \approx -\$8{,}000$ per $+1\%$ recovery bump for that notional under the “hold $Q$ fixed” convention.

### 43.3.2 The Calibration Interaction

The analysis above holds survival fixed, but in practice changing recovery affects implied hazard rates. In a stylized flat-curve approximation (flat hazard, ignore discounting and premium-accrual details), spread, hazard, and recovery are linked via the credit triangle:

$$S \approx \lambda(1-R)$$

If you recalibrate to fixed market spreads after changing recovery, hazard rates must adjust. This creates a more complex sensitivity where:
- Higher assumed $R$ → higher implied $\lambda$ (to match same spread)
- Protection leg PV changes from both $R$ and $Q$ movements

This is why “recovery sensitivity” is not a single number unless you specify what is held fixed: $R$ only, or $R$ plus a recalibrated curve that keeps spreads fixed.

**Check (par-contract intuition):** if you define Rec01 by bumping $R$ and **rebuilding the survival curve to keep market par spreads fixed** (the curve-risk convention from Chapter 42), then an on-market CDS that is itself one of the calibration instruments often stays close to $PV=0$ after the rebuild—so its Rec01 can be small “by construction”. Off-market contracts (or any instrument that depends on the absolute hazard level, not just par repricing) can have materially non-zero Rec01 under the same bump-and-rebuild definition.

### 43.3.3 Auction Uncertainty and Real-World Recovery

Market practice uses auctions to determine settlement price after credit events (see Chapter 40). If the auction final price is $FP$ (price per 1 of par), then the cash-settlement protection payment is:

$$\text{Protection payout} = (1 - FP) \times N$$

Because the final price (and the implied recovery) is uncertain, risk management typically uses recovery scenarios (e.g., 20\%, 40\%, 60\%) rather than a single point estimate. A simple sanity check: if your model assumes $R=40\%$ but the realized recovery is $20\%$, then $LGD$ rises from $60\%$ to $80\%$, increasing the protection payout by $\frac{80}{60}-1 \approx 33\%$ (before accrued premium).

---

## 43.4 Curve-Shape Risk and Credit Key-Rates

### 43.4.1 Reading the Credit Curve: What Shape Tells You

Before diving into curve risk mechanics, it's essential to understand what credit curve shapes signal about market expectations. Unlike rates curves, credit curves carry direct information about perceived default timing.

**Upward Sloping Curve (Normal):**
- Short-term spreads < Long-term spreads
- **Heuristic interpretation:** the market assigns relatively low near-term default risk and higher cumulative longer-horizon risk
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
- **Intuition:** If the firm survives the near-term stress, spreads may normalize; the back end can reflect a “survival” state.

An extreme version is a *steeply* inverted curve, where the front end is dramatically elevated relative to the back end.

> **Desk Reality:** A credit curve is often read as a “when does default happen?” bet, not just a “how risky is the firm?” level.
> **Common break:** Treating the back end as “cheap protection” without noticing that if default happens early you may never reach those later premium dates.
> **What to check:** Compare `RPV01` and `VOD` across tenors; ask whether your trade is implicitly long or short the survival state.

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

> **Desk Reality:** “Curve flip” trades are often steepeners on inverted curves, sized to be roughly parallel-CS01 neutral.
> **Common break:** A CS01-neutral steepener can still be meaningfully **long or short JTD** (and sensitive to recovery), because default affects legs differently (notional, accrual, and maturity).
> **What to check:** Compute `VOD`/JTD for each leg under the same $R$ and $\Delta_0$, then net them.

**Check (why CS01-neutral often implies non-zero net JTD):** consider a 1s5s steepener: long 1Y protection notional $N_1$, short 5Y protection notional $N_5$. A quick CS01-neutral sizing uses $N_1 \cdot RPV01_{1Y} \approx N_5 \cdot RPV01_{5Y}$. If $RPV01_{1Y}\approx 1.0$ and $RPV01_{5Y}\approx 4.0$, then $N_5\approx 0.25N_1$. The net default payoff scale is roughly $LGD\,(N_1-N_5)\approx 0.75\,LGD\,N_1$: you are still materially **long default protection notional** even though parallel CS01 is near zero.

### 43.4.3 The Limitation of Parallel CS01

A 5-year CDS is sensitive to the entire term structure of default probabilities, not just "the 5-year spread." Parallel CS01 misses curve-shape risk—the exposure to non-parallel movements like steepening or flattening.

This parallels the rates world: just as key-rate DV01s (Chapter 14) decompose interest rate sensitivity across the curve, "bucketed CS01" or "credit key-rates" decompose credit spread sensitivity.

### 43.4.4 Equivalent Hedge Notionals (Bucketed CS01)

Parallel CS01 is a single number; curve hedging needs a **vector**. Pick standard liquid tenors $T_1,\ldots,T_m$ (the same points used to build the survival curve in Chapter 42), and define **bucket CS01s** by bumping *one* par spread quote at a time and rebuilding the curve:

$$\text{CS01}_i := PV(\ldots,S_i+1\text{bp},\ldots)-PV(\ldots,S_i,\ldots).$$

Let $s \in \mathbb{R}^m$ be your portfolio’s bucket-CS01 vector. For each candidate hedge instrument $j$ (typically an on-market CDS at one of the curve tenors), compute its own bucket-CS01 vector $h^{(j)} \in \mathbb{R}^m$. If you choose $k$ hedges with notionals $n_1,\ldots,n_k$, your post-hedge bucket exposure is:

$$s_{\text{net}} = s + \sum_{j=1}^{k} n_j\, h^{(j)}.$$

**Equivalent hedge notionals** are the $n_j$ that make selected components of $s_{\text{net}}$ close to zero (exactly zero if the system is square and well-conditioned; otherwise least-squares).

**Toy example (2-bucket hedge, diagonal approximation):** suppose your book has bucket CS01s (long protection convention) of $s=(+12{,}000,\ +8{,}000)$ $\$/\text{bp}$ to the (3Y, 7Y) quotes. Suppose a \$10mm notional on-market 3Y CDS has bucket CS01 $(+3{,}000,\ 0)$ $\$/\text{bp}$ and a \$10mm on-market 7Y CDS has $(0,\ +6{,}000)$ $\$/\text{bp}$. To neutralize, choose hedge notionals (in units of \$10mm notional)
$$
n_3 = -12{,}000/3{,}000=-4.0,\qquad n_7=-8{,}000/6{,}000\approx -1.33.
$$
Negative notionals mean you **sell protection** in those hedges (negative CS01) to offset the book’s positive CS01.

**Check:** In practice the hedge response matrix is often “somewhat local” (a tenor mostly loads on nearby curve points), but it is not perfectly diagonal once you include bootstrap/interpolation details. Always validate by repricing the full book under the same bucket bumps used to compute the hedge.

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
2. Set up the linear system using bucket CS01s:
   $$N_3 \cdot \frac{\partial V_3}{\partial S_{3y}} + N_7 \cdot \frac{\partial V_7}{\partial S_{3y}} = -\frac{\partial V}{\partial S_{3y}}$$
   $$N_3 \cdot \frac{\partial V_3}{\partial S_{7y}} + N_7 \cdot \frac{\partial V_7}{\partial S_{7y}} = -\frac{\partial V}{\partial S_{7y}}$$
3. Solve for hedge notionals $N_3, N_7$

This is directly analogous to key-rate hedging in rates (Chapter 15).

As with rates key-rate hedging, this construction is only a first-order hedge around today’s curve; large moves and defaults require scenario repricing.

---

## 43.5 Spread Convexity (Gamma)

### 43.5.1 Definition: Spread Gamma Under a Bump Design

Spread convexity (often called “spread gamma”) is the **second-order** sensitivity of CDS PV to spread moves, under a specified bump design (parallel vs bucket, rebuild vs hold-fixed curve).

For a parallel bump design, let $V(\delta)$ be PV after shifting the market par-spread curve by $\delta$ (in bp) and rebuilding the survival curve. Then:

$$\Gamma_S := \frac{\partial^2 V}{\partial \delta^2}.$$

A practical finite-difference estimate is:

$$\Gamma_S \approx \frac{V(+1\text{bp})-2V(0)+V(-1\text{bp})}{(1\text{bp})^2}.$$

**Units:** currency per bp$^2$ for the stated notional.

### 43.5.2 Intuition: Why Long Protection Often Has Negative Gamma

The nonlinearity comes from the fact that the calibrated survival curve changes with spreads. As spreads widen, survival probabilities tend to fall and `RPV01` tends to **compress** (fewer expected premium payments). That means the marginal PV gain per additional bp of widening typically **shrinks** as spreads get wider.

With the chapter’s conventions (long protection has positive CS01), this shrinking marginal sensitivity corresponds to **negative** spread gamma for long protection (and positive gamma for short protection).

**Check (cap argument):** For extremely wide spreads, the PV of long protection is bounded above by something close to the default payoff $(1-R)N$ net of accrued premium, so the first derivative with respect to spread must eventually approach zero. That implies negative curvature somewhere along the way.

### 43.5.3 When It Matters (P&L Explain and Stress)

A second-order P&L approximation is:

$$\Delta V \approx \text{CS01}\cdot \Delta \delta + \frac{1}{2}\Gamma_S(\Delta \delta)^2.$$

For small daily moves, the quadratic term is usually small; for large shocks or distressed names, it can be material. In practice, the robust way to capture convexity is to **reprice under scenarios** (e.g., $\pm 50$ bp, $\pm 200$ bp, curve twists) rather than rely on a single gamma number.

---

## 43.6 Basis Risk: CDS vs. Bonds vs. Indices

### 43.6.1 Cash-CDS Basis

The cash–CDS basis is the spread difference between a CDS quote and a bond spread measure for (approximately) the same credit risk:

$$\boxed{\text{Cash--CDS basis} := S_{\text{CDS}} - s_{\text{bond}}.}$$

Here $S_{\text{CDS}}$ is a par CDS spread, and $s_{\text{bond}}$ is a bond spread measure such as an asset-swap spread or a Z-spread for a representative bond.

**Important:** there is no single universal basis definition because you must specify (i) which bond(s), (ii) which spread measure, and (iii) how you treat coupons, accrual, and discounting. Always state your basis definition before comparing levels across names or time.

**Mechanism-level drivers (why the basis is not forced to zero):**
- **Funding and carry:** bonds are funded instruments; CDS is (economically) unfunded but requires margin/collateral.
- **Liquidity and shorting constraints:** it can be easier to short credit via CDS than via cash bonds.
- **Contract and settlement differences:** deliverables/auction mechanics, accrued premium vs accrued coupon, and documentation details can matter around a credit event.
- **Technical supply/demand:** hedging flows and positioning can move one market more than the other.

### 43.6.2 Basis Risk in Bond-CDS Hedges

Buying CDS against a bond can remove a large part of **default loss**, but it does not, by itself, remove **mark-to-market** risk from spread/basis moves.

**Default-hedge sizing (first-order):** Let a bond have face $N_{\text{bond}}$, dirty price $P$ per 100 of face, and assume recovery $R$ (as a fraction of par). A stylized default loss on the bond is:

$$\text{Loss}_{\text{bond}} \approx \left(\frac{P}{100} - R\right) N_{\text{bond}}.$$

A CDS protection payment is $(1-R)N_{\text{CDS}}$. Matching default payout gives:

$$\boxed{N_{\text{CDS}} \approx \frac{\frac{P}{100}-R}{1-R}\,N_{\text{bond}}.}$$

**Checks (limiting cases + toy sizing):**
- If $P=100$ (bond at par), then $N_{\text{CDS}}\approx N_{\text{bond}}$: you hedge roughly “face-for-face”.
- If $P=85$ and $R=40\%$, then $N_{\text{CDS}}\approx \frac{0.85-0.40}{0.60}N_{\text{bond}}=0.75\,N_{\text{bond}}$: you need less CDS notional because the bond is already below par.
- If $P/100 < R$, the stylized formula implies a negative CDS notional (suggesting the bond could *gain* on default in that crude model). Treat this as a warning that you are in a distressed pricing regime where clean “default loss” approximations can break; size hedges with a full scenario repricing instead.

Even with this sizing, **basis risk remains** because the bond’s spread measure and the CDS spread can move differently (liquidity, funding, and instrument-specific technicals). In addition, a bond+CDS package can retain interest-rate DV01 unless you separately hedge rates.

### 43.6.3 Index Basis and Proxy Hedging

An index hedge is a **proxy hedge**: it is designed to reduce **systematic** credit spread risk, not to replicate the exact default and recovery profile of a single name.

Common residuals:
- **Idiosyncratic risk:** the single name can move (or default) without the index moving proportionally.
- **Constituent/weight mismatch:** even if the name is in the index, it is only a small weight; index default payoffs are “diluted” across many names.
- **Index-specific basis:** the index can trade rich/cheap versus its “intrinsic” value (Chapter 46), and that basis can move.

See Chapter 47 for index hedging mechanics and residual-risk management.

---

## 43.7 Risk Management Framework

### 43.7.1 Risk Limits Structure

A comprehensive CDS risk limit framework includes:

| Limit Type | Metric | Rationale |
|------------|--------|-----------|
| CS01 | \$ per bp per name, sector, total | Controls spread exposure |
| JTD | \$ per name, concentration | Controls default loss |
| Recovery scenarios | PV change at R±10\% | Controls recovery uncertainty |
| Curve exposure | Bucket CS01 limits | Controls term structure risk |
| Basis | Cash-CDS mismatch limits | Controls basis divergence |
| Concentration | Notional per issuer | Prevents single-name blow-up |

**Name-level JTD limits are critical:** A book can be CS01-flat but have massive JTD if concentrated in a few names.

### 43.7.2 Portfolio-Level Hedging

A practical desk-style workflow is:

1. **Normalize positions by reference entity and contract features** (seniority, currency, documentation, maturity).
2. **Compute a risk vector per name:** PV, parallel CS01, bucket CS01s at standard tenors, `VOD`/JTD, and `Rec01` (often under a few recovery scenarios).
3. **Choose hedge instruments** (single-name CDS at liquid tenors and/or indices for systematic risk) and solve for hedge notionals that neutralize the bucket-CS01 vector (Section 43.4.4).
4. **Validate by repricing:** run the same spread-bump scenarios used to compute CS01s, plus a few large-move and default scenarios, to see what remains after the linear hedge.

Diversification can reduce higher-order effects in aggregate, but it does not eliminate them; convexity, basis, and concentration risk can still dominate in stress.

### 43.7.3 Stress Testing Scenarios

**Standard stress scenarios for CDS books:**

**Spread scenarios:**
- Parallel +100bp / +500bp / -50bp
- Curve steepening: short-end -25bp, long-end +75bp
- Curve flattening: short-end +50bp, long-end -25bp
- Single-name blowout: specific issuer +500bp

**Default scenarios:**
- Largest single-name default at assumed recovery
- Largest single-name default at recovery = 10\%
- Multiple correlated defaults (sector stress)

**Recovery scenarios:**
- All recovery assumptions +10\% / -10\%
- Distressed names: recovery at 5\% vs. 40\% assumption

**Basis scenarios:**
- Cash-CDS basis widens 50bp
- Index basis inverts

**Operational note:** compute `VOD`/JTD at the trade level (and aggregate by reference entity). That makes default scenarios concrete and helps concentration limits.

### 43.7.4 Counterparty Risk in CDS

In a bilateral CDS, counterparty exposure is driven by the contract’s mark-to-market and the collateral/margin mechanics.

- **If you are long protection:** you can have a large positive MTM in distress (and a large receivable at default). Your main counterparty concern is “will the protection seller pay when I need it most?”
- **If you are short protection:** your counterparty concern is typically the loss of future premium receipts if the buyer defaults, plus any replacement-cost dynamics if you are owed MTM.

Central clearing and daily margining can reduce (but not eliminate) counterparty exposure; wrong-way and liquidity stresses can still matter.

### 43.7.5 Wrong-Way Risk

Wrong-way risk occurs when credit exposure increases precisely when the counterparty's creditworthiness deteriorates. In CDS:

**Classic example:** Bank A buys protection from Bank B on Corporate C. If Bank B and Corporate C are correlated (same country, same sector), then when Corporate C defaults:
- Bank A needs to collect protection payment
- But Bank B may also be in distress
- Counterparty exposure is highest when counterparty credit is worst

**Mitigation strategies:**
- **Choose counterparties carefully:** Avoid buying protection from entities correlated with reference entity
- **Use collateral/margin:** Daily margin calls limit counterparty exposure buildup
- **Central clearing:** Reduces bilateral counterparty risk
- **Collateral posting:** agree clear collateral terms for MTM and disputes

This is a specific case of the correlation structures discussed in CVA (Chapter 34).

### 43.7.6 P&L Attribution and Risk Reporting

A daily P&L explain for CDS should decompose:

$$\Delta V = \underbrace{\text{CS01} \cdot \Delta S_{\parallel}}_{\text{parallel spread}} + \underbrace{\sum_i \text{CS01}_i \cdot \Delta S_i}_{\text{curve shape}} + \underbrace{\text{Rec01} \cdot \Delta R}_{\text{recovery}} + \underbrace{\mathbf{1}_{\text{default}} \cdot \text{JTD}}_{\text{event}} + \underbrace{\Theta}_{\text{time}} + \underbrace{\text{residual}}_{\text{unexplained}}$$

**Residual drivers:**
- Spread convexity (gamma)
- Interpolation/model changes
- Numerical precision
- Basis effects if multi-instrument book

A well-functioning risk system should have a **small** residual most days; persistent large residuals usually mean a missing risk factor (basis, convexity, recovery/curve convention drift) or a methodology mismatch.

> **Desk Reality:** CDS P&L is often split into spread MTM, event/default, and carry/accrual (premium accrual and accrued-premium unwind).
> **Common break:** Double-counting accrual (as cash and as PV change) or mixing PV conventions (e.g., “clean PV” vs PV including accrued premium).
> **What to check:** Confirm how your system defines PV/CS01 (includes or excludes accrued premium) and how it treats accrued-on-default when computing `VOD`.

---

## 43.8 Worked Examples

### Worked Example (Template): CS01-hedged is not default-hedged

**Example Title**: Single-name CDS hedged with an index proxy

**Context**
- You are long protection on a single name and hedge day-to-day spread MTM with an index proxy.
- Goal: compute PV, `CS01`, and `VOD`/JTD with consistent signs/units, then see what the proxy hedge does and does not cover.

**Timeline (Make Dates Concrete)**
- Valuation date: 2026-02-18
- Current accrual period start: 2025-12-20
- Next premium date: 2026-03-20
- Maturity (standard 5Y): 2031-03-20
- Default scenario date: 2026-02-18 (between coupons)

**Inputs**
- Notional: $N=\$10{,}000{,}000$
- Contract running spread: $S_0=150$ bp
- Current market par spread (5Y): $S(t,T)=175$ bp
- `RPV01`: $\text{RPV01}(t,T)=4.20$ years (from the calibrated survival curve)
- Recovery: $R=40\%$ (so $LGD=60\%$)
- Day count for premium accrual: ACT/360
- Proxy hedge: 5Y CDS index with $\text{CS01}_{\text{idx}} = \$3{,}800/\text{bp}$ per \$10mm index notional (assumed)
- Index weight of the name: $w=1\%$ (assumed; used only to illustrate default payoff dilution)

**Outputs (What You Produce)**
- PV (long protection): $PV \approx \$105{,}000$
- Parallel `CS01` (quick check): $\text{CS01} \approx +\$4{,}200/\text{bp}$
- `VOD`/JTD for immediate default on 2026-02-18: $\text{VOD} \approx +\$5.87\text{mm}$
- Index hedge notional to neutralize parallel CS01: sell protection on $\$11.05\text{mm}$ index notional

**Step-by-step**
1. **Translate quote to PV (MTM identity):**
   $$PV \approx N\,(S(t,T)-S_0)\,\text{RPV01} = 10{,}000{,}000\,(0.0175-0.0150)\,4.20 \approx \$105{,}000.$$
2. **Compute a quick CS01 check:**
   $$\text{CS01} \approx N \cdot \text{RPV01} \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.20 \cdot 10^{-4} = \$4{,}200/\text{bp}.$$
3. **Accrued premium to default:**
   - Days from 2025-12-20 to 2026-02-18: $60$ days, so $\Delta_0 = 60/360 = 0.1667$.
   - Accrued premium $= N\,S_0\,\Delta_0 \approx 10{,}000{,}000 \cdot 0.0150 \cdot 0.1667 \approx \$25{,}000$.
4. **Compute VOD (immediate-default PV jump):**
   $$\text{VOD} = -PV + (1-R)N - N S_0 \Delta_0 \approx -105{,}000 + 6{,}000{,}000 - 25{,}000 \approx \$5{,}870{,}000.$$
5. **Size the proxy hedge (parallel CS01-neutral):**
   - Index CS01 per \$1mm notional is $\$3{,}800/10 = \$380/\text{bp}$.
   - Required index notional to offset $\$4{,}200/\text{bp}$: $N_{\text{idx}} \approx 4{,}200/380 \approx \$11.05\text{mm}$.
   - Implement by **selling** index protection (negative CS01) to offset the single-name’s positive CS01.

**Cashflows (table; long protection, illustrative)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-03-20 | $-N S_0 \Delta$ $\approx -\$37{,}500$ | scheduled quarterly premium if survive ($\Delta \approx 0.25$) |
| 2026-02-18 (default) | $+(1-R)N - N S_0 \Delta_0$ $\approx +\$5{,}975{,}000$ | protection payment minus accrued premium; contract terminates |

**P&L / Risk Interpretation**
- The PV ($\$105k$) is small compared to the default jump: the position is primarily **event risk**.
- The CS01 ($\$4.2k/\text{bp}$) means a 10 bp widening is only about $+\$42k$ of MTM.
- A proxy hedge can neutralize systematic spread moves while leaving most name-default risk untouched.

To see the proxy limitation, note that index default payoffs are diluted by constituent weights. A default produces an index payoff of about $w\cdot (1-R)\,N_{\text{idx}}$ on the index leg. With $w=1\%$, $R=40\%$, and $N_{\text{idx}}=\$11.05\text{mm}$, that is only:

$$0.01 \cdot 0.60 \cdot 11.05\text{mm} \approx \$66{,}300,$$

far smaller than the $\approx \$5.87\text{mm}$ name-level `VOD`.

**Sanity Checks**
- **Units:** spread (1/yr) × `RPV01` (yr) × notional (currency) = currency PV.
- **Sign:** long protection has positive CS01 and positive VOD.
- **Limit:** if $S(t,T)=S_0$, PV $\approx 0$ by construction.

**Debug Checklist (When Your Result Looks Wrong)**
- Are you long or short protection (sign of PV/CS01/VOD)?
- Does your PV include accrued premium, or is it a “clean PV”?
- Did you use ACT/360 for CDS accrual and the right accrual start date (IMM coupon dates)?
- For CS01: did you bump par quotes and rebuild the survival curve, or hold $Q$ fixed?

### Additional Toy Examples (Simplified Assumptions)

The following toy examples intentionally simplify conventions (e.g., annual premium payments, flat rates) to make the arithmetic transparent. They are for intuition only; production CDS uses the market coupon schedule and accrual conventions (see Sections 43.1–43.3).

| Convention | Value |
|------------|-------|
| Valuation time | $t = 0$ |
| Notional | $N = 10{,}000{,}000$ |
| Recovery | $R = 40\%$ unless stated |
| Risk-free rate | Flat $r = 3\%$ continuous |
| Premium schedule | Annual payments (toy simplification) |

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

**Step 4: RPV01 (trapezoidal approximation)**

$$\text{RPV01} = \frac{1}{2}\sum_{i=1}^5 Z_i (Q_{i-1} + Q_i) = 4.3694$$

**Step 5: Par spread**

$$S_{5y} = \frac{\text{PV}_{\text{prot}}}{\text{RPV01}} = \frac{0.0544}{4.3694} = 0.01245 = 124.5 \text{ bp}$$

### Example 43.2: CS01 Calculation

**Using Example 43.1 setup with contract at par ($S_0 = 124.5$ bp).**

**CS01 via RPV01 approximation (quick check):**

$$\text{CS01} = N \times \text{RPV01} \times 10^{-4} = 10{,}000{,}000 \times 4.3694 \times 0.0001 = \$4{,}369/\text{bp}$$

**Verification via finite difference:**

If spreads widen 1bp to 125.5bp:
$$V_+ = (0.012545 - 0.01245) \times 4.3694 \times N = 0.0001 \times 4.3694 \times 10{,}000{,}000 = \$4{,}369$$

For a par trade ($S_t=S_0$), this is consistent with the near-par scaling $\text{CS01} \approx N \cdot \text{RPV01}\cdot 1\text{bp}$.

### Example 43.3: VOD and JTD

**Given:** Contract spread $S_0 = 124.5$ bp, $R = 40\%$, default occurs $\Delta_0 = 0.25$ years after last premium.

**VOD for protection buyer (per the formula in Section 43.2.4):**

$$\text{VOD} = -V(t) + (1-R) - \Delta_0 S_0$$

For a par contract ($V(t) = 0$):

$$\text{VOD} = (1 - 0.40) - 0.25 \times 0.01245 = 0.60 - 0.00311 = 0.5969$$

**Dollar VOD:**

$$\text{VOD} \times N = 0.5969 \times 10{,}000{,}000 = \$5{,}968{,}875$$

**For protection seller:**

$$\text{VOD}_{\text{seller}} = -\$5{,}968{,}875$$

This is the seller's loss upon immediate default—nearly \$6mm on a \$10mm notional position. This “event jump” is why CS01-hedged is not the same as default-hedged.

### Example 43.4: Recovery Sensitivity

**Holding survival fixed, reprice at R = 20\%, 40\%, 60\%.**

**From Example 43.1:** $\sum Z_i d_i = 0.0906$ (the "default annuity")

| Recovery | $(1-R)$ | $\text{PV}_{\text{prot}}$ | $V_{\text{buyer}}$ |
|----------|---------|---------------------------|---------------------|
| 20\% | 0.80 | 0.0725 | +\$181,273 |
| 40\% | 0.60 | 0.0544 | \$0 (par) |
| 60\% | 0.40 | 0.0362 | -\$181,273 |

**Rec01:**

$$\text{Rec01} = \frac{-181{,}273 - 181{,}273}{40\%} \times 1\% = -\$9{,}064 \text{ per +1\% recovery}$$

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
- 1Y CS01 per \$1mm = $1,000,000 × 0.95 × 0.0001 = $95/bp
- 5Y CS01 per \$1mm = $1,000,000 × 3.80 × 0.0001 = $380/bp
- Ratio: $380/$95 = 4.0

**Step 2: Structure the trade**
- Buy \$40mm 1Y protection (long front end)
- Sell \$10mm 5Y protection (short back end)

**Step 3: Verify DV01 neutrality**
- 1Y leg CS01: +$40mm × $95/bp per $1mm = +$3,800/bp
- 5Y leg CS01: -$10mm × $380/bp per $1mm = -$3,800/bp
- **Net parallel CS01: \$0**

**Scenario A: Survival and Curve Normalization**
*Curve un-inverts:* 1Y → 200bp (-600bp), 5Y → 250bp (-250bp)

| Leg | Notional | Spread Move | P&L |
|-----|----------|-------------|-----|
| 1Y long prot | $40mm | -600bp | -$2,280,000 |
| 5Y short prot | $10mm | -250bp | +$950,000 |
| **Net** | | | **-\$1,330,000** |

Wait—this is a loss? Yes, but compare to outright short protection:
- If you had sold $10mm 5Y protection outright: P&L = +$950,000
- But you also would have been short JTD risk of \$6mm

**The steepener loses less on survival but also had less JTD exposure.** Let's examine default:

**Scenario B: Default at 40\% Recovery**

| Leg | Notional | Protection Payment | Net |
|-----|----------|-------------------|-----|
| 1Y long prot | \$40mm | +$24mm (you receive) | +$24mm |
| 5Y short prot | \$10mm | -$6mm (you pay) | -$6mm |
| **Net** | | | **+\$18mm** |

**Key Insight:** The steepener is **long JTD** because you have more protection bought than sold. This is a convex trade:
- If company survives: Lose on curve normalization, but gain on carry (you receive net spread)
- If company defaults: Large gain from net long protection

**Actual Steepener P&L Profile:**
| Outcome | Steepener | Outright Short 5Y Prot |
|---------|-----------|------------------------|
| Survival + curve normalizes | -$1.33mm | +$0.95mm |
| Default (R=40\%) | +$18mm | -$6mm |

The steepener is betting on survival with a safety net—if wrong about survival, you still win big on default.

### Example 43.7: CS01 Hedge vs JTD Exposure

**Goal:** Hedge 5y protection buyer with 3y short protection.

**RPV01s:** $\text{RPV01}_{5y} = 4.37$, $\text{RPV01}_{3y} = 2.76$

**Hedge notional (parallel CS01 neutral, using an RPV01 ratio):**

$$N_{3y} = 10\text{mm} \times \frac{4.37}{2.76} = 15.83\text{mm}$$

**Scenario: Curve steepening (5y +5bp, 3y -5bp)**

| Position | $\Delta S$ | $\Delta V$ |
|----------|-----------|------------|
| 5y long prot (\$10mm) | +5bp | +\$21,847 |
| 3y short prot (\$15.83mm) | -5bp | +\$21,847 |
| **Net** | | **+\$43,694** |

The hedge was CS01-neutral to parallel moves but has **positive P&L on steepening**—residual curve risk. A parallel CS01 hedge does not, by itself, immunize curve twists.

### Example 43.8: Full Risk Report

**Position:** 5y protection buyer, \$10mm notional, contract at par.

| Risk Measure | Value |
|--------------|-------|
| **CS01 (parallel)** | +\$4,369/bp |
| **CS01 (0-1y bucket)** | +\$576/bp hazard |
| **CS01 (1-3y bucket)** | +\$1,067/bp hazard |
| **CS01 (3-5y bucket)** | +\$963/bp hazard |
| **JTD (R=40\%)** | +\$5,969k |
| **JTD (R=20\%)** | +\$7,969k |
| **Rec01** | -\$9,064 per +1\% R |

**Stress scenarios:**

| Scenario | P&L |
|----------|-----|
| Spread +100bp parallel | +\$437k |
| Spread +500bp parallel | +\$1,850k (convexity reduces gain) |
| Default at R=40\% | +\$5,969k |
| Default at R=20\% | +\$7,969k |
| Recovery +10\% (no default) | -\$91k |

### Example 43.9: Bond-CDS Hedge Basis Risk

**Setup:** Own \$10mm corporate bond at price 102, buy CDS protection.

**JTD hedge sizing:**
- Bond loss at default (R=40\%): $(102-40)/100 \times 10\text{mm} = \$6.2\text{mm}$
- CDS notional needed: $6.2\text{mm} / 0.6 = \$10.33\text{mm}$

**Scenario: Basis widens (bond spread +80bp, CDS +40bp)**

| Position | Sensitivity | Move | P&L |
|----------|-------------|------|-----|
| Bond | Duration 4.5 | +80bp | -\$367k |
| CDS | RPV01 4.37 | +40bp | +\$181k |
| **Net** | | | **-\$186k** |

Despite being "default hedged," the basis move creates significant loss: bond and CDS spreads can move differently due to funding/liquidity and instrument/settlement differences.

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
| **Residual** | | +\$765 (1.0\%) |

Residual within acceptable tolerance.

---

## Summary

- A CDS quote maps to two legs: premium cashflows (until default/maturity) and a default-contingent protection payment. PV is the net of the two.
- `RPV01` is the risky premium-leg annuity; it scales how spread differences translate into PV.
- `CS01` is a bump-based spread sensitivity. Always state the bump object (par quotes vs hazard), bump size (1 bp $=10^{-4}$), and whether the survival curve is rebuilt.
- Default risk is discontinuous: `VOD`/JTD is the PV jump at default and can be orders of magnitude larger than daily CS01 P&L.
- Recovery affects PV directly (default payoff) and indirectly (through curve calibration if you hold spreads fixed). `Rec01` is only meaningful once you specify what is held fixed.
- Parallel CS01 can miss curve twists; bucket CS01s and “equivalent notionals” are the credit analogue of key-rate DV01 hedging.
- Cross-instrument hedges (bond/CDS/index) introduce basis and proxy risk; validate hedges with stress scenarios (spread, default, recovery, basis).

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CS01 vs JTD | Spread risk (continuous) vs default risk (binary) | Different hedges needed; CS01 hedge ≠ JTD hedge |
| CS01 | PV change per 1bp spread move | Primary day-to-day risk metric |
| RPV01 | Risky premium-leg annuity | Links spread to value; duration analog |
| JTD/VOD | Value change on default | Discontinuous risk; dominates for distressed |
| Inverted credit curve | Front-end spread > back-end spread | Signals imminent distress |
| Credit curve steepener | Long front, short back (DV01-neutral) | Bets on survival; long JTD |
| Credit curve flattener | Short front, long back (DV01-neutral) | Bets on distress; short JTD |
| Rec01 | PV change per 1\% recovery | Often overlooked; material uncertainty |
| Bucket CS01 | Sensitivity to curve points | Captures term structure risk |
| Spread gamma | Second derivative to spread | Matters for large moves |
| Cash-CDS basis | $S_{\text{CDS}}-s_{\text{bond}}$ under a specified bond spread measure | Residual in bond-CDS hedges |
| Equivalent notionals | Hedge notionals that neutralize bucket CS01s | First-order curve hedging tool |

---

## Notation

| Symbol | Meaning | Units / Notes |
|---|---|---|
| $N$ | CDS notional | currency |
| $S_0$ | contractual running spread | decimal per year (e.g., 150 bp = 0.015) |
| $S(t,T)$ | par spread for maturity $T$ | decimal per year |
| $\Delta_i$ | accrual year fraction for period $i$ | years (ACT/360 unless stated) |
| $\tau$ | default time | time/date |
| $R$, $LGD$ | recovery, loss-given-default $=1-R$ | fraction of par |
| $Z(t,u)$ | discount factor | unitless |
| $Q(t,u)$ | survival probability $\mathbb{P}(\tau>u\mid\mathcal{F}_t)$ | unitless |
| `RPV01` | risky premium-leg annuity | years |
| `CS01` | $PV(S+1\text{bp})-PV(S)$ under stated bump design | currency per bp |
| `VOD` | PV jump under immediate default (including accrued premium) | currency |
| `Rec01` | $PV(R+1\%)-PV(R)$ under stated “hold fixed” assumptions | currency per +1\% $R$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is RPV01? | The risky premium-leg annuity: present value of receiving 1bp of spread until maturity or default |
| 2 | Write the CDS MTM identity (per unit notional) | $V(t) = (S(t,T) - S_0) \times \text{RPV01}(t,T)$ |
| 3 | What is CS01? | PV change for a 1bp change in specified credit spread input |
| 4 | What is the key difference between CS01 and JTD risk? | CS01 is continuous (daily spread volatility); JTD is binary (default event)—a CS01 hedge does NOT protect against JTD |
| 5 | Why must bump method be specified for CS01? | Different bumps (par spread vs hazard) produce different sensitivities |
| 6 | What is VOD for a protection buyer (per unit notional)? | $\text{VOD} = -V(t) + (1-R) - \Delta_0 S_0$ |
| 7 | How does accrued premium affect VOD? | Reduces buyer's net receipt by $\Delta_0 S_0 N$ |
| 8 | What does an inverted credit curve signal? | Imminent distress—market fears near-term default (front-end spreads > back-end) |
| 9 | What does an upward-sloping credit curve often indicate? | Lower perceived near-term default risk than long-horizon cumulative risk |
| 10 | What is a credit curve steepener? | Long front-end protection, short back-end protection (DV01-neutral); bets on survival |
| 11 | What is a credit curve flattener? | Short front-end protection, long back-end protection (DV01-neutral); bets on distress |
| 12 | If spreads widen, what happens to protection buyer's PV? | Increases (positive CS01) |
| 13 | What is Rec01? | PV change per +1\% recovery assumption |
| 14 | Why is recovery sensitivity complex? | Changing R affects implied hazard if spreads held fixed (credit triangle) |
| 15 | What is curve-shape credit risk? | Sensitivity to non-parallel spread movements |
| 16 | What does “equivalent hedge notional” mean? | A hedge size chosen so bucket CS01s net close to zero: $s_{\text{net}} = s + \\sum_j n_j h^{(j)} \approx 0$ |
| 17 | What is cash–CDS basis? | $S_{\text{CDS}} - s_{\text{bond}}$ under a specified bond spread measure |
| 18 | Name two basis drivers | Funding/carry differences and liquidity/shorting constraints (also: settlement/contract differences) |
| 19 | Why can bond+CDS hedge leave residual P&L? | Basis moves independently due to fundamental and market factors |
| 20 | What is spread convexity? | Second derivative of V to S; nonlinearity for large moves |
| 21 | For protection buyer, is spread gamma usually positive or negative? | Usually negative (marginal PV gain shrinks as spreads widen) |
| 22 | What is JTD in a P&L explain? | PV jump from immediate default scenario |
| 23 | Why does JTD matter more than CS01 for distressed names? | Default probability is high; CS01 captures small moves only |
| 24 | Who can have large counterparty exposure in stress? | The protection buyer (it is owed a large payment when the reference is distressed/defaulting) |
| 25 | What is the “Friday Night Downgrade” lesson? | A CS01 hedge can look tight day-to-day but still leave large name-default exposure (JTD) |

---

## Mini Problem Set

### Problems

1. A 5y CDS has `RPV01` = 4.2 years. Notional \$20mm. Compute the (approximate) CS01 for a protection buyer.
2. Compute `VOD` for a protection buyer with $R = 35\%$, $S_0 = 150$ bp, $\Delta_0 = 0.25$, $N = \$5\text{mm}$, and $V(t)=0$.
3. If `Rec01` = $-\$12{,}000$ per +1\% recovery, what is PV change for recovery falling from 40\% to 30\% (holding everything else fixed)?
4. Explain why a 5y CDS has exposure to 1y hazard rates even if you “only traded the 5y point”.
5. A bond trades at 95 (dirty price per 100 of face). Recovery assumed 40\%. Size CDS notional $N_{\text{CDS}}$ to hedge the bond’s default loss (first-order, ignore coupons).
6. Spreads widen 50 bp. Initial CS01 = \$5,000/bp. Why might actual P&L be less than \$250,000?
7. Write the 2×2 linear system to hedge a 5y bucket-CS01 exposure using 3y and 7y CDS instruments.
8. Bond spread widens 60 bp, CDS widens 30 bp. Bond duration 5, bond notional \$10mm. CDS `RPV01` = 4.5, CDS notional \$10mm. Compute the approximate net MTM P&L (ignore carry).
9. Why can an index hedge reduce systematic risk while leaving large idiosyncratic risk?
10. Describe a stress scenario where a CS01-neutral book has significant P&L.

### Solution Sketches (Selected)

1. $ \text{CS01} \approx 20{,}000{,}000 \times 4.2 \times 10^{-4} = \$8{,}400/\text{bp}. $

2. Per unit notional, $ \text{VOD} = (1-R) - \Delta_0 S_0 = 0.65 - 0.25\times 0.015 = 0.64625 $.  
   Multiply by \$5mm: $0.64625 \times 5{,}000{,}000 = \$3{,}231{,}250$ (about \$3.23mm).

5. Bond loss fraction $\approx P/100 - R = 0.95-0.40=0.55$. CDS payout fraction $=1-R=0.60$.  
   $N_{\text{CDS}} \approx 0.55/0.60 \times N_{\text{bond}} \approx 0.917\,N_{\text{bond}}.$

4. A CDS PV depends on survival to each future premium date; bumping near-term hazard changes survival probabilities for *all* later dates, so it affects both the premium and protection legs of a 5y trade.

6. Because `RPV01` (and therefore CS01) typically compresses as spreads widen; the PV–spread relationship is nonlinear (negative spread gamma for long protection).

---

## References

- Dominic O’Kane, *Modelling Single-name and Multi-name Credit Derivatives* (CS01/Credit DV01, VOD/JTD, recovery sensitivity, curve hedging, spread convexity).
- Nicolas Privault, *Notes on Financial Risk and Analytics* (“Net Present Value (NPV) of a CDS”: premium and protection leg PV expressions).
- John C. Hull, *Risk Management and Financial Institutions* (credit spread risk vs jump-to-default risk; CDS–bond basis).
- John C. Hull, *Options, Futures, and Other Derivatives* (credit default swaps and bond yields; basis intuition).
- Stéphane Crépey and Tomasz R. Bielecki, *Counterparty Risk and Funding: A Tale of Two Puzzles* (spread risk vs jump-to-default risk discussion).
