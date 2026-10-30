# Ein Spiel oder eine Minecraft-Mod in den DasDimo Launcher bringen

**Für:** den Claude-Agenten eines Spiels oder einer Minecraft-Mod von DasDimo (GitHub-Konto `dasdimo`) – egal mit welcher Engine.
**Version der Anleitung:** 5 (2026-10-29) · Hilfs-Skript `dasdimo_release.js` Version 5 (Anhang).
**Neu in Version 5:** **Minecraft-Mods** (DasDimo Launcher ab 1.4.0): Eine Mod bekommt im Launcher eine eigene Seite, ein Klick legt die `.jar` in ein Minecraft-Profil, und die Seite zeigt, was man sonst noch braucht. Alles dazu im Teil „Minecraft-Mods“ (nach Schritt 13). Für Spiele ändert sich nichts – das Skript aus dem Anhang übernehmen genügt.
**Neu in Version 4:** **Discord-Beitrag pro Version:** Nach `release` legt das Skript im Discord-Patch-Forum deines Spiels von selbst einen neuen Beitrag mit den deutschen Neuigkeiten und Bildern an – sobald der Benutzer für dein Spiel einen Webhook eingerichtet hat. Neuer Befehl `discord` (Abschnitt „Discord-Beitrag“ in Schritt 10). Schon eingerichtet? Skript aus dem Anhang übernehmen, sonst ändert sich nichts.
**Neu in Version 3:** Spiele aus **mehreren Dateien** (Build-Ordner, Manifest-Format 2): Bei einem Update lädt der Launcher nur die geänderten Dateien statt des ganzen Spiels (DasDimo Launcher ab 1.1.0). Neuer Befehl `diff` (zeigt vor dem Release, wie viel die Spieler laden), Feld `manifest_name`, die `.zip`-Variante entfällt. Schon eingerichtet? Skript aus dem Anhang übernehmen – mit `path` (eine exe) läuft alles weiter wie bisher; zum Umstieg siehe „Spiel aus mehreren Dateien“ (nur nach Rücksprache mit dem Benutzer).
**Neu in Version 2:** Screenshots (`catalog.screenshots`), Eintrag mit `"type": "game"`, Spiele nur über den Launcher, Test mit dem Launcher (Schritt 11).
**Öffentliche Adresse:** https://github.com/dasdimo/dasdimo-launcher-releases/blob/main/INTEGRATION.md
Zum Lesen in Claude Code: `gh api repos/dasdimo/dasdimo-launcher-releases/contents/INTEGRATION.md -H "Accept: application/vnd.github.raw"`

Lies diese Datei ganz, bevor du anfängst. Arbeite die Schritte der Reihe nach ab und hake sie in deiner Aufgabenliste ab. **Eine Minecraft-Mod?** Lies zuerst den Teil „Minecraft-Mods“ weiter unten – er sagt, was bei den Schritten 1–13 anders ist.

---

## Worum es geht
Der **DasDimo Launcher** ist ein kleines Windows-Programm für die Freunde des Benutzers. Er zeigt alle Spiele von DasDimo, installiert sie mit einem Klick, holt Updates und startet sie. Damit dein Spiel darin erscheint, braucht es:
1. ein **öffentliches Release-Repo** `dasdimo/<spiel-id>-releases` mit fertigen Builds (der Quellcode bleibt privat),
2. pro Version ein **GitHub-Release** mit den Spieldateien für Windows x64 (und wenn möglich ARM64) und einem Manifest (`manifest.json`) mit SHA-256-Prüfsummen,
3. **Bilder** (Icon, Cover, Screenshots) im Release-Repo,
4. einen **Eintrag in der Spieleliste** `games.json` der Zentrale `dasdimo/dasdimo-launcher-releases`.

Das Hilfs-Skript im Anhang erledigt Manifest, Release, Bilder und Eintrag. Du schreibst nur eine kleine Konfiguration.

