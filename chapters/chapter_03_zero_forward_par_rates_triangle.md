# Chapter 3: Zero Rates, Forward Rates, Par Rates — The Triangle

---

## Introduction

You have a discount factor curve from your risk engine. A colleague has a zero rate curve in Excel. The swap desk quotes par swap rates. The bond trading system reports yields. Are these four different things—or four different ways of describing the same thing?

The answer is both. Discount factors, zero rates, forward rates, and par rates are **equivalent representations** of the term structure of interest rates. They are the four corners of a square (or triangle, if you group yields and par rates together), and once you know any one of them with appropriate conventions, you can derive all the others through pure algebra. Yet professionals routinely misapply these conversions—mixing compounding conventions, confusing par yields with zero yields, or treating forward rates as forecasts when they are actually breakeven rates. These mistakes create P&L breaks, hedging errors, and failed arbitrage trades.

Why does this matter? Because different markets speak different languages:
- **Quants** and pricing engines think in **discount factors** (the primitive).
- **Risk managers** often view **zero rates** (spot rates) to understand term structure shape.
- **Traders** price floating-rate notes and swaps using **forward rates**.
- **The Market** quotes prices in **par rates** (swap rates) or **yields** (bond yields).

If you cannot fluently convert between these representations, you cannot reconcile positions across systems, you cannot price a swap against a bond curve, and you cannot understand why your model disagrees with a trader's quote.

This chapter develops the "Rate Triangle"—the mathematical machinery for converting between these representations. We cover:

1. **Discount factors as the primitive** — the building block from which all rates derive
2. **Zero rates across compounding conventions** — how the same discount factor maps to different rate numbers
3. **Forward rates as breakeven rates** — the no-arbitrage derivation and the trader's interpretation
4. **The spot-forward relationship** — why forward rates are "wigglier" than spot rates
5. **Par rates and the coupon effect** — why par yields differ from zero yields on non-flat curves
6. **The expectations hypothesis** — and why forward rates don't simply forecast future spot rates

Chapter 2 established discount factors and present value. This chapter extends that foundation to the full vocabulary of rates that practitioners use daily. Together, they prepare you for bond pricing in Chapter 5 and curve construction in Chapter 17.

---

## 3.1 The Discount Factor: The Primitive Object

### 3.1.1 Definition and Economic Meaning

Every rate concept derives from one fundamental building block: the discount factor.

Tuckman defines the discount factor $d(t)$ (or $P(0,t)$) as "the value today, or the present value of one unit of currency to be received at the end of that term" (Tuckman Ch 1). Equivalently, it is the price today of a risk-free zero-coupon bond paying 1 unit of currency at maturity $T$.

$$\boxed{P(0,T) = \text{Price today of \$1 paid at time } T}$$

The notation $P(t,T)$ represents the price at time $t$ of a payment at $T$. When $t=0$ (today), we often shorten this to $P(T)$ or $d(T)$.

### 3.1.2 Why Discount Factors Are Fundamental

Discount factors are the "atomic units" of valuation. They tell you directly **how much a future dollar is worth today**, stripping away all conventions about compounding frequencies, day counts, or accrual periods.

The pricing equation for **any** deterministic cashflow stream is simply:

$$PV = \sum_{i} CF_i \times P(0,T_i)$$

Everything else—yields, spreads, durations—is just a layer of interpretation on top of this equation.

### 3.1.3 The No-Arbitrage Requirements

Tuckman (Ch 1) establishes the fundamental requirements that discount factors must satisfy:

1. **Positivity:** $P(0,T) > 0$ for all $T$. If $P(0,T)$ were 0 or negative, you could buy money at $T$ for free (or get paid to take it), leading to infinite arbitrage.
2. **Normalization:** $P(0,0) = 1$. A dollar today is worth a dollar today.
3. **Monotonicity (Typical):** When (appropriately defined) interest rates are non-negative over a range of maturities, $P(0,T)$ decreases with $T$ over that range. If some rates are negative, discount factors can exceed 1 and may increase with maturity over some intervals. The algebra in this chapter only requires $P(0,T) > 0$.

**Note (what curve is “risk‑free”):** In practice, the discount curve used for valuation depends on market conventions and the contract’s discounting/collateral terms. In this chapter, treat $P(0,T)$ as a given input curve; curve construction and collateral discounting are covered later (Ch 17–19 and Ch 33).

---

## 3.2 Zero (Spot) Rates: One Rate for the Whole Horizon

### 3.2.1 The Concept

A zero rate (or spot rate) answers the question: *What single annual interest rate, applied over the entire horizon from 0 to $T$, produces the discount factor $P(0,T)$?*

Tuckman defines the spot rate as "the rate on a spot loan... in which the lender gives money to the borrower at the time of the agreement" (Tuckman Ch 2). Hull similarly defines the $n$-year zero rate as "the rate of interest earned on an investment that starts today and lasts for $n$ years," with **no intermediate payments** (Hull Ch 4).

Crucially, **you cannot quote a zero rate without specifying a compounding convention.** The same discount factor implies different numerical rates depending on how you compound.

### 3.2.2 Continuous Compounding

This is the standard for derivatives pricing code (e.g., Black-Scholes, QuantLib internals). Hull (eq. 4.2) establishes that with continuous compounding, an amount $A$ invested for $n$ years at rate $R$ grows to $Ae^{Rn}$. The fundamental relationship is:

$$P(0,T) = e^{-z_c(T) \cdot T}$$

Solving for the rate (Hull Ch 4):

$$\boxed{z_c(T) = -\frac{\ln P(0,T)}{T}}$$

**Why it's preferred by quants:** Hull explains that "continuously compounded interest rates are used to such a great extent in pricing derivatives that it makes sense to get used to working with them now." The mathematical convenience—particularly that discount factors multiply simply as $e^{-r_1 t_1} \times e^{-r_2 t_2} = e^{-(r_1 t_1 + r_2 t_2)}$—makes continuous compounding the standard in quantitative finance.

### 3.2.3 Semiannual Compounding (Treasury Convention)

This is the standard for U.S. bond markets. Hull (eq. 4.1) gives the general compounding formula: an amount $A$ invested at rate $R$ compounded $m$ times per year grows to $A(1 + R/m)^{mn}$ after $n$ years. For semiannual compounding:

$$P(0,T) = \frac{1}{\left(1 + \frac{z_{sa}(T)}{2}\right)^{2T}}$$

Solving for the rate:

$$\boxed{z_{sa}(T) = 2 \left[ \left( \frac{1}{P(0,T)} \right)^{\frac{1}{2T}} - 1 \right]}$$

### 3.2.4 Simple Interest (Money Market Convention)

Used for short-term rates (typically $T < 1$ year), like SOFR fixings and T-Bills. Tuckman explains that "lending $\$1$ for $d$ days at a rate of $r$ will earn the lender an interest payment of $rd/360$ dollars at the end of the $d$ days" under the actual/360 convention. There is no "compounding" of interest on interest.

$$P(0,T) = \frac{1}{1 + z_{simple}(T) \cdot T}$$

Solving for the rate:

$$\boxed{z_{simple}(T) = \frac{1}{T} \left( \frac{1}{P(0,T)} - 1 \right)}$$

### 3.2.5 Convention Risk: The Map is Not the Territory

