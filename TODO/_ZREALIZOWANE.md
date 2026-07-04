# Status: ZREALIZOWANE (2026-07-04)

Cztery pliki research w tym katalogu (mock lab lokalnych domen social/global)
zostały zrealizowane — ale w mocniejszym wariancie niż proponowany Traefik +
`*.localhost`. Wpięto w istniejący wzorzec bliźniaka: **Caddy + lokalny CA +
prawdziwe nazwy domen + zdarzenia URI**.

Wynik:
- `sites/webmock/brands.json` — rejestr 24 atrap (facebook.com, youtube.com,
  allegro.pl, google.com, gmail.com, wikipedia.org, github.com, …);
- `sites/webmock/app.py` — jeden serwer routujący po nagłówku Host, renderuje
  markową stronę per domena, emituje `web://<domena>/...` na szynę (hasła nie logowane);
- `sites/webmock/build.py` — generator: z brands.json → certy-lista, bloki Caddy, aliasy;
- wpięte w `ca/gen.sh`, `proxy/Caddyfile` (+ `proxy/mocks.caddy`), `compose.net.yml`;
- pełna lista + sposób uruchomienia + jak rozszerzać + linki do repo z klonami
  (clone-wars, MockSocial): `sites/webmock/README.md`.

Zweryfikowane end-to-end: routing po Host (24 marki), akcje POST, zdarzenia web://
na szynie, HTTPS przez proxy z lokalnym CA (ssl_verify=0).

Uruchomienie:
    cd net-user-pl && python3 sites/webmock/build.py && bash ca/gen.sh
    docker compose -f compose.net.yml up -d --build
