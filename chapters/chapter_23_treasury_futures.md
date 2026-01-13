# Chapter 23: Treasury Futures — CTD, Conversion Factor, Delivery Options, and the Link to Repo/Funding

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Treasury futures specify a delivery month and a basket of eligible Treasury securities; the short chooses which bond to deliver (quality option) and when to deliver within the permitted window (timing option). [Tuckman Ch 19-20, Hull Ch 6]
- Invoice price formula: $\text{Invoice}_i(t) = cf_i \cdot F(t) + AI_i(t)$. [Tuckman Ch 20]
- Conversion factor is computed as the bond's value when discounted at 6% per annum with semiannual compounding, with contract-specific rounding. [Hull Ch 6]
- Cost of delivery: $\text{CostDel}_i(t) = P_i(t) - cf_i \cdot F(t)$. [Tuckman Eq 20.1]
- CTD is the bond that minimizes delivery cost. [Tuckman Ch 20, Hull Ch 6]
- Forward price under repo (no coupons before $T$): $P_{\text{fwd}} + AI(T) = (P(0) + AI(0))(1 + rd/360)$. [Tuckman Eq 16.7]
- Carry decomposition: $P_{\text{fwd}} = P(0) - \text{Carry}$. [Tuckman Eq 16.8]
- DV01 definition: $\text{DV01} = -\frac{dP}{dy} \times \frac{1}{10000}$. [Tuckman Ch 5-6]
- Hedge ratio formula: $\text{Hedge ratio} = \frac{\text{DV01 of exposure}}{\text{DV01 of hedge}}$. [Tuckman Ch 6-7]
- Special spread defined as overnight GC rate minus overnight special rate. [Tuckman Ch 15-16]
- End-of-month option arises when last trade date precedes last delivery date. [Tuckman Ch 20]
- Wild card play arises from differences between settlement time and later spot trading. [Hull Ch 6]
- Hull notes exact theoretical futures price is difficult because the short's delivery options cannot be easily valued in closed form. [Hull Ch 6]

### (B) Reasoned Inference (Derived from A)

