# Chess Assistant

On-device chess move assistant for the Chess.com iOS app. Get best-move suggestions from a bundled **Stockfish 18** engine, or human-like moves from the **Maia 3** neural network — drawn as arrows right on the board.

Everything runs locally on the device. No network requests, no move data leaves your phone.

## Features

- **Engine suggestions** — Stockfish 18 (statically linked, NNUE nets embedded) evaluates positions and draws the best-move arrow directly on the board
- **Maia (human-like)** — switch to the Maia 3 CoreML model for human-like moves at ~1300–1900 ELO instead of engine-perfect play
- **Adjustable strength** — ELO slider from 400 to 3500 (mapped to search depth + skill level / UCI_LimitStrength)
- **Multi-line arrows** — show up to 3 candidate moves with per-move eval labels
- **Eval display** — centipawn or win-% evaluation on the floating button
- **Move Analysis** — grades each of your moves (Brilliant → Blunder), tracks accuracy % and a full breakdown in the settings panel
- **Auto Play** — automatically plays the engine's best move after your opponent moves
  - Adjustable move delay (0–5s) with random ±1s jitter so moves look human
  - ~10% chance to play the second-best move for extra realism
  - Puzzle auto-solve support (always plays the correct solution)
- **Quick pause** — long-press the floating ♟ button to instantly pause/resume the assistant
- **Opening names** 📖 — live detection of ~40 openings (Ruy Lopez, Najdorf, Caro-Kann, Queen's Gambit…) shown in the panel
- **Copy FEN** — one tap copies the current position to your clipboard
- **Auto stats reset** — accuracy stats, move grades and opening tracking reset automatically when a game ends (checkmate / stalemate detected on-device)
- **Arrow styling** — color by eval / solid green, thin–thick shafts, adjustable opacity
- **Ban-safe preset** — ⚡ one-tap button that applies safe settings (ELO 1000, Maia on, Auto Play off)
- Works in online games, bot games, and puzzles

## Install

### Jailbroken (rootless)

Add the repo to Sileo/Zebra and install:

```
https://itzzace.github.io/chess-assistant/
```

Respring after install, then open Chess.com.

### TrollStore (no jailbreak)

Grab the iOS 16 / 17 IPA from the [releases page](https://github.com/itzzace/chess-assistant/releases) and sideload with TrollStore.

## Usage

1. Open Chess.com and start any game.
2. Tap the floating ♟ button to open the settings panel (long-press to pause/resume instantly).
3. Set your ELO, toggle Maia / Auto Play / Move Analysis as desired — or hit ⚡ for the ban-safe preset.
4. Arrows appear on the board when it's your turn; enable Auto Play and they'll be played for you.

Tap **Debug Log** in the panel to view recent events (FEN detection, engine responses, hooks installed). Tap **📋 FEN** to copy the current position.

> ⚠️ Using an assistant violates Chess.com's fair-play policy and can get your account banned. Higher strengths (>~1500 ELO) are much easier to detect.

## Building

Requires [Theos](https://theos.dev) with the iOS SDK:

```bash
# clone Stockfish and fetch its NNUE nets
git clone --depth 1 --branch sf_18 https://github.com/official-stockfish/Stockfish.git sf
cd sf/src && make net && cd ../..

# cross-compile Stockfish for iOS arm64 into a static lib
cd sf/src
SDK=$(xcrun -sdk iphoneos --show-sdk-path)
CXX=$(xcrun -sdk iphoneos -f clang++)
for f in $(find . -name '*.cpp'); do
  $CXX -arch arm64 -isysroot "$SDK" -miphoneos-version-min=15.0 \
       -std=c++17 -O3 -DNDEBUG -fno-exceptions \
       -DUSE_PTHREADS -DIS_64BIT -DUSE_POPCNT -DUSE_NEON=8 -I. -w \
       -c "$f" -o "$f.o"
done
ar rcs libstockfish.a $(find . -name '*.cpp.o' ! -name 'main.cpp.o')
cd ../..

# build the deb (Maia model is fetched separately)
make package FINALPACKAGE=1
```

CI does all of this automatically — see [.github/workflows/build.yml](.github/workflows/build.yml). The Maia 3 CoreML model (`maia3_5m.mlpackage`) is downloaded from the repo's `maia-model` release during packaging and installed to `/var/jb/Library/Application Support/Chess/`.

## Project layout

| File | Purpose |
|------|---------|
| `Tweak.xm` | Main tweak: board/FEN detection, arrow rendering, settings UI, auto play |
| `engine.mm` | Stockfish embedded via UCI stream redirection (`EngineGo`, FEN legality, legal-move enumeration) |
| `maia.mm` | Maia 3 CoreML inference wrapper |
| `web/` | Landing page / repo site |
| `Makefile` | Theos tweak build (`com.chess.assistant`, rootless) |
| `.github/workflows/` | CI: builds libstockfish.a + .deb, publishes releases & apt repo |

## Credits

- Made by **@epicccccc** ([YouTube](https://youtube.com/@epicccccc), Discord: `itzzace.`)
- **Yousseif** ([GitHub: usif-x](https://github.com/usif-x)) — Stockfish 18 upgrade, Auto Play (with human-like timing & puzzle solving), quick pause, opening detection, Copy FEN, auto stats reset & tweak updates
- [Stockfish](https://stockfishchess.org) — GPL v3 chess engine
- [Maia 3](https://github.com/CSSLab/maia3) — human-like chess AI

Licensed under GPL v3 where applicable due to Stockfish linkage.
