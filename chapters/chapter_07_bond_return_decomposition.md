# Chapter 7: Bond Return Decomposition — Carry, Rolldown, Curve Moves, and Spread Changes

---

## Introduction

"The bond made money, but I don't understand why." This is among the most frustrating sentences a trader, portfolio manager, or risk officer can utter—or hear. A position that gained $500,000 over the month tells you almost nothing: Was it because rates fell? Because the credit spread tightened? Because the bond simply accrued coupon faster than funding cost? Or was it luck offset by multiple losing bets?

**Return decomposition** is the discipline of answering these questions rigorously. It takes the raw P&L of a bond position and breaks it into economically meaningful components: the *carry* you earn from holding the position, the *rolldown* from aging along the curve, the impact of *curve moves*, and the effect of *spread changes*. Each component tells a different story about what happened and—critically—what risks you were actually taking.

This matters for three reasons. First, it enables **accountability**: if your mandate is to capture carry without taking directional rate risk, you need to prove that's what happened. Second, it supports **risk management**: understanding which component dominated your P&L tells you which risk you were actually exposed to. Third, it informs **trading decisions**: the breakeven yield move that wipes out your carry determines how confident you need to be in your directional view.

Prerequisites: [Chapter 3 — Zero Rates, Forward Rates, Par Rates — The Triangle](chapters/chapter_03_zero_forward_par_rates_triangle.md), [Chapter 5 — Fixed-Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md), [Chapter 6 — Yield-to-Maturity (YTM) and Yield-Based Risk](chapters/chapter_06_ytm_yield_based_risk.md), [Chapter 8 — Spreads 101](chapters/chapter_08_spreads_101.md), [Chapter 9 — Repo — The Bond Market’s Funding Engine](chapters/chapter_09_repo_funding_engine.md)

Follow-on: [Chapter 11 — DV01/PV01 — Definitions, Computation, and "What's Being Bumped"](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12 — Duration](chapters/chapter_12_duration_macaulay_modified_dv01.md), [Chapter 13 — Convexity](chapters/chapter_13_convexity.md), [Chapter 14 — Key-Rate DV01](chapters/chapter_14_key_rate_dv01_bucket_exposures.md), [Chapter 15 — DV01 Hedging](chapters/chapter_15_dv01_hedging.md), [Chapter 16 — Curve Hedging (Twists, Butterflies, PCA)](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md)

## Learning Objectives
- After this chapter, you can compute holding-period P&L/return using clean vs dirty prices, accrued interest, and coupon timing.
- You can compute net carry for a repo-financed bond position and translate it into a breakeven yield/price move.
- You can compute rolldown by repricing an aged bond on an unchanged curve/spread surface.
- You can decompose clean-price change into rolldown, benchmark-curve effect, and spread effect (exact via counterfactual repricing; approximate via DV01/DVOAS).
- You can interpret the decomposition as desk P&L explain and diagnose residuals/pitfalls (convexity, cross-terms, curve shape, funding assumptions).

This chapter develops the full machinery of return decomposition:

- **Section 7.1** establishes the foundation: clean vs dirty prices and the holding-period P&L identity.
- **Section 7.2** defines carry precisely, using Tuckman's repo-financed P&L framework, and shows how to compute breakeven price moves.
- **Section 7.3** introduces rolldown, pull-to-par, the forward rate as breakeven, the classic curve-riding trade, and the term premium framework.
- **Sections 7.4–7.5** develop the curve and spread effect decomposition, showing how to exactly attribute price changes to benchmark moves vs credit spread moves.
- **Section 7.6** addresses the residual, convexity, cross-terms, and crisis behavior that breaks clean decomposition.
- **Section 7.7** presents Tuckman's model-based P&L attribution framework.
- **Section 7.8** covers practical implementation, including attribution politics on the trading desk.

Throughout, we use explicit numerical examples and derive every formula from first principles. The goal is a decomposition you can implement and defend.

---

## 7.1 Clean Price, Dirty Price, and Holding-Period P&L

### 7.1.1 The Clean/Dirty Distinction

Before we can decompose returns, we must be precise about what "return" means. Bond markets quote **clean prices** but settle at **dirty prices**. The relationship is:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + AI}$$

where accrued interest for a semiannual coupon bond is:

$$\boxed{AI = \frac{c}{2} \cdot \frac{t - t_0}{t_1 - t_0}}$$

Here $c$ is the annual coupon rate (as a decimal), $t_0$ is the last coupon date, and $t_1$ is the next coupon date.

Market convention: the bond price quoted in the market is the **clean price**, while the amount exchanged at settlement is the **full/dirty price**. The difference is **accrued interest**, and it exists so the quoted clean price does not experience a mechanical drop when a coupon is paid.

**Why this matters for P&L:** Traders quote and hedge clean prices because clean prices don't jump mechanically on coupon dates. But P&L calculations must ultimately use dirty prices (plus coupons received) to capture actual cash flows. Many P&L explain errors trace back to mixing clean and dirty inconsistently—especially around coupon payment dates.

> **Pitfall — Clean vs dirty mixing:** Using clean price change as “P&L” without adding coupons and the accrued-interest change.
> **Why it matters:** Your P&L explain will show artificial “jumps” around coupon dates and mis-attribute carry vs MTM.
> **Quick check:** Verify that \(P_{\text{dirty}}(h)-P_{\text{dirty}}(0)+\sum C_n\) equals \(\Delta P_{\text{clean}} + \sum C_n + AI(h)-AI(0)\).

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

Tuckman derives a desk-style funded P&L identity in Chapter 15. Defining:

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

This identity cleanly separates the mark-to-market (price change) from carry (interest income minus financing cost)—the desk-style split used for P&L explain.

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

