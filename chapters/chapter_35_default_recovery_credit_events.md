# Chapter 35: Default, Recovery, and Credit Events — Economic vs Contractual Reality

---

## Conventions & Notation

| Convention | Detail |
|------------|--------|
| **Currency & units** | Notional amounts are in dollars; bond prices are quoted per 100 par unless stated otherwise |
| **Recovery rate** | $R \in [0,1]$ denotes recovery as a fraction of par (e.g., 40% recovery $\Rightarrow R = 0.40$). When using a price per 100 $P$, we map $R := P/100$ |
| **Loss-given-default** | $\text{LGD} := 1 - R$ |
| **CDS premium accrual** | Premium accrues on an Actual/360 day-count when we do explicit day-count calculations (consistent with the cited mechanics) |
| **Sign convention** | Payoffs are written from the perspective of the protection buyer unless stated otherwise |

---

## Setup

### Conventions used in this chapter

See notation table above.

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional (dollars) |
| $R$ | Recovery rate as a fraction of par (dimensionless, 0–1) |
| $\text{LGD}$ | Loss given default $= 1 - R$ (dimensionless) |
| $P$ | Observed distressed/defaulted bond price per 100 par (price points) |
| $S$ | CDS spread as annual rate in decimal (e.g., 200 bp = 0.02) |
| $\alpha$ | Day-count fraction from last premium date to default date (Actual/360 when used explicitly) |
| "Credit event" | Contractual trigger for CDS protection leg |

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- A CDS is a bilateral OTC contract that protects the protection buyer from loss from par on a specified face value of bonds/loans (the deliverable obligations) if a credit event occurs before maturity.
- CDS cashflows are organized into two legs: (i) premium leg (periodic premium, typically quarterly) and (ii) protection leg (contingent loss payment upon credit event).
- The standard mechanics include paying premium accrued at default: after a credit event, the protection buyer pays the fraction of premium accrued since the previous premium date.
- The protection leg can be settled via physical settlement (deliver obligations for par) or cash settlement (pay par minus a recovery/settlement price determined by a dealer poll or auction).

### (B) Reasoned Inference (Derived from A)

- The core message of this chapter: what you see economically (distress, inability/willingness to pay, market price collapse) can diverge from what you get contractually (a CDS payout only when a contractual credit event is triggered and processed). This creates timing risk, definition risk, and settlement-mechanics risk.

### (C) Speculation (Clearly Labeled)

- None needed in Setup.

---

## Core Concepts

### 1) Economic Default vs Contractual Credit Event

**Formal Definition (contractual):**

A credit event is the legal term for the event that triggers payment of the CDS protection leg. It is similar to default but not exactly the same as "event of default" as defined by rating agencies; practitioners often say "default" when they mean "credit event."

**Credit event taxonomy (as described in the sources):**

The book lists commonly used credit events and classifies them as hard vs soft:

| Hard Credit Events | Soft Credit Event |
|--------------------|-------------------|
| Bankruptcy | Restructuring |
| Failure to pay | |
| Obligation acceleration | |
| Obligation default | |
| Repudiation/moratorium | |

**Soft vs hard:** Restructuring is described as the only soft credit event; after restructuring, debt can continue trading with a term structure of prices (shorter maturities typically higher prices than longer maturities).

**Triggering mechanics (physical-settlement timeline preview):**

To trigger protection, the protection buyer delivers a Credit Event Notice supported by at least two sources of publicly available information (examples given include Reuters, Wall Street Journal, Financial Times, or similar); the delivery date is the Event Determination Date.

**Intuition:**

Economic distress is continuous (spreads widen, equity falls, funding dries up). The CDS contract pays only when the world crosses a discrete contractual boundary ("credit event") and the operational steps are completed.

Contractual definitions can be broader than bond default: the CDS basis discussion explicitly notes "technical default," i.e., standard credit events may be broader than what constitutes default on a bond.

**Trading / Risk / Portfolio Practice:**

Risk managers distinguish:
- **Mark-to-market credit deterioration** (spread widening, price drop) vs
- **Jump-to-default / credit-event payoff** (discrete payout if contract triggers)

Hedging a bond with CDS can fail if the bond tanks but the CDS does not trigger yet (or triggers under terms different from your intuition).

---

### 2) Recovery and LGD

**Formal Definition:**

