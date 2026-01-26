# Chapter 15: DV01 Hedging — Hedge Ratios, Risk Weights, and Practical Caveats

---

## Introduction

You have calculated your portfolio's DV01. It shows $500,000 per basis point—meaning a mere 10 bp move in rates produces a $5 million P&L swing. The natural question becomes: *how do you neutralize that exposure?*

The answer seems straightforward: find an offsetting instrument and trade it in the right size. But "the right size" depends on a formula that encodes a critical assumption—that rates move together in a specific way. When that assumption fails, so does the hedge. A trader who is "DV01 flat" can still lose substantial money if the curve twists, if basis relationships shift, or if large moves reveal convexity mismatches.

This concept—matching sensitivities to achieve neutrality—has deep roots. Luenberger calls it **immunization**: structuring a portfolio "to protect against interest rate risk," a technique that is "one of the most (if not the most) widely used analytical techniques of investment science." What began as duration matching for pension fund liabilities has evolved into the sophisticated DV01 and key-rate frameworks used on modern trading desks. Yet the fundamental principle remains unchanged: match the sensitivities, and small rate changes produce zero P&L.

This chapter develops the mechanics and limitations of DV01 hedging. We begin with the fundamental hedge ratio formula and work through its application across bonds, swaps, and futures. We then confront the uncomfortable truth: DV01 neutrality is a first-order, local approximation that breaks down in multiple ways. Understanding *when* and *why* DV01 hedges fail is as important as knowing how to construct them. As Tuckman emphasizes, effective hedging requires not just neutralizing the aggregate DV01, but managing the residual risks that remain after the primary hedge is in place.

In this chapter, we cover:

1.  **The fundamental hedge ratio** — deriving the core formula for neutralizing linear price risk
2.  **Instrument-specific mechanics** — how to hedge with swaps (using PV01) and futures (using CTD conversion factors)
3.  **Risk-weighted hedging** — optimizing hedges for variance minimization rather than simple DV01 matching
4.  **Failure modes** — why "DV01 flat" portfolios still lose money (twist, convexity, basis, spread risk)
5.  **Practical diagnostics** — how to verify hedges and attribute residuals

The chapter connects backward to the DV01 and duration concepts of Chapters 11-12 and forward to the key-rate framework of Chapter 16, which addresses the twist risk that single-number DV01 cannot capture.

---

## 15.1 DV01 Revisited: Definition and Units

Before constructing hedges, we must be precise about what DV01 measures and how it is quoted. Tuckman defines DV01 as the change in price for a one basis point change in yield:

$$\boxed{\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}}$$

where $\Delta y$ is a yield change in decimal form, so $10{,}000 \Delta y$ converts the change to basis points. The minus sign represents the convention that DV01 is a positive number for standard fixed-rate bonds (where price falls as yield rises).

Tuckman illustrates this with a concrete example: for the 5s of February 15, 2011, "the DV01 of the option with rates at 5% is .0369, while the DV01 of the bond is .0779." These numbers are per 100 face value—a critical point for position sizing.

The connection to modified duration follows directly. Since modified duration ($D_{\text{mod}}$) gives the percentage price sensitivity ($\Delta P/P \approx -D_{\text{mod}} \Delta y$), multiplying both sides by the price $P$ and adjusting for the basis point scaling yields:

$$\boxed{\text{DV01} = \frac{P \times D_{\text{mod}}}{10{,}000}}$$

This relationship, emphasized by Tuckman in Chapters 5-6, makes clear that DV01 combines price level and duration into a single dollar sensitivity measure. It is the "dollar risk" of the position.

### 15.1.1 Unit Conventions and Traps

DV01 is typically quoted "per 100 face value per 1 bp." This means that for a bond with DV01 = 0.08, a 1 bp yield increase causes the price to fall by approximately $0.08 per $100 of face value. To convert to position-level dollar exposure ($\text{DV01}^{\$}$), one must scale by the position size:

$$\text{DV01}^{\$} = F \times \frac{\text{DV01 (per 100)}}{100}$$

where $F$ is the face value of the position.

**Example:** A $10 million face position in a bond with DV01 = 0.08 per 100:

$$\text{DV01}^{\$} = 10{,}000{,}000 \times \frac{0.08}{100} = \$8{,}000 \text{ per bp}$$

The factor-of-100 unit conversion is a notorious source of errors. Traders who confuse "per 100 face" with "per $1 face" will miscalculate hedge sizes by a factor of 100—a catastrophic mistake in practice.

---

## 15.2 The DV01 Hedge Ratio

### 15.2.1 Derivation

Consider a position in bond A with face value $F_A$ and per-100 DV01 of $\text{DV01}_A$. We wish to hedge this position using bond B with per-100 DV01 of $\text{DV01}_B$. For the combined portfolio to be DV01-neutral, the total dollar sensitivity must be zero:

$$\text{DV01}_A^{\$} + \text{DV01}_B^{\$} = 0$$

Substituting the position-level DV01 expressions:

$$F_A \times \frac{\text{DV01}_A}{100} + F_B \times \frac{\text{DV01}_B}{100} = 0$$

Solving for the hedge face value $F_B$ gives Tuckman's fundamental hedge formula:

$$\boxed{F_{B}=\frac{-F_{A} \times \text{DV01}_{A}}{\text{DV01}_{B}}}$$

This is equation (5.7) in Tuckman. The negative sign indicates that if we are long bond A (positive $F_A$), we must short bond B (negative $F_B$) to offset the duration exposure—provided both DV01s are positive, which they are for conventional fixed-rate bonds.

Tuckman notes an important caveat: "There are occasions in which one DV01 is negative. In these cases equation (5.7) shows that a hedged position consists of simultaneous longs or shorts in both securities." This arises with instruments like interest-only strips or certain mortgage derivatives.

### 15.2.2 What "DV01-Neutral" Actually Means

A portfolio is "DV01-neutral" if its first-order PV change is approximately zero under a specified yield bump. However, Tuckman emphasizes a crucial limitation: **yield-based DV01 hedging works only if the yield on the purchased security and the yield on the hedging security change by the same amount.**

This "parallel yield shift" assumption is the Achilles' heel of simple duration hedging. As Luenberger observes, a portfolio can be "immunized only against parallel shifts in the spot rate curve." A DV01-hedged book provides "low net parallel-shift risk"—nothing more. It does *not* protect against:
- **Curve twists:** When short-term and long-term rates move differently.
- **Large moves:** Where convexity dominates the linear DV01 approximation.
- **Basis divergence:** When the hedge and position are driven by different curves (e.g., Treasury vs. Swap).
- **Basis divergence:** When the hedge and position are driven by different curves (e.g., Treasury vs. Swap).
- **Spread risk:** When credit spreads widen while risk-free rates remain static.

> **Analogy: The Dirty Hedge**
>
> Ideally, a hedge is a mirror image: you lose $10 here, you make $10 there.
> Real life hedges are "dirty." You are hedging a corporate bond with a Treasury.
> *   **Treasury moves**: You are protected.
> *   **Spread moves**: You are naked.
> *   **Result**: You eliminate the *big* risk (rates) but keep the *annoying* risk (basis/tracking error). Perfect hedges only exist in textbooks.

### 15.2.3 Worked Example: Bond-Bond Hedge

This example follows Tuckman's approach in Chapter 5.

**Scenario:** A market maker sells $100 million face value of a call option and must hedge with the underlying bond.

From Tuckman's Table 5.1:
- **Option:** DV01 = 0.0369 per 100 at 5% rates
- **Bond (5s of Feb 15, 2011):** DV01 = 0.0779 per 100 at 5% rates

**Step 1: Compute Required Hedge**

