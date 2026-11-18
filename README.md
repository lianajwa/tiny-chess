[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/lianajwa/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/lianajwa/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01)

# Tiny Chess

A tiny, original chess engine in one Python file (under 4 KB, stdlib only).

## How it plays

- 0x88 board, pseudo-legal move generation with make/unmake in place and a
  king-safety filter.
- Negamax alpha-beta with principal variation search (PVS: null-window +
  re-search), null-move pruning (R=3, never in check), a transposition
  table (Zobrist hashing, exact/lower/upper bounds, mate-distance scoring),
  killer + history ordering on top of MVV-LVA, narrow-window root search,
  quiescence over captures and promotions, and repetition detection against
  the full game history.
- A compact 80-slot opening book (two moves per opening per colour, mined
  from the kit's own practice trials at a 2M-node budget and encoded as a
  640-char string) plays the first two moves of each game when legal.
- Fixed node budget N=250,000 per move (deterministic: no wall clock, no
  randomness), ~5s per move typical, max under the 10s/move limit in practice
  trials (no time losses).
- Evaluation: material plus a tapered PeSTO-style evaluation — separate
  middlegame/endgame piece-square tables per piece (hand-designed, 384 values
  packed into two 192-char strings), interpolated by game phase, plus bishop
  pair. Round 5 practice trial-0: Elo 1654, 5W-0D-1L, spare 539 plies.
- Promotions are always to queen.

## Protocol

It talks to the make.fun environment over HTTP (`$MAKEFUN_ENV_URL`):
`POST /game` for the next position, `POST /move` to play a UCI move,
`POST /resign` to resign. Runs as `python3 -P -s main.py`.

## Strength approach

Depth comes from the node budget, not the clock, so the same position always
yields the same move. The budget is tuned to roughly 1-3 seconds per move.
