# TextNexus · self-host kit

The public server configuration for running the TextNexus site on your own hardware.

[Read the illustrated guide on textnexus.me](https://textnexus.me/self-host.html) · [Download the kit](https://github.com/wwebsitesystem-stack/textnexus-self-host/archive/refs/heads/main.zip) · [Open the one-page guide](index.html)

> This repository is the **kit**, not the website. The site files stay in the TextNexus source; this kit supplies the Dockerfile, Compose file and nginx server block, plus the complete walkthrough.

![The Docker step of the guide](https://textnexus.me/media/self-host/step-docker.svg)

## What's inside

| Path | Purpose |
| --- | --- |
| `index.html` | The complete self-hosting guide as one page |
| `Self hosting/Dockerfile` | nginx image containing the site files and server block |
| `Self hosting/docker-compose.yml` | Builds the image from the site root, publishes port 8080 |
| `Self hosting/nginx.conf` | gzip, cache headers and baseline security headers |
| `Self hosting/README.md` | The same guide in plain Markdown |

## Quick start

1. Clone this kit, or choose **Code → Download ZIP**.
2. Copy the `Self hosting` folder into the root of your TextNexus site files.
3. From that site root, run:

   ```bash
   docker compose -f "Self hosting/docker-compose.yml" up -d --build
   ```

4. Open <http://localhost:8080>.

Then publish `database.rules.json` in Firebase, add your hostname under Firebase Authentication → Settings → Authorized domains, and put a TLS proxy in front before exposing the site to the internet.

Full steps, screenshots and troubleshooting: [textnexus.me/self-host.html](https://textnexus.me/self-host.html).

## The one-page guide

`index.html` carries the whole walkthrough in one file: requirements, seven steps, every command, the nginx alternative, updates, final checks and the five common failures. Open it locally, or read the hosted version above.

## License

MIT — see [LICENSE](LICENSE). TextNexus and the Nexus name belong to the TextNexus project.
