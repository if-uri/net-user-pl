# Mock lab lokalnych domen

Ten pakiet uruchamia zestaw lokalnych atrap stron pod domenami `*.localhost` przez Traefik i Docker Compose.
.localhost jest domeną specjalną używaną lokalnie, więc adresy typu `facebook.localhost` lub `youtube.localhost` są dostępne tylko na komputerze lokalnym po skierowaniu na loopback [RFC 6761].

## Zawarte domeny

- `facebook.localhost` → Facebook
- `instagram.localhost` → Instagram
- `x.localhost` → X / Twitter
- `youtube.localhost` → YouTube
- `reddit.localhost` → Reddit
- `tiktok.localhost` → TikTok
- `linkedin.localhost` → LinkedIn
- `whatsapp.localhost` → WhatsApp
- `discord.localhost` → Discord
- `telegram.localhost` → Telegram
- `messenger.localhost` → Messenger
- `snapchat.localhost` → Snapchat
- `pinterest.localhost` → Pinterest
- `tumblr.localhost` → Tumblr
- `threads.localhost` → Threads
- `netflix.localhost` → Netflix
- `spotify.localhost` → Spotify
- `amazon.localhost` → Amazon
- `airbnb.localhost` → Airbnb
- `google.localhost` → Google
- `gmail.localhost` → Gmail
- `googlemaps.localhost` → Google Maps
- `wikipedia.localhost` → Wikipedia
- `github.localhost` → GitHub
- `stackoverflow.localhost` → Stack Overflow

## Start

1. Uruchom Dockera.
2. Wejdź do katalogu projektu.
3. Wykonaj:

```bash
docker compose up -d
```

4. Otwórz w przeglądarce np.:
   - `http://facebook.localhost`
   - `http://instagram.localhost`
   - `http://youtube.localhost`

5. Panel Traefika jest pod `http://localhost:8080`.

## Jak podmienić placeholder na prawdziwy mock

- Statyczny mock HTML/CSS/JS: nadpisz pliki w `sites/nazwa/`.
- Repo Node/Next/Vite: zamiast `nginx:alpine` dodaj osobny serwis build/run i zostaw regułę `Host(...)`.
- Repo Django/Flask/Rails: wystaw port aplikacji wewnątrz sieci i ustaw `traefik.http.services.<nazwa>.loadbalancer.server.port=<port>`.

## Gdy chcesz inne domeny niż .localhost

Jeśli zamiast `*.localhost` chcesz np. `*.test`, potrzebujesz lokalnego DNS albo wpisów w `/etc/hosts`; do wildcardów wygodny jest `dnsmasq`.

## Struktura

```
mock-lab/
├── docker-compose.yml
├── README.md
└── sites/
    ├── facebook/
    ├── instagram/
    ├── youtube/
    └── ...
```
