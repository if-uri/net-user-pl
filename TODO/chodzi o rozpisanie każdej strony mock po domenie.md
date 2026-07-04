<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# chodzi o rozpisanie każdej strony mock po domenie z proxy dla domen lokalnych mockow z oryginalnymi nazwami domen, ale dostepnych tylklo lokalnie

Tak — chodzi Ci o lokalny lab, gdzie każda atrapa działa pod własną nazwą domenową, np. `facebook.localhost`, `instagram.localhost`, `youtube.localhost`, ale wszystko jest dostępne tylko na Twoim komputerze przez reverse proxy. To najlepiej zrobić przez Docker Compose + Traefik lub nginx-proxy z routingiem po nagłówku `Host`, a najprostszy wariant bez grzebania w DNS to użycie domen `*.localhost`, bo `.localhost` jest zarezerwowaną domeną wskazującą na `127.0.0.1`.[^1]

## Schemat domen

Najbezpieczniej nie używać prawdziwych domen typu `facebook.com`, tylko lokalne odpowiedniki zachowujące nazwę marki, np. `facebook.localhost`, `instagram.localhost`, `x.localhost`, `youtube.localhost`, `reddit.localhost`, `tiktok.localhost`, `linkedin.localhost`, `netflix.localhost`, `spotify.localhost`, `amazon.localhost`, `airbnb.localhost`. Dzięki temu zachowujesz oryginalne nazwy serwisów w adresie, ale nie podszywasz się pod realne publiczne domeny i nie musisz dopisywać ich ręcznie do `/etc/hosts`, jeśli używasz `.localhost`.[^1]

Przykładowa rozpiska może wyglądać tak:

- `facebook.localhost` → mock Facebooka
- `instagram.localhost` → mock Instagrama
- `x.localhost` albo `twitter.localhost` → mock X/Twittera
- `youtube.localhost` → mock YouTube
- `reddit.localhost` → mock Reddita
- `whatsapp.localhost` → mock WhatsAppa
- `tiktok.localhost` → mock TikToka
- `linkedin.localhost` → mock LinkedIna
- `netflix.localhost` → mock Netflixa
- `amazon.localhost` → mock Amazona[^2][^1]


## Proxy lokalne

Traefik bardzo dobrze pasuje do takiego układu, bo w Dockerze może routować ruch na podstawie reguł `Host(...)` zadanych jako etykiety kontenerów. Przykład z dokumentacji i praktycznych porad pokazuje, że można wystawić usługę pod adresem w stylu `example.test` albo `my-foo.localhost` bez portów w URL, a Traefik prześle ruch do właściwego kontenera po wewnętrznej sieci Dockera.[^3][^2][^1]

Minimalny wzorzec Compose wygląda tak:

```yaml
services:
  traefik:
    image: traefik:v3.0
    command:
      - --api.insecure=true
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --entrypoints.web.address=:80
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - mocknet

  facebook:
    image: nginx:alpine
    volumes:
      - ./sites/facebook:/usr/share/nginx/html:ro
    labels:
      - traefik.enable=true
      - traefik.http.routers.facebook.rule=Host(`facebook.localhost`)
      - traefik.http.services.facebook.loadbalancer.server.port=80
    networks:
      - mocknet

  instagram:
    image: nginx:alpine
    volumes:
      - ./sites/instagram:/usr/share/nginx/html:ro
    labels:
      - traefik.enable=true
      - traefik.http.routers.instagram.rule=Host(`instagram.localhost`)
      - traefik.http.services.instagram.loadbalancer.server.port=80
    networks:
      - mocknet

networks:
  mocknet:
    driver: bridge
```

W tym modelu każda „strona” to osobny serwis, a Traefik kieruje ruch po domenie lokalnej do właściwego katalogu lub aplikacji. Podobny efekt da się zrobić przez `jwilder/nginx-proxy`, który mapuje kontenery przez zmienną `VIRTUAL_HOST`, ale Traefik jest wygodniejszy przy większej liczbie usług i etykiet w Compose.[^4][^5][^6][^3][^2]

## DNS lokalny

Masz dwa warianty:

