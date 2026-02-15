# Chapter 39: CDS Credit Events and Settlement

---

## Introduction

When a reference entity hits a CDS-relevant “default”, a CDS trade has to answer two operational questions:

- **Did the contract trigger?** (a **credit event** determination)
- **What cash actually changes hands?** (the **settlement** mechanics)

Both are documentation-driven. A bond can be in distress without triggering a CDS credit event. And even after a trigger, the payout depends on settlement method and (under physical settlement) what is deliverable.

Prerequisites: [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 35 — Default, Recovery, Credit Events](chapters/chapter_35_default_recovery_credit_events.md)  
Follow-on: [Chapter 40 — The CDS Auction Process](chapters/chapter_40_cds_auction_process.md), [Chapter 43 — Risks in CDS and Hedging Strategies](chapters/chapter_43_cds_risks_hedging.md)

## Learning Objectives
- Distinguish **credit event** (legal trigger) from “default” as used in ratings and headlines.
- Translate **settlement method** → **cashflows** → **payout** with explicit units (price points vs fractions).
- Explain why **restructuring** is different (price dispersion across deliverables → a **delivery option**).
- Compute and interpret **accrued premium on default** in a dated timeline.
- Map post‑2009 **standardization** (auction + fixed coupons/upfront) to what your P&L/ops actually book.

This chapter covers:
- Credit events and the hard/soft distinction
- Physical vs cash settlement and the payout formulas
- Deliverables, cheapest‑to‑deliver (CTD), and the delivery option
- Restructuring clauses and why they change spreads/CTD economics
- Accrued premium on default and why it matters for net settlement
- A short, mechanism-level view of CDS‑cash basis and settlement risks

---

## 39.1 Credit Events: The Contractual Trigger

### 39.1.1 Credit Event vs Default

**Anchor:** In CDS documentation, a **credit event** is the legal trigger for the protection leg. It is related to “default”, but it is *not the same thing* as a rating‑agency “default” definition.

**Expand:** A CDS is a contract, so “did it trigger?” depends on the definitions incorporated in the trade (which credit events are included, whether restructuring is included, what obligations are deliverable, and how settlement works). Market conversation often says “default” when it really means “a CDS credit event occurred.”

**Check:** When you hear “the name defaulted,” ask: *which* credit event is alleged, under *which* documentation, and *how* the trade settles.

### 39.1.2 The Six Standard Credit Events

One commonly used list of CDS credit events (often grouped into **hard** vs **soft**) is:

| Credit Event | Type | Description |
|--------------|------|-------------|
| **Bankruptcy** | Hard | Corporate becomes insolvent or is unable to pay its debts. Not relevant for sovereign issuers. |
| **Failure to pay** | Hard | Failure to make due payments, taking into account a grace period. |
| **Obligation acceleration** | Hard | Obligations become due and payable early due to default or similar trigger. Used mostly in emerging market contracts. |
| **Obligation default** | Hard | Obligations have become due prior to maturity. Rarely used. |
| **Repudiation/moratorium** | Hard | Reference entity or government rejects or challenges validity of obligations. Used in emerging market sovereign CDS. |
| **Restructuring** | Soft | Changes in debt obligations associated with credit deterioration, excluding favorable renegotiations. |

In this framing, **restructuring** is the key “soft” event: after a restructuring, different obligations can keep trading at different prices (so “what is deliverable?” becomes economically important).

### 39.1.3 Why Hard Events Make Debt Trade at One Price

**Anchor:** After a hard credit event, the market often treats many obligations as converging to a similar “recovery” level.

**Expand:** Intuitively, once an issuer is in a hard-default state (e.g., bankruptcy/insolvency), *maturity* matters less than *seniority* because the payoff is dominated by the default process. The price of different senior unsecured obligations can compress toward a common recovery level (up to microstructure and legal details).

**Check:** If most eligible deliverables cluster at one price level post‑event, the delivery option is small (CTD ≈ “anything”).

### 39.1.4 Why Restructuring Is Different

**Anchor:** After a restructuring, obligations can keep trading with a **term structure of prices** (dispersion across maturities/coupons).

**Expand:** A restructuring changes the contract terms but the issuer continues operating. That means “short vs long” and “high coupon vs low coupon” can still matter for valuation, so different deliverables can trade at meaningfully different prices.

**Check:** Price dispersion across deliverables is exactly the condition that makes the delivery option economically valuable (Section 39.4).

> **Decision Tree: Is it a Credit Event?**
>
> 1.  **Failure to pay?** (after any applicable grace period; above any payment requirement threshold) → potentially **YES**.
> 2.  **Bankruptcy/insolvency‑type trigger?** → likely **YES**.
> 3.  **Restructuring‑type trigger?** → depends on whether restructuring is included and on the restructuring clause.
> 4.  If unclear: treat as a **documentation question** first (and, for standard contracts, a determinations/auction workflow question).

### 39.1.5 Grace Periods and Payment Thresholds

Failure‑to‑pay definitions are typically designed to avoid “triggering on noise.” Two common ideas (documentation‑specific) are:

- **Grace periods:** a missed payment may have a cure window before it qualifies as a failure‑to‑pay credit event.
- **Materiality / payment requirement thresholds:** tiny or disputed payments may be excluded from triggering.

> **Desk Reality: The Grace Period Gotcha**
>
> **Desk Reality:** A missed payment can create a period where bonds trade “distressed” while the CDS trigger is still uncertain.
> **Common break:** assuming protection pays immediately on the first missed coupon.
> **What to check:** the failure‑to‑pay definition (including any grace period/materiality thresholds) in your trade documentation.