A common rookie mistake is to assume "5% is 5%."

**Example (adapted from Hull Ch 4):**
Suppose $P(0, 1) = 0.9512$ (roughly 5%).
- **Simple Rate:** $(1/0.9512 - 1) / 1 = 5.130\%$
- **Semiannual Rate:** $2 \times ((1/0.9512)^{0.5} - 1) = 5.063\%$
- **Continuous Rate:** $-\ln(0.9512) = 5.003\%$

All three represent the **exact same economic value**. If you plug a 5.13% simple rate into a formula expecting a continuous rate, you will misprice the instrument.

**Practical rule:** Before comparing rates (or hedges) from different systems, confirm the compounding and day-count conventions. If anything is ambiguous, convert everything back to discount factors first, then re-express in the desired convention.

> **Desk Reality: Convention Mismatch = Reconciliation Breaks**
>
> Convention mismatches show up as:
> - curve “shape” disagreements between systems (because one is plotting continuous zeros and another is plotting semiannual zeros);
> - PV/DV01 differences that look like “model risk” but are really a unit conversion issue;
> - cashflow disputes when one system used ACT/360 and another used 30/360 (or different 30/360 variants).
>
> Practical habit: when reconciling, translate everything into **discount factors and cashflows** first (the unambiguous objects), then translate back to the rate language each desk uses.

### 3.2.6 Conversion Between Compounding Conventions

Hull (eq. 4.3, 4.4) provides the general conversion formulas. If $R_c$ is a continuously compounded rate and $R_m$ is a rate with compounding $m$ times per year:

$$\boxed{R_c = m \ln\left(1 + \frac{R_m}{m}\right)}$$

$$\boxed{R_m = m\left(e^{R_c/m} - 1\right)}$$

More generally, to convert from compounding $m_1$ times per year at rate $R_1$ to compounding $m_2$ times per year at rate $R_2$ (Hull Ch 4):

$$R_2 = m_2\left[\left(1 + \frac{R_1}{m_1}\right)^{m_1/m_2} - 1\right]$$

### Example 3.1: Multi-Convention Discount Factor

**Given:** A 2-year discount factor $P(0,2) = 0.9070$.

**Objective:** Express this as zero rates under all major conventions.

**Step 1: Continuous Rate**
$$z_c = -\frac{\ln(0.9070)}{2} = -\frac{-0.0976}{2} = 4.879\%$$

**Step 2: Semiannual Rate**
$$z_{sa} = 2\left[(1/0.9070)^{1/4} - 1\right] = 2\left[1.02470 - 1\right] = 4.939\%$$

**Step 3: Annual Rate**
$$z_1 = (1/0.9070)^{1/2} - 1 = 1.05001 - 1 = 5.001\%$$

**Step 4: Quarterly Rate**
$$z_4 = 4\left[(1/0.9070)^{1/8} - 1\right] = 4\left[1.01227 - 1\right] = 4.909\%$$

**Verification:** All four rates produce the same discount factor when applied with their respective compounding formulas. The ordering follows the mathematical property: for positive rates, more frequent compounding implies a lower quoted rate. Continuous < Quarterly < Semiannual < Annual.

**Sanity Check:** This ordering always holds for positive rates because more frequent compounding "works harder"—interest compounds more often, so you need a lower stated rate to achieve the same terminal value.

---

## 3.3 Forward Rates: Rates for Future Periods

### 3.3.1 The Concept

A forward rate answers the question: *What rate for a future period $[T_1, T_2]$ is consistent with today's discount factors?*

Hull defines forward rates as "the rates of interest implied by current zero rates for periods of time in the future" (Hull Ch 4).

This definition carries a critical implication: **the forward rate is not a forecast of where rates will be in the future.** It is a "breakeven" rate that prevents arbitrage between investing long-term vs. rolling over short-term investments.

### 3.3.2 The No-Arbitrage Derivation: Locking in the Forward Rate

Hull provides a clear illustration of how forward rates can be locked in (Hull Ch 4). Consider an investor who wants to guarantee a rate for a future period.

**Hull's Locking Strategy:**
Suppose the 1-year zero rate is 3% and the 2-year zero rate is 4% (both continuously compounded). An institution can borrow $\$100$ at 3% for 1 year and invest the money at 4% for 2 years:

- Borrow $\$100$ for 1 year at 3%: repay $100e^{0.03 \times 1} = \$103.05$ at end of year 1
- Invest $\$100$ for 2 years at 4%: receive $100e^{0.04 \times 2} = \$108.33$ at end of year 2

**Net Result:**
- Cash outflow of $\$103.05$ at year 1
- Cash inflow of $\$108.33$ at year 2

Since $108.33 = 103.05 \times e^{0.05}$, a return equal to 5% is earned on $\$103.05$ during the second year. This 5% is the forward rate for year 2, and it can be **locked in** today through this borrowing/lending strategy.

### 3.3.3 Forward Rates as Breakeven Rates

Tuckman emphasizes the breakeven interpretation: the forward rate is the rate at which an investor is **indifferent** between:
1. Investing at the short rate and rolling over
2. Locking in the longer rate

This is not a prediction—it's a mathematical relationship enforced by arbitrage.

In practice, forward rates are often treated as **breakeven levels**: if you lock a forward-starting fixed rate at today’s forward, the initial PV is near zero; eventual P&L depends on where the realized reference rate prints versus that level.

> **Desk Reality: “The Forward Is My Pain Threshold”**
>
> Desk language often treats the forward as the breakeven level you can lock in:
> - If you think the future realized reference rate will be **below** the forward, you generally prefer positions that benefit from **receiving fixed** at the forward.
> - If you think the realized rate will be **above** the forward, you prefer positions that benefit from **paying fixed** at the forward.
>
> Hull illustrates (via a historical case study) that large losses can occur when positions are effectively “bets” against the forward curve and the realized path moves the other way. The forward is not a forecast; it is the tradable breakeven embedded in today’s curve.

### 3.3.4 The General Forward Rate Formula

**Strategy A (The Forward Loan):**
Borrow \$1 at $T_1$ and repay $1 + F \cdot \tau$ at $T_2$. Value at $T_1$ is 1.

**Strategy B (Synthetic Construction):**
1. **Borrow Long:** Borrow $1/P(0, T_1)$ dollars today for term $T_2$ at rate $z(T_2)$. You owe $1/P(0, T_1) \div P(0, T_2)$ at time $T_2$.
2. **Lend Short:** Lend the $1/P(0, T_1)$ dollars today for term $T_1$. You receive exactly \$1 at time $T_1$.

**Net Cashflows:**
- **Today:** $+ \frac{1}{P(0,T_1)}$ (from borrowing) $- \frac{1}{P(0,T_1)}$ (from lending) = **0**.
- **Time $T_1$:** Receive **\$1** from the lending leg.
- **Time $T_2$:** Owe $\frac{P(0,T_1)}{P(0,T_2)}$ from the borrowing leg.

This synthetic structure creates a loan of \$1 at $T_1$ with a repayment obligation at $T_2$. To avoid arbitrage, the forward contract rate $F$ must match this repayment obligation.

$$1 + F_{\text{simple}} \cdot \tau = \frac{P(0,T_1)}{P(0,T_2)}$$

### 3.3.5 Forward Rate Formulas by Convention

