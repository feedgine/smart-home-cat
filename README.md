# Smart Home Cat 🐱

A little round "cat" that sits on your desk and shows the mood of your Home Assistant with its eyes.
It purrs when the house is busy, naps when all is quiet, and gets very wide-eyed when something is wrong.
The name of the problem scrolls along the top.

Runs on any **ESP32-S3** with a **round 240×240 GC9A01A screen**, for example the Spotpear / XiaoZhi
ESP32-S3 "robot ball". Built with [ESPHome](https://esphome.io) and Home Assistant.
No cloud, no app.

## Moods

| | Mood | When |
|---|---|---|
| 😴 | **Cat nap**: breathing bars | no alerts, house not busy |
| 😊 | **Purring**: "^ ^", slow blinks | no alerts, house busy (you decide what busy means) |
| 🤨 | **Hmm?**: crescent eyes, yellow text | a `check` alert is on |
| 😳 | **Alarm**: big eyes, fast blinks, red ring, red text | an `alert` alert is on |

See them animated: open [`docs/moods.html`](docs/moods.html) in a browser.

## How it works

```
your alerts (cat_alerts.yaml) ─┐
script.cat_raise_alert ────────┼─▶ sensor.cat_issues ─┐
binary_sensor.cat_busy ────────┘                      ├─▶ sensor.cat_mood ─▶ ESPHome cat
                                                       │
                                    scrolling line ◀───┘
```

- **One config file for alerts.** Each alert is a normal HA template binary sensor with
  `cat_level` (`alert` or `check`), an optional `cat_label` ("what · where") and `cat_minutes`
  (how long the cat fusses, 30 by default).
- **One-off events** come from `script.cat_raise_alert` in any automation.
- **The cat** is an ESPHome package. Your device YAML is about 40 lines, and pins and colours are settings.

## Quick start

1. Copy `homeassistant/packages/smart_home_cat.yaml` and `homeassistant/cat_alerts.example.yaml`
   (as `cat_alerts.yaml`) to `/config/packages/`, edit your alerts, restart HA.
2. Create an ESPHome device from `examples/smart-home-cat.yaml`, fill in your secrets, flash it.
3. Run the script **Cat: demo all moods** to see every face.

The full step-by-step guide is in [`docs/SETUP.md`](docs/SETUP.md).

## Repository

| Path | What |
|---|---|
| `esphome/cat.yaml` | ESPHome package: screen, eyes, scrolling line |
| `examples/smart-home-cat.yaml` | device config that pulls the package from GitHub |
| `examples/smart-home-cat-local.yaml` | same, using the local package (development, CI) |
| `examples/secrets.example.yaml` | secret names to fill in |
| `homeassistant/packages/smart_home_cat.yaml` | HA core: mood, issues, raise-alert script, test mode, demos |
| `homeassistant/cat_alerts.example.yaml` | your alerts + "busy" rule (examples inside) |
| `docs/SETUP.md` | setup guide |
| `docs/moods.html` | animated preview of all moods |

## Customising

- **Colours:** set `cat_eye_color` / `cat_glow_color` (for example a pink cat: `FF8AD8` / `8A2F75`).
- **Other boards:** set the `cat_lcd_*`, `cat_spi_*` and `cat_backlight` pins.
- **Bluetooth proxy:** uncomment the block in the example device YAML.

## Licence

MIT. See [LICENSE](LICENSE).
