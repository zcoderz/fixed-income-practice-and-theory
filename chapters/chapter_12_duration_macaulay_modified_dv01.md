# Chapter 12: Duration (Macaulay/Modified) and Mapping to DV01

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- Yield-based DV01 definition: $\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$ (Tuckman Ch 6)
- Duration definition: $D = -\frac{1}{P}\frac{dP}{dy}$ (Tuckman Ch 6)
- Macaulay-modified relationship: $D_{\text{Mac}} = (1+y/2)D_{\text{Mod}}$ (Tuckman Ch 6)
- Mapping identity: $\mathrm{DV01} = \frac{P \cdot D_{\text{Mod}}}{10{,}000}$ (Tuckman Ch 6)
- Compounding adjustment: $D^* = \frac{D}{1+y/m}$ for $m$-times-per-year compounding (Hull Ch 4)
- Yield-based measures apply only to fixed cash flows (Tuckman Ch 6)

### (B) Reasoned Inference (Derived from A)
- Unit conversions between DV01 per 100, per $1, and per $1mm face (derived from scaling)
- Portfolio DV01 additivity (follows from linearity of derivatives)
- Finite-difference approximations converge to analytic derivatives as bump size shrinks

### (C) Speculation (Clearly Labeled; Minimal)
- None in this chapter

---

## Conventions & Notation

### Defaults Used in This Chapter

| Convention | Default | Notes |
|------------|---------|-------|
| Pricing unit | Per 100 face value | Consistent with Tuckman's $P(y)$ |
| Yield convention | Annual yield, semiannual compounding | Bond market convention (BEY) |
| Coupon convention | $c$ = annual coupon per 100 face; $c/2$ paid semiannually | |
| Payment timing | Cash flows at 6-month intervals from settlement | Times are $t/2$ years for period $t$ |
| Price for sensitivity | Dirty/invoice price | Use full price when AI $\neq 0$ |
| 1 basis point | $1\text{ bp} = 0.0001$ in yield units | |

### Notation Glossary

| Symbol | Meaning | Units |
|--------|---------|-------|
| $P(y)$ | Bond price per 100 face as function of yield $y$ | Dollars per 100 face |
| $c$ | Annual coupon dollars per 100 face | Dollars/year |
| $T$ | Years to maturity | Years |
| $t$ | Semiannual period index ($t = 1, 2, \ldots, 2T$) | Periods |
| $y$ | Yield-to-maturity (annualized, semiannual compounding) | Decimal |
| $\mathrm{DV01}$ | Dollar value of a 1bp yield move | Dollars per 100 face per bp |
| $D$ | Duration $= -\frac{1}{P}\frac{dP}{dy}$ | Years |
| $D_{\text{Mod}}$ | Modified duration | Years |
| $D_{\text{Mac}}$ | Macaulay duration (PV-weighted average time) | Years |
| $F$ | Face/notional value held | Dollars |
| $MV$ | Market value $= (F/100) \cdot P$ | Dollars |

---

## Core Concepts

### 1) Yield-Based Bond Price Function $P(y)$

**Formal Definition:**

For a bond with annual coupon $c$ per 100 face and maturity $T$ years, with semiannual compounding yield $y$:

$$\boxed{P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}}$$

Equivalently (annuity form):

$$P(y) = \frac{c}{y}\left[1 - \frac{1}{(1+y/2)^{2T}}\right] + \frac{100}{(1+y/2)^{2T}}$$

**Intuition:**

Yield-to-maturity is the single discount rate (with the stated compounding convention) that prices the bond's cash flows. In the yield-based framework, you perturb one number $y$ that affects all cash flows simultaneously.

**Trading / Risk / Portfolio Practice:**

- Traders quote "yield" and "price" interchangeably using the bond's pricing function
- Risk systems often compute yield-based DV01/duration by shocking $y$ and repricing
- This is instrument-centric (bond-by-bond) rather than curve-centric (spot/forward/zero-rate bumps)

---

### 2) DV01 (General Definition vs Yield-Based Special Case)

**Formal Definition:**

General definition:

$$\boxed{\mathrm{DV01} \equiv -\frac{1}{10{,}000}\frac{dP(y)}{dy}}$$

Central-difference estimate:

$$\mathrm{DV01} \approx -\frac{P(y+\Delta y) - P(y-\Delta y)}{2 \times 10{,}000 \times \Delta y}$$

**Yield-based special case (Tuckman Ch 6):** The "rate factor" is specifically the bond's yield-to-maturity:

$$\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$$

Tuckman emphasizes that although the formula looks the same as Chapter 5, the **meaning of the derivative differs**: in Chapter 6, the single yield discounting all cash flows is perturbed; in Chapter 5, the derivative could be with respect to other interest-rate shifts (parallel forward shifts, spot shifts, etc.).

**Intuition:**

DV01 is the first-order dollar price change for a 1bp change in the chosen rate factor (here, yield). The minus sign makes DV01 typically positive for standard fixed-coupon bonds because $dP/dy < 0$.

**Trading / Risk / Portfolio Practice:**

- **Risk reports:** "DV01" is often reported as dollars per bp for a position or book
- **Hedging:** DV01 is used to size hedges because it measures absolute sensitivity (dollars), which aggregates cleanly across positions

---

### 3) Modified Duration (Percentage Price Sensitivity to Yield)

**Formal Definition:**

Tuckman defines duration as:

$$\boxed{D = -\frac{1}{P}\frac{dP}{dy}}$$

In the yield-based setting, this derivative-based duration corresponds to **modified duration** $D_{\text{Mod}}$. The explicit expression:

$$D_{\text{Mod}} = \frac{1}{P} \cdot \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]$$

**Intuition:**

Modified duration measures how many percent the bond price changes for a unit change in yield:

$$\frac{\Delta P}{P} \approx -D_{\text{Mod}} \Delta y$$

(First-order/Taylor approximation.)

**Trading / Risk / Portfolio Practice:**

- Used in "duration times spread" thinking, risk limits, and quick P&L approximations for small yield changes
- Often used to compare interest-rate sensitivity across instruments independent of price level (contrast with DV01)

