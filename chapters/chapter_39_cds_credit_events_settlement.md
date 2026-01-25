# Chapter 39: CDS Credit Events and Settlement Choices — Physical vs Cash Settlement, Auctions, and Why Cheapest-to-Deliver Matters

---

## Conventions & Notation

| Symbol | Definition |
|--------|------------|
| **Perspective / sign convention** | Payoffs are written from the protection buyer's perspective (positive = cash received / benefit to protection buyer) |
| $N$ | CDS notional (USD) |
| **Price quotation** | Bond/loan "prices" are quoted per 100 of face unless stated otherwise. A quoted price of 38 means $38/100 = 0.38$ of par |
| $P_{\text{final}}$ | Final price / recovery price (per 100) used to compute the protection payment in cash settlement (as determined by a dealer poll or auction process, per sources) |
| $R_{\text{settle}}$ | Settlement-implied recovery fraction: $R_{\text{settle}} := P_{\text{final}}/100$. (This is a settlement-implied recovery; do not confuse with a long-run model recovery parameter) |
| $\mathcal{D}$ | Set of eligible deliverable obligations (bonds/loans) for physical settlement, as defined by documentation / confirmation |
| $P_i$ | Market price (per 100) of deliverable $i \in \mathcal{D}$ after the credit event |
| $P_{\text{CTD}}$ | Cheapest-to-deliver price: $P_{\text{CTD}} := \min_{i \in \mathcal{D}} P_i$ |

---

## Setup

### Conventions used in this chapter

- We focus on contract mechanics and settlement economics (credit events, physical vs cash settlement, CTD/delivery option, auction concept).
- We treat the settlement payment as occurring "near" the credit event for intuition; discounting/curve building is intentionally omitted (pricing/bootstrapping is later).
- Where the books refer to specific ISDA definition vintages or conventions, we repeat them as book-reported and flag that documentation can evolve. The sources themselves warn that definitions may change and that legal advice should be sought.

### Notation glossary (symbols + definitions)

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional (USD) |
| $P_{\text{final}}$ | Final price (per 100) used for cash settlement; determined via dealer poll or auction process (per sources) |
| $R_{\text{settle}} = P_{\text{final}}/100$ | Settlement-implied recovery fraction |
| $\mathcal{D}$ | Set of eligible deliverable obligations for physical settlement (bonds/loans satisfying contract criteria). The books emphasize the existence of a deliverable basket but do not fully enumerate all eligibility clauses in the retrieved excerpts |
| $P_i$ | Market price (per 100) of deliverable $i$ |
| $P_{\text{CTD}} = \min_{i \in \mathcal{D}} P_i$ | Cheapest-to-deliver price (per 100) |
| $\alpha$ | Accrual fraction between last premium payment date and credit event date (year fraction; day count depends on contract) |
| $s$ | CDS contractual spread (annualized, in decimal; e.g., 200 bp = 0.02) |

---

## Core Concepts

### 1.1 Credit default swap (CDS): premium leg vs protection leg (mechanics context only)

**Formal Definition:**
A CDS is a bilateral OTC contract in which the protection buyer pays a periodic premium (spread) and, if a credit event occurs, the protection seller makes a contingent protection leg payment designed to "make up to par" the value of a deliverable obligation.

**Intuition:**
Think "default insurance-like payoff," but contractual: it pays upon a contract-defined credit event rather than a purely economic notion.

**Trading/Risk Practice:**
Settlement mechanics become crucial at default/restructuring: which event triggers, how settlement is done, and what price is used determine the realized payout.

---

### 1.2 Credit event: contractual trigger vs "economic default"

**Formal Definition:**
In the cited CDS text, "credit event" is the legal term for the event that triggers payment of the protection leg; it is similar to default but not exactly the same as rating-agency "event of default."

**Intuition:**
"Economic default" is a real-world deterioration/failure; a credit event is what the contract recognizes as a trigger.

**Trading/Risk Practice:**
The difference matters for event risk: a CDS may trigger in situations not identical to a bond's default definition (the sources discuss "technical default" as a contractual-breadth issue).

---

### 1.3 Common credit event types: hard vs soft (as supported)

**Formal Definition (as supported by source table):**

The book lists commonly used credit events and labels them hard/soft:

| Credit Event | Type | Description |
|--------------|------|-------------|
| **Bankruptcy** | Hard | Insolvency / inability to pay debts (not relevant for sovereigns per table) |
| **Failure to pay** | Hard | Failure to make due payments (with grace period) |
| **Obligation acceleration** | Hard | Obligations become due earlier due to default/other; used mostly in some emerging market contracts |
| **Obligation default** | Hard | Obligations become due and payable prior to maturity; "hardly ever used" in the source table |
| **Repudiation/moratorium** | Hard | Authority rejects/challenges validity; used in emerging market sovereign CDS |
| **Restructuring** | Soft | Changes in debt obligations excluding changes not associated with credit deterioration |

The same source states restructuring is the only soft credit event in that taxonomy.

**Intuition:**
Hard events tend to collapse the capital structure toward a common distressed price; soft events can leave multiple obligations trading with a term structure of prices.

**Trading/Risk Practice:**
Soft events (restructuring) are where delivery option/CTD effects can be most pronounced because different deliverables can trade at meaningfully different prices.

