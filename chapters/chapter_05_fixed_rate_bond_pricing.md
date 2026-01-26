# Chapter 5: Fixed-Rate Bond Pricing

---

## Introduction

A fixed-rate bond is, at its core, a remarkably simple financial instrument: a promise to pay known amounts of money on known future dates. Yet this simplicity conceals a surprising number of decisions that must be made before you can answer the question "what is this bond worth?"

Consider a trader who wants to buy $10 million face value of a Treasury bond. The market quotes a price of "101-04⁵⁄₈"—but this is the *clean* price, not the cash that will change hands. Before settlement, someone must compute the accrued interest, convert the fractional quote to decimals, and calculate the actual invoice amount. Get any of these steps wrong, and the settlement fails or the wrong amount gets wired.

The stakes are real. A pricing error of even a few ticks on a $100 million position translates to tens of thousands of dollars. A systematic error in a trading system—say, using the wrong day count convention—can accumulate across thousands of trades before anyone notices. This chapter develops the complete pricing framework that prevents such errors.

We begin with the fundamental insight that bonds are simply collections of future cashflows, each of which can be valued using the discount factors introduced in Chapter 2. Hull states the principle directly: "The theoretical price of a bond can be calculated as the present value of all the cash flows that will be received by the owner of the bond... a more accurate approach is to use a different zero rate for each cash flow." We then connect this present value to what traders actually quote (clean price) versus what actually changes hands (dirty price), developing the accrued interest mechanics that bridge these two concepts.

The chapter covers:

1. **Cashflow schedule construction** — How to specify the amounts and dates of a bond's promised payments
2. **Present value using discount factors** — Pricing bonds using a discount curve as input
3. **Clean vs dirty price mechanics** — The market convention that separates "quoted" from "exchanged"
4. **Accrued interest calculation** — How the next coupon gets split between buyer and seller
5. **Treasury price quotation in 32nds** — Converting market quotes to decimal prices
6. **Price sensitivity intuition** — Why bond prices move when rates change (a preview for Chapter 11)

The law of one price, introduced in Chapter 2, provides the foundation: a bond's value must equal the sum of its discounted cashflows. Everything else—the clean/dirty convention, the 32nds quoting system, the day-count rules—is market plumbing that translates this fundamental principle into dollars that can actually be wired.

---

## 5.1 Fixed-Rate Bond Cashflows

### The Bond as a Cashflow Schedule

A fixed-rate bond promises a deterministic stream of payments. Tuckman describes the structure simply: a bond with face value $F$, coupon rate $c$, and semiannual payments promises coupon payments of $F \times c/2$ every six months, plus a principal repayment of $F$ at maturity. There is no uncertainty about the amounts or timing—only about what those future dollars are worth today.

More formally, let:
- $F$ = face value (principal), typically 100 "per 100 par"
- $c$ = annual coupon rate (as a decimal)
- $m$ = coupon payments per year (typically $m=2$ for semiannual)
- $T_1, T_2, \ldots, T_N$ = future payment dates after settlement

The coupon payment each period is:

$$\boxed{\text{Cpn} = \frac{Fc}{m}}$$

And the cashflow schedule is:

$$\text{CF}_i = \begin{cases}
\text{Cpn}, & i = 1, \ldots, N-1 \\
\text{Cpn} + F, & i = N
\end{cases}$$

The final cashflow bundles the last coupon with the principal repayment.

> **Worked Example:** If $F = 100$ and $c = 5.50\%$ with $m = 2$, then $\text{Cpn} = 100 \times 0.055 / 2 = 2.75$. This represents $2.75 payable on each coupon date for every $100 face value.

### Zero-Coupon Bonds as a Limiting Case

Setting $c = 0$ produces a zero-coupon bond: all intermediate cashflows vanish, leaving only the principal at maturity. This is useful as both a conceptual building block and a sanity check for any pricing implementation. As Luenberger notes, "a zero-coupon bond... generates a single cash flow, with no intermediate coupon payments."

