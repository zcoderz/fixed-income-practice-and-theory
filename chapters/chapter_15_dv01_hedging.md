# Chapter 15: DV01 Hedging — Hedge Ratios, Risk Weights, and Practical Caveats

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- DV01 definition and relationship to modified duration (Tuckman Ch 5-6)
- Bond-bond hedge ratio formula (Tuckman (5.7))
- Swap PV01 as annuity (Tuckman Ch 18)
- Duration-based futures hedge ratio (Hull Ch 6)
- Yield-based DV01 hedging assumes yields move together (Tuckman explicit limitation)
- Risk weights and regression-based hedging motivation (Tuckman)
- Duration-convexity approximation (Hull)

### (B) Reasoned Inference (Derived from A)
- Futures DV01 scaling via conversion factor (derived from Tuckman's CTD/invoice-price mechanics)
- Variance-minimizing hedge formulation (derived from Tuckman's regression-based hedging motivation)
- CS01/spread duration decomposition for credit products (derived from credit derivatives text)

### (C) Speculation (Clearly Labeled; Minimal)
- Specific covariance numbers in Example I are toy/illustrative
- Swap spread shock scenario in Example G is a toy basis scenario

---

## Conventions & Notation

### Rates and Bumps

| Convention | Description |
|------------|-------------|
| Rate units | Decimal unless stated (e.g., 5% = 0.05) |
| 1 bp | 0.0001 in decimal rate units |
| "+1 bp" bump | Decimal change $\Delta y = +0.0001$ |

### Prices and Units

| Convention | Description |
|------------|-------------|
| Bond price $P$ | Full (dirty) price per 100 face unless stated |
| Notional/face | Currency (e.g., USD) unless stated |

### DV01 Sign Convention (Default)

- **DV01 is positive** for a standard long, fixed-rate bond (price falls when yields rise)
- With DV01 defined as in Tuckman, the approximate PV change for a small yield increase is:

$$\Delta P \approx -\text{DV01} \times (\Delta y_{\text{bp}})$$

where $\Delta y_{\text{bp}}$ is the yield move in basis points.

### Bump Method

- Unless otherwise stated, "DV01" in worked examples is computed as a **yield-based** (yield-to-maturity / quoted yield) $\pm 1$ bp symmetric bump, consistent with Tuckman's definition and finite-difference discussion.
- When we use curve-based bumps (spot/zero/par-quote with rebuild), we label them explicitly.

### Swaps and PV01

- **PV01** refers to the change in swap value for a 1 bp change in the fixed rate (the swap annuity)
- For plain-vanilla swaps, PV01 equals the sum of discount factors (with accrual factors) used to discount fixed cash flows (times notional and 1 bp)

### Futures

- Treasury bond/note futures hedging uses the **conversion factor** and **cheapest-to-deliver (CTD)** ideas
- The "invoice price" relationship is used for scaling

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $P$ | Bond full (dirty) price per 100 face (unless stated) |
| $y$ | Yield (decimal) |
| $\Delta y$ | Yield change (decimal) |
| $\Delta y_{\text{bp}} = 10{,}000 \Delta y$ | Yield change in basis points |
| $\text{DV01}$ | Dollar value of 1 bp, per 100 face unless otherwise stated |
| $\text{DV01}^{\$}$ | DV01 in currency units for a given position size |
| $D_{\text{mod}}$ | Modified duration (dimensionless) |
| $F$ | Face/notional amount of a bond position (currency) |
| $N$ | Notional of a swap or derivatives position (currency) |
| $\text{PV01}$ | Change in swap value for 1 bp change in fixed rate (currency per bp) |
| $\text{CF}$ | Futures conversion factor |
| $\text{DV01}_{\text{CTD}}$ | DV01 of cheapest-to-deliver bond |
| $\text{CS01}$ | Credit spread 01 (PV change for 1 bp change in credit spread) |

---

## Core Concepts

### 1) DV01 (Dollar Value of a Basis Point)

**Formal Definition:**

For a bond with price $P(y)$, DV01 is defined (in Tuckman) as:

$$\boxed{\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}}$$

where $\Delta y$ is a yield change in decimal, so $10{,}000 \Delta y$ is the change in basis points.

**Intuition:**

- DV01 is the local slope of the price-yield curve expressed in "dollars per 1 bp"
- If DV01 is large, the instrument's PV is very sensitive to small rate changes
- With our sign convention, long fixed-rate bonds have positive DV01, and a +1 bp yield rise causes a PV loss $\approx$ DV01

**Trading / Risk / Portfolio Practice:**

- Risk reports often show "DV01" (or "PV01") as the primary first-order rate risk number
- Traders use DV01 for quick hedge ratios ("how many 5y notes to short against this 10y long?")
- Portfolio managers monitor DV01 to control duration exposure (e.g., keep near benchmark DV01)

---

### 2) DV01-Neutral / DV01-Hedged (Precise Meaning)

**Formal Definition:**

A portfolio is **DV01-hedged** (or DV01-neutral) with respect to a specified bump definition if the first-order PV change under a +1 bp bump is approximately zero:

$$\Delta V \approx 0 \quad \text{when applying the specified +1 bp bump}$$

Equivalently, under a linear approximation:

$$\sum_i \text{DV01}_i^{\$} \approx 0$$

where each $\text{DV01}_i^{\$}$ is the position-scaled DV01, computed under the same bump definition.

**Intuition:**

- DV01-neutral means "small parallel move protection (first order)"—but only for the particular "1 bp bump" you chose

**Trading / Risk / Portfolio Practice:**

- A "DV01-hedged book" is usually shorthand for "low net parallel-shift risk"
- It **does not** mean "fully hedged": twist risk, convexity mismatch, basis, and credit spread risk remain

---

### 3) Hedge Ratio (DV01 Hedge Ratio)

**Formal Definition:**

If position A has (scaled) DV01 $\text{DV01}_A^{\$}$ and hedge instrument B has $\text{DV01}_B^{\$}$, a DV01-neutral hedge solves:

$$\text{DV01}_A^{\$} + \text{DV01}_B^{\$} \approx 0$$

In Tuckman's notation for face amounts $F_A, F_B$ and per-face DV01s, the general hedge formula is:

$$\boxed{F_B = -F_A \frac{\text{DV01}_A}{\text{DV01}_B}} \quad \text{(Tuckman (5.7))}$$

**Intuition:**

- Hedge ratio = "How much hedge DV01 do I need to offset my DV01?"

**Trading / Risk / Portfolio Practice:**

- "Duration hedge" trades: long one bond, short another in DV01 proportion
- Swap overlays: pay-fixed swaps to reduce DV01 of a long bond portfolio
- Futures overlays: short Treasury futures to reduce DV01 without selling cash bonds

---

### 4) Yield-Based DV01 vs Curve-Based DV01 ("What's Being Bumped?")

**Formal Definition:**

- **Yield-based DV01:** sensitivity of price to the bond's yield (YTM/quoted yield)
- **Curve-based DV01:** sensitivity of price to shifts in spot/zero rates (or discount factors). Tuckman notes that if spot rates are shifted, the DV01 is sometimes called DVDZ, and many practitioners still call it DV01 even though the underlying bump differs.

**Intuition:**

- Yield-based DV01 is convenient when you quote the instrument by a single yield
- Curve-based DV01 is natural when you value instruments by discounting multiple cash flows off a curve

**Trading / Risk / Portfolio Practice:**

- Risk systems must specify: Do you bump yields? spot rates? par quotes with rebuild? discount curve only? multiple curves?
- Two systems can report different DV01 for the same trade if the bump definitions differ—leading to "mysterious" hedge slippage

---

### 5) Volatility-Weighted / Regression-Based Hedging ("Risk Weights")

**Formal Definition:**

DV01-neutral hedging targets zero first-order exposure under a particular bump. **Volatility-weighted** or **regression-based** hedging instead targets minimizing P&L variability, using historical relationships among rate changes. Tuckman introduces a "risk weight" idea in which hedge DV01 is scaled by a volatility ratio (and, more generally, correlation), and relates this to regression-based hedging.

**Intuition:**

- If hedge instrument B's yield is less volatile than the yield driving your position's risk, then a "pure DV01" hedge may under-hedge realized P&L variance
- Risk weights aim to account for that

**Trading / Risk / Portfolio Practice:**

- Desk risk managers may use risk weights or betas (e.g., "hedge 20y risk with 30y at 0.88 and 10y at 0.16")
- This can produce hedge notionals that do not match simple DV01 ratios

---

### 6) Why DV01 Hedges Fail

**Formal Definition:**

DV01 hedging is a first-order local approximation. It fails when:

1. The move is **not the one you hedged** (non-parallel curve shifts)
2. The move is **too large** (convexity/second-order effects)
3. The hedge instrument's pricing curve **diverges** from the position's curve (basis)
4. The position has **spread/credit risk** not hedged by a rates instrument
5. **Funding/liquidity/technical effects** dominate

**Intuition:**

DV01 says "how steep is the price-rate curve right here?" But markets move:
- In more than one dimension (level/slope/curvature)
- Across different reference curves (Treasury vs swaps, OIS vs LIBOR)
- With changing spreads and liquidity

**Trading / Risk / Portfolio Practice:**

- "DV01 flat but losing money" is usually explained by curve shape, basis, convexity, or spread moves
- The diagnostic workflow is: check key-rate exposures, then convexity, then basis/spread decomposition

---

## Math and Derivations

### 2.1 From DV01 Definition to Linear P&L Approximation

Start from Tuckman's DV01 definition:

$$\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y} \quad (1)$$

