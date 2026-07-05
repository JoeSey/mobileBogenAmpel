# Handoff: Bogen-Ampel (Trainingsturnier-Signalanlage)

Stand: 03.07.2026. Projekt pausiert wegen Token-Budget, hier der komplette Kontext zum Weitermachen.

## Ziel

Web-App für iPad (Stativ neben den Schießbahnen), zwei Ansichten:
1. **Setup-Screen**: Zeiten, Passen, Gruppen konfigurieren, Presets laden.
2. **Display-Screen**: ganzseitig einfarbig (Rot/Grün/Gelb) nach WA-Ampel-Logik + passende Signaltöne über Bluetooth-Boombox.

Muss offline laufen (PWA, Service Worker), da auf der Schießwiese kein Internet.

## Getroffene Entscheidungen (verbindlich, nicht neu verhandeln)

**Architektur:** Ein iPad, keine zweite Gerät/Netzwerk-Kopplung. Steuerung über einen **2-Tasten-Bluetooth-Clicker** (sendet Tastatur-Events, Keycodes unbekannt/modellabhängig → App braucht eine "Taste zuweisen"-Funktion, die per Knopfdruck den nächsten `KeyboardEvent.code` einfängt statt Codes hart zu kodieren).

**Tastenbelegung:**
- Taste A: Start (auch: Fortsetzen nach Alarm, Countdown läuft von der pausierten Zeit weiter)
- Taste B, 1× Klick: laufende Phase vorzeitig beenden
- Taste B, 2× Klick schnell (< 500 ms Fenster, Timer-basiert erkannt): **Alarm** → Countdown pausiert sofort, Bildschirm wird rot, 5 kurze Signaltöne. Bewusst *nicht* über "lang drücken" gelöst, weil billige BT-Clicker Press-Dauer oft nicht zuverlässig durchreichen (**Spekulation/Annahme, nicht verifiziert** – falls das Zielgerät bekannt ist, gegenchecken).
- Fallback ohne Clicker: Touch auf Bildschirm (optional, per Setup-Toggle aktivierbar, Default aus wegen Fehltrigger-Gefahr durch Regen/Insekten). Linke Bildschirmhälfte = Taste-A-Aktion, rechte Hälfte = Taste-B-Einzelklick-Aktion. Keine Alarm-Funktion über Touch.

**WA-Signalfolge** (Quelle: Web-Recherche zu World-Archery-Regeln, **nicht 1:1 aus dem aktuellen offiziellen Rulebook zitiert** – vor Wettkampfeinsatz mit aktuellem DSB/WA-Regelheft gegenchecken):
1. 2 Signaltöne → aktive Gruppe (A/B oder C/D) an die Linie, Pfeil einnocken, Bogen noch nicht heben. Dauer konfigurierbar (Default 10 s), Bildschirm rot mit Label „An die Linie".
2. 1 Signalton → Grün, Schießzeit-Countdown startet.
3. Letzte 30 Sekunden (konfigurierbar) → Gelb. Ob hier ein zusätzlicher Ton hingehört, ist **unklar/Annahme meinerseits** (ein kurzer Ton als Cue eingeplant, aber optional/abschaltbar machen).
4. Zeit abgelaufen → Rot, 3 Signaltöne bzw. länger, sofort Schuss einstellen.
5. Bei zwei Gruppen (A/B + C/D): nach Ablauf der Rot-Phase für A/B geht's zurück in „Bereit"-Zustand, Kampfrichter startet C/D explizit erneut über Taste A (kein Autostart – bewusste Entscheidung für Kontrolle).
6. Nach beiden Gruppen: Passen-Zähler hoch, nächste Passe wartet auf Taste A. Nach letzter Passe: „Fertig"-Screen.

**Presets/Setup-Felder** (aus ursprünglichem Wunsch, Details noch offen): Schießzeit pro Passe, Vorwarnzeit (Default 30 s), Anzahl Passen, Gruppen-Modus (keine / A-B / A-B+C-D), gängige Presets (z. B. „Halle 3 Pfeile/2 Min", „Freiluft 6 Pfeile/4 Min") vorschlagen, mit Custom-Option.

## Technische Eckpunkte

- **Kein echtes Fullscreen-API auf iPad Safari** für beliebige Elemente. Lösung: App als PWA installieren ("Zum Home-Bildschirm hinzufügen") → Standalone-Modus ohne Safari-Leisten. App sollte erkennen, ob sie standalone läuft (`navigator.standalone` bzw. `matchMedia('(display-mode: standalone)')`) und sonst einen Hinweis-Banner zeigen.
- **Audio-Unlock**: iOS blockt Autoplay ohne Geste. Erster Ton wird durch Tap auf "Turnier starten" ausgelöst (AudioContext dort erzeugen/resumen), danach dürfen programmatisch getriggerte Sounds im Ablauf folgen.
- **Töne per Web Audio API synthetisieren** (Oszillator, kein Audio-Dateien nötig) → vereinfacht Offline-Caching massiv, da keine Asset-Dateien gecacht werden müssen.
- **Wake Lock API** hält Bildschirm an, aber erst ab iOS Safari 16.4 unterstützt. Bei älteren iPads ggf. Fallback nötig (z. B. stummes Loop-Video als Wachhalte-Trick) — noch nicht implementiert, iPad-iOS-Version bei Joe unbekannt.
- **State-Persistenz**: Konfiguration/Zustand in `localStorage`, falls Tab/App neu lädt.
- **Timer-Engine**: Phasenwechsel über Timestamp-Differenz (`Date.now()`), nicht per Countdown-Dekrement, um Drift zu vermeiden. Bei Alarm: verbleibende Zeit der pausierten Phase merken, bei Fortsetzen neuen virtuellen Startzeitpunkt setzen.

## Bereits erstellte Dateien

Liegen unter `/home/claude/bogen-ampel/` (noch nicht nach `/mnt/user-data/outputs` kopiert oder präsentiert):

- `manifest.json` — fertig.
- `sw.js` — fertig, Cache-first-Strategie, cached `index.html`, `manifest.json`, beide Icons.
- `icon-192.png`, `icon-512.png` — fertig, simples Zielscheiben-Icon generiert.
- **`index.html` — NOCH NICHT ERSTELLT.** Das ist der Hauptteil: Setup-Screen, Display-Screen, komplette State-Machine, Tastenzuordnung-UI, Touch-Fallback, Web-Audio-Beeps, Wake Lock, Standalone-Erkennung.

## Nächste Schritte

1. `index.html` bauen (HTML/CSS/JS in einer Datei, siehe Struktur oben).
2. Lokal/im Container grob durchtesten (State-Machine-Logik, Timer-Berechnung).
3. Alles nach `/mnt/user-data/outputs/bogen-ampel/` kopieren, `present_files` aufrufen.
4. Joe testet auf echtem iPad + seinem konkreten Bluetooth-Clicker — insbesondere die als **Annahme markierten Punkte** (Doppelklick-Erkennung, WA-Signalfolge-Details, Wake-Lock-Verhalten je nach iOS-Version) verifizieren.