**Simple Forward Rate (Money Market / FRA style):**
Used for FRAs and floating rate legs. Tuckman (eq. 2.14) provides:

$$\boxed{F_{\text{simple}}(0; T_1, T_2) = \frac{1}{\tau} \left( \frac{P(0,T_1)}{P(0, T_2)} - 1 \right)}$$

**Continuously Compounded Forward Rate:**
Used in theoretical modeling. Hull (eq. 4.5) gives:

$$\boxed{f_c(0; T_1, T_2) = \frac{z_c(T_2) \cdot T_2 - z_c(T_1) \cdot T_1}{T_2 - T_1} = \frac{\ln P(0, T_1) - \ln P(0, T_2)}{T_2 - T_1}}$$

**Semiannual Forward Rate:**
Tuckman (eq. 2.17) derives:

$$\boxed{f_{sa}(0; T_1, T_2) = 2 \left[ \left(\frac{P(0,T_1)}{P(0,T_2)}\right)^{\frac{1}{2(T_2-T_1)}} - 1 \right]}$$

### 3.3.6 Instantaneous Forward Rate

The limit as $T_2 \to T_1$. This describes the forward rate “at” a single maturity point $T$, under continuous compounding. Hull (Ch 4) defines the instantaneous forward rate as:

$$\boxed{f(0,T) = -\frac{\partial}{\partial T} \ln P(0,T)}$$

This is just calculus: if you know the full discount factor curve $P(0,T)$, the derivative of $\ln P(0,T)$ tells you the “marginal” rate at maturity $T$.

**The Integral Relationship:** integrating the definition recovers discount factors from forwards:

$$\boxed{P(0,T) = \exp\left(-\int_0^T f(0,u) \, du\right)}$$

This reminds you that a smooth $P(0,T)$ implies a smooth integral of $f(0,T)$—but in practice, your interpolation choices (Chapter 17) can make one representation look smooth while the other looks jagged.

---

## 3.4 Forward Rate Agreements (FRAs): Turning Forwards into Trades

### 3.4.1 FRA Mechanics

Hull (Ch 4) explains that a **forward rate agreement (FRA)** is an agreement to exchange a predetermined fixed rate for a reference rate observed at a future time, with both rates applying to a notional principal over a specified future accrual period.

Fix two dates $T_1 < T_2$ and let $\tau$ be the year fraction for $[T_1, T_2]$ (according to the contract’s day count). Let $K$ be the FRA fixed rate and let $R$ be the realized reference rate (e.g., a 3‑month term rate) observed at $T_1$ for the period $[T_1, T_2]$. No notional is exchanged; only the net interest difference is exchanged.

If settlement were made at the **end** of the accrual period (at $T_2$), the net interest difference for a “receive fixed, pay floating” FRA would be:

$$\text{Interest difference at }T_2 \;=\; N \, (K - R)\, \tau$$

In practice, because $R$ is known at $T_1$, FRAs are commonly **cash-settled at (or near) $T_1$** for the *present value* of that end-of-period difference. One common way to express this is:

$$\text{Settlement at }T_1 \;\approx\; \frac{N \, (K - R)\, \tau}{1 + R\,\tau}$$

**Worked example (mechanics):** Notional $N=\$100$ million, $\tau=0.25$, $K=3.5\\%$, realized reference rate $R=3.0\\%$.

- End-of-period interest difference: $100{,}000{,}000\\times(0.035-0.030)\\times 0.25 = \$125{,}000$.
- Cash settlement at $T_1$ (PV at the start of the period): $125{,}000/(1+0.030\\times 0.25) \\approx \$124{,}070$.

### 3.4.2 FRA Valuation

Hull’s key valuation insight is: when the fixed rate equals the relevant forward rate, the value of the FRA is zero. As time passes, the forward rate changes, and the FRA’s value moves with the difference between the contracted fixed rate and the current forward rate.

A common (textbook) approximation is to treat the net interest difference as if it is paid at the end of the accrual period and then discount it:

$$\boxed{V_0 \;\approx\; N \,(K - F)\, \tau \, P(0,T_2)}$$

where $F$ is the forward rate for $[T_1,T_2]$ under the relevant compounding convention and $P(0,T_2)$ is the discount factor to the payment date. Hull also emphasizes an important implication: an FRA can be valued by assuming forward rates are realized and discounting appropriately.

### 3.4.3 Trading the Forward: A Worked Example

**Setup:**
- Current 1-year zero rate: 4.00% (continuous)
- Current 2-year zero rate: 4.40% (continuous)
- Implied 1y1y forward: $\frac{0.044 \times 2 - 0.04 \times 1}{1} = 4.80\%$

**Trader's view:** Expects the 1-year rate in one year to be 4.20%, below the 4.80% forward.

**Trade:** Receive fixed at 4.80% on a 1y1y forward swap with $\$50$ million notional. The swap starts in 1 year and runs for 1 year.

**If the trader is correct:**
When the swap starts, the 1-year rate is 4.20%. The trader receives 4.80% and pays 4.20%, a gain of 60bp on $\$50$ million for 1 year:

$$\text{P&L} \approx \$50{,}000{,}000 \times 0.0060 \times 1 = \$300{,}000$$

(Exact P&L depends on discounting and day counts, but this gives the magnitude.)

**If the trader is wrong:**
If rates rise to 5.50%, the trader receives 4.80% but pays 5.50%, losing 70bp:

$$\text{P&L} \approx -\$50{,}000{,}000 \times 0.0070 \times 1 = -\$350{,}000$$

The forward rate is the breakeven. Beat it, you profit. Miss it, you lose.

---

## 3.5 The Spot–Forward Relationship

### 3.5.1 The Key Identity

Under continuous compounding, there is a beautiful relationship between the spot rate curve $z(T)$ and the instantaneous forward rate $f(T)$. Hull (Ch 4) derives this from the instantaneous forward formula:

$$\boxed{f(T) = z(T) + T \cdot \frac{\partial z(T)}{\partial T}}$$

**Derivation:**
Starting from $P(0,T) = e^{-z(T) \cdot T}$, we have $\ln P(0,T) = -z(T) \cdot T$.

Taking the derivative with respect to $T$:
$$f(T) = -\frac{\partial}{\partial T} \ln P(0,T) = -\frac{\partial}{\partial T}[-z(T) \cdot T] = z(T) + T \cdot z'(T)$$

### 3.5.2 Interpreting Curve Shape: The Marginal vs. Average Intuition

