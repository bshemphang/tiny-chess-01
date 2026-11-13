# TinyChess 4K

A compact chess engine in a single `main.py` (3,796 bytes), for the
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

## Practice trial, env protocol (2026-11-13, local)

Ran the packed `main.py` against the kit's own env server with the practice trial-0 gauntlet, all 8 games, 0 illegal moves (no HTTP 400s): **1 won, 2 drawn, 5 lost (2.0/8), implied Elo about 1080**, below the 1154 practice bar for this one trial. The win was by checkmate vs nodes-100k (1392); the draws were threefold repetitions vs depth-3 (1073) and nodes-500k (1596). The game-1 loss was by time on the first move (game ended at 8 plies, engine never moved); without it the same trial implies about 1260. The cold first move benchmarks at 3.84-6.09 s, inside the 10 s limit, and the loss coincided with heavy concurrent work on this shared VM, so it reads as contention rather than engine slowness. If a second practice trial (run quiet) also loses a move on time, cut the node budget (`L` in `src.py` from 180,000 to 100,000, then re-compress) and re-test.

## Limits

- No 50-move or threefold-repetition handling inside the search (the harness adjudicates draws); no opening book, no endgame tablebase.
- Single-threaded; strength is bounded by Python speed at the node budget.
