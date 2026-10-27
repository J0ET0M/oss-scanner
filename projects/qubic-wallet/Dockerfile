# syntax=docker/dockerfile:1
# Build the official Qubic web wallet (Angular 16 PWA).
#
# The project uses Yarn 1 (classic; `packageManager: yarn@1.22.22`) and builds with the
# Angular CLI (`yarn build` -> `ng build`). Dependencies resolve from the public npm
# registry, which is reachable during this build; the audit stage afterwards has no
# network. The `prebuild` step (scripts/update-version.js) only rewrites a version string
# in the env files from package.json — no git or network needed.
#
# Angular 16 requires Node 16.14+ or 18.10+, so we pin Node 18.
FROM node:18-bookworm

ENV CI=1 \
    npm_config_fund=false \
    npm_config_audit=false \
    YARN_ENABLE_TELEMETRY=0

RUN apt-get update && apt-get install -y --no-install-recommends \
      bash ca-certificates git \
    && rm -rf /var/lib/apt/lists/*

# scanner contract: the checkout lives inside the image, at /src
COPY . /src
WORKDIR /src

# Install exactly what yarn.lock pins, then produce the Angular build.
# (ng build defaults to the production configuration in Angular 16.)
RUN yarn install --frozen-lockfile --non-interactive --network-timeout 600000 \
    && yarn build

CMD ["bash"]
