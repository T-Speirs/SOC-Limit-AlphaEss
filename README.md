# AlphaESS Battery SOC Limit – Automation für Home Assistant Package

Dieses Repository enthält eine Automation für **Home Assistant**, welche das Laden
deiner AlphaESS‑Batterie begrenzt, sobald ein frei wählbarer SoC‑Schwellenwert
erreicht ist.

Die Automation läuft auf folgender Hardware Konfiguration:

| | Wechselrichter    | Batterie   | Firmware |
| ------------- | ------------- | ------------- | ------------- |
| :white_check_mark: |  SMILE-G3-S5 |  SMILE-G3-BAT-3.8S  |

Ich gehe davon aus, dass die Begrenzung des Batterielimits auch mit anderen AlphaESS-Systemen kompatibel ist, welche über die gleichen Dispatch Modi verfügen und über Modbus ansprechbar sind - **jedoch wurde dies von mir nicht getestet**.


## Hinweis zu Abhängigkeiten 

Dieses Package baut auf den Sensor‑Entitäten auf, die vom **AlphaESS‑Home‑Assistant‑Script** <https://projects.hillviewlodge.ie/alphaess/> erzeugt werden und ist damit notwendiger Bestandteil für die funktionsweise dieser Automation. 

Solltest du nicht auf diesen Aufbauen möchten, **passe die im Package referenzierten Sensor‑ und Helfer‑Namen entsprechend an**, damit die Automation korrekt funktioniert.

## Motivation

Der AlphaESS-Wechselrichter bietet von Haus aus keine Möglichkeit, Lade- und Entladestrom **dynamisch** anhand des aktuellen SoC *und* der momentanen PV-Leistung zu regeln. In Folge dessen, 

* verharrte die Batterie oft bei 100 % Ladestand – unnötiger Stress und beschleunigter Verschleiß.  
* Im Home-Assistant-Dashboard fehlten eindeutige Schalt- und Anzeige­elemente, um schnell eingreifen zu können.
* Die von AlphaEss angebotenen SoC-Grenzen beheben dieses Problem nicht, die Einspeisung der PV-Anlage grundsätzlich ignorieren - meine Vermutung ist, dass diese Funktion eher für das aktive be- und entladen im Zusammenhang mit der wirtschaftlichen maximierung für die Nutzung dynamischer Stromtarrife konzipiert wurde.

Dieses Add-on schließt genau diese Lücke:

* Mit einem **Slider** setzt du ein SoC-Limit, mit einem **Toggle** aktivierst oder deaktivierst du die Regelung sofort.  
* Eine Automation prüft alle 10 Sekunden (grundsätzlich Anpassbar) den PV-Ertrag und Hausverbrauch und stoppt das Laden, sobald der Ziel-SoC erreicht ist.  
* Alle Befehle werden lokal über Modbus-Register ausgeführt – **keine Cloud, keine Verzögerungen**.  
* Eine Lovelace-Karte zeigt SoC, PV-Überschuss und Exportstatus auf einen Blick.

**Kurz gesagt:** Das Add-on optimiert deinen Eigenverbrauch, schont die Batterie, reduziert manuelles Eingreifen und integriert sich nahtlos in jedes Home-Assistant-Setup.

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

## Installation

1. Kopiere den Ordner `packages` in dein Home‑Assistant‑`config/`‑Verzeichnis (oder
   wohin du deine Packages einbindest).
2. Stelle sicher, dass in deiner `configuration.yaml` der folgende Block vorhanden ist:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
(Der Ordner aus Schritt 1 und die Anpassung in der configuration.yaml aus Schritt 2 sollte bereits vorhanden sein, wenn du *AlphaESS‑Home‑Assistant‑Script** von <https://projects.hillviewlodge.ie/alphaess/> verwendest)

3. **Konfiguration prüfen** → **Neustart**.
4. Lege in deinem Dashboard eine Entities‑Card an und binde dort die Helper
   (`input_number.alphaess_helper_soc_limit` und
   `input_boolean.alphaess_helper_soc_limit_enable`) ein. Siehe nachfolgend:

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
  - entity: sensor.batteriemodus
    name: Betriebsmodus
    icon: mdi:battery-outline
  - type: section
    label: ⚙️ Steuerung
  - entity: input_number.alphaess_helper_soc_limit
    name: SoC-Grenze (Slider)
    icon: mdi:slider-variant
  - entity: input_boolean.alphaess_helper_soc_limit_enable
    name: SoC-Begrenzen
    icon: mdi:power-plug
```

## Funktionsweise

1. **Grenzwert setzen** – Mit dem Slider (`input_number.alphaess_helper_soc_limit`) legst du fest, bis zu welchem
*State of Charge* (SoC) der Akku befüllt werden darf.

2. **Automation aktivieren** – Schalte `input_boolean.alphaess_helper_soc_limit_enable` ein.  
   Ab jetzt prüft die Automation alle 10 Sekunden:
   - PV‑Produktion > Hausverbrauch  
   - Batterie‑SoC ≥ eingestellter Grenzwert  
   - SOC-Limit im Dashboard aktiviert - `input_boolean.alphaess_helper_soc_limit_enable`

3. **Modbus‑Befehle** – Wenn alle Bedingungen erfüllt sind, werden die in der Datei hinterlegten Modbus‑Register
beschrieben. Dadurch wird "Dispatch Mode 2" jeweils aktiviert oder deaktiviert, welche das Laden und Beladen der
Batterie stoppt oder freigibt. 

## Warnhinweis & Haftungsausschluss

> **⚠️ Wichtig – Batteriepﬂege**  
> Eine *dauerhafte* Begrenzung auf einen festen Ladezustand kann die Kalibrierung und Chemie des Speichers beeinträchtigen.  
> 
> Lade den Akku deshalb regelmäßig – vollständig (100 %), damit das BMS einen vollständigen Ladezyklus absolvieren kann.
> 
> **Nutzung auf eigene Gefahr**  
> 
> Dieses Add‑on wird ohne jede Gewähr bereitgestellt. Der Autor übernimmt **keinerlei Haftung** für Sach‑, Personen‑ oder Folgeschäden, die durch Installation, Konfiguration oder Betrieb entstehen.