$R$ is the recovery rate (fraction of par); $\text{LGD} = 1 - R$. The CDS protection payment is commonly expressed as a fraction $(1 - R)$ of the face value.

**Market measurement of recovery (bond-market notion):**

Recovery rate in credit markets is measured as the defaulted bond price divided by face value. Hull similarly describes the bond recovery rate as the value of the bond shortly after default as a percentage of face value.

**Recovery in credit derivatives vs workout recovery:**

The "recovery price" used in credit derivatives is set within a short window (the source notes "within 72 days") and can differ from the amount ultimately received after a full workout/liquidation process; differences can arise from information arrival and supply/demand in distressed markets.

**Intuition:**

Recovery is not a single "true" number; it depends on what claim you hold, when you measure, and what the settlement mechanism uses as the reference price.

**Trading / Risk / Portfolio Practice:**

Traders talk about "recovery" as a market-implied object: the defaulted bond price (or the CDS cash-settlement "final price") is an operational proxy for what the CDS will reference.

---

### 3) Settlement Mechanics (Preview): Physical vs Cash

**Formal Definition (physical settlement):**

The protection buyer delivers face value of deliverable obligations to the protection seller and receives face value in cash (par).

**Formal Definition (cash settlement):**

The protection seller pays "face value minus recovery price" in cash, where the recovery price is determined by a dealer poll or auction process (and in Hull, via a calculation agent or auction determining cheapest-to-deliver value after default).

**Deliverables and the delivery option:**

Because multiple bonds/loans may qualify as deliverable obligations, the protection buyer can choose what to deliver—effectively being "long a delivery option."

**Operational/market impact (short squeeze):**

If CDS notional specifying physical settlement is large relative to deliverable obligations outstanding, protection buyers may scramble to source deliverables, potentially raising defaulted-asset prices ("short squeeze") and reducing the loss payment; a fallback to cash settlement may exist if physical settlement is impossible/illegal.

**Intuition:**

Physical settlement pays par for delivered paper; cash settlement pays par minus an agreed price proxy.

Delivery option matters most in restructuring (soft credit event) because different deliverables can trade at different prices post-event.

**Trading / Risk / Portfolio Practice:**

Basis traders care about settlement details because they change payoffs relative to bonds (e.g., cheapest-to-deliver option, accrued differences, technical default, restructuring).

---

### 4) Premium Accrual at Default

**Formal Definition:**

Premium leg payments terminate following a credit event, but the protection buyer pays premium accrued since the previous premium date (a contingent cashflow).

**Premium schedule convention (mechanics):**

Premium payments are typically quarterly, and payment amounts use an Actual/360 day count in the worked mechanics shown in the source.

**Intuition:**

Premium is paid "in arrears," so default mid-period still leaves a "stub" premium owed.

**Trading / Risk / Portfolio Practice:**

Accrued-on-default matters for hedging and basis: the sources note CDS pays accrued premium on default, whereas bondholders' accrued coupon can be lost in default (a basis driver).

---

## Math and Derivations

### (A) Verified Facts (Source-Backed)

- CDS protection leg aims to "make up to par" the value of a deliverable obligation after a credit event.
- A commonly stated payoff form is $L(1 - R)$ where $L$ is notional and $R$ is recovery rate.

### (B) Reasoned Inference (Derived, with Explicit Assumptions)

#### 1) Core Payoff Identity (Cash Settlement)

**Assume:**
- Notional $N$ is measured in dollars
- Settlement price is interpreted as recovery fraction $R$ of par

**Then** the cash settlement payoff to the protection buyer is:

$$\boxed{\Pi_{\text{prot}} = N(1 - R) = N \cdot \text{LGD}}$$

**Unit check:**
- $N$ is dollars, $R$ is dimensionless, so $\Pi_{\text{prot}}$ is dollars ✓

**Bounds:**
- If $R = 1$ then $\Pi_{\text{prot}} = 0$
- If $R = 0$ then $\Pi_{\text{prot}} = N$ ✓

---

#### 2) Premium Accrual at Default (Simple Accrual Arithmetic)

**Assume:**
- Contract spread $S$ is an annual rate in decimal (e.g., 200 bp = 0.02)
- Accrual fraction $\alpha$ is Actual/360 for the period from last premium date to default date, consistent with the mechanics section
- Accrued premium is owed by the protection buyer at default (standard contract feature described)

**Then:**

$$\boxed{\Pi_{\text{accrued prem}} = N \cdot S \cdot \alpha}$$

