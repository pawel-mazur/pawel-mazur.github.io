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

#### Konfiguracja szablonu maszyny wirtualnej

Do tworzenia nowych maszyn będziemy posługiwali się bezpośrednio szablonem maszyny wirtualnej będącego naszą bazą.
I tak obecnie stosowany standard zakłada wykorzystanie narzędzia Cloud-init. Jest ono ogólnie dostępne w większości 
nowych dystrybucji i pozwala na inicjowanie konfiguracji maszyn w zakresie kart sieciowych, zasobów dyskowych, kluczy SSH
czy instalowanych pakietów. 

Więcej informacji znajdziemy bezpośrednio na stronie projektu [Cloud-init](https://cloudinit.readthedocs.io/), 
dokumentacji Proxmox [Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support),
a także naszego dostawcy w Terraform [Cloud Init Guide](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs/guides/cloud_init). 
Dokładne informacje jak należy stworzyć szablon naszej maszyny, znajdziemy na stronie [Cloud-Init Getting Started](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs/guides/cloud-init%2520getting%2520started).

Do jego utworzenia posłużymy się oficjalnym obrazem dla systemu Debian — [Debian Official Cloud Images](https://cloud.debian.org/images/cloud/).
W tym celu pobierzemy obraz [`debian-13-generic-arm64.qcow2`](https://cloud.debian.org/images/cloud/trixie/latest/debian-13-generic-arm64.qcow2), który następnie zaimportujemy w naszym Proxmoxie.

![10-debian-cloud-init-download.png](/assets/images/terraform/10-debian-cloud-init-download.png)

Po pobraniu obrazu powinniśmy stworzyć nową maszynę, do której następnie zaimportujemy pobrany dysk w formacie `.qcow2`.
Następnie należy wykonać import dysku poleceniem `qm disk import` wskazując kolejno `<vmid>` — Id naszej maszyny,
`<source>` — ścieżkę wskazującą obraz, `<storage>` — storage, do którego zostanie zaimportowany obraz.

```
# root@pve:~# qm disk import 310 /var/lib/vz/import/debian-13-generic-amd64.qcow2 local-lvm 
importing disk '/var/lib/vz/import/debian-13-generic-amd64.qcow2' to VM 310 ...
  Logical volume "vm-310-disk-1" created.
transferred 0.0 B of 3.0 GiB (0.00%)
transferred 30.7 MiB of 3.0 GiB (1.00%)
transferred 61.4 MiB of 3.0 GiB (2.00%)
transferred 92.5 MiB of 3.0 GiB (3.01%)
...
transferred 3.0 GiB of 3.0 GiB (98.77%)
transferred 3.0 GiB of 3.0 GiB (99.78%)
transferred 3.0 GiB of 3.0 GiB (100.00%)
transferred 3.0 GiB of 3.0 GiB (100.00%)
unused0: successfully imported disk 'local-lvm:vm-310-disk-0'
```

Dysk ten musimy zamontować jeszcze do naszej maszyny. Na tym etapie warto upewnić się, że została zdefiniowana odpowiednia
kolejność bootowania. Na koniec pozostało przekonwertować wirtualną maszynę do szablonu.

### Przygotowanie konfiguracji Terraform

Na tym etapie przygotujemy konfigurację, którą posłużymy się do stworzenia naszej nowej maszyny przy użyciu Terraform. Do
tego celu stworzymy plik `main.tf` w katalogu naszego projektu. W moim przypadku będzie to następująca definicja:

```
resource "proxmox_vm_qemu" "test" {
  vmid        = 500
  name        = "test"
  target_node = "pve"
  clone       = "Debian-13"
  full_clone  = true
  memory      = 2048
  agent       = 0
  tags        = "debian,terraform"
  skip_ipv6   = true

  # Cloud-Init configuration
  ciuser       = "debian"
  cipassword   = "debian"
  ipconfig0    = "ip=192.168.1.20/32,gw=192.168.1.1"
  nameserver   = "192.168.1.1"
  searchdomain = ".*"

  cpu {
    cores = 2
  }

  disk {
    slot    = "scsi0"
    size    = "8G"
    type    = "disk"
    storage = "local-lvm"
    format  = "raw"
    discard = true
  }

  disk {
    slot    = "scsi1"
    type    = "cloudinit"
    storage = "local-lvm"
  }

  network {
    id        = 0
    model     = "virtio"
    bridge    = "vmbr0"
    firewall  = false
    link_down = false
  }
}
```

Na tym etapie powinniśmy sprawdzić, jaki jest plan z użyciem polecenia `terraform plan`.

```
$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
+ create

Terraform will perform the following actions:

# proxmox_vm_qemu.test will be created
+ resource "proxmox_vm_qemu" "test" {
    + additional_wait        = 5
    + agent                  = 0
    + agent_timeout          = 90
    + automatic_reboot       = true
    + balloon                = 0
    + bios                   = "seabios"
    + boot                   = (known after apply)
    + bootdisk               = (known after apply)
    + cipassword             = (sensitive value)
    + ciupgrade              = false
    + ciuser                 = "debian"
    + clone                  = "Debian-13"
    + clone_wait             = 10
    + current_node           = (known after apply)
    + default_ipv4_address   = (known after apply)
    + default_ipv6_address   = (known after apply)
    + define_connection_info = true
    + description            = "Managed by Terraform."
    + force_create           = false
    + full_clone             = true
    + hotplug                = "network,disk,usb"
    + id                     = (known after apply)
    + ipconfig0              = "ip=192.168.1.100/32,gw=192.168.1.1"
    + kvm                    = true
    + linked_vmid            = (known after apply)
    + memory                 = 2048
    + name                   = "test"
    + nameserver             = "192.168.1.1"
    + onboot                 = false
    + protection             = false
    + reboot_required        = (known after apply)
    + scsihw                 = "lsi"
    + searchdomain           = ".*"
    + skip_ipv4              = false
    + skip_ipv6              = true
    + ssh_host               = (known after apply)
    + ssh_port               = (known after apply)
    + tablet                 = true
    + tags                   = "debian,terraform"
    + target_node            = "pve"
    + unused_disk            = (known after apply)
    + vm_state               = "running"
    + vmid                   = 500

    + cpu {
        + cores   = 2
        + limit   = 0
        + numa    = false
        + sockets = 1
        + type    = "host"
        + units   = 0
        + vcores  = 0
          }

    + disk {
        + backup               = true
        + discard              = true
        + format               = "raw"
        + id                   = (known after apply)
        + iops_r_burst         = 0
        + iops_r_burst_length  = 0
        + iops_r_concurrent    = 0
        + iops_wr_burst        = 0
        + iops_wr_burst_length = 0
        + iops_wr_concurrent   = 0
        + linked_disk_id       = (known after apply)
        + mbps_r_burst         = 0
        + mbps_r_concurrent    = 0
        + mbps_wr_burst        = 0
        + mbps_wr_concurrent   = 0
        + passthrough          = false
        + size                 = "8G"
        + slot                 = "scsi0"
        + storage              = "local-lvm"
        + type                 = "disk"
          }
    + disk {
        + backup               = true
        + id                   = (known after apply)
        + iops_r_burst         = 0
        + iops_r_burst_length  = 0
        + iops_r_concurrent    = 0
        + iops_wr_burst        = 0
        + iops_wr_burst_length = 0
        + iops_wr_concurrent   = 0
        + linked_disk_id       = (known after apply)
        + mbps_r_burst         = 0
        + mbps_r_concurrent    = 0
        + mbps_wr_burst        = 0
        + mbps_wr_concurrent   = 0
        + passthrough          = false
        + size                 = (known after apply)
        + slot                 = "scsi1"
        + storage              = "local-lvm"
        + type                 = "cloudinit"
          }

    + network {
        + bridge    = "vmbr0"
        + firewall  = false
        + id        = 0
        + link_down = false
        + macaddr   = (known after apply)
        + model     = "virtio"
          }

    + smbios (known after apply)
      }

Plan: 1 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

W tym wypadku zostanie utworzona nowa maszyna o nazwie `test`, która otrzyma id o wartości `500`. Nie pozostaje nam nic
innego jak utworzyć maszynę przy pomocy polecenia `terraform apply`. Całość będziemy musili zatwierdzić.

```
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

...

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

proxmox_vm_qemu.test: Creating...
proxmox_vm_qemu.test: Still creating... [00m10s elapsed]
proxmox_vm_qemu.test: Still creating... [00m20s elapsed]
proxmox_vm_qemu.test: Still creating... [00m30s elapsed]
proxmox_vm_qemu.test: Creation complete after 37s [id=pve/qemu/500]

```

Teraz możemy cieszyć się z nowo uruchomionej maszyny. Na koniec, jeśli nie będzie nam już potrzebna, warto zrobić porządek
i przy pomocy polecenia `terraform destroy` skasować naszą infrastrukturę na podstawie wcześniej przygotowanej definicji.
Tym razem również otrzymamy podgląd wszelkich zmian i ostatecznie będziemy musieli potwierdzić skasowanie całej infrastuktury.

```
$ terraform destroy
proxmox_vm_qemu.test: Refreshing state... [id=pve/qemu/500]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

...

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

proxmox_vm_qemu.test: Destroying... [id=pve/qemu/500]
proxmox_vm_qemu.test: Destruction complete after 2s
```

# Podsumowanie

Terraform to narzędzie, które pozwoli nam na uporządkowanie definicji naszej infrastruktury, co przełoży się również na jej
odpowiednią organizację. Tak przygotowana konfiguracja może też równie dobrze tworzyć jakże istotną dokumentację.