---

### 1.4 Protection leg settlement: physical vs cash

**Formal Definition:**

The protection leg can be settled in two ways:

- **Physical settlement:** buyer delivers face value of deliverable obligations; seller pays face value (par) in cash simultaneously.
- **Cash settlement:** seller pays face value minus the recovery price of a specified reference obligation; recovery price is determined via dealer poll or auction process.

**Intuition:**
- Physical: "deliver a defaulted bond/loan and get par."
- Cash: "keep the bond/loan; get paid cash based on a market-derived price."

**Trading/Risk Practice:**

The choice affects:
- operational requirements (sourcing deliverables),
- exposure to CTD/delivery option,
- dependence on final price formation (auction/poll).

---

### 1.5 Deliverable obligations basket and the delivery option

**Formal Definition:**

Many bonds and loans may qualify as deliverable obligations; defining a basket lets one standard contract hedge multiple eligible obligations and improves liquidity.

A side-effect: the protection buyer can choose which deliverable to deliver in physical settlement—this is the **delivery option**.

**Intuition:**
"You (buyer) hold an option to pick the bond/loan that makes the protection payment most valuable to you."

**Trading/Risk Practice:**
The delivery option is a contractual embedded option that can widen CDS vs cash relationships (the source lists it as a driver of CDS-cash basis).

---

### 1.6 Cheapest-to-deliver (CTD) in CDS settlement

**Formal Definition:**

In physical settlement, the protection buyer will (economically) prefer to deliver the cheapest eligible deliverable (lowest market price per 100) because it maximizes the gap between par received and market value delivered.

The source text explicitly defines the buyer's advantage as the difference between the hedged security value and the cheapest deliverable in a post-event scenario.

**Intuition:**
CTD is the "worst" thing for the seller: the buyer delivers the asset that costs the least in the market but receives par.

**Trading/Risk Practice:**
Sellers monitor deliverable sets, potential CTD candidates, and contractual constraints (e.g., restructuring clauses that restrict deliverables).

---

### 1.7 Auction / dealer poll for cash settlement (only what sources state)

**Formal Definition (as supported):**

Cash settlement uses a recovery/final price determined by a dealer poll or auction process.

A separate risk-management text describes that (when cash settlement is used) a two-stage auction process is used to determine the mid-market value of the cheapest deliverable bond several days after the credit event.

**Intuition:**
The auction/poll is a way to arrive at a single settlement price for cash settlement when there are multiple deliverables and/or market dislocations.

**Trading/Risk Practice:**

Traders care about:
- which deliverables are "in play,"
- how the final price relates to deliverable bond prices,
- uncertainty around the final price until it is fixed.

> **I'm not sure** about the operational steps, roles, bidding rules, or Determinations Committee procedures for auctions from the excerpts retrieved here. To be certain, we would need the exact ISDA Definitions vintage and the specific auction settlement protocol referenced by the trade.

---

### 1.8 Triggering timeline and deliverable scarcity (physical settlement stress)

**Formal Definition (book-reported timeline):**

The CDS text describes a physical-settlement timeline with: Credit Event Notice, then Notice of Physical Settlement within 30 calendar days, then physical delivery within 30 business days; and notes up to ~72 calendar days after initial notification until payment on the protection leg is made.

**Intuition:**
Physical settlement is not "instant cash"; it has a process and timing.

**Trading/Risk Practice:**

If outstanding physically settled protection is large relative to deliverables, a short squeeze can occur: buyers without bonds rush to buy deliverables, pushing up prices and reducing the loss payment.

The source also notes there can be a mechanism to allow fallback to cash settlement if physical settlement is not possible/legal (without detailing the mechanism).

---

## Math and Derivations

### 2.1 Protection leg payoff under cash settlement

**Assumption:** Cash settlement uses a final price $P_{\text{final}}$ per 100 of face, representing the recovery price/value of the reference obligation (determined by dealer poll or auction).

**Payoff to protection buyer (cash settlement):**

$$\boxed{\Pi_{\text{cash}} = N\left(1 - \frac{P_{\text{final}}}{100}\right) = N(1 - R_{\text{settle}})}$$

**Unit check:**
- $N$ is dollars.
- $P_{\text{final}}/100$ is dimensionless.
- $\Pi_{\text{cash}}$ is dollars. ✓

**Sanity checks:**
- If $P_{\text{final}} = 100$: $\Pi_{\text{cash}} = 0$ (no loss). ✓
- If $P_{\text{final}} = 0$: $\Pi_{\text{cash}} = N$ (total loss on par). ✓

---

### 2.2 Protection leg payoff under physical settlement (economic value)

**Assumption (economic):** The buyer can deliver any eligible obligation $i \in \mathcal{D}$ at face value, receiving par in cash.

If the buyer delivers obligation $i$ with market price $P_i$ per 100, the delivered bond has market value $N \cdot P_i/100$.

The buyer receives par $N$ and delivers value $N \cdot P_i/100$.

**Economic payoff to protection buyer if delivering $i$:**

$$\Pi_{\text{phys}}(i) = N - N\frac{P_i}{100} = N\left(1 - \frac{P_i}{100}\right)$$

---

### 2.3 Delivery option and CTD: payoff is a maximum over deliverables

