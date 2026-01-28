# Chapter 41: CDS Indices — Mechanics, Coupons, Rolls

---

## Introduction

A portfolio manager realizes her credit book has drifted long by $50 million DV01-equivalent exposure across 80 investment-grade names. To hedge with single-name CDS, she would need to execute 80 separate trades, negotiate 80 bid-ask spreads, and monitor 80 positions going forward. Or she could execute one trade: buying protection on the CDX NA IG index, a standardized basket of 125 investment-grade credits that captures broad credit risk in a single liquid instrument.

This is the power of CDS indices—but the simplicity comes with mechanical complexity that can create expensive mistakes. Consider the difference between a "Series 35" and a "Series 36" trade: the series number determines not just which 125 companies you reference, but also the fixed coupon, the maturity date, and—critically—whether you're trading the liquid on-the-run contract or a less liquid off-the-run series with wider bid-ask spreads. Misunderstanding the upfront payment convention (is this index spread-quoted or price-quoted?) can turn a $100 million trade into a multi-million-dollar booking error. And as O'Kane emphasizes, the terminology itself is treacherous: "a buyer of the CDS index means a buyer of credit risk—the party which receives the spread," which is *opposite* to the common single-name CDS convention where "buyer" typically means buyer of protection.

**Why this matters for practitioners:** The CDS index market is the most liquid corner of credit derivatives, with O'Kane noting that the bid-offer for CDX IG "is roughly equivalent to a bid-offer of 1 cent on a five-year bond with a price of \$100." Indices serve three roles simultaneously: macro credit hedging vehicles, relative value trading instruments, and the underlying reference portfolios for the tranche products covered in Chapters 48–51. Getting the mechanics wrong—confusing spread with price quoting, misunderstanding roll timing, or miscalculating intrinsic value—creates P&L errors that compound across a book of positions.

**Chapter roadmap:** Section 41.1 introduces the major index families and their composition rules. Section 41.2 details the index lifecycle: settlement mechanics, coupon payments, and default handling. Section 41.3 explains the critical relationship between fixed coupons, quoted spreads, and upfront payments—the heart of index pricing. Section 41.4 covers the semi-annual roll process and resulting liquidity dynamics. Section 41.5 develops the intrinsic value calculation and index basis framework. Section 41.6 presents index risk measures. Section 41.7 addresses index options, including why Black's model fails in its standard form. Section 41.8 details the portfolio swap adjustment algorithm used to reconcile intrinsic and quoted values for tranche pricing. Section 41.9 covers practical notes and P&L attribution. Section 41.10 describes the post-2009 Big Bang/Small Bang market structure and central clearing.

**Relationship to other chapters:** Chapter 38 covers single-name CDS mechanics that underpin index contracts. Chapter 40 explains the auction process used for constituent defaults. Chapter 42 covers survival curve bootstrapping used in intrinsic calculations. Chapter 46 develops intrinsic spread and basis trading further. Chapters 48–51 build on index mechanics for tranche products.

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $M$ | Number of names in the index (e.g., 125 for CDX NA IG) |
| $C(T)$ | Contractual coupon (annualized), fixed at series issuance |
| $S_I(t,T)$ | Market-quoted index spread at time $t$ for maturity $T$ |
| $U_I(t)$ | Upfront payment (percent of notional; can be positive or negative) |
| $\text{RPV01}_I(t,T)$ | Index risky PV01 under flat curve convention |
| $S_m(t,T)$ | CDS spread for constituent $m$ |
| $\text{RPV01}_m(t,T)$ | Risky PV01 for constituent $m$ |
| $Q_m(t,T)$ | Survival probability for constituent $m$ from $t$ to $T$ |
| $\tau_m$ | Default time of constituent $m$ |
| $R_m$ | Recovery rate of constituent $m$ |
| $\Delta(a,b)$ | Accrual fraction between dates $a$ and $b$ (typically Actual/360) |
| $N$ | Index trade notional |
| $d$ | Number of defaults that have occurred |
| $G(K)$ | Exercise price function for index options struck at $K$ |
| $H(S)$ | Index value function at spread $S$ |
| $\alpha(T_n)$ | Portfolio swap adjustment multiplier for maturity $T_n$ |

---

## 41.1 Index Families and Composition

### 41.1.1 The Major Index Families

O'Kane provides a comprehensive overview of the CDS portfolio indices that dominate credit derivatives markets. The two most liquid families are:

**CDX (North America):** The CDX NA IG (Investment Grade) index contains 125 investment grade credits domiciled in North America. Related indices include CDX NA HY (High Yield, 100 names), CDX NA XO (Crossover, 35 names), and CDX EM (Emerging Markets sovereign credits, 15 names).

**iTraxx (Europe and Asia):** The iTraxx Europe index contains 125 investment grade names domiciled in Europe. Regional variants include iTraxx Europe Crossover (approximately 40 names), iTraxx Japan (50 names), iTraxx Asia ex-Japan (50 names), and iTraxx Australia (25 names).

**Table 41.1: Major CDS Index Families**

| Index | Region | Number of Names |
|-------|--------|-----------------|
| CDX.NA.IG | North America (investment grade) | 125 |
| CDX.NA.HY | North America (high yield) | 100 |
| CDX.NA.XO | North America (crossover) | 35 |
| CDX.EM | Emerging markets sovereign | 15 |
| iTraxx Europe | Europe (investment grade) | 125 |
| iTraxx Europe Crossover | Europe (crossover) | ~40 |
| iTraxx Japan | Japan (investment grade) | 50 |
| iTraxx Asia ex-Japan | Asia ex-Japan (investment grade) | 50 |
| iTraxx Australia | Australia (investment grade) | 25 |

*Source: O'Kane, Table 10.1*

Hull notes that these portfolios "are updated on March 20 and September 20 each year. Companies that are no longer investment grade are dropped from the portfolios and new investment grade companies are added."

As O'Kane observes, "the CDX and the iTraxx families of indices follow almost exactly the same rules in terms of how they work," though important differences exist in credit event definitions (discussed below).

### 41.1.2 Composition Rules and Selection Criteria

For the investment grade indices, O'Kane explains that constituents are "chosen to represent the most liquid credits satisfying the index criteria." For iTraxx Europe, "all the issuers in the index are domiciled in Europe and are investment grade quality." For CDX, "the 125 credits are all domiciled in North America and are also rated investment grade quality."

The constituents of a specific series remain fixed for that series' lifetime, except that defaulted names are removed without replacement. However, the composition can change from series to series. O'Kane identifies three reasons for removal at roll:

- Downgrade below investment grade
- A credit event
- Material reduction in liquidity of the credit

When names are removed, "new reference credits which do satisfy the inclusion criteria" replace them.

### 41.1.3 Key Regional Difference: Restructuring Clauses

Hull and O'Kane both highlight an important distinction between CDX and iTraxx. O'Kane states: "While the European iTraxx indices include a restructuring credit event, the North American CDX index protection leg is only triggered by a bankruptcy or failure to pay on a reference credit. Restructuring is not included as a credit event."

This makes CDX consistent with "No-Re" CDS, which typically trade at spreads about 5% lower than Mod-Re (modified restructuring) CDS. As O'Kane explains, "Since No-Re spreads are typically about 5% lower than Mod-Re spreads, we have an immediate basis" between index levels and single-name CDS that include restructuring. This difference is one of the primary drivers of the index basis discussed in Section 41.5.3.

### 41.1.4 Equal Weighting

In the standard index mechanics that O'Kane describes, each name has equal weight: "the portfolio consists of $M$ credits" with "each paying out the loss $(1-R_m)$ at time $\tau_m$" on a share of $1/M$ of the index notional. For a 125-name index with \$125 million notional, each name represents \$1 million of exposure.

Hull illustrates this concretely: "Suppose a trader wants \$800,000 of protection on each company. The total cost is $0.0066 \times 800,000 \times 125$, or \$660,000 per year."

> **Cross-reference:** Chapter 38 covers single-name CDS mechanics. The premium leg, protection leg, and credit event settlement mechanisms for indices follow the same foundational structure, but applied across a portfolio.

### 41.1.5 LCDX: The Loan CDS Index

Beyond the standard corporate CDS indices, O'Kane (Section 10.7.2) describes the **LCDX.NA** index, which references the North American leveraged loan CDS market. LCDX tracks 100 first-lien syndicated secured loans and differs from the standard CDS indices in several important respects:

