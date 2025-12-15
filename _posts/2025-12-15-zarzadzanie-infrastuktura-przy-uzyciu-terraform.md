---
title: Zarządzanie infrastrukturą przy użyciu Terraform
---

[Terraform](https://www.hashicorp.com/en/products/terraform) to narzędzie pozwalające na zarządzanie naszą infrastrukturą 
w duchu IaC (Infrastructure As Code). Dzięki temu można wersjonować nasze zasoby, a także planować i wdrażać aktualizacje
infrastruktury na podstawie wcześniej przygotowanych definicji. Jako alternatywę warto rozważyć wdrożenie [OpenTofu](https://opentofu.org/)
jako bezpośredniego forka ([OpenTofu Announces Fork of Terraform](https://opentofu.org/blog/opentofu-announces-fork-of-terraform/)).

# Instalacja

Chcąc rozpocząć naszą przygodę, koniecznie będzie zainstalowanie na naszej maszynie roboczej narzędzia Terraform.
Zrobimy to zgodnie z instrukcją zawartą na oficjalnej stronie [Install Terraform](https://developer.hashicorp.com/terraform/install).
Jako że sam do pracy wykorzystuje system Linux oparty na Debianie, a wcześniej miałem już skonfigurowane repozytoria
Hashicorp, wystarczyło wydanie polecenia `apt update`, a następnie `apt install terraform`. Dla weryfikacji sprawdziłem
zainstalowaną wersję wykonując polecenie `terraform -v`. Na ten moment została zainstalowana wersja `v1.14.2`.

```
$ terraform -v
Terraform v1.14.2
on linux_amd64
```

# Dostawcy

Aby zarządzać zasobami, będziemy musieli komunikować się z naszym dostawcą tak, aby w naszym imieniu narzędzie mogło tworzyć,
aktualizować, a także kasować nasze zasoby, kiedy nie będą już nam potrzebne.

Chcąc zacząć naszą przygodę, będziemy mogli wykorzystać przygotowane już szablony do zarządzania zasobami w zależności od
wykorzystywanego przez nas narzędzia. Możemy je przeglądać na oficjalnym [Terraform Registry](https://registry.terraform.io/).
Do naszej dyspozycji jest obecnie całkiem spory katalog, w którym znajdziemy odpowiednie szablony do naszych potrzeb.

Szczegóły dot. dostawców zostały opisane w dokumentacji na stronie [Providers](https://developer.hashicorp.com/terraform/language/providers).

# Proxmox

Do zarządzania maszynami po stronie Proxmoxa wykorzystamy dostępny szablon [Proxmox Provider](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs).
Moim dostawcą będzie lokalny serwer Proxmox w wersji v8.4.0.

#### Konfiguracja dostępu

Aby Terraform mógł komunikować się z serwerem Proxmox, konieczne będzie stworzenie użytkownika i nadanie mu odpowiednich uprawnień.
Zrobimy to od strony panelu administracyjnego ([User Management](https://pve.proxmox.com/pve-docs/chapter-pveum.html)) albo
bezpośrednio z użyciem polecenia [pveum](https://pve.proxmox.com/pve-docs/pveum.1.html).

Instrukcje zawarte na stronie [Creating the Proxmox user and role for terraform](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#creating-the-proxmox-user-and-role-for-terraform)
powinny być odpowiedzią na pytanie, jaki jest wymagany zakres uprawnień do zarządzania maszynami.

Po stworzeniu użytkownika nadamy mu możliwość logowania poprzez API ([Using an API Token (Recommended)](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#using-an-api-token-recommended)),
jest to zalecany sposób autoryzacji.

#### Konfiguracja projektu

Po przygotowaniu odpowiednich dostępów, w naszym projekcie należy wskazać dostawcę poprzez konfigurację w pliku `providers.tf`.
Przykład konfiguracji zajdziemy na stronie [Automatic Registry Installation](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs/guides/installation#automatic-registry-installation).

Szczegóły dot. poszczególnych opcji konfiguracyjnych można znaleźć na stronie [Argument Reference](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#argument-reference).
I tak będzie nam potrzebne ustawienie zmiennej `pm_api_url` odpowiedzialnej za adres repozytorium. Co więcej, jeśli nasz
serwer ma niezweryfikowany certyfikat TLS, konieczne będzie ustawienie opcji `pm_tls_insecure`.

W moim przypadku konfiguracja będzie wyglądała następująco:

```
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "3.0.2-rc04"
    }
  }
}

provider "proxmox" {
  pm_api_url = "https://pve.lan:8006/api2/json"
}
```

Następnie musimy ustawić odpowiednie poświadczenia. Najwygodniej zrobimy to z życiem zmiennych środowiskowych, tak aby nie było konieczności
umieszczenia ich bezpośrednio w definicji zadania i ich wersjonowania. Tu również odsyłam do aktualnej dokumentacji szablonu —
[Creating the connection via username and API token](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs#creating-the-connection-via-username-and-password).

Chcąc zacząć naszą przygodę, powinniśmy zainstalować dostawcę w nowym projekcie. Zrobimy to za pomocą polecenia
`terraform init`. Dzięki temu Terraform pobierze potrzebne zależności do katalogu `.terraform`. Po więcej szczegółów
odsyłam na stronę — [Initialize the Working Directory](https://developer.hashicorp.com/terraform/cli/init).

Wynik działania polecenia:

```
$ terraform init 
Initializing the backend...
Initializing provider plugins...
- Finding telmate/proxmox versions matching "3.0.2-rc04"...
- Installing telmate/proxmox v3.0.2-rc04...
- Installed telmate/proxmox v3.0.2-rc04 (self-signed, key ID A9EBBE091B35AFCE)
Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://developer.hashicorp.com/terraform/cli/plugins/signing
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
