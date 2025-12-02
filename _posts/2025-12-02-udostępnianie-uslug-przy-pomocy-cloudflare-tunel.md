---
title: Udostępnianie usług przy pomocy Cloudflare Tunnel
tags: [cloudflare, network, proxy] 
---

Mając do dyspozycji serwer i usługę, którą chcielibyśmy udostępnić w sieci, musimy spełnić kilka warunków, aby umożliwić
do niej dostęp ze świata. Warunkiem koniecznym jest dostęp do sieci internet oraz konieczność przekierowania odpowiednich
portów do naszego serwera. W momecie, kiedy nie mamy publicznie dostępnego adresu IP, ciekawą alternatywą może stać się 
narzędzie [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/).

# Tworzenie tunelu

Na początku jesteśmy zobowiązani stworzyć tunel będący wykorzystywany w daleszej konfiguracji. Możemy to zrobić zarówno
od [strony panelu](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/get-started/create-remote-tunnel/),
jaki i też [API](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/get-started/create-remote-tunnel-api/). 
Na tym etapie zajmiemy się stworzeniem tunelu poprzez panel. W tym celu przejdę do formularza tworzenia nowego tunelu.

![00-create.png](/assets/images/cloudflare-tunnel/00-create.png)

Po stworzeniu tunelu, konieczne jest podłączenie się do niego z poziomu usługi, która będzie miała dostęp do docelowej
strony. Na tym etapie skonfiguruje dedykowany kontener odpowiedzialny za połączenie.

![02-configure.png](/assets/images/cloudflare-tunnel/02-configure.png)

# Konfiguracja usługi

Do uruchomienia przykładowej usługi, do której następnie zostanie przekierowany ruch z internetu, wykorzystam wcześniej 
przygotowany klaster Nomada. Posłużę się następującą definicją zadania.

```
variable "domain" {
  type = string
}

job "example" {
  datacenters = ["dc1"]

  group "example" {
    network {
      port "http" {
        to = "5678"
      }
    }

    task "server" {
      driver = "docker"

      config {
        image = "hashicorp/http-echo"
        ports = ["http"]
        args  = [
          "-listen",
          ":5678",
          "-text",
          "hello world",
        ]
      }

      service {
        port = "http"
        tags = [
          "traefik.enable=true",
          "traefik.http.routers.${NOMAD_JOB_NAME}-${NOMAD_TASK_NAME}.rule=HOST(`${var.domain}`)",
          "traefik.http.routers.${NOMAD_JOB_NAME}-${NOMAD_TASK_NAME}-wan.tls.certResolver=dns",
          "traefik.http.routers.${NOMAD_JOB_NAME}-${NOMAD_TASK_NAME}.tls=true",
        ]
      }
    }
  }
}
```

Definicja została przeze mnie sprawdzona z użyciem polecenia `nomad plan`, żeby potwierdzić poprawność konfiguracji 
i zweryfikować ew. zmiany potrzebne do wdrożenia usługi, a następnie z użyciem `nomad deploy` do uruchomienia procesu deploymentu.

```
$ nomad plan -var-file=example.vars example.hcl 
+ Job: "example"
+ Task Group: "example" (1 create)
  + Task: "server" (forces create)

Scheduler dry-run:
- All tasks successfully allocated.

Job Warnings:
1 warning:

* task "server" in group "example" defines services, but has no shutdown_delay set

Job Modify Index: 0
To submit the job with version verification run:

nomad job run -check-index 0 -var-file="example.vars" example.hcl

When running the job with the check-index flag, the job will only be run if the
job modify index given matches the server-side version. If the index has
changed, another user has modified the job and the plan's results are
potentially invalid.
```
```
$ nomad run -var-file=example.vars example.hcl 
Job Warnings:
1 warning:

* task "server" in group "example" defines services, but has no shutdown_delay set


==> View this job in the Web UI: http://*****:4646/ui/jobs/example@default

==> 2026-08-24T21:30:45+02:00: Monitoring evaluation "091d54f1"
    2026-08-24T21:30:45+02:00: Evaluation triggered by job "example"
    2026-08-24T21:30:46+02:00: Evaluation within deployment: "ac4b1214"
    2026-08-24T21:30:46+02:00: Allocation "7ea91520" created: node "67bd6be4", group "example"
    2026-08-24T21:30:46+02:00: Evaluation status changed: "pending" -> "complete"
==> 2026-08-24T21:30:46+02:00: Evaluation "091d54f1" finished with status "complete"
==> 2026-08-24T21:30:46+02:00: Monitoring deployment "ac4b1214"
  ✓ Deployment "ac4b1214" successful
    
    2026-08-24T21:30:58+02:00
    ID          = ac4b1214
    Job ID      = example
    Job Version = 0
    Status      = successful
    Description = Deployment completed successfully
    
    Deployed
    Task Group  Desired  Placed  Healthy  Unhealthy  Progress Deadline
    example     1        1       1        0          2026-08-24T21:40:56+02:00

```

