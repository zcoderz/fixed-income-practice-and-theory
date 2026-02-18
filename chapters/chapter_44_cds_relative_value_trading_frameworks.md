# Chapter 44: CDS Relative Value Trading Frameworks

---

## Introduction

## Learning Objectives
- Translate a CDS relative value (RV) idea into a precise object being traded (curve, basis, capital structure, equity-credit, index vs single-name).
- Map quotes $\rightarrow$ PV $\rightarrow$ a risk vector: CS01 (total + bucketed), JTD/VOD, recovery sensitivity, and theta (plus funding/liquidity exposures when relevant).
- Construct hedge ratios with explicit bump objects, units, and sign conventions—and interpret what residual risks remain.
- Run a minimal verification checklist (scenario tests + unit/sign checks) before trading or sizing.

Prerequisites: [Chapter 38](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 42](chapters/chapter_42_bootstrapping_cds_survival_curve.md), [Chapter 43](chapters/chapter_43_cds_risks_hedging.md), [Chapter 14](chapters/chapter_14_key_rate_dv01_bucket_exposures.md)  
Follow-on: [Chapter 45](chapters/chapter_45_cds_indices_structure_quoting_lifecycle.md), [Chapter 46](chapters/chapter_46_intrinsic_index_spread_and_index_basis.md), [Chapter 47](chapters/chapter_47_hedging_relative_value_cds_indices.md)

A corporate's 3-year CDS trades at 80bp, its 5-year at 120bp. Is that steep or flat? A hedge fund manager sees the cash bond trading 30bp wider than the CDS. Is that a buying opportunity or a warning sign? A credit trader notices that senior and subordinated CDS spreads have compressed. Should she put on a capital structure trade? The equity desk sees implied volatility on puts at 60% while the credit desk sees 5Y CDS at 150bp—is that relationship in line, or is one market mispricing the credit risk?

These questions sit at the heart of CDS relative value trading. Unlike directional credit positions that simply bet on spreads widening or tightening, relative value (RV) trades attempt to exploit mispricings between related instruments while hedging out broader market exposure. The appeal is obvious: if you can identify a temporary dislocation between the 3-year and 5-year CDS curve, why take outright default risk when you can trade the relationship instead?

The danger lies in what appears market-neutral but is not. A CDS position carries multiple distinct risks: spread-curve moves, rates, recovery assumptions, time decay, and (most importantly) a discontinuous jump on default. A trade that neutralizes one number (e.g., total CS01) can still be dominated by another (e.g., jump-to-default, funding/liquidity, or curve-shape risk).

This chapter takes a risk-first approach to CDS relative value. Building on CDS contract mechanics (Chapter 38), survival-curve bootstrapping (Chapter 42), and CDS risk measures (Chapter 43), we develop a systematic workflow for translating any RV idea into explicit exposures, designing hedges that target specific risks, identifying failure modes before they materialize, and constructing verification tests that expose hidden vulnerabilities. The same workflow applies to curve trades (steepeners/flatteners), cash-CDS basis trades, capital structure trades (senior vs subordinated), equity-credit RV (structural-model language), and index vs single-name RV (with details developed further in Chapters 45–47).

We begin with the core risk concepts that underpin all CDS positions: CS01, jump-to-default, recovery sensitivity, and theta. We then apply these concepts systematically to the major RV trade types, working through the mechanics of constructing and hedging each position. We conclude with equity-credit relative value—a domain that bridges the credit derivatives and equity derivatives desks through the Merton framework—and historical case studies that illuminate how seemingly "hedged" positions can fail catastrophically.

---

## 44.1 Core Risk Concepts for CDS Relative Value

Before constructing any relative value trade, lock down (i) the PV identity you are using and (ii) the exact definition of each risk number (what is bumped, by how much, and what is held fixed).

### 44.1.1 The CDS Mark-to-Market Identity (Spread Difference × Risky Annuity)

For a CDS position with contractual running spread $S(0,T)$ and maturity $T$, a standard mark-to-market (MTM) identity *per unit notional* is:

$$\boxed{V(t) = \bigl(S(t,T) - S(0,T)\bigr)\cdot \text{RPV01}(t,T)}$$

where:
- $S(t,T)$ is the current time-$t$ *par* CDS spread for the same maturity/conventions (use decimal per year; $120\text{bp}=0.012$).
- $S(0,T)$ is the contractual running spread agreed at inception (same units as $S$).
- $\text{RPV01}(t,T)$ is defined so that the remaining premium-leg PV of a running-spread CDS is $PV_{\text{prem}}(t,T)=S_0\cdot \text{RPV01}(t,T)$ *per unit notional* (discounted and survival-weighted). Equivalently, a 1bp-per-annum premium stream has PV $10^{-4}\cdot \text{RPV01}(t,T)$ per unit notional.
- To scale to a trade notional $N$, multiply the per-unit-notional PV by $N$.

For a short-protection (protection seller) position, the PV is the negative of the long-protection PV.

We will often write $S_0$ as shorthand for the contractual spread $S(0,T)$ when the maturity $T$ is clear.

In what follows, when we write dollar PVs and CS01s, we explicitly include $N$.

If premium is paid on dates $t_1,\dots,t_N=T$ with accrual fractions $\Delta_n$ (year fractions) and discount factors $Z(t,t_n)$, a common payment-date approximation that also captures premium accrued at default is:

$$\boxed{\text{RPV01}(t,T) = \frac{1}{2} \sum_{n=1}^{N} \Delta_n \, Z(t, t_n)\, \bigl(Q(t, t_{n-1}) + Q(t, t_n)\bigr)}$$

**Expand (intuition):** $\text{RPV01}$ is an *annuity multiplier*. Once you know it, spread *differences* translate into PV: if $\Delta S$ is in decimal per year, then $\Delta V \approx \Delta S \cdot \text{RPV01}$ per unit notional (multiply by $N$ for dollars).

**Check (units/sign):**
- Use consistent units: $1\text{bp}=10^{-4}$. If you keep spreads in bp in spreadsheets, convert to decimal before using formulas.
- $\Delta$ is in years; $Z$ and $Q$ are unitless; so $\text{RPV01}$ has units of years; (spread in 1/year) × (years) gives PV per unit notional; multiply by $N$ to get currency.
- Sign: if spreads widen (higher $S$), a protection buyer gains (higher $V$); a protection seller loses.

### 44.1.2 CS01 / Credit DV01: What Is Bumped, Units, and Sign

A CS01 number is meaningless unless you specify the bump design. In this chapter we define CS01 (Credit DV01) as the change in PV for a 1bp parallel increase in the CDS spread curve, with a negative sign so that a short-protection position has positive CS01.

- **Bump object:** the observable *par CDS spread curve* $S(t,T_i)$.
- **Bump size:** $+1\text{bp}=10^{-4}$ (additive in spread, expressed in decimal per year).
- **Recalibration:** rebuild the survival curve so the bumped quotes reprice (Chapter 42), then reprice the trade.
- **Units:** currency per 1bp.
- **Sign convention:** report CS01 positive for a protection seller (short protection).

$$\boxed{\text{CS01} := -\bigl(V(S+1\text{bp}) - V(S)\bigr)\;\;\approx\;\; -\frac{\partial V}{\partial S}\times 1\text{bp}}$$

For a near-par contract, combining the MTM identity with a first-order approximation gives the magnitude:

$$\boxed{|\text{CS01}| \approx N \cdot \text{RPV01}(t,T)\cdot 10^{-4}}$$

So under this convention:
- protection seller: $\text{CS01}\gt 0$
- protection buyer: $\text{CS01}\lt 0$

> **Pitfall — “What is being bumped?”:** A CS01 computed by bumping par spreads and re-bootstrapping is not the same object as a CS01 computed by bumping hazard rates or by bumping one tenor while holding others fixed.
> **Why it matters:** hedge ratios can look consistent in one system and be wrong in another.
> **Quick check:** when you compare CS01 across tools, confirm bump object, bump size, and whether the curve was rebuilt.

**Table 44.1: Risk Measures Used in This Chapter (definitions are methodology-dependent)**

