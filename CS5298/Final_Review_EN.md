# CS5298 Final Exam Review (English)

Sources: [Week 12 final-review slides](slides/Week%2012.pptx) (69 slides), the [review lecture recording](record/49ccff7b9dfa456cab43f43b06560d8c.mp3) (about 69 minutes), and the weekly decks. Recording times below are approximate. The instructor says everything covered in this review lecture matters for the final (00:25). The priorities below guide study time; they are not an official exclusion of other course material.

## Exam format and slide priority

- The final is similar to the midterm: **no multiple-choice or true/false questions**; expect larger case questions answered step by step. The instructor expects it to be easier than the midterm (00:00–00:25).
- Study **Week 12 from start to finish first**, then use the original decks below for diagrams and derivations.
- **No calculators** in the final. The instructor says AMM calculations will be **symbolic rather than numerical** (around 33:20). Work through the numerical slide examples to learn the method, rather than memorizing their decimals.

| Priority | Deck and suggested slides | Why |
| --- | --- | --- |
| First | [Week 12](slides/Week%2012.pptx), slides 2–69 | The actual final-review lecture: Merkle trees (2–12), consensus (13–28), AMMs (29–48), flash loans (49–63), MEV (64–69). |
| High | [Week 1](slides/Week%201.pptx), 73–83 | Merkle update paths, membership proofs, and complexity. |
| High | [Week 2](slides/Week%202.pptx), 36–46 | Bitcoin protocol, forks, and probabilistic finality. |
| High | [Week 4](slides/Week%204.pptx), 14–28 | Network assumptions, FLP, PBFT phases, quorums, and message complexity. |
| High | [Week 7](slides/Week%207.pptx), 12–23; [Week 8](slides/Week%208.pptx), 5–14 and 34 | CFMM pricing and slippage; concentrated liquidity and overlapping LP ranges. |
| High | [Week 9](slides/Week%209.pptx), 45–64 | Flash-loan mechanics, use cases, and oracle manipulation. The instructor explicitly places more emphasis on flash loans than general lending (around 39:10). |
| High | [Week 10 - MEV](slides/Week%2010%20-%20MEV.pptx), roughly 12–15 and 28–40 | Transaction-order attacks, the sandwich example, and defenses. |
| Later | [Week 3](slides/Week%203.pptx) and [Week 6](slides/Week%206.pptx) | Solidity/EVM syntax and the Lyquor case study are not revisited as main Week 12 topics. EVM atomicity still helps explain flash loans. The recording does not declare these decks entirely out of scope. |

## What you should be able to do

### 1. Merkle trees — Week 12 slides 2–12; recording 00:33–05:06

- Explain why hashing the concatenation `x1‖…‖xn` is costly: an inclusion proof reveals other data, and updating one item requires rehashing O(n) input.
- On a concrete tree, **list every ancestor hash that changes** when a leaf changes. For a membership proof, **list the sibling hashes at each level** and recompute the root in order. For the lecture's `x100` example, the proof needs `x101`, `x11`, and `x0`.
- Explain why an n-leaf tree requires O(log n) hashes for an update or inclusion proof, while the root digest has fixed length.

### 2. Nakamoto consensus and PBFT — slides 13–28; about 05:17–28:23

- Explain PoW's nonce/target condition, expensive trial-and-error mining, and cheap one-hash verification. Describe gossip, block assembly, mining, broadcast, transaction validation, and extending an accepted block.
- Explain concurrent blocks, forks, possible double spending, longest-chain resolution, and why an appended Bitcoin block needs **k-depth confirmation** rather than immediate finalization.
- Compare the two approaches on **fault threshold, commit rule, and safety**. Nakamoto consensus assumes malicious hash power below 50% and offers probabilistic safety after confirmations. PBFT normally needs `n ≥ 3f+1`; a replica commits after `2f+1` matching commit messages and has deterministic safety under its assumptions. Both seek a consistent transaction order. The instructor says the midterm's particular private-chain probability calculation will **not recur unchanged** (around 12:08); the conceptual comparison remains important.
- Distinguish Byzantine from crash faults. Define synchronous delivery (known bound Δ), asynchronous delivery (no known bound, eventual delivery), and partial synchrony (a bound after GST); discuss their practical tradeoffs. State the FLP limitation: deterministic consensus cannot guarantee termination with even one faulty process in a fully asynchronous system. Know partial synchrony and randomization as ways to change the assumptions.
- Know the purposes of PBFT's three phases: pre-prepare assigns a sequence number, prepare protects safety within a view, and commit protects safety across views. Normal-case communication is **O(n²)**. The instructor does not expect tricky protocol details but explicitly asks for these basics (about 25:40–27:00).