---

## 39.2 Physical Settlement

### 39.2.1 Mechanics

**Anchor:** **Physical settlement:** The protection buyer delivers face value of deliverable obligations to the protection seller. In return, the protection seller makes a simultaneous payment of the face value in cash to the protection buyer.

The economics are straightforward. If the buyer delivers $N$ face value of obligations and receives $N$ in cash, the buyer's economic gain equals $N$ minus the market value of what was delivered. For an obligation trading at price $P_i$ (per 100 of face):

$$\boxed{\Pi_{\text{phys}}(i) = N\left(1 - \frac{P_i}{100}\right)}$$

**Expand:** Physical settlement is “par minus what you hand over.” If you do not already own a deliverable, you must **source** one to deliver.

**Check (units/bounds):**
- $P_i$ is a **price in points per 100** (e.g., $P_i=35$ means \$35 per \$100 face).
- If $0 \le P_i \le 100$, then $0 \le \Pi_{\text{phys}}(i) \le N$.

### 39.2.2 The Settlement Timeline

At a high level, physical settlement is an operational workflow:

1. A credit event is determined/notice is served and the trade is triggered.
2. The protection buyer elects physical settlement and identifies intended deliverables.
3. Delivery occurs and the seller pays par.

Exact notice requirements, deadlines, and evidence standards are **documentation-specific**; treat them as part of the trade’s product specification.

### 39.2.3 The Sourcing Problem and Short Squeezes

Physical settlement creates a practical problem when the outstanding amount of CDS protection that expects physical delivery is large relative to the supply of deliverable obligations. Protection buyers may be forced to buy deliverables in a hurry, pushing prices up and reducing the effective payout.

**Check (direction and magnitude):** A short squeeze that pushes the CTD price up by $\Delta P$ points reduces the physical-settlement payout for long protection by $N\,\Delta P/100$. For example, on $N=\$100\text{mm}$, a 5‑point squeeze in CTD reduces payout by $\$5\text{mm}$.

> **Desk Reality:** Under physical settlement, “CTD” is not only a valuation concept—it can become an operational constraint if deliverables are scarce.

---

## 39.3 Cash Settlement

### 39.3.1 Mechanics

**Anchor:** **Cash settlement:** The protection seller pays the protection buyer the face value of the protection minus the recovery price of the reference obligation in cash. With a cash‑settlement final price $P_{\text{final}}$ in points per 100 face:

The payoff formula is:

$$\boxed{\Pi_{\text{cash}} = N\left(1 - \frac{P_{\text{final}}}{100}\right) = N(1 - R_{\text{settle}})}$$

where $P_{\text{final}}$ is the "final price" or "recovery price" (per 100 face) and $R_{\text{settle}} = P_{\text{final}}/100$ is the implied recovery fraction.

**Expand:** Cash settlement removes the need to source bonds, but it introduces a new object: the **final price** used to compute the payout.

**Check (sanity):**
- If $P_{\text{final}} = 100$ (no loss): $\Pi_{\text{cash}} = 0$ ✓
- If $P_{\text{final}} = 0$ (total loss): $\Pi_{\text{cash}} = N$ ✓

> **Pitfall — Price points vs recovery fraction:** “38” means **38 points per 100**, not 38%. Convert with $R_{\text{settle}}=P_{\text{final}}/100$ before multiplying by notional.

### 39.3.2 How the Final Price Is Determined

The critical question for cash settlement is: who decides $P_{\text{final}}$, and how?

Documentation may reference a dealer poll and/or an ISDA-organized auction process (covered in detail in Chapter 40).

A useful mental model is that the cash settlement price is intended to be representative of what physical settlement with optimal delivery would realize (i.e., it embeds CTD economics).

### 39.3.3 When Physical and Cash Settlement Align

If the cash settlement final price equals the market value of the obligation that would optimally be delivered under physical settlement, then both methods produce the same payout:

$$P_{\text{final}} = P_{\text{CTD}} \implies \Pi_{\text{cash}} = \Pi_{\text{phys}}$$

This alignment is the economic rationale for using "the mid-market value of the cheapest deliverable bond" in auctions—it attempts to replicate what physical settlement would achieve without requiring actual delivery.

### 39.3.4 Why Cash Settlement Became Standard

Cash settlement is especially valuable when physical settlement is impractical because deliverables are scarce relative to CDS notionals.

For example, when Lehman defaulted in September 2008, there was about \$400 billion of CDS contracts and \$155 billion of Lehman debt outstanding. The cash payout to protection buyers (determined by an ISDA auction process) was $91.375 \%$ of principal.

---

## 39.4 The Delivery Option and Cheapest-to-Deliver

### 39.4.1 Deliverables: Why a Basket Exists

**Anchor:** Usually, a CDS specifies that a number of different bonds (and sometimes loans) can be delivered following a credit event. All deliverable obligations must be **pari passu or senior** to the reference obligation.

**Expand:** The purpose of a deliverable basket is practical: one standard CDS can hedge **many** obligations of the same reference entity without writing a separate CDS for each bond/loan. That improves liquidity, but it also creates choice: “which deliverable should be used for settlement?”

**Check:** If there is only one deliverable, then the basket collapses to a single name and there is no deliverability optionality.

### 39.4.2 The Delivery Option and CTD

