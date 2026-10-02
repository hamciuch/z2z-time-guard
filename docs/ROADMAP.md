# Roadmap

## Architektura
- **Trójstopniowe wykrywanie lokalizacji** w demonie:
  1. broker lokalny (sieć domowa / tunel VPN) — surowy TCP
  2. broker zewnętrzny (obca sieć z internetem) — MQTT over WebSocket przez reverse proxy, TLS
  3. offline — zachowanie wg `offline_behavior` (`release` = fail-open, `enforce_cached`)
- **Źródło prawdy:** HA, gdy online; stan także lokalnie (SQLite), więc demon nie czeka na sieć przy starcie.
- **Reverse proxy:** Apache z `mod_proxy_wstunnel`, ścieżka `/mqtt` na porcie 443.

## Etapy

### ✅ 0 — Wspólny `config.yaml` (Mac + Win)

### ✅ 1 — Demon macOS na nowej architekturze
- głęboki merge configu, łączenie wg `try_order` z timeoutami
- **startup grace** — usuwa pętlę start-stop
- publikowanie `location` (local/external/offline), reconnect co `reconnect_interval_seconds`

### ✅ 2 — Transport WebSocket (macOS)
- transport per broker (local=tcp, external=websockets), TLS z systemowym magazynem CA

### ⬜ 3 — Test demona macOS
- [ ] migracja config JSON → YAML
- [ ] test z `action: warning_only`
- [ ] potwierdzenie braku pętli start-stop

### ⬜ 4 — Reverse proxy + Mosquitto
- [ ] listener `protocol websockets` w Mosquitto, dostępny tylko dla proxy
- [ ] VirtualHost z `mod_proxy_wstunnel`, `ProxyTimeout` > keepalive MQTT
- [ ] test z obcej sieci → `location: external`

### ⬜ 5 — Usługa Windows na tej samej architekturze
- [ ] config.yaml, trójstopniowe łączenie, grace, WebSocket

### ⬜ 6 — TLS na odcinku proxy → broker
- [ ] monitoring wygasania certu wewnętrznego (nie odnawia go Let's Encrypt)

### ⬜ 7 — mTLS (opcjonalnie)
- [ ] cert kliencki per komputer + ACL per cert (zakładać, że cert z komputera może wyciec)

## Notatki
- Zmiana `base_topic` wymaga przepięcia automatyzacji w HA.
