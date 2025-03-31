FROM node:20-alpine AS base

RUN apk update && apk add --no-cache libc6-compat
RUN corepack

RUN corepack prepare pnpm@10.4.1 --activate

WORKDIR /app

FROM base AS builder

COPY . .

RUN npx --yes turbo@2.1.2 prune --scope=storefront --docker

FROM base AS installer

COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=builder /app/out/pnpm-workspace.yaml ./pnpm-workspace.yaml

RUN pnpm install --frozen-lockfile

COPY --from=builder /app/out/full/ .

RUN pnpm turbo build --filter=storefront && \
    pnpm install --prod --frozen-lockfile && \
    rm -rf node_modules/.cache

FROM base AS runner

COPY --chown=node:node --from=installer /app/ .

USER 1000

WORKDIR /app/apps/storefront

ENV PORT=80

CMD ["pnpm", "start"]