Rearrange:

$$\boxed{\Delta P \approx -\text{DV01} \cdot (10{,}000 \Delta y) = -\text{DV01} \cdot \Delta y_{\text{bp}}} \quad (2)$$

**Unit Checks:**

| Quantity | Units |
|----------|-------|
| $P$ | "price points per 100 face" (or currency if already scaled) |
| $\Delta y_{\text{bp}}$ | bp |
| DV01 | "price points per bp" (per 100 face) or "currency per bp" if scaled |
| Right side | (price points per bp) $\times$ (bp) = price points $\checkmark$ |

**Sanity Checks:**

- If yields rise ($\Delta y_{\text{bp}} > 0$) and DV01 $> 0$, then $\Delta P < 0$
- This matches the normal fixed-rate bond behavior $\checkmark$

---

### 2.2 DV01 and Modified Duration

Tuckman provides the relationship:

$$\boxed{\text{DV01} = \frac{P \, D_{\text{mod}}}{10{,}000}} \quad (3)$$

**Derivation Sketch:**

Modified duration is (approximately) the percentage price sensitivity:

$$\frac{\Delta P}{P} \approx -D_{\text{mod}} \, \Delta y$$

Multiply both sides by $P$:

$$\Delta P \approx -P \, D_{\text{mod}} \, \Delta y$$

Compare with $\Delta P \approx -\text{DV01} \cdot 10{,}000 \Delta y$, giving (3).

**Unit Checks:**

- $D_{\text{mod}}$ is dimensionless (often interpreted per "year" but mathematically unitless)
- $P D_{\text{mod}} / 10{,}000$ has units of "price per bp" $\checkmark$

---

### 2.3 Bond-Bond DV01 Hedge Ratio

Let bond A position be $F_A$ (face) with per-100 DV01 $\text{DV01}_A$. Its position DV01 in currency is:

$$\text{DV01}_A^{\$} = F_A \cdot \frac{\text{DV01}_A}{100} \quad (4)$$

("per 100 face" means divide by 100 to convert to "per 1 face".)

Similarly for bond B. DV01-neutral condition:

$$\text{DV01}_A^{\$} + \text{DV01}_B^{\$} \approx 0$$

Plug in (4) for both and solve for $F_B$:

$$\boxed{F_B = -F_A \frac{\text{DV01}_A}{\text{DV01}_B}} \quad (5)$$

This is Tuckman's general hedge formula (5.7).

**Sanity Checks:**

- If you are long A ($F_A > 0$) and both DV01s are positive, then $F_B < 0$: you short B
- This matches intuition $\checkmark$

---

### 2.4 "DV01-Neutral" Depends on the Bump Definition (Yield vs Curve)

Tuckman emphasizes a limitation of yield-based DV01 hedging: the hedge works only if the yield on the purchased security and the yield on the hedging security change by the same amount—a "parallel yield shift" assumption that is not generally true.

A more curve-based view says: value depends on many rates $z(t)$, so the PV change under a curve bump is:

$$\Delta V \approx \sum_k \frac{\partial V}{\partial z_k} \Delta z_k$$

If the bump is defined as $\Delta z_k = +1$ bp for all $k$ (parallel zero/spot shift), then DV01 is that aggregate sensitivity. If the bump is "par-quote bump with curve rebuild," then $\Delta z_k$ is induced indirectly by bumping market quotes and refitting the curve.

**Key Caveat:** Two DV01 numbers can disagree simply because "what is being bumped" differs (yield vs spot vs discount factors), a point Tuckman flags when discussing DV01 vs DVDZ terminology.

---

### 2.5 Swap PV01 (Annuity) and Bond-Swap Hedge Mechanics

Tuckman defines swap PV01 as the change in swap value for a 1 bp change in the fixed rate, and notes that for plain-vanilla swaps this PV01 equals the sum of discount factors used to discount the fixed cash flows (with standard cash-flow conventions).

For a fixed leg with payment dates $t_i$ and accrual factors $\tau_i$, the fixed-leg PV is (schematically):

$$V_{\text{fixed}}(K) = N K \sum_i \tau_i P(0, t_i)$$

Then:

$$\boxed{\text{PV01}_{\text{swap}} = \frac{\partial V}{\partial K} \cdot 0.0001 = N \left( \sum_i \tau_i P(0, t_i) \right) \cdot 0.0001} \quad (6)$$

**Important Sign Convention (Trading):**

- If you **receive fixed**, your PV increases when the fixed rate $K$ is higher $\Rightarrow$ PV01 (w.r.t $K$) is positive
- For rate hedging, desks often talk about "swap DV01" as sensitivity to market curve shifts, and Tuckman notes practitioners often manage fixed-side and floating-side risks separately

---

### 2.6 Futures Hedge Ratio: Duration/DV01 View

Hull presents a duration-based hedge ratio for interest rate futures:

$$\boxed{N^* = \frac{P \, D_P}{V_F \, D_F}} \quad (7)$$

where $P$ is the value of the portfolio being hedged, $D_P$ its duration, $V_F$ the value of one futures contract, and $D_F$ the duration of the underlying (often CTD).

Because DV01 is proportional to $P D_{\text{mod}}$ (Section 2.2), this is essentially a DV01 ratio, after aligning conventions and units.

**Treasury Futures Scaling with Conversion Factor:**

Tuckman's invoice price and CTD discussion provides the key scaling:

- Invoice price uses futures quoted price $\times$ conversion factor + accrued interest
- At delivery, the futures price relates to the CTD bond price scaled by conversion factor

A common first-order inference (derivation) is:

$$\Delta P_{\text{CTD}} \approx \text{CF} \cdot \Delta F_{\text{fut}} \quad \Rightarrow \quad \text{DV01}_{\text{fut}} \approx \frac{\text{DV01}_{\text{CTD}}}{\text{CF}}$$

(Here "DV01" must be defined consistently for the CTD bond and the futures price move; accrued interest and CTD switches are practical caveats.)

---

