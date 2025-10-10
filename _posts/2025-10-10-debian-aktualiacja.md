---
layout: post
title: "Aktualizacja Debian 12.7 (bookworm) do 13.1 (trixie)"
---

## Sprawdzenie aktualnej wersji

```
debian@nomad-client:~$ cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
debian@nomad-client:~$ cat /etc/debian_version 
12.7
```

## Aktualizacja repozytoriów

```
debian@nomad-client:~$ sudo apt update
Pobieranie:1 http://security.debian.org/debian-security bookworm-security InRelease [48,0 kB]
Stary:2 http://deb.debian.org/debian bookworm InRelease                                                                                                      
Pobieranie:3 http://deb.debian.org/debian bookworm-updates InRelease [55,4 kB]                                                                               
Pobieranie:4 https://download.docker.com/linux/debian bookworm InRelease [46,6 kB]                                                                           
Pobieranie:5 https://apt.releases.hashicorp.com bookworm InRelease [12,9 kB]                                                                                 
Pobieranie:6 https://pkgs.tailscale.com/stable/debian bookworm InRelease                                                                                     
Pobieranie:7 http://security.debian.org/debian-security bookworm-security/main Sources [161 kB]
Pobieranie:8 https://download.docker.com/linux/debian bookworm/stable amd64 Packages [49,0 kB]
Pobieranie:9 https://apt.releases.hashicorp.com bookworm/main amd64 Packages [205 kB]
Pobieranie:10 http://security.debian.org/debian-security bookworm-security/main amd64 Packages [281 kB]
Pobieranie:11 http://security.debian.org/debian-security bookworm-security/main Translation-en [170 kB]
Pobieranie:12 https://pkgs.tailscale.com/stable/debian bookworm/main amd64 Packages [13,2 kB]
Pobrano 1 048 kB w 2s (502 kB/s)                       
Czytanie list pakietów... Gotowe
Budowanie drzewa zależności... Gotowe
Odczyt informacji o stanie... Gotowe   
121 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

## Sprawdzenie pakietów do aktualizacji

```
debian@nomad-client:~$ apt list --upgradable
Listing... Gotowe
base-files/oldstable 12.4+deb12u12 amd64 [upgradable from: 12.4+deb12u7]
bash/oldstable 5.2.15-2+b9 amd64 [upgradable from: 5.2.15-2+b7]
bind9-dnsutils/oldstable,oldstable-security 1:9.18.33-1~deb12u2 amd64 [upgradable from: 1:9.18.28-1~deb12u2]
bind9-host/oldstable,oldstable-security 1:9.18.33-1~deb12u2 amd64 [upgradable from: 1:9.18.28-1~deb12u2]
bind9-libs/oldstable,oldstable-security 1:9.18.33-1~deb12u2 amd64 [upgradable from: 1:9.18.28-1~deb12u2]
bsdextrautils/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
bsdutils/oldstable 1:2.38.1-5+deb12u3 amd64 [upgradable from: 1:2.38.1-5+deb12u1]
busybox/oldstable 1:1.35.0-4+b5 amd64 [upgradable from: 1:1.35.0-4+b3]
ca-certificates/oldstable,oldstable-updates 20230311+deb12u1 all [upgradable from: 20230311]
consul/bookworm 1.21.5-1 amd64 [upgradable from: 1.21.1-1]
containerd.io/bookworm 1.7.28-1~debian.12~bookworm amd64 [upgradable from: 1.7.21-1]
curl/oldstable 7.88.1-10+deb12u14 amd64 [upgradable from: 7.88.1-10+deb12u7]
debian-archive-keyring/oldstable 2023.3+deb12u2 all [upgradable from: 2023.3+deb12u1]
dirmngr/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
distro-info-data/oldstable 0.58+deb12u5 all [upgradable from: 0.58+deb12u2]
docker-buildx-plugin/bookworm 0.29.1-1~debian.12~bookworm amd64 [upgradable from: 0.16.2-1~debian.12~bookworm]
docker-ce-cli/bookworm 5:28.5.1-1~debian.12~bookworm amd64 [upgradable from: 5:27.2.0-1~debian.12~bookworm]
docker-ce-rootless-extras/bookworm 5:28.5.1-1~debian.12~bookworm amd64 [upgradable from: 5:27.2.0-1~debian.12~bookworm]
docker-ce/bookworm 5:28.5.1-1~debian.12~bookworm amd64 [upgradable from: 5:27.2.0-1~debian.12~bookworm]
docker-compose-plugin/bookworm 2.40.0-1~debian.12~bookworm amd64 [upgradable from: 2.29.2-1~debian.12~bookworm]
e2fsprogs/oldstable 1.47.0-2+b2 amd64 [upgradable from: 1.47.0-2]
eject/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
fdisk/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
gcc-12-base/oldstable 12.2.0-14+deb12u1 amd64 [upgradable from: 12.2.0-14]
git-man/oldstable,oldstable-security 1:2.39.5-0+deb12u2 all [upgradable from: 1:2.39.2-1.1]
git/oldstable,oldstable-security 1:2.39.5-0+deb12u2 amd64 [upgradable from: 1:2.39.2-1.1]
gnupg-l10n/oldstable 2.2.40-1.1+deb12u1 all [upgradable from: 2.2.40-1.1]
gnupg-utils/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
gnupg/oldstable 2.2.40-1.1+deb12u1 all [upgradable from: 2.2.40-1.1]
gpg-agent/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
gpg-wks-client/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
gpg-wks-server/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
gpg/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
gpgconf/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
gpgsm/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
gpgv/oldstable 2.2.40-1.1+deb12u1 amd64 [upgradable from: 2.2.40-1.1]
init-system-helpers/oldstable 1.65.2+deb12u1 all [upgradable from: 1.65.2]
init/oldstable 1.65.2+deb12u1 amd64 [upgradable from: 1.65.2]
initramfs-tools-core/oldstable 0.142+deb12u3 all [upgradable from: 0.142+deb12u1]
initramfs-tools/oldstable 0.142+deb12u3 all [upgradable from: 0.142+deb12u1]
intel-microcode/oldstable,oldstable-security 3.20250512.1~deb12u1 amd64 [upgradable from: 3.20240813.1~deb12u1]
iputils-ping/oldstable 3:20221126-1+deb12u1 amd64 [upgradable from: 3:20221126-1]
krb5-locales/oldstable 1.20.1-2+deb12u4 all [upgradable from: 1.20.1-2+deb12u2]
libblkid1/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
libbpf1/oldstable 1:1.1.2-0+deb12u1 amd64 [upgradable from: 1:1.1.0-1]
libc-bin/oldstable 2.36-9+deb12u13 amd64 [upgradable from: 2.36-9+deb12u8]
libc-l10n/oldstable 2.36-9+deb12u13 all [upgradable from: 2.36-9+deb12u8]
libc6/oldstable 2.36-9+deb12u13 amd64 [upgradable from: 2.36-9+deb12u8]
libcap2-bin/oldstable 1:2.66-4+deb12u2 amd64 [upgradable from: 1:2.66-4]
libcap2/oldstable 1:2.66-4+deb12u2 amd64 [upgradable from: 1:2.66-4]
libcom-err2/oldstable 1.47.0-2+b2 amd64 [upgradable from: 1.47.0-2]
libcurl3-gnutls/oldstable 7.88.1-10+deb12u14 amd64 [upgradable from: 7.88.1-10+deb12u7]
libcurl4/oldstable 7.88.1-10+deb12u14 amd64 [upgradable from: 7.88.1-10+deb12u7]
libexpat1/oldstable 2.5.0-1+deb12u2 amd64 [upgradable from: 2.5.0-1]
libext2fs2/oldstable 1.47.0-2+b2 amd64 [upgradable from: 1.47.0-2]
libfdisk1/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
libfreetype6/oldstable,oldstable-security 2.12.1+dfsg-5+deb12u4 amd64 [upgradable from: 2.12.1+dfsg-5+deb12u3]
libgcc-s1/oldstable 12.2.0-14+deb12u1 amd64 [upgradable from: 12.2.0-14]
libglib2.0-0/oldstable 2.74.6-2+deb12u7 amd64 [upgradable from: 2.74.6-2+deb12u3]
libglib2.0-data/oldstable 2.74.6-2+deb12u7 all [upgradable from: 2.74.6-2+deb12u3]
libgnutls30/oldstable,oldstable-security 3.7.9-2+deb12u5 amd64 [upgradable from: 3.7.9-2+deb12u3]
libgssapi-krb5-2/oldstable 1.20.1-2+deb12u4 amd64 [upgradable from: 1.20.1-2+deb12u2]
libicu72/oldstable,oldstable-security 72.1-3+deb12u1 amd64 [upgradable from: 72.1-3]
libk5crypto3/oldstable 1.20.1-2+deb12u4 amd64 [upgradable from: 1.20.1-2+deb12u2]
libkrb5-3/oldstable 1.20.1-2+deb12u4 amd64 [upgradable from: 1.20.1-2+deb12u2]
libkrb5support0/oldstable 1.20.1-2+deb12u4 amd64 [upgradable from: 1.20.1-2+deb12u2]
liblzma5/oldstable,oldstable-security 5.4.1-1 amd64 [upgradable from: 5.4.1-0.2]
libmount1/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
libnghttp2-14/oldstable 1.52.0-1+deb12u2 amd64 [upgradable from: 1.52.0-1+deb12u1]
libnss-systemd/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
libpam-systemd/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
libperl5.36/oldstable 5.36.0-7+deb12u3 amd64 [upgradable from: 5.36.0-7+deb12u1]
libpython3.11-minimal/oldstable 3.11.2-6+deb12u6 amd64 [upgradable from: 3.11.2-6+deb12u3]
libpython3.11-stdlib/oldstable 3.11.2-6+deb12u6 amd64 [upgradable from: 3.11.2-6+deb12u3]
libsmartcols1/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
libsqlite3-0/oldstable 3.40.1-2+deb12u2 amd64 [upgradable from: 3.40.1-2]
libss2/oldstable 1.47.0-2+b2 amd64 [upgradable from: 1.47.0-2]
libssl3/oldstable-security 3.0.17-1~deb12u3 amd64 [upgradable from: 3.0.14-1~deb12u1]
libstdc++6/oldstable 12.2.0-14+deb12u1 amd64 [upgradable from: 12.2.0-14]
libsystemd-shared/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
libsystemd0/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
libtasn1-6/oldstable,oldstable-security 4.19.0-2+deb12u1 amd64 [upgradable from: 4.19.0-2]
libudev1/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
libuuid1/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
libxml2/oldstable,oldstable-security 2.9.14+dfsg-1.3~deb12u4 amd64 [upgradable from: 2.9.14+dfsg-1.3~deb12u1]
linux-image-amd64/oldstable-security 6.1.153-1 amd64 [upgradable from: 6.1.106-3]
locales/oldstable 2.36-9+deb12u13 all [upgradable from: 2.36-9+deb12u8]
login/oldstable 1:4.13+dfsg1-1+deb12u1 amd64 [upgradable from: 1:4.13+dfsg1-1+b1]
logsave/oldstable 1.47.0-2+b2 amd64 [upgradable from: 1.47.0-2]
mount/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
nomad/bookworm 1.10.5-1 amd64 [upgradable from: 1.10.2-1]
openssh-client/oldstable,oldstable-updates 1:9.2p1-2+deb12u7 amd64 [upgradable from: 1:9.2p1-2+deb12u3]
openssh-server/oldstable,oldstable-updates 1:9.2p1-2+deb12u7 amd64 [upgradable from: 1:9.2p1-2+deb12u3]
openssh-sftp-server/oldstable,oldstable-updates 1:9.2p1-2+deb12u7 amd64 [upgradable from: 1:9.2p1-2+deb12u3]
openssl/oldstable-security 3.0.17-1~deb12u3 amd64 [upgradable from: 3.0.14-1~deb12u1]
passwd/oldstable 1:4.13+dfsg1-1+deb12u1 amd64 [upgradable from: 1:4.13+dfsg1-1+b1]
perl-base/oldstable 5.36.0-7+deb12u3 amd64 [upgradable from: 5.36.0-7+deb12u1]
perl-modules-5.36/oldstable 5.36.0-7+deb12u3 all [upgradable from: 5.36.0-7+deb12u1]
perl/oldstable 5.36.0-7+deb12u3 amd64 [upgradable from: 5.36.0-7+deb12u1]
python3-pkg-resources/oldstable 66.1.1-1+deb12u2 all [upgradable from: 66.1.1-1]
python3-urllib3/oldstable 1.26.12-1+deb12u1 all [upgradable from: 1.26.12-1]
python3.11-minimal/oldstable 3.11.2-6+deb12u6 amd64 [upgradable from: 3.11.2-6+deb12u3]
python3.11/oldstable 3.11.2-6+deb12u6 amd64 [upgradable from: 3.11.2-6+deb12u3]
qemu-guest-agent/oldstable 1:7.2+dfsg-7+deb12u16 amd64 [upgradable from: 1:7.2+dfsg-7+deb12u13]
sudo/oldstable,oldstable-security 1.9.13p3-1+deb12u2 amd64 [upgradable from: 1.9.13p3-1+deb12u1]
systemd-sysv/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
systemd-timesyncd/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
systemd/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
tailscale/unknown 1.88.3 amd64 [upgradable from: 1.88.1]
tzdata/oldstable 2025b-0+deb12u2 all [upgradable from: 2024a-0+deb12u1]
ucf/oldstable 3.0043+nmu1+deb12u1 all [upgradable from: 3.0043+nmu1]
udev/oldstable 252.39-1~deb12u1 amd64 [upgradable from: 252.30-1~deb12u2]
util-linux-extra/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
util-linux-locales/oldstable 2.38.1-5+deb12u3 all [upgradable from: 2.38.1-5+deb12u1]
util-linux/oldstable 2.38.1-5+deb12u3 amd64 [upgradable from: 2.38.1-5+deb12u1]
vim-common/oldstable 2:9.0.1378-2+deb12u2 all [upgradable from: 2:9.0.1378-2]
vim-runtime/oldstable 2:9.0.1378-2+deb12u2 all [upgradable from: 2:9.0.1378-2]
vim-tiny/oldstable 2:9.0.1378-2+deb12u2 amd64 [upgradable from: 2:9.0.1378-2]
vim/oldstable 2:9.0.1378-2+deb12u2 amd64 [upgradable from: 2:9.0.1378-2]
wget/oldstable 1.21.3-1+deb12u1 amd64 [upgradable from: 1.21.3-1+b2]
xz-utils/oldstable,oldstable-security 5.4.1-1 amd64 [upgradable from: 5.4.1-0.2]
```

## Aktualizacja paczek

Podczas aktualizacji zostanie zrestartowana usługa docker, co spowoduje ponowne uruchomienie kontenerów. Usługa Nomad oraz Consul powinna zostać zakutalizowana do najnowszej wersji bez problemów. Dodatkowo zostanie zaktualizowany kernel, który wymagać będzie restartu maszyny.


```
debian@nomad-client:~$ sudo apt upgrade
Czytanie list pakietów... Gotowe
Budowanie drzewa zależności... Gotowe
Odczyt informacji o stanie... Gotowe   
Obliczanie aktualizacji... Gotowe
Następujący pakiet został zainstalowany automatycznie i nie jest już więcej wymagany:
  libltdl7
