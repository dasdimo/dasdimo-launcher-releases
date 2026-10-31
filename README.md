# DasDimo Launcher

**Deutsch** · [English](#english)

Alle Spiele von DasDimo an einem Ort: installieren, aktualisieren und spielen – mit einem Klick.

## Herunterladen

### ➜ [DasDimoLauncher-Setup.exe](https://github.com/dasdimo/dasdimo-launcher-releases/releases/latest/download/DasDimoLauncher-Setup.exe)

Ein Installer für jeden Windows-PC (Windows 10/11, 64 Bit), auch für Snapdragon-Laptops (Windows on ARM) – er wählt die passende Version selbst.

## So geht's
1. **DasDimoLauncher-Setup.exe** herunterladen und starten.
2. Windows warnt beim ersten Start („Der Computer wurde durch Windows geschützt“), weil die Datei nicht signiert ist: auf **„Weitere Informationen“** klicken, dann **„Trotzdem ausführen“**.
3. Im Installer **Weiter** und **Installieren** klicken. Administrator-Rechte brauchst du nicht.
4. Der Launcher startet. Links siehst du alle Spiele – eins anklicken, **Installieren**, danach **Spielen**.

Updates holt sich der Launcher selbst: Gibt es eine neue Version eines Spiels, steht auf dem Knopf **Aktualisieren**. Gibt es einen neuen Launcher, erscheint oben **„Neuer Launcher“** – ein Klick genügt.

## Gut zu wissen
- **Wo liegt alles?** Launcher und Spiele in `%LOCALAPPDATA%\DasDimo-Launcher`. Einstellungen und Spielstände der Spiele bleiben in ihren eigenen Ordnern.
- **Du hattest den RIFTWAKE-Launcher?** Dein installiertes RIFTWAKE und deine Einstellungen werden automatisch übernommen.
- **Minecraft-Mods:** Links unter „Mods“ stehen DasDimos Minecraft-Mods. Der Launcher legt die Mod in dein Minecraft-Profil (Minecraft Launcher, Modrinth App, NoRisk, CurseForge oder Prism), zeigt, was du dafür noch brauchst, und hält sie aktuell. Minecraft startest du wie gewohnt. NOT AN OFFICIAL MINECRAFT PRODUCT. NOT APPROVED BY OR ASSOCIATED WITH MOJANG OR MICROSOFT.
- **Deinstallieren:** in den Einstellungen des Launchers (Zahnrad oben rechts) oder unter Windows **Einstellungen → Apps**. Dabei werden auch alle Spiele gelöscht; ihre Einstellungen und Spielstände bleiben, außer du wählst es anders.
- **Etwas geht nicht?** Schick DasDimo die Datei `%LOCALAPPDATA%\DasDimo-Launcher\Launcher\launcher.log`.
- **DasDimo-Konto:** Mit einem freiwilligen Konto siehst du rechts deine Freunde und was sie spielen. Launcher und Einzelspieler gehen auch ohne. Mehr dazu unter [Datenschutz](DATENSCHUTZ.md).
- Der Quellcode ist nicht öffentlich. Hier liegen nur die fertigen Versionen und die Spieleliste (`games.json`).

Für die Claude-Agenten neuer Spiele und Mods: [INTEGRATION.md](INTEGRATION.md) erklärt, wie ein Spiel oder eine Minecraft-Mod in den Launcher kommt.

---

<a id="english"></a>
# DasDimo Launcher – English

All of DasDimo's games in one place: install, update and play – with one click.

## Download

### ➜ [DasDimoLauncher-Setup.exe](https://github.com/dasdimo/dasdimo-launcher-releases/releases/latest/download/DasDimoLauncher-Setup.exe)

One installer for every Windows PC (Windows 10/11, 64-bit), Snapdragon laptops (Windows on ARM) included – it picks the right version by itself.

## How it works
1. Download and start **DasDimoLauncher-Setup.exe**.
2. Windows warns on the first start (“Windows protected your PC”) because the file is not signed: click **“More info”**, then **“Run anyway”**.
3. In the installer click **Next** and **Install**. No administrator rights needed.
4. The launcher starts. All games are on the left – click one, **Install**, then **Play**.

The launcher fetches updates by itself: when a game has a new version, the button says **Update**. When there is a new launcher, **“New launcher”** appears at the top – one click is enough.

## Good to know
- **Where is everything?** Launcher and games in `%LOCALAPPDATA%\DasDimo-Launcher`. Settings and saves of the games stay in their own folders.
- **You had the RIFTWAKE launcher?** Your installed RIFTWAKE and your settings are taken over automatically.
- **Minecraft mods:** DasDimo's Minecraft mods are listed on the left under “Mods”. The launcher puts a mod into your Minecraft profile (Minecraft Launcher, Modrinth App, NoRisk, CurseForge or Prism), shows what else you need for it and keeps it up to date. You start Minecraft as usual. NOT AN OFFICIAL MINECRAFT PRODUCT. NOT APPROVED BY OR ASSOCIATED WITH MOJANG OR MICROSOFT.
- **Uninstall:** in the launcher's settings (gear at the top right) or in Windows **Settings → Apps**. This also deletes all games; their settings and saves stay unless you choose otherwise.
- **Something does not work?** Send DasDimo the file `%LOCALAPPDATA%\DasDimo-Launcher\Launcher\launcher.log`.
- **DasDimo account:** With an optional account you see your friends on the right and what they play. The launcher and single player work without it. More under [Privacy](DATENSCHUTZ.md#privacy-of-the-dasdimo-account).
- The source code is not public. This repository only holds the finished versions and the game list (`games.json`).

For the Claude agents of new games and mods: [INTEGRATION.md](INTEGRATION.md) explains how a game or a Minecraft mod gets into the launcher.