| Risk Measure | What is bumped / shocked | Units | Typical sign (protection seller) |
|---|---|---|---|
| CS01 / Credit DV01 | +1bp parallel par-spread curve bump (rebuild curve) | currency per 1bp | $+$ |
| Bucket CS01 | +1bp bump to a maturity bucket / key quote (methodology-specific) | currency per 1bp | depends |
| VOD / JTD | immediate default shock | currency | $-$ |
| Rec01 | +1% absolute recovery bump (reprice/recalibrate as specified) | currency per 1% | depends |
| Theta | +1 day with curves held fixed | currency per day | $+$ |

### 44.1.3 Bucket CS01 (Credit Key-Rate Style)

Just as key-rate DV01s decompose interest rate sensitivity into maturity buckets, bucket CS01s decompose credit spread sensitivity. For a CDS position, we can split the RPV01 by time intervals:

| Bucket | RPV01 Contribution | Bucket CS01 |
|--------|-------------------|-------------|
| 0-2 years | $RPV01_{0-2}$ | $N \cdot 0.0001 \cdot RPV01_{0-2}$ |
| 2-5 years | $RPV01_{2-5}$ | $N \cdot 0.0001 \cdot RPV01_{2-5}$ |
| 5-10 years | $RPV01_{5-10}$ | $N \cdot 0.0001 \cdot RPV01_{5-10}$ |

**Why this matters:** A curve steepener that is CS01-neutral in aggregate may have large opposite-sign bucket exposures. This is the intended bet. But failing to report bucket exposures masks the true risk profile.

### 44.1.4 Jump-to-Default (JTD) and Value on Default (VOD)

VOD measures the PV jump if the reference entity defaults “right now” (immediate default). A useful decomposition is:

$$\boxed{\text{VOD} = \begin{cases} -V(t) - (1-R) + \Delta_0 S_0 & \text{for protection seller} \\ -V(t) + (1-R) - \Delta_0 S_0 & \text{for protection buyer} \end{cases}}$$

where $\Delta_0 S_0$ is the premium accrued since the previous coupon date (per unit notional; multiply by $N$ for dollars).

If the position is close to par $(V(t)\approx 0)$ and we ignore accrued premium:

$$\text{JTD}_{\text{buy protection}} \approx +N(1-R)$$
$$\text{JTD}_{\text{sell protection}} \approx -N(1-R)$$

**Expand (intuition):** if spreads drift wider for months before default, the MTM $V(t)$ tends to “pre-load” most of the eventual protection payment $(1-R)$. In that case, VOD tends to be small. Large VOD typically means default was *more sudden* relative to the prior MTM.

**Check (limiting case):** as you approach an anticipated default, $V(t)$ for a protection buyer approaches $(1-R)-\Delta_0 S_0$, making VOD $\to 0$ in the formula above.

**Critical for RV:** A curve trade with zero net CS01 can still have massive net JTD if the notionals are unbalanced. This is the single most important risk often overlooked in curve trades.

### 44.1.5 Recovery Sensitivity

Recovery enters CDS PV in two places:
1. **Default payment:** $(1-R)N$ (so recovery changes directly affect JTD/VOD).
2. **Calibration:** for a given spread curve, changing the assumed recovery changes the implied hazard/survival curve (Chapter 42), which feeds back into $\text{RPV01}$ and PV.

Recovery sensitivity affects both:
1. **The default payoff:** Higher recovery means smaller protection payment
2. **The calibrated hazard rate:** For fixed spreads, $h \approx S/(1-R)$ via the credit triangle (Chapter 43), so recovery changes affect the implied default probability

Empirically, recoveries vary substantially by seniority and are noisy. For CDS intuition, it is often enough to remember: (i) more senior debt tends to recover more, but (ii) dispersion is wide, and realized auction outcomes can differ from “point estimate” assumptions.

**Table 44.2: Illustrative Recovery Statistics by Bond Seniority (Altman et al., as reported in O’Kane)**

| Seniority (bonds) | Median recovery | Mean recovery | Std dev |
|---|---:|---:|---:|
| Senior unsecured | 42.27% | 34.89% | 26.6% |
| Senior subordinated | 32.35% | 30.17% | 25.0% |
| Subordinated | 31.96% | 29.03% | 22.5% |

**Practical point:** recovery risk is often small for day-to-day MTM, but it is first-order for default scenarios (JTD/VOD) and for trades that explicitly compare seniorities.

### 44.1.6 Theta (Time Decay)

Theta measures the sensitivity to the passage of time with curves unchanged:

$$\Theta = \frac{\partial V}{\partial t}$$

For a short protection position (receiving premium), theta is generally positive: the position gains value as time passes without default. For a long protection position, theta is negative.

**Expand (intuition):** theta is the net of (i) earning/owing running premium, (ii) the changing PV of the default leg as time passes, and (iii) the changing annuity value of future premiums. It is not constant through time, especially for distressed names.

**For RV:** Carry and rolldown decompositions use theta logic. "Carry" refers to net premium accrual; "rolldown" refers to the MTM change from moving along a static curve as maturity shortens. These decompositions are conditional on no curve movement and no default—they are accounting tools, not return forecasts.

---

## 44.2 The CDS-Cash Basis: Drivers and Dynamics

**Anchor (definition):** the CDS-cash basis compares a CDS spread quote to a *bond spread measure*:

$$\boxed{\text{CDS-cash basis} := S_{\text{CDS}} - S_{\text{Bond}}}$$

where $S_{\text{Bond}}$ is not uniquely defined (Z-spread, asset swap spread, par floater spread, etc.). Always state which spread measure you are using before you interpret the basis.

**Check (sign):** if a 5Y CDS trades at $120$bp and the bond Z-spread is $150$bp, the basis is $-30$bp. Under the convention above, a **negative** basis means CDS trades *tighter* than the bond spread measure.

A non-zero basis is not automatically a “mispricing.” It can reflect (i) contractual differences between a funded cash bond and an unfunded derivative hedge and (ii) market frictions: liquidity, financing, balance sheet constraints, and technical flows.

### 44.2.1 Fundamental Drivers of the Basis

Think of the bond as a funded position with coupon cashflows and the CDS as a standardized protection contract. Several contractual differences can move $S_{\text{CDS}}-S_{\text{Bond}}$ away from zero:

1. **Funding and balance-sheet usage:** buying a bond typically requires financing (repo, haircuts). Buying CDS protection does not require paying principal up front, but it can create margin/collateral needs and counterparty exposures.

2. **Delivery / settlement option:** in physical settlement the protection buyer may be able to deliver a cheapest-to-deliver (CTD) obligation; in auction settlement the realized payout depends on the auction final price. Either way, the *set of deliverables* matters.

3. **Credit event definition vs “bond default”:** CDS credit events can be broader or different than the bond’s payment default, depending on the contract and jurisdiction (e.g., restructuring language).

4. **Loss-on-default scaling:** a CDS protection payment is typically $N(1-R)$. A bond purchased at dirty price $P_{\text{dirty}}$ has a default loss tied to $P_{\text{dirty}}-R$ per 100 face (and the choice of spread measure matters).

**Check (default-hedge sizing vs. CS01 sizing):** if you want a CDS leg to hedge a bond’s *default* loss in a stylized cash-settlement picture, match $\left(\frac{P_{\text{dirty}}}{100}-R\right)N_{\text{bond}} \;\approx\; (1-R)N_{\text{CDS}}$,
so $N_{\text{CDS}} \approx \frac{\frac{P_{\text{dirty}}}{100}-R}{1-R}\,N_{\text{bond}}$. For example, if $P_{\text{dirty}}=85$ and $R=40\%$, then $N_{\text{CDS}}\approx 0.75\,N_{\text{bond}}$. Sizing instead to make a package “CS01-neutral” can produce a very different notional and leave a large jump at default.

5. **Accrued cashflows around default:** standard CDS settlement includes premium accrued to the default date; bond coupon accrual treatment differs and can create default-scenario P&L mismatches.

### 44.2.2 Market Factors Driving the Basis

Even when the contractual differences are small, the basis can be dominated by liquidity and technicals:

- **Relative liquidity:** CDS liquidity tends to concentrate at standard maturities (often the 3Y/5Y/7Y/10Y points), while a specific cash bond can become off-the-run.
- **Shorting and hedging demand:** it is often operationally easier to express a short-credit view via CDS than by borrowing and shorting a bond.
- **Structured credit and other flow:** large one-way flows (issuance hedging, macro hedging) can move CDS and cash differently.
- **Repo and financing conditions:** specials/GC, haircuts, and the ability to roll funding can dominate the economics of “buy bond + buy CDS” packages.
- **Convertible and optioned-credit flows:** convert arb and capital structure strategies can create persistent demand for CDS protection.

> **Desk Reality:** A basis package can screen “CS01-neutral with positive carry.”
> **Common break:** Funding and haircut shocks (and bond liquidity) sit outside CS01 and can dominate P&L even if the CDS hedge behaves as designed.
> **What to check:** Track carry as bond carry $-$ CDS premium $-$ financing; shock repo/haircuts and bid/ask; verify JTD sizing and the settlement/deliverable assumptions.

> **Deep Dive: The Negative Basis Trade ("The Package")**
>
> **Setup (stylized):** if $S_{\text{CDS}} \lt S_{\text{Bond}}$, you can buy the bond, buy CDS protection, and (in a frictionless sketch) keep the spread difference as carry.
>
> **Why it can exist:** in stress, cash bonds can cheapen because investors need cash or balance sheet, while CDS can remain cheaper-to-trade and easier-to-short.
>
> **The catch:** you still own a funded, mark-to-market bond position. Financing costs, haircuts, and liquidity can move sharply against you.

> **Analogy: The Lock-In**
>
> The negative basis trade can look like picking up a small, steady carry pickup—until the exit door narrows.
>
> - The “pickup”: the spread difference.
> - The “door”: bond liquidity + funding roll.
> - The “trap”: you may be forced to unwind during the worst funding/liquidity conditions.

### 44.2.3 Why Basis Trades Can Fail

A basis trade is not “bond risk minus default risk.” It is a multi-leg position with several failure modes:

1. **Funding blowout:** If repo markets seize, the funding leg of the trade can become prohibitively expensive or impossible to roll.

2. **Correlation breakdown:** The bond and CDS may not move together. A liquidity shock can widen bond spreads while CDS spreads remain stable, or vice versa.

3. **Hedge mismatch:** If the CDS notional is sized for CS01 neutrality rather than JTD neutrality, default produces a P&L discontinuity. Recovery and settlement assumptions can also differ across legs.

4. **Delivery option uncertainty:** At default, the cheapest deliverable may trade far from expectations, affecting realized recovery.

5. **Counterparty risk:** The CDS hedge is only as good as the counterparty providing it. In systemic stress, counterparty credit becomes correlated with the underlying credit.

### 44.2.4 Funding Cost Deep Dive

The economics of a basis trade depend critically on funding levels. Consider how changes in repo rates affect trade viability:

In a simplified carry decomposition (ignoring MTM, haircuts, and bid/ask), the net carry is approximately:

$$\text{Net carry (bp)} \approx \text{Bond spread} - \text{Repo funding cost} - \text{CDS premium}$$

**Table 44.3: Funding Level vs Net Carry (Illustrative)**

| Repo Funding Cost | Bond Spread (L+X) | CDS Premium | Net Carry (bp) |
|-------------------|-------------------|-------------|----------------|
| L + 10bp | 150bp | 120bp | +20bp positive |
| L + 25bp | 150bp | 120bp | +5bp positive |
| L + 50bp | 150bp | 120bp | -20bp (breakeven) |
| L + 100bp | 150bp | 120bp | -70bp (losing) |
| L + 150bp | 150bp | 120bp | -120bp (severely losing) |

The table reveals a funding cliff: a trade that earns carry at benign financing levels can become deeply negative as repo levels and haircuts rise. This is why “basis” positions should be sized and stress-tested as *funding/liquidity trades*, not just as credit trades.

---

## 44.3 Credit Curve Trades: Steepeners and Flatteners

Curve trades express views on the shape of the credit term structure. They are constructed by taking opposite positions at different maturities.

### 44.3.1 Defining Curve Positions

For maturities $T_1 \lt T_2$:

| Trade | Definition | Instrument Position |
|-------|------------|---------------------|
| **Steepener** | Profits if $S(T_2) - S(T_1)$ increases | Buy protection at $T_2$, sell protection at $T_1$ |
| **Flattener** | Profits if $S(T_2) - S(T_1)$ decreases | Sell protection at $T_2$, buy protection at $T_1$ |

The steepener profits when the curve gets steeper (long end widens relative to short end). The flattener profits when the curve flattens.

### 44.3.2 CS01-Neutral Construction

To hedge against parallel curve moves, we choose notionals such that net CS01 equals zero:

$$N_1 \cdot \text{CS01}^{(1)} + N_2 \cdot \text{CS01}^{(2)} = 0$$

where $\text{CS01}^{(1)}$ and $\text{CS01}^{(2)}$ have opposite signs (one buy, one sell protection). Solving:

$$\boxed{\frac{N_1}{N_2} = -\frac{\text{CS01}^{(2)}}{\text{CS01}^{(1)}} = -\frac{\text{RPV01}(T_2)}{\text{RPV01}(T_1)}}$$

Since $\text{RPV01}(T_2) \gt \text{RPV01}(T_1)$ for $T_2 \gt T_1$, we need more notional at the short maturity.

### 44.3.3 The JTD Problem in Curve Trades

CS01 neutrality does not imply JTD neutrality. Consider a 5Y vs 1Y steepener with:
- Buy 5Y protection: $N_5 = 10\text{mm}$, $\text{RPV01}(5) = 4.7$
- Sell 1Y protection: notional chosen for CS01 neutrality

The CS01-neutral notional ratio is approximately $4.7 / 1.0 = 4.7$, so $N_1 \approx 47\text{mm}$.

**JTD calculation:**
- 5Y buy protection JTD: $+10 \times 0.60 = +6.0\text{mm}$ (gain at default)
- 1Y sell protection JTD: $-47 \times 0.60 = -28.2\text{mm}$ (loss at default)
- **Net JTD: $-22.2\text{mm}$**

This trade is massively short default. If the reference entity defaults, the loss overwhelms any curve P&L. This is the central warning: **curve trades can be dominated by default-event risk even when CS01-neutral**.

### 44.3.4 When Curve Trades Historically Failed

Curve trades have historically failed in several scenarios:

1. **Sudden default:** A company defaults before expected. The CS01-neutral steepener with large net short JTD produces catastrophic losses.

2. **Curve inversion:** Credit curves can invert (front end wider than back) during acute stress, as the market prices high near-term default probability.

3. **Liquidity divergence:** The 5Y point typically has better liquidity than off-the-run maturities. In stress, liquidity can evaporate at the short end while remaining at 5Y, causing basis moves that overwhelm the curve bet.

4. **Roll and technical effects:** CDS indices roll semi-annually. Around roll dates, the market's preferred maturity points shift, creating technical moves that are unrelated to credit fundamentals.

### 44.3.5 Carry and Rolldown for Curve Trades

Under a "curves unchanged" assumption, we can decompose expected P&L:

**Carry:** Net premium accrual from premium receipts minus payments. For a steepener (buy long end, sell short end), you typically pay premium on the long leg and receive on the short leg. Whether net carry is positive depends on the spread levels and notional ratios.

