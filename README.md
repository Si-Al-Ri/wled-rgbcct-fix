# WLED (RGB+CCT color fix)

A patched copy of the official **WLED integration** from Home Assistant, so that strips
reporting RGB **and** white **and** color temperature (for example FW1906 / RGBCCT) show and
update their real color in Home Assistant.

---

## The problem

When a WLED segment reports the capabilities RGB + white + color temperature (`lc: 7`), the
integration maps it to the color modes `[COLOR_TEMP, RGBW]` and then takes the **first** entry
as the active color mode:

```python
# light.py
self._attr_color_mode = color_modes[0]
self._attr_supported_color_modes = set(color_modes)
```

So the light is permanently in `color_temp`. Home Assistant then derives the displayed color
from the color temperature, and the actual RGB value never arrives:

- `color_mode` stays `color_temp`, `rgbw_color` is `null`
- `rgb_color` is a near-white value computed from the temperature and barely changes
- color changes made in the WLED web UI are not visible in Home Assistant

Commands from Home Assistant to WLED work fine, only the way back is affected. The raw device
data is correct as well: `http://<wled-ip>/json/state` reports the right values in `seg[].col`.

## What was changed

This is a **verbatim copy of the WLED integration from Home Assistant 2026.6.4** with two
changes, both marked with `CUSTOM-PATCH` in the code.

**1. `const.py` — swap the order of the color modes**

In `LIGHT_CAPABILITIES_COLOR_MODE_MAPPING`, for `RGB_COLOR | WHITE_CHANNEL | COLOR_TEMPERATURE`:

```python
# upstream
[ColorMode.COLOR_TEMP, ColorMode.RGBW]
# here
[ColorMode.RGBW, ColorMode.COLOR_TEMP]
```

Because `light.py` uses `color_modes[0]`, the active mode becomes `rgbw` and the color is passed
through again. `COLOR_TEMP` stays in `supported_color_modes`, so setting a color temperature
still works.

**2. `light.py` — keep the color temperature readable**

Home Assistant clears `color_temp_kelvin` as soon as the active color mode is not `color_temp`.
With the change above that value would always be `null`, so it is additionally exposed as a
state attribute:

```python
@property
def extra_state_attributes(self) -> dict[str, Any] | None:
    return {"cct_kelvin": self.color_temp_kelvin}
```

Nothing else was touched.

### A note on the approach

This is a workaround, not a proper upstream fix. Home Assistant allows only one active color
mode at a time, and this simply picks the more useful one for these strips. A real fix would
select the mode dynamically, based on whether the light currently shows a color or a white
tone.

## Installation via HACS

1. Open HACS, use the menu in the top right corner, **Custom repositories**.
2. Add this repository URL with category **Integration**.
3. Install "WLED (RGB+CCT color fix)".
4. **Restart** Home Assistant.

The integration uses the domain `wled` and therefore replaces the built-in one. Your existing
setup, devices and entities are kept, nothing has to be added again. The log will show a
"custom integration wled" warning, which is expected and confirms that the patched version is
active.

### Verifying

Developer tools → States → the light entity of your device:

- `color_mode` is `rgbw` (was `color_temp`)
- `rgbw_color` has values (was `null`) and changes when you change the color
- `cct_kelvin` shows the current color temperature

## Reverting

Uninstall it in HACS or delete the `custom_components/wled` folder, then restart Home Assistant.
The built-in integration takes over again.

## Home Assistant updates

This copy is frozen on **Home Assistant 2026.6.4**. After a larger Home Assistant update it can
become outdated, because internal interfaces change. Either uninstall it once the issue is
fixed in Home Assistant itself, or update to a newer version published here.

If you only need a quick workaround and your hardware allows it, you can also pick a LED type
without a color temperature channel in WLED.

## Credits and license

Based on the WLED integration from [Home Assistant Core](https://github.com/home-assistant/core)
(version 2026.6.4), originally written by `@frenck` and `@mik-laj`.

Licensed under the **Apache License 2.0**, see [LICENSE](LICENSE). The changes to `const.py` and
`light.py` described above were made to the original files.

This project is not affiliated with the Home Assistant project or with WLED.

## Related

[WLED Control Card](https://github.com/Si-Al-Ri/wled-control-card), a Lovelace card for
controlling WLED devices, which makes use of the `cct_kelvin` attribute from this integration.