# Konfiguracja Tunelu

### Uruchomienie aplikacji

W ten sam sposób opublikuje kontener z narzędziem Cloudflare Tunnel, potrzebnym do obsługi przekierowania ruchu.
Opublikuje następująca definicję zadania, którą przygotowałem. Jako zmienną środowiskową przekazuję token do autoryzacji.

```
variable "token" {
  type = string
}

job "cloudflare-tunnel" {
  datacenters = ["dc1"]

  task "server" {
    driver = "docker"

    config {
      image = "cloudflare/cloudflared"
      command = "tunnel"
      args  = [
        "--no-autoupdate",
        "run",
      ]
    }

    env {
      TUNNEL_TOKEN = "${var.token}"
    }
  }
}
```

```
$ nomad plan -var-file=cloudflare-tunnel.vars cloudflare-tunnel.hcl
+ Job: "cloudflare-tunnel"
+ Task Group: "server" (1 create)
    + Task: "server" (forces create)

Scheduler dry-run:
- All tasks successfully allocated.

Job Modify Index: 0
To submit the job with version verification run:

nomad job run -check-index 0 -var-file="cloudflare-tunnel.vars" cloudflare-tunnel.hcl

When running the job with the check-index flag, the job will only be run if the
job modify index given matches the server-side version. If the index has
changed, another user has modified the job and the plan's results are
potentially invalid.
```
```
$ nomad run -var-file=cloudflare-tunnel.vars cloudflare-tunnel.hcl

==> View this job in the Web UI: http://*****:4646/ui/jobs/cloudflare-tunnel@default

==> 2026-08-24T22:02:50+02:00: Monitoring evaluation "ad0e93bd"
2026-08-24T22:02:50+02:00: Evaluation triggered by job "cloudflare-tunnel"
2026-08-24T22:02:51+02:00: Evaluation within deployment: "9b2e6d33"
2026-08-24T22:02:51+02:00: Allocation "44955bd6" created: node "67bd6be4", group "server"
2026-08-24T22:02:51+02:00: Evaluation status changed: "pending" -> "complete"
==> 2026-08-24T22:02:51+02:00: Evaluation "ad0e93bd" finished with status "complete"
==> 2026-08-24T22:02:51+02:00: Monitoring deployment "9b2e6d33"
✓ Deployment "9b2e6d33" successful

    2026-08-24T22:03:04+02:00
    ID          = 9b2e6d33
    Job ID      = cloudflare-tunnel
    Job Version = 0
    Status      = successful
    Description = Deployment completed successfully
    
    Deployed
    Task Group  Desired  Placed  Healthy  Unhealthy  Progress Deadline
    server      1        1       1        0          2026-08-24T22:13:02+02:00
```

### Podsumowanie

W ten sposób podłączyliśmy się do infrastruktury i możemy przejść do dalszej konfiguracji w panelu Cluodflare.
Zobaczymy jej aktualne podsumowanie.

![10-summary.png](/assets/images/cloudflare-tunnel/10-summary.png)

### Konfiguracja tras

Następnie przechodzimy do sekcji konfigurującej trasy. W tym celu dodamy sobie przykładową konfigurację z użyciem
[opublikowanej aplikacji](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/get-started/create-remote-tunnel/#2a-publish-an-application).

W tym celu wskazujemy oczywiście domenę i adres naszej usługi. W moim przypadku będzie to subdomena `example` i
usluga `https://example.lan`. Jeśli korzystamy z nazw domenowych na naszym serwerze, w sekcji opcji dodatkowych możemy 
również skonfigurować nagłówek `Host`, z którym będzie inicjowane połączenie. Natomiast jeśli na serwerze korzystamy
ręcznie generowanych certyfikatów TLS, możemy również wyłączyć ich weryfikację bądź wskazać lokalizację do CA.
Inaczej połączenie zostanie zablokowane.

![12-configure-route.png](/assets/images/cloudflare-tunnel/12-configure-route.png)

Po zapisaniu formularza możemy zweryfikować nasze aktualne przekierowania.

![14-available-routes.png](/assets/images/cloudflare-tunnel/14-available-routes.png)

### Weryfikacji ruchu

Przydatną funkcją może też okazać się podgląd logów z poziomu panelu.

![16-logs.png](/assets/images/cloudflare-tunnel/16-logs.png)

# Podsumowanie

[Cloudflare Tunel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/) to całkiem proste
narzędzie pozwalające na konfigurację dostępu z sieci internet do wew. infrastruktury, w której publikowane są nasze usługi.
Choć konieczne jest uruchomienie dedykowanej aplikacji bezpośrednio w naszej sieci, możemy to zrobić z użyciem natywnej paczki,
czy też z wykorzystaniem dokera. Sama konfiguracja odbywa się już bezpośrednio w panelu CF, co upraszcza proces konfiguracji
po stronie serwera i nie wymaga do niego dostępu po uruchomieniu usługi.
