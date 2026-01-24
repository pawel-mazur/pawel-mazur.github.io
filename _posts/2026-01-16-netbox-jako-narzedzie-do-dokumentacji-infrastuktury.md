---
title: Netbox jako narzędzie do dokumentacji infrastruktury
---

Chcąc rozbudować swoją domową sieć, stanąłem przed wyzwaniem jej udokumentowania. Bez tego,
ciężko nam będzie sensownie zaplanować, rozbudować, a nawet utrzymywać aktualną infrastrukturę.
Z pomocą przyjdzie nam narzędzie [Netbox](https://netboxlabs.com/), z użyciem, którego postaram się zinwentaryzować bieżaący
stan mojej sieci.

![00-homepage.png](/assets/images/netbox/00-homepage.png)

# Uruchomienie narzędzia

Choć jest możliwa [instalacja](https://netboxlabs.com/docs/netbox/installation/) w naszej własnej infrastrukturze,
skorzystam z dostępnej wersji [NetBox Cloud Free Plan](https://netboxlabs.com/products/free-netbox-cloud/). Ograniczenia
związane z limitami tej wersji będą dla mnie sporym zapasem.

![10-installation.png](/assets/images/netbox/10-installation.png)

# Założenie konta

Założenie konta, zaczniemy od podania adresu email. Będziemy mogli je również założyć przez logowanie za pomocą Google,
Microsoft Account bądź Github.

![20-register.png](/assets/images/netbox/20-register.png)

Po rejestracji konieczne będzie potwierdzenia adresu mailowego oraz zatwierdzenie warunków świadczenia usługi. Usługa
wymaga też skonfigurowania TOTP.

# Konfiguracja konta

Podczas wstępnej konfiguracji, zostaniemy poproszeni o podanie m.in. głównego przeznaczenia naszego konta.
W tym momencie wybrałem `Network documentation`. Skorzystam także z&nbsp;wygenerowania przykładowych danych, 
by lepiej zapoznać się z możliwościami systemu.

![30-configuration.png](/assets/images/netbox/30-configuration.png)

W tym momencie musimy zaczekać na wygenerowanie i uruchomienie naszej instancji.

![32-provisioning.png](/assets/images/netbox/32-provisioning.png)

Po zalogowaniu się do panelu otrzymujemy podsumowanie na następującym dashboardzie.

![34-dashboard.png](/assets/images/netbox/34-dashboard.png)

# Dokumentacja

Po przejrzeniu listy dostępnych obiektów postanowiłem je wyczyścić poprzez ręczne kasowanie danych z poziomu interfejsu
użytkownika. Część z tych rekordów wymagała także skasowania rekordów zależnych, a ja mogę przejść do właściwej dokumentacji
mojej sieci.

#### Gałęzie

Netbox jako narzędzie pozwala na pracę na branchach, pozwalających rozwój infrastuktury i wdrożenie wielu zmian do głownej
bazy jako całości. Jak mówi dokumentacja [NetBox Branching](https://netboxlabs.com/docs/extensions/branching/), mechanizm
ten opiera się o kncept pracy z repozytoriami [git](https://git-scm.com/). Na swoje potrzeby tworzę eksperymantalny brach
z przykładową dokumentacją, który nazwę `Demo`.

![36-branch.png](/assets/images/netbox/36-branch.png)

#### Adresy IP

Po przejściu do zakładki IPAM, w części [IP Addresses](https://netboxlabs.com/docs/netbox/models/ipam/ipaddress/), mamy
możliwość udokumentowania wykorzystywanych w naszej sieci adresów IP v4/v6. I tak dla wprowadzonego rekordu podajemy
sam adres w notacji z maską. Możemy wskazać jego status, określić przeznaczenie czy nawet powiązać z konfiguracją konkretnego
urządzenia w naszej sieci. Dodatkowo na tym ekranie wskażemy także, pod jaką nazwą DNS będzie dostępny dany serwer.

![38-branch.png](/assets/images/netbox/38-address.png)

#### Maszyny wirtualne

Aby udokumentować uruchomione maszyny wirtualne, zrobimy to w sekcji [Virtual Machines](https://netboxlabs.com/docs/netbox/models/virtualization/virtualmachine/).
Na początku będziemy musieli skonfigurować odpowiedni klaster, na który maszyny będą uruchomione. W tym momencie dodając
nową maszynę, skonfigurujemy jego role, wskażemy wcześniej skonfigurowany klaster, czy określimy zasoby dla niej dostępne
tj. VCPU, Pamięć, czy Dysk. Docelowo skonfigurujemy także interfejsy sieciowe, określimy uruchomione usługi, załączymy zdjęcia
przygotowanej maszyny, czy określimy osoby kontaktowe.

![40-virtual-machine.png](/assets/images/netbox/40-virtual-machine.png)


