# Chapter 7: Bond Return Decomposition — Carry, Rolldown, Curve Moves, and Spread Changes

---

## Introduction

"The bond made money, but I don't understand why." This is among the most frustrating sentences a trader, portfolio manager, or risk officer can utter—or hear. A position that gained $500,000 over the month tells you almost nothing: Was it because rates fell? Because the credit spread tightened? Because the bond simply accrued coupon faster than funding cost? Or was it luck offset by multiple losing bets?

**Return decomposition** is the discipline of answering these questions rigorously. It takes the raw P&L of a bond position and breaks it into economically meaningful components: the *carry* you earn from holding the position, the *rolldown* from aging along the curve, the impact of *curve moves*, and the effect of *spread changes*. Each component tells a different story about what happened and—critically—what risks you were actually taking.

This matters for three reasons. First, it enables **accountability**: if your mandate is to capture carry without taking directional rate risk, you need to prove that's what happened. Second, it supports **risk management**: understanding which component dominated your P&L tells you which risk you were actually exposed to. Third, it informs **trading decisions**: the breakeven yield move that wipes out your carry determines how confident you need to be in your directional view.

This chapter develops the full machinery of return decomposition:

- **Section 7.1** establishes the foundation: clean vs dirty prices and the holding-period P&L identity.
- **Section 7.2** defines carry precisely, using Tuckman's repo-financed P&L framework, and shows how to compute breakeven price moves.
- **Section 7.3** introduces rolldown and the pull-to-par effect—what happens to price when "nothing" changes except the passage of time.
- **Sections 7.4–7.5** develop the curve and spread effect decomposition, showing how to exactly attribute price changes to benchmark moves vs credit spread moves.
- **Section 7.6** addresses the residual: convexity, cross-terms, and why linear approximations break down.

Throughout, we use explicit numerical examples and derive every formula from first principles. The goal is a decomposition you can implement and defend.

---

## 7.1 Clean Price, Dirty Price, and Holding-Period P&L

### 7.1.1 The Clean/Dirty Distinction

Before we can decompose returns, we must be precise about what "return" means. Bond markets quote **clean prices** but settle at **dirty prices**. The relationship is:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + AI}$$

where accrued interest for a semiannual coupon bond is:

$$\boxed{AI = \frac{c}{2} \cdot \frac{t - t_0}{t_1 - t_0}}$$

Here $c$ is the annual coupon rate (as a decimal), $t_0$ is the last coupon date, and $t_1$ is the next coupon date.

Clean price isolates the "market value" of the bond from the mechanical accrual of coupon interest. Dirty price is what you actually pay or receive at settlement—it reflects the fact that the seller has earned some coupon interest since the last coupon date. As Tuckman explains in Chapters 1–4, the dirty (or "invoice") price equals the clean (or "flat") price plus accrued interest.

**Why this matters for P&L:** Traders quote and hedge clean prices because clean prices don't jump mechanically on coupon dates. But P&L calculations must ultimately use dirty prices (plus coupons received) to capture actual cash flows. Many P&L explain errors trace back to mixing clean and dirty inconsistently—especially around coupon payment dates.

### 7.1.2 The Holding-Period P&L Identity

Consider a long bond position held from time 0 to horizon $h$. If we ignore funding, the total P&L is simply the change in value plus any coupons received:

$$\text{P\&L}_{\text{unfunded}} = \left(P_{\text{dirty}}(h) - P_{\text{dirty}}(0)\right) + \sum_{n=1}^{N} C_n$$

where $C_n$ denotes each coupon payment received during the holding period.

Substituting $P_{\text{dirty}} = P_{\text{clean}} + AI$ and rearranging:

$$\text{P\&L}_{\text{unfunded}} = \underbrace{\left(P_{\text{clean}}(h) - P_{\text{clean}}(0)\right)}_{\text{clean price change}} + \underbrace{\left(\sum_{n=1}^{N} C_n + AI(h) - AI(0)\right)}_{\text{interest income}}$$

This decomposition is useful because it separates the mark-to-market component (clean price change) from the mechanical income component (coupon accrual and payments).

The **horizon return** on an unfunded (cash) position is:

$$R_{0 \to h} = \frac{\text{P\&L}}{P_{\text{dirty}}(0)}$$

This is the basic cash-on-cash return: what you earned relative to what you paid.

---

## 7.2 Carry: Interest Income Minus Financing Cost

### 7.2.1 The Repo-Funded P&L Identity

Most bond positions are financed. A trading desk buying $100 million face of Treasury bonds typically borrows the purchase amount in the repo market, pledging the bonds as collateral. The cost of this financing must be netted against the interest income to arrive at true economic P&L.

Tuckman provides the definitive formulation in Chapter 15. Defining:

- $P(0), P(d)$: Clean prices on the trade date and $d$ days later
- $AI(0), AI(d)$: Accrued interest on the trade date and $d$ days later
- $r$: Repo rate
- $c$: Coupon rate
- $D$: Actual days between last and next coupon payments

The P&L from purchasing the bond and selling it $d$ days later is derived by Tuckman as follows. The change in full price is:

$$\begin{aligned}
\text{P\&L} &= [P(d) + AI(d)] - [P(0) + AI(0)] - \text{Financing cost} \\
&= P(d) - P(0) + AI(d) - AI(0) - (P(0) + AI(0))(rd/360)
\end{aligned}$$

For the typical case where the holding period does not span a coupon date, $AI(d) - AI(0) = cd/D$ where $D$ is the days in the coupon period. Tuckman then rearranges this to:

$$\boxed{\text{P\&L} = \underbrace{[P(d) - P(0)]}_{\text{Price change}} + \underbrace{\text{Carry}}_{\text{Interest income} - \text{Financing cost}}}$$

where Carry = $cd/D - (P(0) + AI(0))(rd/360)$. When coupons are received during the holding period, the income term includes those cash payments:

$$\boxed{\text{Carry} = \sum C_n + AI(d) - AI(0) - (P(0) + AI(0)) \cdot r\frac{d}{360}}$$

This equation is powerful because it cleanly separates the mark-to-market (price change) from the carry (interest income minus financing cost). Tuckman notes that "practitioners like to divide the profit or loss of a trade into a component due to price changes and a component due to carry."

### 7.2.2 Understanding the Carry Formula

> **Analogy: Carry is Your Calorie Surplus**
>
> Think of a bond position like a body:
> *   **Coupons (Income)**: **Food Intake** (Calories In).
> *   **Repo (Financing)**: **Exercise** (Calories Burned).
> *   **Net Carry**: **Weight Gain** (Profit).
>
> *   **Positive Carry**: You eat more than you burn. You are "gaining weight" (profit) every day, even if you just sit on the couch.
> *   **Negative Carry**: You burn more than you eat. You are "starving" (losing money) every day. You *need* the market to move in your favor (price rise) just to stay alive.
>
> **The Carry Trade**: Borrow at 1% (light exercise), Invest at 5% (heavy eating). Result: Massive weight gain (profit).

The carry term has three components:

