# tessera for jc4827w543

![The Guition JC4827W543 running the demo page: clock, thermostat, cover, media player, light and
robot vacuum tiles on one page of the compact grid](demo-page.jpg)

## Introduction

This is a proof of concept for the Guition JC4827W543 board (4.3-inch 480x272 IPS panel on an
NV3041A over QSPI, GT911 capacitive touch, ESP32-S3 with 4 MB of flash and 8 MB of octal PSRAM)
in [ESP Screens (Tessera)](https://github.com/MaxGramser/homeassistant_espscreen). The board is defined in Tessera's parts style, built on
ESPHome 2026.10.2, and verified on the glass: pages load, tiles respond, the touch matches the
picture, and over-the-air updates run on an encrypted key. The definitions here are meant to be
taken into the Tessera project, and anyone is welcome to pick them up for that.

## Install

Work on the machine that runs Home Assistant, with the ESP Screen Manager add-on installed (its
build runs ESPHome 2026.10.2). The board is on USB for the first flash only.

1. Copy the tree into the ESPHome folder of the add-on (over scp, the Samba share, or a file
   editor): `tessera-board/` and `guition43.yaml` both go to `/config/esphome/`, so the profile
   includes its packages as `tessera-board/...`.
2. Set your secrets:
   - `secrets.yaml` next to the profile with `wifi_ssid` and `wifi_password` (the wifi block reads
     them with `!secret`).
   - The API key sits in `guition43.yaml` under `api: encryption: key:`. Replace
     `REPLACE_WITH_API_KEY_BASE64` with your own base64 key. The bare `encryption:` under `ota:`
     inherits that same key, so one key guards the API and OTA and no secret sits twice.
3. Build the profile. In ESP Screen Manager that is the Firmware page; in an ESPHome dashboard it
   is the usual install dialog. The build is a full ESP-IDF compile and lands at about 1.86 MiB of
   the 1.94 MiB update slot, 95.9 percent (the compact look).
4. Deploy: flash the first firmware over USB from the same page. Every later update goes over the
   air to the screen from the add-on, encrypted with the key from step 2.
5. Add the screen to Home Assistant: Settings, Devices and services, the ESPHome integration, add
   the device by its address, (if asked: paste the API key), and turn on "Allow the device to perform Home
   Assistant actions".
6. First tile: open the screen in ESP Screen Manager, add a tile (a light or a switch is the
   quickest test), and load the page onto the glass.

## Relevant Files

Everything under `tessera-board/` not listed here is copied unchanged from the upstream tree; the
firmware components are byte-identical. These files do not exist in upstream Tessera, or were
changed against it:

| File | Status | What it is |
|---|---|---|
| `guition43.yaml` | new | The screen profile (Tessera ships no profiles; local ones are git-ignored upstream). Secrets sit in `secrets.yaml` and the API key placeholder. |
| `tessera-board/packages/boards/jc4827w543.yaml` | new | The board file: glass facts, the compact look, grid 3x4 lying down and 2x8 standing up, the tile and page ceilings a PSRAM board gets (128 tiles, 24 pages, 12 bar items). |
| `tessera-board/packages/hardware/guition-jc4827w543.yaml` | new | The hardware file: ESP32-S3 with the wide 4 MB partition table, the QSPI panel bus, the GT911 on its own I2C bus with its pixel calibration, the display-level transform, and the backlight PWM. |
| `tessera-board/packages/bridge-s3.yaml` | new | The bridge profile built for the ESP32-S3. Upstream's bridge profile is built for the plain ESP32, and this is the first 4 MB wide-table board that is not one. |
| `boards.json` | changed | The release catalogue with this board added, for patching the add-on's copy in a running install until an official release takes the board. |
| `README-installation.md` | new | The working notes from the build and touch sessions (German). |

## Limits

These are the limits of a 4 MB board on the lite tier, as they stand in this firmware:

- **Nine modules stay out.** Under 8 MB of flash the firmware runs without: the dock (no page
  window from the bottom edge), the vacuum card with map, rooms and routines, cover groups, clock
  faces beyond the digital clock, the doorbell's live camera view, calls between screens, the
  template text card, light presets on light tiles, and the to-do card. Every other card type of
  the catalogue works in every cell of every page.
- **No picture system.** The board has no camera and no snapshot package, so the firmware draws
  no pictures: no picture tiles, no album cover behind the media player card, no artwork on
  favourites. Music Assistant players are driven fully (volume, playback, favourites started by a
  tap), but titles and artists stay text.
- **No audio, no Bluetooth, no battery.** No speaker or microphone hardware, so no sounds and no
  calls; no battery sensor in the top bar.
- **No Wi-Fi hotspot.** 4 MB boards never carry it: the first setup needs the Wi-Fi credentials in
  the YAML before the first flash.
- **Flash is tight.** The standard look does not fit: the best measured combination is 100.2
  percent of the 1.94 MiB update slot. The firmware ships with the compact look at 95.9 percent
  (tiles about 30 by 10 mm, grid 3x4 lying down and 2x8 standing up).
- **Not in the catalogue yet.** Until an official release takes the board, the editor shows it as
  an unknown board and draws its layout from what the screen itself reports. Copying the
  `boards.json` in this repo over the add-on's copy fills the catalogue in a running install; that
  copy is lost on an app update, which is fine once the release carries the board.

## Disclaimer

This README, the board and hardware definitions and the working notes were written with the help
of GLM-5.3, a large language model. Every step was then reviewed by a human and tested: the
firmware was built, flashed and checked on the glass, and the hardware facts behind the
definitions were verified against the ESPHome sources. Still review anything you are about to
flash.
