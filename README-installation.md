# Guition JC4827W543 in einer laufenden Tessera-Instanz, ganz ohne GitHub

Der Ordner enthält alles, was ein Build des Boards braucht, und holt nichts aus dem Netz (nur die
ESPHome-Werkzeuge selbst laden ihre Plattform-Pakete von pypi, wie bei jedem Build):

    guition43.yaml          das Profil des Screens (gehört direkt in den ESPHome-Ordner der Instanz)
    tessera-board/          packages/, components/ und fonts/ der Boardfiles (Kopie des Branches
                            jc4827w543, Commit d0e8d06, Basis dev)

## 1. Dateien einfügen

Der ESPHome-Ordner der Tessera-Instanz ist bei Home Assistant OS der von der App gemountete
`/config/esphome` (derselbe Ordner, den das ESPHome Device Builder Add-on nutzt; bei einer
Docker-Installation der Ordner, den du als ESPHome-Ordner gemountet hast). Reinkopieren:

- `guition43.yaml` direkt in den ESPHome-Ordner (er muss auf der obersten Ebene liegen, damit
  Tessera das Profil in seiner Liste führt),
- den Ordner `tessera-board/` daneben, also ebenfalls in den ESPHome-Ordner (die Includes im
  Profil sind relativ zum Profil: `tessera-board/...`).

Existiert der Ordner noch nicht (noch kein Screen installiert), lege ihn an. Auf den Rechner
kommt der Inhalt zum Beispiel über das Samba- oder Studio-Code-Server-Add-on, oder per scp.

## 2. Schlüssel eintragen

In `guition43.yaml` den einen Platzhalter ersetzen:

    python3 -c "import base64,os;print(base64.b64encode(os.urandom(32)).decode())"   # API-Key

Er schützt die API und wird vom `ota:`-Block übernommen (`encryption:` ohne Schlüssel, ESPHome
2026.9+; ein eigener Schlüssel ginge mit `encryption: {key: "..."}`). Ein OTA-Passwort gibt es
nicht mehr; `password:` und `encryption:` dürfen nicht gemischt werden.

Das WLAN kommt aus `secrets.yaml` im selben Ordner. Hat die Instanz noch keins, leg es an:

    wifi_ssid: "dein-wlan"
    wifi_password: "dein-wlan-passwort"

## 3. Bauen und flashen

Im laufenden Tessera erscheint `guition43` in der Liste der Profile. Über **Firmware & USB** bauen
und über USB installieren (oder die Firmware herunterladen und mit ESPHome Web flashen). Die Instanz
baut mit ESPHome 2026.9.0, das die Boardfiles verlangen (`min_version: 2026.9.0`), und auf dem Board
liegt die Wide-Partitionstabelle, die ein 4-MB-Build braucht; `allow_partition_access: true` steht
schon im Profil.

## 4. In Home Assistant aufnehmen

Das automatische Anpaaren läuft nur für Screens, deren Pakete aus dem offiziellen Repository
kommen; dieses Profil ist lokal, also einmal von Hand:

1. **Settings → Devices & services**: das entdeckte ESPHome-Gerät aufnehmen (oder Add integration
   → ESPHome mit der IP des Screens und Port 6053).
2. Gefragter Schlüssel: der API-Key aus `guition43.yaml` (**api → encryption → key**) — derselbe
   trägt die OTA-Verschlüsselung.
