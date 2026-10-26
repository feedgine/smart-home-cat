# Smart Home Cat — setup guide

About 30–45 minutes. You need:

- Home Assistant with the **ESPHome Device Builder** add-on (or ESPHome on your computer).
- An **ESP32-S3** board with a **round 240×240 GC9A01A screen** on SPI.
  The defaults fit the Spotpear / XiaoZhi ESP32-S3 "robot ball". Other boards work if you change the pins.
- A USB-C cable for the first flash. After that, updates go over Wi-Fi.

There are two parts: **Home Assistant** (decides the mood) and **the cat** (shows it).
Set up Home Assistant first, so the cat has something to show when it starts.

---

## Part 1 — Home Assistant

### 1.1 Turn on packages (once)

Open `configuration.yaml` (File editor or Studio Code Server add-on) and make sure it has:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Create the folder `/config/packages/` if it does not exist.

### 1.2 Copy the two files

| From this repo | To your HA |
|---|---|
| `homeassistant/packages/smart_home_cat.yaml` | `/config/packages/smart_home_cat.yaml` — core, don't edit |
| `homeassistant/cat_alerts.example.yaml` | `/config/packages/cat_alerts.yaml` — **your** alerts |

### 1.3 Decide when the house is "busy"

In `cat_alerts.yaml`, edit **Cat busy**. When it is on, the cat purrs (^ ^). When it is off, it naps.

```yaml
- name: Cat busy
  unique_id: cat_busy
  state: "{{ now().hour >= 7 and now().hour < 23 }}"      # awake by day
  # state: "{{ is_state('climate.living_room', 'heat') }}" # or: heating is on
```

### 1.4 Write your alerts

Each alert is a normal template binary sensor with `cat_level`:

```yaml
- name: "Cat alert · Freezer too warm"
  unique_id: cat_alert_freezer_warm
  state: "{{ states('sensor.freezer_temperature') | float(-99) > -12 }}"
  delay_on: "00:10:00"                 # must be true for 10 min first
  attributes:
    cat_level: alert                   # alert = red, big eyes · check = yellow, crescent eyes
    cat_label: "Freezer too warm · Kitchen"   # WHAT and WHERE
    cat_minutes: 30                    # fuss for 30 min (0 = as long as it lasts)
```

Tips:

- **Find entity ids** in Settings → Devices & services → Entities.
- **Test a condition first** in Developer tools → Template. Paste the `state:` template and check it shows `True` or `False`.
- **Delete the examples** you don't need. A wrong entity id just stays off, but it is clutter.
- **Keep labels short** (about 40 characters) and in plain Latin letters. Emoji are removed.
- **Already have a sensor for it?** You can add the `cat_*` attributes to any template binary sensor you already have.

### 1.5 One-off events (optional)

For things that happen once, call the script from any automation.
For example: a backup failed, or the doorbell rang while you were away.

```yaml
action: script.cat_raise_alert
data:
  title: "Backup failed · NAS"
  level: alert        # or check
  minutes: 60
```

Raising the same `key` (or title) again replaces it. `clear: true` removes it.
**Cat: clear one-off alerts** removes them all.

### 1.6 Check and restart

1. Developer tools → YAML → **Check configuration**. Fix anything it reports.
2. **Restart** Home Assistant.
3. Check that `sensor.cat_mood` shows `sleeping` or `content`, and that `sensor.cat_issues` is empty (no alerts).

---

## Part 2 — The cat (ESPHome)

### 2.1 Create the device

1. ESPHome Device Builder → **New device** → name it `smart-home-cat` → skip the board step.
2. Open the new YAML, replace everything with `examples/smart-home-cat.yaml` from this repo,
   and set `url:` to this repository.
3. Open **Secrets** (top right) and add the values from `examples/secrets.example.yaml`:
   your Wi-Fi name and password, a new API key, and OTA and fallback passwords.
   Generate the API key with the ESPHome wizard or `openssl rand -base64 32`.

### 2.2 Different board? Change pins and colours

Add any of these under `substitutions:` in your device YAML:

```yaml
substitutions:
  cat_lcd_cs: GPIO13
  cat_lcd_dc: GPIO10
  cat_lcd_reset: GPIO18
  cat_backlight: GPIO3
  cat_spi_clk: GPIO14
  cat_spi_mosi: GPIO17
  cat_invert_colors: "true"     # colours look wrong (negative)? flip this
  cat_eye_color: "FF8AD8"       # pink cat
  cat_glow_color: "8A2F75"
```

The full list is at the top of `esphome/cat.yaml`.
If your board has no PSRAM or uses quad PSRAM, change or remove the `psram:` block.

### 2.3 First flash (USB)

1. Plug the board into your computer.
2. **Install** → **Plug into this computer**, then follow the browser flasher.
   If it does not connect, hold **BOOT** while plugging in.
3. After the first flash, use **Install → Wirelessly** for updates.

### 2.4 Add it to Home Assistant

HA finds the new ESPHome device on its own: Settings → Devices & services → **Discovered** → Configure.
Enter the API key from your secrets. The cat now shows its mood.

---

## Part 3 — Try it

1. Settings → Automations & scenes → Scripts → **Cat: demo all moods**: 20 s per mood.
2. Or set **Cat test mode** (`input_select.cat_test_mode`) to a mood, then set it back to `auto`.
3. **Cat: demo reel (14 s)** is a fast version, good for short videos.

Test mode shows sample "TEST …" lines and changes nothing else.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Screen stays black | Check the backlight pin (`cat_backlight`). Check the screen in the ESPHome logs. |
| Colours are negative | Flip `cat_invert_colors`. |
| Picture is shifted or mirrored | Wrong screen model or pins. Check the pinout of your board. |
| Cat always naps | Check that `binary_sensor.cat_busy` turns on. |
| Cat never reacts | Check `sensor.cat_issues` in Developer tools → States. Test your alert template. |
| Text shows boxes | The label has letters the font doesn't have. Use plain Latin letters. |
| Stuck on one mood | Set `input_select.cat_test_mode` back to `auto`. |
| Red "Home Assistant lost" | The cat has had no connection to HA for 2 min. Check Wi-Fi, check that HA is running, and check the device in Settings → ESPHome. Change the time with `cat_ha_lost_seconds`. |
