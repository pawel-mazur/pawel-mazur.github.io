---
title: "Aktualiacja usług Docker, Nomad oraz Consul w praktyce"
tags: aktualizacja,docker, nomad, consul
---

Aktualizacja usług to naturalna kolej rzeczy jeśli chcemy być na bieżąco z nowymi funkcjonalnościami, a co ważniejsze
z naprawionymi błędami i poprawkami bezpieczeństawa celem wyeleminiowania podatności. Jako kolejny krok zw. z aktualizacją systemu,
warto więc też zaktualizować wykorzystywane usług, tym razem będzie środowisko DevOps.

# Aktualiazacja repozytoriów

Przed aktualizacja w środowisku Debian należy zaktualizować informację o dostępnych paczkach do popbrania. Zrobimy to
poleceniem `apt update` lub `apt-get update`. W moim wypadku wyglądało to następująco.

```
debian@vps-3312ca9a:~$ sudo apt-get update 
Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:5 file:/etc/apt/mirrors/debian-security.list Mirrorlist [39 B]       
Hit:7 https://download.docker.com/linux/debian bookworm InRelease        
Get:8 https://apt.releases.hashicorp.com bookworm InRelease [12.9 kB]    
Hit:2 https://deb.debian.org/debian bookworm InRelease                   
Hit:3 https://deb.debian.org/debian bookworm-updates InRelease           
Hit:4 https://deb.debian.org/debian bookworm-backports InRelease         
Hit:6 https://deb.debian.org/debian-security bookworm-security InRelease 
Get:9 https://pkgs.tailscale.com/stable/debian bookworm InRelease        
Hit:10 https://repo.zabbix.com/zabbix/7.2/release/debian bookworm InRelease
Hit:11 https://repo.zabbix.com/zabbix-tools/debian-ubuntu bookworm InRelease
Hit:12 https://repo.zabbix.com/zabbix/7.2/stable/debian bookworm InRelease
Fetched 19.5 kB in 2s (12.3 kB/s)
Reading package lists... Done
```

# Aktualizacja Docker

Na czas aktualizacji usługi Docker należy zaplanowąć okno serwisowe. W tym czasie usługa ta zostanie zrestartowana,
a wszystkie uruchomione kontenery zostaną zatrzyman i uruchomione ponownie po zakończonej aktualizacji.