Applying the hedge ratio formula:
$$F_B = 100{,}000{,}000 \times \frac{0.0369}{0.0779} = \$47{,}370{,}000$$

**Step 2: Verify Hedge**

The dollar DV01 of the bond hedge equals:
$$\$47{,}370{,}000 \times \frac{0.0779}{100} = \$36{,}901 \text{ per bp}$$

This matches the option's dollar DV01 of $\$100{,}000{,}000 \times 0.0369/100 = \$36{,}900$ per bp. The market maker is now DV01-neutral for small rate changes.

**What Tuckman Warns:** This hedge works only at the initial rate level. As rates move, the option's DV01 changes differently than the bond's DV01 (due to different convexities), requiring rebalancing.

---

## 15.3 Hedging with Swaps and Futures

### 15.3.1 Swap PV01 and Bond-Swap Hedges

In the swap market, the risk measure typically used is **PV01** (or simply "Risk"), defined as the change in swap value for a 1 bp change in the fixed rate. Tuckman notes that "PV01 normally meant to mean the change in the value of a swap for a one-basis point change in its fixed rate. Writing a bond pricing equation in terms of discount factors reveals that PV01 equals the sum of the discount factors used to discount the fixed cash flows."

For a plain-vanilla swap:

$$\boxed{\text{PV01}_{\text{swap}} = N \times \sum_{i} \tau_i P(0, t_i) \times 0.0001}$$

where $N$ is the notional amount, $\tau_i$ is the accrual fraction for period $i$, and $P(0, t_i)$ is the discount factor. The sum $A = \sum_i \tau_i P(0, t_i)$ is the **annuity factor**.

**Sign Convention:** To hedge a long bond (which loses money when rates rise), you need a position that *gains* money when rates rise. A **payer swap** (pay fixed, receive floating) gains value when rates rise, providing negative duration to offset the bond.

**Worked Example:** Hedge a $50 million bond position (DV01 = 0.08 per 100) with a 5-year swap.

*   **Bond Risk:** $\text{DV01}^{\$} = 50{,}000{,}000 \times (0.08/100) = \$40{,}000 \text{ per bp}$.
*   **Swap Annuity:** Sum of discount factors is 4.65.
*   **Swap Risk per \$1mm:** $\$1{,}000{,}000 \times 4.65 \times 0.0001 = \$465 \text{ per bp}$.
*   **Hedge Notional:** $N = 40{,}000 / 465 \approx \$86.0 \text{ million}$.
*   **Action:** Pay fixed on $86 million notional.

### 15.3.2 Futures Hedge Ratio

Futures hedging introduces an extra layer of complexity: the **Conversion Factor (CF)** and the **Cheapest-to-Deliver (CTD)** bond. Since Treasury futures track a virtual "6% coupon bond" (or similar standard), physical bonds are deliverable through a conversion factor system.

Hull presents the **duration-based hedge ratio** for futures in equation (6.3):

$$\boxed{N^* = \frac{P \times D_P}{V_F \times D_F}}$$

where $P$ is the portfolio value, $D_P$ is its duration, $V_F$ is the futures contract value, and $D_F$ is the duration of the bond underlying the futures (typically the CTD).

In practice, traders use the DV01 of the underlying CTD bond directly. The relationship is approximately:

$$\text{DV01}_{\text{fut}} \approx \frac{\text{DV01}_{\text{CTD}}}{\text{CF}}$$

This implies that one futures contract acts like $1/\text{CF}$ units of the CTD bond in terms of duration exposure.

**Worked Example:** Hedge a $25,000 position DV01 with Treasury futures.

*   **CTD Stats:** DV01 = 0.0750 per 100; CF = 0.90.
*   **Contract Size:** $100,000 face.
*   **CTD Risk per Contract:** $100{,}000 \times (0.0750/100) = \$75.00$.
*   **Futures Risk per Contract:** $75.00 / 0.90 = \$83.33$.
*   **Hedge Size:** $25{,}000 / 83.33 \approx 300$ contracts (Short).