**Anchor:** Usually, a CDS specifies that a number of different bonds can be delivered following a credit event. If deliverables do not trade at the same percentage of face value after the event, this gives the holder of a CDS a **cheapest‑to‑deliver bond option**. Under physical settlement, the rational choice is to deliver the **cheapest‑to‑deliver (CTD)** obligation:

$$P_{\text{CTD}} = \min_{i \in \mathcal{D}} P_i$$

where $\mathcal{D}$ is the set of eligible deliverables and $P_i$ is the market price (points per 100 face) of deliverable $i$.

With optimal delivery, the physical settlement payout is:

$$\boxed{\Pi_{\text{phys}} = N\left(1 - \frac{P_{\text{CTD}}}{100}\right)}$$

**Expand (replication story):** If you own a particular bond you were hedging, you are not forced to deliver *that* bond. You can:
1. Sell the bond you own at its market price.
2. Buy the CTD obligation.
3. Deliver the CTD for par.

The incremental value of being able to “switch deliverables” is the delivery option.

**Check (toy CTD switch):** Suppose the bond you were hedging trades at 43, but another deliverable trades at 37 (prices in points per 100). Per \$100 face, delivering CTD instead of your hedged bond increases payout by $43-37=6$ points.

**Check (dollar conversion):** “6 points” is 6% of notional. On $N=\$100\text{mm}$, the incremental value of switching to CTD is about $\$6\text{mm}$.

### 39.4.3 Valuing the Delivery Option (Simple Framework)

If $P_{\text{hedged}}$ is the price of the asset you were hedging, a simple expression for the delivery option value is:

$$\boxed{V_{\text{DO}} = N \times \frac{P_{\text{hedged}} - P_{\text{CTD}}}{100}}$$

**Expand:** $V_{\text{DO}}$ is non‑negative: you can always choose to deliver the hedged asset itself (so $P_{\text{CTD}}\le P_{\text{hedged}}$ when the hedged asset is deliverable). The option becomes large when post‑event prices are dispersed across deliverables.

**Check (limit case):** If all deliverables trade at the same price after the event, then $P_{\text{CTD}} \approx P_{\text{hedged}}$ and $V_{\text{DO}}\approx 0$.

### 39.4.4 Why the Seller Cares

**Anchor:** The protection seller is effectively **short** the delivery option: they must pay par regardless of which eligible obligation the buyer delivers.

**Expand:** That is why contracts with broader deliverability (especially around restructuring events) tend to have higher spreads: the buyer pays for the embedded option.

**Check:** If restructuring is excluded (No‑Restructuring), then this particular source of post‑event dispersion is reduced, and the delivery option embedded in *restructuring* settlement is correspondingly smaller.

---

## 39.5 Restructuring Clauses and the Market's Response

### 39.5.1 Why Restructuring Clauses Exist (A Conseco‑Style Mechanism)

**Anchor:** Restructuring is a “soft” credit event: different obligations can continue trading at different prices after the event. That price dispersion makes the delivery option economically important.

**Expand:** Historically, market participants learned that “deliverability” is not a trivial detail. A commonly cited mechanism is:
- A hedger owns a **short‑dated** loan/bond that holds up relatively well.
- The deliverable basket also contains a **long‑dated** deep‑discount bond.
- If restructuring triggers and old deliverability rules allow it, the protection buyer can sell the higher‑priced asset they own, buy the cheaper long‑dated bond, and deliver that for par—receiving more than the loss on the asset they were hedging.

One early restructuring episode often discussed in this context is Conseco (September 2000), where the maturity/price dispersion across deliverables made this “switch to CTD” trade economically meaningful.

**Check (toy sanity):** If your hedged asset trades at 95 but an eligible long bond trades at 70, the ability to deliver CTD can add $95-70=25$ points of incremental value per 100 face—far larger than bid/ask noise.

### 39.5.2 The Four Common Restructuring Clauses

To reduce (or remove) restructuring‑driven delivery optionality, market documentation uses different restructuring clauses. A useful high‑level summary is:

| Clause | Short Name | Description |
|--------|------------|-------------|
| **Old-Restructuring** | Old-Re (OR) | The original standard; maximum maturity deliverable is 30 years. |
| **Modified Restructuring** | Mod-Re (MR) | US standard; restricts maturity of deliverables after restructuring (reduces delivery option); applies when triggered by the buyer. |
| **Modified-Modified Restructuring** | Mod-Mod-Re (MMR) | European standard; similar restriction but somewhat broader (can allow deliverables out to 60 months in some cases); allows conditionally transferable obligations; applies when triggered by the buyer. |
| **No-Restructuring** | No-Re (NR) | Removes restructuring as a credit event. |

### 39.5.3 The Spread Ordering

**Anchor:** The choice of restructuring clause should affect the CDS spread because it changes the value of the **delivery option** embedded in settlement.

**Expand:** All else equal, broader deliverability makes the buyer’s option more valuable and tends to widen spreads:
- **Old‑Re** has the broadest deliverable set (largest potential delivery option).
- **No‑Re** removes restructuring as a credit event (smallest restructuring‑driven optionality).
- **Mod‑Re / Mod‑Mod‑Re** sit in between by restricting maturity and (for Mod‑Mod‑Re) transferability of eligible deliverables after a restructuring.

**Check:** Don’t treat this as a strict ranking. The realized spread differences depend on (i) the probability of a restructuring event, and (ii) post‑event price dispersion across eligible deliverables.

### 39.5.4 Regional Differences: CDX vs iTraxx

**Anchor:** Whether restructuring is included (and which clause applies) is a **contract choice**, and it differs across products and regions.

