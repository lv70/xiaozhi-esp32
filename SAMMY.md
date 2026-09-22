# Sammy fork of xiaozhi-esp32

This is the **development line** for Sammy's firmware. It is a fork of upstream
[78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32), kept as a real git branch so
upstream releases can be merged and board fixes can be offered back as pull requests.

    fork:     https://github.com/lv70/xiaozhi-esp32   (PUBLIC — GitHub does not allow
                                                        private forks of public repos)
    branch:   sammy
    based on: upstream tag v2.5.0 = ac6deed3d8e75348475364bf40ad953c6cd48054 (2026-09-10)
    created:  2026-09-22
    IDF:      ESP-IDF 6.0.2  (source ~/.espressif/tools/activate_idf_v6.0.2.sh)
    target:   esp32s3  ·  board: espressif/esp-vocat  ·  language: en-US

It is consumed by the deployment repo `lv70/SAMMY-firmware` as the submodule
`firmware/xiaozhi-esp32`. The two *vendored* trees there (`xiaozhi-esp32-v2.2.6`, on the
white unit; `xiaozhi-esp32-v2.4.2`) are the known-good fallbacks, not development lines.

## Because this repo is public

Never commit Wi-Fi credentials, server tokens, the Hermes auth token, or device dumps here.
The server URL is a runtime NVS setting and must stay that way. The `# Sammy:` block in
`sdkconfig.defaults` contains no secrets.

## Local changes vs upstream (keep this list current)

- `sdkconfig.defaults` — appended the `# Sammy:` block: `CONFIG_LANGUAGE_EN_US`,
  `CONFIG_BOARD_TYPE_ESP_VOCAT`, `CONFIG_USE_EMOTE_MESSAGE_STYLE`,
  `CONFIG_MMAP_FILE_NAME_LENGTH=32`, `CONFIG_FLASH_EXPRESSION_ASSETS`. The last three mirror
  the board's own `config.json` `sdkconfig_append` so a plain `idf.py build` picks them up
  without `scripts/build.py`.
- `SAMMY.md` — this file.

## Planned work on this branch (see the SAMMY-firmware README and wiki)

1. Mount the ESP-VoCat SD card — SDMMC 1-bit: clk=GPIO16, cmd=GPIO38, d0=GPIO17, internal
   pull-ups, card power shared with the LCD enable. Upstream defines SD_* pins in
   `main/boards/espressif/esp-vocat/config.h` but never mounts anything.
2. Firmware-only alarm — needs SNTP (device time currently comes only from the OTA response),
   schedule in NVS, and local playback; reuse the v2.5.0 `Notifying` state /
   `main/notify/notify_player` plumbing for a local file instead of an HTTP stream.
3. Local music from the card — Ogg/Opus only (`main/audio/demuxer/ogg_demuxer.cc`), 24 kHz mono
   output; convert with `scripts/mp3_to_ogg.sh`.
4. Custom emoticons — `main/boards/espressif/esp-vocat/assets/360_360/emote.json` and a
   same-named `.eaf` in `main/boards/espressif/esp-vocat/emoji/` overrides the component's.

## Updating from upstream

    git fetch upstream --tags
    git merge v2.6.0          # or whichever tag; resolve conflicts, rebuild, test on Redline first

Known upstream state to watch: the IDF-6 gate at `esp_vocat.cc:16` disables the PCB
capacitive slider/button (`touch_slider_sensor` / `touch_button_sensor` need IDF < 6.0).
Lucy does not need them.