**Practical Caveat:** Tuckman emphasizes that the CTD bond can change. If the yield curve shifts significantly, a different bond may become cheapest-to-deliver, discontinuously changing the duration of the futures contract. This "CTD switch option" creates hedge slippage that simple linear formulas cannot capture.

---

## 15.4 Risk-Weighted Hedging and Variance Minimization

### 15.4.1 Beyond Simple DV01 Neutrality

The standard hedge ratio ($h = \text{DV01}_A / \text{DV01}_B$) assumes that yield changes are identical ($\Delta y_A = \Delta y_B$). In reality, yields along the curve are imperfectly correlated and have different volatilities. A 30-year yield might move 0.8 bp for every 1.0 bp move in 10-year yields.

Tuckman addresses this in Chapter 8 using **regression-based hedging**. He notes: "The simplest way to estimate the volatility ratio is to compute the two volatilities from recent data." Using data on 20-year and 30-year yields from January 1995 to September 2001, he finds volatilities of 5.27 and 4.94 basis points per day respectively, giving a ratio of about 1.066.

Instead of assuming $\Delta y_A = \Delta y_B$, we model the statistical relationship:

$$\Delta y_{\text{target}} = \alpha + \beta \Delta y_{\text{hedge}} + \epsilon$$

where $\beta$ represents the **yield beta** or regression coefficient. The variance-minimizing hedge ratio becomes:

$$\boxed{h^* = \beta \times \frac{\text{DV01}_{\text{target}}}{\text{DV01}_{\text{hedge}}}}$$

This matches Hull's minimum variance hedge formula (equation 3.1): $h^* = \rho (\sigma_S / \sigma_F)$, where $\rho$ is correlation and $\sigma$ represents volatility. Tuckman explicitly makes this connection: "The risk weight [from regression] equals the ratio of volatilities... while the regression approach recognizes the imperfect correlation between changes in the two yields."

> **Advanced Concept: Regression Hedging**
>
> Don't just match DV01s ($1:1$ ratio). Run a regression: $\Delta \text{Yield}_{Bond} = \alpha + \beta \times \Delta \text{Yield}_{Hedge}$.
> *   **The Beta ($\beta$)** is your true Hedge Ratio.
> *   If 30-year yields move 1.2x as much as 10-year yields, your hedge ratio isn't 1.0, it's 1.2.
> *   **Result**: You are hedging *variance*, not just face value.

### 15.4.2 Two-Bond Hedges and Risk Weights

Tuckman presents a compelling case for two-variable regression hedging. When hedging a 20-year bond, using only 30-year bonds leaves substantial residual variance. Adding 10-year bonds to the hedge dramatically improves performance.

The regression model becomes:

$$\Delta y_{20} = \alpha + \beta_{10} \Delta y_{10} + \beta_{30} \Delta y_{30} + \epsilon$$

From Tuckman's Table 8.2, the estimated coefficients are:
- $\beta_{10} = 0.1613$ (16.1% risk weight on 10-year)
- $\beta_{30} = 0.8774$ (87.7% risk weight on 30-year)

These coefficients determine the hedge allocation. Tuckman notes: "In the one-variable regression... the 30-year risk weight is 1.057 or 105.7%. In the two-factor regression the risk weight on the 30-year falls to 87.7% because some of the DV01 risk is transferred to the 10-year."

**The R-squared insight:** In the one-factor case, "98.25% of the variance of changes in the 20-year yield can be explained by changes in the 30-year yield." Adding the 10-year increases this only marginally, but the residual variance (standard error of regression) drops meaningfully.

### 15.4.3 Worked Example: Regression-Based Hedge

**Setup:** Hedge a $10 million face position in 20-year bonds using 30-year bonds.

From Tuckman's example:
- **20-year DV01:** 0.118428 per 100
- **30-year DV01:** 0.142940 per 100
- **Regression beta:** 1.057

