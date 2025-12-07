# RMM SaaS – Aktualizacje, Backupy, Okna serwisowe, Języki (PL/EN)

Dokument dla programistów: wymagania i etapy wdrożenia podglądu/wymuszania aktualizacji Windows i aplikacji, okien serwisowych, restartów, backupów (pliki/bazy), destynacji (FTP/SFTP, S3 ogólny: AWS/Wasabi/MinIO, SMB/udział, lokalny dysk), dwujęzycznego UI (PL/EN). Ma być przejrzyście i krok po kroku.

## 1) Zakres i cele
- Podgląd i wymuszenie aktualizacji Windows + restart gdy wymagany.
- Aktualizacje aplikacji (winget + vendor/niestandardowe, np. programy księgowe, Notepad++).
- Okna serwisowe do automatycznych akcji (update/backup/restart).
- Backup plików i baz (pełny/przyrostowy), szyfrowanie, checksumy, wielo-destynacje (FTP/SFTP, S3 ogólny, SMB, lokalny).
- Dwujęzyczny interfejs (PL/EN) – i18n.
- Audyt, RBAC, bezpieczeństwo.

## 2) Wymagania funkcjonalne – skrót
- Windows Update: lista pending KB, kategoria (Security/Critical/Optional), rozmiar, data; akcje „Skanuj”, „Instaluj”, „Instaluj w oknie”, „Restart” (gdy pending restart).
- Aplikacje: winget scan 12–24h, vendor watcher z backendu; akcje „Update teraz”, „Auto-update w oknie”, „Wyklucz z auto-update”.
- Okna serwisowe: dni/godziny; hierarchia: urządzenie > grupa > tenant; kolizje rozstrzyga najwęższy zakres; akcje dozwolone: update, restart, backup.
- Backup: źródła (foldery, pliki DB lub dump), tryby (full/inc), szyfrowanie (AES/GPG), checksum po transferze, retencja osobno dla full/inc.
- Destynacje: 
  - S3 ogólny (AWS/Wasabi/MinIO): endpoint, region, bucket, access/secret; 
  - FTP/SFTP; 
  - SMB (\\host\share + cred); 
  - lokalny path.
- UI: 
  - Widok "Aktualizacje" (tabela urządzeń: pending updates, restart flag, okno serwisowe; przyciski scan/install/restart). 
  - Widok "Aktualizacje aplikacji" (lokalna vs dostępna wersja, źródło, status). 
  - Widok "Backupy" (joby, harmonogram, destynacje, ostatnie runy, "Run now", "Test restore"). 
  - Widok "Okna serwisowe" (definicje i przypisania). 
  - Przełącznik języka PL/EN.
- RBAC: Owner/Admin (pełne), Technician (akcje operacyjne bez zmiany globalnych polityk), Client-view (podgląd swoich urządzeń/backupów, bez wymuszania).
- Audyt: kto/kiedy/na czym: scan/install/update/restart/backup; logi tasków z kodem wyjścia i stdout/stderr (skrót w UI, całość do pobrania).

## 3) Etapy implementacji (krok po kroku)
**Etap 0 – Bezpieczeństwo i fundamenty**
- TLS/pinning w agencie; CA dla agentów, mTLS lub HMAC per agent.
- RBAC + TOTP; audyt akcji; tenantId w każdej tabeli.
- Endpointy bazowe: /agent/register, /agent/config, /agent/metrics, /agent/tasks, /agent/task-result.

**Etap 1 – Windows Update + restart**
- Agent: scan WU (UsoClient/PowerShell) + telemetry pending KB + flag pending restart.
- Backend/API: listowanie pending updates, akcje scan/install per device/grupa, statusy.
- UI: widok Aktualizacje; przycisk Restart gdy pending.

**Etap 2 – Aktualizacje aplikacji (winget + vendor)**
- Agent: winget upgrade --output json (12–24h), FileVersionInfo dla app spoza winget; wykonanie update.
- Backend: vendor watcher (lista wersji dla app niestandardowych); polityka auto-update/wykluczeń.
- UI: widok Aplikacje; akcje Update teraz / Auto w oknie / Wyklucz.

**Etap 3 – Okna serwisowe**
- Backend: model okien (dni/godziny), przypisanie do tenant/grupa/device, rozstrzyganie kolizji.
- Agent: respektowanie okien przy install/update/backup (chyba że wymuszenie manualne).
- UI: definicje okien, podgląd, przypisania; flaga w tabelach Aktualizacje/Backup.

