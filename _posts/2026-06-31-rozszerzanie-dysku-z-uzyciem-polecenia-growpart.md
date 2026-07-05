---
title: Rozszerzanie dysku z użyciem polecenia growpart
tags: [homelab, proxmox, linux, debian, zabbix]
---

Tworząc maszynę wirtualną w Proxmoxie, musimy z góry założyć rozmiar dysku przypisany do naszej maszyny. Odpowiedni rozmiar,
powinniśmy zaplanować z wyprzedzeniem. Z punktu widzenia maszyny nie powinniśmy tworzyć zbyt dużego dysku. Będzie to bowiem
miało wpływ na zarezerwowanie miejsca na naszym serwerze, a także rozmiar backapu czy snapshota maszyny. Z czasem może się jednak okazać,
że wyczerpiemy dostępne miejsce i konieczne będzie jego poszerzenie. Jedną z możliwości będzie dodanie nowego dysku. 
Gdyby jednak konieczne okazało się rozszerzenie istniejącego dysku, z pomocą przyjdzie nam polecenie growpart dostępne w pakiecie
[`cloud-utils`](https://manpages.debian.org/trixie/cloud-guest-utils/growpart.1.en.html).

# Zwiększanie rozmiaru dysku

Przed wykonaniem czynności warto zadbać o wykonanie aktualnego snapshota maszyny. Jeśli nie jest to konieczne,
zrobienie snapshota bez pamięci RAM powinno wykonać się dużo szybciej. Warto na to zwrócić uwagę zwłaszcza w przypadku maszyny
z dużym dyskiem. W tym czasie maszyna będzie dostępna, jednak sam proces tworzenia migawki może wpłynąć na wykorzystanie
zasobów maszyny i spowolnić jej działanie. Ważne, żeby nie przerywać takiego procesu w trakcie, gdyż może to doprowadzić
do jej zatrzymania i zablokowania. Wtedy konieczna będzie ręczna interwencja, a uruchomione usługi w tym czasie nie będą dostępne.

![10-proxmox-snapshot.png](/assets/images/growpart/10-proxmox-snapshot.png)

W kolejnym kroku konieczne będzie zwiększenie fizycznego rozmiaru dysku przypisanego do naszej maszyny. Jak możemy to zrobić,
zostało dokładnie opisane w dokumentacji Proxmox — ["Resize disks"](https://pve.proxmox.com/wiki/Resize_disks). Ja posłużę się
interfejsem webowym, w tym celu kolejno przejdę do menu mojej maszyny i w zakładce `Hardware`, wskazuje dysk, a następnie
w menu górnym wybieram `Disk Action` -> `Resize`.

![12-proxmox-resize.png](/assets/images/growpart/12-proxmox-resize.png)

Całość trwa dosłownie chwilę, a log ze zmiany możemy podejrzeć po rozwinięciu dolnego paska.

![14-proxmox-resize.png](/assets/images/growpart/14-proxmox-log.png)

# Rozszerzania partycji

Po zwiększeniu rozmiaru dysku warto upewnić się, że informacja została przeniesiona na poziomie maszyny. Mając zainstalowany
pakiet [`Qemu-guest-agent`](https://pve.proxmox.com/wiki/Qemu-guest-agent), zmiana powinna być widoczna od razu. Sprawdzimy to
poleceniem `blkid -o list`. Zobaczymy też, że rozmiar naszej dotychczasowej partycji aktualnie nie wypełnia naszego całego dysku. 

```
zabbix@zabbix:~$ lsblk /dev/sdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sdb      8:16   0  14G  0 disk 
└─sdb1   8:17   0  12G  0 part /srv
```

Żeby zwiększyć rozmiar partycji, posłużymy się poleceniem `growpart`. Żeby to zrobić, jako parametr wskażemy najpierw dysk,
a później numer partycji, którą chcemy rozszerzyć. Całość musimy wykonać z podniesionymi uprawnieniami. W moim przypadku będzie to:

```
zabbix@zabbix:~$ sudo growpart /dev/sdb 1 
CHANGED: partition=1 start=2048 old: size=25163743 end=25165790 new: size=29358047 end=29360094
```

Jeśli robimy to pierwszy raz, możemy użyć przełącznika `-N` lub inaczej `-dry-run`, który pozwoli nam podejrzeć zmiany,
które zostaną wykonane bez faktycznego modyfikowania rozmiaru partycji. Trzeba pamiętać, żę nie będziemy mogli zmienić
rozmiaru partycji na dysku, jeśli nie będzie to ostatnia partycja. Polecenie to nie przenosi bowiem żadnych plików,
a jedynie informację o końcu naszej partycji. I tak, jeśli nie będzie możliwości zmiany jej rozmiaru, otrzymamy odpowiednią
informację.

```
zabbix@zabbix:~$ sudo growpart -N /dev/sdb 1
NOCHANGE: partition 1 is size 29358047. it cannot be grown
```

# Rozszerzenie systemu plików

Choć sama partycja ma już odpowiednio zaktualizowaną informację o swoim rozmiarze pozostało nam jeszcze poszerzyć dla niej
system plików. Całość możemy sprawdzić przy użyciu polecenia `df`, które na tym etapie nie wykryje jeszcze żadnych zmian.
Tak prezentuje się nasza tablica partycji.

```
zabbix@zabbix:~$ lsblk /dev/sdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sdb      8:16   0  14G  0 disk 
└─sdb1   8:17   0  14G  0 part /srv
```

A tak informacje na temat dostępnego miejsca.

```
zabbix@zabbix:~$ df -h /srv/
System plików  rozm. użyte dost. %uż. zamont. na
/dev/sdb1        12G  9,0G  2,3G  81% /srv
```

Jako że na dysku zastosowany jest system plików `ext4`, użyjemy polecenia `resize2fs`. Automatycznie rozpozna one
system plików i dokona korekty względem przydzielonego rozmiaru. Punkty montowania możemy zweryfikować poleceniem
`blkid`.

```
zabbix@zabbix:~$ sudo blkid -o list /dev/sdb1
device     fs_type label    mount point    UUID
-------------------------------------------------------------------------------
/dev/sdb1  ext4             /srv           fa00d033-d623-438a-9f69-1c54d641cf9f
```

Sama zmiana rozmiaru powinna być nieinwazyjna, zrobimy to podając adres naszej partycji jako argument.

```
zabbix@zabbix:~$ sudo resize2fs /dev/sdb1 
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/sdb1 is mounted on /srv; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 2
The filesystem on /dev/sdb1 is now 3669755 (4k) blocks long.
```

Po tej operacji powinniśmy zobaczyć, że zwiększyła się ilość dostępnego miejsca na naszej partycji.

```
zabbix@zabbix:~$ df -h /dev/sdb1 
System plików  rozm. użyte dost. %uż. zamont. na
/dev/sdb1        14G  9,0G  4,2G  69% /srv
```

# Monitoring

W przypadku, gdy monitorujemy naszą maszynę, informacja powinna zostać automatycznie rozpropagowana i dostępna w naszym
narzędziu do monitoringu. Jako że monitoruję maszynę z użyciem Zabbixa, informacja ta jest dostępna także w postaci wykresu.
Dodatkowo alert o wykorzystaniu 80% dysku, który otrzymałem, został automatycznie rozwiązany.

![30-zabbix-chart.png](/assets/images/growpart/30-zabbix-chart.png)


# Podsumowanie

Administrując maszyną wirtualną niejednokrotnie spotkamy się z problemem wyczerpania przydzielonego miejsca na dysku.
Choć planowanie wykorzystanie miejsca i utrzymanie porządku jest priorytetem, rozszerzenie przestrzeni dzięki opisanym
narzędziom samo w sobie nie jest problemem i nie wymusza niedostępności całej maszyny. Z całą pewnością, taką operację
możemy wykonać na uruchomionej maszynie, bez konieczności restartu, czy zatrzymania działającej na niej usług.
