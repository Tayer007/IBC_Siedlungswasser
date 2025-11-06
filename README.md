# IBC Abwasserbehandlungssystem

**Kooperationsprojekt Elektrotechnik und Siedlungswasserwirtschaft**
Hochschule Koblenz

Ein modulares, IoT-fähiges Abwasserbehandlungssystem, das Standard-IBC-Container (Intermediate Bulk Container) in funktionsfähige Kläranlagen für dezentrale Sanitärlösungen umrüstet.

## Projektübersicht

Dieses Projekt entwickelt ein vollständiges Steuerungs- und Überwachungssystem für eine IBC-basierte Abwasserbehandlungsanlage. Das System bietet:

- **Automatisierter mehrphasiger Behandlungsprozess** (Füllen → Belüften → Absetzen → Ablassen)
- **Echtzeit-Überwachungsdashboard** mit Live-Datenvisualisierung
- **Fernsteuerungsfähigkeiten** über Weboberfläche
- **Hardware-Abstraktion**, die Entwicklung unter Windows und Einsatz auf Raspberry Pi ermöglicht
- **Datenprotokollierung und Analytik** für Leistungsanalyse
- **Sicherheitsfunktionen** einschließlich Not-Aus und Alarmsystemen

## Architektur

```
┌─────────────────────────────────────────────────────────────┐
│                  Web-Dashboard (React)                       │
│  • Echtzeitüberwachung  • Bedienfeld  • Datenanalyse       │
└────────────────┬────────────────────────────────────────────┘
                 │ WebSocket + REST API
┌────────────────┴────────────────────────────────────────────┐
│                  Backend-Server (Flask)                      │
│  • WebSocket-Server  • REST API  • Datenprotokollierung    │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────────────────┐
│           Behandlungssteuerung (Python)                      │
│  • Phasenverwaltung  • Sicherheitsüberwachung  • Events    │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────────────────┐
│          Hardware-Abstraktionsschicht (GPIO)                 │
│  • Mock-Modus (Windows)  • GPIO-Modus (Raspberry Pi)        │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────────────────┐
│              Physikalische Komponenten (IBC)                 │
│  • Pumpen (Zulauf/Umwälzung/Ablauf)  • Luftgebläse         │
│  • Ultraschall-Füllstandssensor  • Relais-Steuermodul      │
└─────────────────────────────────────────────────────────────┘
```

## Projektstruktur

```
praxis/
├── backend/                    # Python Flask Backend
│   ├── app.py                  # Hauptanwendung
│   ├── config/                 # Konfigurationsdateien
│   │   └── treatment_config.yaml
│   ├── controller/             # Behandlungsprozesslogik
│   │   └── treatment_controller.py
│   ├── database/               # Datenbankmodelle
│   │   └── models.py
│   ├── hardware/               # Hardware-Abstraktion
│   │   └── gpio_interface.py
│   ├── requirements.txt
│   ├── .env                    # Umgebungsvariablen
│   └── README.md
│
├── frontend/                   # React Web-Dashboard
│   ├── src/
│   │   ├── App.jsx             # Hauptanwendung
│   │   ├── components/         # React-Komponenten
│   │   │   ├── StatusCard.jsx
│   │   │   ├── ControlPanel.jsx
│   │   │   ├── WaterLevelChart.jsx
│   │   │   ├── ComponentStatus.jsx
│   │   │   ├── PhaseTimeline.jsx
│   │   │   ├── Statistics.jsx
│   │   │   └── EventLog.jsx
│   │   └── main.jsx
│   ├── package.json
│   ├── vite.config.js
│   └── index.html
│
├── docs/                       # Dokumentation
│   └── lab_manual.md           # Laborpraktikumsanleitung
│
└── README.md                   # Diese Datei
```

## Schnellstart (Windows-Entwicklung)

### Voraussetzungen

- Python 3.8+
- Node.js 16+ und npm
- Git

### 1. Klonen und Backend einrichten

```bash
cd backend

# Virtuelle Umgebung erstellen
python -m venv venv

# Virtuelle Umgebung aktivieren
# Unter Windows:
venv\Scripts\activate
# Unter Linux/Mac:
source venv/bin/activate

# Abhängigkeiten installieren
pip install -r requirements.txt

# Backend-Server starten
python app.py
```

Das Backend startet auf `http://localhost:5000`

### 2. Frontend einrichten (Neues Terminal)

```bash
cd frontend

# Abhängigkeiten installieren
npm install

# Entwicklungsserver starten
npm run dev
```

Das Frontend startet auf `http://localhost:3000`