**Etap 4 – Backup (multi-destynacja)**
- Agent: moduł backupu: full/inc, szyfrowanie, checksum; wysyłka do FTP/SFTP, S3 (AWS/Wasabi/MinIO), SMB, lokalny path. Raport statusu.
- Backend: definicje jobów, retencja full/inc, harmonogram, logi runów, alerty fail/space/checksum.
- UI: widok Backupy (joby, ostatnie runy, "Run now", "Test restore" do sandbox).

**Etap 5 – i18n PL/EN**
- Front: i18n klucze, przełącznik języka (profil użytkownika). 
- Tłumaczenia: menu, etykiety, statusy, błędy, przyciski.

**Etap 6 – Hardening i UX**
- Paginacja/sort w listach; logi tasków do pobrania; obserwowalność (logs/metrics/traces) backendu; rate limiting.

## 4) Telemetria i interwały (propozycja)
- WU scan: 6–12h + on-demand.
- Aplikacje: winget scan 12–24h; vendor list przy każdym pollu config lub co 6h.
- Backup: wg harmonogramu; "Run now" poza oknem serwisowym tylko gdy wymuszone (log w audycie).
- SNMP (jeśli włączone): poll 60s interfejsy, 60–120s CPU/RAM, discovery 15–60 min (nie blokuje update/backup).

## 5) Dane w UI (kolumny przykładowe)
- Aktualizacje (urządzenia): Device, OS, Agent version, Pending updates (#), Required restart (Y/N), Last scan, Maintenance window, Actions (Scan/Install/Restart).
- Aplikacje: App, Local version, Available version, Source (Winget/Vendor), Status, Actions (Update now / Auto / Exclude).
- Backupy: Job name, Scope (folders/db dumps), Destinations, Schedule, Retention (full/inc), Last run (time/type/size/dest/checksum status/result), Actions (Run now, Test restore).

## 6) Destynacje S3 – uwagi implementacyjne
- Konfig: endpoint (URL), region, bucket, accessKey, secretKey, optional prefix, storage class.
- Kompatybilność: AWS S3, Wasabi, MinIO (S3-compatible).
- Transfer: HTTPS, checksum, opcjonalnie multipart dla dużych plików; weryfikacja ETag/hash.

## 7) Backup – tryby i bezpieczeństwo
- Full + Incremental; retencja per typ; opcjonalnie deduplikacja w kolejnej iteracji.
- Szyfrowanie przed wysyłką (AES/GPG), klucze z policy per tenant, rotacja kluczy.
- Checksum po transferze; alert przy niezgodności.
- Test restore (sandbox) cyklicznie lub on-demand.

## 8) Restart – zasady
- Pokazuj przycisk Restart tylko gdy pending restart == true.
- Opcje: natychmiast / opóźnienie (np. 5–60 min) / w oknie serwisowym; powiadomienie użytkownika końcowego (toast/okno) przed restartem.

## 9) RBAC i audyt
- Role: Owner/Admin (pełne), Technician (operacje), Client-view (read-only na swoim tenant). 
- Audyt: kto/kiedy/akcja/target/result; logi tasków (kod wyjścia, skrócony stdout/stderr; pełny do pobrania).

## 10) Checklist deweloperski (per etap)
- Etap 1: endpointy WU (scan/install), flaga restart; UI „Aktualizacje”; agent scan/install.
- Etap 2: winget + vendor, update actions, UI „Aplikacje”.
- Etap 3: model okien, enforcement w agencie, UI okien, integracja flag w tabelach.
- Etap 4: backup engine (agent) + joby/retencja/destynacje (backend) + UI „Backupy”.
- Etap 5: i18n PL/EN (klucze + tłumaczenia); przełącznik w profilu.
- Etap 6: hardening, paginacja, logi do pobrania, alerty na fail backup/update, obserwowalność.

## 11) Notatki wdrożeniowe
- smartctl (SMART) i LibreHardwareMonitor – jak w poprzednich dokumentach; nie są częścią aktualizacji/backupów, ale działają równolegle.
- Upewnić się, że destynacje S3/FTP/SMB nie blokują update/backup przy błędach: clear retry/backoff, dobre komunikaty w UI.
- Wszystkie pola w UI tłumaczalne (i18n).