1. **Coupon income**: $\sum C_n$—any coupon payments actually received during the holding period
2. **Accrual change**: $AI(d) - AI(0)$—the change in accrued interest (which you'll collect when you sell)
3. **Financing cost**: $(P(0) + AI(0)) \cdot r \cdot d/360$—the cost of borrowing the full (dirty) price at the repo rate

Note that repo interest uses an Actual/360 day count in this formulation—you borrow for $d$ actual days but divide by 360. This is standard U.S. money market convention.

**Intuition:** Carry is positive when the bond's coupon rate exceeds the repo rate. Tuckman explains: "Intuitively, the carry in this trade is positive because the coupon rate of the bond... is greater than the repo rate." But the relationship isn't exact—interest income is earned on face value using actual/actual, while repo interest is paid on the dirty price using 30/360.

### 7.2.3 Breakeven Price Moves

Carry enables a critical calculation: how much can the bond's price fall before the position loses money? This is the **breakeven price change**.

> **Back-of-Envelope Breakeven Algebra**
>
> If you know your Carry ($) and your Risk (DV01 in $), you can calculate your survival buffer in your head:
>
> $$\text{Breakeven (bp)} \approx \frac{\text{Net Carry (\$)}}{\text{DV01 (\$)}}$$
>
> *   **Example**:
>     *   Carry = +$10,000 per month.
>     *   DV01 = $1,000 per bp.
>     *   **Survival Buffer**: $10,000 / 1,000 = \text{10 bp}$.
>
> **Result**: You can survive a 10bp adverse move in rates over the next month and still break even.

Tuckman provides a worked example: An investor purchases a 5⅞% Treasury at invoice price 105.103073 and holds for 30 days at a 5.10% repo rate. The carry calculation proceeds:

**Interest income** (30 days in a 181-day coupon period):
$$\$100{,}000{,}000 \times \frac{1}{2} \times 5.875\% \times \frac{30}{181} = \$486{,}878$$

**Financing cost**:
$$\$100{,}000{,}000 \times 105.103073\% \times 5.10\% \times \frac{30}{360} = \$446{,}688$$

**Carry** = $486{,}878 - $446{,}688 = $40{,}190$

Therefore, so long as the price does not fall by more than about 4 cents per 100 face value over the 30-day period, the investment will prove profitable. This is the breakeven price move.

### 7.2.4 Breakeven Holding Periods

The same logic works in reverse: if you have a price target, how quickly must the price reach that target before carry erodes your profit?

Tuckman illustrates with a short position. If a trader shorts a bond at 105.055594 expecting the price to fall to 105, the price change profit would be $55,594 per $100 million face. But a short position has **negative carry** (you pay the coupon rate, receive the repo rate). Setting the carry loss equal to the price profit gives the breakeven holding period—approximately 41 days in Tuckman's example.

### 7.2.5 Carry Is Not Expected Return

A critical warning: positive carry does not imply superior expected return. Tuckman emphasizes this point:

> "Positive carry trades have the desirable property that they earn money as they go. But this by no means implies that the expected return of a positive carry trade is greater than that of a negative carry trade."

Consider the choice between a premium bond (positive carry, but price pulled down toward par) and a discount bond (negative carry, but price pulled up toward par). Carry alone cannot determine which offers better risk-adjusted return.

Tuckman provides the theoretical foundation: "Viewing carry from a theoretical standpoint, Part Three showed that the expected return of any fairly priced portfolio equals the short-term rate plus an appropriate risk premium. This required expected return is the same whether it comes in the form of positive carry and a relatively small or negative expected price change or in the form of negative carry and a relatively large expected price change."

The key insight is that markets price bonds so that expected total returns (carry plus expected price change) are consistent with equilibrium required returns. A bond with high positive carry will have an expected price decline that offsets some of that carry; a bond with negative carry will have an expected price increase. The carry component tells you about the *composition* of expected return, not its *level*.

### 7.2.6 Desk Conventions for Carry

In practice, "carry" can mean different things depending on context:

| Term | Definition |
|------|------------|
| **Income-only carry** | $\sum C_n + AI(h) - AI(0)$ (no funding adjustment) |
| **Net carry** | Income minus financing cost (the repo P&L identity) |
| **Hedged carry** | Net carry after accounting for the carry of hedges (swap, futures, etc.) |

When someone says "the trade has 12bp of carry," always clarify which definition they're using. Hedged carry conventions are instrument- and desk-specific.

---

## 7.3 Rolldown and Pull-to-Par

### 7.3.1 What Happens When "Nothing" Changes?

Even if rates and spreads remain unchanged, bond prices move. A bond held for six months is six months closer to maturity tomorrow than it was today. This passage of time affects price through two channels:

1. **Rolldown**: If the yield curve is upward-sloping, shorter-maturity bonds typically trade at lower yields. As a bond ages, it "rolls down" the curve into a lower-yield region, which can increase its price.

2. **Pull-to-par**: Bonds priced away from par converge toward 100 as maturity approaches. Premium bonds decline toward par; discount bonds rise toward par.

> **Analogy: Rolldown is Sledding**
>
> Think of the Yield Curve as a snowy hill.
> *   **Upward Sloping**: The hill goes down as you move forward (shorter maturity $\to$ lower yield).
> *   **The Trade**: You buy a 5-year bond at the top of the hill (4% yield).
> *   **One Year Passes**: Time pushes you down the hill to the 4-year spot (3% yield).
> *   **Effect**: Your yield drops from 4% to 3%. When yield drops, price rises.
>
> **Result**: Gravity (time) does the work for you. You pick up speed (price gain) just by sliding down the curve.

### 7.3.2 Pull-to-Par: The Mechanics

Tuckman explains this effect clearly in Chapter 3. Consider bonds yielding 5.50%, priced with various coupons:

- A 5.50% coupon bond trades at par (100) at all maturities
- A 7.50% coupon 30-year bond trades around 129; as it matures, the value of above-market coupons declines, and the price falls toward par
- A 3.50% coupon 30-year bond trades around 71; as it matures, the disadvantage of below-market coupons diminishes, and the price rises toward par

Tuckman emphasizes: "It is important to emphasize that to illustrate simply the pull to par [this] assumes that the bonds yield 5.50% at all times. The actual price paths of these bonds over time will differ dramatically from those in the figure depending on the realization of yields."

### 7.3.3 Defining Rolldown Operationally

For this chapter, we define rolldown as a counterfactual: what would the clean price be at the horizon if the yield curve and spread remained unchanged?

$$P_h^{\text{uc}} := P_{\text{clean}}(h; \, z_0(\cdot), \, s_0)$$

where cashflow times are shortened by the passage of time, but the term structures $z_0(\cdot)$ (benchmark curve) and $s_0$ (spread) are held fixed.

Then:

$$\boxed{\text{Rolldown} := P_h^{\text{uc}} - P_0^{\text{clean}}}$$

**How to compute rolldown:**
1. Keep the initial curve and spread surfaces fixed
2. Move the valuation date forward to the horizon and shorten the time-to-cashflows
3. Reprice the bond (clean) at the horizon date using the unchanged curve/spread

This is an exact repricing calculation, not an approximation.

### 7.3.4 Rolldown vs Pull-to-Par: Are They Different?

**Terminology warning:** Many desks use "rolldown" to mean "aging along the curve" and sometimes reserve "pull-to-par" specifically for the time-decay of premium/discount bonds. The sources discuss the mechanics of both effects but do not impose a single industry-wide definition of "rolldown."

In this chapter's framework, our rolldown calculation captures *both* effects—the curve shape effect and the pull-to-par effect—because we reprice the full bond with shortened time-to-cashflows on the original curve. If you need to separate them, you would need additional counterfactual calculations (e.g., pricing as if the bond were par at each horizon).

### 7.3.5 Forward Rates as the Breakeven

Tuckman provides a powerful insight in Chapter 3 about comparing strategies of different maturities. Consider a choice between buying a 6-month zero-coupon bond and rolling versus buying a 1-year zero. The question is: under what rate scenario do these strategies produce equal returns?

The answer: the forward rate is the breakeven. If the 6-month rate six months from now equals today's 6-month forward rate starting in 6 months, then both strategies produce identical returns. More generally, Tuckman notes that "bonds with interest rate risk earn a risk premium" and that "interest rates fall when inflation and expected inflation fall and that low inflation is correlated with good times."

This has direct implications for rolldown analysis. The "rolldown return" assumes rates stay unchanged, but if forward rates are unbiased predictors of future spot rates, then today's forward curve already embeds the expected rolldown. Under pure expectations, the rolldown return is exactly offset by the expected rise in short rates.

### 7.3.6 Why "Carry + Rolldown" Is Not Guaranteed P&L

A common desk heuristic is: "Expected return under unchanged curve = carry + rolldown."

This is correct as a *conditional* expectation—conditional on nothing happening to rates. But realized P&L can be dominated by curve and spread shocks, which can easily overwhelm the carry and rolldown components. Tuckman emphasizes that "Part Three showed that the expected return of any fairly priced portfolio equals the short-term rate plus an appropriate risk premium"—not carry plus rolldown.

In other words, carry + rolldown is a counterfactual return, not a forecast. If the curve is upward-sloping because of term premia (not just expected rate increases), then systematically earning the rolldown may be compensation for bearing interest rate risk.

---

## 7.4 Benchmark Curve Move Effect

### 7.4.1 Defining the Curve Effect

Once we have the rolldown (unchanged curve/spread), we can isolate the impact of curve changes. Let:

- $z_0(\cdot)$: Benchmark curve at time 0
- $z_1(\cdot)$: Benchmark curve at horizon $h$
- $s_0$: Spread held constant

The curve effect is the price change attributable to the benchmark curve moving while spread is held fixed:

$$\boxed{\text{Curve effect} := P_{\text{clean}}(h; \, z_1(\cdot), \, s_0) - P_{\text{clean}}(h; \, z_0(\cdot), \, s_0)}$$

This is an exact calculation using full repricing.

### 7.4.2 First-Order Approximation: DV01

For small curve moves, the curve effect is approximately linear. Tuckman defines DV01 as:

$$\boxed{\text{DV01} = -\frac{1}{10000}\frac{\partial P}{\partial y}}$$

where $y$ is yield (in decimals). The negative sign ensures DV01 is positive for typical bonds (which lose value when rates rise). The 10,000 factor converts from "per unit rate" to "per basis point."

For a yield change of $\Delta y_{\text{bp}}$ basis points:

$$\Delta P \approx -\text{DV01} \times \Delta y_{\text{bp}}$$

### 7.4.3 Second-Order Approximation: Convexity

For larger moves, the linear approximation understates gains and overstates losses (for positive-convexity bonds). Tuckman provides the second-order approximation in Chapter 5:

$$\boxed{\frac{\Delta P}{P} \approx -D \cdot \Delta y + \frac{1}{2} \cdot Cvx \cdot (\Delta y)^2}$$

where $D$ is modified duration and $Cvx$ is convexity. This is Tuckman's equation (5.20).

The convexity term is always positive for option-free bonds: the price-yield relationship is curved, and the curvature benefits the bondholder. As Tuckman notes, "equation (5.20) implied that positive convexity increases return whether rates rise or fall." The greater the change in yield, the greater the performance advantage of bonds with high convexity relative to bonds with low convexity.

---

## 7.5 Spread Change Effect

### 7.5.1 Defining the Spread Effect

After accounting for curve moves, any remaining price change comes from spread changes. Holding the benchmark curve fixed at its horizon value:

$$\boxed{\text{Spread effect} := P_{\text{clean}}(h; \, z_1(\cdot), \, s_1) - P_{\text{clean}}(h; \, z_1(\cdot), \, s_0)}$$

where $s_0$ and $s_1$ are the spread at time 0 and horizon, respectively.

### 7.5.2 Spread DV01 and DVOAS

Just as curve effects can be approximated by DV01, spread effects can be approximated by a spread sensitivity. Tuckman defines DVOAS as "the dollar value of a 0.01 (i.e., 1bp) change in OAS"—essentially a spread DV01 for the option-adjusted spread.

For a Z-spread or similar additive spread measure:

$$\Delta P_{\text{spread}} \approx -\text{DVOAS} \times \Delta s_{\text{bp}}$$

### 7.5.3 Which Spread? Why Definitions Matter

The decomposition depends critically on *which* spread definition you use:

| Spread | Definition | Use Case |
|--------|------------|----------|
| **G-spread** | Bond yield minus government benchmark yield at same maturity | Quick relative value |
| **I-spread** | Bond yield minus swap rate at same maturity | Swap-curve relative value |
| **Z-spread** | Constant spread to benchmark curve making PV = price | Full term structure |
| **OAS** | Spread in option-pricing model making model price = market price | Bonds with embedded options |
| **ASW** | Spread over swap curve in asset swap form | Funding-based relative value |

Tuckman notes that Z-spread definitions can vary by compounding convention (discrete vs continuous). If you change the spread definition, "spread effect" changes, and parts of what you called "curve effect" may migrate into "spread effect."

**Key point:** Always document which spread measure you're using for decomposition.

---

## 7.6 The Exact Decomposition and Residual

### 7.6.1 The Telescoping Identity

The clean price change can be decomposed *exactly* into rolldown, curve effect, and spread effect using intermediate counterfactual prices. Define:

$$\begin{aligned}
P_0 &:= P_{\text{clean}}(0; \, z_0, s_0) \\
P_h^{\text{uc}} &:= P_{\text{clean}}(h; \, z_0, s_0) \quad \text{(unchanged curve/spread)} \\
P_h^{\text{curve}} &:= P_{\text{clean}}(h; \, z_1, s_0) \quad \text{(curve moved only)} \\
P_h^{\text{actual}} &:= P_{\text{clean}}(h; \, z_1, s_1) \quad \text{(actual)}
\end{aligned}$$

Then:

$$\begin{aligned}
\text{Rolldown} &:= P_h^{\text{uc}} - P_0 \\
\text{Curve effect} &:= P_h^{\text{curve}} - P_h^{\text{uc}} \\
\text{Spread effect} &:= P_h^{\text{actual}} - P_h^{\text{curve}}
\end{aligned}$$

By construction:

$$\boxed{\Delta P_{\text{clean}} = P_h^{\text{actual}} - P_0 = \text{Rolldown} + \text{Curve effect} + \text{Spread effect}}$$

This is an **identity** (no approximation) once the intermediate prices are defined.

### 7.6.2 The Residual from Linear Approximations

When curve and spread effects are computed using first-order approximations (DV01 × bp), the remainder is the residual:

$$\text{Residual} = \Delta P_{\text{exact}} - \Delta P_{\text{DV01}}$$

This residual captures:

1. **Convexity**: The price-yield relationship is curved; linear approximations miss second-order effects
2. **Cross-terms**: If both curve and spread move, a Taylor expansion includes a $dr \cdot ds$ cross term
3. **Non-parallel moves**: DV01 assumes parallel shifts; actual curve changes may twist or butterfly

Tuckman provides a second-order expansion for corporate bond price changes including the cross term. The residual is a diagnostic: it should shrink for smaller shocks and grow with longer duration, higher convexity, and larger rate/spread moves.

---

## 7.7 P&L Attribution: The Complete Framework

### 7.7.1 Tuckman's Model-Based Attribution

Tuckman provides an elegant P&L attribution framework in Chapter 14, using a term structure model with a single factor $x$:

$$dP = (r + \text{OAS}) \cdot P \cdot dt + \text{DV01}_x \cdot (dx - E[dx]) + \text{DVOAS} \times d\text{OAS}$$

He interprets these components as:

| Component | Economic Meaning |
|-----------|------------------|
| $(r + \text{OAS}) \cdot P \cdot dt$ | **Carry**: Return from time passage (including OAS if mispriced) |
| $\text{DV01}_x \cdot (dx - E[dx])$ | **Factor exposure**: Return from unexpected rate moves |
| $\text{DVOAS} \times d\text{OAS}$ | **Convergence**: Return from spread normalization |

Tuckman notes: "The term convergence is used since, for models with predictive power, the OAS tends to zero or, equivalently, the security price tends toward its fair value according to the model."

### 7.7.2 The Complete P&L Equation

Combining all components, the full P&L attribution is:

> **Visualization: The Waterfall Chart**
>
> Imagine a bar chart explaining your month's P&L.
> *   **Bar 1 (Carry)**: **The Base**. The steady income you earned (Coupons - Repo). Usually positive.
> *   **Bar 2 (Rolldown)**: **The Slide**. The gain from sliding down the curve. Usually positive if curve is steep.
> *   **Bar 3 (Rates Move)**: **The Market Wave**. The big random swing driven by the Fed/Data. Can be huge positive or huge negative.
> *   **Bar 4 (Spread Chg)**: **The Credit Wave**. The swing driven by credit sentiment.
> *   **Total Net Bar**: The sum of all four.
>
> **Key Insight**: "Yield" is just a guess. These 4 pillars are the actual result.

$$\boxed{\text{P\&L} = \text{Carry} + \text{Rolldown} + \text{Curve effect} + \text{Spread effect} + \text{Residual}}$$

Where:
- **Carry**: $\sum C_n + AI(h) - AI(0) - (P_0 + AI_0) \cdot r \cdot d/360$
- **Rolldown**: $P_h^{\text{uc}} - P_0$ (clean price)
- **Curve effect**: $P_h^{\text{curve}} - P_h^{\text{uc}}$ (or $-\text{DV01} \times \Delta z_{\text{bp}}$ approximately)
- **Spread effect**: $P_h^{\text{actual}} - P_h^{\text{curve}}$ (or $-\text{DVOAS} \times \Delta s_{\text{bp}}$ approximately)
- **Residual**: Convexity + cross-terms + approximation error

---

## Worked Examples

### Example A — Horizon Return Basics (Dirty vs Clean; Coupon Date vs None)

**Bond:**
- Face: 100
- Coupon: 6.00% annual, semiannual coupons $C = 3.00$
- Coupon dates: Apr 30 and Oct 30
- Day count for accrual: 30/360
- Clean/dirty relationship: $P_{\text{dirty}} = P_{\text{clean}} + AI$

**Start (Jan 30, 2026):**
- Given clean price $P_{\text{clean}}(0) = 101.80$
- Last coupon date: Oct 30, 2025; next coupon: Apr 30, 2026
- Days since last coupon: 90, days in period: 180
- Accrued interest:

$$AI(0) = 3.00 \cdot \frac{90}{180} = 1.50$$

- Dirty price:

$$P_{\text{dirty}}(0) = 101.80 + 1.50 = 103.30$$

#### Case A1: Horizon with No Coupon Date (Mar 30, 2026)

- Given horizon clean price $P_{\text{clean}}(h) = 101.50$
- Accrued interest at horizon: Oct 30 → Mar 30 = 150 days (30/360)

$$AI(h) = 3.00 \cdot \frac{150}{180} = 2.50$$

- Dirty price at horizon:

$$P_{\text{dirty}}(h) = 101.50 + 2.50 = 104.00$$

- Coupons received: none

**Total P&L:**

$$\text{P\&L} = P_{\text{dirty}}(h) - P_{\text{dirty}}(0) = 104.00 - 103.30 = 0.70$$

**Holding-period return (on dirty price):**

$$R = \frac{0.70}{103.30} = 0.00678 \approx 0.678\%$$

**Decompose into price vs income:**
- Clean price change: $\Delta P_{\text{clean}} = 101.50 - 101.80 = -0.30$
- Interest income (accrual): $AI(h) - AI(0) = 2.50 - 1.50 = +1.00$
- Check: $-0.30 + 1.00 = 0.70$ ✓

#### Case A2: Horizon Crossing One Coupon Date (May 30, 2026; Coupon on Apr 30)

- Given horizon clean price $P_{\text{clean}}(h) = 101.20$
- Coupon received on Apr 30: $C = 3.00$
- Accrued interest at May 30 (Apr 30 → May 30 = 30 days):

$$AI(h) = 3.00 \cdot \frac{30}{180} = 0.50$$

- Dirty price at horizon:

$$P_{\text{dirty}}(h) = 101.20 + 0.50 = 101.70$$

**Total P&L:**

$$\text{P\&L} = P_{\text{dirty}}(h) + C - P_{\text{dirty}}(0) = 101.70 + 3.00 - 103.30 = 1.40$$

**Holding-period return:**

$$R = \frac{1.40}{103.30} = 0.0136 \approx 1.36\%$$

**Decompose:**
- Clean price change: $101.20 - 101.80 = -0.60$
- Income: $C + AI(h) - AI(0) = 3.00 + 0.50 - 1.50 = 2.00$
- Check: $-0.60 + 2.00 = 1.40$ ✓

---

### Example B — Carry-Only Scenario (Unfunded vs Repo-Funded Carry)

Use the same bond and start point as Example A:
- $P_{\text{clean}}(0) = 101.80$, $AI(0) = 1.50$, so $P_{\text{dirty}}(0) = 103.30$

**Repo carry identity (with 360-day basis for repo interest):**
- Assume repo rate $r = 4.50\% = 0.045$

#### Horizon B1: Jan 30 → Mar 30 (60 days; no coupon)

From Example A1: $AI(h) = 2.50$

**Unfunded carry (income-only):**

$$\text{Carry}_{\text{unfunded}} = AI(h) - AI(0) = 2.50 - 1.50 = 1.00$$

**Repo financing cost:**

$$\text{Fin} = (P_{\text{clean}}(0) + AI(0)) \cdot r\frac{d}{360} = 103.30 \cdot 0.045 \cdot \frac{60}{360}$$

Since $\frac{60}{360} = \frac{1}{6}$:

$$\text{Fin} = 103.30 \cdot 0.045 \cdot \frac{1}{6} = 103.30 \cdot 0.0075 = 0.77475$$

**Repo-funded carry:**

$$\text{Carry}_{\text{repo}} = 1.00 - 0.77475 = 0.22525$$

**Carry returns (on initial dirty price):**
- Unfunded: $1.00 / 103.30 = 0.968\%$
- Repo-funded: $0.22525 / 103.30 = 0.218\%$

#### Horizon B2: Jan 30 → May 30 (120 days; one coupon)

From Example A2: coupon $C = 3.00$, $AI(h) = 0.50$

**Income:**

$$\sum C_n + AI(h) - AI(0) = 3.00 + 0.50 - 1.50 = 2.00$$

**Financing:**

$$103.30 \cdot 0.045 \cdot \frac{120}{360} = 103.30 \cdot 0.045 \cdot \frac{1}{3} = 103.30 \cdot 0.015 = 1.5495$$

**Repo-funded carry:**

$$2.00 - 1.5495 = 0.4505$$

**Carry returns:**
- Unfunded: $2.00 / 103.30 = 1.936\%$
- Repo-funded: $0.4505 / 103.30 = 0.436\%$

---

### Example C — Rolldown-Only Scenario (Curve Held Fixed; Aging Reduces Time-to-Cashflows)

Here we compute rolldown = clean price change under unchanged curve/spread, excluding carry.

**Bond (used for Examples C–F):**
- Face: 100
- Coupon: 5.00% annual, semiannual coupons $C = 2.50$
- Start date $t_0$: Jul 30, 2026 (coupon date → $AI(0) = 0$)
- Maturity: Jul 30, 2028 (2.0 years from $t_0$)
- Cashflows (years from $t_0$): $t = \{0.5, 1.0, 1.5, 2.0\}$ with coupons 2.5 each period and final 102.5

**Benchmark zero curve $z(t)$ (semiannual comp; stylized):**

| Maturity | $z(t)$ |
|----------|--------|
| 0.5 | 4.00% |
| 1.0 | 4.20% |
| 1.5 | 4.35% |
| 2.0 | 4.50% |

**Spread measure:** Z-spread $s_0 = 1.00\%$ (100bp), additive to benchmark rates

**Total discount rates:** $y(t) = z(t) + s_0$

| Maturity | $y(t)$ |
|----------|--------|
| 0.5 | 5.00% |
| 1.0 | 5.20% |
| 1.5 | 5.35% |
| 2.0 | 5.50% |

**Discount factors $df(t) = (1 + y(t)/2)^{-2t}$:**

| Maturity | $df(t)$ |
|----------|---------|
| 0.5 | 0.9756097561 |
| 1.0 | 0.9499599117 |
| 1.5 | 0.9238593648 |
| 2.0 | 0.8971657337 |

**Start clean price $P_0^{\text{clean}}$ (exact repricing):**

$$P_0^{\text{clean}} = 2.5 \cdot df(0.5) + 2.5 \cdot df(1.0) + 2.5 \cdot df(1.5) + 102.5 \cdot df(2.0)$$
$$= 2.43902439 + 2.37489978 + 2.30964841 + 91.95948770$$
$$= 99.08306029$$

**Horizon:** $h = 0.5$ years (Jan 30, 2027), coupon date ⇒ clean pricing, $AI(h) = 0$.

Under unchanged curve/spread, remaining cashflows from horizon are at 0.5, 1.0, 1.5 years:
- 2.5 at 0.5, 2.5 at 1.0, 102.5 at 1.5

**Horizon clean price under unchanged curve/spread:**

$$P_h^{\text{uc}} = 2.5 \cdot df(0.5) + 2.5 \cdot df(1.0) + 102.5 \cdot df(1.5)$$
$$= 2.43902439 + 2.37489978 + 94.69558489$$
$$= 99.50950906$$

**Rolldown (clean price):**

$$\boxed{\text{Rolldown} = P_h^{\text{uc}} - P_0^{\text{clean}} = 99.50950906 - 99.08306029 = 0.42644877}$$

**Rolldown return on initial price:**

$$0.42644877 / 99.08306029 = 0.4304\%$$

---

### Example D — Curve Move Decomposition (Parallel +10bp Benchmark Zero Shock)

Use the start-date bond and curve from Example C.

**Shock:**
- Benchmark curve shifts up +10bp at all maturities: $z'(t) = z(t) + 0.10\%$
- Spread stays $s_0 = 1.00\%$
- Total discount rates become: $y'(0.5) = 5.10\%$, $y'(1.0) = 5.30\%$, $y'(1.5) = 5.45\%$, $y'(2.0) = 5.60\%$

**Exact repricing:**

Shocked discount factors $df'(t) = (1 + y'(t)/2)^{-2t}$:

| Maturity | $df'(t)$ |
|----------|----------|
| 0.5 | 0.9751340809 |
| 1.0 | 0.9490347012 |
| 1.5 | 0.9225109932 |
| 2.0 | 0.8954215481 |

Shocked price:

$$P' = 2.5 \cdot df'(0.5) + 2.5 \cdot df'(1.0) + 2.5 \cdot df'(1.5) + 102.5 \cdot df'(2.0) = 98.89740812$$

**Exact curve-driven price change:**

$$\Delta P_{\text{exact}} = P' - P_0 = 98.89740812 - 99.08306029 = -0.18565217$$

**DV01-based approximation (first order):**

DV01 concept (yield DV01) is defined as $-\frac{1}{10000}\frac{\partial P}{\partial y}$.

Here we compute a parallel curve DV01 by bumping the benchmark curve by ±1bp and using a central difference:

- Price with +1bp parallel shift: $P_{+1\text{bp}} = 99.06447499$
- Price with −1bp parallel shift: $P_{-1\text{bp}} = 99.10165005$

Central-difference DV01 (per 1bp):

$$\text{DV01} \approx \frac{P_{-1\text{bp}} - P_{+1\text{bp}}}{2} = \frac{99.10165005 - 99.06447499}{2} = 0.01858753$$