### 3. Auf das Dashboard zugreifen

Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`

Sie sollten das IBC-Abwasserbehandlungssystem-Dashboard sehen mit:
- Echtzeit-Systemstatus
- Bedienfeld (Start/Stopp/Pause/Not-Aus)
- Live-Wasserstand-Diagramm
- Komponentenstatusanzeigen
- Phasen-Zeitachse
- Ereignisprotokoll und Statistiken

## Behandlungsprozess

Das System verwaltet automatisch einen 4-Phasen-Behandlungszyklus:

### Phase 1: Füllen (Variable Dauer)
- **Ziel**: Tank auf Betriebsniveau füllen
- **Aktive Komponenten**: Zulaufpumpe
- **Abschluss**: Wenn Wasserstand Zielwert erreicht (30cm vom Sensor)
- **Timeout**: 10 Minuten

### Phase 2: Belüftung (2 Stunden)
- **Ziel**: Biologische Behandlung durch Belüftung
- **Aktive Komponenten**: Luftgebläse (kontinuierlich), Umwälzpumpe (intermittierend)
- **Umwälzung**: 1 Minute alle 15 Minuten
- **Zweck**: Sauerstoffversorgung für aerobe Bakterien

### Phase 3: Absetzen (1 Stunde)
- **Ziel**: Feststoffe absetzen lassen
- **Aktive Komponenten**: Keine (alle aus)
- **Zweck**: Schwerkraft-Trennung der Biomasse

### Phase 4: Ablassen (Variable Dauer)
- **Ziel**: Behandeltes Abwasser ableiten
- **Aktive Komponenten**: Ablaufpumpe
- **Abschluss**: Wenn Wasserstand Zielwert erreicht (100cm vom Sensor)
- **Timeout**: 10 Minuten

## Funktionen

### Web-Dashboard
- **Echtzeitüberwachung**: Live-Updates alle 10 Sekunden via WebSocket
- **Bedienfeld**: Start-, Stopp-, Pause-, Fortsetzen- und Not-Aus-Funktionen
- **Datenvisualisierung**: Interaktive Diagramme mit Wasserstand-Trends
- **Komponentenstatus**: Visuelle Anzeigen für alle Pumpen und Gebläse
- **Phasenfortschritt**: Zeitachse zeigt aktuelle Behandlungsphase
- **Ereignisprotokollierung**: Umfassendes Protokoll aller Systemereignisse
- **Statistiken**: Zyklusanzahl, Laufzeit und Fehlerverfolgung

### Backend-System
- **RESTful API**: Vollständige Steuerung und Datenzugriff über HTTP-Endpunkte
- **WebSocket-Server**: Echtzeitkommunikation in beide Richtungen
- **Behandlungssteuerung**: Autonome Mehrphasen-Zyklusverwaltung
- **Sicherheitsüberwachung**: Füllstandsalarme, Timeout-Schutz, Not-Aus
- **Datenprotokollierung**: SQLite-Datenbank für Sensorwerte und Ereignisse
- **Hardware-Abstraktion**: Nahtloser Wechsel zwischen Mock- und echtem GPIO

### Sicherheitsfunktionen
- **Not-Aus**: Sofortiges Abschalten aller Komponenten
- **Hoch-/Niedrig-Füllstandsalarme**: Automatisches Abschalten bei gefährlichen Wasserständen
- **Timeout-Schutz**: Maximale Laufzeitgrenzen für alle Komponenten
- **Zyklusdauerbegrenzung**: 12-Stunden-Maximum für Zyklus-Timeout
- **Pumpenschutz**: Mindestintervall zwischen Pumpenstarts

## Konfiguration

Bearbeiten Sie `backend/config/treatment_config.yaml` zur Anpassung:

```yaml
treatment_phases:
  filling:
    target_level: 30        # cm vom Sensor
    max_duration: 600       # Sekunden
  aeration:
    duration: 7200          # 2 Stunden
    recirculation_interval: 900
    recirculation_duration: 60
  settling:
    duration: 3600          # 1 Stunde
  draining:
    target_level: 100       # cm vom Sensor
    max_duration: 600

safety:
  high_level_alarm: 15      # cm (sehr hoher Wasserstand)
  low_level_alarm: 120      # cm (sehr niedriger Wasserstand)
  max_cycle_duration: 43200 # 12 Stunden

logging:
  interval: 10              # Sekunden
  retain_days: 30
