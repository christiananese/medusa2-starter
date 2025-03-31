FROM public.ecr.aws/docker/library/node:20.9.0-alpine AS base
RUN corepack enable
RUN apk add --no-cache libc6-compat
RUN corepack prepare pnpm@10.4.1 --activate

WORKDIR /app
# Copy root workspace files
COPY pnpm-lock.yaml ./
COPY pnpm-workspace.yaml ./
COPY package.json ./
# Copy all workspace files for dependency resolution
COPY . .

FROM base AS build
WORKDIR /app
RUN pnpm install --frozen-lockfile
RUN pnpm build

FROM public.ecr.aws/docker/library/node:20.9.0-slim AS final
RUN apt-get update && apt-get install -y curl procps && apt-get clean

# Setup pnpm in the final stage
RUN corepack enable
RUN corepack prepare pnpm@10.4.1 --activate

ARG PROJECT_PATH
WORKDIR /final/.medusa/server

# Copy necessary files for dependency installation
COPY --from=build /app/pnpm-lock.yaml /app/pnpm-workspace.yaml /app/package.json ./
COPY --from=build /app/${PROJECT_PATH}/package.json ./package.json
COPY --from=build /app/${PROJECT_PATH}/.medusa/server .

# Install production dependencies
RUN pnpm install --prod --frozen-lockfile


EXPOSE 9000

ENTRYPOINT [ "/bin/bash", "-c", "npx medusa db:migrate --execute-all-links && if [[ ! -z ${USER_INITIAL_PASSWORD} ]]; then npx medusa user -e \"$USER_INITIAL_EMAIL\" -p \"$USER_INITIAL_PASSWORD\" || true; fi && node ./node_modules/.bin/medusa start"]