- Net basis formula: $\text{NB}_i(t) = P_{\text{fwd},i}(t) - cf_i \cdot F(t) = \text{GB}_i(t) - \text{Carry}_i(t \to T)$. [Derived from Tuckman's forward/carry relationship]
- Implied repo rate formula: $r_{\text{imp}} = \left(\frac{cf \cdot F + AI(T)}{P(0) + AI(0)} - 1\right) \frac{360}{d}$. [Derived from break-even cash-and-carry condition]
- Futures DV01 approximation: $\text{DV01}_{\text{fut, per contract}} \approx \frac{N}{100} \cdot \frac{\text{DV01}_{\text{CTD per \$100}}}{cf_{\text{CTD}}}$. [Derived from $F \approx P/cf$ relationship]
- Accrued interest cancels in cost-of-delivery because it appears in both purchase cost and invoice received. [Algebraic derivation]

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure about the exact fails-charge formula — the sources discuss fails conceptually but don't specify penalty mechanics.
- I'm not sure about the precise delivery notice deadlines, settlement timestamps, and their current form for specific contracts (2Y/5Y/10Y/Ultra) without the exact exchange rulebook version.
- I'm not sure about repo day-count conventions outside U.S. Treasury repo without additional source verification.
- I'm not sure how to translate Tuckman's specialness bounds into a precise operational "specialness bound" for specific contracts and settlement conventions.

---

## Conventions & Notation

### Notation Glossary

| Symbol | Meaning | Units |
|--------|---------|-------|
| $t$ | Valuation date (today) | — |
| $T$ | Delivery date (or last delivery date) | — |
| $d$ | Number of days from $t$ to delivery/expiry | days |
| $F(t)$ | Futures settlement price at time $t$ | \$/\$100 notional |
| $P_i(t)$ | Clean (flat) price of deliverable bond $i$ at time $t$ | \$/\$100 face |
| $AI_i(t)$ | Accrued interest of bond $i$ at time $t$ | \$/\$100 face |
| $cf_i$ | Conversion factor for bond $i$ | dimensionless |
| $\text{Invoice}_i(t)$ | Invoice (dirty) price on delivery of bond $i$ | \$/\$100 face |
| $r$ | Repo financing rate (annualized) | decimal |
| $\text{CostDel}_i(t)$ | Cost of delivery for bond $i$ at time $t$ | \$/\$100 face |
| $\text{GB}_i(t)$ | Gross basis of bond $i$ vs futures | \$/\$100 face |
| $\text{NB}_i(t)$ | Net basis (carry-adjusted basis) | \$/\$100 face |
| $P_{\text{fwd},i}(t)$ | Forward clean price for delivery of bond $i$ at $T$ | \$/\$100 face |
| $\text{Carry}_i(t \to T)$ | Carry from holding/financing bond $i$ from $t$ to $T$ | \$/\$100 face |
| DV01 | Dollar value of a 1 bp yield move | \$/bp |
| $N$ | Contract face amount | \$ |

### Defaults Used in Examples

- Prices are per \$100 face
- Repo interest uses simple interest $rd/360$ (actual/360 style)
- Futures contract size $N = \$100{,}000$ face (so \$ per contract $=$ price-per-100 $\times 1000$)
- Invoice price formula: $\text{Invoice} = cf \times F + AI$
- Clean vs dirty (cash) price: quoted (clean) price excludes accrued interest; cash (dirty) price includes accrued interest
- Unless stated otherwise, assume no coupon payment occurs before delivery

---

## Core Concepts

### 1) Treasury Futures as a Deliverable-Basket Contract

**Formal Definition:**
A Treasury futures contract specifies a delivery month and a basket of eligible Treasury securities. At delivery, the short satisfies the contract by delivering one eligible bond (or note) of face amount specified by the contract, receiving an invoice price based on the futures settlement price and the delivered bond's conversion factor (plus accrued interest). The short can choose which eligible bond to deliver (quality option) and can choose when to deliver within the permitted window (timing option).

**Intuition:**
Think of Treasury futures as:
1. A forward-like commitment to exchange a Treasury security in the future, and
2. An embedded set of options held by the short (choose the delivery bond; choose the delivery timing; possibly additional timing-related "end-of-month / wild card" features depending on contract rules).

Those embedded options mean the futures is not a simple forward on a single bond. Hull explicitly notes that an exact theoretical futures price is difficult because the short's delivery options cannot be easily valued in closed form.

**Trading/Risk Practice:**
- **Relative value:** Desks compare cash Treasuries vs futures via basis and implied financing
- **Hedging:** The futures hedge is effectively a hedge against the CTD (cheapest-to-deliver) bond and can jump if the CTD changes
- **Funding matters:** Repo rates (GC vs special) directly influence carry and therefore the basis/trade economics

---

### 2) Conversion Factor (CF)

**Formal Definition:**
The conversion factor $cf_i$ is a fixed number assigned to each deliverable bond $i$. It scales the futures settlement price into a delivery price for that bond:

$$\text{DeliveryPrice}_i(t) = cf_i \cdot F(t)$$

and invoice price adds accrued interest.

In Hull's description (Treasury bond futures context), the conversion factor is computed as the bond's value when discounted at 6% per annum with semiannual compounding, after applying contract-specific rounding/assumptions about time-to-maturity and possibly subtracting accrued interest.

In Tuckman's framing, conversion factors are tied to a notional coupon (historically 6%): conversion factors approximately equal the bond's price per \$1 face if yields were at the notional coupon; they would adjust prices perfectly if the term structure were flat at that notional rate.

**Intuition:**
Bonds in the delivery basket differ in coupon and maturity. If the futures had to specify a single fixed price-to-deliver, high-coupon and low-coupon bonds would be treated unfairly. The conversion factor is a standardization device: it tries to put each deliverable on a comparable footing relative to a "standard" notional bond (6% in the cited sources).

**What CF is NOT:**
- Not a duration, DV01, or hedge ratio by itself
- Not a market price — it is (largely) a contract parameter, not something that moves with yields day-to-day

**Trading/Risk Practice:**
- CF enters invoice price and CTD economics directly
- CF enters the futures DV01 approximation via the CTD (details in Section 3)

---

### 3) Invoice Price Mechanics

**Formal Definition:**
If $F(t)$ is the futures settlement price at time $t$ and $cf_i$ is the conversion factor of delivered bond $i$, then:

- **Delivery (flat) price:** $cf_i \cdot F(t)$
- **Invoice price (cash price):**

$$\boxed{\text{Invoice}_i(t) = cf_i \cdot F(t) + AI_i(t)}$$

This is stated explicitly by Tuckman and aligns with Hull's description that the short receives (settlement $\times$ CF) plus accrued interest.

**Intuition:**
Futures prices are quoted like clean prices, but the delivered bond is exchanged at a cash price that must compensate the seller for accrued coupon interest. So accrued interest is added (under the contracts described in the sources).

**Trading/Risk Practice:**
- Operationally, invoice price determines the cash flow on delivery
- Common desk mistake: mixing clean vs dirty and forgetting to include accrued interest

---

### 4) Cheapest-to-Deliver (CTD) and the Delivery-Cost Criterion

**Formal Definition:**
For deliverable bond $i$, Tuckman defines the cost of delivery at time $t$ as:

$$\boxed{\text{CostDel}_i(t) = (P_i(t) + AI_i(t)) - (cf_i \cdot F(t) + AI_i(t)) = P_i(t) - cf_i \cdot F(t)}$$

(Accrued interest cancels.)

The CTD is the bond that minimizes this cost (equivalently maximizes the delivery profit for the short).

Hull expresses the same decision rule: since the short receives (settlement $\times cf$) + AI and pays (quoted + AI), the CTD minimizes:
$$\text{Quoted bond price} - (\text{settlement} \times cf)$$

**Intuition:**
The short has an embedded option: among all eligible bonds, deliver the one that is cheapest relative to the contract's conversion-factor adjustment.

**Why CTD Can Change:**
- **Yield level and curve shape:** Hull notes systematic tendencies: when yields are above 6%, low-coupon/long-maturity bonds tend to be favored; when yields are below 6%, high-coupon/shorter bonds tend to be favored; curve slope also matters
- **Repo/funding:** If a bond can be financed "special" (below GC), its carry improves, which can make it effectively cheaper to own/deliver

**Trading/Risk Practice:**
- CTD is central for basis trading, hedging, and risk reports
- CTD switching risk is a key nonlinearity in futures hedges

---

### 5) Basis and Net Basis

**Formal Definition:**

**Gross basis (clean basis)** for bond $i$ at time $t$:

$$\boxed{\text{GB}_i(t) = P_i(t) - cf_i \cdot F(t)}$$

This is exactly the delivery-cost expression and is how Tuckman frames "cost of delivery."

**Net basis** uses the forward price $P_{\text{fwd},i}(t)$ (clean forward price for delivery at $T$), giving a carry-adjusted measure:

$$\boxed{\text{NB}_i(t) = P_{\text{fwd},i}(t) - cf_i \cdot F(t)}$$

Tuckman links net basis to basis trades and explicitly interprets net basis as the (lock-in) value of the quality option (under proper tailing).

Substituting $P_{\text{fwd},i}(t) = P_i(t) - \text{Carry}_i(t \to T)$:

$$\text{NB}_i(t) = \underbrace{(P_i(t) - cf_i \cdot F(t))}_{\text{gross basis}} - \text{Carry}_i(t \to T)$$

**Intuition:**
- **Gross basis:** "Is the cash bond rich or cheap vs futures today, after CF adjustment?"
- **Net basis:** "Is it rich/cheap after funding/carry to delivery?"

Net basis is the economically relevant quantity for cash-and-carry vs reverse cash-and-carry style arguments.

**Trading/Risk Practice:**
- Basis traders often focus on implied financing or net basis rather than gross basis
- Net basis is also a practical proxy for delivery-option value (quality option) when comparing deliverables

---

### 6) Delivery Options: Quality, Timing, and Timing-Related Features

**Quality Option (Choose the Bond):**
- **Definition:** Short can deliver any eligible bond in the basket; the short will choose the one that minimizes delivery cost (CTD)
- **Practice:** CTD tracking; "what is CTD?" is a daily relative-value question

**Timing Option (Choose the Date):**
- **Definition:** Short can choose delivery date during the delivery month/window; Tuckman explicitly describes a timing option and emphasizes the trade-off between carry and option value in deciding when to deliver
- **Practice:** Delivery is usually concentrated but not forced on a single date; timing interacts with repo/carry

**End-of-Month Option (Tuckman) / Wild Card Play (Hull):**

- **End-of-month option (Tuckman):** U.S. futures can have a last trade date before the last delivery date; deliveries after last trade date are priced using the final settlement price from the last trade date, giving the short an additional timing-related option. Tuckman points to this feature explicitly for TYH2.

- **Wild card play (Hull):** Hull describes a contract feature (Treasury bond futures) where the settlement price is determined at a particular time (2:00 p.m. Chicago time in his description), but spot trading continues later; the short can choose to issue a delivery notice later the same day. This creates an option-like benefit to the short when bond prices move after the settlement time.

**Important contract-rule caveat:** These timing mechanics are contract-specific. I'm not sure the precise delivery notice deadlines, settlement timestamps, and their current form for your contract without the exact exchange rulebook version and the contract (e.g., 2Y/5Y/10Y/Ultra).

---

### 7) Link to Repo/Funding: Carry, Specials, and Why Financing Affects Basis/CTD

**Formal Definition:**

- **Repo financing:** Holding cash Treasuries is typically financed via repo; repo rates may be general collateral (GC) or special for particular issues. Tuckman describes GC vs special and why on-the-run Treasuries trade special due to liquidity-driven supply/demand in borrowing those securities.

- **Special spread:** Tuckman defines special spread as overnight GC rate minus the overnight special rate for the issue (for on-the-run securities).

**Intuition:**
If a bond finances "special" (low repo rate), it is cheaper to carry; that can make it more attractive as a deliverable (affecting CTD) and can affect futures-vs-cash basis.

Net basis and implied repo summarize "how good is the futures price relative to owning/financing the cash bond."

**Trading/Risk Practice:**
- **Basis trades:** Long cash financed in repo + short futures; profitability depends on funding and the value of delivery options
- **CTD drivers include repo:** A bond that is slightly expensive in price terms can still be CTD if its financing advantage dominates

---

### 8) Hedging Implications: Futures DV01 Depends on CTD and Can Jump

**Formal Definition:**

Tuckman defines DV01 as $-dP/dy$ for a 1 bp move (scaled appropriately), and uses DV01 to compute hedge ratios:

$$\text{Hedge ratio (units of hedge instrument)} = \frac{\text{DV01 of exposure}}{\text{DV01 of hedge}}$$

**Intuition:**
A Treasury futures contract behaves (to first order) like a position in the CTD scaled by conversion factor. Therefore the futures' DV01 is approximately CTD DV01 divided by the conversion factor (then scaled by contract notional).

If CTD switches, the futures' effective DV01 changes abruptly $\Rightarrow$ hedge ratios jump.

**Trading/Risk Practice:**
- Hedgers must monitor CTD and sometimes hedge with "key rate" or multiple futures to reduce CTD-switch sensitivity (basis risk)

---

## Math and Derivations

### 2.1 Invoice Price and Cost of Delivery

**Source-backed starting point.** Tuckman gives:
- Delivery price: $cf_i \cdot F(t)$
- Invoice price: $cf_i \cdot F(t) + AI_i(t)$

If the short chooses to deliver bond $i$, the short:
1. Buys the bond in the market at dirty price $P_i(t) + AI_i(t)$
2. Delivers it into the futures and receives invoice $cf_i \cdot F(t) + AI_i(t)$

Therefore the cost of delivery is:

$$\text{CostDel}_i(t) = (P_i(t) + AI_i(t)) - (cf_i \cdot F(t) + AI_i(t)) = P_i(t) - cf_i \cdot F(t)$$

This is exactly Tuckman's equation (20.1).

**Unit Check:**
- $P_i(t)$: \$ per \$100 face
- $F(t)$: \$ per \$100 notional
- $cf_i$: dimensionless
- $\text{CostDel}_i(t)$: \$ per \$100 face

Accrued interest cancels (as it must, because both buying and invoicing include it).

---

### 2.2 CTD and the Final Settlement Price (Delivery-Date Logic)

If timing options are ignored, Tuckman states the final settlement price satisfies:

$$\boxed{F(T) = \min_i \frac{P_i(T)}{cf_i}}$$

and the CTD is the bond that achieves this minimum ratio.

**Why this makes sense (arbitrage / optimal-delivery reasoning):**
- At $T$, for any deliverable $i$, cost of delivery is $P_i(T) - cf_i \cdot F(T)$
- The short will choose the bond with minimal cost, so the realized cost is $\min_i \{P_i(T) - cf_i \cdot F(T)\}$
- In a competitive market (and in Tuckman's simplifying setting), the CTD cost at final settlement is 0; this pins down $F(T)$ to the minimum price-to-factor ratio

**Sanity Check:**
If $F(T)$ were higher than $\min_i P_i(T)/cf_i$, then for the minimizing bond $j$, $P_j(T) - cf_j \cdot F(T) < 0$, meaning delivering $j$ yields a negative cost (profit) at delivery; that would be inconsistent with the idea of how final settlement is determined in the simplified setting.

---

### 2.3 Forward Price, Carry, and Net Basis

Tuckman derives a coupon-bond forward price under repo financing (no coupons during the forward term):

$$\boxed{P_{\text{fwd}} + AI(d) = (P(0) + AI(0))(1 + rd/360)} \tag{16.7}$$

and equivalently:

$$P_{\text{fwd}} = (P(0) + AI(0))(1 + rd/360) - AI(d)$$

$$= P(0) - \underbrace{[AI(d) - AI(0) - (P(0) + AI(0)) \cdot rd/360]}_{\text{Carry}}$$

$$\boxed{P_{\text{fwd}} = P(0) - \text{Carry}} \tag{16.8}$$

Here Carry is interest income minus financing cost (as defined in Tuckman's repo/carry discussion).

Net basis for bond $i$ can be written as:

$$\text{NB}_i(t) = P_{\text{fwd},i}(t) - cf_i \cdot F(t)$$

Substitute $P_{\text{fwd},i}(t) = P_i(t) - \text{Carry}_i(t \to T)$:

$$\boxed{\text{NB}_i(t) = \text{GB}_i(t) - \text{Carry}_i(t \to T)}$$

**Unit Check:**
- Carry is \$ per \$100 face (interest income and financing cost are \$ amounts per \$100)
- NB and GB are also \$ per \$100

**Coupon payments before $T$:**
Tuckman explicitly notes forward pricing is "more complicated" if the bond pays coupons during the forward term. Hull likewise handles known coupon income in forward/futures pricing by subtracting the present value of coupons.

---

### 2.4 Net Basis as (Locked-In) Quality-Option Value

Tuckman's key interpretation (under "properly tailed" simplification):

- The delivery-date cost of delivering bond $i$ is $P_i(T) - cf_i \cdot F(T)$
- From time $t$, you can lock in a forward price $P_{\text{fwd},i}(t)$ for delivery at $T$ and also lock in futures price $F(t)$ for delivery at $T$
- Therefore the cost you can lock in is $P_{\text{fwd},i}(t) - cf_i \cdot F(t)$, which is net basis
- Since CTD delivery cost is zero by definition (at $T$), net basis is the value of the quality option (relative to bond $i$)

Also, if net basis $\approx 0$, the quality option is nearly worthless and selling the bond forward is equivalent to selling the futures (under proper tailing). When net basis equals zero:

$$F(t) = \frac{P_{\text{fwd},i}(t)}{cf_i}$$

---

### 2.5 Cash-and-Carry P&L and the "Implied Repo Rate" (Derived)

**Goal:** Given observed cash and futures prices, what repo rate is implied by a break-even cash-and-carry delivery?

Consider a simplified strategy for bond $i$ (assume no coupon payment before delivery):

1. At $t = 0$, buy bond $i$ at dirty price $P_i(0) + AI_i(0)$
2. Finance at repo rate $r$ for $d$ days (simple interest)
3. At $T$, deliver bond into futures and receive invoice $\text{Invoice}_i(T) = cf_i \cdot F(T) + AI_i(T)$

**Delivery P&L per \$100 face (simplified):**

$$\Pi_i \approx \underbrace{cf_i \cdot F(T) + AI_i(T)}_{\text{invoice received}} - \underbrace{(P_i(0) + AI_i(0))(1 + rd/360)}_{\text{financing repayment}}$$

Define the implied repo rate $r_{\text{imp},i}$ as the rate that makes $\Pi_i = 0$:

$$(P_i(0) + AI_i(0))(1 + r_{\text{imp},i} \cdot d/360) = cf_i \cdot F(T) + AI_i(T)$$

Solve:

$$\boxed{r_{\text{imp},i} = \left(\frac{cf_i \cdot F(T) + AI_i(T)}{P_i(0) + AI_i(0)} - 1\right) \frac{360}{d}}$$

**Interpretation (desk language):**
- If you can actually finance the bond at $r_{\text{repo}}$ and $r_{\text{imp},i} > r_{\text{repo}}$, then the trade "earns" more implied financing than it costs to fund $\Rightarrow$ cash-and-carry (long basis) tends to be attractive
- If $r_{\text{imp},i} < r_{\text{repo}}$, the carry is not enough $\Rightarrow$ cash-and-carry unattractive

In practice, delivery options complicate this: the bond delivered is chosen optimally by the short (quality option), and timing options can add value to the short; Hull explicitly warns exact theoretical pricing is difficult because of these options.

---

### 2.6 Futures DV01 Approximation (Derived, with Caveats)

Near delivery (and abstracting from option value), the futures is "like" the CTD scaled by conversion factor because at delivery the relationship $F \approx P_{\text{CTD}}/cf_{\text{CTD}}$ is central (Tuckman's min ratio framing).

Assume CTD is stable and $F \approx P/cf$. Differentiate w.r.t yield $y$:

$$\frac{dF}{dy} \approx \frac{1}{cf} \frac{dP}{dy}$$

Tuckman defines:

$$\text{DV01} \equiv -\frac{dP}{dy} \times \frac{1}{10000}$$

so a +1 bp yield change changes price by approximately $-\text{DV01}$.

Thus:

$$\boxed{\text{DV01}_{\text{fut, per \$100}} \approx \frac{\text{DV01}_{\text{CTD, per \$100}}}{cf_{\text{CTD}}}}$$

For a contract with face $N$ (e.g., $N = \$100{,}000$):

$$\boxed{\text{DV01}_{\text{fut, per contract}} \approx \frac{N}{100} \cdot \frac{\text{DV01}_{\text{CTD, per \$100}}}{cf_{\text{CTD}}}}$$

**Caveats:**
- If CTD changes, DV01 jumps
- Delivery options mean futures is not exactly a scaled CTD; option value can vary with yield level/time
- Daily settlement (futures vs forward) can matter; Tuckman sometimes assumes proper tailing to neglect it in certain derivations

---

## Worked Examples

### Global Example Conventions (Apply Unless Overridden)

- Prices are per \$100 face
- Repo interest uses simple interest $rd/360$
- Futures contract size $N = \$100{,}000$ face (so \$ per contract $=$ price-per-100 $\times 1000$)
- Invoice price formula: $\text{Invoice} = cf \times F + AI$

---

### Example A: Invoice Price

**Inputs:**
- Futures settlement price: $F = 112.50$ (\$112.50 per \$100)
- Conversion factor (delivered bond): $cf = 0.9012$
- Accrued interest of delivered bond at delivery: $AI(T) = 1.35$

**Step 1: Delivery (flat) price**
$$cf \cdot F = 0.9012 \times 112.50 = 101.385$$

**Step 2: Invoice price**
$$\text{Invoice} = cf \cdot F + AI = 101.385 + 1.35 = 102.735$$

**Step 3: Convert to dollars per contract**
Contract face $N = \$100{,}000 \Rightarrow N/100 = 1000$
$$\text{Invoice}_{\$} = 102.735 \times 1000 = \$102{,}735$$

**Unit Check:**
$cf \cdot F$ is \$/100; adding $AI$ (\$/100) gives \$/100; multiplying by 1000 gives dollars/contract. $\checkmark$

---

### Example B: Delivery Economics

Compute the short's delivery profit (or cost) from buying the bond now, financing in repo, and delivering into futures at $T$.

**Inputs:**
- Bond clean price today: $P(0) = 101.20$
- Accrued today: $AI(0) = 0.45$ $\Rightarrow$ dirty today $= 101.65$
- Repo rate: $r = 5.00\%$
- Days to delivery: $d = 90$
- Invoice at delivery from Example A: $\text{Invoice}(T) = 102.735$

**Step 1: Financing repayment at delivery**
$$1 + rd/360 = 1 + 0.05 \times \frac{90}{360} = 1 + 0.0125 = 1.0125$$
$$\text{Repay} = 101.65 \times 1.0125 = 102.920625$$

**Step 2: Delivery profit**
$$\Pi = \text{Invoice} - \text{Repay} = 102.735 - 102.920625 = -0.185625$$

**Per contract dollars:**
$$\Pi_{\$} = -0.185625 \times 1000 = -\$185.625 \approx -\$186$$

**Interpretation:**
Negative profit means it costs the short about 18.6 cents per \$100 face to deliver this bond under these financing assumptions.

---

### Example C: CTD Selection from a Basket

Identify CTD by comparing delivery costs across candidates.

**Inputs:**
- Futures settlement price: $F = 112.50$
- Candidate deliverables:

| Bond | $P$ | $cf$ | $AI(T)$ |
|------|-----|------|---------|
| A | 101.20 | 0.9012 | 1.35 |
| B | 109.80 | 0.9775 | 2.10 |
| C | 94.60 | 0.8420 | 0.90 |

Compute cost of delivery $\text{CostDel} = P - cf \cdot F$ (accrued cancels).

**Bond A:**
$$cf \cdot F = 0.9012 \times 112.50 = 101.385$$
$$\text{CostDel}_A = 101.20 - 101.385 = -0.185$$

**Bond B:**
$$cf \cdot F = 0.9775 \times 112.50 = 109.96875$$
$$\text{CostDel}_B = 109.80 - 109.96875 = -0.16875$$

**Bond C:**
$$cf \cdot F = 0.8420 \times 112.50 = 94.725$$
$$\text{CostDel}_C = 94.60 - 94.725 = -0.125$$

**Decision:**
CTD is the bond with the lowest cost of delivery (most negative): **Bond A** ($-0.185$).

**Sanity Check (accrued cancels):**
If you compute using dirty prices: $(P + AI) - (cf \cdot F + AI) = P - cf \cdot F$. Same result. $\checkmark$

---

### Example D: CTD Switch with Yield Level

Show how CTD can switch after a yield shift.

**Inputs:**
- Assume a parallel yield rise of $+50$ bp
- Use DV01-based toy repricing: $\Delta P \approx -\text{DV01} \times \Delta y_{\text{bp}}$

**DV01s per \$100 face (assumed):**
- Bond A: DV01 $= 0.085$
- Bond B: DV01 $= 0.075$
- Bond C: DV01 $= 0.095$

**New futures settlement price (given scenario):** $F' = 107.00$

**Step 1: Reprice bonds**
$$P' = P - \text{DV01} \times 50$$

- A: $P'_A = 101.20 - 0.085 \times 50 = 101.20 - 4.25 = 96.95$
- B: $P'_B = 109.80 - 0.075 \times 50 = 109.80 - 3.75 = 106.05$
- C: $P'_C = 94.60 - 0.095 \times 50 = 94.60 - 4.75 = 89.85$

**Step 2: Compute new delivery costs** $\text{CostDel}' = P' - cf \cdot F'$

- A: $cf \cdot F' = 0.9012 \times 107.00 = 96.4284 \Rightarrow \text{CostDel}'_A = 96.95 - 96.4284 = 0.5216$
- B: $cf \cdot F' = 0.9775 \times 107.00 = 104.5925 \Rightarrow \text{CostDel}'_B = 106.05 - 104.5925 = 1.4575$
- C: $cf \cdot F' = 0.8420 \times 107.00 = 90.094 \Rightarrow \text{CostDel}'_C = 89.85 - 90.094 = -0.2440$

**Decision:**
CTD switches from A (in Example C) to **C** here.

**Interpretation:**
Hull notes that yield levels relative to 6% and curve shape affect which bond the CF system favors (low coupon/long maturity vs high coupon/short maturity). This example illustrates how a yield change can flip the CTD ranking.

---

### Example E: Basis Calculation

Compute gross basis and net basis for bond A and interpret the sign.

**Inputs:**
- Bond A: $P(0) = 101.20$, $cf = 0.9012$, $F = 112.50$
- Accrued today: $AI(0) = 0.45$
- Accrued at delivery: $AI(T) = 1.35$
- Repo rate: $r = 5.00\%$, days to delivery $d = 90$
- Assume no coupon payment before delivery

**Step 1: Gross basis**
$$\text{GB} = P - cf \cdot F = 101.20 - 101.385 = -0.185$$

**Interpretation (gross basis):**
Negative gross basis means $P < cf \cdot F$: after CF scaling, the futures price is "higher" than the bond's clean price.

**Step 2: Carry (from (16.8) decomposition)**

Interest income (via accrued change):
$$AI(T) - AI(0) = 1.35 - 0.45 = 0.90$$

Financing cost:
$$(P + AI(0)) \cdot rd/360 = 101.65 \times 0.05 \times \frac{90}{360} = 101.65 \times 0.0125 = 1.270625$$

Carry:
$$\text{Carry} = 0.90 - 1.270625 = -0.370625$$

**Step 3: Net basis**
$$\text{NB} = \text{GB} - \text{Carry} = -0.185 - (-0.370625) = +0.185625$$

**Interpretation (net basis):**
Net basis $> 0$ indicates that after carry/funding, the bond is not cheap versus futures; a cash-and-carry (long basis) trade would lose roughly 18.6 cents per \$100 (consistent with Example B).

---

### Example F: Implied Repo Rate

Compute the repo rate implied by a break-even cash-and-carry into delivery for bond A.

**Inputs:**
- Dirty price today: $P + AI(0) = 101.20 + 0.45 = 101.65$
- Invoice at delivery: $cf \cdot F + AI(T) = 102.735$ (Example A)
- Days to delivery: $d = 90$
- Assume no coupon payment before delivery

**Break-even condition:**
$$101.65(1 + r_{\text{imp}} \cdot 90/360) = 102.735$$

**Solve:**
$$r_{\text{imp}} = \left(\frac{102.735}{101.65} - 1\right) \frac{360}{90}$$

Compute the ratio increment:
$$\frac{102.735}{101.65} - 1 = \frac{102.735 - 101.65}{101.65} = \frac{1.085}{101.65} \approx 0.010673$$

Annualize:
$$r_{\text{imp}} \approx 0.010673 \times 4 = 0.042692 \approx 4.27\%$$

**Interpretation:**
If you can finance bond A below ~4.27% (e.g., special), cash-and-carry tends to be attractive. If your funding is above ~4.27% (e.g., GC at 5%), it tends to be unattractive.

---

### Example G: Repo Specialness Impact

Repeat Example F under GC vs special repo and show CTD economics change.

**Inputs:**
- Same as Example F, but compare two funding rates:
  - GC repo: $r_{\text{GC}} = 5.00\%$
  - Special repo: $r_{\text{sp}} = 3.50\%$

Tuckman: on-the-run securities can trade special; special spread defined as GC minus special rate.

**Step 1: Profit under GC (5.00%)**
From Example B: $\Pi_{\text{GC}} = -0.185625$ per \$100.

**Step 2: Profit under special (3.50%)**

Financing factor:
$$1 + r_{\text{sp}} \cdot d/360 = 1 + 0.035 \times \frac{90}{360} = 1 + 0.00875 = 1.00875$$

Repayment:
$$101.65 \times 1.00875 = 101.65 + 101.65 \times 0.00875 = 101.65 + 0.8894375 = 102.5394375$$

Profit:
$$\Pi_{\text{sp}} = 102.735 - 102.5394375 = 0.1955625 \approx 19.56 \text{ cents per \$100}$$

Dollar profit per contract:
$$0.1955625 \times 1000 = \$195.56$$

**Step 3: Net basis viewpoint**

Under special, carry becomes slightly positive:
$$\text{Carry} = (AI_T - AI_0) - (P + AI_0) \cdot rd/360 = 0.90 - 0.8894375 = 0.0105625$$

Net basis:
$$\text{NB} = \text{GB} - \text{Carry} = -0.185 - 0.0105625 = -0.1955625$$

and $\Pi = -\text{NB}$ (matches).

**Interpretation:**
Special financing can flip trade economics and can therefore influence CTD selection and basis levels.

**Bound/constraint note:**
Tuckman discusses that specialness is linked to supply/demand and mentions constraints (e.g., fail option logic) in bounding special spreads; however, I'm not sure how to translate that into a precise operational "specialness bound" for your specific contract and settlement conventions without the relevant repo market conventions and contract details.

---

### Example H: Futures DV01 and Hedge Ratio

Compute futures DV01 and hedge a cash Treasury position; then show hedge ratio changes if CTD changes.

**Inputs:**
- Cash position: long \$50,000,000 face of a Treasury with $\text{DV01}_{\text{cash per \$100}} = 0.085$
- CTD scenario 1: Bond A with $\text{DV01}_A = 0.085$, $cf_A = 0.9012$
- CTD scenario 2 (switch): Bond C with $\text{DV01}_C = 0.095$, $cf_C = 0.8420$
- Contract face $N = \$100{,}000 \Rightarrow N/100 = 1000$
- Hedge ratio logic uses DV01s (Tuckman)

**Step 1: Cash DV01 in dollars**
$$\text{DV01}_{\text{cash, \$}} = 0.085 \times \frac{50{,}000{,}000}{100} = 0.085 \times 500{,}000 = 42{,}500 \text{ \$/bp}$$

**Step 2: Futures DV01 per contract (CTD = A)**
$$\text{DV01}_{\text{fut, per contract}} \approx 1000 \times \frac{0.085}{0.9012}$$

Compute ratio:
$$\frac{0.085}{0.9012} \approx 0.09432 \Rightarrow \text{DV01}_{\text{fut}} \approx 94.32 \text{ \$/bp}$$

**Step 3: Hedge ratio (contracts)**
$$n = \frac{42{,}500}{94.32} \approx 450.7 \Rightarrow \boxed{451 \text{ contracts}}$$

**Step 4: If CTD switches to C**
$$\text{DV01}_{\text{fut, per contract}} \approx 1000 \times \frac{0.095}{0.8420}$$

Compute ratio:
$$\frac{0.095}{0.8420} \approx 0.11282 \Rightarrow \text{DV01}_{\text{fut}} \approx 112.82 \text{ \$/bp}$$

New hedge ratio:
$$n' = \frac{42{,}500}{112.82} \approx 376.7 \Rightarrow \boxed{377 \text{ contracts}}$$

**Interpretation:**
The hedge ratio changes by ~74 contracts purely from CTD switching—this is a material risk for large hedges.

---

### Example I: Delivery Option Value — Toy Scenario Analysis

Quantify a simplified "timing option" value by comparing two delivery strategies.

Tuckman: short can deliver any day during delivery month (timing option).

I'm not sure about your contract's exact delivery window timing and notice rules; this is a simplified scenario analysis illustrating economic intuition rather than a full contract-accurate valuation.

**Setup:**
- Deliverable bond: A with $cf = 0.9012$
- Current bond dirty price: 101.65
- Repo rate: $r = 4.50\%$
- Two possible delivery dates:
  - Early: $T_1$ in 30 days
  - Late: $T_2$ in 90 days
- Assume no coupon payment during the window

**Scenario 1: Deliver early (30d)**
- Futures settlement at $T_1$: $F_1 = 112.50$
- Accrued at $T_1$: $AI_1 = 1.10$

Invoice:
$$\text{Inv}_1 = cf \cdot F_1 + AI_1 = 0.9012 \times 112.50 + 1.10 = 101.385 + 1.10 = 102.485$$

Financing repayment:
$$101.65(1 + 0.045 \times 30/360) = 101.65(1 + 0.00375) = 101.65 + 0.3811875 = 102.0311875$$

Profit:
$$\Pi_1 = 102.485 - 102.0311875 = 0.4538125$$

**Scenario 2: Deliver late (90d)**
- Futures settlement at $T_2$: $F_2 = 112.20$
- Accrued at $T_2$: $AI_2 = 1.35$

Invoice:
$$\text{Inv}_2 = 0.9012 \times 112.20 + 1.35 = 101.11464 + 1.35 = 102.46464$$

Financing repayment:
$$101.65(1 + 0.045 \times 90/360) = 101.65(1 + 0.01125) = 101.65 + 1.1435625 = 102.7935625$$

Profit:
$$\Pi_2 = 102.46464 - 102.7935625 = -0.3289225$$

**Toy "option value":**
If the short can choose the better strategy, the incremental value versus being forced into the worse outcome is:
$$\text{OptionValue} \approx \Pi_1 - \Pi_2 = 0.4538125 - (-0.3289225) = 0.782735 \text{ per \$100}$$

Per contract dollars:
$$0.782735 \times 1000 = \$782.74$$

**Interpretation:**
Timing flexibility can be valuable when the combination of futures price movements, accrued-interest evolution, and financing conditions makes one delivery date materially better than another.

---

### Example J: P&L Explain for a Futures Basis Trade

Construct a toy basis trade and decompose P&L.

**Trade at $t_0$:**
- Long cash bond A, face $Q = \$10{,}000{,}000$
- Bond A: clean $P_0 = 101.20$, accrued $AI_0 = 0.45$ $\Rightarrow$ dirty $= 101.65$
- Finance via repo at $r = 4.50\%$ for $d = 30$ days
- Short Treasury futures at $F_0 = 112.50$, number of contracts $n = 90$
  (Close to DV01-neutral given Example H scaling; exact hedge choice is desk-specific)

**Market at $t_1$ (30 days later):**
- Bond A clean $P_1 = 102.90$ (up 1.70)
- Accrued $AI_1 = 0.75$ (up 0.30)
- Futures $F_1 = 114.30$ (up 1.80)

**Step 1: Cash P&L (long cash financed)**

Dirty change per \$100:
$$\Delta\text{Dirty} = (P_1 + AI_1) - (P_0 + AI_0) = (102.90 + 0.75) - (101.20 + 0.45) = 103.65 - 101.65 = 2.00$$

Dollar P&L from dirty move:
$$\text{PnL}_{\text{dirty}} = \frac{Q}{100} \times 2.00 = \frac{10{,}000{,}000}{100} \times 2.00 = 100{,}000 \times 2.00 = \$200{,}000$$

Financing cost over 30 days:
$$\text{FinCost per \$100} = 101.65 \times 0.045 \times \frac{30}{360} = 101.65 \times 0.00375 = 0.3811875$$

Dollar financing cost:
$$\text{FinCost}_{\$} = 0.3811875 \times 100{,}000 = \$38{,}118.75$$

Net cash P&L:
$$\text{PnL}_{\text{cash}} = 200{,}000 - 38{,}118.75 = \$161{,}881.25$$

**Step 2: Futures P&L (short futures)**

Per contract dollar value of 1 point move is $N/100 = 1000$.

Futures price change:
$$\Delta F = 114.30 - 112.50 = 1.80$$

Per-contract P&L (short):
$$\text{PnL}_{\text{fut, 1}} = -1000 \times 1.80 = -\$1{,}800$$

For $n = 90$:
$$\text{PnL}_{\text{fut}} = -1{,}800 \times 90 = -\$162{,}000$$

**Step 3: Total P&L**
$$\text{PnL}_{\text{total}} = \text{PnL}_{\text{cash}} + \text{PnL}_{\text{fut}} = 161{,}881.25 - 162{,}000 = -\$118.75$$

**Step 4: Decomposition**

*Carry/financing component (cash leg):*

Interest income from accrued (over 30d):
$$\Delta AI = 0.75 - 0.45 = 0.30 \Rightarrow \$\text{income} = 0.30 \times 100{,}000 = \$30{,}000$$

Financing cost: \$38,118.75

$$\text{Carry} = \$30{,}000 - \$38{,}118.75 = -\$8{,}118.75$$

*Price-move / rate exposure component:*

Cash clean price gain:
$$\Delta P = 1.70 \Rightarrow \$\text{gain} = 1.70 \times 100{,}000 = \$170{,}000$$

Futures loss: $-\$162{,}000$

Net from rate move: $+\$8{,}000$

*Basis change (gross basis):*

Basis at $t_0$:
$$\text{GB}_0 = P_0 - cf \cdot F_0 = 101.20 - (0.9012 \times 112.50) = 101.20 - 101.385 = -0.185$$

Basis at $t_1$:
$$0.9012 \times 114.30 = 103.00716$$
$$\text{GB}_1 = 102.90 - 103.00716 = -0.10716$$

Change:
$$\Delta\text{GB} = (-0.10716) - (-0.185) = 0.07784$$

Dollar impact:
$$0.07784 \times 100{,}000 = \$7{,}784 \approx \$8{,}000$$

*Roll-down:* Ignored in this toy example (would require curve/roll assumptions).

*CTD/option effects:* Assumed zero here (CTD stable). If CTD switched during the horizon, hedge ratio and basis behavior could shift materially (Example H).

**Link back to Tuckman's net-basis P&L framing:**
Tuckman shows (under proper tailing) that the P&L from a long basis position is proportional to the change in net basis.

---

## Practical Notes

### Contract-Spec Ambiguity Checklist (What Depends on the Exact Exchange Contract)

| Item | Notes |
|------|-------|
| **Deliverable basket rules** | Maturity window, eligible issues, exclusions (e.g., certain on-the-run/off-the-run rules). I'm not sure for your specific contract without the exchange spec. |
| **Conversion factor definition** | Hull describes a 6% semiannual discounting approach and rounding rules, with different rounding for some note futures (e.g., 2Y/5Y rounding to nearest month). Tuckman frames a notional 6% coupon basis. Exact rounding, schedule assumptions, and any contract updates: I'm not sure without the current contract specs. |
| **Delivery window** | First delivery date, last trade date, last delivery date (and whether they coincide). Tuckman provides a specific TYH2 example where last trade date and last delivery date differ. I'm not sure for your contract's exact calendar rules. |
| **Invoice price details and rounding** | Rounding of $cf \cdot F$, accrued interest conventions on delivery, minimum tick, and invoice rounding. I'm not sure without contract rulebook specifics. |
| **Settlement conventions** | Futures are marked-to-market daily; Tuckman's basis-trade derivations sometimes assume "properly tailed" to neglect this effect. Exact timing of settlement price and delivery notice cutoffs (wild card play details): I'm not sure without the rulebook, though Hull describes a specific mechanism for the Treasury bond futures contract. |

### Common Pitfalls

1. **Mixing clean vs dirty prices** and forgetting accrued interest in invoice price (or double-counting it). (Clean/dirty distinction explicitly noted by Hull.)

2. **Treating futures DV01 as stable:** CTD switching risk can change hedge ratios abruptly.

3. **Ignoring repo specialness:** GC vs special materially affects carry and basis economics.

4. **Confusing gross basis (price difference) with net basis (carry-adjusted, economically relevant).**

### Implementation Pitfalls

1. **Date handling:** Delivery window dates (first/last delivery, last trade) and coupon dates (for coupon adjustments).

2. **Consistent day count:**
   - Repo financing often uses actual/360 simple-interest conventions
   - Treasury accrued interest uses actual/actual (noted in Tuckman's footnote)

3. **Sign conventions:** Long/short futures and long/short cash; whether you are financing (repo) or reverse repo.

4. **Carry modeling:** If coupons occur before delivery, adjust forward/futures relationships (Tuckman and Hull both warn that intermediate coupons complicate the forward).

### Verification Tests

1. **Dimensional checks:** Prices per 100, accrued per 100, DV01 in \$/bp, repo annualized rates.

2. **CTD decision reproducibility:** Recompute $P - cf \cdot F$ for all deliverables; CTD must be the minimum.

3. **Implied repo sanity:** Compare implied repo to plausible GC/special funding levels; extreme implied repo warrants re-checking accrued/coupon assumptions.

4. **Hedge sanity:** Small parallel shocks should reduce PV01 when hedged (assuming CTD stability).

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Treasury futures are deliverable-basket contracts:** The short chooses the delivered bond (quality option) and often the delivery timing (timing option).

2. **The conversion factor** $cf_i$ standardizes deliverables; in the cited sources it is anchored to a 6% notional yield/coupon logic.

3. **Delivery price** $= cf_i \cdot F$; **invoice price** $= cf_i \cdot F + AI$.

4. **Cost of delivery** for bond $i$ is $P_i - cf_i \cdot F$; accrued interest cancels.

5. **CTD** is the bond that minimizes delivery cost (equivalently maximizes delivery profit for the short).

6. **CTD can change** with yield level/curve shape (Hull) and with financing/specialness (Tuckman).

7. **Net basis uses forward/carry:** $\text{NB} = P_{\text{fwd}} - cf \cdot F = \text{GB} - \text{Carry}$. Forward pricing under repo ties directly to carry.

8. **Tuckman interprets net basis** (under proper tailing) as the value of the quality option that can be locked in at time $t$.

9. **Repo specialness matters:** Special spread = GC minus special rate; specials can change carry and CTD economics materially.

10. **Futures hedging depends on CTD:** A practical DV01 approximation is $\text{DV01}_{\text{fut}} \approx \text{DV01}_{\text{CTD}}/cf$, so CTD switching creates hedge jumps.

---

### Cheat Sheet: Key Formulas and Steps

**Invoice price (per \$100):**
$$\boxed{\text{Invoice}_i(t) = cf_i \cdot F(t) + AI_i(t)}$$

**Cost of delivery (per \$100, clean):**
$$\boxed{\text{CostDel}_i(t) = P_i(t) - cf_i \cdot F(t)}$$

**CTD criterion:**
$$\boxed{\text{CTD} = \arg\min_i \{P_i(t) - cf_i \cdot F(t)\}}$$

**Forward price under repo (no coupons before $T$):**
$$\boxed{P_{\text{fwd}} + AI(T) = (P(0) + AI(0))(1 + rd/360)}$$

**Carry decomposition (no coupons before $T$):**
$$\boxed{P_{\text{fwd}} = P(0) - \text{Carry}}$$
$$\text{Carry} = AI(T) - AI(0) - (P(0) + AI(0)) \cdot rd/360$$

**Gross basis:**
$$\boxed{\text{GB} = P - cf \cdot F}$$

**Net basis:**
$$\boxed{\text{NB} = P_{\text{fwd}} - cf \cdot F = \text{GB} - \text{Carry}}$$

**Implied repo (derived, no coupons):**
$$\boxed{r_{\text{imp}} = \left(\frac{cf \cdot F + AI(T)}{P(0) + AI(0)} - 1\right) \frac{360}{d}}$$

**Futures DV01 (approx., derived):**
$$\boxed{\text{DV01}_{\text{fut, per contract}} \approx \frac{N}{100} \cdot \frac{\text{DV01}_{\text{CTD per \$100}}}{cf_{\text{CTD}}}}$$

**Hedge ratio:**
$$\boxed{n \approx \frac{\text{DV01 exposure}}{\text{DV01 hedge}}}$$

---

### Flashcards (30 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the "delivery basket" in a Treasury futures contract? | The set of eligible Treasury securities the short may choose to deliver at settlement (quality option). |
| 2 | Who controls the choice of delivered bond? | The short position holder (quality option). |
| 3 | Define the conversion factor $cf_i$. | A contract-set scaling number applied to bond $i$ to translate futures price into delivery price; anchored to a 6% standard in the cited sources. |
| 4 | What is the delivery price for bond $i$? | $cf_i \cdot F(t)$. |
| 5 | What is the invoice price for delivery of bond $i$? | $cf_i \cdot F(t) + AI_i(t)$. |
| 6 | Why is accrued interest added to the delivery price? | Because the cash exchange is for the bond's dirty price; accrued interest compensates for coupon accrual. |
| 7 | Write the cost of delivery in clean-price terms. | $\text{CostDel}_i = P_i - cf_i \cdot F$. |
| 8 | Why does accrued interest cancel in the cost-of-delivery formula? | It appears in both the purchase cost and the invoice received. |
| 9 | Define CTD. | The bond that minimizes delivery cost (maximizes delivery profit) for the short. |
| 10 | How do you compute CTD from a list of deliverables at a given futures price? | Compute $P_i - cf_i \cdot F$ for each; choose the smallest value. |
| 11 | What is "gross basis" in this chapter? | $P_i - cf_i \cdot F$ (clean cash minus futures-adjusted clean). |
| 12 | What is "net basis"? | $P_{\text{fwd},i} - cf_i \cdot F$, i.e., carry-adjusted basis. |
| 13 | Why is net basis more relevant than gross basis for a basis trade? | It adjusts for financing/carry to delivery, which drives actual economics. |
| 14 | What does Tuckman interpret net basis as (under proper tailing)? | The value of the quality option at delivery, relative to bond $i$. |
| 15 | What happens if net basis is near zero? | The quality option is nearly worthless; selling bond forward is nearly equivalent to selling futures (proper tailing). |
| 16 | Give the repo-based forward price equation (no coupons before delivery). | $P_{\text{fwd}} + AI(T) = (P(0) + AI(0))(1 + rd/360)$. |
| 17 | Define carry in Tuckman's forward-price decomposition. | Interest income minus financing cost; $P_{\text{fwd}} = P(0) - \text{Carry}$. |
| 18 | What is the timing option? | The short can choose delivery date within the delivery month/window. |
| 19 | What is the "wild card play" (Hull)? | A timing-related option from settlement time vs later spot trading, allowing the short to decide delivery after observing late-day price moves (contract-specific). |
| 20 | What is the "end-of-month option" (Tuckman)? | An option arising when last trade date precedes last delivery date and deliveries after last trade date use the final settlement price. |
| 21 | Why do delivery options make exact futures pricing hard? | Because the short's optimal choices (bond and timing) are option-like and hard to value precisely. |
| 22 | What is "general collateral" repo? | Repo where lender accepts any Treasury collateral (doesn't care which issue). |
| 23 | What is "special" repo? | Repo requiring particular collateral (specific Treasury issue), often at a lower rate. |
| 24 | Define "special spread" in Tuckman. | Overnight GC rate minus the overnight special rate of the issue. |
| 25 | Why can repo specialness affect CTD? | Cheaper financing improves carry, changing net basis and delivery economics. |
| 26 | Define DV01 in Tuckman's terms. | $\text{DV01} = -\frac{dP}{dy} \times \frac{1}{10000}$ (positive measure of price sensitivity per bp). |
| 27 | How is a hedge ratio computed using DV01? | Hedge ratio $= \frac{\text{DV01 exposure}}{\text{DV01 hedge}}$. |
| 28 | Give the practical approximation linking futures DV01 to CTD DV01. | $\text{DV01}_{\text{fut}} \approx \text{DV01}_{\text{CTD}}/cf_{\text{CTD}}$ (with contract scaling). |
| 29 | What is CTD switching risk? | The bond that is optimal to deliver can change with yields/funding, causing hedge ratios and risk measures to jump. |
| 30 | What is the key practitioner message of this chapter? | Treasury futures pricing and RV are about delivery optionality + financing (repo). |

---

## Mini Problem Set

*Provide brief solution sketches for questions 1–8 only.*

### Questions 1–8 (With Solution Sketches)

**1. Compute invoice price given $F$, $cf$, and $AI$.**

*Sketch:* Use $\text{Invoice} = cf \cdot F + AI$; unit check per \$100; multiply by $N/100$ for dollars.

---

**2. Given three deliverables with $(P, cf)$, identify CTD at a given $F$.**

*Sketch:* Compute $P - cf \cdot F$ for each; smallest is CTD (accrued cancels).

---

**3. Compute gross basis and interpret sign.**

*Sketch:* $\text{GB} = P - cf \cdot F$; negative means cash clean is below futures-adjusted price.

---

**4. Compute carry over $d$ days assuming no coupon payment.**

*Sketch:* $\text{Carry} = (AI_T - AI_0) - (P + AI_0) \cdot rd/360$.

---

**5. Compute net basis from gross basis and carry.**

*Sketch:* $\text{NB} = \text{GB} - \text{Carry}$.

---

**6. Compute implied repo rate $r_{\text{imp}}$ (no coupons) from cash and futures terms.**

*Sketch:* Solve $(P + AI_0)(1 + rd/360) = cf \cdot F + AI_T \Rightarrow r$ as in Section 2.5.

---

**7. Show how changing repo from GC to special affects net basis and delivery economics.**

*Sketch:* Recompute carry using the lower repo rate; net basis changes by $(P + AI_0) \cdot \Delta r \cdot d/360$.

---

**8. Given cash DV01 and CTD $(\text{DV01}, cf)$, compute futures DV01 per contract and hedge ratio.**

*Sketch:* $\text{DV01}_{\text{fut}} \approx (N/100)(\text{DV01}_{\text{CTD}}/cf)$; contracts $= \text{DV01 cash}/\text{DV01 fut}$.

---

### Questions 9–16 (No Solutions Provided)

**9.** Explain qualitatively why the futures price is not a pure cost-of-carry forward price.

**10.** Describe how a CTD switch can create P&L on an otherwise DV01-neutral hedge.

**11.** Construct a scenario where gross basis is negative but net basis is positive. Explain why a basis trade can lose money despite "cheap" gross basis.

**12.** Suppose a bond becomes extremely special in repo. Explain the mechanism by which it might become CTD even if its cash price rises.

**13.** In Hull's description, what market feature creates the wild card play, and how does it benefit the short?

**14.** Tuckman notes that in some European government bond futures, key delivery dates coincide. What option does that remove (relative to the U.S. feature he describes)?

**15.** Explain how net basis relates to the value of the quality option (Tuckman).

**16.** Design a monitoring checklist for a live Treasury futures hedge: what data do you watch daily (prices, repo, CTD, basis, carry) and why?

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Concept | Source |
|---------|--------|
| Deliverable-basket structure, quality option, timing option | Tuckman Ch 19-20, Hull Ch 6 |
| Invoice price formula: $cf \cdot F + AI$ | Tuckman Ch 20 (explicit), Hull Ch 6 |
| Conversion factor as 6% discounting | Hull Ch 6, Tuckman Ch 20 |
| Cost of delivery formula | Tuckman Eq 20.1 |
| CTD minimizes cost of delivery | Tuckman Ch 20, Hull Ch 6 |
| Forward price under repo: $(P + AI)(1 + rd/360)$ | Tuckman Eq 16.7 |
| Carry decomposition: $P_{\text{fwd}} = P - \text{Carry}$ | Tuckman Eq 16.8 |
| DV01 definition: $-dP/dy \times 1/10000$ | Tuckman Ch 5-6 |
| Hedge ratio: DV01 exposure / DV01 hedge | Tuckman Ch 6-7 |
| Special spread: GC minus special rate | Tuckman Ch 15-16 |
| End-of-month option | Tuckman Ch 20 (TYH2 example) |
| Wild card play | Hull Ch 6 |
| Delivery options make exact pricing difficult | Hull Ch 6 (explicit statement) |
| GC vs special repo mechanics | Tuckman Ch 15-16 |
| Yield-level effects on CTD (high/low coupon tendencies) | Hull Ch 6 |

### (B) Reasoned Inference — Note Derivation Logic

| Derived Result | Derivation Chain |
|----------------|------------------|
| Net basis formula: $\text{NB} = \text{GB} - \text{Carry}$ | Substitution of $P_{\text{fwd}} = P - \text{Carry}$ into net basis definition |
| Implied repo formula | Break-even condition on cash-and-carry: $(P + AI_0)(1 + rd/360) = cf \cdot F + AI_T$ |
| Futures DV01 $\approx \text{DV01}_{\text{CTD}}/cf$ | Differentiation of $F \approx P/cf$ w.r.t. yield |
| Accrued cancels in cost of delivery | Algebraic: $(P + AI) - (cf \cdot F + AI) = P - cf \cdot F$ |

### (C) Speculation — Flag Uncertainties

| Item | Uncertainty |
|------|-------------|
| Exact fails-charge formula | Sources discuss fails conceptually but don't specify penalty mechanics |
| Precise delivery notice deadlines, settlement timestamps | Contract-specific; depends on exact exchange rulebook version |
| Repo conventions outside U.S. Treasury repo | Would need additional source verification |
| Specialness bounds as operational constraints | Tuckman discusses conceptually but precise bounds are contract/market specific |

---

*Last Updated: January 2026*
