# WLED (RGB+CCT color fix)

Angepasste Kopie der offiziellen **WLED-Integration** aus Home Assistant, damit Streifen mit
RGB **und** Weiß **und** Farbtemperatur (z. B. FW1906, RGBCCT) ihre Farbe in Home Assistant
korrekt anzeigen und aktualisieren.

> **English:** A patched copy of the official Home Assistant WLED integration for strips that
> report RGB **and** white **and** color temperature (for example FW1906 / RGBCCT). Without the
> patch such lights are stuck in `color_temp` mode, so the real RGB color never reaches Home
> Assistant. See [What is changed](#was-genau-geaendert-wurde).

---

## Das Problem

Meldet ein WLED-Segment die Fähigkeiten RGB + Weiß + Farbtemperatur (`lc: 7`), ordnet Home
Assistant dem Licht die Farbmodi `[color_temp, rgbw]` zu und nimmt davon den **ersten** als
aktiven Modus. Das Licht steckt damit dauerhaft in `color_temp`, und Home Assistant leitet die
angezeigte Farbe aus der Farbtemperatur ab. Ergebnis:

- `color_mode` bleibt `color_temp`, `rgbw_color` ist `null`
- die tatsächliche Farbe kommt nie in Home Assistant an, die Anzeige wirkt eingefroren
- Farbänderungen in der WLED-Oberfläche sind in Home Assistant nicht sichtbar

Befehle von Home Assistant an WLED funktionieren, nur der Rückweg ist betroffen.

## Was genau geändert wurde

Die Integration ist eine **1:1-Kopie aus Home Assistant 2026.6.4** mit zwei Änderungen:

| Datei | Änderung |
|---|---|
| `const.py` | Im `LIGHT_CAPABILITIES_COLOR_MODE_MAPPING` steht für RGB + Weiß + Farbtemperatur jetzt `[RGBW, COLOR_TEMP]` statt `[COLOR_TEMP, RGBW]`. Damit ist `rgbw` der aktive Farbmodus und die Farbe wird durchgereicht. `color_temp` bleibt weiterhin unterstützt. |
| `light.py` | Neues Attribut `cct_kelvin`, das die Farbtemperatur **immer** ausgibt. Home Assistant setzt `color_temp_kelvin` im `rgbw`-Modus auf `null`, wodurch Oberflächen den Wert sonst nicht anzeigen können. |

Beide Stellen sind im Code mit `CUSTOM-PATCH` gekennzeichnet.

## Installation über HACS

1. HACS öffnen, oben rechts das Menü, **Benutzerdefinierte Repositories**.
2. Diese Repository-URL eintragen, Kategorie **Integration**, hinzufügen.
3. „WLED (RGB+CCT color fix)" installieren.
4. Home Assistant **neu starten**.

Die Integration verwendet die Domain `wled` und ersetzt dadurch die eingebaute WLED-Integration.
Deine bestehende Einrichtung, Geräte und Entitäten bleiben erhalten, es ist keine Neuanlage nötig.
Im Protokoll erscheint der Hinweis „custom integration wled", das ist erwartet und bestätigt,
dass die angepasste Version aktiv ist.

### Prüfen

Entwicklerwerkzeuge → Zustände → die Light-Entität deines Geräts:

- `color_mode` ist `rgbw` (vorher `color_temp`)
- `rgbw_color` enthält Werte (vorher `null`) und ändert sich bei Farbwechseln
- `cct_kelvin` zeigt die aktuelle Farbtemperatur

## Rückgängig machen

In HACS deinstallieren oder den Ordner `custom_components/wled` löschen, danach Home Assistant
neu starten. Es greift wieder die eingebaute Integration.

## Wichtig bei Home-Assistant-Updates

Diese Kopie ist auf **Home Assistant 2026.6.4** eingefroren. Nach einem größeren Update von Home
Assistant kann sie veralten, weil sich interne Schnittstellen ändern. Zwei Möglichkeiten:

- die Integration hier deinstallieren, sobald der Fehler in Home Assistant selbst behoben ist
- oder auf eine hier veröffentlichte, neuere Version aktualisieren

Wer nur eine kurzfristige Lösung braucht, kann alternativ in WLED einen LED-Typ ohne
Farbtemperatur-Kanal wählen, sofern die Hardware das zulässt.

## Herkunft und Lizenz

Basiert auf der WLED-Integration aus [Home Assistant Core](https://github.com/home-assistant/core)
(Version 2026.6.4), ursprüngliche Autoren `@frenck` und `@mik-laj`.

Lizenziert unter der **Apache License 2.0**, siehe [LICENSE](LICENSE). Die oben beschriebenen
Änderungen an `const.py` und `light.py` wurden gegenüber dem Original vorgenommen.

Dieses Projekt steht in keiner Verbindung zum Home-Assistant-Projekt oder zu WLED.

## Passend dazu

[WLED Control Card](https://github.com/Si-Al-Ri/wled-control-card), eine Lovelace-Karte zur
Steuerung von WLED-Geräten, die das Attribut `cct_kelvin` dieser Integration nutzt.
