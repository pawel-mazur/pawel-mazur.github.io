---
title: Uruchomienie Proxmox Backup Server
tags: proxmox backup
---

Planując rozbudowę swojego domowego Home Laba bazującego na [Proxmox Virtual Environment](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview),
postanowiłem uruchomić dedykowaną usługę będącą odpowiedzialną za tworzenie i przechowywanie backupu moich maszyn. Choć Proxmox VE
pozwala na wykonanie podstawowego backupu, [Proxmox Backup Server](https://www.proxmox.com/en/products/proxmox-backup-server/overview),
zostało wzbogacone o szereg funkcjonalności niedostępnych w samym PVE.

# Pobranie obrazu

Chcąc uruchomić PBS, powinniśmy w pierwszej kolejności pobrać odpowiedni obraz maszyny wirtualnej, którą uruchomimy. Obraz
jest do pobrania z oficjalnej strony — [Downloads](https://www.proxmox.com/en/downloads/proxmox-backup-server). Pobierzemy
go bezpośrednio z interfejsu PVE, wskazując link do obrazu. Dodatkowo pobrany obraz zweryfikujemy sumą `SHA-256`.

![10-download.png](/assets/images/proxmox-backup/10-download.png)

# Przygotowanie maszyny

W następnym kroku uruchomimy maszynę VM, w której zainstalujemy PBS. Na te potrzeby przygotowałem konfigurację Terraform.
Odsyłam do mojego artykułu — [Zarządzanie infrastrukturą przy użyciu Trraform]({% post_url 2025-12-15-zarzadzanie-infrastruktura-przy-uzyciu-terraform %}).

```
resource "proxmox_vm_qemu" "proxmox-backup" {
  vmid        = 200
  name        = "proxmox-backup"
  target_node = "pve"
  clone       = "Debian-13"
  full_clone  = true
  memory      = 2048
  agent       = 0
  tags        = "debian,terraform,proxmox"
  skip_ipv6   = true
  boot        = "order=sata0"
  vm_state    = "stopped"

  cpu {
    cores = 2
  }

  disk {
    slot = "sata0"
    type = "cdrom"
    iso  = "local:iso/proxmox-backup-server_4.1-1.iso"
  }

  disk {
    slot    = "sata1"
    type    = "cloudinit"
    storage = "local-lvm"
  }

  disk {
    slot      = "scsi0"
    size      = "16G"
    type      = "disk"
    storage   = "local-lvm"
    disk_file = "local-lvm:vm-200-disk-0"
    format    = "raw"
    discard   = true
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

# Instalacja

Po uruchomieniu maszyny, w pierwszym kroku, musimy zdecydować w jakiej formie zostanie uruchomiony instalator. Ja zdecydowałem
się na tryb graficzny.

![20-boot.png](/assets/images/proxmox-backup/20-boot.png)

W kolejnym kroku musimy zaakceptować postanowienie licencyjne i przejść do wyboru dysku na, którym zostanie zainstalowane
narzędzie.

![22-partitioning.png](/assets/images/proxmox-backup/22-partitioning.png)

Kolejno wybieramy kraj, strefę czasową i układ klawiatury.

![24-location-and-timezone.png](/assets/images/proxmox-backup/24-location-and-timezone.png)

Dalej dla konta administratora ustawiamy hasło użytkownika oraz email.

![26-password-and-email.png](/assets/images/proxmox-backup/26-password-and-email.png)

W przedostatnim kroku konfigurujemy sieć. Proxmox zaproponować aktualną konfigurację.

![28-network.png](/assets/images/proxmox-backup/28-network.png)

Na koniec pozostaje nam jeszcze potwierdzić wprowadzone informacje na ekranie podsumowania w formie tabeli.

![30-summary.png](/assets/images/proxmox-backup/30-summary.png)

Podczas instalacji Proxmox Backup Server na bieżąco informuje nas o postępach, ale także podsumowuje
najważniejsze funkcje tego rozwiązania.

![32-installation.png](/assets/images/proxmox-backup/32-installation.png)

Po zakończeniu procesu maszyna została automatycznie zrestartowana. Trzeba pamiętać o wyjęciu płyty CD z napędu
lub zmienić kolejność boootowania, wskazując na dysk, na którym zainstalowano PBS.

# Uruchomienie

Po uruchomieniu maszyny możemy się zalogować panelu PBS. Strona www jest dostępną pod wcześniej skonfigurowanym adresem ip,
na porcie 8007, w moim przypadku skonfigurowałem adres [https://backup.lan:8007/](https://backup.lan:8007/). 
Następnie mamy do dyspozycji pulpit, bliźniaczo przypominający PVE.

![90-dashboard.png](/assets/images/proxmox-backup/90-dashboard.png)

# Konfiguracja

#### Domena i Certyfikat

Na początku, skonfigurowałem domenę i [wygenerowałem certyfikat]({% post_url 2025-10-30-tworzenie-wlasnego-ca-na-potrzeby-generowania-certyfikatow-ssl %}),
którym będę się posługiwał na swoim środowisku. Do obsługi DNS aktualnie wykorzystuje Pi-hole, certyfikat możemy podegrać
bezpośrednio z interfejsu www — [HTTPS Certificate Configuration](https://pbs.proxmox.com/wiki/HTTPS_Certificate_Configuration).

![91-certificate.png](/assets/images/proxmox-backup/91-certificate.png)

#### Dysk

Utworzyłem dedykowany dysk, który wykorzystam do przechowywania backupów. Na dysku stworzyłem katalog `pve`.

![92-disk.png](/assets/images/proxmox-backup/92-disk.png)

#### Datastore

Katalog został automatycznie wykorzystany jako datastore z nazwą `pve`.

![94-disk.png](/assets/images/proxmox-backup/94-datastore.png)

#### Użytkownik

Utworzyłem dedykowanego użytkownika. Będzie on miał uprawnienia z roli `DatastoreAdmin` do ścieżki `/datastore/pve`.

![96-permissions.png](/assets/images/proxmox-backup/96-permissions.png)

#### Storage

Po stronie PVE należy skonfigurować wcześniej przygotowany w PBS storage do magazynowania backupu. Do autoryzacji posłużymy
się założonym przed chwilą kontem. Na tym etapie mamy możliwość konfiguracji retencji przechowywania danych
oraz wprowadzenia ewentualnego szyfrowania. Zrobimy to w sekcji `Datacenter` — [Storage: Proxmox Backup Server](https://pve.proxmox.com/wiki/Storage:_Proxmox_Backup_Server).

![97-storage.png](/assets/images/proxmox-backup/97-storage.png)

#### Backup

W konfiguracji `Backup` dla mojego Node `pve`, skonfigurowałem dodatkową sekcję tworzenia backupu zgodnie z harmonogramem.
Wybrałem do tego wcześniej przygotowany storage `pbe`, a następnie wykonałem próbne uruchomienie. 

![98-backup.png](/assets/images/proxmox-backup/98-backup.png)


#### Podgląd

Po wykonaniu kopii mamy możliwość podglądu każdego wykonanego backupu PBS w sekcji `Datatore` -> `pve` -> `Content`.

![99-content.png](/assets/images/proxmox-backup/99-content.png)

# Podsumowanie

PBS pozwala na rozbudowę naszego Laba, o dedykowane narzędzie do wykonania backupu. Dzięki niemu zyskujemy backupy przyrostowe
oraz deduplikację, co pozwoli na ograniczenie potrzebnego miejsca na przechowywanie kopii zapasowych.
Nasze kopie możemy też synchronizować do zdalnych lokalizacji, jak i zaszyfrować, co skonfigurujemy bezpośrednio z interfejsu PBS.
Istotną funkcją jest także możliwość przywrócenia tylko wybranych plików bezpośrednio z wybranego backupu, co w przypadku
podstawowej funkcji PVE wymaga przywrócenia całego dysku — [Restoring Data](https://pbs.proxmox.com/docs/backup-client.html#restoring-data).
