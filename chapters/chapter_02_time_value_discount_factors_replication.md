# Chapter 2: Time Value of Money, Discount Factors, and Replication

---

## Introduction

A client asks you to price a 10-year zero-coupon bond. The trade seems simple—no coupons, just a single payment of $100 at maturity. But before you can quote a price, you must answer a question that sits at the heart of every fixed income calculation: *what is future money worth today?*

This question has consumed economists for centuries, and the answer—the **discount factor**—is the single most fundamental concept in fixed income. Get it right, and you can price any deterministic cash flow stream: bonds, swaps, loans, structured products. Get it wrong, and every valuation you produce becomes suspect.

To see why desk teams care: the PV of a single cash flow $N$ paid at time $T$ is $\text{PV} = N \cdot P(0,T)$. If rates shift and $P(0,T)$ moves, the dollars move. Under continuous compounding with $P(0,T)=e^{-z(T)T}$, a small change $\Delta z$ changes PV by about:
$$\Delta \text{PV} \approx -N \cdot T \cdot P(0,T) \cdot \Delta z$$
So a 1bp move ($\Delta z=0.0001$) on a 10-year $N=\$100\text{mm}$ cash flow can easily be a **tens-of-thousands of dollars** PV move. Multiply that across a trading book and you get the “mystery P&L” that product control spends nights reconciling.

This chapter establishes the foundational machinery for pricing deterministic cash flows:

1. **Discount factors**—the fundamental building blocks that convert future dollars into present value
2. **Present value**—the "pricing kernel" that values any stream of known cash flows
3. **The law of one price and replication**—why identical cash flows must have identical prices, and how arbitrage enforces this
4. **Treasury STRIPS**—zero-coupon securities that make discount factors directly observable
5. **Zero rates and forward rates**—alternative expressions of the time value of money
6. **Modern discounting**—why collateral determines the "risk-free" rate in today's markets
7. **Limits to arbitrage**—when theory meets the reality of balance sheet constraints

Chapter 1 established the conventions—day counts, compounding, settlement. This chapter uses those conventions to build the valuation framework. Together, they provide the foundation for everything that follows: bond pricing in Chapter 5, duration and convexity in Chapters 11-13, curve construction in Part IV, and the multi-curve framework in Chapter 19.

---

## 2.1 Discount Factors: The Price of Future Money

> **The Rosetta Stone of Rates: A Translation Guide**
>
> In fixed income, we often express the same economic reality using three different languages.
>
> 1. **Discount Factor $P(0,T)$**: The **Price** of a Zero-Coupon Bond.
>    - *Intuition:* "The Price Tag." ($0.95$)
> 2. **Zero Rate $z(T)$**: The **Yield** of a Zero-Coupon Bond.
>    - *Intuition:* "The Sticker Percentage." ($5\%$)
> 3. **Par Rate $y(T)$**: The **Coupon** of a Par Bond.
>    - *Intuition:* "The Periodic Payment Rate."
>
> They are mathematically equivalent—you can convert one to another—but they label the same item differently. We start with the **Discount Factor** because it is the most fundamental: it is a price.

### 2.1.1 Definition

Tuckman defines the discount factor precisely: "The discount factor for a particular term gives the value today, or the present value of one unit of currency to be received at the end of that term." We write $d(T)$ or equivalently $P(0,T)$ for the discount factor for maturity $T$.

If $d(0.5) = 0.97557$, then the present value of $1 to be received in six months is 97.557 cents. Equivalently, $d(0.5)$ is the price today of a zero-coupon bond paying $1 in six months.

This simple definition has powerful implications. If we know the discount factor for each future date, we can price any security with deterministic cash flows by multiplying each cash flow by its corresponding discount factor and summing them up.

> **Desk Reality: The "Exchange Rate in Time"**
>
> Think of a discount factor as an exchange rate—but instead of converting between currencies, it converts between *times*. Just as the EUR/USD rate tells you how many dollars you get for one euro, the discount factor $P(0,T)$ tells you how many dollars today you get for one dollar at time $T$.
>
> When a PM says "I need the 5-year discount factor," they're asking: "What's the exchange rate between today's dollars and 5-year dollars?" The answer might be 0.78—meaning a dollar in 5 years is worth 78 cents today.

### 2.1.2 Extracting Discount Factors from Bond Prices

Discount factors are not directly quoted in the market—they must be extracted from traded instruments. Tuckman demonstrates this process using Treasury bonds.

**Example from Tuckman (Table 1.1):** On February 15, 2001, the $7\frac{7}{8}$s of August 15, 2001, traded at $101\text{-}12\frac{3}{4}$ (i.e., $101 + 12.75/32 = 101.3984\%$ of face). This bond pays its final coupon plus principal in six months: $100 + 7.875/2 = 103.9375$. Therefore:

$$101.3984 = 103.9375 \times d(0.5)$$

Solving implies $d(0.5) = 0.97557$.

For a one-year bond—say, the $14\frac{1}{4}$s of February 15, 2002, trading at $108\text{-}31+$ (or $108.984375$)—the cash flows are $7.125$ in six months and $107.125$ in one year. The pricing equation is:

$$108.984375 = 7.125 \times d(0.5) + 107.125 \times d(1)$$

Since we already know $d(0.5) = 0.97557$, we can substitute it in:

$$108.984375 = 7.125 \times 0.97557 + 107.125 \times d(1)$$

$$108.984375 = 6.9509 + 107.125 \times d(1)$$

$$102.0334 = 107.125 \times d(1)$$

$$d(1) = 0.95247$$

### 2.1.3 Sequential Extraction (Bootstrapping Preview)

**Continuing sequentially**, each additional bond with a longer maturity provides one more equation to solve for one more discount factor. Tuckman shows that the five bonds in his Table 1.1 yield discount factors out to 2.5 years (Table 1.2):

| Maturity | Discount Factor |
|----------|-----------------|
| 0.5 | 0.97557 |
| 1.0 | 0.95247 |
| 1.5 | 0.93045 |
| 2.0 | 0.90796 |
| 2.5 | 0.88630 |

This sequential approach—solving for one discount factor at a time using known shorter-maturity factors—is the essence of **bootstrapping**, which we formalize in Chapter 17 on curve construction.

### 2.1.4 The Discount Factor Curve

The collection of discount factors across all maturities is called the **discount function** or discount factor curve. Tuckman emphasizes a key property: "Because of the time value of money, discount factors fall with maturity. The longer the payment of $1$ is delayed, the less it is worth today."

Figure 1.2 in Tuckman graphs this relationship, showing that $1 to be received in 30 years is worth only about 19 cents today when rates are around 5%. This dramatic decline illustrates why discounting matters so much for long-dated cash flows.

