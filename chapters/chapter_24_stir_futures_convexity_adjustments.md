# Chapter 24: STIR Futures and Convexity Adjustments

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- STIR futures quoting convention: $Q = 100 - R^{\text{fut}}$ where $R^{\text{fut}}$ is in percent (Hull, Tuckman, Interest-rate-modeling)
- Forward rate formula from discount factors (Interest-rate-modeling)
- Convexity adjustment definition: Forward = Futures $- c$ (Hull)
- Eurodollar/SOFR futures DV01 = \$25/bp per contract (Hull, Interest-rate-modeling)
- Fed funds futures DV01 = \$41.67/bp for 30-day month (Tuckman)
- Tuckman approximation for futures-forward difference under normal model assumptions
- When interest rates are constant, forward and futures prices coincide (Hull)

### (B) Reasoned Inference (Derived from A)
- Convexity adjustment grows with $\sigma^2$ and $t$ (follows from Tuckman approximation formula)
- Deterministic-rate limit implies $c \to 0$ (follows from Hull's statement about constant rates)
- Ignoring convexity adjustment biases forward curve extraction (follows from definition)

### (C) Speculation (Clearly Labeled; Minimal)
- Exact exchange tick sizes, settlement calendars, or index definitions beyond what is cited: I'm not sure
- Repo day-count conventions outside U.S. Treasury repo: I'm not sure without additional source verification

---

## Conventions & Notation

| Symbol | Meaning | Notes |
|--------|---------|-------|
| $t$ | Valuation time (years) | Typically $t = 0$ in examples |
| $T$ | Generic maturity date (years from now) | |
| $T_1$ | Start date of underlying accrual period | |
| $T_2$ | End date of accrual period | $T_2 = T_1 + \tau$ |
| $\tau$ | Accrual year-fraction for $[T_1, T_2]$ | $\tau = \text{DC}(T_1, T_2)$; often $\tau = 0.25$ for 3M |
| $P(t, T)$ | Discount factor at $t$ for maturity $T$ | PV of 1 unit paid at $T$ |
| $L(t; T_1, T_2)$ | Simple forward rate for $[T_1, T_2]$ observed at $t$ | Money-market convention |
| $Q$ | Quoted STIR futures price | In "price points" |
| $r^{\text{fut}}$ | Futures-implied annualized rate (decimal) | |
| $R^{\text{fut}}$ | Futures-implied annualized rate (percent) | $r^{\text{fut}} = R^{\text{fut}}/100$ |
| $c$ | Convexity adjustment (rate units) | Forward = Futures $- c$ |
| $\sigma$ | Annual absolute volatility of short rate (decimal) | e.g., 100 bp = 0.01 |
| $\beta$ | Length of underlying deposit period (years) | e.g., $\beta \approx 0.25$ for 3M |

**Rate conventions:**
- Decimal rate $r = 0.05$ means 5% p.a.
- Percent rate $R = 5\%$ means $r = R/100 = 0.05$
- 1 bp $= 0.01\% = 10^{-4}$ in decimal

**STIR futures quote convention:**
Many STIR futures are quoted so that Price $= 100 -$ rate (with rate in percent) or equivalently $Q = 100(1 - r^{\text{fut}})$ (with $r^{\text{fut}}$ in decimal). This convention is explicitly described for Eurodollar and SOFR futures in the sources.

---

## Core Concepts

### 1) STIR Futures (Short-Term Interest Rate Futures)

**Formal Definition:**
A STIR futures contract is an exchange-traded futures contract whose final settlement is linked to a short-term interest rate (or an average/compound of overnight rates) over a specified short accrual period (often 1M or 3M). The sources give concrete examples:

- **Eurodollar futures**: a futures contract on a 3-month USD LIBOR fixing on a specified notional "bank deposit," cash-settled at maturity
- **3-month SOFR futures**: a futures contract based on a 3-month compounded SOFR measure, also cash-settled
- **Fed funds futures**: designed as a hedge to a 30-day deposit in fed funds, cash-settled using the month's average effective fed funds rate

**Intuition:**
STIR futures let you take a view on, or hedge, short-end interest rate levels with an exchange-traded instrument that is marked-to-market daily (margining + daily settlement).

**Trading/Risk/Portfolio Practice:**
- Used to hedge or express views on front-end rate expectations (policy path, funding rates, "front-end curve")
- Frequently used in "bucket hedging" of short-end exposures (you hedge a maturity bucket with the nearest liquid futures contracts). Hull explicitly notes Eurodollar futures are used for short-term exposures and describes a bucket-hedging use case.

---

### 2) Quoting Convention: Futures Price ↔ Implied Futures Rate

**Formal Definition:**
Many STIR futures are quoted so that the futures price is 100 minus an annualized rate (with the rate often expressed in percent). For Eurodollar futures, Hull states final settlement is $100 - R$ where $R$ is the relevant 3M rate (in percent), and that the contract is designed so that a 1 bp move is worth a fixed dollar amount. Interest-rate-modeling notes the same convention for Eurodollar futures: if the futures rate is 5%, the quoted futures price is 95. Tuckman gives an analogous "100 minus ... average rate" rule for fed funds futures.

**Intuition:**
This quote convention turns "higher rate" into "lower price," similar to bond prices.

**Trading/Risk/Portfolio Practice:**
- Front-end traders often think directly in rates, but the exchange prints prices. You constantly convert back and forth.
- Since "price" changes are proportional to "rate" changes, DV01 computations are straightforward once contract design is known.

---

### 3) Forward Rates, FRAs, and Discount Factors (The Forward Benchmark)

**Formal Definition:**
A forward rate over $[T_1, T_2]$ is the rate implied by discount factors so that a forward lending/borrowing contract is fairly priced. Interest-rate-modeling gives a standard FRA valuation and shows the fair FRA rate $k$ satisfies:

$$k = \frac{P(t, T) - P(t, T + \tau)}{\tau \, P(t, T + \tau)}$$

where $T$ is the FRA start date and $T + \tau$ its end date.

**Intuition:**
If you know the discount curve $P(t, \cdot)$, you can "read off" the market-implied forward deposit rate between two dates.

**Trading/Risk/Portfolio Practice:**
A key question in practice: "Does a futures strip give me forwards?"
Answer: not exactly—you often need a convexity adjustment.

---

### 4) Futures vs Forwards: Why They Can Differ

**Formal Definition:**
A forward contract typically settles once at maturity; a futures contract is marked-to-market daily with intermediate cashflows (margin gains/losses). Hull states that when interest rates are constant, forward and futures prices coincide; when interest rates vary unpredictably, the difference depends on correlation between the underlying and interest rates (positive correlation tends to make futures > forwards, negative correlation the opposite).

**Intuition (Mark-to-Market Mechanism):**
Daily settlement means gains/losses are realized earlier and can be reinvested/financed at then-prevailing short rates. If the contract's underlying is correlated with rates, that reinvestment/financing effect changes the fair futures price (relative to a forward).

**Trading/Risk/Portfolio Practice:**
- For interest rate futures, the underlying is itself tied to rates; Hull notes they are an "exception" where futures and forward rates can differ even when thinking in rate space.
- Practically: if you infer forwards from STIR futures without adjustment, you can bias curve construction and hedges.

---

### 5) Convexity Adjustment (The Key Bridge)

**Formal Definition:**
Hull defines the convexity adjustment $c$ by:

$$\boxed{\text{Forward rate} = \text{Futures rate} - c}$$

and states $c$ is usually positive and increases with both the contract's life and interest-rate volatility. Earlier in the same discussion, Hull also explains (for Eurodollar futures) why the forward rate tends to be lower than the futures rate, and labels their difference a convexity adjustment.

**Intuition:**
The "convexity adjustment" corrects for:
1. Daily settlement (mark-to-market) + stochastic rates, and
2. The fact that the rate underlying (or its price representation) is correlated with the discounting/reinvestment environment.

**Trading/Risk/Portfolio Practice:**
When building a forward curve from futures, you often:
1. Convert prices → futures-implied rates, then
2. Subtract an adjustment $c$ to estimate forwards.

For longer-dated STIR futures (several years out), the adjustment can reach multiple bp and matter for pricing/hedging (Tuckman provides explicit magnitudes in an example).

---

## Math and Derivations

### 2.1 Quote ↔ Futures-Implied Rate

For Eurodollar-style STIR futures, sources state the settlement/quote convention is "100 minus the rate."

Let:
- $Q$ = quoted futures price (e.g., 95.25)
- $R^{\text{fut}}$ = futures-implied rate in percent (e.g., 4.75%)
- $r^{\text{fut}} = R^{\text{fut}}/100$ = futures-implied rate in decimal

Then:

$$\boxed{Q = 100 - R^{\text{fut}} \iff R^{\text{fut}} = 100 - Q}$$

and

$$r^{\text{fut}} = \frac{100 - Q}{100} = 1 - \frac{Q}{100}$$

**Unit check:**
If $Q$ is in "price points," then $R^{\text{fut}}$ is in percent. A 1 bp move in rate = $0.01\%$ = 0.01 price points under this convention.

---

### 2.2 Forward (FRA) Rate from Discount Factors

Interest-rate-modeling provides the no-arbitrage relationship for an FRA and the implied forward rate:

$$\boxed{k = \frac{P(t, T) - P(t, T + \tau)}{\tau \, P(t, T + \tau)}}$$

**Derivation (standard replication intuition):**

1. Investing 1 unit at $t$ into a zero maturing at $T$ costs $P(t, T)$.
2. Rolling that investment to $T + \tau$ at the forward rate $k$ gives a payoff at $T + \tau$ of:
   $$\text{Payoff at } T + \tau = 1 + \tau k$$
3. No-arbitrage equates the value of receiving 1 at $T$ and then accruing at $k$ to receiving 1 at $T + \tau$:
   $$P(t, T) = (1 + \tau k) \, P(t, T + \tau)$$
4. Solve:
   $$k = \frac{P(t, T)}{P(t, T + \tau)} \cdot \frac{1}{\tau} - \frac{1}{\tau} = \frac{P(t, T) - P(t, T + \tau)}{\tau \, P(t, T + \tau)}$$

**Unit check:**
$P$ is dimensionless, $\tau$ is in years ⇒ $k$ has units "per year" (annualized simple rate).

---

### 2.3 Futures vs Forwards: Deterministic-Rate Limit

Hull states that when interest rates are constant, forward and futures prices are the same; differences arise when rates are uncertain and correlated with the underlying.

**Implication:**
If short rates are deterministic, the mark-to-market reinvestment effect does not create an additional pricing wedge. Therefore, in the deterministic-rate limit, the convexity adjustment should satisfy:

$$c \to 0, \quad \text{(futures-implied rate)} \to \text{(forward rate)}$$

This sanity check becomes a practical verification test (see Section 5).

---

### 2.4 Convexity Adjustment: Definition and Sign Intuition

Hull defines the convexity adjustment $c$ by:

$$L(0; T_1, T_2) = r^{\text{fut}}(0; T_1, T_2) - c$$

with $c$ usually positive.

Tuckman provides an inequality (in a particular setup) indicating the futures rate exceeds the forward rate:

$$r^{\text{fut}} > r^{\text{fwd}}$$

So the "common" sign convention in these sources is:
- $c > 0$
- Futures-implied rate $>$ forward rate
- Forward rate = futures rate $- c$

**Important scope note:** This sign statement is model- and convention-dependent; the sources present it as typical in the settings discussed (Eurodollar futures and related modeling assumptions).

---

### 2.5 A Sourced Model-Based Approximation for the Futures–Forward Difference (Tuckman)

Tuckman derives (under a **normal model with no mean reversion, continuous compounding, and continuous mark-to-market payments**) an approximation for the **difference between futures and forward rates** (split into "pure futures-forward effect" and "convexity effect"):

**Pure futures-forward effect:**
$$r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 t^2}{2}$$