### 2.7 Risk Weights and Variance-Minimizing Hedges (Link to Regression)

Tuckman motivates "risk weights" via volatility ratios and connects them to regression-based hedging: regression-based hedging chooses hedge ratios to reduce the standard deviation of P&L per unit of risk, and "risk weights" can be interpreted through yield correlations/volatilities.

A compact one-hedge version can be stated as:

$$\text{Hedge DV01 exposure} \approx \text{RiskWeight} \times \text{DV01 exposure}$$

where risk weight may take the form (conceptually) $\rho \, \sigma_A / \sigma_B$ in the regression setting (details depend on the regression specification and the "risk" variable being hedged).

---

## Measurement & Risk (Chapter 15 Specifics)

### 3.1 "DV01-Hedged" — Precise Definition

We call a portfolio **DV01-hedged** under a specific bump definition $B$ if:

1. Compute each position's DV01 under bump definition $B$ (same curve, same rebuild rules, same pricing model inputs)
2. Apply a +1 bp version of bump $B$ to the valuation inputs and revalue
3. The first-order PV change is approximately zero:

$$\Delta V_B \approx 0 \quad \text{for +1 bp under } B$$

**What DV01-Hedged Does NOT Mean:**

| NOT Hedged For | Explanation |
|----------------|-------------|
| Twists | Non-parallel moves |
| Big moves | Convexity |
| Basis | Treasury vs swap, discount vs projection curve |
| Spread | Credit/agency/MBS spreads |

---

### 3.2 Hedge Ratio Mechanics

#### 3.2.1 Bond vs Bond DV01 Hedge Ratio

**Given:**
- Position A face $F_A$, per-100 DV01 $\text{DV01}_A$
- Hedge bond B face $F_B$, per-100 DV01 $\text{DV01}_B$

**DV01-neutral hedge:**

$$F_B = -F_A \frac{\text{DV01}_A}{\text{DV01}_B} \quad \text{(Tuckman (5.7))}$$

**Practical Workflow:**

1. Pull DV01s from risk system (ensure same bump definition)
2. Convert to common scaling (per 100 vs per \$1mm)
3. Compute $F_B$
4. Check signs (if long duration, you short duration)
5. Run parallel shift scenario to confirm

#### 3.2.2 Bond vs Swap PV01 Hedge Ratio (Rates Overlay)

Use swap PV01 (annuity) to translate between bond DV01 and swap notional:

1. Compute $\text{DV01}_{\text{bond}}^{\$}$ for the bond position
2. Compute $\text{PV01}_{\text{swap}}^{\$}(N=1)$ per unit notional (e.g., per \$1mm)
3. Choose swap notional $N_{\text{swap}}$ so that:

$$\text{DV01}_{\text{bond}}^{\$} + \text{DV01}_{\text{swap}}^{\$}(N_{\text{swap}}) \approx 0$$

**Curve Bump Clarification:**

Tuckman defines swap PV01 as change in value for 1 bp change in fixed rate and relates it to discount factors. In practice, a desk must specify whether "swap DV01" is computed by:
- Bumping the swap curve (and rebuilding)
- Bumping only the discount curve
- Bumping both discount and projection curves (multi-curve)

#### 3.2.3 Bond vs Futures DV01 Hedge Ratio

If you have:
- Bond position DV01 in dollars: $\text{DV01}_{\text{bond}}^{\$}$
- Futures DV01 per contract: $\text{DV01}_{\text{fut}}^{\$/\text{contract}}$

Then the number of futures contracts $n$ (short if long bond DV01) is:

$$n \approx \frac{\text{DV01}_{\text{bond}}^{\$}}{\text{DV01}_{\text{fut}}^{\$/\text{contract}}}$$

**Practical Caveat:** Futures DV01 depends on CTD and conversion factor; CTD can change as the curve moves, creating hedge slippage (a classic futures-hedge caveat).

---

### 3.3 Risk Weights: Why Notional Hedge Ratios Differ from DV01 Ratios

#### 3.3.1 Two Different Problems

| Problem | Objective |
|---------|-----------|
| DV01-neutral hedge | First-order exposure = 0. Solve: "make $\Delta V \approx 0$ for a specified +1 bp bump" |
| Variance-minimizing hedge | Minimize P&L volatility. Solve: "minimize $\text{Var}(\Delta V)$" using volatilities/correlations |

These can give **different hedge notionals**.

#### 3.3.2 Volatility-Weighted Intuition (Tuckman)

Tuckman shows that if one yield is more volatile than another, hedging one instrument with another may require scaling DV01 by a risk weight tied to volatility (and, in regression form, correlation).

**Conceptually:**
- If hedge instrument's yield moves less than the risk factor driving your position, you may need more hedge DV01 to reduce realized P&L variance
- If correlation is less than 1, even a perfect DV01 match can leave residual variance

#### 3.3.3 Simple Explicit Framework

Let portfolio P&L (first order) be:

$$\Delta V \approx -\sum_i x_i \Delta y_i$$

where $x_i$ is a DV01-like exposure in dollars per bp, and $\Delta y_i$ are rate changes (in bp) of the relevant risk factors.

- **DV01-neutral for a parallel shift** assumes $\Delta y_i$ are equal across $i$; then you want $\sum_i x_i = 0$
- **Variance-minimizing** uses the covariance matrix of $\Delta y$:

$$\min_x \text{Var}(\Delta V) = \min_x x^\top \Sigma x \quad \text{(possibly subject to constraints)}$$

Tuckman's regression-based hedging provides a source-backed motivation for this approach (minimizing P&L variability).

---

### 3.4 Practical Caveats: When DV01 Hedges Fail

#### 3.4.1 Curve Twist / Non-Parallel Moves (Preview: Key-Rate DV01)

- DV01 hedges assume a specific bump, often a parallel shift
- Tuckman explicitly highlights dangers of parallel-shift assumptions and motivates key rate/bucket exposures to represent non-parallel curve moves
- **Preview only:** if the curve twists, the portfolio's PV change depends on exposures at different maturities, not just the total DV01

#### 3.4.2 Convexity Mismatch (Second-Order Effects)

DV01 hedging is first order. For larger moves, convexity matters:

$$\frac{\Delta P}{P} \approx -D \, \Delta y + \frac{1}{2} C (\Delta y)^2$$