**Unit check:**
- $S$ is "per year," $\alpha$ is "years," product is dimensionless; times $N$ gives dollars ✓

---

#### 3) Physical vs Cash Settlement Equivalence (Economic Alignment)

**From the contract mechanics:**
- Physical settlement: buyer delivers face value of deliverables; receives par
- Cash settlement: buyer receives par minus recovery price in cash

Let the market price of a deliverable immediately after the event be $P$ per 100 par, and set $R := P/100$.

If a protection buyer can acquire $N$ face amount of deliverables at price $P$, the economic gain from delivering into the CDS is:

$$\text{Gain} = N - N \cdot \frac{P}{100} = N(1 - R)$$

matching the cash-settlement identity.

**Assumptions (must be stated):**
- We ignore bid/ask, financing, and the delivery option's impact on which bond is cheapest-to-deliver
- We assume the "recovery price" used for cash settlement equals the market price of the deliverable used in the physical example

### (C) Speculation (Clearly Labeled)

None needed for the algebra; the only uncertainty is operational detail (e.g., exact auction steps) beyond what the sources specify.

---

## Measurement & Risk (Only What Belongs in Chapter 35)

### 1) Economic Default vs Contractual Credit Event (Must-Distinguish)

#### (A) Verified Facts

- Credit event is a contractual/legal trigger, similar to but not the same as rating-agency "event of default"; market participants often conflate terminology.
- The contract specifies a taxonomy of credit events (hard/soft) including restructuring as the only soft credit event.
- Protection is triggered via a Credit Event Notice supported by publicly available information; the notice date is the Event Determination Date.

#### (B) Reasoned Inference

- Economic default (in a trading sense) often manifests as spread blowout and price collapse.
- Contractual reality: your CDS may not pay until (i) a defined credit event occurs, (ii) notice/evidence conditions are satisfied, and (iii) settlement steps complete—creating timing mismatch risk for hedges.

#### (C) Speculation

None.

---

### 2) Recovery Concepts: $R$, LGD, and "Recovery" Ambiguity

#### (A) Verified Facts

- Market recovery rate is measured as defaulted bond price divided by face value; recovery in credit derivatives is tied to a recovery price determined within a short window and can differ from ultimate workout recovery.
- CDS loss payment is commonly expressed as $(1 - R)$ of face value.

#### (B) Reasoned Inference

**Recovery as market-implied:**

- If a bond trades at $P = 35$ shortly after default, a market-implied recovery proxy is $R \approx 0.35$. This is consistent with the "defaulted bond price / face" measure.
- If a CDS auction/cash-settlement process produces a final price $P_{\text{settle}}$, you can interpret $R := P_{\text{settle}}/100$ as the recovery used for settlement (subject to contract specifics).

**"Recovery" can mean different things:**

Sources explicitly contrast credit-derivatives recovery price vs workout recovery.

If you need exact distinctions such as "recovery of par" vs "recovery of market value" conventions beyond these statements: **I'm not sure.** You'd need the specific contract's definitions and the modeling convention you want to adopt.

#### (C) Speculation

None.

---

### 3) CDS Settlement Mechanics (Preview; No Full Pricing)

#### (A) Verified Facts

- Physical settlement: deliver obligations for par.
- Cash settlement: pay par minus a recovery/settlement price, determined by dealer poll or auction; Hull describes a calculation agent or auction determining cheapest-to-deliver value after default.
- There can be practical frictions in physical settlement (short squeeze) and a mechanism may allow fallback to cash settlement if physical settlement is impossible/illegal.
- Premium accrued at default is paid by the protection buyer.

#### (B) Reasoned Inference

Settlement mechanics determine the exact cashflows, so they directly affect:
- Hedge effectiveness vs cash bonds
- Basis between CDS and bonds (delivery option, technical default, accrued differences)

#### (C) Speculation

**Auction process operational steps:** the sources here only say "dealer poll or auction process" and do not fully specify all operational steps.

If you need the exact mechanics: **I'm not sure.** We would need the specific ISDA Definitions vintage and the auction settlement protocol referenced by the contract.

---

### 4) Credit Event Taxonomy (Preview) + Contract Dependence

#### (A) Verified Facts

Credit event categories and definitions (as provided) include:

| Credit Event | Description |
|--------------|-------------|
| **Bankruptcy** | Hard event |
| **Failure to Pay** | Hard event |
| **Obligation Acceleration** | Hard event |
| **Obligation Default** | Hard event |
| **Repudiation/Moratorium** | Hard event |
| **Restructuring** | Soft event — can leave a term structure of post-event prices |

**Restructuring clause choices** differ across regions and change deliverable constraints; common labels include:

| Clause | Description |
|--------|-------------|
| Old-Re (OR) | Original restructuring |
| Mod-Re (MR) | Modified restructuring (US standard) |
| Mod-Mod-Re (MMR) | Modified-modified restructuring |
| No-Re (NR) | No restructuring — removes restructuring as credit event |

#### (B) Reasoned Inference

Credit events and settlement depend on the contract's specified definitions and restructuring clause. Even when two traders say "default," they might not mean the same contractual trigger—hence event-definition risk.

The sources explicitly caution that ISDA definitions evolve; the discussion is intended to sensitize modelers to "precise rules" and to seek up-to-date legal advice.

#### (C) Speculation

Who determines credit events in practice (e.g., committees): not specified in the cited excerpts. **I'm not sure.** We would need the relevant ISDA Definitions and trade confirmation language.

---

### 5) Practical Risk Implications (Preview-Level)

#### (A) Verified Facts

- CDS payoff form $N(1 - R)$ creates a discrete jump exposure at credit event.
- Physical settlement can involve delays and market frictions (timeline and potential short squeeze).

#### (B) Reasoned Inference

- **Jump-to-default exposure:** for a protection seller, the instantaneous cash outflow upon credit event is approximately $N(1 - R)$ (ignoring accrued premium and close-out nuances).
- **Recovery risk:** uncertainty in $R$ directly scales payout: $\partial\Pi/\partial R = -N$.
- **Event risk:** ambiguity/timing around whether/when a credit event is triggered, plus settlement timing and deliverables.

#### (C) Speculation

None.

---

## Worked Examples

*All examples are toy, focus on cashflow plumbing, and use price-per-100 and $ notionals consistently.*

---

### Example A: LGD Arithmetic

**Given:** $R \in \{20\%, 40\%, 60\%\}$ and notional $N = \$10{,}000{,}000$. Compute LGD and loss.

| Recovery $R$ | LGD $= 1 - R$ | Loss $= N \cdot \text{LGD}$ |
|--------------|---------------|----------------------------|
| 0.20 | 0.80 | $\$8{,}000{,}000$ |
| 0.40 | 0.60 | $\$6{,}000{,}000$ |
| 0.60 | 0.40 | $\$4{,}000{,}000$ |

**Unit check:** $N$ dollars × LGD (dimensionless) = dollars ✓

---

### Example B: Bond Price → Implied Recovery Intuition

A bond is observed at $P = 35$ (per 100 par) shortly after default/credit event. Market-implied recovery proxy:

$$R \approx \frac{P}{100} = \frac{35}{100} = 0.35$$

So implied LGD $\approx 0.65$.

**Limitations (source-backed framing):**
- The credit-derivatives "recovery price" can differ from ultimate workout recovery and is vulnerable to new information and supply/demand in distressed markets.
- Bond-price-as-recovery is a timing-dependent proxy (the sources define it as a "shortly after default" price / face).

---

### Example C: CDS Protection Leg Payoff with Recovery

CDS notional $N = \$10{,}000{,}000$. Payoff to protection buyer:

$$\Pi_{\text{prot}} = N(1 - R)$$

| Recovery $R$ | Payoff $\Pi$ |
|--------------|--------------|
| 0.20 | $10{,}000{,}000 \times 0.80 = \$8{,}000{,}000$ |
| 0.40 | $\$6{,}000{,}000$ |
| 0.60 | $\$4{,}000{,}000$ |

This matches the payoff form $L(1 - R)$.

---

### Example D: Accrued Premium on Default

**Assume:**
- Notional $N = \$10{,}000{,}000$
- CDS spread $S = 300$ bp $= 0.03$ per year
- Premium dates quarterly; last premium date to default is 50 days
- Use Actual/360: $\alpha = 50/360 = 0.1388889$

**Accrued premium** owed by protection buyer at default:

$$\Pi_{\text{accrued prem}} = N \cdot S \cdot \alpha = 10{,}000{,}000 \times 0.03 \times 0.1388889 = \$41{,}666.67$$

So $41,666.67 is paid as accrued premium (in addition to any scheduled payments already made).