**Simple DV01 Hedge:**
$$F_{30} = -10{,}000{,}000 \times \frac{0.118428}{0.142940} = -\$8{,}286{,}000$$

**Regression-Adjusted Hedge:**
$$F_{30} = -10{,}000{,}000 \times 1.057 \times \frac{0.118428}{0.142940} = -\$8{,}758{,}000$$

The regression approach requires shorting an additional $472,000 face of 30-year bonds because historical data shows 20-year yields are slightly more volatile than 30-year yields.

**Residual Risk:** Tuckman calculates that with the regression hedge, "the hedged $10 million face position... would be subject to a daily one-standard-deviation profit or loss of $8,258." This represents the unavoidable basis risk.

---

## 15.5 When DV01 Hedges Fail

DV01 hedging is a first-order local approximation. It works perfectly only for infinitesimal parallel shifts. In the real world, "DV01 flat" portfolios can and do hemorrhage cash. The primary failure modes are:

### 15.5.1 Curve Twist (Non-Parallel Moves)

DV01 aggregates risk into a single number, masking exposure to curve shape. A portfolio long 30-year bonds and short 2-year bonds could have zero net DV01. However, if the curve steepens (30y yields rise, 2y yields fall), both positions lose money.

Tuckman's analysis of the 20-year sector illustrates this vividly. He constructs an index $I = \beta_{10} y_{10} + \beta_{30} y_{30} - y_{20}$ that tracks relative value. When risk weights are chosen poorly (e.g., equal weights), the index becomes highly correlated with the 10s-30s curve slope, meaning "the trade will exhibit a P&L profile similar to that of a simple curve trade." The regression-derived weights, by contrast, "adequately hedge against curve risk."

This **curve risk** is the primary motivation for the **Key Rate DV01** framework discussed in Chapter 14.

### 15.5.2 Convexity Mismatch

DV01 is linear. Convexity is quadratic. The Taylor series expansion shows the P&L impact:

$$\frac{\Delta P}{P} \approx -D \Delta y + \frac{1}{2} C (\Delta y)^2$$

Tuckman provides a striking example with callable bonds. In "A Hedging Example, Part III," he shows that mixing a positively convex bond with a negatively convex callable bond creates a hedge that is "quite unstable away from 5%. Not only do the values of the two securities increase or decrease away from 5% at different rates... but in Figure 5.9 the values are driven even further apart by opposite curvatures."

For large moves (+/- 50 bp), the convexity term $\frac{1}{2}C(\Delta y)^2$ generates profit for the high-convexity holder regardless of direction. A DV01-neutral hedger who is short convexity will suffer "gamma bleed" in volatile markets.

### 15.5.3 Basis Risk

Basis risk arises when the hedge instrument and the hedged position are driven by different curves. While they may be correlated, they are not identical. Common disconnects include:

*   **Treasury vs. Swap:** Hedging a corporate bond (priced off swaps) with Treasuries exposes you to swap spread changes.
*   **Discount vs. Projection:** In the post-crisis multi-curve world, a hedge might match the projection curve risk (e.g., SOFR) but leave discount curve risk (OIS) open.
*   **On-the-Run vs. Off-the-Run:** Liquidity premiums can cause On-the-Run Treasuries to yield significantly less than Off-the-Run counterparts. A hedge across this boundary carries "liquidity risk" disguised as basis risk.

> **Warning: The Basis Trap**
>
> A "perfect" DV01 hedge can still bankrupt you.
> *   **Trade**: Long Corporate Bond, Short Treasury.
> *   **DV01**: Net Zero.
> *   **Scenario**: Financial Crisis. Treasuries rally (yields down), Corporates crash (yields up).
> *   **Result**: You lose on the Corporate (spreads widen). You lose on the Treasury short (rates rallied). You get hit by a bus from both directions. This is **Basis Risk**.

### 15.5.4 Spread and Credit Risk