(Hull's standard duration-convexity approximation)

If two instruments have the same DV01 but different convexity, a DV01-neutral hedge will leave a systematic residual under large moves.

#### 3.4.3 Basis Risk (Treasury vs Swap Curve; Swap Spreads; Multi-Curve Preview)

Even if DV01 is neutral under a chosen definition, hedges fail if:
- The hedged instrument's yield changes do not match the hedge instrument's yield changes (Tuckman's explicit limitation)
- The relevant curves diverge (e.g., bond yield moves while swap rate stays fixed), producing P&L as "spread" changes; Tuckman's asset swap discussion emphasizes P&L from both rate moves and asset swap spread changes

**Multi-Curve Preview (Post-Crisis):**
Interest rate modeling practice often builds a discount curve (e.g., OIS) and separate forward/projection curves (e.g., IBOR tenors), so "what's being bumped" can mean bumping one curve or several.

#### 3.4.4 Liquidity/Funding/Repo and Idiosyncratic Effects (Conceptual)

DV01 does not capture changes in:
- Financing rates, special repo, funding haircuts
- Deliverability technicals (for futures)
- Bond-specific supply/demand shocks

These can dominate short-horizon P&L even when DV01 is hedged.

#### 3.4.5 Model/Curve Construction Dependence

DV01 depends on:
- Input curve set (Treasury curve vs swap curve vs OIS)
- Interpolation method
- Bump-and-rebuild vs direct shift
- Treatment of accrued interest/settlement

Tuckman notes that even the name "DV01" changes depending on whether yields or spot rates are shifted (DV01 vs DVDZ).

---

## Worked Examples

**Global Example Conventions (unless overridden):**

- DV01 sign: positive for long; PV change for +1 bp yield increase is $-\text{DV01}$
- DV01 inputs are per 100 face unless explicitly converted
- "Clean vs dirty": assume settlement on a coupon date so accrued interest $\approx 0$ (clean $\approx$ dirty) unless stated
- "Bump method": yield-based $\pm 1$ bp symmetric unless stated

---

### Example A: Bond-Bond DV01 Hedge Ratio

**Task:** Given DV01s for Bond A (position) and Bond B (hedge), compute hedge notional to make net DV01 $\approx 0$.

**Given:**
- Position: Long Bond A, face $F_A = \$10{,}000{,}000$
- Bond A DV01: $\text{DV01}_A = 0.0500$ per 100 face per 1 bp
- Hedge instrument: Bond B with $\text{DV01}_B = 0.044366$ per 100 face per 1 bp

**Goal:** Net DV01 = 0 under a +1 bp yield bump.

**Step 1: Convert per-100 DV01s to dollar DV01 per \$1 face**

$$\text{DV01}_A^{\$/\$1\text{ face}} = \frac{0.0500}{100} = 0.000500$$

$$\text{DV01}_B^{\$/\$1\text{ face}} = \frac{0.044366}{100} = 0.00044366$$

**Step 2: Compute position DV01 in dollars**

$$\text{DV01}_A^{\$} = F_A \cdot \frac{\text{DV01}_A}{100} = 10{,}000{,}000 \times 0.000500 = \$5{,}000 \text{ per bp}$$

**Step 3: Solve hedge face using Tuckman hedge ratio**

$$F_B = -F_A \frac{\text{DV01}_A}{\text{DV01}_B} = -10{,}000{,}000 \times \frac{0.0500}{0.044366}$$

Compute the ratio:

$$\frac{0.0500}{0.044366} \approx 1.1270$$

So:

$$F_B \approx -11{,}270{,}000$$

**Answer:** Short \$11.27 mm face of Bond B.

**Quick Sanity Check:**
- Bond A DV01: +\$5,000/bp
- Bond B DV01 per \$1 face: 0.00044366 $\Rightarrow$ per \$11.27mm: $11.27\text{mm} \times 0.00044366 \approx \$5{,}000$
- Shorting that offsets $\checkmark$

---

### Example B: Scaling and Units

**Task:** Convert DV01 quoted "per 100 price" into DV01 per \$1mm notional, and redo the hedge ratio. Show how wrong unit handling breaks the hedge.

**Given:**
- Bond A DV01: $0.0857$ per 100 face per bp
- Bond B DV01: $0.0600$ per 100 face per bp
- Position: long $F_A = \$25{,}000{,}000$

**Step 1: Convert DV01 per 100 face $\rightarrow$ DV01 per \$1mm face**

Per \$1mm, face = 1,000,000. DV01 in dollars:

$$\text{DV01}_A^{\$/\$1\text{mm}} = 1{,}000{,}000 \times \frac{0.0857}{100} = 1{,}000{,}000 \times 0.000857 = \$857 \text{ per bp}$$

Similarly:

$$\text{DV01}_B^{\$/\$1\text{mm}} = 1{,}000{,}000 \times \frac{0.0600}{100} = \$600 \text{ per bp}$$

**Step 2: Position DV01 for \$25mm**

$$\text{DV01}_A^{\$} = 25 \times 857 = \$21{,}425 \text{ per bp}$$

**Step 3: Hedge ratio using per-\$1mm DV01**

Need $\text{DV01}_B^{\$} = -21{,}425$. With \$600/bp per \$1mm:

$$\text{Hedge notional (mm)} = -\frac{21{,}425}{600} = -35.7083$$

**Answer:** Short \$35.71 mm of Bond B.

**How Unit Mistakes Break the Hedge (Common Failure):**

If someone forgets "per 100" and treats 0.0857 as "per \$1 face," they compute:

$$\text{Wrong DV01}_A^{\$} = 25{,}000{,}000 \times 0.0857 = \$2{,}142{,}500 \text{ per bp}$$

That is **100$\times$ too large**.

Then "wrong hedge notional" becomes:

$$-\frac{2{,}142{,}500}{600} \approx -3{,}570.8 \text{ mm} = -\$3.57 \text{ bn}$$

**Lesson:** DV01 hedging is extremely sensitive to scaling and units.

---

### Example C: Bond-Swap Hedge

**Task:** Assume the swap PV01 and compute swap notional required to hedge a bond's DV01. Explain what curve bump the swap PV01 corresponds to.

**Conventions for this example:**
- Bond DV01 is yield-based
- Swap "PV01" is computed as the swap annuity (change in value per 1 bp change in fixed rate), per Tuckman's definition
- We use a **payer swap** (pay fixed, receive float) to hedge a long bond's positive DV01; thus swap contributes negative DV01 exposure (short fixed)

**Given:**
- Bond position: long \$50mm face
- Bond DV01: $0.0800$ per 100 face per bp
- Swap: 5y annual-pay, notional $N$ to be determined
- Discount factors (toy, for annuity calculation):

| Maturity | Discount Factor |
|----------|-----------------|
| $P(0,1)$ | 0.98 |
| $P(0,2)$ | 0.955 |
| $P(0,3)$ | 0.93 |
| $P(0,4)$ | 0.905 |
| $P(0,5)$ | 0.88 |

- Accrual factors: $\tau_i = 1$ (annual)

**Step 1: Bond DV01 in dollars**

Per \$1 face DV01:

$$\frac{0.0800}{100} = 0.000800$$

Position DV01:

$$\text{DV01}_{\text{bond}}^{\$} = 50{,}000{,}000 \times 0.000800 = \$40{,}000 \text{ per bp}$$

**Step 2: Swap PV01 per \$1 notional**

Annuity sum:

$$A = \sum_{i=1}^{5} \tau_i P(0,t_i) = 0.98 + 0.955 + 0.93 + 0.905 + 0.88 = 4.65$$

PV01 per \$1 notional (1 bp = 0.0001):

$$\text{PV01}_{\text{swap}}^{\$}(N=1) = 0.0001 \times A = 0.000465$$

Per \$1mm notional:

$$\text{PV01}_{\text{swap}}^{\$}(N=\$1\text{mm}) = 1{,}000{,}000 \times 0.000465 = \$465 \text{ per bp}$$

**Step 3: Solve swap notional**

We need payer swap DV01 $\approx -\text{PV01} \times N$ to offset bond:

$$40{,}000 - 465 \times N_{\text{mm}} \approx 0$$

So:

$$N_{\text{mm}} = \frac{40{,}000}{465} \approx 86.02$$

**Answer:** Enter a payer swap of notional \$86.0 mm (annual-pay toy swap) to DV01-hedge the bond.

**What bump is this PV01 tied to?**

This PV01 is sensitivity to the fixed-rate parameter in the swap PV, which is computed from discount factors. In practice, "swap DV01" for rate risk depends on how the swap curve (and possibly discount/projection curves) is bumped and rebuilt—this is a curve-construction dependence caveat.

---

### Example D: Bond-Futures Hedge

**Task:** Provide a futures DV01/PV01 proxy (or compute under stated assumptions) and determine number of contracts to DV01-hedge a cash bond position.

**Conventions for this example:**
- Futures DV01 is approximated using CTD bond DV01 and conversion factor scaling (first-order inference from Tuckman's CTD/conversion-factor mechanics)

**Given:**
- Cash bond position DV01: $\text{DV01}_{\text{bond}}^{\$} = \$25{,}000$ per bp (assume already position-scaled by risk system)
- Treasury futures contract:
  - Contract size: \$100,000 face
  - CTD bond DV01: $0.0750$ per 100 face per bp
  - Conversion factor: $\text{CF} = 0.90$

**Step 1: Compute CTD DV01 per contract**

Per \$1 face DV01:

$$\frac{0.0750}{100} = 0.000750$$

Per \$100,000 contract:

$$\text{DV01}_{\text{CTD, contract}}^{\$} = 100{,}000 \times 0.000750 = \$75.00 \text{ per bp}$$

**Step 2: Scale to futures DV01 using CF**

Approximate:

$$\text{DV01}_{\text{fut}}^{\$} \approx \frac{\text{DV01}_{\text{CTD, contract}}^{\$}}{\text{CF}} = \frac{75.00}{0.90} = \$83.33 \text{ per bp per contract}$$

**Step 3: Number of contracts**

$$n \approx \frac{25{,}000}{83.33} \approx 300.0$$

**Answer:** Short approximately **300 futures contracts**.

**Practical Caveats:**
- CTD may change as the curve moves, changing DV01 per contract
- Invoice-price mechanics include accrued interest; ignoring it is a simplification

---

### Example E: DV01-Neutral but Twist Loss

**Task:** Construct a DV01-neutral portfolio (A + hedge). Apply a twist scenario and compute P&L using a simple key-rate DV01 preview. Show why net DV01 = 0 did not protect.

**Conventions for this example:**
- We use key-rate DV01-like bucket exposures (preview only). Tuckman defines bucket/key rate exposures by shifting specific segments and computing PV changes.

**Given:**

Bond A position (long): key-rate DV01s (in \$ per bp)

| Bucket | $\text{KDV01}_{A}$ |
|--------|---------------------|
| 2y | 2,000 |
| 10y | 8,000 |
| **Total DV01** | **10,000** |

Hedge bond B (per "unit" of hedge):

| Bucket | $\text{KDV01}_{B}$ |
|--------|---------------------|
| 2y | 4,000 |
| 10y | 4,000 |
| **Total DV01** | **8,000** |

**Step 1: Make total DV01 neutral**

Let hedge multiplier be $h$ (we short $h$ units of B). Total DV01 condition:

$$10{,}000 - h \cdot 8{,}000 = 0 \quad \Rightarrow \quad h = 1.25$$

**Portfolio (constructed):**
- Long 1 unit of A
- Short 1.25 units of B

**Step 2: Compute key-rate DV01 exposures of portfolio**

2y:
$$\text{KDV01}_{\text{port},2} = 2{,}000 - 1.25 \times 4{,}000 = 2{,}000 - 5{,}000 = -3{,}000$$

10y:
$$\text{KDV01}_{\text{port},10} = 8{,}000 - 1.25 \times 4{,}000 = 8{,}000 - 5{,}000 = 3{,}000$$

Total DV01:
$$-3{,}000 + 3{,}000 = 0 \quad \checkmark$$

**Step 3: Apply a twist scenario**

| Bucket | Rate Change |
|--------|-------------|
| Short end (2y) | $\Delta y_2 = +10$ bp |
| Long end (10y) | $\Delta y_{10} = -5$ bp |

Approximate PV change:

$$\Delta V \approx -(\text{KDV01}_2 \Delta y_2 + \text{KDV01}_{10} \Delta y_{10})$$

Compute:

2y contribution:
$$\Delta V_2 \approx -(-3{,}000) \times 10 = +30{,}000$$

10y contribution:
$$\Delta V_{10} \approx -(3{,}000) \times (-5) = +15{,}000$$

Total:
$$\Delta V \approx +45{,}000$$

**Conclusion:** The portfolio is DV01-neutral for a parallel move, yet it gains (or could lose) under a twist because bucket exposures are not neutral. This motivates moving from "one number (DV01)" to "many numbers (key-rate DV01)" in the next chapter.

---

### Example F: Convexity Mismatch

**Task:** Build a DV01-neutral hedge between a high-convexity bond and a low-convexity bond. Apply a large move ($\pm 100$ bp) and show residual P&L due to convexity.

**Conventions for this example:**
- Use duration-convexity approximation from Hull:

$$\frac{\Delta P}{P} \approx -D \, \Delta y + \frac{1}{2} C (\Delta y)^2$$

- Prices initially $P = 100$ for both bonds; face notionals equal
- DV01 is matched at the initial yield level

**Given:**

| Bond | Duration $D$ | Convexity $C$ |
|------|--------------|---------------|
| H (high convexity) | 8 | 120 |
| L (low convexity) | 8 | 60 |

Positions: long \$10mm face of H, short \$10mm face of L (DV01-neutral initially since same duration and price).

**Step 1: Confirm initial DV01 per 100 (using DV01 $\approx$ P D / 10000)**

$$\text{DV01} \approx \frac{100 \times 8}{10{,}000} = 0.0800 \text{ per 100 per bp}$$

Same for both $\Rightarrow$ DV01-neutral with equal face amounts.

**Step 2: Apply +100 bp move ($\Delta y = +0.01$)**

For Bond H:
$$\frac{\Delta P_H}{100} \approx -8(0.01) + \frac{1}{2} 120 (0.01)^2 = -0.08 + 60(0.0001) = -0.08 + 0.006 = -0.074$$

So $\Delta P_H \approx -7.4$ points.

For Bond L:
$$\frac{\Delta P_L}{100} \approx -0.08 + \frac{1}{2} 60 (0.0001) = -0.08 + 30(0.0001) = -0.08 + 0.003 = -0.077$$

So $\Delta P_L \approx -7.7$ points.

**Portfolio P&L per 100 face:**
- Long H loses 7.4
- Short L gains 7.7
- Net: $+0.3$ points

Convert to dollars for \$10mm:

$$\text{P\&L} \approx 0.3\% \times 10{,}000{,}000 = \$30{,}000$$

**Step 3: Apply $-100$ bp move ($\Delta y = -0.01$)**

Bond H:
$$\frac{\Delta P_H}{100} \approx -8(-0.01) + \frac{1}{2} 120 (0.01)^2 = +0.08 + 0.006 = +0.086 \Rightarrow +8.6 \text{ points}$$

Bond L:
$$\frac{\Delta P_L}{100} \approx +0.08 + 0.003 = +0.083 \Rightarrow +8.3 \text{ points}$$

Portfolio per 100: long H gains 8.6, short L loses 8.3 $\Rightarrow$ net $+0.3$ points $\Rightarrow$ \$30,000.

**Conclusion:** DV01-neutral $\neq$ P&L neutral for large moves. Convexity mismatch leaves systematic residual.

---

### Example G: Treasury vs Swap Basis Risk

**Task:** Hedge a bond's DV01 using a swap. Then apply a "swap spread move" (swap rates shift relative to bond yields) and show hedge breakdown. *(Scenario is a toy basis example.)*

**Conventions for this example:**
- We treat this as a **toy basis scenario**: bond yield changes are not identical to swap rate changes
- Motivation: Tuckman emphasizes DV01 hedging requires yields to move together; if bond yield increases while swap rate stays the same, the "spread" increases and the trade can lose money

**Given:**
- Bond position: long, DV01 (position-scaled) $= \$10{,}000$ per bp w.r.t the bond's yield
- Swap hedge: payer swap with "swap DV01 magnitude" $= \$10{,}000$ per bp w.r.t swap-rate moves (constructed to hedge under the assumption bond yields and swap rates move 1-for-1)

**Step 1: DV01-neutral under the assumed common move**

If both bond yield and swap rate rise by +1 bp:
- Bond P&L: $\Delta V_{\text{bond}} \approx -10{,}000$
- Swap P&L (payer benefits): $\Delta V_{\text{swap}} \approx +10{,}000$
- Net $\approx 0$

**Step 2: Basis / "swap spread" shock scenario**

Assume:
- Bond yield rises by +10 bp
- Swap rate rises by +5 bp (swap spread "tightens" by 5 bp in this toy setup)

Compute P&L:

Bond:
$$\Delta V_{\text{bond}} \approx -10{,}000 \times 10 = -\$100{,}000$$

Swap:
$$\Delta V_{\text{swap}} \approx +10{,}000 \times 5 = +\$50{,}000$$

Net:
$$\Delta V_{\text{net}} \approx -\$50{,}000$$

**Conclusion:** Even with perfect DV01 match under a "common curve" assumption, divergence between bond yields and swap rates produces basis P&L. This aligns with Tuckman's warning that DV01 hedging depends on yields moving by the same amount.

---

### Example H: Hedging a Spread Product with a Rates Hedge

**Task:** Price a bond off a benchmark curve + spread. DV01-hedge the rates component. Then widen credit spread and show that rates DV01 hedge does not protect credit-spread P&L (define spread duration/CS01 briefly).

**Conventions for this example:**
- Use credit-yield decomposition $y = y_T + s$ (Treasury/benchmark yield plus spread)
- Use spread and rate duration decomposition (from the credit derivatives text):

$$-\frac{dP}{P} = D_T \, dy_T + D_S \, ds \quad (8)$$

- Define:
  - Rate DV01 (per \$notional) from $D_T$
  - CS01 from $D_S$

**Given:**

Corporate/agency bond:

| Parameter | Value |
|-----------|-------|
| Price $P$ | 95 per 100 |
| Face position $F$ | \$20,000,000 |
| Rate duration $D_T$ | 6.0 |
| Spread duration $D_S$ | 5.5 |

Rates hedge chosen to neutralize rate DV01 only (e.g., Treasury futures or swap).

**Scenario:** Benchmark rates unchanged, credit spread widens by +50 bp.

**Step 1: Compute rate DV01 of the bond position**

Using DV01 $\approx P D_T / 10{,}000$ per 100:

$$\text{DV01}_{\text{rate, per 100}} \approx \frac{95 \times 6.0}{10{,}000} = 0.0570$$

Position DV01 in dollars:

$$\text{DV01}_{\text{rate}}^{\$} = F \cdot \frac{0.0570}{100} = 20{,}000{,}000 \times 0.000570 = \$11{,}400 \text{ per bp}$$

**Step 2: Hedge rate DV01**

Assume we take a rates hedge with DV01 $= -\$11{,}400$ per bp (constructed by DV01 ratio). Net rate DV01 $\approx 0$.

**Step 3: Compute CS01 (credit spread 01)**

Spread DV01 per 100:

$$\text{CS01}_{\text{per 100}} \approx \frac{P D_S}{10{,}000} = \frac{95 \times 5.5}{10{,}000} = 0.05225$$

Position CS01 in dollars:

$$\text{CS01}^{\$} = 20{,}000{,}000 \times \frac{0.05225}{100} = 20{,}000{,}000 \times 0.0005225 = \$10{,}450 \text{ per bp}$$

**Step 4: Apply spread widening +50 bp (rates unchanged)**

Bond P&L from spread move:

$$\Delta V_{\text{spread}} \approx -\text{CS01}^{\$} \times 50 = -10{,}450 \times 50 = -\$522{,}500$$

Rates hedge P&L $\approx 0$ (rates unchanged).

**Conclusion:** A DV01 hedge removes rate risk, not spread/credit risk. Spread duration/CS01 is a separate exposure dimension (even if the hedge is "DV01-neutral").

---

### Example I: Risk-Weighted Hedge, Minimal Variance

**Task:** Given volatilities and correlations for two candidate hedge instruments (toy numbers), compute hedge weights that (i) neutralize DV01 and (ii) minimize P&L variance subject to DV01-neutral constraint. Keep linear algebra transparent and small (2-3 instruments).

**Source Check:** Tuckman supports the motivation for variance-reducing hedges via volatility weights and regression-based hedging. The specific constrained optimization "minimize variance subject to DV01-neutral" is a derived extension of that idea (not presented as a named formula in the cited excerpt). We present it transparently and label assumptions.

**Setup:**

We hedge a position A using two hedges B and C.

- DV01 exposure of A (position-scaled): $\text{DV01}_A^{\$} = +10{,}000$ per bp
- Hedge instruments:
  - B DV01 per \$1mm notional: $\text{DV01}_B^{\$/\text{mm}} = 600$
  - C DV01 per \$1mm notional: $\text{DV01}_C^{\$/\text{mm}} = 900$

Define decision variables as dollar DV01 exposures contributed by hedges:
- $x_B$: DV01 exposure from B (in \$ per bp); if we short duration, $x_B < 0$
- $x_C$: DV01 exposure from C (in \$ per bp)

Then hedge notionals (in \$mm) are:

$$N_B^{\text{mm}} = \frac{x_B}{600}, \quad N_C^{\text{mm}} = \frac{x_C}{900}$$

**(i) DV01-neutral constraint**

DV01-neutral for a parallel shift:

$$\text{DV01}_A^{\$} + x_B + x_C = 0 \quad \Rightarrow \quad x_B + x_C = -10{,}000 \quad (C)$$

**(ii) Variance model (toy)**

Assume daily yield changes (in bp) have volatilities and correlations:

| Parameter | Value |
|-----------|-------|
| $\sigma_A$ | 8 |
| $\sigma_B$ | 6 |
| $\sigma_C$ | 10 |
| $\rho_{AB}$ | 0.95 |
| $\rho_{AC}$ | 0.90 |
| $\rho_{BC}$ | 0.85 |

Covariances: $\text{Cov}(\Delta y_i, \Delta y_j) = \rho_{ij} \sigma_i \sigma_j$

Compute needed covariances:

| Quantity | Value |
|----------|-------|
| $\text{Var}(A)$ | $8^2 = 64$ |
| $\text{Var}(B)$ | $6^2 = 36$ |
| $\text{Var}(C)$ | $10^2 = 100$ |
| $\text{Cov}(A,B)$ | $0.95 \times 8 \times 6 = 45.6$ |
| $\text{Cov}(A,C)$ | $0.90 \times 8 \times 10 = 72$ |
| $\text{Cov}(B,C)$ | $0.85 \times 6 \times 10 = 51$ |

First-order P&L (ignoring sign convention since variance uses squares):

$$\Delta V \approx -(10{,}000 \, \Delta y_A + x_B \, \Delta y_B + x_C \, \Delta y_C)$$

Variance (drop the minus sign):

$$\text{Var}(\Delta V) = \text{Var}(10{,}000 \, \Delta y_A + x_B \, \Delta y_B + x_C \, \Delta y_C)$$

**Step 1: Use constraint to reduce to one variable**

From (C): $x_C = -10{,}000 - x_B$

**Step 2: Compute variance as a quadratic in $x_B$**

Let $a = 10{,}000$, $b = x_B$, $c = x_C = -10{,}000 - x_B$.

Then:

$$\text{Var} = a^2 \text{Var}(A) + b^2 \text{Var}(B) + c^2 \text{Var}(C) + 2ab \, \text{Cov}(A,B) + 2ac \, \text{Cov}(A,C) + 2bc \, \text{Cov}(B,C)$$

Plug numbers (in bp$^2$ units):

- $a^2 \text{Var}(A) = 10{,}000^2 \times 64$
- $b^2 \text{Var}(B) = 36 b^2$
- $c^2 \text{Var}(C) = 100(-10{,}000 - b)^2$
- $2ab \, \text{Cov}(A,B) = 2 \times 10{,}000 \times b \times 45.6 = 912{,}000 \, b$
- $2ac \, \text{Cov}(A,C) = 2 \times 10{,}000 \times (-10{,}000 - b) \times 72 = 1{,}440{,}000 \times (-10{,}000 - b)$
- $2bc \, \text{Cov}(B,C) = 2 \times b \times (-10{,}000 - b) \times 51 = 102 \, b(-10{,}000 - b)$

Differentiate w.r.t $b$ and set to zero (algebra shown compactly):

$$\frac{d}{db} \text{Var} = 72b + 200(-10{,}000 - b) + 912{,}000 - 1{,}440{,}000 + 102(-10{,}000 - 2b) = 0$$

Simplify term-by-term:

- $72b$
- $200(-10{,}000 - b) = -2{,}000{,}000 - 200b$
- $+912{,}000 - 1{,}440{,}000 = -528{,}000$
- $102(-10{,}000 - 2b) = -1{,}020{,}000 - 204b$

Sum:
- Constant: $-2{,}000{,}000 - 528{,}000 - 1{,}020{,}000 = -3{,}548{,}000$
- $b$-terms: $72b - 200b - 204b = -332b$

Set to zero:

$$-332b - 3{,}548{,}000 = 0 \quad \Rightarrow \quad b = -\frac{3{,}548{,}000}{332} \approx -10{,}686.75$$

Then:

$$x_C = -10{,}000 - x_B = -10{,}000 - (-10{,}686.75) = +686.75$$

**Interpretation:**

Optimal (variance-minimizing under this toy covariance) DV01 exposures:

| Instrument | DV01 Exposure (\$ per bp) |
|------------|---------------------------|
| $x_B$ | $\approx -10{,}687$ |
| $x_C$ | $\approx +687$ |

This is DV01-neutral: $-10{,}687 + 687 = -10{,}000$ $\checkmark$

**Convert to hedge notionals:**

$$N_B^{\text{mm}} = \frac{-10{,}686.75}{600} \approx -17.81 \text{ mm}$$

$$N_C^{\text{mm}} = \frac{686.75}{900} \approx +0.76 \text{ mm}$$

**Final Numeric Result:**
- Short \$17.81 mm of B
- Long \$0.76 mm of C

subject to DV01-neutrality and minimal variance in this toy model.

**Why can one hedge be long?**

Variance minimization can tilt into an instrument that is "good" for correlation/volatility reasons, even if that means taking a small long DV01 in one hedge and a larger short DV01 in the other, while maintaining net DV01-neutrality.

---

## Practical Notes

### Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| **DV01 sign convention confusion** | State your chosen convention and stick to it. In this chapter DV01 is positive for long fixed-rate exposure; +1 bp yields $\Rightarrow$ PV down by DV01 |
| **Mixing yield-DV01 with curve-DV01** | Yield DV01 works under strong assumptions about yield comovement; curve DV01 (DVDZ) is different |
| **Clean vs dirty price inconsistency** | Accrued interest is not rate-sensitive at an instant, but mismatched system conventions can create reconciliation errors |
| **Per-100 vs per-\$1mm unit mistakes** | See Example B |
| **Ignoring convexity and curve-shape risk** | See Examples E and F |
| **Ignoring basis and multi-curve issues** | Different curves move differently; "what's being bumped?" matters |

### Implementation Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Bump size selection and numerical noise** | Tuckman notes practical issues in choosing a rate difference when calculating DV01, especially in the presence of price errors; too small a bump can be swamped by noise, too large can introduce approximation error |
| **"Bump-and-rebuild" vs "direct zero bump" inconsistencies** | If one system rebuilds the curve from bumped par quotes and another shifts zeros directly, reported DV01s can differ |
| **Unstable hedges near coupon dates / settlement** | Carry/accrual and settlement conventions can dominate short-horizon P&L; ensure DV01 is computed at the correct settlement date |

### Verification Tests

| Test | Description |
|------|-------------|
| **DV01-neutral check** | Revalue under +1 bp using the same rules used to report DV01 |
| **Sanity: hedge notional direction** | Long duration $\rightarrow$ hedge should typically be short duration (bond short / payer swap / futures short) |
| **Scenario checks** | Parallel shift (should be near-flat), Twist (will show residual unless key-rate exposures hedged), Large move (convexity residual) |
| **Attribution** | Residual explained by convexity/basis/spread terms. Separate rate moves, spread moves, and basis moves when possible (Example H) |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **DV01 measures first-order PV sensitivity** to a 1 bp rate move under a specified bump definition
2. **DV01-neutral means net PV change $\approx 0$** for the hedged +1 bp bump—nothing more
3. **The bond-bond DV01 hedge ratio** is $F_B = -F_A (\text{DV01}_A / \text{DV01}_B)$
4. **DV01 can be expressed via modified duration:** $\text{DV01} = P D_{\text{mod}} / 10{,}000$
5. **Yield-based DV01 hedging assumes** the yields of hedged and hedge instruments move together (often implicitly "parallel"), which can be wrong
6. **"What's being bumped?" matters:** yield DV01 differs from curve DV01 (DVDZ) and can change hedge ratios
7. **Swap hedging uses PV01** (swap annuity) tied to discount factors; sign depends on payer/receiver
8. **Futures hedging depends on CTD and conversion factors;** DV01 per contract can change when CTD changes
9. **DV01 hedges fail** under twists (need key-rate exposure view), large moves (convexity), basis divergence, and spread/credit moves
10. **Risk weights/regression hedges target P&L variance,** not just DV01 neutrality; different objectives produce different notionals

---

### Cheat Sheet: Formulas, Unit Conversions, Failure Modes

**DV01 definition (per 100 face):**
$$\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \Delta y}$$

**PV change approximation:**
$$\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}}$$