**Expand:** A useful way to think about this is “single‑name standards” vs “index standards”:
- In the US, **Mod‑Re** became a common market standard for single‑name CDS, and some contracts trade **No‑Re**.
- In Europe, **Mod‑Mod‑Re** is commonly used.
- In indices, iTraxx Europe indices commonly include restructuring, while North American CDX index protection is often consistent with **No‑Re** (triggered only by bankruptcy or failure to pay).

**Check:** When comparing spreads across two CDS quotes (or across an index vs a single name), first verify they share the same restructuring clause. If not, you are comparing “credit risk + option” to “credit risk with a different option.”

---

## 39.6 Accrued Premium at Default

**Anchor:** CDS premium is typically paid **in arrears**. In the standard contract, following a credit event, the protection buyer must pay the fraction of the premium which has accrued since the previous premium payment date. This “accrued premium on default” is part of the **premium leg** and is handled separately from (or netted against) the protection payment.

For contractual spread $s$ (annualized, in decimals), notional $N$, and accrual fraction $\alpha$ from the last premium date to the credit‑event date:

$$\boxed{\text{Accrued Premium} = N \cdot s \cdot \alpha}$$

**Expand:** Mechanically, think of the CDS as “insurance coverage for a time interval.” Even if the credit event happens mid‑period, you had protection from the last premium date up to the event date—so that premium accrues. In a simple netting view for cash settlement:

$$\text{Net cash to buyer} \approx N\left(1-\frac{P_{\text{final}}}{100}\right) - N\cdot s\cdot \alpha$$

where $P_{\text{final}}$ is the cash‑settlement final price (points per 100).

**Check (rule of thumb):** If the credit event happens roughly mid‑coupon, accrued premium is roughly half a coupon:
\[
\text{Accrued}\approx \tfrac{1}{2}\,N\,s\,\Delta(\text{full period}).
\]
This is only an intuition aid—production systems accrue to the actual event date using the contract day count.

**Check (dated accrual):** Consider the example timeline used in Section 39.9: last premium date March 20, 2023 and credit event May 20, 2023. Using ACT/360 for illustration, $\alpha=61/360$. If $N=\$100\text{mm}$ and $s=90$ bp $=0.009$, then accrued premium is:

$$100{,}000{,}000 \times 0.009 \times \frac{61}{360} = \$152{,}500.$$

> **Pitfall — Forgetting accrued premium on default:** Mixing “payout” with “net settlement.”
> **Why it matters:** You can be off by \$10k–\$100k+ on a single-name trade (and much more on large notionals) if you ignore the final accrual.
> **Quick check:** Compute $\alpha$ from concrete dates and confirm accrued premium has the same sign as a normal premium payment (buyer pays seller).

---

## 39.7 Post-2009 Market Standardization

### 39.7.1 The “Big Bang” Standardization (April 2009)

**Anchor:** On April 8, 2009, a “Big Bang” occurred in the market for CDS contracts and the way in which they are traded. The standardization had three main parts: **auction hardwiring**, **standardizing trading conventions**, and **central clearing**.

**Expand:** You can think of this as turning CDS settlement into a predictable “protocol” rather than a bespoke bilateral negotiation. Standardization matters because settlement is exactly where documentation ambiguity becomes cash ambiguity.

**Check:** A good operational question is: “If a credit event is determined, do I know *exactly* which process sets $P_{\text{final}}$ and on which date the resulting cashflows settle?”

### 39.7.2 Auction Hardwiring and Determinations Committees

**Anchor:** ISDA released a supplement and a protocol for credit and succession events in 2009. This led to the creation of regional **Determinations Committees (DCs)** whose decisions are binding. If a committee decides on a specific credit event, an automatic and mandatory CDS auction settlement process takes place.

**Expand:** From a desk perspective, this is a state machine:
1. A potential event is raised.
2. The DC determines whether the event is in‑scope for the contract definitions.
3. If in‑scope, an auction process produces a cash‑settlement price that the contract uses.

**Check:** This is why “did it trigger?” and “what is $P_{\text{final}}$?” are separate questions: one is a **legal/process** determination; the other is a **price‑formation** problem.

### 39.7.3 Fixed Coupons and Points‑Upfront Quotes

**Anchor:** Under **fixed coupon + upfront** (“points upfront”) quoting, a standard running coupon $S_{\text{coupon}}$ is specified and an upfront payment at trade inception makes the contract’s PV consistent with the market spread. The standardization of North American Corporate CDS saw the introduction of fixed coupons of $100-500 \mathrm{bp}$.

A useful first‑order approximation is:

$$\boxed{\text{Upfront} \approx (S_{\text{market}}-S_{\text{coupon}})_{\text{bp}} \times RPV01}$$

where $RPV01$ is the risky PV of 1 bp of running premium for the specified contract (in currency per 1 bp, for the stated notional).

**Check (sign):** If $S_{\text{market}} > S_{\text{coupon}}$, the protection buyer pays an upfront amount (in addition to the fixed running coupon). If $S_{\text{market}} < S_{\text{coupon}}$, the upfront flows the other way.

> **Desk Reality:** Many systems treat a CDS quote like a bond quote: “running coupon” is fixed, and “price” is the upfront.
> **Common break:** Mixing up whether $RPV01$ is per 1 bp **for this notional** (currency/bp) or per 1 bp **per unit notional**.
> **What to check:** Units: $(\text{bp})\times(\text{currency/bp})=\text{currency}$. If your upfront comes out in “years,” your $RPV01$ definition is inconsistent.

---

## 39.8 CDS-Cash Basis and Settlement Risk

