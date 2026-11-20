# TextNexus — static site image.
#
# Build from the repository root so the site files are in the build context:
#   docker build -f "Self hosting/Dockerfile" -t textnexus-site .
# or simply:
#   docker compose -f "Self hosting/docker-compose.yml" up -d --build
FROM nginx:1.27-alpine

LABEL org.opencontainers.image.title="TextNexus" \
      org.opencontainers.image.description="Static front end for the TextNexus site" \
      org.opencontainers.image.source="https://github.com/wwebsitesystem-stack/textnexus-self-host"

WORKDIR /usr/share/nginx/html

# Copy the served pages and their assets. Globs keep new top-level pages,
# scripts and styles working without editing this list on every deploy; media
# and games are copied as directories because their pages reference them.
# Build output, documentation, Cloudflare Functions and this folder stay out.
COPY *.html ./
COPY *.css *.js *.json *.svg *.png *.webmanifest *.xml *.txt ./
COPY media ./media
COPY games ./games
COPY ghostlink-games ./ghostlink-games

COPY ["Self hosting/nginx.conf", "/etc/nginx/conf.d/default.conf"]

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -q --spider http://127.0.0.1/ || exit 1
