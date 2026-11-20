# Self-hosting TextNexus

This folder is the server configuration for running the TextNexus site on your own
server instead of Cloudflare Pages. The site is a static front end: HTML, CSS and
JavaScript that talk to Firebase directly from the browser. There is no build
step and no application server to keep running.

> **Public kit.** This copy ships in
> [textnexus-self-host](https://github.com/wwebsitesystem-stack/textnexus-self-host).
> The site files themselves stay in the TextNexus source, which is private. Copy
> this `Self hosting` folder into the root of your copy of those files before
> running Docker or nginx; the paths below assume that layout.

The live site is still published on **Cloudflare Pages** (project `textnexus`)
from the private repository. This folder is the way to run the same files on your
own server; the Docker and plain-nginx instructions work unchanged.

## Requirements

- Docker 24+ with Compose v2 (recommended), **or** nginx 1.24+ if you install it yourself.
- Outbound HTTPS from the browser to Firebase, Google Fonts, Cloudflare Turnstile
  and the donation widget. Nothing is bundled locally.
- A copy of the TextNexus site files (see the note above).

## Quick start (Docker)

From the site root — the folder that contains `index.html` and this
`Self hosting` directory:

```bash
docker compose -f "Self hosting/docker-compose.yml" up -d --build
```

Then open <http://localhost:8080>.

The image is built from the site root, because the site files sit one level above
this folder. It copies the pages and their assets — HTML, CSS, JavaScript, JSON,
images, `media/`, `games/` and `ghostlink-games/` — while documentation,
Cloudflare Functions, build output and this folder stay out.

Stop it with:

```bash
docker compose -f "Self hosting/docker-compose.yml" down
```

## Without Docker

Copy the site files and the server block into place:

```bash
sudo mkdir -p /var/www/textnexus
sudo cp -R *.html *.css *.js *.json *.svg *.png *.webmanifest *.xml *.txt \
        media games ghostlink-games /var/www/textnexus/
sudo cp "Self hosting/nginx.conf" /etc/nginx/sites-available/textnexus
sudo ln -s /etc/nginx/sites-available/textnexus /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

Change the `root` line in `nginx.conf` from `/usr/share/nginx/html` to
`/var/www/textnexus` for a host install.

`python3 -m http.server` also works for a quick local look, but it is not a real
deployment: it serves the wrong MIME types for some assets and the service-worker
pages (Ghostlink, Jeoweb) will not behave correctly.

## What is in this folder

| File | Purpose |
| --- | --- |
| `Dockerfile` | nginx image containing the site files and the server block |
| `docker-compose.yml` | Builds the image from the site root and publishes port 8080 |
| `nginx.conf` | Server block: gzip, cache headers, baseline security headers |
| `README.md` | This document |

The site also ships `404.html` at the site root. `nginx.conf` wires it up with
`error_page 404 /404.html;` so unknown paths return the branded page with a real
404 status.

## Firebase

The Firebase web config is written into each page, pointing at the
`nexus-web-development-official` project. That key is public by design; access is
controlled by the Realtime Database rules, not by hiding the key.

The rules live in `database.rules.json` at the site root. **nginx does not apply
them** — Firebase does. Until they are deployed, the database keeps whatever rules
were last published:

- Firebase Console → Realtime Database → Rules → paste `database.rules.json` → Publish, or
- `firebase deploy --only database --project nexus-web-development-official`.

Also make sure, in the Firebase Console:

- Authentication → Sign-in method: Google is enabled.
- Authentication → Settings → Authorized domains: add the hostname you serve this
  site from (for example `localhost` during testing, or your domain).
- If the Google OAuth consent screen is in Testing, add test accounts under
  Audience → Test users, or publish the app.

## Cloudflare Functions

Cloudflare Pages also runs `functions/api/` for the feedback and registration
forms. The static image does not execute them, so `/api/*` returns 404 on a
self-hosted copy; the Firebase-backed pages still work. Host the Functions
separately if you need those forms.

## AI with Lofin.dev

AI is provided through our partnership with [Lofin.dev](https://lofin.dev/). The
Nexus AI item in the sidebar links directly to Lofin.dev, and `creator.html` is a
bare redirect for old links. There is no AI service, API key or additional
container to run for this integration.

## Custom domain and HTTPS

Put a TLS-terminating proxy in front of the container. With Caddy:

```caddyfile
nexus.example.com {
    reverse_proxy 127.0.0.1:8080
}
```

With nginx and certbot, proxy port 80/443 to `127.0.0.1:8080` and run
`certbot --nginx`. A Cloudflare Tunnel also works and avoids opening ports.

Then add the hostname to Firebase's authorized domains.

## Updating

```bash
git pull
docker compose -f "Self hosting/docker-compose.yml" up -d --build
```

## Troubleshooting

- **`/textnexus.html` returns 404.** Server filesystems are case-sensitive and
  the real file is `TextNexus.html`. Link to it with the exact casing.
- **Pages load but data is empty, console shows `PERMISSION_DENIED`.** The
  published database rules do not allow that read. Deploy `database.rules.json`
  and confirm the user is signed in.
- **Ghostlink or Jeoweb misbehave.** They register a service worker and load
  remote scripts; make sure the site is served over HTTP(S) from a single
  origin, then hard-reload to drop a stale worker.
- **Ads or the Turnstile gate never appear.** An ad blocker or a missing
  third-party script will do that; both are expected on a filtered network.

## Security notes

- Admin and Owner pages are gated in JavaScript; the page markup is public. The
  database rules are the real boundary, so keep them deployed. `creator.html`
  is a public partner page and contains no account data.
- No secrets belong in this image. The only credential in the site files is the
  public Firebase web API key, which should be restricted by HTTP referrer in
  the Google Cloud console.
- `nginx.conf` intentionally has no Content-Security-Policy: the pages use
  inline scripts and several CDNs. Add one only with a policy tested per page.
