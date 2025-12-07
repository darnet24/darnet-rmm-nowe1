# RMM SaaS – Aktualizacje, Backupy, Okna serwisowe, Skrypty, Bilety, CVE/CVSS, Języki (PL/EN)

Dokument dla programistów: wymagania i etapy wdrożenia podglądu/wymuszania aktualizacji Windows i aplikacji, okien serwisowych, restartów, backupów (pliki/bazy), destynacji (FTP/SFTP, S3 ogólny: AWS/Wasabi/MinIO, SMB/udział, lokalny dysk), dwujęzycznego UI (PL/EN), skryptów PowerShell/CMD, ticketingu, polityk dziedziczonych per tenant/lokalizacja/OS/urządzenie, CVE/CVSS, monitoringu temperatur/SMART (LibreHardwareMonitor + smartctl). Przejrzyście i krok po kroku.

## 1) Zakres i cele
- Podgląd i wymuszenie aktualizacji Windows + restart gdy wymagany.
- Aktualizacje aplikacji (winget + vendor/niestandardowe).
- Okna serwisowe: update/backup/restart/skrypty.
- Backup (full/inc) z szyfrowaniem, checksumami, wieloma destynacjami (FTP/SFTP, S3, SMB, lokalny).
- i18n PL/EN.
- Skrypty PowerShell/CMD: on-demand, harmonogram, live output.
- Ticketing z alertów/akcji; CVE/CVSS; polityki dziedziczone.
- Audyt, RBAC, bezpieczeństwo.

## 2) Wymagania funkcjonalne – skrót
- Windows Update: pending KB (Security/Critical/Optional), rozmiar, data; Skanuj/Instaluj/Instaluj w oknie/Restart (gdy pending restart).
- Aplikacje: winget scan 12–24h, vendor watcher; Update teraz / Auto w oknie / Wyklucz z auto-update.
- Okna serwisowe: hierarchia urządzenie > grupa/OS > lokalizacja > tenant; kolizje wygrywa najwęższy zakres; akcje: update, restart, backup, skrypty.
- Backup: źródła (foldery, pliki DB/dump), full/inc, szyfrowanie AES/GPG, checksum; destynacje S3 (AWS/Wasabi/MinIO), FTP/SFTP, SMB, lokalny path; retencja osobno full/inc.
- UI: widoki Aktualizacje, Aplikacje, Backupy, Okna serwisowe, Skrypty/Zadania, Bilety, CVE/CVSS, przełącznik PL/EN.
- RBAC: Owner/Admin, Technician, Client-view, opcjonalnie Script-restricted.
- Audyt: kto/kiedy/na czym (scan/install/update/restart/backup/skrypt/bilet), logi tasków (kod wyjścia, stdout/stderr – skrót w UI, pełny do pobrania).
- Temperatury/SMART: LibreHardwareMonitor (CPU/GPU/płyta) + smartctl static (SMART NVMe/SATA/HDD).
- Ticketing: statusy Open/In progress/Resolved/Closed, assignee, komentarze, SLA/priority, powiązania z device/alert/CVE.
- CVE/CVSS: import feed (NVD/OSV), mapowanie do inventory, scoring CVSS, widok ryzyka per tenant/lokalizacja/urządzenie.

## 3) Etapy implementacji
- Etap 0: Bezpieczeństwo (TLS/pinning, CA/mTLS lub HMAC), RBAC+TOTP, audyt, tenantId, polityki dziedziczenia; endpointy bazowe.
- Etap 1: Windows Update + restart (agent scan/install, flaga restart, UI Aktualizacje).
- Etap 2: Aplikacje (winget + vendor, UI Aplikacje, polityki auto/wykluczeń).
- Etap 3: Okna serwisowe + dziedziczenie, enforcement w agencie, UI okien, flagi w tabelach.
- Etap 4: Backup (agent engine full/inc + szyfrowanie/checksum; backend joby/retencja/destynacje; UI Backupy).
- Etap 5: Skrypty (PowerShell/CMD, live output, scheduler, role; UI Skrypty/Zadania).
- Etap 6: Ticketing (encje, API, UI; powiązania z alertami/taskami/CVE).
- Etap 7: CVE/CVSS feed + mapowanie do inventory; UI CVE; akcje: ticket/patch/accept.
- Etap 8: Temperatury/SMART w UI + alerty progowe.
- Etap 9: i18n PL/EN (klucze, tłumaczenia, przełącznik w profilu).
- Etap 10: Hardening, paginacja/sort, logi do pobrania, alerty na fail update/backup, obserwowalność.