### 2.1.5 Discount Factors in Negative Rate Environments

Negative rates do not break the math. **When a (continuously compounded) zero rate is negative, the corresponding discount factor exceeds 1.** For example, if the one-year rate is -0.5%, then:

$$P(0,1) = e^{-(-0.005) \times 1} = e^{0.005} = 1.00501$$

**Economic meaning:** you would have to pay about $1.005 today to receive $1 in one year. In other words, “safe storage of value” can have a price; negative yields are one way markets express that price.

This matters operationally because some “sanity checks” built for positive-rate environments implicitly assume \(P(0,T)\le 1\).

> **Desk Reality: Operational Impacts of Negative Rates**
>
> Negative rates tend to break “folk assumptions” that are hard-coded into spreadsheets and risk systems, for example:
> - assuming \(P(0,T)\le 1\) or assuming discount factors are strictly decreasing;
> - assuming all coupon bonds price below par for low coupons;
> - sanity checks that flag negative forwards as “errors” even when they are economically possible.
>
> The practical lesson is to make your checks conditional on the regime: validate internal consistency first (repricing, curve smoothness, day count correctness), then worry about economic reasonableness.

### 2.1.6 Sanity Checks for Discount Factors

Before using any discount factor curve, verify:

1. **$P(0,0) = 1$**: A dollar today is worth a dollar today
2. **$P(0,T) > 0$**: Discount factors must be positive (no infinite rates)
3. **Monotonicity (positive rates)**: $P(0,T_1) > P(0,T_2)$ when $T_1 < T_2$
4. **Repricing**: Bonds used to extract discount factors should reprice exactly

---

## 2.2 Present Value: The Pricing Kernel for Deterministic Cash Flows

### 2.2.1 The Formula

Given a set of deterministic cash flows $\{C_1, C_2, \ldots, C_n\}$ paid at times $\{T_1, T_2, \ldots, T_n\}$, the present value is:

$$\boxed{\text{PV}_0 = \sum_{i=1}^{n} C_i \, P(0, T_i)}$$

This formula is the "pricing kernel" for all fixed cash-flow instruments. Hull states the principle simply: "The theoretical price of a bond can be calculated as the present value of all the cash flows that will be received by the owner of the bond... a more accurate approach is to use a different zero rate for each cash flow."

The formula works because each cash flow $C_i$ at time $T_i$ can be viewed as $C_i$ units of a zero-coupon bond maturing at $T_i$. The price of that zero is $P(0, T_i)$ per unit, so the price of $C_i$ units is $C_i \times P(0, T_i)$.

### 2.2.2 Linearity

> **Analogy: The Grocery Basket Principle**
>
> Pricing a bond is exactly like pricing a basket of groceries at the checkout counter.
> - **Bond** = 10 Coupons (Apples) + 1 Principal (Steak).
> - **Price** = $10 \times \text{Price}(\text{Apple}) + 1 \times \text{Price}(\text{Steak})$.
>
> You don't need a special "Basket Pricing Theory" to know the total. You just sum the value of the parts. That is exactly what the formula $\sum C_i \, P(0, T_i)$ does. It values each cash flow as a separate item using its specific price tag (the discount factor for that date).

Notice that present value is **linear** in the cash flows: doubling all cash flows doubles the present value. This linearity is fundamental—it means we can decompose complex securities into simpler pieces, value each piece independently, and add the results.

### 2.2.3 Unit Check and Sanity Checks

**Unit check:** $C_i$ is in currency units; $P(0, T_i)$ is dimensionless (currency per currency). The product is currency, and the sum is currency. Correct.

**Sanity checks:**
- If $C_k = 1$ and all other $C_i = 0$, then $\text{PV}_0 = P(0, T_k)$—the price of a zero.
- If all $T_i \to 0$, then $P(0, T_i) \to 1$ and $\text{PV}_0 \to \sum_i C_i$—immediate cash is undiscounted.
- With positive rates, if discount factors fall, PV falls—consistent with rates moving inversely to prices.

---

## 2.3 The Law of One Price and No-Arbitrage

### 2.3.1 The Economic Principle

Tuckman states the law of one price: "Absent confounding factors (e.g., liquidity, special financing rates, taxes, credit risk), two securities (or portfolios of securities) with exactly the same cash flows should sell for the same price."

The intuition is straightforward: "An investor should not care whether $1 on a particular date comes from one bond or another." If two instruments produce identical future cash flows, why would anyone pay more for one than the other?

This principle has immediate practical implications. Once you extract discount factors from one set of bonds, you can use those factors to price *any* other bond with cash flows on the same dates. Tuckman notes that "discount factors extracted from one set of bonds may be used to price any other bond with cash flows on the same set of dates."

### 2.3.2 Confounding Factors

The law of one price holds "absent confounding factors." In practice, several factors can cause prices to deviate:

| Factor | Effect |
|--------|--------|
| **Liquidity / benchmark status** | More liquid “benchmark” bonds can trade rich (higher prices / lower yields) |
| **Special financing (repo specials)** | Bonds in high demand for shorting may trade rich |
| **Taxes** | Different tax treatment affects after-tax returns |
| **Credit risk** | Different default/recovery expectations change required yields (relevant outside government benchmarks) |
| **Balance sheet costs** | Holding positions requires capital and funding |

When prices deviate from the law of one price, practitioners describe securities as trading **rich** (price above fair value) or **cheap** (price below fair value). These deviations create relative value opportunities—though exploiting them requires understanding *why* they exist.

> **Desk Reality: Rich and Cheap in Practice**
>
> When a trader says "the 10-year is trading 3 basis points cheap to the curve," they mean:
> 1. The bond's market yield is 3bp higher than what the fitted curve implies
> 2. The market price is *below* the model price (cheap = low price = high yield)
> 3. There might be a reason (illiquidity, supply pressure, repo dislocation)
>
> Traders don't automatically buy "cheap" bonds. They ask: *why* is it cheap? If it's cheap because of temporary technical factors (auction supply, index rebalancing), they might buy. If it's cheap because it's hard to finance in repo, the "cheapness" might persist indefinitely.

---

## 2.4 Arbitrage and Replication

### 2.4.1 What Is Arbitrage?

Tuckman defines arbitrage as "a trade that generates or that might generate profits without any risk." More formally, an arbitrage opportunity is a trading strategy that:

$$\boxed{\text{Arbitrage: Initial Cost} \leq 0, \; \text{Payoff} \geq 0, \; P[\text{Payoff} > 0] > 0}$$