**Spiele gibt es nur über den Launcher.** Freunde installieren einmal den DasDimo Launcher (Installer `DasDimoLauncher-Setup.exe` aus https://github.com/dasdimo/dasdimo-launcher-releases) und bekommen darüber alle Spiele. README und Release-Notizen deines Spiels verlinken deshalb den Launcher, nie die Spiel-Dateien direkt. Der Launcher liest die Spieleliste bei jedem Start neu: Sobald dein Eintrag drin ist, erscheint dein Spiel bei allen – ohne Launcher-Update.

## Regeln (gelten immer)
1. **Sprache:** Mit dem Benutzer Deutsch, einfache Worte. Er kann nicht programmieren. Entscheidungen als Auswahl-Fragen mit 2–4 Vorschlägen, die empfohlene zuerst mit „(Empfohlen)“.
2. **Erst fragen, dann veröffentlichen:** Repo anlegen, Release hochladen (samt Discord-Beitrag), Bilder hochladen, Spieleliste ändern – jedes Mal vorher sagen, was öffentlich wird, und auf ein klares Ja warten.
3. **Keine private E-Mail in öffentlichen Repos:** In Release-Repos und in der Zentrale **nie** `git commit`/`git push` und kein `gh repo create --add-readme`. Dateien kommen nur über das Hilfs-Skript (GitHub-API mit der noreply-Adresse `<id>+dasdimo@users.noreply.github.com`). Releases selbst erzeugen keine Commits.
4. **Releases nur per Skript:** immer `dasdimo_release.js release`, nie per Hand im Browser oder mit einzelnen `gh release`-Befehlen (Ausnahme: Test-Release löschen, Schritt 13).
5. **Downloads nur mit Erlaubnis:** Dateiname, Quelle und Größe nennen und fragen.
6. **Nur fertige Builds öffentlich:** kein Quellcode, keine Zugangsdaten/Schlüssel, keine Rohdateien von Assets, die nicht weitergegeben werden dürfen (z. B. Mixamo-FBX). In einer fertigen exe oder einem Paket (`.pck`) eingebaut ist in Ordnung.
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
| Build-Ordner oder eine exe | Build-Ordner (Format 2, Updates laden nur geänderte Dateien) oder eine exe (Format 1, jedes Update lädt alles) – Schritt 2 | Build-Ordner |

## Schritt 2 – Spiel vorbereiten
Dein Build muss diese Punkte erfüllen. Prüfe jeden und repariere, was fehlt:
- **Startet ohne Installer und ohne Admin-Rechte** per Doppelklick aus jedem Ordner, z. B. `%LOCALAPPDATA%\DasDimo-Launcher\Games\<game_id>\game\`. Pfade mit Leerzeichen und Umlauten müssen gehen.
- **Schreibt nichts neben die exe.** Der Spielordner gehört dem Launcher: Updates tauschen dort Dateien aus, Deinstallieren löscht ihn. Einstellungen, Spielstände und Logs gehören nach `%APPDATA%`, `%LOCALAPPDATA%` oder `%USERPROFILE%\AppData\LocalLow` (Godot `user://` = `%APPDATA%\Godot\app_userdata\<Projektname>` – das passt). Notiere diese Ordner für Schritt 5 (`user_data_dirs`).
- **Verlässt sich nicht auf das Arbeitsverzeichnis** und braucht **keine Start-Argumente** (der Launcher startet die exe ohne Argumente).
- **Kein eigenes Update-System** im Spiel, keine Zusatz-Installer (z. B. VC++-Pakete) nötig.
- **Build-Ordner oder eine exe:**
  - **Build-Ordner (Format 2, empfohlen):** Startdatei + Spieldaten als einzelne Dateien. Bei einem Update lädt der Launcher nur, was sich geändert hat – oft wenige MB statt des ganzen Spiels. Details im nächsten Abschnitt.
  - **Eine exe (Format 1):** am einfachsten, aber jedes Update lädt das ganze Spiel. Gut für kleine Spiele (unter etwa 50 MB).
- **Version an einer Stelle** im Projekt (z. B. `project.godot` → `config/version`), Form `1.2.3`.

**Engine-Hinweise:**
- **Godot 4:** Export „Windows Desktop“. Build-Ordner: `binary_format/embed_pck=false` → `Spiel.exe` + `Spiel.pck`, jede Plattform in einen eigenen Ordner exportieren (die exe sucht die `.pck` mit gleichem Namen daneben). Eine exe: `embed_pck=true`. Für ARM64 ein zweites Preset mit `binary_format/architecture="arm64"` (die offiziellen Export-Vorlagen enthalten `windows_release_arm64.exe`; prüfe den Vorlagen-Ordner und frag vor einem Download). Auf Snapdragon läuft Direct3D 12 meist besser als Vulkan: RIFTWAKE setzt im ARM64-Preset `custom_features="force_d3d12"` und im Projekt `rendering/rendering_device/driver.force_d3d12="d3d12"`.
- **Unity:** Der Windows-Build ist ein Ordner (`Spiel.exe`, `Spiel_Data/`, `UnityPlayer.dll` …) → Build-Ordner. Weglassen (`exclude`): `*_BurstDebugInformation_DoNotShip/**`, `*.pdb`.
- **Unreal:** Ordner `Windows/` → Build-Ordner, meist nur x64. Weglassen: `*.pdb`.
- **Andere Engines:** Ordner → Build-Ordner, eine einzelne exe → Format 1.

### Spiel aus mehreren Dateien (Format 2)
**So funktioniert es:** Das Manifest nennt jede Datei des Build-Ordners mit Pfad, Größe und SHA-256. Der Launcher vergleicht das mit dem installierten Spiel und lädt nur neue und geänderte Dateien; Dateien, die es nicht mehr gibt, löscht er. Alles landet zuerst in einem Zwischenordner und wird dann in einem Schritt ausgetauscht – bricht ein Update ab, bleibt das alte Spiel spielbar, und fertige Dateien werden beim nächsten Versuch nicht noch einmal geladen. Im Menü der Spielseite gibt es „Spieldateien prüfen“: Der Launcher findet kaputte Dateien und lädt nur diese neu.

**Launcher-Version:** Format 2 braucht den DasDimo Launcher 1.1.0 oder neuer. Ältere Launcher zeigen bei deinem Spiel „ließ sich nicht prüfen“ (die installierte Version bleibt spielbar), bis sie sich aktualisiert haben. Ein Spiel, das schon im Launcher ist, stellst du deshalb nur nach Rücksprache mit dem Benutzer um. Beim Umstieg laden die Spieler einmal das ganze Spiel, danach nur noch Änderungen.

**Damit Updates wirklich klein werden:**
1. **Die exe bleibt gleich.** Sie ist meist die Engine (bei Godot rund 100 MB) und soll sich nur ändern, wenn sich die Engine-Version ändert.
   - Godot: Im Export-Preset `application/file_version` und `application/product_version` fest setzen, z. B. `1.0.0.0`. Leer heißt „Spielversion“, und dann ist die exe bei jedem Release neu. Icon und Firmenname nicht ändern.
   - Dateinamen ohne Versionsnummer (`Beispielspiel.exe`, nicht `Beispielspiel-v0.2.0.exe`): Ein neuer Name ist für den Launcher eine neue Datei.
2. **Große Inhalte in eigene Pakete** (optional, lohnt sich ab etwa 50 MB Spieldaten). Das Hauptpaket ändert sich bei jeder Code-Änderung. Liegen Texturen, Modelle, Karten und Sounds in eigenen Dateien, lädt ein normales Update nur das kleine Hauptpaket.
   - Godot: zusätzliche Export-Presets mit Ressourcen-Filter, exportiert mit `godot --headless --export-pack "<Preset>" builds/x64/packs/textures.pck`; das Hauptpaket schließt diese Ordner aus. Skripte bleiben im Hauptpaket.
   - Godot laden: ganz am Anfang, im `_init()` des ersten Autoloads, jedes Paket mit `ProjectSettings.load_resource_pack(OS.get_executable_path().get_base_dir().path_join("packs/textures.pck"))`. Im Editor nicht (`OS.has_feature("editor")`). Fehlt ein Paket, zeigt das Spiel einen klaren Fehler.
   - Pakete einmal exportieren und in beide Build-Ordner (x64 und ARM64) kopieren. Gleiche Dateien lädt das Skript nur einmal hoch (`"arch": "all"`).
   - Unity: Addressables/AssetBundles; Unreal: Pak-Chunks. Nur wenn der Benutzer es will – ein Build-Ordner ohne Pakete funktioniert auch.
3. **Gleiche Eingabe → gleiche Datei:** Zweimal ohne Änderung exportieren muss dieselbe SHA-256 ergeben (`Get-FileHash`). Sonst laden die Spieler jedes Mal alles neu.
4. **Vor jedem Release `diff`** (Schritt 9): zeigt pro Plattform, welche Dateien und wie viele MB die Spieler laden, und warnt, wenn sich die exe geändert hat.

**Grenzen** (prüft das Skript): Pfade höchstens 200 Zeichen und 16 Ordner tief, keine leeren Dateien, keine Windows-Sondernamen (`CON`, `NUL` …), höchstens 900 Release-Dateien (GitHub erlaubt 1000), jede Datei höchstens 2 GB. Viele kleine Dateien gehen, laden aber langsamer – lieber bündeln.

## Schritt 3 – Release-Repo anlegen (erst nach Ja des Benutzers)
```powershell
gh repo create dasdimo/<game_id>-releases --public --description "<Name> – Spielversionen zum Herunterladen / game downloads" --disable-issues --disable-wiki
```
Keine Optionen `--add-readme`, `--clone` oder `--source` (sonst Commit mit privater E-Mail). Die README kommt beim ersten Release über das Skript.

## Schritt 4 – Builds und Dateinamen
- Builds landen im Projekt unter `builds/` (nicht in Git). Das Skript liest die Version aus deinem Projekt und setzt sie für `{version}` ein.
- **Build-Ordner (Format 2):** ein Ordner pro Plattform, z. B. `builds/x64/` und `builds/arm64/`, darin die Startdatei mit festem Namen (`Beispielspiel.exe`). Nur Dateien, die zum Spiel gehören – Debug-Dateien (`*.pdb`) über `exclude` weglassen.
- **Eine exe (Format 1):** `<Name>-v<version>.exe` (x64) und `<Name>-v<version>-arm64.exe` (ARM64). Nur `A–Z a–z 0–9 . _ -`, keine Leerzeichen/Umlaute (`Beispielspiel-v0.1.0.exe`).

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
		{"arch": "x86_64", "dir": "builds/x64", "exe": "Beispielspiel.exe", "exclude": ["*.pdb"]},
		{"arch": "arm64", "dir": "builds/arm64", "exe": "Beispielspiel.exe", "exclude": ["*.pdb"]}
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
Mit einer einzelnen exe (Format 1) stehen in `files` stattdessen `{"arch": "x86_64", "path": "builds/Beispielspiel-v{version}.exe"}` und `{"arch": "arm64", "path": "builds/Beispielspiel-v{version}-arm64.exe"}`.

| Feld | Bedeutung |
|---|---|
| `version_file` + `version_pattern` | Datei und regulärer Ausdruck (eine Klammer = die Version). Alternativ fest: `"version": "0.1.0"` |
| `files` | eine Zeile pro Plattform, `x86_64` Pflicht. Build-Ordner: `dir` (Ordner, darf `{version}` enthalten) + `exe` (Startdatei darin, bei allen Plattformen gleich), optional `exclude` (`*.pdb` gilt für Dateinamen überall, `logs/**` für einen Pfad ab dem Build-Ordner). Eine exe: `path`. Nicht mischen |
| `manifest_name` | optional, Standard `manifest.json`. Nur ändern, wenn der Benutzer es sagt (RIFTWAKE: `riftwake.json`), und danach nie mehr – `games.json` zeigt auf diesen Namen |
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

## Schritt 7 – Das Manifest (schreibt das Skript – nur zum Verstehen)
Das Skript schreibt pro Release diese Datei (`manifest.json` oder `manifest_name`). Der Launcher prüft sie streng; eine ungültige Datei führt nie zu einem Download. Format 2 (Build-Ordner):

```json
{
	"format": 2,
	"game": "Beispielspiel",
	"game_id": "beispielspiel",
	"repo": "dasdimo/beispielspiel-releases",
	"version": "0.2.0",
	"tag": "v0.2.0",
	"date": "2026-11-15",
	"launcher_version": "",
	"exe": "Beispielspiel.exe",
	"files": [
		{"path": "Beispielspiel.exe", "arch": "x86_64", "size": 109268480, "sha256": "64 Hex-Zeichen …", "url": "https://github.com/dasdimo/beispielspiel-releases/releases/download/v0.2.0/3f9a1c2b7d12-Beispielspiel.exe"},
		{"path": "Beispielspiel.exe", "arch": "arm64", "size": 91662336, "sha256": "…", "url": "…/v0.2.0/8b20e4f5a913-Beispielspiel.exe"},
		{"path": "Beispielspiel.pck", "arch": "all", "size": 6100000, "sha256": "…", "url": "…/v0.2.0/c41d9e07ab55-Beispielspiel.pck"},
		{"path": "packs/textures.pck", "arch": "all", "size": 31000000, "sha256": "…", "url": "…/v0.2.0/77fe0a1c3b42-textures.pck"}
	],
	"news": [
		{"version": "0.2.0", "date": "2026-11-15", "entries": [{"de": "Neue Karte: Hafen.", "en": "New map: Harbour."}]}
	]
}
```
Format 1 (eine exe) hat kein `exe`, und jede Zeile in `files` sieht so aus: `{"kind": "game", "arch": "x86_64", "name": "Beispielspiel-v0.1.0.exe", "size": …, "sha256": "…", "url": "…/v0.1.0/Beispielspiel-v0.1.0.exe"}`.

| Feld | Regel |
|---|---|
| `format` | `2` = Build-Ordner, jede Datei einzeln (DasDimo Launcher ab 1.1.0). `1` = eine exe pro Plattform. Der Launcher lehnt höhere Formate ab, als er kennt |
| `version` | `MAJOR.MINOR.PATCH`, optional `-zusatz` aus `0-9 A-Z a-z .` (z. B. `0.2.1-test1`); ohne führendes `v`. Eine Version mit Zusatz ist **älter** als ohne: `0.2.1-test1` < `0.2.1` < `0.2.2` |
| `tag` | `v` + Version (Git-Tag des Releases) |
| `exe` | Format 2: Startdatei, Pfad im Spielordner; muss in den Dateien jeder Plattform stecken |
| `files[].path` | Format 2: Pfad im Spielordner mit `/`, z. B. `packs/maps.pck`. Nie `..`, `\`, `:` oder Laufwerk, höchstens 200 Zeichen; Groß-/Kleinschreibung zählt nicht (kein Pfad doppelt) |
| `files[].arch` | `x86_64`, `arm64` oder (Format 2) `all` = für beide. Ein PC bekommt die Dateien seiner Plattform + `all`. Gibt es keine `arm64`-Dateien, nimmt ein Snapdragon-PC die x64-Dateien (läuft emuliert) |
| `files[].kind`, `files[].name` | Format 1: `kind` immer `game` (`launcher` und `setup` gibt es nur in der Zentrale und für den Wechsel vom alten RIFTWAKE-Launcher); `name` = Dateiname ohne Pfad, beginnt mit Buchstabe/Ziffer, dann nur `A–Z a–z 0–9 . _ -`, endet auf `.exe` |
| `files[].size`, `sha256` | Bytes (mehr als 0) und SHA-256 (64 Hex-Zeichen). Der Launcher prüft beides nach dem Download und erkennt daran, was sich geändert hat |
| `files[].url` | voller Link zur Datei im Release (`https://github.com/dasdimo/…`). Format 2: Name im Release = die ersten 12 Zeichen der SHA-256 + Dateiname, weil GitHub keine Ordner kennt und beide Plattformen eine `Beispielspiel.exe` haben |
| `news` | die letzten 3 Versionen, neueste zuerst, jeder Eintrag DE **und** EN |
| `launcher_version` | bei Spielen immer `""` |
| `min_launcher` | optional: die älteste DasDimo-Launcher-Version, mit der diese Spielversion installiert und gespielt werden darf, z. B. `"1.1.0"` (wie `version`, ohne `v`). Ist der Launcher älter, sperrt er „Aktualisieren“ und „Spielen“ (auch die installierte Version) und zeigt „Launcher aktualisieren“; ohne Internet bleibt die installierte Version spielbar. Gilt ab Launcher 1.1.1, ältere ignorieren das Feld. Nur setzen, wenn das Spiel wirklich einen neueren Launcher braucht. Das Hilfs-Skript schreibt es bei Spielen nicht – bei Bedarf beim Benutzer melden (bei Mods immer `1.4.0`) |

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

Optional pro Version: **Bilder für den Discord-Beitrag**, z. B. `"images": ["tools/dasdimo/news_images/0.2.0-wirtin.jpg"]` (Pfade im Repo deines Spiels, `.jpg`/`.png`/`.webp`/`.gif`, höchstens 4 pro Version, zusammen höchstens 8 MB). Sie gehen nur nach Discord, nicht ins Manifest und nicht in den Launcher. Ohne `images` zeigt der Beitrag das Cover und die ersten 3 Screenshots aus `catalog`. Auf den Bildern dürfen keine persönlichen Daten stehen (z. B. Ordner mit dem Windows-Namen des Benutzers).

**`tools/dasdimo/README.md`** – Startseite des Release-Repos, Deutsch oben, Englisch darunter: was das Spiel ist (2 Sätze), wie man es bekommt (**nur** über den DasDimo Launcher: Link https://github.com/dasdimo/dasdimo-launcher-releases, dort den Installer `DasDimoLauncher-Setup.exe` laden – keine direkten Links auf Spiel-Dateien), SmartScreen-Hinweis für den Installer („Weitere Informationen“ → „Trotzdem ausführen“, die Datei ist nicht signiert), wo Einstellungen liegen, an wen man Fehler schickt. Der Quellcode ist nicht öffentlich – das kurz erwähnen.

## Schritt 9 – Hilfs-Skript anlegen und prüfen
1. Kopiere das Skript aus dem **Anhang** unverändert nach `tools/dasdimo/dasdimo_release.js`.
2. Baue die Builds (x64 + ARM64) mit deinem gewohnten Export.
3. Prüfen:
   ```powershell
   node tools/dasdimo/dasdimo_release.js check
   ```
   Muss mit `CHECK OK` enden. Jede `PROBLEM`-Zeile sagt, was fehlt (Größe der Bilder, Version, Dateien, Neuigkeiten …). Bei einem Build-Ordner steht pro Plattform, wie viele Dateien und MB es sind.
4. Probelauf ohne Upload:
   ```powershell
   node tools/dasdimo/dasdimo_release.js manifest
   ```
   Schreibt `builds/dasdimo-release/v<version>/manifest.json` (bzw. `manifest_name`) + `notes.md`. Kurz ansehen.
5. Gibt es schon ein Release: vergleichen, was die Spieler laden würden:
   ```powershell
   node tools/dasdimo/dasdimo_release.js diff
   ```
   Zeigt pro Plattform die geänderten, neuen und entfernten Dateien und die MB (mit `--against <datei>` gegen ein Manifest auf der Festplatte). Nenne dem Benutzer vor jedem Release die MB. `WARNING: the start file … changed` ohne Engine-Wechsel heißt: Die exe ist nicht fest (Abschnitt „Spiel aus mehreren Dateien“, Punkt 1).
6. Leg im eigenen Projekt ein kleines Release-Skript oder einen Abschnitt in der Doku an („Neue Version veröffentlichen“ – Schritt 13), damit spätere Sitzungen den Ablauf finden.

## Schritt 10 – Erstes Release (erst nach Ja des Benutzers)
Sag dem Benutzer, was hochgeladen wird (Dateien, Größen, Repo) und frag. Dann:
```powershell
node tools/dasdimo/dasdimo_release.js release --readme
```
Das Skript prüft das Konto und lädt die README (noreply). Dann erstellt es das Release `v<version>`, zuerst nicht als „latest“ – so lange liest der Launcher noch das vorige Release. Es lädt alle Spieldateien und **zuletzt** das Manifest hoch, markiert das Release als „latest“ und vergleicht jede Datei per SHA-256 mit GitHub. Am Ende steht `RELEASE OK` mit Link.
- Jedes Release enthält alle Dateien des Spiels; die Spieler laden trotzdem nur geänderte. Alte Releases darfst du darum später löschen.
- Gibt es das Release mit dieser Version schon, lädt das Skript nur geänderte Dateien neu und entfernt Dateien, die das neue Manifest nicht mehr nennt.

### Discord-Beitrag
Nach `RELEASE OK` legt das Skript im Discord-Patch-Forum deines Spiels einen **neuen Beitrag** an: Titel `<game_name> <version>`, darin die deutschen Neuigkeiten dieser Version aus `news.json` als Liste, dann „Aktualisiere im DasDimo Launcher“ mit dem Link zur Download-Seite. Niemand wird erwähnt (kein @everyone). Lange Neuigkeiten gehen in weiteren Nachrichten im selben Beitrag weiter. Test-Releases (`--suffix`) bekommen keinen Beitrag.
- **Die Webhook-Adresse ist geheim.** Sie liegt nur auf dem PC des Benutzers in `%APPDATA%\DasDimo\discord.json` (Schlüssel = deine `game_id`). Lies, zeig oder kopier diese Datei nie, frag den Benutzer nie im Chat nach der Adresse und schreib sie in kein Repo und keine andere Datei.
- **Einrichten macht der Benutzer selbst:** in Discord ein Forum für die Patches deines Spiels anlegen → Zahnrad (Kanal bearbeiten) → Integrationen → Webhooks → Neuer Webhook → „Webhook-URL kopieren“. Dann im Projekt des DasDimo Launchers `tools\discord\webhooks_einrichten.cmd` doppelklicken (liegt bei ihm unter `Dokumente\Claude Code\DasDimo Launcher`), bei „weiteres Spiel“ deine `game_id` eingeben und die Adresse einfügen. Das Fenster prüft die Adresse und speichert sie nur auf dem PC. RIFTWAKE ist dort schon vorgesehen.
- Ohne Webhook für deine `game_id` steht nach dem Release nur ein Hinweis (`note: no Discord webhook …`), kein Fehler.
- Vorher ansehen (postet nichts): `node tools/dasdimo/dasdimo_release.js discord --dry-run`. Zeig dem Benutzer den Text, wenn du ihn nach dem Ja zum Release fragst – das Ja gilt dann für Release und Beitrag. Will er keinen Beitrag: `release --no-discord`.
- Einzeln posten – wenn es nach dem Release nicht klappte (`DISCORD NOT POSTED: …`) oder wenn dein Spiel ein eigenes Release-Skript hat (dann direkt nach dessen Release aufrufen): `node tools/dasdimo/dasdimo_release.js discord`. Die Version kommt aus `version`/`version_file`, mit `--version 0.4.0` eine bestimmte. Eine Version, die schon gepostet wurde, wird übersprungen (gemerkt in `%APPDATA%\DasDimo\discord_posts.json`); `--again` postet sie noch einmal – nur nach Ja des Benutzers.
- **Bilder:** Unter der Liste zeigt der Beitrag die Bilder der Version (`images` in `news.json`, Schritt 8) als Galerie, sonst das Cover und die ersten 3 Screenshots. Der Probelauf nennt jedes Bild (`picture 1-cover.jpg (386 KB): …`). Fehlt ein Bild oder ist es zu groß, wird es weggelassen (`note: picture …`), der Beitrag kommt trotzdem; `check` meldet es vorher als `PROBLEM discord: …`.
- Stehen deine Neuigkeiten nicht in `news.json` (eigenes Release-Skript), gib die Datei mit: `discord --news <datei>`. Das kann eine Liste wie `news.json` sein oder dein gerade gebautes Spiel-Manifest (dessen `news`).
- Am Ende steht `DISCORD OK: <Link zum Beitrag>`. Den Link nennst du im Bericht.

## Schritt 11 – Mit dem Launcher testen
Du testest mit dem **DasDimo Launcher** und einer lokalen Spieleliste, die nur dein Spiel enthält – so siehst du die Spielseite genau wie später, bevor etwas in der Zentrale steht.

**Launcher finden:** Hat der Benutzer ihn installiert, liegt er unter `%LOCALAPPDATA%\DasDimo-Launcher\Launcher\DasDimoLauncher.exe` (für Format 2 mindestens Version 1.1.0 – steht in den Einstellungen). Fehlt er, bitte den Benutzer, ihn über den Installer von https://github.com/dasdimo/dasdimo-launcher-releases zu installieren (ein Klick, keine Admin-Rechte).

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
   Erwartet: im Log `games.json: 1 game(s)` und `installed <version>`; in `installed.json` `"version": "<version>"`, das Spiel liegt in `$test\Games\<game_id>\game\`. Mit `--arch=arm64` prüfst du die ARM64-Auswahl. `--root-dir` hält alles im Testordner (die echte Installation des Benutzers bleibt unberührt), `--portable` verhindert die Frage „Launcher installieren?“. Bei Format 2 steht im Log nach jedem Update `loaded <n> of <m> files` – so siehst du, dass nur geänderte Dateien kommen.
3. Test mit dem Benutzer: denselben Befehl **ohne** `--auto-update --quit-when-done` starten. Der Launcher zeigt nur dein Spiel – mit Icon, Cover, Screenshots und Neuigkeiten. Bitte den Benutzer, sich die Seite anzusehen, auf **Spielen** zu klicken, und frag, ob alles passt und das Spiel startet.
4. Danach den Testordner löschen (`$env:TEMP\dasdimo-launcher-test\<game_id>`).
5. Meldet der Launcher einen Fehler: der Grund steht in `$test\Launcher\launcher.log` (z. B. `files[0].path invalid` oder `HTTP 404`, wenn das Release fehlt).

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
   3. `node tools/dasdimo/dasdimo_release.js check`, dann `diff` (dem Benutzer die MB nennen) und `discord --dry-run` (den Discord-Text zeigen), dann `release` (nur wenn der Benutzer veröffentlichen will) – der Discord-Beitrag kommt danach von selbst.
   4. Test-Release zum Ausprobieren: `release --suffix test1` → Version `<version>-test1` wird „latest“. Danach mit OK des Benutzers löschen: `gh release delete v<version>-test1 -R dasdimo/<game_id>-releases --cleanup-tag --yes` – dann ist wieder das vorige Release aktuell.
   5. Neue Bilder oder Screenshots, neuer Untertitel oder anderer Status: `tools/dasdimo/dasdimo.json` ändern, dann `register`.
2. Committe `tools/dasdimo/` (Skript, Konfiguration, Neuigkeiten, README, Bilder) im Repo deines Spiels, mit verständlicher deutscher Commit-Nachricht.
3. Bericht an den Benutzer auf Deutsch: Link zum Release-Repo, Link zum Discord-Beitrag (falls gepostet), Ergebnis des Launcher-Tests, bei Format 2 die MB eines normalen Updates (aus `diff`), was noch offen ist, und eine kurze Nachricht für die Freunde („Neues Spiel im DasDimo Launcher: einfach den Launcher öffnen“).

## Minecraft-Mods
Ab **DasDimo Launcher 1.4.0** stehen auch die Minecraft-Mods des Benutzers im Launcher: links unter „Mods“, mit eigener Seite. Die Schritte 1–13 gelten auch für eine Mod – mit den Unterschieden in diesem Teil. Am Launcher änderst du nichts, alles kommt aus deiner Konfiguration.

**So sieht es im Launcher aus:**
- **Mod-Seite:** Cover, Name mit dem Abzeichen „Mod“, Untertitel, Chips (Minecraft-Version, Loader + Mindestversion, Java, Client/Server), ein Knopf, darunter „Was du brauchst“, aufklappbare Abschnitte (Erste Schritte, Befehle …), Screenshots und rechts die Neuigkeiten.
- **Installieren:** Beim ersten Klick fragt der Launcher „Wohin installieren?“. Er findet die Profile auf dem PC (Minecraft Launcher, Modrinth App, NoRisk, CurseForge, Prism) – passende (gleiche Minecraft-Version und gleicher Loader) stehen oben. Oder ein Ordner von Hand, oder „Nur Datei herunterladen …“ (die geprüfte `.jar` irgendwohin speichern). Die Wahl merkt er sich pro Mod.
- **Alte Datei derselben Mod** im Mods-Ordner (gleiche `id` in `fabric.mod.json`, z. B. eine von Hand kopierte ältere Version): Der Launcher fragt und benennt sie in `.jar.disabled` um. Fremde Dateien löscht er nie; „Entfernen …“ löscht nur seine eigene `.jar`, und nur wenn die SHA-256 passt.
- **Updates** prüft er wie bei Spielen (beim Start und alle 30 Minuten). Ein Update ersetzt nur seine eigene Datei im gewählten Profil.
- **„Was du brauchst“** prüft das gewählte Profil: ✓ gefunden, ⚠ zu alt, ✗ fehlt (mit „Seite öffnen“), ☆ empfohlen, ⚙ Zusatz-Programm (aufklappbar mit den Schritten). Andere Mods erkennt er an der `id` in ihren `.jar`, Minecraft- und Loader-Version an der letzten Startzeile im Log des Profils.
- Launcher bis 1.3.x zeigen Mods nicht – sie überspringen den Eintrag leise.

### Unterschiede Schritt für Schritt
**Schritt 1 – Grunddaten:** Keine Frage nach ARM64 (eine `.jar` läuft überall) und kein „Build-Ordner oder eine exe“. Dafür klärst du:

| Was | Regel | Beispiel |
|---|---|---|
| `game_id` | wie bei Spielen; Name des Release-Repos | `beispielmod` |
| `mod_id` | die `id` aus deiner `fabric.mod.json` – nur angeben, wenn sie anders ist als `game_id` | `beispielmod` |
| Minecraft / Loader / Java | wie in `fabric.mod.json` (`depends`) | 26.3 / Fabric ab 0.20.1 / 25 |
| Client / Server | `client`, `server` oder `both` (Mod gehört auf beide) | `both` |
| Was die Mod braucht | Pflicht-Mods (Mod-ID, Mindestversion, Link), empfohlene Mods (was man ohne verpasst), Programme außerhalb von Minecraft (Schritte zum Einrichten) | Fabric API ab 0.165.0+26.3 |
| Wichtiger Hinweis | höchstens 3 Sätze, die man nicht übersehen darf | „Ohne Spracherkennung werden neue Items gelöscht.“ |

**Schritt 2 – Mod vorbereiten:**
- Gebaut wird wie immer (`gradlew build`). Die fertige Datei liegt in `build/libs/` – die **ohne** `-sources` oder `-dev` im Namen.
- **Fabric/Quilt:** In der `.jar` liegt `fabric.mod.json` (bzw. `quilt.mod.json`) mit `id` und `version`; `version` = Release-Version (z. B. `${version}` aus `gradle.properties`). Daran erkennt der Launcher alte Dateien deiner Mod. Forge/NeoForge geht auch, dann kann der Launcher alte Dateien und andere Mods im Profil aber nicht erkennen.
- **Dateiname:** nur `A–Z a–z 0–9 . + _ -`, endet auf `.jar`, höchstens 128 Zeichen. Die Version darf im Namen stehen (`beispielmod-0.2.0.jar`) – bei einem Update tauscht der Launcher seine eigene alte Datei aus.
- **Lizenzen:** Eingebaute Bibliotheken brauchen ihren Lizenzhinweis (in der `.jar` und in der README), das Projekt eine `LICENSE`-Datei. Vor dem ersten Release mit dem Benutzer klären.
- Eine Mod hat **keine** `user_data_dirs`: Ihre Einstellungen liegen im Minecraft-Profil (`config/`) und gehören dem Spieler.

**Schritt 3 – Release-Repo:** wie bei Spielen, Beschreibung z. B. `"<Name> – Minecraft-Mod (fertige .jar) / Minecraft mod downloads"`.

**Schritt 4** entfällt.

**Schritt 5 – Konfiguration:** `"type": "minecraft_mod"`; `files` nennt die `.jar`; `mod` enthält die Texte der Mod-Seite. Beispiel:

```json
{
	"type": "minecraft_mod",
	"game_id": "beispielmod",
	"game_name": "Beispielmod",
	"repo": "dasdimo/beispielmod-releases",
	"version_file": "gradle.properties",
	"version_pattern": "^mod_version\\s*=\\s*(\\S+)",
	"files": [{"jar": "build/libs/beispielmod-{version}.jar", "loader": "fabric"}],
	"news_file": "tools/dasdimo/news.json",
	"readme_file": "tools/dasdimo/README.md",
	"mod": {
		"mod_id": "beispielmod",
		"minecraft": {"version": "26.3", "loader": "fabric", "loader_version": "0.20.1", "java": "25", "side": "both"},
		"important": [
			{"de": "Ohne Spracherkennung schlägt jeder Versuch fehl – neue Items werden gelöscht.", "en": "Without speech recognition every try fails – new items get deleted."}
		],
		"requires": [
			{"type": "loader", "name": "Fabric Loader", "min_version": "0.20.1", "url": "https://fabricmc.net/use/installer/",
				"steps": [{"de": "Fabric-Installer starten, Minecraft 26.3 und Loader 0.20.1 wählen.", "en": "Run the Fabric installer, pick Minecraft 26.3 and loader 0.20.1."}]},
			{"type": "mod", "name": "Fabric API", "mod_id": "fabric-api", "min_version": "0.165.0+26.3", "url": "https://modrinth.com/mod/fabric-api",
				"why": {"de": "Schnittstellen für Anzeige, Befehle und Netzwerk.", "en": "Hooks for the display, commands and networking."}},
			{"type": "mod", "name": "Simple Voice Chat", "mod_id": "voicechat", "optional": true, "url": "https://modrinth.com/plugin/simple-voice-chat",
				"why": {"de": "Nutzt das Mikrofon des Voice Chats.", "en": "Uses the voice chat's microphone."},
				"without": {"de": "Die Mod nimmt dann selbst auf – geht genauso.", "en": "The mod records by itself – works just as well."}},
			{"type": "program", "name": {"de": "whisper.cpp + Sprachmodell", "en": "whisper.cpp + speech model"}, "size": "0,5–1,6 GB",
				"url": "https://github.com/ggml-org/whisper.cpp/releases",
				"why": {"de": "Erkennt, was du sagst – lokal und kostenlos.", "en": "Recognises what you say – locally and for free."},
				"links": [{"label": {"de": "Sprachmodell", "en": "Speech model"}, "url": "https://huggingface.co/ggerganov/whisper.cpp"}],
				"steps": [
					{"de": "Zip laden und nach C:\\whisper entpacken, das Modell nach C:\\whisper\\models legen.", "en": "Extract the zip to C:\\whisper, put the model into C:\\whisper\\models."},
					{"de": "Starten und das Fenster beim Spielen offen lassen:", "en": "Run it and keep the window open while playing:", "code": "whisper-server.exe -m models\\ggml-small.bin --port 8080"}
				],
				"check": {"de": "Im Spiel /beispielmod mictest ausführen.", "en": "Run /beispielmod mictest in game."},
				"alternative": {"de": "Oder ein OpenAI-Schlüssel in der Einstellungsdatei der Mod (kostet Geld).", "en": "Or an OpenAI key in the mod's settings file (costs money)."}}
		],
		"sections": [
			{"id": "first_steps", "items": [{"de": "Mikrofon und Erkennung prüfen:", "en": "Check microphone and recognition:", "code": "/beispielmod mictest"}]},
			{"id": "commands", "items": [{"de": "Deine Statistik", "en": "Your statistics", "code": "/beispielmod stats"}]},
			{"id": "known_issues", "items": [{"de": "Profile mit Loader 0.19.x: im Profil auf 0.20.1 umstellen.", "en": "Profiles with loader 0.19.x: switch the profile to 0.20.1."}]}
		],
		"guide_url": "https://github.com/dasdimo/beispielmod-releases#readme"
	},
	"catalog": {
		"tagline": {"de": "Sag jedes neue Item laut auf Japanisch – sonst ist es weg.", "en": "Say every new item out loud in Japanese – or it's gone."},
		"accent": "#FFD000",
		"status": "testing",
		"icon": "tools/dasdimo/art/icon.png",
		"cover": "tools/dasdimo/art/cover.jpg",
		"screenshots": ["tools/dasdimo/art/shot1.jpg", "tools/dasdimo/art/shot2.jpg", "tools/dasdimo/art/shot3.jpg"]
	}
}
```

| Feld | Regel |
|---|---|
| `type` | `"minecraft_mod"` (bei Spielen weglassen oder `"game"`) |
| `files` | eine Zeile pro Loader (`fabric`, `quilt`, `forge`, `neoforge`), höchstens 4: `jar` = Pfad zur fertigen `.jar` (darf `{version}` enthalten), `loader`. Kein `arch`, `dir`, `exe` oder `path` – eine `.jar` ist auf jedem PC gleich |
| `mod.mod_id` | die `id` aus `fabric.mod.json`, Standard = `game_id`. `check` vergleicht sie mit der `.jar` |
| `mod.minecraft` | `version` (Pflicht, z. B. `26.3`), `loader` (Pflicht, wie in `files`), `loader_version` (Mindestversion des Loaders), `java` (nur die Zahl), `side` (`client`, `server`, `both`). Daraus werden die Chips, und passende Profile stehen oben |
| `mod.important` | 0–3 Sätze, gelber Kasten oben in „Was du brauchst“. Nur für Dinge, die wirklich schiefgehen |
| `mod.requires` | 0–12 Einträge, siehe nächste Tabelle, in der gewünschten Reihenfolge (Loader zuerst) |
| `mod.sections` | aufklappbare Abschnitte mit festen `id`s (die Überschrift kommt vom Launcher): `first_steps` „Erste Schritte“ (offen), `commands` „Befehle“, `controls` „Steuerung“, `multiplayer` „Multiplayer“, `server` „Eigener Server“, `known_issues` „Bekannte Probleme“. `items`: 1–12 Zeilen `{de, en}`, mit `code` in Festbreitenschrift und mit Kopieren-Knopf. `commands` zeigt `code` links und den Text rechts als Tabelle |
| `mod.guide_url` | optional: ausführliche Anleitung (z. B. die README im Release-Repo), Knopf „Anleitung öffnen“ |
| `catalog` | wie bei Spielen, aber **ohne** `engine` und `user_data_dirs`; **mindestens 2** `screenshots` |

Einträge in `mod.requires`:

| Feld | Regel |
|---|---|
| `type` | `loader` = der Mod-Loader (geprüft über das Profil), `mod` = eine andere Mod (geprüft über die `.jar` im Mods-Ordner), `program` = etwas außerhalb von Minecraft (nicht prüfbar, aufklappbar mit Schritten) |
| `name` | Anzeigename, Text oder `{de, en}`, höchstens 48 Zeichen |
| `mod_id` | bei `mod` Pflicht: die `id` aus der `fabric.mod.json` dieser Mod (`fabric-api`, `voicechat`). Bei `loader` optional (Standard `fabricloader`), bei `program` nie |
| `min_version` | optional: Mindestversion wie in deren `fabric.mod.json` (`0.165.0+26.3`); ist sie im Profil älter, zeigt der Launcher ⚠ „zu alt“ |
| `optional` | `true` = empfohlen (☆) statt Pflicht. Nicht beim Loader |
| `url` | Seite zum Herunterladen („Seite öffnen“) |
| `why` | ein Satz: wofür es gebraucht wird |
| `without` | bei `optional`: was man ohne verpasst |
| `size` | bei `program`: Downloadgröße, z. B. `"0,5–1,6 GB"` (höchstens 24 Zeichen) |
| `links` | bis zu 4 weitere Links `{"label": {de, en}, "url": …}` |
| `steps` | bis zu 6 Schritte „So geht's“: `{de, en}`, mit oder ohne `code` |
| `check` | bei `program`: wie man prüft, ob es läuft |
| `alternative` | bei `program`: ein anderer Weg |

**Erlaubte Links** (`url`, `links`, `guide_url`): nur `https://` zu modrinth.com, curseforge.com, fabricmc.net, github.com und huggingface.co (auch Unterseiten wie `www.curseforge.com`). Andere Links zeigt der Launcher nicht.
**Grenzen:** jeder Satz DE **und** EN, höchstens 200 Zeichen; Namen höchstens 48, `code` höchstens 300 Zeichen. Der Launcher kürzt Zu-Langes und lässt Kaputtes weg – `check` meldet beides vorher als `PROBLEM`, auch unbekannte Felder (Tippfehler wie `minVersion`). Schreib die Texte für Freunde, nicht für Programmierer.

**Schritt 6 – Bilder:** wie bei Spielen (Icon 256×256, Cover 1920×1080, Screenshots 1920×1080), aber **mindestens 2 Screenshots** – die Mod-Seite lebt von ihren Bildern. In Minecraft: Vollbild auf einem Full-HD-Bildschirm, F1 blendet die Leiste aus, F2 speichert nach `<Profil>\screenshots\`. Kleine Bilder (z. B. 854×480 aus einem Test-Client) nicht hochrechnen, lieber neu aufnehmen. Keine Spielernamen, Server-Adressen oder Chats mit Namen im Bild. Hat die Mod kein Icon, baue eines aus einem Motiv der Mod und zeig es dem Benutzer.

**Schritt 7 – Manifest:** Format 2 mit einer Datei der Art `mod` und den Texten aus `mod` (so wie in `dasdimo.json`):
```json
{
	"format": 2,
	"game": "Beispielmod",
	"game_id": "beispielmod",
	"repo": "dasdimo/beispielmod-releases",
	"version": "0.2.0",
	"tag": "v0.2.0",
	"date": "2026-11-02",
	"launcher_version": "",
	"min_launcher": "1.4.0",
	"mod_id": "beispielmod",
	"minecraft": {"version": "26.3", "loader": "fabric", "loader_version": "0.20.1", "java": "25", "side": "both"},
	"important": […], "requires": […], "sections": […], "guide_url": "…",
	"files": [
		{"kind": "mod", "loader": "fabric", "path": "beispielmod-0.2.0.jar", "arch": "all", "size": 13293908, "sha256": "64 Hex-Zeichen …", "url": "https://github.com/dasdimo/beispielmod-releases/releases/download/v0.2.0/beispielmod-0.2.0.jar"}
	],
	"news": […]
}
```
`min_launcher` ist bei Mods immer `1.4.0` (schreibt das Skript). Kein `exe`. Die `.jar` behält im Release ihren Namen (`path` = Dateiname im Mods-Ordner).

**Schritt 8 – README:** Deutsch oben, Englisch darunter: was die Mod macht (2–3 Sätze), was man braucht (Minecraft-Version, Loader, Pflicht-Mods, Programme), wie man sie bekommt: über den DasDimo Launcher (Link https://github.com/dasdimo/dasdimo-launcher-releases) – er legt sie ins Profil, hält sie aktuell und zeigt, was fehlt. Ob die README auch sagt „Wer Mods selbst verwaltet: die `.jar` aus dem neuesten Release“, entscheidet der Benutzer. Dazu die ausführliche Einrichtung der Zusatz-Programme (`guide_url` zeigt hierher), Lizenzen/Credits, an wen man Fehler schickt. Die Release-Notizen schreibt das Skript („So bekommst du es“, „Braucht: Minecraft 26.3 · Fabric Loader ≥ 0.20.1 · …“, Tabelle mit der `.jar` und SHA-256).

**Schritt 9 – Prüfen:** `check` zeigt die `.jar` und was ihre `fabric.mod.json` sagt (`id`, `version`, `depends`). Dazu `NOTE`-Zeilen – kein Fehler, aber anschauen: eine andere Version als das Release, oder eine Abhängigkeit aus `depends`, die `requires` nicht nennt (die sieht der Spieler dann nicht). `diff` sagt, ob die Spieler eine neue `.jar` laden.

**Schritt 10 – Release:** wie bei Spielen. **Discord:** Ein Beitrag kommt nur, wenn der Benutzer für die Mod ein eigenes Forum mit Webhook eingerichtet hat (Schlüssel = `game_id`, siehe „Discord-Beitrag“). Frag ihn beim ersten Release, ob er eines will – ohne Webhook steht nur ein Hinweis.

**Schritt 11 – Test:** Der Launcher braucht dafür Version 1.4.0 oder neuer. Automatisch, alles im Testordner (die echten Minecraft-Profile bleiben unberührt):
```powershell
$launcher = "$env:LOCALAPPDATA\DasDimo-Launcher\Launcher\DasDimoLauncher.exe"
$test = "$env:TEMP\dasdimo-launcher-test\<game_id>"
$list = (Resolve-Path "builds\dasdimo-release\games.local.json").Path
New-Item -ItemType Directory -Force "$test\mods" | Out-Null
& $launcher -- "--catalog-url=$list" "--root-dir=$test\launcher" --portable --game=<game_id> "--mod-target=$test\mods" --auto-update --quit-when-done | Out-Null
Get-Content "$test\launcher\Launcher\launcher.log"; Get-Content "$test\launcher\Games\<game_id>\installed.json"; Get-ChildItem "$test\mods"
```
Erwartet: im Log `<game_id>: installed <version> as …`, in `installed.json` `"kind": "mod"` und `"version": "<version>"`, die `.jar` in `$test\mods`. Mit `--root-dir` sieht der Launcher keine echten Profile; `--mod-target` gibt ihm den Testordner als Ziel.
Test mit dem Benutzer: denselben Befehl ohne `--mod-target`, `--auto-update` und `--quit-when-done`, dafür mit `"--minecraft-home=$env:APPDATA"` – dann liest der Launcher die echten Profile (nur lesen), und „Was du brauchst“ prüft sie wirklich. Bitte den Benutzer, die Seite anzuschauen und „Wohin installieren?“ zu öffnen. Installiert er dabei in ein echtes Profil, liegt die Mod dort wirklich – vor dem Löschen des Testordners über ⋯ → „Entfernen …“ wieder herausnehmen.

**Schritt 12 – Eintragen:** wie bei Spielen. `register` trägt `"type": "minecraft_mod"` und beide Plattformen ein. Launcher bis 1.3.x überspringen den Eintrag.

**Schritt 13 – Neue Version:** Version in `gradle.properties` erhöhen, `news.json` ergänzen, `gradlew build` (und die Tests der Mod), dann `check`, `diff`, `discord --dry-run` und – nach Ja des Benutzers – `release`. Neue Texte für die Mod-Seite (z. B. ein neuer Befehl) stehen in `mod` und kommen mit dem nächsten Release; neue Bilder, Untertitel oder Status mit `register`.

## Fehlerhilfe
| Meldung | Lösung |
|---|---|
| `gh is logged in as '…'` | `gh auth status`; mit dem Konto `dasdimo` anmelden (Benutzer fragen) |
| `… does not exist yet` | Schritt 3 |
| `build missing: …`, `build folder missing: …` | Build fehlt, oder `path`/`dir` passt nicht zum Build |
| `start file … not in …` | `exe` passt nicht zum Namen der Startdatei im Build-Ordner |
| `empty file`, `name reserved by Windows`, `character not allowed in Windows` | Datei umbenennen oder über `exclude` weglassen |
| `… release files - at most 900` | kleine Dateien zu Paketen bündeln |
| `no file of its own` | x64- und ARM64-Ordner sind gleich – nur `x86_64` eintragen |
| `WARNING: the start file … changed` | Abschnitt „Spiel aus mehreren Dateien“, Punkt 1 (nach einem Engine-Update normal) |
| `news: no entry for <version>` | `news.json` für die neue Version ergänzen |
| `catalog.cover … must be 1920x1080` | Bild in genau dieser Größe neu speichern (gilt auch für `catalog.screenshots[…]`) |
| `… has no release yet` | erst Schritt 10, dann `register` |
| Launcher: „Manifest ungültig“ | `launcher.log` lesen; meist ein Dateiname mit Leerzeichen/Umlaut oder eine Version mit `v` |
| Launcher: „ließ sich nicht prüfen“ bei einem Format-2-Spiel | Launcher älter als 1.1.0 – er aktualisiert sich selbst, sonst die Pille „Neuer Launcher“ anklicken |
| `note: no Discord webhook for '<id>'` | Für dein Spiel ist noch kein Webhook eingerichtet (Schritt 10, „Discord-Beitrag“) – kein Fehler |
| `DISCORD NOT POSTED: Discord HTTP 401`/`404` | Webhook in Discord gelöscht: Benutzer legt ihn neu an und startet `webhooks_einrichten.cmd`, dann `discord` |
| `… A tag is required …` | Im Forum ist „Tag erforderlich“ eingeschaltet – der Benutzer schaltet es in den Forum-Einstellungen aus, dann `discord` |
| `did not create a forum post` / `HTTP 400 … thread_name` | Der Webhook gehört zu einem normalen Kanal statt zu einem Forum – Webhook im Forum anlegen |
| `… is not a Discord webhook address` | Eintrag in `discord.json` kaputt – Benutzer startet `webhooks_einrichten.cmd` und fügt die Adresse neu ein |
| Mod: `mod missing: build/libs/…` | `gradlew build` laufen lassen; `files[].jar` passt nicht zum Namen in `build/libs/` |
| Mod: `no fabric.mod.json inside` | falsche `.jar` (`-sources`/`-dev`), oder eine Forge-Mod mit `"loader": "fabric"` |
| Mod: `… says id '…', the launcher looks for '…'` | `mod.mod_id` auf die `id` aus `fabric.mod.json` setzen |
| Mod: `… an https link to modrinth.com, …` | Link auf eine erlaubte Seite ändern (Modrinth, CurseForge, fabricmc.net, GitHub, Hugging Face) oder weglassen |
| Mod: `… unknown (…)` | Tippfehler im Feldnamen (z. B. `minVersion` statt `min_version`) |
| Mod: `catalog.screenshots: a mod needs at least 2` | mindestens 2 Screenshots (Teil „Minecraft-Mods“, Schritt 6) |
| Mod: `NOTE … depends on '…'` | Die Mod braucht noch etwas, das `requires` nicht nennt – ergänzen, oder bewusst weglassen (z. B. eine eingebaute Bibliothek) |
| Launcher: Mod erscheint nicht | Launcher älter als 1.4.0 (aktualisiert sich selbst), oder der Eintrag hat weniger als 2 Screenshots (`launcher.log`: `skipped`) |

---

## Anhang – `tools/dasdimo/dasdimo_release.js`
Unverändert kopieren. Bei einer neuen Version dieser Anleitung das Skript aus der neuen Anleitung übernehmen (die Version steht in der ersten Zeile).

```js
// DasDimo Launcher - release helper for one game or Minecraft mod (helper version 5, from INTEGRATION.md).
// Reads tools/dasdimo/dasdimo.json (or --config <file>). Needs Node 18+ and the GitHub CLI (account dasdimo).
//   node tools/dasdimo/dasdimo_release.js check                      config, builds, news, pictures
//   node tools/dasdimo/dasdimo_release.js diff [--against <file>]    what players load compared to the latest release
//   node tools/dasdimo/dasdimo_release.js manifest [--suffix test1]  only write the manifest + notes.md
//   node tools/dasdimo/dasdimo_release.js release [--suffix test1] [--readme] [--no-discord]   publish a GitHub release
//   node tools/dasdimo/dasdimo_release.js register [--dry-run]       pictures + entry in the launcher's games.json
//     (--dry-run also writes <out_dir>/games.local.json: the entry with local pictures, for a test in the launcher)
//   node tools/dasdimo/dasdimo_release.js discord [--dry-run] [--again] [--version x] [--news <file>]   Discord forum post
//     (runs by itself after "release" when %APPDATA%\DasDimo\discord.json has a webhook for the game_id)
// A game is one exe per arch ("path", manifest format 1: every update loads it whole) or a build folder
// ("dir" + "exe", format 2: the launcher loads only the files that changed).
// A Minecraft mod ("type": "minecraft_mod") is one .jar per loader ("jar" + "loader": format 2, files of kind "mod")
// plus the texts of its page in the launcher ("mod": minecraft, important, requires, sections, guide_url).
// Public commits (README, pictures, games.json) use the GitHub noreply address, never the local git e-mail.
"use strict";
const fs = require("fs");
const os = require("os");
const path = require("path");
const crypto = require("crypto");
const zlib = require("zlib");
const { spawnSync } = require("child_process");

const HELPER_VERSION = 5;
const OWNER = "dasdimo";
const HUB_REPO = "dasdimo/dasdimo-launcher-releases";
const CATALOG_FILE = "games.json";
const NEWS_VERSIONS = 3;
const ARCHES = ["x86_64", "arm64"];
const STATUSES = ["testing", "released", "hidden"];
const VERSION_RE = /^[0-9]+\.[0-9]+\.[0-9]+(-[0-9A-Za-z.]+)?$/;
const NAME_RE = /^[0-9A-Za-z][0-9A-Za-z._-]*\.exe$/;
const MANIFEST_NAME_RE = /^[0-9A-Za-z][0-9A-Za-z._-]*\.json$/;
const ID_RE = /^[a-z0-9][a-z0-9-]{1,31}$/;
const REPO_RE = /^dasdimo\/[A-Za-z0-9._-]+-releases$/;
const DATA_DIR_RE = /^%(APPDATA|LOCALAPPDATA)%\\.+|^%USERPROFILE%\\AppData\\LocalLow\\.+/;
// Game file paths: the same rules as the launcher (core/game_files.gd).
const MAX_PATH_LENGTH = 200;
const MAX_DEPTH = 16;
const RESERVED_RE = /^(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])$/i;
const MAX_FILES = 2000;
// GitHub allows 1000 files per release.
const MAX_ASSETS = 900;
// Release file of a format 2 game: SHA-256 start + file name (GitHub has no folders, both builds have Game.exe).
const ASSET_RE = /^[0-9a-f]{12}-/;
const UPLOAD_BATCH = 40;
const LIST_LINES = 30;
const PICTURES = {
	icon: { type: "png", width: 256, height: 256, maxKb: 300, required: true },
	cover: { type: "jpg", width: 1920, height: 1080, maxKb: 800, required: true },
	logo: { type: "png", maxWidth: 1200, maxHeight: 400, maxKb: 300, required: false },
};
// Pictures of the demo page (catalog.screenshots), shown in this order.
const SCREENSHOT = { type: "jpg", width: 1920, height: 1080, maxKb: 800 };
const MAX_SCREENSHOTS = 8;
const LAUNCHER_PAGE = `https://github.com/${HUB_REPO}`;
// Discord: one forum post per version through the forum's webhook. The webhook addresses are secret and live only
// on the user's PC (%APPDATA%\DasDimo\discord.json, set up by the user) - never in git, a repo or a chat.
const DISCORD_FILE = "discord.json";
const DISCORD_POSTS_FILE = "discord_posts.json";
const WEBHOOK_RE = /^https:\/\/discord\.com\/api\/webhooks\/\d{15,22}\/[A-Za-z0-9_-]{30,}$/;
// Embed text: Discord allows 4096 characters, longer news continue in the next message of the post.
const DISCORD_LIMIT = 4000;
const DISCORD_TITLE_LIMIT = 100;
const DISCORD_COLOR = 0xfd7a10;
// Pictures of a post: Discord shows up to 4 as one gallery. Together at most 8 MB (Discord: 10 MB per file).
const DISCORD_IMAGES = 4;
const DISCORD_IMAGE_BYTES = 8 * 1024 * 1024;
const IMAGE_TYPES = { ".jpg": "image/jpeg", ".jpeg": "image/jpeg", ".png": "image/png", ".webp": "image/webp", ".gif": "image/gif" };
// Minecraft mods: the launcher's rules (core/mod_info.gd, core/release_manifest.gd, core/game_catalog.gd).
const GAME_TYPE = "game";
const MOD_TYPE = "minecraft_mod";
const MOD_MIN_LAUNCHER = "1.4.0";
const MOD_MIN_SCREENSHOTS = 2;
const JAR_RE = /^[0-9A-Za-z][0-9A-Za-z.+_-]{0,123}\.jar$/;
const MAX_MOD_FILES = 4;
const LOADERS = ["fabric", "quilt", "forge", "neoforge"];
const LOADER_NAMES = { fabric: "Fabric", quilt: "Quilt", forge: "Forge", neoforge: "NeoForge" };
const SIDES = ["client", "server", "both"];
const REQUIRE_TYPES = ["loader", "mod", "program"];
const SECTION_IDS = ["first_steps", "commands", "controls", "multiplayer", "server", "known_issues"];
const LINK_HOSTS = ["modrinth.com", "curseforge.com", "fabricmc.net", "github.com", "huggingface.co"];
const MOD_VERSION_RE = /^[0-9A-Za-z][0-9A-Za-z.+_-]{0,39}$/;
const MOD_ID_RE = /^[a-z][a-z0-9_.-]{1,63}$/;
const MOD_LIMITS = { text: 200, name: 48, code: 300, important: 3, requires: 12, steps: 6, links: 4, items: 12, size: 24 };
// Dependencies in fabric.mod.json that "requires" does not need to name.
const BUILT_IN_DEPENDS = ["minecraft", "java", "fabricloader", "quilt_loader"];

const ROOT = path.resolve(__dirname, "..", "..");

function fail(message) {
	console.error("STOPPED: " + message);
	process.exit(1);
}

function parseArgs(argv) {
	const args = { command: argv[0] || "help", suffix: "", readme: false, dryRun: false, again: false, noDiscord: false, version: "", news: "", against: "", config: "tools/dasdimo/dasdimo.json" };
	for (let i = 1; i < argv.length; i++) {
		const key = argv[i];
		if (key === "--readme") args.readme = true;
		else if (key === "--dry-run") args.dryRun = true;
		else if (key === "--again") args.again = true;
		else if (key === "--no-discord") args.noDiscord = true;
		else if (key === "--version") args.version = argv[++i] || "";
		else if (key === "--news") args.news = argv[++i] || "";
		else if (key === "--suffix") args.suffix = argv[++i] || "";
		else if (key === "--against") args.against = argv[++i] || "";
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

function sum(files) {
	return files.reduce((total, file) => total + file.size, 0);
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

function manifestName(cfg) {
	return cfg.manifest_name || "manifest.json";
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
	cfg.type = cfg.type || GAME_TYPE;
	return cfg;
}

function configProblems(cfg) {
	const problems = [];
	if (!ID_RE.test(cfg.game_id || "")) problems.push("game_id: lowercase letters, digits, '-' (2-32 chars)");
	if (typeof cfg.game_name !== "string" || !cfg.game_name.trim()) problems.push("game_name missing");
	if (!REPO_RE.test(cfg.repo || "")) problems.push("repo must look like dasdimo/<name>-releases");
	if (cfg.manifest_name !== undefined && !MANIFEST_NAME_RE.test(cfg.manifest_name)) problems.push("manifest_name: a file name like manifest.json");
	if (![GAME_TYPE, MOD_TYPE].includes(cfg.type)) problems.push(`type: "${GAME_TYPE}" (default) or "${MOD_TYPE}"`);
	if (!isMod(cfg) && cfg.mod !== undefined) problems.push(`mod: only for "type": "${MOD_TYPE}"`);
	if (isMod(cfg)) problems.push(...modFileProblems(cfg), ...modProblems(cfg));
	else if (!Array.isArray(cfg.files) || !cfg.files.length) problems.push("files missing");
	else {
		for (const [i, file] of cfg.files.entries()) {
			const hasPath = typeof file.path === "string" && file.path !== "";
			const hasDir = typeof file.dir === "string" && file.dir !== "";
			if (!ARCHES.includes(file.arch)) problems.push(`files[${i}].arch must be x86_64 or arm64`);
			if (hasPath === hasDir) problems.push(`files[${i}]: either path (one exe) or dir (build folder) + exe`);
			else if (hasPath && !file.path.toLowerCase().endsWith(".exe")) {
				problems.push(`files[${i}].path must end with .exe - for a game made of several files use dir + exe`);
			} else if (hasDir) {
				if (typeof file.exe !== "string" || pathProblem(file.exe) || !file.exe.toLowerCase().endsWith(".exe")) {
					problems.push(`files[${i}].exe: start file inside dir, like "Game.exe" or "bin/Game.exe"`);
				}
				if (file.exclude !== undefined && (!Array.isArray(file.exclude) || file.exclude.some((p) => typeof p !== "string" || !p))) {
					problems.push(`files[${i}].exclude: list of patterns like "*.pdb" or "logs/**"`);
				}
			}
		}
		if (!cfg.files.some((file) => file.arch === "x86_64")) problems.push("an x86_64 build is required");
		if (new Set(cfg.files.map((file) => file.arch)).size !== cfg.files.length) problems.push("one entry per arch");
		const dirs = cfg.files.filter((file) => file.dir);
		if (dirs.length && dirs.length !== cfg.files.length) problems.push("files: all with path or all with dir");
		if (new Set(dirs.map((file) => String(file.exe).toLowerCase())).size > 1) problems.push("files: the same exe for every arch");
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
	if (isMod(cfg)) {
		if (c.engine !== undefined) problems.push("catalog.engine: not for a mod (its page shows Minecraft, loader and Java)");
		if (c.user_data_dirs !== undefined) problems.push("catalog.user_data_dirs: not for a mod - its settings live in the Minecraft profile");
	} else {
		if (typeof c.engine !== "string" || !c.engine.trim()) problems.push("catalog.engine missing (e.g. \"Godot 4.5\")");
		if (!Array.isArray(c.user_data_dirs) || c.user_data_dirs.some((dir) => !DATA_DIR_RE.test(dir))) {
			problems.push("catalog.user_data_dirs: list of folders below %APPDATA%\\, %LOCALAPPDATA%\\ or %USERPROFILE%\\AppData\\LocalLow\\ (may be empty)");
		}
	}
	if (!STATUSES.includes(c.status)) problems.push("catalog.status: testing, released or hidden");
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
		if (isMod(cfg) && shots.length < MOD_MIN_SCREENSHOTS) {
			problems.push(`catalog.screenshots: a mod needs at least ${MOD_MIN_SCREENSHOTS} (its page lives from its pictures)`);
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

// ---------- game files ----------

// "" for a safe relative path like "Game.exe" or "packs/maps.pck", else a short reason.
function pathProblem(file) {
	if (!file) return "empty";
	if (file.length > MAX_PATH_LENGTH) return `longer than ${MAX_PATH_LENGTH} characters`;
	const parts = file.split("/");
	if (parts.length > MAX_DEPTH) return `deeper than ${MAX_DEPTH} folders`;
	for (const part of parts) {
		if (!part || part === "." || part === "..") return "bad folder";
		if (part.startsWith(" ") || part.endsWith(" ") || part.endsWith(".")) return "space or dot at the edge of a name";
		if (/[<>:"\\|?*\x00-\x1f]/.test(part)) return "character not allowed in Windows";
		if (RESERVED_RE.test(part.split(".")[0])) return "name reserved by Windows";
	}
	return "";
}

// "" when the paths fit into one folder: no path twice (Windows ignores case), nothing that is a file and a folder.
function setProblem(paths) {
	const keys = new Set();
	for (const file of paths) {
		if (keys.has(file.toLowerCase())) return `'${file}' twice (case does not count)`;
		keys.add(file.toLowerCase());
	}
	for (const file of paths) {
		const parts = file.split("/");
		for (let depth = 1; depth < parts.length; depth++) {
			const folder = parts.slice(0, depth).join("/");
			if (keys.has(folder.toLowerCase())) return `'${folder}' is a file and a folder`;
		}
	}
	return "";
}

// "*.pdb" matches a name anywhere, "logs/**" a path from the build folder; ignores case.
function excludeRule(pattern) {
	const source = pattern.replace(/\\/g, "/").split(/(\*\*|\*|\?)/).map((part) => {
		if (part === "**") return ".*";
		if (part === "*") return "[^/]*";
		if (part === "?") return "[^/]";
		return part.replace(/[.+^${}()|[\]\\]/g, "\\$&");
	}).join("");
	return { regex: new RegExp(`^${source}$`, "i"), wholePath: pattern.includes("/") };
}

// The builds as [{arch, exe, source, files: [{path, full, size}]}]; problems are collected for "check".
function readBuilds(cfg, baseVersion) {
	const problems = [];
	const format = cfg.files.some((entry) => entry.dir) ? 2 : 1;
	const builds = [];
	for (const entry of cfg.files) {
		if (format === 1) {
			const full = rel(entry.path.replace(/\{version\}/g, baseVersion));
			const name = path.basename(full);
			if (!fs.existsSync(full)) problems.push("build missing: " + path.relative(ROOT, full) + " - build the game first");
			else if (!NAME_RE.test(name)) problems.push(`file name '${name}': only letters, digits, . _ - and ending .exe`);
			else builds.push({ arch: entry.arch, exe: name, source: full, files: [{ path: name, full, size: fs.statSync(full).size }] });
			continue;
		}
		const source = rel(entry.dir.replace(/\{version\}/g, baseVersion));
		if (!fs.existsSync(source) || !fs.statSync(source).isDirectory()) {
			problems.push("build folder missing: " + path.relative(ROOT, source) + " - build the game first");
			continue;
		}
		const rules = (entry.exclude || []).map(excludeRule);
		const skip = (relative) => rules.some((rule) => rule.regex.test(rule.wholePath ? relative : path.posix.basename(relative)));
		const files = [];
		const walk = (folder, prefix) => {
			const items = fs.readdirSync(folder, { withFileTypes: true }).sort((a, b) => (a.name < b.name ? -1 : a.name > b.name ? 1 : 0));
			for (const item of items) {
				const relative = prefix + item.name;
				const full = path.join(folder, item.name);
				if (skip(relative)) continue;
				if (item.isSymbolicLink()) problems.push(`${entry.arch} ${relative}: links are not supported`);
				else if (item.isDirectory()) walk(full, relative + "/");
				else if (item.isFile()) {
					const problem = pathProblem(relative);
					const size = fs.statSync(full).size;
					if (problem) problems.push(`${entry.arch} ${relative}: ${problem} - rename it or add it to exclude`);
					else if (!size) problems.push(`${entry.arch} ${relative}: empty file - remove it or add it to exclude`);
					else files.push({ path: relative, full, size });
				}
			}
		};
		walk(source, "");
		const exe = files.find((file) => file.path.toLowerCase() === entry.exe.toLowerCase());
		if (!exe) problems.push(`${entry.arch}: start file ${entry.exe} not in ${entry.dir}`);
		const set = setProblem(files.map((file) => file.path));
		if (set) problems.push(`${entry.arch}: ${set}`);
		builds.push({ arch: entry.arch, exe: exe ? exe.path : entry.exe, source, files });
	}
	return { format, builds, problems };
}

// Format 2 file list: a file with the same path and content in both builds is listed once as arch "all".
function gameFiles(builds, hash) {
	const entries = [];
	for (const build of builds) {
		for (const file of build.files) entries.push({ path: file.path, arch: build.arch, size: file.size, sha256: hash(file.full), full: file.full });
	}
	if (builds.length > 1) {
		const same = (file) => `${file.path.toLowerCase()}\n${file.sha256}`;
		const counts = new Map();
		for (const file of entries) counts.set(same(file), (counts.get(same(file)) || 0) + 1);
		const seen = new Set();
		for (let i = entries.length - 1; i >= 0; i--) {
			if (counts.get(same(entries[i])) < builds.length) continue;
			if (seen.has(same(entries[i]))) entries.splice(i, 1);
			else {
				seen.add(same(entries[i]));
				entries[i].arch = "all";
			}
		}
	}
	return entries.sort((a, b) => (a.path.toLowerCase() < b.path.toLowerCase() ? -1 : a.path.toLowerCase() > b.path.toLowerCase() ? 1
		: ARCHES.indexOf(a.arch) - ARCHES.indexOf(b.arch)));
}

// The files a PC gets, like the launcher (core/release_manifest.gd files_for): ARM64 falls back to x86_64.
function filesFor(manifest, arch) {
	const files = Array.isArray(manifest.files) ? manifest.files : [];
	const tries = arch === "arm64" ? ["arm64", "x86_64"] : [arch];
	if ((manifest.format || 1) < 2) {
		for (const wanted of tries) {
			const file = files.find((f) => f.kind === "game" && f.arch === wanted);
			if (file) return [{ path: file.name, size: file.size, sha256: String(file.sha256).toLowerCase() }];
		}
		return [];
	}
	const offers = (wanted) => files.some((f) => f.arch === wanted)
		|| (wanted === "x86_64" && files.length > 0 && files.every((f) => f.arch === "all"));
	const used = tries.find(offers);
	if (!used) return [];
	return files.filter((f) => f.arch === used || f.arch === "all")
		.map((f) => ({ path: f.path, size: f.size, sha256: String(f.sha256).toLowerCase() }));
}

// What turns the installed files into the wanted ones (the launcher's GameFiles.diff).
function diffFiles(installed, wanted) {
	const have = new Map(installed.map((file) => [file.path.toLowerCase(), file]));
	const keys = new Set(wanted.map((file) => file.path.toLowerCase()));
	const download = [];
	for (const file of wanted) {
		const old = have.get(file.path.toLowerCase());
		if (!old || old.sha256 !== file.sha256 || old.size !== file.size) download.push({ ...file, change: old ? "changed" : "new" });
	}
	const remove = installed.filter((file) => !keys.has(file.path.toLowerCase())).map((file) => file.path);
	return { download, remove, downloadBytes: sum(download), totalBytes: sum(wanted) };
}

// ---------- Minecraft mods ----------

function isObject(value) {
	return value !== null && typeof value === "object" && !Array.isArray(value);
}

function isMod(cfg) {
	return cfg.type === MOD_TYPE;
}

// The id in the mod's fabric.mod.json: "mod.mod_id", else the game_id.
function modId(cfg) {
	const mod = isObject(cfg.mod) ? cfg.mod : {};
	return String(mod.mod_id === undefined ? cfg.game_id : mod.mod_id);
}

// A mod's files: [{jar, loader}], one .jar per loader, the same for every PC.
function modFileProblems(cfg) {
	const example = '{"jar": "build/libs/mymod-{version}.jar", "loader": "fabric"}';
	if (!Array.isArray(cfg.files) || !cfg.files.length || cfg.files.length > MAX_MOD_FILES) {
		return [`files: 1-${MAX_MOD_FILES} entries like ${example}`];
	}
	const problems = [];
	for (const [i, file] of cfg.files.entries()) {
		if (!isObject(file)) {
			problems.push(`files[${i}]: ${example}`);
			continue;
		}
		if (typeof file.jar !== "string" || !file.jar.toLowerCase().endsWith(".jar")) {
			problems.push(`files[${i}].jar: the built mod, like "build/libs/mymod-{version}.jar" (not -sources.jar)`);
		}
		if (!LOADERS.includes(file.loader)) problems.push(`files[${i}].loader: ${LOADERS.join(", ")}`);
		for (const key of Object.keys(file)) {
			if (!["jar", "loader"].includes(key)) problems.push(`files[${i}].${key}: not for a mod (a .jar is the same on every PC)`);
		}
	}
	if (new Set(cfg.files.map((file) => isObject(file) && file.loader)).size !== cfg.files.length) problems.push("files: one .jar per loader");
	return problems;
}

// {de, en} with 1-limit characters each; a name may also be one text for both languages.
function textsProblem(value, where, limit, plain = false) {
	if (plain && typeof value === "string") return value.trim() && value.trim().length <= limit ? "" : `${where}: 1-${limit} characters`;
	if (!isObject(value)) return `${where}: {"de": "...", "en": "..."}`;
	const extra = Object.keys(value).find((key) => !["de", "en"].includes(key));
	if (extra) return `${where}.${extra}: only de and en`;
	const lang = ["de", "en"].find((key) => typeof value[key] !== "string" || !value[key].trim() || value[key].trim().length > limit);
	return lang ? `${where}.${lang}: 1-${limit} characters` : "";
}

// A step or a line of a section: {de, en}, {code} (a command, shown with a copy button) or both.
function stepProblem(value, where) {
	if (!isObject(value)) return `${where}: {"de": "...", "en": "...", "code": "..."}`;
	const { code, ...texts } = value;
	if (code !== undefined && (typeof code !== "string" || !code.trim() || code.trim().length > MOD_LIMITS.code)) {
		return `${where}.code: 1-${MOD_LIMITS.code} characters`;
	}
	return code !== undefined && !Object.keys(texts).length ? "" : textsProblem(texts, where, MOD_LIMITS.text);
}

function linkProblem(url, where) {
	return allowedLink(url) ? "" : `${where}: an https link to ${LINK_HOSTS.join(", ")} (the launcher shows no others)`;
}

// The launcher's rule (core/mod_info.gd is_allowed_link): https, one of LINK_HOSTS or a subdomain, no user, no port.
function allowedLink(url) {
	if (typeof url !== "string" || !url.startsWith("https://") || url.length > 300 || /[\s\\]/.test(url)) return false;
	const host = url.slice(8).split(/[/?#]/)[0].toLowerCase();
	if (!host || host.includes("@") || host.includes(":")) return false;
	return LINK_HOSTS.some((allowed) => host === allowed || host.endsWith("." + allowed));
}

// The texts of the mod's page ("mod" in dasdimo.json). The launcher cuts or leaves out what breaks its rules
// (core/mod_info.gd) - the helper stops instead, so nothing goes missing unnoticed.
function modProblems(cfg) {
	const mod = cfg.mod;
	if (!isObject(mod)) return ['mod missing: the texts of the mod page {"minecraft": {...}, "requires": [...], ...} (INTEGRATION.md, "Minecraft-Mods")'];
	const problems = [];
	const add = (problem) => {
		if (problem) problems.push(problem);
	};
	const keys = (object, allowed, where) => {
		for (const key of Object.keys(object)) if (!allowed.includes(key)) add(`${where}.${key}: unknown (${allowed.join(", ")})`);
	};
	const list = (value, limit, where, each) => {
		if (value === undefined) return;
		if (!Array.isArray(value) || value.length > limit) return add(`${where}: a list of at most ${limit}`);
		value.forEach((item, i) => each(item, `${where}[${i}]`));
	};
	keys(mod, ["mod_id", "minecraft", "important", "requires", "sections", "guide_url"], "mod");
	if (!MOD_ID_RE.test(modId(cfg))) add("mod.mod_id: the id in fabric.mod.json (a-z 0-9 _ . -, starts with a letter) - needed when it is not the game_id");
	const mc = mod.minecraft;
	if (!isObject(mc)) add('mod.minecraft: {"version": "26.3", "loader": "fabric", "loader_version": "0.20.1", "java": "25", "side": "both"}');
	else {
		keys(mc, ["version", "loader", "loader_version", "java", "side"], "mod.minecraft");
		if (!MOD_VERSION_RE.test(mc.version || "")) add('mod.minecraft.version: like "26.3"');
		if (!LOADERS.includes(mc.loader)) add(`mod.minecraft.loader: ${LOADERS.join(", ")}`);
		else if (Array.isArray(cfg.files) && !cfg.files.some((file) => isObject(file) && file.loader === mc.loader)) {
			add(`mod.minecraft.loader: no .jar for ${mc.loader} in files`);
		}
		if (mc.loader_version !== undefined && !MOD_VERSION_RE.test(mc.loader_version)) add('mod.minecraft.loader_version: like "0.20.1" (or leave it out)');
		if (mc.java !== undefined && !/^[0-9]{1,3}$/.test(mc.java)) add('mod.minecraft.java: like "25" (or leave it out)');
		if (mc.side !== undefined && !SIDES.includes(mc.side)) add(`mod.minecraft.side: ${SIDES.join(", ")} (default both)`);
	}
	list(mod.important, MOD_LIMITS.important, "mod.important", (item, where) => add(textsProblem(item, where, MOD_LIMITS.text)));
	list(mod.requires, MOD_LIMITS.requires, "mod.requires", (need, where) => {
		if (!isObject(need)) return add(`${where}: {"type": "mod", "name": "...", "mod_id": "..."}`);
		keys(need, ["type", "name", "mod_id", "min_version", "optional", "url", "why", "without", "check", "alternative", "size", "links", "steps"], where);
		if (!REQUIRE_TYPES.includes(need.type)) add(`${where}.type: ${REQUIRE_TYPES.join(", ")}`);
		add(textsProblem(need.name, where + ".name", MOD_LIMITS.name, true));
		if (need.type === "mod" && !MOD_ID_RE.test(need.mod_id || "")) add(`${where}.mod_id: the id in that mod's fabric.mod.json, like "fabric-api"`);
		if (need.type === "loader" && need.mod_id !== undefined && !MOD_ID_RE.test(need.mod_id)) add(`${where}.mod_id: like "fabricloader" (or leave it out)`);
		if (need.type === "program" && need.mod_id !== undefined) add(`${where}.mod_id: a program has none (the launcher cannot check it)`);
		if (need.min_version !== undefined && !MOD_VERSION_RE.test(need.min_version)) add(`${where}.min_version: like "0.165.0+26.3"`);
		if (need.optional !== undefined && typeof need.optional !== "boolean") add(`${where}.optional: true or false`);
		if (need.type === "loader" && need.optional) add(`${where}.optional: the loader is always needed`);
		if (need.url !== undefined) add(linkProblem(need.url, where + ".url"));
		for (const key of ["why", "without", "check", "alternative"]) {
			if (need[key] !== undefined) add(textsProblem(need[key], `${where}.${key}`, MOD_LIMITS.text));
		}
		if (need.size !== undefined && (typeof need.size !== "string" || !need.size.trim() || need.size.trim().length > MOD_LIMITS.size)) {
			add(`${where}.size: like "0,5-1,6 GB" (at most ${MOD_LIMITS.size} characters)`);
		}
		list(need.links, MOD_LIMITS.links, where + ".links", (link, at) => {
			if (!isObject(link)) return add(`${at}: {"label": {"de": "...", "en": "..."}, "url": "https://..."}`);
			keys(link, ["label", "url"], at);
			add(textsProblem(link.label, at + ".label", MOD_LIMITS.name, true));
			add(linkProblem(link.url, at + ".url"));
		});
		list(need.steps, MOD_LIMITS.steps, where + ".steps", (step, at) => add(stepProblem(step, at)));
	});
	const seen = new Set();
	list(mod.sections, SECTION_IDS.length, "mod.sections", (section, where) => {
		if (!isObject(section)) return add(`${where}: {"id": "commands", "items": [...]}`);
		keys(section, ["id", "items"], where);
		if (!SECTION_IDS.includes(section.id)) add(`${where}.id: ${SECTION_IDS.join(", ")}`);
		else if (seen.has(section.id)) add(`${where}.id: ${section.id} twice`);
		seen.add(section.id);
		if (!Array.isArray(section.items) || !section.items.length) add(`${where}.items: 1-${MOD_LIMITS.items} lines`);
		else list(section.items, MOD_LIMITS.items, where + ".items", (step, at) => add(stepProblem(step, at)));
	});
	if (mod.guide_url !== undefined) add(linkProblem(mod.guide_url, "mod.guide_url"));
	return problems;
}

// One file of a zip (a .jar) by its exact name, null when it is not there or broken. Reads only the zip's directory
// (at the end) and that entry.
function zipEntry(buf, wanted) {
	let end = -1;
	for (let i = buf.length - 22; i >= Math.max(0, buf.length - 22 - 0xffff); i--) {
		if (buf.readUInt32LE(i) === 0x06054b50) {
			end = i;
			break;
		}
	}
	if (end < 0) return null;
	const count = buf.readUInt16LE(end + 10);
	let at = buf.readUInt32LE(end + 16);
	for (let n = 0; n < count && at + 46 <= buf.length && buf.readUInt32LE(at) === 0x02014b50; n++) {
		const nameLength = buf.readUInt16LE(at + 28);
		if (buf.toString("utf8", at + 46, at + 46 + nameLength) === wanted) {
			const method = buf.readUInt16LE(at + 10);
			const packed = buf.readUInt32LE(at + 20);
			const local = buf.readUInt32LE(at + 42);
			if (local + 30 > buf.length || buf.readUInt32LE(local) !== 0x04034b50) return null;
			const start = local + 30 + buf.readUInt16LE(local + 26) + buf.readUInt16LE(local + 28);
			const data = buf.subarray(start, start + packed);
			try {
				if (method === 0) return data;
				if (method === 8) return zlib.inflateRawSync(data);
			} catch (error) {
				return null;
			}
			return null;
		}
		at += 46 + nameLength + buf.readUInt16LE(at + 30) + buf.readUInt16LE(at + 32);
	}
	return null;
}

// What the .jar says about itself, like the launcher (core/jar_info.gd): fabric.mod.json, else quilt.mod.json.
// {file, id, version, depends: {id: versions}}, null for none (a Forge mod, the wrong .jar).
function jarInfo(full) {
	let buf;
	try {
		buf = fs.readFileSync(full);
	} catch (error) {
		return null;
	}
	const json = (name) => {
		const data = zipEntry(buf, name);
		try {
			return data ? JSON.parse(data.toString("utf8").replace(/^﻿/, "")) : null;
		} catch (error) {
			return null;
		}
	};
	const fabric = json("fabric.mod.json");
	if (isObject(fabric) && typeof fabric.id === "string" && fabric.id.trim()) {
		return { file: "fabric.mod.json", id: fabric.id.trim().toLowerCase(), version: String(fabric.version || "").trim(),
			depends: isObject(fabric.depends) ? fabric.depends : {} };
	}
	const quilt = json("quilt.mod.json");
	const loader = isObject(quilt) && isObject(quilt.quilt_loader) ? quilt.quilt_loader : null;
	if (loader && typeof loader.id === "string" && loader.id.trim()) {
		const depends = {};
		for (const item of Array.isArray(loader.depends) ? loader.depends : []) {
			if (typeof item === "string") depends[item] = "*";
			else if (isObject(item) && typeof item.id === "string") depends[item.id] = item.versions || "*";
		}
		return { file: "quilt.mod.json", id: loader.id.trim().toLowerCase(), version: String(loader.version || "").trim(), depends };
	}
	return null;
}

// The .jar files as builds [{arch: "all", loader, source, files: [{path, full, size}], info}].
function readJars(cfg, baseVersion) {
	const problems = [];
	const builds = [];
	for (const entry of cfg.files) {
		const full = rel(entry.jar.replace(/\{version\}/g, baseVersion));
		const name = path.basename(full);
		if (!fs.existsSync(full)) {
			problems.push("mod missing: " + path.relative(ROOT, full) + " - build the mod first (gradlew build)");
			continue;
		}
		if (!JAR_RE.test(name)) {
			problems.push(`file name '${name}': only letters, digits, . + _ - and ending .jar`);
			continue;
		}
		const info = jarInfo(full);
		if (["fabric", "quilt"].includes(entry.loader)) {
			if (!info) problems.push(`${name}: no fabric.mod.json inside - the right .jar (not -sources.jar or -dev.jar)?`);
			else if (info.id !== modId(cfg)) {
				problems.push(`${name}: ${info.file} says id '${info.id}', the launcher looks for '${modId(cfg)}' (set mod.mod_id) - it finds old files of the mod by this id`);
			}
		}
		builds.push({ arch: "all", loader: entry.loader, source: full, files: [{ path: name, full, size: fs.statSync(full).size }], info });
	}
	if (new Set(builds.map((build) => build.files[0].path.toLowerCase())).size !== builds.length) problems.push("files: every loader needs its own .jar name");
	return { format: 2, builds, problems };
}

// Hints from the .jar's own fabric.mod.json - no problem, but worth a look.
function jarNotes(cfg, build, version) {
	const notes = [];
	const name = build.files[0].path;
	if (!build.info) {
		notes.push(`${name}: the launcher reads only fabric.mod.json / quilt.mod.json - it cannot see old files of a ${LOADER_NAMES[build.loader]} mod`);
		return notes;
	}
	if (build.info.version !== version) notes.push(`${name}: ${build.info.file} says version '${build.info.version}', the release is ${version}`);
	const named = new Set((Array.isArray(cfg.mod.requires) ? cfg.mod.requires : []).filter(isObject).map((need) => need.mod_id));
	for (const id of Object.keys(build.info.depends)) {
		if (BUILT_IN_DEPENDS.includes(id) || named.has(id) || (id === "fabric" && named.has("fabric-api"))) continue;
		notes.push(`${name}: ${build.info.file} depends on '${id}', mod.requires does not name it - players would not know they need it`);
	}
	return notes;
}

// The page texts for the manifest, as in dasdimo.json.
function modPage(cfg) {
	const page = { mod_id: modId(cfg), minecraft: cfg.mod.minecraft };
	for (const key of ["important", "requires", "sections", "guide_url"]) {
		if (cfg.mod[key] !== undefined) page[key] = cfg.mod[key];
	}
	return page;
}

// "Minecraft 26.3", "Fabric Loader ≥ 0.20.1", "Java 25", then the needed mods and programs (release notes).
function modNeeds(manifest, lang) {
	const mc = manifest.minecraft;
	const name = (value) => (typeof value === "string" ? value : value[lang] || value.en || value.de);
	const requires = Array.isArray(manifest.requires) ? manifest.requires : [];
	const loader = requires.find((need) => need.type === "loader");
	const loaderName = loader ? name(loader.name) : `${LOADER_NAMES[mc.loader]} Loader`;
	const loaderVersion = (loader && loader.min_version) || mc.loader_version;
	const needs = [`Minecraft ${mc.version}`, loaderVersion ? `${loaderName} ≥ ${loaderVersion}` : loaderName];
	if (mc.java) needs.push(`Java ${mc.java}`);
	for (const need of requires) {
		if (need.type !== "loader" && !need.optional) needs.push(need.min_version ? `${name(need.name)} ≥ ${need.min_version}` : name(need.name));
	}
	return needs;
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
	// A game manifest works too (its "news").
	if (list && !Array.isArray(list) && Array.isArray(list.news)) list = list.news;
	if (!Array.isArray(list)) return { news: [], problems: [cfg.news_file + " must be a list"] };
	const problems = [];
	const groups = [];
	const images = {};
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
		// Pictures for the Discord post only, never in the manifest.
		if (block.images !== undefined && !Array.isArray(block.images)) problems.push(`news ${block.version}: images must be a list of picture files`);
		for (const file of Array.isArray(block.images) ? block.images : []) {
			if (typeof file === "string" && file.trim()) (images[block.version] = images[block.version] || []).push(file.trim());
		}
	}
	return { news: groups, problems, images };
}

// ---------- manifest ----------

// Hashes the builds and makes the manifest (nothing written). assets = the release files [{name, full, size, sha256}].
function makeRelease(cfg, suffix) {
	const baseVersion = readVersion(cfg);
	const version = suffix ? `${baseVersion}-${suffix}` : baseVersion;
	if (!VERSION_RE.test(version)) fail(`invalid version '${version}'`);
	const tag = "v" + version;
	const mod = isMod(cfg);
	const { format, builds, problems } = mod ? readJars(cfg, baseVersion) : readBuilds(cfg, baseVersion);
	const result = { version, tag, format, builds, manifest: null, assets: [], problems, mod };
	if (problems.length) return result;
	const hashes = new Map();
	const hash = (full) => {
		if (!hashes.has(full)) hashes.set(full, sha256File(full));
		return hashes.get(full);
	};
	const assets = new Map();
	const addAsset = (name, file) => {
		const old = assets.get(name.toLowerCase());
		if (old && old.sha256 !== file.sha256) problems.push(`two different files would be called ${name} on GitHub`);
		else if (!old) assets.set(name.toLowerCase(), { name, full: file.full, size: file.size, sha256: file.sha256 });
		return `https://github.com/${cfg.repo}/releases/download/${tag}/${encodeURIComponent(name)}`;
	};
	let files;
	if (mod) {
		// The .jar keeps its name: it goes into a mods folder as it is.
		files = builds.map((build) => {
			const file = { ...build.files[0], sha256: hash(build.files[0].full) };
			return { kind: "mod", loader: build.loader, path: file.path, arch: "all", size: file.size, sha256: file.sha256, url: addAsset(file.path, file) };
		});
	} else if (format === 1) {
		files = builds.map((build) => {
			const file = { ...build.files[0], sha256: hash(build.files[0].full) };
			return { kind: "game", arch: build.arch, name: file.path, size: file.size, sha256: file.sha256, url: addAsset(file.path, file) };
		});
	} else {
		files = gameFiles(builds, hash).map((file) => {
			const name = file.sha256.slice(0, 12) + "-" + path.posix.basename(file.path).replace(/[^0-9A-Za-z._-]/g, "_");
			return { path: file.path, arch: file.arch, size: file.size, sha256: file.sha256, url: addAsset(name, file) };
		});
		for (const build of builds) {
			if (!files.some((file) => file.arch === build.arch)) {
				problems.push(`${build.arch}: no file of its own (usually the exe) - with the same files for both, use only x86_64`);
			}
		}
		if (files.length > MAX_FILES) problems.push(`${files.length} files - the launcher takes at most ${MAX_FILES}`);
	}
	if (assets.size > MAX_ASSETS) problems.push(`${assets.size} release files - at most ${MAX_ASSETS} (GitHub limit 1000): put small files into packs`);
	const manifest = { format, game: cfg.game_name, game_id: cfg.game_id, repo: cfg.repo, version, tag, date: today(), launcher_version: "" };
	if (mod) Object.assign(manifest, { min_launcher: MOD_MIN_LAUNCHER }, modPage(cfg));
	else if (format === 2) manifest.exe = builds[0].exe;
	manifest.files = files;
	manifest.news = readNews(cfg, version).news;
	return Object.assign(result, { manifest, assets: [...assets.values()] });
}

function printSummary(release) {
	const files = release.manifest.files;
	if (release.mod) {
		for (const file of files) console.log(`${file.loader.padEnd(8)} ${megabytes(file.size).padStart(7)} MB  ${file.sha256}  ${file.path}`);
		return;
	}
	if (release.format === 1 || files.length <= LIST_LINES) {
		for (const file of files) {
			console.log(`${file.arch.padEnd(6)} ${megabytes(file.size).padStart(7)} MB  ${file.sha256}  ${file.path || file.name}`);
		}
	}
	if (release.format === 2) {
		for (const build of release.builds) {
			const set = filesFor(release.manifest, build.arch);
			console.log(`${build.arch}: ${set.length} file(s), ${megabytes(sum(set))} MB, start file ${release.manifest.exe}`);
		}
		console.log(`${files.filter((file) => file.arch === "all").length} file(s) shared by both builds, ${release.assets.length} release file(s)`);
	}
}

function buildRelease(cfg, suffix) {
	const release = makeRelease(cfg, suffix);
	if (release.problems.length) fail(release.problems.join("; "));
	const out = rel(path.join(cfg.out_dir, release.tag));
	fs.mkdirSync(out, { recursive: true });
	const manifestPath = path.join(out, manifestName(cfg));
	const notesPath = path.join(out, "notes.md");
	fs.writeFileSync(manifestPath, JSON.stringify(release.manifest, null, "\t") + "\n", "utf8");
	fs.writeFileSync(notesPath, releaseNotes(cfg, release.manifest, Boolean(suffix)), "utf8");
	printSummary(release);
	console.log("news: " + (release.manifest.news.map((g) => `${g.version} (${g.entries.length})`).join(", ") || "none"));
	console.log("written: " + path.relative(ROOT, manifestPath));
	return Object.assign(release, { manifestPath, notesPath });
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
		if (manifest.minecraft) {
			lines.push(de
				? `**So bekommst du es:** im [DasDimo Launcher](${LAUNCHER_PAGE}) – er legt die Mod in dein Minecraft-Profil, hält sie aktuell und zeigt, was du sonst noch brauchst.`
				: `**How to get it:** in the [DasDimo Launcher](${LAUNCHER_PAGE}) – it puts the mod into your Minecraft profile, keeps it up to date and shows what else you need.`, "");
			lines.push(`**${de ? "Braucht" : "Needs"}:** ${modNeeds(manifest, lang).join(" · ")}`, "");
		} else {
			lines.push(de
				? `**So bekommst du es:** im [DasDimo Launcher](${LAUNCHER_PAGE}) – dort installieren, aktualisieren und spielen.`
				: `**How to get it:** in the [DasDimo Launcher](${LAUNCHER_PAGE}) – install, update and play there.`, "");
		}
		if (news.entries.length) {
			lines.push(de ? "### Neu" : "### New", "");
			for (const entry of news.entries) lines.push("- " + entry[lang]);
			lines.push("");
		}
		lines.push("---", "");
	}
	if (manifest.minecraft) {
		lines.push("| Datei / file | Loader | Minecraft | MB | SHA-256 |", "|---|---|---|---|---|");
		for (const file of manifest.files) {
			lines.push(`| ${file.path} | ${file.loader} | ${manifest.minecraft.version} | ${megabytes(file.size)} | \`${file.sha256}\` |`);
		}
	} else if (manifest.format >= 2) {
		lines.push("| Build | Dateien / files | MB |", "|---|---|---|");
		for (const arch of ARCHES) {
			if (arch === "arm64" && !manifest.files.some((file) => file.arch === "arm64")) continue;
			const set = filesFor(manifest, arch);
			lines.push(`| ${arch} | ${set.length} | ${megabytes(sum(set))} |`);
		}
	} else {
		lines.push("| Datei / file | MB | SHA-256 |", "|---|---|---|");
		for (const file of manifest.files) lines.push(`| ${file.name} | ${megabytes(file.size)} | \`${file.sha256}\` |`);
	}
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

function sameOnGitHub(assets, name, sha256) {
	const asset = assets.find((a) => a.name === name);
	return Boolean(asset) && String(asset.digest || "") === "sha256:" + sha256;
}

// GitHub names a release file after the local file: format 2 files are linked (or copied) under their release name
// (a mod's .jar keeps its own).
function stageAssets(release) {
	if (release.format < 2 || release.mod) return release.assets.map((asset) => ({ ...asset, file: asset.full }));
	const dir = path.join(path.dirname(release.manifestPath), "files");
	fs.rmSync(dir, { recursive: true, force: true });
	fs.mkdirSync(dir, { recursive: true });
	return release.assets.map((asset) => {
		const file = path.join(dir, asset.name);
		try {
			fs.linkSync(asset.full, file);
		} catch (error) {
			fs.copyFileSync(asset.full, file);
		}
		return { ...asset, file };
	});
}

// The manifest of the latest release ({manifest, label}), or null when there is none yet.
function latestManifest(cfg, against) {
	let file = against ? rel(against) : "";
	let tmp = "";
	if (!file) {
		tmp = fs.mkdtempSync(path.join(os.tmpdir(), "dasdimo_diff_"));
		const result = gh(["release", "download", "-R", cfg.repo, "-p", manifestName(cfg), "-D", tmp], { allowFail: true });
		if (!result.ok) {
			fs.rmSync(tmp, { recursive: true, force: true });
			if (/release not found|no assets match|Not Found|HTTP 404/i.test(result.err)) return null;
			fail("reading the latest release failed: " + result.err.trim());
		}
		file = path.join(tmp, manifestName(cfg));
	}
	const text = fs.existsSync(file) ? fs.readFileSync(file, "utf8") : "";
	if (tmp) fs.rmSync(tmp, { recursive: true, force: true });
	if (!text) fail("manifest not found: " + file);
	let manifest;
	try {
		manifest = JSON.parse(text);
	} catch (error) {
		fail(path.basename(file) + " is not valid JSON");
	}
	return { manifest, label: `${manifest.tag || manifest.version} (${path.basename(file)}, format ${manifest.format || 1})` };
}

// ---------- Discord ----------

// %APPDATA%\DasDimo on the user's PC (DASDIMO_SETTINGS_DIR: another folder, for tests).
function settingsDir() {
	return process.env.DASDIMO_SETTINGS_DIR || path.join(process.env.APPDATA || path.join(os.homedir(), "AppData", "Roaming"), "DasDimo");
}

function readSettings(name) {
	const file = path.join(settingsDir(), name);
	if (!fs.existsSync(file)) return {};
	let data;
	try {
		data = JSON.parse(fs.readFileSync(file, "utf8").replace(/^﻿/, ""));
	} catch (error) {
		throw new Error(file + " is not valid JSON");
	}
	return data && typeof data === "object" && !Array.isArray(data) ? data : {};
}

// Messages and errors never show a webhook address.
function hideWebhook(text) {
	return String(text).replace(/webhooks\/\d+\/[A-Za-z0-9_-]+/g, "webhooks/<hidden>");
}

// The German news and the listed pictures of a version (the exact version, else its base version: 1.2.0-test1 -> 1.2.0).
function discordNews(cfg, version) {
	const { news, images = {} } = readNews(cfg, version);
	const group = news.find((g) => g.version === version) || news.find((g) => g.version === version.split("-")[0]);
	if (!group) return { entries: [], images: [] };
	return { entries: group.entries.map((entry) => entry.de), images: images[group.version] || [] };
}

// The picture files of a post: the version's "images" from the news, else the game's cover and screenshots.
// At most DISCORD_IMAGES, together at most DISCORD_IMAGE_BYTES; the others are left out with a note.
function discordImages(cfg, listed) {
	const c = cfg.catalog || {};
	const wanted = listed.length ? listed : [c.cover, ...(c.screenshots || [])].filter(Boolean).slice(0, DISCORD_IMAGES);
	const files = [];
	const problems = [];
	let total = 0;
	for (const file of wanted) {
		const full = rel(file);
		const type = IMAGE_TYPES[path.extname(full).toLowerCase()];
		const size = fs.existsSync(full) && fs.statSync(full).isFile() ? fs.statSync(full).size : -1;
		if (files.length >= DISCORD_IMAGES) problems.push(`picture left out, at most ${DISCORD_IMAGES} per post: ${file}`);
		else if (!type) problems.push(`picture must be .jpg, .png, .webp or .gif: ${file}`);
		else if (size < 0) problems.push(`picture missing: ${file}`);
		else if (total + size > DISCORD_IMAGE_BYTES) problems.push(`picture left out, all pictures together at most 8 MB: ${file}`);
		else {
			total += size;
			files.push({ name: `${files.length + 1}-${path.basename(full).replace(/[^0-9A-Za-z._-]/g, "_")}`, full, type, size });
		}
	}
	return { files, problems };
}

function discordColor(cfg) {
	const accent = String(cfg.catalog.accent || "");
	return /^#[0-9A-Fa-f]{6}$/.test(accent) ? parseInt(accent.slice(1), 16) : DISCORD_COLOR;
}

// The forum post of a version: a new post "<title>", the German news as a list, then where to get it, the pictures
// (attachment names) under the text. Longer news continue in further messages of the same post.
// tools/discord/discord_post.ps1 of the launcher builds the same.
function discordMessages(title, version, entries, color, { images = [], limit = DISCORD_LIMIT } = {}) {
	const pieces = [];
	for (const entry of entries.length ? entries : ["Kleine Verbesserungen und Fehlerbehebungen."]) {
		let line = "- " + entry;
		while (line.length > limit) {
			let cut = line.lastIndexOf(" ", limit);
			if (cut < limit / 2) cut = limit;
			pieces.push(line.slice(0, cut));
			line = line.slice(cut).trimStart();
		}
		if (line) pieces.push(line);
	}
	const chunks = [""];
	const add = (text, gap) => {
		const last = chunks[chunks.length - 1];
		if (!last) chunks[chunks.length - 1] = text;
		else if (last.length + gap.length + text.length <= limit) chunks[chunks.length - 1] = last + gap + text;
		else chunks.push(text);
	};
	for (const piece of pieces) add(piece, "\n");
	add(`**Aktualisiere im DasDimo Launcher**\n${LAUNCHER_PAGE}`, "\n\n");
	return chunks.map((description, i) => {
		// No pings: allowed_mentions with an empty "parse" ignores @everyone, @here and names in the news.
		const message = { embeds: [{ description, color }], allowed_mentions: { parse: [] } };
		if (i) return message;
		if (images.length) {
			// Embeds with the same url: Discord shows their pictures as one gallery in the box under the text.
			Object.assign(message.embeds[0], { url: LAUNCHER_PAGE, image: { url: "attachment://" + images[0] } });
			for (const name of images.slice(1)) message.embeds.push({ url: LAUNCHER_PAGE, image: { url: "attachment://" + name }, color });
			message.attachments = images.map((filename, id) => ({ id, filename }));
		}
		return { thread_name: title.slice(0, DISCORD_TITLE_LIMIT), content: `Neue Version **${version}** ist da. Das ist neu:`, ...message };
	});
}

// One webhook call, Discord's answer as JSON; files ({name, full, type}) go along as attachments.
// Waits and tries again when Discord says "too many requests".
async function discordCall(url, body, fetchImpl, files = []) {
	const headers = { "User-Agent": "DasDimo-release-helper/" + HELPER_VERSION };
	let payload;
	if (files.length) {
		payload = new FormData();
		payload.append("payload_json", JSON.stringify(body));
		files.forEach((file, i) => payload.append(`files[${i}]`, new Blob([fs.readFileSync(file.full)], { type: file.type }), file.name));
	} else if (body) {
		headers["Content-Type"] = "application/json";
		payload = JSON.stringify(body);
	}
	for (let attempt = 0; ; attempt++) {
		let response;
		try {
			response = await fetchImpl(url, { method: body ? "POST" : "GET", headers, body: payload });
		} catch (error) {
			throw new Error(hideWebhook("Discord not reachable: " + error.message));
		}
		const text = await response.text();
		if (response.status === 429 && attempt < 3) {
			let wait = 2;
			try {
				wait = Number(JSON.parse(text).retry_after) || 2;
			} catch (error) {
				// keep 2 s
			}
			await new Promise((done) => setTimeout(done, Math.min(wait, 10) * 1000));
			continue;
		}
		if (!response.ok) throw new Error(hideWebhook(`Discord HTTP ${response.status}: ${text.slice(0, 300)}`));
		return text ? JSON.parse(text) : {};
	}
}

// Creates the post with the first message and its pictures (onCreated gets {thread, link} right then), adds the
// other messages to it.
async function postDiscord(webhook, messages, { files = [], onCreated = () => {}, fetchImpl = fetch } = {}) {
	const hook = await discordCall(webhook, null, fetchImpl);
	const first = await discordCall(webhook + "?wait=true", messages[0], fetchImpl, files);
	const thread = String(first.channel_id || "");
	if (!/^\d+$/.test(thread)) throw new Error("Discord did not create a forum post (is the webhook in a forum?)");
	const post = { thread, link: `https://discord.com/channels/${hook.guild_id}/${thread}` };
	onCreated(post);
	for (let i = 1; i < messages.length; i++) {
		try {
			await discordCall(`${webhook}?wait=true&thread_id=${thread}`, messages[i], fetchImpl);
		} catch (error) {
			throw new Error(`post ${post.link} created, but message ${i + 1} of ${messages.length} failed: ${error.message}`);
		}
	}
	return post;
}

function rememberPost(id, version, post) {
	const posts = readSettings(DISCORD_POSTS_FILE);
	posts[id] = { ...(posts[id] || {}), [version]: { thread: post.thread, link: post.link, date: today() } };
	fs.mkdirSync(settingsDir(), { recursive: true });
	fs.writeFileSync(path.join(settingsDir(), DISCORD_POSTS_FILE), JSON.stringify(posts, null, "\t") + "\n", "utf8");
}

function printDiscord(title, messages, files) {
	console.log(`== Discord post (dry run): ${title} ==`);
	console.log(messages[0].content);
	messages.forEach((message, i) => {
		const text = message.embeds[0].description;
		console.log(`--- message ${i + 1} of ${messages.length} (${text.length} characters) ---`);
		console.log(text);
	});
	for (const file of files) console.log(`picture ${file.name} (${Math.ceil(file.size / 1024)} KB): ${path.relative(ROOT, file.full)}`);
}

// ---------- commands ----------

function commandCheck(cfg) {
	const problems = configProblems(cfg);
	const notes = [];
	if (!problems.length) {
		const baseVersion = readVersion(cfg);
		console.log(`${cfg.game_name} ${baseVersion} -> ${cfg.repo}`);
		const release = makeRelease(cfg, "");
		for (const build of release.builds) {
			if (release.mod) {
				console.log(`ok      ${build.loader.padEnd(8)} ${path.relative(ROOT, build.source)} (${megabytes(build.files[0].size)} MB)`);
				if (build.info) {
					const depends = Object.entries(build.info.depends).map(([id, versions]) => `${id} ${Array.isArray(versions) ? versions.join(" ") : versions}`);
					console.log(`        ${build.info.file}: ${build.info.id} ${build.info.version}` + (depends.length ? ` - depends on ${depends.join(", ")}` : ""));
				}
				notes.push(...jarNotes(cfg, build, baseVersion));
				continue;
			}
			const details = release.format === 2 ? ` (${build.files.length} file(s), ${megabytes(sum(build.files))} MB, start file ${build.exe})` : "";
			console.log(`ok      ${build.arch.padEnd(6)} ${path.relative(ROOT, build.source)}${details}`);
		}
		if (release.manifest && release.format === 2 && !release.mod) console.log(`${release.assets.length} release file(s), same files of both builds only once`);
		problems.push(...release.problems);
		const news = readNews(cfg, baseVersion);
		problems.push(...news.problems);
		if (!news.news.some((group) => group.version === baseVersion)) problems.push(`news: no entry for ${baseVersion} yet (${cfg.news_file})`);
		problems.push(...discordImages(cfg, discordNews(cfg, baseVersion).images).problems.map((problem) => "discord: " + problem));
	}
	if (!fs.existsSync(rel(cfg.readme_file))) problems.push("README missing: " + cfg.readme_file);
	problems.push(...catalogProblems(cfg));
	for (const note of notes) console.log("NOTE    " + note);
	for (const problem of problems) console.log("PROBLEM " + problem);
	console.log(problems.length ? `${problems.length} problem(s)` : "CHECK OK");
	process.exit(problems.length ? 1 : 0);
}

// What a player with the latest release loads for this build (the launcher compares path + SHA-256).
function commandDiff(cfg, args) {
	const problems = configProblems(cfg);
	if (problems.length) fail(problems.join("; "));
	const release = makeRelease(cfg, "");
	if (release.problems.length) fail(release.problems.join("; "));
	const old = latestManifest(cfg, args.against);
	console.log(`${cfg.game_name} ${release.version} compared with ` + (old ? old.label : "nothing (no release yet)"));
	if (release.mod) {
		const before = old ? (Array.isArray(old.manifest.files) ? old.manifest.files : []).filter((file) => file.kind === "mod") : [];
		for (const file of release.manifest.files) {
			const was = before.find((other) => (other.loader || "fabric") === file.loader);
			if (!was) console.log(`${file.loader}: players load ${file.path} (${megabytes(file.size)} MB)`);
			else if (String(was.sha256).toLowerCase() === file.sha256) console.log(`${file.loader}: the same .jar as before - players load nothing`);
			else console.log(`${file.loader}: players load ${file.path} (${megabytes(file.size)} MB), the launcher takes out ${was.path}`);
		}
		return;
	}
	const exe = (release.manifest.exe || "").toLowerCase();
	const exeChanged = [];
	for (const build of release.builds) {
		const wanted = filesFor(release.manifest, build.arch);
		const result = diffFiles(old ? filesFor(old.manifest, build.arch) : [], wanted);
		const less = old && result.totalBytes ? ` - ${Math.round(100 * (1 - result.downloadBytes / result.totalBytes))}% less than the whole game` : "";
		console.log(`${build.arch}: players load ${result.download.length} of ${wanted.length} file(s), `
			+ `${megabytes(result.downloadBytes)} of ${megabytes(result.totalBytes)} MB${less}`);
		const lines = [...result.download.map((file) => `  ${file.change.padEnd(8)} ${megabytes(file.size).padStart(7)} MB  ${file.path}`),
			...result.remove.map((file) => `  removed  ${file}`)];
		for (const line of lines.slice(0, LIST_LINES)) console.log(line);
		if (lines.length > LIST_LINES) console.log(`  ... and ${lines.length - LIST_LINES} more`);
		const startFile = result.download.find((file) => file.path.toLowerCase() === exe);
		if (startFile) exeChanged.push(`${build.arch} ${megabytes(startFile.size)} MB`);
	}
	if (!cfg.files.some((entry) => entry.arch === "arm64")) console.log("arm64: uses the x86_64 build (emulated)");
	if (release.format === 1) {
		console.log("note: one exe per build - every update loads the whole game. A build folder (dir + exe) loads only changed files (INTEGRATION.md).");
	} else if (old && (old.manifest.format || 1) < 2) {
		console.log("note: first release made of several files - players load the whole game once, after that only changed files.");
	} else if (old && exeChanged.length) {
		console.log(`WARNING: the start file ${release.manifest.exe} changed (${exeChanged.join(", ")}): every player loads it again.`
			+ " Normal after an engine update - otherwise keep the exe the same (INTEGRATION.md, \"Spiel aus mehreren Dateien\").");
	}
}

async function commandRelease(cfg, args) {
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
	const uploads = stageAssets(release);
	let existing = releaseAssets(cfg.repo, release.tag);
	if (existing) {
		console.log(`== update release ${release.tag} ==`);
	} else {
		// Not "latest" until every file is up: until then the launcher keeps reading the previous release.
		console.log(`== create release ${release.tag} ==`);
		gh(["release", "create", release.tag, "-R", cfg.repo, "--title", title, "--notes-file", release.notesPath, "--latest=false"]);
		existing = [];
	}
	const changed = uploads.filter((asset) => !sameOnGitHub(existing, asset.name, asset.sha256));
	console.log(`${uploads.length - changed.length} file(s) already on GitHub, uploading ${changed.length} (${megabytes(sum(changed))} MB)`);
	// Game files first, manifest last: the launcher never sees a manifest whose files are missing.
	for (let i = 0; i < changed.length; i += UPLOAD_BATCH) {
		gh(["release", "upload", release.tag, ...changed.slice(i, i + UPLOAD_BATCH).map((asset) => asset.file), "-R", cfg.repo, "--clobber"], { inherit: true });
	}
	gh(["release", "upload", release.tag, release.manifestPath, "-R", cfg.repo, "--clobber"], { inherit: true });
	gh(["release", "edit", release.tag, "-R", cfg.repo, "--title", title, "--notes-file", release.notesPath, "--latest"]);
	const remote = releaseAssets(cfg.repo, release.tag) || [];
	const wanted = [...uploads, { name: manifestName(cfg), sha256: sha256File(release.manifestPath) }];
	const missing = wanted.filter((asset) => !sameOnGitHub(remote, asset.name, asset.sha256));
	if (missing.length) fail("missing or different on GitHub: " + missing.map((asset) => asset.name).join(", "));
	if (release.format === 2 && !release.mod) {
		// Files of an earlier upload of this release that the manifest no longer names.
		const names = new Set(uploads.map((asset) => asset.name));
		for (const asset of remote) {
			if (!ASSET_RE.test(asset.name) || names.has(asset.name)) continue;
			gh(["release", "delete-asset", release.tag, asset.name, "-R", cfg.repo, "--yes"]);
			console.log("removed unused: " + asset.name);
		}
		fs.rmSync(path.join(path.dirname(release.manifestPath), "files"), { recursive: true, force: true });
	}
	console.log(`RELEASE OK: https://github.com/${cfg.repo}/releases/tag/${release.tag}`);
	console.log(`manifest:   https://github.com/${cfg.repo}/releases/latest/download/${manifestName(cfg)}`);
	if (args.suffix) console.log("Discord: no forum post for a test release");
	else if (!args.noDiscord) await commandDiscord(cfg, { version: release.version }, true);
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
		type: cfg.type,
		name: cfg.game_name,
		repo: cfg.repo,
		manifest_url: `https://github.com/${cfg.repo}/releases/latest/download/${manifestName(cfg)}`,
		// A .jar is the same on every PC.
		platforms: isMod(cfg) ? [...ARCHES] : cfg.files.map((file) => file.arch),
		engine: isMod(cfg) ? undefined : c.engine.trim(),
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
	if (!isMod(cfg)) entry.user_data_dirs = local.user_data_dirs = c.user_data_dirs;
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

// The Discord forum post of a version, on its own or right after "release" (auto: the release is done, problems are
// only reported). A version that was posted before is skipped (--again posts it once more).
async function commandDiscord(cfg, args, auto = false) {
	const stop = (message) => {
		if (!auto) fail(message);
		console.log(`DISCORD NOT POSTED: ${message} - later: node tools/dasdimo/dasdimo_release.js discord`);
	};
	let version = args.version;
	try {
		version = version || readVersion(cfg);
	} catch (error) {
		return stop(error.message);
	}
	if (!VERSION_RE.test(version)) return stop(`invalid version '${version}'`);
	if (args.news) cfg = { ...cfg, news_file: args.news };
	const title = `${cfg.game_name} ${version}`;
	const news = discordNews(cfg, version);
	const pictures = discordImages(cfg, news.images);
	for (const problem of pictures.problems) console.log("note: " + problem);
	const messages = discordMessages(title, version, news.entries, discordColor(cfg), { images: pictures.files.map((file) => file.name) });
	const hooksFile = path.join(settingsDir(), DISCORD_FILE);
	let hooks;
	let posts;
	try {
		hooks = readSettings(DISCORD_FILE);
		posts = readSettings(DISCORD_POSTS_FILE);
	} catch (error) {
		return stop(error.message);
	}
	const webhook = typeof hooks[cfg.game_id] === "string" ? hooks[cfg.game_id].trim() : "";
	const before = (posts[cfg.game_id] || {})[version];
	if (!news.entries.length) console.log(`note: no news for ${version} in ${cfg.news_file} - the post says "Kleine Verbesserungen und Fehlerbehebungen."`);
	if (args.dryRun) {
		printDiscord(title, messages, pictures.files);
		console.log(webhook ? `webhook for '${cfg.game_id}': set` : `note: no webhook for '${cfg.game_id}' in ${hooksFile} - nothing would be posted`);
		if (before) console.log(`note: already posted on ${before.date}: ${before.link}`);
		return console.log("DRY RUN: nothing posted");
	}
	if (!webhook) return console.log(`note: no Discord webhook for '${cfg.game_id}' in ${hooksFile} - no forum post (INTEGRATION.md, "Discord-Beitrag")`);
	if (!WEBHOOK_RE.test(webhook)) return stop(`the entry '${cfg.game_id}' in ${hooksFile} is not a Discord webhook address`);
	if (before && !args.again) return console.log(`Discord: ${title} was already posted (${before.link}) - --again posts it once more`);
	console.log(`== Discord post: ${title} (${messages.length} message(s)) ==`);
	try {
		const post = await postDiscord(webhook, messages, {
			files: pictures.files,
			onCreated: (created) => rememberPost(cfg.game_id, version, created),
		});
		console.log("DISCORD OK: " + post.link);
	} catch (error) {
		return stop(hideWebhook(error.message));
	}
}

async function main() {
	const args = parseArgs(process.argv.slice(2));
	if (!["check", "diff", "manifest", "release", "register", "discord"].includes(args.command)) {
		console.log("usage: node tools/dasdimo/dasdimo_release.js check | diff [--against <manifest>] | manifest [--suffix x]"
			+ " | release [--suffix x] [--readme] [--no-discord] | register [--dry-run] | discord [--dry-run] [--again] [--version x] [--news <file>]");
		console.log("helper version " + HELPER_VERSION);
		process.exit(args.command === "help" ? 0 : 1);
	}
	const cfg = loadConfig(args.config);
	if (args.command === "check") return commandCheck(cfg);
	if (args.command === "diff") return commandDiff(cfg, args);
	if (args.command === "manifest") {
		const problems = configProblems(cfg);
		if (problems.length) fail(problems.join("; "));
		buildRelease(cfg, args.suffix);
		return console.log("MANIFEST OK (nothing uploaded)");
	}
	if (args.command === "release") return commandRelease(cfg, args);
	if (args.command === "discord") return commandDiscord(cfg, args);
	return commandRegister(cfg, args);
}

// tools/discord/test_discord.js loads the functions without running a command.
if (require.main === module) main().catch((error) => fail(error.message));
else module.exports = { discordMessages, discordNews, discordImages, postDiscord, hideWebhook, allowedLink, jarInfo, LAUNCHER_PAGE };
```
