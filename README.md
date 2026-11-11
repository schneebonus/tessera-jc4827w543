# Tessera for jc4827w543

## Introduction

This board was bought because it is the cheapest way to get a proper 4.3-inch glass into ESP
Screens: a Guition JC4827W543 with a 480x272 IPS panel (NV3041A on a four-line QSPI bus), a GT911
capacitive touch, and an ESP32-S3-WROOM-1 with 4 MB of flash and 8 MB of octal PSRAM. At the time
of buying, ESP Screens (Tessera) had no profile for it, and it was not obvious how a new board is
taken in at all: which file holds the hardware, what a board file may and may not say, and what a
build needs beyond the catalogue.

This repo is the result of working that out. The hardware is defined against ESPHome's own drivers
(the mipi_spi model JC4827W543 and the gt911 touchscreen platform), the board is taken into
Tessera in its parts style (a board file with the glass and the grid, a hardware file with the
chip, the panel bus, the touch and the backlight, and a bridge profile built for the ESP32-S3
instead of the plain ESP32 one the 4 MB boards usually share), and everything is verified on the
glass: pages load, tiles respond, the touch matches the picture, and over-the-air updates run on
an encrypted key.

What this repo publishes:

- `guition43.yaml`: the screen profile you build the firmware from.
- `tessera-board/`: the package tree the profile includes (core, this board's board and hardware
  files, the compact look, the features, the firmware components and the fonts).
- `boards.json`: the add-on catalogue of the release it was built against, plus this board, for
  patching a running install so the editor knows the board before an official release does.
- `README-installation.md`: the detailed install notes from the working sessions (German).

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
   is the usual install dialog. The build is a full ESP-IDF compile and lands at about 1.86 MiB of the
   1.94 MiB update slot, 95.9 percent (the compact look).
4. Deploy: flash the first firmware over USB from the same page. Every later update goes over the
   air to the screen from the add-on, encrypted with the key from step 2.
5. Add the screen to Home Assistant: Settings, Devices and services, the ESPHome integration, add
   the device by its address, paste the API key, and turn on "Allow the device to perform Home
   Assistant actions". Home Assistant leaves that off for every new ESPHome device, and without
   it a tap on a tile does nothing.
6. First tile: open the screen in ESP Screen Manager, add a tile (a light or a switch is the
   quickest test), and load the page onto the glass.

Note on pairing: the profile is local, so the add-on's automatic pairing (which answers Home
Assistant's discovery for screens it built itself) does not step in. Adding the device by hand as
in step 5 is the one manual step.

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
- **The glass reads turned around.** Measured on the glass, the GT911's axes run a half turn
  against the panel's own scan, and a touchscreen-level transform does not reach the build. The
  correction sits at the display level: a transform turns the picture the same half turn, and the
  board ships mounted a half turn from the panel's native direction, so picture and touch agree.
  A device that stands up from this mount still needs a check.
- **Not in the catalogue yet.** Until an official release takes the board, the editor shows it as
  an unknown board and draws its layout from what the screen itself reports. Copying the
  `boards.json` in this repo over the add-on's copy fills the catalogue in a running install; that
  copy is lost on an app update, which is fine once the release carries the board.