**Source support:** premium payments are typically quarterly, use Actual/360 in the mechanics, and accrued premium at default is a standard feature.

---

### Example E: Physical vs Cash Settlement Economics

**Assume:**
- Notional $N = \$10{,}000{,}000$
- Deliverable bond price post-event $P = 35$ per 100 par, so $R = 0.35$

**Cash settlement:**

$$\Pi = N(1 - R) = 10{,}000{,}000 \times 0.65 = \$6{,}500{,}000$$

**Physical settlement (economic replication):**

1. Buy $10mm face of deliverables at 35% of par → cost $= 0.35 \times 10{,}000{,}000 = \$3{,}500{,}000$
2. Deliver deliverables into CDS; receive par $10,000,000 (deliver for par)
3. Net gain $= 10{,}000{,}000 - 3{,}500{,}000 = \$6{,}500{,}000$, matching cash settlement

**Assumptions / "I'm not sure" flags:**

We ignore bid/ask, funding, and any constraints on what counts as deliverable. The exact deliverable set depends on contract terms (and for restructuring, on the restructuring clause).

---

### Example F: Auction "Final Price" → Recovery

Hull's Lehman example reports recovery "about eight cents on the dollar," determined by an auction process, implying about a 92% payout.

**Let:**
- $P_{\text{final}} = 8$ per 100 → $R = 0.08$
- Notional $N = \$10{,}000{,}000$

**Then CDS payoff:**

$$\Pi = N(1 - R) = 10{,}000{,}000 \times 0.92 = \$9{,}200{,}000$$

**Unit check:** dollars × dimensionless = dollars ✓

---

### Example G: Recovery Uncertainty as Risk

**Assume** recovery $R$ is random:
- Scenario 1: $R = 0.20$ with probability 60%
- Scenario 2: $R = 0.50$ with probability 40%
- Notional $N = \$10{,}000{,}000$

Loss to protection seller (or payoff to buyer) is $N(1 - R)$.

| Scenario | $R$ | Payoff |
|----------|-----|--------|
| 1 | 0.20 | $10{,}000{,}000 \times 0.80 = \$8{,}000{,}000$ |
| 2 | 0.50 | $10{,}000{,}000 \times 0.50 = \$5{,}000{,}000$ |

**Expected payoff:**

$$E[\Pi] = 0.6 \times 8{,}000{,}000 + 0.4 \times 5{,}000{,}000 = 4{,}800{,}000 + 2{,}000{,}000 = \$6{,}800{,}000$$

**Distribution matters:** same mean could hide very different tail outcomes (risk management concern).

---

### Example H: Event vs Economic Default Mismatch

**Goal:** illustrate timing mismatch between market distress and contractual credit event.

**Source-backed anchor:** the protection buyer may sometimes delay triggering, e.g., if they expect a restructuring to turn into a full default and produce a larger payoff.

**Toy timeline** (numbers are illustrative):

| Day | Event |
|-----|-------|
| 0 | Firm distress; bond drops from 95 to 60. (Economic deterioration; no contract trigger yet.) |
| 10 | A restructuring credit event occurs (soft event) but buyer delays notice hoping for harder default. |
| 40 | Buyer serves Credit Event Notice (Event Determination Date at Day 40), settlement proceeds. |

**Suppose:**
- Notional $N = \$10{,}000{,}000$
- If restructure is triggered now, settlement price $P = 40 \Rightarrow R = 0.40 \Rightarrow \Pi = \$6{,}000{,}000$
- If the situation worsens to a hard default, settlement price $P = 25 \Rightarrow R = 0.25 \Rightarrow \Pi = \$7{,}500{,}000$

**Hedge effectiveness intuition:**

A bond+CDS hedge can show P&L volatility during the "gray zone" because bond prices react continuously while CDS payout is discrete and timing-dependent.

**I'm not sure (scope/limits):**

Exact rules governing when delay is permissible and how it interacts with determinations/notice requirements depend on contract definitions and legal process beyond what is fully specified here.

---

### Example I: Restructuring Clause Difference — Conceptual with Numbers

**Source-backed facts to use:**
- Restructuring clauses: Old-Re (OR), Mod-Re (MR), Mod-Mod-Re (MMR), No-Re (NR), each imposing different deliverable constraints; No-Re removes restructuring as a credit event.
- The contract choice affects spreads; the source suggests an ordering reflecting delivery option value:

$$S_{\text{Old-Re}} > S_{\text{Mod-Mod-Re}} > S_{\text{Mod-Re}} > S_{\text{No-Re}}$$

**Toy numeric illustration:**

Two deliverables after a restructuring: Bond A at 43, Bond B at 37 (per 100).

Notional $N = \$10{,}000{,}000$ (face)

- If physical settlement is allowed with a broad deliverable set (delivery option present), buyer chooses cheapest-to-deliver (37) and effectively realizes $1 - 0.37 = 0.63$ payout-equivalent → $\$6.3$mm.
- If clause restricts deliverables such that only Bond A (43) is effectively deliverable, payout-equivalent is $1 - 0.43 = 0.57$ → $\$5.7$mm.

This mirrors the delivery option example logic (buyer can sell held asset at 43, buy cheaper at 37, deliver for par, capturing extra value).

**I'm not sure (important):**

The exact eligibility rules for deliverables, maturity limits, and when clauses apply must be read from the trade's ISDA Definitions and confirmation; the source itself says "for a full description, refer to ISDA 2003 definitions."

---

### Example J: Portfolio Jump-to-Default Exposure

**Assume** a portfolio where you are short protection (you pay on credit event). Three single-name CDS positions:

| Name | Notional $N_i$ |
|------|----------------|
| 1 | $\$5$mm |
| 2 | $\$8$mm |
| 3 | $\$12$mm |

For each name, jump-to-default cash outflow is approximately $N_i(1 - R_i)$. (This is the core payoff form.)

**Case 1:** $R = 40\%$ for all → LGD = 60%

| Name | Exposure |
|------|----------|
| 1 | $5 \times 0.60 = \$3.0$mm |
| 2 | $8 \times 0.60 = \$4.8$mm |
| 3 | $12 \times 0.60 = \$7.2$mm |
| **Total** | $3.0 + 4.8 + 7.2 = \$15.0$mm |

**Case 2:** Mixed recoveries $R_1 = 20\%, R_2 = 40\%, R_3 = 60\%$

| Name | $R$ | Exposure |
|------|-----|----------|
| 1 | 0.20 | $5 \times 0.80 = \$4.0$mm |
| 2 | 0.40 | $8 \times 0.60 = \$4.8$mm |
| 3 | 0.60 | $12 \times 0.40 = \$4.8$mm |
| **Total** | | $\$13.6$mm |

**Unit/bounds check:** exposures are between 0 and total notional (25mm) ✓

---

## Practical Notes

### 1) "Economic vs Contractual" Checklist

#### (A) Verified Facts

- Traders often say "default" when the contract trigger is "credit event."
- Credit event notice requires publicly available information (at least two sources) and the notice date is the Event Determination Date.
- Physical vs cash settlement distinction and auction/calc-agent determination of settlement price is described.

#### (B) Reasoned Inference

- **What traders mean by "default":** often "the credit has blown out / is distressed," or "a credit event is expected."
- **What the CDS contract requires:** a defined credit event, notice/evidence, then settlement mechanics.
- **Who determines the credit event:** I'm not sure based on the provided excerpts; you need the contract language / ISDA Definitions vintage and confirmation terms.

---

### 2) Common Pitfalls

- Treating recovery $R$ as fixed/known (recovery distributions are broad; sources note significant dispersion by seniority and type).
- Confusing bond price with "ultimate recovery": credit-derivatives recovery price can differ from workout recovery and can reflect supply/demand.
- Mixing up physical vs cash settlement payoffs; forgetting cheapest-to-deliver/delivery option effects.
- Ignoring accrued premium on default (it is part of the standard CDS cashflows).
- Assuming credit event definitions are universal across contracts: restructuring clauses differ (Old-Re, Mod-Re, Mod-Mod-Re, No-Re).

---

### 3) Implementation Pitfalls

- **Date handling:** payment schedule generation and accrual fractions (Actual/360; stub periods).
- **Notional and sign conventions:** buyer receives protection payment; seller pays it.
- **Settlement timing and operational frictions:** physical settlement can involve long timelines; short squeeze risk.

---

### 4) Verification Tests

