---
title: Netbox jako narzędzie do dokumentacji infrastruktury
tags: network homelab documentation netbox
---

Chcąc rozbudować swoją domową sieć, stanąłem przed wyzwaniem jej udokumentowania. Bez tego,
ciężko nam będzie sensownie zaplanować, rozbudować, a nawet utrzymywać aktualną infrastrukturę.
Z&nbsp;pomocą przyjdzie nam narzędzie [Netbox](https://netboxlabs.com/), z użyciem, którego postaram się zinwentaryzować bieżący
stan mojej sieci.

![00-homepage.png](/assets/images/netbox/00-homepage.png)

# Uruchomienie narzędzia

Choć jest możliwa [instalacja](https://netboxlabs.com/docs/netbox/installation/) w naszej własnej infrastrukturze,
skorzystam z dostępnej wersji [NetBox Cloud Free Plan](https://netboxlabs.com/products/free-netbox-cloud/). Ograniczenia
związane z limitami tej wersji będą dla mnie sporym zapasem.

![10-installation.png](/assets/images/netbox/10-installation.png)

#### Założenie konta

Założenie konta, zaczniemy od podania adresu email. Będziemy mogli je również założyć przez logowanie za pomocą Google,
Microsoft Account bądź Github.

![20-register.png](/assets/images/netbox/20-register.png)

Po rejestracji konieczne będzie potwierdzenia adresu mailowego oraz zatwierdzenie warunków świadczenia usługi. Usługa
wymaga też skonfigurowania TOTP.

#### Konfiguracja konta

Podczas wstępnej konfiguracji, zostaniemy poproszeni o podanie m.in. głównego przeznaczenia naszego konta.
W tym momencie wybrałem `Network documentation`. Skorzystam także z&nbsp;wygenerowania przykładowych danych, 
by lepiej zapoznać się z możliwościami systemu.

![30-configuration.png](/assets/images/netbox/30-configuration.png)

Następnie musimy zaczekać chwilę na wygenerowanie i uruchomienie naszej instancji.

![32-provisioning.png](/assets/images/netbox/32-provisioning.png)

Po zalogowaniu się do panelu otrzymujemy podsumowanie na następującym dashboardzie.

![34-dashboard.png](/assets/images/netbox/34-dashboard.png)

# Dokumentacja

Po przejrzeniu listy dostępnych obiektów postanowiłem je wyczyścić poprzez ręczne kasowanie danych z poziomu interfejsu
użytkownika. Część z tych rekordów wymagała także skasowania rekordów zależnych, a ja mogę przejść do właściwej dokumentacji
sieci.

#### Gałęzie

Netbox jako narzędzie pozwala na pracę na branchach, pozwalających rozwój infrastuktury i wdrożenie wielu zmian do głownej
bazy jako całości. Jak mówi dokumentacja [NetBox Branching](https://netboxlabs.com/docs/extensions/branching/), mechanizm
ten opiera się o kncept pracy z repozytoriami [git](https://git-scm.com/). Na swoje potrzeby tworzę eksperymantalny brach
z przykładową dokumentacją, który nazwę `Demo`.

![36-branch.png](/assets/images/netbox/36-branch.png)

#### Lokalizacje

Aby móc zinwentaryzować i opisać urządzenia, konieczne jest zdefiniowanie miejsca, do którego będzie ono przypisane. 
I tak możemy grupować obiekty w [Regiony](https://netboxlabs.com/docs/netbox/models/dcim/region/) >
[Witryny](https://netboxlabs.com/docs/netbox/models/dcim/site/) > [Lokalizacje](https://netboxlabs.com/docs/netbox/models/dcim/location/).
Na potrzeby testów założyłem witrynę `Main`, nie precyzując dokładnie jej przeznaczenia.

![38-site.png](/assets/images/netbox/38-site.png)

#### Szafy

Do rozmieszczenia sprzętu, możemy się posłużyć konfiguracją [szafy](https://netboxlabs.com/docs/netbox/models/dcim/rack/).
Wskażemy tu m.in. jej położenie, identyfikator czy chociażby wymiary, w tym szerokość w calach i wysokość w jednostkach U.
Wprowadzimy także wagę, dopuszczalne obciążenie i głębokość montażu. Docelowo, graficznie zdefiniujemy, też rozmieszczenie
sprzętu oraz udokumentujemy stan obiektu za pomocą przesłanych przez nas zdjęć.

![40-rack.png](/assets/images/netbox/40-rack.png)

#### Urządzenia

Kolejnym krokiem jest dokumentacja [urządzeń](https://netboxlabs.com/docs/netbox/models/dcim/device/). Aby przejść do ich tworzenia,
konieczne będzie wprowadzenie [roli urządzenia](https://netboxlabs.com/docs/netbox/models/dcim/devicerole/). Jako przykład,
posłużę się tutaj typem `Serwer`. Dalej będziemy musieli stworzyć [typ urządzenia](https://netboxlabs.com/docs/netbox/models/dcim/devicetype/),
w którym określimy [producenta](https://netboxlabs.com/docs/netbox/models/dcim/manufacturer/) oraz model. Dla urządzenia 
możemy również przypisać zdefiniowany dla niego adres IP.

![42-device.png](/assets/images/netbox/42-device.png)

#### Adresy IP

Po przejściu do zakładki IPAM, w części [Adresy IP](https://netboxlabs.com/docs/netbox/models/ipam/ipaddress/), mamy
możliwość udokumentowania wykorzystywanych w naszej sieci adresów IP v4/v6. I tak dla wprowadzonego rekordu podajemy
sam adres w notacji z maską. Możemy wskazać jego status, określić przeznaczenie czy nawet powiązać z konfiguracją konkretnego
urządzenia w naszej sieci. Dodatkowo na tym ekranie wskażemy także, pod jaką nazwą DNS będzie dostępny dany adres.

![44-branch.png](/assets/images/netbox/44-address.png)

#### Maszyny wirtualne

Aby udokumentować uruchomione maszyny wirtualne, zrobimy to w sekcji [Maszyny wirtualne](https://netboxlabs.com/docs/netbox/models/virtualization/virtualmachine/).
Na początku będziemy musieli skonfigurować odpowiedni klaster, na który będą one uruchomione. W tym momencie dodając
nową maszynę, skonfigurujemy jego role, wskażemy wcześniej skonfigurowany klaster, czy określimy zasoby dla niej dostępne
tj. vCPU, Pamięć, czy Dysk. Docelowo skonfigurujemy także interfejsy sieciowe, określimy uruchomione usługi, załączymy zdjęcia
przygotowanej maszyny, czy określimy osoby kontaktowe.

![46-virtual-machine.png](/assets/images/netbox/46-virtual-machine.png)

#### Akceptowanie zmian

Na koniec, jeśli pracujemy z gałęziami, mamy możliwość zrewidowania wprowadzonych zmian i zatwierdzenia ich jako całości,
do głównej gałęzi. Zrobimy to, przechodząc do sekcji [Change Management](https://netboxlabs.com/docs/developer/plugins-extensions/changes/).

![48-change-request.png](/assets/images/netbox/48-change-request.png)

#### Dziennik zmian

Przeglądając [dziennik](https://netboxlabs.com/docs/netbox/features/journaling/), mamy dostęp do historii wprowadzonych 
zmian w naszej dokumentacji. Co ważne, na stronie tej, możemy dzięki zastosowaniu filtrów ograniczyć dziennik do zadanych
przez nas kryteriów tj. czas, typu działania, użytkowniku czy typu obiektu.

![48-journal.png](/assets/images/netbox/50-journal.png)

# Podsumowanie

Netbox to zaawansowane narzędzie, które pomoże nam na zamodelowanie i zbudowanie dokumentacji infrastruktury. Dzięki
niemu uporządkujemy temat urządzeń, maszyn wirtualnych, ich interfejsów sieciowych, adresacji IP, punktów WiFi, połączeń
VPN, czy nawet źródeł zasilania. Z pewnością wykorzystam je do planowania i rozbudowy mojego domowego 
[Homelaba](https://github.com/pawel-mazur/homelab).