A bond's yield $y$ is the sum of the risk-free rate $y_T$ and a credit spread $s$:

$$y = y_T + s$$

A rates hedge neutralizes $y_T$ but does nothing for $s$. The sensitivity to spread changes is called **CS01** (Credit Spread 01):

$$\text{CS01} = \frac{P \times D_{\text{spread}}}{10{,}000}$$

A trader who hedges a corporate bond with Treasuries is "Rate DV01 flat" but "Spread DV01 long." If spreads widen, the corporate bond price falls, and the Treasury hedge provides no offset.

---

## 15.6 Additional Worked Examples

### Example E: DV01-Neutral but Twist Loss

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

### Example F: Convexity Mismatch Under Large Move

This example follows the logic of Tuckman's callable bond hedging discussion.

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

---

## 15.7 Practical Notes

### Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Sign Convention Confusion** | State your convention explicitly. This book uses DV01 > 0 for long bonds. Ensure your hedge (short bond or payer swap) has negative DV01. |
| **Unit Mistakes** | "Per 100 face" vs "per $1 face" is a factor of 100. Always check if the DV01 is ~0.08 (per 100) or ~0.0008 (per 1). |
| **Mixing Methodologies** | Bumping par rates vs. shifting zero curves can produce slightly different DV01s. Stick to one consistency (e.g., "bump-and-rebuild"). |
| **Ignoring Curve Shape** | DV01-neutral is not risk-free. Always check key-rate exposures (Chapter 14) for non-parallel risk. |
| **Liquidity vs. Basis** | Hedging an illiquid bond with a liquid Treasury creates cash-to-close risk. You may be hedged on paper but unable to monetize the bond's gain if the hedge moves against you. |

### Verification Tests

Practitioners should run specific "stress tests" to validate hedges:
1.  **Parallel Shift:** Verify Net P&L $\approx 0$ for a small (+1 bp) parallel bump.
2.  **Twist Scenario:** Apply a steepening (+10 bp long end, -10 bp short end) to reveal bucket mismatches.
3.  **Large Move:** Apply a +/- 50 bp shock to verify convexity exposure isn't catastrophic.
4.  **Basis Shock:** Move the hedge curve (e.g., Swap) by +5 bp while holding the asset curve (e.g., Treasury) constant to check basis sensitivity.

---

## 15.8 Summary

