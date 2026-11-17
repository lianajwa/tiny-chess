[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/lianajwa/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/lianajwa/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01)

# Tiny Chess

A tiny, original chess engine in one Python file (under 4 KB, stdlib only).

## How it plays

- 0x88 board, pseudo-legal move generation with make/unmake in place and a
  king-safety filter.
- Negamax alpha-beta with principal variation search (PVS: null-window +
  re-search), a transposition table (Zobrist hashing, exact/lower/upper
  bounds, mate-distance scoring), killer + history ordering on top of
  MVV-LVA, narrow-window root search, quiescence over captures and
  promotions, and repetition detection against the full game history.
- A compact 80-slot opening book (two moves per opening per colour, mined
  from the kit's own practice trials at a 2M-node budget and encoded as a
  640-char string) plays the first two moves of each game when legal.
- Fixed node budget N=250,000 per move (deterministic: no wall clock, no
  randomness), ~3.5s per move typical, max 5.4s in practice trials.
- Evaluation: material, a shared centre piece-square table, pawn advancement,
  rook-on-7th, bishop pair, and a tapered king-safety term that rewards king
  centralisation in the endgame and discourages it in the middlegame.
- Promotions are always to queen.

## Protocol

It talks to the make.fun environment over HTTP (`$MAKEFUN_ENV_URL`):
`POST /game` for the next position, `POST /move` to play a UCI move,
`POST /resign` to resign. Runs as `python3 -P -s main.py`.

## Strength approach

Depth comes from the node budget, not the clock, so the same position always
yields the same move. The budget is tuned to roughly 1-3 seconds per move.