**DV01 from modified duration:**
$$\text{DV01} = \frac{P D_{\text{mod}}}{10{,}000}$$

**Convert DV01 per 100 $\rightarrow$ DV01 per \$1mm face:**
$$\text{DV01}^{\$/\text{mm}} = 1{,}000{,}000 \cdot \frac{\text{DV01 (per 100)}}{100}$$

**Bond-bond DV01 hedge:**
$$F_B = -F_A \frac{\text{DV01}_A}{\text{DV01}_B}$$

**Swap PV01 (annuity) per notional:**
$$\text{PV01}_{\text{swap}} = N \left( \sum_i \tau_i P(0,t_i) \right) \cdot 0.0001$$

**Futures hedge (DV01 form):**
$$n \approx \frac{\text{DV01}_{\text{bond}}^{\$}}{\text{DV01}_{\text{fut}}^{\$/\text{contract}}}$$

**Failure Modes Checklist:**

| Mode | Description |
|------|-------------|
| Twist risk | Needs key-rate |
| Convexity mismatch | Second-order |
| Basis risk | Treasury vs swap; discount vs projection curves |
| Spread/credit risk | CS01 not hedged by DV01 |
| Model/curve-rebuild dependence | Different systems, different DV01 |

---

### Flashcards (25 Q/A)

