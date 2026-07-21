# STM32-Projekte mit CubeMX, CMake, GCC und VS Code — Anleitung für Linux (Fedora)

Diese Anleitung beschreibt, wie jedes neue STM32-Projekt aufgesetzt wird,
damit es dem etablierten Stil entspricht. Sie ist so geschrieben, dass sie ohne Vorwissen
durchgearbeitet werden kann.

---

## Glossar: Unsere Werkzeuge und warum

| Werkzeug | Was es ist | Warum wir es benutzen |
|---|---|---|
| **STM32CubeMX** | Grafischer Hardware-Konfigurator und Code-Generator von ST | Erzeugt aus Klicks (Pins, Takt, Peripherie) das komplette Projektgerüst — fehlerfrei und in Minuten statt Stunden Handarbeit. Kompiliert selbst NIE. |
| **STM32CubeCLT** | ST-Werkzeugkasten: Compiler (arm-none-eabi-gcc), Flasher (STM32_Programmer_CLI), Debug-Server (ST-LINK_gdbserver) | Eine feste, versionierte Installation statt wandernder Systempakete → in fünf Jahren exakt reproduzierbare Builds. Kostenlos. |
| **arm-none-eabi-gcc** | Der eigentliche C-Compiler für ARM-Controller (Teil von CubeCLT) | Industriestandard, kostenlos, quelloffen — keine IAR-/Keil-Lizenzen nötig. |
| **CMake + Ninja** | Build-System: CMake beschreibt WAS gebaut wird, Ninja führt es schnell aus | Der Build steckt in versionierten Textdateien statt in IDE-Einstellungen → identisches Ergebnis auf jedem Rechner, egal ob Terminal oder VS Code. |
| **VS Code** | Editor mit Erweiterungen für STM32, Debugging und CMake | Kostenlos, plattformübergreifend; ersetzt CubeIDE/IAR als tägliche Arbeitsumgebung. |
| **Git** | Versionsverwaltung: protokolliert jede Änderung am Projekt lokal | Jeder Stand ist wiederherstellbar, jede Änderung nachvollziehbar ("wer/wann/warum"). |
| **GitHub** | Online-Ablage für Git-Projekte | Backup, Austausch im Team, zentrale "Wahrheit" — was nicht auf GitHub liegt, existiert offiziell nicht. |

Das Zusammenspiel in einem Satz: **CubeMX generiert das Projekt, CMake+GCC
bauen es, VS Code ist der Arbeitsplatz, Git/GitHub sind das Gedächtnis.**
CubeMX wird nur beim Anlegen und bei Hardware-Änderungen gebraucht — die
tägliche Arbeit (Code, Build, Flash, Debug, Commit) läuft ohne.

---

## Teil 1: Rechner einrichten (einmalig pro Rechner)

### 1.1 STM32CubeCLT installieren (Compiler, Flasher, Debugger)

Von st.com → Suche "STM32CubeCLT" → Get Software → Generic Linux Installer
(ST-Konto oder E-Mail nötig). Dann:

```bash
cd ~/Downloads
unzip stm32cubeclt_*.sh.zip
chmod +x stm32cubeclt_*.sh          # ls zeigt den exakten Dateinamen
sudo sh ./stm32cubeclt_*.sh
```

Lizenz mit `y` bestätigen, Installationsziel `/opt/st/stm32cubeclt_<version>/`
übernehmen, udev-Regeln für den ST-Link mit Ja beantworten (sonst darf nur
root auf den Debugger zugreifen). Danach **einmal ab- und wieder anmelden** —
erst dann sind die Werkzeuge im PATH. Kontrolle, alle drei müssen antworten:

```bash
arm-none-eabi-gcc --version
STM32_Programmer_CLI --version
ST-LINK_gdbserver --version
```

Die installierte Version (steht im Pfad unter /opt/st/) gehört in das
README jedes Projekts — das ist die Reproduzierbarkeits-Dokumentation.

### 1.2 CubeMX installieren (Hardware-Konfigurator)