Note that the financing term is written as \(r\,d/360\) in this formulation. In practice, repo accrual uses a stated money-market day-count convention; always document it, because carry can differ slightly across systems that make different day-count assumptions.

**Intuition:** Carry tends to be positive when the bond's coupon rate exceeds its financing rate, but two nuances matter: (i) coupon income accrues on face value while repo interest is paid on the full (dirty) price, and (ii) the day count used for coupon accrual can differ from the day count used for repo interest.

### 7.2.3 Breakeven Price Moves

Carry enables a critical calculation: how much can the bond's price fall before the position loses money? This is the **breakeven price change**.

> **Back-of-Envelope Breakeven Algebra**
>
> If you know your Carry ($) and your Risk (DV01 in $), you can calculate your survival buffer in your head:
>
> $$\boxed{\text{Breakeven (bp)} \approx \frac{\text{Net Carry (\$)}}{\text{DV01 (\$)}}}$$
>
> *   **Example**:
>     *   Carry = +$10,000 per month.
>     *   DV01 = $1,000 per bp.
>     *   **Survival Buffer**: $10,000 / 1,000 = \text{10 bp}$.
>
> **Result**: You can survive a 10bp adverse move in rates over the next month and still break even.

Tuckman provides a worked example: An investor purchases a \(5\frac{7}{8}\%\) Treasury at invoice price 105.103073 and holds for 30 days at a 5.10% repo rate. The carry calculation proceeds:

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

A critical warning: positive carry does not imply superior expected return. In equilibrium-style framing, a fairly priced position has a required expected return (often described as the short-term rate plus a risk premium); that required return can show up as **carry**, as **expected price change**, or as a mix of the two.

This is why carry comparisons can be misleading. A premium bond can have positive carry while (under unchanged yields) its price drifts down toward par; a discount bond can have negative carry while its price drifts up toward par. Carry tells you about the *composition* of return, not the level of expected return.

### 7.2.6 Desk Conventions for Carry

In practice, "carry" can mean different things depending on context:

| Term | Definition |
|------|------------|
| **Income-only carry** | $\sum C_n + AI(h) - AI(0)$ (no funding adjustment) |
| **Net carry** | Income minus financing cost (the repo P&L identity) |
| **Hedged carry** | Net carry after accounting for the carry of hedges (swap, futures, etc.) |

When someone says "the trade has 12bp of carry," always clarify which definition they're using. Hedged carry conventions are instrument- and desk-specific.

---

## 7.3 Rolldown, Pull-to-Par, and the Forward Rate Framework

### 7.3.1 What Happens When "Nothing" Changes?

Even if rates and spreads remain unchanged, bond prices move. A bond held for six months is six months closer to maturity tomorrow than it was today. This passage of time affects price through two channels:

1. **Rolldown**: If the yield curve is upward-sloping, shorter-maturity bonds typically trade at lower yields. As a bond ages, it "rolls down" the curve into a lower-yield region, which can increase its price.

2. **Pull-to-par**: Bonds priced away from par converge toward 100 as maturity approaches. Premium bonds decline toward par; discount bonds rise toward par.

> **Analogy: Rolldown is Sledding**
>
> Think of the Yield Curve as a snowy hill.
> *   **Upward Sloping**: The hill goes down as you move forward (shorter maturity → lower yield).
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

Important caveat: this “pull-to-par” intuition is a *holding-yields-constant* thought experiment. Realized price paths depend on how yields actually evolve.

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

### 7.3.5 Forward Rates as the Breakeven for Maturity Strategies

A central question in fixed income is: should I buy a long-maturity bond and hold it, or buy a short-maturity bond and roll it? Tuckman provides the framework in Chapters 2–3 for comparing these strategies.

**The Setup:** Consider two strategies starting with $1 to invest for one year:

1. **Strategy A (Buy-and-Hold):** Buy a 1-year zero-coupon bond at today's 1-year spot rate $\hat{r}(1)$.
2. **Strategy B (Roll):** Buy a 6-month zero-coupon bond at today's 6-month spot rate $\hat{r}(0.5)$, then reinvest the proceeds in another 6-month bond at whatever the 6-month spot rate is in 6 months.

**The Question:** Under what rate scenario do these strategies produce the same return?

**The Derivation:**

Strategy A terminal value:
$$V_A = (1 + \hat{r}(1)/2)^2$$

Strategy B terminal value:
$$V_B = (1 + \hat{r}(0.5)/2) \times (1 + r_{0.5}/2)$$

where $r_{0.5}$ is the 6-month spot rate realized 6 months from now.

Setting $V_A = V_B$ and solving for $r_{0.5}$:

$$(1 + \hat{r}(1)/2)^2 = (1 + \hat{r}(0.5)/2) \times (1 + r_{0.5}/2)$$

Solving for the breakeven rate $r_{0.5}$:

$$r_{0.5}^{\text{breakeven}} = 2 \left[ \frac{(1 + \hat{r}(1)/2)^2}{1 + \hat{r}(0.5)/2} - 1 \right]$$

But from Tuckman's forward rate definition, the 6-month rate 6 months forward satisfies:

$$(1 + \hat{r}(0.5)/2) \times (1 + f(0.5,1)/2) = (1 + \hat{r}(1)/2)^2$$

Therefore:

$$\boxed{r_{0.5}^{\text{breakeven}} = f(0.5,1)}$$

**The forward rate is the breakeven rate for maturity choice.** If the future 6-month spot rate equals today's implied forward rate, then buying the 1-year zero and holding produces exactly the same return as buying the 6-month zero and rolling.

**Worked Example:**

Suppose:
- 6-month spot rate: $\hat{r}(0.5) = 5.00\%$
- 1-year spot rate: $\hat{r}(1) = 5.50\%$

The implied 6-month forward rate starting in 6 months:

$$f(0.5,1) = 2 \left[ \frac{(1.0275)^2}{1.025} - 1 \right] = 2 \left[ \frac{1.055756}{1.025} - 1 \right] \approx 2 \times 0.030004 = 6.00\%$$

**Interpretation:**
- If the 6-month rate in 6 months is **above 5.98%**, Strategy B (roll) wins.
- If the 6-month rate in 6 months is **below 5.98%**, Strategy A (buy-and-hold) wins.
- At exactly 5.98%, both strategies produce identical returns.

> **Desk Reality: The Forward Rate as Your Indifference Point**
>
> When a trader says "I'm buying the 10-year instead of rolling 5-years because I think the curve will stay steep," they're implicitly saying: "I don't believe the 5-year rate in 5 years will be as high as today's forward implies."
>
> The forward curve is your **null hypothesis**. Earning rolldown return means betting against the forwards—you believe rates won't rise as much as the curve implies.

### 7.3.6 Why "Carry + Rolldown" Is Not Guaranteed P&L

A common desk heuristic is: "Expected return under unchanged curve = carry + rolldown."

This is correct as a *conditional* expectation—conditional on nothing happening to rates. But realized P&L can be dominated by curve and spread shocks, which can easily overwhelm the carry and rolldown components. Tuckman emphasizes that "Part Three showed that the expected return of any fairly priced portfolio equals the short-term rate plus an appropriate risk premium"—not carry plus rolldown.

In other words, carry + rolldown is a counterfactual return, not a forecast. If the curve is upward-sloping because of term premia (not just expected rate increases), then systematically earning the rolldown may be compensation for bearing interest rate risk.

### 7.3.7 The Classic Curve-Riding Trade

One of the most common fixed income trading strategies is **riding the yield curve** (also called "roll-down trades"). The strategy exploits steep yield curves by buying longer-maturity instruments, holding them as they age into lower-yield territory, and selling.

**The Mechanics:**

1. **Entry:** Buy a bond at maturity $T$ with yield $y(T)$.
2. **Hold:** Wait for time $\Delta t$ to pass. The bond now has maturity $T - \Delta t$.
3. **Exit:** If the yield curve hasn't changed, the bond now yields $y(T - \Delta t)$. On an upward-sloping curve, $y(T - \Delta t) < y(T)$, so the price has risen beyond what carry alone would provide.

**Worked Example: Riding 2-Year to 1.5-Year**

Suppose the yield curve is:

| Maturity | Yield |
|----------|-------|
| 1.5 years | 4.20% |
| 2.0 years | 4.50% |

You buy a 2-year zero-coupon bond at 4.50% yield. Price per $100 face:

$$P_0 = \frac{100}{(1 + 0.045/2)^4} = 91.484$$

After 6 months, **if the curve doesn't change**, the bond is now a 1.5-year zero yielding 4.20%:

$$P_{0.5} = \frac{100}{(1 + 0.042/2)^3} = 93.956$$

**Rolldown return** (6-month holding):

$$\frac{93.956 - 91.484}{91.484} = 2.70\%$$

Annualized, this is roughly 5.4%—significantly more than the 4.50% yield would suggest from "holding to maturity" logic.

> **Desk Reality: "Buying 2s and Selling as 1.5s"**
>
> Traders express this trade as "buying 2-year paper and selling it as 1.5-year paper." The strategy works when:
> - The curve is steep enough (large yield differential)
> - The curve doesn't flatten or invert
> - Financing costs don't eat the rolldown

**When the Trade Fails:**

The risk is **curve flattening**. If the 1.5-year yield rises to 4.50% while you're holding:

$$P_{0.5}^{\text{flat}} = \frac{100}{(1 + 0.045/2)^3} = 93.543$$

Return: $\frac{93.543 - 91.484}{91.484} = 2.25\%$ (annualized ~4.5%)

You've earned less than the initial yield—the curve flattening eroded your rolldown.

**Extreme Case: Curve Inversion**

If the curve is inverted, “aging” can move you into *higher* yields rather than lower yields. In that case the unchanged-curve counterfactual can produce **negative rolldown** (you are effectively rolling *up* the curve).

**P&L Scenarios Table (Zero-Coupon Illustration):**

Entry: 2-year yield = 4.50%, entry price $P_0 = 91.484$.

Your **breakeven exit yield** is the 1.5-year yield that makes the exit price equal to $P_0$ (zero P&L). In this setup it’s about **6.02%**—very close to the “implied forward” breakeven logic from the prior section.

| Scenario | 1.5Y Yield at Exit | Exit Price | Price Change | 6M Return |
|----------|--------------------|------------|--------------|-----------|
| Stable curve (rolldown benefit) | 4.20% | 93.956 | +2.471 | +2.70% |
| Flat curve (no rolldown) | 4.50% | 93.543 | +2.058 | +2.25% |
| Breakeven (forward-ish) | 6.02% | 91.484 | 0.000 | 0.00% |
| Adverse sell-off | 6.50% | 90.851 | -0.633 | -0.69% |

### 7.3.8 Expectations vs Risk Premia: Why Rolldown May Be Compensation

In “Maturity and Bond Return,” Tuckman notes that when the **six-month rate evolves according to the initial forward rate curve**, investors rolling short-term bonds and investors buying long-term bonds perform equally well. More generally, he states that rollover investors do better when realized short-term rates exceed the forward rates built into bond prices, while longer-maturity bond investors do better when realized short-term rates fall below those forwards.

But this breakeven logic does *not* imply that forward rates are forecasts of future spot rates. Jarrow gives a simple example where \(f(0,T) \neq E[r(T)]\), and where the slope of the forward curve does not forecast the direction of expected future spot rates.

Tuckman then frames the wedge in expected-return terms: risk-neutral investors require an expected return equal to the short rate, while risk-averse investors demand a risk premium proportional to interest-rate risk (often proxied by duration). In his stylized formulation (Equation 10.25):