| Test | Condition |
|------|-----------|
| Payout bounds | $0 \leq \Pi_{\text{prot}} \leq N$ |
| Recovery bounds | $0 \leq R \leq 1$ |
| Consistency | If settlement price equals recovery ($R = P/100$), physical and cash settlement align economically (Example E) |
| Accrued premium non-negativity | $N \cdot S \cdot \alpha \geq 0$ for $S, \alpha \geq 0$ |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. Economic default (distress) is not the same as a contractual credit event (legal trigger).
2. CDS pays on credit event, not on "spread widening."
3. Credit events include hard events (e.g., bankruptcy, failure to pay) and restructuring as the only soft event in the cited taxonomy.
4. Recovery $R$ and LGD $1 - R$ scale the protection payment.
5. "Recovery" can differ between credit-derivatives recovery price and workout recovery.
6. Bond price shortly after default is a common recovery proxy $R \approx P/100$.
7. Settlement: physical (deliver for par) vs cash (par minus settlement price from dealer poll/auction).
8. Multiple deliverables imply a delivery option (cheapest-to-deliver risk).
9. CDS premium includes accrued premium on default (a real cashflow).
10. Practical risks around default: jump-to-default, recovery uncertainty, and event/settlement risk (definition + timing + mechanics).

---

### Cheat Sheet (Definitions + Key Payoff Identities)

| Concept | Formula / Definition |
|---------|---------------------|
| Recovery rate | $R \in [0,1]$, fraction of par |
| LGD | $\text{LGD} = 1 - R$ |
| Market-implied recovery proxy | $R \approx P/100$ (from price per 100) |
| CDS protection payoff (to buyer) | $\Pi_{\text{prot}} = N(1 - R)$ |
| Accrued premium at default | $\Pi_{\text{accrued prem}} = N \cdot S \cdot \alpha$ |
| Physical settlement | Deliver obligations for par |
| Cash settlement | Par minus recovery/settlement price (dealer poll/auction; calculation agent/auction for cheapest-to-deliver) |

---

### Flashcards (30 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a "credit event"? | The legal/contractual trigger for CDS protection payment; similar to default but not identical. |
| 2 | Hard vs soft credit event—what's the key difference? | Hard events make debt immediately due/at one price; soft (restructuring) can leave a term structure of prices. |
| 3 | Name the soft credit event listed. | Restructuring. |
| 4 | Define recovery rate $R$. | Fraction of par recovered/settlement price as a fraction of par. |
| 5 | Define LGD. | $1 - R$. |
| 6 | CDS payoff to buyer in simplest form? | $N(1 - R)$. |
| 7 | Physical settlement meaning? | Buyer delivers deliverables; receives par cash. |
| 8 | Cash settlement meaning? | Seller pays par minus recovery price determined by dealer poll/auction. |
| 9 | What are "deliverable obligations"? | Specified bonds/loans protected and eligible to settle physically. |
| 10 | Why is there a "delivery option"? | Buyer chooses which deliverable(s) to deliver; valuable if post-event prices differ. |
| 11 | When is delivery option most valuable? | In soft events like restructuring with heterogeneous prices. |
| 12 | What happens to premium leg after credit event? | Stops, but accrued premium since last date is paid. |
| 13 | How is accrual fraction commonly computed in the mechanics section? | Actual/360 day count. |
| 14 | What is the Event Determination Date? | The date the Credit Event Notice is delivered. |
| 15 | What evidence supports a Credit Event Notice (per source)? | At least two sources of publicly available information. |
| 16 | Market measure of recovery rate? | Defaulted bond price divided by face value. |
| 17 | Why can derivatives recovery differ from workout recovery? | Timing, new information, and distressed-market supply/demand effects. |
| 18 | What is "technical default" in basis context? | Credit events may be broader than bond-default triggers. |
| 19 | Name a hard credit event: | Bankruptcy. |
| 20 | Name another hard credit event: | Failure to pay. |
| 21 | What is obligation acceleration? | Obligations become due earlier due to default/other and are accelerated. |
| 22 | Repudiation/moratorium applies most to? | Emerging market sovereign CDS (per description). |
| 23 | What does No-Re mean? | No-Restructuring; removes restructuring as a credit event. |
| 24 | What is Mod-Re? | Modified restructuring clause; limits deliverable maturities (US standard per table). |
| 25 | Why might physical settlement be problematic in aggregate? | Notional of CDS can exceed deliverable obligations causing sourcing pressure/short squeeze. |
| 26 | What mechanism may mitigate impossible physical settlement? | Fallback to cash settlement (source notes such mechanisms exist). |
| 27 | What happens to bond accrued coupons vs CDS accrued premium on default (basis implication)? | CDS pays accrued premium; bondholders can lose accrued coupon (basis driver). |
| 28 | How can restructuring clause affect spreads? | Broader deliverable sets increase delivery option value, tending to widen spreads (ordering given). |
| 29 | Example of auction-implied recovery? | Lehman recovery about 8 cents → payout about 92% (per source example). |
| 30 | What must you verify before relying on these rules for a trade? | The specific ISDA Definitions vintage and confirmation language (sources caution definitions evolve). |

