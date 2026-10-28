# Ein Spiel in den DasDimo Launcher bringen

**Für:** den Claude-Agenten eines Spiels von DasDimo (GitHub-Konto `dasdimo`) – egal mit welcher Engine.
**Version der Anleitung:** 2 (2026-10-29) · Hilfs-Skript `dasdimo_release.js` Version 2 (Anhang).
**Neu in Version 2:** Screenshots für die Spielseite (`catalog.screenshots`), Eintrag mit `"type": "game"`, Spiele gibt es nur noch über den Launcher (keine direkten Links auf Spiel-Dateien), Test mit dem DasDimo Launcher (Schritt 11). Hast du schon Version 1 eingerichtet: Skript aus dem Anhang übernehmen, Screenshots ergänzen, README anpassen, dann `register`.
**Öffentliche Adresse:** https://github.com/dasdimo/dasdimo-launcher-releases/blob/main/INTEGRATION.md
Zum Lesen in Claude Code: `gh api repos/dasdimo/dasdimo-launcher-releases/contents/INTEGRATION.md -H "Accept: application/vnd.github.raw"`

Lies diese Datei ganz, bevor du anfängst. Arbeite die Schritte der Reihe nach ab und hake sie in deiner Aufgabenliste ab.

---

## Worum es geht
Der **DasDimo Launcher** ist ein kleines Windows-Programm für die Freunde des Benutzers. Er zeigt alle Spiele von DasDimo, installiert sie mit einem Klick, holt Updates und startet sie. Damit dein Spiel darin erscheint, braucht es:
1. ein **öffentliches Release-Repo** `dasdimo/<spiel-id>-releases` mit fertigen Builds (der Quellcode bleibt privat),
2. pro Version ein **GitHub-Release** mit der Spiel-exe für Windows x64 (und wenn möglich ARM64) und einer `manifest.json` mit SHA-256-Prüfsummen,
3. **Bilder** (Icon, Cover, Screenshots) im Release-Repo,
4. einen **Eintrag in der Spieleliste** `games.json` der Zentrale `dasdimo/dasdimo-launcher-releases`.

Das Hilfs-Skript im Anhang erledigt Manifest, Release, Bilder und Eintrag. Du schreibst nur eine kleine Konfiguration.