**Higher expected recovery:** Loans are typically secured by collateral, so expected recovery rates are substantially higher than for unsecured bonds. O'Kane notes that "loan CDS trade at a tighter (lower) credit default spread than standard CDS on the same name. This effect can be ascribed to the generally superior recovery rates of secured loans."

**Cancellation feature:** Loan CDS (LCDS) contracts include a cancellation feature because the underlying loan may be refinanced or prepaid. O'Kane models this with a separate cancellation hazard rate $\lambda_C(t)$, giving the survival-and-not-cancelled probability as $Q(t,T) = Q_D(t,T) \cdot Q_C(t,T)$.

**European variant (LevX):** O'Kane notes the LevX index, which "references the 35 most liquid first-lien credit agreements traded in the European leveraged loan CDS market."

> **Practitioner Note:** LCDX trades less liquidly than CDX IG and requires separate survival curve calibration accounting for both default and cancellation risk. The different recovery dynamics mean that hedging LCDX with standard CDS creates basis risk.

---

## 41.2 Index Lifecycle: Settlement, Coupons, and Defaults

### 41.2.1 Trade Date and Settlement

O'Kane describes the settlement mechanics: "The contract is entered into on trade date $t$ and is cash settled three days later." At settlement, "the buyer of the contract makes an upfront payment to the seller equal to the value of the contract."

**Inception timing:** At the inception of a new series, the upfront value is "typically close to zero. This is because at inception, the index coupon is set close to the fair-value spread of the index. It is not set exactly equal to the fair-value spread as the index coupon is usually chosen to be a multiple of 5 bp."

This rounding convention means that even at launch, there is typically a small upfront payment to reconcile the fixed coupon with the exact market spread.

### 41.2.2 Coupon Payments

Following settlement, the index buyer (the party receiving spread, i.e., the protection seller) receives periodic coupon payments. O'Kane notes that coupons are "usually paid quarterly according to an Actual 360 basis." This matches the standard single-name CDS premium leg convention.

The premium payment at coupon date $t_k$, when the outstanding notional at the start of the accrual period is $N_k$, follows:

$$\boxed{\text{Premium Payment}_k = N_k \cdot C(T) \cdot \Delta(t_{k-1}, t_k)}$$

where $C(T)$ is the contractual coupon (annualized) and $\Delta(t_{k-1}, t_k)$ is the accrual fraction under Actual/360.

Hull notes that "when a company defaults, the protection buyer receives the usual CDS payoff and the annual payment is reduced by $660,000/125 = \$5,280$," illustrating how coupon payments scale with the remaining names.

### 41.2.3 Maturity and IMM Dates

Issue dates align to CDS "IMM" dates: March 20, June 20, September 20, and December 20. O'Kane explains: "Those issued on 20 March will mature on 20 June of the appropriate year depending on the index maturity and likewise those issued on 20 September will mature on 20 December."

Hull adds that "the maturity dates for these types of contracts on the index are usually December 20 and June 20. (This means that a '5-year' contract actually lasts between $4\frac{3}{4}$ and $5\frac{1}{4}$ years.)"

An important subtlety is that tenor labels are approximate: "the so-called 3Y, 5Y, 7Y or 10Y on-the-run indices do not have exactly 3, 5, 7 or 10 years to maturity on their issue date. At issuance the time to maturity of a T-year index is typically T+3 months, declining to T-3 months when the next index is issued."

**Table 41.2: Maturity Alignment Example**

| Issue Date | Maturity Label | Actual Maturity | Months at Issuance |
|------------|----------------|-----------------|-------------------|
| March 20, 2024 | 5Y | June 20, 2029 | 63 months |
| September 20, 2024 | 5Y | December 20, 2029 | 63 months |

### 41.2.4 How Defaults Are Handled

When a constituent defaults, O'Kane describes three consequences:

1. **Settlement:** "The buyer pays $1/M$ of the face value of the contract to the seller in return for delivery of a defaulted asset also on $1/M$ of the contract notional."

2. **Accrued premium:** "The buyer receives the fraction of premium which has accrued from the previous coupon date on the defaulted credit."

3. **Notional reduction:** "The notional of the contract is reduced by $1/M$. As a result, the absolute amount of spread received on the premium leg is reduced."

After $d$ defaults, the outstanding notional fraction is:

$$\boxed{\text{Outstanding notional} = N \cdot \left(1 - \frac{d}{M}\right)}$$

For physical delivery constraints, O'Kane notes that ISDA introduced a protocol in 2005 "which allows a fallback to cash settlement in which case an auction method is used to determine a cash settlement price." This auction mechanism addresses situations "where the outstanding notional of derivative contracts exceeds the supply of deliverable obligations."

> **Cross-reference:** Chapter 40 covers the CDS auction process in detail. The same mechanism applies to index constituents.

---

## 41.3 Coupon, Quoted Spread, and Upfront

The relationship between fixed coupons, market-quoted spreads, and upfront payments is central to index trading. Hull provides a clear description: for each underlying and maturity, "a coupon and a recovery rate are specified. A price is calculated from the quoted spread."

### 41.3.1 The Fixed Coupon Convention

Unlike single-name CDS which can trade at any spread, indices trade with a fixed coupon $C(T)$ that is set at issuance and does not change for the life of that series. As market conditions change, the difference between fair value and the fixed coupon is reconciled through an upfront payment.

Hull explains the advantage: "This arrangement facilitates trading because the instruments trade like bonds. The regular quarterly payments made by the buyer of protection are independent of the spread at the time the buyer enters into the contract."

### 41.3.2 Converting Between Spread and Upfront

The market convention treats the index as if it were a single-name CDS with a flat spread curve. O'Kane derives the fundamental relationship:

$$\boxed{U_I(t) = \big(S_I(t,T) - C(T)\big) \cdot \text{RPV01}_I(t,T)}$$

where:
- $U_I(t)$ is the upfront payment (can be positive or negative)
- $S_I(t,T)$ is the quoted market spread
- $C(T)$ is the fixed contractual coupon
- $\text{RPV01}_I(t,T)$ is the risky PV01 of the index

**Sign convention for protection buyer:**
- When $S_I > C$: Market spread exceeds coupon → protection buyer *receives* upfront (compensated for paying below-market coupon)
- When $S_I < C$: Market spread below coupon → protection buyer *pays* upfront (compensates seller for receiving above-market coupon)

### 41.3.3 The Exact Intrinsic Spread Equation

O'Kane (Equation 10.8) provides the *exact* relationship between intrinsic upfront and intrinsic spread. The intrinsic upfront from the constituent sum (Equation 10.5) must equal the index-level upfront:

$$\boxed{U_I(t,T) = \big(S_I(t,T) - C(T)\big) \cdot \text{RPV01}_I(t,T,S_I)}$$

where critically, the RPV01 itself depends on the spread $S_I$ (through the flat-curve survival probability). This creates a circularity: to find $S_I$ from $U_I$, we must solve the equation numerically because RPV01 changes with spread.

O'Kane states this explicitly: "there is no simple linear solution... We therefore have to resort to some one-dimensional root solver."

**Table 41.3: Exact vs. Approximate Intrinsic Spread**

| Index | Maturity | Exact Intrinsic | Approximate RPV01-Weighted |
|-------|----------|-----------------|---------------------------|
| CDX NA IG S7 | 5Y | 33.1 bp | 33.1 bp |
| CDX NA HY S7 | 5Y | 275 bp | 280 bp |

*Source: O'Kane Table 10.4 (modified). For IG indices, the approximation is excellent. For HY, where spreads are high and survival probability materially affects RPV01, the difference matters.*

### 41.3.4 Hull's Price Calculation Method

Hull provides a concrete procedure for calculating index prices:

1. "Imply a hazard rate from the quoted spread. This involves calculations similar to those in Section 25.2. An iterative search is used to determine the hazard rate that leads to the quoted spread."

2. "Calculate a 'duration' $D$ for the CDS payments. This is the number that the spread is multiplied by to get the present value of the spread payments."

3. "The price $P$ is given by $P = 100 - 100 \times D \times (s - c)$, where $s$ is the spread and $c$ is the coupon expressed in decimal form."