### 39.8.1 Defining the Basis

**Anchor:** The **CDS‑cash basis** is the difference between the CDS spread and a comparable bond spread (often an asset‑swap spread):

$$\text{CDS-Bond Basis} = S_{\text{CDS}} - S_{\text{ASW}}$$

**Expand:** In a frictionless world, buying a bond and buying CDS protection (same maturity, same seniority) would approximately replicate a risk‑free position, so spreads would line up and the basis would be close to zero. In practice, the basis can be meaningfully positive or negative because CDS and cash bonds differ in funding, liquidity, settlement mechanics, and embedded options.

**Check:** “Bond spread” is not unique. If you change the benchmark curve (Treasury vs swap) or the bond‑spread measure (Z‑spread vs ASW), you will change the reported basis.

### 39.8.2 Factors Driving the Basis

Mechanism‑level drivers include:

1. **Funding / balance sheet:** CDS is typically unfunded; bonds are funded. Different funding levels across investors can push the basis either way.
2. **Delivery option (restructuring clause):** If restructuring is included, long protection embeds a delivery option (Section 39.4), which tends to make CDS protection more valuable and can widen CDS spreads relative to cash.
3. **Credit‑event breadth (“technical default”):** CDS credit events can be broader than “bond default” in everyday language, so protection sellers may demand compensation for triggers that do not map cleanly to a bondholder loss.
4. **Loss‑on‑default mismatch when bonds trade away from par:** CDS protection is tied to a par‑based loss‑from‑par settlement convention, while a bond purchased at price $P$ has a different dollar loss profile than a par instrument.
5. **Accrued premium on default:** The buyer pays the accrued premium to the event date (Section 39.6), which affects the net economics of long protection versus holding the cash bond.
6. **Market microstructure:** Differences in liquidity, supply/demand for protection, and technical positioning can dominate in stressed markets.

**Check (toy mismatch):** If you buy a bond at 70 and the auction final price is 35, the bond’s loss is 35 points (70 → 35), while a CDS on \$100 notional pays 65 points (100 → 35). A 1:1 notional hedge can therefore **over‑hedge** a discounted bond; hedge ratios depend on your bond entry price and on the settlement price you expect.

**Check:** The basis is not an arbitrage identity; it can be positive in some periods and negative in others.

### 39.8.3 Final Price Uncertainty (Cash Settlement)

Under cash settlement, the protection payout is not known until the auction (or other contract‑specified process) determines $P_{\text{final}}$. The payout sensitivity to the final price is:

$$\frac{\partial \Pi_{\text{cash}}}{\partial P_{\text{final}}} = -\frac{N}{100}$$

Define the settlement‑price risk scalar for a **long‑protection** position:
- **Bump object:** $P_{\text{final}}$ (points per 100)
- **Bump size:** $-1$ price point
- **Units:** currency per 1 price point
- **Sign convention:** positive for long protection

With this convention,

$$\text{Settlement01} := \Pi(P_{\text{final}}-1) - \Pi(P_{\text{final}}) = \frac{N}{100}.$$

**Check (units):** 1 price point is 1% of par. For $N=\$10\text{mm}$, Settlement01 $=\$100{,}000$ per 1‑point move down in $P_{\text{final}}$.

### 39.8.4 Deliverable Scarcity (Physical Settlement)

As discussed in Section 39.2.3, when CDS notional exceeds deliverable supply, protection buyers face sourcing risk. The short squeeze dynamic can push up deliverable prices, reducing effective protection.

> **Desk Reality:** Traders monitor the basis because it bundles funding, liquidity, and contract‑feature differences into one number.
> **Common break:** Comparing a CDS spread to a bond spread computed with a different benchmark curve or a different restructuring clause.
> **What to check:** Align definitions (spread measure, curve, restructuring clause, and accrual treatment) before interpreting the sign of the basis.

---

## 39.9 Worked Examples

**Example Title**: Net cash settlement (payout minus accrued premium)

**Context**
- You are long protection on a single name and a credit event occurs.
- You need the **net cash** that will settle (protection payout minus accrued premium), plus the key risk scalar to the settlement price.

**Timeline (Make Dates Concrete)**
- Trade date: March 20, 2020
- Scheduled premium dates: Mar 20 / Jun 20 / Sep 20 / Dec 20 (quarterly, in arrears)
- Last premium date before the event: March 20, 2023
- Credit event / notice date: May 20, 2023
- Cash settlement date: assume May 25, 2023 for the cashflow table (the protocol sets the actual date)

**Inputs**
- Notional: $N=\$100\text{mm}$
- Contractual spread: $s=90$ bp per year $=0.009$
- Settlement method: cash settlement
- Auction final price: $P_{\text{final}}=35$ (points per 100 face)
- Day count (for this example): ACT/360, so $\alpha=\text{DayDiff}/360$

**Outputs (What You Produce)**
- Protection payout: $\Pi_{\text{cash}}$
- Accrued premium to the event date: $N\cdot s\cdot \alpha$
- Net settlement cash to the protection buyer (if netted)
- Risk metric: Settlement01 (state bump object + units)

**Step-by-step**
1. Translate auction price to protection payout:
   $$\Pi_{\text{cash}} = N\left(1-\frac{P_{\text{final}}}{100}\right) = 100{,}000{,}000 \times (1-0.35)=\$65{,}000{,}000.$$
2. Build the accrual factor from concrete dates:
   - March 20, 2023 → May 20, 2023 is 61 days
   - $\alpha = 61/360$