---

### 4) Macaulay Duration (PV-Weighted Average Time)

**Formal Definition:**

Macaulay duration is related to modified duration by:

$$\boxed{D_{\text{Mac}} = (1+y/2) D_{\text{Mod}}}$$

Explicit PV-weighted average time form:

$$D_{\text{Mac}} = \frac{1}{P}\left[\sum_{t=1}^{2T} \left(\frac{t}{2}\right) \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]$$

**Intuition:**

Think of the bond as a portfolio of cash flows; Macaulay duration is the average "time" of the cash flows, where the averaging weights are each cash flow's share of the bond's PV.

**Trading / Risk / Portfolio Practice:**

- Less common directly on risk reports than modified duration or DV01
- Useful for intuition: higher coupons shift PV earlier $\Rightarrow$ lower Macaulay duration (Tuckman discusses this around Figure 6.1)

---

### 5) Dollar Duration and Mapping to DV01

**Formal Definition:**

From $D = -\frac{1}{P}\frac{dP}{dy}$, we have:

$$-\frac{dP}{dy} = P \cdot D_{\text{Mod}}$$

This quantity has units "dollars per unit yield" (per 1.00 = 100% yield move). Dividing by $10{,}000$ converts "per unit yield" to "per bp":

$$\boxed{\mathrm{DV01} = \frac{P \cdot D_{\text{Mod}}}{10{,}000} = \frac{P \cdot D_{\text{Mac}}}{10{,}000(1+y/2)}}$$

**Intuition:**

- Modified duration is a percent sensitivity
- DV01 is a dollar sensitivity

The difference matters: DV01 depends on both duration and the price level ("duration effect" vs "price effect").

**Trading / Risk / Portfolio Practice:**

- Traders hedge "DV01-neutral" to immunize small parallel yield shifts
- Risk managers aggregate DV01 across positions: portfolio DV01 is (to first order) the sum of position DV01s

---

### 6) Compounding Convention Differences

**Formal Definition:**

Hull notes that if yields are not continuously compounded, the "modified duration" adjustment depends on compounding frequency $m$:

$$\Delta B = -\frac{BD\Delta y}{1+y/m}, \qquad D^* = \frac{D}{1+y/m}$$

where $D^*$ is called modified duration in that convention.

**Intuition:**

The "duration number" you compute depends on how yield enters the discount function.

**Trading / Risk / Portfolio Practice:**

If one desk quotes yield semiannually (BEY) while another uses continuous compounding, the same bond can have different "modified duration" numbers unless you harmonize conventions.

---

## Math and Derivations

### Assumptions Used Throughout

- Cash flows are fixed and occur every six months from settlement (for simplicity)
- Yield-to-maturity $y$ is the single discount rate with semiannual compounding in the price function $P(y)$
- When mapping to P&L, we use the full/invoice price if accrued interest is nonzero

### 2.1 From Price Function to Derivative $dP/dy$

Start with the yield-based bond price function:

$$P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

Differentiate a generic term $(1+y/2)^{-t}$ with respect to $y$:

$$\frac{d}{dy}(1+y/2)^{-t} = -t(1+y/2)^{-t-1} \cdot \frac{1}{2} = -\frac{t}{2} \cdot \frac{1}{(1+y/2)^{t+1}}$$

Apply term-by-term:

$$\frac{dP}{dy} = \sum_{t=1}^{2T} \left(\frac{c}{2}\right)\left(-\frac{t}{2} \cdot \frac{1}{(1+y/2)^{t+1}}\right) + 100\left(-\frac{2T}{2} \cdot \frac{1}{(1+y/2)^{2T+1}}\right)$$

Factor out $-\frac{1}{1+y/2}$:

$$\boxed{\frac{dP}{dy} = -\frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

This matches Tuckman's equation (6.4).

**Unit check:**
- $P$: dollars per 100 face
- $y$: dimensionless "per year" rate
- $dP/dy$: dollars per 100 face per unit change in yield (i.e., per 1.00 = 100% yield move)

### 2.2 DV01 from $dP/dy$

Tuckman's yield-based DV01:

$$\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$$

Substitute (6.4) into the DV01 definition:

$$\boxed{\mathrm{DV01} = \frac{1}{10{,}000} \cdot \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

This is Tuckman's (6.5).

**Sanity check:** For normal fixed-coupon bonds, $dP/dy < 0$, so DV01 $> 0$.

### 2.3 Duration and Its Link to Macaulay/Modified

Tuckman's duration definition:

$$D = -\frac{1}{P}\frac{dP}{dy}$$

Substituting (6.4) yields:

$$D = \frac{1}{P} \cdot \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]$$

This is precisely Tuckman's expression for modified duration $D_{\text{Mod}}$.

Then define Macaulay duration via:

$$D_{\text{Mac}} = (1+y/2)D_{\text{Mod}}$$

So:

$$\boxed{D_{\text{Mac}} = \frac{1}{P}\left[\sum_{t=1}^{2T} \left(\frac{t}{2}\right) \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

**Unit check:**
- $D_{\text{Mac}}$: weighted average of times $t/2$ $\Rightarrow$ units are years
- $D_{\text{Mod}}$: equals $D_{\text{Mac}}/(1+y/2)$ $\Rightarrow$ also in years

### 2.4 Mapping Duration to DV01

Combine:

$$D_{\text{Mod}} = -\frac{1}{P}\frac{dP}{dy} \quad \Rightarrow \quad -\frac{dP}{dy} = P \cdot D_{\text{Mod}}$$

Divide by $10{,}000$:

$$\boxed{\mathrm{DV01} = \frac{P \cdot D_{\text{Mod}}}{10{,}000}}$$

Tuckman explicitly writes:

$$\mathrm{DV01} = \frac{P \times D_{\text{Mod}}}{10{,}000} = \frac{P \times D_{\text{Mac}}}{10{,}000(1+y/2)}$$

**Sanity check:**
- If $P$ doubles holding duration fixed, DV01 doubles
- If $D_{\text{Mod}}$ doubles holding price fixed, DV01 doubles

### 2.5 Compounding Convention Differences

Hull warns that the "modified duration" adjustment depends on how yield is compounded. If yield $y$ is compounded $m$ times per year:

$$\Delta B = -\frac{BD\Delta y}{1+y/m}, \qquad D^* = \frac{D}{1+y/m}$$

**Practical implication:** If your desk uses BEY (semiannual, $m=2$), then the $(1+y/2)$ adjustment in Tuckman's Macaulay-modified relationship is exactly the $m=2$ special case of this broader compounding logic.

---

## Measurement & Risk

### Macaulay Duration: PV-Weighted Average Time

Under the yield-to-maturity pricing function with semiannual compounding, Macaulay duration is the PV-weighted average time of cash flows in years.

**Interpretation:** A bond with higher coupons has more PV in early cash flows, reducing this weighted average time (Tuckman's Figure 6.1 discussion).

### Modified Duration: Percent Price Sensitivity

Defined by the derivative-based duration $D = -\frac{1}{P}\frac{dP}{dy}$.

**First-order approximation:**

$$\frac{\Delta P}{P} \approx -D_{\text{Mod}} \Delta y \quad \Longleftrightarrow \quad \Delta P \approx -P \cdot D_{\text{Mod}} \Delta y$$

**Yield dependence:** Increasing yield tends to lower duration because the longest cash flows lose relative weight in PV terms as yields rise (Tuckman).

### Dollar Duration / DV01 Mapping

Using Tuckman's definitions:

- **Dollar Duration** (per 100 face) is $-\frac{dP}{dy}$
- **DV01** (per 100 face) is Dollar Duration scaled to 1bp:

$$\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$$

**Mapping to duration:**

$$-\frac{dP}{dy} = P \cdot D_{\text{Mod}} \quad \Rightarrow \quad \mathrm{DV01} = \frac{P \cdot D_{\text{Mod}}}{10{,}000}$$

**Naming reconciliation:** Many practitioners casually call $P \cdot D_{\text{Mod}}$ "dollar duration" and $P \cdot D_{\text{Mod}}/10{,}000$ "DV01". Under Tuckman's convention, DV01 is explicitly the per-bp scaled derivative.

### DV01 = Modified Duration $\times$ Price $\times$ 1bp

Since $1\text{ bp} = 0.0001 = 1/10{,}000$:

$$\mathrm{DV01} = P \cdot D_{\text{Mod}} \cdot 0.0001$$

### Notional Unit Clarification

If a bond price $P$ is quoted "per 100", it means: $P paid for $100 face.

If $\mathrm{DV01}_{100}$ is computed using $P$ per 100 face, then:

| Quantity | Formula |
|----------|---------|
| DV01 per $1 face | $\mathrm{DV01}_{\$1} = \frac{\mathrm{DV01}_{100}}{100}$ |
| DV01 per $1mm face | $\mathrm{DV01}_{\$1\text{mm}} = \mathrm{DV01}_{\$1} \times 1{,}000{,}000 = \mathrm{DV01}_{100} \times 10{,}000$ |
| Total DV01 for face $F$ | $\mathrm{DV01}_{\text{total}} = \frac{F}{100} \cdot \mathrm{DV01}_{100}$ |

### Limitations

**Duration is a first-order measure (convexity error for larger moves):**

DV01 and modified duration come from first derivatives. For larger yield shocks, the price-yield curve curvature makes the linear approximation imperfect (residual is related to convexity; full treatment is next chapter). Tuckman defines convexity as $C = \frac{1}{P}\frac{d^2P}{dy^2}$.

**Duration depends on yield definition and cash flow assumptions:**

- Yield compounding convention matters (semiannual vs other $m$), and Hull explicitly shows the adjustment depends on $m$
- Using clean vs dirty price inconsistently will distort DV01 scaling; use invoice price when accrued is nonzero

**Option-embedded instruments require effective duration (preview only):**

Yield-based measures assume a fixed set of cash flows. For securities where cash flows change with rates (e.g., callable bonds, MBS), revaluing under shifted rates is needed. Tuckman flags that yield-based measures "can be reasonably used only for securities with fixed cash flows."

**Effective duration idea (preview):** Compute a central-difference sensitivity by repricing the option-embedded instrument under $y \pm \Delta y$ (or under a curve shift) and forming $-\frac{1}{P}\frac{\Delta P}{\Delta y}$. This is conceptually consistent with DV01's finite-difference definition.

---

## Worked Examples

### Common Conventions for All Examples

- Price is quoted per 100 face
- Yield $y$ is annual with semiannual compounding; discount factor per period is $(1+y/2)^{-t}$
- Settlement assumed on a coupon date $\Rightarrow$ accrued interest $= 0$ $\Rightarrow$ clean $=$ dirty
- Day count: ignored for pedagogy; each coupon period is exactly 0.5 years

---

### Example A: Zero-Coupon Benchmark

**Instrument:**
- Zero-coupon bond, Face $= 100$, Maturity $T = 5$ years
- Yield $y = 4.00\% = 0.04$ (semiannual compounding)

**Step 1: Price**

$$P = \frac{100}{(1+y/2)^{2T}} = \frac{100}{(1.02)^{10}} = \frac{100}{1.02^{10}} = 82.03482999$$

**Step 2: Macaulay Duration**

A zero has a single cash flow at time $T$, so PV-weighted average time is exactly:

$$D_{\text{Mac}} = T = 5 \text{ years}$$

**Step 3: Modified Duration**

Tuckman gives the zero-coupon special case:

$$\left. D_{\text{Mod}} \right|_{c=0} = \frac{T}{1+y/2} = \frac{5}{1.02} = 4.90196078 \text{ years}$$

**Step 4: DV01 (two equivalent ways)**

From the zero-coupon special case:

$$\left. \mathrm{DV01} \right|_{c=0} = \frac{T}{100(1+y/2)^{2T+1}} = \frac{5}{100 \cdot 1.02^{11}} = 0.040213152$$

From the mapping $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$:

$$\mathrm{DV01} = \frac{82.03482999 \times 4.90196078}{10{,}000} = 0.040213152$$

**Unit checks:**
- $D_{\text{Mac}}$ and $D_{\text{Mod}}$ are in years
- DV01 is dollars per 100 face per 1bp

**Maturity limit sanity checks:**
- As $y \to 0$: $D_{\text{Mod}} = T/(1+y/2) \to T$
- As $T \to 0$: $D_{\text{Mac}} \to 0$ and $D_{\text{Mod}} \to 0$

---

### Example B: Coupon Bond Duration by Cash Flow Weights

**Instrument:**
- Coupon bond, maturity $T = 3$ years, semiannual coupons
- Annual coupon $c = 6$ $\Rightarrow$ coupon each 6 months $= c/2 = 3$
- Face $= 100$, Yield $y = 5.00\% = 0.05$ (semiannual compounding)

**Step 1: Discount Rate per Period**

$$r = \frac{y}{2} = 0.025, \quad (1+r) = 1.025$$

**Step 2: Cash Flows and PVs**

| Period $t$ | Time $t/2$ (yrs) | Cash Flow | DF $(1.025)^{-t}$ | PV $= \text{CF} \times \text{DF}$ | Weight $w_t = \text{PV}/P$ | $w_t \cdot (t/2)$ |
|------------|------------------|-----------|-------------------|-----------------------------------|----------------------------|-------------------|
| 1 | 0.5 | 3 | 0.975609756 | 2.926829268 | 0.028483830 | 0.014241915 |
| 2 | 1.0 | 3 | 0.951814396 | 2.855443189 | 0.027789103 | 0.027789103 |
| 3 | 1.5 | 3 | 0.928599411 | 2.785798233 | 0.027111320 | 0.040666979 |
| 4 | 2.0 | 3 | 0.905950645 | 2.717851934 | 0.026450068 | 0.052900136 |
| 5 | 2.5 | 3 | 0.883854288 | 2.651562863 | 0.025804944 | 0.064512361 |
| 6 | 3.0 | 103 | 0.862296866 | 88.816577194 | 0.864360735 | 2.593082206 |

**Step 3: Price**

$$P = \sum \text{PV} = 102.754062681$$

**Step 4: Macaulay Duration (PV-Weighted Average Time)**

$$D_{\text{Mac}} = \sum_{t=1}^{6} w_t \cdot (t/2) = 2.793192700 \text{ years}$$

**Step 5: Modified Duration**

Using $D_{\text{Mac}} = (1+y/2) D_{\text{Mod}}$:

$$D_{\text{Mod}} = \frac{D_{\text{Mac}}}{1+y/2} = \frac{2.793192700}{1.025} = 2.725066049 \text{ years}$$

---

### Example C: Finite-Difference Check

Use the same bond as Example B.

**Goal:** Compute modified duration by central difference around $y = 5\%$ with $\Delta y = 1\text{ bp} = 0.0001$, then compare to Example B's cash-flow-weight result.

**Step 1: Reprice at $y \pm 1\text{bp}$**

- $y^+ = 0.0501 \Rightarrow r^+ = 0.02505$
- $y^- = 0.0499 \Rightarrow r^- = 0.02495$

Using the annuity form of $P(y)$:

$$P(y) = \frac{c}{y}\left[1 - \frac{1}{(1+y/2)^{2T}}\right] + \frac{100}{(1+y/2)^{2T}}$$

Computed prices:
- $P(y^+) = 102.726066184$
- $P(y^-) = 102.782068507$

**Step 2: Central Difference Approximation to Derivative**

$$\frac{dP}{dy} \approx \frac{P(y^+) - P(y^-)}{2\Delta y} = \frac{102.726066184 - 102.782068507}{0.0002} = -280.011613583$$

**Step 3: Modified Duration**

From $D_{\text{Mod}} = -\frac{1}{P}\frac{dP}{dy}$:

$$D_{\text{Mod,FD}} = -\frac{1}{P(y)}\left(\frac{dP}{dy}\right) = \frac{280.011613583}{102.754062681} = 2.725066107$$

**Comparison:**
- Cash-flow-weight formula (Example B): $D_{\text{Mod}} = 2.725066049$
- Finite difference: $D_{\text{Mod,FD}} = 2.725066107$
- Difference is tiny (finite-difference and rounding)

---

### Example D: Map to DV01

Use Example B/C bond.

**Step 1: DV01 per 100 Face**

From Tuckman's identity:

$$\mathrm{DV01} = \frac{P \cdot D_{\text{Mod}}}{10{,}000}$$

Compute:

$$\mathrm{DV01}_{100} = \frac{102.754062681 \times 2.725066049}{10{,}000} = 0.02800116076$$

**Interpretation + Sign:**

For a +1bp increase in yield:
$$\Delta P \approx \frac{dP}{dy} \cdot 0.0001 \approx -\mathrm{DV01}_{100} = -0.02800116$$

For a -1bp decrease in yield:
$$\Delta P \approx +0.02800116$$

**Step 2: Scale to a $10mm Face Position**

Face $F = 10{,}000{,}000$. Number of "per-100" units is $F/100 = 100{,}000$.

$$\mathrm{DV01}_{\text{total}} = \frac{F}{100} \cdot \mathrm{DV01}_{100} = 100{,}000 \times 0.02800116076 = 2{,}800.116076$$

So the position changes by about **\$2,800 per bp** (in absolute value).

---

### Example E: "Per 100" vs "Per $" vs "Per $1mm"

Use Example B bond, with $\mathrm{DV01}_{100} = 0.02800116076$.

**DV01 per 100 Face (Price-Quote Units):**

$$\mathrm{DV01}_{100} = 0.02800116076 \text{ dollars per bp per 100 face}$$

**DV01 per \$1 Face:**

$$\mathrm{DV01}_{\$1} = \frac{0.02800116076}{100} = 0.000280011608 \text{ dollars per bp per \$1 face}$$

**DV01 per \$1mm Face:**

$$\mathrm{DV01}_{\$1\text{mm}} = \mathrm{DV01}_{\$1} \times 1{,}000{,}000 = 280.0116076 \text{ dollars per bp per \$1mm face}$$

**Conversion Quick Rule:**

When price is per 100, multiply $\mathrm{DV01}_{100}$ by 10,000 to get DV01 per \$1mm face:

$$\mathrm{DV01}_{\$1\text{mm}} = \mathrm{DV01}_{100} \times 10{,}000$$

---

### Example F: Premium vs Discount Bond

Two bonds with same maturity and coupon frequency but different yields/prices.

**Instruments:**
- Both: $T = 5$ years, semiannual coupons, coupon $c = 6$ (i.e., \$3 each 6 months), face $= 100$
- Bond 1 (premium): yield $y = 4\%$
- Bond 2 (discount): yield $y = 8\%$

We use $P(y)$ and the compact DV01 expression derived from (6.2):

$$\mathrm{DV01} = \frac{1}{10{,}000}\left[\frac{c}{y^2}\left(1 - \frac{1}{(1+y/2)^{2T}}\right) + \left(100 - \frac{c}{y}\right)\frac{T}{(1+y/2)^{2T+1}}\right] \tag{6.11}$$

Then $D_{\text{Mod}} = \mathrm{DV01} \cdot 10{,}000 / P$.

---

**Bond 1: $y = 4\%$ (Premium)**

**Step 1: Price**

$$P = 3 \cdot \frac{1 - (1.02)^{-10}}{0.02} + 100 \cdot (1.02)^{-10} = 108.98258501$$

**Step 2: DV01 via (6.11) Components**

- $df = (1.02)^{-10} = 0.8203482999$
- $1 - df = 0.1796517001$

Term A:
$$\frac{c}{y^2}(1-df) = \frac{6}{0.04^2} \cdot 0.1796517001 = 673.69387547$$

Term B:
$$\left(100 - \frac{c}{y}\right)\frac{T}{(1+y/2)^{2T+1}} = \left(100 - \frac{6}{0.04}\right)\frac{5}{1.02^{11}} = -201.06575977$$

Sum $= 472.62811569$

So:
$$\mathrm{DV01}_{100} = \frac{472.62811569}{10{,}000} = 0.04726281157$$

**Step 3: Modified Duration**

$$D_{\text{Mod}} = \frac{\mathrm{DV01} \cdot 10{,}000}{P} = \frac{0.04726281157 \cdot 10{,}000}{108.98258501} = 4.336730641$$

---

**Bond 2: $y = 8\%$ (Discount)**

**Step 1: Price**

$$P = 3 \cdot \frac{1 - (1.04)^{-10}}{0.04} + 100 \cdot (1.04)^{-10} = 91.88910422$$

**Step 2: DV01 via (6.11) Components**

- $df = (1.04)^{-10} = 0.6755641688$
- $1 - df = 0.3244358312$

Term A:
$$\frac{6}{0.08^2} \cdot 0.3244358312 = 304.15859173$$

Term B:
$$\left(100 - \frac{6}{0.08}\right)\frac{5}{1.04^{11}} = 81.19761645$$

Sum $= 385.35620817$

So:
$$\mathrm{DV01}_{100} = \frac{385.35620817}{10{,}000} = 0.03853562082$$

**Step 3: Modified Duration**

$$D_{\text{Mod}} = \frac{0.03853562082 \cdot 10{,}000}{91.88910422} = 4.193709488$$

**Comparison + Explanation:**

| Bond | Price $P$ | $D_{\text{Mod}}$ | DV01 |
|------|-----------|------------------|------|
| Premium ($y=4\%$) | 108.98 | 4.34 | 0.0473 |
| Discount ($y=8\%$) | 91.89 | 4.19 | 0.0385 |

DV01 is larger for the premium bond because DV01 scales with price $\times$ duration (absolute vs percentage sensitivity). Tuckman explicitly frames this as "duration effect" and "price effect" in $\mathrm{DV01} = P \times D_{\text{Mod}} / 10{,}000$.

---

### Example G: Duration-Based Price Change Approximation

Use Example B bond, and apply a +25bp yield shock.

**Inputs:**
- $P_0 = 102.754062681$
- $D_{\text{Mod}} = 2.725066049$
- $\mathrm{DV01}_{100} = 0.02800116076$
- $\Delta y = +25\text{ bp} = +0.0025$

**(i) Linear Approximation Using Modified Duration:**

$$\Delta P_{\text{lin}} \approx -P_0 \cdot D_{\text{Mod}} \cdot \Delta y = -(102.754062681)(2.725066049)(0.0025) = -0.700029019$$

**(ii) Using DV01 $\times$ 25:**

$$\Delta P_{\text{DV01}} \approx -\mathrm{DV01}_{100} \times 25 = -(0.02800116076) \times 25 = -0.700029019$$

(Exactly the same because $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$.)

**(iii) Exact Repricing:**

Reprice at $y_1 = 0.0525$ using $P(y)$:

$$P_1 = 102.056939713$$

So:

$$\Delta P_{\text{exact}} = P_1 - P_0 = 102.056939713 - 102.754062681 = -0.697122968$$

**Residual (Convexity Effect):**

$$\Delta P_{\text{exact}} - \Delta P_{\text{lin}} = (-0.697122968) - (-0.700029019) = +0.002906051$$

The linear approximation slightly overstates the price drop for a yield increase because standard fixed-coupon bonds typically exhibit **positive convexity** (curvature makes the price-yield curve "bow" upward). Tuckman defines convexity as a second-derivative measure; full derivation is next chapter.

---

### Example H: Portfolio Duration and DV01 Additivity

**Goal:** Show:
1. Portfolio DV01 is the signed sum of position DV01s
2. "Portfolio duration = DV01 / price" can be misleading depending on what you mean by "price," especially for long/short portfolios

**Bond A (same as Example B):**
- $T = 3$, $c = 6$, $y = 5\%$
- $P_A = 102.754062681$
- $D_{\text{Mod},A} = 2.725066049$
- $\mathrm{DV01}_{A,100} = 0.02800116076$

**Bond B (new bond):**
- $T = 7$ years, semiannual coupons
- $c = 4$ $\Rightarrow$ coupon each 6 months $= 2$
- $y = 5\%$

Price:
$$P_B = 2 \cdot \frac{1 - (1.025)^{-14}}{0.025} + 100 \cdot (1.025)^{-14} = 94.154543915$$

Using the mapping $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$:
- $D_{\text{Mod},B} = 5.993355618$
- $\mathrm{DV01}_{B,100} = 0.05643016647$

**Portfolio Positions:**
- Long $F_A = \$5{,}000{,}000$ face of Bond A
- Short $F_B = \$5{,}456{,}670.41$ face of Bond B (chosen to make market values match)

**Step 1: Market Value of Each Position**

$$MV_A = \frac{F_A}{100} \cdot P_A = 50{,}000 \times 102.754062681 = 5{,}137{,}703.134$$

$$MV_B = \frac{F_B}{100} \cdot P_B = 54{,}566.7041 \times 94.154543915 = 5{,}137{,}703.134$$

So net market value (signed) is:
$$MV_{\text{net}} = MV_A - MV_B = 0$$

**Step 2: Position DV01s (Dollars per bp)**

$$\mathrm{DV01}_A = \frac{F_A}{100} \cdot \mathrm{DV01}_{A,100} = 50{,}000 \times 0.02800116076 = 1{,}400.058038$$

$$\mathrm{DV01}_B = \frac{F_B}{100} \cdot \mathrm{DV01}_{B,100} = 54{,}566.7041 \times 0.05643016647 = 3{,}079.208194$$

Short position contributes $-\mathrm{DV01}_B$.

**Step 3: Portfolio DV01 Additivity**

$$\mathrm{DV01}_{\text{port}} = \mathrm{DV01}_A - \mathrm{DV01}_B = 1{,}400.058038 - 3{,}079.208194 = -1{,}679.150156$$

**Interpretation:** Portfolio is **net short DV01** (it loses when yields fall by 1bp; gains when yields rise by 1bp, to first order).

**Step 4: Portfolio "Duration" Computed from DV01 / Price**

Using the identity $D_{\text{Mod}} = \frac{\mathrm{DV01} \cdot 10{,}000}{P}$:

If you define portfolio price/value as **net market value** $MV_{\text{net}}$, then:
$$D_{\text{Mod,port(net)}} = \frac{\mathrm{DV01}_{\text{port}} \cdot 10{,}000}{MV_{\text{net}}}$$
is **undefined** because $MV_{\text{net}} = 0$.

If instead you define portfolio value as gross market value $MV_{\text{gross}} = |MV_A| + |MV_B|$:
$$MV_{\text{gross}} = 5{,}137{,}703.134 + 5{,}137{,}703.134 = 10{,}275{,}406.268$$

Then:
$$D_{\text{Mod,port(gross)}} = \frac{-1{,}679.150156 \cdot 10{,}000}{10{,}275{,}406.268} = -1.633953478$$

**Discussion: Why Portfolio Duration Can Mislead**

- **DV01 is in dollars per bp and adds cleanly** across long/short positions (it directly predicts small-shock P&L)
- **"Portfolio duration = DV01 / price"** depends critically on which "price" you use:
  - Net MV can be near zero for hedged trades $\Rightarrow$ duration can blow up or be undefined
  - Gross MV produces a stable ratio but corresponds to a different economic question (risk per unit gross exposure)
- For portfolios, duration-based summaries also assume a parallel shift in the relevant rates/curve; Hull emphasizes that portfolio duration estimates are reliable only when the zero curve experiences a parallel shift

---

## Practical Notes

### Common Pitfalls

**Mixing clean vs dirty price when computing duration/DV01:**

If you compute DV01 from a dirty price but report it against a clean price (or vice versa), the mapping $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$ becomes inconsistent. Use invoice price when accrued is nonzero.

**Inconsistent yield conventions (BEY vs effective annual vs continuous):**

Modified duration depends on the yield compounding convention; Hull shows the $1 + y/m$ adjustment explicitly.

**Forgetting coupon frequency in modified duration formula:**

The $(1+y/2)$ term is specific to semiannual compounding in Tuckman's Chapter 6 setting.

**Confusing Macaulay duration (time measure) with modified duration (% sensitivity):**

They are related but not identical: $D_{\text{Mac}} = (1+y/2) D_{\text{Mod}}$.

**Reporting DV01 with the wrong notional unit (per 100 vs per 1mm):**

Always state whether DV01 is per 100 face, per \$1, or per \$1mm.

### Implementation Pitfalls

**Stubs/odd first/last coupons:**

Tuckman notes the 6-month spacing is an expositional simplification; real schedules need the general case.

**Day-count mismatches between accrual and yield conventions:**

Accrued interest uses day count; yield compounding conventions may not align. If accrued interest is nonzero, use invoice price for sensitivity mapping.

**Numerical stability: bump size choice and rounding:**

Central differences reduce error relative to forward differences (consistent with the central-difference DV01 approximation).

### Verification Tests

**Monotonicity: yield $\uparrow$ $\Rightarrow$ price $\downarrow$ $\Rightarrow$ modified duration should be positive:**

Since $D = -\frac{1}{P}\frac{dP}{dy}$, and typically $dP/dy < 0$, $D > 0$.

**Zero-coupon check: Macaulay $\approx$ maturity; modified depends on yield compounding:**

For a zero, $D_{\text{Mod}} = T/(1+y/2)$.

**Scaling: doubling notional doubles DV01:**

DV01 is linear in $P$ and in position size: $\mathrm{DV01} \propto P$.

**Repricing check: "duration-implied" price change matches exact repricing for very small shocks:**

Use the finite-difference DV01 approximation as a consistency check.

---

## Summary & Recall

### 10-Bullet Executive Summary

1. In Tuckman's yield-based framework, bond price is $P(y)$ with semiannual compounding discounting $(1+y/2)^{-t}$

2. Yield-based DV01 is $\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$, i.e., dollars per 100 face per 1bp yield move

3. Duration is $D = -\frac{1}{P}\frac{dP}{dy}$; in this setting it corresponds to modified duration $D_{\text{Mod}}$

4. Macaulay and modified duration relate by $D_{\text{Mac}} = (1+y/2) D_{\text{Mod}}$

5. Macaulay duration is a PV-weighted average time of cash flows (years)

6. Mapping identity: $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$

7. DV01 is an absolute dollar sensitivity; duration is a percentage sensitivity—so DV01 depends on price as well as duration ("price effect" vs "duration effect")

8. DV01 per 100 face scales to a position by multiplying by $F/100$

9. Yield-based DV01 hedges assume parallel yield shifts across the hedged instruments; otherwise the hedge may fail

10. Conventions matter: yield compounding frequency affects modified duration (Hull's $1+y/m$ adjustment)

### Cheat Sheet: Key Definitions + Mapping Identities + Unit Conversions

**Price (per 100 face):**
$$P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

**DV01 (yield-based):**
$$\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$$

**Modified duration:**
$$D_{\text{Mod}} = -\frac{1}{P}\frac{dP}{dy}$$

**Macaulay duration:**
$$D_{\text{Mac}} = (1+y/2) D_{\text{Mod}}$$

**Mapping:**
$$\mathrm{DV01} = \frac{P \cdot D_{\text{Mod}}}{10{,}000} = P \cdot D_{\text{Mod}} \cdot 0.0001$$

**Units:**
- $D_{\text{Mod}}$: years (interpreted as % change per unit yield)
- $\mathrm{DV01}_{100}$: dollars per bp per 100 face

**Conversions:**
$$\mathrm{DV01}_{\$1} = \frac{\mathrm{DV01}_{100}}{100}, \quad \mathrm{DV01}_{\$1\text{mm}} = \mathrm{DV01}_{100} \times 10{,}000, \quad \mathrm{DV01}_{\text{total}} = \frac{F}{100} \cdot \mathrm{DV01}_{100}$$

---

### Flashcards (25 Q/A)

**Q1:** What is the yield-based bond price function $P(y)$ in Tuckman's Chapter 6?
**A:** $P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$

**Q2:** Define yield-based DV01.
**A:** $\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$ where $y$ is yield-to-maturity.

**Q3:** Why does Tuckman say the derivative in (6.3) "means something different" than in (5.2)?
**A:** In Chapter 6 the single bond yield is perturbed; in Chapter 5 DV01 can be with respect to other interest-rate shifts (forward/spot/volatility-weighted shifts).

**Q4:** Define duration $D$ in Tuckman.
**A:** $D = -\frac{1}{P}\frac{dP}{dy}$

**Q5:** In the yield-based setting, what does $D$ correspond to?
**A:** Modified duration $D_{\text{Mod}}$

**Q6:** What is the relationship between Macaulay and modified duration under semiannual compounding?
**A:** $D_{\text{Mac}} = (1+y/2) D_{\text{Mod}}$

**Q7:** What is Macaulay duration conceptually?
**A:** PV-weighted average time of cash flows (in years).

**Q8:** What is the key mapping from duration to DV01?
**A:** $\mathrm{DV01} = \frac{P \cdot D_{\text{Mod}}}{10{,}000}$

**Q9:** What does "DV01 per 100" mean?
**A:** Dollar price change per 1bp for a bond with \$100 face, when price is quoted per 100 face.

**Q10:** How do you scale DV01 per 100 to a position with face $F$?
**A:** Multiply by $F/100$: $\mathrm{DV01}_{\text{total}} = (F/100) \cdot \mathrm{DV01}_{100}$

**Q11:** Under standard definitions, is DV01 positive or negative for a plain fixed-coupon bond?
**A:** Positive, because $dP/dy < 0$ and DV01 includes a minus sign.

**Q12:** What is the sign of $\Delta P$ for a +1bp yield shock?
**A:** Approximately $\Delta P \approx -\mathrm{DV01}$ (price decreases).

**Q13:** How is modified duration interpreted in risk terms?
**A:** Percent price sensitivity: $\Delta P / P \approx -D_{\text{Mod}} \Delta y$

**Q14:** Why can two bonds have the same duration but different DV01?
**A:** DV01 depends on price level as well: $\mathrm{DV01} = P \times D_{\text{Mod}} / 10{,}000$

**Q15:** What are Tuckman's "duration effect" and "price effect"?
**A:** Duration tends to rise with maturity, raising DV01; but DV01 also depends on how price changes with maturity (price effect can raise or lower DV01).

**Q16:** What is the central-difference estimator for DV01?
**A:** $\mathrm{DV01} \approx -\frac{P(y+\Delta y) - P(y-\Delta y)}{2 \cdot 10{,}000 \cdot \Delta y}$

**Q17:** What is the modified duration of a zero-coupon bond in Tuckman's setting?
**A:** $D_{\text{Mod}} = T/(1+y/2)$

**Q18:** What is the Macaulay duration of a zero-coupon bond?
**A:** Exactly $T$ (only cash flow is at maturity).

**Q19:** What is the yield-based DV01 limitation emphasized by Tuckman?
**A:** Hedge works only if yields move by the same amount (parallel yield shifts).

**Q20:** Why does yield-based DV01 fail for callable/MBS-like instruments?
**A:** Cash flows are not fixed; yield-based measures are reasonably used only for fixed cash flows.

**Q21:** What does Hull say happens to the duration mapping when yield compounding changes?
**A:** Modified duration adjusts by $1+y/m$: $D^* = D/(1+y/m)$

**Q22:** What is the quick conversion from DV01 per 100 to DV01 per \$1mm face?
**A:** Multiply by 10,000: $\mathrm{DV01}_{\$1\text{mm}} = \mathrm{DV01}_{100} \times 10{,}000$

**Q23:** Why can portfolio duration be misleading for long/short portfolios?
**A:** If you divide portfolio DV01 by net market value, net MV can be near zero, making the ratio explode or undefined.

**Q24:** What curve-shift assumption underlies duration/DV01 summaries for portfolios?
**A:** Parallel shifts; Hull notes portfolio duration approximations require a parallel shift in the zero curve.

**Q25:** What is the simplest internal consistency check between DV01 and modified duration?
**A:** Verify $\mathrm{DV01} \times 10{,}000 / P \approx D_{\text{Mod}}$

---

## Mini Problem Set (14 Questions)

*Increasing difficulty. Solution sketches provided for questions 1-7 only.*

---

**Q1.** A 4-year zero-coupon bond has yield $y = 6\%$ (semiannual compounding). Compute $P$, $D_{\text{Mac}}$, $D_{\text{Mod}}$, and DV01 per 100.

*Solution sketch:* Use $P = 100/(1+y/2)^{2T}$. For a zero, $D_{\text{Mac}} = T$ and $D_{\text{Mod}} = T/(1+y/2)$. Use $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$ or the zero special case.

---

**Q2.** For a 2-year bond with coupon $c = 8$ and yield $y = 8\%$, compute the price per 100 and the Macaulay duration by PV weights.

*Solution sketch:* Compute 4 semiannual cash flows (\$4 each; last is \$104). Discount with $1+y/2$. Compute $P$. Compute weights $w_t = PV_t / P$. Then $D_{\text{Mac}} = \sum (t/2) w_t$.

---

**Q3.** Using the bond from Q2, compute modified duration using $D_{\text{Mod}} = D_{\text{Mac}} / (1+y/2)$.

*Solution sketch:* Apply $D_{\text{Mac}} = (1+y/2) D_{\text{Mod}}$.

---

**Q4.** Using the bond from Q2, compute DV01 per 100 using the mapping identity.

*Solution sketch:* $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$.

---

**Q5.** For the bond from Q2, compute DV01 by central difference with $\Delta y = 1\text{bp}$ and compare to Q4.

*Solution sketch:* Compute $P(y+\Delta)$ and $P(y-\Delta)$, then use $\mathrm{DV01} \approx -\frac{P(y+\Delta) - P(y-\Delta)}{2 \cdot 10{,}000 \cdot \Delta}$. Differences should be tiny if bump is small.

---

**Q6.** A position holds \$25mm face of a bond with $\mathrm{DV01}_{100} = 0.055$. Compute DV01 in dollars per bp for the position.

*Solution sketch:* Multiply by face/100: $25{,}000{,}000/100 = 250{,}000$ units; $\mathrm{DV01} = 250{,}000 \times 0.055$.

---

**Q7.** You have two bonds with DV01s $\mathrm{DV01}_{1,100} = 0.040$ and $\mathrm{DV01}_{2,100} = 0.060$. You are long \$10mm face of bond 1. How much face of bond 2 should you short to be DV01-neutral?

*Solution sketch:* Set $(F_1/100) \cdot \mathrm{DV01}_{1,100} - (F_2/100) \cdot \mathrm{DV01}_{2,100} = 0$. Solve $F_2 = F_1 \cdot \mathrm{DV01}_{1,100} / \mathrm{DV01}_{2,100}$. (DV01 neutrality is a first-order hedge.)

---

**Q8.** Show algebraically that $\mathrm{DV01} = P \times D_{\text{Mac}} / [10{,}000(1+y/2)]$.

---

**Q9.** For a 10-year par bond at yield 5%, use Tuckman's par-bond special case formula to compute $D_{\text{Mod}}$ and $D_{\text{Mac}}$.

---

**Q10.** Explain why increasing yield tends to lower modified duration for fixed-coupon bonds. Provide the PV-weight intuition.

---

**Q11.** Consider a market-value-neutral long/short portfolio. Explain why "portfolio duration = DV01/price" can become unstable or undefined.

---

**Q12.** Compare DV01 and duration for two bonds with identical cash-flow timing but different prices due to different yields.

---

**Q13.** If your system reports yield in annual effective terms instead of BEY, what changes in duration formulas?

---

**Q14.** Give one scenario where a yield-based DV01 hedge fails because yield shifts are not parallel.

---

## Source Map

### (A) Verified Facts — Source Citations

| Fact | Source |
|------|--------|
| Yield-based bond price function $P(y)$ | Tuckman Ch 6, Eq. (6.1)-(6.2) |
| DV01 definition $= -\frac{1}{10{,}000}\frac{dP}{dy}$ | Tuckman Ch 6, Eq. (6.3) |
| Duration definition $D = -\frac{1}{P}\frac{dP}{dy}$ | Tuckman Ch 6, Eq. (6.6) |
| Modified duration explicit formula | Tuckman Ch 6, Eq. (6.7) |
| Macaulay-modified relationship $D_{\text{Mac}} = (1+y/2)D_{\text{Mod}}$ | Tuckman Ch 6 |
| DV01-duration mapping $\mathrm{DV01} = P \cdot D_{\text{Mod}} / 10{,}000$ | Tuckman Ch 6, Eq. (6.8) |
| Central-difference DV01 approximation | Tuckman Ch 5 |
| Compounding adjustment $D^* = D/(1+y/m)$ | Hull Ch 4 |
| Yield-based measures valid only for fixed cash flows | Tuckman Ch 6 |
| Portfolio duration requires parallel curve shift | Hull Ch 4 |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Unit conversions (per 100 $\to$ per \$1mm) | Linear scaling from price-per-100 convention |
| Portfolio DV01 additivity | Linearity of derivatives; sum of position sensitivities |
| Finite-difference convergence | Taylor series truncation; central difference has $O(\Delta y^2)$ error |
| Premium bond has higher DV01 than discount at same duration | $\mathrm{DV01} = P \cdot D / 10{,}000$; higher $P$ raises DV01 |

### (C) Speculation — Flagged Uncertainties

None in this chapter. All content is sourced from Tuckman Ch 5-6 or Hull Ch 4.

---

*Chapter 12 complete.*
