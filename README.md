# RFP — Native Options Trading on Flare

A request for proposal for bringing native covered calls (CC) and cash-secured puts (CSP) to FLR and FXRP, built on Flare itself.

**Author:** Janus the Watcher · [@WatcherJanus](https://x.com/WatcherJanus)<br>
**Status:** Draft 20 — open for community review<br>
**Discussion:** [The Unbuilt Monopoly](https://janusthewatcher.substack.com/p/the-unbuilt-monopoly) on Substack

---

## TL;DR

Flare has the primitives for a native options market — FAssets, FTSO, FCC, FDC, and a maturing DeFi stack. It has no options market of its own. An FXRP holder who wants premium income has one route today: bridge to Derive, post FXRP as collateral, and write cash-settled options on XRP against a margined position that can be liquidated.

A covered call in the strict sense — underlier locked, underlier delivered, no margin call possible — does not exist for XRP anywhere. For FLR there is no options market at all.

This RFP lays out the business case, stakeholder economics, spread mechanics, order-matching options, smart-contract architecture, and the curated-vault distribution layer for a Flare-native venue. It is intentionally falsifiable — Section 11 lists the thirteen scenarios that would kill the project.

**Draft 20 (November 2026)** revises the competitive case rather than defending it. Rysk has withdrawn from FXRP options, so the "locked-out segment" argument of earlier drafts is void and has been removed. Derive now accepts FXRP as collateral. The Flare Foundation has published a confidential orderbook reference implementation on FCC, evaluated here as a candidate matching layer. And Dream — a mobile options brokerage routing by RFQ over Derive Chain — now occupies the distribution layer this proposal argues for, at negligible volume. See the changelog at the top of the RFP.

## What's here

| File | Contents |
|---|---|
| [`RFP-Options-Flare-Native.md`](RFP-Options-Flare-Native.md) | Full RFP, readable on GitHub |
| [`RFP-Options-Flare-Native.pdf`](RFP-Options-Flare-Native.pdf) | PDF version for download / print |

## Structure

1. Executive Summary
2. Business Case — why options beyond yield, why CC + CSP, the competitive landscape (revised Nov 2026)
3. Asset Roadmap — FLR/FXRP → FBTC → sFLR/stXRP
4. Stakeholder Economics — sellers, buyers, protocol, LPs
5. Spread Calculation — FTSO-based IV surface, quanto-adjusted pricing
6. Order Matching — Spectra-pattern rails, pricing yardstick vs. settlement currency, AMM/RFQ/CLOB, LP pool mechanism, trading lifecycle (primary / secondary / structured phases)
7. Smart Contract Architecture — vaults, pricer, settlement, rail factory, strategy interface, confidential matching on FCC
8. Curated Vault Layer — distribution through human and AI curators (Sentora + Firelight)
9. Capital Requirements
10. Open Questions — builder selection, insurance, tokenomics, metrics, compliance, yardstick/settlement choice, confidential matching
11. Failure Modes — thirteen scenarios that would kill the project (incl. XRPL-native competition and TEE risk)
12. Sources

## Contributing

Issues are open for discussion. Pull requests are welcome **as proposals** — they will be reviewed and merged at the author's discretion to keep the document coherent. This is a curated RFP, not a wiki.

## License

[CC BY 4.0](LICENSE) — free to share and adapt with attribution.