Because the buyer chooses which deliverable to deliver:

$$\boxed{\Pi_{\text{phys}} = \max_{i \in \mathcal{D}} \Pi_{\text{phys}}(i) = N\left(1 - \frac{\min_{i \in \mathcal{D}} P_i}{100}\right) = N\left(1 - \frac{P_{\text{CTD}}}{100}\right)}$$

This is the **delivery option**: the buyer is long an option on the set of deliverables.

---

### 2.4 Economic alignment: when should physical and cash settlement match?

If the cash settlement final price equals the market value of the deliverable that would be optimally delivered (i.e., $P_{\text{final}} = P_{\text{CTD}}$), then:

$$\Pi_{\text{cash}} = N\left(1 - \frac{P_{\text{final}}}{100}\right) = N\left(1 - \frac{P_{\text{CTD}}}{100}\right) = \Pi_{\text{phys}}$$

**Interpretation (reasoned):** Cash settlement is economically aligned with physical settlement when it uses a final price that reflects the relevant deliverable value (often discussed as "cheapest deliverable" in the sources' cash-settlement descriptions).

> **I'm not sure** whether a given market auction's "final price" is designed to match the CTD exactly or some broader deliverables measure in all cases; to be certain we need the specific auction protocol and definition set.

---

### 2.5 Accrued premium at default (mechanics-only reminder)

The sources note that following a credit event the protection buyer typically pays premium accrued since the previous premium payment date (a contingent cashflow) and that this is handled as part of the premium leg.

**Illustrative accrual formula (mechanics):**

$$\text{Accrued Premium} \approx N \cdot s \cdot \alpha$$

where:
- $s$ is the annual CDS spread (decimal),
- $\alpha$ is the year-fraction from last payment date to event date.

> **I'm not sure** of the exact day-count convention to apply for your specific contract without the documentation terms; the source text here highlights the existence of accrued premium at default but does not fully specify conventions in the excerpt.

---

## Measurement & Risk (only what belongs in Chapter 39)

### A) Credit events: "economic vs contractual trigger"

**Contractual trigger (source-backed):**
A credit event is the legal/contract term that triggers the protection leg payment, and it is similar to but not exactly the same as rating-agency default definitions.

**Economic deterioration (interpretation):**
A firm can be "in trouble" economically without a contract credit event occurring yet; conversely, a contract credit event may capture "technical" triggers broader than a bond's default definition (the sources explicitly discuss "technical default" as a CDS-vs-bond contractual breadth issue).

**Credit event categories:**
Supported list (hard/soft) is in the source table (Bankruptcy, Failure to Pay, Obligation Acceleration, Obligation Default, Repudiation/Moratorium, Restructuring).

**Documentation dependence warning:** The same CDS text warns that definitions evolve and market participants should seek up-to-date legal advice.

---

### B) Settlement choices: physical vs cash

**Physical settlement (source-backed):**
- Buyer delivers face value of a deliverable obligation; seller pays par cash.
- The CDS text describes a physical-settlement timeline: Credit Event Notice → Notice of Physical Settlement → Physical Settlement date (book-reported, referencing ISDA 2003).
- **Deliverable obligations definition:** The book emphasizes a basket of deliverable obligations but does not fully define all eligibility criteria in the excerpt. *I'm not sure of the complete deliverability rules without the precise ISDA Definitions and confirmation.*

**Cash settlement (source-backed):**
- Seller pays $\text{face} - \text{recovery price}$ in cash; recovery price is determined by dealer poll or auction.
- A risk-management text states that (when cash settled, "as is now usual" in that text) a two-stage auction determines the mid-market value of the cheapest deliverable bond.

**Why physical and cash should align (reasoned):**
If the cash final price equals the market value of the deliverable that would be delivered under physical settlement (notably CTD), then both settlement methods give the same economic protection payment (Section 2.4 derivation).

---

### C) Cheapest-to-deliver (CTD) in CDS settlement (this chapter's centerpiece)

**Why CTD matters for the protection seller (reasoned inference):**

Under physical settlement, the buyer can choose from $\mathcal{D}$. Because payoff increases as the delivered obligation's price decreases, the buyer chooses the lowest-priced eligible deliverable, maximizing payout.

Therefore, the seller's realized cost depends on the delivered obligation choice: **the seller is short the buyer's delivery option.**

**CTD definition in CDS terms:**

$$\text{CTD} := \arg\min_{i \in \mathcal{D}} P_i, \qquad P_{\text{CTD}} := \min_{i \in \mathcal{D}} P_i$$

**Economic payout under physical:** $\Pi_{\text{phys}} = N(1 - P_{\text{CTD}}/100)$.

**What moves CTD economics:**

1. **Deliverable price dispersion:** The delivery option only has value when deliverables trade at different prices after the credit event; the CDS text expects this mainly for a soft credit event like restructuring, since the debt can continue trading with a term structure of prices.

2. **Deliverability constraints:** The CDS text discusses restructuring clause variations that restrict deliverable obligations (e.g., Mod-Re, Mod-Mod-Re) and notes regional differences; these constraints affect the range of deliverables and thus CTD.

3. **Liquidity / scarcity:** If physical settlement demand overwhelms deliverable supply, a short squeeze can raise deliverable prices and reduce payout.

**Link CTD to auction (only as supported + careful reasoning):**

The sources connect cash settlement to auction/poll and explicitly mention "cheapest deliverable" as a key object in cash settlement price determination.

**Conceptual link (reasoned):** An auction that targets the mid-market value of the cheapest deliverable tends to internalize CTD economics into a single cash settlement price, reducing disputes/operational frictions relative to sourcing physical bonds.

> **I'm not sure** about the precise mapping from deliverable set to final price in all auction protocols (operational steps are not detailed in the retrieved excerpts).

**Optional analogy (very light):** In bond futures, "cheapest-to-deliver" arises because the short can choose among deliverable bonds; this is structurally analogous to the CDS buyer's delivery option, except the party holding the delivery option differs by product.

---

### D) Risks specific to credit events + settlement

**Event risk:**

*Source-backed aspects:*
- Credit event is a legal trigger and not exactly rating-agency default.
- The protection buyer generally triggers upon awareness, but may delay in some cases (e.g., expecting restructuring to turn into full default with larger payoff).

> **I'm not sure** about modern determination procedures (e.g., Determinations Committee rules) from these excerpts; need the relevant ISDA Definitions vintage and governance documents.

**Settlement risk:**
- **Final price uncertainty:** under cash settlement, payout depends on $P_{\text{final}}$ until it is fixed (auction/poll).
- **Deliverable scarcity / short squeeze risk:** physical settlement can create scarcity-driven price moves that change the economic payout.

**Recovery risk:**

Mechanically, payout sensitivity is:

$$\frac{\partial \Pi_{\text{cash}}}{\partial P_{\text{final}}} = -\frac{N}{100}$$

A risk-management text defines recovery rate (for a bond) as its value shortly after default as a percentage of face value.

**Basis risk preview (only):**

The CDS text defines the CDS-cash basis and lists contractual differences such as the delivery option and technical default as drivers.

**Preview takeaway:** even if CDS and bonds are "about the same credit," settlement mechanics and contract triggers can cause divergences.

---

### E) Keep it focused

- We do not build survival curves or par-spread pricing here.
- We only include accrued premium as a settlement-adjacent reminder (pricing details belong elsewhere).

---

## Worked Examples

### Common conventions for all examples unless stated

- Notional $N = \$10{,}000{,}000$
- Prices are quoted per 100 of face
- Protection payment (cash) uses: $\Pi_{\text{cash}} = N(1 - P_{\text{final}}/100)$

---

### Example A: Protection payout from final price

**Given:** $N = \$10\text{mm}$, $P_{\text{final}} = 38$.

**Step 1:** Convert price to fraction of par:
$$R_{\text{settle}} = \frac{38}{100} = 0.38$$

**Step 2:** Compute payout:
$$\Pi_{\text{cash}} = 10{,}000{,}000(1 - 0.38) = 10{,}000{,}000 \times 0.62 = 6{,}200{,}000$$

**Answer:** Protection buyer receives **\$6.2mm**.

**Unit check:** price is dimensionless after /100; payoff in USD. ✓

---

### Example B: Physical settlement economics and equivalence to Example A

**Assume:** A deliverable bond is trading at 38 per 100, and buyer can source it.

**Step 1:** Market value of $N$ face deliverable at 38:
$$\text{Cost to buy deliverable} = 10{,}000{,}000 \times \frac{38}{100} = 3{,}800{,}000$$

**Step 2:** Under physical settlement, buyer delivers $N$ face and receives par $N$ in cash:
$$\text{Cash received from seller} = 10{,}000{,}000$$

**Step 3:** Economic gain:
$$\Pi_{\text{phys}} = 10{,}000{,}000 - 3{,}800{,}000 = 6{,}200{,}000$$

**Answer:** **\$6.2mm**, same as Example A when $P_{\text{final}} = P_{\text{deliverable}} = 38$.

---

### Example C: Multiple deliverables → CTD option

**Deliverable set:** two eligible deliverables with prices:
- Deliverable 1: $P_1 = 35$
- Deliverable 2: $P_2 = 45$

#### Physical settlement (buyer chooses CTD)

**Step 1:** CTD price:
$$P_{\text{CTD}} = \min(35, 45) = 35$$

**Step 2:** Buyer's economic payoff:
$$\Pi_{\text{phys}} = 10{,}000{,}000\left(1 - \frac{35}{100}\right) = 10{,}000{,}000 \times 0.65 = 6{,}500{,}000$$

#### Cash settlement comparisons

- If cash settlement used 35: payout $= 6.5\text{mm}$ (matches CTD physical)
- If cash settlement used 45: payout $= 10{,}000{,}000(1 - 0.45) = 5{,}500{,}000$

**Takeaway:** Under physical settlement, buyer delivers the cheapest eligible obligation, increasing seller's cost.

---

### Example D: CTD value to buyer: scenario-based "option value"

Using Example C, define a toy "average deliverable price":
$$\bar{P} = \frac{35 + 45}{2} = 40$$

**Payoff if forced to settle at average price (toy cash benchmark):**
$$\Pi(\bar{P}) = 10{,}000{,}000\left(1 - \frac{40}{100}\right) = 10{,}000{,}000 \times 0.60 = 6{,}000{,}000$$

**Payoff with CTD delivery:**
$$\Pi(P_{\text{CTD}}) = 6{,}500{,}000$$

**"Option value" in this scenario:**
$$6{,}500{,}000 - 6{,}000{,}000 = 500{,}000$$

**Answer:** Scenario CTD uplift = **\$0.5mm**.

**Note:** This is not a universal pricing formula; it's a scenario comparison illustrating why the delivery option is valuable.

---

### Example E: Auction final price mapping: implied recovery and payout

**Given:** auction final price $P_{\text{final}} = 42$.

**Step 1:** Implied settlement recovery fraction:
$$R_{\text{settle}} = 0.42$$

**Step 2:** Payout:
$$\Pi_{\text{cash}} = 10{,}000{,}000(1 - 0.42) = 10{,}000{,}000 \times 0.58 = 5{,}800{,}000$$

**Compare to bond "recovery proxies" if bonds trade 40–44:**
- If 40: payout $= 10{,}000{,}000(1 - 0.40) = 6{,}000{,}000$
- If 44: payout $= 10{,}000{,}000(1 - 0.44) = 5{,}600{,}000$

**Takeaway:** A final price of 42 sits within that bond price band and gives payout **\$5.8mm**.

---

### Example F: Accrued premium reminder — simplified illustrative accrual

Sources state accrued premium is typically owed from buyer to seller after credit event (premium leg).

**Assume (illustrative):**
- Annual CDS spread $s = 200$ bp $= 0.02$
- Premium period is quarterly (0.25 years)
- Default occurs halfway through the quarter $\Rightarrow \alpha = 0.125$ years

**Accrued premium:**
$$\text{Accrued} \approx N \cdot s \cdot \alpha = 10{,}000{,}000 \times 0.02 \times 0.125$$

**Step-by-step:**
1. $10{,}000{,}000 \times 0.02 = 200{,}000$ (annual premium dollars)
2. $200{,}000 \times 0.125 = 25{,}000$

**Answer:** accrued premium $\approx$ **\$25,000** paid by buyer to seller.

> **I'm not sure** of the exact accrual day-count and stub handling for your trade without the contract terms.

---

### Example G: Cash vs physical under deliverable scarcity: toy bid/ask range

**Motivation:** deliverable scarcity/short squeeze can affect deliverable prices.

**Assume:** CTD deliverable is illiquid with post-event quotes:
- Bid 30, Ask 40 (per 100). Mid = 35.

**Compute payout range if settlement price could plausibly land anywhere in 30–40 (toy uncertainty):**

- If $P_{\text{final}} = 30$: payout $= 10{,}000{,}000(1 - 0.30) = 7{,}000{,}000$
- If $P_{\text{final}} = 40$: payout $= 10{,}000{,}000(1 - 0.40) = 6{,}000{,}000$

**Answer:** payout range **\$6.0mm to \$7.0mm** (range width \$1.0mm).

**Interpretation:** settlement/liquidity risk can be material at default, even if the contract notional is fixed.

---

### Example H: Deliverable set restriction sensitivity

Using Example C deliverables, now remove the cheapest deliverable from eligibility:
- Original eligible prices: 35 and 45 → CTD 35
- New eligible set: only 45 → CTD 45

**Physical settlement payout changes from:**

*Old:*
$$10{,}000{,}000(1 - 0.35) = 6{,}500{,}000$$

*New:*
$$10{,}000{,}000(1 - 0.45) = 5{,}500{,}000$$

**Answer:** payout decreases by:
$$6{,}500{,}000 - 5{,}500{,}000 = 1{,}000{,}000$$

i.e., **\$1.0mm**.

**Interpretation:** deliverability rules materially change CTD and economics.

---

### Example I: Seller vs buyer cashflows sign table at settlement

**Given:** $N = \$10\text{mm}$, final price 38.

We show contract settlement cashflows (ignoring premium accrual here).

| Settlement type | Protection buyer (long protection) | Protection seller (short protection) |
|-----------------|-----------------------------------|-------------------------------------|
| **Physical settlement** | Receives +\$10.0mm cash; delivers \$10.0mm face of eligible obligations | Pays -\$10.0mm cash; receives \$10.0mm face obligations |
| **Cash settlement** | Receives +\$6.2mm cash (since $1 - 0.38 = 0.62$) | Pays -\$6.2mm cash |

**Economic equivalence view (optional):** Under physical, if delivered obligations are worth \$3.8mm in the market, seller's economic cost is \$10.0mm − \$3.8mm = \$6.2mm, aligning with cash settlement when $P_{\text{final}} = 38$.

---

### Example J: Timing uncertainty toy

**Assume:** default could occur:
- Early with $P_{\text{final}} = 30$ (more distressed outcome)
- Late with $P_{\text{final}} = 45$ (less distressed outcome)

**Compute payouts:**

*Early payout:*
$$10{,}000{,}000(1 - 0.30) = 7{,}000{,}000$$

*Late payout:*
$$10{,}000{,}000(1 - 0.45) = 5{,}500{,}000$$

Now assume probabilities (toy): 50% early, 50% late.

**Expected payout:**
$$E[\Pi] = 0.5(7.0\text{mm}) + 0.5(5.5\text{mm}) = 3.5\text{mm} + 2.75\text{mm} = 6.25\text{mm}$$

**Answer:** Expected payout = **\$6.25mm** under these toy assumptions.

**Interpretation:** event timing uncertainty matters because the settlement price/recovery can differ across scenarios.

---

### Example K: Portfolio settlement exposure additivity

**Portfolio:** 3 CDS positions on three names:
- Name 1: $N_1 = \$5\text{mm}$
- Name 2: $N_2 = \$10\text{mm}$
- Name 3: $N_3 = \$15\text{mm}$

#### Case 1: common final price $P_{\text{final}} = 38$ for all

Payout per \$1 notional is $0.62$.

- Name 1: $5\text{mm} \times 0.62 = 3.10\text{mm}$
- Name 2: $10\text{mm} \times 0.62 = 6.20\text{mm}$
- Name 3: $15\text{mm} \times 0.62 = 9.30\text{mm}$

**Total:**
$$3.10 + 6.20 + 9.30 = 18.60 \text{ mm}$$

**Answer:** Total payout = **\$18.6mm**.

#### Case 2: name-specific final prices

Assume:
- Name 1: $P_{\text{final}} = 20$ → payout fraction $0.80$ → $5\text{mm} \times 0.80 = 4.00\text{mm}$
- Name 2: $P_{\text{final}} = 38$ → $0.62$ → $10\text{mm} \times 0.62 = 6.20\text{mm}$
- Name 3: $P_{\text{final}} = 60$ → $0.40$ → $15\text{mm} \times 0.40 = 6.00\text{mm}$

**Total:**
$$4.00 + 6.20 + 6.00 = 16.20 \text{ mm}$$

**Answer:** Total payout = **\$16.2mm**.

**Interpretation:** payouts add linearly in notional, but concentration arises in names with low final prices.

---

### Example L: Cheapest-to-deliver vs "single final price" contrast

Use Example C deliverables: 35 and 45.

**Physical settlement payout uses CTD = 35:**
$$\Pi_{\text{phys}} = 10{,}000{,}000(1 - 0.35) = 6{,}500{,}000$$

**Now compare with hypothetical cash settlement using different candidate prices:**

| Final Price Used | Payout |
|------------------|--------|
| 35 | $\Pi = 6.5\text{mm}$ (matches physical) |
| 45 | $\Pi = 5.5\text{mm}$ |
| Average 40 | $\Pi = 6.0\text{mm}$ |

**Interpretation (conceptual + sourced anchor):**

The risk-management source states cash settlement auction aims to determine the mid-market value of the cheapest deliverable bond.

Conceptually, that choice aligns cash settlement with CTD economics and reduces the "which deliverable?" variability inherent in physical settlement.

> **I'm not sure** about the detailed auction mechanics that ensure this alignment in every protocol; we would need the precise auction documentation.

---

## Practical Notes

### Mechanics checklist: what must be known/checked on a real trade

1. Reference entity and (if applicable) reference obligation(s).
2. Documentation set / definitions vintage (ISDA Definitions; the CDS text warns definitions evolve).
3. Credit event definitions included in the contract (which events trigger; whether restructuring included, etc.).
   - *Example of variability:* sources note restructuring may be excluded in some contexts/contracts.
4. Settlement method: physical vs cash (and any fallback language).
5. **If physical:**
   - Deliverable obligations criteria / deliverable set $\mathcal{D}$ and any constraints (e.g., restructuring clause restrictions).
   - Operational readiness to source and deliver.
6. **If cash:**
   - How $P_{\text{final}}$ is determined (dealer poll or auction; two-stage auction mentioned in one source).
7. Notional $N$, currency, and confirmation details.
8. Timeline / notices (book-reported process exists; check your contract for actual deadlines).
9. Accrued premium at default handling.

---

### Common pitfalls

1. **Confusing "recovery" as a structural constant** with the settlement final price (a market-determined number around the event).

2. **Assuming CTD is irrelevant under cash settlement:**
   - Sources explicitly connect cash settlement (auction) to the value of the cheapest deliverable.

3. **Forgetting accrued premium at default** (premium paid in arrears implies an accrual is typically owed).

4. **Mixing up price-per-100 with fraction form:**
   - 38 (per 100) vs 0.38 (fraction) → huge payout errors.

5. **Assuming auction operational details without sourcing:**
   - *I'm not sure beyond "dealer poll or auction" / "two-stage auction" from these excerpts.*

---

### Implementation pitfalls (risk systems / P&L explain)

1. **Date handling:**
   - event date vs settlement date vs accrual start; avoid off-by-one and stub issues.

2. **Converting final price to payout:**
   - Ensure consistent use of $P_{\text{final}}/100$.

3. **Sign conventions:**
   - Buyer receives $+\Pi$; seller pays $-\Pi$.

4. **Physical settlement economics:**
   - Model the buyer's right to choose CTD: use $\min P_i$ over eligible deliverables.

---

### Verification tests

1. **Payout bounds** (assuming $0 \le P_{\text{final}} \le 100$):
   $$0 \le \Pi_{\text{cash}} \le N$$

2. **Physical/cash alignment sanity:**
   - If $P_{\text{final}} = P_{\text{CTD}}$, then $\Pi_{\text{cash}} = \Pi_{\text{phys}}$.

3. **CTD logic test:**
   - Under physical settlement, buyer chooses lowest-priced eligible deliverable (max payout).

---

## Summary & Recall

### 10-Bullet Executive Summary

1. A CDS protection payment is triggered by a contract-defined credit event, not merely "economic distress."
2. The sources list common credit events and classify them as hard vs soft; restructuring is the only "soft" one in that table.
3. The protection leg can settle via physical or cash settlement.
4. Physical settlement exchanges deliverable obligations (face) for par cash.
5. Cash settlement pays $N(1 - P_{\text{final}}/100)$ using a final price determined via dealer poll or auction.
6. Cash settlement is described as using an auction to determine the mid-market value of the cheapest deliverable bond (in one source), linking auctions to CTD economics.
7. A basket of deliverables improves hedging flexibility and liquidity but gives the buyer a delivery option.
8. The delivery option means the buyer will deliver the CTD (lowest price), which is adverse for the seller.
9. Deliverable scarcity can cause short squeezes that move distressed prices and change payouts.
10. Always verify the exact contract's credit event definitions, deliverability rules, and settlement protocol; the sources warn definitions evolve.

---

### Cheat Sheet: Settlement Payoffs, CTD Concept, Sign Table, Key Sanity Checks

**Cash settlement payoff (buyer):**
$$\boxed{\Pi_{\text{cash}} = N\left(1 - \frac{P_{\text{final}}}{100}\right)}$$

**Physical settlement payoff (buyer, delivering $i$):**
$$\Pi_{\text{phys}}(i) = N\left(1 - \frac{P_i}{100}\right)$$

**Physical settlement payoff with delivery option:**
$$\boxed{\Pi_{\text{phys}} = N\left(1 - \frac{P_{\text{CTD}}}{100}\right), \qquad P_{\text{CTD}} = \min_{i \in \mathcal{D}} P_i}$$

**Buyer vs seller sign (cash settlement):**
| Party | Cashflow |
|-------|----------|
| Buyer | $+\Pi_{\text{cash}}$ |
| Seller | $-\Pi_{\text{cash}}$ |

**Sanity checks:**
- $0 \le \Pi \le N$ when $0 \le P \le 100$
- If $P_{\text{final}} = P_{\text{CTD}}$, cash and physical align

---

### 35 Flashcards (Q/A)

1. **Q:** What is a "credit event" in CDS terms?
   **A:** A contract-defined legal trigger for protection leg payment; similar to default but not identical to rating-agency default.

2. **Q:** Name two settlement methods for the protection leg.
   **A:** Physical settlement and cash settlement.

3. **Q:** Physical settlement: who delivers what?
   **A:** Buyer delivers face amount of deliverable obligations; seller pays par cash.

4. **Q:** Cash settlement payoff form?
   **A:** $N(1 - P_{\text{final}}/100)$.

5. **Q:** What determines $P_{\text{final}}$ in cash settlement (as supported)?
   **A:** Dealer poll or auction process.

6. **Q:** What does one source say the auction targets?
   **A:** Mid-market value of the cheapest deliverable bond.

7. **Q:** What is CTD in CDS physical settlement?
   **A:** The lowest-priced eligible deliverable obligation.

8. **Q:** Why is CTD bad for the protection seller?
   **A:** Buyer chooses the deliverable that maximizes payout (lowest price), increasing seller's economic cost.

9. **Q:** What is the "delivery option"?
   **A:** Buyer's right to choose among deliverable obligations; embedded optionality.

10. **Q:** When does the delivery option have value (source intuition)?
    **A:** When deliverables trade at different prices after a credit event.

11. **Q:** Which event is "soft" in the source table?
    **A:** Restructuring.

12. **Q:** What happens to debt pricing after restructuring (source)?
    **A:** Debt can keep trading with a term structure; short-dated higher than long-dated.

13. **Q:** Give one hard credit event from the source table.
    **A:** Bankruptcy / failure to pay / obligation acceleration / obligation default / repudiation-moratorium.

14. **Q:** Why do deliverable baskets exist?
    **A:** To hedge multiple eligible bonds/loans with one standard contract and enhance liquidity.

15. **Q:** What is a key operational risk of physical settlement?
    **A:** Sourcing deliverable obligations; scarcity can occur.

16. **Q:** What is a "short squeeze" in this context (source)?
    **A:** Buyers rush to buy deliverables to settle, pushing prices up and reducing loss payment.

17. **Q:** Does premium accrue at default?
    **A:** Yes; the standard contract includes premium accrued since last payment date (source mentions this feature).

18. **Q:** Is accrued premium part of the protection leg?
    **A:** No; it is handled as part of the premium leg in the cited CDS text.

19. **Q:** If $P_{\text{final}} = 100$, what is payout?
    **A:** 0.

20. **Q:** If $P_{\text{final}} = 0$, what is payout?
    **A:** $N$.

21. **Q:** Convert price 38 per 100 to fraction.
    **A:** 0.38.

22. **Q:** For $N = \$10\text{mm}$ and price 38, payout?
    **A:** \$6.2mm.

23. **Q:** Under physical settlement, what does the buyer optimally deliver?
    **A:** The cheapest eligible deliverable (CTD).

24. **Q:** What makes CTD economics "move"?
    **A:** Deliverable price dispersion, deliverability constraints, liquidity/scarcity.

25. **Q:** What documentation element can restrict deliverables after restructuring (source mentions)?
    **A:** Restructuring clause choices (e.g., Mod-Re / Mod-Mod-Re).

26. **Q:** Why might a protection buyer delay triggering (source)?
    **A:** Expect restructuring to turn into full default producing larger payoff.

27. **Q:** What is "technical default" in basis discussion (source)?
    **A:** Contractual credit events may be broader than bond default.

28. **Q:** How does delivery option affect CDS vs bond (basis intuition, source)?
    **A:** Makes long protection CDS more valuable; can widen CDS spread vs bond spread.

29. **Q:** What is the cash settlement reference price intended to represent?
    **A:** Market value (recovery price) of the reference/cheapest deliverable bond via poll/auction (as stated in sources).

30. **Q:** Key sanity check linking cash and physical?
    **A:** If cash final price equals CTD price, payoffs match.

31. **Q:** What is "deliverable obligation" precisely?
    **A:** I'm not sure from excerpts; it is contract-defined by ISDA definitions/confirmation.

32. **Q:** Do auction details vary by protocol?
    **A:** Yes; I'm not sure of details without the specific protocol text.

33. **Q:** Why do auctions help markets at default?
    **A:** Provide a single price for cash settlement; reduce need to source bonds (conceptually; sources anchor to auction-determined cheapest deliverable value).

34. **Q:** What must be verified on a real trade?
    **A:** Definitions vintage, credit events included, settlement method, deliverables, notional, accrual conventions.

35. **Q:** What final warning does the CDS text provide?
    **A:** Definitions evolve; seek up-to-date professional legal advice.

---

## Mini Problem Set (18 questions)

1. A CDS has $N = \$25\text{mm}$ and auction final price $P_{\text{final}} = 12$. Compute cash settlement payout.

2. For $N = \$10\text{mm}$, compare payout if final price is 38 vs 42. What is the payout difference?

3. Two deliverables are eligible with prices 55 and 60. Under physical settlement, what is CTD and payout?

4. Three deliverables priced 20, 35, 50. Compute physical settlement payout and compare to cash payout if $P_{\text{final}} = 35$.

5. A contract's deliverable set changes so the 20-priced bond is no longer eligible. Recompute payout from Q4 and compute the change.

6. If a bond trades at 40–44 around the event and the auction final price is 42, compute payout range implied by the bond band and locate the auction payout within it.

7. Explain (in words) why a soft credit event can create larger delivery option value than a hard credit event (use the source's "term structure of prices" idea).

8. Suppose $N = \$10\text{mm}$, spread $s = 500$ bp, and default occurs 30 days into a 90-day accrual period. Using simple ACT/360-like reasoning, approximate accrued premium. (If you need day count details, state assumptions.)

9. Build a sign table of settlement cashflows between buyer and seller under physical vs cash settlement (no numbers).

10. Consider an illiquid CTD that can trade from 25 to 45. For $N = \$50\text{mm}$, compute the payout range.

11. Show that physical settlement payoff equals cash settlement payoff if $P_{\text{final}} = P_{\text{CTD}}$.

12. Describe two settlement risks unique to physical settlement and two unique to cash settlement.

13. Using the source's "technical default" idea, discuss how contractual triggers can create basis risk between bonds and CDS.

14. You hedge a specific bond priced 48 with CDS where CTD is 40 after a restructuring credit event. Compute the incremental gain from the delivery option (per 100 face) and then for $N = \$10\text{mm}$.

15. Give a toy scenario where delaying the trigger could increase payoff, and compute the payoff difference (two final prices).

16. For a portfolio of notionals $N_1, N_2, \ldots, N_k$, show the total cash settlement payout formula under a common final price.

17. Discuss why an auction aimed at "mid-market value of the cheapest deliverable bond" conceptually aligns with CTD economics.

18. List the minimum contract fields you would demand before modeling settlement outcomes in a risk system.

---

### Brief Solution Sketches (1–9 only)

1. $\Pi = 25\text{mm}(1 - 0.12) = 25\text{mm} \times 0.88 = 22\text{mm}$.

2. $10\text{mm}[(1 - 0.38) - (1 - 0.42)] = 10\text{mm}(0.62 - 0.58) = 0.4\text{mm}$.

3. CTD = 55. Payout $= 10\text{mm}(1 - 0.55) = 4.5\text{mm}$.

4. Physical uses CTD=20 → $10\text{mm}(0.80) = 8\text{mm}$. Cash at 35 → $10\text{mm}(0.65) = 6.5\text{mm}$.

5. Remove 20 → CTD=35 → physical payout becomes 6.5mm; change = $-1.5\text{mm}$ vs 8mm.

6. Bond 40 → payout 6.0mm; bond 44 → 5.6mm; auction 42 → 5.8mm (in the middle).

7. Soft event → different obligations can keep trading at different prices/term structure; buyer can choose cheapest, increasing option value.

8. Annual premium $= Ns$. Accrual fraction $= 30/360 = 1/12$ if ACT/360-like. Accrued $\approx Ns/12$. State assumptions.

9. Physical: buyer delivers obligations, receives par; seller opposite. Cash: seller pays cash amount $N(1 - P_{\text{final}}/100)$ to buyer; buyer opposite.