- `*.localhost` — najprostszy, zwykle bez dodatkowej konfiguracji DNS, bo `.localhost` rozwiązuje się lokalnie do loopback.[^1]
- `*.test`, `*.docker`, `*.loc` albo własna strefa — wtedy potrzebujesz `/etc/hosts` albo lokalnego resolvera typu `dnsmasq`, który może mapować wildcardy na `127.0.0.1`, np. `*.test 127.0.0.1` albo `address=/.docker/127.0.0.1`.[^4][^3]

Jeśli chcesz zachować jeszcze bardziej „realistyczny” zapis, możesz użyć np. `facebook.test`, `instagram.test`, `youtube.test`, ale wtedy trzeba skonfigurować lokalny DNS lub ręczne wpisy hostów, bo w przeciwieństwie do `.localhost` nie zadziała to samo z siebie. Dla większego labu z wieloma mockami sensowniejszy jest lokalny wildcard DNS przez `dnsmasq` niż ręczne dopisywanie kilkunastu rekordów.[^3][^4]

## Nazewnictwo mocków

Jeśli chcesz „rozpisać każdą stronę mock po domenie”, proponuję standard:

- brand-based host: `facebook.localhost`, `instagram.localhost`, `reddit.localhost`
- opcjonalny prefiks środowiska: `mock-facebook.localhost`, `mock-instagram.localhost`
- warianty funkcjonalne: `auth.facebook.localhost`, `admin.instagram.localhost`, `api.reddit.localhost`, jeśli mock ma kilka części[^2][^1]

Praktycznie najczyściej wygląda jednak jedna domena na jeden mock, np. `facebook.localhost`, bo wtedy reverse proxy i plik Compose pozostają czytelne. Jeśli chcesz uruchamiać jednocześnie kilka wersji tej samej atrapy, możesz stosować np. `facebook-v1.localhost` i `facebook-v2.localhost`, nadal w tej samej sieci Docker.[^2]

## Plan wdrożenia

Żeby to zrobić porządnie, trzeba przygotować:

1. listę domen lokalnych dla wszystkich marek,
2. osobny serwis dla każdego mocka,
3. wspólną sieć Docker,
4. reverse proxy z regułami `Host(...)`,
5. opcjonalnie `dnsmasq`, jeśli nie chcesz używać `.localhost`.[^4][^1][^2]

Mogę teraz rozpisać gotowy projekt: pełną tabelę domen dla kilkudziesięciu serwisów oraz kompletny `docker-compose.yml` z Traefikiem i placeholderami dla każdego mocka.
<span style="display:none">[^10][^11][^12][^13][^14][^15][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: https://dev.to/jawira/defining-dynamic-host-rule-in-traefik-v2-4doj?comments_sort=top

[^2]: https://doc.traefik.io/traefik/routing/providers/docker/

[^3]: https://stackoverflow.com/questions/73628758/using-custom-local-domain-with-docker

[^4]: https://gist.github.com/tobiashm/b43d24a5204d8bb623a7edcec1aeea6e

[^5]: https://stackoverflow.com/questions/45416518/subdomains-nginx-proxy-and-docker-compose

[^6]: https://hub.docker.com/r/jwilder/nginx-proxy

[^7]: https://medium.com/@juan_cortes/local-domains-through-nginx-proxy-and-docker-13d97ee8c010

[^8]: https://hirocks.jp/dockerでdnsmasqを使ってワイルドカードでサブドメインを/

[^9]: https://github.com/acwstudio/nginx-reverse-proxy-ssl-local

[^10]: https://hub.docker.com/r/vmeijer/nginx-proxy

[^11]: https://chuniversiteit.nl/operations/reverse-proxy-for-docker-compose

[^12]: https://stackoverflow.com/questions/48398338/wildcard-for-local-dev-hosts/70057218

[^13]: https://reprodev.com/custom-local-hostnames-with-nginx-proxy-manager-and-pi-hole/

[^14]: https://github.com/AnalogJ/thesparktree-blog/blob/master/_posts/2020-05-27-traefik-advanced-config.md

[^15]: https://github.com/simba77/docker-php/blob/main/traefik/README.md