$$E\!\left[\frac{dP}{P}\right] + \frac{c\,dt}{P} = r\,dt + \lambda D\,dt$$

From a data perspective, Tuckman notes an identification problem: increasing the risk premium by a fixed amount can be empirically indistinguishable from increasing the expected path of short rates by the same amount. So the term structure at a point in time does not, by itself, uniquely separate “expectations” from “risk premium.”

> **Desk Reality — “Carry + roll” is a risk position:**
> Carry and rolldown are computed under a counterfactual (“unchanged curve/spread”). Your realized P&L is dominated by what actually happens to the curve, spreads, and funding.
> **What to check:** stress test the trade under a flattening and an inversion scenario; verify that the expected carry/roll is large enough relative to the DV01 you are warehousing.

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

For small moves, we often linearize the curve effect using a one-basis-point sensitivity.

**Yield-based definition (DV01; sign convention):** In the book-wide convention (matching Chapter 11), DV01 is the change in full price for a 1bp **decline** in the chosen rate measure, so a long option-free bond typically has a positive DV01:

$$\boxed{\text{DV01} := P(y-1\text{bp})-P(y) \approx -\frac{\partial P}{\partial y}\cdot (1\text{bp})}$$

**Curve-bump DV01 for attribution (bump object):** in this chapter’s curve/spread decomposition, the object being “bumped” is the **benchmark curve** \(z(\cdot)\) (parallel shift), holding the spread \(s\) fixed. Define the curve DV01 at a valuation point (e.g., at horizon \(h\)) by:

$$\boxed{\text{DV01}_z(h) := P_{\text{clean}}(h;\,z(\cdot)-1\text{bp},\,s)-P_{\text{clean}}(h;\,z(\cdot),\,s)}$$

Units: **price points per 1bp per 100 notional** (multiply by \(N/100\) to convert to currency DV01 for notional \(N\)).

With the sign convention above, a parallel *up* move of \(\Delta z_{\text{bp}}>0\) gives the first-order approximation:

$$\boxed{\Delta P_{\text{curve}} \approx -\text{DV01}_z \cdot \Delta z_{\text{bp}}}$$

### 7.4.3 Second-Order Approximation: Convexity

For larger moves, the linear approximation understates gains and overstates losses (for positive-convexity bonds). Tuckman provides the second-order approximation in Chapter 5:

$$\boxed{\frac{\Delta P}{P} \approx -D \cdot \Delta y + \frac{1}{2} \cdot Cvx \cdot (\Delta y)^2}$$

where $D$ is modified duration and $Cvx$ is convexity. This is Tuckman's equation (5.20).

The convexity term is always positive for option-free bonds: the price-yield relationship is curved, and the curvature benefits the bondholder. With positive convexity, the convexity term increases return for large moves in either direction.

---

## 7.5 Spread Change Effect

### 7.5.1 Defining the Spread Effect

After accounting for curve moves, any remaining price change comes from spread changes. Holding the benchmark curve fixed at its horizon value:

$$\boxed{\text{Spread effect} := P_{\text{clean}}(h; \, z_1(\cdot), \, s_1) - P_{\text{clean}}(h; \, z_1(\cdot), \, s_0)}$$

where $s_0$ and $s_1$ are the spread at time 0 and horizon, respectively.

### 7.5.2 Spread DV01 and DVOAS

Just as curve effects can be approximated by a curve DV01, spread effects can be approximated by a spread sensitivity.

**Definition (bump object):** Tuckman defines DVOAS as the sensitivity of model price to a 1bp **decrease** in OAS, computed analogously to DV01 (central difference). In this chapter we use the same “tightening bump” convention for any additive spread \(s\) (Z-spread or OAS, as stated):

$$\boxed{\text{DVOAS}(h) := P_{\text{clean}}(h;\,z(\cdot),\,s-1\text{bp})-P_{\text{clean}}(h;\,z(\cdot),\,s)}$$

Units: **price points per 1bp per 100 notional** (multiply by \(N/100\) for currency DVOAS on notional \(N\)).

With \(\Delta s_{\text{bp}}>0\) meaning *spread widening*, the first-order approximation is:

$$\boxed{\Delta P_{\text{spread}} \approx -\text{DVOAS} \cdot \Delta s_{\text{bp}}}$$

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
2. **Cross-terms**: If both curve and spread move, a Taylor expansion includes a $\frac{\partial^2 P}{\partial z \,\partial s} \cdot dz \cdot ds$ interaction term
3. **Non-parallel moves**: DV01 assumes parallel shifts; actual curve changes may twist or butterfly

A second-order expansion explains why the residual tends to shrink for smaller shocks and grow with longer duration, higher convexity, and larger rate/spread moves.

### 7.6.3 When Decomposition Breaks Down: Crisis Behavior

Hull’s discussion of the 2007–2009 financial crisis is a useful mental model: he describes a “flight to quality” and liquidity problems for institutions that relied on short-term funding. He also notes that credit spreads increased dramatically during the crisis. In such regimes, curve moves, spread moves, and funding conditions can all shift at the same time.

For a corporate bond, that can produce offsetting first-order components:
- benchmark yields down (a rally) → **positive curve effect**
- credit spreads up (wider) → **negative spread effect**

Two additional effects then make attribution harder:
1. **Big moves:** both first-order and second-order approximations work well for very small changes in rate; for larger changes the second-order (convexity-adjusted) approximation works better, but for very large changes it, too, can fail.
2. **Joint moves:** when both the curve and the spread move, a second-order expansion includes an interaction term proportional to \(dz\,ds\) (often called a “cross-gamma”).

When the residual in your P&L explain becomes large, treat DV01/DVOAS × bp as a rough approximation and lean on **exact repricing** via counterfactual curves (curve-only, spread-only) to validate the explain.

