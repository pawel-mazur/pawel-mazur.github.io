---
title: "Tworzenie własnego CA na potrzeby generowania certyfikatów dla stron www"
tags: web ca ssl tls https
---

Przy hostowaniu stron w sieci domowej, z pewnością przyda się możliwość udostępnienia ich z przy wykorzystaniu
protokołu HTTPs. Żeby osiągnąć ten cel, koniecznym jest dostarczenie certyfikatu TLS, z którego użyciem nasza komunikacja
będzie szyfrowana. Standardowo dla każdej ze stron należałoby zakupić taki certyfikat u zaufanego dostawcy. Alternatywnie
możemy stworzyć własne CA, z którego użyciem będziemy podpisywać używane certyfikaty, jednak konieczne będzie dodanie go
bazy zaufanych CA, na każdym z urządzeń, na którym będziemy chcieli nawiązać komunikację z naszą stroną.

Do wygenerowania certyfikatu posłużę się artykułem zamieszczonym na stronie [Generate an Azure Application Gateway self-signed certificate with a custom root CA](https://learn.microsoft.com/en-us/azure/application-gateway/self-signed-certificates),
a także [How to create a valid self signed SSL Certificate?](https://www.patreon.com/posts/109937722).

# Tworzenie CA

W pierwszym kroku musimy przygotować nasze CA, zrobimy to przy pomocy oprogramowania [OpenSSL](https://www.openssl.org/).

```bash
$ openssl genrsa -aes256 -out ca-key.pem 4096
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

W tym momencie mamy jedynie wygenerowany klucz RSA, z którego użyciem wygenerujemy klucz naszego CA.
Będziemy musieli odpowiedzieć na kilka zadanych pytań. 

```bash
$ openssl req -new -x509 -sha256 -days 3650 -key ca-key.pem -out ca.pem
Enter pass phrase for ca-key.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:PL
State or Province Name (full name) [Some-State]:dolnoslaskie
Locality Name (eg, city) []:Wroclaw
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Pawel Mazur
Organizational Unit Name (eg, section) []:.
Common Name (e.g. server FQDN or YOUR name) []:www.pmazur.pl
Email Address []:
```

W tym momencie możemy podejrzeć informacje zawarte w certyfikacie naszego CA.

```bash
$ openssl x509 -in ca.pem -text -noout 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            0e:43:17:67:ad:72:9b:7b:ff:7a:5c:ae:f3:c2:3c:5e:45:c0:a8:16
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = PL, ST = dolnoslaskie, L = Wroclaw, O = Pawel Mazur, CN = www.pmazur.pl
        Validity
            Not Before: Nov  1 04:34:39 2025 GMT
            Not After : Oct 30 04:34:39 2035 GMT
        Subject: C = PL, ST = dolnoslaskie, L = Wroclaw, O = Pawel Mazur, CN = www.pmazur.pl
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:bd:fa:50:4d:fa:72:71:ab:87:4f:db:86:e1:f0:
                    78:05:ce:7f:21:bb:0e:2c:b5:d4:b7:36:db:cc:a7:
                    1f:64:89:57:71:f3:76:12:a7:9c:10:1b:83:e2:12:
                    f4:ba:3a:c1:be:08:a7:cb:13:98:fb:16:a4:54:73:
                    ff:66:e4:88:d8:ae:00:cc:d2:91:22:7b:ef:25:bc:
                    da:64:44:51:1b:04:44:50:76:88:c4:6e:28:13:5b:
                    c5:de:7d:f1:84:a3:95:3d:0d:f5:0d:1c:c8:ba:13:
                    90:06:2f:8f:56:8d:a1:72:74:b6:64:40:13:8d:cd:
                    81:4e:64:f4:95:92:cd:f9:fd:0c:df:b4:9a:5a:ca:
                    2d:25:14:b4:66:f1:e1:af:68:63:85:75:9b:3f:3c:
                    16:62:e3:37:0c:7f:f1:81:c8:4f:9e:8c:fb:cc:18:
                    eb:e0:9c:19:67:4d:c8:4f:21:b7:de:a1:9a:0c:43:
                    d0:17:5f:a3:37:f8:67:6d:26:b2:6b:28:e1:7c:33:
                    39:b2:d1:9f:71:49:ff:23:d9:33:b8:cc:18:ad:d5:
                    66:4b:01:6e:e7:e3:8f:ba:7d:be:7f:a7:54:36:54:
                    29:e0:2a:c0:f3:46:1a:19:86:06:c8:89:db:ea:4b:
                    b5:2b:da:f9:75:18:7c:cb:6d:cf:52:3b:de:ea:ca:
                    7a:ad:97:75:b3:14:92:7e:06:8d:70:f8:a5:02:2f:
                    12:2e:df:72:b9:11:a7:af:7a:94:c9:a4:13:a8:01:
                    4f:fa:2f:03:61:67:33:a0:4f:b8:3b:bf:af:b8:f2:
                    8f:a4:72:0c:e1:df:07:4d:e2:fd:ef:73:1c:31:b9:
                    81:39:66:2d:38:c6:fb:a0:30:bc:28:95:29:8c:d7:
                    1a:27:3f:b8:18:27:54:69:21:b7:12:64:40:fa:79:
                    da:ac:85:88:69:88:5b:02:15:38:e0:34:97:f6:ea:
                    8a:78:f5:21:30:8c:59:d9:b8:18:c7:9f:16:93:59:
                    b3:1f:f8:ce:32:6c:36:f6:ed:d7:5d:ec:57:05:f6:
                    49:06:2f:68:7b:78:10:67:8e:f2:e4:d0:1f:b6:95:
                    17:d1:26:28:fe:1b:18:5b:46:e5:6d:7d:20:b7:a0:
                    4b:fb:02:12:a9:83:10:b3:52:84:4e:35:f3:97:d9:
                    94:31:6e:4e:40:a4:f4:8a:d2:95:46:5f:24:85:9a:
                    01:7e:ce:19:cd:bc:85:05:03:a2:cc:2e:09:d5:49:
                    fb:66:ed:78:46:7d:a2:95:92:b9:97:1e:e2:74:86:
                    44:86:bb:31:06:cb:38:4a:d5:ed:98:90:c0:b8:ac:
                    90:3d:45:f4:0c:d8:4d:4d:93:13:bf:85:7b:e4:46:
                    b8:85:93
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                91:E9:84:F5:08:A0:C6:42:5A:0C:C3:51:27:96:3A:82:2E:E1:77:A4
            X509v3 Authority Key Identifier: 
                91:E9:84:F5:08:A0:C6:42:5A:0C:C3:51:27:96:3A:82:2E:E1:77:A4
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        33:3c:39:e3:6d:a3:8b:27:0a:10:99:85:c9:aa:d3:95:d7:fe:
        0a:27:5b:27:37:1b:f2:9c:30:a5:fd:5d:fe:e4:9e:4b:40:6c:
        dd:18:7d:da:7f:37:f3:13:28:15:4c:8f:0b:d6:21:e8:49:c2:
        67:c6:57:d8:25:93:c8:1b:5f:65:11:fc:df:1a:8d:ea:e8:04:
        59:74:19:a6:a1:e5:b1:87:d3:0b:b7:ba:72:29:0b:c3:f1:68:
        76:13:d9:ed:e5:d3:85:c1:fb:95:79:88:30:9d:f0:06:89:8c:
        84:89:fb:64:9d:58:90:53:b1:50:07:21:78:8a:22:d9:1a:d1:
        1d:03:53:2b:9a:82:4d:1b:51:c9:21:9f:02:0c:64:de:fa:97:
        6f:49:7c:97:2c:00:51:17:91:dd:5f:dc:f2:dd:3e:71:a4:56:
        08:c6:52:eb:f6:24:86:ee:47:9b:16:a5:99:eb:27:0a:92:10:
        b7:78:58:cb:e7:64:9d:4d:c2:d8:28:62:a6:5e:af:4b:59:c9:
        23:39:0b:7f:0d:88:72:f5:7c:64:00:32:a2:e2:85:58:39:9b:
        4e:7e:7f:0c:2e:76:bb:57:90:8b:5c:e9:51:15:7f:79:8a:9e:
        72:2b:25:5a:91:0b:bd:bf:8c:fb:68:f8:9a:01:0e:dc:c6:4e:
        9a:3e:e9:c1:1a:8d:5d:f3:80:8e:d6:08:4a:34:52:43:10:cd:
        e4:fa:26:c8:c5:eb:62:0e:29:69:71:12:7e:36:31:b7:27:86:
        3a:dd:0d:d7:28:c5:dc:b1:e3:7c:90:82:93:c1:c6:b9:5f:00:
        b9:26:4d:ae:3f:47:f3:c3:58:62:58:96:54:32:93:d1:4c:8c:
        5a:18:a9:2f:4d:41:5c:68:01:c5:86:d7:37:9b:50:da:45:65:
        8b:85:ce:12:10:c2:9f:49:20:1d:66:d5:ed:8f:c6:b2:aa:a0:
        73:c6:0d:2a:d5:7c:69:f8:86:23:01:f5:6b:ce:f1:e0:6b:36:
        8e:2a:26:bb:62:12:3e:c6:1e:18:33:8d:f8:aa:c5:ea:07:5a:
        65:00:12:76:15:39:4f:7c:f1:06:a6:18:6e:64:0a:01:06:1c:
        d1:14:1f:9a:fc:c3:84:ed:34:5d:77:54:32:5c:64:fe:0e:e7:
        2e:50:3b:5e:1d:5d:24:18:4d:02:41:e8:21:f8:fc:1c:c1:30:
        a1:a9:1b:2e:be:ba:7f:75:74:ff:84:68:db:38:e2:47:89:51:
        a1:65:14:c4:e7:9d:26:b0:f6:9d:a6:9a:37:c8:2e:a9:f8:d6:
        9f:e6:1c:d2:0a:7d:1a:e2:50:2c:c8:73:fa:db:47:52:2a:3e:
        26:e4:dc:fe:e3:18:89:c8
```

Następnie generujemy klucz prywatny naszego certyfikatu.

```bash
openssl genrsa -out cert-key.pem 4096
```

I tworzymy żadanie podpisu certyfikatu (CSR),

```bash
openssl req -new -sha256 -subj "/CN=*.lan" -key cert-key.pem -out cert.csr
```
Dodajemy informacje o alternatywnych nazwach domenowych.

```bash
echo "subjectAltName=DNS:*.lan" >> extfile.cnf
```

Na końcu możemy wygenerować sam certyfikat.

```bash
$ openssl x509 -req -sha256 -days 3650 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial
Certificate request self-signature ok
subject=CN = *.lan
Enter pass phrase for ca-key.pem:
```

A tak wygląda podgląd certyfikatu w formie tekstowej.

```bash
$ openssl x509 -in cert.pem -noout -nocert -text
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            50:00:1e:65:c6:65:14:e3:3e:2d:f8:4d:86:70:8f:9d:6e:6a:48:66
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = PL, ST = dolnoslaskie, L = Wroclaw, O = Pawel Mazur, CN = www.pmazur.pl
        Validity
            Not Before: Nov  1 04:56:26 2025 GMT
            Not After : Oct 30 04:56:26 2035 GMT
        Subject: CN = *.lan
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:b1:71:58:79:43:1f:91:39:46:d6:09:c4:45:32:
                    6e:56:02:37:68:88:c2:30:f9:0e:0e:1c:87:fc:df:
                    49:b8:d8:3e:9c:a8:26:fb:33:13:fa:2b:fc:7e:5c:
                    b5:e0:bb:2e:d3:05:9f:94:1e:e0:d6:30:7d:6f:5d:
                    1b:81:35:e3:ef:a4:90:5f:b0:0e:8f:8f:20:a0:9b:
                    e7:33:de:f1:88:a4:e4:4b:eb:9c:6a:b7:e6:f9:87:
                    51:2b:b8:d6:a4:67:e8:35:e9:b1:58:5d:77:4a:4d:
                    5d:3a:94:97:2e:99:4a:de:e2:43:ed:00:72:4a:82:
                    fe:0c:ca:52:df:72:93:c5:21:10:0b:ba:6c:9c:32:
                    0e:43:5b:56:c2:36:f8:5e:b1:fd:f7:bc:92:26:46:
                    0e:d8:fb:03:d4:16:d5:a2:89:c7:10:07:87:17:6b:
                    3b:c0:a7:2f:50:6b:ab:68:2d:76:af:5f:99:67:4d:
                    b4:e1:79:4f:d1:3b:0d:dd:bc:29:e4:5f:84:52:fa:
                    95:2b:5e:fa:8c:55:87:f4:94:d1:41:85:d6:d2:32:
                    8d:55:63:04:62:92:cb:f1:89:cf:7d:4d:d5:3b:e7:
                    08:3e:62:5c:a7:99:6b:f8:a3:47:a2:e8:83:55:37:
                    52:ea:f1:f3:db:12:ee:a8:12:52:38:90:36:80:f7:
                    c5:8c:0a:29:a7:e4:50:9f:55:3f:74:61:8f:2f:d9:
                    0b:49:0d:ee:37:63:b8:61:6b:41:9b:a5:9d:2b:e6:
                    14:62:41:f7:01:e1:bf:ec:b0:b5:b8:cd:63:40:73:
                    dc:6c:6a:04:0c:31:38:ba:44:12:cc:48:63:bf:8d:
                    47:fe:d1:e3:f8:26:a4:02:ff:04:93:5b:42:c1:61:
                    bf:e2:87:35:7c:80:ca:e4:a6:e7:88:28:05:12:3e:
                    43:c7:23:91:26:4d:92:61:11:c8:f9:9d:91:68:96:
                    d0:fb:53:af:ed:f2:12:83:e3:23:70:67:0e:71:bf:
                    31:ae:7a:cc:53:d9:32:a6:61:7d:c0:71:f8:58:b9:
                    9c:26:11:16:e2:5d:fc:99:85:ad:63:7d:3e:ea:9a:
                    b3:3d:fb:99:59:d7:5f:11:0f:75:08:6c:b0:22:4a:
                    ca:9d:ed:9d:39:01:7f:28:0c:e1:09:a9:5e:e2:b0:
                    18:0f:49:9e:45:1f:9d:87:dd:71:ad:53:f6:3f:2a:
                    6d:d5:82:f1:8d:82:00:2b:4e:1c:c4:0d:35:c8:89:
                    8b:0e:3a:24:55:c5:2c:48:6c:c3:21:1d:17:15:90:
                    4c:fd:24:ba:cc:a6:ad:88:2b:91:b6:bf:e1:77:d7:
                    7e:d8:c8:c5:12:de:05:55:15:90:8d:ef:ed:40:06:
                    82:c1:93
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        51:65:a5:43:c5:a3:6e:fb:35:07:7e:27:5c:fb:f1:b5:48:a8:
        64:e6:52:29:0d:0c:70:48:c3:48:15:2a:0e:0c:22:23:78:11:
        24:f3:86:7b:91:71:99:6c:45:a0:6e:87:ad:40:ed:38:f4:f5:
        10:09:64:ff:be:d1:f3:95:68:6d:2e:44:7a:0e:c1:f0:89:37:
        e6:a8:a8:57:cf:01:e3:94:c0:e9:44:c4:8c:cc:00:37:0c:34:
        c9:d0:62:a0:27:bf:56:7b:ea:49:c1:a0:11:ec:d6:e6:64:14:
        20:2d:46:90:6d:da:74:0c:48:61:43:6c:52:01:f0:e1:44:10:
        ac:a5:80:a6:a5:3f:a4:57:45:06:da:14:9c:2a:69:bc:16:e4:
        bd:34:88:19:19:a8:d7:55:15:1b:e1:54:d6:62:e9:ce:57:66:
        7e:c9:b4:55:62:2b:38:36:81:f3:06:0c:87:b3:f9:03:3b:48:
        d1:b0:33:f0:0e:96:d5:27:6a:20:36:80:ac:a7:c4:96:e8:f3:
        8f:3e:2f:a7:02:d6:5b:15:96:26:84:b0:c4:f1:01:d4:c6:69:
        2f:36:ad:8b:e7:ea:18:a5:43:13:88:90:56:a1:4c:52:ae:39:
        9f:c2:87:40:86:9a:a2:ad:b1:10:e9:36:aa:9b:09:c1:01:22:
        1f:e9:c4:9f:1f:67:9a:8f:1e:5c:0d:a2:24:2d:f3:3c:fd:d0:
        75:ef:2b:4c:06:f8:aa:f3:91:f7:1b:54:1c:02:75:0e:ee:62:
        22:cf:42:22:35:62:b6:59:9a:32:86:11:e1:9e:54:5d:e8:1f:
        57:34:7a:ba:f6:4e:2d:26:6f:53:52:76:a9:3f:e1:b8:15:db:
        25:ea:ed:01:25:96:6a:56:d8:7f:15:c0:b9:1b:fc:59:3c:f6:
        ee:9f:38:15:bd:07:af:69:ac:64:9d:b3:2b:b7:3e:9f:10:31:
        74:d4:41:88:7c:9d:70:20:00:bb:18:27:0c:21:37:a7:d9:d5:
        66:71:50:1a:47:dd:c1:86:f9:1e:c1:a0:85:9b:8e:8e:a4:4a:
        f3:57:d5:34:08:f5:5c:ff:a9:4d:22:fe:28:62:2d:da:7e:45:
        9c:34:6f:cb:60:02:fa:59:86:81:af:8b:08:4c:6d:5b:ab:77:
        55:bb:34:b1:39:c5:a8:59:af:f8:b5:c3:6a:76:58:67:4d:ea:
        c0:55:55:20:97:03:a6:fc:33:fe:20:1e:c5:1c:98:36:f8:b1:
        fc:2b:ab:b2:77:33:b8:04:95:dd:46:f4:d7:20:05:7a:98:4f:
        be:ab:02:6b:6f:18:3c:88:9f:af:a4:1f:b1:02:a7:86:ef:55:
        0c:fc:6a:49:ab:ae:ba:cb
```

# Wykorzystanie certyfikatu

Tak przygotowany certyfikat należy skonfigurować w aplikacji odpowiedzialnej za komunikację HTTPs. W moim przypadku
zostanie skonfigurowana usługa Proxmox. Szczegóły zostały opisane w dokumentacji [Certificate Management](https://pve.proxmox.com/wiki/Certificate_Management).

![ssl-cert.png](/assets/images/ssl-cert.png)

# Instalacja CA

Aby nasze CA było zaufane, na wszystkich urządzeniach klienckich należy zainstalować je w systemie. W przypadku Debiana
zrobimy to umieszczając plik CA w katalogu `/usr/local/share/ca-certificates/` i aktualizując Cert Store.

```bash
$ sudo cp ca.pem /usr/local/share/ca-certificates/lan-ca.crt
```
```bash
$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...

Adding debian:lan-ca.pem
done.
done.
```


