---
title: Uruchomienie HashiCorp Vault do zarządzania sekretami
tags: [vault, hashicorp, secrets, security, devops]
---

[HashiCorp Vault](https://developer.hashicorp.com/vault) pozwala przechowywać i udostępniać sekrety, takie jak hasła,
tokeny API czy certyfikaty, bez wpisywania ich w plikach konfiguracyjnych i repozytoriach. Może też generować krótkotrwałe
poświadczenia, na przykład do bazy danych. W tym wpisie uruchamiam pojedynczą instancję Vaulta na Debianie z trwałym
magazynem plikowym, panelem www oraz komunikacją zabezpieczoną TLS.

Taka konfiguracja sprawdzi się w Home Labie albo jako punkt wyjścia do dalszej nauki. Dla systemu produkcyjnego o wysokiej
dostępności trzeba uruchomić co najmniej trzy węzły, przygotować load balancer oraz zaplanować auto-unseal. Nie należy też
traktować trybu developerskiego jako wdrożenia produkcyjnego: nie zapisuje on danych trwale i uruchamia Vaulta w
niebezpiecznej konfiguracji.

# Założenia

Przyjmuję, że serwer działa na Debianie. Pakiet `vault` instaluję w kolejnym kroku. Usługa będzie działała bezpośrednio
na hoście, a nie w kontenerze.

Przed rozpoczęciem warto przygotować:

* katalog na trwałe dane Vaulta,
* certyfikat i klucz prywatny dla publicznego adresu usługi,
* bezpieczne, rozdzielone miejsca dla kluczy unseal i początkowego tokenu root,
* wyłączony swap albo szyfrowany swap,
* proces wykonywania i okresowego sprawdzania kopii zapasowych.

Vault przechowuje sekrety zaszyfrowane w magazynie danych, ale bezpieczeństwo całego rozwiązania zależy również od ochrony
kluczy unseal. Nie zapisuję ich w repozytorium, w historii terminala ani w zwykłym pliku na tym samym serwerze.

# Instalacja na Debianie

HashiCorp Vault nie jest dostępny w domyślnych repozytoriach Debiana. Na tym systemie instaluję go jednak jako pakiet
`.deb` obsługiwany przez APT — źródłem pakietu jest oficjalne repozytorium HashiCorp dla Debiana. Dzięki temu aktualizacje
są obsługiwane przez standardowy mechanizm pakietów systemu.

## Klucz podpisujący HashiCorp

Przed dodaniem repozytorium upewniam się, że dostępne są `wget` i `gpg`, a następnie zapisuję klucz GPG HashiCorp w
systemowym katalogu keyringów. APT użyje go do weryfikacji podpisu pobieranych pakietów:

```
sudo apt update
sudo apt install --yes wget gpg
```

```
wget -O - https://apt.releases.hashicorp.com/gpg | \
  sudo gpg --dearmor --yes -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

Opcja `--yes` jest celowa: podczas rotacji klucza HashiCorp nadpisuje wcześniejszy plik keyringu aktualną wersją. Jeżeli
`apt update` zgłasza brak klucza potrzebnego do weryfikacji podpisu, ponowne wykonanie tego polecenia odświeża keyring.

HashiCorp zmienił klucz podpisujący pakiety Linux 9 września 2026 roku. Obecny odcisk palca powinien mieć wartość
`D55C 0D1A C78A 8D81 26CB 631C FC9C A96A CA02 6560`; można go sprawdzić lokalnie poleceniem:

```
gpg --no-default-keyring \
  --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
  --fingerprint
```

```
/usr/share/keyrings/hashicorp-archive-keyring.gpg
-------------------------------------------------
pub   rsa4096 2026-09-09 [SC] [wygasa: 2031-09-08]
      D55C 0D1A C78A 8D81 26CB  631C FC9C A96A CA02 6560
uid    [    nieznane   ] HashiCorp Security (HashiCorp Package Signing) <security+packaging@hashicorp.com>
```

Wartość należy porównać z aktualnym odciskiem w sekcji [Linux package checksum verification](https://www.hashicorp.com/en/trust/security#linux-package-checksum-verification) na stronie bezpieczeństwa HashiCorp.

## Repozytorium APT

Następnie dodaję oficjalne repozytorium HashiCorp. Polecenie pobiera kodową nazwę wydania z `/etc/os-release` albo z
`lsb_release`, dlatego wpis zostanie dopasowany do Debiana lub Ubuntu używanego przez serwer:

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list
```

Warto od razu sprawdzić zapisany plik:

```
cat /etc/apt/sources.list.d/hashicorp.list
```

Na Debianie 13 (Trixie) wpis powinien kończyć się `trixie main` i wskazywać ten sam plik keyringu,
`/usr/share/keyrings/hashicorp-archive-keyring.gpg`, który został użyty podczas importu klucza.

Następnie instaluję pakiet i sprawdzam wersję programu:

```
sudo apt update
sudo apt install vault
vault version
```

Przykładowy wynik instalacji:

```
$ sudo apt install vault
Instalowane:
  vault

Podsumowanie:
  Aktualizowanych: 0, Instalowanych: 1, Usuwanych: 0, Nieaktualizowanych: 154
  Do pobrania: 178 MB
  Wymagane miejsce: 538 MB / 4 824 MB dostępnych

Pobr:1 https://apt.releases.hashicorp.com trixie/main amd64 vault amd64 2.1.1-1 [178 MB]
Pobrano 178 MB w 7s (25,7 MB/s)
Wybieranie wcześniej niewybranego pakietu vault.
Przygotowywanie do rozpakowania pakietu .../vault_2.1.1-1_amd64.deb ...
Rozpakowywanie pakietu vault (2.1.1-1) ...
Konfigurowanie pakietu vault (2.1.1-1) ...
Generating Vault TLS key and self-signed certificate...
...
Vault TLS key and self-signed certificate have been generated in '/opt/vault/tls'.

$ vault version
Vault v2.1.1 (d78bbbe2d2f3d289c1ec29d431071beb23668212), built 2026-09-15T21:21:56Z
```

Podczas instalacji pakiet tworzy lokalny klucz TLS i certyfikat self-signed w `/opt/vault/tls`. W dalszej konfiguracji
zastępuję je certyfikatem przeznaczonym dla publicznego adresu usługi.

Pakiet tworzy użytkownika systemowego `vault` i usługę systemd.

# Konfiguracja serwera

Podstawowa konfiguracja znajduje się w pliku `/etc/vault.d/vault.hcl`. Poniżej znajduje się domyślna konfiguracja używana na
serwerze zaraz po instalacji: korzysta z trwałego magazynu plikowego i nasłuchuje po HTTPS na porcie `8200`.

```
# Copyright IBM Corp. 2016, 2025
# SPDX-License-Identifier: BUSL-1.1

# Full configuration options can be found at https://developer.hashicorp.com/vault/docs/configuration

ui = true

#mlock = true
#disable_mlock = true

storage "file" {
  path = "/opt/vault/data"
}

#storage "consul" {
#  address = "127.0.0.1:8500"
#  path    = "vault"
#}

# HTTP listener
#listener "tcp" {
#  address = "127.0.0.1:8200"
#  tls_disable = 1
#}

# HTTPS listener
listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/opt/vault/tls/vault.crt"
  tls_key_file  = "/opt/vault/tls/vault.key"
}
```

Magazyn `file` zapisuje dane Vaulta w `/opt/vault/data`. Jest to trwałe rozwiązanie dla pojedynczego serwera, ale nie
obsługuje wysokiej dostępności. Zakomentowane bloki `consul` i HTTP są jedynie alternatywami — aktywna pozostaje
konfiguracja `file` oraz listener HTTPS.

Listener nasłuchuje na wszystkich interfejsach, dlatego dostęp do portu `8200` powinien być ograniczony zaporą sieciową.
Pakiet tworzy w `/opt/vault/tls` pliki `tls.crt` i `tls.key`. W tej instalacji `tls.crt` jest certyfikatem CA (`CA:TRUE`),
a nie certyfikatem serwera, więc nie może być użyty bezpośrednio jako `tls_cert_file` listenera. Poniżej tworzę osobny
certyfikat serwera `vault.crt`, podpisany przez moje własne CA.

## Certyfikat serwera

Do podpisywania certyfikatu używam własnego CA, którego przygotowanie opisałem we wpisie
[Tworzenie własnego CA na potrzeby generowania certyfikatów dla stron www]({% post_url 2025-10-30-tworzenie-wlasnego-ca-na-potrzeby-generowania-certyfikatow-ssl %}).
Certyfikat serwera zawiera nazwę `vault.lan` w `CN` oraz w `subjectAltName`.

Klucz prywatny CA pozostaje na maszynie CA. Na serwer Vaulta kopiuję wyłącznie wystawiony certyfikat serwera oraz
pasujący do niego klucz prywatny, a następnie ustawiam ich właściciela i uprawnienia:

```
sudo install -o vault -g vault -m 0644 vault.crt /opt/vault/tls/vault.crt
sudo install -o vault -g vault -m 0640 vault.key /opt/vault/tls/vault.key
```

Po zmianie plików należy ponownie uruchomić Vaulta i sprawdzić konfigurację. Urządzenia klienckie muszą ufać własnemu CA,
aby poprawnie zweryfikować certyfikat usługi.

Katalog z certyfikatem oraz sam klucz prywatny muszą być dostępne dla użytkownika `vault`, ale nie dla innych użytkowników
systemu. Przed uruchomieniem usługi sprawdzam konfigurację oraz podstawowe wymagania systemowe poleceniem
`vault operator diagnose`:

```
$ sudo vault operator diagnose -config /etc/vault.d/vault.hcl 
Vault v2.1.1 (d78bbbe2d2f3d289c1ec29d431071beb23668212), built 2026-09-15T21:21:56Z

Results:
[ warning ] Vault Diagnose: HCP link check will not run on OSS Vault.
  [ success ] Check Operating System
    [ success ] Check Open File Limits: Open file limits are set to 524287.
    [ success ] Check Disk Usage: / usage ok.
  [ success ] Parse Configuration
  [ warning ] Check Telemetry: Telemetry is using default configuration
    By default only Prometheus and JSON metrics are available.  Ignore this warning if you are using telemetry or are using these metrics and are
    satisfied with the default retention time and gauge period.
  [ success ] Check Storage
    [ success ] Create Storage Backend
    [ success ] Check Storage Access
  [ skipped ] Check Service Discovery: No service registration configured.
  [ success ] Create Vault Server Configuration Seals
  [ skipped ] Check Transit Seal TLS: No transit seal found in seal configuration.
  [ success ] Create Core Configuration
    [ success ] Initialize Randomness for Core
  [ success ] HA Storage
    [ success ] Create HA Storage Backend
    [ skipped ] Check HA Consul Direct Storage Access: No HA storage stanza is configured.
  [ success ] Determine Redirect Address
  [ success ] Check Cluster Address: Cluster address is logically valid and can be found.
  [ success ] Check Core Creation
  [ skipped ] Check For Autoloaded License: License check will not run on OSS Vault.
  [ success ] Start Listeners
    [ success ] Check Listener TLS
    [ success ] Create Listeners
  [ skipped ] Check Autounseal Encryption: Skipping barrier encryption test. Only supported for auto-unseal.
  [ success ] Check Server Before Runtime
  [ success ] Finalize Shamir Seal
```

Polecenie `vault operator diagnose` wykonuje testy środowiska i konfiguracji, między innymi sprawdza, czy da się utworzyć
skonfigurowany backend magazynu. Jeżeli Vault jest wystawiony przez reverse proxy, certyfikat i port listenera można
zakończyć na proxy, ale sam ruch pomiędzy proxy a Vaultem nadal powinien być odpowiednio zabezpieczony i ograniczony do
zaufanej sieci.

Teraz możemy uruchomić naszą usługę `vault`, zrobimy to z pomocą polecenia `service vault start`.

```
$ sudo service vault start
$ sudo service vault status
● vault.service - "HashiCorp Vault - A tool for managing secrets"
Loaded: loaded (/usr/lib/systemd/system/vault.service; enabled; preset: enabled)
Active: active (running) since Fri 2026-10-02 06:28:49 CEST; 4s ago
Invocation: 327563dd476643db8f0f1d945cda880e
Docs: https://developer.hashicorp.com/vault/docs
Main PID: 3057458 (vault)
Tasks: 9 (limit: 4640)
Memory: 95.2M (peak: 95.7M)
CPU: 290ms
CGroup: /system.slice/vault.service
└─3057458 /usr/bin/vault server -config=/etc/vault.d/vault.hcl

paź 02 06:28:49 nomad-server.lan vault[3057458]:                  Storage: file
paź 02 06:28:49 nomad-server.lan vault[3057458]:                  Version: Vault v2.1.1, built 2026-09-15T21:21:56Z
paź 02 06:28:49 nomad-server.lan vault[3057458]:              Version Sha: d78bbbe2d2f3d289c1ec29d431071beb23668212
paź 02 06:28:49 nomad-server.lan vault[3057458]: ==> Vault server started! Log data will stream in below:
paź 02 06:28:49 nomad-server.lan vault[3057458]: 2026-10-02T06:28:49.851+0200 [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
paź 02 06:28:49 nomad-server.lan vault[3057458]: 2026-10-02T06:28:49.852+0200 [INFO]  incrementing seal generation: generation=1
paź 02 06:28:49 nomad-server.lan vault[3057458]: 2026-10-02T06:28:49.852+0200 [WARN]  no `api_addr` value specified in config or in VAULT_API_ADDR; falling b>
paź 02 06:28:49 nomad-server.lan vault[3057458]: 2026-10-02T06:28:49.938+0200 [INFO]  core: Initializing version history cache for core
paź 02 06:28:49 nomad-server.lan vault[3057458]: 2026-10-02T06:28:49.938+0200 [INFO]  events: Starting event system
paź 02 06:28:49 nomad-server.lan systemd[1]: Started vault.service - "HashiCorp Vault - A tool for managing secrets".
```

# Inicjalizacja i odpieczętowanie

[Vault](https://developer.hashicorp.com/vault/docs/about-vault/how-vault-works#the-encryption-barrier) przechowuje
w katalogu danych wyłącznie informacje zabezpieczone przez barierę szyfrowania.
Po odpieczętowaniu sprawdza, kim jest klient i czy jego polityka pozwala na żądaną operację. Restart ponownie blokuje
dostęp do sekretów, dopóki Vault nie zostanie odpieczętowany.

Jednorazowa inicjalizacja tworzy materiał kryptograficzny, dzieli klucze unseal
metodą [Shamir's Secret Sharing](https://developer.hashicorp.com/vault/docs/concepts/seal#shamir-seals)
oraz tworzy token root. Możemy ją wykonać zarówno z interfejsu web jak i cli.

![10-initialization.png](/assets/images/vault/10-initialization.png)

W tym przykładzie trzy z pięciu osób posiadających udział w kluczu muszą współpracować, aby odpieczętować usługę. 
Adres w `VAULT_ADDR` musi odpowiadać nazwie z certyfikatu TLS. W przypadku gdy nie dysponujemy zaufanym CA, roboczo możemy
pominąć weryfikację klucza przy użyciu zmiennej `VAULT_SKIP_VERIFY`.

```
export VAULT_ADDR="https://adres-vaulta:8200"
export VAULT_SKIP_VERIFY=1
vault operator init -key-shares=5 -key-threshold=3
```

Wynik zawiera pięć kluczy unseal i token root. Należy przekazać je do wcześniej przygotowanych, oddzielnych bezpiecznych
lokalizacji. Token root służy wyłącznie do pierwszej konfiguracji i sytuacji awaryjnych — nie powinien być używany na co
dzień ani przechowywany w zmiennej środowiskowej dłużej niż jest to konieczne.

![20-unseal.png](/assets/images/vault/20-unseal.png)

Następnie trzy różne udziały klucza odpieczętowują serwer:

```
$ vault operator unseal
Unseal Key (will be hidden): 
$ vault operator unseal
Unseal Key (will be hidden): 
$ vault operator unseal
Unseal Key (will be hidden): 
$ vault status 
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         2.1.1
Build Date      2026-09-15T21:21:56Z
Storage Type    file
Cluster Name    vault-cluster-e460715d
Cluster ID      9f36ebf5-87f4-72a7-9a3c-3a6f9e33f998
HA Enabled      false
```

Po restarcie usługi Vault ponownie będzie `sealed`, więc procedurę unseal trzeba powtórzyć. W środowiskach, w których
interwencja operatorów po każdym restarcie jest niepraktyczna, warto skonfigurować auto-unseal z zaufanym KMS lub HSM.

![30-dashboard.png](/assets/images/vault/30-dashboard.png)

# Pierwszy sekret i polityka

Wszystkie opsiane niżej operacje można wykonać zarówno z poziomu interfejsu webowego jaki i z użyciem CLI.
Po zalogowaniu tokenem root włączam silnik sekretów KV w wersji 2 i zapisuję przykładową wartość:

![40-secrets-engine.png](/assets/images/vault/40-secrets-engine.png)

```
vault login
vault secrets enable -path=kv kv-v2
vault kv put kv/example username=service password='zmien-to-haslo'
vault kv get kv/example
```

To tylko test działania. Sekret podany bezpośrednio w poleceniu może zostać zapisany w historii powłoki, dlatego w
rzeczywistym użyciu lepiej przekazać go przez bezpieczny proces wdrożeniowy albo wskazać plik wejściowy o ograniczonych
uprawnieniach.

![50-example-secret.png](/assets/images/vault/50-example-secret.png)

Zamiast rozdawać token root tworzę minimalną politykę dostępu. Plik `example-read.hcl` daje aplikacji wyłącznie odczyt
jednego prefiksu:

```
path "kv/data/example/*" {
  capabilities = ["read"]
}
```

Politykę można następnie wgrać i powiązać z odpowiednią metodą uwierzytelniania, na przykład AppRole, Kubernetes lub OIDC:

```
vault policy write example-read example-read.hcl
```

# Audyt, backup i bieżąca kontrola

Audyt jest domyślnie wyłączony, dlatego włączam go od razu po pierwszej konfiguracji. W przypadku usługi zarządzanej przez
systemd wygodne jest kierowanie wpisów do standardowego wyjścia, skąd trafią do dziennika systemowego:

```
vault audit enable file file_path=stdout
```

Wpisy można wtedy przeglądać przez `journalctl -u vault`. Logi audytowe są istotne dla bezpieczeństwa, ale zawierają
metadane operacji, więc również wymagają kontroli dostępu i retencji.

Magazyn `file` nie obsługuje snapshotów Raft, dlatego kopię całej maszyny wirtualnej wykonuję przez
[Proxmox Backup Server]({% post_url 2025-12-21-uruchomienie-proxmox-backup-server %}). PBS obejmuje wolumeny z
`/opt/vault/data`, `/opt/vault/tls` oraz konfiguracją w `/etc/vault.d`.

Ponieważ backend plikowy nie zapewnia atomowego snapshotu danych Vaulta, przed rozpoczęciem backapu należało by zatrzymać
usługę Vaulta albo całą maszynę wirtualną. Takie podejście jest zgodne z
[oficjalnymi zaleceniami backupu Vaulta](https://developer.hashicorp.com/vault/docs/concepts/storage#backing-up-vault-s-persisted-data).
Docelowo, należałoby rozważyć użycie [Integrated storage (Raft) backend
](https://developer.hashicorp.com/vault/docs/configuration/storage/raft) jako backendu, który to jednak wspiera wykonywanie
snapshotów i nie wymusza na nas zatrzymania usługi.

Kopie PBS należy szyfrować, przechowywać poza serwerem oraz regularnie testować ich odtworzenie. Backup maszyny nie
zastępuje oddzielnej ochrony kluczy unseal ani tokenu odzyskiwania, jeśli używany jest auto-unseal.

Podstawowy stan usługi sprawdzam poleceniami:

```
vault status
sudo service vault status
sudo journalctl -u vault --since "1 hour ago"
```

# Podsumowanie

Pojedyncza instancja Vaulta z magazynem plikowym jest dobrym początkiem do centralnego zarządzania sekretami. Najważniejsze
elementy to trwały magazyn danych, TLS, dobrze zabezpieczone klucze unseal, polityki o minimalnych uprawnieniach, audyt
oraz zweryfikowane kopie zapasowe.

Gdy usługa stanie się krytyczna, kolejnym krokiem powinno być przejście na klaster HA, automatyczne odpieczętowanie i
integracja aplikacji z wybraną metodą uwierzytelniania zamiast ręcznego używania tokenów.