In words: you pay nothing (or receive cash) upfront, you never lose money, and there's a positive probability of making money. This is a "free lunch"—and in efficient markets, free lunches get eaten quickly.

The existence of arbitrage opportunities is what enforces the law of one price. If two portfolios with identical cash flows traded at different prices, arbitrageurs would buy the cheap one and sell the expensive one, earning a risk-free profit. This trading pressure would quickly eliminate the price difference.

### 2.4.2 Replication: Constructing Identical Cash Flows

**Replication** means constructing a portfolio that produces the same cash flows as a target security. Under no-arbitrage, the target must be priced at the cost of its replicating portfolio.

Tuckman provides a detailed example in his Appendix 1A and Table 1.4. The $10\frac{3}{4}$s of February 15, 2003, with its four remaining cash flows ($5.375$ at each of 0.5, 1.0, 1.5 years, and $105.375$ at 2.0 years), can be replicated by a portfolio of four bonds from Table 1.1.

The key insight of replication is that we can match cash flows at each date by working backward from the final payment.

> **Replication in Action: Matching the Cash Flows**
>
> To replicate a hypothetical bond paying $50 at 0.5y and $1050 at 1.0y, using zeros:
>
> | Maturity | Target Bond CF | Bond A (0.5y Zero) | Bond B (1.0y Zero) | **Portfolio Net CF** |
> | :--- | :--- | :--- | :--- | :--- |
> | **0.5y** | $50 | $50 (50 units) | $0 | **$50** |
> | **1.0y** | $1050 | $0 | $1050 (1050 units) | **$1050** |
>
> *Result:* The Portfolio Net CF column matches the Target Bond CF column exactly. Therefore, the price of the portfolio today **must** equal the price of the target bond.

### 2.4.3 Replication Example: Two Methods Must Agree

**Method 1: Discount Factor Valuation**

Using the discount factors extracted in Section 2.1:

$$V_0 = 5.375 \times 0.97557 + 5.375 \times 0.95247 + 5.375 \times 0.93045 + 105.375 \times 0.90796$$

$$V_0 = 5.244 + 5.120 + 5.001 + 95.676 = 111.041$$

**Method 2: Replicating Portfolio Cost**

The replicating portfolio constructed from the Table 1.1 bonds has a total cost of 111.041 (computed by summing the portfolio weights times bond prices).

**The two methods produce identical values.** This is not a coincidence—it is guaranteed by the law of one price. If the discount factors were extracted correctly from the bond prices, both methods are mathematically equivalent.

### 2.4.4 Identifying Rich/Cheap Securities

Tuckman notes that the $10\frac{3}{4}$s actually traded at 110.938 (from Table 1.4), which is below the replication value of 111.041. The bond is therefore **cheap** by $0.103$ (or about 3.3 32nds).

An arbitrageur could exploit this by:
1. **Buy** the $10\frac{3}{4}$s at 110.938
2. **Sell** the replicating portfolio at 111.041
3. **Pocket** $0.103$ today with zero net future cash flows

Tuckman observes: "Absent any confounding factors, arbitrageurs would do as much of this trade as possible... the price of the $10\frac{3}{4}$s would be driven up and/or the price of the replicating portfolio would be driven down."

### 2.4.5 Static vs. Dynamic Replication

For deterministic cash flows, replication is **static**: once you construct the replicating portfolio, you hold it to maturity. The portfolio automatically produces the same cash flows as the target.

For instruments with optionality or rate-dependent cash flows (covered in later chapters on derivatives), replication becomes **dynamic**: the portfolio must be adjusted continuously as rates change.

---

## 2.5 Treasury STRIPS: Directly Observable Discount Factors

### 2.5.1 What Are STRIPS?

Tuckman explains that Treasury STRIPS (Separate Trading of Registered Interest and Principal of Securities) are zero-coupon securities created by stripping the individual cash flows from Treasury notes and bonds. When a Treasury is stripped, each coupon payment and the principal payment become separate, tradeable securities.

For example, a 10-year Treasury note with semiannual coupons can be stripped into 21 separate securities: 20 coupon strips (one for each semiannual coupon) and 1 principal strip (the final $100 payment).

### 2.5.2 C-STRIPS vs. P-STRIPS

Tuckman distinguishes two types:

- **C-STRIPS (Coupon strips)**: Created from the coupon payments. Since many bonds have coupons on the same date (e.g., February 15 and August 15), C-STRIPS are fungible—a coupon strip from one bond is interchangeable with a coupon strip of the same maturity date from another bond.

- **P-STRIPS (Principal strips)**: Created from the principal payment. Each P-STRIP is unique to the bond it came from and is not fungible with other P-STRIPS. Tuckman notes: "P-STRIPS are identified with particular bonds: P-STRIPS created from the stripping of a particular bond may be used to reconstitute only that bond. This difference implies that P-STRIPS inherit the cheapness or richness of the bonds from which they are derived."

### 2.5.3 STRIPS and Discount Factors

Because STRIPS are true zero-coupon securities, their prices directly reveal discount factors:

$$P(0, T) = \frac{\text{STRIP Price}}{100}$$

For example, if an August 15, 2031 C-STRIP trades at 63.50, then:

$$P(0, 10.5) = 0.6350$$

This direct observability makes STRIPS valuable for curve construction. However, Tuckman notes that STRIPS prices may not perfectly match discount factors implied by coupon bonds due to:

- **Liquidity differences**: STRIPS are less liquid than the underlying bonds
- **Reconstitution option**: The ability to recombine STRIPS into whole bonds creates pricing relationships
- **Tax treatment**: Zero-coupon bonds have phantom income (imputed interest) that may be taxed annually

### 2.5.4 STRIPS as Building Blocks

Tuckman emphasizes that "any Treasury can be regarded as a portfolio of STRIPS." A 2-year Treasury with semiannual coupons of $C$ and principal of $100$ is equivalent to:
- $C$ units of the 0.5-year strip
- $C$ units of the 1.0-year strip
- $C$ units of the 1.5-year strip
- $(C + 100)$ units of the 2.0-year strip

This decomposition is the economic foundation of the present value formula.

### 2.5.5 Stripping and Reconstitution: The Arbitrage Mechanism

The Treasury permits both **stripping** (breaking a bond into its component STRIPS) and **reconstitution** (reassembling STRIPS back into a whole bond). This creates an arbitrage mechanism that keeps STRIPS prices consistent with whole bond prices.

**Stripping arbitrage**: If the sum of STRIPS prices exceeds the whole bond price, professionals can:
1. Buy the whole bond
2. Strip it into components
3. Sell the STRIPS at a profit

