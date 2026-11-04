# Malibu Boats Surf Band

Wrist-worn 8-button remote with an LED, used on Malibu and Axis wake boats to
control the surf system, ballast wedge, cruise speed and stereo volume.
The tested unit is model "26DRF8/LR2000" (PCB marked `26DRF8 LR2000 PCB1.0 2016.12.13`).
There appear to be at least 2 models of this remote.

The receiver on the boat is a separate Medallion module, "Surf Band Dongle
Universal", Medallion PN 1539-10855-01, Malibu PN 3962071.1.

FCC ID: [2AFN315RM01](https://fccid.io/2AFN315RM01).
The FCC filing describes a different, ASK-modulated 433.92 MHz "Go Rider
Controller" by Medallion, not this unit.

Photos: [front](front.jpg), [back](back.jpg), [inside front](inside_front.jpg),
[inside back](inside_back.jpg).

## Hardware

Two labelled chips on the board:

```
1026AA
G2D0A
```

```
S14031
BPS1W7
1637
```

The second is a SiLabs [Si4031](https://www.silabs.com/documents/public/data-sheets/Si4030-31-32.pdf)
transmitter, used with its packet handler in the default configuration.

## Signal

- 433.00 MHz (not 433.92 MHz)
- FSK, 1200 baud (833 us per bit), NRZ, no Manchester coding or whitening
- Very small deviation: the two tones are about 4.1 kHz apart (about +-2 kHz)
- Each frame is about 123 ms long

Frame format (Si443x packet handler defaults):

    PREAMBLE:40b SYNC:2dd4 LEN:08 'B' 'U' BTN BTN 'O' 'N' BTN '\r' CRC:16

- `LEN` is always 0x08, the number of payload bytes that follow
- `BTN` is the button code, sent three times
- `CRC` is CRC-16, poly 0x8005, init 0x0000, over `LEN` and the payload

A brief press sends one frame with the button code, then a Release frame.
Holding a button repeats a Hold frame every 131 ms, followed by a Release frame.
There is no device ID: every remote sends identical frames.

| Code | Button | PCB label |
|------|--------|-----------|
| 0x01 | Surf Left | SW1 |
| 0x07 | Surf Right | SW3 |
| 0x0a | Volume Up | SW4 |
| 0x03 | Volume Down | SW9 |
| 0x02 | Wedge Up | SW5 |
| 0x05 | Wedge Down | SW6 |
| 0x08 | Speed Up | SW7 |
| 0x0b | Speed Down | SW8 |
| 0xaa | Hold (repeat while held) | |
| 0x00 | Release | |

Volume Up/Down follow the speaker icons on the faceplate (SW4 is under the
louder icon). The PCB silkscreen labels them the other way around, `SW4_VOL-`
and `SW9_VOL+`; this has not been checked against the boat.

All frames are listed in `codes_test.txt`.

Flex decoder:

    -X 'n=Surf-Band,m=FSK_PCM,s=833,l=833,r=20000,preamble=2dd408'

## Samples

One brief press per file: the button frame, then the Release frame.

- `sw1_surf_left_433M_250k.cu8`
- `sw3_surf_right_433M_250k.cu8`
- `sw5_wedge_up_433M_250k.cu8`
- `sw7_speed_up_433M_250k.cu8`

Volume Up (SW4) held for about 1 second: the button frame, 6 Hold frames
131 ms apart, then the Release frame.

- `hold_volume_up_433M_250k.cu8`

The hold sample's expected output has only 5 of its 6 Hold frames.
The Hold frame at 0.706 s is detected, but the hysteresis detector slips
one bit after the sync word and the CRC fails.

Captured with a HackRF One at 433.0 MHz, 2 MS/s
(`hackrf_transfer -f 432900000 -s 2000000 -a 0 -l 16 -g 20`),
then shifted to 433.0 MHz, low-pass filtered to 60 kHz, decimated to 250 kS/s
and trimmed to the frames with 50 ms margins.

The deviation is too small for the default FSK pulse detectors.
The samples decode with the hysteresis min/max detector from
[merbanan/rtl_433#3623](https://github.com/merbanan/rtl_433/pull/3623),
see the `demod` file (`-Y hysteresis -Y level=-15`).