Von st.com → "STM32CubeMX" → Linux-Version laden, entpacken,
`./SetupSTM32CubeMX-<version>` ausführen (grafischer Installer).
**Mindestens Version 6.11** — ältere können kein CMake generieren.

### 1.3 Basiswerkzeuge und VS Code

```bash
sudo dnf install cmake ninja-build git gh
```

VS Code als **RPM aus dem Microsoft-Repo** installieren — **NICHT als
Flatpak** aus GNOME Software! Die Flatpak-Sandbox blockiert den Zugriff
auf die Toolchain unter /opt/st und auf den ST-Link (haben wir schmerzhaft
gelernt):

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo tee /etc/yum.repos.d/vscode.repo << 'EOF'
[code]
name=Visual Studio Code
baseurl=https://packages.microsoft.com/yumrepos/vscode
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc
EOF
sudo dnf install code
```

Erweiterungen (in VS Code: Strg+Umschalt+X, dann suchen und Install):
**STM32Cube for Visual Studio Code** (Herausgeber STMicroelectronics),
**Cortex-Debug**, **C/C++**, **CMake Tools**. Bei Projekten mit der
mitgelieferten `.vscode/extensions.json` schlägt VS Code sie beim Öffnen
automatisch vor — dann reicht ein Klick auf "Install".

### 1.4 Git und GitHub einmalig verbinden

```bash
git config --global user.name  "Vorname Nachname"
git config --global user.email "mail@zur-github-adresse.de"
gh auth login        # GitHub.com -> HTTPS -> "Login with a web browser"
```

Die E-Mail sollte die des GitHub-Kontos sein, damit Commits dem Konto
zugeordnet werden. `gh auth login` erledigt die Anmeldung einmalig über
den Browser — danach funktioniert `git push` dauerhaft ohne
Passwortabfrage. (Das normale GitHub-Passwort funktioniert beim Push
übrigens grundsätzlich NICHT — GitHub verlangt Token, und genau die
verwaltet `gh` für uns.)

---

## Teil 2: Neues Projekt anlegen (pro Projekt)

### 2.1 In CubeMX erzeugen

1. CubeMX starten → *File → New Project* → im MCU-Selector den Controller
   suchen (z. B. STM32G071CBTx) → Doppelklick / *Start Project*.
2. Reiter **Pinout & Configuration**: Pins per Klick belegen, Peripherie
   und Middleware (z. B. FreeRTOS) aktivieren.
   Reiter **Clock Configuration**: Takt einstellen.
3. Reiter **Project Manager** — die drei entscheidenden Felder:
   - **Project Name:** kurz, OHNE Leerzeichen und Umlaute
     (z. B. `Projekt123`) — wird auch der Name des Build-Ziels.
   - **Project Location:** z. B. `~/projekte/`.
   - **Toolchain / IDE: `CMake`** ← DAS ist der Kern des ganzen Stils.
     Steht hier CubeIDE oder EWARM, entsteht das falsche Projektformat.
4. **GENERATE CODE** klicken.

Ergebnis ist die bekannte Struktur: `.ioc` (die Hardware-Konfiguration
als Datei), `CMakeLists.txt`, `CMakePresets.json`,
`cmake/gcc-arm-none-eabi.cmake`, `cmake/stm32cubemx/CMakeLists.txt`,
`Core/` (dein Arbeitsbereich), `Drivers/` (ST-Bibliotheken),
Startup-Datei und Linkerscript.

### 2.2 Erster Build (Terminal oder VS Code, identisches Ergebnis)

```bash
cd ~/projekte/Projekt123
cmake --preset Debug          # einmalig: Build-Ordner konfigurieren
cmake --build --preset Debug  # bauen
```

Oder in VS Code: `code .`, Erweiterungs-Empfehlung annehmen, wenn nach
einem "Kit"/Compiler gefragt wird **"[Unspecified]"** wählen (das Preset
regelt den Compiler selbst!), Preset "Debug" wählen, F7.

Erfolgskriterium ist immer die Speichertabelle am Ende:

```
Memory region   Used Size  Region Size  %age Used
         RAM:      xxxx B        xx KB     x.xx%
       FLASH:      xxxx B       xxx KB     x.xx%
