# RFP — Native Options Trading on Flare

**Covered Calls and Cash-Secured Puts**

*Phase 1: FLR and FXRP · Roadmap: FBTC, sFLR, stXRP*

**Author:** Janus the Watcher ([@XRPWatcherJanus](https://x.com/XRPWatcherJanus))
**Status:** Draft 12 — community review
**Date:** 3 May 2026

---

> This is a request for proposal, not a finished spec. It is intentionally falsifiable. Section 11 lists the scenarios that would kill the project. If they cannot be mitigated, it should not be built.

---

# 1. Executive Summary

Flare has the primitives for a native options market — FAssets, FTSO, FCC, FDC, and a maturing DeFi stack.

Yet covered calls on FXRP exist only on an external chain (Rysk on HyperEVM), and cash-secured puts do not exist at all.

The single existing provider for FXRP options has positioned itself as a B2B vault platform with a $250K minimum allocation. Retail and mid-cap holders have no access. Flare-native holders pay bridging cost and cross-chain risk to participate at all.

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

Mature ecosystems (ETH, BTC, SOL) all have on-chain options venues. Flare does not.

## 2.2 Why CC and CSP belong together

Covered calls and cash-secured puts are mirror strategies.

A CC seller commits underlying as collateral to receive premium for capping upside. A CSP seller commits stablecoin as collateral to receive premium for accepting downside risk at a chosen entry price.

Launching one without the other halves the addressable market and forces every user into one directional posture.

CC alone serves bag-holders. CSP alone serves cash-holders waiting for an entry. Both together serve the full holder lifecycle: accumulation (CSP), holding (CC), and rotation (CC into CSP after assignment).

Default collateral anchor: USDT0. Stable, on-chain, accepted as base unit for CSP collateral and premium settlement on the Phase 1 launch rails.

Settlement currency is a separate axis from the pricing yardstick (Section 6.1.1, Open Question 10.6). Coin-settled rails let holders post collateral and collect premium in FXRP, holding no stablecoin, even when the strike references USD for a clean volatility surface. The market decides which configuration attracts liquidity.

## 2.3 The structural opening

Rysk's contact form (May 2026) requires a $250K minimum allocation and a sales-pipeline conversation to access an options vault.

Their tagline reads "We can support any asset and EVM chain." That is not a competing product. That is a B2B treasury service.

The opening for a Flare-native venue:

|                     |                             |                                    |
| ------------------- | --------------------------- | ---------------------------------- |
| **Dimension**       | **Rysk (HyperEVM)**         | **Flare-Native (RFP Vision)**      |
| **Access**          | Allocation-gated, $250K min | Permissionless                     |
| **Setup**           | Sales pipeline              | Self-serve                         |
| **Asset alignment** | Multi-chain generic         | FAsset-native (FXRP, FBTC) and FLR |
| **Bridging**        | Required                    | None                               |
| **Product scope**   | CC live, CSP "in progress"  | CC + CSP from launch               |
| **Asset roadmap**   | Per sales-call basis        | FLR/FXRP → FBTC → sFLR/stXRP       |
| **Target user**     | Institutional / treasury    | Retail, mid-cap, treasury          |

The argument is not "a better Rysk."

The argument is: Rysk priced itself out of an entire user segment. That segment is concentrated on Flare.

## 2.4 Lot size: the second moat

Traditional equity options trade in standardised hundred-share contracts. That convention sets a hard capital floor on any single position before the first premium clears.

On-chain, a covered call or cash-secured put writes against any fraction of the underlying. The minimum lot is whatever the contract allows, with gas as the only practical floor, not a hundred-unit convention inherited from the trading pit.

This collapses the entry threshold from Rysk's $250K gate to near-dust positions, and expands the addressable market by an order of magnitude.

The unserved segment is not only holders below $250K. It is every position size that standardised contract conventions never accommodated. Fractional lot capability is a structural advantage of on-chain settlement that no wrapped or hosted venue inherits for free.

# 3. Asset Roadmap

The venue's contract architecture is asset-agnostic. Each new underlier is a configuration deployment on the existing rail factory (Section 7.5), not a re-architecture.

Launch scope is bounded by demand, audit cost, and oracle coverage rather than design constraints.

## 3.1 Phase 1 — Launch Assets

  - FLR — Flare-native, FTSO-priced, broadest holder base, no bridging dependency. Anchor for the sFLR roadmap.

  - FXRP — FAsset-wrapped XRP. Largest current bag-holder pool with no native options access. Highest immediate demand signal (per Rysk's gating evidence in Section 2.3).

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

Recommendation for Phase 1: AMM with RFQ overlay for trades above a notional threshold (e.g., $50K).

CLOB deferred until volume justifies professional MM onboarding. This mirrors the path Lyra and Premia took.

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

  - Phase 1 approved lists: yardstick USD (default); settlement currency USDT0 (default) with FXRP enabled as opt-in coin-settled rails. FXRP-as-yardstick for non-XRP underliers (FBTC, FLR) added when their FAsset volume validates.

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

# 8. Curated Vault Layer

A curated vault aggregates depositor capital and deploys it through the venue's primitives under a defined strategy.

The curator — human team, algorithmic engine, or AI agent — handles strike selection, maturity rotation, position sizing, and risk caps.

Depositors get a single entry point and exposure to a chosen strategy. The curator earns a performance fee.

This is currently a heavily contested DeFi category.

Sentora and others are building curated-vault infrastructure with both manual and AI-driven execution.

Morpho's MetaMorpho proved the pattern in lending. Yearn v3 proved it in yield aggregation.

The pattern has not yet been applied to on-chain options at scale. The opening exists.

## 8.1 Why curated vaults belong in this RFP

Four structural problems the venue would otherwise own:

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

|                                     |                 |                                            |
| ----------------------------------- | --------------- | ------------------------------------------ |
| **Item**                            | **Range (USD)** | **Note**                                   |
| **Initial LP for option quoting**   | 1M – 10M        | Bootstrap from incentive pool + private LP |
| **Development (12–18 months)**      | 500K – 2M       | Smart contract + frontend + ops            |
| **Audits (3 rounds)**               | 150K – 600K     | Sequential, not parallel                   |
| **Marketing / GTM**                 | 100K – 500K     | Phase 1 lean, scale post-launch            |
| **Reference vault implementations** | 75K – 200K      | 5 archetypes (Section 8.2)                 |
| **Insurance reserve (optional)**    | TBD             | See Section 10.2 — Sentora                 |
| **Phase 2 increment (FBTC add)**    | 150K – 400K     | Audit + integration only                   |
| **Total ex-LP (Phase 1)**           | 825K – 3.3M     |                                            |

Funding sources to evaluate:

  - Flare Foundation grant (tap into \~20B FLR remaining incentive pool, per @SchwetyBigBags reply on May 3, 2026)

  - Private LP from Flare-native treasuries

  - Optional protocol token launch tied to LP participation

# 10. Open Questions for Builders and Community

This RFP is incomplete by design.

The following questions need community input before a build commits.

## 10.1 Builder selection

Three credible paths. None are exclusive.

10. SparkDEX module: leverage existing perps engine (Eternal), liquidity pools, V4 hooks. Fastest path. Constraint: limited team capacity (per @SchwetyBigBags).

11. Greenfield protocol: new team, purpose-built. Maximum design freedom. Slowest path.

12. Lyra v2 / Premia v3 fork: proven codebase, faster than greenfield, requires Flare adaptation (FTSO integration, USDT0 anchor, FAsset compatibility).

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

15. How are reference vaults (Section 8.2) split across settlement currencies? One "FXRP-Sovereign" archetype is in the table; should there be coin-settled variants of the other archetypes too, or do we let curators self-organise?

Comparable: Deribit runs both stable-margined and coin-margined options on BTC/ETH in parallel, with sophisticated MMs arbitraging between them. This requires institutional MM presence the venue does not have at Phase 1 launch. The decision is whether to wait for that presence or accept thinner cross-numéraire arbitrage initially.

# 11. Failure Modes

Eleven scenarios that would kill this project. Listed in descending order of likelihood.

Mitigations are first drafts, not solutions.

## 11.1 Buyer drought

On-chain options venues consistently fail on buyer flow, not seller capacity.

If buy-side notional stays below 50% of sell-side, LPs become net long the underlying with no premium income.

Mitigation: subsidise buyer fees in Phase 1 from the incentive pool.

Make this a launch precondition, not an afterthought.

## 11.2 LP-token reflexivity

Token-coupled LP design risks death-spiral in bear markets.

Mitigation: hybrid LP with USDT0 tranche (see Section 10.3).

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

# 12. Sources and Context

Original thread:

[Janus thread on options on Flare (May 3, 2026)](https://x.com/XRPWatcherJanus/status/2050813866196488675)

Endorsement and SparkDEX hint:

[@SchwetyBigBags reply](https://x.com/SchwetyBigBags)

Comparable: Rysk Finance on HyperEVM. Contact form (May 2026) requires $250K minimum allocation.