**Spiele gibt es nur über den Launcher.** Freunde installieren einmal den DasDimo Launcher (Installer `DasDimoLauncher-Setup.exe` aus https://github.com/dasdimo/dasdimo-launcher-releases) und bekommen darüber alle Spiele. README und Release-Notizen deines Spiels verlinken deshalb den Launcher, nie die Spiel-Dateien direkt. Der Launcher liest die Spieleliste bei jedem Start neu: Sobald dein Eintrag drin ist, erscheint dein Spiel bei allen – ohne Launcher-Update.

## Regeln (gelten immer)
1. **Sprache:** Mit dem Benutzer Deutsch, einfache Worte. Er kann nicht programmieren. Entscheidungen als Auswahl-Fragen mit 2–4 Vorschlägen, die empfohlene zuerst mit „(Empfohlen)“.
2. **Erst fragen, dann veröffentlichen:** Repo anlegen, Release hochladen, Bilder hochladen, Spieleliste ändern – jedes Mal vorher sagen, was öffentlich wird, und auf ein klares Ja warten.
3. **Keine private E-Mail in öffentlichen Repos:** In Release-Repos und in der Zentrale **nie** `git commit`/`git push` und kein `gh repo create --add-readme`. Dateien kommen nur über das Hilfs-Skript (GitHub-API mit der noreply-Adresse `<id>+dasdimo@users.noreply.github.com`). Releases selbst erzeugen keine Commits.
4. **Releases nur per Skript:** immer `dasdimo_release.js release`, nie per Hand im Browser oder mit einzelnen `gh release`-Befehlen (Ausnahme: Test-Release löschen, Schritt 13).
5. **Downloads nur mit Erlaubnis:** Dateiname, Quelle und Größe nennen und fragen.
6. **Nur fertige Builds öffentlich:** kein Quellcode, keine Zugangsdaten/Schlüssel, keine Rohdateien von Assets, die nicht weitergegeben werden dürfen (z. B. Mixamo-FBX). In einer fertigen exe eingebaut ist in Ordnung.
7. **Nur dein eigener Eintrag:** Ändere in `games.json` nie andere Spiele (das Skript ändert nur den Eintrag mit deiner `game_id`). Am Launcher-Code arbeitest du nicht; Wünsche oder Fehler am Launcher meldest du dem Benutzer.
8. `builds/` und `builds/dasdimo-release/` nicht ins Git deines Spiels committen. PowerShell-Skripte (`.ps1`) nur mit ASCII-Zeichen schreiben (Windows PowerShell 5.1).

## Werkzeuge
- **GitHub CLI** `gh`, angemeldet als `dasdimo` (`gh auth status`). Auf dem PC des Benutzers: `C:\Program Files\GitHub CLI\gh.exe`.
- **Node.js 18+** (`node --version`) für das Hilfs-Skript.
- Die Engine deines Spiels zum Exportieren der Windows-Builds.

---

## Schritt 1 – Plan zeigen und Grunddaten klären
Lies die Projektregeln deines Spiels (z. B. `CLAUDE.md`). Zeig dem Benutzer kurz den Plan (diese Schritte) und frag die Grunddaten ab (AskUserQuestion):

| Was | Regel | Beispiel |
|---|---|---|
| `game_id` | klein, `a-z`, `0-9`, `-`, 2–32 Zeichen, **nie mehr ändern** (wird Ordnername) | `beispielspiel` |
| Anzeigename | wie im Spiel | `Beispielspiel` |
| Release-Repo | `dasdimo/<game_id>-releases` | `dasdimo/beispielspiel-releases` |
| Untertitel DE/EN | ein Satz, höchstens 90 Zeichen, für Freunde verständlich | „Kleines Partyspiel für zwischendurch.“ |
| Akzentfarbe | Hex `#RRGGBB`, passend zum Spiel (Knopf und Leuchten im Launcher) | `#FF5A1F` |
| ARM64-Version | ja, wenn die Engine es kann (Snapdragon-Laptops), sonst nur x64 | ja |
| Status | `testing` (sichtbar mit Abzeichen „Test“), `released`, `hidden` (nur zum Ausprobieren) | `testing` |

## Schritt 2 – Spiel vorbereiten
Dein Build muss diese Punkte erfüllen. Prüfe jeden und repariere, was fehlt:
- **Startet ohne Installer und ohne Admin-Rechte** per Doppelklick aus jedem Ordner, z. B. `%LOCALAPPDATA%\DasDimo-Launcher\Games\<game_id>\versions\1.2.0\`. Pfade mit Leerzeichen und Umlauten müssen gehen.
- **Schreibt nichts neben die exe.** Der Versionsordner wird bei Updates gelöscht. Einstellungen, Spielstände und Logs gehören nach `%APPDATA%`, `%LOCALAPPDATA%` oder `%USERPROFILE%\AppData\LocalLow` (Godot `user://` = `%APPDATA%\Godot\app_userdata\<Projektname>` – das passt). Notiere diese Ordner für Schritt 5 (`user_data_dirs`).
- **Verlässt sich nicht auf das Arbeitsverzeichnis** und braucht **keine Start-Argumente** (der Launcher startet die exe ohne Argumente).
- **Kein eigenes Update-System** im Spiel, keine Zusatz-Installer (z. B. VC++-Pakete) nötig.
- **Eine Datei:** am besten eine einzelne `.exe` (Format 1). Besteht dein Spiel aus einem Ordner, wird er als `.zip` verpackt (Format 2, Schritt 7) – das kann der Launcher erst ab Phase L4, frag den Benutzer, ob er so lange warten will.
- **Version an einer Stelle** im Projekt (z. B. `project.godot` → `config/version`), Form `1.2.3`, und dieselbe Version in der Windows-Dateiinfo der exe.

**Engine-Hinweise:**
- **Godot 4:** Export „Windows Desktop“ mit `binary_format/embed_pck=true` → eine exe. Für ARM64 ein zweites Preset mit `binary_format/architecture="arm64"` (die offiziellen Export-Vorlagen enthalten `windows_release_arm64.exe`; prüfe den Vorlagen-Ordner und frag vor einem Download). Auf Snapdragon läuft Direct3D 12 meist besser als Vulkan: RIFTWAKE setzt im ARM64-Preset `custom_features="force_d3d12"` und im Projekt `rendering/rendering_device/driver.force_d3d12="d3d12"`.
- **Unity:** Windows-Build x64 (und ARM64, wenn die Unity-Version es anbietet) ist ein Ordner → `.zip`, Format 2.
- **Unreal:** Ordner → `.zip`, Format 2; meist nur x64.
- **Andere Engines:** eine exe → Format 1, sonst `.zip` → Format 2.

## Schritt 3 – Release-Repo anlegen (erst nach Ja des Benutzers)
```powershell
gh repo create dasdimo/<game_id>-releases --public --description "<Name> – Spielversionen zum Herunterladen / game downloads" --disable-issues --disable-wiki
```
Keine Optionen `--add-readme`, `--clone` oder `--source` (sonst Commit mit privater E-Mail). Die README kommt beim ersten Release über das Skript.

## Schritt 4 – Builds und Dateinamen
- Builds landen im Projekt unter `builds/` (nicht in Git).
- Dateinamen: `<Name>-v<version>.exe` (x64) und `<Name>-v<version>-arm64.exe` (ARM64), bei Zips `.zip`. Nur `A–Z a–z 0–9 . _ -`, keine Leerzeichen/Umlaute (`Beispielspiel-v0.1.0.exe`).
- Das Skript liest die Version aus deinem Projekt und setzt sie für `{version}` ein.

## Schritt 5 – Konfiguration `tools/dasdimo/dasdimo.json`
Lege diese Datei in deinem Projekt an (Beispiel, anpassen):

```json
{
	"game_id": "beispielspiel",
	"game_name": "Beispielspiel",
	"repo": "dasdimo/beispielspiel-releases",
	"version_file": "project.godot",
	"version_pattern": "^config/version=\"([^\"]+)\"",
	"files": [
		{"arch": "x86_64", "path": "builds/Beispielspiel-v{version}.exe"},
		{"arch": "arm64", "path": "builds/Beispielspiel-v{version}-arm64.exe"}
	],
	"news_file": "tools/dasdimo/news.json",
	"readme_file": "tools/dasdimo/README.md",
	"catalog": {
		"tagline": {"de": "Kleines Partyspiel für zwischendurch.", "en": "A small party game for in between."},
		"accent": "#FF5A1F",
		"engine": "Godot 4.5",
		"status": "testing",
		"user_data_dirs": ["%APPDATA%\\Godot\\app_userdata\\Beispielspiel"],
		"icon": "tools/dasdimo/art/icon.png",
		"cover": "tools/dasdimo/art/cover.jpg",
		"logo": "",
		"screenshots": ["tools/dasdimo/art/shot1.jpg", "tools/dasdimo/art/shot2.jpg", "tools/dasdimo/art/shot3.jpg"]
	}
}
```

| Feld | Bedeutung |
|---|---|
| `version_file` + `version_pattern` | Datei und regulärer Ausdruck (eine Klammer = die Version). Alternativ fest: `"version": "0.1.0"` |
| `files` | eine Zeile pro Plattform, `x86_64` Pflicht. Bei `.zip` zusätzlich `"exe": "Spiel.exe"` (Startdatei im Zip, relativer Pfad) |
| `news_file`, `readme_file` | Standardwerte wie oben; `out_dir` (Standard `builds/dasdimo-release`) für Manifest + Notizen |
| `catalog.user_data_dirs` | Ordner aus Schritt 2, nur unter `%APPDATA%\`, `%LOCALAPPDATA%\`, `%USERPROFILE%\AppData\LocalLow\`. Werden nur gelöscht, wenn ein Spieler beim Deinstallieren „Auch Einstellungen löschen“ anhakt. Leer lassen (`[]`), wenn unklar |
| `catalog.status` | aus Schritt 1 |
| `catalog.screenshots` | 0–8 Bilder für die Spielseite, in dieser Reihenfolge (Schritt 6). Leer lassen (`[]`) geht, sieht aber leer aus |

## Schritt 6 – Bilder für den Launcher
Lege sie unter `tools/dasdimo/art/` ab (sie gehören ins Git deines Spiels):

| Datei | Format | Größe | Höchstens | Wofür |
|---|---|---|---|---|
| `icon.png` | PNG | **genau 256×256** | 300 KB | Spieleliste links (wird bis 32 px verkleinert): ein klares Motiv, kein kleiner Text, Hintergrund darf transparent sein |
| `cover.jpg` | JPEG (Qualität ~85) | **genau 1920×1080** | 800 KB | großes Bild der Spielseite. **Kein Text im Bild** (Name/Logo zeichnet der Launcher). Unten links liegen Titel und Knopf, rechts die Neuigkeiten – das Hauptmotiv in die Mitte/oben |
| `logo.png` (optional) | PNG, transparent | höchstens 1200×400 | 300 KB | Schriftzug statt Textnamen |
| `shot1.jpg` … `shot8.jpg` | JPEG (Qualität ~85) | **genau 1920×1080** | je 800 KB | Screenshot-Reihe der Spielseite (klein, ein Klick öffnet die Großansicht mit Pfeilen). 3–5 starke Momente aus echtem Spielen, verschiedene Orte/Situationen, kein Text, keine Menüs |

Herkunft: echte Szenen aus dem Spiel (Screenshot-Funktion der Engine in voller Auflösung, gute Kamera) und das vorhandene Spiel-Icon. Keine Klötzchen-Platzhalter. **Zeig die Bilder dem Benutzer** (Datei schicken) und warte auf sein OK. Externe Bilder nur mit passender Lizenz und Eintrag in den Credits deines Spiels.

## Schritt 7 – `manifest.json` (schreibt das Skript – nur zum Verstehen)
Das Skript schreibt pro Release diese Datei. Der Launcher prüft sie streng; eine ungültige Datei führt nie zu einem Download.

```json
{
	"format": 1,
	"game": "Beispielspiel",
	"game_id": "beispielspiel",
	"repo": "dasdimo/beispielspiel-releases",
	"version": "0.1.0",
	"tag": "v0.1.0",
	"date": "2026-10-28",
	"launcher_version": "",
	"files": [
		{
			"kind": "game",
			"arch": "x86_64",
			"name": "Beispielspiel-v0.1.0.exe",
			"size": 98765432,
			"sha256": "64 Hex-Zeichen …",
			"url": "https://github.com/dasdimo/beispielspiel-releases/releases/download/v0.1.0/Beispielspiel-v0.1.0.exe"
		}
	],
	"news": [
		{"version": "0.1.0", "date": "2026-10-28", "entries": [{"de": "Erste Version zum Testen.", "en": "First version for testing."}]}
	]
}
```

| Feld | Regel |
|---|---|
| `format` | `1` = einzelne exe-Dateien. `2` = mindestens eine `.zip` (dann pro Zip-Datei `"exe": "<Startdatei im Zip>"`). Der Launcher lehnt höhere Formate ab, als er kennt |
| `version` | `MAJOR.MINOR.PATCH`, optional `-zusatz` aus `0-9 A-Z a-z .` (z. B. `0.2.1-test1`); ohne führendes `v`. Eine Version mit Zusatz ist **älter** als ohne: `0.2.1-test1` < `0.2.1` < `0.2.2` |
| `tag` | `v` + Version (Git-Tag des Releases) |
| `files[].kind` | immer `game` (`launcher` und `setup` gibt es nur in der Zentrale, `launcher` außerdem im RIFTWAKE-Repo für den Wechsel vom alten RIFTWAKE-Launcher) |
| `files[].arch` | `x86_64` oder `arm64`. Fehlt `arm64`, nimmt ein Snapdragon-PC die x64-Datei (läuft emuliert) |
| `files[].name` | Dateiname ohne Pfad: beginnt mit Buchstabe/Ziffer, dann nur `A–Z a–z 0–9 . _ -`, endet auf `.exe` (Format 1) oder `.zip` (Format 2) |
| `files[].size`, `sha256` | Bytes und SHA-256 (64 Hex-Zeichen) – der Launcher prüft beides nach dem Download |
| `files[].url` | voller `https://`-Link zur Datei im Release |
| `news` | die letzten 3 Versionen, neueste zuerst, jeder Eintrag DE **und** EN |
| `launcher_version` | bei Spielen immer `""` |

## Schritt 8 – Neuigkeiten und README
**`tools/dasdimo/news.json`** – alle Versionen, neueste zuerst. Das Skript nimmt die letzten 3 (Versionen mit `-`, z. B. `0.3.0-dev2`, werden ausgelassen, außer es ist die gerade veröffentlichte). Mehrere Blöcke mit gleicher Version werden zusammengefasst.
```json
[
	{"version": "0.2.0", "date": "2026-11-15", "entries": [
		{"de": "Neue Figur: Die Wirtin.", "en": "New character: The innkeeper."},
		{"de": "Runden gehen schneller.", "en": "Rounds are faster."}
	]},
	{"version": "0.1.0", "date": "2026-10-28", "entries": [{"de": "Erste Version zum Testen.", "en": "First version for testing."}]}
]
```
Schreib die Einträge für Freunde, nicht für Programmierer (kurz, was man im Spiel merkt). Hat dein Projekt schon ein Änderungsprotokoll, übernimm daraus – oder lass dir vom Benutzer sagen, was drinstehen soll.

**`tools/dasdimo/README.md`** – Startseite des Release-Repos, Deutsch oben, Englisch darunter: was das Spiel ist (2 Sätze), wie man es bekommt (**nur** über den DasDimo Launcher: Link https://github.com/dasdimo/dasdimo-launcher-releases, dort den Installer `DasDimoLauncher-Setup.exe` laden – keine direkten Links auf Spiel-Dateien), SmartScreen-Hinweis für den Installer („Weitere Informationen“ → „Trotzdem ausführen“, die Datei ist nicht signiert), wo Einstellungen liegen, an wen man Fehler schickt. Der Quellcode ist nicht öffentlich – das kurz erwähnen.

## Schritt 9 – Hilfs-Skript anlegen und prüfen
1. Kopiere das Skript aus dem **Anhang** unverändert nach `tools/dasdimo/dasdimo_release.js`.
2. Baue die Builds (x64 + ARM64) mit deinem gewohnten Export.
3. Prüfen:
   ```powershell
   node tools/dasdimo/dasdimo_release.js check
   ```
   Muss mit `CHECK OK` enden. Jede `PROBLEM`-Zeile sagt, was fehlt (Größe der Bilder, Version, Dateien, Neuigkeiten …).
4. Probelauf ohne Upload:
   ```powershell
   node tools/dasdimo/dasdimo_release.js manifest
   ```
   Schreibt `builds/dasdimo-release/v<version>/manifest.json` + `notes.md`. Kurz ansehen.
5. Leg im eigenen Projekt ein kleines Release-Skript oder einen Abschnitt in der Doku an („Neue Version veröffentlichen“ – Schritt 13), damit spätere Sitzungen den Ablauf finden.

## Schritt 10 – Erstes Release (erst nach Ja des Benutzers)
Sag dem Benutzer, was hochgeladen wird (Dateien, Größen, Repo) und frag. Dann:
```powershell
node tools/dasdimo/dasdimo_release.js release --readme
```
Das Skript prüft das Konto, lädt die README (noreply), erstellt das Release `v<version>` mit allen Builds und **zuletzt** `manifest.json`, markiert es als „latest“ und vergleicht danach jede Datei per SHA-256 mit GitHub. Am Ende steht `RELEASE OK` mit Link. Bei einem schon vorhandenen Release mit gleicher Version lädt es nur geänderte Dateien neu.

## Schritt 11 – Mit dem Launcher testen
Du testest mit dem **DasDimo Launcher** und einer lokalen Spieleliste, die nur dein Spiel enthält – so siehst du die Spielseite genau wie später, bevor etwas in der Zentrale steht.

**Launcher finden:** Hat der Benutzer ihn installiert, liegt er unter `%LOCALAPPDATA%\DasDimo-Launcher\Launcher\DasDimoLauncher.exe`. Fehlt er, bitte den Benutzer, ihn über den Installer von https://github.com/dasdimo/dasdimo-launcher-releases zu installieren (ein Klick, keine Admin-Rechte).

1. Lokale Spieleliste schreiben (lädt nichts hoch):
   ```powershell
   node tools/dasdimo/dasdimo_release.js register --dry-run
   ```
   Am Ende steht der Pfad von `builds/dasdimo-release/games.local.json` (dein Eintrag mit den Bildern von deiner Festplatte und dem echten Manifest aus Schritt 10).
2. Automatischer Test (installiert in einen Testordner und beendet sich):
   ```powershell
   $launcher = "$env:LOCALAPPDATA\DasDimo-Launcher\Launcher\DasDimoLauncher.exe"
   $test = "$env:TEMP\dasdimo-launcher-test\<game_id>"
   $list = (Resolve-Path "builds\dasdimo-release\games.local.json").Path
   & $launcher -- "--catalog-url=$list" "--root-dir=$test" --portable --game=<game_id> --auto-update --quit-when-done | Out-Null
   Get-Content "$test\Launcher\launcher.log"; Get-Content "$test\Games\<game_id>\installed.json"
   ```
   Erwartet: im Log `games.json: 1 game(s)` und `installed <version>`; in `installed.json` `"current": "<version>"`. Mit `--arch=arm64` prüfst du die ARM64-Auswahl. `--root-dir` hält alles im Testordner (die echte Installation des Benutzers bleibt unberührt), `--portable` verhindert die Frage „Launcher installieren?“.
3. Test mit dem Benutzer: denselben Befehl **ohne** `--auto-update --quit-when-done` starten. Der Launcher zeigt nur dein Spiel – mit Icon, Cover, Screenshots und Neuigkeiten. Bitte den Benutzer, sich die Seite anzusehen, auf **Spielen** zu klicken, und frag, ob alles passt und das Spiel startet.
4. Danach den Testordner löschen (`$env:TEMP\dasdimo-launcher-test\<game_id>`).
5. Meldet der Launcher einen Fehler: der Grund steht in `$test\Launcher\launcher.log` (z. B. `files[0].name invalid` oder `HTTP 404`, wenn das Release fehlt).

Mit `status: hidden` ist das Spiel nach Schritt 12 nur sichtbar, wenn der Launcher mit `-- --show-hidden` gestartet wird.

## Schritt 12 – In die Spieleliste eintragen (erst nach Ja des Benutzers)
```powershell
node tools/dasdimo/dasdimo_release.js register --dry-run
node tools/dasdimo/dasdimo_release.js register
```
Der Probelauf zeigt den Eintrag. Der echte Lauf lädt `icon.png`/`cover.jpg`/`logo.png` und die Screenshots (`shot1.jpg` …) nach `dasdimo/<game_id>-releases` → Ordner `launcher/` (nur wenn geändert) und trägt dein Spiel in `games.json` der Zentrale ein bzw. aktualisiert den Eintrag (Reihenfolge und Datum der Aufnahme bleiben). Voraussetzung: mindestens ein Release (Schritt 10). Meldet GitHub einen Konflikt (jemand hat gleichzeitig eingetragen), einfach noch einmal ausführen.

## Schritt 13 – Festhalten und Bericht
1. Im eigenen Projekt dokumentieren (z. B. `CLAUDE.md`/Übergabe-Datei): Release-Repo, `tools/dasdimo/`, Ablauf „Neue Version veröffentlichen“:
   1. Version im Projekt erhöhen, `news.json` ergänzen (DE+EN).
   2. Builds x64 + ARM64 bauen, Tests deines Spiels grün.
   3. `node tools/dasdimo/dasdimo_release.js check`, dann `release` (nur wenn der Benutzer veröffentlichen will).
   4. Test-Release zum Ausprobieren: `release --suffix test1` → Version `<version>-test1` wird „latest“. Danach mit OK des Benutzers löschen: `gh release delete v<version>-test1 -R dasdimo/<game_id>-releases --cleanup-tag --yes` – dann ist wieder das vorige Release aktuell.
   5. Neue Bilder oder Screenshots, neuer Untertitel oder anderer Status: `tools/dasdimo/dasdimo.json` ändern, dann `register`.
2. Committe `tools/dasdimo/` (Skript, Konfiguration, Neuigkeiten, README, Bilder) im Repo deines Spiels, mit verständlicher deutscher Commit-Nachricht.
3. Bericht an den Benutzer auf Deutsch: Link zum Release-Repo, Ergebnis des Launcher-Tests, was noch offen ist (z. B. Zip-Spiel wartet auf L4), und eine kurze Nachricht für die Freunde („Neues Spiel im DasDimo Launcher: einfach den Launcher öffnen“).

## Fehlerhilfe
| Meldung | Lösung |
|---|---|
| `gh is logged in as '…'` | `gh auth status`; mit dem Konto `dasdimo` anmelden (Benutzer fragen) |
| `… does not exist yet` | Schritt 3 |
| `build missing: …` | Build fehlt oder Dateiname passt nicht zu `files[].path` |
| `news: no entry for <version>` | `news.json` für die neue Version ergänzen |
| `catalog.cover … must be 1920x1080` | Bild in genau dieser Größe neu speichern (gilt auch für `catalog.screenshots[…]`) |
| `… has no release yet` | erst Schritt 10, dann `register` |
| Launcher: „Manifest ungültig“ | `launcher.log` lesen; meist Dateiname mit Leerzeichen/Umlaut oder Version mit `v` |

---

## Anhang – `tools/dasdimo/dasdimo_release.js`
Unverändert kopieren. Bei einer neuen Version dieser Anleitung das Skript aus der neuen Anleitung übernehmen (die Version steht in der ersten Zeile).

```js
// DasDimo Launcher - release helper for one game (helper version 2, from INTEGRATION.md).
// Reads tools/dasdimo/dasdimo.json (or --config <file>). Needs Node 18+ and the GitHub CLI (account dasdimo).
//   node tools/dasdimo/dasdimo_release.js check                      config, builds, news, pictures
//   node tools/dasdimo/dasdimo_release.js manifest [--suffix test1]  only write manifest.json + notes.md
//   node tools/dasdimo/dasdimo_release.js release [--suffix test1] [--readme]   publish a GitHub release
//   node tools/dasdimo/dasdimo_release.js register [--dry-run]       pictures + entry in the launcher's games.json
//     (--dry-run also writes <out_dir>/games.local.json: the entry with local pictures, for a test in the launcher)
// Public commits (README, pictures, games.json) use the GitHub noreply address, never the local git e-mail.
"use strict";
const fs = require("fs");
const os = require("os");
const path = require("path");
const crypto = require("crypto");
const { spawnSync } = require("child_process");

const HELPER_VERSION = 2;
const OWNER = "dasdimo";
const HUB_REPO = "dasdimo/dasdimo-launcher-releases";
const CATALOG_FILE = "games.json";
const NEWS_VERSIONS = 3;
const ARCHES = ["x86_64", "arm64"];
const STATUSES = ["testing", "released", "hidden"];
const VERSION_RE = /^[0-9]+\.[0-9]+\.[0-9]+(-[0-9A-Za-z.]+)?$/;
const NAME_RE = /^[0-9A-Za-z][0-9A-Za-z._-]*\.(exe|zip)$/;
const ID_RE = /^[a-z0-9][a-z0-9-]{1,31}$/;
const REPO_RE = /^dasdimo\/[A-Za-z0-9._-]+-releases$/;
const DATA_DIR_RE = /^%(APPDATA|LOCALAPPDATA)%\\.+|^%USERPROFILE%\\AppData\\LocalLow\\.+/;
const PICTURES = {
	icon: { type: "png", width: 256, height: 256, maxKb: 300, required: true },
	cover: { type: "jpg", width: 1920, height: 1080, maxKb: 800, required: true },
	logo: { type: "png", maxWidth: 1200, maxHeight: 400, maxKb: 300, required: false },
};
// Pictures of the demo page (catalog.screenshots), shown in this order.
const SCREENSHOT = { type: "jpg", width: 1920, height: 1080, maxKb: 800 };
const MAX_SCREENSHOTS = 8;
const LAUNCHER_PAGE = `https://github.com/${HUB_REPO}`;

const ROOT = path.resolve(__dirname, "..", "..");

function fail(message) {
	console.error("STOPPED: " + message);
	process.exit(1);
}

function parseArgs(argv) {
	const args = { command: argv[0] || "help", suffix: "", readme: false, dryRun: false, config: "tools/dasdimo/dasdimo.json" };
	for (let i = 1; i < argv.length; i++) {
		const key = argv[i];
		if (key === "--readme") args.readme = true;
		else if (key === "--dry-run") args.dryRun = true;
		else if (key === "--suffix") args.suffix = argv[++i] || "";
		else if (key === "--config") args.config = argv[++i] || "";
		else fail("unknown option " + key);
	}
	return args;
}

function rel(file) {
	return path.resolve(ROOT, file);
}

function today() {
	return new Date().toISOString().slice(0, 10);
}

function megabytes(bytes) {
	return (bytes / (1024 * 1024)).toFixed(1);
}

function sha256File(file) {
	const hash = crypto.createHash("sha256");
	const fd = fs.openSync(file, "r");
	const buffer = Buffer.alloc(1 << 20);
	let read;
	while ((read = fs.readSync(fd, buffer, 0, buffer.length, null)) > 0) hash.update(buffer.subarray(0, read));
	fs.closeSync(fd);
	return hash.digest("hex");
}

// ---------- config ----------

function loadConfig(file) {
	const full = rel(file);
	if (!fs.existsSync(full)) fail("missing " + file + " (INTEGRATION.md step 5)");
	let cfg;
	try {
		cfg = JSON.parse(fs.readFileSync(full, "utf8"));
	} catch (error) {
		fail(file + " is not valid JSON: " + error.message);
	}
	cfg.news_file = cfg.news_file || "tools/dasdimo/news.json";
	cfg.readme_file = cfg.readme_file || "tools/dasdimo/README.md";
	cfg.out_dir = cfg.out_dir || "builds/dasdimo-release";
	cfg.catalog = cfg.catalog || {};
	return cfg;
}

function configProblems(cfg) {
	const problems = [];
	if (!ID_RE.test(cfg.game_id || "")) problems.push("game_id: lowercase letters, digits, '-' (2-32 chars)");
	if (typeof cfg.game_name !== "string" || !cfg.game_name.trim()) problems.push("game_name missing");
	if (!REPO_RE.test(cfg.repo || "")) problems.push("repo must look like dasdimo/<name>-releases");
	if (!Array.isArray(cfg.files) || !cfg.files.length) problems.push("files missing");
	else {
		for (const [i, file] of cfg.files.entries()) {
			if (!ARCHES.includes(file.arch)) problems.push(`files[${i}].arch must be x86_64 or arm64`);
			if (typeof file.path !== "string" || !file.path) problems.push(`files[${i}].path missing`);
			else if (file.path.endsWith(".zip") && (!/^[^\\/:.][^:]*\.exe$/.test(file.exe || "") || /(^|[\\/])\.\.([\\/]|$)/.test(file.exe))) {
				problems.push(`files[${i}].exe: start file inside the zip, relative path like "Game.exe" or "bin/Game.exe"`);
			}
		}
		if (!cfg.files.some((file) => file.arch === "x86_64")) problems.push("an x86_64 file is required");
		if (new Set(cfg.files.map((file) => file.arch)).size !== cfg.files.length) problems.push("one file per arch");
	}
	try {
		const version = readVersion(cfg);
		if (!VERSION_RE.test(version)) problems.push(`version '${version}' is not like 1.2.3 or 1.2.3-test1`);
	} catch (error) {
		problems.push(error.message);
	}
	return problems;
}

function catalogProblems(cfg) {
	const problems = [];
	const c = cfg.catalog;
	for (const lang of ["de", "en"]) {
		const text = c.tagline && c.tagline[lang];
		if (typeof text !== "string" || !text.trim() || text.length > 90) problems.push(`catalog.tagline.${lang}: 1-90 characters`);
	}
	if (!/^#[0-9A-Fa-f]{6}$/.test(c.accent || "")) problems.push("catalog.accent like #FF7A1A");
	if (typeof c.engine !== "string" || !c.engine.trim()) problems.push("catalog.engine missing (e.g. \"Godot 4.5\")");
	if (!STATUSES.includes(c.status)) problems.push("catalog.status: testing, released or hidden");
	if (!Array.isArray(c.user_data_dirs) || c.user_data_dirs.some((dir) => !DATA_DIR_RE.test(dir))) {
		problems.push("catalog.user_data_dirs: list of folders below %APPDATA%\\, %LOCALAPPDATA%\\ or %USERPROFILE%\\AppData\\LocalLow\\ (may be empty)");
	}
	for (const [key, rule] of Object.entries(PICTURES)) {
		const file = c[key];
		if (!file) {
			if (rule.required) problems.push(`catalog.${key} missing`);
			continue;
		}
		const problem = pictureProblem(rel(file), rule);
		if (problem) problems.push(`catalog.${key} (${file}): ${problem}`);
	}
	const shots = c.screenshots === undefined ? [] : c.screenshots;
	if (!Array.isArray(shots) || shots.length > MAX_SCREENSHOTS || shots.some((file) => typeof file !== "string" || !file)) {
		problems.push(`catalog.screenshots: list of up to ${MAX_SCREENSHOTS} JPEG files (may be empty)`);
	} else {
		for (const [i, file] of shots.entries()) {
			const problem = pictureProblem(rel(file), SCREENSHOT);
			if (problem) problems.push(`catalog.screenshots[${i}] (${file}): ${problem}`);
		}
	}
	return problems;
}

function readVersion(cfg) {
	if (cfg.version) return String(cfg.version).trim();
	if (!cfg.version_file || !cfg.version_pattern) throw new Error("set version, or version_file + version_pattern");
	const file = rel(cfg.version_file);
	if (!fs.existsSync(file)) throw new Error("version_file not found: " + cfg.version_file);
	const found = new RegExp(cfg.version_pattern, "m").exec(fs.readFileSync(file, "utf8"));
	if (!found || !found[1]) throw new Error("version_pattern finds nothing in " + cfg.version_file);
	return found[1].trim();
}

function resolveFiles(cfg, baseVersion) {
	return cfg.files.map((file) => {
		const full = rel(file.path.replace(/\{version\}/g, baseVersion));
		if (!fs.existsSync(full)) fail("build missing: " + path.relative(ROOT, full) + " - build the game first");
		const name = path.basename(full);
		if (!NAME_RE.test(name)) fail(`file name '${name}': only letters, digits, . _ - and ending .exe or .zip`);
		return { arch: file.arch, path: full, name, exe: file.exe || "" };
	});
}

// ---------- pictures ----------

function pictureSize(buf) {
	if (buf.length > 24 && buf.readUInt32BE(0) === 0x89504e47) {
		return { type: "png", width: buf.readUInt32BE(16), height: buf.readUInt32BE(20) };
	}
	if (buf.length > 4 && buf[0] === 0xff && buf[1] === 0xd8) {
		let i = 2;
		while (i + 9 < buf.length) {
			if (buf[i] !== 0xff) { i++; continue; }
			const marker = buf[i + 1];
			if (marker === 0xff) { i++; continue; }
			if (marker === 0x01 || (marker >= 0xd0 && marker <= 0xd8)) { i += 2; continue; }
			if (marker >= 0xc0 && marker <= 0xcf && ![0xc4, 0xc8, 0xcc].includes(marker)) {
				return { type: "jpg", width: buf.readUInt16BE(i + 7), height: buf.readUInt16BE(i + 5) };
			}
			i += 2 + buf.readUInt16BE(i + 2);
		}
	}
	return null;
}

function pictureProblem(file, rule) {
	if (!fs.existsSync(file)) return "file not found";
	const buf = fs.readFileSync(file);
	const size = pictureSize(buf);
	if (!size || size.type !== rule.type) return `must be a ${rule.type.toUpperCase()} file`;
	if (rule.width && (size.width !== rule.width || size.height !== rule.height)) {
		return `must be ${rule.width}x${rule.height}, is ${size.width}x${size.height}`;
	}
	if (rule.maxWidth && (size.width > rule.maxWidth || size.height > rule.maxHeight)) {
		return `at most ${rule.maxWidth}x${rule.maxHeight}, is ${size.width}x${size.height}`;
	}
	if (buf.length > rule.maxKb * 1024) return `at most ${rule.maxKb} KB, is ${Math.ceil(buf.length / 1024)} KB`;
	return "";
}

// ---------- news ----------

// news.json: newest first, [{version, date, entries: [{de, en}]}]. Several blocks may share a version.
// Dev versions ("0.3.0-dev2") are left out unless they are the released version itself.
function readNews(cfg, version) {
	const file = rel(cfg.news_file);
	if (!fs.existsSync(file)) return { news: [], problems: ["news file missing: " + cfg.news_file] };
	let list;
	try {
		list = JSON.parse(fs.readFileSync(file, "utf8"));
	} catch (error) {
		return { news: [], problems: [cfg.news_file + " is not valid JSON"] };
	}
	if (!Array.isArray(list)) return { news: [], problems: [cfg.news_file + " must be a list"] };
	const problems = [];
	const groups = [];
	for (const block of list) {
		if (!block || !VERSION_RE.test(block.version || "") || !Array.isArray(block.entries)) {
			problems.push("news block without valid version/entries");
			continue;
		}
		if (block.version.includes("-") && block.version !== version) continue;
		let group = groups.find((g) => g.version === block.version);
		if (!group) {
			if (groups.length >= NEWS_VERSIONS) continue;
			group = { version: block.version, date: block.date || "", entries: [] };
			groups.push(group);
		}
		if ((block.date || "") > group.date) group.date = block.date;
		for (const entry of block.entries) {
			if (!entry || typeof entry.de !== "string" || typeof entry.en !== "string" || !entry.de.trim() || !entry.en.trim()) {
				problems.push(`news ${block.version}: every entry needs de + en`);
				continue;
			}
			group.entries.push({ de: entry.de.trim(), en: entry.en.trim() });
		}
	}
	return { news: groups, problems };
}

// ---------- manifest ----------

function buildRelease(cfg, suffix) {
	const baseVersion = readVersion(cfg);
	const version = suffix ? `${baseVersion}-${suffix}` : baseVersion;
	if (!VERSION_RE.test(version)) fail(`invalid version '${version}'`);
	const tag = "v" + version;
	const files = resolveFiles(cfg, baseVersion).map((file) => {
		const entry = {
			kind: "game",
			arch: file.arch,
			name: file.name,
			size: fs.statSync(file.path).size,
			sha256: sha256File(file.path),
			url: `https://github.com/${cfg.repo}/releases/download/${tag}/${file.name}`,
		};
		if (file.exe) entry.exe = file.exe;
		return Object.assign(entry, { path: file.path });
	});
	const news = readNews(cfg, version).news;
	const manifest = {
		format: files.some((file) => file.name.endsWith(".zip")) ? 2 : 1,
		game: cfg.game_name,
		game_id: cfg.game_id,
		repo: cfg.repo,
		version,
		tag,
		date: today(),
		launcher_version: "",
		files: files.map(({ path: _path, ...entry }) => entry),
		news,
	};
	const out = rel(path.join(cfg.out_dir, tag));
	fs.mkdirSync(out, { recursive: true });
	const manifestPath = path.join(out, "manifest.json");
	const notesPath = path.join(out, "notes.md");
	fs.writeFileSync(manifestPath, JSON.stringify(manifest, null, "\t") + "\n", "utf8");
	fs.writeFileSync(notesPath, releaseNotes(cfg, manifest, Boolean(suffix)), "utf8");
	for (const file of manifest.files) {
		console.log(`${file.arch.padEnd(6)} ${megabytes(file.size).padStart(7)} MB  ${file.sha256}  ${file.name}`);
	}
	console.log("news: " + (news.map((g) => `${g.version} (${g.entries.length})`).join(", ") || "none"));
	console.log("written: " + path.relative(ROOT, manifestPath));
	return { version, tag, manifest, manifestPath, notesPath, assets: files.map((file) => file.path) };
}

// Games come only through the launcher: the notes link the launcher, not the game files.
function releaseNotes(cfg, manifest, test) {
	const base = manifest.version.split("-")[0];
	const news = manifest.news.find((group) => group.version === manifest.version)
		|| manifest.news.find((group) => group.version === base) || { entries: [] };
	const lines = [];
	if (test) lines.push("> **Test-Release / test release** – nur zum Testen. Only for testing.", "");
	for (const lang of ["de", "en"]) {
		const de = lang === "de";
		lines.push(`## ${cfg.game_name} ${manifest.version}`, "");
		lines.push(de
			? `**So bekommst du es:** im [DasDimo Launcher](${LAUNCHER_PAGE}) – dort installieren, aktualisieren und spielen.`
			: `**How to get it:** in the [DasDimo Launcher](${LAUNCHER_PAGE}) – install, update and play there.`, "");
		if (news.entries.length) {
			lines.push(de ? "### Neu" : "### New", "");
			for (const entry of news.entries) lines.push("- " + entry[lang]);
			lines.push("");
		}
		lines.push("---", "");
	}
	lines.push("| Datei / file | MB | SHA-256 |", "|---|---|---|");
	for (const file of manifest.files) lines.push(`| ${file.name} | ${megabytes(file.size)} | \`${file.sha256}\` |`);
	lines.push("");
	return lines.join("\n");
}

// ---------- GitHub ----------

function ghBin() {
	if (process.env.GH_BIN) return process.env.GH_BIN;
	const windows = "C:\\Program Files\\GitHub CLI\\gh.exe";
	return fs.existsSync(windows) ? windows : "gh";
}

function gh(args, { allowFail = false, inherit = false } = {}) {
	const result = spawnSync(ghBin(), args, { encoding: "utf8", stdio: inherit ? "inherit" : "pipe", maxBuffer: 64 << 20 });
	if (result.error) fail("GitHub CLI not found: " + result.error.message);
	const ok = result.status === 0;
	if (!ok && !allowFail) fail(`gh ${args.slice(0, 3).join(" ")} failed: ${(result.stderr || result.stdout || "").trim()}`);
	return { ok, out: result.stdout || "", err: (result.stderr || "") + (result.stdout || "") };
}

function identity() {
	const user = JSON.parse(gh(["api", "user"]).out);
	if (String(user.login).toLowerCase() !== OWNER) fail(`gh is logged in as '${user.login}', expected '${OWNER}' (gh auth status)`);
	return { name: user.login, email: `${user.id}+${user.login}@users.noreply.github.com` };
}

function getFile(repo, filePath) {
	const result = gh(["api", `repos/${repo}/contents/${filePath}`], { allowFail: true });
	if (!result.ok) {
		if (/Not Found|repository is empty|HTTP 404/i.test(result.err)) return null;
		fail(`reading ${repo}/${filePath} failed: ${result.err.trim()}`);
	}
	const data = JSON.parse(result.out);
	return { sha: data.sha, content: Buffer.from(data.content || "", "base64") };
}

function gitBlobSha(bytes) {
	return crypto.createHash("sha1").update(`blob ${bytes.length}\0`).update(bytes).digest("hex");
}

// Creates or updates one file in a public repo through the API, as the noreply identity.
function putFile(repo, filePath, bytes, message, who) {
	const existing = getFile(repo, filePath);
	if (existing && existing.sha === gitBlobSha(bytes)) {
		console.log(`unchanged: ${repo}/${filePath}`);
		return;
	}
	const body = { message, content: bytes.toString("base64"), author: who, committer: who };
	if (existing) body.sha = existing.sha;
	const tmp = path.join(os.tmpdir(), `dasdimo_put_${process.pid}_${Date.now()}.json`);
	fs.writeFileSync(tmp, JSON.stringify(body), "utf8");
	try {
		gh(["api", "-X", "PUT", `repos/${repo}/contents/${filePath}`, "--input", tmp]);
	} finally {
		fs.rmSync(tmp, { force: true });
	}
	console.log(`uploaded: ${repo}/${filePath}`);
}

function releaseAssets(repo, tag) {
	const result = gh(["release", "view", tag, "-R", repo, "--json", "assets"], { allowFail: true });
	return result.ok ? JSON.parse(result.out).assets || [] : null;
}

function sameOnGitHub(assets, file) {
	const asset = assets.find((a) => a.name === path.basename(file));
	return Boolean(asset) && String(asset.digest || "") === "sha256:" + sha256File(file);
}

// ---------- commands ----------

function commandCheck(cfg) {
	const problems = configProblems(cfg);
	if (!problems.length) {
		const baseVersion = readVersion(cfg);
		console.log(`${cfg.game_name} ${baseVersion} -> ${cfg.repo}`);
		for (const file of cfg.files) {
			const full = rel(file.path.replace(/\{version\}/g, baseVersion));
			console.log(`${fs.existsSync(full) ? "ok     " : "MISSING"} ${file.arch.padEnd(6)} ${path.relative(ROOT, full)}`);
		}
		const news = readNews(cfg, baseVersion);
		problems.push(...news.problems);
		if (!news.news.some((group) => group.version === baseVersion)) problems.push(`news: no entry for ${baseVersion} yet (${cfg.news_file})`);
	}
	if (!fs.existsSync(rel(cfg.readme_file))) problems.push("README missing: " + cfg.readme_file);
	problems.push(...catalogProblems(cfg));
	for (const problem of problems) console.log("PROBLEM " + problem);
	console.log(problems.length ? `${problems.length} problem(s)` : "CHECK OK");
	process.exit(problems.length ? 1 : 0);
}

function commandRelease(cfg, args) {
	const problems = configProblems(cfg);
	if (problems.length) fail(problems.join("; "));
	const who = identity();
	const repoInfo = gh(["repo", "view", cfg.repo, "--json", "visibility"], { allowFail: true });
	if (!repoInfo.ok) fail(`${cfg.repo} does not exist yet (INTEGRATION.md step 3)`);
	if (JSON.parse(repoInfo.out).visibility !== "PUBLIC") fail(`${cfg.repo} must be public`);
	const release = buildRelease(cfg, args.suffix);
	// A release needs at least one commit in the repo: the README is the first one.
	if (args.readme || !getFile(cfg.repo, "README.md")) {
		const readme = rel(cfg.readme_file);
		if (!fs.existsSync(readme)) fail("README missing: " + cfg.readme_file);
		putFile(cfg.repo, "README.md", fs.readFileSync(readme), "Download-Anleitung aktualisiert", who);
	}
	const title = `${cfg.game_name} ${release.version}` + (args.suffix ? " (Test)" : "");
	const existing = releaseAssets(cfg.repo, release.tag);
	if (existing) {
		console.log(`== update release ${release.tag} ==`);
		const changed = release.assets.filter((file) => !sameOnGitHub(existing, file));
		for (const file of release.assets) if (!changed.includes(file)) console.log("unchanged: " + path.basename(file));
		// Builds first, manifest last: the launcher never sees a manifest whose files are missing.
		if (changed.length) gh(["release", "upload", release.tag, ...changed, "-R", cfg.repo, "--clobber"], { inherit: true });
		gh(["release", "upload", release.tag, release.manifestPath, "-R", cfg.repo, "--clobber"], { inherit: true });
		gh(["release", "edit", release.tag, "-R", cfg.repo, "--title", title, "--notes-file", release.notesPath, "--latest"]);
	} else {
		console.log(`== create release ${release.tag} ==`);
		gh(["release", "create", release.tag, ...release.assets, release.manifestPath, "-R", cfg.repo,
			"--title", title, "--notes-file", release.notesPath, "--latest"], { inherit: true });
	}
	const remote = releaseAssets(cfg.repo, release.tag) || [];
	const missing = [...release.assets, release.manifestPath].filter((file) => !sameOnGitHub(remote, file));
	if (missing.length) fail("missing or different on GitHub: " + missing.map((file) => path.basename(file)).join(", "));
	console.log(`RELEASE OK: https://github.com/${cfg.repo}/releases/tag/${release.tag}`);
	console.log(`manifest:   https://github.com/${cfg.repo}/releases/latest/download/manifest.json`);
}

function commandRegister(cfg, args) {
	const problems = [...configProblems(cfg), ...catalogProblems(cfg)];
	if (problems.length) fail(problems.join("; "));
	const who = identity();
	const latest = gh(["release", "view", "-R", cfg.repo, "--json", "tagName"], { allowFail: true });
	if (!latest.ok) {
		if (!args.dryRun) fail(`${cfg.repo} has no release yet - run "release" first`);
		console.log(`note: ${cfg.repo} has no release yet`);
	}
	const branchInfo = gh(["api", `repos/${cfg.repo}`, "--jq", ".default_branch"], { allowFail: true });
	const branch = branchInfo.ok && branchInfo.out.trim() ? branchInfo.out.trim() : "main";
	const c = cfg.catalog;
	const entry = {
		id: cfg.game_id,
		type: "game",
		name: cfg.game_name,
		repo: cfg.repo,
		manifest_url: `https://github.com/${cfg.repo}/releases/latest/download/manifest.json`,
		platforms: cfg.files.map((file) => file.arch),
		engine: c.engine.trim(),
		status: c.status,
		tagline: { de: c.tagline.de.trim(), en: c.tagline.en.trim() },
		accent: c.accent.toUpperCase(),
	};
	// The same entry with local pictures: a local games.json may point to local files (test in the launcher).
	const local = { ...entry };
	const upload = (file, remotePath, message) => {
		const bytes = fs.readFileSync(rel(file));
		if (args.dryRun) console.log(`would upload ${file} -> ${cfg.repo}/${remotePath}`);
		else putFile(cfg.repo, remotePath, bytes, message, who);
		const sha256 = crypto.createHash("sha256").update(bytes).digest("hex");
		return { url: `https://raw.githubusercontent.com/${cfg.repo}/${branch}/${remotePath}`, local: rel(file).replace(/\\/g, "/"), sha256 };
	};
	for (const key of Object.keys(PICTURES)) {
		if (!c[key]) continue;
		const picture = upload(c[key], `launcher/${key}.${PICTURES[key].type}`, "Launcher-Bild: " + key);
		entry[`${key}_url`] = picture.url;
		local[`${key}_url`] = picture.local;
		entry[`${key}_sha256`] = local[`${key}_sha256`] = picture.sha256;
	}
	const shots = (c.screenshots || []).map((file, i) => upload(file, `launcher/shot${i + 1}.jpg`, `Launcher-Screenshot ${i + 1}`));
	if (shots.length) {
		entry.screenshots = shots.map((shot) => ({ url: shot.url, sha256: shot.sha256 }));
		local.screenshots = shots.map((shot) => ({ url: shot.local, sha256: shot.sha256 }));
	}
	entry.user_data_dirs = local.user_data_dirs = c.user_data_dirs;
	const hubFile = getFile(HUB_REPO, CATALOG_FILE);
	if (!hubFile && !args.dryRun) fail(`${HUB_REPO}/${CATALOG_FILE} not found`);
	const catalog = hubFile ? JSON.parse(hubFile.content.toString("utf8")) : { format: 1, games: [] };
	if (!Array.isArray(catalog.games)) fail(`${CATALOG_FILE} has no games list`);
	const index = catalog.games.findIndex((game) => game.id === entry.id);
	const old = index >= 0 ? catalog.games[index] : null;
	entry.order = old && Number.isInteger(old.order) ? old.order : 10 * (catalog.games.length + 1);
	entry.added = old && old.added ? old.added : today();
	entry.updated = today();
	if (old) catalog.games[index] = entry;
	else catalog.games.push(entry);
	catalog.games.sort((a, b) => (a.order || 0) - (b.order || 0));
	catalog.updated = today();
	const text = JSON.stringify(catalog, null, "\t") + "\n";
	if (args.dryRun) {
		console.log(JSON.stringify(entry, null, "\t"));
		console.log(`DRY RUN: would ${old ? "update" : "add"} '${entry.id}' in ${HUB_REPO}/${CATALOG_FILE} (${catalog.games.length} game(s))`);
		Object.assign(local, { order: entry.order, added: entry.added, updated: entry.updated });
		const localFile = path.join(rel(cfg.out_dir), "games.local.json");
		fs.mkdirSync(path.dirname(localFile), { recursive: true });
		fs.writeFileSync(localFile, JSON.stringify({ format: 1, updated: today(), games: [local] }, null, "\t") + "\n", "utf8");
		console.log(`test list for the launcher: ${localFile} (INTEGRATION.md step 11)`);
		return;
	}
	putFile(HUB_REPO, CATALOG_FILE, Buffer.from(text, "utf8"), `Spieleliste: ${cfg.game_name} ${old ? "aktualisiert" : "eingetragen"}`, who);
	console.log(`REGISTER OK: ${cfg.game_name} is in https://github.com/${HUB_REPO}/blob/main/${CATALOG_FILE}`);
}

function main() {
	const args = parseArgs(process.argv.slice(2));
	if (!["check", "manifest", "release", "register"].includes(args.command)) {
		console.log("usage: node tools/dasdimo/dasdimo_release.js check | manifest [--suffix x] | release [--suffix x] [--readme] | register [--dry-run]");
		console.log("helper version " + HELPER_VERSION);
		process.exit(args.command === "help" ? 0 : 1);
	}
	const cfg = loadConfig(args.config);
	if (args.command === "check") return commandCheck(cfg);
	if (args.command === "manifest") {
		const problems = configProblems(cfg);
		if (problems.length) fail(problems.join("; "));
		buildRelease(cfg, args.suffix);
		return console.log("MANIFEST OK (nothing uploaded)");
	}
	if (args.command === "release") return commandRelease(cfg, args);
	return commandRegister(cfg, args);
}

main();
```