**Reconstitution arbitrage**: If the whole bond trades above the sum of its STRIPS, professionals can:
1. Buy the required STRIPS
2. Reconstitute the whole bond
3. Sell the bond at a profit

Tuckman observes: "At times these professionals find it profitable to buy coupon bonds, strip them, and then sell the STRIPS. At other times these professionals find it profitable to buy the STRIPS, reconstitute them, and then sell the bonds."

> **Desk Reality: Why Whole Bonds Sometimes Trade Rich**
>
> When investors prefer to hold whole bonds rather than STRIPS (perhaps for liquidity, benchmark eligibility, or operational simplicity), the whole bond can trade rich to its theoretical value. This creates "negative stripping value"—it costs more to buy the bond than to synthesize it from STRIPS.
>
> The persistence of this richness reflects the *value of convenience*. A bond that's benchmark-eligible, liquid in repo, and easy to settle is worth more than a synthetic equivalent that's operationally burdensome.

---

## 2.6 Zero (Spot) Rates: An Alternative Expression

### 2.6.1 Definition

While discount factors directly answer "what is $1 at time $T$ worth today?", investors often prefer to express the time value of money as *rates*. The **zero rate** (or **spot rate**) $z(T)$ is the rate that, applied to $1$ for $T$ years, produces a future value equal to $1/P(0, T)$.

The relationship depends on the compounding convention.

**Continuous compounding** (used extensively in derivatives):

$$\boxed{P(0,T) = e^{-z(T) \cdot T}}$$

Inverting to solve for the rate:

$$\boxed{z(T) = -\frac{1}{T} \ln P(0,T)}$$

> **Mental Math Tip: The Rule of 72**
>
> A quick way to estimate doubling time or effective rates:
> $$\text{Rate} \times \text{Years} \approx 72 \implies \text{Doubling}$$
>
> - If the interest rate is 6%, it takes $\approx 12$ years to double your money ($6 \times 12 = 72$).
> - In discount factor terms: If $z = 6\%$, then $P(0, 12) \approx 0.50$.
> - (Exact check: $e^{-0.06 \times 12} = e^{-0.72} \approx 0.487$. Close enough for mental math!)

**Semiannual compounding** (used for U.S. Treasury yields):

$$P(0,T) = \frac{1}{\left(1 + \frac{\hat{r}(T)}{2}\right)^{2T}}$$

where $\hat{r}(T)$ denotes the semiannually compounded spot rate.

Hull explains the distinction clearly: "Compounding a sum of money at a continuously compounded rate $R$ for $n$ years involves multiplying it by $e^{Rn}$. Discounting it at a continuously compounded rate $R$ for $n$ years involves multiplying by $e^{-Rn}$."

### 2.6.2 Why Continuous Compounding?

Hull advocates for continuous compounding in derivatives pricing: "Continuously compounded interest rates are used to such a great extent in pricing derivatives that it makes sense to get used to working with them now."

Continuous compounding simplifies the algebra of time and rates. Combining rates over sequential periods amounts to averaging them. If you earn $r_1$ for the first year and $r_2$ for the second year (both continuously compounded), your average rate over two years is simply $(r_1 + r_2)/2$. This additivity makes continuous compounding particularly elegant for forward rate calculations.

### 2.6.3 Conversion Formulas

Hull provides the conversion between continuous and $m$-times-per-year compounding:

$$\boxed{R_c = m \ln\left(1 + \frac{R_m}{m}\right)}$$

$$\boxed{R_m = m\left(e^{R_c/m} - 1\right)}$$

**Example (from Hull):** A rate of 10% with semiannual compounding converts to continuous as:

$$R_c = 2 \ln(1 + 0.10/2) = 2 \ln(1.05) = 0.09758 = 9.758\%$$

The continuous rate is lower because continuous compounding "works harder"—interest compounds more frequently, so a lower rate achieves the same final value.

---

## 2.7 Forward Rates: Locking in Future Borrowing Costs

### 2.7.1 What Is a Forward Rate?

A **forward rate** is the interest rate on a loan that starts at some future date, with the rate agreed upon today. Tuckman explains: "A forward loan is an agreement made to lend money at some future date. The rate of interest on a forward loan, specified at the time of the agreement as opposed to the time of the loan, is called a forward rate."

### 2.7.2 The Investor's Choice: Spot vs. Forward

Tuckman illustrates forward rates through an investor's decision. Suppose an investor wants to invest $1,000 for one year. Using Tuckman's February 15, 2001 data, two strategies are available:

**Strategy 1: Buy the one-year zero**
- Invest $1,000 at the one-year spot rate of 4.93% (semiannually compounded)
- Terminal value: $1,000 \times (1 + 0.0493/2)^2 = $1,050.01$

**Strategy 2: Roll over six-month zeros**
- Invest $1,000 for six months at 5.01%
- Terminal value after six months: $1,000 \times (1 + 0.0501/2) = $1,025.05$
- Reinvest for another six months at the then-prevailing rate

The key question is: what six-month rate in six months makes these strategies equivalent?

### 2.7.3 The Forward Rate Calculation

For strategies to be equivalent, the forward rate $f(0; 0.5, 1.0)$ must satisfy:

$$(1 + 0.0501/2)(1 + f/2) = (1 + 0.0493/2)^2$$

Solving:

$$1.02505 \times (1 + f/2) = 1.05001$$

$$1 + f/2 = 1.02436$$

$$f = 4.87\%$$

Tuckman notes this result: "The forward rate, at 4.87%, is below the six-month rate of 5.01%. This makes sense. To choose between investing in the six-month zero versus the one-year zero, an investor in February 2001 should have compared his view of the six-month rate in six months with the break-even rate of 4.87%."

### 2.7.4 Forward Rates from Spot Rates (Continuous Compounding)

With continuous compounding, the relationship is particularly clean. If $z(T_1)$ and $z(T_2)$ are the continuously compounded spot rates to times $T_1$ and $T_2$, the continuously compounded forward rate $f(0; T_1, T_2)$ for the period $[T_1, T_2]$ satisfies:

$$\boxed{f(0; T_1, T_2) = \frac{z(T_2) \cdot T_2 - z(T_1) \cdot T_1}{T_2 - T_1}}$$

This formula essentially says that the forward rate is the marginal rate required to bridge the gap between the return at $T_1$ and the return at $T_2$.

### 2.7.5 Forward Rates from Discount Factors (Simple Compounding)

For FRA-style forward rates (Forward Rate Agreements) which typically use simple compounding:

$$\boxed{F(0; T_1, T_2) = \frac{1}{\tau(T_1, T_2)} \left(\frac{P(0, T_1)}{P(0, T_2)} - 1\right)}$$

