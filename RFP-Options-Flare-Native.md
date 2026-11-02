# RFP — Native Options Trading on Flare

**Covered Calls and Cash-Secured Puts**

*Phase 1: FLR and FXRP · Roadmap: FBTC, sFLR, stXRP*

**Author:** Janus the Watcher ([@WatcherJanus](https://x.com/WatcherJanus))
**Status:** Draft 20 — community review
**Date:** 2 November 2026 (first published 3 May 2026)

---

> This is a request for proposal, not a finished spec. It is intentionally falsifiable. Section 11 lists the scenarios that would kill the project. If they cannot be mitigated, it should not be built.

---

## Changelog — Draft 20 (2 November 2026)

Six months of market movement have invalidated part of Draft 19. Rather than quietly patch it, the changes are listed here.

  - **Section 2.3 rewritten.** The original structural argument rested on Rysk gating FXRP options behind a $250K minimum. Rysk no longer offers FXRP options, so that argument is void. Derive now lists a full XRP options chain and accepts FXRP as collateral. The opening is narrower than Draft 19 claimed and is restated accordingly.
  - **Section 7.8 added.** The Flare Foundation has published a confidential orderbook reference implementation on Flare Confidential Compute. Evaluated here as a candidate matching and custody layer, with its limitations stated.
  - **Section 11.13 added.** Failure modes specific to a TEE-based confidential venue.
  - **Section 6.2 revised.** Confidential CLOB added as a fourth matching architecture; the Phase 1 recommendation is reopened.
  - **Section 2.3 extended** with Dream, a mobile options brokerage routing by RFQ to institutional makers over Derive Chain — the distribution layer of this proposal's own Section 8 thesis, built by someone else and currently negligible in volume. Section 8.1 corrected accordingly.
  - **Sections 2.4, 9, 10, 11.1 and 12 adjusted** to match.

What has not changed: the product thesis (CC and CSP together, physically covered, FTSO-priced) and the falsifiability standard. What has weakened: the claim that a user segment is locked out. See 2.3.

---

# 1. Executive Summary

Flare has the primitives for a native options market — FAssets, FTSO, FCC, FDC, and a maturing DeFi stack.

Yet no options market exists on Flare itself. An FXRP holder who wants premium income has exactly one route today, and it leads off-chain-of-Flare: post FXRP as collateral on Derive and write options on XRP there, cash-settled in USDC, against a margined position that can be liquidated.

That is a different instrument from the one described here. A covered call in the strict sense — underlier locked, underlier delivered, no margin call possible — does not exist for XRP on any venue. For FLR there is no options market at all. This document argues that the gap is one of product form and location, not of access; Section 2.3 states the case and its limits.

This document lays out the business case for building a native options venue on Flare. The launch is structured around the simultaneous deployment of covered calls (CC) and cash-secured puts (CSP), denominated in USDT0 collateral by default, priced through FTSO data, and architected after the Spectra pattern of permissionless per-maturity rails.

The rail factory pattern (Section 7.5) separates two things most venues fuse: the pricing yardstick and the settlement currency. USD-priced, USDT0-settled is the Phase 1 default. But rails can be coin-settled — collateral and premium in FXRP, no stablecoin held — and FXRP can serve as the pricing yardstick for non-XRP underliers like FBTC. Used that way, FXRP becomes a potential reserve unit of the Flare options economy, not merely a wrapped asset.

Phase 1 covers FLR and FXRP. The contract architecture is asset-agnostic.

Phase 2 extends to FBTC on the same FAsset stack. Phase 3 evaluates yield-bearing underliers (sFLR, stXRP) once Phase 1 LP behaviour validates. Section 3 maps the full roadmap.

Distribution layer: the venue is built as infrastructure for curated vaults (Section 8). Curators — Sentora and others currently building this category — handle strategy execution, attract LP capital at scale, and own the user relationship.

The venue stays neutral and asset-agnostic.

This is a draft for community review. It is intentionally falsifiable. If the failure modes outlined in Section 11 cannot be mitigated, the project should not be built.

# 2. Business Case

## 2.1 Why options matter beyond yield

Spot markets price one variable: direction.

Options price three: direction, time, and implied volatility (IV).

Holders who refuse to sell their underlying have no way to monetise time and volatility on a token-only venue. They lose optionality without compensation.

For FLR and FXRP holders today — and FBTC holders tomorrow — this is not a comfort gap. It is a structural absence in the DeFi stack.

Mature ecosystems (ETH, BTC, SOL) all have on-chain options venues. Flare does not — its holders must bridge to reach one, and what they reach is a margined, cash-settled instrument rather than a covered position (2.3).

## 2.2 Why CC and CSP belong together

Covered calls and cash-secured puts are mirror strategies.

A CC seller commits underlying as collateral to receive premium for capping upside. A CSP seller commits stablecoin as collateral to receive premium for accepting downside risk at a chosen entry price.

Launching one without the other halves the addressable market and forces every user into one directional posture.

CC alone serves bag-holders. CSP alone serves cash-holders waiting for an entry. Both together serve the full holder lifecycle: accumulation (CSP), holding (CC), and rotation (CC into CSP after assignment).

Default collateral anchor: USDT0. Stable, on-chain, accepted as base unit for CSP collateral and premium settlement on the Phase 1 launch rails.

Settlement currency is a separate axis from the pricing yardstick (Section 6.1.1, Open Question 10.6). Coin-settled rails let holders post collateral and collect premium in FXRP, holding no stablecoin, even when the strike references USD for a clean volatility surface. The market decides which configuration attracts liquidity.

## 2.3 The competitive landscape (revised November 2026)

Draft 19 argued against Rysk: one FXRP options provider, on a foreign chain, behind a $250K minimum and a sales call. A locked-out segment, concentrated on Flare. That argument is void, and the honest move is to say so rather than quietly delete it.

### What changed

**Rysk is out.** It no longer offers FXRP options. The offering was never complete — covered calls only, no cash-secured puts — and is currently unavailable. As a competitor for this use case it has left the field. It remains instructive as an architecture: a vault model on HyperEVM built on Opyn/Gamma, epoch-based, with settlement type configurable per vault (its materials describe cash-settled covered calls and physically settled cash-secured puts — settlement is a configuration choice, not a property of the vault model).

**Derive is in.** Derive (formerly Lyra) lists a full XRP options chain: calls and puts, multiple expiries, strikes well either side of spot, permissionless, no minimum. Since August 2026 FXRP is accepted there as collateral. An FXRP holder can post FXRP and write XRP options today, self-custodial, with no sales conversation and no minimum ticket.

Note the construction, because it is not intuitive: **FXRP is collateral there, not underlier.** The contract references XRP and settles in cash. There is no FXRP options product; there is the ability to pledge FXRP against XRP options.

So the gate is gone. No segment is locked out by a threshold. Any business case resting on that premise is dead.

### What the opening actually is

|                       |                              |                                 |
| --------------------- | ---------------------------- | ------------------------------- |
| **Dimension**         | **Derive (Derive Chain)**    | **Flare-Native (RFP Vision)**   |
| **Underlier**         | XRP                          | FXRP and FLR                    |
| **Role of FXRP**      | collateral only              | underlier and collateral        |
| **Access**            | permissionless, no minimum   | permissionless, no minimum      |
| **Bridging**          | required                     | none                            |
| **Settlement**        | cash, European, USDC         | delivery of the underlier       |
| **Collateral**        | margined, marked to market   | locked                          |
| **Liquidation risk**  | yes                          | none (fully covered)            |
| **FLR as underlier**  | not listed                   | yes                             |

Four things survive, and they are narrower than Draft 19 claimed:

1. **Settlement mechanics.** Margined and cash-settled is not the same product as locked and physically delivered. The difference is not the terminal payoff, which is broadly equivalent; it is the path. A margined writer can be liquidated by an intraday move or a volatility spike even when the position would have expired cleanly. A fully covered writer cannot. Whether that distinction is purchase-decisive for the target holder is an open question, not an established fact.
2. **FLR has no options market anywhere.** Not Derive, not Deribit, not CME. Before treating that as an opportunity, consider the likelier cause: a market maker cannot quote what it cannot hedge, and hedging demands a liquid spot or perpetual market in the underlier. This is as much a constraint on the roadmap as an opening.
3. **No bridge.** Real but weak on its own; the bridges work.
4. **Distribution and the curated vault layer** (Section 8). Harder to replicate than a listing, and on current evidence the most defensible of the four.

### A second layer: Dream

Derive is where matching happens. A distribution layer is forming above it, and for this proposal that layer matters more.

Dream is a mobile options brokerage. The user picks an asset, a direction, a strike and an expiry. There is no order book of its own; requests are routed by RFQ to institutional market makers, with settlement running in part over Derive Chain. Its app-store pitch is "leverage without liquidations" and self-custody.

That is close to this document's own argument, shipped by someone else: hide the machinery, spare the holder from becoming an options trader, remove liquidation risk.

Two consequences.

It confirms the layering thesis in Section 8. Matching and distribution separate. Whoever builds the venue need not own the customer relationship, and whoever owns the customer relationship need not build the venue.

And it is the sharpest available test of the strongest claim left in 2.3. DefiLlama shows Dream at roughly $7,900 in fees over thirty days, ranked 69th by TVL, effectively 0% of a $206.8m category (November 2026; the protocol's own numbers and DefiLlama disagree, treat as unverified).

If a finished consumer app with institutional liquidity and precisely the right pitch produces that, then "the product form is wrong, fix the form and demand follows" has a live counterexample. The usual rebuttals remain available — too young, wrong assets, no distribution — but they now have to be demonstrated rather than asserted.

### The uncomfortable evidence

Observation of the Derive XRP chain (August 2026, one expiry, to be re-measured): visible open interest concentrated on a handful of near-the-money strikes at low six-figure notional; the volume column empty; bid and ask sizes uniform across strikes, the signature of a single market maker; implied volatility pinned to a constant on far strikes, i.e. a model value rather than a price.

Three caveats before anyone concludes from that. The market carries Derive's "NEW" tag and is weeks old, so it has had no opportunity to accumulate flow. Derive also operates an RFQ channel, and trades struck there never appear as resting orders — a thin visible book structurally understates activity. And a snapshot of one expiry is not a usage figure; establishing usage requires active addresses and trade history.

What can be stated with a source: across the first nine months of 2026, combined notional across Derive, Aevo and Paradex came to roughly 2.4% of Deribit's options notional, with Derive holding about 88% of recorded on-chain premiums. On-chain options remain a rounding error against the centralised market.

That figure supports two opposite readings, and this document does not pretend to settle it. Either on-chain options are a niche with an entrenched leader, in which case entry is unattractive. Or the category sits at 2.4% penetration and an 88% share of very little is not a moat.

### The revised claim

This RFP no longer claims that a segment is excluded. It claims something weaker and testable: that the form in which the product is currently offered — margined, cash-settled, on a foreign chain, and operationally demanding to run — may not be the form in which holder demand can be met. Four candidate explanations for the thin book follow from that, and they are not mutually exclusive:

  - demand does not exist (the thesis fails);
  - the market is too young to judge (measure it);
  - no one can hedge it, so no one quotes it (Section 6, and fatal for FLR specifically);
  - the route is too long and the instrument too demanding for the target holder (the thesis holds, and the answer is the vault layer in Section 8, not a better order book).

Separating these four is the single most valuable piece of work anyone could do on this proposal. Section 11.1 carries the failure mode if the first explanation is the right one.

One further datum deserves weight: a provider has already withdrawn from this market. Either the product did not pay for him, which is a warning, or it was his particular shape — minimum size and sales-led distribution — which supports the thesis. That is a research question, not a matter of opinion.

## 2.4 Lot size: the second moat

Traditional equity options trade in standardised hundred-share contracts. That convention sets a hard capital floor on any single position before the first premium clears.

On-chain, a covered call or cash-secured put writes against any fraction of the underlying. The minimum lot is whatever the contract allows, with gas as the only practical floor, not a hundred-unit convention inherited from the trading pit.

This collapses the entry threshold to near-dust positions and expands the addressable market by an order of magnitude relative to any venue that inherits the contract-size convention.

Two honest qualifications. Fractional lots are no longer a differentiator against Derive, which also imposes no minimum — this is an advantage over traditional venues and over vault products with allocation floors, not over the one on-chain competitor. And a formal absence of minimum size is not the same as economic access: if a venue charges a fixed fee per newly opened position, small tickets are priced out regardless. Fee structure per position, not stated minimums, is the number that matters. Verify it before relying on this moat.

# 3. Asset Roadmap

The venue's contract architecture is asset-agnostic. Each new underlier is a configuration deployment on the existing rail factory (Section 7.5), not a re-architecture.

Launch scope is bounded by demand, audit cost, and oracle coverage rather than design constraints.

## 3.1 Phase 1 — Launch Assets

  - FLR — Flare-native, FTSO-priced, broadest holder base, no bridging dependency. Anchor for the sFLR roadmap.

  - FXRP — FAsset-wrapped XRP. Largest current bag-holder pool with no native options access. Largest holder base reachable without a new FAsset integration (see Section 2.3 for the revised competitive picture).

## 3.2 Phase 2 — FAsset Extension

FBTC — FAsset-wrapped BTC. Same FAsset infrastructure as FXRP.

Engineering delta is minimal: a new FTSO oracle pair, a new vault deployment via the RailFactory, an audit increment scoped to the asset-specific configuration.

Strategic case: BTC has the deepest TradFi options market and the longest IV history.

BTC holders on-chain are yield-starved — no native staking, lending only.

CC on FBTC turns a static bag into recurring premium income. CSP on FBTC lets cash-holders bid for BTC at a chosen entry without leaving the Flare stack.

Best case: FBTC inherits the BTC institutional demand profile, brings treasury-scale notional.

Worst case: FBTC FAsset volume stays thin, options venue offers no marginal benefit over CEX.

Base case: organic flow from existing FBTC holders, slow ramp, validates after 6 months.

Phase 2 trigger: FBTC FAsset volume and holder concentration on Flare must pass a defined threshold (TBD with foundation data).

## 3.3 Phase 3 — Yield-Bearing Underliers (Conditional)

sFLR — staked FLR, yield-bearing through exchange-rate appreciation.

Two design paths, each with a trade-off:

  - Path A: settle in sFLR. Option strikes track the appreciating sFLR/FLR rate. Richer yield-on-yield strategies. LP infrastructure must absorb duration drift.

  - Path B: settle vs. underlying FLR. sFLR used only as discounted collateral. Cleaner pricing model. Less novel.

Path B is the conservative default. Path A is post-validation territory.

Decision deferred until Phase 1 LP behaviour is observed and stress-tested.

stXRP — staked XRP, or the Flare-native staked-FXRP equivalent if that primitive matures first.

Same architectural question as sFLR: settle in stXRP vs. settle vs. FXRP.

Conditional on the staked product reaching liquidity-depth that justifies an options layer.

## 3.4 What this roadmap is not

This is a sequencing document, not a commitment.

Each phase is gated by the previous phase's volume validation and audit clearance.

If FXRP volume fails to validate Phase 1, FBTC does not launch on schedule. If FBTC validates, sFLR/stXRP get evaluated. Not before.

The benefit of stating the roadmap explicitly: builders architect the contract layer once for asset-extensibility (RailFactory pattern, Section 7.5) instead of refactoring at every phase.

Auditors scope incremental reviews instead of fresh audits per asset. Token-design conversations (Section 10.3) account for multi-asset LP exposure from day one.

# 4. Stakeholder Economics

Every options venue lives or dies by whether each participant has a clean reason to show up.

Below: what each side gains, what each side risks, and where the spread comes from. The mechanics generalise across Phase 1 and roadmap assets.

## 4.1 The CC Seller (covered call writer)

Holds FLR, FXRP (or FBTC in Phase 2). Wants premium income while maintaining exposure.

Sells a call with a strike above current price.

  - Gains: premium credited in the rail's settlement currency (USDT0 by default, FXRP on coin-settled rails — see Section 6.1.1) immediately on writing. Premium scales with IV and time to expiry.

  - Risks: capped upside above strike. If price moves above strike at expiry, underlying is assigned and seller loses upside.

  - Best case: premium collected, expires worthless, repeat next maturity.

  - Base case: premium collected, occasional assignment near strike, slight underperformance vs. pure hold in strong rallies.

  - Worst case: violent rally past strike, underlying assigned at strike, foregone upside larger than cumulative premium.

## 4.2 The CSP Seller (cash-secured put writer)

Holds the settlement currency and wants to accumulate the underlying at a lower entry. USDT0 to accumulate FXRP or FBTC; FXRP itself to accumulate FBTC or FLR on coin-settled rails. Premium income accrues while waiting. (A put on FXRP settled in FXRP is degenerate — you cannot get paid to buy an asset with the same asset — so FXRP-underlying puts settle in USDT0.)

  - Gains: premium credited in the settlement currency (USDT0 default, FXRP on coin-settled rails) immediately on writing. Effectively gets paid to set a limit-buy order.

  - Risks: assigned underlying at strike if price falls below strike at expiry. Locked stablecoin until expiry or close.

  - Best case: premium collected, no assignment, USDT0 freed at expiry, repeat.

  - Base case: occasional assignment at desired entry price, premium reduces effective cost basis.

  - Worst case: deep crash, assigned at strike that is now far above market, immediate paper loss on assignment.

## 4.3 The Buyer (call or put buyer)

Wants leveraged directional or hedging exposure without margin lending.

  - Gains: convex payoff. Defined maximum loss (premium paid). Hedge against crash (puts) or capture upside without full capital (calls).

  - Risks: theta decay. Option expires worthless if underlying does not move sufficiently in the right direction.

  - Why this matters for the venue: buyers pay the premium that funds seller yield. No buyers, no yield. Buyer-side flow is the bottleneck on most on-chain options venues, not seller-side.

## 4.4 The Protocol

Two revenue surfaces:

1.  Trading fees on every option open, close, exercise, and settlement (target range: 5-15 bps of notional).

2.  Spread capture: the protocol's pricing curve quotes a bid-ask around theoretical IV. The spread accrues to the LP pool, with a configurable cut to the protocol treasury (target range: 10-30% of spread).

Token model under discussion (see Section 10): protocol token where holders provide LP, fees distribute pro-rata.

Risk of reflexive coupling addressed in Section 11.

### 4.4.1 Data is the lure, not the toll

The venue generates valuable data — IV surfaces per asset and maturity, Open Interest distribution, put/call flow ratios, realised-vs-implied vol divergence.

Comparable venues monetise this through API tiers and licensing (Deribit, Kaiko).

This RFP recommends the opposite path: publish all derived data openly via free APIs, public dashboards, and an indexable subgraph.

Strategic logic: open data lowers integration cost for curators (Section 8), structured-product builders, and external analysts.

More integrations → more order flow → more spread captured. Revenue compounds at the trading layer, not the data layer.

Caveat: this only works if trading fees and spread capture are sized to be sustainable on their own. Data subsidisation cannot be expected to scale revenue beyond a marginal effect on volume.

Concrete deliverables in Phase 1:

  - Public IV-surface dashboard per asset and maturity

  - Open subgraph (theGraph or equivalent) for any party to query positions, settlements, flow

  - Free WebSocket trade feed for real-time integration

  - Quarterly "State of Flare Options Vol" report — narrative distribution amplifier

## 4.5 The Liquidity Provider

LPs underwrite the option supply. They are the counterparty to every buyer.

  - Gains: share of trading fees + spread, plus net premium income on the pool's net option-writing position.

  - Risks: adverse selection (informed buyers hit the LP at off-market IV), assignment risk on net short positions, impermanent-loss-like exposure during regime shifts.

  - Critical design constraint: LPs need a kill-switch and circuit-breaker on extreme IV moves. Without this, one bad week can wipe a year of yield.

  - Multi-asset implication (post-Phase 1): LP capital writes across FLR, FXRP, FBTC. Cross-asset correlation matters — a BTC crash that drags XRP simultaneously concentrates LP losses. Per-asset isolation pools are an option, with a meta-pool layered on top.

  - Curated-vault implication (Section 8): curators aggregate retail and mid-cap LP into single vault deposits. The venue scales LP depth faster through five curators with $5M each than through five hundred direct LPs.

  - Multi-configuration implication (Section 6.1): LP-vaults are deployed per rail, which means per (underlying, yardstick, settlement-currency). A USDT0-settled LP and an FXRP-settled LP are separate pools. Curators can run cross-rail arbitrage. Liquidity fragments if too many configurations launch at once without buyer flow on each (see Failure Mode 11.11).

# 5. Spread Calculation

The spread is the venue's risk premium.

It compensates LPs for adverse selection and inventory risk.

The construction proposed:

Theoretical mid = Black-Scholes (or a chosen successor model) using FTSO spot for the underlying and a venue-specific IV surface.

Quanto-adjusted pricing: when a rail's settlement currency differs from its pricing yardstick — a USD-priced option settled in FXRP, for example — the payoff needs a quanto adjustment to standard Black-Scholes, because the settlement asset's own USD-volatility shifts the effective payoff. The pricer must implement both plain and quanto modes, selectable per rail at deployment.

The IV surface is sourced from three inputs:

3.  Realised volatility on FTSO price feeds over rolling 7-day, 30-day, and 90-day windows.

4.  Open Interest (OI, the total notional in active outstanding contracts) weighted IV from current options on the venue.

5.  LP-side override: the LP pool can set a floor and ceiling on quoted IV per maturity. Outside that band, no quotes are issued.

Bid = mid − (LP risk premium + protocol spread cut).

Ask = mid + (LP risk premium + protocol spread cut).

Both widen automatically when realised vol deviates more than X sigma from quoted IV.

Falsification: if the spread calculation cannot keep LP losses within configurable bounds during a realistic stress scenario (e.g., XRP −40% in 48h, or BTC −30% with XRP correlated), the design fails.

Stress simulations should be a precondition for mainnet.

# 6. Order Matching

## 6.1 The Spectra pattern: per-rail markets

Each rail is a distinct market, defined by its underlying, pricing yardstick, settlement currency, strike, and maturity.

Rails are deployed as independent contract instances, not entries in a global order book.

Permissionless rail creation: any party can deploy a rail for an approved (underlying, yardstick, settlement-currency, maturity) combination.

The protocol curates allowed maturities, approved underliers, approved yardsticks, and approved settlement currencies. It does not curate deployers.

### 6.1.1 Two axes: pricing yardstick and settlement currency

Most venues fuse two distinct choices into one. Separating them is the design unlock here.

The pricing yardstick is the unit the strike and premium are quoted in. It must differ from the underlying, because an asset has no volatility surface against itself. An FXRP option priced in XRP is degenerate: FXRP tracks XRP one-to-one through the FAsset peg, so the result is a de-peg bet, not a volatility market. FXRP options must be priced against USD, FLR, or FBTC.

The settlement currency is what collateral is posted in and premium is paid in. It is independent of the yardstick. A USD-priced option can settle in FXRP. That is coin-margining, exactly what Deribit runs alongside stable-margined options for BTC and ETH.

On the stablecoin side, USDT0 is the Phase 1 default — live and liquid on Flare today. RLUSD, Ripple's regulated stablecoin, is the ecosystem-aligned alternative: an XRP-ecosystem options venue settling in the XRP ecosystem's own dollar.

The caveat is availability. As of May 2026 RLUSD is live on XRPL and Ethereum and expanding to Ethereum L2s (Base, Optimism, Unichain, Ink) via Wormhole, but not yet on Flare. The architecture lists RLUSD as an approved settlement currency that activates if and when it bridges to Flare. A Flare-native options venue is itself a demand argument for that bridge.

Why this matters: the sovereignty thesis lives on the settlement axis, not the pricing axis. A holder who refuses stablecoin posts FXRP, collects premium in FXRP, and never touches USDT0, even while the strike references USD for a clean surface. No stablecoin is held or settled, only referenced as a measuring stick.

FXRP can also be the yardstick itself, but only for non-XRP underliers. FBTC priced in FXRP, FLR priced in FXRP, FDOGE priced in FXRP: these are genuine volatility markets, because the cross-rate moves. Used this way, FXRP becomes the unit of account of the Flare options economy, the reserve asset other FAssets are quoted against. Every such rail generates FXRP demand and velocity, feeding the fee-burn and FIRE flywheel the FLR thesis depends on.

The protocol does not pick a winner. The rail factory deploys whatever configuration the community asks for, within the curated approved lists.

### 6.1.2 Cadence and fragmentation

Cadence proposal: one maturity per month per rail configuration.

This reduces fragmentation, aligns with TradFi monthly options conventions, and simplifies LP duration management.

Quarterly maturities (3, 6, 9, 12 months) optional after volume validates.

Configuration fragmentation risk is real. Each added yardstick or settlement currency multiplies the rails per underlying and splits per-rail Open Interest. See Failure Mode 11.11 for the mitigation logic.

## 6.2 AMM vs. RFQ vs. CLOB

Three matching architectures, each with a clean fit case:

|                                   |                                                         |                                                           |
| --------------------------------- | ------------------------------------------------------- | --------------------------------------------------------- |
| **Architecture**                  | **Fit**                                                 | **Trade-off**                                             |
| **AMM (Lyra-style)**              | Always-on quoting, retail self-serve, deep tail strikes | LP capital inefficient, vulnerable to toxic flow          |
| **RFQ (Hegic / Premia v3 style)** | Large-size institutional, lower slippage on big tickets | Quote latency, requires market maker presence             |
| **CLOB (Schwety pitch)**          | Tightest spreads at-the-money, professional flow        | Cold-start liquidity, requires active market makers (MMs) |
| **Confidential CLOB (FCC/TEE)**   | Private resting orders, gasless requoting, attested price-time priority | TEE and liveness dependency, no external auditability of the book, risk engine still to build |

Draft 19 recommended an AMM with an RFQ overlay above a notional threshold (e.g. $50K), deferring a CLOB until volume justified professional MM onboarding — the path Lyra and Premia took.

That recommendation is reopened in Draft 20. The fourth row did not exist when it was written. A confidential CLOB addresses the specific reason market makers avoid on-chain options — public resting quotes and the cost of requoting a full surface — rather than working around it. Section 7.8 evaluates the implementation and its limits.

The RFQ-first observation stands regardless of architecture and is worth stating plainly: a request-for-quote channel is the standard answer to a cold start, because a dealer who need not post standing quotes will engage a market that cannot yet support a book. Derive follows exactly this sequence, labelling its youngest markets "RFQ Only" before giving them an order book. Whatever matching layer is chosen, Phase 1 should be able to run RFQ-first.

## 6.3 Liquidity provision: the pool as counterparty

The AMM path runs on a liquidity pool that is the counterparty to every trade. LP economics live here, and they are worth stating as one mechanism rather than scattering them across the document.

  - LPs deposit capital: USDT0 on stable rails, the FAsset itself on coin-settled rails. The pool quotes a bid and an ask around theoretical IV, sourced from FTSO (Section 5).

  - The pool stands opposite both buyers and writers. Its income is the spread and trading fees it captures on flow, plus net premium on whatever position it carries. It holds the net Greek exposure and hedges it dynamically, via SparkDEX Eternal or spot.

  - LPs are paid for risk, not for nothing. Spread and premium compensate adverse selection and inventory exposure. A kill-switch and an IV circuit-breaker cap the downside (Section 4.5).

  - Capital efficiency: the pool should support concentrated, tick-based liquidity (Uniswap v3 / XRPL XLS-Curves style), not uniform constant-product depth. LPs target a price band, yielding far more usable depth per unit deposited. Use structured tick keys, not bitmaps; the Raydium bitmap vulnerability is the cautionary precedent.

Curated vaults (Section 8) layer on top. They aggregate retail and mid-cap LP into single deposits and run defined CC and CSP strategies. The depositor supplies collateral, the curator runs the strategy, premium and fees flow back to the depositor.

The boundary that matters: this mechanism scales the supply of options. It does not create demand.

A pool can write options without limit. If no one buys them, LPs end up holding the underlying with no premium income (Failure Mode 11.1, buyer drought). Ribbon and Thetanuts industrialised option writing without solving demand; oversupply crushed premiums, and depositor yields collapsed.

Two consequences follow. Quote both sides: an AMM that quotes bid and ask earns the spread on any trade, regardless of direction, which decouples LP income from one-directional buyer flow. And pair writing with buyer acquisition: subsidise buyer fees early (Section 11.1) and treat the buy-side flow ratio as a primary health metric.

The LP mechanism makes the supply side elegant and scalable. The binding constraint stays on the demand side.

## 6.4 Trading lifecycle and product phases

These phases describe the trading product's maturity. They are orthogonal to the asset roadmap (Section 3): asset phases add underliers (FBTC, sFLR), product phases add trading sophistication. Both advance independently, each gated by its own validation.

### 6.4.1 Phase 1 — Primary trading

Positions are opened and closed against the venue's liquidity (Section 6.3). A writer exits by buying the position back; a buyer exits by selling it back. Two-sided, pool-mediated. This is the launch scope.

### 6.4.2 Phase 2 — Secondary market for positions

Each open position is represented by a transferable receipt token (ERC-1155, keyed to the rail and strike). A secondary AMM lets holders trade these position tokens peer-to-peer, not just close against the pool.

The use case: a writer sold a covered call Monday; by Wednesday the underlying dropped, theta decayed the option toward worthless, and the writer wants out to free capital for the next trade. Instead of waiting for expiry, they sell the position to someone who wants the cheap downside-tail exposure as a hedge. Time decay (theta) and volatility (vega) become tradeable before expiry. That is what Pendle and Spectra do for yield, applied to options.

Precision: this is option-position tokenisation, not "derivatives of derivatives." The option is already the derivative; Phase 2 makes the position liquid and transferable before expiry.

Honest caveat: a secondary market does not eliminate illiquidity, it relocates it. The secondary AMM needs its own liquidity; if it is thin, the exit is thin. It also compounds buyer drought (Failure Mode 11.1) by requiring flow on both the primary and secondary markets. Phase 2 ships only after Phase 1 primary volume validates.

### 6.4.3 Phase 3 — Structured and second-order (frontier)

Genuine second-order instruments: stripping an option into its two separately tradeable components, time value (theta) and volatility (vega). This is the Pendle/Spectra yield-stripping pattern applied to optionality. Compound options (options on options) sit here too.

This is exotic, hard, and unproven on-chain at scale. It is a research direction, not a commitment. Phase 3 is conditional on a liquid, validated Phase 2, and named here only so the architecture is built to not preclude it.

# 7. Smart Contract Architecture

Five core contract domains. Each independently auditable, each replaceable behind an upgrade path.

The architecture is asset-agnostic by design — adding FBTC, sFLR, or stXRP later is a deployment exercise, not a refactor.

## 7.1 Vault Contracts

  - CCVault: holds underlying collateral (FLR, FXRP, FBTC, ...), issues call options against it, handles assignment by transferring underlying.

  - CSPVault: holds settlement-currency collateral (USDT0 by default, FXRP on coin-settled rails), issues put options against it, handles assignment by buying underlying at strike.

  - LPVault: pools capital from token-holders, writes options on the user's behalf, distributes fees and premium. Multi-asset with optional per-asset isolation pools.

## 7.2 Pricing Engine

  - OptionPricer: deterministic Black-Scholes (or chosen successor) implementation in Solidity. Asset-agnostic — takes spot, strike, IV, time as inputs.

  - IVOracle: aggregates FTSO realised vol, OI-weighted IV, LP overrides into a current IV surface per (underlying, yardstick, maturity). The surface depends on the pricing yardstick, not the settlement currency.

  - All math via fixed-point library (PRBMath or similar). Floating point is not an option.

## 7.3 Strike Settlement

  - StrikeOracle: pulls FTSO TWAP (Time-Weighted Average Price) at expiry timestamp, computes intrinsic value, marks option as in-the-money (ITM) or out-of-the-money (OTM).

  - Use a TWAP window (proposal: 30 minutes) to mitigate single-block oracle manipulation.

  - FDC (Flare Data Connector) as fallback for cross-chain data if FTSO does not cover the needed asset pair.

## 7.4 Margin and Liquidation

  - MarginEngine: enforces collateral ratios. CC requires 1.0x underlying; CSP requires 1.0x stablecoin. No partial collateralisation in Phase 1.

  - Yield-bearing collateral (sFLR, stXRP) requires a discount factor to absorb staking-rate volatility. Phase 3 work.

  - LiquidationEngine: only relevant if a later phase introduces under-collateralised positions. Out of scope for this draft.

## 7.5 Rail Factory

  - RailFactory: deploys new (underlying, yardstick, settlement-currency, strike, maturity) rails on demand. Permissionless within the curated calendar and approved lists.

  - Each rail = (CCVault | CSPVault) + (Pricer reference, quanto mode when settlement currency differs from yardstick) + (Settlement reference) + (underlying config) + (yardstick config) + (settlement-currency config). Composable, upgradable, isolated.

  - Adding FBTC in Phase 2: new underlying config, FTSO pair binding, vault deployment. No changes to OptionPricer core (the quanto-mode toggle handles coin-settled and cross-asset-yardstick rails).

  - Phase 1 approved lists: yardstick USD (default); settlement currency USDT0 (default), FXRP for opt-in coin-settled rails, and RLUSD added once it reaches Flare. FXRP-as-yardstick for non-XRP underliers (FBTC, FLR) added when their FAsset volume validates.

## 7.6 Strategy Vault Interface

  - StrategyVault: a clean ABI (Application Binary Interface) for curators (Section 8) to implement against. Exposes deposit, withdraw, claim, and strategy-execute hooks.

  - Curator-agnostic: any party — human team, algorithmic, AI agent — can implement the interface and ship a vault on top of the venue.

  - Rebate hook: optional protocol-fee rebate on curator-routed flow, configurable per curator whitelist tier.

## 7.7 Audit Pathway

Three independent audits before mainnet, sequential not parallel:

  - One specialist firm (options-specific)

  - One generalist Tier 1 (general DeFi)

  - One community audit competition (Code4rena / Sherlock)

Bug bounty live from testnet onward.

Phase 2/3 asset additions trigger incremental audit scoped to the asset-specific configuration, not full re-audits.

Curator strategy contracts are out of scope for venue audits — curator-side audits are the curator's responsibility.

Audit cost is path-dependent (Section 10.1). A fork of a proven codebase (Lyra v2) inherits audit heritage; the review narrows to the Flare-specific surface, which is cheaper but also where the real risk concentrates — a fork on a new oracle and new collateral is materially different from the audited original. A greenfield build needs the full cycle. Licensing from Derive may include shared audit artifacts that reduce both cost and risk.

## 7.8 Confidential matching: the FCC path

In late 2026 the Flare Foundation published a reference implementation of a confidential orderbook built on Flare Confidential Compute: [flare-foundation/fce-orderbook](https://github.com/flare-foundation/fce-orderbook).

Matching runs inside a TEE. Resting orders exist only in enclave memory — never on-chain, never in a public mempool. An on-chain vault holds the tokens and releases them only against a signature from the TEE, whose code hash is registered on-chain and cannot be swapped silently. It ships with price-time priority matching, deposit and withdrawal custody, a working frontend, a load-testing harness, and XRPL wallet onboarding (Xaman) through Flare Smart Accounts. The repository positions itself explicitly as a template for "exchanges, structured-product vaults, settlement layers."

This document treats it as an evaluated path, not a decision.

### Why it is material for an options venue specifically

**Quote privacy.** The dominant objection a market maker raises to on-chain options is not gas; it is that resting quotes are public and get picked off. A maker quoting a full surface faces expiries times strikes times two sides. On Derive in November 2026 that is five maturities on XRP, nine on HYPE and twelve on BTC; at roughly twenty strikes a side, somewhere between 200 and 480 separate books. Every resting quote visible, every one of them pickable. The maker prices that exposure into the spread or declines to quote at all.

Note the progression. The instrument count more than doubles between a new listing and a major, which means fragmentation scales with success: the better a venue does, the more maturities it lists, and the thinner each individual book becomes. Growth does not dissolve this problem, it compounds it. The only levers are deliberately limiting the surface or cutting the cost of quoting it — both architecture decisions, taken before launch. In-enclave matching removes book-level MEV and front-running on resting orders.

**Requoting cost.** An options maker rewrites quotes across the surface on every meaningful move in spot, because delta hedging requires it. With no per-fill gas and no on-chain settlement round trip, continuous requoting becomes economically possible. On a public chain it is not.

**Market structure.** Price-time priority enforced by attested code approaches the quoting discipline of a regulated venue — the property Section 6 identifies as structurally absent from permissionless venues — without a central operator.

**Onboarding.** XRPL wallets connect directly via Smart Accounts, with a deterministic EVM account derived from the r-address. For a venue whose target user is an XRP holder, that removes a step this document has otherwise treated as unavoidable friction.

**Governance path.** The template comes from the Foundation itself. Section 11.5 treats Foundation gating as a failure mode; a Foundation-published exchange reference changes that calculation.

### What it does not solve

**Liquidity.** Private quoting makes market making cheaper and safer. It does not produce a market maker. Every cold-start consideration in Section 6 stands unchanged.

**Hedgeability.** Unaffected. The FLR constraint named in 2.3 is untouched by matching architecture.

**The risk engine.** The repository provides matching, custody, deposits and withdrawals. It does not provide margin computation, expiry settlement against an oracle, cross-strike netting, or liquidation. For an options venue that is the hard part and the expensive part. Treat FCE as roughly the matching and custody third of the stack, not the stack.

**Ordering.** The documentation is explicit: FCC is fire-and-forget, two direct actions may arrive in either order, and the engine is single-writer per pair. Multi-leg strategies, atomic spreads and cross-instrument risk checks must be designed without ordering assumptions. For an options venue that is a live design constraint, not a footnote.

**Liveness.** A sustained proxy outage halts new trading. Withdrawals remain recoverable through the consensus-signed on-chain instruction path. For instruments that lose time value daily and must be rolled, a trading outage is a real loss.

**Maturity.** This is a reference implementation, not a production venue, and its audit status for this use case is unestablished.

### The trade-off nobody names

Confidentiality and auditability pull against each other.

If resting orders live only in enclave memory, the book cannot be measured from outside. No third party can assess quote quality, maker concentration, or wash trading. Settled positions and vault flows remain on-chain, so the loss is bounded to the order layer — but the on-chain thesis in Section 2.1 leans on public verifiability, and any claim that such a venue is "transparent" would need that qualification attached.

There is an irony worth stating. The property that makes the architecture attractive to market makers is the same property that removes the analyst's ability to verify the market is healthy. A prospective LP or curator may find that a harder trade than the MEV it prevents.

### Open question for builders

What does an options venue require that this template does not provide, and what does supplying it cost?

That question reshapes Section 9. If matching, custody and the frontend are no longer greenfield work, the development range stated there is too high — and the saving should be redirected into the risk engine, which is where options venues actually fail.

# 8. Curated Vault Layer

A curated vault aggregates depositor capital and deploys it through the venue's primitives under a defined strategy.

The curator — human team, algorithmic engine, or AI agent — handles strike selection, maturity rotation, position sizing, and risk caps.

Depositors get a single entry point and exposure to a chosen strategy. The curator earns a performance fee.

This is currently a heavily contested DeFi category.

Sentora and others are building curated-vault infrastructure with both manual and AI-driven execution.

Morpho's MetaMorpho proved the pattern in lending. Yearn v3 proved it in yield aggregation.

The pattern has not yet been applied to on-chain options at scale. The opening exists.

## 8.1 Why curated vaults belong in this RFP

Four structural problems the venue would otherwise own.

Before the list, one correction of emphasis. Draft 19 treated this layer as an advantage the venue could design for. Dream (see 2.3) shows it is also a competitive layer that others occupy independently, and that occupying it is not sufficient on its own — its current volume is negligible. Curation remains the right answer to distribution. It is no longer evidence that distribution is the binding constraint.

6.  Distribution. Most retail and mid-cap users do not want to choose strikes, maturities, or position sizes. They want a yield. A curated vault is the right user-facing product for that segment.

7.  LP recruitment. Curators aggregate LP capital faster than the protocol can recruit individual LPs. Each curator becomes a multi-million-dollar LP relationship managed by a single integration.

8.  Strategy diversity. A single LP pool cannot serve "conservative yield" and "aggressive premium" depositors simultaneously. Curated vaults segment risk appetites without the venue building separate frontends.

9.  Open data integration. The venue's IV-surface and flow data (Section 4.4.1) are published openly. No licensing barrier sits between curators and the data they need to execute strategies — including AI-driven curators who require clean machine-readable feeds.

The venue stays as infrastructure. Curators own the user relationship.

The relationship is symbiotic: more curators means more LP depth, tighter spreads, more curator strategies viable.

## 8.2 Vault archetypes to seed Phase 1

Five candidate strategies.

The venue ships reference implementations, then invites third-party curators to clone, customise, and compete on them.

|                        |                                                                                    |                                  |
| ---------------------- | ---------------------------------------------------------------------------------- | -------------------------------- |
| **Archetype**          | **Strategy**                                                                       | **Target user**                  |
| **Conservative Yield** | OTM CCs only, FXRP/FBTC, monthly maturities                                        | Bag-holders, treasury            |
| **Premium Income**     | Near-the-money CCs + CSPs, balanced delta                                          | Yield-seekers, mid-cap           |
| **Accumulation**       | CSPs only, target entry below spot                                                 | Cash-holders, accumulators       |
| **Collar**             | CC + protective put, defined risk band                                             | Conservative institutions        |
| **Vol-Regime Curated** | Strategy adjusts to IV regime (rule-based or AI)                                   | Sophisticated DeFi users         |
| **FXRP-Sovereign**     | Coin-settled rails: collateral and premium in FXRP, USD strike, no stablecoin held | Stablecoin-skeptical XRP holders |

Reference implementations bootstrap the ecosystem. Third-party curators (Sentora, others) extend it.

## 8.3 Curator-side economics

A curator earns:

  - Performance fee on net positive returns (typical: 10-20% of profit)

  - Management fee on AUM (Assets Under Management, optional, typical: 0-2% annual)

  - Optional: rebate from the venue's protocol fee on curator-routed flow

A curator pays:

  - Venue trading fees on every position

  - Smart contract risk on the curator's own strategy contract

  - Adverse-selection risk (informed flow against the curator's quotes)

## 8.4 AI-curated vaults — the live narrative

Sentora and others are building curated-vault infrastructure where AI agents execute the strategy.

This is currently a heavily marketed narrative in DeFi.

The RFP should not over-index on it. The RFP should also not preclude it.

Architectural requirement: vault contracts must expose a clean strategy interface (Section 7.6) that any curator — human-driven or AI-driven — can implement.

The venue stays neutral on which class of curator dominates. Let the market decide.

What this is not: an endorsement of AI-curated strategies as superior.

The historical record on AI-driven trading on-chain is mixed.

The RFP recognises the category, builds for it, stays neutral on the prediction.

## 8.5 Sentora and Firelight as integrated partners

The Flare ecosystem already has specialised players for the two roles this venue depends on. Sentora builds curation and valuation infrastructure. Firelight runs a coverage protocol.

The venue composes them rather than rebuilding either. Sentora curates the strategy vaults (Sections 8.1 to 8.4). Firelight underwrites the tail-risk insurance layer (Section 10.2).

Keeping the two roles with separate specialists avoids a structural conflict: a curator that insured its own vaults would be marking its own homework. Separation of concerns is the stronger design.

Both relationships are conversations to open in the RFP response phase, not preconditions for the build.

# 9. Capital Requirements

|                                     |                                        |                                            |
| ----------------------------------- | -------------------------------------- | ------------------------------------------ |
| **Item**                            | **Range (USD)**                        | **Note**                                   |
| **Initial LP for option quoting**   | 1M – 10M                               | Bootstrap from incentive pool + private LP |
| **Development (12–18 months)**      | 500K – 2M                              | Smart contract + frontend + ops            |
| **Audits**                          | Fork: \<150K / Greenfield: 150K – 600K | Path-dependent — see 7.7 and 10.1          |
| **Marketing / GTM**                 | 100K – 500K                            | Phase 1 lean, scale post-launch            |
| **Reference vault implementations** | 75K – 200K                             | 5 archetypes (Section 8.2)                 |
| **Insurance reserve (optional)**    | TBD                                    | See Section 10.2 — Sentora                 |
| **Phase 2 increment (FBTC add)**    | 150K – 400K                            | Audit + integration only                   |
| **Total ex-LP (Phase 1)**           | 825K – 3.3M                            | Pre-FCC estimate — see note below          |

**Revision pending (Draft 20).** The development range above assumes matching, custody and frontend are built from scratch. The Flare Foundation's confidential orderbook reference implementation (Section 7.8) supplies all three as working code. If that path is taken, the development line is too high and the audit line changes shape — an attested TEE binary and a vault contract are a different review from a full options protocol.

The saving should not be banked as a lower total. It should be redirected into the margin, settlement and liquidation engine, which the template does not provide and which is where options venues fail. A credible Draft 21 needs a costed answer to one question: what does an options venue require beyond the FCE template, and what does it cost to add? Until that exists, treat the figures above as an upper bound on the matching-and-custody component and an unchanged estimate on the rest.

Funding sources to evaluate:

  - Flare Foundation grant (tap into \~20B FLR remaining incentive pool, per @SchwetyBigBags reply on May 3, 2026)

  - Private LP from Flare-native treasuries

  - Optional protocol token launch tied to LP participation

  - Foundation as anchor LP: deposit capital directly into the LP pool (Section 6.3), earning spread and fees on flow plus net premium on the position. Durable, productive, incentive-aligned. Caveats and trade-offs in the next paragraphs.

Builder-sourced estimate (May 2026): a Flare builder (@flareforward) costed a lean fork-path build at roughly $400K development, audit under $100K, and \~$250K to bootstrap day-1 liquidity. That floor sits well within reach of the incentive pool. The constraint is allocation will, not capital.

A structurally different funding option deserves explicit naming: the Foundation as anchor LP. Rather than distributing rFLR rewards to attract mercenary liquidity, the Foundation deposits capital directly into the LP pool (Section 6.3). The pool earns the spread and trading fees it captures on flow, plus net premium on whatever position it carries. The Foundation's capital becomes a productive asset that compounds, not a cost line that drains.

Two consequences follow. The TVL becomes durable and incentive-aligned: the capital stays because it earns, and it earns insofar as the venue succeeds. And the flow the position generates feeds the same FIRE/MEV buyback the burn thesis depends on, with the fee revenue captured by an aligned holder rather than a mercenary one.

The honest caveats. This puts the Foundation on the principal-risk side of options underwriting (real tail-event exposure, not just dilution). It picks a winner, where rFLR distribution is mechanism-neutral. It raises a conflict of interest (Foundation as governor and LP in the same venue) that requires explicit governance and disclosure. And it does not replicate rFLR's broad onboarding effect, which seeds users and attention a single anchor deposit cannot.

The realistic shape is a hybrid: anchor LP positions in a small number of high-conviction venues, with broad rFLR distribution scaled to the bodies in the room. This is the prescription that follows from the rented-TVL critique (see Janus, "Rented Deflation," 2026): own the productive infrastructure rather than paying mercenaries to rent it. The same litmus applies to both diagnosis and prescription. If the venue requires subsidy to produce its flow, the loop has only relocated.

# 10. Open Questions for Builders and Community

This RFP is incomplete by design.

The following questions need community input before a build commits.

## 10.1 Builder selection

Three credible paths. None are exclusive.

10. SparkDEX module: leverage existing perps engine (Eternal), liquidity pools, V4 hooks. Fastest path. Constraint: limited team capacity (per @SchwetyBigBags).

11. Greenfield protocol: new team, purpose-built. Maximum design freedom. Slowest path.

12. Lyra v2 / Premia v3 fork: proven codebase, faster than greenfield, requires Flare adaptation (FTSO integration, USDT0 anchor, FAsset compatibility). Audit scope narrows to the adaptation surface, materially cheaper than a greenfield multi-audit cycle. Licensing from Derive (vs. open-source fork) may add shared audit artifacts and support.

## 10.2 Insurance layer

Open question: is there a viable insurance product for LP losses on extreme tail events?

Two parties to approach: @sentora as curated-vault partner (Section 8.5), and @firelightfi as insurance underwriter.

Keeping curation and coverage with separate specialists, Sentora for strategy and Firelight for tail-risk underwriting, avoids the conflict of a curator insuring its own book.

Mechanism options regardless: pooled reserve, third-party underwriter, on-chain mutual.

Multi-asset Phase 2/3 increases tail risk concentration. Insurance evaluation should account for cross-asset correlated drawdowns, not single-asset events.

## 10.3 Tokenomics and LP coupling

Proposed: protocol token whose holders provide LP. Fees distribute pro-rata.

The risk is reflexive: token price down → LP TVL (Total Value Locked) down → spreads widen → volume falls → token price down further.

Lyra hit exactly this in 2022.

Mitigation candidates:

  - Hybrid LP: token-LP for the upside-leverage tranche, USDT0-LP for the safe tranche. Decouples spread depth from token price.

  - ve-token model: locked governance + fee share, but LP capital separate.

  - Direct USDT0-only LP, with token used purely for fee discount and governance.

  - Fee-reinforced liquidity (no token): reinvest protocol fees directly into LP depth, as Rysk's ROSF does (announced May 2026, up to $50K/month). Liquidity scales with protocol revenue and is fully decoupled from any token price — the cleanest answer to reflexivity, at the cost of slower initial bootstrap.

## 10.4 Success metrics

Proposed Phase-1 metrics, for community ratification:

  - Open Interest in USD per active maturity per asset (target threshold: TBD)

  - Number of active writers (CC + CSP) per week, per asset

  - LP TVL and 30-day LP yield (net of premium losses)

  - Bid-ask spread in IV-points vs. realised vol benchmark

  - Buy-side flow ratio (buyer notional / seller notional). Below 0.5 sustained = buyer drought, design problem.

  - Phase 2 readiness: per-asset OI distribution, LP capital utilisation across assets.

## 10.5 Compliance and geo-blocking

Options can trigger securities classification in several jurisdictions (CFTC, SEC, MiCA).

Minimum baseline: geo-block US and UK at the frontend, no KYC for permissionless contract access.

Open: legal wrapper for any token issuance, treatment under MiCA Article 4.

## 10.6 Yardstick, settlement currency, and approved-list governance

Phase 1 default is USD-priced, USDT0-settled across all rails. The architecture (Section 6.1.1) separates pricing yardstick from settlement currency and supports coin-settled and FXRP-yardstick rails from day one.

Three open questions for the community:

13. Does Phase 1 launch USDT0-settled only, or also coin-settled (FXRP) rails in parallel? Risk on parallel: liquidity fragmentation (Failure Mode 11.11). Risk on sequential: missing the sovereignty segment at launch.

14. Who governs the approved yardstick and settlement-currency lists? Foundation, DAO vote, builder-team, no-curation? Each path has different latency and capture risk.

15. RLUSD as settlement currency is ecosystem-aligned but not yet on Flare (Ripple's 2026 expansion targets Ethereum L2s, not Flare). Open: launch USDT0-only and add RLUSD when it bridges, or actively make the case to Ripple for an RLUSD-on-Flare deployment, with this venue as the demand driver?

16. How are reference vaults (Section 8.2) split across settlement currencies? One "FXRP-Sovereign" archetype is in the table; should there be coin-settled variants of the other archetypes too, or do we let curators self-organise?

Comparable: Deribit runs both stable-margined and coin-margined options on BTC/ETH in parallel, with sophisticated MMs arbitraging between them. This requires institutional MM presence the venue does not have at Phase 1 launch. The decision is whether to wait for that presence or accept thinner cross-numéraire arbitrage initially.

## 10.7 Confidential matching: build on FCE or not?

Section 7.8 sets out the case and the limits. The decision is not purely technical and should not be made by a builder alone.

Open points:

  - What does the margin, settlement and liquidation layer cost on top of the template, and who has built one before?
  - Is the fire-and-forget ordering model workable for multi-leg strategies and cross-strike risk checks, or does it force a single-pair design that fragments the surface further?
  - Will prospective LPs and curators accept a venue whose order book they cannot independently audit? This is an allocator question, not an engineering one, and it should be asked before the architecture is fixed.
  - What is the fallback if the TEE path proves unacceptable — and can the vault and settlement contracts be designed so that the matching layer remains replaceable?
  - Does building on the Foundation's own reference implementation materially change the grant and governance path described in 11.5?

# 11. Failure Modes

Thirteen scenarios that would kill this project. Listed in descending order of likelihood.

Mitigations are first drafts, not solutions.

## 11.1 Buyer drought

On-chain options venues consistently fail on buyer flow, not seller capacity.

If buy-side notional stays below 50% of sell-side, LPs become net long the underlying with no premium income.

Draft 20 adds evidence and a competitor. The Derive XRP chain is permissionless, has no minimum, accepts FXRP as collateral and presents a holder-facing interface that quotes an annualised return — and its visible book remains thin (2.3). Whatever the explanation, nothing in that picture suggests buyers are queuing.

The competitor also matters: buyers seeking leveraged upside already have perpetual futures on Flare. A long call offers something a perp does not — it cannot be liquidated, and its maximum loss is known at purchase — but it costs time value and expires. The buyer-side pitch has to win that comparison explicitly, and this document has never made it.

Mitigation: subsidise buyer fees in Phase 1 from the incentive pool, and state the call-versus-perp case as a product requirement rather than assuming buyers arrive because sellers do.

Make this a launch precondition, not an afterthought.

## 11.2 LP-token reflexivity

Token-coupled LP design risks death-spiral in bear markets.

Mitigation: hybrid LP with USDT0 tranche, or fee-reinforced liquidity that removes the token from the loop entirely (see Section 10.3).

## 11.3 Adverse selection

Informed flow hits LP quotes at stale IV.

Mitigation: dynamic spread widening when realised vol diverges from quoted IV beyond a threshold.

Auto-pause quotes if FTSO feed quality degrades.

## 11.4 Oracle manipulation at expiry

Single-block FTSO snapshot at expiry is exploitable.

Mitigation: 30-minute TWAP window for strike settlement. Audit FTSO update cadence and validator distribution.

## 11.5 Flare Foundation gating

Per @SchwetyBigBags: proposals not from Hugo struggle for traction.

Mitigation: lead with the FTSO-native infrastructure argument. Make the case structurally, not relationally.

If the Foundation refuses to engage, the project may proceed without grants but loses incentive-pool access.

## 11.6 SparkDEX team capacity

If SparkDEX is the chosen builder and they cannot allocate, the timeline doubles.

Mitigation: parallel evaluation of greenfield team during initial scoping. Do not single-thread on SparkDEX.

## 11.7 Audit-driven delays

Sequential 3-audit pathway adds 3-6 months.

Mitigation: budget the time honestly. Do not promise mainnet dates until Audit 1 completes.

## 11.8 Regulatory shift

MiCA Phase 2 or US enforcement could reclassify on-chain options.

Mitigation: design contracts to be permissionless and non-upgradeable in their core settlement layer; geo-block at frontend; no KYC dependency that could be weaponised.

## 11.9 Multi-asset roadmap fragmentation

Adding FBTC, sFLR, stXRP without volume validation per asset fragments LP capital and dilutes per-asset book depth.

Mitigation: hard volume gates per phase.

No new asset launches without the previous asset clearing defined OI and writer-count thresholds (Section 10.4).

## 11.10 Curator strategy blow-up

A curator with a flawed strategy or a buggy strategy contract blows up depositor capital.

Brand damage flows back to the venue regardless of fault.

Mitigation: curator whitelisting at launch, with permissionless curation as a Phase 2/3 evolution after monitoring tools mature.

Mandatory curator-side audits for whitelisted strategies.

Clear UI labelling that depositors transact with the curator, not the venue.

Falsification: if no whitelisting framework can be designed that doesn't recreate Hugo-style gatekeeping, this entire layer is suspect.

## 11.11 Configuration fragmentation

Parallel rail configurations (USDT0-settled and FXRP-settled, or multiple yardsticks — Section 6.1.1) split per-rail Open Interest. Buyer drought (Failure Mode 11.1) compounds: thin buyer-side flow on each rail produces wide spreads, LPs underwrite at adverse selection, parallel rails enter death-spiral simultaneously.

Mitigation candidates:

  - Phase 1 launch USDT0-settled only. Coin-settled rails enabled architecturally (RailFactory accepts the deployment) but the venue's reference frontend and Curated-Vault archetypes default to USDT0-settled until volume validates.

  - Liquidity-gating: alternative-configuration rails require a minimum LP commitment threshold before they activate. Below threshold, the rails sit deployed but inactive.

  - Cross-rail MM incentives: protocol-fee rebate for market makers who quote multiple settlement currencies for the same (underlying, yardstick, strike, maturity). Bridges the parallel markets without forcing users into one.

Falsification: if cross-rail arbitrage cannot be incentivised cheaply enough to keep parallel configurations liquid, multi-configuration rails are a Phase 2/3 feature, not a Phase 1 feature.

## 11.12 XRPL-native options

XRPL is adding pluggable AMM curves (XLS AMM Swappable Curves, draft May 2026): concentrated liquidity and StableSwap now, a programmable Smart AMM (WASM pools, lifecycle hooks) planned. A mature Smart AMM could host options on XRPL directly, bypassing the Flare-via-FXRP route. This is the most credible long-term threat to the thesis, and dismissing it would be dishonest.

Why it does not break the thesis today:

  - Smart AMM is deferred to a separate, unsupported amendment, dependent on a WASM runtime and module-caching infrastructure the spec itself flags as not yet viable for per-swap execution. Years out, multiple dependencies.

  - The curves shipping now (concentrated liquidity, StableSwap) are spot primitives. A swap curve is not an options venue: no strike, expiry, settlement, or Greeks.

  - XRPL has no native high-frequency oracle. The oracle-cost problem that burdens Ethereum options returns on XRPL. Flare's FTSO is the structural edge XRPL cannot replicate.

  - XRPL transactions are not composable intra-transaction (its own flash-loan resistance). Options need atomic composition: position plus hedge, liquidation plus settlement. EVM-based Flare provides it; XRPL structurally restricts it.

Mitigation: ship while the window is open. The Flare-native advantage is largest now and compresses as XRPL's AMM matures.

Falsification: if XRPL Smart AMM launches with a working high-frequency oracle and composable hooks before a Flare-native venue reaches mainnet, the structural case weakens materially.

## 11.13 Confidential-venue and TEE risk

Applies only if the FCC path in Section 7.8 is taken. It introduces a failure class a pure smart-contract venue does not carry.

**Hardware trust.** If a TEE vendor is compromised or the attestation chain breaks, the code-hash guarantee collapses. The FCC documentation is explicit that this is mitigated operationally — multiple vendors, rotated attestation — and not by the platform itself. That is an honest disclosure and a real residual risk.

**Liveness dependency.** A proxy outage halts trading. Options decay; an inability to roll or close is a loss, not an inconvenience. Withdrawals remain recoverable through the on-chain path, which bounds the damage to trading rather than custody.

**Opacity.** A confidential book cannot be independently audited for quote quality, maker concentration or wash trading. "Trust the attestation" replaces "verify the chain." For an institutional allocator that may be the harder sell, and it is the opposite of the argument this document makes elsewhere for on-chain venues.

**Ordering.** Fire-and-forget delivery with no ordering guarantee is a constraint on every multi-leg product the roadmap might want.

Mitigation: treat FCC as one evaluated path rather than the architecture; keep the matching layer replaceable behind the vault and settlement contracts; require independent attestation review before mainnet; and test allocator appetite for an unauditable book before committing.

Falsification: if the target LP and curator audience will not accept the TEE dependency, the confidential path is not viable regardless of its technical merits — and the venue falls back to a public book with the market-making economics described in Section 6.

# 12. Sources and Context

Original thread:

[Janus thread on options on Flare (May 3, 2026)](https://x.com/WatcherJanus/status/2050813866196488675)

Long-form background:

[The Unbuilt Monopoly](https://janusthewatcher.substack.com/p/the-unbuilt-monopoly) — Janus the Watcher, Substack

Endorsement and SparkDEX hint:

[@SchwetyBigBags reply](https://x.com/SchwetyBigBags)

Comparable, historical: Rysk Finance on HyperEVM. Contact form (May 2026) required a $250K minimum allocation. As of November 2026 the FXRP offering is no longer available; see 2.3. Rysk Premium documentation remains useful on vault architecture and per-vault settlement configuration: [ryskfi.com/premium](https://ryskfi.com/premium/)

Comparable, current: Derive (formerly Lyra). XRP options chain: [app.derive.xyz/trade/options?symbol=XRP](https://app.derive.xyz/trade/options?symbol=XRP). Product and collateral documentation: [docs.derive.xyz/docs/supported-products-1](https://docs.derive.xyz/docs/supported-products-1)

Confidential orderbook reference implementation, Flare Foundation: [github.com/flare-foundation/fce-orderbook](https://github.com/flare-foundation/fce-orderbook). Underlying platform: [Flare Confidential Compute overview](https://dev.flare.network/fcc/overview)

Distribution layer: Dream, a mobile options brokerage routing by RFQ to institutional market makers, settling in part over Derive Chain. [DefiLlama — Dream](https://defillama.com/protocol/dream) · [@dreaming](https://x.com/dreaming)

Market size context: on-chain options premium and notional rankings via [DefiLlama](https://defillama.com/options); the 2.4% on-chain-to-Deribit notional ratio for the first nine months of 2026 via [CryptoRank research](https://cryptorank.io/insights/research/onchain-options-premiums-more-than-tripled-in-2026-as-derive-pulled-ahead)

All market observations in this draft date from August to November 2026 and should be re-verified before being relied upon. The competitive picture has changed twice in six months.