**Q1:** What is DV01 in Tuckman's definition?
**A:** $\text{DV01} \equiv -\Delta P / (10{,}000 \Delta y)$, i.e., dollar (or price-point) change per 1 bp yield change, with a minus sign so DV01 is positive for normal bonds.

**Q2:** With DV01 > 0, what is the approximate PV change for a +1 bp yield move?
**A:** $\Delta P \approx -\text{DV01}$.

**Q3:** What does "DV01-hedged" mean precisely?
**A:** Net PV change is approximately zero under the specified +1 bp bump definition.

**Q4:** What is the bond-bond hedge ratio in face amounts?
**A:** $F_B = -F_A (\text{DV01}_A / \text{DV01}_B)$.

**Q5:** Why can two systems report different DV01 for the same position?
**A:** Different bump definitions ("what's being bumped?"), curve construction/rebuild rules, or pricing conventions.

**Q6:** What is yield-based DV01 implicitly assuming when used for hedging?
**A:** That yields of hedged and hedge instruments change by the same amount (a strong parallel-comovement assumption).

**Q7:** How is DV01 related to modified duration?
**A:** $\text{DV01} = P D_{\text{mod}} / 10{,}000$.

**Q8:** What is swap PV01 conceptually?
**A:** Change in swap value for a 1 bp change in the fixed rate; equals the swap annuity times 1 bp.

