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
doskonale integruje się z tym rozwiązaniem i pozwala na zautomatyzowanie procesu generowania certyfikatów po odpowiednim
skonfigurowaniu resolvera [ACME](https://doc.traefik.io/traefik/reference/install-configuration/tls/certificate-resolvers/acme/).

Na początku warto wspomnieć o kilku różnych trybach konfiguracji tj. dnsChallenge, tlsChallenge oraz httpChallenge, które
zostały opisane w wyżej przytoczonej dokumentacji. Z uwagi iż moja domena została zarejstrowana u dostawcy, którego nie ma
na liście wspieranych [dostawców DNS](https://go-acme.github.io/lego/dns/) na którym to kliencie [LEGO](https://go-acme.github.io/) 
bezpośrednio została zautomatyzowana obsługa wpisów DNS, nie mogę bezpośrednio skorzystać z trybu dnsChallenge. W zwiążku z powyższym,
zajmiemy się dzisiaj konfiguracją httpChallenge.

# Konfiguracja httpChallenge

Na początku musimy zadbać aby nasza strona była dostępna na porcie 80 bezpośrednio z sieci internet. Na tym porcie usłga
będzie wyrefikowała czy na naszym serwerze znajdzie się automatycznie wygenerowany klucz, który potwierdzi, że jesteśmy
włąścicielami tej domeny.

W kolejnym kroku musimy przygotować konfigurację samego resovera. Konieczne będzie więc dodanie do naszej kofiguracji sekcji
`certifiacteResolvers`. W konfiguracji tej musimy podać adres email wykorzystawany przy rejestracji oraz wskaząć z jakiego entryPointu będzie
przeprowadzona weryfikacja. W moim przypadku będzie to entrypoint `web`. Warto również wskaząć lokalizację do pliku z naszymi wygenerowanymi certyfikatami. Plik ten należy zabezpieczyć,
aby certyfikaty nie przepadły po ew. restarcie kontenera Traefika i nie było potrzeby ponownego ich generowania. Trzeba pomiętać,
że generowanie certyfikatów jest limitowane i ograniczne do pewnego okna czasowego. 

```
entryPoints:
  web:
    address: ":80"

  websecure:
    address: ":443"

certificatesResolvers:
  myresolver:
    acme:
      # ...
      httpChallenge:
        email: admin@example.com
        storage: /etc/traefik/acme.json
        entryPoint: web
```

# Konfiguracja routera

W tym momecie możemy przejść do konfiguracji routera, tak aby nasz kontener otrzymał możliwosć generowania certyfikatu.
Dla naszego kontenera musimy zdefioniować kolejno regułę dla której będzie obsługiwany np. `Host('domena')`, wskazać resolver `acme`,
oraz wybrać zdefioniowany entryPoint `websecure`.

```
"traefik.http.routers.vps.rule=Host(`vps.pmazur.pl`)",
"traefik.http.routers.vps.tls.certresolver=acme",
"traefik.http.routers.vps.entryPoints=websecure",
```

# Weryfikacja certyfikatu

Dla tak przygotowanej konfiguracji nasza strona została zabezpieczona zaufanym certyfikatem TLS, a po wejściu na stronę,
możemy zobaczyć wygenerowany certyfikat wystawiony przez Let's Encrypt.

![ssl-letsencrypt.png](/assets/images/ssl-letsencrypt.png)