Aby go usunąć należy użyć "sudo apt autoremove".
Zostaną zainstalowane następujące NOWE pakiety:
  linux-image-6.1.0-40-amd64
Następujące pakiety zostaną zaktualizowane:
  base-files bash bind9-dnsutils bind9-host bind9-libs bsdextrautils bsdutils busybox ca-certificates consul containerd.io curl debian-archive-keyring
  dirmngr distro-info-data docker-buildx-plugin docker-ce docker-ce-cli docker-ce-rootless-extras docker-compose-plugin e2fsprogs eject fdisk gcc-12-base
  git git-man gnupg gnupg-l10n gnupg-utils gpg gpg-agent gpg-wks-client gpg-wks-server gpgconf gpgsm gpgv init init-system-helpers initramfs-tools
  initramfs-tools-core intel-microcode iputils-ping krb5-locales libblkid1 libbpf1 libc-bin libc-l10n libc6 libcap2 libcap2-bin libcom-err2 libcurl3-gnutls
  libcurl4 libexpat1 libext2fs2 libfdisk1 libfreetype6 libgcc-s1 libglib2.0-0 libglib2.0-data libgnutls30 libgssapi-krb5-2 libicu72 libk5crypto3 libkrb5-3
  libkrb5support0 liblzma5 libmount1 libnghttp2-14 libnss-systemd libpam-systemd libperl5.36 libpython3.11-minimal libpython3.11-stdlib libsmartcols1
  libsqlite3-0 libss2 libssl3 libstdc++6 libsystemd-shared libsystemd0 libtasn1-6 libudev1 libuuid1 libxml2 linux-image-amd64 locales login logsave mount
  nomad openssh-client openssh-server openssh-sftp-server openssl passwd perl perl-base perl-modules-5.36 python3-pkg-resources python3-urllib3 python3.11
  python3.11-minimal qemu-guest-agent sudo systemd systemd-sysv systemd-timesyncd tailscale tzdata ucf udev util-linux util-linux-extra util-linux-locales
  vim vim-common vim-runtime vim-tiny wget xz-utils