**Q9:** For hedging a long bond DV01 with swaps, do you typically pay or receive fixed?
**A:** Typically pay fixed (payer swap) to obtain negative fixed-rate DV01 exposure.

**Q10:** What is the key practical futures hedging input besides DV01?
**A:** Cheapest-to-deliver (CTD) and its conversion factor.

**Q11:** What is a common reason DV01-neutral hedges lose money in a steepening/flattening move?
**A:** Curve twist / non-parallel moves (bucket/key-rate exposures differ).

**Q12:** Why does convexity matter for DV01 hedging?
**A:** DV01 is first-order; convexity is second-order and becomes important for large moves.

**Q13:** What does "basis risk" mean in rates hedging?
**A:** The hedged instrument and hedge instrument are driven by different curves/spreads that can move differently.

**Q14:** What is CS01?
**A:** PV change for a 1 bp change in credit spread (spread risk), separate from rate DV01.

**Q15:** Why doesn't a rates DV01 hedge protect a corporate bond from spread widening?
**A:** Because spread duration/CS01 drives PV changes when spreads move, not rates.

**Q16:** What is the main unit trap in DV01 numbers?
**A:** Confusing DV01 "per 100 face" with DV01 "per \$1mm" (factor of 100 error).

**Q17:** What is "risk-weighted hedging" trying to improve vs DV01-neutral hedging?
**A:** Reduce realized P&L volatility by accounting for volatility/correlation differences.