1.  **DV01** is the primary first-order measure of interest rate risk, defining the dollar P&L per basis point move.
2.  **The Hedge Ratio** $F_B = -F_A (\text{DV01}_A / \text{DV01}_B)$ determines the face amount needed to neutralize this linear risk (Tuckman equation 5.7).
3.  **Immunization**, as Luenberger emphasizes, "immunizes" the portfolio against interest rate changes—but only for the specified type of curve shift.
4.  **Swap Hedges** rely on the PV01 (annuity value); pay-fixed swaps provide negative duration to hedge long bond positions.
5.  **Futures Hedges** require adjusting for the Conversion Factor (CF) of the Cheapest-to-Deliver (CTD) bond.
6.  **Risk Weights** (regression betas) account for the fact that different points on the curve have different volatilities and correlations; simple DV01 matching may under- or over-hedge.
7.  **DV01-Neutrality** is a local approximation. It assumes parallel yield moves and small changes.
8.  **Residual Risks** include curve twists, convexity (gamma), basis risk, and credit spread widening. These are not captured by a simple DV01 sum.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **DV01** | Dollar PV change per 1 bp yield change | The standard unit of risk for trading desks. |
| **Hedge Ratio** | $F_B = -F_A (\text{DV01}_A / \text{DV01}_B)$ | The formula to zero out linear risk (Tuckman 5.7). |
| **Immunization** | Structuring portfolios to protect against rate changes | Foundation of duration hedging (Luenberger). |
| **PV01** | Annuity $\times$ 1 bp $\times$ Notional | The swap market equivalent of DV01 (Tuckman Ch 18). |
| **Risk Weight** | Regression coefficient $\beta$ | Adjusts hedges for relative volatility (Tuckman Ch 8). |
| **Basis Risk** | Imperfect correlation between asset and hedge | The risk that remains even when DV01 is flat. |
| **CS01** | Credit Spread 01 | P&L sensitivity to credit spreads, distinct from rate sensitivity. |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $\text{DV01}$ | Dollar value of a 0.01% (1 bp) yield change |
| $\text{DV01}^{\$}$ | Position-level DV01 (scaled by face amount) |
| $D_{\text{mod}}$ | Modified Duration |
| $F$ | Face Value / Notional |
| $\text{PV01}$ | Swap Risk (Annuity $\times$ 0.0001 $\times$ Notional) |
| $\text{CF}$ | Conversion Factor for futures |
| $\beta$ | Yield beta / Risk weight from regression |
| $\text{CS01}$ | Credit Spread 01 Sensitivity |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the definition of DV01? | $-\Delta P / (10{,}000 \times \Delta y)$; the dollar change in value for a 1 bp parallel yield shift. |
| 2 | How do you convert DV01 "per 100" to position risk? | Multiply by (Position Face / 100). |
| 3 | What is Tuckman's hedge ratio formula (equation 5.7)? | $F_B = -F_A (\text{DV01}_A / \text{DV01}_B)$. |
| 4 | If you own a bond (positive DV01), do you buy or sell futures to hedge? | Sell futures (short position gains when rates rise). |
| 5 | What does a "payer swap" do to your duration profile? | Paying fixed decreases duration (you profit if rates rise), so it hedges a long bond position. |
| 6 | Why might a hedge ratio differ from the simple DV01 ratio? | Because of "Risk Weights" (yield betas from regression) that account for volatility differences. |
| 7 | What does Luenberger call the process of matching duration to protect against rate changes? | Immunization. |
| 8 | What is "Basis Risk"? | The risk that the spread between the hedge instrument and the position changes. |
| 9 | Does a DV01-neutral portfolio have zero risk? | No. It still has curve twist risk, convexity risk, and basis risk. |
| 10 | What is the "CTD" in futures hedging? | Cheapest-to-Deliver bond; the bond that determines the futures contract's effective duration. |
| 11 | What is CS01? | Sensitivity to credit spread changes (distinct from risk-free rate changes). |
| 12 | Why does convexity matter for large moves? | DV01 is linear; convexity adds a quadratic payoff that dominates when yield changes are large. |
| 13 | What is a "Twist" scenario? | When short-term and long-term rates move in opposite directions or by different magnitudes. |
| 14 | What does the R-squared of a regression hedge tell you? | The percentage of target yield variance explained by the hedge instrument's yield changes. |
| 15 | What is the main unit trap in DV01 calculations? | Confusing face value units (per 1 vs per 100). |

---

## Mini Problem Set

*Solution sketches provided for questions 1-7.*

**Problem 1:** A bond has DV01 = 0.072 per 100 and position face $15mm. Compute position DV01 in $/bp.

*Sketch:* $15{,}000{,}000 \times (0.072/100) = \$10{,}800$ per bp.

**Problem 2:** Long $15mm of Bond A (DV01 = 0.072 per 100). Hedge with Bond B (DV01 = 0.060 per 100). Compute hedge face.

*Sketch:* $F_B = -15\text{mm} \times (0.072/0.060) = -18\text{mm}$ (Short $18mm).

**Problem 3:** DV01 is quoted as 0.085 per 100. A trader mistakenly treats it as 0.085 per $1 face. By what factor is the risk overstated?

*Sketch:* Factor = 100.

**Problem 4:** Portfolio value $120mm, duration 6.5. Futures contract value $110k, CTD duration 7.8. Estimate contracts using Hull's formula.

*Sketch:* $N^* = (120{,}000{,}000 \times 6.5) / (110{,}000 \times 7.8) \approx 909$ contracts.

