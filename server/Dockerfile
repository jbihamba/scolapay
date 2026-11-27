
# syntax=docker/dockerfile:1

# Image officielle Node.js 24
FROM node:24-bookworm-slim AS base

# Répertoire de travail de l'application
WORKDIR /app

# Utiliser les headers Node.js locaux
ENV npm_config_nodedir=/usr/local

# Les dépendances natives (better-sqlite3 notamment) peuvent nécessiter une
# compilation pendant l'installation de npm.
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*

FROM base AS dependencies

# Copier les fichiers de dépendances avant le code pour profiter du cache Docker
COPY package*.json ./

RUN npm ci

FROM dependencies AS development

# Copier le code source
COPY . .

# Exposer le port AdonisJS
EXPOSE 3333

# Le compose utilise cette cible pour le développement avec HMR.
CMD ["npm", "run", "dev"]

FROM dependencies AS build

COPY . .
RUN npm run build && npm prune --omit=dev

FROM node:24-bookworm-slim AS production

WORKDIR /app
ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV PORT=3333

COPY --from=build --chown=node:node /app/package*.json ./
COPY --from=build --chown=node:node /app/node_modules ./node_modules
COPY --from=build --chown=node:node /app/build ./build

USER node
EXPOSE 3333

CMD ["node", "build/bin/server.js"]

# La cible par défaut reste le développement pour préserver `docker compose up`.
FROM development AS default