**Q18:** What is regression-based hedging conceptually?
**A:** Choose hedge ratios to minimize variance (or standard deviation) of the hedged position's P&L.

**Q19:** In a futures hedge, why can the hedge ratio change even if your cash bond position is unchanged?
**A:** CTD can switch or its effective duration changes with the curve level.

**Q20:** What is the first diagnostic if DV01-neutral book loses money in a rates move?
**A:** Check curve-shape exposure (key-rate/bucket DV01).

**Q21:** What is a "parallel shift" assumption?
**A:** All relevant yields/zero rates move by the same number of basis points.

**Q22:** What does "bump-and-rebuild" mean in curve risk?
**A:** Bump market quotes (or curve inputs) and refit/rebuild the curve before revaluing.

**Q23:** Why can swap DV01 be ambiguous in a multi-curve framework?
**A:** Discounting and projection curves differ; you must specify which curve(s) are bumped.

**Q24:** What is the simplest verification test after putting on a DV01 hedge?
**A:** Revalue the full portfolio under the exact +1 bp bump definition and check $\Delta V \approx 0$.

**Q25:** What is the most important statement to remember about DV01 hedging?
**A:** It is local, first-order, and bump-definition-dependent—so it hedges only what you define.

---

## Mini Problem Set (14 Questions)

*Brief solution sketches provided for questions 1-7 only.*

---

**Problem 1:** A bond position has $\text{DV01} = 0.072$ per 100 and face \$15mm. Compute position DV01 in dollars per bp.

**Sketch:** Convert per-100 to per-\$1: $0.072/100 = 0.00072$. Multiply by face: $15{,}000{,}000 \times 0.00072 = \$10{,}800$.

---

**Problem 2:** You are long \$15mm of bond A with DV01 $0.072$ per 100. Hedge with bond B with DV01 $0.060$ per 100. Compute hedge face for DV01-neutral.

**Sketch:** $F_B = -15\text{mm} \times 0.072/0.060 = -18\text{mm}$ (short).

---

**Problem 3:** A risk report shows DV01 of a bond as 0.085 per 100. A trader mistakenly treats it as 0.085 per \$1 face. By what factor is the risk overstated?

**Sketch:** Factor $= 100$ (per-100 vs per-1).

---

**Problem 4:** A bond portfolio has value \$120mm and duration 6.5. A futures contract has value \$110k and CTD duration 7.8. Use Hull's duration hedge formula to estimate contracts.

**Sketch:** $N^* = (P D_P)/(V_F D_F) = (120{,}000{,}000 \times 6.5)/(110{,}000 \times 7.8)$. Compute numeric.

---

**Problem 5:** A 5y annual swap has discount factors 0.99, 0.97, 0.95, 0.93, 0.91. Compute PV01 per \$1mm notional.

**Sketch:** Annuity $A = \sum P = 4.75$. PV01 per \$1mm = $1{,}000{,}000 \times 0.0001 \times 4.75 = \$475$.

---

**Problem 6:** A DV01-neutral bond-bond hedge has net DV01 = 0. Under a twist where 2y yields +20 bp and 10y yields $-10$ bp, your book loses money. Name the most likely risk explanation.

**Sketch:** Curve-shape risk / bucket mismatch; net DV01 neutral doesn't hedge non-parallel moves.

---

**Problem 7:** You DV01-hedged a corporate bond with Treasuries. Credit spreads widen by 30 bp with no rate change. What risk measure explains the loss?

**Sketch:** Spread duration / CS01 exposure (rates DV01 hedge does not hedge credit spread moves).

---

**Problem 8:** Explain (in words) why a DV01-neutral hedge between a callable bond and a noncallable bond can be unstable as yields change.

---

**Problem 9:** Give two reasons why Treasury futures DV01 per contract can change over time even without rolling the contract.

---

**Problem 10:** Describe a bump-definition mismatch that could cause a desk's "DV01-hedged" book to show residual PV change under a +1 bp scenario.

---

**Problem 11:** Suppose your hedge instrument is much less liquid than your position. What additional risk beyond DV01 might this create?

---

**Problem 12:** For a swap in a multi-curve framework, list which curves might be bumped to compute "DV01" and why that choice matters.

---

**Problem 13:** In a two-hedge variance-minimizing setup, why might the optimizer suggest a small long position in one hedge and a large short in the other?

---

**Problem 14:** Design a scenario test suite (3-5 scenarios) to validate a DV01 hedge for a mortgage-related instrument with negative convexity.

---

## Source Map

### (A) Verified Facts — Specific Source Citations

| Content | Source |
|---------|--------|
| DV01 definition, modified duration relationship | Tuckman Ch 5-6 |
| Bond-bond hedge ratio formula | Tuckman (5.7) |
| Yield-based DV01 limitation (requires yields to move together) | Tuckman Ch 5 (explicit) |
| Swap PV01 as annuity | Tuckman Ch 18 |
| Duration-based futures hedge ratio | Hull Ch 6 |
| Duration-convexity approximation | Hull |
| Risk weights, volatility ratios, regression-based hedging | Tuckman |
| CTD and conversion factor mechanics | Tuckman Ch 19-20 |
| Key-rate/bucket exposure motivation | Tuckman Ch 6 |
| DV01 vs DVDZ terminology | Tuckman |
| Asset swap spread and P&L decomposition | Tuckman |

### (B) Reasoned Inference — Derivation Logic

| Content | Derivation |
|---------|------------|
| Futures DV01 $\approx$ DV01$_{\text{CTD}}$/CF | Derived from invoice-price relationship (Tuckman CTD mechanics) |
| Variance-minimizing hedge formulation | Derived from Tuckman's regression-based hedging motivation |
| CS01 / spread duration decomposition | Derived from credit derivatives text rate+spread decomposition |

### (C) Speculation — Flagged Uncertainties

| Content | Flag |
|---------|------|
| Specific covariance numbers in Example I | Toy/illustrative numbers |
| Swap spread shock scenario in Example G | Toy basis scenario for pedagogical purposes |

---

*Last Updated: January 2026*