**Example from Hull (Example 25.1):** "Suppose that the iTraxx Europe index quote is 34 basis points and the coupon is 40 basis points for a contract lasting exactly 5 years, with both quotes being expressed using an actual/360 day count... The duration is 4.447 years. The price is therefore $100 - 100 \times 4.447 \times (0.00345 - 0.00406) = 100.27$"

When the price exceeds 100, the seller of protection pays the buyer upfront. In Hull's example, "the seller of protection would pay the buyer \$1,000,000 × 125 × 0.0027" initially.

### 41.3.5 Investment Grade vs. High Yield Quoting Conventions

O'Kane describes different quoting conventions for different credit qualities:

**Investment grade (spread quote):** "For investment grade contracts such as the CDX IG and the iTraxx Europe, the market quotes an index spread which represents the flat spread at which the index should be priced if it were a simple CDS contract."

**High yield and emerging markets (price quote):** "Other lower credit quality indices such as the EM and HY indices are quoted using the bond price convention... in which the upfront value is computed simply by subtracting 100. This convention has the advantage that it avoids any disagreement about the value of the index PV01 which is required to convert a spread-based quotation into an upfront value."

**Table 41.4: Quoting Conventions by Index Type**

| Index | Quoting Convention | Reason |
|-------|-------------------|--------|
| CDX.NA.IG | Spread (bp) | Low PV01 dispute risk |
| iTraxx Europe | Spread (bp) | Low PV01 dispute risk |
| CDX.NA.HY | Bond price (%) | Avoids PV01 disputes |
| CDX.EM | Bond price (%) | Avoids PV01 disputes |

**Price interpretation:** A bond price of 102.18% means "payment of 2.18% of face value to enter into the index." A price of 99.5% means "a payment to the investor of 0.50% of face value."

### 41.3.6 Accrued Premium Mechanics

Index valuation follows the same clean vs. full price distinction as bonds. O'Kane's index valuation outputs "compute the accrued interest based on 50 days of accrual using an Actual 360 basis. Since we are long protection, this has a negative sign."

The general relationship:

$$\text{Clean MTM} = \text{Full MTM} - \text{Accrued}$$

with the accrued sign depending on whether you are long or short protection.

---

## 41.4 Roll Mechanics and Liquidity Dynamics

### 41.4.1 The Semi-Annual Roll Process

O'Kane explains that "most CDS indices are issued or rolled semi-annually with a coupon which is fixed for the lifetime of the issue." The roll creates a new on-the-run series and relegates the previous series to off-the-run status.

The roll timeline follows the IMM date structure:

```
Roll Timeline
=============

Series N issued                 Series N+1 issued             Series N+2 issued
(e.g., March 20)               (e.g., September 20)          (e.g., March 20)
     |                              |                              |
     |<--- Series N on-the-run ---->|<--- Series N+1 on-the-run -->|
     |                              |                              |
     |                    Series N becomes                Series N+1 becomes
     |                    off-the-run                     off-the-run
```

Hull notes that "on September 20, 2020, the Series 34 iTraxx Europe portfolio and the Series 35 CDX NA IG portfolio were defined. The series numbers indicate that, by the end of September 2020, the iTraxx Europe portfolio had been updated 33 times and the CDX NA IG portfolio had been updated 34 times."

> **Visual: The Roll (Series 30 → Series 31)**
>
> Every March 20 and Sept 20, the "On-The-Run" (liquid) contract changes.
>
> 1.  **Old Series (30)**: Becomes "Off-The-Run." Liquidity vanishes. Bid-Ask widens (1 cent → 5 cents).
> 2.  **New Series (31)**: New constituents (remove defaulters/downgrades). Maturity extends by 6 months.
> 3.  **The Roll Trade**: Investors sell Series 30 and buy Series 31 to stay liquid. This huge flow creates P&L opportunities.

### 41.4.2 P&L Impact from Rolling

When investors roll positions, O'Kane identifies two drivers of P&L impact:

1. **Composition changes:** "Any changes in the composition of the index which change its perceived credit quality. For example, the new index will not contain any names which have been downgraded below investment grade or whose liquidity has declined."

2. **Maturity extension:** "The new index has a longer maturity, by six months, than the previous index. As credit curves tend to be upward sloping, this would tend to cause the new index spread to be wider, all other things being the same."

**Worked Example 41.A: Roll P&L Calculation**

Suppose you hold a \$50 million long protection position in Series 6 that you want to roll into Series 7.

*Series 6 (off-the-run at roll):*
- Coupon: 40 bp
- Current spread: 35 bp
- RPV01: 4.05 years

*Series 7 (new on-the-run):*
- Coupon: 45 bp
- Current spread: 38 bp
- RPV01: 4.35 years

**Step 1:** Calculate Series 6 upfront value

$$U_6 = (35 - 40) \times 4.05 = -20.25 \text{ bp} = -0.2025\%$$

Since you're long protection and the spread is below the coupon (favorable terms for you), to unwind you receive:

$$\text{Unwind cash} = +\$50\text{mm} \times 0.002025 = +\$101,250$$

**Step 2:** Calculate Series 7 upfront value

$$U_7 = (38 - 45) \times 4.35 = -30.45 \text{ bp} = -0.3045\%$$

The new series also has positive value (spread below coupon). You pay upfront to enter:

$$\text{Entry cash} = -\$50\text{mm} \times 0.003045 = -\$152,250$$

**Step 3:** Net roll cash

$$\text{Roll P\&L} = +\$101,250 - \$152,250 = -\$51,000$$

The roll costs \$51,000, reflecting the maturity extension and any composition differences.

### 41.4.3 On-the-Run vs. Off-the-Run Liquidity

The concentration of liquidity in the on-the-run series is a defining feature of index markets. O'Kane notes the practical consequence for options: "Option expiries beyond six months are rare because the index rolls every six months and the liquidity drops immediately after the next roll, increasing hedging costs."

**Quantitative evidence from bid-ask spreads:** O'Kane reports that for the CDX IG index, "this is roughly equivalent to a bid-offer of 1 cent on a five-year bond with a price of \$100." Off-the-run series typically trade wider.

The liquidity premium also affects the basis between index spreads and constituent CDS spreads. O'Kane lists reasons for the index basis:

- "The considerable size and liquidity of the CDS index market means that the CDS index spread embeds a lower liquidity risk premium than the less liquid CDS spreads."
- "As the CDS index is more liquid, it tends to be the preferred instrument used by market participants to express a changing view about the credit market as a whole... As a result, the CDS index may be considered to lead the CDS market."

### 41.4.4 Hedging Considerations

O'Kane explicitly warns about hedging off-the-run with on-the-run: "if we hedge an off-the-run index swap position with an on-the-run index swap position, there may be a mismatch in terms of some credits being in one index but not in the other."

This constituent mismatch creates basis risk. If a name that exists in the off-the-run series but not the on-the-run series experiences a credit event or large spread move, the hedge will be imperfect.

---

## 41.5 Intrinsic Value and the Index Basis

The intrinsic value of an index is computed by treating it as the sum of its constituent single-name CDS positions. Understanding the relationship between intrinsic value and market value is essential for relative value trading.

### 41.5.1 Intrinsic Value Calculation

O'Kane derives the intrinsic value by summing protection and premium legs across all constituents. For a long protection position, the intrinsic value at time $t$ is:

$$\boxed{V_I(t) = \frac{1}{M}\sum_{m=1}^{M}\big(S_m(t,T) - C(T)\big) \cdot \text{RPV01}_m(t,T)}$$

where $S_m(t,T)$ is the CDS spread for constituent $m$ and $\text{RPV01}_m(t,T)$ is its risky PV01.

**Worked Example 41.B: Intrinsic Value from Constituents**

Consider a simplified 5-name index with coupon $C = 50$ bp.

| Name | CDS Spread (bp) | RPV01 (years) |
|------|-----------------|---------------|
| A | 40 | 4.20 |
| B | 55 | 4.15 |
| C | 45 | 4.25 |
| D | 60 | 4.10 |
| E | 50 | 4.20 |

**Step 1:** Calculate $(S_m - C) \times \text{RPV01}_m$ for each name (long protection perspective):
- A: $(40-50) \times 4.20 = -42$ bp (below coupon, unfavorable)
- B: $(55-50) \times 4.15 = +20.75$ bp
- C: $(45-50) \times 4.25 = -21.25$ bp
- D: $(60-50) \times 4.10 = +41$ bp
- E: $(50-50) \times 4.20 = 0$ bp

