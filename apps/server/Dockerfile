# Stage 1: Build the TypeScript code
FROM node:16-alpine AS builder

# Install additional dependencies if needed
RUN apk add --no-cache libc6-compat

WORKDIR /app

COPY package*.json ./
COPY apps/server/package.json ./temp/dev/apps/server/

RUN npm ci --legacy-peer-deps

COPY tsconfig.json ./
COPY . .

RUN npx nx run server:build

FROM node:16-alpine

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=5500

# Create a non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 shakil

COPY --from=builder --chown=shakil:nodejs /app/package*.json ./
COPY --from=builder --chown=shakil:nodejs /app/apps/server/package*.json ./
RUN npm ci --only=production

COPY --from=builder --chown=shakil:nodejs /app/apps/server/dist ./dist

USER shakil
EXPOSE 5500

CMD ["node", "apps/server/dist/app.js"]