**First-order price change for +10bp:**

$$\Delta P_{\text{DV01}} \approx -\text{DV01} \cdot 10 = -0.18587527$$

**Convexity error (DV01 approx − exact):**

$$-0.18587527 - (-0.18565217) = -0.00022310$$

DV01 (linear) slightly overstates the price drop because convexity is positive.

**Convexity-adjusted (second order; optional check):**

Using the second difference from the ±1bp prices, the convexity correction for $\Delta y = 10\text{bp} = 0.0010$ is $\approx +0.00022332$, so the 2nd-order approximation matches the exact repricing extremely closely in this small-shock example.

---

### Example E — Spread Change Decomposition (Z-Spread +25bp; Benchmark Curve Fixed)

Use Example C at start date $t_0$.

**Spread definition:**
We use a Z-spread/ZVS-style definition: spread $s$ added to benchmark discount rates, with compounding convention specified.

**Shock:**
- Benchmark curve $z(t)$ unchanged
- Spread widens: $s_1 = s_0 + 25\text{bp} = 1.25\%$
- New total rates: $y(0.5) = 5.25\%$, $y(1.0) = 5.45\%$, $y(1.5) = 5.60\%$, $y(2.0) = 5.75\%$

**Exact repricing:**

Discount factors:

| Maturity | $df(t)$ |
|----------|---------|
| 0.5 | 0.9744214373 |
| 1.0 | 0.9476494178 |
| 1.5 | 0.9204933514 |
| 2.0 | 0.8928132094 |