### 3. AMMs, CFMMs, and concentrated liquidity — slides 29–48; about 28:30–39:05

- LPs supply the reserves and earn fees. A CFMM constrains reserve changes with `φ(x,y)=k`; the constant-product example is `xy=k`.
- Derive the **positive marginal exchange price** from `dφ=φ_x dx+φ_y dy=0`, giving `-dy/dx=φ_x/φ_y`. If x is ETH reserves and y is USD reserves, `xy=k` gives a USD-per-ETH price of `y/x`. Keep the axis and quote-currency convention explicit. The instructor stresses deriving the price, not memorizing a formula (around 32:00).
- Given starting reserves, compute post-trade reserves, output, average execution price, and slippage. For a sale of `q` ETH with input fee rate `f`, under the slide's fee-on-input convention, `q_eff=q(1-f)` and USD output is `y-xy/(x+q_eff)`. Larger trades relative to pool depth generally have larger price impact. Practice the problem on Week 12 slides 39–40.
- Explain capital inefficiency and concentrated liquidity. In `xy=L²`, L measures liquidity depth and adds across LPs active at the same price. **Draw and justify a piecewise AMM curve for overlapping LP price ranges**, including the L in each region. Redo Week 12 slides 46–48 and Week 8 slide 34 (about 37:02–39:05).

### 4. Flash loans and oracle manipulation — slides 49–63; about 39:10–54:05

- Explain the four features: borrowing, operations, and repayment occur in one transaction; callback execution; failure to repay reverts the entire transaction, removing lender default exposure; and no collateral is required. The instructor says platform-specific callback interfaces need not be memorized (about 41:30).
- Give the **full action sequence** for at least one ordinary use. For arbitrage: borrow → buy on the cheaper market → sell on the more expensive market → repay principal and fee → keep the remainder. Also practice flash-loan-funded liquidation and collateral swapping (about 41:50–46:40). Naming a use case alone is insufficient.
- Explain how zero upfront capital, transaction atomicity, lack of identity/collateral checks, and limited loss on failed attempts can amplify attacks. The underlying flaw is commonly a manipulable oracle or another protocol design error.
- **Work through the spot-oracle attack** on slide 63: borrow USDC from a flash-loan pool → buy TOKEN in an AMM to raise its spot price → deposit TOKEN as collateral into a lending pool that relies on that one spot price → borrow too much USDC → repay the flash loan. Be able to explain the three pools, the reserve/price arithmetic, and why the lending pool gets bad debt. Discuss defenses such as a TWAP or the median of independent price feeds (about 48:45–54:05).

### 5. MEV — slides 64–69; about 54:05–67:40

- Distinguish front-running (before the target), back-running (immediately after a state change), sandwiching (buy before and sell after the victim), suppression (delay/exclusion), and the less common time-bandit reorganization. The instructor calls the **sandwich** especially important (about 54:25).
- **Recalculate the sandwich diagram on slide 67.** Begin with approximately `(100 ETH, 100 USD)` in a constant-product pool. The attacker buys 10 ETH, the victim buys 50 ETH, and the attacker sells 10 ETH. Show every reserve change, the victim's extra payment, and the attacker's profit. The slide rounds to **about 39 USD** for both the victim's extra cost and the attacker's profit; exact decimals depend on rounding and fee assumptions. The instructor wants you to explain who gains, who loses, and by how much (about 59:00–63:30).
- Organize defenses by what they disrupt: private/encrypted mempools hide pending transactions; order-fair consensus limits reordering; batch auctions or uniform clearing prices reduce the value of ordering; contract-level access control, profit control, execution splitting, and obfuscation protect particular strategies. **Explain at least two defenses in depth**, including what they prevent and their limitations (about 63:40–67:40).

## Closed-book check

On paper, complete an eight-leaf Merkle update and proof; compare Nakamoto/PBFT in three dimensions; derive CFMM price, swap output, and an overlapping-range curve; narrate a benign flash-loan transaction and the oracle exploit; recalculate the sandwich and explain two MEV defenses. The instructor's expected case-question style rewards the steps and the causal explanation.
