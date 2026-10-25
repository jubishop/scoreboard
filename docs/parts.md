---
status: current
---

# Parts and accounts

What to buy and which cloud accounts to set up for the
[scoreboard design](design.md). Prices and stock were checked on 2026-10-24
where a source is named; recheck them before ordering. Expect about $400 in
hardware.

## Display

| Item | Qty | Source | Price | Notes |
| --- | --- | --- | --- | --- |
| 64×32 RGB LED matrix, 4 mm pitch | 4 | Adafruit 2278 | $39.95 each | Order all four together for a matched batch |
| Matrix Portal S3 | 1 | Adafruit 5778 | $19.95 | 9 in stock on 2026-10-24; plugs into the first panel's HUB75 input |
| VEML7700 light sensor, STEMMA QT | 1 | Adafruit 4162 | $4.95 | Sees the room through a pinhole in the bezel |
| STEMMA QT cable | 1 | Adafruit | $0.95 (50 mm, 4399) | Choose the length once the pinhole position is known |

## Voice and computer

| Item | Qty | Source | Price | Notes |
| --- | --- | --- | --- | --- |
| Raspberry Pi Zero 2 WH (pre-soldered header) | 1 | Approved reseller | About $20 | Adafruit 6008 was out of stock on 2026-10-24; the header takes the three UART jumpers |
| microSD card, 32 GB, A1 or A2 | 1 | Reputable brand | About $10 | |
| reSpeaker XVF3800 USB 4-Mic Array, without XIAO | 1 | Seeed Studio | $60.99 | Echo cancellation, beamforming, noise suppression, a 5 W amplifier with a JST speaker port, and a 3.5 mm output |
| Enclosed speaker, 3 W 4 Ω | 1 | Adafruit 3351 | $3.95 | Plain wires; crimp the plug that matches the XVF3800 speaker port once its type is confirmed |
| Micro-USB (OTG) to USB-C data cable, short | 1 | Any | About $8 | Connects the Zero's data port to the mic array; a right-angle end keeps the case shallow |
| Female-to-female jumper wires | 3 | Any | About $2 | UART link: TX, RX, and ground |

## Power

The design's [proposed 12 V power](design.md#proposed-implementation) is not
accepted yet. The 5 V alternative is the Adafruit 5 V 10 A supply (658,
$29.95, 74 in stock on 2026-10-24), but its cord must stay short.

| Item | Qty | Source | Price | Notes |
| --- | --- | --- | --- | --- |
| 12 V desktop power brick, UL-listed, 5 A or more | 1 | Mean Well or similar | About $20–30 | Sits at the outlet |
| 12→5 V buck converter, 8 A or more | 1 | Pololu or a potted module | About $15–30 | Inside the case; powers the panels, Portal, and Zero |
| Panel-mount DC jack plus an inline fuse | 1 | Any | About $5 | Rate both for the brick's current |
| 18 AWG silicone wire and lever connectors or a small bus bar | — | Any | About $10 | 5 V distribution inside the case |
| Paintable cord raceway | 1 | Hardware store | About $10 | Only where the cord runs exposed |

## Enclosure and mount

| Item | Qty | Source | Price | Notes |
| --- | --- | --- | --- | --- |
| Chemcast Black LED acrylic, 1/8 in (3 mm), cut to size | 1 | [TAP Plastics](https://www.tapplastics.com/product/plastics/cut_to_size_plastic/black_led_sheet/) | About $20–40 (from $15) | Matte black when unlit, about 29% light transmission. Order after the frame CAD sets the size, and check a sample over a lit panel first, because cut-to-size pieces can't be returned |
| Matte black PLA (louver grids) and PETG (frame and back shell) | — | Already owned | — | Printed on the Bambu Lab X2D (256×256 mm bed) |
| M3 heat-set inserts and screws | — | Any | About $10 | |
| French cleat and wall anchors | 1 | Printed or aluminum | — | |

## Accounts and keys

| Service | Use | Status |
| --- | --- | --- |
| Deepgram | Nova-3 streaming speech to text with keyterms | New account; $200 starting credit |
| ElevenLabs | Text to speech for cached callout clips | New account; expected to cost cents |
| TypeSafe | Jev intent decisions | Existing `JEV_API_KEY` |
| Anthropic | Claude Haiku 5.5 fallback | Existing `ANTHROPIC_LLM_API_KEY` |

Keep the keys only in the Pi Zero's untracked environment file. This
repository is public.