**Step 2:** Sum and divide by $M = 5$:

$$V_I = \frac{1}{5}(-42 + 20.75 - 21.25 + 41 + 0) = \frac{-1.5}{5} = -0.3 \text{ bp}$$

The intrinsic value is -0.3 bp of notional (slightly negative for protection buyer).

### 41.5.2 The Intrinsic Spread

O'Kane defines the intrinsic spread as "the flat spread at which a CDS contract with a coupon $C(T)$ has the same upfront value as the index." He provides an approximation:

$$\boxed{S_{\text{intrinsic}}(t,T) \approx \frac{\sum_{m=1}^{M} S_m(t,T) \cdot \text{RPV01}_m(t,T)}{\sum_{m=1}^{M} \text{RPV01}_m(t,T)}}$$

This is the RPV01-weighted average of constituent spreads—*not* a simple average. O'Kane emphasizes: "the index spread is not a simple average of the individual CDS spreads but is close to the RPV01-weighted average."

Hull makes a similar observation: "More precisely, the index is slightly lower than the average of the credit default swap spreads for the companies in the portfolio. To understand the reason for this consider a portfolio consisting of two companies, one with a spread of 1,000 basis points and the other with a spread of 10 basis points. To buy protection on the companies would cost slightly less than 505 basis points per company. This is because the 1,000 basis points is not expected to be paid for as long as the 10 basis points and should therefore carry less weight."

**Why intrinsic spread is below the average spread:** When spread dispersion exists, names with higher spreads have lower RPV01 (higher default probability reduces the PV of premium payments). This pulls the weighted average below the arithmetic mean. O'Kane reports data showing the difference:

**Table 41.5: Intrinsic vs. Average Spread**

| Index | 5Y Intrinsic Spread | 5Y Average Spread | Std Dev |
|-------|---------------------|-------------------|---------|
| CDX NA IG Series 7 | 33.1 bp | 33.5 bp | 36 bp |
| iTraxx Europe Series 6 | 26.9 bp | 27.1 bp | 20 bp |
| CDX NA HY Series 7 | 275 bp | 305 bp | 340 bp |

*Source: O'Kane, Table 10.5*

The effect is small for investment grade (less than 1 bp) but substantial for high yield (30 bp difference) due to the much higher spread dispersion.

### 41.5.3 The Index Basis

The *market-quoted* index spread often differs from the intrinsic spread. O'Kane identifies several reasons:

1. **Restructuring clause differences:** "In the case of the North American CDX index, the payment of protection on the index protection leg is only triggered when the credit event is a bankruptcy or failure to pay. Restructuring is not included as a credit event... Since No-Re spreads are typically about 5% lower than Mod-Re spreads, we have an immediate basis."

2. **Liquidity premium:** "The considerable size and liquidity of the CDS index market means that the CDS index spread embeds a lower liquidity risk premium than the less liquid CDS spreads."

3. **Market leadership:** "As the CDS index is more liquid, it tends to be the preferred instrument used by market participants to express a changing view about the credit market as a whole... the CDS index may be considered to lead the CDS market. This is especially true in a widening market where investors use long protection positions in the index to hedge illiquid long credit positions."

> **Deep Dive: The London Whale (2012)**
>
> JPMorgan's "CIO" office tried to defend a massive position in the CDX IG Index.
>
> *   **The Trade**: They sold protection on the Index (IG9) but bought protection on specific Tranches.
> *   **The Squeeze**: Hedge funds realized the Whale was trapped. They aggressively bought Index protection, driving the spread wider.
> *   **The Basis**: The Index spread widened *massively* relative to the Single Names (Intrinsic). The Whale couldn't exit because selling the Index would widen spreads further.
> *   **Result**: $6 Billion Loss. The Index became "cheaper" to buy than the sum of its parts, solely due to flow.

**Table 41.6: Empirical Index Basis Data**

| Index | Term | Quoted Spread | Intrinsic Spread | Basis |
|-------|------|---------------|------------------|-------|
| CDX NA IG Series 7 | 5Y | 34 bp | 33.1 bp | +0.9 bp |
| CDX NA IG Series 7 | 10Y | 55 bp | 55.3 bp | -0.3 bp |
| iTraxx Europe Series 6 | 5Y | 24 bp | 26.9 bp | -2.9 bp |
| iTraxx Europe Series 6 | 10Y | 42 bp | 45.8 bp | -3.8 bp |

*Source: O'Kane, Table 10.6*

Note that the basis can be positive or negative, varies across indices, and can differ by maturity within the same index.

> **Cross-reference:** Chapter 46 covers intrinsic spread calculation and basis trading in depth.

### 41.5.4 Effect of a Constituent Default on Index Spread

When a constituent defaults, it is removed from the index without replacement. This mechanically affects the index spread level.

**Worked Example 41.C: Default Impact on Index**

Consider a 125-name index trading at 40 bp. Name X, which traded at 500 bp (distressed), defaults.

*Before default:*
- Approximate intrinsic spread ≈ weighted average of 125 names including X at 500 bp

*After default:*
- Name X is removed; notional reduces to 124/125
- The 500 bp name no longer contributes to the spread calculation
- Remaining 124 names now determine the spread

If the other 124 names averaged 36 bp, removing the 500 bp outlier causes the intrinsic spread to *tighten*. The protection buyer also receives a settlement payment of $(1-R) \times (N/125)$.

For index options, O'Kane shows this matters for exercise decisions: "if there is one default, the holder of a payer option will exercise the option as soon as $S(t_E, T) > 42.7$ bp which is about 12 bp below the option strike" of 55 bp. "Each default is worth about 12 bp of spread widening" in terms of effective strike adjustment.

---

## 41.6 Index Risk Measures

### 41.6.1 Index CS01 (Credit Spread DV01)

The sensitivity of index value to a parallel shift in the quoted spread is:

$$\text{CS01} \equiv \frac{\partial \text{PV}}{\partial S_I} \times 1\text{ bp}$$

To first order, when treating $\text{RPV01}$ as locally constant:

$$\boxed{\text{CS01} \approx N \cdot \text{RPV01}_I(t,T) \cdot 10^{-4}}$$

O'Kane's index valuation outputs explicitly report "Risky PV01" and "Credit DV01" as standard risk measures.

**Worked Example 41.D: CS01 Calculation**

For a \$100 million index position with RPV01 = 4.25 years:

$$\text{CS01} = 100,000,000 \times 4.25 \times 0.0001 = \$42,500 \text{ per bp}$$

### 41.6.2 Jump-to-Default Exposure

For an equal-weighted index, a single-name default produces an immediate loss (for protection seller) or gain (for protection buyer) of:

$$\text{JTD exposure per name} = \frac{N(1-R_m)}{M}$$

On a \$125 million notional with 125 names and 40% recovery, each name's JTD exposure is:

$$\$125\text{mm} \times 0.60 / 125 = \$600,000$$

### 41.6.3 Series Basis Risk

Rolling from off-the-run to on-the-run introduces basis risk because:
- Constituent composition differs
- Maturity differs by approximately 6 months
- Liquidity profiles differ significantly

O'Kane warns: "if we hedge an off-the-run index swap position with an on-the-run index swap position, there may be a mismatch in terms of some credits being in one index but not in the other."

---

## 41.7 Index Options: The Knockout Problem and Why Black's Model Fails

### 41.7.1 What an Index Option References

An index option (portfolio swaption) is an OTC contract to buy or sell protection on a specified index (usually the **5Y on-the-run** at option origination) with underlying maturity $T$, struck at spread $K$, expiring at $t_E$.

O'Kane (Chapter 11) expresses the index option exercise price as:

$$\boxed{G(K) = (K - C(T)) \cdot \text{RPV01}_I(t_E, T, K)}$$

where $\text{RPV01}_I(t_E, T, K)$ is an index risky PV01 from expiry to maturity priced using a flat curve at spread $K$.

**Payer option:** The right to buy protection at strike $K$. Valuable when spreads widen beyond $K$.

**Receiver option:** The right to sell protection at strike $K$. Valuable when spreads tighten below $K$.

### 41.7.2 Why Long-Dated Options Are Rare

O'Kane explicitly notes: "Option expiries beyond six months are rare because the index rolls every six months and the liquidity drops immediately after the next roll, increasing hedging costs."

Since underlying liquidity drops after roll, longer-dated expiries that would straddle a roll face higher hedging costs, leading to reduced trading activity in such options.