where $\tau(T_1, T_2)$ is the year fraction between $T_1$ and $T_2$.

**Intuition:** The ratio $P(0, T_1)/P(0, T_2)$ represents the growth factor from $T_1$ to $T_2$—how much $1$ at $T_1$ grows to by $T_2$, implied by today's discount factors. Subtracting 1 gives the total interest earned over the period, and dividing by $\tau$ annualizes it to a simple rate.

### 2.7.6 Locking In the Forward Rate: A Worked Example

The forward rate is not just an abstract calculation—it can be *locked in* using today's bonds. Here's how:

**Goal:** Lock in a borrowing rate for the period from $T_1 = 0.5$ to $T_2 = 1.0$.

**Strategy:**
1. **Sell** $P(0, T_1)/P(0, T_2)$ units of the $T_2$-maturity zero
2. **Buy** 1 unit of the $T_1$-maturity zero

**Cash flows:**

| Time | From $T_1$ Zero | From $T_2$ Zero | Net |
|------|-----------------|-----------------|-----|
| $t=0$ | $-P(0,T_1)$ | $+\frac{P(0,T_1)}{P(0,T_2)} \times P(0,T_2) = P(0,T_1)$ | **$0$** |
| $t=T_1$ | $+1$ | $0$ | **$+1$** |
| $t=T_2$ | $0$ | $-\frac{P(0,T_1)}{P(0,T_2)}$ | **$-\frac{P(0,T_1)}{P(0,T_2)}$** |

**Interpretation:** You pay nothing today, receive $1$ at $T_1$, and repay $P(0,T_1)/P(0,T_2)$ at $T_2$. This is economically equivalent to borrowing $1$ at $T_1$ and repaying principal plus interest at $T_2$. The implied interest rate is exactly the forward rate.

> **Desk Reality: FRAs and Forward-Starting Swaps**
>
> This lock-in mechanism is the theoretical foundation of Forward Rate Agreements (FRAs) and forward-starting swaps. When a corporate treasurer wants to hedge future borrowing costs, they're effectively executing this strategy through a derivatives contract.
>
> The forward rate is the breakeven rate implied by today’s discount factors: it is the rate you can lock in today for that future period. Realized future spot rates can differ.

### 2.7.7 Forward Rates and the Spot Rate Curve

Tuckman observes an important geometric relationship: "When the forward rate curve is above the spot rate curve, the spot rate curve is rising or sloping upward. But, when the forward rate curve is below the spot rate curve, the spot rate curve slopes downward or is falling."

The intuition is that each spot rate is an average of all forward rates up to that maturity. If the next forward rate is above the current average (the spot rate), the new average must rise. If the next forward rate is below, the new average must fall.

---

## 2.8 Modern Discounting: From LIBOR to OIS

### 2.8.1 The Risk-Free Rate Question

The discount factor framework assumes we know the "risk-free" rate. But what rate is truly risk-free? Before 2008, practitioners used **LIBOR** (the London Interbank Offered Rate) as a proxy for the risk-free rate. Hull explains: "Prior to the 2007-8 credit crisis, LIBOR was used as a proxy for the risk-free rate when valuing derivatives."

The 2008 financial crisis revealed that LIBOR was not risk-free—it embedded significant bank credit risk. When Lehman Brothers failed, the spread between LIBOR and Treasury rates widened dramatically, showing that LIBOR reflected bank credit conditions, not pure time value.

### 2.8.2 OIS: The Collateralized Benchmark

Hull explains the shift: "The OIS rate is increasingly regarded as a better measure of the risk-free rate than LIBOR... The overnight rate underlying the OIS is considered to have very little credit risk."

An **Overnight Index Swap (OIS)** exchanges a fixed rate for a reference rate calculated from realized overnight rates (compounded over the payment period). The OIS rate is the fixed rate that makes the swap have zero value at inception.

The practical motivation is that overnight-based benchmarks have been treated as closer proxies to “risk‑free” discounting than term unsecured benchmarks like LIBOR.

### 2.8.3 Collateral and the CSA

Hull provides the economic rationale for OIS discounting: "When a derivative is fully collateralized with variation margin being posted daily, it should be discounted at the OIS rate."

A **Credit Support Annex (CSA)** specifies the collateral arrangements for an OTC derivative:
- **Variation margin**: Daily cash posted to cover mark-to-market changes
- **Eligible collateral**: What assets can be posted (cash, government bonds, etc.)
- **Thresholds**: Minimum amounts before collateral is required

When derivatives are fully collateralized:
- **The "winner" receives collateral** equal to the derivative's value
- **The collateral earns interest** (typically at the overnight rate)
- **Credit exposure is nearly eliminated** (assuming daily margin calls)

Since the collateral earns the overnight rate, **the appropriate discount rate for a collateralized derivative is the overnight rate**—not LIBOR or any other rate. This is the economic foundation of OIS discounting.

Using the wrong discounting assumptions is a common source of valuation differences between systems. In practice, you need to know the collateral terms (CSA/clearing) before you can say what “risk‑free discounting” means for a given trade.

> **Desk Reality: “Which Curve Are You Using?”**
>
> A frequent source of P&L breaks between front office, risk, and product control is that different systems silently use different discounting assumptions. A useful mental model is:
>
> | Trade context | Common discounting anchor (high level) |
> |---|---|
> | Fully collateralized OTC (daily VM) | Overnight/OIS-type curve consistent with collateral rate |
> | Cleared derivatives | Clearinghouse margin/discounting conventions (often OIS-based) |
> | Uncollateralized / weakly collateralized | Credit/funding adjustments become material (CVA/FVA, etc.) |
>
> The point is not that there is one universal curve, but that **discounting is a contract term**. Before arguing about PV differences, confirm the CSA/clearing setup and the curve configuration in each system.

### 2.8.4 Multi-Curve Framework Preview

The shift to OIS discounting created the **multi-curve framework**: we now use *different curves* for discounting and projecting future cash flows.

Conceptually:
- **Discount curve**: used to discount cash flows to PV.
- **Projection curve**: used to generate forward-looking expectations for floating cash flows.

Before the crisis, many practitioners used the same curve for both roles; in modern practice, separating them is common. We return to this in Chapter 19.

---

## 2.9 Limits to Arbitrage: When Theory Meets Reality

### 2.9.1 The Idealized vs. Real Arbitrage

The textbook arbitrage is frictionless: if prices diverge, arbitrageurs instantly exploit the gap, and prices snap back. Reality is messier. Tuckman observes: "The fact that the market price of the $10\frac{3}{4}$s remains below that of the replicating portfolio strongly indicates that some set of confounding factors inhibits arbitrage activity."