```

Diese Zahlen ins README notieren — sie sind der Referenzwert, an dem
jeder spätere Build und jeder andere Rechner gemessen wird. Das fertige
Programm liegt unter `build/Debug/<Name>.elf` (+ `.hex`/`.bin`).

### 2.3 Projekt-Hygiene ergänzen (unser Standard)

Vier Dateien gehören zusätzlich in jedes Projekt. Entweder aus einem
bestehenden Projekt kopieren — oder mit genau den folgenden Inhalten neu
anlegen. Als Beispiel-Projektname dient überall `Projekt123`.


#### `.gitignore` (im Projekt-Hauptordner)

Sagt Git, welche Dateien NICHT versioniert werden. Grundregel: Alles, was
aus den Quellen jederzeit neu erzeugt werden kann (Build-Ausgaben) oder
reine Werkzeug-Caches sind, gehört nicht ins Repository — sonst bläht es
auf und jeder Build erzeugt Schein-Änderungen.

```gitignore
# Build-Ausgaben - entstehen aus den Quellen jederzeit neu
build/
*.elf
*.hex
*.bin
*.map

# Editor- und Werkzeug-Caches
.cache/
.vscode/ipch

# CubeMX-Arbeitsdateien und Sicherungskopien
.mxproject
*.bak
```

#### `.vscode/extensions.json`

Die Erweiterungs-Empfehlungen des Projekts. Öffnet jemand das Projekt in
VS Code, erscheint automatisch "Do you want to install the recommended
extensions?" — ein Klick, und der Rechner ist arbeitsfähig. Die IDs
folgen dem Muster `herausgeber.erweiterungsname`.

```json
{
    "recommendations": [
        "stmicroelectronics.stm32-vscode-extension",
        "marus25.cortex-debug",
        "ms-vscode.cpptools",
        "ms-vscode.cmake-tools"
    ]
}
```

#### `.vscode/launch.json`

Die Debug-Konfiguration — sie macht aus F5 den kompletten Ablauf
"flashen, GDB-Server starten, an main() anhalten".

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug (ST-Link)",
            "type": "cortex-debug",
            "request": "launch",
            "servertype": "stlink",
            "cwd": "${workspaceFolder}",
            "executable": "${workspaceFolder}/build/Debug/Projekt123.elf",
            "device": "STM32G071CB",
            "svdFile": "",
            "runToEntryPoint": "main"
        }
    ]
}
```

Die Felder im Einzelnen: `type: cortex-debug` wählt die
Debug-Erweiterung, `servertype: stlink` den GDB-Server aus CubeCLT.
**Anpassen pro Projekt:** `executable` (Projektname der .elf) und
`device` (exakter Controller — bestimmt Flash-Layout und
Registeransicht). Bei FreeRTOS-Projekten zusätzlich die Zeile
`"rtos": "FreeRTOS",` einfügen — dann zeigt der Debugger die Task-Liste
mit Zuständen und Stack-Auslastung an. `${workspaceFolder}` ist eine
VS-Code-Variable für den Projektordner und bleibt wörtlich so stehen.

#### `README.md`

Die Visitenkarte des Projekts — GitHub zeigt sie als Startseite an.
Regel: Ein neuer Kollege muss allein mit dem README bauen, flashen und
den Stand einordnen können. Bewährtes Gerüst zum Ausfüllen:

````markdown
# Projekt123 — <Einzeiler: was ist das Gerät/die Firmware?>

- MCU: <z. B. STM32G071CBT6 (Cortex-M0+, 128 KB Flash, 36 KB RAM)>
- Takt: <z. B. 16 MHz HSI>
- RTOS: <keins / FreeRTOS (CMSIS-RTOS v2), Tasks: ...>
- Besonderheiten: <EEPROM-Emulation, externer Watchdog, ...>

## Voraussetzungen

STM32CubeCLT <VERSION EINTRAGEN — Pflichtangabe!>, STM32CubeMX,
VS Code mit den Erweiterungen aus .vscode/extensions.json.

## Bauen

```bash
cmake --preset Debug
cmake --build --preset Debug
```