### 41.7.3 Default Impact on Option Exercise

In index options, defaults between option initiation and expiry affect the payoff. O'Kane describes that a payer index swaption holder "can settle protection payments on defaults occurring between initiation and expiry by purchasing defaulted credits and delivering for par; each defaulted credit is worth $(1-R)$ times its index notional exposure."

This creates a complex exercise decision where the effective strike moves with defaults. If defaults occur during the option life:
- The protection value already received from settled defaults must be accounted for
- The remaining index has fewer names, affecting the spread calculation
- The effective exercise boundary shifts downward for payer options

### 41.7.4 Why Black's Model Fails for Index Options

O'Kane (Section 11.3) demonstrates that Black's model, which works for single-name CDS swaptions, **cannot be directly applied to index options**. The reasons are fundamental:

**Problem 1: The Knockout Boundary Failure**

As O'Kane explains: "This is most easily seen if we consider a payer swaption in the limit of $K \rightarrow \infty$. In this limit, the option to buy protection on the index at a very high strike should tend to zero—we would never exercise an option to pay an infinite spread."

But using the naive Black's model adaptation (adding front-end protection value):

$$V^{\text{Non-KO pay}}(t) = V^{\text{KO pay}} + \text{FEP}(t)$$

where $\text{FEP}(t,t_E) = (1-R)Z(t,t_E)(1-Q(t,t_E))$ is the front-end protection value.

The problem: even as $K \to \infty$, the knockout payer option value goes to zero but the **front-end protection value remains positive**. This means the model gives a positive option price for an option that should be worthless.

**Problem 2: Put-Call Parity Violation**

O'Kane shows that Black's model "does not reprice the underlying correctly." Specifically, a portfolio of long payer and short receiver swaptions should replicate a forward CDS position. Black's model implies:

$$V^{\text{Non-KO pay}}(t) - V^{\text{Rec}}(t) = (S_I(t,t_E,T) - K) \cdot \text{RPV01}_I(t,t_E,T) + \text{FEP}(t,t_E)$$

But this differs from the true forward CDS value because the front-end protection terms don't cancel properly for the index case.

**Problem 3: Flat Curve Inconsistency**

O'Kane notes: "Black's model typically assumes a flat term structure of index spreads for the purpose of calculating the term structure of survival probabilities. This is not necessarily consistent with the spreads of the underlying CDS or the price of shorter maturity CDS indices."

> **The Knockout Catch: An Analogy**
>
> Imagine insurance that pays if your house is robbed during a vacation. But if the house *burns down* before the vacation, the policy becomes void (knockouts).
>
> For a payer index option at a very high strike, you're buying insurance that pays only if spreads go extremely wide. But the front-end protection component says you still get paid if defaults happen early—even though you'd never exercise the spread option. This is economically nonsensical.

### 41.7.5 The Arbitrage-Free Pricing Framework

O'Kane (Section 11.4) presents the correct approach, following Pedersen (2003) and Andersen (2006a). The key insight is that the **exercise decision must account for all three value components** at expiry:

1. **Spread value:** $(S_I(t_E,T) - K) \cdot \text{RPV01}_I(t_E,T)$ — the gain/loss from entering the CDS at strike vs. market
2. **Front-end defaults:** Value of defaults already settled between option initiation and expiry
3. **Index composition:** The fact that the index has fewer names after defaults

Define the value function at expiry:

$$H(t_E, S_I(t_E,T)) = \frac{1}{M}\sum_{m=1}^{M}\big(S_m(t_E,T) - C(T)\big) \cdot \text{RPV01}_m(t_E,T) + H_0$$

where $H_0$ captures the accumulated value from defaults before $t_E$.

The payer option payoff is:

$$\boxed{V^{\text{pay}}(t_E) = \max\big[H(t_E, S_I(t_E,T)) - G(K), 0\big]}$$

This framework correctly handles the exercise boundary—the option is exercised when the *total* value $H$ exceeds the exercise price $G(K)$, not simply when $S > K$.

**Worked Example 41.E: Default Impact on Exercise Decision**

From O'Kane's example: CDX Series 7 payer option with strike $K = 55$ bp, coupon $C = 60$ bp.

*No defaults case:*
- Exercise when $S(t_E,T) > 55$ bp

*One default case:*
- The front-end protection is worth approximately $(1-R)/M = 0.6/125 = 0.48\%$ of notional
- This is equivalent to about 12 bp of spread (dividing by RPV01 ≈ 4)
- Exercise when $S(t_E,T) > 55 - 12 = 43$ bp approximately

"Each default is worth about 12 bp of spread widening" in terms of effective strike adjustment.

### 41.7.6 How Index Options Are Actually Quoted

> **Practitioner Note (market convention):** Index options are typically quoted in premium (cents per $100 notional) rather than implied volatility. O'Kane notes this is "one reason why the market tends to simply quote in terms of option premium."