Price with widened spread:

$$P_{\text{wide}} = 98.61976448$$

**Exact spread-driven price change:**

$$\Delta P_{\text{exact}} = 98.61976448 - 99.08306029 = -0.46329581$$

**Spread DV01 / DVOAS-style approximation:**

Tuckman defines DVOAS as the dollar value of a 1bp change in OAS.

We compute an analogous spread DV01 by bumping $s$ by ±1bp:
- $P(s_0 + 1\text{bp}) = 99.06447499$
- $P(s_0 - 1\text{bp}) = 99.10165005$

Spread DV01:

$$\text{SDV01} \approx \frac{99.10165005 - 99.06447499}{2} = 0.01858753$$

**First-order approximation for +25bp:**

$$\Delta P_{\text{SDV01}} \approx -0.01858753 \cdot 25 = -0.46468817$$

**Error (linear − exact):**

$$-0.46468817 - (-0.46329581) = -0.00139236$$

Convexity is more visible for 25bp than for 10bp.

**Convexity-adjusted check (optional):**

Using the same second-difference logic, the convexity correction for $\Delta s = 25\text{bp} = 0.0025$ is about $+0.00139573$, bringing the approximation nearly on top of the exact repricing.

---

### Example F — Full Decomposition Over a Horizon (Carry + Rolldown + Curve + Spread + Residual)