3. Compute accrued premium:
   $$\text{Accrued} = N\cdot s\cdot \alpha = 100{,}000{,}000 \times 0.009 \times \frac{61}{360}=\$152{,}500.$$
4. Net the two legs (net cash from seller to buyer):
   $$\text{Net} = 65{,}000{,}000 - 152{,}500 = \$64{,}847{,}500.$$
5. Settlement‑price risk scalar (for long protection):
   - **Bump object:** $P_{\text{final}}$ (points per 100)
   - **Bump size:** $-1$ point
   - **Units:** currency per 1 point
   - **Sign:** positive for long protection
   $$\text{Settlement01}=\Pi(P_{\text{final}}-1)-\Pi(P_{\text{final}})=\frac{N}{100}=\$1{,}000{,}000.$$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2023-05-25 | +65,000,000 | Protection payout (cash settlement) |
| 2023-05-25 | -152,500 | Accrued premium owed to event date |
| 2023-05-25 | +64,847,500 | Net cash (if operationally netted) |

**P&L / Risk Interpretation**
- The payout is linear in $P_{\text{final}}$: 1 price point is 1% of notional.
- For this trade, a 5‑point change in $P_{\text{final}}$ changes payout by \$5mm.
- Operationally, the key “gotchas” are: misreading price points as fractions, using the wrong day count for $\alpha$, and forgetting to include the final accrual payment.

**Sanity Checks**
- Units check: $P_{\text{final}}$ is points per 100, so $1-P_{\text{final}}/100$ is a fraction.
- Sign check: lower $P_{\text{final}}$ increases $\Pi_{\text{cash}}$ for long protection.
- Limit check: $P_{\text{final}}=100 \Rightarrow \Pi=0$; $P_{\text{final}}=0 \Rightarrow \Pi=N$.

**Debug Checklist (When Your Result Looks Wrong)**
- Confirm $N$ units (10mm vs 100mm) and whether the system uses per‑unit or per‑100 quoting.
- Confirm whether your “credit event date” equals the accrual end date used by the system.
- Confirm day count for premium accrual and whether the platform nets the premium accrual with the protection payment.

**References:** See `## References`.

---

## Practical Notes

### Mechanics Checklist for a Real Trade

Before modeling settlement outcomes, verify:

1. **Reference entity** and reference obligation(s)
2. **Documentation set** — ISDA Definitions vintage (2003? 2014?)
3. **Credit events included** — which events trigger; is restructuring included?
4. **Restructuring clause** — Old-Re / Mod-Re / Mod-Mod-Re / No-Re
5. **Settlement method** — physical, cash, or fallback language
6. **If physical:** deliverable obligation criteria and any maturity restrictions
7. **If cash:** how $P_{\text{final}}$ is determined (auction protocol)
8. **Notional, currency, settlement conventions**
9. **Accrued premium handling**
10. **Fixed coupon convention** (post-Big Bang: 100bp or 500bp)

### Common Pitfalls

1. **Confusing "recovery" concepts:** The settlement final price is a market-determined number around the event, not a long-run modeling parameter.

2. **Assuming CTD is irrelevant under cash settlement:** Auction-style cash settlement is designed to reference the mid‑market value of the CTD deliverable, so CTD economics still matter even if nothing is physically delivered.

3. **Forgetting accrued premium at default:** Premium is paid in arrears; a partial period accrual is owed even if the buyer never receives a full premium payment in that period.

4. **Mixing price conventions:** 38 (per 100) vs 0.38 (fraction) → factor of 100 payout error.

5. **Ignoring documentation differences across products:** A CDS quote with No‑Restructuring is not directly comparable to one that includes restructuring unless you explicitly adjust for the embedded delivery option.

6. **Overlooking succession events:** M&A activity can change the reference entity; monitor DC announcements.

### Verification Tests for Risk Systems

1. **Payout bounds:** $0 \leq \Pi_{\text{cash}} \leq N$ when $0 \leq P_{\text{final}} \leq 100$
2. **Physical/cash alignment:** If $P_{\text{final}} = P_{\text{CTD}}$, then $\Pi_{\text{cash}} = \Pi_{\text{phys}}$
3. **CTD logic:** Under physical, buyer delivers lowest-priced eligible deliverable
4. **Delivery option non-negativity:** $V_{\text{DO}} \geq 0$

---

## Summary

