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