Use the bond and initial curve/spread from Example C.

**Horizon:**
- $t_0 =$ Jul 30, 2026 (coupon date ⇒ $AI(0) = 0$)
- $t_1 =$ Jan 30, 2027 (coupon date ⇒ $AI(1) = 0$)
- Horizon length: 180 days on 30/360 basis (0.5 year)

**Start price:**
- $P_0^{\text{clean}} = 99.08306029$
- $P_0^{\text{dirty}} = 99.08306029$ (since $AI(0) = 0$)

**Assumed realized market changes over horizon:**
- Benchmark curve shifts up +8bp parallel
- Spread widens +15bp (from 1.00% to 1.15%)

#### Step 1: Carry (income and financing)

- Coupon received at $t_1$: $C = 2.50$

**Unfunded carry (income-only):**

$$\text{Carry}_{\text{unfunded}} = 2.50 + AI(1) - AI(0) = 2.50$$

**Repo-funded carry (if financed at repo $r = 4.50\%$, 30/360):**

$$\text{Fin} = P_0^{\text{dirty}} \cdot r\frac{d}{360} = 99.08306029 \cdot 0.045 \cdot \frac{180}{360} = 99.08306029 \cdot 0.0225 = 2.22936886$$

$$\text{Carry}_{\text{repo}} = 2.50 - 2.22936886 = 0.27063114$$

