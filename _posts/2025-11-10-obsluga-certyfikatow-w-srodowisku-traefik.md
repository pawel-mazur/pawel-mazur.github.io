---
title: Obsługa certyfikatów w środowisku Traefik
---

Chcąc hostować dowolny kontener serwujący strony www, warto rozważyć wykorzystanie serwer proxy, będący
pośrednikiem pomiędzy naszym serwerem, a usługą www dowolnego uruchomionego kontenera. Dodatkowo pozwoli to nam na 
uruchomienie wielu usług na jednym wspólnym porcie http (80) oraz https (443), bez konieczności konfigurowania go dla
każdego kontenera z osobna.

# Automatyczne generowanie certyfikatów

W przypadku kiedy planujemy uruchomić usługę tak aby była dostępna z internetu, warto posłużyć się rozwiązaniem
do generowania darmowych certyfikatów pod nasze strony wwww. Mowa oczywiście o [Let's Encrypt ](https://letsencrypt.org/pl/).
Należy jedynie pamiętać, że certyfikaty te będą wymagały ponownego wygenerowania ich co 3 miesiące. Jeśli chodzi o [Traefic](https://traefik.io/traefik), 
doskonale integruje się z tym rozwiązaniem i pozwala na zautomatyzowanie procesu generowania certyfikatów po odpowiednim
skonfigurowaniu resolvera [ACME](https://doc.traefik.io/traefik/reference/install-configuration/tls/certificate-resolvers/acme/).

Na początku warto wspomnieć o kilku różnych trybach konfiguracji tj. dnsChallenge, tlsChallenge oraz httpChallenge, które
zostały opisane w wyżej przytoczonej dokumentacji, która w zależności od trybu będzie różniła się sposobem którym
zostanie zweryfikowane prawo do wygenerowania certyfikatu przez wystawcę.

### Konfiguracja httpChallenge

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
      email: admin@example.com
      storage: /etc/traefik/acme.json
      httpChallenge:
        entryPoint: web
```
### Konfiguracja dnsChallenge

Alternatywnym sposobem konfiguracji weryfikacji naszej domeny jest weryfikacja z użciem DNS. Wiąże się to z koniecznością,
umieszczenia wpisu TXT w naszej domenie. Do automatyzacji tego procesu konieczny będzie dostęp do API naszego dostawcy domeny.
Dla tego sposobu zostanie wykorzystany klient [LEGO](https://go-acme.github.io/lego), dla którego lista wspieranych dostawców jest dostępna [tutaj](https://go-acme.github.io/lego/dns/).
Aby przejść przez ten proces skonfigurowałem i wykorzystam do tego pośrednika Cloudflare.

W naszej konfiguracji tym razem powinna się znaleść następująca definicja resolvera:

```
certificatesResolvers:
  myresolver:
    acme:
      email: admin@example.com
      storage: /etc/traefik/acme.json
      dnsChallenge:
        provider: cloudflare
```

Dodatkowo nasza usługa Traefika powinna zostać uruchomiona z następującymi [zmiennymi środowiskowymi](https://go-acme.github.io/lego/dns/cloudflare/index.html#credentials). Aby się zautoryzować
konieczne będzie przekazanie zmiennych `CF_API_EMAIL` oraz `CF_DNS_API_TOKEN` do kontenera. W moim wypadku token ten uzyskamy,
przez wejście na strone [User API Tokens](https://dash.cloudflare.com/profile/api-tokens) i generując go z uprawnieniami 
"`Edit zone DNS`". 

### Konfiguracja routera

W tym momecie możemy przejść do konfiguracji routera, tak aby nasz kontener otrzymał możliwosć generowania certyfikatu.
Dla naszego kontenera musimy zdefioniować kolejno regułę dla której będzie obsługiwany np. `Host('domena')`, wskazać resolver `acme`,
oraz wybrać zdefioniowany entryPoint `websecure`.

```
"traefik.http.routers.vps.rule=Host(`homelab.pmazur.pl`)",
"traefik.http.routers.vps.tls.certresolver=acme",
"traefik.http.routers.vps.entryPoints=websecure",
```

### Weryfikacja certyfikatu

Dla tak przygotowanej konfiguracji nasza strona została zabezpieczona zaufanym certyfikatem TLS, a po wejściu na stronę,
możemy zobaczyć wygenerowany certyfikat wystawiony przez Let's Encrypt.

![ssl-letsencrypt.png](/assets/images/ssl-letsencrypt.png)

# Dostarczanie własnych certyfikatów

Oczywiście jeśli dysponujemy zakupionym bądź [własnym wygenerowanym certyfiaktem]({% post_url 2025-10-30-tworzenie-wlasnego-ca-na-potrzeby-generowania-certyfikatow-ssl %}),
możemy go również wykorzystać przy konfiguracji Traefika. Taki certyfikat będziemy musieli dostarczyć do magazynu certyfikatów,
a ten na podstawie konifugracji routera, dostarczy dopasowany certyfikat dla naszego kontenera. Konfigurację należy więc
zacząć od dodania konfiguracji naszego certyfikatu.

```
tls:
  certificates:
    - certFile: /var/lib/traefik/tls/Homelab.pem
      keyFile: /var/lib/traefik/tls/Homelab.key
```

Należy także pamiętać o włączeniu trybu [TLS](https://doc.traefik.io/traefik/reference/routing-configuration/http/routing/router/#opt-tls) dla naszego routera.
W moim przypadku dla kontenera dodałem następującą konfigurację.

```
"traefik.enable=true",
"traefik.http.routers.home.rule=HOST(`homelab.lan`)",
"traefik.http.routers.home.tls=true",
```

### Weryfikacja certyfikatu

Po przeładowaniu konfugracji Traefika, możemy zobaczyć że nasz kontener jest obecnie obsłużony z użciem wcześniej wygenerowanego i
podłączonego certyfikatu.

![ssl-letsencrypt.png](/assets/images/ssl-home-cert.png)
