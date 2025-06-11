# AlphaESS Battery SOC Limit – Home Assistant Package

Dieses Repository enthält ein **Home Assistant Package**, das den Export (Einspeisung)
deiner AlphaESS‑Batterie begrenzt, sobald ein frei wählbarer SoC‑Schwellenwert
erreicht ist.
> **Hinweis**  
> Dieses Package baut auf den Sensor‑Entitäten auf, die vom
> **AlphaESS‑Home‑Assistant‑Script** aus  
> <https://projects.hillviewlodge.ie/alphaess/>  
> erzeugt werden.  
>
> Wenn du ein anderes Skript oder eine eigene Modbus‑Einbindung verwendest,
> **passe die im Package referenzierten Sensor‑ und Helper‑Namen entsprechend an**,
> damit die Automation korrekt funktioniert.

## Dateien

```
alphaess-ha-addon/
├── packages/
│   └── integration_alpha_ess_addon_soc.yaml
├── LICENSE
└── README.md
```

- **packages/integration_alpha_ess_addon_soc.yaml**  
  Enthält *Input Helpers* und die Automation zum Limitieren der Batterie‑Einspeisung.
- **README.md**  
  Diese Anleitung.
- **LICENSE**  
  MIT-Lizenz.

## Installation

1. Kopiere den Ordner `packages` in dein Home‑Assistant‑`config/`‑Verzeichnis (oder
   wohin du deine Packages einbindest).
2. Stelle sicher, dass in deiner `configuration.yaml` der folgende Block vorhanden ist:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

3. **Konfiguration prüfen** → **Neustart**.
4. Lege in deinem Dashboard eine Entities‑Card an und binde dort die Helper
   (`input_number.alphaess_helper_soc_limit` und
   `input_boolean.alphaess_helper_export_enable`) ein.

## GitHub Veröffentlichung

```bash
# Repo klonen oder entpacken
cd alphaess-ha-addon
git init
git add .
git commit -m "Initial commit – AlphaESS SOC limiter package"
gh repo create alphaess-ha-addon --public --source=. --remote=origin
git push -u origin main
```

**Fertig!** Dein Package ist jetzt öffentlich verfügbar.

## Dashboard / Lovelace‑Karte

```
type: entities
title: ⚡️ AlphaESS - Batteriebegrenzung
show_header_toggle: false
entities:
  - type: section
    label: 🔋 Batterie
  - entity: sensor.alphaess_power_battery
    name: Lade / Entlade­leistung
    icon: mdi:battery-charging
  - entity: sensor.alphaess_soc_battery
    name: Ladestand (SoC)
    icon: mdi:battery-80
  - entity: sensor.alphaess_excess_power
    name: PV-Überschuss (W)
    icon: mdi:flash
  - entity: sensor.alphaess_dispatch_mode
  - type: section
    label: ⚙️ Steuerung
  - entity: input_number.alphaess_helper_soc_limit
    name: SoC-Grenze (Slider)
    icon: mdi:slider-variant
  - entity: input_boolean.alphaess_helper_export_enable
    name: SoC-Begrenzen
    icon: mdi:power-plug
  - type: section
```