**Convexity effect** (when the forward is on a "zero" of maturity $\beta$):
$$r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 \beta t}{2}$$

**Combining:**

$$\boxed{r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}}$$

For a 90-day (3M) deposit underlying, Tuckman notes $\beta \approx 0.25$, giving the second term $\approx \sigma^2 t / 8$.

**Assumptions (stated clearly):**
- "Normal" here means rates move with absolute volatility $\sigma$ (not lognormal)
- No mean reversion (so long-horizon variance grows like $t$)
- Continuous-time approximation to daily settlement

**Unit check:**
$\sigma$ is in decimal rate per $\sqrt{\text{year}}$, so $\sigma^2$ is "rate$^2$ per year". Multiply by $t^2$ or $\beta t$ (years$^2$) ⇒ result is "rate$^2 \cdot$year", but in this model setup it is used as a rate increment; Tuckman's examples convert to bp by multiplying by 10,000.

**Sanity checks:**
- If $\sigma = 0$ ⇒ adjustment 0
- If $t$ is small ⇒ adjustment small
- Adjustment grows with volatility and time to expiry, consistent with Hull's qualitative statement

---

## Measurement & Risk

### 3.1 What STIR Futures Reference

**Generic:** STIR futures reference a short-term interest rate over a future accrual period or an average/compound of overnight rates over a month/quarter.