> **Desk Reality:** in crisis regimes, treat component attribution as approximate.
> **What to check:** compute the exact counterfactual prices (curve-only, spread-only) and compare them to the linear approximation; if the residual is large, focus first on total P&L and net risk.

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

Tuckman calls the spread-move component “convergence” because, in a model-based view, a positive OAS represents mispricing relative to that model and (with predictive power) should tend to move back toward fair value.

**Understanding Convergence:**

In this framing, a “cheap” security (positive OAS) earns extra return in two ways: (1) by accruing OAS over time while the mispricing persists, and (2) by earning when OAS narrows, scaled by DVOAS.

**When OAS-Based Attribution Makes Sense:**

OAS-based P&L attribution requires a pricing model. This is common for:
- **MBS desks**: where prepayment optionality is significant
- **Callable bond portfolios**: where call options affect duration
- **Complex structured products**: where embedded options matter

For vanilla bullet bonds, simpler decompositions (carry + rolldown + curve + spread) often suffice.

> **Desk Reality: Model Risk in OAS Attribution**
>
> Different OAS models give different attribution. An MBS trader using one prepayment model might show "convergence gains" while another model shows "factor losses." Always document which model underlies your OAS-based P&L explain.

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

**Example Title**: Full 6-month return decomposition for a bullet bond (unfunded vs repo-funded)

**Context**
- We want a desk-style P&L explain: carry, rolldown, benchmark-curve effect, spread effect, and the residual from linear approximations.
- This is the “quote → cashflows → PV → risk” pipeline in one concrete example.

**Timeline (Make Dates Concrete)**
- Trade date: Jul 30, 2026
- Settlement date: Jul 30, 2026 (assume settlement on a coupon date, so \(AI(0)=0\))
- Accrual start/end: Jul 30, 2026 → Jan 30, 2027
- Payment date(s): Jan 30, 2027 (coupon)
- Horizon date: Jan 30, 2027 (coupon date, so \(AI(1)=0\))

**Inputs**
- Instrument: 5.00% annual coupon bond, semiannual coupons, face 100, maturity Jul 30, 2028 (2.0y from trade date)
- Pricing conventions: prices per 100 notional; discounting uses semiannual-compounded rates
- Curves/spreads: use the benchmark zero curve and additive Z-spread from Example C
- Financing (repo-funded variant): repo rate \(r=4.50\%\) with money-market style \(d/360\) accrual

**Outputs (What You Produce)**
- Clean-price components (per 100): carry (unfunded and repo-funded), rolldown, curve effect, spread effect, residual
- Total P&L (per 100) and holding-period return (unfunded and repo-funded)
- Risk (as used in the approximation): \(DV01_z\) at the horizon (price points per bp per 100)

**Step-by-step**
1. Record the initial quote and settlement convention (\(P_0^{clean}\), \(AI(0)\), horizon \(h\)).
2. Compute carry over the horizon (unfunded vs repo-funded using the stated day counts).
3. Reprice the aged bond on the **unchanged** curve/spread to get rolldown.
4. Reprice on the shifted **benchmark curve** (spread fixed) to get the curve effect.
5. Reprice on the shifted **spread** (curve fixed at the shifted level) to get the spread effect.
6. Check the telescoping identity, then compute total P&L and holding-period return.
7. Compute \(DV01_z(h)\), compare linear vs exact effects, and interpret the residual.

**Cashflows (table; unfunded, per 100 notional)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-07-30 | \(-99.0831\) | Buy bond at dirty price (here \(=P^{clean}\) since \(AI(0)=0\)) |
| 2027-01-30 | \(+2.50\) | Coupon received |
| 2027-01-30 | \(+99.1840\) | Sell bond at dirty price (here \(=P^{clean}\) since \(AI(1)=0\)) |

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

**P&L / Risk Interpretation**
- Clean-price explain (per 100): rolldown \(+0.4264\) is mostly offset by rates \(-0.1134\) and spread widening \(-0.2121\), leaving only \(+0.1009\) of net clean-price change.
- Unfunded: the coupon dominates total P&L (\(+2.50\) income + \(+0.1009\) MTM \(= +2.6009\) points over 6 months).
- Repo-funded: financing absorbs most of the coupon (carry drops to \(+0.2706\)), so the same MTM now drives a much smaller total (\(+0.3716\)). This is why “carry trades” are funding-sensitive.
- Risk view: with \(DV01_z(h)\approx 0.0142\) points per bp per 100, an 8bp parallel sell-off costs about \(0.11\) points—consistent with the curve effect; the residual (~0.0007) is the second-order remainder (convexity + cross-terms) at these move sizes.

**Sanity Checks**
- Units check: all price/P&L numbers are “per 100 notional”; convert to currency by multiplying by \(N/100\).
- Sign check: rates/spreads up → price down for a long option-free bond.
- Limit check: as \(\Delta z,\Delta s \to 0\), linear (DV01/DVOAS) and exact repricing should coincide and the residual should go to zero.
- Reproduction check: a spreadsheet that reprices the cashflows under the three counterfactual curves (\(z_0,s_0\)), (\(z_1,s_0\)), (\(z_1,s_1\)) should recover the same components.

**References**
- (Bruce Tuckman, *Fixed Income Securities: Tools for Today’s Markets*, “Carry”)
- (Bruce Tuckman, *Fixed Income Securities: Tools for Today’s Markets*, “Estimating Price Changes and Returns with DV01, Duration, and Convexity”)

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
| **Day count mismatch in carry** | Repo uses Actual/360; Treasury coupon accrual uses Actual/Actual. This mismatch can create small carry calculation discrepancies. Verify against your system's conventions and document the day count basis used for each leg. |

