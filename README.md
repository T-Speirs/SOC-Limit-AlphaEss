# AlphaESS Battery SOC Limit – Home Assistant Package

Dieses Repository enthält ein **Home Assistant Package**, das den Export (Einspeisung)
deiner AlphaESS‑Batterie begrenzt, sobald ein frei wählbarer SoC‑Schwellenwert
erreicht ist.
> **Hinweis**  
> Dieses Package baut auf den Sensor‑Entitäten auf, die vom
> **AlphaESS‑Home‑Assistant‑Script** aus  
> <https://projects.hillviewlodge.ie/alphaess/>  
> erzeugt werden (Danke Alex!).
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
   `input_boolean.alphaess_helper_export_enable`) ein. Siehe nachfolgend:

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


## Funktionsweise

1. **Grenzwert setzen** – Mit dem Slider (`input_number.alphaess_helper_soc_limit`) legst du fest, bis zu welchem *State of Charge* (SoC) der Akku befüllt werden darf.
2. **Regelung aktivieren** – Schalte `input_boolean.alphaess_helper_export_enable` ein.  
   Ab jetzt prüft die Automation alle&nbsp;10 s:
   - PV‑Produktion > Hausverbrauch  
   - Batterie‑SoC ≥ eingestellter Grenzwert  
   - (Optional) Wechselrichter ist *nicht* im Modus 2 **oder** die Dispatch‑Zeit < 15 min
3. **Modbus‑Befehle** – Wenn alle Bedingungen erfüllt sind, werden die in der Datei hinterlegten Modbus‑Register beschrieben.  
   Dadurch wird das Laden des Akkus reduziert bzw. die Einspeisung ins Netz freigegeben (abhängig von deinem AlphaESS‑Setup).
4. **Status‑Helper** – Die Automation setzt außerdem `input_boolean.alphaess_excess_export_status` auf *on*, damit du im Dashboard siehst, wann die Begrenzung aktiv ist.
5. **Sicherheit** – Durch `mode: single` + `max_exceeded: silent` läuft immer nur *eine* Instanz, Log‑Spam wird unterdrückt, falls mehrere Trigger innerhalb von 10 s eintreffen.

## Warnhinweis & Haftungsausschluss

> **⚠️ Wichtig – Batteriepﬂege**  
> Eine *dauerhafte* Begrenzung auf einen festen Ladezustand kann die Kalibrierung und Chemie des Speichers beeinträchtigen.  
> Lade den Akku deshalb regelmäßig – vollständig (100 %), damit das BMS einen vollständigen Ladezyklus absolvieren kann.

> **Nutzung auf eigene Gefahr**  
> Dieses Add‑on wird ohne jede Gewähr bereitgestellt. Der Autor übernimmt **keinerlei Haftung** für Sach‑, Personen‑ oder Folgeschäden, die durch Installation, Konfiguration oder Betrieb entstehen.