**Sourced examples:**
- **Eurodollar futures** reference a 3M USD LIBOR fixing (Hull)
- **3M SOFR futures** reference a 3M compounded SOFR measure (Hull)
- **Fed funds futures** reference the average effective fed funds rate over a month (Tuckman)

---

### 3.2 Quoting Convention: Price ↔ Implied Rate

For Eurodollar-style and SOFR-style examples in these sources:

$$Q = 100 - R^{\text{fut}} \quad \text{(with } R^{\text{fut}} \text{ in percent)}$$

For fed funds futures (as described by Tuckman):

$$Q^{\text{FF}} = 100 - 100 \times \bar{r}^{\text{eff}}$$

where $\bar{r}^{\text{eff}}$ is the monthly average effective fed funds rate in decimal.

If you need exact exchange tick sizes, settlement calendars, or index definitions beyond what is cited above: **I'm not sure.** We would need the exact contract specification from the exchange rulebook for your chosen contract.

---

### 3.3 Futures vs Forwards: Why a Futures-Implied Rate Can Differ from a Forward Rate

**Mechanism:** Futures are marked-to-market daily; forwards settle once. Hull explains that with stochastic rates, daily settlement creates a difference if the underlying is correlated with interest rates.

**Interest rate futures are "special":** The underlying is itself a short-term rate (or a price mechanically linked to it), making correlation effects material. Hull explicitly flags interest rate futures as an exception where the rate implied by futures differs from the forward rate.

**Extra Eurodollar-specific effect in Hull:** Eurodollar futures are settled at the beginning of the 3-month period, while the forward/"futures interest rate" corresponds to the start of that period; Hull notes this tends to make the forward rate lower than the futures rate (positive convexity adjustment).

---

### 3.4 Convexity Adjustment: Definition, Intuition, and Model-Based Approximation

**Definition:**
- Hull: $\text{Forward} = \text{Futures} - c$
- Equivalently: $c = r^{\text{fut}} - r^{\text{fwd}}$ (positive under typical conditions described)

**What it corrects:**
The wedge created by daily settlement interacting with stochastic interest rates and correlation.

**Sign intuition (do not overclaim):**
- Hull states $c$ is usually positive, implying forward rates are usually below futures rates
- Tuckman's inequality $r^{\text{fut}} > r^{\text{fwd}}$ supports the same direction in his model setup

**Sourced model-based approximation:**
Under the specific assumptions in Tuckman (normal model, no mean reversion, continuous mark-to-market), the difference is approximated by:

$$r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$$

This provides a way to compute order-of-magnitude convexity adjustments in bp (Examples F–H).

---

### 3.5 Risk/Hedging Implications

**PV01/DV01 of a STIR futures position:**

Eurodollar and 3M SOFR futures are designed so that a 1 bp move corresponds to a fixed dollar amount per contract; Hull states it is **\$25 per contract** for these contracts. Interest-rate-modeling derives the same \$25/bp figure from a \$1,000,000 notional and a 3M accrual factor 0.25:

$$1{,}000{,}000 \times 0.25 \times 0.0001 = 25$$

**Fed funds futures:** Tuckman shows the contract is designed around a \$5,000,000 30-day deposit, giving **\$41.67 per bp** for a 30-day month:

$$5{,}000{,}000 \times \frac{0.0001 \times 30}{360} = 41.67$$

**Hedging short-end curve buckets:**

If you have an exposure whose PV changes by $\text{DV01}_{\text{target}}$ dollars per bp, and a futures contract with $\text{DV01}_{\text{fut}}$ dollars per bp, then:

