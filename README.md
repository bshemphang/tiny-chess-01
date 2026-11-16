[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/bshemphang/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/bshemphang/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# TinyChess 4K

A compact chess engine in a single `main.py` (3,630 bytes), for the
"strongest chess engine in 4 KB of Python" bounty. Python 3.13 standard
library only. No model calls; its only network use is the trial env.

## How to run

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:8080 python3 main.py
```

It plays the trial through the env at `MAKEFUN_ENV_URL`: `POST /game
{}` serves the game in play, `POST /move {"game": n, "move": "<uci>"}` plays a move (the house answers at once with its own), and the loop ends when the env answers status `"done"`. It exits if `MAKEFUN_ENV_URL` is not set. If the env refuses a move (HTTP 400), it falls back to the first move of the env's own `legal_moves` list, and resigns only if that is refused too.

`main.py` is a packed form: the readable source is `src.py` in this folder (uncommitted), packed by the kit's `makefun compress` into the makefun-compress/v1 stub around an LZMA stream of the source. Re-pack after any change with, as a command of its own:

```sh
node <kit>/cli/makefun.mjs compress src.py --out main.py
```

## Design

- **Board:** 0x88 array, signed piece codes, incremental Zobrist hash (fixed-seed LCG, so hashes are deterministic).
- **Search:** negamax with alpha-beta, iterative deepening to fixed depth 6, quiescence on captures (with check evasions and delta pruning), null-move pruning, check extensions, transposition table, killer moves and history heuristic, MVV-LVA ordering.
- **Evaluation:** material plus piece-square tables (midgame and endgame king tables), bishop-pair bonus. No randomness, no wall clock.
- **Determinism:** every move is decided by a fixed depth plus a fixed 180,000-node budget. Worst case observed is about 6 s per move on a shared 1.5-CPU VM, inside the 10 s/move limit.

## Correctness

Move generation was validated with perft against known counts (startpos, Kiwipete, and four tactical positions including en passant, castling and promotions, all exact at depth 3).

## Practice trial, env protocol (2026-11-14, local)

v2 (PVS search, tapered PeSTO-style eval, bishop-pair bonus) ran the packed `main.py` against the kit's env server, practice trial-0, all 8 games, 0 illegal moves: **2 won, 4 drawn, 2 lost, Elo 1396**.

## Limits

- No 50-move or threefold-repetition handling inside the search (the harness adjudicates draws); no opening book, no endgame tablebase.
- Single-threaded; strength is bounded by Python speed at the node budget.
Built for make.fun: https://make.fun/bounty/tiny-chess-01, referral https://make.fun/r/v6wvv0dh
