# Obsidian-Agent – Installationsanleitung

> Lokale Weboberfläche zur Verwaltung eines Obsidian-Vaults mit SQLite-Planer und täglicher Tagesübersicht um 04:00.
> Die Oberfläche läuft ausschliesslich auf dem eigenen Rechner unter `http://127.0.0.1:5055`.

---

## Inhalt

1. [Überblick](#1-überblick)
2. [Voraussetzungen](#2-voraussetzungen) – inkl. [Obsidian-Plugins](#21-obsidian-plugins-zuerst-installieren)
3. [Installation unter Windows](#3-installation-unter-windows)
4. [Installation unter macOS](#4-installation-unter-macos)
5. [Installation unter Linux](#5-installation-unter-linux)
6. [Erster Start und Grundkonfiguration](#6-erster-start-und-grundkonfiguration)
7. [Installation prüfen](#7-installation-prüfen)
8. [Konfiguration (`config.json`)](#8-konfiguration-configjson)
9. [Deinstallation](#9-deinstallation)
10. [Umzug auf einen anderen Ordner oder PC](#10-umzug-auf-einen-anderen-ordner-oder-pc)
11. [Fehlerbehebung](#11-fehlerbehebung)
12. [Kurzreferenz](#12-kurzreferenz)

---

## 1. Überblick

**Grundprinzip:** Die Markdown-Dateien im Vault sind die einzige Wahrheit. Die SQLite-Datenbank `planer.db` ist nur Index und Planer und wird bei jedem Sync aus dem Vault neu aufgebaut. Du kannst also jederzeit direkt in Obsidian weiterarbeiten – der Agent liest beim nächsten Sync alles wieder ein.

**Lieferumfang des ZIP-Archivs:**

| Datei / Ordner | Zweck |
|---|---|
| `installieren.bat` (+ `installieren.ps1`) | Windows: komplette Installation inkl. Hintergrund-Aufgabe und Tages-Sync |
| `start.bat` | Windows: Weboberfläche öffnen (startet den Agent bei Bedarf im Hintergrund) |
| `hintergrund.pyw` | Windows: Start der Oberfläche ohne Fenster (wird von der Aufgabe benutzt) |
| `deinstallieren.bat` | Windows: Deinstallation mit Backup |
| `mac/installieren_mac.command` | macOS: Installation inkl. launchd-Sync und Autostart |
| `mac/starten_mac.command` | macOS: Start |
| `mac/deinstallieren_mac.command` | macOS: Deinstallation mit Backup |
| `start.sh` | Linux (und macOS manuell): Start |
| `agent.py` | Kernlogik und Kommandozeile |
| `app.py` | Weboberfläche (Flask) |
| `requirements.txt` | Python-Abhängigkeit (`flask>=3.0`) |
| `templates/`, `static/` | HTML-Vorlagen und Stylesheet der Oberfläche |

---

## 2. Voraussetzungen

| | Windows | macOS | Linux |
|---|---|---|---|
| Python | 3.10 oder neuer | 3.9 oder neuer | 3.9 oder neuer, mit `venv` |
| Internet | nur bei der Installation (Flask wird via `pip` geladen) | dito | dito |
| Obsidian | ein bestehender Vault (Obsidian selbst muss nicht laufen) | dito | dito |
| Rechte | normale Benutzerrechte genügen | dito | dito |

> ⚠️ **Wichtig:** Den Programmordner `obsidian-agent` **nicht in den Vault** entpacken. Er gehört an einen eigenen Ort, z. B. `C:\Group\obsidian-agent` oder `~/Programme/obsidian-agent`.

### 2.1 Obsidian-Plugins zuerst installieren

Vor der Installation des Agents müssen in Obsidian diese zwei Community-Plugins installiert und aktiviert sein:

| Plugin | Zweck | Link | Hinweise |
|---|---|---|---|
| **Timeline Calendar** | Kalender mit Monats-, Wochen-, Tages- und Agenda-Ansicht für Daily Notes und Tasks | [community.obsidian.md/plugins/timeline-calendar](https://community.obsidian.md/plugins/timeline-calendar) | ab Obsidian 1.7.2 · Desktop und Mobile |
| **Tasks Timeline** | Tasks des Vaults auf einer Zeitachse anzeigen und bearbeiten | [community.obsidian.md/plugins/tasks-timeline](https://community.obsidian.md/plugins/tasks-timeline) | ab Obsidian 1.11.4 · **nur Desktop** |

**So installierst du die Plugins:**

1. Obsidian öffnen und den Vault laden.
2. **Einstellungen → Community-Plugins** öffnen.
3. Falls aktiv: **Eingeschränkten Modus ausschalten** („Community-Plugins aktivieren“).
4. **Durchsuchen** klicken, nach `Timeline Calendar` suchen → **Installieren** → **Aktivieren**.
5. Dasselbe für `Tasks Timeline` wiederholen.
6. Kontrolle: Beide Plugins erscheinen unter *Einstellungen → Community-Plugins → Installierte Plugins* mit aktiviertem Schalter.

> 💡 Alternativ lässt sich jedes Plugin direkt über den Button **„Add to Obsidian“** auf der jeweiligen Plugin-Seite installieren.

> ℹ️ Tasks Timeline läuft nur in der Desktop-App. Seine optionalen KI- und Spracheingabe-Funktionen brauchen einen eigenen Anbieter (OpenAI oder Gemini) und sind für den Agent nicht nötig.

---

## 3. Installation unter Windows

Unter Windows richtet **ein einziges Skript** alles ein: `installieren.bat`. Danach läuft die Weboberfläche als **Aufgabe im Hintergrund**. Sie startet nach jedem Neustart und jeder Anmeldung von selbst, ohne Konsolenfenster. Administratorrechte sind nicht nötig.

### 3.1 Python installieren

1. Python 3.10+ von <https://www.python.org/downloads/> herunterladen.
2. Im Installer unbedingt **„Add python.exe to PATH“** anhaken.
3. Kontrolle in einer neuen Eingabeaufforderung:

   ```bat
   py -3 --version
   ```

### 3.2 Programm entpacken

ZIP-Archiv entpacken, zum Beispiel nach:

```
C:\Group\obsidian-agent
```

### 3.3 Installieren

1. Läuft noch ein altes `start.bat`-Fenster, dieses schliessen.
2. **`installieren.bat`** doppelklicken. Das Skript

   | Schritt | Was passiert |
   |---|---|
   | Python-Umgebung | Sucht Python ≥ 3.10, legt `.venv` an und installiert Flask. Fehlt Python, öffnet sich python.org. |
   | Vault | Fragt per Ordner-Dialog nach dem Vault, falls noch keiner gesetzt ist. Der Dialog kann hinter dem Fenster aufgehen. Danach folgt der erste Sync. |
   | Aufgabe **„Obsidian-Agent Weboberflaeche“** | Startet die Oberfläche **bei jeder Anmeldung im Hintergrund**. Sie läuft ohne Fenster und ohne Zeitlimit, auch im Akkubetrieb. Stürzt sie ab, startet sie bis zu dreimal neu. |
   | Aufgabe **„Obsidian-Agent Tages-Sync“** | Führt täglich zur `sync_zeit` (Standard 04:00) den Sync aus. War der PC zu dieser Zeit aus, holt Windows den Lauf automatisch nach. |
   | Start & Kontrolle | Startet den Agent sofort, prüft, ob er antwortet, und öffnet den Browser. |

3. Am Ende steht `[OK] Fertig! Weboberflaeche: http://127.0.0.1:5055`.

> 💡 `installieren.bat` kann jederzeit erneut ausgeführt werden, zum Beispiel nach einer Änderung der Sync-Zeit oder des Ports. Bestehende Einträge werden aktualisiert, und es geht nichts verloren.

> ℹ️ Lässt die Firmenrichtlinie keine geplante Aufgabe zu, legt das Skript stattdessen eine Verknüpfung im persönlichen Autostart-Ordner an (`shell:startup`). Die Oberfläche startet dann ebenfalls bei der Anmeldung im Hintergrund.

### 3.4 Öffnen

- Lesezeichen auf **`http://127.0.0.1:5055`** setzen. Das ist der normale Weg, denn der Agent läuft ja im Hintergrund.
- Oder **`start.bat`** doppelklicken. Läuft der Agent, öffnet sich nur der Browser. Läuft er nicht, startet `start.bat` ihn im Hintergrund. Ist er noch gar nicht installiert, startet die Installation.

### 3.5 Agent beenden oder neu starten

Der Hintergrund-Agent hat kein Fenster. Zum Beenden oder Neustarten:

```bat
schtasks /End /TN "Obsidian-Agent Weboberflaeche"
schtasks /Run /TN "Obsidian-Agent Weboberflaeche"
```

Das geht auch über die **Aufgabenplanung** (`taskschd.msc`): Aufgabe anklicken, dann *Beenden* oder *Ausführen*.

**Protokoll der Weboberfläche:** `%LOCALAPPDATA%\obsidian-agent\gui.log`

## 4. Installation unter macOS

### 4.1 Programm entpacken

ZIP-Archiv entpacken, zum Beispiel nach:

```
~/Programme/obsidian-agent
```

### 4.2 Installer ausführen

1. Im Ordner `mac/` die Datei **`installieren_mac.command`** doppelklicken.
2. Meldet macOS *„nicht verifizierter Entwickler“*: Rechtsklick → **Öffnen** → **Öffnen**.

Der Installer führt dich durch folgende Schritte:

| Schritt | Was passiert |
|---|---|
| Quarantäne | Entfernt das Download-Flag und macht die Skripte ausführbar. |
| Python | Sucht Python ≥ 3.9. Fehlt es, öffnet sich python.org (alternativ `brew install python`). Danach den Installer erneut starten. |
| Python-Umgebung | Legt `.venv` an und installiert Flask. |
| Vault | Fragt per Ordner-Dialog nach dem Vault (nur, wenn noch kein gültiger Pfad in `config.json` steht). |
| Erster Sync | Liest den Vault ein. macOS fragt ggf. nach dem Ordnerzugriff – **erlauben**. |
| Täglicher Sync | Richtet den launchd-Job `ch.obsidian-agent.sync` ein. Er läuft auch ohne offene Oberfläche; schläft der Mac um 04:00, holt macOS den Lauf beim Aufwachen nach. |
| Autostart | Frage *„Agent bei jeder Anmeldung automatisch im Hintergrund starten?“* – bei **j** entsteht der Job `ch.obsidian-agent.gui`. |
| Selbsttest | Startet den Hintergrund-Sync einmal und prüft, ob macOS den Zugriff blockiert. |

Am Ende kannst du den Agent direkt öffnen.

### 4.3 Festplattenvollzugriff (nur falls nötig)

Liegt der Vault in **Dokumente**, auf dem **Schreibtisch** oder in **iCloud Drive** (Standard-Speicherort von Obsidian auf dem Mac), kann macOS den Hintergrund-Sync blockieren. Der Installer erkennt das und zeigt den Pfad zur Python-Datei an. Dann:

1. **Systemeinstellungen → Datenschutz & Sicherheit → Festplattenvollzugriff** öffnen (der Installer öffnet die Seite automatisch).
2. Auf **„+“** klicken, mit `Cmd+Shift+G` den angezeigten Pfad einfügen und die Datei hinzufügen.
3. `installieren_mac.command` erneut ausführen, um den Test zu wiederholen.

### 4.4 Starten

Doppelklick auf **`mac/starten_mac.command`**. Läuft der Agent schon im Hintergrund (Autostart), öffnet sich nur der Browser.

> 💡 Wird die Sync-Zeit in den Einstellungen geändert, `installieren_mac.command` einfach nochmals ausführen. Dabei wird nichts überschrieben.

**Protokolle:** `~/Library/Logs/obsidian-agent/` (`sync.log`, `gui.log`)

---

## 5. Installation unter Linux

### 5.1 Voraussetzungen

```bash
# Debian / Ubuntu
sudo apt install python3 python3-venv
```

### 5.2 Entpacken und starten

```bash
unzip obsidian-agent.zip -d ~/programme
cd ~/programme/obsidian-agent
chmod +x start.sh
./start.sh
```

Beim ersten Start legt `start.sh` die `.venv` an und installiert Flask. Danach im Browser `http://127.0.0.1:5055` öffnen und unter **Einstellungen** den Vault-Pfad eintragen.

### 5.3 Täglichen Sync per cron einrichten

```bash
crontab -e
```

Folgende Zeile ergänzen (Pfad anpassen):

```cron
0 4 * * * cd /home/<benutzer>/programme/obsidian-agent && .venv/bin/python agent.py sync --geplant
```

---

## 6. Erster Start und Grundkonfiguration

1. Oberfläche öffnen: `http://127.0.0.1:5055`.
2. **Einstellungen** → Vault-Pfad eintragen → **Speichern & einlesen**.
3. Beim ersten Sync legt der Agent im Vault diese Grundstruktur an (bestehende Inhalte bleiben unangetastet):

   ```
   <Vault>/
   ├── _Vorlagen/       Notiz-Vorlagen (Task, Meeting, ADR, Runbook, Change, Störung, …)
   ├── Projekte/
   ├── Tasks/
   ├── Notizen/
   ├── Wiki/            Kategorien = Unterordner, inkl. automatisch gepflegtem _Index.md
   ├── LifeCycle/
   ├── DevOps/
   ├── Ideenbox/
   ├── Team/
   ├── JourFixe/        Jour-Fixe-Serien mit Sitzungen und Pendenzen
   └── Planer/
       ├── Heute.md
       └── Archiv/Tagesuebersicht_JJJJ-MM-TT.md
   ```

4. Erstes Projekt anlegen unter **Projekte** – es erhält automatisch die Standardstruktur `01_Vertraege_und_SLA` bis `10_Korrespondenz` und eine Übersichtsnotiz `_Projekt_<Name>.md`.
5. Beim Anlegen eines Tasks mit Projekt erscheint die Auswahl **Ordner im Projekt**. Sie zeigt alle Unterordner des Projekts. Ordner, die du in Obsidian von Hand anlegst, sind sofort dabei, ohne Sync. Tasks, die schon im Vault liegen, kannst du in Obsidian einfach in einen anderen Ordner verschieben. Der Agent findet sie beim nächsten Sync.
6. Optional: Unter **Wiki** die Kategorien anlegen (Formular *Neue Kategorie*) und unter **Jour Fixe** die erste Sitzung einer Serie anlegen. Dabei entsteht die Serie.

> 💡 Jeder Titel in der Oberfläche ist ein `obsidian://`-Link und öffnet die Notiz direkt in Obsidian.

### Task-Format

Tasks sind normale Markdown-Dateien `Task_*.md` – sie können auch von Hand in Obsidian angelegt werden:

```markdown
---
typ: task
titel: Firewall-Update vorbereiten
projekt: Kunde Muster AG
start: 2026-11-18        # auch 18.11.2026 geht
ende: 2026-11-20
status: offen            # offen | in_arbeit | wartend | erledigt
prioritaet: hoch         # hoch | mittel | niedrig
verantwortlich: Max
---
- [ ] Schritt 1
- [x] Schritt 2
```

---

## 7. Installation prüfen

Im Programmordner (Windows: `.venv\Scripts\python.exe`, macOS/Linux: `.venv/bin/python`):

```bash
.venv/bin/python agent.py sync     # Vault jetzt einlesen
.venv/bin/python agent.py heute    # heutige Tasks in der Konsole
```

Erwartete Ausgabe beim Sync, z. B.:

```
Sync OK – 13 Dateien, 0 Tasks heute, 0 offen, 5 ms
```

Checkliste:

- [ ] Obsidian-Plugins **Timeline Calendar** und **Tasks Timeline** installiert und aktiviert
- [ ] Oberfläche unter `http://127.0.0.1:5055` erreichbar
- [ ] Vault-Pfad unter **Einstellungen** gesetzt
- [ ] `Planer/Heute.md` im Vault vorhanden
- [ ] Windows: Unter `taskschd.msc` gibt es die Aufgaben **„Obsidian-Agent Weboberflaeche“** und **„Obsidian-Agent Tages-Sync“**.
- [ ] Windows: Nach einem Neustart ist `http://127.0.0.1:5055` erreichbar, ohne dass du etwas startest.
- [ ] Täglicher Sync eingerichtet (Windows: Aufgabenplanung · macOS: launchd · Linux: cron)

---

## 8. Konfiguration (`config.json`)

Die Datei entsteht beim ersten Start im Programmordner und lässt sich auch über **Einstellungen** ändern.

| Schlüssel | Standard | Bedeutung |
|---|---|---|
| `vault_path` | `~/Documents/Obsidian` | Pfad zum Vault |
| `db_path` | `planer.db` | SQLite-Datei (relativ zum Programmordner oder absolut) |
| `projekt_ordner` | `Projekte` | leer = Projekte direkt im Vault-Root |
| `task_ordner` | `Tasks` | Tasks ohne Projekt |
| `wiki_ordner`, `lcm_ordner`, `notiz_ordner`, `devops_ordner`, `ideen_ordner`, `team_ordner`, `jourfixe_ordner`, `planer_ordner`, `vorlagen_ordner` | siehe Kapitel 6 | Ordnernamen im Vault |
| `task_standard_ordner` | *(leer)* | Unterordner, der beim Anlegen eines Tasks im Projekt vorgewählt ist, z. B. `06_Projekte`. Leer bedeutet: Projektordner (oberste Ebene). Fehlt der Ordner in einem Projekt, gilt ebenfalls die oberste Ebene. |
| `sync_zeit` | `04:00` | Zeit des täglichen Syncs |
| `lcm_warnung_tage` | `90` | Vorwarnzeit für EOS/EOL im LifeCycle |
| `demnaechst_tage` | `7` | Zeitraum für „Demnächst“ |
| `port` | `5055` | Port der Oberfläche |
| `eigene_projekt_ordner` | `{}` | zusätzliche Standard-Ordner für Projekte |
| `obsidian_vault_name` | *(leer)* | Nur bei Bedarf setzen. Leer bedeutet, dass die Links den vollständigen Dateipfad verwenden und Obsidian den Vault selbst findet (empfohlen). Mit einem Namen bauen die Links `vault=<Name>&file=…`. Das funktioniert nur, wenn `vault_path` genau der Vault-Ordner ist. |

> ⚠️ Nach einer Änderung von `sync_zeit` oder `port`: unter Windows `installieren.bat`, unter macOS `installieren_mac.command` erneut ausführen, unter Linux die crontab-Zeile anpassen.

---

## 9. Deinstallation

| | Windows | macOS |
|---|---|---|
| Skript | `deinstallieren.bat` | `mac/deinstallieren_mac.command` |

Das Skript

1. beendet den laufenden Agent,
2. entfernt die geplanten Aufgaben und den Autostart (Windows: „Obsidian-Agent Weboberflaeche“, „Obsidian-Agent Tages-Sync“ und eine allfällige Verknüpfung in `shell:startup`; macOS: launchd),
3. **sichert `config.json` und `planer.db`** nach `obsidian-agent-backup_<Datum>` auf dem Desktop,
4. löscht die Python-Umgebung `.venv` und – auf Nachfrage – den ganzen Programmordner.

> ✅ **Der Obsidian-Vault wird nie verändert oder gelöscht.**

**Linux:** Agent beenden (`Ctrl+C`), cron-Zeile mit `crontab -e` entfernen, `config.json` und `planer.db` sichern, Programmordner löschen.

---

## 10. Umzug auf einen anderen Ordner oder PC

Gilt auch für Windows → Mac.

1. Am alten Ort das Deinstallations-Skript ausführen.
2. Programm am neuen Ort aus dem ZIP entpacken.
3. `config.json` und `planer.db` aus dem Backup in den neuen Programmordner kopieren.
4. Installieren:
   - **Windows:** `installieren.bat`
   - **macOS:** `mac/installieren_mac.command`
5. Hat sich der Vault-Pfad geändert (z. B. von `C:\…` zu `/Users/…`), fragt der Mac-Installer automatisch nach dem Vault. Unter Windows den Pfad in den **Einstellungen** eintragen.

**Warum nicht einfach den Ordner verschieben?** Die `.venv` enthält feste Pfade, und die geplante Aufgabe zeigt auf den alten Ort. Beides wird beim Neuinstallieren korrekt angelegt.

> 💡 `planer.db` ist optional – sie wird komplett aus dem Vault neu aufgebaut. Ohne sie geht nur die Tageshistorie verloren.
> **Nur den Vault verschieben?** Kein Umzug nötig – einfach den neuen Pfad in den Einstellungen eintragen.

---

## 11. Fehlerbehebung

| Problem | Ursache / Lösung |
|---|---|
| **Weboberfläche nach Neustart nicht erreichbar** (`ERR_CONNECTION_REFUSED`) | Agent läuft nicht. `installieren.bat` (erneut) ausführen. Das richtet die Hintergrund-Aufgabe ein. Sofort-Hilfe: `start.bat`. |
| Aufgabe eingerichtet, Oberfläche trotzdem nicht erreichbar | In `taskschd.msc` die Aufgabe **„Obsidian-Agent Weboberflaeche“** prüfen: Unter *Letztes Ausführungsergebnis* muss `0x0` oder „Wird ausgeführt“ stehen. Danach `%LOCALAPPDATA%\obsidian-agent\gui.log` lesen. Wenn `start.bat` den Agent nicht starten kann, startet es ihn im Vordergrund und zeigt die Fehlermeldung an. |
| *„Aufgabenplanung nicht erlaubt“* bei der Installation | Die Firmenrichtlinie sperrt geplante Aufgaben. Der Installer legt automatisch eine Autostart-Verknüpfung an, und der Agent startet trotzdem bei der Anmeldung. Ohne Tages-Sync-Aufgabe wird die Übersicht nur aktualisiert, solange die Oberfläche läuft. |
| PowerShell-Skript wird blockiert | `installieren.bat` startet PowerShell mit `-ExecutionPolicy Bypass`. Wird auch das gesperrt, die IT fragen. |
| *„Python … wurde nicht gefunden“* (Windows) | Python 3.10+ installieren, **„Add python.exe to PATH“** anhaken, `installieren.bat` erneut starten. |
| *„nicht verifizierter Entwickler“* (macOS) | Rechtsklick → **Öffnen** → **Öffnen**. |
| *„Noch nicht installiert“* (macOS) | Zuerst `mac/installieren_mac.command` ausführen. |
| Installation von Flask schlägt fehl | Internetverbindung bzw. Proxy prüfen und Installer erneut starten. |
| Hintergrund-Sync auf dem Mac läuft nicht | Festplattenvollzugriff erteilen (siehe [4.3](#43-festplattenvollzugriff-nur-falls-nötig)); Details in `~/Library/Logs/obsidian-agent/sync.log`. |
| Port 5055 ist belegt | In `config.json` einen anderen `port` setzen. Unter Windows danach `installieren.bat` erneut ausführen. |
| *„Vault-Ordner nicht gefunden“* – Oberfläche springt immer zu den Einstellungen | Pfad unter **Einstellungen** prüfen – er muss auf den Vault-Ordner selbst zeigen (den Ordner, der `.obsidian/` enthält). |
| Sync verpasst, weil der PC um 04:00 aus war | Windows holt den Lauf automatisch nach (vom Installer so eingestellt). Sonst in der Oberfläche manuell **Sync** auslösen. |
| Klick auf einen Titel meldet in Obsidian *„Vault nicht gefunden“* | Das betraf ältere Versionen, deren Links den Ordnernamen als Vault-Namen verwendeten. Mit der aktuellen `agent.py` öffnen die Links die Datei über den vollständigen Pfad. Danach den Agent neu starten (`schtasks /End` und `/Run /TN "Obsidian-Agent Weboberflaeche"`). Die Datei muss in einem Ordner liegen, den Obsidian als Vault geöffnet hat. |
| Vorlage versehentlich gelöscht | Wird beim nächsten Sync automatisch wieder angelegt. |

---

## 12. Kurzreferenz

```bash
python agent.py sync              # Vault jetzt einlesen
python agent.py sync --geplant    # wie oben, zählt als geplanter 04:00-Lauf
python agent.py heute             # heutige Tasks in der Konsole
python agent.py projekt "Name"    # neues Projekt anlegen
```

| Aktion | Windows | macOS | Linux |
|---|---|---|---|
| Installieren | `installieren.bat` | `mac/installieren_mac.command` | `./start.sh` |
| Starten | `start.bat` | `mac/starten_mac.command` | `./start.sh` |
| Täglicher Sync | in `installieren.bat` enthalten | im Installer enthalten | `crontab -e` |
| Autostart der Oberfläche | in `installieren.bat` enthalten (Aufgabe im Hintergrund) | im Installer (Frage „bei jeder Anmeldung starten“) | z. B. systemd-User-Service |
| Beenden | `schtasks /End /TN "Obsidian-Agent Weboberflaeche"` | `mac/deinstallieren_mac.command` bzw. Autostart abwählen | `Ctrl+C` |
| Deinstallieren | `deinstallieren.bat` | `mac/deinstallieren_mac.command` | manuell |
| Protokolle | `%LOCALAPPDATA%\obsidian-agent\gui.log` | `~/Library/Logs/obsidian-agent/` | Terminal |
| Adresse | `http://127.0.0.1:5055` | dito | dito |