Referenzwerte (mit obiger Toolchain):
- Debug:   Flash <...> B, RAM <...> B
- Release: Flash <...> B

## Flashen und Debuggen

```bash
STM32_Programmer_CLI -c port=SWD -w build/Debug/Projekt123.hex -v -rst
```

Debuggen: F5 in VS Code ("Debug (ST-Link)").

## Workflow

Hardware-Konfiguration ausschließlich über Projekt123.ioc in CubeMX
ändern (Generate Code; eigener Code nur in USER-CODE-Blöcken).
Logik-Änderungen direkt in Core/Src/.

## Versionen

- v1.0 (<Datum>): <was ist drin> — gebaut mit CubeCLT <Version>

## Offene Punkte

- <bekannte Baustellen, ungeklärte Fragen, geplante Validierungen>
````

Warum die CubeCLT-Version Pflicht ist: Sie macht jeden Build in Jahren
exakt reproduzierbar. Warum Referenz-Buildgrößen: Ein Build auf einem
anderen Rechner, der (bis auf wenige Bytes bei anderer GCC-Generation)
dieselben Zahlen liefert, ist der schnellste Beweis, dass die Umgebung
korrekt eingerichtet ist.

### 2.4 Git und GitHub — was wir tun und warum

Kurz die Begriffe, dann die Befehle:

**Git** führt im versteckten Unterordner `.git/` ein Protokoll über jeden
festgeschriebenen Stand des Projekts. Ein festgeschriebener Stand heißt
**Commit** — wie ein beschriftetes Foto des kompletten Projektordners zu
einem Zeitpunkt. **GitHub** ist der Server, auf den diese Commits
hochgeladen werden (**push**). `main` ist der Name der Hauptlinie
(**Branch**) der Historie. Ein **Tag** ist ein Etikett an einem
bestimmten Commit ("das hier ist v1.0").

Erst auf github.com (eingeloggt, "New repository") ein LEERES Repository
anlegen — ohne Häkchen bei "Add README", sonst kollidiert es gleich mit
unserem lokalen Stand. Dann im Projektordner:

```bash
git init                       # macht den Ordner zu einem Git-Projekt (legt .git/ an)
git add .                      # legt alle Dateien in den "Warenkorb" für den nächsten Commit
                               #   (alles, was .gitignore nicht ausschließt)
git commit -m "Projektgeruest: CubeMX-generiert (CMake), STM32CubeCLT <version>"
                               # schreibt den Warenkorb als Stand Nr. 1 fest -
                               #   die Meldung (-m) beschreibt, WAS und WARUM
git branch -M main             # nennt die Hauptlinie "main" (GitHub-Standard)
git remote add origin https://github.com/<benutzer>/<repo>.git
                               # verknüpft das lokale Projekt mit dem GitHub-Repo
                               #   ("origin" ist nur der übliche Spitzname dafür)
git push -u origin main        # lädt alle Commits zu GitHub hoch; -u merkt sich
                               #   die Verbindung, ab jetzt reicht "git push"
```

**Warum der frisch generierte Stand der ERSTE Commit sein soll, noch vor
eigenem Code:** Dann trennt die Historie für immer sauber zwischen "das
hat der Generator erzeugt" und "das haben wir geändert" — und nach jedem
späteren "Generate Code" zeigt `git diff` exakt, was CubeMX angefasst hat.

Die drei Kontrollbefehle für den Alltag (kosten nichts, ändern nichts):

```bash
git status          # Welche Dateien sind geändert/neu? Wo stehe ich?
git diff            # Was GENAU wurde geändert (Zeile für Zeile)?
git log --oneline   # Die Historie: ein Commit pro Zeile
```

Angewöhnen: **vor jedem Commit einmal `git status` und `git diff`** —
committen, was man gesehen und verstanden hat, nicht blind.

Und für Releases:

```bash
git tag -a v1.0 -m "v1.0 - gebaut mit STM32CubeCLT <version>"
git push origin v1.0
```

Warum taggen: Der Tag friert den exakten Quellstand einer Auslieferung
ein. "Welcher Code läuft auf den Geräten von Kunde X?" ist damit in
Sekunden beantwortbar: `git checkout v1.0` stellt genau diesen Stand
wieder her.

