---
title: Obsługa certyfikatów w środowisku Nomad z użyciem proxy Traefik
---

Chcąc hostować dowolny kontener serwujący strony www, warto rozważyć wykorzystanie serwer proxy, będący
pośrednikiem pomiędzy naszym serwerem, a usługą www dowolnego uruchomionego kontenera. Dodatkowo pozwoli to nam na 
uruchomienie wielu usług na jednym wspólnym porcie http (80) oraz https (443), bez konieczności konfigurowania go dla
każdego kontenera z osobna.

# Let's Encrypt

W przypadku kiedy planujemy uruchomić usługę tak aby była dostępna z internetu, warto posłużyć się rozwiązaniem
do generowania darmowych certyfikatów pod nasze strony wwww. Mowa oczywiście o [Let's Encrypt ](https://letsencrypt.org/pl/).
Należy jedynie pamiętać, że certyfikaty te będą wymagały ponownego wygenerowania ich co 3 miesiące. Jeśli chodzi o [Traefic](https://traefik.io/traefik), 
doskonale integruje się z tym rozwiązaniem i pozwala na zautomatyzowanie procesu generowania certyfikatów.