## 4) Telemetria (propozycja)
- WU scan: 6–12h + on-demand.
- Aplikacje: winget 12–24h; vendor list co config poll lub 6h.
- Backup: wg harmonogramu; „Run now” poza oknem tylko z logiem w audycie.
- Skrypty: on-demand natychmiast; harmonogram; live output jeśli kanał dostępny.
- Temperatury: 60s; SMART: 15–30 min pełny; alerty progowe.
- SNMP (opcjonalne): 60s interfejsy, 60–120s CPU/RAM, discovery 15–60 min.

## 5) UI – kolumny przykładowe
- Aktualizacje: Device, OS, Agent version, Pending updates, Required restart, Last scan, Maintenance window, Actions (Scan/Install/Restart).
- Aplikacje: App, Local version, Available version, Source (Winget/Vendor), Status, Actions (Update now/Auto/Exclude).
- Backupy: Job, Scope, Destinations, Schedule, Retention (full/inc), Last run (time/type/size/dest/checksum/result), Actions (Run now, Test restore).
- Skrypty/Zadania: Name, Target scope, Schedule, Last run result, Stdout/stderr preview, Actions (Run now, View log).
- Bilety: ID, Device, Alert/CVE link, Status, Assignee, Priority/SLA, Last update.
- CVE: CVE ID, CVSS, Affected software/version, Devices impacted, Status (open/accepted/mitigated), Actions (ticket/update).

## 6) Backup – bezpieczeństwo i destynacje
- Full + Incremental, retencja per typ; szyfrowanie przed wysyłką (AES/GPG), klucze per tenant, rotacja.
- Checksum po transferze; alert przy niezgodności; test restore (sandbox).
- Destynacje: S3 (endpoint, region, bucket, access/secret, prefix, storage class), FTP/SFTP, SMB, lokalny path; retry/backoff, jasne komunikaty błędów.

## 7) Temperatury/SMART – implementacja
- LibreHardwareMonitorLib (NuGet) – bez GUI; CPU/GPU/płyta. 
- smartctl.exe static – jeden plik + LICENSE-smartmontools; `smartctl -A /dev/sdX --json` (NVMe/SATA/HDD); wysyłane: tempC, reallocated, pending, media errors, wear, power_on_hours.

## 8) Skrypty i harmonogram
- PowerShell/CMD; ograniczenia: limit czasu, logowanie, role; opcja „zezwól na zdalne skrypty”.
- Live output (stream/SSE/WebSocket) jeśli kanał utrzymany; podgląd w UI.
- Planowanie: cron/once; przypinane do okien serwisowych.

## 9) Ticketing
- Tworzenie z alertów (monitoring/temp/SMART/CVE), zadań (update/backup fail), manualnie.
- Statusy: Open, In progress, Resolved, Closed; SLA/Priority; assignee; komentarze; audyt.
- Powiązania: deviceId, alertId, cveId, taskId; link do logów zadania/backup/update.

## 10) CVE/CVSS
- Import feed (NVD/OSV), cache w DB, mapowanie do software inventory (name/version) + OS build.
- Widok ryzyka: per tenant/lokalizacja/urządzenie; filtry po CVSS, dacie, produkcie.
- Akcje: utwórz bilet, otwórz update/patch, oznacz jako zaakceptowane/mitigated.

## 11) RBAC i audyt
- Role: Owner/Admin, Technician, Client-view, Script-restricted, Viewer.
- Audyt: kto/kiedy/akcja/target/result; logi tasków; zmiany polityk, okien, destynacji backup, biletów, statusów CVE.

## 12) Notatki wdrożeniowe
- Uniwersalny instalator + token jednorazowy; polityki dziedziczone (tenant > lokalizacja > grupa/OS > device); brak stałych sekretów w instalkach.
- smartctl/LibreHardwareMonitor bez GUI; dołącz LICENSE-smartmontools.
- Błędy destynacji S3/FTP/SMB nie blokują innych zadań: retry/backoff, komunikaty w UI.
- Wszystkie teksty w i18n (PL/EN).