---

## Teil 3: Der tägliche Arbeitszyklus

```
Code ändern -> speichern -> bauen -> flashen/testen -> git status/diff -> committen -> pushen
```

- **Eigener Code kommt AUSSCHLIESSLICH in die
  `/* USER CODE BEGIN ... */ ... /* USER CODE END ... */`-Blöcke** der
  generierten Dateien — oder in komplett eigene .c/.h-Dateien, die in der
  Root-`CMakeLists.txt` unter `target_sources(...)` eingetragen werden.
  Nur dann überlebt er das nächste "Generate Code". Code außerhalb der
  Blöcke wird beim Generieren KOMMENTARLOS GELÖSCHT.
- Bauen: `cmake --build --preset Debug` oder F7 in VS Code.
  Garantiert frischer Komplettbau: `cmake --build --preset Debug --clean-first`
- Flashen: `STM32_Programmer_CLI -c port=SWD -w build/Debug/<name>.hex -v -rst`
  (`-v` = nach dem Schreiben verifizieren, `-rst` = danach neu starten)
- Debuggen: F5 in VS Code — flasht, startet den GDB-Server, hält an
  `main()`. F10 = Zeile für Zeile, F5 = weiterlaufen.
- Committen: `git add -u` (alle geänderten, bereits versionierten
  Dateien) plus `git add <datei>` für neue Dateien, dann
  `git commit -m "..."` und `git push`.

**Hardware-Änderung nötig?** `.ioc` in CubeMX öffnen → ändern →
Generate Code → **`git diff` ansehen** (was hat der Generator getan?) →
bauen → als eigenen Commit festschreiben ("CubeMX: UART3 ergänzt").
Generator-Änderungen und Handänderungen nie im selben Commit mischen.

**Serienstand:** Release-Build aus dem Terminal
(`cmake --preset Release && cmake --build --preset Release`), auf
Hardware validieren, Version taggen, README (Größen, Toolchain) nachführen.

---

## Teil 4: Checkliste für jedes neue Projekt

- [ ] CubeMX: Toolchain = CMake, Name ohne Leerzeichen/Umlaute
- [ ] Erster Build läuft (Terminal), Speichertabelle im README notiert
- [ ] .gitignore, .vscode/, README.md ergänzt (inkl. CubeCLT-Version)
- [ ] Generierter Stand = erster Commit, Repo auf GitHub, Push erfolgreich
- [ ] Eigener Code nur in USER-CODE-Blöcken
- [ ] Keine leeren Zählschleifen für Timing — GCC optimiert sie ersatzlos
      weg! Immer HAL_Delay oder einen Hardware-Timer verwenden.
- [ ] Timing-Konstanten als benannte #define, nicht als nackte Zahlen
- [ ] Bei Flash-EEPROM-Emulation: Seiten im Linkerscript reservieren
      (FLASH-LENGTH kürzen)
- [ ] Release-Build getestet, auf Hardware validiert, Version getaggt

---

## Teil 5: Troubleshooting — typische Fehler und ihre Lösung

**`bash: code: Befehl nicht gefunden`**
VS Code ist als Flatpak installiert (prüfen: `flatpak list | grep -i code`).
Flatpak entfernen und die RPM-Version aus Schritt 1.3 installieren — die
Flatpak-Sandbox verträgt sich nicht mit Toolchain und ST-Link.

**`arm-none-eabi-gcc: Befehl nicht gefunden` direkt nach der CubeCLT-Installation**
Der PATH greift erst nach Ab- und Wiederanmelden. Zur Not prüfen, ob die
Dateien da sind: `ls /opt/st/`.

**`sudo`-Meldung "Das hat nicht funktioniert, bitte nochmal probieren"**
Das ist nur eine falsche Passworteingabe bei sudo — kein Fehler des
eigentlichen Befehls. Viele Befehle (z. B. `rpm --import`) geben bei
Erfolg schlicht GAR NICHTS aus: keine Ausgabe = gut.

