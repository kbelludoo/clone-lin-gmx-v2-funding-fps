# clone-lin-gmx-v2-funding-fps

Experimental i64-scale LIN clone of GMX V2 `MarketUtils.getNextFundingFactorPerSecond` / `Calc.boundMagnitude` / `Precision.applyFactor`.

**Proofs and evidence live in [kbelludoo/lin-open](https://github.com/kbelludoo/lin-open)** (`src/lin_gmx_v2_funding_fps.lin`, `test/prove_gmx_v2_funding_fps_external.py`, `examples/gmx_v2_funding_fps/`). This repository is the separate LIN copy only.

## Kernel (EXPERIMENTAL, not mainnet uint256, not FLOAT=1e30)

```
FLOAT' = 1e6
static (incFps==0): fps = min(diff * factor / total, max); saved = 0
adaptive increase: nextSaved += sign(skew) * applyFactor(imb, inc) * duration
adaptive decrease: magnitude -= decFps * duration  (to +/-1, not 0)
charged fps = abs(bound(nextSaved, min, max))
stored saved = bound(nextSaved, 0, max)   # min is NOT applied
```

This is **not** V1 `getNextFundingRate` and **not** `getNextFundingAmountPerSize`. DataStore params are arguments. Exponent/pow is an identity skip.

Named contrasts (recomputed in lin-open against C11 `__int128`, limb muldiv, LIN Compiler 0 VM, Python bigint, and libtcc `roundtrip-jit`):

| Path | Result |
|---|---|
| static 600/400 factor 50 | fps `10` |
| static 900/100 max 20 | fps `20` (uncapped `40`) |
| adaptive long-heavy saved 100 dur 5 inc 10 | fps `120` |
| skew flip shorts-heavy saved 100 | fps `80` still longs-pay |
| skew flip saved 10 | saved `-10`, shorts-pay |
| min=10 on saved `-3` | charged fps `10`, saved stays `-3` |
| wrongly min-bind saved | saved `-10` (blocks sign flip) |
| decrease past magnitude | saved `+1` (not `0`) |
| decrease raw vs applyFactor | saved `44` vs `72` |
| naive i64 `7e9*2e9/1e6` | wraps; 128-bit `14000000000000` |

## Upstream

- https://github.com/gmx-io/gmx-synthetics `a85ea3491c19c93bb4b5a002d9b358fb769b7849`
- `contracts/market/MarketUtils.sol` blob `ad7a0b2b5a516ec86ce0afed89ff84ecba11038c`
- `contracts/utils/Precision.sol` blob `047143722beaf6adceac3fc2ef1817f054dfe00a`
- License: GPL-2.0-or-later after BUSL Change Date 2026-08-31

## Files

- `src/lin_gmx_v2_funding_fps.lin` — LIN kernel (`g2r_*`)
- `gmx_v2_funding_fps_c11.c` — independent C11 oracle (selftest)
