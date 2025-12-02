---
title: Udostępnianie usług przy pomocy Cloudflare Tunnel
tags: cloudflare 
---

Mając do dyspozycji serwer i usługę którą chcelibyśmy udostępnić w sieci, musimy spełnić kilka warunków aby umożliwić do
niej dostęp ze świata. Warunkiem koniecznym jest dostęp do sieci internet oraz konieczność przekierowania odpowiednich
portów do naszego serwera. W momecie, kiedy nie mamy publicznie dostępnego adresu IP, ciekawą alternatywą może stać się 
narzędzie [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/).

# Tworzenie tunelu

Na początku jesteśmy zobowiązanie stworzyć tunel będący wykorzystywany w daleszej konfiguracji. Możemy to zrobić zarówno
od [strony panelu](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/get-started/create-remote-tunnel/)
jaki i też [API](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/get-started/create-remote-tunnel-api/). 
Na tym etapie zajmiemy się stworzeniem tunelu poprzez panel.
