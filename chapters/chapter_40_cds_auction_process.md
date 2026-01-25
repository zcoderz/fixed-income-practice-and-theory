# Chapter 40: The CDS Auction Process — What It Does and Why It Exists

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- A credit event is the legal trigger for payment of the CDS protection leg (Hull Ch 25, O'Kane Ch 5)
- Standard CDS requires payment of accrued premium at default by the protection buyer (Hull Ch 25)
- Protection leg can be settled physically (deliver bonds/loans for par) or in cash (par minus recovery price) (O'Kane Ch 5, Hull Ch 25)
- Cash settlement requires a recovery/settlement price determined by a dealer poll or auction process (Hull Ch 25)
- Hull describes the (now usual) use of a two-stage auction to determine the mid-market value of the cheapest deliverable bond several days after the credit event
- A basket of deliverables supports contract standardization and liquidity, creating a delivery option when deliverables trade at different prices (O'Kane Ch 5)
- Physical settlement can create short squeezes when deliverables are scarce; cash settlement helps mitigate sourcing frictions (O'Kane Ch 5)
- ISDA organizes the auction process used to determine the value of the cheapest-to-deliver bond (Hull Ch 25)

### (B) Reasoned Inference (Derived from A)

- The cash settlement payout formula $N(1-R) = N(1 - FP_{100}/100)$ follows directly from the definition of recovery and the protection leg economics
- Physical and cash settlement deliver the same economic value transfer under the assumption that the cash-settlement final price equals the market value of the cheapest deliverable
- Every 1-point change in $FP_{100}$ changes payout by $0.01 \times N$

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure about the full operational step-by-step flow of the "two-stage auction," including how dealers submit bids/offers, how open interest is calculated, and the precise algorithm for determining the final price
- I'm not sure about exact timelines (cutoffs, settlement date conventions) beyond "several days after the credit event"
- I'm not sure about the full eligibility criteria and deliverable list construction rules without the applicable ISDA definitions

---

## Conventions & Notation

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional principal / face value (USD) |
| $FP_{100}$ | Auction/determination "final price" per 100 face (e.g., 35 means 35% of par) |
| $R$ | Recovery proxy as a fraction of par: $R = FP_{100}/100$ |
| $s$ | CDS running spread (per year, as a decimal; e.g., 90 bps $= 0.0090$) |
| $\alpha$ | Accrual fraction from last premium date to the credit event date (year fraction) |
| $\text{AccPrem}$ | Accrued premium: $N \cdot s \cdot \alpha$ |

### Key Identities

$$\boxed{R = \frac{FP_{100}}{100}}$$

$$\boxed{\text{Payout}_{\text{buyer}} = N(1-R) = N\left(1 - \frac{FP_{100}}{100}\right)}$$

### Price Convention

- Bond and auction prices are quoted per 100 of face value (e.g., "35" means 35% of par)
- Final price $FP_{100} \in [0, 100]$
- Recovery proxy $R \in [0, 1]$

### Sign Convention

- Positive cashflow is received by the protection buyer (paid by the protection seller), unless stated otherwise

### Defaults Used in Examples

| Convention | Default |
|------------|---------|
| Notional units | $N$ stated in \$mm, converted to dollars when computing |
| Price units | Per 100 of face value |
| Accrual day count | Actual/360 or as specified |

---

## Setup

This chapter focuses on standard CDS post–credit event settlement concepts: physical settlement vs cash settlement, the idea of a "cheapest deliverable," and how an auction-derived settlement price maps into CDS cashflows.

We describe auction mechanics only at the level supported by the reference books. Where the books do not specify operational steps, we explicitly say "I'm not sure."

---

## Core Concepts

### 1) Credit Event

**Formal Definition:**

The legal term for the event that triggers payment of the CDS protection leg. (Market participants often say "default" when they mean "credit event.")

**Intuition:**

A contractual trigger, not merely a rating action; once triggered, the CDS transitions from "premium-paying insurance" to a settlement process.

**Trading/Risk Practice:**

Event monitoring and confirmation matters because the settlement process (physical vs cash) and the valuation jump depend on the credit event trigger.

---

### 2) Premium Leg and Accrued Premium at Default

**Formal Definition:**

The premium leg consists of periodic premium payments. A standard CDS also requires the payment of coupon accrued at default: after a credit event, the protection buyer pays the fraction of premium accrued since the previous premium payment date.

**Intuition:**

Premium is paid "in arrears" over an accrual period; if the contract terminates early due to a credit event, the accrued portion up to the event date is still owed.

**Trading/Risk Practice:**

On event day, P&L includes (i) a large protection-leg receipt/payment and (ii) a smaller, but nonzero, accrued premium cashflow.

---

### 3) Protection Leg

**Formal Definition:**

The protection leg is a contingent payment from the protection seller to the protection buyer intended to "make up to par" the value of a qualifying obligation following a credit event.

**Intuition:**

Economically, the protection seller compensates the buyer for the shortfall from par caused by the default/credit event.

**Trading/Risk Practice:**

The protection leg is the dominant cashflow at default; its size depends on recovery / final price and (in physical settlement) on what can be delivered.

---

### 4) Physical Settlement

**Formal Definition:**

After a credit event, the protection buyer delivers face value of deliverable obligations to the protection seller; the seller pays par (face value) in cash simultaneously.

**Intuition:**

The CDS becomes a "put at par" on eligible defaulted obligations.

**Trading/Risk Practice:**

Physical settlement can create sourcing problems when many protection buyers need deliverables.

---

### 5) Cash Settlement

**Formal Definition:**

The protection seller pays the protection buyer the face value of protection minus a recovery price (in cash). In one framing, the recovery price is for a specified reference obligation and is determined by a dealer poll or auction process.

**Intuition:**

Instead of exchanging bonds/loans, the parties exchange cash equal to the "loss."

**Trading/Risk Practice:**

The key operational requirement is: what price/recovery is used? The market uses a poll/auction mechanism for that purpose.

---

### 6) Reference Obligation

**Formal Definition (as supported):**

A single specified bond or loan of the reference entity used in the cash-settlement framing in the source; deliverable obligations must be pari passu or senior to the reference obligation.

**Intuition:**

A contractual anchor instrument that helps define seniority and what can be delivered.

**Trading/Risk Practice:**

Traders must verify what the contract specifies as the reference obligation and how it interacts with deliverables and seniority.

---

### 7) Deliverable Obligations (Deliverables)

**Formal Definition (as supported):**

Bonds and/or loans that qualify to be delivered into a physically settled CDS; there can be many qualifying deliverables, forming a basket.

**Intuition:**

A basket of deliverables supports a standard, liquid contract that can hedge multiple cash instruments of the same issuer.

**Trading/Risk Practice:**

Deliverable sets matter because different deliverables can trade at different prices after a credit event.

---

### 8) Cheapest-to-Deliver and the Delivery Option

**Formal Definition:**

If multiple deliverables exist and can trade at different prices, the protection buyer can choose which deliverable(s) to deliver, creating a delivery option (effectively, a "cheapest-to-deliver" option).

**Intuition:**

Under physical settlement, the buyer will prefer delivering the cheapest eligible obligation because they receive par in exchange.

**Trading/Risk Practice:**

This optionality is beneficial to protection buyers and adverse to protection sellers; it also motivates careful contract specification and motivates cash settlement approaches that reference "cheapest deliverable" economics.

---

### 9) Auction / Poll-Derived Recovery Price (the "Final Price" Concept)

**Formal Definition (as supported):**

- Cash settlement uses a recovery price determined by a dealer poll or auction process
- In the Hull exposition, "as is now usual" a two-stage auction process determines the mid-market value of the cheapest deliverable bond several days after the credit event; that value determines the cash payoff

**Intuition:**

The auction is a mechanism to transform a messy post-default market into a single settlement price used for cash settlement.

**Trading/Risk Practice:**

Traders use the "final price" as the settlement input converting a CDS position into a cash payment and as a realized recovery proxy for P&L explain.

---

## Auction Purpose and Economic Logic

### Why an Auction Exists (The Problem It Solves)

**Settlement frictions in physical settlement:**

Physical settlement requires sourcing and delivering eligible bonds/loans. When outstanding CDS protection is large relative to deliverable obligations, a short squeeze can occur: protection buyers who lack deliverables try to buy them, pushing prices up and reducing the loss payment.

**Why cash settlement is allowed / used:**

Allowing cash settlement is intended to overcome the problem of sourcing physical deliverables, especially when CDS notional outstanding is similar to or larger than outstanding deliverable obligations.

In the Hull text, cash settlement is described as "now usual," with a two-stage auction used to obtain the settlement value.

**Standardization and transparency (economic logic, derived from the above):**

A single, commonly accepted final price supports standardized cash settlement across many contracts referencing the same credit event, rather than forcing each pair of counterparties to negotiate or source physical bonds.

---

### How Auction Settlement Connects to CDS Cash Settlement Payoff

**Core payoff identity (cash settlement):**

If the auction/poll determines a bond value of $FP_{100}$ per 100 of face value, the loss per 100 is $100 - FP_{100}$.

Therefore the cash settlement amount (received by the protection buyer) is:

$$\boxed{\text{Payout}_{\text{buyer}} = N\left(1 - \frac{FP_{100}}{100}\right) = N(1-R)}$$

This is directly consistent with: "cash payoff is based on the excess of the face value … over the estimated value" and the payoff $L(1-R)$.

**Numerical illustration in the source:** if the auction indicates $FP_{100} = 35$, the payoff is $65\%$ of notional (e.g., \$65mm on \$100mm).

---

### Physical Settlement vs Cash Settlement and the Role of "Cheapest-to-Deliver"

**Physical settlement deliverables:**

Under physical settlement, the buyer can deliver qualifying obligations and receive par, creating a delivery option when deliverables trade at different prices.

**Cash settlement requires a single price:**

If cash settlement is used, the contract still needs a number that represents the post-event value used to compute the loss. The sources describe that this is obtained via a dealer poll or auction and is often framed around the cheapest deliverable bond and its mid-market value.

**Economic connection:**

- Physical settlement "naturally" embeds cheapest-to-deliver behavior (buyer delivers cheapest eligible asset)
- Cash settlement aims to embed similar economics by using a settlement price linked to cheapest deliverable (as described in Hull) so that the cash payout corresponds to the loss implied by the cheapest deliverable's value

---

## Auction Mechanics (Describe ONLY What the Sources Support)

### What We Can Say with Certainty (Supported by the Books)

- A dealer poll or auction process determines the recovery/settlement price used for cash settlement
- A two-stage auction process is used (in Hull's description) to determine the mid-market value of the cheapest deliverable bond several days after a credit event (cash settlement is described as "now usual")
- An auction (or calculation agent) determines the value of the cheapest-to-deliver bonds a specified number of days after the default event, and the cash payoff is based on face value minus that estimated value
- ISDA is identified (in Hull) as organizing the auction process used to determine the value of the cheapest-to-deliver bond and therefore the payoff

### What Is Unknown / Not Fully Specified in the Sources

I'm not sure about the full operational step-by-step flow of the "two-stage auction," including (examples):

- How dealers submit bids/offers and how these are constrained
- How "open interest" (net physical settlement demand/supply) is calculated and incorporated
- The precise algorithm for converting submissions into the final price
- Exact timelines (e.g., "day 0," "day 1," cutoffs, settlement date conventions) beyond "several days after the credit event"

**External document(s) needed to confirm these details (not relied on here):**

- ISDA Credit Derivatives Auction Settlement documentation (protocols and auction methodology)
- ISDA Determinations Committee (DC) rules / procedures for credit event determinations (if relevant to the user's trade)
- The applicable ISDA Credit Derivatives Definitions for the contract (the books themselves point readers to ISDA definitions for full descriptions in related contexts)

### "Final Price" Definition and Mapping (As Used in These Notes)

**Final price (definition in this chapter):** $FP_{100}$ is the cash settlement price per 100 determined by a dealer poll or auction process, described in the sources as the mid-market value of the cheapest deliverable bond used for cash settlement.

**Recovery mapping:** We use $R = FP_{100}/100$ as a recovery proxy consistent with "recovery rate … is the value of the bond shortly after the issuer defaults as a percentage of its face value."

**CDS payout mapping (cash settlement):** $\text{Payout} = N(1-R) = N(1 - FP_{100}/100)$

### Deliverable Obligations Definition (Only to the Extent Supported)

**Deliverable obligations:** the set of qualifying bonds/loans that can be delivered under physical settlement; there may be many, and defining a basket supports standard contracts and liquidity.

I'm not sure about the full eligibility criteria, documentation tests, and specific deliverable list construction rules for a given auction/event without the applicable ISDA definitions and auction settlement documentation (see above).

---

## Math and Derivations

### 1) Cash Settlement Payout Formula

**Step 1: Define price and recovery proxy.**

Let $FP_{100}$ be the final price per 100 of face (e.g., 35 means 35% of par). Define recovery fraction:

$$R = \frac{FP_{100}}{100}$$

**Step 2: Loss per unit notional.**

Per 1 dollar of face, the recovered value is $R$ dollars; the loss is $1-R$.

**Step 3: Scale by notional.**

For notional $N$ (USD), the protection buyer receives:

$$\boxed{\text{Payout}_{\text{buyer}} = N(1-R) = N\left(1 - \frac{FP_{100}}{100}\right)}$$

This matches the source statement that cash payoff is based on face value minus estimated value and the payoff is $L(1-R)$.

**Unit Check:**
- $N$: USD
- $R$: dimensionless
- $\text{Payout}$: USD ✓

**Sanity Checks:**
- If $FP_{100} = 100 \Rightarrow R = 1$: $\text{Payout} = 0$. No loss, no protection payment.
- If $FP_{100} = 0 \Rightarrow R = 0$: $\text{Payout} = N$. Total loss, full notional paid.
- Bounds: for $0 \le FP_{100} \le 100$, $0 \le \text{Payout} \le N$.

---

### 2) Relationship Between Bond Price (Per 100) and Recovery Proxy

Hull defines the recovery rate for a bond as the bond's value shortly after default as a percentage of face value.

Thus if a defaulted bond trades at $P_{100}$ per 100 face, it is natural (and consistent with the definition) to interpret:

$$R \approx \frac{P_{100}}{100}$$

In the auction framing used in Hull's CDS example, $FP_{100}$ is the mid-market value of the cheapest deliverable bond and functions as the settlement price used for cash settlement.

---

### 3) Accrued Premium Reminder (Cashflow Around the Event)

The sources emphasize that following a credit event, the protection buyer typically owes the premium accrued since the previous premium payment date; this is a distinct cashflow from the protection payout.

Let $s$ be the running spread (per year), and let $\alpha$ be the year fraction accrued from last payment date to the credit event date. Then:

$$\boxed{\text{Accrued Premium} = N \cdot s \cdot \alpha}$$

**Interpretation:** this is paid by the protection buyer to the protection seller; the protection leg payout flows in the opposite direction.

**Unit Check:**
- $N$: USD
- $s$: 1/year
- $\alpha$: years
- $N \cdot s \cdot \alpha$: USD ✓

---

## Measurement & Risk (Only What Belongs in Chapter 40)

### Final Price Uncertainty (Recovery / Settlement Price Risk)

**What it is:** The risk that the realized settlement price $FP_{100}$ (and hence $R$) differs from what was implied by market pricing or an internal recovery assumption before the event.

**Why it matters:** The default-time cashflow $N(1-R)$ is highly sensitive to $R$. A 10-point move in $FP_{100}$ changes payout by $0.10 \times N$.

**Where it appears in P&L explain:** For a protection buyer, lower-than-expected $FP_{100}$ increases realized payout; for a protection seller it increases realized loss.

---

### Deliverable Liquidity / Dispersion Risk

**What it is:** Different deliverable obligations can trade at different prices after a credit event; the delivery option has value only when such dispersion exists.

**Why dispersion can occur (supported examples):** The Hull text notes reasons bonds may not trade at the same percentage of face value immediately after default, including accrued interest differences and differing expectations about outcomes across bondholders.

**P&L explain channel:** If you hedge a specific bond with CDS, the hedge effectiveness depends on how that bond's post-event value compares to the "cheapest deliverable" reference used to set the cash settlement price. (This is the economic origin of basis-type residuals.)

---

### Basis Risk Preview: Bond vs CDS Settlement Price

**Preview only (supported framing):** O'Kane notes multiple market factors (including relative liquidity and supply/demand effects) that can drive divergences between cash bond and CDS markets.

**Auction-specific angle (reasoned):** Even at the credit event, if the auction final price is anchored to a cheapest deliverable while your bond position is in a different deliverable (or a non-deliverable instrument), your realized hedge outcome may deviate from "perfect" recovery locking.

---

### Operational / Timing Risk (As Supported; Otherwise "I'm Not Sure")

**Physical settlement timing can be lengthy:** O'Kane's description implies up to 72 calendar days after initial notification until payment on the protection leg must be made in physical settlement.

**Cash settlement timing (limited support):** Hull describes the auction as occurring several days after the credit event.

I'm not sure about the detailed auction timeline milestones (submission windows, publication times, settlement date conventions) without ISDA auction settlement documentation.

---

### A Simple P&L Explain Decomposition Around the Event (Conceptual + Numeric)

Let $FP_{100}^{\text{exp}}$ be the expected final price used in risk, and $FP_{100}^{\text{real}}$ be the realized auction final price.

For a protection buyer, the realized protection-leg cashflow is:

$$N\left(1 - \frac{FP_{100}^{\text{real}}}{100}\right)$$

A simple "auction shock" component is the difference:

$$\Delta\text{Payout} \approx N\left(\frac{FP_{100}^{\text{exp}} - FP_{100}^{\text{real}}}{100}\right)$$

**Numeric illustration:** If $N = 50$mm, $FP_{100}^{\text{exp}} = 40$, $FP_{100}^{\text{real}} = 30$,

$$\Delta\text{Payout} = 50 \times \frac{40-30}{100} = 50 \times 0.10 = 5 \text{ mm}$$

So the protection buyer receives \$5mm more than expected; the seller loses \$5mm more than expected. (Accrued premium is a separate, smaller offset cashflow.)

---

## Worked Examples (At Least 10 Numeric Examples)

**Convention used in all examples:** prices are per 100 of face; notional $N$ is in \$mm; payout amounts are shown in \$mm.

---

### Example A — Cash Settlement Payout and Implied Recovery

**Given:** $N = 10$mm, $FP_{100} = 38$.

**Recovery fraction:**

$$R = \frac{38}{100} = 0.38$$

**Payout to protection buyer:**

$$\text{Payout} = 10 \times (1 - 0.38) = 10 \times 0.62 = 6.2 \text{ mm}$$

**Answer:** Implied recovery $R = 0.38$; cash settlement payout $= \$6.2$mm.

---

### Example B — Include Accrued Premium at Default and Net Cashflows (Buyer vs Seller)

*Supported by the sources: accrued premium is usually paid, and Hull provides an illustrative computation approach.*

**Given (similar to Hull's example values):**

- $N = 100$mm
- Running spread $s = 90$ bps $= 0.0090$
- Credit event occurs 2 months into an accrual year (use $\alpha = 2/12$)
- Final price $FP_{100} = 35 \Rightarrow R = 0.35$

**Step 1: Protection payout**

$$\text{Payout} = 100 \times (1 - 0.35) = 100 \times 0.65 = 65 \text{ mm}$$

**Step 2: Accrued premium**

$$\text{AccPrem} = 100 \times 0.0090 \times \frac{2}{12} = 100 \times 0.0090 \times 0.1667 = 0.15 \text{ mm}$$

**Step 3: Net cashflows**

- **Protection buyer:** receives $+65.00$mm and pays accrued premium $-0.15$mm
  $\Rightarrow \text{Net} = +64.85$mm
- **Protection seller:** pays $-65.00$mm and receives accrued premium $+0.15$mm
  $\Rightarrow \text{Net} = -64.85$mm

**Answer:** Accrued premium $= \$0.15$mm; net to buyer $+\$64.85$mm; net to seller $-\$64.85$mm.

---

### Example C — Two Deliverables Priced 35 and 45: CTD Under Physical Settlement

*Supported: deliverable baskets exist; buyer has a delivery option; dispersion makes it valuable; auction used to set value tied to cheapest deliverable.*

**Given:** $N = 10$mm, two eligible deliverables trade at 35 and 45 per 100.

**Physical settlement logic:**

Protection buyer wants to deliver the cheapest deliverable (35).

If buyer can purchase the deliverable at 35 and deliver for par (100), value transfer per 100 is:

$$100 - 35 = 65$$

Scaling:

$$10 \times 0.65 = 6.5 \text{ mm}$$

**Why a single final price is needed for cash settlement:**

In cash settlement, you need one number $FP_{100}$ to compute $N(1 - FP_{100}/100)$.

If different deliverables trade at 35 and 45, cash settlement must specify which economic reference (e.g., "cheapest deliverable mid") to avoid disputes and inconsistent bilateral outcomes. Hull describes using the mid-market value of the cheapest deliverable bond determined by auction.

**Answer:** Physical settlement embeds CTD (35). An auction-derived $FP_{100}$ provides a single cash-settlement price consistent with that CTD framing.

---

### Example D — Sensitivity of Payout to Final Price: 35 vs 45 vs 40

**Given:** $N = 25$mm.

**Compute payout for each $FP_{100}$:**

| $FP_{100}$ | $R$ | Payout |
|------------|-----|--------|
| 35 | 0.35 | $25 \times 0.65 = 16.25$mm |
| 40 | 0.40 | $25 \times 0.60 = 15.00$mm |
| 45 | 0.45 | $25 \times 0.55 = 13.75$mm |

**Interpretation:** Every 1-point change in $FP_{100}$ changes payout by $25 \times 0.01 = 0.25$mm.

---

### Example E — "Jump-to-Recovery" Exposure Proxy: Recovery Shifts 40% to 25%

**Given:** $N = 20$mm.

**Payout at $R = 0.40$:**

$$20 \times (1 - 0.40) = 20 \times 0.60 = 12.0 \text{ mm}$$

**Payout at $R = 0.25$:**

$$20 \times (1 - 0.25) = 20 \times 0.75 = 15.0 \text{ mm}$$

**Difference (buyer's benefit / seller's loss):**

$$15.0 - 12.0 = 3.0 \text{ mm}$$

**Answer:** A recovery drop from 40% to 25% increases payout by \$3.0mm on \$20mm notional.

---

### Example F — Portfolio Aggregation: Three CDS Positions with Different Final Prices

**Given:**

| Trade | Notional $N_i$ | $FP_{100,i}$ |
|-------|----------------|--------------|
| 1 | 10mm | 30 |
| 2 | 15mm | 55 |
| 3 | 5mm | 80 |

**Compute each payout:**

| Trade | $R_i$ | Payout |
|-------|-------|--------|
| 1 | 0.30 | $10 \times 0.70 = 7.0$mm |
| 2 | 0.55 | $15 \times 0.45 = 6.75$mm |
| 3 | 0.80 | $5 \times 0.20 = 1.0$mm |

**Total payout:**

$$7.0 + 6.75 + 1.0 = 14.75 \text{ mm}$$

**Concentration:** largest contribution is Trade 1 (7.0mm), which is $7/14.75 \approx 47.5\%$.

---

### Example G — Toy "Auction Shock" on a Hedged Position (Bond + CDS) and Residual

*Supported: CDS hedges bond default risk; auction gives settlement price; basis factors exist (preview).*

**Setup (toy hedge):**

- Hold a defaultable bond with face $N = 10$mm
- Buy CDS protection on same reference entity with notional $N = 10$mm
- Expected $FP_{100}^{\text{exp}} = 40$, realized $FP_{100}^{\text{real}} = 30$

**Assumption (stated clearly):** The bond's post-event value equals the auction final price (i.e., the bond you hold is priced at $FP_{100}^{\text{real}}$). This assumption may fail in practice if your bond is not aligned with the "cheapest deliverable" concept.

**Step 1: Bond value after event**

$$\text{Bond value} = 10 \times 0.30 = 3.0 \text{ mm}$$

**Step 2: CDS payout after event**

$$\text{CDS payout} = 10 \times (1 - 0.30) = 7.0 \text{ mm}$$

**Step 3: Combined value**

$$3.0 + 7.0 = 10.0 \text{ mm (par)}$$

**Auction shock P&L relative to expectation:**

- Expected bond value: $10 \times 0.40 = 4.0$mm
- Expected CDS payout: $10 \times 0.60 = 6.0$mm
- Expected combined: $10.0$mm
- Realized combined: $10.0$mm

**Residual explanation (conceptual):** Under the stated assumption, the hedge is perfect regardless of $FP$. Residual arises if:
- your bond is not priced at the auction-referenced price (deliverable dispersion / non-CTD),
- timing and accrued premium cashflows differ,
- or market frictions create bond–CDS basis effects.

---

### Example H — Reconcile Physical vs Cash Settlement Economics (Under a Stated Assumption)

*Supported: physical settlement is delivery for par; cash settlement is par minus recovery price; auction determines value used for cash settlement.*

**Given:** $N = 5$mm, suppose the cheapest deliverable trades at $FP_{100} = 35$.

**Assumption:** The cash-settlement final price equals the market value of the deliverable used in physical settlement (the cheapest deliverable).

**Physical settlement economics:**

- Buyer delivers bonds with face 5mm (market value $= 5 \times 0.35 = 1.75$mm)
- Seller pays par $= 5.00$mm
- Economic gain to buyer from protection leg $= 5.00 - 1.75 = 3.25$mm

**Cash settlement economics:**

$$\text{Payout} = 5 \times (1 - 0.35) = 5 \times 0.65 = 3.25 \text{ mm}$$

**Answer:** Under the assumption, physical and cash settlement deliver the same economic value transfer.

---

### Example I — Timeline Cashflow Table: Default Between Coupon Dates (Buyer/Seller Sign Table)

*Supported: premium accrues and a final accrual payment is usually required; cash settlement happens after the event (several days in Hull's description).*

**Given:**

- $N = 50$mm
- $s = 200$ bps $= 0.0200$
- Default occurs halfway through the quarter: $\alpha = 0.25/2 = 0.125$ years
- $FP_{100} = 40 \Rightarrow R = 0.40$

**Compute amounts:**

- Protection payout: $50 \times (1 - 0.40) = 50 \times 0.60 = 30.0$mm
- Accrued premium: $50 \times 0.0200 \times 0.125 = 0.125$mm

**Cashflow sign table (positive = received by the row party):**

| Cashflow Item | Protection Buyer | Protection Seller |
|---------------|------------------|-------------------|
| Protection leg settlement (after event) | +30.000 mm | −30.000 mm |
| Accrued premium at event | −0.125 mm | +0.125 mm |
| **Net** | **+29.875 mm** | **−29.875 mm** |

---

### Example J — "What If" Deliverable Set Changes (CTD Economics)

*Sources support that deliverables can be a basket and CTD matters; exact deliverable rules are contractual and not fully specified here.*

**Given:** $N = 10$mm. Two potential deliverables: A at 30, B at 50.

**Case 1 (both deliverables allowed):** CTD is A at 30.

$$\text{Payout proxy} = 10 \times (1 - 0.30) = 7.0\text{mm}$$

**Case 2 (CTD removed; only B allowed):** CTD becomes B at 50.

$$\text{Payout proxy} = 10 \times (1 - 0.50) = 5.0\text{mm}$$

**Difference:** $7.0 - 5.0 = 2.0$mm.

I'm not sure how an actual CDS auction's deliverable set can be changed or what rules govern inclusion/exclusion without the applicable ISDA definitions and auction settlement documentation.

---

## Practical Notes

### What a Practitioner Must Know on Event Day

1. Reference entity and which contracts are impacted (legal entity mapping)
2. Whether the contract is physical or cash settled (and whether a fallback may apply)
3. Notional $N$
4. Running spread $s$, premium schedule, and accrued premium calculation inputs (last coupon date, credit event date)
5. The settlement input: auction/poll derived final price $FP_{100}$ and how it maps to payout $N(1 - FP_{100}/100)$

### Common Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Mixing price-per-100 with a fraction (38 vs 0.38) | Off by factor of 100 |
| Forgetting accrued premium | Often small relative to protection payout, but not zero |
| Assuming auction mechanics (stages, bids/offers, open interest) without sourcing | Don't — verify against ISDA docs |
| Treating auction final price as "true recovery" without caveats | Deliverable dispersion and liquidity can create gaps between different obligations' prices |

### Verification Tests

1. **Payout bounds:** if $0 \le FP_{100} \le 100$, then $0 \le N(1 - FP_{100}/100) \le N$
2. **Mirror symmetry:** buyer's net cashflows = $-$(seller's net cashflows)
3. **Physical vs cash alignment (under stated assumption):** if cash settlement price equals the deliverable's market value, then physical and cash settlement economics match (Example H)

---

## Summary & Recall

### Executive Summary (10 Bullets)

1. A CDS credit event triggers settlement of the protection leg; "credit event" is the legal trigger
2. Standard CDS requires payment of accrued premium at default by the protection buyer
3. Protection leg can be settled physically (deliver bonds/loans for par) or in cash (par minus recovery price)
4. Cash settlement requires a recovery/settlement price determined by a dealer poll or auction
5. Hull describes the (now usual) use of a two-stage auction to determine the mid-market value of the cheapest deliverable bond several days after the credit event
6. The cash settlement payout is $N(1-R) = N(1 - FP_{100}/100)$
7. A basket of deliverables supports contract standardization and liquidity, but gives the buyer a delivery (CTD) option if deliverables trade at different prices
8. Physical settlement can create short squeezes when deliverables are scarce; cash settlement helps mitigate sourcing frictions
9. Auction final price is a settlement price / recovery proxy, not necessarily a unique "true" recovery for every obligation (deliverable dispersion matters)
10. Many operational auction details are not specified in these sources; verify against ISDA auction documentation for your specific trade

### Cheat Sheet: Definitions + Formulas + Day-of-Event Checklist

**Key Definitions:**

| Term | Definition |
|------|------------|
| Credit event | Legal trigger for protection leg settlement |
| Deliverables | Qualifying bonds/loans that can be delivered; basket supports standardization |
| Final price $FP_{100}$ | Auction/poll-derived settlement price per 100 (often described via cheapest deliverable mid) |

**Key Formulas:**

| Formula | Expression |
|---------|------------|
| Recovery fraction | $R = FP_{100}/100$ |
| Cash settlement payout | $\boxed{N(1-R) = N\left(1 - \frac{FP_{100}}{100}\right)}$ |
| Accrued premium | $\boxed{N \cdot s \cdot \alpha}$ |

**Day-of-Event Checklist:**

- [ ] Confirm credit event trigger per contract (legal determination)
- [ ] Identify settlement type: physical vs cash; note any fallback language
- [ ] Record notional $N$, spread $s$, last premium date, and event date for accrual
- [ ] Obtain $FP_{100}$ (auction/poll) used for settlement and compute payout
- [ ] Reconcile signs: buyer receives protection, pays accrued premium

---

### Flashcards (30)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a "credit event" in CDS terminology? | The legal trigger for the protection leg settlement |
| 2 | What are the two main ways a CDS protection leg can be settled? | Physical settlement or cash settlement |
| 3 | In physical settlement, what does the protection buyer deliver and receive? | Delivers face value of deliverables; receives par in cash |
| 4 | In cash settlement, what determines the settlement price? | A dealer poll or auction process determines a recovery/settlement price |
| 5 | What does Hull describe the auction as determining? | Mid-market value of the cheapest deliverable bond several days after the credit event |
| 6 | What is the cash payout formula using recovery fraction $R$? | $N(1-R)$ |
| 7 | How do you convert $FP_{100}$ to $R$? | $R = FP_{100}/100$ |
| 8 | What is "accrued premium at default"? | Premium accrued since last payment date that the buyer pays upon credit event |
| 9 | Why does a basket of deliverables improve CDS liquidity? | One standard contract can hedge multiple bonds/loans meeting criteria |
| 10 | What is the "delivery option"? | Buyer can choose among deliverables; valuable if deliverables trade at different prices |
| 11 | Why can deliverables trade at different prices after default? | Accrued interest differences and differing expectations about outcomes across bondholders, among others |
| 12 | What is "cheapest-to-deliver" in CDS context? | The deliverable obligation with the lowest price that the buyer would prefer to deliver |
| 13 | How does an auction help cash settlement? | Produces a single settlement price used to compute cash payouts |
| 14 | What problem can arise when many CDS are physically settled? | A short squeeze in deliverables, raising prices and affecting payouts |
| 15 | Why is cash settlement allowed (per O'Kane)? | To overcome sourcing issues when CDS notional is large vs deliverables outstanding |
| 16 | What is the reference obligation in O'Kane's cash-settlement framing? | A single specified bond or loan of the reference entity |
| 17 | What seniority restriction is noted for deliverables vs reference obligation? | Deliverables must be pari passu or senior to the reference obligation |
| 18 | In Hull's example, what does $FP_{100} = 35$ imply about payout? | Payout is 65% of notional |
| 19 | What is final price uncertainty? | Risk that realized $FP_{100}$ differs from expected, changing payout |
| 20 | What is deliverable dispersion risk? | Risk that deliverables trade at different prices affecting CTD economics |
| 21 | What is basis risk (preview)? | Bond vs CDS differences due to market factors like liquidity/supply-demand |
| 22 | What does "mid-market value" mean in the auction description? | A midpoint valuation (exact construction not specified in these sources) |
| 23 | What auction detail is not specified in these books? | Exact bid/offer submission and pricing algorithm steps (need ISDA docs) |
| 24 | What cashflow stops after a credit event? | Regular premium payments cease, but a final accrued payment is usually required |
| 25 | How can you sanity-check a computed payout? | Ensure $0 \le \text{payout} \le N$ when $0 \le FP_{100} \le 100$ |
| 26 | If $FP_{100}$ falls by 10 points, how does payout change? | Increases by $0.10 \times N$ for buyer (decreases for seller) |
| 27 | Why isn't the auction final price always "true recovery"? | Different obligations can trade at different prices; liquidity matters |
| 28 | What is a "short squeeze" impact on defaulted asset price? | Buying pressure can increase prices, reducing loss payment |
| 29 | What did Hull report about Lehman's recovery rate? | About eight cents on the dollar, determined by an auction process |
| 30 | What documents should you consult for exact auction protocol? | ISDA Credit Derivatives Auction Settlement docs and related ISDA rules (not in these books) |

---

## Mini Problem Set (16 Questions)

1. A CDS has $N = 12$mm and $FP_{100} = 25$. Compute payout and implied recovery.

2. If $N = 40$mm, compare payouts for $FP_{100} = 15$ and $FP_{100} = 55$. What is the difference?

3. A CDS has spread $s = 500$ bps and notional $N = 30$mm. A credit event occurs 1 month into the quarter. Using $\alpha = 1/12$, compute accrued premium.

4. A protection buyer has $N = 50$mm. If $FP_{100}$ is 10 points lower than expected, what is the payout surprise?

5. Two deliverables trade at 32 and 48. Under physical settlement, what is the CTD and the implied "loss per 100"?

6. Show that $0 \le \text{payout} \le N$ when $0 \le FP_{100} \le 100$.

7. Given $N = 20$mm, compute payout for $R = 0.30$. Then compute payout for $R = 0.60$. Interpret.

8. Construct a net cashflow for buyer and seller including a $0.05$mm accrued premium and a $6.0$mm protection payout.

9. Explain (conceptually) why deliverable dispersion creates hedge residuals for a bond + CDS hedge.

10. Explain (conceptually) why a short squeeze can reduce the protection payout under physical settlement.

11. Provide a scenario where a cash-settled payoff might differ from your specific bond's post-default value.

12. Describe how final price uncertainty affects risk limits and reserves for a CDS book near distress.

13. Explain why the auction final price might differ across seniority tiers (conceptual; do not assume rules).

14. If you have three CDS positions, how would you attribute total default P&L to (i) recovery surprise and (ii) accrual?

15. Discuss how the presence of a delivery option affects CDS spreads (conceptual).

16. List at least five operational data items you would need to process a credit event settlement.

---

### Solution Sketches (Questions 1–8)

**1.** $R = 0.25$. Payout $= 12 \times (1 - 0.25) = 9$mm.

**2.** $FP = 15 \Rightarrow 40 \times 0.85 = 34$mm; $FP = 55 \Rightarrow 40 \times 0.45 = 18$mm; difference $= 16$mm.

**3.** Accrued premium $= 30 \times 0.05 \times (1/12) = 0.125$mm.

**4.** Surprise $= N \times 0.10 = 50 \times 0.10 = 5$mm.

**5.** CTD is 32; loss per 100 is $100 - 32 = 68$.

**6.** Since $FP_{100}/100 \in [0,1]$, we have $1 - FP_{100}/100 \in [0,1]$; multiplying by $N \ge 0$ preserves bounds.

**7.** $R = 0.30 \Rightarrow 20 \times 0.70 = 14$mm; $R = 0.60 \Rightarrow 20 \times 0.40 = 8$mm; higher recovery → lower payout.

**8.** Buyer net $= +6.0 - 0.05 = +5.95$mm; seller net $= -6.0 + 0.05 = -5.95$mm.

---

## Source Map

### (A) Verified Facts

- Credit event definition, protection leg mechanics, physical vs cash settlement: Hull Ch 25, O'Kane Ch 5
- Accrued premium at default: Hull Ch 25
- Auction process and "now usual" cash settlement: Hull Ch 25
- Deliverable baskets, delivery option, cheapest-to-deliver: O'Kane Ch 5, Hull Ch 25
- Short squeeze risk in physical settlement: O'Kane Ch 5
- ISDA organizing auction: Hull Ch 25

### (B) Reasoned Inference

- Cash settlement payout formula: derived from protection leg definition and recovery definition
- Physical/cash equivalence under stated assumption: algebraic verification
- Sensitivity ($\Delta$payout per point of $FP_{100}$): direct differentiation

### (C) Speculation / Uncertainty

- Full auction operational mechanics (two-stage process details, bid/offer constraints, open interest calculation, timeline milestones): Not specified in sources; flagged as "I'm not sure"
- Deliverable eligibility criteria and documentation tests: Not fully specified; refer to ISDA definitions
