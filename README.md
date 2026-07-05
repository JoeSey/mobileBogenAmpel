# 🎯 Bogen-Ampel

Eine minimalistische Progressive Web App (PWA) zur Steuerung von Schießzeiten
in Bogen-Trainingsturnieren (die Konformität mit allen Verbands-Regularien ist ungetestet,
daher die Empfehlung nur für interne Turniere) – als Ampel-Anzeige mit 
Bluetooth-Clicker-Unterstützung, gedacht für den Einsatz direkt auf dem Schießplatz, 
komplett offline, falls kein Internet verfügbar.

Auf dem Schießplatz für „einfach-so-Turniere“ ist es meist aufwändig, ein Laptop, Monitor
und Lautsprecher mitzuschleppen. Ein Stativ, Halterung, Tablet und ein Bluetooth-Clicker 
und eine Bluetooth-Box reichen da völlig aus!

## Was macht die App?

mobileBogenAmpel bildet den klassischen Ablauf einer Schießpasse ab:

1. **Bereit** – Warten auf Start
2. **An die Linie** (Vorwarnzeit) – akustisches Signal, Schützen gehen an die Linie
3. **Schießzeit** – Countdown, wechselt in den letzten X Sekunden automatisch in die **Gelbphase**
4. **Zeit abgelaufen** – kurzes rotes Signal am Ende der Passe
5. Automatischer Übergang zur nächsten Gruppe / nächsten Passe, bis alle Passen **fertig** sind

Die App läuft im Vollbild und ist für die Anzeige auf einem Tablet (oder Smartphone) 
gedacht, das am Rand der Schießbahnen aufgestellt wird.

## Features

- ⏱️ Frei konfigurierbare Zeiten (Schießzeit, Vorwarnzeit, Gelbphase, Anzahl Passen)
- 👥 Gruppen-Modi: keine Gruppen, A-B, oder A-B + C-D nacheinander
- 🔊 Akustische Signale (Hupe/Ton) bei Phasenwechseln, optional zusätzlicher Ton beim Wechsel in die Gelbphase
- 🎮 Steuerung per Bluetooth-Clicker (frei zuweisbare Tasten):
  - **Taste A** – Start / Fortsetzen nach Alarm
  - **Taste B** – Phase manuell beenden (Einzelklick), **Alarm auslösen** (Doppelklick)
- 👆 Touch-Fallback, falls kein BT-Clicker zur Hand ist (linke Hälfte = Taste A, rechte Hälfte = Taste B, ohne Alarm-Funktion)
- 📋 Vordefinierte Presets für gängige Szenarien (Halle, Freiluft, Freiluft mit zwei Gruppen)
- 📴 Funktioniert komplett offline dank Service Worker – ideal für die Schießwiese ohne Netzempfang
- 💾 Einstellungen werden lokal gespeichert (localStorage) und bleiben nach dem Schließen erhalten
- 🔒 Wake Lock, damit der Bildschirm während des Turniers nicht einschläft

## Installation & Nutzung

Die App braucht keinen Server-Unterbau, sie besteht aus reinem HTML/CSS/JS. Die Seite wird einmal im Browser
aufgerufen und dann auf den Home-Bildschirm gespeichert, fertig.

Wer die App selbst installieren möchte:

1. Repository klonen oder Dateien herunterladen
2. Über einen einfachen HTTP-Server in einem Unterverzeichnis ausliefern
3. (Service Worker & PWA-Installation brauchen `https://` bzw. `localhost`), z. B.:
   ```bash
   npx serve .
   ```
4. Dann geht es weiter wie oben beschrieben: Im Browser öffnen und **„Zum Home-Bildschirm hinzufügen"** wählen, um die App im
5. Vollbild-/Standalone-Modus zu nutzen (empfohlen für den Einsatz auf der Wiese)

### Setup vor dem Turnier

1. Preset wählen oder Zeiten manuell eintragen
2. Gruppen-Modus festlegen
3. Bluetooth-Clicker koppeln und Tasten über „zuweisen" belegen
4. Optional Touch-Fallback aktivieren
5. „Turnier starten" tippen

## Projektstruktur

```
├── index.html       # komplette App (Markup, Styles, Logik)
├── manifest.json    # PWA-Manifest
├── sw.js            # Service Worker (Cache-first, Offline-Betrieb)
├── icon-192.png     # App-Icon (192×192)
├── icon-512.png     # App-Icon (512×512)
└── honk.m4a         # Signalton
```

## Technischer Hintergrund

- Kein Framework, kein Build-Schritt – reines Vanilla JS
- State Machine steuert die Phasenübergänge (`bereit → warning → shooting →
  yellow → red_end → fertig`)
- Audio-Freigabe erfolgt bewusst über eine Nutzergeste (Tippen auf „Turnier starten"),
  um iOS/Safari-Einschränkungen bei programmatischer Wiedergabe zu umgehen

## Lizenz

[MIT](LICENSE)
