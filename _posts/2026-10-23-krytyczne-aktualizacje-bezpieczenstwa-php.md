---
title: Krytyczne aktualizacje bezpieczeństwa PHP z października i listopada 2026
tags: [php, security, debian, aktualizacje, cve]
---

**Aktualizacja z 19 listopada 2026:** PHP wydało kolejną serię poprawek bezpieczeństwa. Jeżeli serwer został
zaktualizowany w październiku, należy wykonać aktualizację ponownie — wydania 8.2.36, 8.3.37, 8.4.28 i 8.5.13
zastępują październikowe 8.2.35–8.5.12.

PHP opublikowało komunikaty dla [8.2.36](https://www.php.net/archive/2026.php#2026-11-19-1),
[8.3.37](https://www.php.net/archive/2026.php#2026-11-19-2),
[8.4.28](https://www.php.net/archive/2026.php#2026-11-19-3),
[8.5.13](https://www.php.net/archive/2026.php#2026-11-19-4) oraz nowej stabilnej gałęzi
[8.6.0](https://www.php.net/archive/2026.php#2026-11-19-5).

W [zapowiedzi z 18 listopada](https://www.php.net/archive/2026.php#2026-11-18-1) zespół PHP określił najwyższą
ocenę nowych problemów jako **CRITICAL**. Najpoważniejsze z nich wymagają niestandardowej konfiguracji lub
rzadko spotykanego sposobu wdrożenia, ale producent nadal zaleca szybkie wdrożenie aktualizacji.

Pierwsza seria z 21 października usuwała dwie krytyczne luki umożliwiające anonimowe zdalne wykonanie kodu. Listopadowe
wydania naprawiają kolejne problemy, w tym krytyczne podatności w obsłudze sesji, PHP-FPM i stref czasowych. Nie warto
odkładać tej aktualizacji na zwykłe okno serwisowe. Szczególnie istotne są instalacje obsługujące żądania HTTP, w tym
PHP-FPM, Apache z `mod_php`, `php-cgi`, FrankenPHP oraz LiteSpeed.

# Poprawione wydania

Aktualnie bezpieczne wydania dla obsługiwanych gałęzi to:

* PHP 8.5.13,
* PHP 8.4.28,
* PHP 8.3.37,
* PHP 8.2.36,
* PHP 8.6.0 — pierwsze stabilne wydanie nowej gałęzi zawiera te poprawki.

Październikowa aktualizacja naprawiła dwie krytyczne podatności:

* [CVE-2026-109618](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) — błąd w obsłudze braku pamięci, który może
  doprowadzić do użycia niepoprawnie zainicjalizowanych danych żądania;
* [CVE-2026-109619](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) — użycie nieaktualnego stanu wewnętrznego,
  który może pozostać po zwolnieniu pamięci i zostać wykorzystany przez kolejne żądanie obsługiwane przez ten sam worker.

19 listopada dodano między innymi poprawki dla następujących krytycznych problemów:

* [CVE-2026-115560](https://github.com/php/php-src/security/advisories/GHSA-658j-r8rw-q6mx) — niespójność serializatora
  sesji mogąca prowadzić do wykonania kodu;
* [CVE-2026-126714](https://github.com/php/php-src/security/advisories/GHSA-fm2m-fjf5-v3cp) — częściowo nadpisane pliki
  sesji, które mogą zostać niebezpiecznie zdeserializowane;
* [CVE-2026-129496](https://github.com/php/php-src/security/advisories/GHSA-gr4f-wgx5-jpfv) — use-after-free w pamięci
  podręcznej stref czasowych;
* [podatność na request smuggling w PHP-FPM](https://github.com/php/php-src/security/advisories/GHSA-qc49-hpvq-g2w6),
  pozwalająca pozostawionym danym żądania utworzyć kolejne żądanie FastCGI.

Pełną listę zmian zawiera [oficjalny changelog PHP 8](https://www.php.net/ChangeLog-8.php). Wydanie z października
zostało zapowiedziane jako poprawka dla dwóch luk o ocenie CVSS v4.0 9.2.

# Aktualizacja na Debianie

Na serwerach korzystających z pakietów Debiana nie należy zakładać, że numer pakietu będzie dokładnie równy numerowi
wydania upstream. Opiekun dystrybucji może przenieść poprawkę bezpieczeństwa do starszego numeru wersji pakietu. Dlatego
najpierw odświeżam indeks pakietów i sprawdzam dostępne aktualizacje:

```
sudo apt update
apt list --upgradable
```

Następnie aktualizuję zainstalowane pakiety PHP. Poniższy przykład dotyczy PHP 8.4 z FPM; listę pakietów trzeba dopasować
do faktycznie używanej wersji i rozszerzeń:

```
sudo apt install --only-upgrade php8.4-cli php8.4-common php8.4-fpm
```

Jeżeli serwer ma zainstalowane dodatkowe rozszerzenia, aktualizuję je w tym samym przebiegu, na przykład:

```
sudo apt install --only-upgrade php8.4-curl php8.4-gd php8.4-mbstring php8.4-mysql php8.4-xml php8.4-zip
```

W małym środowisku, w którym aktualizacje systemowe są regularnie przeglądane, właściwym rozwiązaniem może być także:

```
sudo apt upgrade
```

Przed zastosowaniem zmian warto sprawdzić listę pakietów proponowanych przez APT i zaplanować restart usługi. W środowisku
produkcyjnym wykonuję aktualizację najpierw na instancji testowej albo na jednym węźle, a przed wdrożeniem upewniam się,
że mam aktualną kopię zapasową oraz możliwość szybkiego wycofania zmiany.

# Aktualizacja obrazu Docker

Oficjalny obraz `php` w Docker Hub zawiera warianty `cli`, `apache` i `fpm`. Gdy aplikacja korzysta z własnego
`Dockerfile`, poprawka musi trafić do obrazu aplikacji: samo pobranie nowego obrazu bazowego nie aktualizuje już działającego
kontenera.

Najpierw zmieniam w `Dockerfile` tag bazowy na wydanie zawierające poprawkę, zachowując dotychczasowy wariant obrazu.
Przykładowo dla PHP-FPM 8.5:

```
FROM php:8.5.13-fpm
```

W analogiczny sposób można użyć `php:8.4.28-fpm`, `php:8.3.37-fpm`, `php:8.2.36-fpm` albo `php:8.6.0-fpm`.
Warto przypinać pełny numer wersji zamiast korzystać wyłącznie z tagu `8.5-fpm` lub `latest`: ułatwia to powtarzalne
wdrożenie i świadome planowanie kolejnej aktualizacji.

Następnie buduję obraz, wymuszając pobranie aktualnego obrazu bazowego, oraz odtwarzam usługę:

```
docker compose build --pull
docker compose up -d --force-recreate
```

Jeżeli obraz jest budowany w systemie CI, odpowiednikiem jest przebudowanie, opublikowanie nowego tagu obrazu aplikacji i
wdrożenie go w orkiestratorze. Po wdrożeniu sprawdzam wersję wewnątrz kontenera:

```
docker compose exec <nazwa-usługi> php --version
```

Wariant `fpm` nie powinien być publikowany bezpośrednio do Internetu przez `-p`: protokół FastCGI jest przeznaczony do
użycia w prywatnej sieci kontenerów, za reverse proxy obsługującym ruch HTTP.

# Restart workerów i weryfikacja

Sam zapis nowych plików pakietu nie wystarczy, jeżeli działają stare procesy PHP-FPM lub Apache. Po aktualizacji restartuję
usługę odpowiednią dla używanego sposobu uruchomienia PHP:

```
sudo systemctl restart php8.4-fpm
```

albo, gdy PHP działa jako moduł Apache:

```
sudo systemctl restart apache2
```

Wersję interpretera wiersza poleceń można sprawdzić poleceniem:

```
php --version
```

Nie jest to jednak pełne potwierdzenie stanu usługi WWW: CLI i FPM mogą być różnymi wersjami, a dystrybucja może dostarczać
poprawkę jako backport. Po aktualizacji sprawdzam więc również wersję zainstalowanych pakietów oraz stan usługi:

```
dpkg-query -W 'php*' | sort
systemctl status php8.4-fpm
```

Na końcu warto wykonać podstawowy test aplikacji — zalogowanie, obsługę formularza oraz zadanie wymagające używanych
rozszerzeń PHP. Pozwala to potwierdzić, że usługa wróciła do pracy po restarcie.

# Co, jeśli aktualizacja musi poczekać

Pełnym rozwiązaniem jest aktualizacja. Październikowe zalecenie tymczasowego wyłączenia postępu wysyłania plików
(`session.upload_progress.enabled=0`) dotyczyło CVE-2026-115560; poprawka dla tej luki jest już w wydaniach z 19 listopada.
Jeżeli aktualizacja nadal musi zostać odroczona, warto pozostawić tę opcję wyłączoną w konfiguracji ładowanej przed
przetwarzaniem żądania:

```
session.upload_progress.enabled = 0
```

Po zmianie należy zrestartować wszystkie workery PHP. Dla nieaktualnych instalacji PHP 8.5 można dodatkowo ustawić
`fatal_error_backtraces=0`. Nie są to zamienniki aktualizacji — ograniczają tylko wybrane, znane warianty ataku.

Dla CVE-2026-109618 komunikat PHP wskazuje także na ograniczenie rozmiaru danych `POST` oraz uploadów. Takie limity są
dobrą praktyką niezależnie od tej luki, ale również nie zastępują poprawionej wersji PHP.

## Cloudflare WAF jako dodatkowa warstwa ochrony

Jeżeli strona jest wystawiona przez Cloudflare, warto pozostawić aktywne zarządzane reguły WAF również po aktualizacji.
Cloudflare opisało październikowe podatności jako
[**php2shell**](https://blog.cloudflare.com/php-vulnerabilities-waf-protection/) i wdrożyło ochronę przed znanymi
próbami wykorzystania luk dla klientów korzystających z WAF — także w planie Free. Dla planów płatnych należy sprawdzić,
czy Cloudflare Managed Ruleset działa w trybie blokowania.

W panelu Cloudflare zdarzenia można przejrzeć w **Security → Security Analytics → Events**, filtrując ruch zablokowany
przez zarządzane reguły PHP. Pozwala to wykryć próby ataku oraz ocenić ewentualny wpływ reguł na aplikację.
Instrukcję włączenia i dostosowania reguł zawiera dokumentacja
[Cloudflare Managed Ruleset](https://developers.cloudflare.com/waf/managed-rules/reference/cloudflare-managed-ruleset/).

WAF nie usuwa błędu z interpretera PHP i nie zastępuje aktualizacji. Cloudflare zaleca zastosowanie poprawki dostawcy
przy utrzymaniu ochrony WAF jako dodatkowej warstwy. Należy też pamiętać, że ochrona przed znanymi próbami nie gwarantuje
zablokowania każdej obecnej lub przyszłej techniki obejścia.

# Starsze gałęzie PHP

PHP 8.1 i starsze nie otrzymują już regularnych aktualizacji od projektu PHP. Dostawca systemu lub wsparcie LTS może
dostarczyć backport, dlatego należy sprawdzić jego biuletyn bezpieczeństwa. Jeżeli takiej poprawki nie ma, rozsądną drogą
jest migracja do wspieranej wersji PHP. Dla osób budujących PHP ze źródeł zespół bezpieczeństwa opublikował minimalne
backporty także dla PHP 7.4, 8.0 i 8.1, lecz nie przywraca to tym gałęziom pełnego wsparcia.

PHP 8.2 pozostaje objęte tą aktualizacją, ale jego koniec wsparcia przypada na 31 grudnia 2026 roku. Warto potraktować tę
aktualizację jako okazję do zaplanowania przejścia na nowszą gałąź.

# Źródła

* [PHP: Security advisory: Critical remote code execution vulnerabilities, update now](https://www.php.net/archive/2026.php#2026-10-21-5)
* [PHP: Upcoming Critical PHP Security Release](https://www.php.net/archive/2026.php#2026-11-18-1)
* [PHP: PHP 8.2.36 Released!](https://www.php.net/archive/2026.php#2026-11-19-1)
* [PHP: PHP 8.3.37 Released!](https://www.php.net/archive/2026.php#2026-11-19-2)
* [PHP: PHP 8.4.28 Released!](https://www.php.net/archive/2026.php#2026-11-19-3)
* [PHP: PHP 8.5.13 Released!](https://www.php.net/archive/2026.php#2026-11-19-4)
* [PHP: PHP 8.6.0 Released!](https://www.php.net/archive/2026.php#2026-11-19-5)
* [PHP: PHP 8 ChangeLog](https://www.php.net/ChangeLog-8.php)
* [PHP: Upcoming Critical PHP Security Release](https://www.php.net/archive/2026.php#2026-10-20-1)
* [PHP/php-src: RCE via OOM corruption](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w)
* [PHP/php-src: RCE via stale global state](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp)
* [Cloudflare: Cloudflare deploys protections against php2shell vulnerability](https://blog.cloudflare.com/php-vulnerabilities-waf-protection/)
* [Cloudflare: Cloudflare Managed Ruleset](https://developers.cloudflare.com/waf/managed-rules/reference/cloudflare-managed-ruleset/)
* [Docker Hub: php — Official Image](https://hub.docker.com/_/php)
