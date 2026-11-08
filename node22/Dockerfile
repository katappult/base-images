# Images de base des frontends Next.js : Node 22 (LTS) sur Alpine.
#   cible build   -> ghcr.io/katappult/base-node22-build : étapes deps / builder (npm présent)
#   cible runtime -> ghcr.io/katappult/base-node22       : étape runner (npm / npx / corepack retirés : inutiles au
#                    runtime, leurs dépendances embarquées (tar, glob, cross-spawn...) remontent des CVE)
# Amont épinglé par digest, mis à jour par Dependabot (PR). Correctifs OS (apk upgrade) à chaque build.
# Pas de USER ici : chaque produit choisit le sien (OpenShift impose un UID aléatoire, groupe 0).
FROM node:22-alpine@sha256:0e1754758c2df4cfa3e68d59e2e1b7374bb7fd9ea6328eaaacf15cc934af8202 AS build
ENV TZ=UTC NEXT_TELEMETRY_DISABLED=1
RUN apk upgrade --no-cache && apk add --no-cache ca-certificates tzdata

FROM build AS runtime
# gettext : envsubst (config runtime public/config.js générée au démarrage par certains frontends)
RUN apk add --no-cache gettext \
 && rm -rf /usr/local/lib/node_modules/npm /usr/local/lib/node_modules/corepack \
      /usr/local/bin/npm /usr/local/bin/npx /usr/local/bin/corepack
ENV NODE_ENV=production