> **Analogy: The Grade Point Average (GPA)**
>
> Think of the **Spot Rate** ($z$) as your cumulative **GPA**.
> Think of the **Forward Rate** ($f$) as your **Semester Grade**.
>
> - If your Semester Grade (Forward) is **higher** than your GPA, your GPA **rises** ($f > z \implies z' > 0$).
> - If your Semester Grade is **lower** than your GPA, your GPA **falls** ($f < z \implies z' < 0$).
> - If your Semester Grade equals your GPA, your GPA stays **flat** ($f = z \implies z' = 0$).
>
> The forward rate is the "marginal" rate pulling the "average" (spot) rate up or down.

This equation gives us the **"Marginal vs. Average"** intuition, similar to marginal and average costs in economics:

1. **Upward Sloping Curve ($z' > 0$):** If spot rates are rising, the forward rate must be **higher** than the spot rate ($f > z$). To raise the average, the new marginal rate must be higher.
2. **Downward Sloping Curve ($z' < 0$):** If spot rates are falling (inverted curve), the forward rate must be **lower** than the spot rate ($f < z$).
3. **Flat Curve ($z' = 0$):** Spot and forward rates are equal.

**Numerical Example:**
If the 5-year spot rate is 4% and rising at 0.20% per year (i.e., $z'(5) = 0.002$), then:
$$f(5) = 0.04 + 5 \times 0.002 = 0.04 + 0.01 = 5\%$$

The forward rate at 5 years is 100bp higher than the spot rate because the curve is upward sloping.

### 3.5.3 Practical Implication: Forwards are "Wigglier"

Because forward rates depend on the *derivative* (slope) of the spot curve, small bumps in the spot curve become large spikes in the forward curve.

> **Analogy: The Speedometer**
>
> - **Spot Rate**: Your Average Speed over a trip ($D/T$).
> - **Forward Rate**: Your Instantaneous Speed *right now* (reading on the speedometer).
>
> You can maintain a smooth Average Speed of 60mph (Spot) even if you slam on the brakes or gas (Forwards) periodically. But if you want to change your Average Speed quickly (a kink in the Spot curve), you have to drive *extremely* fast or slow for a short time.
>
> *Result:* Small bumps in the "smooth" spot curve require huge spikes in the forward curve.

- A localized "kink" in spot rates → A massive jump in forward rates.
- This is why **smoothness** is a key constraint in curve fitting (Chapter 17). If you bootstrap a curve naively, your forward rates will look like a saw blade, which implies arbitrage opportunities (e.g., implausible calendar spread valuations).

> **Practical troubleshooting: Forward rate spikes**
>
> If your forward curve has sharp spikes (or surprising negatives) at a few tenors, don’t interpret it first—**debug it first**:
>
> 1. **Interpolation choice** — some methods make forwards discontinuous or noisy (see Chapter 17).
> 2. **Inputs** — missing/stale quotes, wrong instrument definitions, wrong day counts, wrong calendars.
> 3. **Local kinks** — sometimes the market really is kinked at a tenor boundary, but you need to confirm it’s not a modeling artifact.

---

## 3.6 Par Rates: Annuity-Weighted Summaries

### 3.6.1 The Concept

A par rate answers: *What coupon rate $C$ makes a bond (or swap) worth exactly 100% of par?*

While zero rates apply to single cashflows, par rates apply to **series** of cashflows. Hull defines the par yield as "the coupon rate that causes the bond price to equal its par value" (Hull Ch 4). They are weighted averages of the discount curve.

### 3.6.2 The Par Equation

For a bond paying coupon $C$ semiannually on dates $T_1, \dots, T_n$:

$$\text{Price} = C \sum_{i=1}^n \tau_i P(0,T_i) + 1 \cdot P(0,T_n) = 1$$

Solving for $C$:

$$\boxed{C_{\text{par}} = \frac{1 - P(0,T_n)}{\sum_{i=1}^n \tau_i P(0,T_i)}}$$

Hull (Ch 4) provides an equivalent form:
$$c = \frac{(100 - 100d)m}{A}$$
where $d$ is the final discount factor, $m$ is the payment frequency, and $A$ is the annuity factor.

### 3.6.3 The Annuity

The denominator is called the **PV01** or the **Annuity** of the bond/swap:

$$A(0) = \sum_{i=1}^n \tau_i P(0,T_i)$$

This represents the present value of receiving 1 unit of currency per period (or equivalently, 1 basis point of coupon payments).

### 3.6.4 Par Swap Rates

This is exactly how swap rates are quoted. A "5-year Swap Rate" is simply the fixed rate that makes the PV of the fixed leg equal the PV of the floating leg. Since the floating leg is typically valued at par (initially), the swap rate is just the par coupon calculation above.

### 3.6.5 Why Par Rates Matter

Par rates are the **observable market quotes**. You don't typically observe zero rates or discount factors directly—you observe Treasury yields (which are par-like) and swap rates (which are par rates). The bootstrap process (Chapter 17) inverts these par quotes to recover the underlying discount factors.

---

## 3.7 The Coupon Effect: Why Par Yields Differ from Zero Yields

### 3.7.1 The Phenomenon

Tuckman (Ch 3) explains that "the impact of coupon level on the yield-to-maturity of coupon bonds with the same maturity is called the coupon effect." On a non-flat curve, par bonds and zero-coupon bonds of the same maturity have different yields—even though both are fairly priced.

### 3.7.2 Why Par Yields Are Below Zero Yields on Steep Curves

On an **upward-sloping** (steep) yield curve, par yields are **below** zero yields of the same maturity. The intuition is:

- A par bond has intermediate coupon payments that get discounted at the *lower* short-term rates
- Only the final principal gets discounted at the *higher* long-term rate
- This "front-loading" of cash flows at lower discount rates pulls the par yield below the zero yield

Tuckman explains: "As term increases, the number of coupon payments increases and discounting reduces the relative importance of the final principal payment... intermediate spot rates have a larger impact on coupon bond yields."

### 3.7.3 Mathematical Demonstration

Consider a 2-year par bond with semiannual coupons. Suppose:
- 6-month zero rate: 3.0%
- 1-year zero rate: 3.5%
- 1.5-year zero rate: 3.8%
- 2-year zero rate: 4.0%

The discount factors are:
$$P(0.5) = e^{-0.03 \times 0.5} = 0.9851$$
$$P(1.0) = e^{-0.035 \times 1.0} = 0.9656$$
$$P(1.5) = e^{-0.038 \times 1.5} = 0.9446$$
$$P(2.0) = e^{-0.04 \times 2.0} = 0.9231$$

The annuity is:
$$A = 0.5 \times (0.9851 + 0.9656 + 0.9446 + 0.9231) = 1.9092$$

The par coupon is:
$$C = \frac{1 - 0.9231}{1.9092} = \frac{0.0769}{1.9092} = 4.03\%$$

The par yield (4.03%) is **below** the 2-year zero rate (4.00% continuous, or about 4.08% semiannually equivalent). The coupon effect pulled the par yield down because the early coupons were discounted at lower rates.

### 3.7.4 The Inverted Curve Case

On an **inverted** (downward-sloping) curve, the relationship **reverses**: par yields **exceed** zero yields.

When short rates are higher than long rates:
- Early coupons are discounted at *higher* short-term rates
- The principal is discounted at a *lower* long-term rate
- This "front-loading" at higher discount rates pushes up the IRR (par yield) needed for par pricing

> **Desk Reality: The "Cheap" Par Bond Illusion**
>
> Junior traders sometimes think a par bond looks "cheap" relative to a zero-coupon bond when par yield < zero yield on a steep curve. This is not cheapness—it's the coupon effect. Both bonds are fairly priced; they just have different duration profiles and different reinvestment assumptions.
>
> **Key insight:** Par yield equals zero yield **only on a flat curve**. The steeper the curve, the larger the gap. On a 250bp steep curve, the difference can be 10-15bp at long maturities (Tuckman reports 14bp at 20 years on one sample date).

### 3.7.5 Quantifying the Coupon Effect

Tuckman provides empirical magnitudes for one historical date: "The difference between the zero and par rates is about 1.3 basis points at a term of 5 years, 6.1 at 10 years, and 14.1 at 20 years." However, he cautions that "these quantities cannot be easily extrapolated to other yield curves" because the effect depends critically on the shape of the term structure.

---

## 3.8 The Expectations Hypothesis and Its Critique

### 3.8.1 What the Expectations Hypothesis Says

Luenberger (Ch 4) presents the **expectations hypothesis**: "This more precise formulation is the expectations hypothesis. To outline this hypothesis, consider the forward rate $f_{12}$, which is the implied rate for money loaned for 1 year, a year from now. According to the expectations hypothesis, this forward rate is exactly equal to the market expectation of what the 1-year spot rate will be next year."

In other words, forward rates are **unbiased predictors** of future spot rates.

### 3.8.2 Why the Expectations Hypothesis Fails

Luenberger acknowledges the fundamental weakness: "According to this explanation, the market expects rates to increase whenever the spot rate curve slopes upward; and this is practically all the time. Thus the expectations cannot be right even on average, since rates do not go up as often as expectations would imply."

The yield curve is usually upward sloping, which would imply markets perpetually expect rising rates. But rates don't actually rise that often. Something else must be going on.

### 3.8.3 Liquidity Preference Theory

Luenberger presents the alternative: "The liquidity preference explanation asserts that investors usually prefer short-term fixed income securities over long-term securities."

Why? "Investors anticipate that they may need to sell their bonds soon, and they recognize that long-term bonds are more sensitive to interest rate changes than are short-term bonds. Hence an investor who may need funds in a year or so will be reluctant to place these funds in long-term bonds because of the relatively high near-term risk."

**The implication:** Investors demand a **term premium** (or **risk premium**) for holding longer-dated bonds. This premium makes forward rates systematically **higher** than expected future spot rates.

$$\text{Forward Rate} = \text{Expected Future Spot Rate} + \text{Term Premium}$$

### 3.8.4 Practical Implications

Luenberger concludes: "The expectations theory is the most analytical of the three, in the sense that it offers concrete numerical values for expectations, and hence it can be tested. These tests show that it works reasonably well with a deviation that seems to be explained by liquidity preference. Hence expectations tempered by the risk considerations of liquidity preference seem to offer a good straightforward explanation."

> **Desk Reality: Forward ≠ Expected (Term Premium Matters)**
>
> In trading language, forward rates are *tradable breakevens*, but they need not equal the market’s expectation of future spot rates. A gap can arise because investors demand compensation for bearing interest-rate risk over longer horizons (a term premium).
>
> Why this matters on the desk:
> - **Curve trades:** A view on “forwards are too high/low” implicitly includes a view on term premia, not just policy expectations.
> - **Carry/rolldown:** Roll-down intuition often assumes rates don’t jump around; term premia and risk aversion can dominate realized outcomes.
>
> `NOT SURE: If you want this book to use a specific numerical term-premium model/estimator (and typical magnitudes), specify which framework and data source to adopt.`

---

## 3.9 The Complete Triangle: Converting Between Representations

### 3.9.1 Visualizing the Rate Relationships

> **The Triangular Array (Luenberger)**
>
> Luenberger introduces the "Triangular Array" to visualize how today's curve implies the entire future:
>
> | | $T=1$ | $T=2$ | $T=3$ | $T=4$ |
> | :--- | :--- | :--- | :--- | :--- |
> | **Today ($t=0$)** | $z_1$ (Spot) | $z_2$ (Spot) | $z_3$ (Spot) | $z_4$ (Spot) |
> | **Future ($t=1$)** | | $f_{1,2}$ (Fwd) | $f_{1,3}$ | $f_{1,4}$ |
> | **Future ($t=2$)** | | | $f_{2,3}$ (Fwd)| $f_{2,4}$ |
> | **Future ($t=3$)** | | | | $f_{3,4}$ (Fwd)|
>
> - **Row 1**: The Spot Curve observed today.
> - **Diagonals**: The path of short-term rates implied by the curve.
> - **Insight**: One spot curve implies an entire matrix of future forward rates.

### 3.9.2 The Conversion Map

We can now navigate the full map:

| From $\downarrow$ To $\rightarrow$ | **Discount Factors $P(0,T)$** | **Zero Rates $z_c(T)$** (Continuous) | **Par Rates $C_{par}$** |
| :--- | :--- | :--- | :--- |
| **Discount Factors** | — | $z_c = -\frac{\ln P}{T}$ | $C = \frac{1 - P(T_n)}{A(0)}$ |
| **Zero Rates** | $P = e^{-z_c T}$ | — | Sample $P(T_i)$ from curve, then use Par formula |
| **Par Rates** | **Bootstrapping** (See Ch 17) | Convert Par → DF → Zero | — |

- **Discount Factors ↔ Zero Rates**: Simple algebraic inversion.
- **Discount Factors → Par Rates**: Weighted average calculation.
- **Par Rates → Discount Factors**: Requires **Bootstrapping**.

> **Analogy: The Bootstrapping Ladder**
>
> You cannot reach the 5-year rung (5y Zero) without climbing past the 1, 2, 3, and 4-year rungs.
> - To value a 5-year Par Bond, you first need to value its coupons at years 1, 2, 3, and 4.
> - You must use the *already known* discount factors for years 1-4 to "strip out" those coupons.
> - Only then is the 5-year principal exposed, allowing you to solve for the 5-year discount factor.
>
> This is why we call it *bootstrapping* (pulling yourself up step-by-step).

You cannot convert a single par rate to a single discount factor in isolation; you need the entire path of prior discount factors to strip out the coupons.

### 3.9.3 Forward Rates in the Triangle

Forward rates can be derived from either discount factors or zero rates:
- From discount factors: $F = \frac{1}{\tau}\left(\frac{P_1}{P_2} - 1\right)$ (Tuckman eq. 2.14)
- From zero rates: $f_c = \frac{z_2 T_2 - z_1 T_1}{T_2 - T_1}$ (Hull eq. 4.5)

---

## 3.10 Worked Examples

### Example A: Discount Factor to Zero Rates (Multiple Conventions)

**Given:** $P(0, 1y) = 0.9700$.

1. **Simple Zero:**
   $$z_s = \frac{1}{1} \left( \frac{1}{0.9700} - 1 \right) = 3.092\%$$
2. **Continuous Zero:**
   $$z_c = -\frac{\ln(0.9700)}{1} = 3.046\%$$
3. **Semiannual Zero:**
   $$z_{sa} = 2 \left( (1/0.9700)^{0.5} - 1 \right) = 2(1.01535 - 1) = 3.069\%$$

**Sanity Check:** Continuous < Semiannual < Simple. This ordering always holds for positive rates.

### Example B: Forward Rate Calculation

**Given:** $P(0, 1y) = 0.9615$ and $P(0, 2y) = 0.9157$.

**Objective:** Find the 1-year forward rate starting in 1 year ($1y \times 1y$). $\tau = 1$.

**Simple forward rate:**
$$F = \frac{1}{1} \left( \frac{0.9615}{0.9157} - 1 \right) = 1.0500 - 1 = 5.00\%$$

**Continuous forward rate:**
$$f_c = \frac{\ln(0.9615) - \ln(0.9157)}{1} = \frac{-0.0393 - (-0.0881)}{1} = 4.88\%$$

**Verification:** Check that rolling over gives the same result as investing for 2 years:
- Invest for 2 years: investing \$1 today grows to $1/P(0,2)=1/0.9157=1.0920$ at year 2.
- Invest for 1 year, then reinvest for year 2 at the 1y1y simple forward: $1/P(0,1)\times(1+F)=1/0.9615\times 1.05=1.0921$ ✓ (rounding)

### Example C: Calculating a Forward Rate (Negative Rate Case)

**Given:** $P(0, 1y) = 1.0020$ and $P(0, 1.5y) = 1.0030$.
*Note: $P > 1$ implies negative interest rates.*

**Objective:** Find the 6-month forward rate $6m \times 12m$ (starting in 1y, ending 1.5y). $\tau = 0.5$.

$$1 + F \cdot \tau = \frac{P(0, 1y)}{P(0, 1.5y)} = \frac{1.0020}{1.0030} \approx 0.999003$$

$$F = \frac{0.999003 - 1}{0.5} \approx -0.199\%$$

**Interpretation:** The forward rate is negative. You would *pay* interest to lend money for this period. Negative rates are economically possible; the algebra works the same way.

### Example D: Par Rate Calculation

**Given:** Discount Factors: $P(0.5) = 0.99$, $P(1.0) = 0.97$.
**Objective:** Calculate the 1-year semiannual par rate $C$.

1. **Calculate Annuity ($A$):**
   $$A = 0.5 \times 0.99 + 0.5 \times 0.97 = 0.495 + 0.485 = 0.980$$
2. **Calculate Par Numerator:**
   $$1 - P(0, 1y) = 1 - 0.97 = 0.03$$
3. **Solve:**
   $$C = \frac{0.03}{0.980} \approx 0.03061 \implies 3.061\%$$

**Sanity Check:** The par rate should be between the 6-month and 1-year zero rates. Check: 6-month zero ≈ 2.02%, 1-year zero ≈ 3.09%. Par rate 3.061% is in range. ✓

### Example E: Spot-Forward Relationship

**Given:** Zero rate curve $z(T) = 0.03 + 0.005T$ (continuous compounding, linear in $T$).

**Objective:** Find the instantaneous forward rate curve $f(T)$.

Using $f(T) = z(T) + T \cdot z'(T)$:
- $z'(T) = 0.005$
- $f(T) = (0.03 + 0.005T) + T \cdot 0.005 = 0.03 + 0.01T$

**Interpretation:** The forward curve has **twice the slope** of the zero curve. At $T=5$: $z(5) = 5.5\%$ but $f(5) = 8\%$.

### Example F: Full Triangle Conversion

**Given:** 1-year zero = 3.5%, 2-year zero = 4.0%, 3-year zero = 4.3% (all semiannual compounding).

**Step 1: Convert to Discount Factors**
$$P(1) = \frac{1}{(1 + 0.035/2)^2} = \frac{1}{1.0175^2} = 0.9659$$
$$P(2) = \frac{1}{(1 + 0.040/2)^4} = \frac{1}{1.02^4} = 0.9238$$
$$P(3) = \frac{1}{(1 + 0.043/2)^6} = \frac{1}{1.0215^6} = 0.8802$$

**Step 2: Calculate Forward Rates**
1y1y forward (semiannual compounding over the year 1→2):
$$f_{1,2} = 2\left[\left(\frac{0.9659}{0.9238}\right)^{1/2} - 1\right] = 2\left[1.0225 - 1\right] = 4.50\%$$

2y1y forward (semiannual compounding over the year 2→3):
$$f_{2,3} = 2\left[\left(\frac{0.9238}{0.8802}\right)^{1/2} - 1\right] = 2\left[1.0245 - 1\right] = 4.90\%$$

**Step 3: Calculate Par Coupons (Annual-Pay)**

If we define a **par bond** here as paying an **annual** coupon $C$ at $T=1,2,3$ and principal at $T=3$ (price = 1), then:

$$1 = C\,[P(1)+P(2)+P(3)] + P(3)$$

So the 3-year annual-pay par coupon is:
$$C_3 = \frac{1 - P(3)}{P(1)+P(2)+P(3)} = \frac{1-0.8802}{0.9659+0.9238+0.8802} = 4.33\%$$

Similarly, the 2-year annual-pay par coupon is:
$$C_2 = \frac{1 - P(2)}{P(1)+P(2)} = \frac{1-0.9238}{0.9659+0.9238} = 4.03\%$$

**Note:** If you want a **semiannual-pay** par coupon (Treasury-style) you need discount factors at $0.5$-year intervals. That is exactly why curve construction (Chapter 17) is not optional in real pricing work: instruments don’t line up neatly on annual dates.

### Example G: Steep vs Flat Curve Comparison

**Steep Curve:** 1y = 2%, 2y = 4% (semiannual)
$$P(1) = 1/1.01^2 = 0.9803$$
$$P(2) = 1/1.02^4 = 0.9238$$

Forward rate 1y1y:
$$f = 2\left[(0.9803/0.9238)^{0.5} - 1\right] = 2\left[1.0301 - 1\right] = 6.02\%$$

This forward (6.02%) is significantly higher than both spot rates—a steep curve implies high forward rates.

**Flat Curve:** 1y = 3%, 2y = 3% (semiannual)
$$P(1) = 1/1.015^2 = 0.9707$$
$$P(2) = 1/1.015^4 = 0.9422$$

Forward rate 1y1y:
$$f = 2\left[(0.9707/0.9422)^{0.5} - 1\right] = 2\left[1.0150 - 1\right] = 3.00\%$$

On a flat curve, the forward equals the spot. ✓

---

## 3.11 Sanity Checks and Common Pitfalls

### 3.11.1 Monotonicity Check for Discount Factors

Discount factors must be strictly decreasing in maturity (assuming positive rates):
$$P(T_1) > P(T_2) \text{ for } T_1 < T_2$$

If this fails, either:
- Rates are negative over some maturities (so discount factors can increase with $T$)
- Data error (check inputs)

**Normal range:** Discount factors should be between 0 and 1 for positive rates, and can exceed 1 for negative rates.

### 3.11.2 Forward Rate Sign Check

Forward rates can be negative in low/negative-rate environments. What you want to catch is *inconsistency*:
- **Jagged forwards:** big spikes between adjacent tenors often come from interpolation choices or bad inputs.
- **Stale/misaligned quotes:** wrong instrument definitions, day counts, calendars, or settlement assumptions.
- **Repricing failure:** if the curve does not reprice the instruments used to build it, don’t interpret the forwards.

If you want numerical “red flag” thresholds for a specific market, you need a defined dataset and market convention set (tenors, instruments, and interpolation method).

### 3.11.3 The Triangle Must Close

The three rates (zero, forward, par) must be consistent:
- Given any two, the third is determined
- If you have independent quotes for all three, they should be arbitrage-consistent
- If not, there's either a trading opportunity or (more likely) a data error

### 3.11.4 Common Pitfalls

> **Common Pitfalls to Avoid:**
>
> 1. **Convention mismatch:** Comparing semiannual zero to continuous forward—always convert to common convention first
> 2. **Day count confusion:** Using ACT/365 zero rate with ACT/360 forward rate calculation
> 3. **Interpolation artifacts:** Forward rates can become negative or spike at tenor boundaries due to interpolation method (see Chapter 17)
> 4. **Stale data:** Using yesterday's curve with today's trades
> 5. **Treating forwards as forecasts:** Forward rates are breakevens, not predictions
> 6. **Par-zero confusion:** Thinking par yield < zero yield means the par bond is cheap (it's just the coupon effect)

---

## 3.12 Interpolation Preview (Brief)

Curve construction (Chapter 17) requires interpolation choices that profoundly affect forward rates:
- **Piecewise linear on zeros** → discontinuous forwards (steps)
- **Piecewise linear on forwards** → continuous forwards but potentially kinked zeros
- **Splines on zeros** → smooth forwards but can oscillate

The key insight: what looks smooth in one representation may look jagged in another. Forward rates are particularly sensitive because they depend on the *derivative* of the discount curve.

> For detailed treatment of interpolation methods and their impact on forward rate stability, see Chapter 17 (Curve Bootstrapping).

---

## 3.13 HJM Framework Connection (Brief)

The instantaneous forward rate $f(0,T)$ is the foundation of the **Heath-Jarrow-Morton (HJM) framework** for interest rate modeling.

The key HJM insight: in a no-arbitrage world, the drift of forward rates is determined by their volatilities. Specifically, under the risk-neutral measure, the drift of $f(t,T)$ is constrained by:

$$\mu_f(t,T) = \sigma_f(t,T) \int_t^T \sigma_f(t,u) \, du$$

This "HJM drift restriction" has profound implications: you can choose the volatility structure freely, but the drift is then pinned down by no-arbitrage.

> For rigorous treatment of the HJM framework, see Appendix A3. For now, the key takeaway is that forward rates are not just accounting artifacts—they are the state variables in continuous-time interest rate models.

---

## Summary

1. **Discount Factors ($P(0,T)$)** are the primitive input: the price today of \$1 paid at maturity $T$ under the chosen discounting assumptions. Which curve you use depends on the product and collateral/discounting conventions (for many collateralized swaps, an overnight/OIS-type curve is commonly used).

2. **Zero Rates** are just discount factors expressed in per-annum terms. They require a **compounding convention** to be meaningful. Same DF → different rate numbers.

3. **Forward Rates** are the rates for future periods implied by the term structure to prevent arbitrage. They are **breakeven rates**, not forecasts. They can be locked in via FRAs or synthetic borrowing/lending strategies.

4. **The Spot-Forward Relationship** ($f = z + Tz'$) shows that forwards depend on the slope of the zero curve. Forward rates are "wigglier" than spot rates—small bumps in spots become large spikes in forwards.

5. **Par Rates** are the coupons that value a bond/swap at par. They are annuity-weighted averages of the discount curve.

6. **The Coupon Effect** explains why par yields differ from zero yields on non-flat curves. On steep (upward) curves, par < zero; on inverted curves, par > zero.

7. **The Expectations Hypothesis** fails empirically. Forward rates systematically exceed expected future spot rates due to the **term premium** (liquidity preference).

8. **The Triangle:** You can translate freely between these forms. Quants use DFs; Traders use Forwards/Pars; Risk uses Zeros. The key equations are:
   - $P = e^{-zT}$ (DF from zero)
   - $z = -\ln(P)/T$ (zero from DF)
   - $F = (P_1/P_2 - 1)/\tau$ (forward from DFs)
   - $f(T) = z(T) + Tz'(T)$ (instantaneous forward from zero curve)
   - $C_{par} = (1 - P_n) / A$ (par from DFs)

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
| :--- | :--- | :--- |
| **Discount Factor** | $P(0,T) = \text{PV of } \$1$ | The universal valuation primitive; curve choice depends on discounting/collateral conventions. |
| **Zero Rate (Spot)** | Rate solving $P = e^{-zT}$ or $(1+z/m)^{-mT}$ | Useful for visualizing term structure levels. Convention-dependent. |
| **Forward Rate** | $F = \frac{1}{\tau}(\frac{P_1}{P_2}-1)$ | Breakeven rate for future periods. Can be locked in via FRAs. |
| **Instantaneous Forward** | $f(T) = -\frac{\partial}{\partial T}\ln P(0,T)$ | Foundation of HJM framework; drives curve smoothness requirements. |
| **Par Rate** | $C = \frac{1-P_n}{\text{Annuity}}$ | The language of the market (Swap/Bond quotes). |
| **Annuity (PV01)** | $\sum \tau_i P(T_i)$ | Sensitivity to parallel rate shifts; denominator for par rates. |
| **Spot-Forward Relation** | $f = z + Tz'$ | Links curve shape to forward rates; marginal vs average intuition. |
| **Coupon Effect** | Par ≠ Zero yield on non-flat curves | Explains apparent "cheapness" that isn't real cheapness. |
| **Expectations Hypothesis** | Forwards = Expected future spots | Fails empirically; term premium exists. |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P(0,T)$ or $d(T)$ | Discount factor for maturity $T$ |
| $z_c(T)$ | Continuously compounded zero rate |
| $z_{sa}(T)$ | Semiannually compounded zero rate |
| $z_s(T)$ | Simple (money market) zero rate |
| $f(0;T_1,T_2)$ | Forward rate from $T_1$ to $T_2$ |
| $f(0,T)$ | Instantaneous forward rate at $T$ |
| $C_{par}$ | Par coupon rate |
| $A(0)$ | Annuity (PV01) |
| $\tau$ | Year fraction ($T_2 - T_1$) |
| $R_c$ | Continuously compounded rate |
| $R_m$ | Rate compounded $m$ times per year |

---

## Flashcards

| # | Question | Answer |
|---|---|---|
| 1 | What is the fundamental difference between a Zero Rate and a Par Rate? | Zero rate applies to a single cashflow (terminal); Par rate applies to a series of cashflows (coupon/annuity). |
| 2 | What is the formula for a continuous zero rate given $P(0,T)$? | $z_c = -\frac{\ln P(0,T)}{T}$ |
| 3 | If the Zero Curve is upward sloping, is the Forward Rate higher or lower than the Zero Rate? | Higher. Marginal > Average to pull the average up. |
| 4 | How do you calculate a simple forward rate from $T_1$ to $T_2$? | $F = \frac{1}{\tau} (\frac{P(T_1)}{P(T_2)} - 1)$ |
| 5 | For collateralized swaps, what discount curve is commonly used? | An overnight/OIS-type curve consistent with the collateral rate (often called “OIS discounting”). |
| 6 | Why are forward rates "wigglier" than zero rates? | Forward rates depend on the derivative (slope) of the zero curve; differentiation amplifies noise. |
| 7 | What is the "Annuity" in the context of par rates? | The PV of receiving 1 unit of currency per period: $\sum \tau_i P(T_i)$. |
| 8 | Can Discount Factors be greater than 1? | Yes, if interest rates are negative. |
| 9 | What arbitrage strategy enforces the forward rate? | Borrowing long and lending short (or vice versa) to lock in a future rate. |
| 10 | Why is Linear Interpolation of Discount Factors dangerous? | It creates discontinuous forward rates, leading to pricing anomalies in forward-starting products. |
| 11 | What is the instantaneous forward rate $f(T)$ mathematically? | $f(T) = -\frac{\partial}{\partial T} \ln P(0,T)$ |
| 12 | How do you recover $P(0,T)$ from instantaneous forwards? | $P(0,T) = \exp(-\int_0^T f(0,u)\,du)$ |
| 13 | State the spot-forward relationship for continuous compounding. | $f(T) = z(T) + T \cdot z'(T)$ |
| 14 | Convert 5% continuous to semiannual compounding. | $R_{sa} = 2(e^{0.05/2} - 1) \approx 5.06\%$ |
| 15 | If the zero curve is flat at 4%, what is the forward rate? | 4% — flat curve means forward equals spot. |
| 16 | Given $P(1) = 0.95$ and $P(2) = 0.89$, what is the 1y×1y simple forward? | $F = (0.95/0.89 - 1)/1 = 6.74\%$ |
| 17 | What does a negative forward rate mean economically? | You pay interest to lend money (lender pays borrower). |
| 18 | Why can't you convert a single par rate to a single discount factor? | Par rates depend on multiple DFs (the annuity). You need the full path via bootstrapping. |
| 19 | What does it mean when we say the forward rate is the "breakeven" rate? | The forward rate is the rate at which an investor is indifferent between (1) investing short and rolling, or (2) locking in the longer rate. It's not a forecast. |
| 20 | Why is par yield < zero yield on a steep curve? | The "coupon effect": early coupons are discounted at lower rates, pulling down the average yield needed for par pricing. |
| 21 | What does the liquidity preference theory say about forward rates? | Forward rates = expected future spot rates + term premium. The term premium compensates for duration risk in longer maturities. |

---

## Mini Problem Set

1. **Conversion:** $P(0, 2y) = 0.92$. Calculate the continuous zero rate and the semiannual zero rate.

2. **Forward:** $P(0, 1y) = 0.98, P(0, 2y) = 0.95$. Calculate the simple forward rate for the 2nd year ($1y \times 1y$).

3. **Par Rate:** A 2-year annual-pay bond has DFs: $P(1)=0.98, P(2)=0.95$. Calculate the Par Coupon.

4. **Arbitrage:** You see a quoted forward rate of 4% for year 2. Your calculated forward rate from the curve is 3%. Describe the arbitrage trade.

5. **Slope:** The zero curve is flat at 5%. What is the instantaneous forward rate?

6. **Negative Rates:** $P(0, 1) = 1.01$. What is the simple zero rate? What is the continuous zero rate?

7. **Annuity:** Calculate the PV01 of a 2-year annual bond if $P(1)=0.95, P(2)=0.90$.

8. **Spot-Forward:** If $z(T) = 0.05 + 0.01T$ (continuous), find $f(T)$.

9. **Convention:** Convert 5% continuous to semiannual.

10. **Bootstrapping Logic:** If you know the 1y par rate and 2y par rate, can you find $P(0,2)$? Explain the steps.

11. **Instantaneous Forward:** Given $P(0,T) = e^{-0.04T - 0.002T^2}$, find $f(0,T)$.

12. **Integral Form:** If $f(0,T) = 0.03 + 0.01T$, find $P(0,3)$ using the integral formula.

13. **Coupon Effect:** Given a steep curve (1y = 2%, 2y = 4%, 3y = 5% semiannual), calculate the 1y1y and 1y2y forward rates. Explain why they exceed the 2y and 3y spot rates.

14. **Trading Forwards:** A trader believes the 1-year rate one year from now will be 3.8%, but the 1y1y forward is 4.5%. Describe a trade to profit from this view and calculate the approximate P&L on $100mm notional if the trader is correct.

15. **Inverted Curve:** On an inverted curve, is par yield above or below zero yield? Explain intuitively.

### Solutions

1. **Conversion**: $z_c = -\ln(0.92)/2 = 0.0834/2 = 4.17\%$. $z_{sa} = 2((1/0.92)^{1/4} - 1) = 2(1.0210 - 1) = 4.21\%$.

2. **Forward**: $F = (0.98/0.95 - 1)/1 = 0.0316 = 3.16\%$.

3. **Par Rate**: $A = 1 \times 0.98 + 1 \times 0.95 = 1.93$. $C = (1 - 0.95)/1.93 = 0.05/1.93 = 2.59\%$.

4. **Arb**: Market forward (4%) is too high relative to curve (3%). Strategy: "Lend" at the market forward (buy FRA/receive fixed at 4%), "Borrow" at the theoretical forward (via synthetic using curve at 3%). You receive 4%, pay ~3%, profit ~1% on notional.

5. **Slope**: 5%. Flat curve means $z'=0$, so $f = z + T \cdot 0 = z = 5\%$.

6. **Negative**: Simple: $z_s = (1/1.01 - 1)/1 = -0.99\%$. Continuous: $z_c = -\ln(1.01)/1 = -0.995\%$.

7. **Annuity**: $A = 1 \times 0.95 + 1 \times 0.90 = 1.85$.

8. **Spot-Forward**: $f(T) = (0.05+0.01T) + T(0.01) = 0.05 + 0.02T$.

9. **Convention**: $R_{sa} = 2(e^{0.05/2} - 1) = 2(1.0253 - 1) = 5.06\%$.

10. **Bootstrapping**: Yes. Step 1: Use 1y par rate $C_1 = (1-P_1)/(P_1)$ to solve for $P(1)$ (assuming annual, $P_1 = 1/(1+C_1)$). Step 2: Use 2y par equation: $1 = C_2 \cdot P_1 + C_2 \cdot P_2 + P_2 = C_2(P_1 + P_2) + P_2$. Rearrange: $P_2 = (1 - C_2 \cdot P_1)/(1 + C_2)$.

11. **Instantaneous Forward**: $f(0,T) = -\frac{d}{dT}(-0.04T - 0.002T^2) = 0.04 + 0.004T$.

12. **Integral Form**: $\int_0^3 f(0,u)\,du = \int_0^3 (0.03 + 0.01u)\,du = [0.03u + 0.005u^2]_0^3 = 0.09 + 0.045 = 0.135$. So $P(0,3) = e^{-0.135} = 0.8737$.

13. **Coupon Effect**: Convert to DFs: $P(1) = 1/1.01^2 = 0.9803$, $P(2) = 1/1.02^4 = 0.9238$, $P(3) = 1/1.025^6 = 0.8623$. 1y1y forward = $2[(0.9803/0.9238)^{0.5} - 1] = 6.02\%$. 1y2y forward = $[(0.9803/0.8623)^{0.5} - 1] \times 2 / 2 = 6.54\%$ (annualized). Forwards exceed spots because on steep curves, the marginal rate must exceed the average to pull it up.

14. **Trading Forwards**: Receive fixed at 4.5% on a 1y1y forward swap, $100mm notional. If realized rate is 3.8%, gain 70bp for 1 year: $100mm \times 0.007 \times 1 = \$700,000$ approximate P&L.

15. **Inverted Curve**: Par yield is **above** zero yield. On an inverted curve, early coupons are discounted at higher (short-term) rates, requiring a higher coupon to achieve par pricing.

---

## References

- Bruce Tuckman, *Fixed Income Securities* (discount factors; spot/forward rates; coupon effect).
- John C. Hull, *Options, Futures, and Other Derivatives* (zero/forward rates; FRAs; par yields; discounting discussion).
- David G. Luenberger, *Investment Science* (expectations hypothesis; liquidity preference intuition).