121 aktualizowanych, 1 nowo instalowanych, 0 usuwanych i 0 nieaktualizowanych.
Konieczne pobranie 438 MB archiwów.
Po tej operacji zostanie dodatkowo użyte 419 MB miejsca na dysku.
Kontynuować? [T/n] 

...

Przetwarzanie wyzwalaczy pakietu libc-bin (2.36-9+deb12u13)...
Przetwarzanie wyzwalaczy pakietu systemd (252.39-1~deb12u1)...
Przetwarzanie wyzwalaczy pakietu man-db (2.11.2-2)...
Przetwarzanie wyzwalaczy pakietu dbus (1.14.10-1~deb12u1)...
Przetwarzanie wyzwalaczy pakietu debianutils (5.7-0.5~deb12u1)...
Przetwarzanie wyzwalaczy pakietu mailcap (3.70+nmu1)...
Przetwarzanie wyzwalaczy pakietu initramfs-tools (0.142+deb12u3)...
update-initramfs: Generating /boot/initrd.img-6.1.0-40-amd64
Przetwarzanie wyzwalaczy pakietu ca-certificates (20230311+deb12u1)...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.

```

## Sprawdznie wersji Debiana

Po aktualizacji system został podniesiony do wersji 12.12, co jest wymagane przy przejściu na wersję 13 (trixie).

```
debian@nomad-client:~$ cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
debian@nomad-client:~$ cat /etc/debian_version 
12.12
```

## Zmiana repozytoriów

Aby przejść do aktualizacji repozytoriów na wersję trixi, należy w konfiguracji `/etc/apt/source.list` podmienić wersję w plikach `source.list`.

```
debian@nomad-client:~$ sudo sed -i 's/bookworm/trixie/g' /etc/apt/sources.list /etc/apt/sources.list.d/*
debian@nomad-client:~$ cat /etc/apt/sources.list /etc/apt/sources.list.d/*
#deb cdrom:[Debian GNU/Linux 12.7.0 _Bookworm_ - Official amd64 NETINST with firmware 20240831-10:38]/ trixie contrib main non-free-firmware

deb http://deb.debian.org/debian/ trixie main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie main non-free-firmware

deb http://security.debian.org/debian-security trixie-security main non-free-firmware
deb-src http://security.debian.org/debian-security trixie-security main non-free-firmware

# trixie-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://deb.debian.org/debian/ trixie-updates main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie-updates main non-free-firmware

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
# see the sources.list(5) manual.
deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian   trixie stable
deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com trixie main

# Tailscale packages for debian trixie
deb [signed-by=/usr/share/keyrings/tailscale-archive-keyring.gpg] https://pkgs.tailscale.com/stable/debian trixie main
```

## Ponownie aktualizujemy repozytoria

```
debian@nomad-client:~$ sudo apt update
Stary:1 http://security.debian.org/debian-security trixie-security InRelease
Stary:2 http://deb.debian.org/debian trixie InRelease                                                                                                        
Stary:3 http://deb.debian.org/debian trixie-updates InRelease                                                                                                
Stary:4 https://apt.releases.hashicorp.com trixie InRelease                                                                                                  
Stary:5 https://download.docker.com/linux/debian trixie InRelease                                                                                            
Pobieranie:6 https://pkgs.tailscale.com/stable/debian trixie InRelease           
Pobrano 6 579 B w 2s (3 927 B/s)                     
Czytanie list pakietów... Gotowe
Budowanie drzewa zależności... Gotowe
Odczyt informacji o stanie... Gotowe   
341 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

## Aktualizacja pakietów

Podczas aktualizacji pakietów otrzymujemy informację, że następujące usługi zostaną zrestartowane.

```
Restarting services possibly affected by the upgrade:
  ssh: restarting...done.
  cron: restarting...done.
```


Na koniec otrzymujmy informacje o zakończeniu aktualizacji.

```
Przetwarzanie wyzwalaczy pakietu debianutils (5.23.2)...
Przetwarzanie wyzwalaczy pakietu initramfs-tools (0.142+deb12u3)...
update-initramfs: Generating /boot/initrd.img-6.12.48+deb13-amd64
Przetwarzanie wyzwalaczy pakietu libc-bin (2.41-12)...
Przetwarzanie wyzwalaczy pakietu man-db (2.11.2-2)...
Przetwarzanie wyzwalaczy pakietu ca-certificates (20250419)...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
Przetwarzanie wyzwalaczy pakietu dictionaries-common (1.30.10)...
ispell-autobuildhash: Processing 'polish' dict.
aspell-autobuildhash: processing: pl [pl].
```

## Sprawdzenie wersji

Ponowne sprawdzenie wersji pozwala potwierdzić przejście na Debian 13 (trixie).

```
debian@nomad-client:~$ cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
VERSION_ID="13"
VERSION="13 (trixie)"
VERSION_CODENAME=trixie
DEBIAN_VERSION_FULL=13.1
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
debian@nomad-client:~$ cat /etc/debian_version 
13.1
```

## Full upgrade

Na koniec zostało wykonanie `apt full-upgrade`.

```
debian@nomad-client:~$ sudo apt full-upgrade 
Czytanie list pakietów... Gotowe
Budowanie drzewa zależności... Gotowe
Odczyt informacji o stanie... Gotowe   
Obliczanie aktualizacji... Gotowe
Następujące pakiety zostały zainstalowane automatycznie i nie są już więcej wymagane:
  libassuan0 libcbor0.8 libfuse2 libicu72 libldap-2.5-0 libltdl7 libnl-3-200 libnl-genl-3-200 libperl5.36 libpython3.11-minimal libpython3.11-stdlib
  perl-modules-5.36 python3-autocommand python3-httplib2 python3-inflect python3-jaraco.context python3-jaraco.functools python3-more-itertools
  python3-pkg-resources python3-pycurl python3-pyparsing python3-pysimplesoap python3-six python3-typeguard python3-typing-extensions python3.11
  python3.11-minimal
Aby je usunąć należy użyć "sudo apt autoremove".
Następujące pakiety zostaną USUNIĘTE:
  libcurl3-gnutls libcurl4 libdb5.3 libefiboot1 libefivar1 libelf1 libext2fs2 libgdbm-compat4 libgdbm6 libglib2.0-0 libgnutls30 libhogweed6 libmagic1
  libnettle8 libnpth0 libpng16-16 libpsl5 libreadline8 libssh2-1 libssl3 libtirpc3 libuv1 linux-image-6.1.0-40-amd64
Zostaną zainstalowane następujące NOWE pakiety:
  bsd-mailx dracut-install exim4-base exim4-config exim4-daemon-light initramfs-tools-bin libapt-pkg7.0 libatomic1 libcbor0.10 libcurl3t64-gnutls
  libcurl4t64 libdb5.3t64 libefiboot1t64 libefivar1t64 libelf1t64 libevent-2.1-7t64 libext2fs2t64 libfile-fcntllock-perl libfuse3-4 libgdbm-compat4t64
  libgdbm6t64 libglib2.0-0t64 libgnutls-dane0t64 libgnutls30t64 libhogweed6t64 libidn12 libldap2 liblockfile1 liblsof0 libmagic1t64 libnettle8t64
  libnghttp3-9 libngtcp2-16 libngtcp2-crypto-gnutls8 libnpth0t64 libperl5.40 libpng16-16t64 libpsl5t64 libpython3.13-minimal libpython3.13-stdlib
  libreadline8t64 libssh2-1t64 libssl3t64 libtirpc3t64 libunbound8 liburcu8t64 libuv1t64 libwtmpdb0 openssl-provider-legacy perl-modules-5.40
  python3-autocommand python3-inflect python3-jaraco.context python3-jaraco.functools python3-more-itertools python3-typeguard python3-typing-extensions
  python3.13 python3.13-minimal sqv systemd-cryptsetup
Następujące pakiety zostaną zaktualizowane:
  apt apt-utils bind9-dnsutils bind9-host bind9-libs bsdutils coreutils curl dirmngr e2fsprogs fdisk file git git-man gnupg gnupg-l10n gnupg-utils gpg
  gpg-agent gpg-wks-client gpg-wks-server gpgconf gpgsm gpgv grub-common grub-pc grub-pc-bin grub2-common initramfs-tools initramfs-tools-core iproute2 kmod
  libbpf1 libcryptsetup12 libfido2-1 libfreetype6 libgssapi-krb5-2 libk5crypto3 libkmod2 libkrb5-3 libkrb5support0 liblocale-gettext-perl libmagic-mgc
  libnsl2 libnss-systemd libpam-modules libpam-modules-bin libpam-runtime libpam-systemd libpython3-stdlib librtmp1 libsasl2-2 libsasl2-modules
  libsasl2-modules-db libslirp0 libsystemd-shared libsystemd0 libtext-charwidth-perl libtext-iconv-perl libudev1 lsof man-db mc mc-data openssh-client
  openssh-server openssh-sftp-server openssl perl perl-base procps python3 python3-apt python3-charset-normalizer python3-minimal python3-pkg-resources
  python3-pycurl qemu-guest-agent shared-mime-info slirp4netns sudo systemd systemd-sysv systemd-timesyncd udev wget zabbix-agent2
87 aktualizowanych, 61 nowo instalowanych, 23 usuwanych i 0 nieaktualizowanych.
Konieczne pobranie 88,3 MB archiwów.
Po tej operacji zostanie zwolnione 299 MB miejsca na dysku.
Kontynuować? [T/n] 
```

## Restart systemu 

Na koniec pozostało zrestartować serwer celem zweryfikowania zmian grub oraz przeładowania
nowej wersji krenela.

```
debian@nomad-client:~$ sudo reboot 

Broadcast message from root@nomad-client.lan on pts/1 (Fri 2025-10-10 22:55:54 CEST):

The system will reboot now!
```

## Weryfikacja dostępnych aktualizacji

Tym razem próba zaktualizowania pakietów powinna zakończyć się informacją, że
wszystkie pakiety są w najnowszych wersjach.

```
debian@nomad-client:~$ sudo apt update
Było:1 http://deb.debian.org/debian trixie InRelease
Pobr:2 http://deb.debian.org/debian trixie-updates InRelease [47,3 kB]                                                                                       
Było:3 http://security.debian.org/debian-security trixie-security InRelease                                                                                  
Było:4 https://download.docker.com/linux/debian trixie InRelease                                                                                             
Było:5 https://apt.releases.hashicorp.com trixie InRelease                                                                                                   
Pobr:6 https://pkgs.tailscale.com/stable/debian trixie InRelease                  
Pobrano 53,9 kB w 1s (74,1 kB/s)
Wszystkie pakiety są aktualne.       
debian@nomad-client:~$ sudo apt full-upgrade 
Podsumowanie:                        
  Aktualizowanych: 0, Instalowanych: 0, Usuwanych: 0, Nieaktualizowanych: 0
```