### From Schedule to Value

> **Analogy: The Bag of Zeroes**
>
> A Coupon Bond is not a single instrument. It is just a "stapler" holding together a bag of different zero-coupon bonds.
> *   **$F=100$, 5% Coupon, 10-Year Bond** =
>     *   20 small coupons (each is a $2.50 zero-coupon bond).
>     *   1 large principal repayment (a $100 zero-coupon bond).
>
> **The Insight**: You don't price "The Bond". You price the 21 individual zeroes using their specific discount factors, then add them up. A bond is just a portfolio.

Once the cashflow schedule is specified, pricing reduces to a single question: what is the present value of these future payments? The answer depends on the discount curve, which brings us to the law of one price.

---

## 5.2 Present Value Using Discount Factors

### The Law of One Price

Chapter 2 introduced the discount factor $P(0,t)$ as "the value today of receiving one unit of currency at time $t$." Tuckman emphasizes that the **law of one price** allows "discount factors extracted from one set of bonds... to price any other bond with cash flows on the same set of dates."

This means we can value a bond by decomposing it into its component cashflows and pricing each separately. The total price is simply the sum of the present values of all future cashflows:

$$\boxed{P_{\text{dirty}} = \sum_{i=1}^{N} \text{CF}_i \cdot P(0, t_i)}$$

where $t_i$ is the time (in years) from the valuation date to payment date $T_i$.

### Curve-Based vs Yield-Based Pricing

Hull makes an important distinction that practitioners should internalize: "Sometimes bond traders use the same discount rate for all the cash flows underlying a bond, but a more accurate approach is to use a different zero rate for each cash flow."

The formula above uses the curve-based approach—each cashflow is discounted at its own rate. The alternative, yield-to-maturity pricing (covered in Chapter 6), forces a single rate on all cashflows. While yield-based pricing is convenient for quoting, curve-based pricing is more accurate and is what arbitrage-free pricing models use.

### Why "Dirty" Price?

The formula above produces what traders call the **dirty price** (also called **full price**, **invoice price**, or **cash price**). This is the actual amount paid for the bond at settlement. It represents the comprehensive economic value of the remaining cashflows. Tuckman states clearly: "the money paid by the buyer and received by the seller" is the invoice price.

### Worked Example: Pricing from a Discount Curve

Consider a bond with the following characteristics:
- **Face Value:** $F=100$
- **Coupon:** $5.50\%$ semiannual ($2.75$ per period)
- **Settlement Date:** February 15, 2001
- **Remaining Payments:** Four payments remaining, with the final one at maturity on January 31, 2003

**Step 1: Identify Cashflows**
- $T_1$ (Jul 31, 2001): $2.75$
- $T_2$ (Jan 31, 2002): $2.75$
- $T_3$ (Jul 31, 2002): $2.75$
- $T_4$ (Jan 31, 2003): $102.75$ (coupon + principal)

**Step 2: Apply Discount Factors**
Assume the market discount curve provides the following factors for these dates:
- $P(0, t_1) = 0.9800$
- $P(0, t_2) = 0.9550$
- $P(0, t_3) = 0.9300$
- $P(0, t_4) = 0.9099$

**Step 3: Calculate PV**
$$P_{\text{dirty}} = 2.75(0.9800) + 2.75(0.9550) + 2.75(0.9300) + 102.75(0.9099)$$
$$P_{\text{dirty}} = 2.6950 + 2.6263 + 2.5575 + 93.4922$$
$$\boxed{P_{\text{dirty}} = 101.3710}$$

The invoice price tells us that to buy this bond, one would pay $101.3710 per $100 face value. The price exceeds par ($>100$) because the coupon (5.50%) is higher than the market rates implied by the discount factors (roughly 4.8% for 2-year money).

---

## 5.3 Premium, Discount, and Par Bonds