Why does apparent mispricing persist? Because **real arbitrage is not free**.

### 2.9.2 Why “Arbitrage” Can Persist

Even when two portfolios have the *same* contractual cash flows on paper, real-world frictions can keep prices apart. Common examples include:

- **Funding / financing costs** (including repo economics)
- **Liquidity and market impact**
- **Balance-sheet / capital constraints**
- **Counterparty, settlement, and operational constraints**

For this book, the important message is: replication gives a benchmark price in an idealized world; in practice you must ask what frictions are embedded in the traded prices you calibrate to.

> **Desk Reality: “Near-Arbitrage” Still Has Risk**
>
> A trade can be “arbitrage” in cash-flow space and still have meaningful P&L volatility because:
> - convergence timing is uncertain;
> - funding, margin, and repo terms can change;
> - the hedge you use is not perfectly aligned with the instrument you own.
>
> This is why desk risk limits (VaR, stress, drawdown) matter even for relative-value strategies that are “supposed to converge.”

---

## 2.10 PV Sensitivity: A First Look at Rate Risk

How does present value change when rates move?

For deterministic cash flows priced using continuously compounded zero rates:

$$\text{PV}_0 = \sum_i C_i \, e^{-z(T_i) \cdot T_i}$$

Taking the derivative with respect to a small change in a specific zero rate $z(T_i)$:

$$\frac{\partial \text{PV}_0}{\partial z(T_i)} = -T_i \cdot C_i \cdot e^{-z(T_i) \cdot T_i} = -T_i \cdot C_i \cdot P(0, T_i)$$

Therefore, for small parallel shifts in the zero curve:

$$\boxed{\Delta \text{PV}_0 \approx -\sum_i C_i \, T_i \, P(0, T_i) \, \Delta z}$$

**Interpretation:** Longer-dated cash flows (larger $T_i$) have greater PV sensitivity to rate changes. This relationship is the economic foundation of **duration**, which quantifies interest rate risk and is formalized in Chapter 11.

---

## 2.11 Worked Examples

### Example A: Building Discount Factors from Money-Market Quotes

**Goal:** Given deposit-style annualized rates, construct discount factors and compute implied zero and forward rates.

**Inputs:** We observe three money-market rates (simple interest over the accrual fraction):

| Maturity | Rate (Simple) |
|----------|---------------|
| 3M (0.25Y)| 4.80% |
| 6M (0.50Y)| 5.00% |
| 12M (1.00Y)| 5.30% |

Using year fractions $\tau = T$ and the simple interest convention, a deposit of $1$ grows to $1 + R_T \cdot T$ at maturity.

**Step 1: Discount Factors**
The discount factor is the reciprocal of the growth factor: $P(0,T) = \frac{1}{1 + R_T \cdot T}$.

| Maturity | Calculation | Discount Factor |
|----------|-------------|-----------------|
| 0.25 | $1/(1 + 0.048 \times 0.25)$ | 0.98814 |
| 0.50 | $1/(1 + 0.050 \times 0.50)$ | 0.97561 |
| 1.00 | $1/(1 + 0.053 \times 1.00)$ | 0.94967 |

**Step 2: Continuously Compounded Zero Rates**
Using $z(T) = -\frac{1}{T} \ln P(0,T)$:

| Maturity | Zero Rate (continuous) |
|----------|------------------------|
| 0.25 | 4.77% |
| 0.50 | 4.94% |
| 1.00 | 5.16% |

**Step 3: Forward Rate from 6M to 12M**
Simple (FRA-style):
$$F(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.97561}{0.94967} - 1\right) = 2 \times (1.02731 - 1) = 5.46\%$$

Continuous:
$$f(0; 0.5, 1.0) = \frac{0.0516 \times 1.0 - 0.0494 \times 0.5}{1.0 - 0.5} = \frac{0.0516 - 0.0247}{0.5} = 5.38\%$$

**Sanity checks:**
- Discount factors are positive and decrease with maturity ($0.988 > 0.975 > 0.949$). ✓
- Zero rates increase with maturity (upward-sloping curve). ✓
- The forward rate (5.46%) exceeds the six-month spot rate (5.00%)—consistent with the rising spot curve. ✓

---

### Example B: Pricing a Coupon Instrument

**Goal:** Price a 1-year note paying a 6% annual coupon semiannually.

**Cash flows:**
- Time 0.5: Coupon of $3 (6%/2 × 100)
- Time 1.0: Coupon + Principal of $103

**Calculation:**
Using the discount factors from Example A:

$$\text{PV}_0 = 3 \times 0.97561 + 103 \times 0.94967$$
$$\text{PV}_0 = 2.927 + 97.816 = \boxed{100.74}$$

**Interpretation:** The bond is worth 100.74 per 100 face. Since this exceeds par, the coupon rate (6%) must be higher than the prevailing market yield (which we saw was around 5-5.3%).

**Sanity check:** If rates were zero, PV would be $3 + 103 = 106$. With positive rates, PV must be less than 106. Determining it is $100.74$ passes this check. ✓

---

### Example C: Replication and No-Arbitrage Pricing

**Goal:** Price a claim paying 50 at $T = 0.5$ and 105 at $T = 1.0$ by constructing a replicating portfolio.

**Replicating portfolio:**
- 50 units of the 6-month zero (each pays $1 at 0.5)
- 105 units of the 1-year zero (each pays $1 at 1.0)

This portfolio produces exactly the claim's cash flows.

**No-arbitrage price:**
Cost of replication = $50 \times P(0, 0.5) + 105 \times P(0, 1.0)$
$$V_0 = 50 \times 0.97561 + 105 \times 0.94967 = 48.78 + 99.72 = \boxed{148.50}$$

**Arbitrage thought experiment:** If the claim traded at 149.50 while replication costs 148.50, you could:
1. Sell the claim at 149.50 (inflow)
2. Buy the replicating portfolio at 148.50 (outflow)
3. Pocket $1.00$ today
4. At $T=0.5$ and $T=1.0$, the portfolio payoffs exactly cover the claim liabilities.

You have locked in a risk-free profit of $1.00$ with zero net future obligations. Market forces (arbitrageurs executing this trade) would drive the claim price down and/or the component prices up until the gap closes.

---

### Example D: STRIPS Pricing Check

**Goal:** Verify consistency between STRIPS prices and discount factors.

**Given:** From Tuckman's Table 1.2, $d(1.0) = 0.95247$.

**Expected STRIPS price:** A 1-year C-STRIP should trade at $95.247$ per $100 face.