```

## API-Endpunkte

### Steuerung
- `POST /api/control/start` - Behandlungszyklus starten
- `POST /api/control/stop` - Behandlungszyklus stoppen
- `POST /api/control/pause` - Aktuellen Zyklus pausieren
- `POST /api/control/resume` - Pausierten Zyklus fortsetzen
- `POST /api/control/emergency-stop` - Not-Abschaltung
- `POST /api/control/reset-emergency` - Not-Aus zurücksetzen
- `POST /api/control/component` - Manuelle Komponentensteuerung

### Überwachung
- `GET /api/status` - Aktueller Systemstatus
- `GET /api/data/readings` - Sensorwert-Historie
- `GET /api/data/events` - Systemereignis-Protokoll
- `GET /api/data/cycles` - Behandlungszyklus-Historie

## Einsatz auf Raspberry Pi

### Hardware-Anforderungen
- Raspberry Pi 4 (empfohlen) oder Pi 3B+
- 4-Kanal-Relaismodul
- HC-SR04 Ultraschall-Abstandssensor
- 12V/24V Pumpen (Zulauf, Umwälzung, Ablauf)
- Luftgebläse
- Stromversorgung

### Software-Bereitstellung

1. **Dateien übertragen**
```bash
# Von Ihrem Entwicklungsrechner
scp -r backend/ pi@raspberrypi.local:~/ibc-treatment/
scp -r frontend/dist pi@raspberrypi.local:~/ibc-treatment/
```

2. **Konfiguration aktualisieren**
```bash
# Auf dem Raspberry Pi
cd ~/ibc-treatment/backend
nano .env
# Ändern: HARDWARE_MODE=gpio
```

3. **GPIO-Verbindungen verkabeln** (gemäß Konfiguration)
- GPIO 17: Zulaufpumpe
- GPIO 27: Umwälzpumpe
- GPIO 22: Ablaufpumpe
- GPIO 23: Luftgebläse
- GPIO 24: Ultraschall-Trigger
- GPIO 25: Ultraschall-Echo

4. **Backend ausführen**
```bash
cd ~/ibc-treatment/backend
python3 app.py
```

5. **Frontend bereitstellen** (nginx oder ähnlich verwenden)

## Entwicklung vs. Produktion

### Entwicklung (Windows/Mac)
- **Hardware-Modus**: `mock` (simuliertes GPIO)
- **Zweck**: Entwicklung, Tests, Demonstration
- **Funktionen**: Volle Funktionalität mit simulierten Wasserstandsänderungen

### Produktion (Raspberry Pi)
- **Hardware-Modus**: `gpio` (echte GPIO-Steuerung)
- **Zweck**: Tatsächliche Abwasserbehandlung
- **Anforderungen**: Ordnungsgemäße elektrische Verkabelung, Sicherheitsmaßnahmen

## Tests

### Backend-Tests
```bash
cd backend
pytest tests/
```

### Frontend-Tests
```bash
cd frontend
npm test
```

### Manuelle Test-Checkliste
- [ ] Start/Stopp-Zyklusfunktionalität
- [ ] Pausieren/Fortsetzen während jeder Phase
- [ ] Not-Aus-Aktivierung und -Zurücksetzung
- [ ] Manuelle Komponentensteuerung (wenn nicht laufend)
- [ ] WebSocket-Verbindungsstabilität
- [ ] Datenprotokollierung und -abruf
- [ ] Phasenübergänge
- [ ] Sicherheitsalarmauslösung

## Laborpraktische Verwendung

Dieses System ist für den Einsatz in Laborpraktika an der Hochschule Koblenz konzipiert:

1. **Verstehen von Abwasserbehandlungsprozessen**
2. **Lernen von Prozesssteuerung und Automatisierung**
3. **Üben von Sensorintegration und -überwachung**
4. **Erleben von Echtzeit-Datenvisualisierung**
5. **Erkunden von IoT- und Fernsteuerungskonzepten**

Siehe `docs/lab_manual.md` für vollständige Laboranweisungen.

## Fehlerbehebung

### Backend startet nicht
- Python-Version überprüfen (3.8+)
- Sicherstellen, dass virtuelle Umgebung aktiviert ist
- Alle Abhängigkeiten überprüfen: `pip install -r requirements.txt`
- Prüfen, ob Port 5000 nicht verwendet wird

### Frontend verbindet sich nicht
- Überprüfen, ob Backend auf Port 5000 läuft
- Browser-Konsole auf Fehler prüfen
- Sicherstellen, dass WebSocket-Verbindung durch Firewall erlaubt ist
- Browser-Cache leeren versuchen

### GPIO-Fehler auf Pi
- Mit sudo ausführen: `sudo python3 app.py`
- GPIO-Pin-Verbindungen überprüfen
- BCM-Pin-Nummerierung in Konfiguration verifizieren
- Sicherstellen, dass kein anderer Prozess GPIO verwendet

## Zukünftige Erweiterungen

- [ ] pH- und Gelöstsauerstoff-Sensoren
- [ ] Erweiterte Planung (zeitbasierte Zyklen)
- [ ] Mobile App (React Native)
- [ ] Mehrfach-Tank-Verwaltung
- [ ] Machine Learning zur Optimierung
- [ ] Cloud-Datensicherung
- [ ] E-Mail/SMS-Benachrichtigungen
- [ ] Energieverbrauchsüberwachung

## Autoren

**Praxisphase-Projekt**
Hochschule Koblenz - University of Applied Sciences
Fachbereich Ingenieurwesen

**Kooperation zwischen:**
- Elektrotechnik (Electrical Engineering)
- Siedlungswasserwirtschaft (Water Resources Management)

## Lizenz

Dieses Projekt wurde für Bildungszwecke an der Hochschule Koblenz entwickelt.

## Danksagungen

- Hochschule Koblenz für die Projektunterstützung
- Fakultätsberater und Laborpersonal
- Open-Source-Community für Werkzeuge und Bibliotheken

---

Bei Fragen oder Problemen wenden Sie sich bitte an den Projektbetreuer der Hochschule Koblenz.