(This matches the repo carry structure in Tuckman's P&L decomposition.)

#### Step 2: Rolldown (unchanged curve/spread)

From Example C:

$$P_1^{\text{uc}} = 99.50950906$$

$$\text{Rolldown} = P_1^{\text{uc}} - P_0 = 0.42644877$$

#### Step 3: Curve effect (curve moved, spread held at $s_0$)

Compute horizon price with curve shifted +8bp and spread still 1.00%:

$$P_1^{\text{curve}} = 99.39611893$$

$$\text{Curve effect} = P_1^{\text{curve}} - P_1^{\text{uc}} = 99.39611893 - 99.50950906 = -0.11339013$$

#### Step 4: Spread effect (spread moved, curve held at new level)

Now widen spread by +15bp (to 1.15%) keeping the +8bp curve shift:

$$P_1^{\text{actual}} = 99.18398395$$

$$\text{Spread effect} = P_1^{\text{actual}} - P_1^{\text{curve}} = 99.18398395 - 99.39611893 = -0.21213498$$

#### Step 5: Check the clean-price decomposition (exact)

Sum of clean-price components:

$$0.42644877 - 0.11339013 - 0.21213498 = 0.10092367$$

Actual clean price change:

$$P_1^{\text{actual}} - P_0 = 99.18398395 - 99.08306029 = 0.10092367$$

**Matches exactly (telescoping identity).** ✓

#### Step 6: Total P&L and return

**Unfunded P&L:**

$$\text{P\&L}_{\text{unfunded}} = (P_1^{\text{actual}} - P_0) + 2.50 = 0.10092367 + 2.50 = 2.60092367$$

Return on dirty price:

$$R_{\text{unfunded}} = \frac{2.60092367}{99.08306029} = 2.6250\% \text{ over 6 months}$$

**Repo-funded P&L (on full price; ignoring haircut/leverage):**

$$\text{P\&L}_{\text{repo}} = (P_1^{\text{actual}} - P_0) + 0.27063114 = 0.37155481$$

Return on full price:

$$R_{\text{repo}} = \frac{0.37155481}{99.08306029} = 0.3750\% \text{ over 6 months}$$

(Return on equity depends on haircut/leverage; not specified here.)

#### Step 7: First-order approximation vs exact (residual)

Compute horizon DV01 (for the aged bond) by ±1bp bump around the unchanged horizon curve/spread:

- $P_{1,-1\text{bp}} = 99.52369515$
- $P_{1,+1\text{bp}} = 99.49532571$

$$\text{DV01}_h \approx \frac{99.52369515 - 99.49532571}{2} = 0.01418472$$

**Approximate effects:**

Curve effect (+8bp):

$$\Delta P_{\text{curve}}^{(1)} \approx -0.01418472 \cdot 8 = -0.11347776$$

Spread effect (+15bp):

$$\Delta P_{\text{spread}}^{(1)} \approx -0.01418472 \cdot 15 = -0.21277079$$

**Approximate total P&L (unfunded):**

$$\widehat{\text{P\&L}} = 2.50 + 0.42644877 - 0.11347776 - 0.21277079 = 2.60020022$$

**Residual:**

$$\text{Residual} = \text{P\&L}_{\text{exact}} - \widehat{\text{P\&L}} = 2.60092367 - 2.60020022 = 0.00072345$$

**Interpretation:**
This is the convexity/cross-term remainder from linearizing curve and spread effects, consistent with second-order expansions that include squared and cross terms in rate/spread changes.

---

## Practical Notes

### 7.8.1 Common Desk Definitions and Ambiguity Traps

**Different meanings of "carry":**
- The repo-funded identity explicitly defines carry to include the financing term (negative for long positions financed in repo)
- "Hedged carry" depends on what hedge is used and how its own carry/roll is measured
- Always clarify: income-only, net, or hedged?

**Spread measure ambiguity:**
If you change the spread definition, "spread effect" changes, and parts of what you called "curve effect" may migrate into "spread effect." Document your choice clearly.

**Clean vs dirty price mistakes:**
Returns computed on clean prices without adjusting for accrued interest and coupons can be badly wrong around coupon dates. Always ensure:
- Price change is measured on clean price
- Income includes $AI(h) - AI(0)$ plus coupons received
- Or equivalently, work directly with dirty prices plus coupons

### 7.8.2 Implementation Pitfalls

Return decomposition systems are prone to subtle data alignment errors. The following table highlights common implementation traps:

| Issue | Notes |
|-------|-------|
| **Schedule generation** | Coupon dates, ex-coupon conventions, and settlement calendars materially affect AI and which cashflows are included |
| **Horizon-date accrued handling** | If horizon date is between coupon dates, you must compute $AI(h)$ and use dirty price consistently |
| **Coupon reinvestment** | State whether coupons are reinvested and at what rate. Otherwise, "total return" is ambiguous |
| **Modern multi-curve context** | In collateralized markets, discounting may use an OIS discount curve, while floating projections use a LIBOR (or other) curve. If your decomposition uses a benchmark curve that is not your discount curve, explicitly state the mapping |

### 7.8.3 Verification Tests (Quick Sanity Checks)

| Test | Expected Result |
|------|-----------------|
| **If curve and spread are unchanged** | Total return ≈ carry + rolldown (within rounding) |
| **Sign checks** | Higher yields/spreads → lower prices (long bond loses) |
| **Approximation consistency** | Residual shrinks as shocks become small (linearization becomes accurate) |

---

## Summary

This chapter developed a complete framework for decomposing bond returns:

1. **Dirty = clean + accrued interest**: Use dirty prices (or clean + AI consistently) to compute horizon returns; coupon timing matters.

2. **Carry = income − funding**: Tuckman's repo P&L identity separates mechanical carry from mark-to-market. Carry is useful for computing breakeven price moves.

3. **Carry is not expected return**: Positive carry does not imply superior expected return. Premium bonds have positive carry but prices pulled toward par.

4. **Rolldown = aging on unchanged curve**: Define rolldown operationally as the clean price change from revaluing the aged bond on the unchanged curve/spread surface.

5. **Exact decomposition via counterfactuals**: Clean price change = rolldown + curve effect + spread effect (a telescoping identity).

6. **DV01/DVOAS approximations**: Curve and spread effects are approximately linear for small moves; convexity matters for larger shocks.

7. **The residual is diagnostic**: It captures convexity and cross-terms; should shrink as shocks become small.

8. **Decomposition depends on definitions**: Which curve is "benchmark"? Which spread measure? Day count and compounding conventions all matter.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Dirty price | Clean + accrued interest | What you actually pay/receive |
| Carry | Interest income minus financing cost | Mechanical P&L component; determines breakevens |
| Rolldown | Price change from aging on unchanged curve | Captures curve shape + pull-to-par effects |
| Curve effect | Price change from benchmark moves | Rate risk attribution |
| Spread effect | Price change from spread moves | Credit/liquidity risk attribution |
| DV01 | Price sensitivity per 1bp | First-order approximation for rate moves |
| DVOAS | Price sensitivity to OAS per 1bp | First-order approximation for spread moves |
| Residual | Exact minus approximate | Convexity + cross-terms + model error |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P_{\text{clean}}(t)$ | Clean price at time $t$ (per 100 face) |
| $P_{\text{dirty}}(t)$ | Dirty/full price at time $t$ (per 100 face), $= P_{\text{clean}}(t) + AI(t)$ |
| $AI(t)$ | Accrued interest at time $t$ |
| $c$ | Annual coupon rate (decimal, e.g., 0.05 for 5%) |
| $C_n$ | Coupon payment received during the horizon |
| $h$ or $d$ | Horizon length (years or days) |
| $r$ | Repo rate (decimal) |
| $z(t)$ | Benchmark zero rate for maturity $t$ |
| $s$ | Spread measure (Z-spread / ZVS) |
| $y(t)$ | Total discount rate ($= z(t) + s$ in this chapter) |
| $df(t)$ | Discount factor to maturity $t$ |
| DV01 | Dollar value of a 1bp change in yield |
| DVOAS | Dollar value of a 1bp change in OAS/spread |
| $D$ | (Modified) duration |
| $Cvx$ | Convexity |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the relationship between dirty and clean price? | $P_{\text{dirty}} = P_{\text{clean}} + AI$ |
| 2 | What does accrued interest represent? | Coupon interest earned since the last coupon date (pro-rata through the period) |
| 3 | In a repo-funded position, what is "carry" (in Tuckman's formulation)? | $\sum C_n + AI(h) - AI(0) - (P(0) + AI(0)) \cdot r \cdot (d/360)$ |
| 4 | Why separate clean price change from carry? | Clean price change captures MTM; carry captures coupon accrual and funding—useful for P&L explain |
| 5 | Define horizon return (unfunded) | $R = \text{P\&L} / P_{\text{dirty}}(0)$ where $\text{P\&L} = (P_{\text{dirty}}(h) - P_{\text{dirty}}(0)) + \sum C_n$ |
| 6 | What is rolldown (as defined in this chapter)? | $P_{\text{clean}}(h; z_0, s_0) - P_{\text{clean}}(0; z_0, s_0)$ (aging with curve/spread fixed) |
| 7 | What is pull-to-par? | The tendency of a bond price to approach par at maturity as time passes and cashflows are received |
| 8 | What is DV01? | $\text{DV01} = -\frac{1}{10000}\frac{\partial P}{\partial y}$ |
| 9 | How do you use DV01 to approximate a price change for $\Delta y_{\text{bp}}$? | $\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}}$ |
| 10 | What does convexity correct in DV01-based approximations? | Nonlinearity of the price–yield relationship; linear DV01 misses 2nd-order effects |
| 11 | State the duration/convexity approximation for $\Delta P/P$ | $\Delta P/P \approx -D\Delta y + \frac{1}{2}Cvx(\Delta y)^2$ |
| 12 | What is OAS? | Constant spread (in a tree-based model) that makes model price equal market price |
| 13 | What is DVOAS? | Dollar value of a 1bp change in OAS |
| 14 | What is Z-spread/ZVS? | Spread added to benchmark discount rates so discounted cashflows equal the bond's price |
| 15 | Why does spread attribution depend on the spread measure? | Different spread measures embed different curve assumptions and option adjustments |
| 16 | What is the "curve effect" in horizon clean-price explain? | Price change at horizon from changing benchmark curve $z_0 \to z_1$ holding spread fixed |
| 17 | What is the "spread effect" in horizon clean-price explain? | Price change at horizon from changing spread $s_0 \to s_1$ holding benchmark curve fixed |
| 18 | Why can carry+rolldown be positive while realized P&L is negative? | Adverse curve and/or spread moves can overwhelm carry and rolldown |
| 19 | Give a quick sign check for a long bond when yields rise | Price falls ⇒ P&L negative (absent large carry offsets) |
| 20 | What happens to accrued interest on a coupon date? | It resets (AI goes back to ~0) after the coupon is paid |
| 21 | Why does positive carry not imply superior expected return? | Premium bonds have positive carry but prices pulled down toward par; discount bonds have negative carry but prices pulled up |
| 22 | What is a breakeven price change? | The price move that exactly offsets carry, resulting in zero P&L |
| 23 | Why does decomposition depend on benchmark curve choice? | The "curve move" component is defined relative to that benchmark; changing the benchmark changes measured components |
| 24 | What should happen to the residual as shocks become tiny? | It should approach zero as linear approximations become accurate |
| 25 | What three components appear in Tuckman's model-based P&L attribution? | Carry, factor exposure, and convergence |
| 26 | What is the forward rate's role in maturity choice decisions? | The forward rate is the breakeven: if the future spot rate equals today's forward, strategies of different maturities produce equal returns |
| 27 | Why might systematically earning rolldown be fair compensation? | If the upward-sloping curve reflects term premia (not just expected rate rises), then rolldown return is compensation for bearing interest rate risk |

