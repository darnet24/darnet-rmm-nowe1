# 02 – Aktualizacje i restart (Windows/Server, Linux, macOS)

Cel: podgląd i wymuszanie aktualizacji OS/app, respektowanie okien serwisowych, bezpieczny restart. Czytelne dla programistów: zakres, MVP, kolejne iteracje.

## Zakres
- Windows/Windows Server: Windows Update (scan/install), restart flag, komunikat/odroczenie, winget + vendor watcher.
- Linux: apt/yum/dnf + bash (scan/install), restart info (needrestart lub plik /var/run/reboot-required), okna serwisowe.
- macOS: softwareupdate/brew + bash, restart gdy wymagany.
- Okna serwisowe: kolizje wygrywa najwęższy zakres (device > grupa/OS > lokalizacja > tenant). On-demand poza oknem loguje wyjątek w audycie.

## Widok/UI – skrót kolumn
Device, OS, Agent version, Pending updates (OS/app), Required restart, Last scan, Maintenance window, Actions (Scan/Install/Restart), Source (WU/winget/vendor/apt/yum/dnf/brew/softwareupdate).

## Sterowanie restartem
- Windows: restart flag po instalacji; komunikat użytkownikowi + opcja odroczenia X min; wymuszenie w oknie serwisowym.
- Linux/macOS: restart jeśli wymagany (np. kernel/OS update), log w audycie.

## Telemetria / interwały
- WU scan: 6–12h + on-demand.
- Aplikacje (winget/vendor): 12–24h; vendor watcher co config poll lub 6h.
- Linux/macOS: analogicznie 12–24h scan.

## Bezpieczeństwo i limity
- Brak auto-instalacji poza oknem (chyba że wymuszone z audytem wyjątku).
- Rozmiar/typ aktualizacji w logu (Security/Critical/Optional, app name/version).

## MVP – checklist
- Agent: scan/install WU (Windows), scan/install winget (Windows), restart flag; scan/install apt/yum/dnf (Linux), restart info; scan/install softwareupdate/brew (macOS).
- Backend/API: task creation, status, audyt, wymuszenie w oknie, on-demand loguje wyjątek.
- UI: lista aktualizacji, akcje Scan/Install/Restart, kolumny jw., statusy per device.

## Kolejne iteracje
- Fine-grained selekcja aktualizacji (per KB/per app).
- Pre-download cache, throttling.
- Rollback info (jeśli dostępne z OS/app).
- Policy wyjątków (wykluczenia app/KB).