1. A CDS triggers on a contract-defined **credit event**; “default” in headlines is not sufficient without the documentation mapping.
2. **Settlement** maps a trigger into cash: physical settlement delivers eligible obligations for par; cash settlement pays $N(1-P_{\text{final}}/100)$.
3. A **deliverable basket** improves liquidity but embeds a **delivery option**: the buyer chooses the **CTD** deliverable; the seller is short that option.
4. **Restructuring** is the soft event that creates post‑event price dispersion; restructuring clauses (Old‑Re / Mod‑Re / Mod‑Mod‑Re / No‑Re) control CTD optionality and can affect spreads.
5. Premium is paid in arrears; on default the buyer owes **accrued premium to the event date**, so net settlement is protection payout minus that accrual.
6. Post‑2009 standardization hardwired auction/DC workflows and popularized **fixed coupon + upfront** (“points‑upfront”) quoting; a useful approximation is $(S_{\text{market}}-S_{\text{coupon}})_{\text{bp}}\times RPV01$.
7. Settlement price risk is linear: for long protection, a 1‑point decrease in $P_{\text{final}}$ increases payout by $N/100$.
8. The CDS‑cash basis reflects contractual differences (funding, delivery option, credit‑event breadth, accrual) plus market microstructure—so compare like‑for‑like before interpreting the sign.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Credit event | Contractual trigger for protection leg (may differ from bond default) | Determines whether you receive protection |
| Hard vs soft event | Hard: all debt trades at one price; Soft: term structure persists | Affects delivery option value |
| Physical settlement | Deliver bonds, receive par | Requires sourcing deliverables |
| Cash settlement | Receive $N(1 - P_{\text{final}}/100)$ | Requires price determination mechanism |
| Delivery option | Buyer's right to choose deliverable | Creates embedded optionality for buyer |
| CTD | Lowest-priced eligible deliverable | Maximizes buyer's payout |
| Restructuring clause | Limits on deliverables after restructuring | Controls delivery option value |
| Accrued premium | Partial premium for coverage until credit event | Owed by buyer at default |
| Auction/DC workflow | Standard process to confirm events and produce $P_{\text{final}}$ | Turns legal trigger into cash settlement |
| Fixed coupon + upfront | Quote as running coupon plus upfront | Makes CDS trade “like a bond price” |
| Settlement01 | $\Pi(P_{\text{final}}-1)-\Pi(P_{\text{final}})=N/100$ for long protection | Quick risk check for payout uncertainty |
| CDS-cash basis | $S_{\text{CDS}}-S_{\text{ASW}}$ (definition depends on spread measure) | Bundles funding, optionality, and technicals |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $N$ | CDS notional | currency |
| $P_{\text{final}}$ | Cash settlement final price | points per 100 face |
| $R_{\text{settle}}:=P_{\text{final}}/100$ | Settlement‑implied recovery fraction | unitless |
| $\mathcal{D}$ | Eligible deliverables set | n/a |
| $P_i$ | Price of deliverable $i$ | points per 100 face |
| $P_{\text{CTD}}:=\min_{i\in\mathcal{D}}P_i$ | Cheapest‑to‑deliver price | points per 100 face |
| $P_{\text{hedged}}$ | Price of the hedged asset | points per 100 face |
| $\Pi_{\text{cash}},\Pi_{\text{phys}}$ | Protection payout (cash / physical) | currency; positive = receive (buyer) |
| $V_{\text{DO}}$ | Delivery option value | currency |
| $s$ | Contractual CDS spread | bp/year or decimal/year (state which) |
| $\alpha$ | Premium accrual fraction to event date | year fraction under premium day count |
| $RPV01$ | Risky PV of 1 bp of running premium | currency per 1 bp for the stated notional |
| $S_{\text{market}},S_{\text{coupon}}$ | Market spread and fixed coupon | bp/year |
| Settlement01 | $\Pi(P_{\text{final}}-1)-\Pi(P_{\text{final}})$ | currency per 1 price point; long protection positive |
| $S_{\text{CDS}},S_{\text{ASW}}$ | CDS spread and bond spread measure | bp/year (be explicit) |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CDS credit event? | The contract-defined legal trigger for the protection payment. |
| 2 | Why is “credit event” not the same as a rating-agency default? | CDS triggers are documentation-driven and can differ from ratings definitions and headlines. |
| 3 | Name the six commonly used credit events. | Bankruptcy, failure to pay, obligation acceleration, obligation default, repudiation/moratorium, restructuring. |
| 4 | What is physical settlement? | Buyer delivers eligible obligations with face $N$ and receives $N$ cash (par). |
| 5 | Physical settlement payout if you deliver an obligation priced $P_i$? | $\Pi_{\text{phys}}(i)=N(1-P_i/100)$. |
| 6 | What is cash settlement? | Seller pays $\Pi_{\text{cash}}=N(1-P_{\text{final}}/100)$ using a contract-specified final price. |
| 7 | What is $P_{\text{final}}$? | Cash-settlement final price in points per 100 (often from an auction process). |
| 8 | What is a deliverable obligation? | A bond/loan eligible for delivery under the contract criteria; often pari passu or senior to the reference obligation. |
| 9 | What is CTD? | Cheapest-to-deliver: $P_{\text{CTD}}=\min_{i\in\mathcal{D}}P_i$. |
| 10 | What is the delivery option? | The buyer’s right to choose which deliverable to deliver; seller is short this option. |
| 11 | Simple delivery option value formula? | $V_{\text{DO}}=N(P_{\text{hedged}}-P_{\text{CTD}})/100$. |
| 12 | Why does restructuring matter for CTD? | Restructuring can leave a term structure of prices across obligations, creating price dispersion and a valuable CTD choice. |
| 13 | What do restructuring clauses do? | They limit (or remove) restructuring-driven deliverability optionality (Old‑Re / Mod‑Re / Mod‑Mod‑Re / No‑Re). |
| 14 | What is accrued premium on default? | $N\cdot s\cdot \alpha$, owed by the buyer for coverage from the last premium date to the event date. |
| 15 | Net cash (conceptually) for cash settlement? | Protection payout minus accrued premium (if netted operationally). |
| 16 | Unit pitfall: “38” means what? | 38 points per 100 (i.e., $R_{\text{settle}}=0.38$), not 38%. |
| 17 | What is Settlement01 for long protection? | $\Pi(P_{\text{final}}-1)-\Pi(P_{\text{final}})=N/100$ (currency per 1 point). |
| 18 | What are the three “Big Bang” elements (high level)? | Auction hardwiring, standardized trading conventions, and central clearing enablement. |
| 19 | What is fixed coupon + upfront quoting? | Running coupon is standardized (e.g., 100/500 bp) and an upfront payment clears the trade to the market spread. |
| 20 | Useful upfront approximation? | $\text{Upfront}\approx (S_{\text{market}}-S_{\text{coupon}})_{\text{bp}}\times RPV01$. |
| 21 | What is $RPV01$ in this chapter’s units? | PV of 1 bp of running premium for the stated notional, in currency per 1 bp. |
| 22 | What is the CDS‑cash basis? | $S_{\text{CDS}}-S_{\text{ASW}}$; interpret only after aligning spread measures and contract features. |