---

## Mini Problem Set

**Problem 1:** A bond has clean price 99.20 and accrued interest 1.10. What is dirty price?

**Problem 2:** A semiannual 6% bond has 120 days elapsed in a 180-day coupon period (30/360). Compute accrued interest per 100.

**Problem 3:** A bond is bought at clean 101.80 with AI 1.50 and sold at clean 101.50 with AI 2.50. No coupon is received. Compute P&L and horizon return on dirty price.

**Problem 4:** Using the repo carry formula, compute repo-funded carry over 60 days when dirty price is 103.30, repo rate is 4.5%, and interest income is 1.00.

**Problem 5:** For a bond with DV01 = 0.0186, estimate the price change for a +12bp yield move.

**Problem 6:** A bond's exact price change for +25bp is −0.4633. DV01 approximation gives −0.4647. What does the difference represent?

**Problem 7:** Define an exact telescoping decomposition of clean price change into rolldown, curve effect, and spread effect using counterfactual horizon prices.

**Problem 8:** In Example C, compute the rolldown return (rolldown / initial price) to 4 decimals.

**Problem 9:** Explain two reasons why "carry + rolldown" can be a poor forecast of realized P&L.

**Problem 10:** Give two different market definitions of "spread" used in credit relative value and explain how the decomposition changes across them.

**Problem 11:** Describe how you would compute spread DV01 for a Z-spreaded bond (no options) using a finite-difference bump.

**Problem 12:** If both benchmark curve and spread move, what second-order interaction term can appear in a Taylor approximation?

**Problem 13:** A trader says "this bond has 50bp of carry per year." What clarifying questions should you ask?

**Problem 14:** Propose a robust set of verification tests for a P&L explain system implementing carry/rolldown/curve/spread decomposition.

### Brief Solution Sketches

**1.** $P_{\text{dirty}} = 99.20 + 1.10 = 100.30$

**2.** Semiannual coupon $= 3.00$. AI $= 3.00 \cdot (120/180) = 2.00$

**3.** Dirty start $= 103.30$. Dirty end $= 104.00$. P&L $= 0.70$. Return $= 0.70/103.30 \approx 0.678\%$

**4.** Financing cost $= 103.30 \cdot 0.045 \cdot (60/360) = 0.77475$. Repo-funded carry $= 1.00 - 0.77475 = 0.22525$

**5.** $\Delta P \approx -0.0186 \cdot 12 = -0.2232$ price points

**6.** The difference is primarily convexity (nonlinearity) and any mismatch between the sensitivity point and the actual move; it is the "residual."

**7.** Define $P_0 = P_{\text{clean}}(0; z_0, s_0)$, $P_h^{\text{uc}} = P_{\text{clean}}(h; z_0, s_0)$, $P_h^{\text{curve}} = P_{\text{clean}}(h; z_1, s_0)$, $P_h^{\text{actual}} = P_{\text{clean}}(h; z_1, s_1)$. Then rolldown $= P_h^{\text{uc}} - P_0$, curve effect $= P_h^{\text{curve}} - P_h^{\text{uc}}$, spread effect $= P_h^{\text{actual}} - P_h^{\text{curve}}$, and they sum to $P_h^{\text{actual}} - P_0$.

**8.** $0.42644877 / 99.08306029 = 0.4304\%$

**9.** (1) Adverse curve moves can overwhelm positive carry; (2) Spread widening can dominate all other components. Carry+rolldown is conditional on "nothing happening," which is rarely the case.

**10.** Z-spread (constant spread to zero curve) vs ASW (spread over swap curve in asset swap form). Changing from Z-spread to ASW changes which curve is "benchmark," so the same price change gets attributed differently between curve effect and spread effect.

**11.** Compute $P(s+1\text{bp})$ and $P(s-1\text{bp})$ by repricing with spread bumped up/down 1bp, then SDV01 $\approx (P(s-1\text{bp}) - P(s+1\text{bp}))/2$.

**12.** A cross-term $dr \cdot ds$ appears in the second-order expansion, representing the interaction between curve and spread moves.

**13.** Is that income-only carry or net carry? What repo rate assumption? What day count? Annualized how (simple or compounded)? Is it carry per 100 face or per dollar invested?

**14.** Tests: (a) If curve/spread unchanged, verify P&L = carry + rolldown; (b) Sign checks (higher yields → lower prices); (c) Sum of components = total; (d) Residual shrinks for smaller shocks; (e) Reconcile to actual cash P&L.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Content | Source |
|---------|--------|
| Dirty = clean + AI; accrued interest proportional formula | Tuckman Ch 1-4 |
| Repo P&L identity and carry definition; 360-day basis | Tuckman Ch 15 |
| Carry = interest earned minus cost of financing | Tuckman Ch 15: "Carry is defined as the interest earned on a position minus the cost of financing the position" |
| Breakeven price change and breakeven holding period calculations | Tuckman Ch 15 |
| Positive carry does not imply superior expected return | Tuckman Ch 15: "Positive carry trades... by no means implies that the expected return... is greater than that of a negative carry trade" |
| DV01 definition: $-\frac{1}{10000}\frac{\partial P}{\partial y}$ | Tuckman Ch 5-6 |
| Duration/convexity 2nd-order approximation (equation 5.20); convexity increases return whether rates rise or fall | Tuckman Ch 5 |
| Pull-to-par mechanism and diagram | Tuckman Ch 3 |
| Forward rates as breakeven for maturity strategies; bonds with interest rate risk earn a risk premium | Tuckman Ch 3, Ch 10 |
| Expected return = short-term rate + risk premium; carry composition vs level | Tuckman Ch 15 (Part Three reference) |
| OAS definition; DVOAS as dollar value of 1bp OAS change | Tuckman Ch 14 |
| P&L attribution: carry, factor exposure, convergence | Tuckman Ch 14 |
| Z-spread/ZVS definition; compounding variants noted | Tuckman Ch 4 |
| Cost of carry concept | Hull Ch 5 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation Logic |
|-----------|------------------|
| Rolldown = counterfactual reprice at horizon with unchanged curve/spread | Operational definition using source mechanics (aging along the curve) |
| Exact telescoping decomposition via intermediate prices | Direct algebraic construction |
| Residual = convexity + cross terms | Follows from 2nd-order Taylor expansion in Tuckman Ch 14 |
| Spread attribution depends on spread definition choice | Implied by multiple spread definitions in sources |

### (C) Speculation / Uncertainty

| Topic | Notes |
|-------|-------|
| "Rolldown" terminology | Sources discuss mechanics (pull-to-par, aging) but do not impose a single industry-wide definition. Adopted explicit operational definition; desks may differ. |
| Hedged carry conventions | Instrument- and desk-specific; not fully specified in sources |

---

*Cross-references:*
- Clean/dirty price and accrued interest: see Chapter 4, Chapter 5
- DV01 and duration: see Chapter 11, Chapter 12
- Repo mechanics and financing: see Chapter 9
- Spread measures (Z-spread, OAS, ASW): see Chapter 8
