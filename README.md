# RFP — Native Options Trading on Flare

A request for proposal for bringing native covered calls (CC) and cash-secured puts (CSP) to FLR and FXRP, built on Flare itself.

**Author:** Janus the Watcher · [@XRPWatcherJanus](https://x.com/XRPWatcherJanus)<br>
**Status:** Draft 19 — open for community review<br>
**Discussion:** [Substack post](#) *(link added at publication)*

---

## TL;DR

Flare has the primitives for a native options market — FAssets, FTSO, FCC, FDC, and a maturing DeFi stack. Covered calls on FXRP exist only on an external chain (Rysk on HyperEVM); cash-secured puts do not exist at all. The single existing provider gates access behind a $250K minimum allocation.

This RFP lays out the business case, stakeholder economics, spread mechanics, order-matching options, smart-contract architecture, and the curated-vault distribution layer for a Flare-native venue. It is intentionally falsifiable — Section 11 lists the eleven scenarios that would kill the project.

## What's here

| File | Contents |
|---|---|
| [`RFP-Options-Flare-Native.md`](RFP-Options-Flare-Native.md) | Full RFP, readable on GitHub |
| [`RFP-Options-Flare-Native.pdf`](RFP-Options-Flare-Native.pdf) | PDF version for download / print |

## Structure

1. Executive Summary
2. Business Case — why options beyond yield, why CC + CSP, the structural opening
3. Asset Roadmap — FLR/FXRP → FBTC → sFLR/stXRP
4. Stakeholder Economics — sellers, buyers, protocol, LPs
5. Spread Calculation — FTSO-based IV surface, quanto-adjusted pricing
6. Order Matching — Spectra-pattern rails, pricing yardstick vs. settlement currency, AMM/RFQ/CLOB, LP pool mechanism, trading lifecycle (primary / secondary / structured phases)
7. Smart Contract Architecture — vaults, pricer, settlement, rail factory, strategy interface
8. Curated Vault Layer — distribution through human and AI curators (Sentora + Firelight)
9. Capital Requirements
10. Open Questions — builder selection, insurance, tokenomics, metrics, compliance, yardstick/settlement choice
11. Failure Modes — twelve scenarios that would kill the project (incl. XRPL-native competition)
12. Sources

## Contributing

Issues are open for discussion. Pull requests are welcome **as proposals** — they will be reviewed and merged at the author's discretion to keep the document coherent. This is a curated RFP, not a wiki.

## License

[CC BY 4.0](LICENSE) — free to share and adapt with attribution.