3. Danach verwaltet Tessera den Screen wie jeden anderen: Kacheln, Editor, Alerts, Standby. Der
   Screen meldet selbst sein Aussehen („480x272 2x2 128dpi standard"), davon zeichnet der Editor
   sein Bild, auch wenn diese Instanz das Board noch nie gehört hat.

## Was (bis zum App-Update) anders bleibt

- **New screen** führt das Board nicht auf (der Katalog der App, `boards.json`, liegt im App-Image).
  Neue Screens dieses Boards legst du mit Kopien dieses Profils an (eigener Name, eigene Schlüssel).
- Die Firmware-WASM-Vorschau des Editors kennt die Dichte 128 dpi noch nicht: der Editor zeigt den
  gezeichneten Mockup statt des Renderbildes.
- Intern führt der Screen sein Board als „unknown"; sein Hello liefert die Facts, die der Betrieb
  braucht (Kachelbudget 128 Tiles / 24 Pages, Dimmen, Standby).
- Spätere Firmware-Updates: neuere Files nach `tessera-board/` kopieren und über dasselbe Profil
  neu bauen (USB oder Wi-Fi), wie oben.

## Offen (am Gerät zu prüfen)

Der erste Build sagt, wie viel vom 2.031.616-Byte-Slot die Firmware belegt; der Flash-Budget-Regel
nach gehören nie mehr als 97 % in einen Slot. Läuft der Build über dieses Limit, ist das die eine
echte offene Frage des Boards (Dichte 128 mit dem Standard-Look, 4 MB Flash) und kein Fehler des
Profils: dann Merker an mich oder den Maintainer, damit die Fonts/der Flash-Plan nachgezogen werden.

Am Glas nach dem ersten Flash prüfen: Touch in den vier Ecken und ein Wisch, Farbreihenfolge,
Drehung, ein Seitenwechsel, stehend ebenfalls. Die Touch-Achsen sind im Boardfile ungespiegelt
angenommen; lägen sie gespiegelt, stünden `TOUCH_MIRROR_X`/`TOUCH_MIRROR_Y` in einem Override-YAML
zur Hand (auf dem Screen unter Override, oder als `<name>.local.yaml` neben dem Profil).

## Anhang: Das Board in "New screen" sichtbar machen (ohne Image-Neubau)

Der Katalog von "New screen" liegt als `boards.json` im App-Image und ist der eine Teil, den der
lokale Weg nicht ändert; `boards.json` in diesem Ordner ist die Fassung mit dem Board (32 Boards,
26 Familien). Wer die Karte im Assistenten sehen will, trägt sie in den laufenden Container ein:

1. Advanced SSH & Web Terminal Add-on mit Docker-Zugriff (in dessen Einstellungen), oder ssh auf
   den Host. Dann die Datei in den Container kopieren und die App neu starten:

       docker cp boards.json addon_esp_screen_manager:/app/boards.json

   und im Home Assistant: Einstellungen → Add-ons → Tessera → **Neustart** (kein Update, kein
   Rebuild; der Neustart behält das Dateisystem des Containers). Danach die Editor-Seite neu laden:
   "Alle 26", und die Karte "Guition · 4,3 Zoll / 480 × 272 · GT911 · Experimentell" neben der
   des JC4880P443 (800 × 480, ESP32-P4). Der Editor fasst Karten nur zusammen, wenn Marke,
   Zoll, Chip und Auflösung gleich sind; das neue Board unterscheidet sich in beidem.

2. Wichtig: Auch mit sichtbarer Karte holt der Build aus "New screen" die Boardfiles von GitHub
   (`packages/jc4827w543.yaml` steht dort nicht, es wurde nichts gepusht) und bricht ab. Der Assistent
   ist trotzdem nützlich: er schreibt das Profil mit Name, WLAN, Sprache und Schlüsseln. Danach in
   der erzeugten Datei den Block `packages:` gegen die lokalen Includes tauschen (wie in
   `guition43.yaml` dieser Mappe) und die beiden Include-Pfade prüfen, dann bauen wie oben.

3. Grenzen des Eintrags: Ein Update oder Rebuild des Add-ons stellt das Image wieder her und die
   Karte ist weg (dann ist sie, nach dem Merge, auch offiziell da). Der Neustart der App allein
   behält sie.

## Nachtrag 1: Der Build brach am Text-Tabellen-Guard ab (behoben)

Der erste Build lief bis zum letzten Schritt durch und scheiterte nur an main.cpp:
`static assertion failed: screen_text_keys.h does not match screen_manager/translations/en.json`,
mit Hash 0x811C9DC5 (FNV-Startwert) und KEY_COUNT 0, also einer leeren Schlüsselliste.

Ursache: `components/smart_display/screen_text_gen.py` liest die Übersetzungen aus
`screen_manager/translations/` im Ordner zwei Ebenen über dem Component (bei einem Build über
GitHub klont ESPHome das ganze Repository, darum reiten sie dort mit). Die lokale Kopie hatte
diesen Ordner nicht, der Generator fand kein en.json, und der Guard des Builds stoppte die
Zeile-für-Zeile-Tabelle der Glass-Texte, bevor ein Screen mit falschen Indizes entstehen kann.

Behoben: `tessera-board/screen_manager/translations/` liegt jetzt im Paket (geprüft: der FNV-Hash
über en.json ergibt exakt 0xCBF6789C über 562 Schlüssel, wie screen_text_keys.h ihn verlangt, und
`de` löst sich auf, der Screen baut seine Texte in Deutsch). Zum Nachziehen im ESPHome-Ordner
reicht es, den Ordner `screen_manager/` in `tessera-board/` zu kopieren und neu zu bauen.

Gut zu wissen aus dem Log: Die Partitionstabelle war bereits die breite (app0/app1 je 1984K,
nvs_more) und der Build war bei 1650 von 1658 Objekten; nach dem Fix fehlen nur der letzte
Übersetzungsschritt und das Linken. Die Platzwarnungen (`image:`, `platformio_options->
build_flags`) gelten der core.yaml aller Boards gegen ESPHome 2026.10.2, nicht diesem Profil.

## Nachtrag 2: Trims greifen, 3,6 KB bleiben über (Behebung: eine Zeile)

Der zweite Build bestätigt die Text-Tabelle (main.cpp kompiliert) und die Trims wirken: .rodata
588.336 → 460.896 Bytes, Total 2.162.559 → 2.035.099. Der Flash-Check bleibt trotzdem stehen:

    binary guition43.bin size 0x1f0e10  -  Part 'app0' size 0x1f0000 (overflow 0xe10)

Es fehlen also nur 3,6 KB (Bin mit Padding). Der saubere Hebel dafür ist die Compiler-Optimierung
auf Größe statt Tempo; das erzeugt eine Neukompilierung (ccache läuft leer, ~15-25 Minuten), ändert
aber nichts am Aussehen und liefert schätzungsweise 75-150 KB. Wichtig: `sdkconfig_options` sitzt
direkt unter `framework:` (Nebeneinander zu `advanced:`; unter `advanced:` ist es ungültig, und
`password:`/`encryption:`-Mischung am OTA bleibt wie gehabt):

    # in guition43.yaml ergänzen (Hauptebene, neben packages/api/ota/wifi):
    esp32:
      framework:
        sdkconfig_options:
          CONFIG_COMPILER_OPTIMIZATION_SIZE: "y"

Erwartung danach: ~1,89-1,96 MB = 93-97 % des Slots, damit drin und nahe an der 97-%-Regel.
Zielwerte: Bin ≤ 0x1F0000 (2.031.616 Bytes) zum Linken, 97-%-Ziel ~1.970.667 Bytes.

Alternativen, falls gewünscht (mit Preis):

- Die beiden Tessera-Mark-Bilder aus core.yaml (38×38 und 18×18, RGB565) tragen 3.536 Bytes; sie
  wegzulassen reicht knapp, lässt aber das Markenbild auf Boot-Screen und Top-Bar weg, und es ist
  eine Änderung an der lokalen core.yaml, nicht am Profil.
- FONT_ICON_BPP: "1" spart noch ~71 KB, aber Icons ohne Kantenglättung sehen rau aus.
- Der Compact-Look (kanonisch bei 4 MB, CYD/Hosyond) spart deutlich mehr, aber Kacheln werden
  30 x 10 mm im 3x4-Raster (2x8 stehend) - spürbar kleinere Texte.

## Nachtrag 3: Der dritte Build war identisch — die Option muss in die Kette

Der dritte Build lief mit exakt denselben Zahlen (0x1f0e10, .text byte-gleich): Das SDK-Konfig
wurde nicht neu geschrieben. `main.cpp` und `build_info_data.cpp` bauen bei jedem Build neu
(Build-Info mit Datum), also war das kein Zeichen einer Änderung. Entweder war der korrigierte
esp32-Block im Profil beim Build noch nicht drin, oder er erreichte das SDK-Konfig nicht.

Sicherer Weg, unabhängig vom Profil: `sdkconfig_options` sitzt jetzt in der Hardware-Datei
(`tessera-board/packages/hardware/guition-jc4827w543.yaml`, esp32-Block), also in der Kette —
genau das Muster von hardware/esp32s3-rgb.yaml. Der Merge wurde nachgebildet: type, version,
advanced, board, variant, flash_size und cpu_frequency bleiben, die Option kommt durch. Das
Profil darf seinen esp32-Block behalten; gleicher Wert, gleiches Ergebnis.

Woran du den Erfolg am Build erkennst: Der nächste Build muss VOLL neu kompilieren (hunderte
Schritte, wie der erste), und die Größe-Tabelle sagt USING Os statt USING O3. Danach:
Total image size ≤ 2.031.616, Ziel ~1,9-1,96 MB. Läuft er immer noch über, die Linker-Zeilen hierher.

## Nachtrag 4: Clean Build, gleiches Ergebnis — ein Befehl entscheidet, dann Plan B

Der Clean-Build (1658 Schritte) zeigt unverändert USING O3 und byte-gleiches .text (1.470.876).
Die Optimierungszeile im SDK-Konfig hat sich also nie geändert — weder durch den Profil-Block noch
durch die Hardware-Datei. Entweder steht dort schon lange das Gegenteil (dann hilft -Os nichts,
denn es wäre schon aktiv), oder eine alte SDK-Konfig überlebt. Das klärt ein Befehl:

    docker exec addon_esp_screen_manager grep OPTIMIZATION /data/build/guition43/sdkconfig.guition43

- CONFIG_COMPILER_OPTIMIZATION_SIZE=y: die Optimierung ist schon auf Größe; der O3-Hinweis kommt
  aus woanders, und -Os bringt nichts mehr. Dann Plan B.
- CONFIG_COMPILER_OPTIMIZATION_PERF=y: die Option kam nicht an — dann die Hardware-Datei im
  ESPHome-Ordner gegen die aus dieser Mappe tauschen (Nachtrag 3) und erneut bauen.

Plan B, unabhängig vom Ergebnis und der kanonische Weg der 4-MB-Boards (CYD, Hosyond): der
Compact-Look. Schrumpft die Schrift-Glyphen deutlich (die .rodata-Masse ist Font-Bitmap),
erwartet 150-250 KB weniger; das Profil braucht dann im packages- und substitutions-Block:

    # packages:
    look: !include tessera-board/packages/looks/compact.yaml
    # substitutions (neu/geändert):
    GRID_COLS: "3"
    GRID_ROWS: "4"
    GRID_COLS_PORTRAIT: "2"
    GRID_ROWS_PORTRAIT: "8"

Preis: Kacheln 30 x 10 mm, spürbar kleinere Texte auf dem 4,3-Zoll-Glas. Die Trims (2 bpp Icons,
Bedside-Digits) bleiben drin — das ist exakt das Muster von CYD und Hosyond. Für den späteren PR
sind damit die Messzahlen da: Standard-Look auf 4 MB liegt am Rand (2.035.099 von 2.031.616,
100,2 %), der Maintainer entscheidet Compact-Look oder Optimierung.

## Nachtrag 5: Erfolg - der Build läuft durch

Der Compact-Look hat es gerettet: Total 1.948.903 Bytes, Bin 0x1dbd60, im 2.031.616-Slot
**95,9 %, 4 % frei** - unter der 97-%-Budget-Regel. .rodata 460.896 → 374.704 (kleinere
Glyphen), .text unverändert. Raster jetzt 3×4 liegend / 2×8 stehend, Kacheln 30×10 mm.

Erkenntnisse für das Protokoll:
- Der Hebel war der **Compact-Look**, nicht -Os: USING O3 blieb in allen Builds konstant und
  .text byte-gleich (1.470.872), egal was im SDK-Konfig stand. Der Log-Hinweis
  "CONFIG_COMPILER_OPTIMIZATION_DEFAULT was replaced with CONFIG_COMPILER_OPTIMIZATION_DEBUG"
  zeigt, dass die ESPHome-Options ankommen; das O3-Label stammt aus der CMake-Ebene und folgt
  dem SDK-Konfig offenbar nicht. Die -Os-Versuche waren in dieser Kette ein No-op.
- Verkleinerte Mark-Bilder (30×30/16×16 statt 38×38/18×18) tragen ein paar hundert Bytes.
- Bleibende Warnungen (image: deprecated, build_flags deprecated, USING O3) sind Kosmetik und
  betreffen die core.yaml aller Boards gegen ESPHome 2026.10.2.

## Nachtrag 6: Touch-Achsen am Glas bestätigt - Hardware-Datei tauschen

Die Seite liest sich, aber die Achsen sind verdreht: Tippen unten links trifft oben rechts, beide
Achsen gespiegelt (180 Grad). Damit ist der agillis-Widerspruch entschieden: Der GT911 dieses Glases
liest es verkehrt herum, die devices-Seite schweigt dazu. Fix: `transform: {mirror_x: true,
mirror_y: true}` im touchscreen-Block der Hardware-Datei - beide Kopien haben ihn jetzt. Nur die
**Hardware-Datei** rüberkopieren nach `tessera-board/packages/hardware/`, dann neu bauen (inkrementell,
nur der Touch-Teil) und neu flashen. Danach nochmal die Ecken tippen und den Wisch prüfen.

## Nachtrag 7: Touch gelöst - display-Transform plus physische Drehung

Diagnose-Build (display-transform sichtbar gedreht) bewies: Der Build liest die Hardware-Datei und
generiert die display-Transform — aber die touchscreen-Transform wirkt in keinem Build. Messung am
Glas: Der GT911 liest sein Glas einen halben Umschwung gegen den Panel-Sweep. Endstand:
- touchscreen-Transform **entfernt** (nur Kalibrierung bleibt)
- display-Transform `swap_xy: false, mirror_x: true, mirror_y: true` **bleibt**
- das Gerät wird physisch einen halben Umschwung gedreht montiert

Bild und Touch stimmen dann beide (am Glas bestätigt: Seiten laden, Tips treffen richtig).
Offen für den PR: Warum die touchscreen-Transform wirkungslos ist — Beweis wäre der main.cpp-Grep
im Add-on-Container (Advanced SSH & Web Terminal Add-on nötig, core-ssh hat kein docker).

## Nachtrag 8: Aufgeräumt - entbehrliche Dateien ins Archiv (Struktur erhalten)

Aus der Build-Kopie (tessera-board/) wanderten 105 Elemente in `Archiv/` im Projekt-Root, in der
ursprünglichen Ordnerstruktur (Archiv/guition43-local/tessera-board/...). Verbleiben ist genau das,
was der Build liest: core.yaml, boards/jc4827w543.yaml, die 3 Hardware-Files (guition-, flash-4mb,
esp-idf), die 3 Features (capacitive-touch, backlight, self-test), looks/compact.yaml + shared/,
components/smart_display + flash_layout, fonts/ und screen_manager/translations (ALLE Sprachen -
die Sprachdateien außer en+de sind im Text-Generator ein offenes Risiko und bleiben). Auch der
hardware/parts/-Baum (Audio/Backlight-Teile, nur von den archivierten audio-Features gebunden) wanderte.
Bewusst NICHT verschoben: guition-jc3248w535.yaml (Referenz), guition43-local/boards.json (Container-Patch
noch offen), tessera/ + .venv (PR offen), AGENTS.md, der Patch. Der erste Build nach der Ausdünnung muss
identisch landen (1.948.903 B) - falls nicht, steht die fehlende Datei in Archiv/ an exakt ihrem Pfad.