The challenge with volatility quoting is that different dealers may use different models (naive Black's vs. full model), making volatility quotes non-comparable. Premium quoting avoids this model dependency, at the cost of requiring the dealer to run their own model to convert to risk measures.

---

## 41.8 The Portfolio Swap Adjustment Algorithm

### 41.8.1 The Calibration Problem

When pricing index tranches (Chapters 48-51), practitioners need the intrinsic value to exactly match the quoted market value. However, as shown in Section 41.5.3, the bottom-up intrinsic spread typically differs from the quoted spread due to the index basis.

This creates a calibration problem: the survival curves implied by single-name CDS do not, in general, produce an intrinsic value that matches the observed index price. O'Kane states: "One way to ensure that the index swap spread equals the intrinsic swap spread is to see if there is an adjustment that can be made to the individual issuer curves which can enforce [consistency]."

### 41.8.2 The Mathematical Framework

O'Kane (Section 10.6.1) formalizes the problem. We need adjusted spreads $S_m^*(t,T_n)$ such that for each index maturity $T_n$:

$$\frac{1}{M}\sum_{m=1}^{M}\big(S_m^*(t,T_n) - C(T_n)\big) \cdot \text{RPV01}_m(t,T_n,S_m^*) = U_I(t,T_n)$$

where $U_I(t,T_n)$ is the observed index upfront at maturity $T_n$.

O'Kane's preferred approach is to use a **spread multiplier** $\alpha(T_n)$ applied uniformly to all names:

$$S_m^*(t,T_n) = \alpha(T_n) \cdot S_m(t,T_n)$$

The adjusted spreads satisfy:

$$\boxed{\frac{1}{M}\sum_{m=1}^{M}\big(\alpha(T_n) S_m(t,T_n) - C(T_n)\big) \cdot \text{RPV01}_m(t,T_n,\alpha S_m) = U_I(t,T_n)}$$

### 41.8.3 The Fixed-Point Iteration Algorithm

O'Kane provides an explicit algorithm for finding $\alpha(T_n)$. The key insight is that while there's no closed-form solution (RPV01 depends on the adjusted spread), fixed-point iteration converges quickly:

**Algorithm: Spread Multiplier Portfolio Swap Adjustment**

1. Build all $m = 1, \ldots, M$ survival curves $Q_m(t,T)$ using the standard market CDS maturities.
2. Calculate the interpolated CDS spreads to the index curve maturities $S_m(t,T_n)$. Then rebuild all of the CDS curves using just these spreads.
3. Set $n = 1$.
4. Set $k = 0$, $S_m^{(k)}(t,T_n) = S_m(t,T_n)$, and $\alpha_n(k) = 1.0$.
5. For all issuers $m = 1, \ldots, M$, adjust the spread: $S_m^{(k+1)}(t,T_n) = \alpha_n(k) \cdot S_m^{(k)}(t,T_n)$.
6. Build all $M$ survival curves using the adjusted spread curves $S_m^{(k+1)}(t,T_n)$.
7. Calculate $\alpha_n(k+1)$ using:

$$\boxed{\alpha_n(k+1) = \alpha_n(k) \times \frac{U_I(t,T_n) + C(T_n) \cdot \frac{1}{M}\sum_{m=1}^{M}\text{RPV01}_m(t,T_n,S_m^{(k+1)})}{\frac{1}{M}\sum_{m=1}^{M}S_m^{(k+1)}(t,T_n) \cdot \text{RPV01}_m(t,T_n,S_m^{(k+1)})}}$$

8. If $|\alpha_n(k+1) - 1| \le \epsilon$ where $\epsilon$ is the required tolerance, continue to step (9). Otherwise set $k = k+1$ and return to step (5).
9. Set $\alpha(T_n) = \alpha_n(k+1)$.
10. Set $n = n+1$. If $n > N$ then stop. Otherwise return to step (4).

O'Kane notes: "Most of the work of this iteration scheme is done in the first iteration since the dependence of the issuer PV01s on the spread is second order. In most cases we find that the algorithm converges with a tolerance in present value terms of the order of $O(10^{-8})$ within five iteration steps."

### 41.8.4 Faster Alternative: Forward Default Probability Multiplier

O'Kane provides a faster variant that avoids rebuilding survival curves in each iteration by adjusting survival probabilities directly:

$$Q_m^*(t,T_n) = Q_m^*(t,T_{n-1}) \cdot Q_m(T_{n-1},T_n)^{\alpha(n)}$$

This raises the forward survival probability to a power $\alpha(n)$. The same iteration logic applies, but "because we avoid the time-consuming bootstrap of all $M$ curves in each iteration step... this method is about 50 times faster than the first method and typically takes less than a second for a 125-name portfolio using four maturity points."

**Table 41.7: Portfolio Swap Adjustment Example**

| Reference Entity | 3Y Spread Unadj. (bp) | 3Y Spread Adj. (bp) | Ratio | 5Y Spread Unadj. (bp) | 5Y Spread Adj. (bp) | Ratio |
|-----------------|----------------------|--------------------|----|----------------------|--------------------|----|
| Index (quoted) | — | — | — | — | — | — |
| Intrinsic (unadj.) | 16.2 | — | — | 26.9 | — | — |
| **Ratio** | — | — | **0.741** | — | — | **0.892** |
| Adecco SA | 23.2 | 17.1 | 0.737 | 38.8 | 34.4 | 0.89 |
| Bertelsmann AG | 16.9 | 12.5 | 0.737 | 28.8 | 25.7 | 0.89 |
| DaimlerChrysler AG | 34.4 | 25.4 | 0.737 | 52.9 | 44.6 | 0.84 |
| Deutsche Telekom AG | 21.2 | 15.6 | 0.737 | 37.0 | 33.5 | 0.91 |

*Source: O'Kane, Table 10.7 (abbreviated). The 3Y adjustment ratio (0.74) differs from the 5Y ratio (0.89) because the basis varies by maturity.*

> **Desk Reality: When Portfolio Swap Adjustment Matters**
>
> The adjustment is **essential** when pricing tranches. If you use unadjusted curves, your tranche model will show an arbitrage: the sum of tranche values won't equal the index value. Central to all consistent tranche pricing is first adjusting single-name curves to match the liquid index market.
>
> **Common pitfall:** Not updating the adjustment when spreads move significantly. The ratios in Table 41.7 are point-in-time; during market stress, the adjustment can change materially.

---

## 41.9 Practical Notes

### 41.9.1 Booking Checklist

Before booking an index trade, verify:

1. **Index family and series:** On-the-run vs. off-the-run
2. **Maturity date:** IMM-aligned; "5Y" is not exactly 5 years
3. **Coupon:** Fixed at series issuance (typically a multiple of 5 bp)
4. **Day count:** Actual/360, quarterly payments
5. **Quoting convention:** Spread (IG) vs. price (HY/EM)
6. **Outstanding notional:** May be reduced by prior defaults
7. **Settlement convention:** T+3 business days (note: cleared trades may differ)
8. **Direction and terminology:** Index "buyer" receives spread (opposite to single-name convention)
9. **Clearing status:** Mandatory for standard index trades post-Dodd-Frank

### 41.9.2 Common Pitfalls

- **Terminology confusion:** Index "buyer" ≠ protection buyer. Index buyer *receives* spread (sells protection).
- **Spread vs. upfront:** Must use consistent conversion with correct RPV01
- **Series vs. maturity:** These are different concepts—Series 35 5Y is not a 35-year contract
- **Accrued premium:** Include in settlement calculations
- **Notional after defaults:** Premium payments scale with *outstanding* notional, not original notional
- **Roll date mechanics:** Verify IMM dates for your specific series
- **Quoting convention errors:** IG quotes as spread; HY/EM quote as price—mixing these is expensive
- **Cleared vs. bilateral:** Different documentation, margin mechanics, and settlement timing

### 41.9.3 Sanity Checks

- **Premium scaling:** Doubling notional doubles premium payments
- **Default payout bounds:** Loss fraction $\in [0, 1]$, so payout $\in [0, N/M]$
- **Upfront sign:** When spread > coupon, protection buyer receives upfront
- **Intrinsic ≤ average:** For dispersed portfolios, RPV01-weighted average is below arithmetic mean
- **Basis direction:** Positive basis means index quoted wider than intrinsic
- **Adjustment ratio:** Typically $< 1$ when index trades tight to intrinsic

### 41.9.4 P&L Attribution for Index Positions

> **Practitioner Note:** For middle-office professionals transitioning to front office, understanding how to explain daily P&L is essential. Here's the decomposition framework.

**Daily P&L Components for an Index Position:**

1. **Spread P&L:** $\Delta S_I \times \text{CS01}$ — the first-order impact of index spread changes
2. **Carry:** Accrued premium earned (if short protection) or paid (if long protection)
3. **Roll-down:** As time passes, the index "rolls down" the spread curve if curves are upward sloping
4. **Default P&L:** Settlement value received/paid on any constituent defaults
5. **Convexity:** Second-order effect from large spread moves (usually small for indices)
6. **Funding/Collateral:** Margin interest, collateral financing costs

**Worked Example 41.F: P&L Attribution**

Position: \$100mm short protection on CDX IG (receive 50bp coupon)
RPV01: 4.25 years
Day: 91 days since last coupon

| Component | Calculation | Amount |
|-----------|-------------|--------|
| Spread P&L | Index spread widened 2bp: $-2 \times \$100\text{mm} \times 4.25 \times 0.0001$ | -$85,000 |
| Carry | $\$100\text{mm} \times 0.0050 \times (1/360)$ | +$1,389 |
| Default P&L | Name X defaults, FP=35: $-\$100\text{mm}/125 \times (1-0.35)$ | -$520,000 |
| **Net P&L** | | **-$603,611** |

> **How to explain to risk committee:** "We lost $604k. The default in Name X cost $520k immediately (JTD loss). Spreads also widened 2bp, costing another $85k. Partially offset by one day of carry at $1.4k."

---

## 41.10 Post-2009 Market Structure: Big Bang and Small Bang

> **Practitioner Note (post-2009 market structure):** The following section describes changes implemented after O'Kane's book (2008). This content is marked as (B) Claude-Extended.

### 41.10.1 The Big Bang Protocol (April 2009)

Following the 2008 financial crisis, ISDA implemented the "Big Bang" protocol for North American CDS in April 2009. The key standardization elements were:

**Fixed coupons:** All single-name and index CDS now trade with standardized coupons:
- 100 bp for investment grade names
- 500 bp for high yield names (or 100 bp if spread is below ~300 bp)

**Upfront quoting:** All trades now quote in terms of upfront points plus running coupon, eliminating disputes about fair spread at trade execution.

**Standardized accrual:** Premium accrual begins on the previous IMM date (March 20, June 20, September 20, December 20), even for trades executed mid-cycle. This means a trade done on April 15 would show accrued premium since March 20.

**Auction hardwiring:** Credit event settlement via ISDA auction became the default mechanism, replacing physical settlement for most purposes.

### 41.10.2 Small Bang (June 2009)

Europe followed with the "Small Bang" protocol in June 2009, implementing similar standardization with some regional adaptations:

- Fixed coupons of 25 bp, 100 bp, 500 bp, or 1000 bp depending on credit quality
- Restructuring remains a credit event (unlike North American No-Re)
- Similar accrual and settlement standardization

### 41.10.3 Central Clearing for Indices

> **Practitioner Note:** Regulatory requirements vary by jurisdiction. The following describes general principles; specific rules require consultation of current regulatory texts.

Following Dodd-Frank (US) and EMIR (Europe), standard CDS index trades are subject to mandatory clearing:

**US clearing (ICE Clear Credit):**
- Mandatory for CDX.NA.IG and CDX.NA.HY indices
- Initial margin required (varies with portfolio composition, typically 2-5% for IG)
- Variation margin exchanged daily
- Settlement through ICE Clear Credit

**European clearing (LCH.Clearnet, ICE Clear Europe):**
- Mandatory for iTraxx Europe indices
- Similar margin framework to US

**Impact on trading:**
- Reduced counterparty risk (CCP becomes counterparty to all cleared trades)
- Standardized documentation (no bilateral ISDA negotiation for cleared trades)
- Margin requirements tie up capital but reduce credit exposure
- Settlement timing may differ from bilateral trades

**Table 41.8: Pre-Big Bang vs. Post-Big Bang Conventions**

| Aspect | Pre-Big Bang (Pre-2009) | Post-Big Bang (Current) |
|--------|------------------------|------------------------|
| **Coupon** | Quoted at-market spread | Fixed 100bp (IG) or 500bp (HY) |
| **Upfront** | Small or zero at trade | Upfront points = (Spread - Coupon) × RPV01 |
| **Accrual start** | Trade date | Previous IMM date |
| **Settlement** | Primarily physical | ISDA auction (hardwired) |
| **Clearing** | Bilateral | Mandatory for standard indices |

> **I'm not sure** about the exact current initial margin percentages for index trades—these are set by the CCPs and vary with market conditions and portfolio composition. Consult CCP rulebooks for current requirements.

---

## Summary

CDS indices package credit exposure to standardized portfolios of names into single, liquid instruments. The two dominant families—CDX for North America and iTraxx for Europe—share similar mechanics but differ in credit event definitions (CDX excludes restructuring, creating a basis versus single-name CDS that include it).

Indices trade with fixed coupons set at issuance, with the difference from fair value reconciled through upfront payments using the identity $U = (S-C) \times \text{RPV01}$. Investment grade indices quote by spread; high yield and emerging market indices quote by bond price to avoid PV01 disputes.

The semi-annual roll process creates on-the-run and off-the-run series with significant liquidity differences. Rolling positions generates P&L through composition changes and maturity extension. Hedging off-the-run with on-the-run creates constituent mismatch risk.

The intrinsic value calculation connects index levels to constituent spreads through RPV01-weighted averaging, though market quotes often differ due to liquidity premia, contract differences, and market dynamics. The portfolio swap adjustment forces intrinsic to match quoted for tranche calibration purposes.

Index options cannot be priced with naive Black's model due to the knockout problem and front-end protection issues. The correct framework (Pedersen/Andersen) accounts for the full exercise decision including default settlements.

Post-2009, the Big Bang/Small Bang protocols standardized coupons (100/500 bp), established mandatory upfront quoting, and central clearing became required for standard indices.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **On-the-run** | Most recently issued series | Concentrates liquidity; preferred for hedging |
| **Off-the-run** | Previous series after roll | Lower liquidity; constituent mismatch risk |
| **Fixed coupon** | Contractual spread set at issuance | Creates need for upfront payments as markets move |
| **Upfront payment** | PV of (spread − coupon) × RPV01 | Reconciles fixed coupon to current fair value |
| **Index roll** | Semi-annual issuance of new series | Changes composition and extends maturity |
| **Intrinsic value** | Sum of constituent CDS values | Benchmark for relative value vs. market quote |
| **Index basis** | Market spread − intrinsic spread | Reflects liquidity and contract differences |
| **Notional reduction** | Outstanding falls by 1/M per default | Affects premium payments and risk exposure |
| **Portfolio swap adjustment** | Calibration to force intrinsic = quoted | Required for consistent tranche pricing |
| **Front-end protection** | Value of defaults before option expiry | Why Black's model fails for index options |
| **Big Bang** | 2009 standardization (NA) | Fixed coupons, upfront quoting, clearing |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CDS index? | An OTC bilateral contract referencing a portfolio of names with premium and protection legs across the basket |
| 2 | What are the two major index families? | CDX (North America) and iTraxx (Europe/Asia) |
| 3 | How many names in CDX NA IG and iTraxx Europe? | 125 each |
| 4 | What is a "series"? | A specific issuance with fixed coupon and fixed constituents (except default removals) |
| 5 | What does "on-the-run" mean? | The most recently issued series, from issuance until the next roll |
| 6 | How often do indices roll? | Every six months (March 20 and September 20) |
| 7 | What are the IMM dates for index issuance? | March 20, June 20, September 20, December 20 |
| 8 | Is a "5-year" index exactly 5 years at issuance? | No; typically T+3 months at issuance, declining to T-3 months by next roll |
| 9 | How is the coupon paid? | Quarterly on Actual/360 |
| 10 | What is the upfront-spread relationship? | $U = (S - C) \times \text{RPV01}$ |
| 11 | What quoting convention do IG indices use? | Spread (basis points) |
| 12 | What quoting convention do HY/EM indices use? | Bond price (percentage of par) |
| 13 | What happens to notional when a constituent defaults? | Reduced by 1/M |
| 14 | How does default affect future coupon payments? | They decrease proportionally with outstanding notional |
| 15 | What is the key difference between CDX and iTraxx credit events? | CDX excludes restructuring (No-Re); iTraxx includes it (Mod-Re) |
| 16 | What is the intrinsic spread? | RPV01-weighted average of constituent CDS spreads |
| 17 | Why is intrinsic spread below average spread? | Higher-spread names have lower RPV01, reducing their weight |
| 18 | What causes the index basis? | Liquidity differences, restructuring clause differences, market leadership |
| 19 | Why are long-dated index options rare? | Roll reduces underlying liquidity, increasing hedging costs |
| 20 | What risk arises from hedging off-the-run with on-the-run? | Constituent mismatch between series |
| 21 | What does an index "buyer" do? | Receives spread (sells protection)—opposite of single-name convention |
| 22 | What is the portfolio swap adjustment? | Calibration tweak to force intrinsic value to match quoted market value |
| 23 | Why does HY/EM quote by price instead of spread? | Avoids disputes about the correct PV01 for spread-to-upfront conversion |
| 24 | What is the "knockout problem" in index options? | A payer option struck at infinite spread should be worthless, but naive Black's model gives positive value due to front-end protection |
| 25 | What is the payer option exercise payoff formula? | $\max[H(t_E, S) - G(K), 0]$ where $H$ is total index value and $G(K)$ is exercise price |
| 26 | Why can't Black's model be used directly for index options? | (1) Knockout boundary fails, (2) Put-call parity violated, (3) Flat curve assumption inconsistent |
| 27 | What is the Big Bang (2009)? | ISDA protocol standardizing NA CDS: fixed 100/500 bp coupons, upfront quoting, auction hardwiring |
| 28 | What is the portfolio swap adjustment algorithm? | Fixed-point iteration: multiply all single-name spreads by $\alpha$ until intrinsic matches quoted |
| 29 | What happens to index spread when a distressed name defaults? | It typically tightens—the high-spread outlier is removed from the weighted average |
| 30 | What is the primary US clearing house for credit indices? | ICE Clear Credit |

---

## Mini Problem Set

**Problems 1-4: Basic Mechanics**

1. Compute the quarterly premium payment for an index with notional \$100mm, coupon 35 bp, and accrual fraction 91/360.

2. After 3 defaults in a 125-name index, what is the outstanding notional fraction?

3. If a name defaults with final price FP = 30 (per 100) on an index with \$125mm notional and 125 names, what is the protection payment?

4. Build a 4-quarter premium schedule for \$50mm notional, 50 bp coupon, with day counts (90, 91, 92, 90).

**Problems 5-8: Upfront and Spread Conversion**

5. Given coupon 40 bp, quoted spread 52 bp, and RPV01 = 4.25, compute the upfront payment as % of notional.

6. Given coupon 100 bp, upfront = −2.5% of notional, and RPV01 = 4.00, compute the implied spread.

7. A bond-price quote of 97.5% implies what upfront payment direction and amount?

8. Explain why solving for spread from upfront requires iteration.

**Problems 9-12: Roll Mechanics**

9. You hold \$80mm protection in Series 6 and roll to Series 7. Series 6: spread 38 bp, coupon 40 bp, RPV01 4.10. Series 7: spread 42 bp, coupon 45 bp, RPV01 4.40. Calculate roll P&L.

10. Why does maturity extension at roll tend to widen the new series spread?

11. Name two risks of hedging off-the-run with on-the-run.

12. Why do index option expiries rarely exceed 6 months?

**Problems 13-16: Intrinsic Value**

13. A 3-name index has constituents with spreads 40, 50, 60 bp and RPV01s 4.2, 4.1, 4.0 years. Index coupon is 50 bp. Calculate intrinsic value per bp of notional.

14. For the same index, calculate the intrinsic spread using the RPV01-weighted formula.

15. Compare the intrinsic spread to the arithmetic average spread (50 bp). Why is the intrinsic lower?

16. If the market quotes this index at 48 bp, what is the basis?

**Problems 17-20: Advanced Topics**

17. (Portfolio swap adjustment) If the intrinsic spread is 35 bp but the index trades at 32 bp, what direction of adjustment ratio $\alpha$ is needed?

18. (Index options) Explain why a payer option on an index can be exercised even when the index spread is below the strike.

19. (Big Bang) Under post-Big Bang conventions, a new CDX IG trade is done at 85 bp spread. What coupon will be used, and what is the approximate upfront? (Assume RPV01 = 4.2 years.)

20. (Default impact on spread) A 125-name IG index trades at 50 bp. One name trading at 800 bp defaults. Assuming the remaining 124 names average 44 bp, estimate the post-default intrinsic spread.

---

### Solution Sketches

1. $100\text{mm} \times 0.0035 \times (91/360) = \$88,472$

2. $1 - 3/125 = 122/125 = 97.6\%$

3. Per-name notional = \$1mm; Loss fraction = $1 - 0.30 = 0.70$; Payment = \$700,000

4. $\alpha_i = (90, 91, 92, 90)/360$; Pay$_i$ = $50\text{mm} \times 0.0050 \times \alpha_i = (\$62,500, \$63,194, \$63,889, \$62,500)$

5. $U = (52 - 40) \times 4.25 = 51$ bp $= 0.51\%$ of notional (protection buyer receives)

6. $S = C + U/\text{RPV01} = 100 + (-250 \text{ bp})/4.00 = 100 - 62.5 = 37.5$ bp

7. Price below par → protection buyer *receives* upfront; amount = $(100 - 97.5)\% = 2.5\%$ of notional

8. RPV01 depends on spread (survival probability), so $U = (S-C) \times \text{RPV01}(S)$ requires root-finding

9. Unwind S6: $(38-40) \times 4.10 = -8.2$ bp → receive 0.082%; Enter S7: $(42-45) \times 4.40 = -13.2$ bp → pay 0.132%; Net: $(0.082 - 0.132)\% \times 80\text{mm} = -\$40,000$

10. Credit curves are typically upward sloping, so longer maturities imply higher spreads

11. (a) Constituent mismatch—names in one series but not the other; (b) Different maturities create duration mismatch

12. Roll reduces underlying index liquidity, increasing hedging costs for options that straddle the roll date

13. Sum = $(40-50) \times 4.2 + (50-50) \times 4.1 + (60-50) \times 4.0 = -42 + 0 + 40 = -2$; Intrinsic = $-2/3 = -0.67$ bp

14. Intrinsic spread = $(40 \times 4.2 + 50 \times 4.1 + 60 \times 4.0)/(4.2 + 4.1 + 4.0) = 613/12.3 = 49.84$ bp

15. Average = 50 bp; Intrinsic = 49.84 bp. Lower because higher-spread names (60 bp) have lower RPV01, reducing their weight.

16. Basis = Market − Intrinsic = 48 − 49.84 = −1.84 bp (index trades tight to intrinsic)

17. $\alpha < 1$ — we need to multiply all single-name spreads by a factor less than 1 to reduce the intrinsic spread from 35 to 32 bp.

18. Defaults between option initiation and expiry add value to the payer position. Each default is worth approximately $(1-R)/M$ divided by RPV01 in spread terms. If enough defaults occur, the effective strike is reduced.

19. Coupon = 100 bp (standard IG). Upfront ≈ $(85 - 100) \times 4.2 = -63$ bp = protection buyer pays 0.63% upfront.

20. The 800 bp name contributed heavily to the weighted average. After removal, intrinsic spread ≈ 44 bp (approximately, assuming flat RPV01s across remaining names). The exact calculation requires weighting by RPV01.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Index families, composition, number of names | O'Kane Table 10.1; Hull Section 25.3 |
| Semi-annual roll, IMM dates, maturity conventions | O'Kane Section 10.2; Hull Section 25.3 |
| CDX excludes restructuring; iTraxx includes it | O'Kane Section 10.2.1; Hull Section 25.3 footnote |
| Coupon paid quarterly on Actual/360 | O'Kane Section 10.2.1; Hull Section 25.4 |
| Default handling: settlement, accrued, notional reduction | O'Kane Section 10.2, Figure 10.2 |
| Upfront-spread conversion formula $U = (S-C) \times \text{RPV01}$ | O'Kane Equation 10.7; Hull Section 25.4 |
| Quoting conventions (spread vs. price) by index type | O'Kane Section 10.2.1 |
| Intrinsic value and intrinsic spread formulas | O'Kane Sections 10.3-10.5, Equations 10.5, 10.9 |
| Intrinsic spread is RPV01-weighted, below arithmetic average | O'Kane Section 10.5.1; Hull Section 25.3 |
| Index basis explanations and empirical data | O'Kane Section 10.5.2, Tables 10.5, 10.6 |
| Roll P&L drivers (composition, maturity) | O'Kane Section 10.2.2 |
| Liquidity concentration in on-the-run; bid-ask ~1 cent | O'Kane Section 10.2; footnote on bid-offer |
| Hedging off-the-run with on-the-run creates mismatch | O'Kane Section 10.5.3 |
| Black's model limitations for index options | O'Kane Section 11.3, points 1-3 |
| Arbitrage-free index option pricing framework | O'Kane Section 11.4, Pedersen (2003), Andersen (2006a) |
| Front-end protection formula | O'Kane Section 9.3.8, Equation 11.6 |
| Portfolio swap adjustment algorithm | O'Kane Section 10.6, Equations 10.11 |
| Fixed-point iteration for adjustment | O'Kane Section 10.6.1-10.6.3, algorithm steps |
| Adjustment ratio empirical data | O'Kane Table 10.7 |
| LCDX and LevX indices | O'Kane Sections 10.7.2-10.7.3 |
| Hull's price calculation example (100.27) | Hull Example 25.1 |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| Big Bang/Small Bang protocol details (Section 41.10) | Extends O'Kane's pre-2009 framework with post-crisis standardization |
| Central clearing mechanics | Post-2009 regulatory requirements (Dodd-Frank, EMIR) |
| P&L attribution framework (Section 41.9.4) | Standard trading desk practice; extends O'Kane's risk measures |
| Table 41.8: Pre/Post Big Bang comparison | Synthesized from multiple practitioner sources |

### (C) Reasoned Inference (Derived from A)

- Premium payment formula $N \cdot C \cdot \Delta$ is direct implementation of O'Kane's described mechanics
- CS01 approximation follows from differentiating the upfront formula
- Roll P&L worked example applies O'Kane's drivers quantitatively
- Intrinsic value worked example applies O'Kane's summation formula
- Default impact on spread follows from removing high-spread name from weighted average
- Exercise payoff derivation follows O'Kane's framework with explicit formula

### (D) Flagged Uncertainties

- **Exact bid-ask spread magnitudes for off-the-run:** O'Kane notes qualitative widening but doesn't quantify the ratio
- **Business day conventions and holiday calendars:** Require rulebook verification for specific index families
- **Whether index options "switch" underlying series at roll:** Requires ISDA confirmation language review
- **Current standard coupons:** Vary by series and evolve with market conditions
- **Exact CCP initial margin percentages:** Set by CCPs and vary with market conditions—consult current rulebooks
- **Specific clearing rulebook details:** Require CCP documentation verification

---

## Cross-References

- **Chapter 38** — CDS Contract Mechanics: Single-name foundation for index premium and protection legs
- **Chapter 40** — CDS Auction Process: How constituent defaults are settled
- **Chapter 42** — Bootstrapping Survival Curves: Used in intrinsic calculations and portfolio swap adjustment
- **Chapter 46** — Intrinsic Spread and Index Basis: Detailed treatment of basis calculation and trading
- **Chapter 47** — Index Hedging: Managing index vs. single-name hedge mismatches
- **Chapters 48-51** — Tranche Products: How index mechanics feed into structured credit pricing