**Verification:** If the STRIPS traded at $95.50$, it would be **rich** by $0.253$. Arbitrageurs could:
1. Strip a coupon bond to create the synthetic zero
2. Sell the C-STRIP at $95.50$
3. Pocket the difference

In practice, STRIPS often trade slightly rich or cheap due to liquidity and tax effects, but large deviations are arbitraged away.

---

### Example E: Locking in a Forward Rate

**Goal:** Show how to lock in the 6M×12M forward rate using today's bonds.

**Given:** $P(0, 0.5) = 0.97561$, $P(0, 1.0) = 0.94967$

**Forward rate:** $F(0; 0.5, 1.0) = 5.46\%$

**Lock-in strategy:**
1. Sell $0.97561/0.94967 = 1.0273$ units of the 1-year zero
2. Buy 1 unit of the 6-month zero

**Cash flow verification:**

| Time | 6M Zero | 1Y Zero | Net |
|------|---------|---------|-----|
| Today | $-0.97561$ | $+1.0273 \times 0.94967 = +0.97561$ | **$0$** |
| 6M | $+1$ | $0$ | **$+1$** |
| 12M | $0$ | $-1.0273$ | **$-1.0273$** |

**Interpretation:** You pay $0$ today, receive $1$ at 6M, and repay $1.0273$ at 12M. The implied interest rate for the 6-month period is:

$$\frac{1.0273 - 1}{0.5} = 5.46\%$$

This exactly equals the forward rate—confirming the lock-in works.

---

## 2.12 Practical Notes

### Common Pitfalls

| Issue | Description |
|-------|-------------|
| **Compounding mismatch** | A "5%" rate means different things under different compounding (e.g., continuous vs. semiannual). Always verify the convention. |
| **Day count ignored** | Discount factor extraction depends on year fractions $\tau$, which depend on day counts (e.g., ACT/360 vs. 30/360). |
| **Wrong discount curve** | Using LIBOR discounting for collateralized trades is now incorrect—use OIS. |
| **Interpolation deferred** | Market quotes don't provide discount factors at every single date; interpolation methods (Chapter 17) fill the gaps. |
| **Confounding factors ignored** | Rich/cheap analysis requires understanding *why* prices deviate from the model. Is it liquidity? Credit? |
| **STRIPS vs. synthetic zeros** | STRIPS prices may differ from discount factors implied by coupon bonds due to liquidity and tax differences. |
| **Ignoring balance sheet costs** | A "profitable" arbitrage may be unprofitable after funding, capital, and operational costs. |

### Verification Tests

**Boundary checks:**
- $P(0, 0) = 1$ (A dollar today is worth a dollar today)
- $P(0, T) > 0$ for all sensible finite rates.

**Monotonicity (positive rates):**
- $P(0, T)$ should decrease as $T$ increases.

**Arbitrage test:**
- Reprice calibration instruments using your derived curve: the model PV must match the market price exactly.
- Two portfolios with identical cash flows must have identical model PVs.

---

## Summary

This chapter established the fundamental pricing framework for deterministic cash flows:

1. **Discount factors** $P(0, T)$ give the present value of $1 at time $T$. They can be extracted from bond prices or money market rates by solving pricing equations sequentially.

2. **Present value** is computed as $\text{PV}_0 = \sum_i C_i P(0, T_i)$—multiply each cash flow by its discount factor and sum. This is the "pricing kernel" for fixed cash flows.

3. **The law of one price** states that identical cash flows must have identical prices, absent confounding factors (liquidity, repo specials, taxes, credit, balance sheet).

4. **Replication** constructs a portfolio with the same cash flows as a target. Under no-arbitrage, the target's price must equal the replication cost.

5. **Treasury STRIPS** are zero-coupon securities that make discount factors directly observable, though liquidity and tax effects can cause small deviations.

6. **Zero rates** are an alternative expression: $P(0, T) = e^{-z(T) \cdot T}$ under continuous compounding. They summarize the discount factor curve.

7. **Forward rates** are implied future borrowing costs that can be locked in using today's bonds. They link spot rates at adjacent maturities.

8. **Modern discounting** uses OIS rates for collateralized derivatives. The CSA determines which rate is appropriate—a material distinction post-2008.

9. **Limits to arbitrage** mean that theoretical mispricings can persist due to balance sheet constraints, funding costs, and liquidity gaps. Real arbitrage is never truly "free."

10. **Rate sensitivity** reveals that longer-dated cash flows are more sensitive to rate changes—this is the foundation of duration.

The key conceptual insight: **arbitrage and replication turn pricing into a mechanical step**. If you can match a security's cash flows using traded instruments, the law of one price pins down its value. But the real world adds friction—and understanding those frictions separates textbook pricing from desk-level reality.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Discount factor $P(0,T)$ | PV today of $1 at time $T$ | The fundamental building block for pricing. |
| Present value | $\sum_i C_i P(0,T_i)$ | The price of any deterministic cash-flow stream. |
| Law of one price | Same cash flows → same price | Justifies using one curve to price all instruments. |
| Replication | Portfolio matching target's cash flows | Provides the proof for the no-arbitrage price. |
| STRIPS | Zero-coupon securities from stripped Treasuries | Make discount factors directly observable. |
| Zero rate $z(T)$ | Rate such that $P(0,T) = e^{-z(T)T}$ | A convenient summary of the discount curve. |
| Forward rate | Implied future borrowing rate | Expresses curve slope; can be locked in today. |
| OIS discounting | Using overnight rates for collateralized PV | The post-2008 standard for derivatives. |
| CSA | Credit Support Annex specifying collateral terms | Determines which discount curve to use. |
| Limits to arbitrage | Constraints preventing perfect price enforcement | Balance sheet, funding, liquidity, counterparty. |

---

## Notation for This Book

