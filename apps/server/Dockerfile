FROM node:16-alpine AS base
LABEL authors="SHAKIL"


# Install additional dependencies if needed
RUN apk add --no-cache libc6-compat

WORKDIR /usr/src/app

# Install dependencies into a temp directory to cache them
FROM base AS install

RUN mkdir -p /temp/dev
COPY package.json package-lock.json ./temp/dev/
COPY apps/server/package.json ./temp/dev/apps/server/
RUN cd /temp/dev && npm ci --legacy-peer-deps
 
# Install production dependencies only
RUN mkdir -p /temp/prod
COPY package.json package-lock.json /temp/prod/
COPY apps/server/package.json /temp/prod/apps/server/
RUN cd /temp/prod && npm ci --only=production

# Copy node_modules from temp directory
FROM base AS prerelease
COPY --from=install /temp/dev/node_modules node_modules
COPY . .

# Optional: tests & build
ENV NODE_ENV=production
RUN npx nx run server:build

# Production image
FROM base AS release
COPY --from=install /temp/prod/node_modules node_modules
COPY --from=prerelease /usr/src/app/apps/server/dist ./dist
COPY --from=prerelease /usr/src/app/apps/server/package.json .

# Set environment variables
ENV NODE_ENV=production
EXPOSE 5500/tcp

# run
CMD ["npm", "run", "apps/server/app.js"]