**Problem 5:** 5y annual swap with discount factors 0.99, 0.97, 0.95, 0.93, 0.91. Compute PV01 per $1mm notional.

*Sketch:* Annuity = $0.99+0.97+\dots+0.91 = 4.75$. PV01 = $1\text{mm} \times 4.75 \times 1\text{bp} = \$475$.

**Problem 6:** A DV01-neutral bond hedge loses money when 2y yields +20bp and 10y yields -10bp. What is the most likely explanation?

*Sketch:* Curve twist / key-rate mismatch. The portfolio was not twist-hedged.

**Problem 7:** A corporate bond is DV01-hedged with Treasuries. Credit spreads widen 30bp with no rate change. What risk measure explains the loss?

*Sketch:* CS01 (Credit Spread 01).

**Problem 8:** Explain why a DV01-neutral hedge between a callable and noncallable bond can be unstable as yields change.

**Problem 9:** Give two reasons Treasury futures DV01 per contract can change even without rolling.

**Problem 10:** Describe a bump-definition mismatch that could cause a "DV01-hedged" book to show residual PV change.

**Problem 11:** If your hedge instrument is much less liquid than your position, what additional risk beyond DV01 arises?

**Problem 12:** For a swap in a multi-curve framework, list which curves might be bumped for "DV01" and why that choice matters.

**Problem 13:** In two-hedge variance minimization, why might the optimizer suggest a small long in one hedge and large short in the other?

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| DV01 definition: $-\Delta P / (10{,}000 \times \Delta y)$ | Tuckman Ch 5 |
| Bond-bond hedge ratio formula: $F_B = -F_A (\text{DV01}_A / \text{DV01}_B)$ | Tuckman (5.7) |
| DV01 of option = 0.0369, bond = 0.0779 at 5% rates | Tuckman Table 5.1 |
| Yield-based DV01 limitation: "yields must move together" | Tuckman Ch 5 |
| Immunization terminology and importance | Luenberger Ch 3 ("one of the most widely used analytical techniques") |
| Immunization protects only against parallel shifts | Luenberger Ch 4 ("immunized only against parallel shifts") |
| PV01 = sum of discount factors for fixed leg | Tuckman Ch 18 |
| Duration-based futures hedge ratio | Hull Ch 6, equation (6.3) |
| Minimum variance hedge ratio $h^* = \rho \sigma_S / \sigma_F$ | Hull Ch 3, equation (3.1) |
| Risk weights derived from regression coefficients | Tuckman Ch 8 |
| Two-factor regression: $\beta_{10} = 0.1613$, $\beta_{30} = 0.8774$ | Tuckman Table 8.2 |
| Regression R-squared = 98.25% for 20y on 30y yields | Tuckman Ch 8 |
| Callable bond hedges are "unstable away from 5%" | Tuckman Ch 5 ("A Hedging Example, Part III") |
| CTD can change, causing hedge slippage | Tuckman Ch 20 |
| Conversion factor mechanics | Tuckman Ch 20 |

### (B) Reasoned Inference (Derived from A)

| Content | Derivation |
|---------|------------|
| Futures DV01 $\approx$ DV01_CTD / CF | From invoice-price mechanics (Tuckman Ch 20) and linear approximation. |
| Variance-minimizing hedge = $\beta \times$ DV01 ratio | Generalization of Tuckman's regression approach; equivalent to Hull's correlation formula. |
| CS01 / spread duration decomposition | From standard credit decomposition ($y = r + s$). |
| Twist P&L calculation | Direct application of key-rate shift definitions to portfolio weights. |

### (C) Flagged Uncertainties

| Content | Flag |
|---------|------|
| Specific numerical values in worked examples beyond source data | Illustrative values chosen for clarity; consistent with source methodology. |
| Multi-curve PV01 decomposition details | Current market practice; sources discuss concepts but not all operational specifics. |