Sama procedura nie różni się od procesu instalacji, co zostało opisane w oficjalnej dokumentacji [Upgrade Docker Engine](https://docs.docker.com/engine/install/ubuntu/#upgrade-docker-engine).

Przed aktualizacją warto przejrzeć changelog i zweryfikować jakie zmiany zostały wprowadzone w kolejnej wersji. Można go znaleźć tutaj [Docker Engine version 28 release notes](https://docs.docker.com/engine/release-notes/).

Przed aktualizacja sprawdzimy obecnie zainstalowaną wersję narzędzia Docker.

```
debian@vps-3312ca9a:~$ docker -v
Docker version 24.0.7, build afdd53b
```

Wykonamy aktualizację z wykorzystaniem polecenia `apt install docker-ce`

```
debian@vps-3312ca9a:~$ sudo apt install docker-ce
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following package was automatically installed and is no longer required:
  libltdl7
Use 'sudo apt autoremove' to remove it.
The following additional packages will be installed:
  containerd.io docker-ce-cli docker-ce-rootless-extras
Suggested packages:
  cgroupfs-mount | cgroup-lite docker-model-plugin
The following packages will be upgraded:
  containerd.io docker-ce docker-ce-cli docker-ce-rootless-extras
4 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
Need to get 74.6 MB of archives.
After this operation, 11.6 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://download.docker.com/linux/debian bookworm/stable amd64 docker-ce-cli amd64 5:28.5.1-1~debian.12~bookworm [16.5 MB]
Get:2 https://download.docker.com/linux/debian bookworm/stable amd64 containerd.io amd64 1.7.28-1~debian.12~bookworm [31.9 MB]
Get:3 https://download.docker.com/linux/debian bookworm/stable amd64 docker-ce amd64 5:28.5.1-1~debian.12~bookworm [19.7 MB]
Get:4 https://download.docker.com/linux/debian bookworm/stable amd64 docker-ce-rootless-extras amd64 5:28.5.1-1~debian.12~bookworm [6481 kB]
Fetched 74.6 MB in 3s (29.6 MB/s)               
Reading changelogs... Done
(Reading database ... 37323 files and directories currently installed.)
Preparing to unpack .../docker-ce-cli_5%3a28.5.1-1~debian.12~bookworm_amd64.deb ...
Unpacking docker-ce-cli (5:28.5.1-1~debian.12~bookworm) over (5:24.0.7-1~debian.12~bookworm) ...
Preparing to unpack .../containerd.io_1.7.28-1~debian.12~bookworm_amd64.deb ...
Unpacking containerd.io (1.7.28-1~debian.12~bookworm) over (1.6.25-1) ...
Preparing to unpack .../docker-ce_5%3a28.5.1-1~debian.12~bookworm_amd64.deb ...
Unpacking docker-ce (5:28.5.1-1~debian.12~bookworm) over (5:24.0.7-1~debian.12~bookworm) ...
Preparing to unpack .../docker-ce-rootless-extras_5%3a28.5.1-1~debian.12~bookworm_amd64.deb ...
Unpacking docker-ce-rootless-extras (5:28.5.1-1~debian.12~bookworm) over (5:24.0.7-1~debian.12~bookworm) ...
Setting up containerd.io (1.7.28-1~debian.12~bookworm) ...
Setting up docker-ce-cli (5:28.5.1-1~debian.12~bookworm) ...
Setting up docker-ce-rootless-extras (5:28.5.1-1~debian.12~bookworm) ...
Setting up docker-ce (5:28.5.1-1~debian.12~bookworm) ...
Installing new version of config file /etc/init.d/docker ...
Processing triggers for man-db (2.11.2-2) ...
```

W tym momencie usługa docker została zrestartowana, a kontenery zostały ponownie uruchomione.

![docker-restart.png](/assets/images/docker-restart.png)

Obecnie docker został zaktualizowany do najnowszej wersji:

```
debian@vps-3312ca9a:~$ docker -v
Docker version 28.5.1, build e180ab8
```

Potwierdzimy jeszcze status uruchomionej usługi poleceniem `service docker status`:

```
debian@vps-3312ca9a:~$ sudo service docker status
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; preset:>
     Active: active (running) since Thu 2025-10-23 06:09:38 CEST; 10min a>
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 2053618 (dockerd)
      Tasks: 56
     Memory: 127.3M
        CPU: 8.705s
     CGroup: /system.slice/docker.service
```

# Aktualizacja Nomad

W celu aktualizacji środowiska Nomad warto zacząć od przejścia oficjalnej dokumetacji w tym temacie. Znajdziemy ją tutaj 
[Upgrade Nomad](https://developer.hashicorp.com/nomad/docs/upgrade).

Podobnie jak w przypadku Dockera, powinniśmy zapoznać się z ostatnimi zmianami. Informacje te są dostępne na stronie 
[Specific version details](https://developer.hashicorp.com/nomad/docs/upgrade/upgrade-specific).

Sama aktualizacja sprowadza się do instalacji paczki `nomad`, w moim środowisku wyglądało to następująco.

```
debian@vps-3312ca9a:~$ sudo apt install nomad
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  dmidecode
The following NEW packages will be installed:
  dmidecode
The following packages will be upgraded:
  nomad
1 upgraded, 1 newly installed, 0 to remove and 11 not upgraded.
Need to get 55.4 MB of archives.
After this operation, 25.3 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:2 https://deb.debian.org/debian bookworm/main amd64 dmidecode amd64 3.4-1 [68.8 kB]
Get:3 https://apt.releases.hashicorp.com bookworm/main amd64 nomad amd64 1.10.5-1 [55.4 MB]
Fetched 55.4 MB in 3s (18.9 MB/s)
Reading changelogs... Done
Selecting previously unselected package dmidecode.
(Reading database ... 37305 files and directories currently installed.)
Preparing to unpack .../dmidecode_3.4-1_amd64.deb ...
Unpacking dmidecode (3.4-1) ...
Preparing to unpack .../nomad_1.10.5-1_amd64.deb ...
Unpacking nomad (1.10.5-1) over (1.7.2-1) ...
Setting up dmidecode (3.4-1) ...
Setting up nomad (1.10.5-1) ...
Installing new version of config file /usr/lib/systemd/system/nomad.service ...
Processing triggers for man-db (2.11.2-2) ...
```

W tym czasie uległa restartowi usługa `nomad` na hoście na którym przeprowadzamy aktualizację. Wg. `journalctl`,
otrzymałem następujące informacje.

```
debian@vps-3312ca9a:/opt/nomad$ journalctl -u nomad -f
Oct 23 03:55:24 vps-3312ca9a systemd[1]: Stopping nomad.service - Nomad...
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]: ==> Caught signal: interrupt
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.175+0200 [INFO]  agent: requesting shutdown
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.176+0200 [INFO]  client: shutting down
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.192+0200 [INFO]  client.plugin: shutting down plugin manager: plugin-type=device
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.193+0200 [INFO]  client.plugin: plugin manager finished: plugin-type=device
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.193+0200 [INFO]  client.plugin: shutting down plugin manager: plugin-type=driver
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.197+0200 [INFO]  client.plugin: plugin manager finished: plugin-type=driver
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.197+0200 [INFO]  client.plugin: shutting down plugin manager: plugin-type=csi
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.203+0200 [INFO]  client.plugin: plugin manager finished: plugin-type=csi
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.219+0200 [INFO]  nomad: shutting down server
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.219+0200 [WARN]  nomad: serf: Shutdown without a Leave
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.221+0200 [INFO]  nomad: cluster leadership lost
Oct 23 03:55:24 vps-3312ca9a nomad[2044205]:     2025-10-23T03:55:24.252+0200 [INFO]  agent: shutdown complete
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Main process exited, code=exited, status=1/FAILURE
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Failed with result 'exit-code'.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 1892 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 1905 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 1953 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 1955 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 2306 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 2403 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 2703 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Unit process 2704 (nomad) remains running after unit stopped.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: Stopped nomad.service - Nomad.
Oct 23 03:55:24 vps-3312ca9a systemd[1]: nomad.service: Consumed 4.845s CPU time.
```

Potwierdzienie statusu uruchomionej usługi:

```
debian@vps-3312ca9a:~$ sudo service nomad status
● nomad.service - Nomad
     Loaded: loaded (/lib/systemd/system/nomad.service; enabled; preset: >
     Active: active (running) since Thu 2025-10-23 03:55:24 CEST; 2h 26mi>
       Docs: https://nomadproject.io/docs/
   Main PID: 2045603 (nomad)
      Tasks: 73
     Memory: 237.9M
        CPU: 2min 29.172s
     CGroup: /system.slice/nomad.service
             ├─2045603 /usr/bin/nomad agent -config /etc/nomad.d
             ├─2053864 /usr/bin/nomad logmon
             ├─2053865 /usr/bin/nomad logmon
             ├─2053866 /usr/bin/nomad logmon
             ├─2053955 /usr/bin/nomad docker_logger
             ├─2053980 /usr/bin/nomad logmon
             ├─2054079 /usr/bin/nomad docker_logger
             ├─2054269 /usr/bin/nomad docker_logger
             └─2054296 /usr/bin/nomad docker_logger
```

Aplikacja po aktualizacji zwróciła aktualną wersję.

```
debian@vps-3312ca9a:~$ nomad version 
Nomad v1.10.5
BuildDate 2025-09-09T14:36:45Z
Revision a3b86c697f38ab032e1acaae8503ed10815bc4a2
```

Usługa po restarcie wskazuje prawidłowo uruchomione zadania.

```
debian@vps-3312ca9a:~$ nomad status 
ID       Type     Priority  Status   Submit Date
dotnet   service  50        running  2025-03-14T22:15:31+01:00
httpd    service  50        running  2025-04-12T02:58:33+02:00
traefik  service  50        running  2025-04-11T23:55:57+02:00

==> View and manage Nomad jobs in the Web UI: http://vps:4646//ui/jobs
```

W tym czasie żadne zadanie nie zostało przerwane.

# Aktualizacja Consul

Procedura aktualizacja usługi Consul została opisana tutaj [Upgrading Consul](https://developer.hashicorp.com/consul/docs/upgrade).

Podobnie jak wcześniej warto zapoznać się z ostatnimi zmianami, ich lista jest dostępna na stronie [Specific version details](https://developer.hashicorp.com/consul/docs/upgrade/version-specific).

Oficjalna dokumentacja, podobnie jak wypadku Nomada, zaznacza, że aktualizacja powinna być wykonana maksymalnie pomiędzy dwoma wersjami typu `minor`.

Listę dostępnych wersji sprawdzimy poleceniam `apt list -a consul`:

```
debian@vps-3312ca9a:~$ apt list -a consul
Listing... Done
consul/bookworm 1.21.5-1 amd64 [upgradable from: 1.19.0-1]
consul/bookworm 1.21.4-1 amd64
consul/bookworm 1.21.3-1 amd64
consul/bookworm 1.21.2-1 amd64
consul/bookworm 1.21.1-1 amd64
consul/bookworm 1.21.0-1 amd64
consul/bookworm 1.20.6-1 amd64
consul/bookworm 1.20.5-1 amd64
consul/bookworm 1.20.4-1 amd64
consul/bookworm 1.20.3-1 amd64
consul/bookworm 1.20.2-1 amd64
consul/bookworm 1.20.1-1 amd64
consul/bookworm 1.20.0-1 amd64
consul/bookworm 1.19.2-1 amd64
consul/bookworm 1.19.1-1 amd64
consul/bookworm,now 1.19.0-1 amd64 [installed,upgradable to: 1.21.5-1]
consul/bookworm 1.18.2-1 amd64
consul/bookworm 1.18.1-1 amd64
consul/bookworm 1.18.0-1 amd64
consul/bookworm 1.17.3-1 amd64
consul/bookworm 1.17.2-1 amd64
consul/bookworm 1.17.1-1 amd64
consul/bookworm 1.17.0-1 amd64
```

Całość sprowadza się do wykonania polecenia w moim wypadku będize to aktualizacja `v1.17.1` -> `v1.19.0` -> `v1.21.5`.

```
debian@vps-3312ca9a:~$ sudo apt install consul=1.19.0-1
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  consul
1 upgraded, 0 newly installed, 0 to remove and 10 not upgraded.
Need to get 70.5 MB of archives.
After this operation, 10.1 MB of additional disk space will be used.
Get:1 https://apt.releases.hashicorp.com bookworm/main amd64 consul amd64 1.19.0-1 [70.5 MB]
Fetched 70.5 MB in 3s (20.4 MB/s) 
Reading changelogs... Done
(Reading database ... 37321 files and directories currently installed.)
Preparing to unpack .../consul_1.19.0-1_amd64.deb ...
Unpacking consul (1.19.0-1) over (1.17.1-1) ...
Setting up consul (1.19.0-1) ...
```

Podczas drugiej aktualizacji otrzymałem informacje o zmianach w pliki konfiugracji `consul.hcl`. Zmiany są kosmetyczne i
modyfikują jednie adres dokumentacji w komentarzach. Wybrałem pozostanie przy dotychczasowej wersji konfiguracji.

```
Configuration file '/etc/consul.d/consul.hcl'
 ==> Modified (by you or by a script) since installation.
 ==> Package distributor has shipped an updated version.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      Z     : start a shell to examine the situation
 The default action is to keep your current version.
*** consul.hcl (Y/I/N/O/D/Z) [default=N] ? 
Installing new version of config file /usr/lib/systemd/system/consul.service ...
debian@vps-3312ca9a:~$ sudo apt install consul
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
consul is already the newest version (1.21.5-1).
0 upgraded, 0 newly installed, 0 to remove and 10 not upgraded.
```

Po aktualizacji możemy potwierdzić jesze zainstalowaną wersję consula.

```
debian@vps-3312ca9a:~$ consul -v
Consul v1.21.5
Revision 3261d11a
Build Date 2025-09-21T15:22:53Z
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

Oraz zweryfikować status usługi poleceniem `journalctl`

```
debian@vps-3312ca9a:~$ sudo service consul status
● consul.service - "HashiCorp Consul - A service mesh solution"
     Loaded: loaded (/lib/systemd/system/consul.service; enabled; preset:>
     Active: active (running) since Thu 2025-10-23 05:50:35 CEST; 28min a>
       Docs: https://developer.hashicorp.com/
   Main PID: 2052176 (consul)
      Tasks: 7 (limit: 2289)
     Memory: 70.4M
        CPU: 8.300s
     CGroup: /system.slice/consul.service
             └─2052176 /usr/bin/consul agent -config-dir=/etc/consul.d/
```

# Blokada wersji

Aby uniknąć przypadkowej aktualizacji pakietów podczas uruchomienia polecenia `apt upgrade`, możemy zablokować aktualizację 
poszczególnych pakietów. Zrobimy to poleceniem `apt-mark hold`. W moim przypadku będize to wyglądało następująco.

```
debian@vps-3312ca9a:~$ sudo apt-mark hold docker docker-ce nomad consul
docker set on hold.
docker-ce set on hold.
nomad set on hold.
consul set on hold.
```

Listę zablokowanych pakietów zweryfikujemy poleceniem `apt-mark showhold`

```
debian@vps-3312ca9a:~$ apt-mark showhold
consul
docker
docker-ce
nomad
```

# Podsumowanie

Aktualizacja narzędzi przeszła bezproblemowo i nie wymagała dużych nakładów pracy. Warto więc o tym pamiętać i cyklicznie 
wykonać aktualizację w zaplanowanych oknach serwisowych. Proces ten można by usprawnić i wykorzystać
m. in. [Ansible](https://docs.ansible.com/) jako narzędzie do automatyzacji.