**Rolldown:** The MTM change from the remaining maturities shortening along a static curve. If the curve is upward sloping, buying protection at 5Y and letting it roll to 4.9Y produces a gain (you're now at a lower spread point on the curve).

**Caution:** These decompositions are conditional on no curve movement and no default. They are accounting tools, not return forecasts.

---

## 44.4 Capital Structure Trades: Senior vs Subordinated

Capital structure trades exploit the relationship between CDS referencing different seniority tiers of the same issuer.

### 44.4.1 The Spread Ratio Approximation

A common first-pass approximation links senior and subordinated spreads through recovery:

$$\frac{S_{\text{sub}}}{S_{\text{senior}}} \approx \frac{1 - R_{\text{sub}}}{1 - R_{\text{senior}}}$$

This is a **very rough** relationship in practice; treat it as an intuition check, not as a pricing identity.

Using the recovery data from Table 44.2:
- Senior unsecured: mean recovery 34.89%
- Subordinated: mean recovery 29.03%

The theoretical ratio would be: $(1-0.29)/(1-0.35) = 0.71/0.65 = 1.09$. But empirically, subordinated spreads are often 1.5x to 3x senior spreads, implying the market prices additional factors beyond the simple recovery differential.

### 44.4.2 Why the Relationship Is Very Rough

Several factors cause deviations from the theoretical spread ratio:

1. **Liquidity differences:** Senior CDS is far more liquid than subordinated CDS for most issuers.

2. **Recovery uncertainty and legal process:** realized recoveries are noisy and can deviate from simplified “priority waterfall” intuition due to the legal process, negotiations, and instrument-specific features.

3. **Technical supply/demand:** Bank regulatory capital rules, CDO structuring, and index composition create supply/demand imbalances that vary by seniority.

4. **Different default triggers:** Some credit events may trigger protection on one seniority tier but not another, particularly around restructuring.

### 44.4.3 Constructing a Capital Structure Trade

A typical trade expresses a view on the senior-sub spread differential:

- **Compression trade:** Buy senior protection, sell subordinated protection. Profits if the sub/senior spread ratio narrows.
- **Decompression trade:** Sell senior protection, buy subordinated protection. Profits if the sub/senior spread ratio widens.

**Sizing:** CS01-neutral sizing uses the same hedge ratio logic:

$$\frac{N_{\text{senior}}}{N_{\text{sub}}} = -\frac{\text{CS01}_{\text{sub}}}{\text{CS01}_{\text{senior}}}$$

**JTD consideration:** Because recovery assumptions differ, JTD depends on both notional and assumed recovery:
- Sub buy protection JTD: $+N_{\text{sub}}(1-R_{\text{sub}})$
- Senior sell protection JTD: $-N_{\text{senior}}(1-R_{\text{senior}})$

Even with CS01 neutrality, the different recoveries create JTD imbalance.

**Check (toy capital-structure JTD mismatch):** suppose you put on a compression trade that is CS01-neutral by construction: buy $N_{\text{senior}}=USD 10\text{mm}$ of senior protection with $RPV01_{\text{senior}}=4.5$, and sell subordinated protection with $RPV01_{\text{sub}}=4.0$ so that $N_{\text{sub}}\approx 10\times 4.5/4.0= USD 11.25\text{mm}$. If you assume $R_{\text{senior}}=40\%$ and $R_{\text{sub}}=20\%$, then the net jump at default is approximately $JTD_{\text{net}} \approx +10\times 0.60 \;-\; 11.25\times 0.80 \;=\; -USD 3.0\text{mm}$,
so the “CS01-neutral” trade is still meaningfully short default.

### 44.4.4 When Capital Structure Trades Widen or Tighten

Events that affect the senior-sub spread:

**Compression (sub/senior ratio tightens):**
- Credit improvement: as default risk falls, the absolute spreads compress and the recovery differential matters less
- Regulatory changes favoring sub debt

**Decompression (sub/senior ratio widens):**
- Credit deterioration: higher default probability amplifies the recovery differential
- Distress: market prices in that sub will recover less than expected
- Liquidity crisis: sub CDS liquidity evaporates first

### 44.4.5 Event Risk: LBO and M&A Effects

> **Practitioner Note: The LBO Trade**
>
> Leveraged buyout (LBO) announcements create distinctive patterns in credit spreads that sophisticated traders anticipate. When a company is taken private via LBO, the new owners typically add substantial debt to the capital structure, which affects existing creditors asymmetrically.
>
> **The Mechanics:**
> - New secured debt is layered on top of existing obligations
> - Existing senior unsecured debt becomes effectively subordinated
> - Cash flow available for debt service is stretched across more claims
> - Covenants may be weakened or removed
>
> **Spread Reactions (stylized):**
> - Senior spreads can widen materially on an LBO announcement (magnitude depends on leverage, structure, and market regime)
> - Subordinated spreads often widen more, so the sub/senior ratio can increase (decompression)
>
> **The "LBO Trade":**
> Before LBO announcements, event-driven funds identify likely targets based on:
> - Strong free cash flow (can service new debt)
> - Low existing leverage (room for more debt)
> - Stable, predictable business (attractive to PE sponsors)
> - Cheap valuation (attractive entry point)
>
> They then buy protection on senior CDS and/or put on curve steepeners, anticipating the spread widening and decompression that accompany leveraging events.
>
> **Curve Implications:**
> LBO risk manifests in the 5Y point but not the 1Y point—deals take time to consummate. This creates curve steepening opportunities: buy 5Y protection, sell 1Y protection, anticipating that announcement effects hit the long end first.
>
> **Warning:** Event risk is distinct from credit deterioration. A company can be a high-quality credit yet have significant LBO risk precisely because its quality makes it an attractive PE target.

---

## 44.5 Equity-Credit Relative Value: The Merton Framework

Equity-credit relative value compares two markets’ implied views of default risk: equity (via equity price/volatility and options) and credit (via CDS spreads). A structural model provides a *translation layer*—not a guarantee of arbitrage.

### 44.5.1 The Merton Model: Equity as a Call Option

**Anchor:** in the basic Merton model, equity is the residual claim on firm assets. If the firm has debt face value $D$ due at time $T$, then equity at $T$ is:

$$E_T = \max(V_T - D, 0)$$

Under the model assumptions, equity today is priced like a Black–Scholes call option on firm value:

$$\boxed{E_0 = V_0 N(d_1) - D e^{-rT} N(d_2)}$$

where:
$$d_1 = \frac{\ln(V_0/D) + (r + \sigma_V^2/2)T}{\sigma_V \sqrt{T}}$$
$$d_2 = d_1 - \sigma_V \sqrt{T}$$

and $N(\cdot)$ is the cumulative normal distribution function.

### 44.5.2 The Equity Volatility Link

The equity and asset volatility are linked through the option delta:

$$\boxed{\sigma_E E_0 = N(d_1) \sigma_V V_0}$$

This equation, combined with the equity pricing formula, gives two equations in two unknowns $(V_0,\sigma_V)$. Given observable equity value $E_0$ and equity volatility $\sigma_E$, you can solve for the unobservable firm value and asset volatility numerically.

**Expand (what is observable vs not):**
- Observable: $E_0$ and $\sigma_E$.
- Latent: $V_0$ and $\sigma_V$.

**Check (leverage amplifies equity vol):** because $N(d_1)\in(0,1)$, the relation $\sigma_E E_0 = N(d_1)\sigma_V V_0$ says equity volatility is a *levered* version of asset volatility. When leverage is high (small $E_0$ relative to $V_0$), a given $\sigma_V$ can correspond to a much larger $\sigma_E$.

**Check (sanity):** higher leverage (smaller $V_0/D$) or higher $\sigma_V$ lowers $d_2$ and increases model default risk.

### 44.5.3 Distance to Default

Distance-to-default (DD) is the model’s standardized “buffer” to the default barrier:

$$\boxed{\text{Distance to Default} = d_2 = \frac{\ln(V_0/D) + (r - \sigma_V^2/2)T}{\sigma_V \sqrt{T}}}$$

In the model, $N(-d_2)$ is the (risk-neutral) probability that the firm’s assets finish below $D$ at $T$.

**Expand (how to use DD):**
- Treat DD as a *monotone credit risk score*: DD down $\Rightarrow$ default probability up $\Rightarrow$ credit spreads “should” be wider, all else equal.
- Mapping DD into an actual CDS spread requires additional assumptions (debt structure, recovery convention, and term structure). Use DD as an RV diagnostic and a scenario-test input, not as a one-line “fair spread.”

**Check (numerical):** Example E (Section 44.9) walks through a simple calibration and DD calculation.

### 44.5.4 Turning the Signal into a Trade (and Why It Breaks)

An equity-credit RV workflow:

1. **Equity side:** infer $(V_0,\sigma_V)$ or DD from $(E_0,\sigma_E)$.
2. **Credit side:** infer an implied hazard/spread level from CDS (using your CDS curve bootstrap).
3. **Compare:** decide whether “equity-implied credit risk” looks rich/cheap versus CDS.
4. **Express:** trade a hedged package (e.g., long equity protection + short CDS protection, or the reverse), and explicitly track residual risks.

**What breaks (risk-first):**
- **Jump risk and path dependence:** structural models are smooth; real credit has jumps, discrete events, and legal uncertainty.
- **Correlation regime shifts:** equity-credit correlation can change quickly in stress.
- **Financing and margin:** the equity leg can have adverse MTM and margin calls even if terminal value converges.
- **Instrument mismatch:** CDS references specific obligations and settlement mechanics; equity options trade under different microstructure and liquidity.

### 44.5.5 CreditGrades (One Useful Refinement)

Basic Merton often produces very small short-dated credit spreads because default requires diffusing to a fixed barrier. CreditGrades-style models address this by making the default barrier uncertain (stochastic), which generates more realistic short-maturity default risk while still tying credit to equity volatility. In practice, treat CreditGrades (and similar models) as ways to build *relative value diagnostics* and scenario tests, not as black-box “fair spread” engines.

---

## 44.6 Index vs Single-Name Relative Value

The relationship between a CDS index and its constituent single-name CDS creates another RV opportunity. This section provides a brief treatment; Chapter 45 covers index mechanics in depth.

### 44.6.1 Intrinsic Spread and Index Basis

**Anchor:** the *intrinsic value* of an index is the sum of the PVs of its constituent single-name CDS. The *intrinsic spread* is the spread level that makes that intrinsic PV equal to zero.

The **index basis** is:

$$\text{Index basis} = S_{\text{index}} - S_{\text{intrinsic}}$$

With constituent spreads $S_m$ and risky annuities $RPV01_m$, a common approximation is the RPV01-weighted average:

$$\boxed{S_{\text{intrinsic}} = \frac{\sum_{m=1}^{M} S_m \cdot RPV01_m}{\sum_{m=1}^{M} RPV01_m}}$$

**Expand (intuition):** this is *not* a simple average of spreads. Names with larger $\text{RPV01}$ (often the wider, riskier names) carry more weight in “intrinsic,” because a 1bp move on those names is worth more PV.

**Expand (mechanics):** $\text{RPV01}$ is larger when a name is expected to survive longer and pay more premium (typically *tighter* names), and smaller when default is more likely (often *wider/distressed* names). So the intrinsic spread is usually **pulled toward the tighter names** because distressed constituents tend to get down-weighted by their smaller $\text{RPV01}$.

**Check (toy):** if all names have the same $\text{RPV01}$, intrinsic reduces to the simple average. If one name is very wide but has a much smaller $\text{RPV01}$, it gets down-weighted and intrinsic can sit well below the arithmetic mean. For example, 4 names at 50 bp with $\text{RPV01}=4$ and 1 distressed name at 1000 bp with $\text{RPV01}=1$ give:
- Simple average $=(4\times 50 + 1000)/5=240$ bp
- Intrinsic $=(4\times 50\times 4 + 1000\times 1)/(4\times 4 + 1)\approx 106$ bp

**Table 44.4: Intrinsic vs Average Spread**

| Spread Type | Formula | When Different |
|-------------|---------|----------------|
| Simple Average | $\bar{S} = \frac{1}{M}\sum_{m=1}^{M} S_m$ | All names equal weight |
| Intrinsic | $S_{\text{intrinsic}} = \frac{\sum S_m \cdot RPV01_m}{\sum RPV01_m}$ | Higher-$\text{RPV01}$ names get more weight (distressed names often get less) |

The difference between the intrinsic spread and the simple average can be material when spread dispersion is high.

### 44.6.2 Drivers of Index Basis

The index basis can be positive or negative depending on:

1. **Liquidity premium:** The index is more liquid than individual constituents, so it may trade tighter.

2. **Transaction costs:** Replicating the index via singles incurs more transaction costs.

3. **Supply/demand:** Macro hedgers often use indices, creating technical demand.

4. **CDO issuance:** Synthetic CDO issuance creates supply of index protection.

5. **Skew effects:** When a few names are much wider than the rest, the intrinsic spread is pulled higher, potentially creating a negative basis.

### 44.6.3 Trading the Index Basis

The trade involves going long (short) the index versus short (long) the constituents:

- **Long basis:** Buy index protection, sell protection on constituents
- **Short basis:** Sell index protection, buy protection on constituents

Sizing requires matching the overall CS01, typically by scaling constituent positions to match the index notional divided by the number of names.

---

## 44.7 Historical Case Studies

Understanding past failures is essential for risk management. These case studies illustrate how "hedged" positions can produce catastrophic losses.

### 44.7.1 Case Study: Funding-Liquidity Unwind in a Stress Episode

> **Practitioner Note: The Basis Trade Failure Mode**
>
> **The Setup (stylized):** Negative basis trades can look like "money machines." You buy a bond that screens cheap versus CDS, buy CDS protection, and expect to earn carry plus basis convergence.
>
> **What goes wrong in stress:**
> - **Financing deteriorates:** repo rates and/or haircuts rise and funding may not roll.
> - **Liquidity vanishes in cash bonds:** selling the bond can become costly; marks gap out; bid-ask widens.
> - **Margin and risk limits shorten the holding period:** interim losses trigger margin calls and position cuts.
> - **Hedge effectiveness is not enough:** even if CDS hedges default risk, P&L can be dominated by funding and liquidity.
>
> **Lesson:** Basis trades are not just credit trades. They are funding/liquidity trades disguised as arbitrage. Always decompose P&L into (1) basis/spread, (2) carry/theta, and (3) funding.
>
> **What risk reports often show vs reality:**
> - Reports: CS01 neutral, small VaR, positive carry
> - Reality: exposure to haircuts, funding roll risk, and liquidity spirals

### 44.7.2 Case Study: Equity–Credit Disconnect in Bankruptcy (Illustrative)

> **Practitioner Note: Timing Risk in Equity–Credit RV**
>
> It can happen that a company's credit trades at distressed levels while its equity price rallies sharply. At first glance this looks like a free arbitrage ("equity must be worthless if the debt is impaired"), but in practice the trade can be dangerous.
>
> **Why equity can rally while credit stays distressed:**
> - Equity is a residual claim with limited liability; in structural models it is option-like.
> - Technicals matter: borrow constraints, squeezes, and flow-driven markets.
> - Restructuring outcomes are uncertain; headline risk can move equity faster than credit.
>
> **Why the trade can lose money even if equity ultimately goes to zero:**
> - Mark-to-market on the short equity leg can be very adverse.
> - Borrow and funding costs can rise; margin calls can force exit.
> - **Convergence timing** dominates: you must survive to see terminal value.
>
> **Lesson:** Treat equity–credit RV as a multi-leg, path-dependent trade. Size for interim volatility, not just terminal payout.

---

## 44.8 The Risk-First Framework: Putting It Together

For any CDS relative value position, apply this systematic framework:

### 44.8.1 Step 1: Identify the Object

Explicitly state what relationship you are trading:

| Object | Description |
|--------|-------------|
| Single-name curve point | Level of 5Y CDS vs model/sector |
| CDS curve spread | 5Y-1Y steepener/flattener |
| Cash-CDS basis | Bond Z-spread vs CDS spread |
| Capital structure | Senior vs subordinated |
| Equity-credit | Equity vol vs CDS spread (Merton-based) |
| Index basis | Index vs intrinsic |

### 44.8.2 Step 2: Decompose into Exposures

For the identified trade, compute:

| Exposure | Metric |
|----------|--------|
| Spread risk (total) | Total CS01 (USD per bp) |
| Spread risk (bucketed) | Bucket CS01 by maturity |
| Default risk | JTD/VOD (USD) |
| Recovery risk | Recovery DV01 (USD per 1% R) |
| Rates risk | IR DV01 (USD per bp) |
| Time decay | Theta (USD per day) |
| Equity exposure (for Merton trades) | Equity delta, vega |
| Liquidity/technical | Qualitative assessment |
| Funding exposure | Sensitivity to repo/funding costs |

### 44.8.3 Step 3: Design Hedges

For each exposure, specify the hedge instrument and what risk it targets:

| Target | Hedge Instrument |
|--------|------------------|
| Total CS01 | Opposite CDS position |
| Bucket CS01 | Multiple CDS maturities |
| JTD | Balanced notional positions |
| IR DV01 | Interest rate swaps |
| Equity delta | Stock or equity options |
| Recovery | Often unhedgeable; scenario management |
| Funding | Funding derivatives (if available) |

### 44.8.4 Step 4: Identify Failure Modes

Before entering the trade, enumerate what can go wrong:

1. **Default event:** JTD dominates
2. **Curve shape moves:** Bucket CS01 mismatch
3. **Basis blowout:** Cash-CDS disconnection
4. **Liquidity evaporation:** Unable to exit or rebalance
5. **Recovery surprise:** Auction final price differs from assumption
6. **Roll/technical:** Index rolls, liquidity point shifts
7. **Correlation breakdown:** Equity-credit relationship breaks (Merton trades)
8. **Funding crisis:** Repo or margin costs spike (basis trades)
9. **Convergence delay:** Arbitrage takes longer than funding allows

### 44.8.5 Step 5: Define Verification Tests

Run the following scenarios before and during the trade:

| Test | Description |
|------|-------------|
| Parallel spread shock | All spreads +10bp, +50bp |
| Curve twist | Front +10bp, back +50bp; reverse |
| Default scenario | Apply VOD/JTD |
| Recovery shock | R ± 10% absolute |
| Repricing check | Linear CS01 vs full repricing |
| Funding shock | Repo +100bp, +200bp |
| Equity move (Merton) | Stock ±20% |
| Correlation break | Equity moves opposite to credit |

---

## 44.9 Worked Examples

### Example A: CS01-Neutral 5Y vs 1Y Steepener (With a Concrete Schedule)

**Context**
- Trade: curve steepener (long-end widens vs front-end).
- Construction goal: neutralize *parallel* spread moves (total CS01 $\approx 0$), then explicitly measure what is left (JTD, curve shape).

**Timeline (Make Dates Concrete)**
- Trade date: 2026-01-19
- Effective date: 2026-01-20 (T+1 calendar; see Chapter 38 for schedule conventions)
- Premium payment grid (unadjusted): 20 Mar / 20 Jun / 20 Sep / 20 Dec
- First coupon after effective: 2026-03-20
- Scheduled termination dates (first standard date at least tenor after effective; business-day adjusted per confirmation):
  - 1Y: 2027-03-20
  - 5Y: 2031-03-20

**Inputs**
- Par CDS spreads (running, per annum): $S_{1Y}=80$bp, $S_{5Y}=160$bp
- Recovery assumption for analytics: $R=40\%$
- Premium day count: ACT/360
- Notional: buy 5Y protection $N_5=10$mm; sell 1Y protection $N_1$ unknown

**Outputs (What You Produce)**
- CS01-neutral hedge ratio $N_1$ (bump: +1bp parallel par-spread curve; curve rebuilt; units: USD per bp; sign: protection seller positive)
- Net JTD at immediate default (USD)

**Step-by-step**
1. Compute/obtain $\text{RPV01}$ for each leg from your CDS analytics (bootstrapped survival curve + discounting; Chapter 42). For illustration, use:
   - $\text{RPV01}(1Y)\approx 0.993$
   - $\text{RPV01}(5Y)\approx 4.74$

2. Compute CS01 magnitudes:

   $$|\text{CS01}| \approx N \cdot \text{RPV01} \cdot 10^{-4}.$$

3. Solve for CS01 neutrality:

   $$N_1 \approx N_5 \cdot \frac{RPV01_{5Y}}{RPV01_{1Y}} \approx 10\text{mm}\times\frac{4.74}{0.993}\approx 47.7\text{mm}.$$

4. Compute net JTD (jump approximation; ignore accrued premium):

   $$\text{Net JTD} \approx +N_5(1-R) - N_1(1-R) \approx -(47.7-10)\times 0.60 \approx -USD 22.6\text{mm}.$$

**Cashflows (table)**
(Stub premium from 2026-01-20 to 2026-03-20 has 59 days, so $\Delta = 59/360$.)

| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-03-20 | $-N_5 \cdot 0.0160 \cdot \frac{59}{360}$ $\approx -26{,}222$ | Pay 5Y premium (buy protection) |
| 2026-03-20 | $+N_1 \cdot 0.0080 \cdot \frac{59}{360}$ $\approx +62{,}540$ | Receive 1Y premium (sell protection) |
| default date | $+N_5(1-R)$ | 5Y protection payment received if default occurs |
| default date | $-N_1(1-R)$ | 1Y protection payment owed if default occurs |

**P&L / Risk Interpretation**
- If spreads move in parallel by +1bp, the two legs’ CS01s approximately offset (that is the hedge design).
- If the curve steepens (long end wider vs front end), the steepener gains.
- If the name defaults, the trade is **short default** by about USD 22.6mm for $R=40\%$: JTD dominates.

**Sanity Checks**
- Units: $1\text{bp}=10^{-4}$; $|CS01| \approx N \times RPV01 \times 10^{-4}$ gives currency per bp.
- Sign: buy protection has negative CS01; sell protection has positive CS01 under this chapter’s convention.
- Limit: because $\text{RPV01}(5Y)\gt\text{RPV01}(1Y)$, CS01-neutral sizing requires $|N_1|\gt|N_5|$, creating the JTD problem.

**Debug Checklist (When Your Result Looks Wrong)**
- Are you using the same CS01 *methodology* on both legs (par-spread bump + curve rebuild)?
- Did you convert bp to decimal correctly (1bp = $10^{-4}$)?
- Did you align schedule conventions (effective date, standard dates, stubs, ACT/360)?
- Did you compute JTD with the same recovery convention and include accrued premium only if your JTD/VOD definition includes it?

---

### Example B: Curve Trade P&L Scenarios

Using Example A's steepener, compute P&L under various scenarios.

**Scenario 1: Parallel +20bp**

$$\Delta V = 4,740 \times 20 + (-4,740) \times 20 = 0$$

**Scenario 2: Steepening (front +5bp, back +25bp)**

$$\Delta V = 4,740 \times 25 + (-4,740) \times 5 = 4,740 \times 20 = +USD 94,800$$

**Scenario 3: Flattening (front +25bp, back +5bp)**

$$\Delta V = 4,740 \times 5 + (-4,740) \times 25 = -USD 94,800$$

**Scenario 4: Default**

$$\Delta V = \text{Net JTD} = -USD 22.6\text{mm}$$

The default scenario dominates all spread scenarios by two orders of magnitude.

---

### Example C: Cash-CDS Basis Trade with Financing

**Setup:**
- Bond: USD 10mm face, price 102, duration 4.5, Z-spread 150bp
- CDS: 5Y spread 120bp, RPV01 = 4.7
- Basis: CDS 30bp cheap to bond
- Financing: repo at Libor + 25bp

**Step 1: Size for JTD neutrality**

Bond loss at default (price 102, recovery 40): $(1.02 - 0.40) \times 10 = USD 6.2\text{mm}$

CDS notional for JTD match: $6.2 / 0.60 = USD 10.33\text{mm}$

**Step 2: Compute CS01s**

Bond DV01: $4.5 \times 10.2 \times 0.0001 = USD 4,590/\text{bp}$

CDS CS01: $10.33 \times 0.0001 \times 4.7 = USD 4,855/\text{bp}$

**Step 3: Monthly carry**

Bond spread income: $150\text{bp} \times 10.2\text{mm} / 12 = USD 12,750$

CDS premium: $120\text{bp} \times 10.33\text{mm} / 12 = USD 10,330$ (paid)

Financing cost: $25\text{bp} \times 10.2\text{mm} / 12 = USD 2,125$

**Net monthly carry: $12,750 - 10,330 - 2,125 = +USD 295$**

**Step 4: Funding stress scenario**

If repo rises to L+150bp:

Financing cost: $150\text{bp} \times 10.2\text{mm} / 12 = USD 12,750$

**Net monthly carry: $12,750 - 10,330 - 12,750 = -USD 10,330$**

The trade flips from positive to deeply negative carry when funding costs spike.

---

### Example D: Senior vs Subordinated Trade

**Setup:**
- Senior 5Y CDS: $S = 120\text{bp}$, $R = 40\%$
- Sub 5Y CDS: $S = 250\text{bp}$, $R = 20\%$
- Buy sub protection: $N_{\text{sub}} = 10\text{mm}$
- Sell senior protection: notional for CS01 neutrality

**Step 1: Compute hazards and RPV01s**

Senior: $h = 0.012/0.60 = 0.02$, $\text{RPV01} \approx 4.76$

Sub: $h = 0.025/0.80 = 0.03125$, $\text{RPV01} \approx 4.63$

**Step 2: CS01 neutrality**

$$\frac{N_{\text{senior}}}{10} = \frac{4.63}{4.76} \implies N_{\text{senior}} = 9.73\text{mm}$$

**Step 3: JTD**

$$\text{JTD}_{\text{sub}} = +10 \times 0.80 = +USD 8.0\text{mm}$$
$$\text{JTD}_{\text{senior}} = -9.73 \times 0.60 = -USD 5.84\text{mm}$$
$$\text{Net JTD} = +USD 2.16\text{mm}$$

The trade is net long default due to the different recoveries, even with CS01 neutrality.

**Step 4: Spread ratio check**

Theoretical ratio: $(1-0.20)/(1-0.40) = 0.80/0.60 = 1.33$

Market ratio: $250/120 = 2.08$

The market prices sub materially wider than the simple recovery model predicts, suggesting either (a) the market expects lower sub recovery than 20%, (b) liquidity premium in sub, or (c) the simple model is inadequate.

---

### Example E: Merton Model Calibration

**Setup (illustrative numbers):**
- Equity value: $E_0 = USD 3$ million
- Equity volatility: $\sigma_E = 80\%$
- Debt due in 1 year: $D = USD 10$ million
- Risk-free rate: $r = 5\%$

**Step 1: Set up simultaneous equations**

From Merton:
$$E_0 = V_0 N(d_1) - D e^{-rT} N(d_2)$$
$$\sigma_E E_0 = N(d_1) \sigma_V V_0$$

**Step 2: Solve numerically**

Using solver (e.g., Excel Solver minimizing sum of squared residuals):

$$V_0 = USD 12.40 \text{ million}$$
$$\sigma_V = 21.23\%$$

**Step 3: Compute default probability**

$$d_2 = \frac{\ln(12.40/10) + (0.05 - 0.2123^2/2)(1)}{0.2123 \sqrt{1}} = 1.14$$

$$P(\text{default}) = N(-d_2) = N(-1.14) = 12.7\%$$

**Interpretation:** The firm has distance-to-default $DD=d_2=1.14$ standard deviations and a model default probability $N(-d_2)=12.7\%$ over 1 year. Turning this into a CDS “fair spread” requires additional assumptions (recovery convention, term structure, and instrument details); use it as an RV diagnostic, not as a one-line arbitrage.

---

### Example F: Three-Leg Trade for JTD Neutrality

To achieve both CS01 and JTD neutrality, we need three instruments.

**Setup:**
- Buy 5Y protection: $N_5 = 10\text{mm}$ (given)
- Sell 3Y protection: $N_3$ (to be determined)
- Buy 1Y protection: $N_1$ (to be determined)

**Constraints:**

1. JTD neutral: $N_5 + N_3 + N_1 = 0$ (with signs for buy/sell)
2. CS01 neutral: $N_5 \cdot \text{RPV01}(5) + N_3 \cdot \text{RPV01}(3) + N_1 \cdot \text{RPV01}(1) = 0$

Using RPV01(1) = 0.99, RPV01(3) = 2.92, RPV01(5) = 4.74:

From JTD neutral: $N_1 = -10 - N_3$

Substitute into CS01:
$$10 \times 4.74 + N_3 \times 2.92 + (-10 - N_3) \times 0.99 = 0$$
$$47.4 + 2.92 N_3 - 9.9 - 0.99 N_3 = 0$$
$$37.5 + 1.93 N_3 = 0$$
$$N_3 = -19.4\text{mm}$$

Therefore: $N_1 = -10 - (-19.4) = +9.4\text{mm}$

**Result:**
- Buy 5Y: 10mm
- Sell 3Y: 19.4mm
- Buy 1Y: 9.4mm

Net notional = 0 (JTD neutral); Net CS01 $\approx$ 0. The remaining exposure is pure curve shape.

---

## 44.10 Practical Notes and Common Pitfalls

### 44.10.1 Convention Mismatches

| Pitfall | Description |
|---------|-------------|
| **CS01 sign confusion** | Different systems use different sign conventions. Always verify whether "CS01" means signed (value change) or unsigned (magnitude). |
| **Quote bump vs hazard bump** | Bumping par spreads and recalibrating gives different bucket decomposition than bumping hazard rates directly. |
| **Recovery assumption mismatch** | Bond valuations may use different recovery assumptions than CDS analytics. |
| **Day count differences** | Bond accrual and CDS accrual may use different conventions. |

### 44.10.2 Implementation Issues

1. **Survival curve interpolation:** Different interpolation schemes (linear on $-\log Q$, piecewise constant hazard) give different bucket CS01s.

2. **Accrual at default:** VOD calculations must include accrued premium; ignoring this creates small but systematic errors.

3. **Roll dates:** Index rolls and single-name CDS rolls on different schedules can create basis moves.

4. **Liquidity cliff:** 5Y is typically liquid; 4Y and 6Y may not be. Hedging curve positions requires trading at illiquid points.

5. **Merton calibration sensitivity:** Small changes in equity volatility input can produce large changes in implied default probability.

### 44.10.3 Why RV Trades Fail

The most common failure modes:

1. **Underestimating JTD:** Curve trades with large notional imbalances
2. **Funding assumptions:** Basis trades that assume stable repo funding
3. **Correlation spikes:** "Hedged" positions that become correlated in stress
4. **Liquidity evaporation:** Inability to rebalance or exit
5. **Model error:** Analytics that don't match market pricing conventions
6. **Convergence timing:** Arbitrage takes longer than funding allows (equity-credit RV, basis trades)
7. **Correlation breakdown:** Equity-credit relationship deviates from Merton predictions

---

## Summary

- Start from a clear PV identity: the CDS MTM is approximately $(S_{\text{market}}-S_0)\times \text{RPV01}\times N$. Do unit and sign checks.
- A “CS01” number is only meaningful once you lock the bump object, bump size, curve rebuild rule, units, and sign convention (and bucket CS01 matters for curve trades).
- CS01-neutral is not default-neutral: JTD/VOD is a discontinuous risk that can dominate RV trades built from multiple maturities or seniorities.
- The CDS-cash basis is multi-factor: contractual differences + liquidity/financing/technicals. Many basis packages behave like funding/liquidity trades, not pure credit trades.
- Index RV depends on *intrinsic*: an RPV01-weighted spread benchmark. Index basis is the difference between traded index spread and intrinsic.
- Equity-credit RV uses a structural model as a translation layer (equity $\rightarrow$ DD $\rightarrow$ credit risk signal), but model/correlation/margin risks are first-order.
- A pre-trade verification suite (parallel shocks, twists, default, recovery, funding, and repricing checks) is the fastest way to surface hidden exposures.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CS01 | $-\bigl(V(S+1\text{bp})-V(S)\bigr)$ under a stated bump methodology | Primary spread risk metric; must be comparable across systems |
| Bucket CS01 | CS01 decomposed by maturity | Reveals curve shape exposure |
| JTD/VOD | Value change on immediate default | Discontinuous risk that can dominate |
| CDS-cash basis | $S_{\text{CDS}}-S_{\text{Bond}}$ (bond spread measure must be specified) | Packages mix credit, funding, and liquidity |
| Intrinsic spread | RPV01-weighted spread of index constituents | Benchmark for index RV trades |
| Carry / rolldown | “Curves unchanged” P&L decomposition tools | Useful for P&L explain; not a forecast |
| Recovery DV01 | PV change per 1% recovery bump (specify what is held fixed) | Critical for default scenarios and seniority trades |
| Merton model | Equity = call option on firm assets | Links equity vol to credit spreads |
| Distance to default | Standard deviations to default barrier | Key metric for equity-credit RV |
| CreditGrades | Merton with uncertain barrier | Generates realistic short-dated spreads |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $S(t,T)$ | Par CDS spread at time $t$ for maturity $T$ |
| $\text{RPV01}(t,T)$ | Risky PV01 from $t$ to $T$ |
| $Q(t,T)$ | Survival probability to time $T$ |
| $h(t)$ | Hazard rate at time $t$ |
| $Z(t,T)$ | Risk-free discount factor |
| $R$ | Recovery rate |
| $V(t)$ | CDS mark-to-market value |
| $N$ | Notional |
| VOD | Value on default |
| JTD | Jump-to-default (often used interchangeably with VOD) |
| $S_{\text{intrinsic}}$ | Intrinsic spread of a CDS index |
| $V_0$ | Firm value (Merton model) |
| $\sigma_V$ | Asset volatility (Merton model) |
| $E_0$ | Equity value (Merton model) |
| $\sigma_E$ | Equity volatility |
| $d_1, d_2$ | Black-Scholes parameters in Merton model |
| DD | Distance to default |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the CDS mark-to-market formula? | $V \approx (S_{\text{market}} - S_{\text{contract}}) \cdot \text{RPV01} \cdot N$ |
| 2 | How is Credit DV01 (CS01) defined? | $-(V(S+1\text{bp}) - V(S))$, with negative sign for convention |
| 3 | Why is bucket CS01 important for curve trades? | Total CS01 can be zero while bucket exposures are large and opposite |
| 4 | What is VOD for a protection buyer? | $-V(t) + (1-R) - \Delta_0 S_0$ |
| 5 | Why can CS01-neutral curve trades have large JTD? | Notional imbalance: longer maturity needs less notional for same CS01 |
| 6 | What is the CDS-cash basis? | $S_{\text{CDS}} - S_{\text{Bond}}$ (spread measure must be specified) |
| 7 | Name three fundamental drivers of the basis | Funding, delivery option, loss on default |
| 8 | Name three market drivers of the basis | Relative liquidity, CDO technical, demand for protection |
| 9 | What is a curve steepener? | Buy long-maturity protection, sell short-maturity protection |
| 10 | What is a curve flattener? | Sell long-maturity protection, buy short-maturity protection |
| 11 | What is the senior/sub spread ratio approximation? | $S_{\text{sub}}/S_{\text{senior}} \approx (1-R_{\text{sub}})/(1-R_{\text{senior}})$ |
| 12 | Why is this ratio very rough in practice? | Liquidity, recovery uncertainty, legal/process complexity, technicals |
| 13 | What is the intrinsic spread of an index? | RPV01-weighted average spread that makes sum of constituent values zero |
| 14 | What is index basis? | Index spread minus intrinsic spread |
| 15 | What is "carry" in a CDS position? | Net premium accrual under curves unchanged |
| 16 | What is "rolldown" in a CDS position? | MTM change from maturity shortening on static curve |
| 17 | How many instruments to achieve CS01 and JTD neutrality? | Three (two constraints, three unknowns) |
| 18 | What is the biggest failure mode in curve trades? | JTD dominance: default produces outsized loss |
| 19 | Why do basis trades fail? | Funding blowout, correlation breakdown, JTD mismatch |
| 20 | What is the key test for any RV position? | Scenario suite: parallel, twist, default, recovery, repricing |
| 21 | What is the mean recovery for senior unsecured bonds (Altman data)? | 34.89% (median 42.27%) |
| 22 | What is the mean recovery for subordinated bonds (Altman data)? | 29.03% (median 31.96%) |
| 23 | Why is VOD called a measure of "unexpected shock"? | If spreads widen gradually before default, V(t) already reflects default risk, so VOD is small |
| 24 | What determines the sign of theta for a CDS position? | Short protection (receiving premium) has positive theta; long protection has negative theta |
| 25 | What is Recovery DV01? | Change in MTM for 1% absolute change in recovery rate assumption |
| 26 | What is capital structure arbitrage (equity-credit)? | Exploiting mispricings between equity derivatives and CDS based on Merton framework |
| 27 | What is the Merton equity formula? | $E = V N(d_1) - D e^{-rT} N(d_2)$ — equity is a call on firm assets |
| 28 | What is distance-to-default? | Number of standard deviations firm value must fall to trigger default: $d_2$ |
| 29 | What is CreditGrades? | Extension of Merton with uncertain default barrier, generates short-dated spreads |
| 30 | What is the LBO trade? | Buying protection anticipating leverage event that widens spreads |
| 31 | Why did capital structure arbitrage fail for some? | Model risk, correlation breakdown, jump risk, convergence timing |
| 32 | Why can basis trades fail in stress? | Funding/haircut shocks and bond illiquidity can dominate even if the CDS hedge behaves as designed |
| 33 | How does convertible arb affect the CDS basis? | Creates natural buying of CDS protection (to hedge converts), can tighten basis |
| 34 | What is event risk in credit? | Risk of M&A, LBO, or other corporate action that affects credit quality |
| 35 | How does equity volatility relate to credit spreads (Merton)? | $\sigma_E E = N(d_1) \sigma_V V$ — higher equity vol implies higher asset vol and wider spreads |
| 36 | How can equity-credit RV fail? | Equity can rally/squeeze while credit stays distressed; timing and margin/funding can force exit |

---

## Mini Problem Set

1. A 5Y CDS has RPV01 = 4.5 and a 2Y CDS has RPV01 = 1.9. For a 5Y vs 2Y steepener with $N_5=10\text{mm}$ (buy protection), what $N_2$ (sell protection) achieves CS01 neutrality?

2. Using Problem 1, compute net JTD assuming $R=40\%$.

3. A bond has Z-spread 180bp and the 5Y CDS trades at 150bp. What is the CDS-cash basis under the convention $\text{basis}=S_{\text{CDS}}-S_{\text{Bond}}$, and how do you interpret the sign?

4. Explain why a negative basis package (buy bond, buy CDS protection) can lose money even if the basis converges.

5. Senior CDS trades at 100bp ($R_{\text{senior}}=40\%$), subordinated trades at 200bp. What recovery assumption for sub would make the spread ratio match $\frac{S_{\text{sub}}}{S_{\text{senior}}}\approx\frac{1-R_{\text{sub}}}{1-R_{\text{senior}}}$, and what does your answer imply?

6. For a three-leg trade with $N_1=8$, $N_3=-20$, $N_5=12$ (positive = buy protection), verify JTD neutrality and describe what risk is left if CS01 is also neutralized.

7. Explain why VOD can be small when default is widely anticipated (even though default is a large economic event).

8. Design a scenario test suite for a senior-sub compression trade (include at least one liquidity or “can’t rebalance” scenario).

9. An index has 100 names. The simple average spread is 80bp, but the intrinsic spread is 95bp. What does this imply about the spread distribution and/or $\text{RPV01}$ weights?

10. A trader has a 5Y CDS position with CS01 = USD 5,000/bp and theta $=+USD 200/\text{day}$. How many days of carry equals a 1bp parallel spread move?

11. Outline how you would infer $(V_0,\sigma_V)$ (or DD) from equity value and equity volatility in the Merton setup.

12. Using Merton, if $V_0=USD 25$ million, $D=USD 20$ million, $\sigma_V=25\%$, $T=2$ years, and $r=4\%$, compute the distance-to-default $d_2$.

### Solution Sketches (Selected)

- **1.** $N_2 = 10\times 4.5/1.9 \approx 23.7\text{mm}$ (sell protection in 2Y to offset the 5Y CS01).
- **2.** Net JTD $\approx (10-23.7)\times(1-0.40)\approx -USD 8.2\text{mm}$ (short default).
- **4.** Even with basis convergence, the package can lose from funding/haircuts, bond liquidity and bid/ask, and timing/margin constraints; the CDS leg hedges default, not financing and microstructure.
- **7.** If spreads widen before default, the MTM $V(t)$ tends to move toward the eventual protection payment $(1-R)$. The “surprise jump” at default is then small because much of the loss was already priced in.
- **10.** $5{,}000/200 = 25$ days.

---

## References

- O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (CDS MTM identity, RPV01, CS01/JTD/VOD, basis, intrinsic spread)
- Hull, *Options, Futures, and Other Derivatives* (Merton-style structural model and equity-credit intuition)
- Hull, *Risk Management and Financial Institutions* (credit risk modeling and risk framing)
- Neftci, *Principles of Financial Engineering* (negative basis intuition and funding/liquidity linkage)
- Gatheral, *The Volatility Surface* (CreditGrades and structural-model refinements)
- Merton, “On the Pricing of Corporate Debt: The Risk Structure of Interest Rates” (1974)
