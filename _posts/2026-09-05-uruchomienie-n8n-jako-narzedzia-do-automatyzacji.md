---
title: Uruchomienie n8n jako narzędzia do automatyzacji
tags: [n8n, automatyzacja, docker, nomad, traefik]
---

Przy okazji rozbudowy swojego Home Laba szukałem narzędzia, które pozwoli mi szybko łączyć różne usługi bez pisania
osobnych skryptów do każdej drobnej automatyzacji. Wybór padł na [n8n](https://n8n.io/), czyli platformę do budowania
workflow, integracji z API oraz prostych procesów działających według harmonogramu albo wywoływanych przez webhooki.

W moim przypadku szczególnie istotna była możliwość uruchomienia n8n we własnej infrastrukturze. Dzięki temu mam kontrolę
nad danymi, mogę wystawić usługę przez własny reverse proxy, a konfiguracja trafia do tego samego podejścia, którego używam
dla pozostałych usług działających w środowisku [Nomad](https://developer.hashicorp.com/nomad).

# Założenia

Przed uruchomieniem przyjąłem kilka prostych założeń:

* n8n będzie działał jako kontener Docker zarządzany przez Nomada,
* aplikacja będzie dostępna przez Traefika pod własną domeną,
* dane konfiguracyjne n8n będą zapisywane na trwałym wolumenie hosta,
* klucz szyfrowania poświadczeń nie będzie zapisany na stałe w definicji joba,
* strefa czasowa zostanie ustawiona na `Europe/Warsaw`.

Oficjalna dokumentacja n8n opisuje kilka sposobów uruchomienia, m.in. instalację przez npm, Dockera oraz Docker Compose.
Ponieważ w moim środowisku standardem jest Nomad z driverem Docker, przygotowałem osobną definicję joba zamiast dodatkowego
pliku compose.

# Wolumen

Na początku przygotowałem wolumen hosta, który będzie wykorzystywany do przechowywania katalogu roboczego n8n. To ważny
element konfiguracji, bo w tym miejscu aplikacja zapisuje między innymi lokalną bazę, ustawienia instancji oraz informacje
potrzebne do działania workflowów.

```
type      = "host"
name      = "n8n"
node_id   = "67bd6be4-9b53-f5b2-2619-6e2084ea342d"
host_path = "/srv/n8n"

capability {
  access_mode     = "single-node-writer"
  attachment_mode = "file-system"
}
```

Wolumen wskazuje na katalog `/srv/n8n` na hoście. Dla pojedynczej instancji jest to wystarczające rozwiązanie, a jednocześnie
upraszcza backup, ponieważ wiadomo dokładnie, gdzie znajdują się dane usługi.

# Definicja joba

Sam job Nomada jest typu `service`. W ramach grupy uruchamiam jeden task korzystający z obrazu `docker.n8n.io/n8nio/n8n:latest`.
Kontener wystawia port `5678`, czyli domyślny port panelu www n8n.

```
job "n8n" {
  type = "service"

  group "n8n" {
    network {
      port "http" {
        to = 5678
      }
    }

    volume "n8n" {
      type   = "host"
      source = "n8n"
    }

    task "server" {
      driver = "docker"

      config {
        image = "docker.n8n.io/n8nio/n8n:latest"
        ports = ["http"]
      }

      volume_mount {
        volume      = "n8n"
        destination = "/home/node/.n8n"
      }
    }
  }
}
```

Warto zwrócić uwagę na montowanie wolumenu do katalogu `/home/node/.n8n`. Dzięki temu restart kontenera nie powoduje utraty
konfiguracji. W środowisku produkcyjnym rozważyłbym też przypięcie konkretnej wersji obrazu zamiast korzystania z tagu
`latest`, ale w moim labie wygoda aktualizacji była na tym etapie ważniejsza.

# Konfiguracja środowiska

Kolejnym krokiem było dodanie zmiennych środowiskowych. Najważniejsze z nich dotyczą adresu, pod którym działa instancja,
obsługi reverse proxy oraz klucza szyfrującego dane uwierzytelniające.

```
env {
  GENERIC_TIMEZONE   = "Europe/Warsaw"
  N8N_ENCRYPTION_KEY = "${var.encryption_key}"
  N8N_WEBHOOK_URL    = "https://${var.domain}/"
  N8N_HOST           = "${var.domain}"
  N8N_PORT           = "5678"
  N8N_PROTOCOL       = "https"
  N8N_PROXY_HOPS     = "1"
  NODE_ENV           = "production"
  TZ                 = "Europe/Warsaw"
}
```

`N8N_ENCRYPTION_KEY` jest szczególnie istotny, ponieważ n8n wykorzystuje go do szyfrowania poświadczeń. Nie warto zostawiać
go przypadkowi ani generować od nowa przy każdym wdrożeniu. Utrata albo zmiana tego klucza może utrudnić korzystanie z
zapisanych credentiali.

Z kolei `N8N_WEBHOOK_URL`, `N8N_HOST`, `N8N_PROTOCOL` oraz `N8N_PROXY_HOPS` porządkują działanie aplikacji za reverse proxy.
Dzięki temu n8n wie, pod jakim publicznym adresem generować webhooki i linki używane przez integracje zewnętrzne.

# Traefik

Do wystawienia usługi użyłem Traefika, czyli tego samego reverse proxy, które obsługuje u mnie pozostałe aplikacje webowe.
W definicji serwisu wystarczyło dodać odpowiednie tagi.

```
service {
  port = "http"
  tags = [
    "traefik.enable=true",
    "traefik.http.routers.${NOMAD_JOB_NAME}.rule=Host(`${var.domain}`)",
    "traefik.http.routers.${NOMAD_JOB_NAME}.tls=true",
    "traefik.http.routers.${NOMAD_JOB_NAME}.entryPoints=websecure",
  ]
}
```

Router Traefika kieruje ruch dla wskazanej domeny do usługi zarejestrowanej przez Nomada. Po stronie n8n nie musiałem
konfigurować certyfikatu bezpośrednio w kontenerze, ponieważ terminacja TLS odbywa się na poziomie reverse proxy. Sam temat
obsługi certyfikatów przez Traefika opisywałem już wcześniej we wpisie
[Obsługa certyfikatów w środowisku Traefik]({% post_url 2025-11-10-obsluga-certyfikatow-w-srodowisku-traefik %}).

# Healthcheck

Dodałem również prosty healthcheck odpytywany przez Nomada. Sprawdza on endpoint `/healthz` i pozwala szybciej zauważyć,
czy aplikacja poprawnie wystartowała.

```
check {
  type     = "http"
  name     = "healthz"
  path     = "/healthz"
  interval = "20s"
  timeout  = "5s"
}
```

Nie jest to rozbudowany monitoring, ale przy codziennym utrzymaniu wystarcza jako pierwsza informacja o stanie usługi.
Docelowo można dołożyć do tego metryki, alerty albo osobny workflow, który będzie testował krytyczne automatyzacje.

# Uruchomienie

Po przygotowaniu wolumenu i joba pozostaje przekazać wartości zmiennych oraz uruchomić usługę w Nomadzie. W moim przypadku
parametry `domain` oraz `encryption_key` trafiły do joba jako zmienne, dzięki czemu definicja może zostać w repozytorium,
a wrażliwe dane nie muszą być wpisane wprost w pliku.

Po wdrożeniu n8n jest dostępny pod skonfigurowaną domeną przez HTTPS. Pierwsze wejście do panelu prowadzi przez utworzenie
konta właściciela instancji, a następnie można już przejść do budowania workflowów.

![Dashboard](/assets/images/n8n/10-dashboard.png)

# Pierwsze zastosowania

Na start traktuję n8n jako miejsce na automatyzacje, które są zbyt małe na osobną aplikację, ale zbyt powtarzalne, żeby
robić je ręcznie. Dobrymi kandydatami są między innymi:

* cykliczne sprawdzanie wybranych endpointów,
* wysyłanie powiadomień po zdarzeniach z innych systemów,
* proste integracje między API,
* porządkowanie danych otrzymanych przez webhooki,
* zadania uruchamiane według harmonogramu.

Największą zaletą jest dla mnie niski próg wejścia. Workflow można złożyć wizualnie, ale kiedy trzeba, nadal można dodać
logikę w kodzie albo wykonać własne zapytanie HTTP.

![Workflow](/assets/images/n8n/20-workflow.png)

# Backup i utrzymanie

Ponieważ dane aplikacji znajdują się w `/srv/n8n`, backup sprowadza się do objęcia tego katalogu standardowym procesem
kopii zapasowych. Warto jednak pamiętać, że same pliki to nie wszystko. Klucz `N8N_ENCRYPTION_KEY` jest równie ważny jak
dane na dysku, bo bez niego zapisane poświadczenia mogą okazać się bezużyteczne.

Przy aktualizacjach planuję zachować prostą procedurę:

* sprawdzić zmiany w dokumentacji n8n,
* wykonać backup katalogu z danymi,
* zaktualizować obraz kontenera,
* po restarcie sprawdzić panel, healthcheck oraz najważniejsze workflow.

# Podsumowanie

Uruchomienie n8n w Nomadzie okazało się dość proste, bo aplikacja dobrze pasuje do kontenerowego modelu wdrożeń. Kluczowe
było przygotowanie trwałego wolumenu, poprawne ustawienie adresów dla webhooków oraz wystawienie całości przez Traefika.

Na tym etapie mam gotową bazę pod automatyzacje w Home Labie. Najpierw wykorzystam ją do prostych zadań administracyjnych,
a później pewnie także do integracji usług, które do tej pory żyły jako pojedyncze skrypty albo ręczne checklisty.