---

## Mini Problem Set (16 Questions)

1. A CDS has notional $25mm and recovery $R = 0.35$. Compute protection payment.

2. Same as (1) but $R$ is quoted as price 42 per 100. Compute $R$ and payment.

3. A bond trades at 28 per 100 shortly after default. Compute market-implied recovery proxy and LGD.

4. Premium accrual: Notional $50mm, spread 150 bp, 40 days since last premium date on Act/360. Compute accrued premium.

5. Show algebraically that if the cash settlement price equals the market price of the deliverable, physical and cash settlement are economically aligned.

6. Delivery option: two deliverables at 45 and 38 after restructuring. What is the delivery-option value per 100 and for $10mm notional?

7. Explain why restructuring is classified as "soft" and how that impacts post-event pricing across maturities.

8. Identify three basis drivers related to contractual differences between CDS and bonds (use at least one from each source excerpt).

9. Consider a portfolio of 4 CDS shorts with notionals (5, 5, 10, 20) mm. Compute total jump-to-default exposure under $R = 20\%$ and $R = 60\%$.

10. A trader hedges a bond with CDS. The bond price collapses from 80 to 40 but no credit event is triggered for 30 days. Describe the hedge P&L risks qualitatively and list what contractual facts you would check.

11. For an index-like position, explain why "default definition" might differ from single-name CDS and how that could affect hedging.

12. Using the idea that credit-derivatives recovery price may differ from workout recovery, propose two reasons (from the sources) why they might diverge.

13. Given $R$ uncertainty with three scenarios, compute expected protection payment and standard deviation.

14. Consider a restructuring clause switch from Mod-Re to No-Re. What risks change for protection buyer/seller? Use only what is supported by the clause descriptions.

15. Explain how a "short squeeze" in deliverables could affect CDS payouts under physical settlement.

16. Provide a checklist for validating that a model's default cashflows are implemented consistently with premium accrual and settlement type.

---

## Solution Sketches (Questions 1–8)

**1.** $\Pi = N(1 - R) = 25 \times (1 - 0.35) = \$16.25$mm.

**2.** $R = 0.42$. $\Pi = 25 \times 0.58 = \$14.5$mm.

**3.** $R \approx 0.28$, LGD $\approx 0.72$.

**4.** $\alpha = 40/360 = 0.111111$. Accrued $= 50{,}000{,}000 \times 0.015 \times 0.111111 = \$83{,}333.33$.

**5.** Show $N - N(P/100) = N(1 - P/100)$ and set $R = P/100$.

**6.** Per 100: $45 - 38 = 7$. For $10mm: $0.07 \times 10{,}000{,}000 = \$700{,}000$.

**7.** Restructuring leaves a term structure; different maturities/coupons trade at different prices, unlike hard events tending to unify prices.

**8.** Examples: delivery option/cheapest-to-deliver; technical default broader triggers; accrued premium on default vs lost accrued coupon; restructuring clause payoff when no default.

---

## Source Map

### (A) Verified Facts

- CDS structure, legs, settlement mechanics: O'Kane Ch 5, Hull Ch 25
- Credit event taxonomy (hard/soft): O'Kane Ch 5
- Recovery rate definition and measurement: O'Kane Ch 3, Hull Ch 24
- Premium accrual at default mechanics: O'Kane Ch 5
- Restructuring clause variants: O'Kane Ch 5
- Lehman auction recovery example: Hull Ch 25

### (B) Reasoned Inference

- Economic vs contractual default distinction: derived from credit event definitions
- Physical/cash settlement equivalence: derived from contract mechanics descriptions
- Delivery option value logic: derived from cheapest-to-deliver principle

### (C) Speculation

- Exact auction operational steps: sources describe "dealer poll or auction" but do not fully specify mechanics
- Committee determination process: not specified in cited excerpts
- Specific ISDA Definitions vintage details: flagged as requiring contract-specific verification
