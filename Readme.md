# Othello Kit

A kit for running Othello matches and implementing the alpha-beta pruning algorithm.

## Table of contents

- [Contents](#contents)
- [Requirements](#requirements)
- [Quick start](#quick-start)
- [Usage](#usage)
- [How it works](#how-it-works)
- [Board representation](#board-representation)
- [Coordinate system](#coordinate-system)
- [Testing your agent](#testing-your-agent)
- [Notes](#notes)

## Contents

The kit contains the following files (all `__init__.py` files are omitted):

```text
kit_othello
|-- advsearch
|   |-- othello
|   |   \-- board.py
|   |-- randomplayer
|   |   \-- agent.py       <-- agent that plays randomly
|   |-- timer.py           <-- timing helper functions
|   \-- your_agent         <-- rename this directory to your agent's name (you may add other files here if needed)
|       |-- agent.py       <-- implement make_move here
|-- server.py
\-- test_agent.py          <-- test your agent's basic behavior (you may add other test cases)
```

## Requirements

- Python 3.7+ (the server was developed and tested on GNU/Linux with Python 3.7; other interpreter versions and operating systems may work but are untested)

No third-party packages are required — the kit only uses the standard library.

## Quick start

1. Rename `advsearch/your_agent` to a name of your choosing (e.g. `advsearch/my_agent`).
2. Implement `make_move(board, color)` in that directory's `agent.py`.
3. Run a match against the random player to confirm everything works:

   ```bash
   python server.py advsearch.randomplayer advsearch.randomplayer -d 1 -p 0.3
   ```

   A delay of 1 second is enough because the random player is very fast (and very incompetent). The pace of 0.3 seconds keeps the match easy to follow in the terminal — speed it up or slow it down as needed.

## Usage

```bash
python server.py [-h] [-d delay] [-p pace] [-o output-file] [-l log-history] player1 player2
```

`player1` and `player2` are the dotted module paths (relative to `advsearch`) of the directories where each player's `agent.py` lives, e.g. `advsearch.randomplayer` or `advsearch.my_agent`.

| Flag | Description | Default |
| --- | --- | --- |
| `-h`, `--help` | Show the help message | — |
| `-d delay`, `--delay delay` | Time (seconds) allotted for a player to make a move | `5.0` |
| `-p pace`, `--pace pace` | Minimum time (seconds) the server waits before showing the next move, so fast matches remain easy to follow | `0` |
| `-l log-history`, `--log-history log-history` | Plain-text file where the move log is written | `history.txt` |
| `-o output-file`, `--output-file output-file` | XML file with full match details (includes the move history) | `results.xml` |

The `random` player is located at `advsearch/randomplayer`. To play against it, use `advsearch.randomplayer` as either `player1` or `player2`.

## How it works

Starting with the first player, who plays the black pieces, the server calls `make_move(board, color)` from your `agent.py`:

- `board` — a `Board` object (see `advsearch/othello/board.py`) with the current game state.
- `color` — a character indicating which color should move (`'B'` for black, `'W'` for white).

The server waits up to `delay` seconds and expects the `(x, y)` tuple returned by `make_move`, where `x` is the column and `y` is the row. It then processes the move, prints the new board state, and passes the turn to the opponent — repeating this cycle until the game ends.

A player who fails to return a move in time, or returns an illegal move, five times in a row is disqualified.

At the end of the game, the server prints each player's score and writes:

- the history file (default `history.txt`) with every move attempted, including illegal ones;
- the output file (default `results.xml`) with full match details, such as timing, results, and move history.

## Board representation

A `Board` object's `tiles` attribute holds the board as a list of strings (one per row). `W` represents a white piece, `B` a black piece, and `.` an empty space. Below is the initial state of an Othello game:

```text
[
"........",
"........",
"........",
"...WB...",
"...BW...",
"........",
"........",
"........",
]
```

## Coordinate system

The x-axis grows left to right, and the y-axis grows top to bottom:

```text
  01234567 --> x axis
0 ........
1 ........
2 ........
3 ...WB...
4 ...BW...
5 ........
6 ........
7 ........
|
|
v
y axis
```

> **Important:** don't confuse the coordinate system with matrix indexing. `make_move` must return `(x, y)` — column, then row — while `tiles` is indexed row-first, then column (`tiles[y][x]`).

## Testing your agent

`test_agent.py` includes a couple of sanity checks for your agent (edit the import at the top of the file to point to your renamed agent module, then run):

```bash
python -m unittest test_agent.py
```

Feel free to add more test cases as you develop your algorithm.

## Notes

- The server checks the legality of every move before applying it.
- Illegal or missed (timed-out) moves count against a player; five in a row result in disqualification.
- The `random` player simply picks a random move from the ones that are valid in the current state.
- If you run into problems with the server, report them via Moodle or email.