**`unzip: cannot find or open ...`**
Die Datei liegt nicht dort, wo der Befehl sie sucht. Fundort ermitteln:
`ls ~/Downloads/ | grep -i <namensteil>` oder
`find ~ -name "*<namensteil>*" 2>/dev/null`, dann den Pfad im Befehl
anpassen.

**`ninja: no work to do` — obwohl Dateien geändert/ersetzt wurden**
Ninja entscheidet nach Zeitstempel; entpackte Dateien tragen oft ein
altes Datum. Lösung: `touch Core/Src/*.c Core/Inc/*.h` und neu bauen,
oder gleich `cmake --build --preset Debug --clean-first`.
Merksatz: Nach jedem Entpacken über ein Projekt einmal sauber neu bauen.

**Build nutzt den falschen Compiler (x86-Fehler, riesige .elf, Linkerfehler zu Systembibliotheken)**
In VS Code wurde ein "Kit" gewählt (z. B. /usr/bin/gcc). Beheben:
Strg+Umschalt+P → "CMake: Select a Kit" → **[Unspecified]** →
"CMake: Delete Cache and Reconfigure". Kontrolle: Im Output muss
`Found assembler: /opt/st/stm32cubeclt_.../arm-none-eabi-gcc` stehen.

**ST-Erweiterung: "Checking toolchain step failed" / Bundle-Fehler**
Bekannter Schluckauf der Erweiterung, betrifft das Bauen NICHT — Terminal
und CMake-Tools funktionieren unabhängig davon. Fenster neu laden
("Developer: Reload Window"); zur Not Erweiterung de- und neu installieren.
Löst es sich oft nach dem ersten erfolgreichen Terminal-Build von selbst.

**`STM32_Programmer_CLI -l` findet keinen ST-Link**
USB ab- und wieder anstecken (udev-Regeln greifen erst beim
Neu-Enumerieren). Weitere Kandidaten: reines Ladekabel statt Datenkabel,
Zielplatine ohne Versorgung (der ST-Link speist das Ziel i. d. R. nicht),
udev-Regeln bei der Installation abgelehnt (CubeCLT-Installer erneut
laufen lassen).

**`git commit` → "Please tell me who you are"**
Einmalig `git config --global user.name/user.email` setzen (Schritt 1.4).

**`git push` → Authentifizierungsfehler / Passwort wird abgelehnt**
GitHub akzeptiert keine Kontopasswörter. `gh auth login` ausführen
(Schritt 1.4), danach klappt der Push dauerhaft.

**`git status`: "nichts zu committen" — obwohl doch etwas geändert wurde**
Fast immer: falscher Ordner, oder die Änderung ist nie angekommen
(z. B. unzip in ein anderes Verzeichnis). `pwd` prüfen, dann
`git diff` — wenn der leer ist, wurde real nichts geändert.

**Eigener Code ist nach "Generate Code" verschwunden**
Er stand außerhalb der USER-CODE-Blöcke. Wiederherstellen mit Git:
`git diff` zeigt den Verlust, `git checkout -- <datei>` holt den letzten
committeten Stand zurück (deshalb: VOR jedem Generate committen!).
Danach den Code in einen USER-Block umziehen.

**Linkerfehler `region 'FLASH' overflowed by N bytes`**
Das Programm ist größer als der (ggf. für EEPROM-Emulation gekürzte)
Flash. Erst mit Release-Build (-Os) prüfen; dann Code verkleinern oder
Reservierung überdenken.

**Linker-Warnungen `_close/_read/_write is not implemented`**
Harmlos: Stubs der C-Bibliothek für Dateifunktionen, die es auf dem
Controller nicht gibt. Nur relevant, falls printf/scanf genutzt werden
sollen — dann eine echte Ausgabe (z. B. UART) hinterlegen.

**`HAL_Delay()` hängt oder Zeiten stimmen nicht**
Zeitbasis prüfen: Läuft der Tick-Timer/SysTick? Bei exotischen Takten
(sehr niedrige Frequenzen) die generierte Timebase-Berechnung
kontrollieren — Standardformeln setzen ≥ 1 MHz voraus
(Datei stm32g0xx_hal_timebase_tim.c).
