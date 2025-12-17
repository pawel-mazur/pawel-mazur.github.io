---
title: Wykorzystanie Cloudflare w dostępie do aplikacji webowych
tags: cloudflare web waf app
---

Obecnie udostępnianie stron w sieci internet wiąże się z wieloma ryzykami. Potrzeba zapewnienia
wysokiej dostępności, tak aby zaspokoić potrzeby użytkowników dot. stabilnego i niezawodnego dostępu do ich usług, czy
ataki nakierowywane na naszą stronę, to część z nich, przed którymi będziemy chcieli się zabezpieczyć. W pewnych
aspektach pomocne będzie nam dobrze znane narzędzie, jakim jest [Cloudflare](https://www.cloudflare.com/).

# Zakładanie konta

Pierwszym krokiem, który będziemy musieli wykonać, jest założenie konta. Zrobimy to bezpośrednio, na stronie [Sign Up](https://dash.cloudflare.com/sign-up).
Tam będziemy musieli podać nasz adres mail, na który otrzymamy link potwierdzający oraz hasło do naszego konta.
Po chwili powinniśmy móc się zalogować do naszego panelu.

![sign-up.png](/assets/images/cloudflare/00-sign-up.png)

# Konfiguracja konta

Po zalogowaniu w naszym panelu będziemy musieli dodać naszą domenę do konfiguracji. Będzie się to również wiązało z koniecznością
przekierowania wpisów DNS na serwer Cloudflare. Dzięki temu aplikacją będzie mogła zarządzać potrzebnymi wpisami DNS w naszym imieniu.

Do wyboru mamy możliwość automatycznego przeskanowania i dodania rekordów DNS, ręcznego ich wprowadzenia lub przesłania pliku
ze zdefiniowanymi strefami dns. W moim przypadku wystarczyło automatyczne skanowanie rekordów.

![configre.png](/assets/images/cloudflare/10-configure.png)

# Wybranie planu

W kolejnym kroku będziemy musieli określić plan, na który będziemy chcieli się zdecydować. Na ten moment dostępne są następujące
plany taryfowe: Free, Pro, Buissnes oraz Enterprice. Dokładny opis aktualnie dostępnych planów jest także dostępny na stronie
[oferty](https://www.cloudflare.com/plans/). Na tym etapie zdecydowałem się wybrać plan Free.

![plans.png](/assets/images/cloudflare/20-plans.png)

# Konfiguracja serwerów nazw

Na tym etapie konieczne będzie skonfigurowanie serwerów nazw naszej domeny poprzez przekierowanie ich do usługi Cloudflare. Zrobimy to u dostawcy
naszej domeny, poprzez przypisanie dla niej rekordów NS kierujących do wskazanych w panelu adresów nameserwerów Cloudfalre.

![dns-nameservers.png](/assets/images/cloudflare/30-dns-nameservers.png)

[//]: # (# Obsługa certyfikatów SSL/TLS)

# Bezpieczeństwo strony

Gdy nasza domena jest już poprawnie skonfigurowana, ruch kierowany do naszej strony domyślnie będzie przechodził przez
[Cloudflare WAF](https://developers.cloudflare.com/waf/). Niewątpliwą korzyścią będzie dla nas wstępne odfiltrowanie
niepożądanego ruchu w tym prób ataku wykonania na stronę poprzez wykorzystanie podatności, nadmiernego obciążenia naszej
strony czy chociażby ruch botów. W sekcji `Security` zobaczmy także podsumowanie naszych statystyk.

![security.png](/assets/images/cloudflare/40-security.png)

### Custom rules

Z użyciem [Custom rules](https://developers.cloudflare.com/waf/custom-rules/), możemy zdefiniować, jak ma wyglądać ruch kierowany do naszej strony.
Dzięki nim możemy wybrać możliwość ograniczenia ruchu z konkretnych adresów IP, kontynentów czy krajów. I tak dla przykładu
skonfigurujemy sobie zasób, dla którego dostęp będzie możliwy zawsze jedynie po rozwiązaniu wyzwania.

![custom-rule.png](/assets/images/cloudflare/50-custom-rule.png)

W tym momencie na stronie dostaliśmy zagadkę do rozwiązania.

![challange.png](/assets/images/cloudflare/60-challange.png)

Po jej rozwiązaniu jesteśmy kierowani do naszej właściwej strony.

### Rate limiting rules

Dzięki regułom [Rate limiting rules](https://developers.cloudflare.com/waf/rate-limiting-rules/) będziemy mieli możliwość
ograniczenia ruchu do naszej strony na podstawie zadanych kryteriów dot. ograniczenia przepustowości. I tak dla przykładu możemy
ograniczyć dostęp do naszej strony po przekroczeniu liczby wyświetleń na podstawie adresu IP na zadany okres.

![rate-limit.png](/assets/images/cloudflare/70-rate-limit.png)

Po przekroczeniu liczby prób użytkownikowi zostanie zwrócona strona z informacją o jej przekroczeniu, a jedynie po upływie
założonego czasu będzie mógł uzyskać ponownie dostęp do strony.

![banned.png](/assets/images/cloudflare/80-banned.png)

### Managed rules

Konfiguracja pozwala na włączenie wstępnie skonfigurowanych reguł ([Managed Rules](https://developers.cloudflare.com/waf/managed-rules/)),
z których użyciem ruch będzie przetwarzany i kierowany do naszej strony po spełnieniu odpowiednich warunków. Domyślnie będzie zawsze włączona reguła
[Cloudflare Managed Ruleset](https://developers.cloudflare.com/waf/managed-rules/reference/cloudflare-managed-ruleset/).
Do naszej dyspozycji jest dostępnych również kilka innych np. [Cloudflare OWASP Core Ruleset](https://developers.cloudflare.com/waf/managed-rules/reference/owasp-core-ruleset/)
[Cloudflare Exposed Credentials Check Managed Ruleset](https://developers.cloudflare.com/waf/managed-rules/reference/exposed-credentials-check/),
czy [Cloudflare Sensitive Data Detection](https://developers.cloudflare.com/waf/managed-rules/reference/sensitive-data-detection/).
Darmowy plan pozwala jednak na skorzystanie jedynie z tej domyślnie włączonej, pozostałe są dostępne w wyższych planach.

# Awarie

Jak można by było przypuszczać, jeśli chodzi o zewnętrzne narzędzia, jest też druga strona medalu. I tak dla przykładu
w ostatnim czasie pojawiła się awaria infrastruktury Cloudflare, co w dużej mierze przyczyniło się do paraliżu sporej liczby
stron internetowych, zwłaszcza tych globalnie dostępnych. Szczegóły tego incydentu zostały opisane na oficjalnej stronie
[Cloudflare Global Network experiencing issues](https://www.cloudflarestatus.com/incidents/8gmgl950y3h7).

Trzeba więc pamiętać, że w takiej sytuacji nasze usługi także będą narażone na konsekwencje z tym związane.
W przypadku problemów z działaniem naszych usług warto będzie zweryfikować czy na pewno Cloudflare ma pełną
zdolność operacyjną. Sprawdzimy to na stronie [Cloudflare System Status](https://www.cloudflarestatus.com/).

# Podsumowanie

Cloudflare jako narzędzie pozwala na proste i skuteczne skonfigurowanie zabezpieczeń oraz ograniczenie ruchu do naszej
strony na podstawie przyjętych reguł. Niewątpliwie taką pracę musiałaby wykonać po stronie serwera nasza aplikacja bądź
inne dostępne narzędzia typu WAF np. [OWASP Modsecurity](https://modsecurity.org/) uruchomione w naszej infrastrukturze.
Wymaga to jednak zarządzania oraz utrzymania takich usług, co niewątpliwie wiąże się z dodatkowymi kosztami. Myślę, że na
tym etapie rozwiązanie jest co najmniej warte poświęcenia uwagi i nawet w darmowym planie oferuje całkiem sporo.