> **Practitioner Note on Multi-Curve:**
>
> Post-2008 markets use multi-curve frameworks: discounting typically uses OIS (SOFR for USD), while bond coupons are fixed (not floating). Your "benchmark curve" choice affects both curve effect and spread effect attribution.
>
> Example: If you decompose a corporate bond's return using Treasury curves as the benchmark, "spread" captures everything non-Treasury. If you use swap curves, "spread" captures credit risk but not swap-Treasury basis. Document your choice.

### 7.8.3 Verification Tests (Quick Sanity Checks)

| Test | Expected Result |
|------|-----------------|
| **If curve and spread are unchanged** | Total return ≈ carry + rolldown (within rounding) |
| **Sign checks** | Higher yields/spreads → lower prices (long bond loses) |
| **Approximation consistency** | Residual shrinks as shocks become small (linearization becomes accurate) |

### 7.8.4 Attribution Politics: How P&L Explain Is Used on the Desk

P&L attribution isn't just a technical exercise—it's a communication tool with incentive implications. Understanding how attribution is used (and potentially misused) is essential for anyone moving from middle office to front office.

**The Predictable vs Unpredictable Frame:**

Traders and portfolio managers love to frame their P&L in terms of "predictable" vs "unpredictable" components:

| Component | Perception | Reality |
|-----------|------------|---------|
| **Carry** | "Predictable"—the coupon income I earned | True for income; funding cost varies |
| **Rolldown** | "Skill"—I positioned for the curve shape | Only materializes if curve doesn't move |
| **Curve moves** | "Market"—can't control the Fed | True, but duration is a choice |
| **Spread moves** | "Credit"—macro driven | True, but credit exposure is a choice |

> **Desk Reality: The Attribution Game**
>
> - **Good Month:** "We captured carry and rolldown as expected. Our duration positioning was also well-timed."
> - **Bad Month:** "The market moved against us unexpectedly. Our carry and rolldown were in line with plan."
>
> Notice how "carry and rolldown" are always claimed while rate/spread moves are disclaimed when unfavorable.

**How Risk Officers Should Evaluate Attribution:**

1. **Compare ex-ante to ex-post:** Before the period, what did the PM *say* they expected from carry/rolldown vs rate views? After the period, does the attribution match the stated strategy?

2. **Look at the residual:** A persistently large residual suggests the PM is taking risks not captured by simple DV01 measures (curve shape bets, convexity bets, basis trades).

3. **Consider the counterfactual:** If carry + rolldown was +$1M but rate moves were -$3M, the PM was running a directional rate bet. The "predictable" carry doesn't offset the directional loss—it was always part of the directional position.

4. **Track through time:** If a PM consistently "earns" carry and rolldown but has high rate/spread volatility, they're taking more risk than the carry suggests. Sharpe ratios tell the truth.

> **Practitioner Note:** Good attribution systems show both ex-ante (budgeted/expected) and ex-post (realized) components. This makes it harder to reframe outcomes after the fact.

---

## Summary

This chapter developed a complete framework for decomposing bond returns:

1. **Dirty = clean + accrued interest**: Use dirty prices (or clean + AI consistently) to compute horizon returns; coupon timing matters.

2. **Carry = income − funding**: Tuckman's repo P&L identity separates mechanical carry from mark-to-market. Carry is useful for computing breakeven price moves.

3. **Carry is not expected return**: Positive carry does not imply superior expected return. Premium bonds have positive carry but prices pulled toward par.

4. **Rolldown = aging on unchanged curve**: Define rolldown operationally as the clean price change from revaluing the aged bond on the unchanged curve/spread surface.

5. **Forward rate = breakeven for maturity choice**: If the future spot rate equals today's forward, strategies of different maturities produce equal returns. Earning rolldown means betting against forwards.

6. **Term premium may explain rolldown**: If term premium is positive, systematic rolldown returns may be compensation for duration risk, not alpha.

7. **Exact decomposition via counterfactuals**: Clean price change = rolldown + curve effect + spread effect (a telescoping identity).

8. **DV01/DVOAS approximations**: Curve and spread effects are approximately linear for small moves; convexity matters for larger shocks.

9. **Crisis behavior breaks decomposition**: When rates and spreads move together, cross-gamma terms dominate and the linear approximation fails.

