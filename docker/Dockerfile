# syntax = docker/dockerfile:experimental

FROM ghcr.io/linuxserver/baseimage-ubuntu:noble as base

RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
RUN apt install -y --no-install-recommends nodejs \
  libglib2.0-0t64 \
  libnss3 \
  libatk1.0-0t64 \
  libatk-bridge2.0-0t64 \
  libcups2t64 \
  libdrm2 \
  libxcb1 \
  libxkbcommon0 \
  libxcomposite1 \
  libxdamage1 \
  libxfixes3 \
  libxrandr2 \
  libgbm1 \
  libpango-1.0-0 \
  libcairo2 \
  libasound2t64 && \
  npm i -g pnpm

##### DEPENDENCIES

FROM base AS deps

WORKDIR /app

COPY prisma ./
COPY package.json pnpm-lock.yaml ./

RUN --mount=type=cache,id=pnpm-store,target=/root/.local/share/pnpm/store \
  pnpm install --frozen-lockfile

##### BUILDER

FROM base AS builder

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN pnpm build

##### RUNNER

FROM base as runner

WORKDIR /tmp

RUN \
  if [ "$(uname -m)" = "x86_64" ] ; then \
  curl -L "https://github.com/CircuitQuest/mangal/releases/download/v5.11.4/mangal_5.11.4_linux_amd64.tar.gz" -o mangal.tar.gz ;\
  elif [ "$(uname -m)" = "i386" ] ; then \
  curl -L "https://github.com/CircuitQuest/mangal/releases/download/v5.11.4/mangal_5.11.4_linux_386.tar.gz" -o mangal.tar.gz ;\
  elif [ "$(uname -m)" = "aarch64" ] ; then \
  curl -L "https://github.com/CircuitQuest/mangal/releases/download/v5.11.4/mangal_5.11.4_linux_arm64.tar.gz" -o mangal.tar.gz ;\
  fi
RUN tar xf mangal.tar.gz
RUN mv mangal /usr/bin/mangal
RUN chmod +x /usr/bin/mangal

WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1
ENV HOME="/config"
ENV KAIZOKU_LOG_PATH="/logs"

COPY --from=builder /app/next.config.mjs ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

COPY docker/mangal /mangal
COPY docker/root/ /

VOLUME [ "/logs", "/config", "/data" ]
