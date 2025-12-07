# 01 – Bezpieczeństwo, polityki, instalator

Zakres: TLS/pinning, mTLS/HMAC, RBAC+TOTP, audyt, polityki dziedziczone (tenant > lokalizacja > grupa/OS > device), instalator z tokenem jednorazowym, i18n założenia globalne.

## Bezpieczeństwo transportu
- HTTPS + pinning certu/CA.
- Agent: mTLS lub HMAC per agent (klucz rotowalny), rejestracja z krótkim tokenem.
- Brak stałych sekretów w binariach/instalkach; token jednorazowy.

## RBAC + MFA
- Role: Owner/Admin, Technician, Client-view, Script-restricted, Viewer.
- TOTP w panelu; enforce MFA per tenant.

## Audyt
- Logujemy: kto/kiedy/akcja/target/result; zmiany polityk, okien, destynacji backup, biletów, statusów CVE, uruchomienia skryptów, update/backup/restart.
- Task logs: kod wyjścia, skrót stdout/stderr (pełny do pobrania).

## Polityki dziedziczone
- Hierarchia: tenant > lokalizacja > grupa/OS > urządzenie; najwęższy zakres wygrywa.
- Zakresy polityk: okna serwisowe, update (WU/winget/vendor), backup (schedule/retencja/destynacje), skrypty (allowlista), monitoring (temp/SMART/AV/firewall/usługi), alerting/webhooki.

## Instalator
- Jeden installer per OS (Windows/Server, Linux, macOS) + token jednorazowy.
- Po rejestracji pobiera polityki i okna serwisowe.
- Aktualizacje agenta: auto z backendu, wersjonowanie, kanał stable.

## i18n (globalne)
- Klucze i18n od początku; domyślnie PL/EN; format czasu UTC+offset, liczby zgodnie z locale.