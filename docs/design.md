---
status: current
---

# Scoreboard design

A voice-controlled ping pong scoreboard that hangs on the wall beside the
table. Players say "Scoreboard, point Sam" (or "undo", "new game", "Sam and
Alex versus Jordan and Casey") and a 128×64 RGB LED matrix updates the score
and shows who is serving. This page records the accepted design decisions,
the proposed implementation, and what is deferred. The [parts list](parts.md)
says what to buy, and the [build plan](build-plan.md) says in what order to
build it.

## Architecture

```text
            ┌──────────── cloud (only after the wake word) ────────────┐
            │  Deepgram Nova-3   →   TypeSafe Jev   →   Claude Haiku   │
            │  speech to text        intent             (fallback)     │
            └──────────────────────────▲───────────────────────────────┘
                                       │ Wi-Fi
reSpeaker XVF3800 ──USB──► Raspberry Pi Zero 2 W ──UART──► Matrix Portal S3 ──HUB75──► 4× 64×32 P4 panels
 4 mics, AEC, amp                 wake word, rules,          DMA refresh, fonts,          (2×2, 512×256 mm)
     │                            roster, game log,          clock, brightness
  speaker ◄── callouts, chimes    callouts                        ▲
                                                           VEML7700 light sensor
```

1. openWakeWord on the Pi Zero hears "Scoreboard", plays a chime, and the
   matrix shows a listening indicator.
2. The command audio streams to Deepgram, biased toward the roster names.
   Local voice activity detection ends the utterance and forces the final
   transcript.
3. One Jev request picks the action and its arguments from closed sets: the
   current players, sides, and command types. A confident answer is applied.
   A low-confidence answer makes the scoreboard ask, for example "Who got that
   point?". An open-ended request, such as a new player's name or a compound
   command, goes to Claude Haiku 5.5.
4. The game engine applies the action, persists it, and sends the new display
   state to the Matrix Portal, which redraws. Callouts play only at key
   moments.

## Accepted decisions

All decisions below were made with the user on 2026-10-24.

### Display

- **HUB75 RGB LED matrix.** The user chose it because "a matrix panel is going
  to look cooler" than a monitor or tablet. The display shows scores, a serve
  indicator, and initials rather than full names, which suits the low
  resolution. Tradeoff: more hardware work than a monitor.
- **128×64 pixels from four 64×32 P4 panels in a 2×2 grid, about
  512×256 mm (20×10 in).** Digits can be about 7 in tall with room for serve
  dots and initials, and each 256×128 mm panel's diffuser grid fits the
  printer's 256×256 mm bed.
- **Adafruit panels (product 2278).** They are the panels the Matrix Portal is
  guaranteed to work with, and one batch keeps the 2×2 seams and colors
  consistent. Tradeoff: about twice the cost of generic panels.
- **Front: a printed black louver grid behind 3 mm smoked acrylic.** Each LED
  reads as a crisp square pixel. When off, the scoreboard is a black slab, and
  the acrylic protects the LEDs from ball strikes.
- **Idle: a dim clock** when no game is in progress.
- **Brightness follows the room** through an ambient light sensor: bright in
  daylight, dim at night, and dark when the room lights are off.

### Placement and power

- **Side wall, centered on the net line, about 6 ft (1.8 m) up.** Both ends see
  it equally, and one mic array on the scoreboard sits about 2–3 m from each
  player.
- **An external power brick and one DC cord** up the wall to a panel-mount
  jack on the case. No mains wiring goes inside the 3D-printed enclosure.
- **12 V power with a converter in the case** (chosen 2026-10-24). A UL-listed
  12 V brick feeds a fused 12→5 V buck converter rated at least 8 A inside
  the case, which powers the panels, the Portal, and the Pi Zero. At 5 V, 3–5 A
  down a 2 m cord sags enough to brown out the Pi Zero, which needs at least
  about 4.65 V. Tradeoffs: one more part ($15–30), some heat in the case, and
  possible switching noise in the audio, so choose a quality, filtered
  converter. The Portal firmware caps total brightness to fit the power budget.

### Voice

- **Cloud speech understanding.** The user was skeptical that an all-local
  approach would be smart enough. Tradeoffs: the scoreboard needs internet,
  each command takes about a second, and it costs a few cents a month.
- **A local wake word before every command: "Scoreboard".** Banter ("that
  should've been my point!") never changes the score, and audio leaves the
  device only after the wake word. Tradeoff: one extra word per point.
- **Speech to text: Deepgram Nova-3 streaming with keyterm prompting.** It was
  chosen over ElevenLabs Scribe v2 Realtime because Deepgram transcribes from
  the first audio frame. ElevenLabs documents that its server buffers the first
  ~2 seconds of audio, which would delay short commands. Deepgram was also the
  cheapest: about $0.37/hour with keyterms (promotional rate on 2026-10-24) and
  a $200 starting credit.
- **Intent: TypeSafe Jev first, with Claude Haiku 5.5 as the fallback.** Jev
  returns typed choices with calibrated confidence, and that confidence decides
  when to ask instead of guessing. Claude (`claude-haiku-5-5`, called with the
  existing Anthropic key) handles what Jev cannot pick from a closed set. The
  Claude API has no audio input, so it cannot replace speech to text.
- **Feedback: chimes plus key callouts.** A chime on the wake word, a flash on
  the scoring side, and spoken callouts only for game point, deuce, the
  winner, and clarifying questions. The callout clips are generated once with
  ElevenLabs text to speech and cached on the device.
- **Voice only, with no phone page or table buttons.** Corrections such as
  "undo" or "that point was Alex's" are voice commands, and an internet
  outage means no scoring.

### Game

- **ITTF rules for singles and doubles:** games to 11, win by 2, serve changes
  every 2 points and every point from 10–10, and the full doubles
  server/receiver rotation.
- **Single games only for now.** Best-of-3, best-of-5, and possibly best-of-7
  matches are planned for later (see [Deferred](#deferred)).
- **A roster plus a game log** persist on the device. Roster names also feed
  the speech-to-text keyterms.

### Computer

- **A Raspberry Pi Zero 2 W for voice and logic, plus an Adafruit Matrix
  Portal S3 for the display, linked by a wired UART.** A Pi 5 is expensive in
  late 2026 (the user cited the AI-driven hardware crunch), and cloud speech
  leaves the Pi little local work: the wake word takes about 5–15% of one Zero
  core. The Portal's ESP32-S3 refreshes the panels by DMA, so the Pi's load
  cannot cause the flicker or black lines that CPU-timed refresh can. The user
  put a clean look ahead of having a single codebase. Tradeoff: two codebases.
  The Portal costs about $3.50 more than the Matrix Bonnet it replaces.
- **Alternatives considered for the Zero's role (2026-10-24).** The Orange Pi
  Zero 2W and Radxa Zero 3W have faster CPUs and more RAM, which this role
  does not need, and their Wi-Fi drivers have recent stability problems. DietPi
  v10.6 (August 2026) fixed Orange Pi Zero 2W instability caused by its
  onboard Wi-Fi driver. The Radxa's AIC8800 Wi-Fi has no mainline driver and
  broke after a DietPi update in December 2025. For a sealed, always-on Wi-Fi
  device, first-party OS and Wi-Fi support matter more.
- **Zero 2 W supply (2026-10-24).** The Zero 2 W was sold out at US online
  retailers (Adafruit, PiShop.us, CanaKit, SparkFun, Vilros, Chicago
  Electronic Distributors), and Micro Center sells it only in its stores,
  none of which is near Seattle. The user backordered the with-header version
  (SC0721) from Digi-Key that day, expected on 2026-11-23. The Pi isn't
  needed until integration, because the display, game engine, and voice
  pipeline can all be prototyped on the Mac first. Proposed fallback if the
  backorder slips past integration: a Raspberry Pi 5 1 GB ($45, in stock at
  PiShop.us that day), in the same role. The Pi 3 A+ and Pi 4 1 GB were also sold out online.

### Repository

- **This public repository, `jubishop/scoreboard`, under the MIT license,**
  created with Project Starter.

## Proposed implementation

These are recommendations that have not been accepted yet. Confirm or replace
each one when the related work starts.

- **Pi Zero software:** a Python asyncio service under systemd with
  openWakeWord (a custom "scoreboard" model trained on synthetic voices), a
  local voice activity detector, the Deepgram, TypeSafe, and Anthropic SDKs,
  and SQLite for the roster, game log, and crash recovery mid-game. The game
  engine is pure Python with unit tests and has no audio dependencies.
- **Portal firmware:** Arduino through PlatformIO with the
  ESP32-HUB75-MatrixPanel-DMA library, using a 2×2 virtual panel layout and
  bitmap fonts. Adafruit Protomatter is the fallback library.
- **UART protocol:** at 3.3 V on both sides, the Zero sends display state
  rather than frames, as JSON lines with sequence numbers and acknowledgments,
  for example `{"seq":41,"score":[7,5],"server":"L","initials":["SB","JO"],"flash":"R"}`.
  The Zero also supplies the time for the clock. If the heartbeat stops, the
  Portal keeps the last score and shows a small offline dot.
- **Firmware updates:** over USB-C during bench work. After assembly, the Zero
  triggers an over-the-air update that the Portal downloads over Wi-Fi, so the
  case never needs opening.
- **Speech-to-text connection:** keep the Deepgram WebSocket open with
  KeepAlive messages during a game, and send audio only after the wake word.
  About 300 ms of local silence triggers `Finalize`.
- **Follow-up window:** after a clarifying question, listen for about 4 seconds
  without needing the wake word.
- **Remote maintenance:** Tailscale on the Pi Zero, as on the existing home
  Raspberry Pi.
- **Latency target:** the display updates within about 1.5 seconds after the
  speaker stops talking. Estimated budget: 0.3 s end-of-speech detection,
  0.1–0.3 s final transcript, and Jev latency (to be measured).

### Commands

Names stand for the current roster; sides are "left" and "right" as seen
facing the wall.

| Say | Effect |
| --- | --- |
| "point Sam", "point left" | Award a point to that player's side |
| "undo", "take that back" | Revert the last action |
| "that point was Alex's" | Undo the last point, then award it to Alex |
| "Sam versus Jordan", "Sam and Alex versus Jordan and Casey" | Set the players for a new game |
| "Sam serves", "Sam serves to Jordan" | Set the first server and, in doubles, the first receiver |
| "new game" | Restart with the same players |
| "switch sides" | Mirror the display after the players change ends |
| "what's the score?" | Speak the score |
| "never mind" | Cancel the current command |

### Display layout

```text
+--------------------------------+--------------------------------+
| (o) SB                         |                             JO |  serve dot and initials
|                                |                                |
|              7                 |               5                |  digits about 44 px tall
|                                |                                |
|            GAME POINT  (banner overlay when it applies)         |
+--------------------------------+--------------------------------+
```

In doubles, the serving side shows the server's initials and the receiving
side shows the receiver's initials, dimmed.

## Open questions

- Jev's latency for this request shape; measure it in the voice phase of the
  [build plan](build-plan.md).
- The connector type of the XVF3800 speaker port; check it when the board
  arrives (see the [parts list](parts.md)).
- Whether the recommended acrylic is bright enough over the louver grid.
  Chemcast Black LED acrylic from TAP Plastics is made for LED pixel displays
  behind printed light-isolation grids: it is matte black when unlit and
  passes about 29% of the light (checked 2026-10-24). Plain smoked gray (for
  example, a 57% grade) is see-through and would show the grid and wiring.
  On 2026-10-24 the user chose to skip TAP's sample, which costs $12.40 with
  shipping. Instead, view a lit panel through ordinary sunglasses, which pass
  about 18–43% of light, before ordering the cut sheet, which cannot be
  returned.

## Deferred

- Best-of-3, best-of-5, and best-of-7 matches with games-won indicators,
  requested by the user on 2026-10-24 for later. Keep the game engine able to
  wrap games in a match.
- A local fast path for "point X" if cloud latency proves annoying.
- Stats and a leaderboard built from the game log.
