# TELx Merkl LP Bot — Design Handoff

**Date:** 2026-11-08 (America/New_York)  
**Objective:** Acquire more TEL via concentrated LP on TELx Merkl (Uniswap v4), scored in TEL accumulated (Merkl stream + fees + inventory), not USD.  
**Canary pool:** Base eUSD/TEL (confirmed best of 7 live TELx campaigns).  
**Capital:** ~$2,000. Free-tier / public RPC only.

## How to use this package

1. Start with this index, then `project-context.md` and `preferences.md`.
2. Read strategy docs in the order listed under "Core design".
3. Implementation starts from `build-plan.md` Phase 0 once layout is confirmed.
4. Raw scripts and 30-day replay data live under `internal/` (not required for reading the design).

## Core design (read in order)

| Doc | What it is |
|-----|------------|
| [project-context.md](./project-context.md) | Goals, constraints, TEL unit of account |
| [telx-merkl-reward-rules.md](./telx-merkl-reward-rules.md) | Live Merkl scoring (50/25/25), registry vs legacy |
| [pool-selection.md](./pool-selection.md) | Ranking of all 7 TELx pools; Base eUSD/TEL wins |
| [edge-strategy.md](./edge-strategy.md) | Where edge comes from; core±600 + anchor±2220; no spark |
| [measure-rebalance-mechanics.md](./measure-rebalance-mechanics.md) | Measurement cadence, state machine, PosM multicall, low-RPC |
| [adaptive-width-controller.md](./adaptive-width-controller.md) | Competitor policy: ship fixed ±600/±2220; widen on tight entrant; reshape off |
| [bot-blueprint.md](./bot-blueprint.md) | Module layout, addresses, subscribe pattern |
| [build-plan.md](./build-plan.md) | Phases 0–5, stack, acceptance checklists |
| [deployment-stack-and-wireframe.md](./deployment-stack-and-wireframe.md) | Deploy stack + 5-layer subscribe assurance |

## Supporting

| Doc | What it is |
|-----|------------|
| [design-references.md](./design-references.md) | Patterns from 4 public Uniswap v4 / volume repos |
| [telx-private-repo-findings.md](./telx-private-repo-findings.md) | Public TELx repo findings; private access still blocked |

## Locked decisions

- Pool: Base eUSD/TEL.
- Unit of account: TEL accumulated; Merkl reward stream is the primary term.
- Layout (pending user confirm): $1k core ±600 ticks (inventory-matched re-center after 60 min outside) + $1k anchor ±2220 ticks; pre-subscribed dust staging. Adaptive full width-reshape loses ≈34k TEL/wk vs fixed in 30d replay — keep reshape off; on a ≥$1k tight entrant, widen core toward ±1200–1500 instead of matching.
- Rejected: one-sided A/B/C straddles (net-negative); volume spark (Merkl fees axis is liquidity-share, not volume; blacklist risk).
- RPC: low-RPC profile (~455 req/h), free-tier Alchemy/dRPC/Ankr or publicnode rotation.
- Subscribe: PosM `subscribe(tokenId, 0x19ED…, 0x)` in same multicall as mint; registry `0xcebC…`.

## Open for the next builder

- Confirm core+anchor layout (or wait for adaptive-width controller defaults).
- Alert channel (Telegram / Discord / email).
- GitHub private TELx repos (Cursor App scope or PAT) — optional; public data already sufficient for v1.
- Merkl claim proof endpoint format; PosM version pin.
- Scaffold repo (Phase 0) — no Project repository attached yet.

## Addresses (Base, verified 2026-11-08)

- PoolManager `0x498581fF718922c3f8e6A244956aF099B2652b2b`
- PositionManager `0x7C5f5A4bBd8fD63184577525326123B519429bDc`
- StateView `0xa3c0c9b65bad0b08107aa264b0f3db444b867a71`
- Merkl registry `0xcebC0b00473Ca00e429eF431826d0A32cdc945E1`
- Merkl subscriber `0x19EDFa380ead0Bb26010Ca3d1C7AbC7213938c86`
- Distributor `0x3Ef3D8bA38EBe18DB133cEc108f4D14CE00Dd9Ae`
- eUSD/TEL poolId `0x1266df876a41a4f4250dbfa9887e70f20a40a3ccd802c8d75b51b7fd4eb36982`
- TEL v3 `0x7E13B43065380aCdeC1c2d138c579cbBbafA0731` (18 dec)
- eUSD `0x14913815bCFDE78BAeAd2111F463D038Ac9C2949` (6 dec)

## Internal data (optional for implementers)

- `internal/edge-data/` — 30d log scan, position reconstruction, simulator for eUSD/TEL
- `internal/pool-selection-data/` — all 7 pools
- `internal/adaptive-width-data/` — competitor profiles, objective curves, skew, backtest scripts
- `internal/ref-*.md` — per-repo research briefs
