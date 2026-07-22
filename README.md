# STM32-Projekte mit CubeMX, CMake, GCC und VS Code — Anleitung für Linux (Fedora)

Diese Anleitung beschreibt, wie jedes neue STM32-Projekt aufgesetzt wird,
damit es dem etablierten Stil entspricht. Sie ist so geschrieben, dass sie ohne Vorwissen
durchgearbeitet werden kann.

---

## Inhalt

- [Glossar: Unsere Werkzeuge und warum](#glossar-unsere-werkzeuge-und-warum)
- [VS Code in Kürze: Oberfläche und Tastenkürzel](#vs-code-in-kürze-oberfläche-und-tastenkürzel)
  - [Ausgabe, Debug-Konsole, Terminal — wer zeigt was?](#ausgabe-debug-konsole-terminal--wer-zeigt-was)
- [Teil 1: Rechner einrichten (einmalig pro Rechner)](#teil-1-rechner-einrichten-einmalig-pro-rechner)
  - [1.1 STM32CubeCLT installieren (Compiler, Flasher, Debugger)](#11-stm32cubeclt-installieren-compiler-flasher-debugger)
  - [1.2 CubeMX installieren (Hardware-Konfigurator)](#12-cubemx-installieren-hardware-konfigurator)
  - [1.3 Basiswerkzeuge und VS Code](#13-basiswerkzeuge-und-vs-code)
  - [1.4 Git und GitHub einmalig verbinden](#14-git-und-github-einmalig-verbinden)
- [Teil 2: Neues Projekt anlegen (pro Projekt)](#teil-2-neues-projekt-anlegen-pro-projekt)
  - [2.1 In CubeMX erzeugen](#21-in-cubemx-erzeugen)
  - [2.2 Erster Build (Terminal oder VS Code, identisches Ergebnis)](#22-erster-build-terminal-oder-vs-code-identisches-ergebnis)
  - [2.3 Ausgabedateien: .hex und .bin erzeugen (und optional .dfu)](#23-ausgabedateien-hex-und-bin-erzeugen-und-optional-dfu)
  - [2.4 Projekt-Hygiene ergänzen (unser Standard)](#24-projekt-hygiene-ergänzen-unser-standard)
  - [2.5 Git und GitHub — was wir tun und warum](#25-git-und-github--was-wir-tun-und-warum)
- [Teil 3: Der tägliche Arbeitszyklus](#teil-3-der-tägliche-arbeitszyklus)
- [Exkurs: FreeRTOS — über den CubeMX-Haken oder von Hand einbinden?](#exkurs-freertos--über-den-cubemx-haken-oder-von-hand-einbinden)
- [Exkurs: Peripherie-Init als eigene .c/.h-Dateien pro Peripherie](#exkurs-peripherie-init-als-eigene-ch-dateien-pro-peripherie)
- [Teil 4: Checkliste für jedes neue Projekt](#teil-4-checkliste-für-jedes-neue-projekt)
- [Teil 5: Troubleshooting — typische Fehler und ihre Lösung](#teil-5-troubleshooting--typische-fehler-und-ihre-lösung)

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

## VS Code in Kürze: Oberfläche und Tastenkürzel

Für alle, die neu in VS Code sind — vier Bereiche reichen zur
Orientierung: Die **Seitenleiste** links (Symbole für Dateien, Suche,
Git, Debug, Erweiterungen), der **Editor** in der Mitte, das **Panel**
unten (Terminal, Ausgabe, Probleme) und die **Statusleiste** ganz unten
(dort sitzen CMake-Preset und Build-Knopf). Der wichtigste Griff
überhaupt ist die **Befehlspalette**: Strg+Umschalt+P öffnet ein
Suchfeld, über das JEDER Befehl erreichbar ist — wann immer diese
Anleitung "Developer: Reload Window" oder "CMake: ..." sagt, ist genau
das gemeint: Palette öffnen, Befehl eintippen, Enter.

| Kürzel | Wirkung |
|---|---|
| **Strg+Umschalt+P** | Befehlspalette — der Generalschlüssel zu allem |
| Strg+P | Datei im Projekt schnell öffnen (Namen tippen) |
| Strg+S | Speichern (vor jedem Build! ungespeichert = alter Stand) |
| Strg+Umschalt+E | Seitenleiste: Datei-Explorer |
| Strg+Umschalt+F | Seitenleiste: Suchen im ganzen Projekt |
| Strg+Umschalt+X | Seitenleiste: Erweiterungen |
| **Strg+Umschalt+D** | Seitenleiste: Ausführen & Debuggen (Dropdown + F5) |
| Strg+Ö (bzw. Strg+` auf US-Tastatur) | Panel: integriertes Terminal ein/aus |
| Strg+Umschalt+U | Panel: Ausgabe (Build-/CMake-Meldungen) |
| Strg+, | Einstellungen |
| **F7** | Bauen (CMake Tools; wie Build-Knopf in der Statusleiste) |
| **F5** | Debuggen: startet die im Dropdown gewählte Konfiguration |
| F10 / F11 | Im Debugger: Zeile überspringen / in Funktion hinein |
| Umschalt+F5 | Debug-Sitzung beenden (bei "Attach": nur trennen) |

Die zwei Tasten, die man verwechseln kann: **F7 baut, F5 flasht und
debuggt** — und F5 nimmt immer die Konfiguration, die oben in der
Debug-Ansicht (Strg+Umschalt+D) ausgewählt ist. Die Play-/Käfer-Symbole
unten in der Statusleiste dagegen NICHT verwenden (siehe
Troubleshooting: sie versuchen, die Firmware auf dem PC zu starten).

---

### Ausgabe, Debug-Konsole, Terminal — wer zeigt was?

Im Panel unten liegen drei Reiter, die Einsteiger ständig verwechseln,
weil alle drei "Text ausgeben". Die Unterscheidung ist einfach, wenn
man fragt: WER redet hier?

- **Ausgabe (Output, Strg+Umschalt+U)** — hier reden die
  **Erweiterungen**. Nur lesen, nichts eintippen. Über das Dropdown
  rechts oben wählt man den Kanal (CMake/Build, die ST-Erweiterung, …).
  *Beispiel:* Nach F7 erscheinen hier die Compiler-Aufrufe und am Ende
  die Speichertabelle `FLASH: … B` — das ist der Kanal "CMake/Build".
- **Debug-Konsole (Debug Console)** — hier redet der **Debugger**
  während einer F5-Sitzung, und hier darf man auch MIT ihm reden: Unten
  in der Eingabezeile lassen sich Ausdrücke auswerten.
  *Beispiel:* Während das Programm an einem Breakpoint steht,
  `ir_sensor_flankenwechsel` eintippen und Enter — die Konsole zeigt
  den aktuellen Wert der Variablen. (Auch die Cortex-Debug-Startmeldungen
  und der `-cp`-Pfad aus dem Troubleshooting stehen hier.)
- **Terminal (Strg+Ö)** — eine **echte Shell** in deinem Projektordner,
  identisch zum externen Terminal-Fenster. Hier tippst du selbst
  Befehle. *Beispiel:* `cmake --build --preset Release` oder
  `git status`. Zusätzlich legt Cortex-Debug hier während einer
  Debug-Sitzung den Reiter **"gdb-server"** an — dort steht die Ausgabe
  des ST-LINK_gdbserver, also der Ort, an dem man nachliest, WARUM ein
  "GDB Server Quit Unexpectedly" passierte (z. B. "kein ST-Link
  gefunden").

Merkhilfe: **Ausgabe = Protokoll der Werkzeuge, Debug-Konsole = Dialog
mit dem Debugger, Terminal = Dialog mit dem System.**

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
automatisch vor — dann reicht ein Klick auf "Install". Kommt keine
Abfrage: in der Erweiterungs-Ansicht `@recommended` ins Suchfeld
eingeben und dort gesammelt installieren.

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
4. Reiter **Code Generator** (ebenfalls im Project Manager): Haken bei
   **"Generate peripheral initialization as a pair of '.c/.h' files
   per peripheral"** setzen — jede Peripherie bekommt ihre eigene
   Datei statt alles in main.c (Begründung siehe Exkurs weiter unten).
5. **GENERATE CODE** klicken.

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

#### Warum zwei Presets? Debug und Release sind zwei verschiedene Programme

Die Presets unterscheiden sich nicht nur im Ablageordner, sondern in den
Compiler-Schaltern (hinterlegt in `cmake/gcc-arm-none-eabi.cmake`):
**Debug** baut mit `-O0 -g3` — keine Optimierung, volle
Debug-Informationen. Jede C-Zeile entspricht 1:1 dem Maschinencode,
deshalb funktionieren Einzelschritt, Breakpoints und Variablenansicht
sauber. **Release** baut mit `-Os` — der Compiler optimiert auf Größe
(deutlich kleineres, schnelleres Binary), dafür ist es kaum noch
debugbar, und unsauberer Code (fehlendes `volatile`, Timing per
Leerschleife) verhält sich plötzlich anders, weil der Optimierer ihn
umbaut oder entfernt. Daraus folgt die Regel: **Entwickelt und gesucht
wird im Debug — validiert und ausgeliefert wird IMMER der Release.**
Was nur im Debug getestet wurde, ist ungetestet, denn auf dem
Seriengerät läuft ein anderes Maschinenprogramm.

Das Debug-Preset definiert außerdem das Makro `DEBUG` (siehe
`$<$<CONFIG:Debug>:DEBUG>` in `cmake/stm32cubemx/CMakeLists.txt`). Damit
lassen sich Testausgaben einbauen, die im Serienstand garantiert fehlen
— Beispiel (setzt einen in CubeMX aktivierten UART voraus):

```c
/* USER CODE BEGIN PD */
#ifdef DEBUG
  // Nur im Debug-Build vorhanden - im Release ersatzlos leer
  #define DBG_MELDUNG(txt)  HAL_UART_Transmit(&huart2, (uint8_t *)(txt), sizeof(txt) - 1, 100)
#else
  #define DBG_MELDUNG(txt)
#endif
/* USER CODE END PD */
```

Aufruf an beliebiger Stelle im Code:

```c
DBG_MELDUNG("Alarm ausgeloest\r\n");
```

Im Debug-Build erscheint die Zeile am UART (mitlesen z. B. mit `screen /dev/ttyUSB0 115200` (Paket `screen`,
Beenden: Strg+A, dann K) — 115200 8N1); im Release-Build
erzeugt exakt dieselbe Quellzeile **null Bytes Code** — keine Laufzeit,
kein Flash-Verbrauch, kein versehentliches Geschwätz im Seriengerät.

### 2.3 Ausgabedateien: .hex und .bin erzeugen (und optional .dfu)

Frisch von CubeMX generierte CMake-Projekte erzeugen beim Bauen **nur die
`.elf`**. Zum Flashen mit dem CubeProgrammer ist aber die `.hex` der
Standard. Deshalb gehört in JEDES neue Projekt einmalig dieser Block —
**ans Ende der `CMakeLists.txt` im Projekt-Hauptordner** (NICHT in die
unter `cmake/stm32cubemx/`, die überschreibt CubeMX bei jedem Generate;
die Root-Datei dagegen erzeugt CubeMX nur beim allerersten Mal und fasst
sie danach nie wieder an — Ergänzungen dort sind regenerierungssicher):

```cmake
# Nach dem Linken: .hex und .bin fuer den CubeProgrammer erzeugen
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex   $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${CMAKE_PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${CMAKE_PROJECT_NAME}.bin
)
```

Was der Block tut: `objcopy` (Teil der GCC-Toolchain) wandelt nach jedem
Linken die .elf zusätzlich in die beiden Formate um. Nach dem Eintragen
einmal frisch bauen (`cmake --build --preset Debug --clean-first`, weil
der Schritt nur beim Linken läuft) — ab dann liegen bei jedem Build
`<Name>.elf`, `.hex`, `.bin` und `.map` unter `build/Debug/` bzw.
`build/Release/`.

**Optional: .dfu für Firmware-Updates im Feld über USB.**
Voraussetzung: Der Controller hat USB-Hardware (z. B. STM32G0B1, F4, H5
— ein STM32G071 hat KEIN USB; dort läuft der ROM-Bootloader stattdessen
über UART). Zwei Wege:

1. **Ohne .dfu-Datei** — der CubeProgrammer flasht die normale .hex
   direkt über den DFU-Bootloader (Gerät vorher per BOOT0 in den
   Bootloader versetzen):

   `STM32_Programmer_CLI -c port=USB1 -w build/Release/Projekt123.hex`

2. **Echte .dfu-Datei** — nötig, wenn Endanwender mit einem
   DfuSe-Update-Tool flashen sollen. Erzeugt wird sie aus der .hex mit
   dem Skript `dfuse-pack.py` aus dem dfu-util-Projekt
   (`0483:df11` ist die USB-Kennung des ST-Bootloaders):

```bash
sudo dnf install dfu-util                  # Flash-Werkzeug; das Skript dfuse-pack.py
                                           # liegt im Quell-Repo des dfu-util-Projekts
python3 dfuse-pack.py -i build/Release/Projekt123.hex -D 0x0483:0xdf11 Projekt123.dfu
```

Wer die .dfu bei jedem Build automatisch haben will, hängt den Aufruf
als weitere `COMMAND`-Zeile in den objcopy-Block oben.

### 2.4 Projekt-Hygiene ergänzen (unser Standard)

Vier Dateien gehören zusätzlich in jedes Projekt. Entweder aus einem
bestehenden Projekt kopieren — oder mit genau den folgenden Inhalten neu
anlegen. Als Beispiel-Projektname dient überall `Projekt123`.


#### `.gitignore` (im Projekt-Hauptordner)

Sagt Git, welche Dateien NICHT versioniert werden. Grundregel: Alles, was
aus den Quellen jederzeit neu erzeugt werden kann (Build-Ausgaben) oder
reine Werkzeug-/Betriebssystem-Artefakte sind, gehört nicht ins
Repository. `.gitignore` erlaubt Kommentare mit `#`:

```gitignore
# ==== Build-Ausgaben - entstehen aus den Quellen jederzeit neu ====
build/
*.elf
*.hex
*.bin
*.map
*.dfu

# ==== Editor- und Werkzeug-Caches ====
.cache/
.vscode/ipch
compile_commands.json

# ==== CubeMX-Arbeitsdateien und Sicherungskopien ====
.mxproject
*.bak

# ==== Betriebssystem-Artefakte (tauchen frueher oder spaeter auf) ====
.DS_Store               # macOS Finder
Thumbs.db               # Windows Explorer
desktop.ini             # Windows Explorer

# ==== Falls Kollegen andere Editoren/IDEs verwenden ====
*.swp                   # vim-Arbeitsdateien
*~                      # Backup-Dateien diverser Editoren
.idea/                  # JetBrains-IDEs (CLion etc.)
```

Gut zu wissen für später: Mit einem vorangestellten `!` lässt sich eine
einzelne Datei von einer Ausschlussregel wieder ausnehmen (z. B.
`!referenz.hex`) — für ausgelieferte Binaries ist der bessere Ort aber
ein GitHub-Release (Abschnitt 2.5), nicht das Repository.

#### `.vscode/extensions.json`

Die Erweiterungs-Empfehlungen des Projekts. Öffnet jemand das Projekt in
VS Code, erscheint automatisch "Do you want to install the recommended
extensions?" — ein Klick, und der Rechner ist arbeitsfähig. Erscheint
die Abfrage nicht (z. B. weil schon alles installiert ist oder sie
früher weggeklickt wurde): Erweiterungs-Ansicht öffnen
(Strg+Umschalt+X) und oben `@recommended` ins Suchfeld eingeben — die
Rubrik "Workspace Recommendations" zeigt die Empfehlungen des Projekts,
und das Wolken-Symbol daneben installiert alle fehlenden auf einmal.
Die IDs folgen dem Muster `herausgeber.erweiterungsname`.

Wichtig zu wissen: VS Code liest seine Konfigurationsdateien als **"JSON
mit Kommentaren"** — `//`-Kommentare sind hier (und in launch.json)
erlaubt und werden von uns bewusst zur Dokumentation genutzt. In
"normalem" JSON (z. B. Daten für ein Programm) sind Kommentare dagegen
verboten — nicht wundern, wenn es woanders Fehler gibt.

```jsonc
{
    "recommendations": [
        // Pflicht fuer diesen Workflow:
        "stmicroelectronics.stm32-vscode-extension",    // ST: findet CubeCLT, importiert Projekte
        "marus25.cortex-debug",                         // Debugging ueber ST-Link + GDB-Server
        "ms-vscode.cpptools",                           // C/C++ Grundausstattung (s. Hinweis unten)
        "ms-vscode.cmake-tools"                         // CMake: Presets, Build-Button, F7

        // Optional, aber bewaehrt - zum Aktivieren Zeile einkommentieren
        // (und Komma am Vorgaenger ergaenzen!):
        // "ms-vscode.hexeditor",                       // .bin und Speicherinhalte als Hexdump ansehen
        // "eamodio.gitlens",                           // Git-Historie im Editor: wer/wann/warum pro Zeile
        // "dan-c-underwood.arm"                        // Syntax-Farben fuer ARM-Assembler (Startup-Datei)
    ]
}
```

Hinweis zum Zusammenspiel: Die ST-Erweiterung bringt einen eigenen
IntelliSense-Motor mit (stm32-cube-clangd). Meldet VS Code einen
Konflikt mit der C/C++-Erweiterung, das Microsoft-IntelliSense
deaktivieren (Details im Troubleshooting, Teil 5) — clangd kennt über
die compile_commands.json exakt die Flags des echten Builds.

#### `.vscode/launch.json`

Die Debug-Konfiguration — sie macht aus F5 den kompletten Ablauf
"flashen, GDB-Server starten, an main() anhalten". Wir legen gleich
ZWEI Konfigurationen an; die zweite ist im Alltag Gold wert:

```jsonc
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug (ST-Link)",          // Anzeigename im Auswahlmenue der Debug-Ansicht
            "type": "cortex-debug",             // die Cortex-Debug-Erweiterung
            "request": "launch",                // launch = flashen, resetten, neu starten
            "servertype": "stlink",             // GDB-Server aus CubeCLT (ST-LINK_gdbserver)
            "cwd": "${workspaceFolder}",
            "executable": "${workspaceFolder}/build/Debug/Projekt123.elf",
            "device": "STM32G071CB",            // Controller - siehe Erlaeuterung unten
            "interface": "swd",                 // SWD ist bei STM32 der Standard (Alternative: jtag)
            "runToEntryPoint": "main",          // nach dem Start bis main() laufen, dort anhalten
            "svdFile": "",                      // Pfad zu .svd-Datei -> Peripherie-Register im Debugger
            "preLaunchTask": ""                 // z. B. Build-Task -> vor F5 automatisch bauen
            // ,"rtos": "FreeRTOS"              // bei FreeRTOS-Projekten: Task-Liste im Debugger
            // ,"showDevDebugOutput": "raw"     // bei Verbindungsproblemen: alle GDB-Meldungen anzeigen
        },
        {
            // Haengt sich an das LAUFENDE Geraet an: nichts wird geflasht,
            // nichts resettet. Ideal, um einen gerade auftretenden Fehler
            // zu untersuchen, ohne ihn durch einen Neustart zu verlieren.
            "name": "Attach (ohne Flashen)",
            "type": "cortex-debug",
            "request": "attach",                // attach statt launch = nur anhaengen
            "servertype": "stlink",
            "cwd": "${workspaceFolder}",
            "executable": "${workspaceFolder}/build/Debug/Projekt123.elf",
            "device": "STM32G071CB"
        }

        // Optionale dritte Konfiguration fuer den Sonderfall "Fehler tritt
        // NUR im Release auf" - zum Aktivieren einkommentieren und das
        // Komma an den Anfang uebernehmen (Erlaeuterung siehe unten):
        // ,{
        //     "name": "Release debuggen (optimiert!)",
        //     "type": "cortex-debug",
        //     "request": "launch",
        //     "servertype": "stlink",
        //     "cwd": "${workspaceFolder}",
        //     "executable": "${workspaceFolder}/build/Release/Projekt123.elf",
        //     "device": "STM32G071CB"
        // }
    ]
}
```

Die wichtigsten Felder und ihre Spielräume:

- **`device`** — Wie exakt muss der Name sein? Bei `servertype: stlink`
  ist das Feld tolerant: Der ST-GDB-Server erkennt den angeschlossenen
  Chip selbst, `STM32G071CB` und `STM32G071CBT6` funktionieren beide
  (Konvention: ohne Gehäuse-/Temperatur-Suffix). ABER: Wer später mal
  mit einem SEGGER J-Link arbeitet (`servertype: jlink`), muss den Namen
  EXAKT so schreiben, wie er in der SEGGER-Gerätedatenbank steht — dort
  ist das Feld streng.
- **`svdFile`** — Eine .svd-Datei beschreibt alle Peripherie-Register
  des Controllers; mit eingetragenem Pfad zeigt der Debugger unter
  "XPeripherals" jedes Register live an (z. B. Timer-Zählerstand,
  GPIO-Zustände) — extrem hilfreich bei Hardware-Fehlersuche. CubeCLT
  liefert die Dateien mit, z. B.:
  `/opt/st/stm32cubeclt_<version>/STMicroelectronics_CMSIS_SVD/STM32G071.svd`
- **`executable` bei "Attach"** — muss zu der Firmware passen, die auf
  dem Gerät gerade LÄUFT. Hintergrund: Beim Attach wird nichts geflasht.
  Der Debugger lädt aus der .elf auf der Festplatte nur die "Landkarte"
  — welche Funktion und welche Quellzeile liegt an welcher
  Flash-Adresse — und legt sie über das, was im Chip tatsächlich steckt.
  Passen Karte und Gelände nicht zusammen, zeigt der Debugger Unsinn.
  **Beispiel:** Morgens per F5 geflasht und debuggt. Nachmittags
  main.c geändert und mit F7 neu GEBAUT — aber nicht geflasht. Auf dem
  Gerät läuft noch der Morgen-Stand, die .elf auf der Platte ist schon
  der Nachmittags-Stand. Ein Attach behauptet dann z. B. "angehalten in
  Zeile 120, LED-Routine", während die CPU real irgendwo im ADC-Code
  steht, und Variablen zeigen Zufallswerte — nichts davon ist ein
  Defekt, nur eine veraltete Landkarte. Faustregel: Attach nur
  verwenden, wenn seit dem letzten Flashen NICHT neu gebaut wurde; im
  Zweifel einmal per F5 flashen, dann passt es wieder. Für Feldgeräte
  gilt dasselbe: Zum getaggten Stand v1.0 gehört die .elf aus genau
  diesem Build — auch deshalb Release-Stände samt Dateien als
  GitHub-Release archivieren.
- **`preLaunchTask`** — verweist auf einen Task in `.vscode/tasks.json`;
  damit baut F5 vor dem Flashen automatisch. Anfangs leer lassen —
  bewusstes Bauen mit F7 verhindert, dass man versehentlich einen
  veralteten Stand debuggt und sich wundert.
- `${workspaceFolder}` ist eine VS-Code-Variable für den Projektordner
  und bleibt wörtlich so stehen.

**Anpassen pro Projekt:** `executable` (Projektname der .elf, in allen
Konfigurationen) und `device`; bei FreeRTOS die `rtos`-Zeile aktivieren.

**Braucht es eine eigene Release-Konfiguration?** Für den Alltag nicht —
deshalb liegt sie in der Vorlage nur auskommentiert bei. Debuggt wird
normalerweise der Debug-Build; die Release-Konfiguration ist für den
seltenen, aber wichtigen Sonderfall "der Fehler tritt NUR im Release
auf" (typisch bei Optimierungs-empfindlichem Code, z. B. fehlendem
`volatile`). Zwei Dinge muss man dazu wissen: Erstens baut unser
Release mit `-g0`, also OHNE Debug-Symbole — der Debugger wäre damit
fast blind. Abhilfe: in `cmake/gcc-arm-none-eabi.cmake` beim
Release-Zweig `-g0` durch `-g3` ersetzen. Das ist gefahrlos, denn `-g`
ändert ausschließlich die Zusatzinformationen IN der .elf-Datei — das
erzeugte Maschinenprogramm und damit die geflashte .hex bleiben
byte-identisch. Zweitens bleibt Release-Debugging auch mit Symbolen
holprig (der Optimierer ordnet um: scheinbares Springen beim Steppen,
Variablen zeigen `<optimized out>`) — das ist normal und kein Defekt.
Und zur Abgrenzung: Zum ERSTELLEN und FLASHEN eines Release trägt die
launch.json nichts bei — dieser Weg läuft immer über das Terminal
(`cmake --preset Release`, Build, `STM32_Programmer_CLI`; kompletter
Ablauf im Beispiel-Durchlauf in Teil 3). F5 und die launch.json sind
reine Debug-Werkzeuge.

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

### 2.5 Git und GitHub — was wir tun und warum

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

#### Serienstände veröffentlichen: GitHub-Releases

Binärdateien (.hex/.bin/.dfu) gehören NICHT ins Repository — dafür steht
`build/` ja in der .gitignore. Trotzdem soll ein ausgelieferter Stand
dauerhaft griffbereit sein, ohne ihn neu bauen zu müssen. Genau dafür
gibt es **Releases**: Sie hängen Dateien an einen Tag — also an exakt
den Quellstand, aus dem sie gebaut wurden. "Welche Firmware bekam
Kunde X?" ist damit ein Klick.

Weg 1 — Kommandozeile (empfohlen, ein Befehl macht Tag, Release-Seite
und Datei-Anhang in einem):

```bash
cmake --preset Release
cmake --build --preset Release
gh release create v1.0 build/Release/Projekt123.hex --title "v1.0" --notes "Erste Serienversion. Gebaut mit STM32CubeCLT <Version>."
```

Mehrere Dateien (z. B. zusätzlich .bin oder .dfu) einfach hintereinander
auflisten. Existiert der Tag v1.0 schon, wird er verwendet; sonst legt
`gh` ihn auf dem aktuellen Commit an.

Weg 2 — Browser: Repo-Seite auf github.com → rechts **"Releases"** →
**"Draft a new release"** → Tag wählen oder neu anlegen (v1.0) → Titel
und Beschreibung (Toolchain-Version!) → die .hex per Drag & Drop in das
Anhangsfeld ziehen → **"Publish release"**.

Das Ergebnis liegt unter `github.com/<benutzer>/<repo>/releases`: pro
Version die Beschreibung, der Quellstand (automatisch als zip/tar) und
deine angehängten Binärdateien zum Download.

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

**Hardware-Änderung nötig? Der komplette Ablauf Schritt für Schritt:**

Vorweg zur Frage "wo gebe ich das ein": Alle `git`- und `cmake`-Befehle
laufen in einem Terminal im Projektordner — am bequemsten das
**integrierte Terminal in VS Code** (Strg+Ö), das öffnet bereits im
richtigen Ordner. Ein externes Terminal-Fenster tut es genauso, die
Befehle sind identisch. Für fast alles gibt es zusätzlich einen
Klick-Weg in VS Code; beide stehen unten jeweils dabei.

**Schritt 0 — sauberer Ausgangspunkt:**

```bash
git status        # Soll: "nichts zu committen"
```

Falls doch noch eigene Änderungen offen sind: erst committen. Nur so
zeigt das Diff nachher AUSSCHLIESSLICH, was der Generator getan hat.

**Schritt 1 — Ändern:** `.ioc` in CubeMX öffnen, Hardware anpassen
(Pin, Peripherie, Takt), **GENERATE CODE**.

**Schritt 2 — Ansehen, was der Generator getan hat:**

```bash
git status        # welche Dateien wurden angefasst?
git diff          # was genau, Zeile fuer Zeile? (Blaettern: Leertaste, Beenden: q)
```

Klick-Weg: Quellcode-Ansicht öffnen (**Strg+Umschalt+G**) — sie listet
alle geänderten Dateien; ein Klick auf eine Datei öffnet den
Seite-an-Seite-Vergleich (links alt, rechts neu). Für Einsteiger meist
angenehmer als das Terminal-Diff. Erwartung: Änderungen nur in
generierten Bereichen (MX_..._Init, ioc, ggf. msp/it) — die eigenen
USER-CODE-Blöcke müssen unangetastet sein.

**Schritt 3 — Bauen (ja, das ist wichtig):**

```bash
cmake --build --preset Debug
```

(oder F7.) Erst der erfolgreiche Build beweist, dass der neue
generierte Stand kompiliert — NIEMALS einen ungebauten Stand committen,
sonst liegt ein kaputter Zwischenstand in der Historie, und der nächste
`git checkout` darauf fällt auf die Nase.

**Schritt 4 — Festschreiben:**

```bash
git add -u
git commit -m "CubeMX: UART3 ergaenzt"
git push
```

Generator-Änderungen und Handänderungen nie im selben Commit mischen —
erst der CubeMX-Commit, danach separat der Code, der die neue Hardware
benutzt.

**Und wenn etwas schiefging? Rückgängig machen in drei Stufen:**

*Stufe 1 — Änderungen sind noch NICHT committet* (der häufigste Fall,
z. B. versehentlich das Falsche in CubeMX generiert):

```bash
git restore Core/Src/main.c   # eine Datei auf den letzten Commit zuruecksetzen
git restore .                 # ALLE offenen Aenderungen verwerfen (Vorsicht: endgueltig!)
```

Klick-Weg: Quellcode-Ansicht → Pfeil-Symbol ("Änderungen verwerfen") an
der Datei bzw. oben für alle.

*Stufe 2 — die Änderung ist schon committet:* Nicht löschen, sondern
umkehren — `git revert` erzeugt einen neuen Commit, der den alten exakt
rückgängig macht. Die Historie bleibt vollständig und ehrlich:

```bash
git log --oneline             # Historie auflisten: ein Commit pro Zeile,
                              #   vorne der Kurz-Hash (z. B. a1b2c3d); Beenden: q
git revert a1b2c3d            # diesen Commit umkehren (Editor bestaetigen)
git push
```

*Stufe 3 — nur nachschauen, wie es früher war:*

```bash
git log --oneline                          # Hash des gewuenschten Stands suchen
git checkout a1b2c3d                       # kompletten alten Stand ansehen ("detached HEAD" -
                                           #   nur Leseausflug, hier nichts committen!)
git checkout main                          # zurueck in die Gegenwart
git restore --source a1b2c3d Core/Src/main.c   # oder: nur EINE Datei aus einem
                                               #   alten Commit in die Gegenwart holen
```

Klick-Weg für die Datei-Historie: Datei im Explorer markieren → unten
in der Seitenleiste die **"Zeitachse" (Timeline)** aufklappen — sie
listet alle Commits, die diese Datei geändert haben; ein Klick zeigt
den Vergleich. (Noch komfortabler wird das mit der optionalen
GitLens-Erweiterung aus der extensions.json.)

Was es bewusst NICHT braucht: `git reset --hard` wirft committete
Historie unwiederbringlich weg — für unseren Workflow gibt es dafür
keinen Grund; `revert` und `restore` decken alles ab und bleiben
nachvollziehbar.


#### Beispiel: kompletter Durchlauf vom Debug bis zur Auslieferung

Wichtigste Erkenntnis vorweg: **Für den Wechsel von Debug auf Release
wird KEINE einzige Quelldatei geändert.** Genau dafür gibt es die
Presets — derselbe Code, zwei Übersetzungen. Angepasst werden höchstens
Pfade in Befehlen (`build/Debug/` → `build/Release/`).

**Schritt 1 — Entwickeln, tage- oder wochenlang, alles im Debug:**

```bash
cmake --preset Debug              # einmalig pro Projekt
cmake --build --preset Debug      # nach jeder Aenderung (oder F7)
```

F5 flasht und debuggt den Debug-Build; `DBG_MELDUNG(...)`-Ausgaben sind
aktiv. Zwischenstände wie gewohnt committen und pushen.

**Schritt 2 — Release bauen (identischer Quellstand!):**

```bash
cmake --preset Release            # einmalig pro Projekt
cmake --build --preset Release
```

Ergebnis liegt in `build/Release/`: deutlich kleinere .hex, und alle
`#ifdef DEBUG`-Ausgaben fehlen automatisch — ohne dass irgendeine Datei
angefasst wurde.

**Schritt 3 — DIESEN Build flashen und auf der Hardware validieren:**

```bash
STM32_Programmer_CLI -c port=SWD -w build/Release/Projekt123.hex -v -rst
```

Validiert wird der Release, nicht der Debug — denn der Release ist das,
was ausgeliefert wird (die Optimierung erzeugt ein anderes
Maschinenprogramm, siehe Abschnitt 2.2).

**Schritt 4 — Festschreiben und veröffentlichen:**

```bash
git tag -a v1.0 -m "v1.0 - gebaut mit STM32CubeCLT <Version>"
git push origin v1.0
gh release create v1.0 build/Release/Projekt123.hex --title "v1.0" --notes "Erste Serienversion."
```

**Wo Pfade angepasst werden — und wo nichts angefasst wird:**

- `launch.json` zeigt bewusst auf `build/Debug/…` und bleibt so:
  Release-Builds sind wegen der Optimierung kaum sinnvoll debugbar.
  Einzige Ausnahme — ein Fehler tritt NUR im Release auf: dafür die
  auskommentierte Konfiguration "Release debuggen" in der launch.json
  aktivieren (und für brauchbare Symbole `-g0` → `-g3`, siehe
  Abschnitt 2.4).
- Flash-Befehl: Pfad je nach Zweck `build/Debug/` (Entwicklung) oder
  `build/Release/` (Validierung/Auslieferung).
- Quellcode, CMakeLists, Presets, .ioc: **unverändert** — der gesamte
  Unterschied steckt in der Preset-Wahl.

**Serienstand:** Release-Build aus dem Terminal
(`cmake --preset Release && cmake --build --preset Release`), auf
Hardware validieren, Version taggen, README (Größen, Toolchain) nachführen und die .hex
als GitHub-Release veröffentlichen (Abschnitt 2.5).

---

## Exkurs: FreeRTOS — über den CubeMX-Haken oder von Hand einbinden?

Für RTOS-Projekte gibt es zwei legitime Wege; für den hier beschriebenen
Standard ist **Weg A der Normalfall**, Weg B braucht einen konkreten
Grund.

**Weg A — Haken in CubeMX** (Pinout & Configuration → Middleware →
FREERTOS, Schnittstelle CMSIS_V2): CubeMX legt eine Kernel-Kopie unter
`Middlewares/` ab, erzeugt die `FreeRTOSConfig.h` aus den
ioc-Einstellungen, stellt die HAL-Zeitbasis automatisch auf einen
Hardware-Timer um (der SysTick gehört dann dem RTOS) und generiert
Task-Gerüste mit der CMSIS-RTOS2-API (`osThreadNew`, `osDelay`).
*Vorteile:* null Integrationsaufwand, Konfiguration per GUI,
regenerierungssicher, von ST getestete Kombination. *Nachteile:* Die
Kernel-Version hängt am STM32Cube-Firmwarepaket (meist nicht die
neueste), und die CMSIS-RTOS2-Schicht liegt als Wrapper zwischen Code
und Kernel — bequem, aber eine Indirektion, die nicht jedes
FreeRTOS-Feature durchreicht.

**Weg B — Kernel selbst einbinden** (Haken NICHT setzen) lohnt sich,
wenn mindestens einer dieser Gründe zutrifft: eine aktuelle
Kernel-Version wird gebraucht (Bugfixes, neue Features); die native
FreeRTOS-API ist gewünscht (`xTaskCreate` statt Wrapper); die
Kernel-Version soll selbst versioniert sein (Git-Submodule mit
Versions-Tag = reproduzierbar und über mehrere Projekte identisch);
oder es werden Features jenseits des Wrappers genutzt (durchgängig
statische Allokation, Stream Buffers direkt, Kernel-Hooks).

Vorgehen für Weg B in Kurzform:

1. **CubeMX:** FREERTOS nicht aktivieren, aber trotzdem
   *SYS → Timebase Source = TIM1* (o. ä.) einstellen, damit der SysTick
   für das RTOS frei bleibt. In den NVIC-Codegenerierungs-Einstellungen
   die Erzeugung von SVC-, PendSV- und SysTick-Handler abwählen — diese
   drei Vektoren gehören dem Kernel.
2. **Kernel als versioniertes Submodule holen** (der Versions-Tag macht
   den Stand reproduzierbar):

```bash
git submodule add https://github.com/FreeRTOS/FreeRTOS-Kernel.git Middlewares/FreeRTOS-Kernel
cd Middlewares/FreeRTOS-Kernel
git checkout V11.1.0
cd ../..
```

   Kollegen-Hinweis: Ein Repo mit Submodule klont man mit
   `git clone --recurse-submodules …` (oder danach
   `git submodule update --init`).
3. **`Core/Inc/FreeRTOSConfig.h` selbst anlegen** (Vorlagen liegen im
   Kernel-Repo unter examples). Drei Zeilen darin ersparen jede
   Handler-Bastelei, weil sie die Kernel-Routinen direkt auf die
   Cortex-M-Vektoren verdrahten:

```c
#define vPortSVCHandler     SVC_Handler
#define xPortPendSVHandler  PendSV_Handler
#define xPortSysTickHandler SysTick_Handler
```

4. **Root-CMakeLists** (im User-Bereich, regenerierungssicher) — der
   Kernel bringt offizielle CMake-Unterstützung mit:

```cmake
set(FREERTOS_PORT GCC_ARM_CM0 CACHE STRING "")   # Port zum Kern waehlen (M0+: CM0, M4F: CM4F, ...)
add_library(freertos_config INTERFACE)
target_include_directories(freertos_config INTERFACE Core/Inc)
add_subdirectory(Middlewares/FreeRTOS-Kernel)
target_link_libraries(${CMAKE_PROJECT_NAME} freertos_kernel)
```

5. **Tasks mit nativer API** anlegen (`xTaskCreate`, `vTaskDelay`, …) —
   wie immer in USER-CODE-Blöcken oder eigenen Dateien.

Was sich gegenüber Weg A ändert: kein `Middlewares/Third_Party/`-Ordner
von CubeMX, keine CMSIS-RTOS2-Dateien, `FreeRTOSConfig.h` wird von Hand
gepflegt statt aus der ioc generiert — dafür bestimmt das Projekt die
Kernel-Version selbst. Nicht mischen: Entweder verwaltet CubeMX das
RTOS oder das Projekt, beides zugleich führt zu doppelten Kernel-Kopien
und Handler-Konflikten.

**Warum ein Submodule statt "herunterladen und ablegen"?** Ein
Submodule speichert im eigenen Repo nicht die fremden Dateien, sondern
nur einen ZEIGER auf ein exaktes Commit/Tag des fremden Repos — die
Version ist damit technisch garantiert, nicht bloß irgendwo notiert.
Updates sind ein bewusster Mini-Commit ("V11.1.0 → V11.2.0") statt
eines Riesen-Diffs aus Ordner-löschen-und-neu-kopieren, bei dem lokal
hineingeflickte Änderungen unbemerkt überschrieben würden; und mehrere
Projekte können nachweisbar auf exakt denselben Kernel-Stand zeigen.
Die Kopier-Variante ("Vendoring") ist dagegen dort richtig, wo ein
GENERATOR den Fremdcode pflegt — genau deshalb liegt die HAL als von
CubeMX verwaltete Kopie unter `Drivers/` im Repo (Version steht in der
ioc, Erneuerung nur über Generate, Repo bleibt offline baubar).
Faustregel: Generator-gepflegt → Kopie im Repo; selbst verwalteter
Fremdcode → Submodule mit Versions-Tag.

**So sieht ein Update konkret aus — beide Wege im Vergleich:**

*Update bei Weg A (CubeMX verwaltet Kernel und HAL):* Die Version hängt
am STM32Cube-Firmwarepaket. Ablauf:

1. Sauberer Stand: `git status` muss leer sein — sonst erst committen.
2. CubeMX: *Help → Manage Embedded Software Packages* → neuere Version
   des Pakets (z. B. STM32Cube FW_G0) installieren. Dann die `.ioc`
   öffnen — CubeMX bietet die Migration auf das neue Paket an —
   bestätigen und **Generate Code**.
3. Jetzt die Diff-Arbeit. Das Diff ist riesig (tausende Zeilen in
   `Drivers/` und `Middlewares/`) — das liest niemand Zeile für Zeile,
   und das muss man auch nicht. Stattdessen in drei Blicken:

```bash
git diff --stat                              # Blick 1: Uebersicht - WELCHE Dateien,
                                             #   wie viele Zeilen? (Erwartung: fast
                                             #   alles in Drivers/ und Middlewares/)
git diff -- ':!Drivers' ':!Middlewares'      # Blick 2: das Diff OHNE die generierten
                                             #   Bibliotheken - uebrig bleibt nur, was
                                             #   den eigenen Bereich beruehrt
git diff -- Core/                            # Blick 3: gezielt die eigenen Dateien -
                                             #   die USER-CODE-Bloecke muessen
                                             #   unveraendert sein
```

   Die Ausrufezeichen-Schreibweise `':!Ordner'` blendet Pfade aus dem
   Diff aus — der wichtigste Diff-Trick bei Generator-Updates.
4. Bauen, auf der Hardware gegentesten, dann als EIN Commit:
   `git commit -m "CubeMX: FW-Paket G0 1.6.x -> 1.7.x"`. Fällt danach
   etwas auf: `git revert` dieses einen Commits macht das komplette
   Update sauber rückgängig.

*Update bei Weg B (Submodule):* Hier ist das Update ein bewusster
Zeigerwechsel — und das Diff im Hauptprojekt besteht aus EINER Zeile:

```bash
cd Middlewares/FreeRTOS-Kernel
git fetch --tags                             # neue Versions-Tags holen
git tag --list "V11.*"                       # was gibt es?
git log --oneline V11.1.0..V11.2.0           # was hat sich im Kernel geaendert?
                                             #   (plus CHANGELOG/Release Notes lesen)
git checkout V11.2.0
cd ../..
git diff                                     # zeigt genau eine Aenderung:
                                             #   -Subproject commit abc123...
                                             #   +Subproject commit def456...
git add Middlewares/FreeRTOS-Kernel
git commit -m "FreeRTOS-Kernel V11.1.0 -> V11.2.0"
git push
```

   Danach ebenfalls: bauen und auf der Hardware gegentesten. Gefällt
   das Ergebnis nicht, ist der Rückweg trivial: im Submodule wieder
   `git checkout V11.1.0`, Zeiger committen — oder gleich
   `git revert` des Update-Commits.

Der Unterschied in einem Satz: Bei Weg A vertraut man dem Generator
und prüft per Ausschluss-Diff, dass der eigene Bereich unberührt
blieb; bei Weg B ist das Update selbst nur eine Zeile, und was sich
inhaltlich geändert hat, liest man gezielt im Log des Fremd-Repos.

---

## Exkurs: Peripherie-Init als eigene .c/.h-Dateien pro Peripherie

Im CubeMX Project Manager gibt es unter dem Reiter **Code Generator**
den Haken **"Generate peripheral initialization as a pair of '.c/.h'
files per peripheral"**. Er entscheidet, WO der generierte
Initialisierungscode landet — und ist damit eine Weichenstellung für
die Lesbarkeit des ganzen Projekts.

**Weg A — Haken AUS (CubeMX-Voreinstellung):** Sämtliche
`MX_..._Init()`-Funktionen, alle Peripherie-Handles (`hadc1`,
`huart2`, …) und deren USER-CODE-Blöcke landen gesammelt in `main.c`.
*Vorteil:* alles an einer Stelle, eine Datei weniger zu denken — bei
Mini-Projekten mit zwei Peripherien okay. *Nachteil:* `main.c` wächst
schnell auf viele hundert Zeilen, Applikationslogik und
Hardware-Initialisierung vermischen sich, und nach jedem "Generate
Code" ist das `git diff` von main.c groß und unübersichtlich, weil
Generator- und Handcode in derselben Datei leben.

**Weg B — Haken AN (unser Standard für neue Projekte):** CubeMX erzeugt
pro Peripherie ein Dateipaar in `Core/`: `gpio.c/.h`, `adc.c/.h`,
`usart.c/.h`, `tim.c/.h` usw. Dort liegen jeweils die
`MX_..._Init()`-Funktion, das Handle (im Header als `extern`
verfügbar) — und **eigene USER-CODE-Blöcke pro Datei**. `main.c`
schrumpft auf Takt-Setup und die Init-Aufrufe.

*Was sich dadurch ändert bzw. angepasst werden muss:*

- **Nichts am Build:** CubeMX trägt die neuen Dateien selbst in
  `cmake/stm32cubemx/CMakeLists.txt` ein — bauen, flashen, debuggen
  laufen unverändert.
- **Eigener Code bekommt ein natürliches Zuhause:** Hilfsfunktionen zu
  einer Peripherie gehören in DEREN Datei (z. B. eine
  `UART_Sende_Debugzeile()` in den USER-Block von `usart.c`, nicht nach
  main.c). Das hält main.c bei der Applikationslogik.
- **`git diff` nach "Generate Code" wird präzise:** Eine
  ADC-Änderung in CubeMX ändert `adc.c/.h` — und sonst fast nichts.
  Generator-Commits werden dadurch klein und selbsterklärend.

*Vorteile* insgesamt: Struktur und Auffindbarkeit ("wo ist der UART?"
→ `usart.c`), kleine und lesbare Diffs, weniger Konflikte bei
Teamarbeit, Peripherie-Code ist einzeln wiederverwendbar. *Nachteile:*
mehr Dateien (kosmetisch), und die Handles liegen nicht mehr in main.c
— wer `hadc1` in einer eigenen Datei braucht, inkludiert `adc.h`
(statt eines `extern` von Hand).

**Umstellen eines BESTEHENDEN Projekts** (Haken nachträglich setzen)
geht, braucht aber Sorgfalt: vorher committen (sauberer Stand!), Haken
setzen, Generate Code, dann per `git diff` prüfen — die
Init-Funktionen wandern aus main.c in die neuen Dateien, und **eigener
Code, der in main.c innerhalb der verschobenen Init-Funktionen stand
(z. B. im USER-Block von MX_ADC1_Init), muss von Hand in die
USER-Blöcke der neuen Datei umziehen**. Danach bauen und als eigener
Commit festschreiben ("CubeMX: Peripherie-Init in eigene Dateien").
Bei laufenden, stabilen Projekten lohnt die Umstellung meist nicht —
bei jedem NEUEN Projekt den Haken von Anfang an setzen.

---

## Teil 4: Checkliste für jedes neue Projekt

- [ ] CubeMX: Toolchain = CMake, Name ohne Leerzeichen/Umlaute
- [ ] Code Generator: Haken \"pair of '.c/.h' files per peripheral\" gesetzt
- [ ] Erster Build läuft (Terminal), Speichertabelle im README notiert
- [ ] .hex/.bin-Block ans Ende der Root-CMakeLists angehängt (Abschnitt 2.3)
- [ ] .gitignore, .vscode/, README.md ergänzt (inkl. CubeCLT-Version)
- [ ] Generierter Stand = erster Commit, Repo auf GitHub, Push erfolgreich
- [ ] Eigener Code nur in USER-CODE-Blöcken
- [ ] Keine leeren Zählschleifen für Timing — GCC optimiert sie ersatzlos
      weg! Immer HAL_Delay oder einen Hardware-Timer verwenden.
- [ ] Timing-Konstanten als benannte #define, nicht als nackte Zahlen
- [ ] Bei Flash-EEPROM-Emulation: Seiten im Linkerscript reservieren
      (FLASH-LENGTH kürzen)
- [ ] Release-Build getestet, auf Hardware validiert, Version getaggt
      und als GitHub-Release (.hex im Anhang) veröffentlicht

---

## Teil 5: Troubleshooting — typische Fehler und ihre Lösung

**F5: "ST-LINK: GDB Server Quit Unexpectedly"**
Zwei häufige Ursachen. (1) Kein ST-Link angeschlossen — dann ist die
Meldung normal; Details stehen im Terminal-Reiter "gdb-server".
(2) Cortex-Debug übergibt dem GDB-Server einen falschen
CubeProgrammer-Pfad — erkennbar im Debug-Log an `-cp` mit einem Pfad,
den es auf dem Rechner nicht gibt (Standard einer separaten
CubeProgrammer-Installation statt CubeCLT). Fix in
`.vscode/settings.json` des Projekts:

```json
{
    "cortex-debug.stm32cubeprogrammer": "/opt/st/stm32cubeclt_<version>/STM32CubeProgrammer/bin"
}
```

Danach Fenster neu laden; im nächsten Debug-Log muss `-cp` auf den
CubeCLT-Pfad zeigen. Diese settings.json ist auch der richtige Ort für
die IntelliSense-Einstellung (siehe cpptools/clangd-Eintrag) — beides
zusammen versioniert im Projekt.


**"Binärdatei kann nicht ausgeführt werden: Fehler im Format der Programmdatei" oder "No compiler found in cache file"**
Es wurde der falsche Play-Knopf gedrückt: Die Symbole Play/Käfer unten
in der STATUSLEISTE gehören zu CMake Tools und versuchen, das Programm
AUF DEM PC zu starten bzw. zu debuggen — die .elf ist aber
ARM-Maschinencode für den Controller, das kann nicht funktionieren
(beide Meldungen sind harmlos, nichts ist beschädigt). Richtiger Weg
zum Flashen/Debuggen: Debug-ANSICHT öffnen (Strg+Umschalt+D), oben im
Dropdown die Konfiguration wählen, dann F5. Merkregel: Statusleiste
unten = bauen (F7); Debug-Ansicht links = auf den Controller (F5).


**Warnung: "cpptools extension and stm32-cube-clangd extension enabled ... conflict"**
Die ST-Erweiterung installiert einen eigenen IntelliSense-Motor mit
(stm32-cube-clangd); zusammen mit dem der C/C++-Erweiterung liefern
dann zwei Motoren gleichzeitig Vervollständigung und Fehleranzeige.
Lösung: den in der Meldung angebotenen Knopf zum Deaktivieren des
Microsoft-IntelliSense klicken — oder manuell in den Einstellungen
"C_Cpp: Intelli Sense Engine" auf `disabled` stellen (Reiter
"Workspace"). clangd ist hier die bessere Wahl: Es liest die vom
CMake-Build erzeugte compile_commands.json und kennt damit exakt die
Flags und Defines des echten Builds. Bauen und Debuggen sind von der
Wahl nicht betroffen. Selbsttest, dass clangd sauber arbeitet: in
main.c testweise `HAL_GPIO_` tippen — die Vervollständigung muss die
HAL-Funktionen anbieten; ein absichtlicher Tippfehler wird rot
unterkringelt, und die Markierung verschwindet nach der Korrektur
sofort. Dann ist die Editor-Anzeige an den echten Build gekoppelt.


**Die Empfehlungs-Abfrage für Erweiterungen erscheint nicht**
Drei mögliche Ursachen: (1) Alle empfohlenen Erweiterungen sind bereits
installiert — dann gibt es nichts zu fragen (Erweiterungen gelten
VS-Code-weit, nicht pro Projekt). (2) Die extensions.json liegt am
falschen Ort (siehe Doppel-.vscode unten). (3) Das Popup wurde früher
weggeklickt. Der zuverlässige Weg ohne Popup: Erweiterungs-Ansicht
(Strg+Umschalt+X) → oben ins Suchfeld `@recommended` eingeben → Rubrik
"Workspace Recommendations" zeigt die Einträge aus der Datei, das
Wolken-Symbol daneben installiert alle fehlenden auf einen Schlag.
Ist die Rubrik leer, wird die Datei nicht gelesen: Ort oder Syntax
prüfen (fehlendes/überzähliges Komma).

**launch.json: "Property servertype/device/... is not allowed" und "The debug type is not recognized"**
Das heißt fast nie "Feld falsch", sondern fast immer: Die zugehörige
Debug-Erweiterung fehlt oder ist nicht geladen — ohne Cortex-Debug
kennt VS Code den Typ `cortex-debug` nicht und prüft die Datei gegen
die falschen Schemas. Cortex-Debug installieren/aktivieren, Fenster neu
laden ("Developer: Reload Window") — die Warnungen verschwinden.

**Dateien in .vscode/ werden ignoriert (Konfigurationen tauchen nirgends auf)**
Klassiker: Beim Anlegen im Explorer-Panel war der .vscode-Ordner
markiert, und es entstand ein VERSCHACHTELTER Ordner `.vscode/.vscode/`.
Kontrolle und Korrektur:

```bash
ls .vscode                      # Soll: extensions.json  launch.json
mv .vscode/.vscode/* .vscode/ && rmdir .vscode/.vscode   # falls verschachtelt
```


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
