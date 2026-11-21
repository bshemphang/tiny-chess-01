[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/bshemphang/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/bshemphang/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# TinyChess 4K

A chess engine in a single `main.py` (3,632 bytes), for the
"strongest chess engine in 4 KB of Python" bounty. Python 3.13 standard
library only. No model calls; network use is the trial env.

## How to run

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:8080 python3 main.py
```

It plays the trial through the env at `MAKEFUN_ENV_URL`: `POST /game
{}` serves the game, `POST /move {"game": n, "move": "<uci>"}` plays a move, and the loop ends when the env answers status `"done"`. It exits without `MAKEFUN_ENV_URL`. If the env refuses a move (HTTP 400), it falls back to the first `legal_moves` move, resigning only if refused too.

`main.py` is a packed form: readable source is `src.py` here (uncommitted), packed by `makefun compress` into a makefun-compress/v1 stub around an LZMA stream. Re-pack after any change with:

```sh
node <kit>/cli/makefun.mjs compress src.py --out main.py
```

## Design

- **Board:** 0x88 array, signed piece codes, incremental Zobrist hash (fixed-seed LCG).
- **Search:** negamax with alpha-beta, iterative deepening to a depth cap of 13, quiescence on captures (check evasions, delta pruning), null-move pruning, check extensions, transposition table, killer and history heuristics, MVV-LVA ordering.
- **Evaluation:** material plus piece-square tables (midgame/endgame king tables), bishop-pair bonus, tempo bonus. No randomness, no wall clock.
- **Determinism:** every move is decided by a fixed depth cap plus a 100,000-node budget. Worst case is 5.6 s per move on a shared 1.5-CPU VM, inside the 10 s/move limit.
- The node budget guards forfeits; no wall clock allowed.

## Correctness

Move generation validated with perft against known counts (startpos, Kiwipete, four tactical positions with en passant, castling, promotions, all exact at depth 3).
Perft depth 3 exact: startpos 8,902; Kiwipete 97,862.

## Practice trial, env protocol (2026-11-14, local)

v2 (PVS search, tapered PeSTO-style eval, bishop-pair bonus) ran the packed `main.py` on trial-0, all 8 games, 0 illegal moves: **2 won, 4 drawn, 2 lost, Elo 1396**.

v3 (depth cap 13, 100k-node budget, null-move H>2 guard, tempo +10) ran trial-0, all 8 games, 0 illegal moves, 0 forfeits: **4 won, 1 drawn, 3 lost, Elo 1464** (2026-11-21, local). Worst move 5.6 s.
Worst move 5.6 s on a 1.5-CPU VM; 0 forfeits in 8 games.

## Limits

- No 50-move or threefold-repetition handling in the search (the harness adjudicates draws); no opening book, no endgame tablebase.
- Single-threaded; strength bounded by Python speed at the node budget.
Built for make.fun: https://make.fun/bounty/tiny-chess-01, referral https://make.fun/r/v6wvv0dh