10. **Attribution politics matter**: Understand how PMs use P&L attribution to frame performance; evaluate ex-ante vs ex-post and watch the residual.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Dirty price | Clean + accrued interest | What you actually pay/receive |
| Carry | Interest income minus financing cost | Mechanical P&L component; determines breakevens |
| Rolldown | Price change from aging on unchanged curve | Captures curve shape + pull-to-par effects |
| Forward rate as breakeven | $f(t,T)$ = rate at which maturity strategies are indifferent | Theoretical anchor for rolldown analysis |
| Term premium | Forward − expected future spot | May explain why rolldown is "earnable" |
| Curve effect | Price change from benchmark moves | Rate risk attribution |
| Spread effect | Price change from spread moves | Credit/liquidity risk attribution |
| DV01 | PV change for a 1bp **decline** in the stated rate/curve bump object (units: price points per bp per 100) | First-order approximation for rate moves and hedges |
| DVOAS | PV change for a 1bp **tightening** in the stated spread/OAS bump object (units: price points per bp per 100) | First-order approximation for spread moves and hedges |
| Residual | Exact minus approximate | Convexity + cross-terms + model error |
| Cross-gamma | $\frac{\partial^2 P}{\partial z \,\partial s}$ (interaction between curve and spread) | Interaction term; can matter when curve and spreads move together |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(P_{\text{clean}}(t)\) | Clean price at time \(t\) | price points per 100 notional; market quote convention |
| \(P_{\text{dirty}}(t)\) | Dirty/full price at time \(t\), \(=P_{\text{clean}}(t)+AI(t)\) | price points per 100; settlement amount |
| \(AI(t)\) | Accrued interest at time \(t\) | price points per 100; computed per the bond’s coupon day count |
| \(c\) | Annual coupon rate | decimal per year (e.g., 0.05 for 5%) |
| \(C_n\) | Coupon payment during the horizon | price points per 100; positive cashflow for a long position |
| \(h\), \(d\) | Horizon length | years or days; state the day count used (e.g., 30/360, Act/360) |
| \(r\) | Repo rate | decimal per year; financing accrues on a money-market basis (often Act/360) |
| \(z(t)\) | Benchmark zero rate for maturity \(t\) | rate per year; compounding convention must match the discounting formula used |
| \(s\) | Additive spread (Z-spread / OAS as stated) | rate per year; widening is \(\Delta s_{\text{bp}}>0\) |
| \(y(t)\) | Total discount rate, \(=z(t)+s\) (in this chapter) | rate per year |
| \(f(t_1,t_2)\) | Forward rate from \(t_1\) to \(t_2\) | rate per year; defined consistently with the spot/discounting convention |
| \(df(t)\) | Discount factor to maturity \(t\) | unitless; \(df(0)=1\) |
| \(DV01\) | Rate/curve PV sensitivity | **Book convention:** \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\) for the stated bump object; units: price points per bp per 100 |
| \(DVOAS\) | Spread/OAS PV sensitivity | **Book convention:** \(DVOAS := PV(\text{spread tightens }1\text{bp})-PV(\text{base})\) for the stated spread definition; same units as \(DV01\) |
| \(D\) | Modified duration | years; connects to \(DV01\) via scaling (given a stated compounding convention) |
| \(Cvx\) | Convexity | year\(^2\) (conceptually); used in second-order price-change approximations |

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
| 8 | What is DV01 (sign convention)? | $\text{DV01} := P(y-1\text{bp})-P(y) \approx -\frac{1}{10000}\frac{\partial P}{\partial y}$ (price change for a 1bp *fall*) |
| 9 | How do you use DV01 to approximate a price change for $\Delta y_{\text{bp}}$? | $\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}}$ |
| 10 | What does convexity correct in DV01-based approximations? | Nonlinearity of the price–yield relationship; linear DV01 misses 2nd-order effects |
| 11 | State the duration/convexity approximation for $\Delta P/P$ | $\Delta P/P \approx -D\Delta y + \frac{1}{2}Cvx(\Delta y)^2$ |
| 12 | What is the forward rate's role in maturity choice decisions? | The forward rate is the breakeven: if the future spot rate equals today's forward, strategies of different maturities produce equal returns |
| 13 | What is term premium? | The difference between the forward rate and the expected future spot rate; compensates for duration risk |
| 14 | Why might systematically earning rolldown be fair compensation? | If term premium is positive, the yield curve is "too steep" relative to rate expectations; rolldown is compensation for bearing interest rate risk |
| 15 | What is DVOAS (sign convention)? | $\text{DVOAS} := P(s-1\text{bp})-P(s)$ (price change for a 1bp *tightening* in the stated spread/OAS) |
| 16 | What is the "curve effect" in horizon clean-price explain? | Price change at horizon from changing benchmark curve $z_0 \to z_1$ holding spread fixed |
| 17 | What is the "spread effect" in horizon clean-price explain? | Price change at horizon from changing spread $s_0 \to s_1$ holding benchmark curve fixed |
| 18 | Why can carry+rolldown be positive while realized P&L is negative? | Adverse curve and/or spread moves can overwhelm carry and rolldown |
| 19 | What is the "riding the yield curve" trade? | Buy long-maturity bonds, hold as they age into lower-yield territory (on a steep curve), sell for capital gain |
| 20 | When does riding the yield curve fail? | When the curve flattens or inverts; the aging bond rolls into higher yields, producing capital losses |
| 21 | What is cross-gamma and when does it matter? | $\frac{\partial^2 P}{\partial z \,\partial s}$; matters when curve and spreads move together (often in stress regimes) |
| 22 | When should you distrust your P&L decomposition? | When the residual is a meaningful fraction of total P&L; indicates non-parallel moves, convexity/cross-terms, or model mismatch |
| 23 | Why does positive carry not imply superior expected return? | Premium bonds have positive carry but prices pulled down toward par; discount bonds have negative carry but prices pulled up |
| 24 | What is a breakeven price change? | The price move that exactly offsets carry, resulting in zero P&L |
| 25 | What three components appear in Tuckman's model-based P&L attribution? | Carry, factor exposure, and convergence |
| 26 | What is "convergence" in OAS-based P&L? | Return from the bond's OAS normalizing toward zero (the model's fair value) |
| 27 | How do desks use P&L attribution strategically? | Claim "predictable" carry/rolldown for good performance; blame "unpredictable" rate moves for losses |
| 28 | What day count mismatch affects carry calculations? | Repo accrues on a money-market day-count basis (often 360-day based) while coupon accrual uses the bond’s coupon day count; mismatches create small carry discrepancies. |

---

## Mini Problem Set

**Problem 1:** A bond has clean price 99.20 and accrued interest 1.10. What is dirty price?

**Problem 2:** A semiannual 6% bond has 120 days elapsed in a 180-day coupon period (30/360). Compute accrued interest per 100.

**Problem 3:** A bond is bought at clean 101.80 with AI 1.50 and sold at clean 101.50 with AI 2.50. No coupon is received. Compute P&L and horizon return on dirty price.

**Problem 4:** Using the repo carry formula, compute repo-funded carry over 60 days when dirty price is 103.30, repo rate is 4.5%, and interest income is 1.00.

**Problem 5:** For a bond with DV01 = 0.0186, estimate the price change for a +12bp yield move.

**Problem 6:** A bond's exact price change for +25bp is −0.4633. DV01 approximation gives −0.4647. What does the difference represent?