---

## Mini Problem Set

1. A CDS has $N = \$25\text{mm}$ and auction final price 12. Compute the cash settlement payout.

2. For $N = \$10\text{mm}$, compare payouts if final price is 38 vs 42. What is the difference?

3. Two deliverables are priced at 55 and 60. Under physical settlement with $N = \$10\text{mm}$, what is the CTD and payout?

4. Three deliverables priced 20, 35, 50 with $N = \$10\text{mm}$. Compute physical settlement payout. Compare to cash at $P_{\text{final}} = 35$.

5. In Q4, the 20-priced bond is ruled ineligible. Recompute the payout and the change.

6. Explain why a soft credit event creates larger delivery option value than a hard event.

7. A buyer holds a bond at 48 with CDS, and CTD after restructuring is 40. Per \$100 face, what is the delivery option value? For $N = \$10\text{mm}$?

8. Spread $s = 400$ bp, default 60 days into a 90-day period, $N = \$20\text{mm}$. Estimate accrued premium (state assumptions).

9. Derive: if $P_{\text{final}} = P_{\text{CTD}}$, then $\Pi_{\text{cash}} = \Pi_{\text{phys}}$.

10. CTD bond has bid/ask of 25/35 in distressed market. For $N = \$50\text{mm}$, compute the payout range.

11. A CDS trades with a 100 bp fixed coupon. Market spread is 350 bp. For $N = \$10\text{mm}$ and $RPV01 = \$4{,}200$ per bp (for this notional), estimate the upfront payment at trade inception. Who pays whom?

12. Following a credit event, the DC determines auction final price = 22. Protection buyer owes 50 days of accrued premium on a 300 bp contract with $N = \$5\text{mm}$. What is the net payment from seller to buyer?

---

### Solution Sketches (Selected)

**Q1:** $\Pi = 25\text{mm} \times (1 - 0.12) = 25\text{mm} \times 0.88 = \$22\text{mm}$

**Q2:** At 38: $\Pi = 10\text{mm} \times 0.62 = \$6.2\text{mm}$. At 42: $\Pi = 10\text{mm} \times 0.58 = \$5.8\text{mm}$. Difference: $\$0.4\text{mm}$.

**Q3:** CTD = 55. $\Pi = 10\text{mm} \times (1 - 0.55) = 10\text{mm} \times 0.45 = \$4.5\text{mm}$

**Q4:** Physical uses CTD = 20: $\Pi = 10\text{mm} \times 0.80 = \$8.0\text{mm}$. Cash at 35: $\Pi = 10\text{mm} \times 0.65 = \$6.5\text{mm}$. Physical payout is $\$1.5\text{mm}$ higher due to CTD.

**Q5:** New CTD = 35: $\Pi = 10\text{mm} \times 0.65 = \$6.5\text{mm}$. Change: $-\$1.5\text{mm}$ from Q4.

**Q6:** Soft events preserve term structure → different obligations trade at different prices → buyer can choose cheapest. Hard events collapse all debt to one price → no choice benefit.

**Q7:** Delivery option value = \$48 - 40 = \$8$ per $100. For $\$10\text{mm}$: $10{,}000{,}000 \times 0.08 = \$800{,}000$.

**Q9:** $\Pi_{\text{cash}} = N(1 - P_{\text{final}}/100)$. $\Pi_{\text{phys}} = N(1 - P_{\text{CTD}}/100)$. If $P_{\text{final}} = P_{\text{CTD}}$, both equal $N(1 - P_{\text{CTD}}/100)$. QED.

**Q11:** Spread difference = $350-100=250$ bp. Upfront $\approx 250 \times \$4{,}200 = \$1{,}050{,}000$. Protection buyer pays upfront (market spread > coupon).

**Q12:** Protection payout: $5{,}000{,}000 \times (1 - 0.22) = \$3{,}900{,}000$. Accrued premium: $5{,}000{,}000 \times 0.03 \times (50/360) = \$20{,}833$. Net: $\$3{,}900{,}000 - \$20{,}833 = \$3{,}879{,}167$ from seller to buyer.

---

## References

- (O’Kane, *Modelling Single-name and Multi-name Credit Derivatives*, Chapter 5 “Mechanics of the Protection Leg”; “The Delivery Option”; “The Restructuring Clause”; “The CDS‑Cash Basis”)
- (Hull, *Options, Futures, and Other Derivatives*, Chapter 25 “Credit Default Swaps” and “Credit Default Swaps and Bond Yields”; Business Snapshot on Lehman settlement)
- (Neftci, *Principles of Financial Engineering*, Section 18.4.1 “CDS Standardization and 2009 ‘Big Bang’”; Section 18.4.2 “Restructuring”)
- (Hull, *Risk Management and Financial Institutions*, section “The Use of Fixed Coupons”)
- (Damiano et al., *Counterparty Credit Risk, Collateral and Funding*, section on default/credit-event cases)
- (ISDA, *2014 ISDA Credit Derivatives Definitions*, 2014)
