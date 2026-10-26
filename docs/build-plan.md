---
status: current
---

# Build plan

The order in which to build the [scoreboard design](design.md), from bench
prototypes to a wall-mounted unit. Each phase ends with something
demonstrable. Display, game engine, and
voice work can proceed in parallel once parts arrive.

## 1. Accounts and parts

Order everything on the [parts list](parts.md), except the acrylic, which
waits for the frame size. Create the Deepgram and ElevenLabs accounts.

**Done when:** the parts are ordered and every service has a working API key.

## 2. Bench display

Drive one panel from the Matrix Portal S3, then all four in the 2×2 layout.
Build the score layout, fonts, serve indicator, flash and banner animations,
clock mode, and light-sensor brightness. Define the UART protocol and test it
from a computer through a USB serial adapter standing in for the Zero.

**Done when:** a computer can drive every display state over serial with no
visible flicker, including in phone video.

## 3. Game engine

Write the pure Python rules engine, test first: ITTF scoring, deuce, serve
rotation for singles and doubles, undo and corrections, switching sides, and
the roster and game log in SQLite with crash recovery. Add a text-command
simulator for driving it without voice.

**Done when:** tests cover full singles and doubles games, including deuce and
corrections.

## 4. Voice pipeline

On the Pi Zero, capture audio from the XVF3800, train and tune the
"Scoreboard" wake word, stream to Deepgram with roster keyterms, and finalize
on local end of speech. Write the Jev question set and the Claude Haiku
fallback. Build a set of real phrasings recorded at the table to measure
accuracy. Generate and cache the callout clips.

**Done when:** recorded commands resolve correctly on the evaluation set, and
the measured time from end of speech to decision meets the design's latency
target.

## 5. Integration

Connect the Zero to the Portal over UART and run complete games by voice at
the table, with ball noise and banter, from the actual mounting position.

**Done when:** a full doubles game can be scored by voice with only rare
corrections.

## 6. Enclosure

Design the frame in keyed sections, a louver grid for each panel, the back
shell, mic and speaker grilles facing the table, the light-sensor pinhole,
cable strain relief, and the French cleat. Test-fit the prints, then order the
acrylic to the final opening size and assemble. Consult the
[printing repository](../memory/printing-repo.md) before printing, and push
generic printing lessons back to it.

**Done when:** everything fits inside the case, with one DC cord out.

## 7. Install

Mount the scoreboard at the net line, run the cord through the raceway, and set
up auto-start, Tailscale, and over-the-air Portal updates. Then document
deployment in the [development workflow](development-workflow.md).

**Done when:** after a power cycle, the scoreboard comes back to a working clock
and accepts voice commands with no manual steps.