### The Relationship Between Coupon and Price

A fundamental relationship connects the bond's coupon rate to its price relative to par:

| Condition | Bond Type | Price |
|-----------|-----------|-------|
| Coupon > Market Yield | **Premium** | $P > 100$ |
| Coupon = Market Yield | **Par** | $P = 100$ |
| Coupon < Market Yield | **Discount** | $P < 100$ |

**Why this holds:** If a bond's coupon is higher than current market rates, investors are willing to pay more than $100 for the stream of above-market payments. Conversely, if the coupon is below market rates, the bond must sell below $100 to compensate buyers for the sub-market cashflows.

Hull defines the **par yield** as "the coupon rate that causes the bond price to equal its par value." When the coupon rate exactly matches the par yield for that maturity, the bond trades at exactly 100.

### Par Bond Sanity Check

This provides a critical sanity check for any pricing implementation. If you input:
- A bond with coupon $c$
- A flat yield curve at rate $c$

Then the computed dirty price (at a coupon date, so AI = 0) should be exactly 100. If it's not, there's a bug.

> **Implementation Note:** This check catches numerous errors: wrong compounding conventions, off-by-one errors in cashflow schedules, incorrect day count fractions, and more.

### Visualizing "Pull to Par"

> **The Gravity of Par**
>
> If yields remain constant, a bond's price will naturally drift toward 100 as it approaches maturity.
>
> ```mermaid
> graph LR
>     A[Premium Bond (105)] -->|Time Passes| Par(100)
>     B[Discount Bond (95)] -->|Time Passes| Par(100)
>     C[Par Bond (100)] -->|Time Passes| Par(100)
> ```
>
> *   **Premium Bond**: You paid extra upfront. You "lose" slightly every day as price falls to 100. This loss offsets the high coupon.
> *   **Discount Bond**: You paid less upfront. You "gain" slightly every day as price rises to 100. This gain supplements the low coupon.
>
> **Result**: Total Return $\approx$ Yield, regardless of coupon.

---

## 5.4 Clean Price, Dirty Price, and Accrued Interest

### The Problem Clean Pricing Solves

If markets quoted dirty prices, bond prices would exhibit a sawtooth pattern: rising smoothly between coupon dates as interest accrues day by day, then dropping sharply by the coupon amount when the payment is made. This mechanical drop would hide true market movements. A trader wouldn't know if a price drop was due to rising interest rates or just a coupon payment.

### The Market Convention

Markets solve this by quoting the **clean price** (also called **flat price** or **quoted price**) and separately tracking **accrued interest**. The relationship is defined by the identity:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

Where:
- $P_{\text{dirty}}$ is the invoice price—the actual cash exchanged
- $P_{\text{clean}}$ is the quoted price
- $\text{AI}$ is accrued interest

- $\text{AI}$ is accrued interest

> **Analogy: The Menu vs. The Bill**
>
> *   **Clean Price** is the **Menu Price** ($20.00 for steak). It's what you see quoted.
> *   **Dirty Price** is the **Bill Amount** ($21.50 with tax). It's the check you actually write.
>
> If you show up halfway through a meal (coupon period), you also have to pay the previous guy for the part of the steak he already "earned".

Hull describes this clearly: "The quote... is referred to by traders as the clean price," while "the cash price paid by the purchaser... is referred to by traders as the dirty price."

### Why This Convention Works

Tuckman explains a subtle but important point: "The particular market convention used in calculating accrued interest does not really matter." Why? Because "the only quantity that matters is the invoice price (i.e., the money that changes hands), and it is this quantity that the market sets equal to the present value of the future cash flows."

If the accrued interest convention were too generous to the seller (AI too high), the market would simply lower the clean price to compensate. The invoice price—what actually matters economically—remains anchored to the present value of cashflows.

### Algebraic Proof of Clean Price Continuity

Tuckman demonstrates that "if yield does not change then the quoted price of a bond does not fall as a result of a coupon payment." The proof is elegant:

Let $P^b$ and $P^a$ be the quoted prices immediately before and after a coupon payment of $c/2$. Right before the coupon date:
- Accrued interest equals the full coupon: $\text{AI} = c/2$
- The present value of the imminent coupon equals $c/2$ (it's about to be paid)

So: $P^b + c/2 = c/2 + \text{PV}(\text{remaining cashflows})$

Which simplifies to: $P^b = \text{PV}(\text{remaining cashflows})$

Right after the coupon is paid, accrued interest resets to zero:
$P^a + 0 = \text{PV}(\text{remaining cashflows})$

Therefore $P^a = P^b$—the clean price is continuous across coupon dates.

> **P&L Perspective:** When traders mark a bond position to market, they track the change in *clean* price to measure performance. The coupon payments and accrued interest are handled as separate cash accounting entries. This isolates "price risk" (rates moving) from "time" (accrual).

### Example: Clean Price Continuity

To illustrate the sawtooth removal, consider our bond around the July 31, 2001 coupon date. Assume market yields are unchanged, so the clean price remains steady at 101.14.

| Date | Days Elapsed | Accrued Interest (AI) | Dirty Price | Note |
|------|--------------|-----------------------|-------------|------|
| Jul 30 | 180 (of 181) | $2.73$ | $103.87$ | Full coupon almost accrued |
| **Jul 31** | **Coupon Paid** | **Reset to 0** | **101.14** | **Dirty price drops by coupon** |
| Aug 01 | 1 (of 184) | $0.015$ | $101.155$ | Accrual starts again |

The *dirty* price drops from 103.87 to 101.14 (the coupon amount is paid out). The *clean* price stays at 101.14. This stability allows traders to compare prices easily across time.

---

## 5.5 Accrued Interest Mechanics

### Economic Purpose

Accrued interest compensates the seller for the portion of the coupon period during which they held the bond. As Tuckman explains with his 5½s example: if the bond makes a coupon payment on July 31 but the seller sells on February 15, the seller should receive compensation for having held the bond from the prior coupon date (January 31) to settlement. The buyer pays this amount at settlement, then collects the full coupon when it's paid.

### Calculation (Actual/Actual in Period)

For U.S. Treasury bonds, accrued interest is calculated using the **Actual/Actual** day count convention.

Let:
- $d_{\text{elapsed}}$ = actual number of days from the previous coupon date to the settlement date
- $d_{\text{period}}$ = actual number of days in the full coupon period

The accrued interest is the pro-rated share of the full coupon:

$$\boxed{\text{AI} = \frac{d_{\text{elapsed}}}{d_{\text{period}}} \times \text{Cpn}}$$

### Worked Example: Tuckman's 5½s of January 31, 2003

This is Tuckman's canonical example, which we reproduce exactly:

**Scenario:**
- **Bond:** 5½s of January 31, 2003
- **Coupon:** 5.50% ($2.75 semiannual payment per $100 face, or $275 per $10,000 face)
- **Settlement Date:** February 15, 2001
- **Previous Coupon:** January 31, 2001
- **Next Coupon:** July 31, 2001

**Calculation:**
1. **Days in period ($d_{\text{period}}$):** From January 31 to July 31 is 181 days.
2. **Days elapsed ($d_{\text{elapsed}}$):** From January 31 to February 15 is 15 days.
3. **Accrual Fraction:** $15 / 181 = 0.08287$.
4. **Accrued Interest per $100 face:**
   $$\text{AI} = \frac{15}{181} \times 2.75 = 0.2279$$
5. **Accrued Interest per $10,000 face:** $0.2279 \times 100 = \$22.79$

As Tuckman states: the buyer pays "$22.79 of accrued interest" and "having paid this $22.79 of accrued interest, investor B may keep the entire $275 coupon payment of July 31, 2001."

> **Market Practice Warning:** Different markets use different day counts. Corporate bonds often use **30/360**, which assumes every month has 30 days. Using the wrong convention will result in a settlement error. Always verify the convention before calculating.

---

## 5.6 Treasury Price Quotation in 32nds

### The Quote Format

In the U.S. Treasury market, prices are quoted in **32nds** of a dollar rather than decimals. Tuckman notes that "numbers after the hyphens denote 32nds, often called ticks."

A quote consists of the full dollar amount (the "handle") followed by the number of 32nds:

- A quote of "**99-16**" means $99 + 16/32 = 99.50$
- A quote of "**101-04**" means $101 + 4/32 = 101.125$

For finer precision, markets use **half-ticks** (indicated by "+") representing 1/64th:

- "**108-31+**" means $108 + 31/32 + 1/64 = 108 + 31.5/32 = 108.984375$

> **The "32nds Trap": A Common Rookie Mistake**
>
> A quote of **120-05** is **NOT** 120.05.
> *   **Wrong**: $120.05$
> *   **Right**: $120 \frac{5}{32} = 120.15625$
>
> The difference is huge (~10 cents vs ~1.5 cents). Always convert ticks to decimals before doing **any** math.

Tuckman provides the example: "the quote of 108-31+ would mean 108+31.5/32."

### Worked Example: From Quote to Invoice Price

This is Tuckman's complete example, which we reproduce in full.

**Inputs:**
- **Bond:** 5½s of January 31, 2003
- **Quote:** "101-4⁵⁄₈" (which is 101-4.625)
- **Accrued Interest:** $0.2279$ per $100 face (from previous example)
- **Face Amount:** $10,000

**Step 1: Convert Quote to Decimal Clean Price**
$$P_{\text{clean}} = 101 + \frac{4.625}{32} = 101 + 0.14453125 = 101.14453125$$

**Step 2: Add Accrued Interest to Get Dirty Price**
$$P_{\text{dirty}} = 101.14453125 + 0.2279 = 101.37243125$$

**Step 3: Calculate Invoice Amount**
For $10,000 face value (which is 100 "units" of $100 par):
$$\text{Invoice} = 100 \times 101.37243125 = \$10,137.24$$

Tuckman confirms: "the invoice price—that is, the money paid by the buyer and received by the seller—is $10,137.24."

---

## 5.7 Price Sensitivity to Rate Changes (Preview)

### The Mechanism

Since a bond's dirty price is the sum of discounted cashflows, any change in the discount curve immediately affects the price:
- If **rates rise**, discount factors fall ($1/(1+r)^t$ decreases). **Price falls.**
- If **rates fall**, discount factors rise. **Price rises.**

This inverse relationship is the most fundamental property of fixed-income risk.

### Why Longer Bonds Are More Sensitive

Cashflows further in the future are more sensitive to rate changes than near-term cashflows. This is because the discounting effect compounds over time.

**Sensitivity Check:**
Consider two cashflows, each of $100:
1. Payment in 6 months ($t=0.5$)
2. Payment in 10 years ($t=10$)

If rates rise by 1% (100 basis points):
- The 6-month discount factor drops slightly (e.g., from 0.975 to 0.970), a ~0.5% loss in PV.
- The 10-year discount factor drops substantially (e.g., from 0.60 to 0.55), a ~8% loss in PV.

Because the final repayment of principal occurs at maturity ($T_N$), the price of a long-term bond is dominated by this highly sensitive distant cashflow. This is why "long duration" bonds fall harder when rates rise. We formalize this as **DV01** (Dollar Value of a 01) and **Duration** in Chapters 11-12.

---

## 5.8 Practical Notes

### Settlement and Accrued

Accrued interest is always computed to the **settlement date**, not the trade date.
- **Treasuries:** Typically $T+1$ (next business day).
- **Corporates:** Typically $T+2$ (varies by market).

Trading on a Friday for $T+1$ settlement means settling on Monday, so accrued interest includes Friday, Saturday, and Sunday (the seller "earns" interest for holding through the weekend).

### Common Implementation Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| **Wrong Day Count** | Using 30/360 for a Treasury (or vice versa) results in incorrect AI and incorrect dirty price. |
| **Clean/Dirty Confusion** | Settling based on the clean price ignores the AI payment, causing a P&L break. |
| **Stub Periods** | The first or last coupon period may be "short" or "long." Standard formulas fail here; you must generate the schedule dates explicitly. |
| **32nds Handling** | Excel and programming languages use decimals. Manually entering "101.16" for "101-16" is a common data entry error (101.5 vs 101.16). |
| **Off-by-One in Schedules** | Miscounting the number of remaining cashflows leads to systematic pricing errors. |

### Verification Tests (Sanity Checks)

Any pricing engine should pass these checks:

1. **Zero Coupon Check:** If coupon $c=0$, price must equal $F \times P(0, T_N)$.
2. **Par Check:** If coupon rate equals the par yield for the bond's maturity, and we're on a coupon date (AI=0), the dirty price should equal 100.
3. **Continuity:** Dirty price should be continuous except at coupon dates; clean price should be continuous across coupon dates.
4. **AI Bounds:** $0 \le \text{AI} < \text{Coupon}$. An AI larger than the coupon is a red flag.
5. **Premium/Discount:** If coupon > market yield, price should be > 100; if coupon < market yield, price should be < 100.

---

## Summary

1. **A bond is a cashflow schedule.** It consists of known coupon payments ($Fc/m$) and a final principal payment ($F$).

2. **Value equals discounted cashflows.** The **law of one price** dictates that $P_{\text{dirty}} = \sum \text{CF}_i \times P(0, t_i)$.

3. **Curve-based pricing uses different rates for each cashflow.** As Hull notes, "a more accurate approach is to use a different zero rate for each cash flow."

4. **Markets quote clean, settle dirty.** The invoice price is the clean price plus accrued interest: $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$.

5. **Accrued interest is fair compensation.** It ensures the seller receives the interest earned during their holding period.

6. **Day counts matter.** Treasuries use Actual/Actual; corporates often use 30/360. Mixing conventions causes errors.

7. **Treasuries quote in 32nds.** "101-04" means $101 + 4/32 = 101.125$.

8. **Premium bonds have coupon > yield; discount bonds have coupon < yield.** Par bonds have coupon = yield and trade at 100.

9. **Prices move inversely to rates.** Rising rates lower bond prices; longer maturities are more sensitive.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Cashflow Schedule** | The series of future dates and amounts ($T_i, \text{CF}_i$) a bond promises to pay. | The starting point for all valuation. |
| **Discount Factor** | The value today of $1 at a future date $t$. | Converts future cashflows to present value. |
| **Dirty (Invoice) Price** | PV of all future cashflows. | The actual money transferred at settlement. |
| **Clean (Quoted) Price** | Dirty Price minus Accrued Interest. | Removes accrual drift for price comparison. |
| **Accrued Interest (AI)** | Pro-rata share of the next coupon earned since the last payment. | Compensates seller for holding time. |
| **Premium Bond** | Bond trading above par (coupon > yield). | Indicates above-market coupon payments. |
| **Discount Bond** | Bond trading below par (coupon < yield). | Indicates below-market coupon payments. |
| **32nds (Ticks)** | Treasury pricing unit ($1/32 \approx 0.03125$). | Historical convention for quoting. |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $F$ | Face value (principal), typically 100 |
| $c$ | Annual coupon rate (decimal) |
| $m$ | Coupon frequency (e.g., 2 for semiannual) |
| $\text{Cpn}$ | Coupon payment amount ($F \cdot c / m$) |
| $T_i$ | Future payment date |
| $d_{\text{elapsed}}$ | Days from last coupon to settlement |
| $d_{\text{period}}$ | Days in current coupon period |
| $P_{\text{clean}}$ | Quoted (flat) price |
| $P_{\text{dirty}}$ | Invoice (full) price |
| $\text{AI}$ | Accrued Interest |
| $P(0,t)$ | Discount factor for maturity $t$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental formula for bond pricing? | $P_{\text{dirty}} = \sum \text{CF}_i \cdot P(0, t_i)$ |
| 2 | What is the relationship between clean and dirty price? | Dirty = Clean + Accrued Interest |
| 3 | Which price do traders usually quote? | Clean price |
| 4 | Which price is actually paid at settlement? | Dirty (Invoice) price |
| 5 | Why do markets use clean prices? | To avoid the "sawtooth" price drop when coupons are paid; makes prices comparable over time |
| 6 | How do you calculate the semiannual coupon payment? | Face Value × Coupon Rate / 2 |
| 7 | What is the formula for Accrued Interest (Actual/Actual)? | (Days Elapsed / Days in Period) × Coupon |
| 8 | Convert the Treasury quote "99-16" to decimal. | $99 + 16/32 = 99.50$ |
| 9 | Convert the Treasury quote "100-01+" to decimal. | $100 + 1.5/32 = 100.046875$ |
| 10 | If rates rise, what happens to bond prices? | Prices fall (inverse relationship). |
| 11 | Which bond is more sensitive to rate changes: 2-year or 10-year? | 10-year (longer duration). |
| 12 | What is the "Law of One Price" in bond pricing? | Two securities with the same cashflows must have the same price. |
| 13 | What is a "zero-coupon" bond? | A bond with $c=0$; pays only principal at maturity. |
| 14 | Does accrued interest belong to the buyer or seller? | It is paid by the buyer to the seller. |
| 15 | When does a bond trade at a premium? | When its coupon rate exceeds the market yield. |
| 16 | When does a bond trade at par (price = 100)? | When its coupon rate equals the par yield for its maturity. |
| 17 | What does Hull say about curve vs yield pricing? | "A more accurate approach is to use a different zero rate for each cash flow." |
| 18 | What happens to clean price across a coupon payment date? | It is continuous (does not jump). |
| 19 | What happens to dirty price when a coupon is paid? | It drops by approximately the coupon amount. |
| 20 | What is a "tick" in Treasury markets? | 1/32nd of a dollar, approximately 3.125 cents per $100 face. |

---

## Mini Problem Set

**Problem 1:** Calculate the semiannual coupon payment for a $10,000 face value bond with a 4.50% coupon.
*Solution:* $10,000 \times 0.045 / 2 = \$225$.

**Problem 2:** A bond is settled with 60 days elapsed since the last coupon. The full coupon period is 182 days. The semiannual coupon payment is $3.00. Calculate the Accrued Interest.
*Solution:* $\text{AI} = \$3.00 \times (60/182) = \$0.989$.

**Problem 3:** Discount factors are: $P(0.5)=0.98$, $P(1.0)=0.96$. A bond pays $5 in 6 months and $105 in 1 year. Calculate the dirty price.
*Solution:* $P = 5(0.98) + 105(0.96) = 4.90 + 100.80 = 105.70$.

**Problem 4:** A Treasury bond is quoted at **98-08**. Accrued interest is **0.35**. What is the invoice price?
*Solution:* Clean = $98 + 8/32 = 98.25$. Dirty = $98.25 + 0.35 = 98.60$.

**Problem 5:** Why does the dirty price of a bond drop immediately after a coupon is paid?
*Solution:* Because the holder no longer receives that cashflow; the value of the bond decreases by the PV of the coupon (which at that moment equals the coupon amount).

**Problem 6:** You buy a bond on Friday. Settlement is T+1 (Monday). How many days of accrued interest do you pay for over the weekend?
*Solution:* You pay accrued interest calculated *up to* the settlement date (Monday). The seller "earns" interest for Friday, Saturday, and Sunday because they hold the bond until settlement.

**Problem 7:** Convert **102-15+** to a decimal price.
*Solution:* $102 + 15.5/32 = 102 + 0.484375 = 102.484375$.

**Problem 8:** If a bond's coupon rate is higher than the market yield, will it trade at a discount or premium?
*Solution:* Premium (Price > 100). The bond pays more than the going market rate.

**Problem 9:** A 5% coupon bond is priced using a flat 5% yield curve. What should the dirty price be at a coupon date?
*Solution:* Exactly 100 (par). When coupon equals yield and AI=0, the bond is at par.

**Problem 10:** A Treasury quotes at 100-00. Accrued interest is 1.50. What is the invoice price per $1 million face?
*Solution:* Clean = 100.00. Dirty = 101.50. Invoice = $1,000,000 × 1.0150 = $1,015,000.

**Problem 11:** Why is curve-based pricing "more accurate" than yield-based pricing according to Hull?
*Solution:* Because it uses the appropriate zero rate for each cashflow's maturity, rather than forcing a single average rate on cashflows of different maturities.

**Problem 12:** Verify the Tuckman example: 5½s of Jan 31, 2003, quoted at 101-4⁵⁄₈ with AI = 0.2279. What is the dirty price?
*Solution:* Clean = $101 + 4.625/32 = 101.1445$. Dirty = $101.1445 + 0.2279 = 101.3724$.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| "Invoice price—the money paid by the buyer and received by the seller" | Tuckman Ch 4, discussing 5½s example |
| "The only quantity that matters is the invoice price" and it "equals the present value of future cash flows" | Tuckman Ch 4 |
| Quote examples: "101-4⁵⁄₈", invoice = $10,137.24 | Tuckman Ch 4 (5½s of Jan 31, 2003 example) |
| Dirty Price = Clean Price + Accrued Interest | Tuckman Ch 4, Hull Ch 4 |
| Treasury Quote Syntax (32nds, "+" for half-ticks) | Tuckman Ch 1 ("numbers after the hyphens denote 32nds, often called ticks") |
| Accrued Interest formula (Actual/Actual): AI = (days elapsed / days in period) × coupon | Tuckman Ch 4 |
| Clean price continuous across coupon dates: "$P^a = P^b$" | Tuckman Ch 4 (algebraic proof) |
| "A more accurate approach is to use a different zero rate for each cash flow" | Hull Ch 4 (Bond Pricing section) |
| "Par yield... is the coupon rate that causes the bond price to equal its par value" | Hull Ch 4 |
| "Discount factors extracted from one set of bonds may be used to price any other bond with cash flows on the same set of dates" | Tuckman Ch 1 (Law of One Price) |
| T+1 settlement for Treasuries | Tuckman Ch 1 |
| Zero-coupon bond "generates a single cash flow" | Luenberger Ch 3 |

### (B) Reasoned Inference (Derived from A)

- **Price Sensitivity:** Derived from the PV formula—falling discount factors imply falling price. The magnitude grows with time-to-cashflow because $(1+r)^{-t}$ is more sensitive to $r$ for larger $t$.
- **Premium/Discount Logic:** If coupon > yield, the stream of above-market coupons has positive NPV relative to par, so price > 100. Symmetric argument for discount bonds.
- **Settlement Timing:** Weekend accrual follows from the definition that AI is computed to settlement date.
- **Unit Examples:** Derived from standard arithmetic scaling ($10,000 face = 100 units of $100 par).

### (C) Flagged Uncertainties

- **Specific Holiday Calendars:** Exact "business day" definitions vary by exchange and country. This chapter uses general rules.
- **Ex-Dividend Dates:** Some non-U.S. markets have specific ex-dividend rules not detailed here.
- **Fractional 32nds Beyond Half-Ticks:** Some platforms use quarter-ticks (1/128ths); conventions vary by trading venue.