| Symbol | Definition |
|--------|------------|
| $P(0,T)$ or $d(T)$ | Discount factor for maturity $T$ |
| $z(T)$ | Continuously compounded zero rate to $T$ |
| $\hat{r}(T)$ | Semiannually compounded spot rate to $T$ |
| $f(0; T_1, T_2)$ | Continuously compounded forward rate for $[T_1, T_2]$ |
| $F(0; T_1, T_2)$ | Simply compounded (FRA-style) forward rate |
| $\tau(T_1, T_2)$ | Year fraction between $T_1$ and $T_2$ |
| $\text{PV}_0$ | Present value at time 0 |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is $d(T)$ or $P(0,T)$? | The present value today of $1 received at time $T$. |
| 2 | How do you price deterministic cash flows $\{(T_i, C_i)\}$? | $\sum_i C_i P(0, T_i)$ |
| 3 | What is the law of one price? | Identical cash flows imply identical prices, absent confounding factors. |
| 4 | What are examples of confounding factors? | Liquidity, special financing (repo specials), taxes, credit risk, balance sheet costs. |
| 5 | What is $P(0,T)$ economically? | The price of a zero-coupon bond paying $1 at $T$. |
| 6 | Under continuous compounding, what is the PV of $A$ at $T$ discounted at rate $R$? | $A e^{-RT}$ |
| 7 | Define the continuously compounded zero rate $z(T)$. | The rate such that $P(0,T) = e^{-z(T)T}$. |
| 8 | How do you extract $z(T)$ from $P(0,T)$? | $z(T) = -\frac{1}{T} \ln P(0,T)$ |
| 9 | What is the continuous forward rate formula from zero rates? | $f = \frac{z(T_2) T_2 - z(T_1) T_1}{T_2 - T_1}$ |
| 10 | What is the FRA-style forward rate from discount factors? | $F = \frac{1}{\tau}\left(\frac{P(0,T_1)}{P(0,T_2)} - 1\right)$ |
| 11 | What is replication? | Constructing a portfolio with the same cash flows as the target security. |
| 12 | If a claim's market price differs from replication cost, what exists? | An arbitrage opportunity (buy cheap, sell rich). |
| 13 | Why do discount factors fall with maturity (positive rates)? | Money received sooner can earn interest; delayed money is worth less. |
| 14 | Convert 10% semiannual to continuous compounding. | $R_c = 2 \ln(1.05) = 9.76\%$ |
| 15 | When is the spot rate curve upward-sloping? | When forward rates exceed spot rates. |
| 16 | What are Treasury STRIPS? | Zero-coupon securities created by stripping coupon and principal payments from Treasury bonds. |
| 17 | What is the difference between C-STRIPS and P-STRIPS? | C-STRIPS (coupon strips) are fungible across bonds; P-STRIPS (principal strips) are unique to each bond. |
| 18 | How do STRIPS prices relate to discount factors? | $P(0,T) = \text{STRIPS Price}/100$ |
| 19 | Why might STRIPS trade differently from implied zeros? | Liquidity differences, tax treatment (phantom income), reconstitution option. |
| 20 | What does it mean for a bond to trade "cheap"? | Its market price is below its theoretical value (replication cost). |
| 21 | What is OIS discounting? | Using overnight index swap rates to discount collateralized derivatives. |
| 22 | Why did the market shift from LIBOR to OIS discounting? | LIBOR embedded bank credit risk; OIS better approximates the risk-free rate. |
| 23 | What is a CSA? | Credit Support Annex—specifies collateral arrangements for OTC derivatives. |
| 24 | What are limits to arbitrage? | Balance sheet, funding, liquidity, and counterparty constraints that prevent perfect price enforcement. |
| 25 | How can you lock in a forward rate? | Sell units of the longer-dated zero and buy the shorter-dated zero, resulting in a forward loan. |

---

## Mini Problem Set

### Questions

1. **Discount factor basics.** If $P(0,1) = 0.95$, what is the PV of $200 paid at $T = 1$?

2. **Zero rate extraction.** Using $P(0,2) = 0.90$, compute the continuously compounded 2-year zero rate $z(2)$.

3. **Forward rate (FRA-style).** If $P(0, 0.5) = 0.98$ and $P(0, 1.0) = 0.95$, compute the simply compounded forward rate $F(0; 0.5, 1.0)$ with $\tau = 0.5$.

4. **Replication.** A claim pays 10 at $T = 1$ and 110 at $T = 2$. Express its price in terms of $P(0,1)$ and $P(0,2)$.

5. **Curve shock intuition.** Given cash flows at $T = 1, 3, 5$ years, which contributes most to PV sensitivity under a parallel rate increase? Why?

6. **Law of one price test.** Two portfolios have identical cash flows but trade at different prices. Construct the arbitrage and state time-0 profit.

7. **Deposit-to-DF conversion.** A 9-month deposit rate is 6% on Actual/360 with 270 days. Using $\tau = 270/360 = 0.75$, compute $P(0, 0.75)$.

8. **Forward from zero rates.** Zero rates (continuous) are $z(1) = 4\%$ and $z(2) = 5\%$. Compute $f(0; 1, 2)$.

9. **Compounding conversion.** Convert 8% quarterly compounding to continuous.

10. **Negative rates.** If the 1-year rate is -0.3% (continuous), what is $P(0,1)$? Interpret economically.

11. **OIS vs. LIBOR impact.** A 10-year swap has PV = $5,000,000 when discounted at LIBOR and PV = $4,950,000 when discounted at OIS. If the swap is collateralized, which valuation is correct? What is the error from using the wrong curve?

12. **STRIPS arbitrage.** A 1-year C-STRIP trades at 96.00 while the implied discount factor from coupon bonds is $d(1) = 0.952$. Is the STRIP rich or cheap? By how much?

13. **Investor choice.** The 6-month spot rate is 5.5% and the 1-year spot rate is 5.2% (both semiannual). What is the implied 6-month forward rate in 6 months? Would you buy the 1-year zero if you expect the 6-month rate to be 5.0% in six months?

14. **Limits to arbitrage.** An off-the-run Treasury trades 5bp cheap to the fitted curve. Explain why this "cheapness" might persist despite the apparent arbitrage opportunity.

### Brief Solutions (1–8)

1. $\text{PV} = 200 \times 0.95 = 190$

2. $z(2) = -\frac{1}{2} \ln(0.90) = -0.5 \times (-0.1054) = 0.0527 = 5.27\%$

3. $F = \frac{1}{0.5}\left(\frac{0.98}{0.95} - 1\right) = 2 \times (1.0316 - 1) = 6.32\%$

4. $V_0 = 10 \, P(0,1) + 110 \, P(0,2)$

5. The $T = 5$ cash flow contributes most because PV sensitivity is proportional to $T_i \cdot C_i \cdot P(0, T_i)$; the longer maturity (multiplier $T_i$) generally dominates.

6. Buy the cheap portfolio, sell the expensive one. Profit = (expensive price) − (cheap price). Future cash flows net to zero.

7. $P(0, 0.75) = \frac{1}{1 + 0.06 \times 0.75} = \frac{1}{1.045} = 0.9569$

8. $f(0; 1, 2) = \frac{0.05 \times 2 - 0.04 \times 1}{2 - 1} = \frac{0.10 - 0.04}{1} = 6\%$

---

## References

- Bruce Tuckman, *Fixed Income Securities* (discount factors; the law of one price; replication; Treasury STRIPS).
- John C. Hull, *Options, Futures, and Other Derivatives* (zero/forward rates; compounding conventions; OIS discounting).
