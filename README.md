# z2z-time-guard

Limity czasu korzystania z komputera, sterowane z Home Assistant przez MQTT (autodiscovery).

| Komponent | Platforma |
|---|---|
| **WinTimeGuard** | Windows 10/11 — usługa (pywin32) |
| **MacTimeGuard** | macOS (Apple Silicon) — LaunchDaemon |

## Co robi

- liczy aktywny czas użytkowania (Windows: idle per sesja przez `WTSQuerySessionInformationW`; macOS: tryb `strict` — czas leci, gdy ekran odblokowany i użytkownik na konsoli, plus detekcja audio)
- dzienny limit w minutach + okna czasowe z kalendarza HA
- akcje po przekroczeniu: `lock` / `logoff` / `shutdown` / `warning_only`
- ostrzeżenie X minut przed końcem, bonus/kara minut z HA
- stan w SQLite — przeżywa restart
- (macOS) trójstopniowe łączenie: broker lokalny → broker zewnętrzny po MQTT-over-WebSocket (TLS) → offline (`offline_behavior`), startup grace

## Struktura

```
windows/         usługa, install.ps1, uninstall.ps1, config.json.example, requirements.txt
macos/           demon, plist, install.sh, uninstall.sh, config.yaml.example
homeassistant/   przykładowe pakiety HA i karty dashboardu
server/          przykład: Apache (mod_proxy_wstunnel) + listener WebSocket Mosquitto
docs/            ROADMAP.md
```

## Instalacja — Windows

Python 3.10+ **dla wszystkich użytkowników** (usługa działa jako LocalSystem i nie widzi instalacji per-user):

```powershell
winget install -e --id Python.Python.3.12 --scope machine --override "/quiet PrependPath=1 Include_launcher=1 InstallAllUsers=1"
```

PowerShell jako Administrator:

```powershell
Set-ExecutionPolicy -Scope Process Bypass
cd <folder>\windows
.\install.ps1
# edytuj C:\ProgramData\WinTimeGuard\config.json (broker, login/hasło MQTT, konto użytkownika)
Start-Service WinTimeGuard
Get-Content C:\ProgramData\WinTimeGuard\service.log -Tail 30
```

Instalator kopiuje kod do `C:\Program Files\WinTimeGuard`, instaluje zależności, rejestruje usługę (`--startup auto`) i wymusza `LocalSystem`. Debug bez usługi: `python wintimeguard_service.py debug`.

## Instalacja — macOS

Z konta administratora (nie z limitowanego konta):

```bash
cd <folder>/macos
chmod +x install.sh
sudo ./install.sh
# edytuj /Library/Application Support/TimeGuard/config.yaml
sudo launchctl kickstart -k system/<label-demona>
tail -f /Library/Logs/TimeGuard/daemon.log
```

Instalator tworzy `/usr/local/bin` i `/usr/local/lib/mactimeguard` (na świeżym Apple Silicon ich nie ma), kopiuje kod, plist do `/Library/LaunchDaemons` i ładuje go przez `launchctl bootstrap system`.

## Home Assistant

Pakiety z `homeassistant/` do `/config/packages/` — **nazwy plików z podkreślnikami, nie myślnikami**. Dashboard korzysta z `card-mod` (HACS), żeby oznaczać karty, gdy komputer jest offline.

## Sekrety

Prawdziwe configi (`config.json`, `config.yaml`) są w `.gitignore` — w repo tylko `*.example` z placeholderami.