**Problem 7:** Define an exact telescoping decomposition of clean price change into rolldown, curve effect, and spread effect using counterfactual horizon prices.

**Problem 8:** Suppose the 6-month spot rate is 5.00% and the 1-year spot rate is 5.50%. Calculate the implied forward rate for months 6-12 and interpret it as a breakeven.

**Problem 9:** Explain two reasons why "carry + rolldown" can be a poor forecast of realized P&L.

**Problem 10:** A PM claims: "I earned rolldown consistently because I'm skilled at curve positioning." What counter-argument might a skeptic make based on term premium?

**Problem 11:** In a flight-to-quality event, Treasury yields fall 50bp while corporate spreads widen 100bp. Qualitatively, what happens to each decomposition component for a corporate bond? Why might the residual be large?

**Problem 12:** If both benchmark curve and spread move, what second-order interaction term can appear in a Taylor approximation?

**Problem 13:** A trader says "this bond has 50bp of carry per year." What clarifying questions should you ask?

**Problem 14:** A PM's P&L explain shows: Carry +$200K, Rolldown +$100K, Rates -$500K, Spread -$150K, Residual -$50K. Total: -$400K. How would you evaluate this PM's attribution narrative if they blame the loss on "unexpected rate moves"?

### Brief Solution Sketches

**1.** $P_{\text{dirty}} = 99.20 + 1.10 = 100.30$

**2.** Semiannual coupon $= 3.00$. AI $= 3.00 \cdot (120/180) = 2.00$

**3.** Dirty start $= 103.30$. Dirty end $= 104.00$. P&L $= 0.70$. Return $= 0.70/103.30 \approx 0.678\%$

**4.** Financing cost $= 103.30 \cdot 0.045 \cdot (60/360) = 0.77475$. Repo-funded carry $= 1.00 - 0.77475 = 0.22525$

**5.** $\Delta P \approx -0.0186 \cdot 12 = -0.2232$ price points

**6.** The difference is primarily convexity (nonlinearity) and any mismatch between the sensitivity point and the actual move; it is the "residual."

**7.** Define $P_0 = P_{\text{clean}}(0; z_0, s_0)$, $P_h^{\text{uc}} = P_{\text{clean}}(h; z_0, s_0)$, $P_h^{\text{curve}} = P_{\text{clean}}(h; z_1, s_0)$, $P_h^{\text{actual}} = P_{\text{clean}}(h; z_1, s_1)$. Then rolldown $= P_h^{\text{uc}} - P_0$, curve effect $= P_h^{\text{curve}} - P_h^{\text{uc}}$, spread effect $= P_h^{\text{actual}} - P_h^{\text{curve}}$, and they sum to $P_h^{\text{actual}} - P_0$.

**8.** Forward rate: $f = 2[(1.0275)^2/1.025 - 1] \approx 6.00\%$. Interpretation: If the 6-month rate in 6 months equals about 6.00%, buying the 1-year zero produces the same return as rolling 6-month zeros.

**9.** (1) Adverse curve moves can overwhelm positive carry; (2) Spread widening can dominate all other components. Carry+rolldown is conditional on "nothing happening," which is rarely the case.

**10.** The skeptic would argue: "The upward-sloping curve reflects term premium—investors demand extra yield for duration risk. Your 'rolldown earnings' are compensation for that risk, not skill. When term premium inverts or compresses, you'll give it all back."

**11.** Rates falling → positive curve effect (long bond gains). Spreads widening → negative spread effect (corporate loses credit value). Net effect depends on magnitudes. The residual may be large because the two moves are correlated (flight-to-quality), so the interaction term $\frac{\partial^2 P}{\partial z \,\partial s} \cdot dz \cdot ds$ becomes significant.

**12.** A cross-term $\frac{\partial^2 P}{\partial z \,\partial s} \cdot dz \cdot ds$ appears in the second-order expansion, representing the interaction between curve and spread moves.

**13.** Is that income-only carry or net carry? What repo rate assumption? What day count? Annualized how (simple or compounded)? Is it carry per 100 face or per dollar invested?

**14.** The PM lost $400K total with $200K carry + $100K rolldown = $300K of "predictable" income. But rate exposure lost $500K and spread exposure lost $150K. Questions: (1) Was this duration intentional or accidental? (2) What was the ex-ante duration budget? (3) If the PM was "just capturing carry," why is there $500K of rate exposure P&L? The narrative that "rate moves were unexpected" doesn't address whether the PM should have had this much duration exposure in the first place. The residual of -$50K is small (~12% of component sum), so the decomposition is reasonably clean—the issue is the directional bet, not the attribution methodology.

---

## References

- (Bruce Tuckman, *Fixed Income Securities: Tools for Today’s Markets*, “Carry”; “Maturity and Bond Return”; “Yield-Based DV01”; “DVOAS”; “Pull to Par”)
- (John C. Hull, *Options, Futures, and Other Derivatives*, “Forward Rates”; “Duration and Convexity”)
- (Edwin J. Elton, Martin J. Gruber, Stephen J. Brown, and William N. Goetzmann, *Modern Portfolio Theory and Investment Analysis*, “Bond Pricing (Clean vs Dirty)”; “Price Change due to Passage of Time”)
- (Dessislava Pachamanova and Frank J. Fabozzi, *Simulation and Optimization in Finance*, “Total Return and Holding-Period Return”)
- (Robert A. Jarrow, *Modeling Fixed Income Securities and Interest Rate Options*, “Expectations Hypothesis and Forward Rates”)
- (Leif B. G. Andersen and Vladimir V. Piterbarg, *Interest Rate Modeling*, “Bonds and Forward Rates”)
- (David G. Luenberger, *Investment Science*, “Term Structure, Forward Rates, and Term Premia”)