$$\boxed{\# \text{contracts} \approx -\frac{\text{DV01}_{\text{target}}}{\text{DV01}_{\text{fut}}}}$$

(sign depends on whether you want gains when rates rise or fall).

Hull highlights using Eurodollar futures as a convenient short-end hedge instrument (bucket hedging).

**What breaks if you ignore convexity adjustment:**
You will tend to treat futures-implied rates as forwards, biasing forward extraction. Hull explicitly frames the difference as a convexity adjustment and notes it grows with volatility and maturity. Downstream consequences: mis-specified forward curve ⇒ mis-priced FRAs/swaps/caps (even if slightly) and slightly biased hedges (Example J).

---

## Worked Examples

**General example conventions (unless stated otherwise):**
- Rate quote convention: $Q = 100 - R^{\text{fut}}$ with $R^{\text{fut}}$ in percent (Eurodollar/SOFR style)
- Convert percent to decimal via $r = R/100$
- For 3M accrual: $\tau = 0.25$ (approx Actual/360 quarter)
- If we use per-contract DV01 for Eurodollar/SOFR futures, we use \$25/bp as sourced

---

### Example A (Quote to Rate)

**Task:** Convert a futures price quote into an implied futures rate.

**Given:**
- Quoted futures price: $Q = 95.25$

**Step 1: Implied futures rate in percent**
$$R^{\text{fut}} = 100 - Q = 100 - 95.25 = 4.75\%$$

**Step 2: Convert to decimal**
$$r^{\text{fut}} = \frac{R^{\text{fut}}}{100} = \frac{4.75}{100} = 0.0475$$

**Unit check:**
1 bp $= 0.01\%$. If $Q$ moves by 0.01 (e.g., 95.25 → 95.24), then $R^{\text{fut}}$ moves by 0.01% = 1 bp.

*(Quote convention sourced for Eurodollar/SOFR-style STIR futures.)*

---

### Example B (Rate to Price)

**Task:** Given an implied futures rate, compute the futures price quote.

**Given:**
- Futures-implied rate: $R^{\text{fut}} = 3.875\%$

**Compute quote:**
$$Q = 100 - R^{\text{fut}} = 100 - 3.875 = 96.125$$

**Unit check:**
If the rate increases by 1 bp (0.01%), the quote decreases by 0.01.

---

### Example C (Forward Rate from Discount Factors)

**Task:** Given discount factors $P(0, T_1)$, $P(0, T_2)$, compute the simple forward rate for $[T_1, T_2]$.

**Given:**
- $P(0, T_1) = 0.9780$
- $P(0, T_2) = 0.9655$
- $\tau = 0.25$

**Formula (simple forward):**
$$L(0; T_1, T_2) = \frac{P(0, T_1) - P(0, T_2)}{\tau \, P(0, T_2)}$$

**Step-by-step:**

1. Numerator:
   $$P(0, T_1) - P(0, T_2) = 0.9780 - 0.9655 = 0.0125$$

2. Denominator:
   $$\tau \, P(0, T_2) = 0.25 \times 0.9655 = 0.241375$$

3. Forward:
   $$L = \frac{0.0125}{0.241375} \approx 0.051785 \;(= 5.1785\%)$$

**Interpretation:**
This is the annualized simple rate for a forward "deposit" over $[T_1, T_2]$ implied by the discount factors.

---

### Example D (Futures-Implied vs Forward, No Convexity)

**Task:** In a deterministic-rates world, show futures-implied rate = forward rate.

**Assumption (deterministic rates):**
Convexity adjustment $c = 0$ (deterministic-rate limit).

**Using Example C:**
- Forward rate: $L = 5.1785\%$

**Then futures-implied rate:**
$$R^{\text{fut}} = L = 5.1785\%$$

**Implied futures quote:**
$$Q = 100 - 5.1785 = 94.8215$$

**Reconciliation with A–C:**
- A–B convert between $Q$ and $R^{\text{fut}}$
- C gives the forward $L$
- In deterministic rates, $R^{\text{fut}} = L$, so the futures quote is consistent with the curve-implied forward

---

### Example E (Convexity Intuition with a Toy Stochastic Scenario — Illustrative)

**Goal:** Show (mechanistically) that expected futures-implied rate can differ from forward rate due to settlement effects.

**Important label:**
I'm not sure this toy maps exactly to any specific STIR contract's measure/settlement model; it is an illustrative mechanism consistent with Hull's qualitative statement that correlation + stochastic rates create futures/forward differences.

**Toy setup:**
- Two states (probability 0.5 each) for a one-year horizon split into two half-years (to mimic intermediate settlement)
- Let the "underlying quoted price" $Q$ be negatively correlated with the short rate $r$ (as in bond-price intuition and in STIR quote $Q = 100 - \text{rate}$)
- At the intermediate date, the futures is marked to market; the forward is not

**State values (at the intermediate date):**
- State H (high rates): $r = 8\%$, $Q = 95$
- State L (low rates): $r = 2\%$, $Q = 97$

**Step 1: Compute the fair futures price $F_0$ (intermediate cashflow)**

In a simplified two-date marked-to-market toy, the fair futures price solves:

$$F_0 = \frac{E\left[\frac{Q}{1+r}\right]}{E\left[\frac{1}{1+r}\right]}$$

Compute components:

- State H:
  $$\frac{Q}{1+r} = \frac{95}{1.08} = 87.96296, \quad \frac{1}{1+r} = 0.9259259$$

- State L:
  $$\frac{Q}{1+r} = \frac{97}{1.02} = 95.09804, \quad \frac{1}{1+r} = 0.9803922$$

Expectations:
$$E\left[\frac{Q}{1+r}\right] = 0.5(87.96296 + 95.09804) = 91.53050$$
$$E\left[\frac{1}{1+r}\right] = 0.5(0.9259259 + 0.9803922) = 0.9531590$$

So:
$$F_0 = \frac{91.53050}{0.9531590} \approx 96.0286$$

**Step 2: Compute the fair forward price $K$ (single settlement later)**

A one-shot forward (settling later) would weight outcomes with a "longer" discounting exposure; in a toy with two periods at the same state rate:

$$K = \frac{E\left[\frac{Q}{(1+r)^2}\right]}{E\left[\frac{1}{(1+r)^2}\right]}$$

Compute components:

- State H:
  $$\frac{1}{(1.08)^2} = 0.8573388, \quad \frac{95}{(1.08)^2} = 81.44719$$

- State L:
  $$\frac{1}{(1.02)^2} = 0.9611688, \quad \frac{97}{(1.02)^2} = 93.23337$$

Expectations:
$$E\left[\frac{Q}{(1+r)^2}\right] = 0.5(81.44719 + 93.23337) = 87.34028$$
$$E\left[\frac{1}{(1+r)^2}\right] = 0.5(0.8573388 + 0.9611688) = 0.9092538$$

So:
$$K = \frac{87.34028}{0.9092538} \approx 96.0571$$

**Conclusion:**

Forward price $K \approx 96.0571$ is greater than futures price $F_0 \approx 96.0286$ when the underlying price is negatively correlated with rates, consistent with Hull's correlation sign rule.

Translating to "implied rates" via $R = 100 - Q$:
$$R^{\text{fut}} \approx 100 - 96.0286 = 3.9714\%$$
$$R^{\text{fwd}} \approx 100 - 96.0571 = 3.9429\%$$

So $R^{\text{fut}} - R^{\text{fwd}} \approx 0.0285\% \approx 2.85$ bp, illustrating a positive convexity adjustment direction (futures rate above forward).

---

### Example F (Model-Based Convexity Adjustment — Sourced Tuckman Approximation)

**Task:** Use a sourced formula to compute a convexity adjustment.

Tuckman provides (under stated assumptions) a futures–forward difference approximation and shows a numerical example for EDZ6 (time to expiration $t = 5.05$ years, $\sigma = 100$ bp).

**Given (toy but aligned with Tuckman's example structure):**
- Annual absolute rate volatility: $\sigma = 100$ bp $= 0.01$ (decimal)
- Time to futures expiration: $t = 5.05$ years
- Underlying period length: $\beta = 0.25$ years (3M deposit proxy)

**Formula:**
$$r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$$

**Step-by-step:**

- $\sigma^2 = 0.01^2 = 0.0001$
- $t^2 = 5.05^2 = 25.5025$

Term 1:
$$\frac{\sigma^2 t^2}{2} = \frac{0.0001 \times 25.5025}{2} = \frac{0.00255025}{2} = 0.001275125$$

Term 2:
$$\frac{\sigma^2 \beta t}{2} = \frac{0.0001 \times 0.25 \times 5.05}{2} = \frac{0.0001 \times 1.2625}{2} = \frac{0.00012625}{2} = 0.000063125$$

Sum:
$$r^{\text{fut}} - r^{\text{fwd}} \approx 0.00133825$$

Convert to bp:
$$0.00133825 \times 10{,}000 = 13.3825 \text{ bp} \approx 13.4 \text{ bp}$$

**Direction:**
Futures rate exceeds forward rate by $\approx 13.4$ bp.

Convexity adjustment (Hull style) $c \approx 13.4$ bp so:
$$r^{\text{fwd}} \approx r^{\text{fut}} - 13.4 \text{ bp}$$

---

### Example G (Sensitivity to Volatility)

**Task:** Show how the convexity adjustment scales with volatility.

Using the same Tuckman approximation, the adjustment scales with $\sigma^2$.

Keep: $t = 5.05$, $\beta = 0.25$.

We already computed for $\sigma = 0.01$ (100 bp): $c \approx 13.38$ bp.

Now vary $\sigma$:

**$\sigma = 50$ bp = 0.005:**
- Scaling factor: $(0.005/0.01)^2 = 0.25$
- Adjustment: $c \approx 13.38 \times 0.25 = 3.345$ bp

**$\sigma = 150$ bp = 0.015:**
- Scaling factor: $(0.015/0.01)^2 = 2.25$
- Adjustment: $c \approx 13.38 \times 2.25 = 30.105$ bp

**Intuition:**
Higher rate volatility increases the dispersion of rates and thus magnifies the mark-to-market/reinvestment effect; Hull qualitatively states the adjustment increases with volatility and life.

---

### Example H (Extracting a Forward Curve from a Strip of Futures)

**Task:** Given 5 futures prices, convert to futures rates, then:
1. Treat as forwards (naïve)
2. Apply convexity adjustments (Tuckman approximation)
3. Compare the implied forward curve

**Given:**

5 quarterly STIR futures quotes (Eurodollar-style $Q = 100 - R$):
- Contract 1 (expires $t = 4.00$y): $Q_1 = 95.00$
- Contract 2 ($t = 4.25$y): $Q_2 = 94.90$
- Contract 3 ($t = 4.50$y): $Q_3 = 94.80$
- Contract 4 ($t = 4.75$y): $Q_4 = 94.70$
- Contract 5 ($t = 5.00$y): $Q_5 = 94.60$

Model inputs (for convexity adjustment):
- $\sigma = 100$ bp = 0.01
- $\beta = 0.25$ (3M)

Optional bootstrap seed discount factor:
- $P(0, 4.00) = 0.8200$ (given as a starting point)

**Step 1: Convert prices to futures rates**

Using $R^{\text{fut}} = 100 - Q$:
- $R_1^{\text{fut}} = 5.00\%$
- $R_2^{\text{fut}} = 5.10\%$
- $R_3^{\text{fut}} = 5.20\%$
- $R_4^{\text{fut}} = 5.30\%$
- $R_5^{\text{fut}} = 5.40\%$

Naïve forward curve: $R_i^{\text{fwd, naive}} = R_i^{\text{fut}}$

**Step 2: Compute convexity adjustments $c_i$**

Tuckman approximation:
$$c_i \approx \left(\frac{\sigma^2 t_i^2}{2} + \frac{\sigma^2 \beta t_i}{2}\right) \times 10{,}000 \text{ bp}$$

With $\sigma = 0.01 \Rightarrow \sigma^2 = 0.0001$ and multiplying by 10,000, the bp adjustment simplifies numerically to:
$$c_i(\text{bp}) = \frac{t_i^2}{2} + \frac{\beta t_i}{2} \quad \text{(only for } \sigma = 0.01\text{)}$$

Compute each:

**$t_1 = 4.00$:**
$$c_1 = \frac{16}{2} + \frac{0.25 \times 4}{2} = 8 + 0.5 = 8.5 \text{ bp}$$

**$t_2 = 4.25$:**
$$c_2 = \frac{4.25^2}{2} + \frac{0.25 \times 4.25}{2} = \frac{18.0625}{2} + \frac{1.0625}{2} = 9.03125 + 0.53125 = 9.5625 \text{ bp}$$

**$t_3 = 4.50$:**
$$c_3 = \frac{20.25}{2} + \frac{1.125}{2} = 10.125 + 0.5625 = 10.6875 \text{ bp}$$

**$t_4 = 4.75$:**
$$c_4 = \frac{22.5625}{2} + \frac{1.1875}{2} = 11.28125 + 0.59375 = 11.875 \text{ bp}$$

**$t_5 = 5.00$:**
$$c_5 = \frac{25}{2} + \frac{1.25}{2} = 12.5 + 0.625 = 13.125 \text{ bp}$$

**Step 3: Adjust futures rates to get forward rates**

Hull convention: forward = futures $- c$.

Convert bp to percent by dividing by 100:
- 8.5 bp = 0.085%
- 9.5625 bp = 0.095625%
- 10.6875 bp = 0.106875%
- 11.875 bp = 0.11875%
- 13.125 bp = 0.13125%

So:
- $R_1^{\text{fwd}} \approx 5.00\% - 0.085\% = 4.915\%$
- $R_2^{\text{fwd}} \approx 5.10\% - 0.095625\% = 5.004375\%$
- $R_3^{\text{fwd}} \approx 5.20\% - 0.106875\% = 5.093125\%$
- $R_4^{\text{fwd}} \approx 5.30\% - 0.11875\% = 5.18125\%$
- $R_5^{\text{fwd}} \approx 5.40\% - 0.13125\% = 5.26875\%$

**Comparison (forward curve points):**

| Contract | Naïve Forward | Adjusted Forward | Difference |
|----------|---------------|------------------|------------|
| 1 (4.00y) | 5.00% | 4.915% | 8.5 bp |
| 2 (4.25y) | 5.10% | 5.004% | 9.6 bp |
| 3 (4.50y) | 5.20% | 5.093% | 10.7 bp |
| 4 (4.75y) | 5.30% | 5.181% | 11.9 bp |
| 5 (5.00y) | 5.40% | 5.269% | 13.1 bp |

**Optional: show discount factor impact (simple compounding over each 3M period)**

Let $\tau = 0.25$ and $P_{i+1} = P_i / (1 + \tau L_i)$ where $L_i$ is the decimal forward.

Start $P(0, 4.00) = 0.8200$.

**Naïve first step** ($L_1 = 0.0500$):
$$1 + \tau L_1 = 1 + 0.25 \times 0.05 = 1.0125$$
$$P(0, 4.25) = \frac{0.8200}{1.0125} \approx 0.8099$$

**Adjusted first step** ($L_1 = 0.04915$):
$$1 + \tau L_1 = 1 + 0.25 \times 0.04915 = 1.0122875$$
$$P(0, 4.25) = \frac{0.8200}{1.0122875} \approx 0.8100$$

**Interpretation:** Applying convexity adjustment reduces forwards slightly, raising discount factors slightly.

---

### Example I (STIR Futures PV01 / BP Value — Generic)

**Task:** Compute PV impact of a 1 bp change in the underlying futures-implied rate.

**Case 1: Eurodollar / 3M SOFR futures per-contract DV01 (sourced)**

Hull states Eurodollar and 3M SOFR futures are designed so that a 1 bp move is worth \$25 per contract. Interest-rate-modeling derives:

$$1{,}000{,}000 \times 0.25 \times 0.0001 = 25$$

So:
$$\boxed{\text{DV01}_{\text{fut}} = \$25/\text{bp per contract}}$$

**Case 2: Generic "per 1 unit notional" DV01 (no contract multiplier)**

If the underlying is a forward simple rate over $\tau$, then a 1 bp change changes interest by $\tau \times 0.0001$ per unit notional. So:

DV01 per unit notional = $\tau \times 0.0001$

Example with $\tau = 0.25$:
$$0.25 \times 0.0001 = 0.000025$$
(currency units per 1 unit notional per bp)

**Case 3: Fed funds futures per-contract bp value (sourced)**

Tuckman: contract corresponds to \$5,000,000 30-day deposit; 1 bp changes interest by:
$$5{,}000{,}000 \times \frac{0.0001 \times 30}{360} = 41.67$$

So:
$$\text{DV01}_{\text{FF}} = \$41.67/\text{bp per contract} \text{ for a 30-day month}$$

---

### Example J (Hedge Ratio to a Short-End Exposure)

**Task:** Given a target short-end DV01 bucket, compute number of futures to hedge it. Then show how convexity adjustment can change the hedge slightly via the discount factor used in DV01.

**Exposure:**
- You will pay a floating coupon proportional to a 3M forward rate over $[4.00, 4.25]$ on notional $N = 500{,}000{,}000$ (500mm)
- Cashflow at $T_2 = 4.25$ is approximately $N \tau L$
- DV01 w.r.t. the forward rate (per 1 bp change in $L$) is approximately:
  $$\text{DV01}_{\text{target}} \approx N \tau \, P(0, 4.25) \times 0.0001$$

**Use discount factors from Example H:**
- Naïve: $P^{\text{naive}}(0, 4.25) \approx 0.8099$
- Adjusted: $P^{\text{adj}}(0, 4.25) \approx 0.8100$

(These arose from using futures-as-forwards vs convexity-adjusted forwards.)

**Step 1: Compute DV01 (naïve)**

$N \tau = 500{,}000{,}000 \times 0.25 = 125{,}000{,}000$

Multiply by 1 bp:
$$125{,}000{,}000 \times 0.0001 = 12{,}500$$

Multiply by discount factor:
$$\text{DV01}_{\text{target, naive}} \approx 12{,}500 \times 0.8099 = 10{,}123.75$$

**Step 2: Compute DV01 (convexity-adjusted curve)**

$$\text{DV01}_{\text{target, adj}} \approx 12{,}500 \times 0.8100 = 10{,}125.00$$

**Step 3: Hedge with Eurodollar/SOFR futures**

Use $\text{DV01}_{\text{fut}} = \$25/\text{bp per contract}$ (sourced).

Hedge ratio:
$$\# \text{contracts} \approx -\frac{\text{DV01}_{\text{target}}}{25}$$

Naïve:
$$\#_{\text{naive}} \approx -\frac{10{,}123.75}{25} = -404.95 \approx -405 \text{ contracts}$$

Adjusted:
$$\#_{\text{adj}} \approx -\frac{10{,}125.00}{25} = -405.00 \text{ contracts}$$

**Interpretation:**
The hedge changes only slightly here because convexity mainly nudges the discount factor modestly in this toy. In real curve construction, convexity adjustments affect the forward curve across many maturities; compounded across instruments, ignoring them can bias valuations and hedges more meaningfully (especially farther out).

---

## Practical Notes

### 5.1 Common Pitfalls

**Confusing futures-implied rates with forward rates:**
Hull explicitly distinguishes them and defines a convexity adjustment $c$ to translate between the two.

**Forgetting that daily settlement is the key mechanism:**
Futures are marked to market daily; the effect interacts with stochastic rates and correlation.

**Mixing day-count/compounding conventions:**
- Eurodollar and SOFR futures conventions differ (e.g., Eurodollar settled at the beginning of the 3M period; Hull notes 3M SOFR futures are settled at the end of the period)
- Forward-rate extraction must be consistent with the curve's compounding basis

**Assuming contract specs without sourcing:**
Tick size, exact settlement calendar, and exact index definitions must be taken from the exchange rulebook. If not in the sources: **I'm not sure.**

**Using the wrong "bp" definition:**
For "100 − rate" quotes, 1 bp in rate corresponds to 0.01 in price points; DV01 depends on contract design.

---

### 5.2 Implementation Pitfalls (Curve Building from Futures)

**Stitching adjacent contracts:**
Ensure the accrual periods are consecutive with no gaps/overlaps; otherwise you must interpolate or add instruments.

**Forward extraction vs discount curve consistency:**
Interest-rate-modeling emphasizes that forwards relate to discount factors via no-arbitrage. If you treat futures rates as forwards, you implicitly define a curve; convexity adjustments alter that implied curve.

**Applying small convexity corrections across many maturities:**
Numerically stable approach: compute $c(t)$ smoothly as a function of expiry and apply consistently, rather than "piecewise noisy" corrections.

---

### 5.3 Verification Tests (Desk-Relevant)

**Deterministic-rate limit:**
Set $\sigma = 0$ in the Tuckman approximation ⇒ $c = 0$.

**Reasonableness with maturity and volatility:**
- Hull: $c$ increases with contract life and volatility
- Tuckman: difference grows with $t$ and $\sigma^2$

**Sign sanity:**
Hull and Tuckman both present the "typical" sign: forward < futures (positive adjustment).

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **STIR futures** are exchange-traded futures linked to a short-term interest rate (term or overnight-based).

2. Many STIR futures are quoted as $Q = 100 -$ rate (rate in percent), so lower price means higher implied rate.

3. A **forward rate** $L(t; T_1, T_2)$ is implied by discount factors: $L = (P(t, T_1) - P(t, T_2)) / (\tau P(t, T_2))$.

4. **Futures differ from forwards** mainly because futures are marked-to-market daily, changing the timing of cashflows.

5. With **deterministic rates**, futures and forwards coincide (no convexity adjustment).

6. Hull defines the **convexity adjustment** $c$ so that Forward = Futures $- c$, typically $c > 0$.

7. Tuckman provides a **simple-model approximation** where $r^{\text{fut}} - r^{\text{fwd}}$ grows with $\sigma^2$ and time to expiry.

8. For **Eurodollar/3M SOFR futures**, sources state a 1 bp move corresponds to \$25 per contract (contract design).

9. STIR futures are widely used for **short-end hedging** (bucket hedges), but you must map futures → forwards carefully.

10. **Ignoring convexity adjustment** can bias forward extraction and, downstream, valuation/hedges—especially for longer-dated STIR strips.

---

### Cheat Sheet of Formulas

**Quote ↔ rate:**
$$R^{\text{fut}}(\%) = 100 - Q$$
$$r^{\text{fut}} = 1 - Q/100$$

**Forward from discount factors:**
$$L(t; T, T+\tau) = \frac{P(t, T) - P(t, T+\tau)}{\tau \, P(t, T+\tau)}$$

**Convexity adjustment definition (Hull):**
$$r^{\text{fwd}} = r^{\text{fut}} - c$$

**Tuckman approximation (specific assumptions):**
$$r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$$

**Eurodollar/SOFR per-contract DV01 (sourced):**
$$\text{DV01} \approx \$25/\text{bp}$$

**Fed funds futures per-contract DV01 (30-day month):**
$$\text{DV01} \approx \$41.67/\text{bp}$$

**Hedge ratio:**
$$\# \text{contracts} \approx -\frac{\text{DV01}_{\text{target}}}{\text{DV01}_{\text{fut}}}$$

---

### Flashcards (30 Q/A)

1. **Q:** What does "STIR" stand for?
   **A:** Short-Term Interest Rate (futures).

2. **Q:** Give one example of a term-rate STIR futures contract from the sources.
   **A:** Eurodollar futures (on 3M USD LIBOR fixing).

3. **Q:** Give one example of an overnight-based STIR futures contract from the sources.
   **A:** Fed funds futures (based on average effective fed funds rate over a month).

4. **Q:** What is the common Eurodollar/SOFR quote convention in the sources?
   **A:** $Q = 100 - R^{\text{fut}}$ (rate in percent).

5. **Q:** If a Eurodollar futures price is 95.25, what rate does it imply?
   **A:** 4.75% (since 100 − 95.25 = 4.75).

6. **Q:** What is daily settlement / marking-to-market?
   **A:** Futures gains/losses are settled daily via margin, creating intermediate cashflows.

7. **Q:** In Hull's discussion, when are forward and futures prices the same?
   **A:** When interest rates are constant (deterministic).

8. **Q:** What is the key mechanism that makes futures differ from forwards?
   **A:** Daily mark-to-market cashflows + stochastic rates + correlation.

9. **Q:** Define the convexity adjustment $c$ (Hull convention).
   **A:** Forward rate = futures rate $- c$.

10. **Q:** Is $c$ usually positive or negative in Hull's description?
    **A:** Usually positive.

11. **Q:** What does $c > 0$ imply about futures vs forward rates?
    **A:** Futures-implied rates are above forward rates.

12. **Q:** Provide the forward-rate-from-discount-factors formula.
    **A:** $L = (P(t, T) - P(t, T+\tau)) / (\tau \, P(t, T+\tau))$.

13. **Q:** What is DV01?
    **A:** Dollar value of a 1 bp increase in rates (PV change per bp).

14. **Q:** What is the per-contract bp value of Eurodollar/3M SOFR futures in Hull?
    **A:** \$25 per bp.

15. **Q:** What is the per-contract bp value of fed funds futures in Tuckman's example?
    **A:** \$41.67 per bp for a 30-day month.

16. **Q:** Why is ignoring convexity adjustment dangerous in curve building?
    **A:** Futures rates won't equal forwards, so implied curve can be biased.

17. **Q:** Under what conditions does convexity adjustment grow larger?
    **A:** With higher volatility and longer time to expiry.

18. **Q:** State Tuckman's "pure futures-forward effect" approximation.
    **A:** $r^{\text{fut}} - r^{\text{fwd}} \approx \sigma^2 t^2 / 2$.

19. **Q:** State Tuckman's "convexity effect" approximation.
    **A:** $r^{\text{fut}} - r^{\text{fwd}} \approx \sigma^2 \beta t / 2$.

20. **Q:** For a 3M period, what is $\beta$ approximately in Tuckman?
    **A:** $\beta \approx 0.25$.

21. **Q:** What is the deterministic-rate sanity check for convexity adjustment?
    **A:** Set $\sigma = 0$ ⇒ $c = 0$.

22. **Q:** In Hull, what happens to the convexity adjustment as volatility increases?
    **A:** It increases.

23. **Q:** What is "bucket hedging" with STIR futures?
    **A:** Hedging exposures in a maturity bucket using the most liquid nearby futures.

24. **Q:** What is the basic futures hedge ratio using DV01?
    **A:** $\# \text{contracts} \approx -\text{DV01}_{\text{target}} / \text{DV01}_{\text{fut}}$.

25. **Q:** What is the implied rate if a SOFR futures price is 99.35 (Hull example style)?
    **A:** 0.65% (since 100 − 99.35 = 0.65).

26. **Q:** What is the main practical reason futures options can be popular (general futures property)?
    **A:** Liquidity and ease of trading compared to some underlying markets.

27. **Q:** What is a forward rate in words?
    **A:** The market-implied rate for lending/borrowing over a future period consistent with discount factors.

28. **Q:** What does it mean that Eurodollar futures settle at the beginning of the 3M period (Hull)?
    **A:** The final settlement aligns with the period start, affecting the futures/forward relationship.

29. **Q:** What does it mean that 3M SOFR futures settle at the end of the 3M period (Hull)?
    **A:** The settlement timing differs from Eurodollars, affecting mapping to forwards.

30. **Q:** What is the convexity adjustment used for in practice?
    **A:** Translating futures-implied rates into forward rates for pricing/curve building.

---

## Mini Problem Set (16 Questions)

1. A STIR future is quoted at 96.40. What is the implied futures rate in percent and decimal?

2. A STIR future implies a rate of 2.875%. What is the quoted price?

3. Given $P(0, 1.0) = 0.9600$, $P(0, 1.25) = 0.9480$, and $\tau = 0.25$, compute the simple forward rate $L(0; 1.0, 1.25)$.

4. Using Hull's definition, explain in one sentence what the convexity adjustment corrects for.

5. Under the Tuckman approximation with $\sigma = 0.01$, $\beta = 0.25$, $t = 3$, compute $r^{\text{fut}} - r^{\text{fwd}}$ in bp.

6. With the same parameters as (5), what happens to the adjustment if $\sigma$ doubles?

7. Suppose your exposure has DV01 = $-\$250{,}000$/bp, and Eurodollar futures DV01 is \$25/bp per contract. How many contracts hedge the exposure?

8. In a deterministic-rate world, what should the convexity adjustment be? Cite the relevant intuition from Hull.

9. Consider a strip of 4 futures contracts expiring at 1y, 2y, 3y, 4y with quoted prices 97.50, 97.00, 96.50, 96.00. Convert to futures rates. Then compute convexity adjustments using $\sigma = 0.01$, $\beta = 0.25$ and obtain forward rates.

10. Using the forward rates from (9), bootstrap discount factors from $P(0, 1) = 0.9700$ forward one quarter at a time for the first two periods (assume $\tau = 0.25$).

11. Explain why the sign of the convexity adjustment depends on correlation and settlement timing (conceptual).

12. For fed funds futures, compute the bp value for a 31-day month if the contract is designed around a \$5,000,000 deposit (adapt Tuckman's 30-day computation).

13. A desk builds a curve by treating STIR futures rates as forwards. List two diagnostics you would run to detect convexity-related bias.

14. Qualitatively, why might convexity adjustment matter more for longer-dated STIR futures?

15. Describe basis risk in using STIR futures to hedge a corporate borrowing rate (one paragraph). *(Hint: see Tuckman discussion on basis risk in fed funds hedging context.)*

16. Explain (at a high level) how a term-structure model could be used to compute the convexity adjustment more precisely than the simple approximation.

---

### Brief Solution Sketches (Questions 1–8 Only)

1. $R^{\text{fut}} = 100 - 96.40 = 3.60\%$. Decimal $r^{\text{fut}} = 0.0360$.

2. $Q = 100 - 2.875 = 97.125$.

3. $L = (0.9600 - 0.9480)/(0.25 \times 0.9480) = 0.0120/0.2370 \approx 0.0506 = 5.06\%$.

4. It corrects the difference between futures-implied rates and forward rates caused by daily settlement + stochastic rates (Hull: forward = futures $- c$).

5. Compute in decimal: $\sigma^2 = 0.0001$.
   - $\sigma^2 t^2 / 2 = 0.0001 \times 9 / 2 = 0.00045$
   - $\sigma^2 \beta t / 2 = 0.0001 \times 0.25 \times 3 / 2 = 0.0000375$
   - Sum $= 0.0004875$. In bp: $0.0004875 \times 10{,}000 = 4.875$ bp.

6. If $\sigma$ doubles, $\sigma^2$ quadruples ⇒ adjustment $\times 4$.

7. $\# \approx -(-250{,}000)/25 = +10{,}000$ contracts (direction depends on which way your exposure moves with rates).

8. Deterministic rates ⇒ forward and futures coincide ⇒ convexity adjustment 0.

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| STIR futures quote convention $Q = 100 - R$ | Hull Ch 6, Tuckman Ch 17, Interest-rate-modeling |
| Eurodollar futures reference 3M USD LIBOR | Hull Ch 6 |
| 3M SOFR futures reference compounded SOFR | Hull Ch 6 |
| Fed funds futures reference monthly average eff. fed funds | Tuckman Ch 17 |
| Forward rate formula from discount factors | Interest-rate-modeling |
| Convexity adjustment definition: Forward = Futures $- c$ | Hull Ch 6 |
| Convexity adjustment is usually positive | Hull Ch 6 |
| Futures = forwards when rates are constant | Hull Ch 6 |
| Eurodollar/SOFR futures DV01 = \$25/bp | Hull Ch 6, Interest-rate-modeling |
| Fed funds futures DV01 = \$41.67/bp (30-day) | Tuckman Ch 17 |
| Tuckman approximation for futures-forward difference | Tuckman Ch 17 |
| Eurodollar futures settle at beginning of 3M period | Hull Ch 6 |
| 3M SOFR futures settle at end of 3M period | Hull Ch 6 |

### (B) Reasoned Inference — Derivation Logic

| Inference | Logic |
|-----------|-------|
| Convexity adjustment grows with $\sigma^2$ and $t$ | Direct from Tuckman approximation formula |
| Deterministic-rate limit implies $c \to 0$ | Follows from Hull's statement about constant rates |
| Ignoring convexity adjustment biases forward curve | Follows from definition: if you use futures as forwards, you mis-specify |
| Hedge ratio formula | Standard DV01 matching logic |

### (C) Speculation — Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Exact exchange tick sizes, settlement calendars | I'm not sure — need exchange rulebook |
| Exact index definitions beyond cited examples | I'm not sure — contract-specific |
| Whether toy Example E maps to any specific contract | Illustrative mechanism only |

---

*Last Updated: January 2026*
