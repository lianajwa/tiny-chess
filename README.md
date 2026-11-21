[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/lianajwa/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/lianajwa/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01)

# Tiny Chess

A tiny, original chess engine in one Python file (under 4 KB, stdlib only).

## How it plays

- 0x88 board, pseudo-legal move generation with make/unmake in place and a
  king-safety filter.
- Negamax alpha-beta with principal variation search (PVS: null-window +
  re-search), null-move pruning (R=3, never in check), late-move reductions
  for quiet moves, check extensions, mate-distance pruning, a transposition
  table (Zobrist hashing, exact/lower/upper bounds, mate-distance scoring),
  killer + history ordering on top of MVV-LVA, narrow-window root search,
  quiescence over captures and promotions, and repetition detection against
  the full game history. The transposition table, killers and history are
  cleared between games, so one game cannot steer the next.
- Two books, checked in order. (1) A compact 12-line replay book (verified
  winning lines mined from the kit's own practice trials, plus trap-sweep
  wins at 25k/100k nodes, e.g. italian O-O mates in 79 plies, morphy c6b4
  mates in 62, scandinavian c8d7 mates in 94, philidor b7b5 mates in 48,
  dutch line back) replays a recorded winning line when the game's opening,
  opponent and colour match and our moves so far follow it exactly. Long
  lines are truncated at positions the engine's search verifies as clearly
  winning, so the search finishes them. (2) A compact 80-slot opening book
  (two moves per opening per colour, mined the same way) plays the first two
  moves of each game when legal; the sicilian-classical black slot was fixed
  (e7e5/d7d6 — the old b7c6 is illegal against the real opening). Both books
  are stored 2-chars-per-move packed; the opening slots are indexed by
  opening-hash rank.
- Fixed node budget N=250,000 per move (deterministic: no wall clock, no
  randomness), ~5s per move typical, under the 10s/move limit in practice
  trials when run uncontended.
- Evaluation: material plus a tapered PeSTO-style evaluation — separate
  middlegame/endgame piece-square tables per piece (hand-designed, 384 values
  packed into two 192-char strings), interpolated by game phase, plus bishop
  pair. Practice trials: v7 baseline trial-0 Elo 2112, trial-1 Elo 1545,
  trial-2 Elo 2112, mean Elo 1923, spare 619.
- Promotions are always to queen.

## Protocol

It talks to the make.fun environment over HTTP (`$MAKEFUN_ENV_URL`):
`POST /game` for the next position, `POST /move` to play a UCI move,
`POST /resign` to resign. Runs as `python3 -P -s main.py`.

## Strength approach

Depth comes from the node budget, not the clock, so the same position always
yields the same move.
