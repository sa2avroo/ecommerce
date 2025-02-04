FROM node:16-alpine AS base
LABEL authors="SHAKIL"

WORKDIR /usr/src/app

# Install dependencies into a temp directory to cache them
FROM base AS install

RUN mkdir -p /temp/dev
COPY package.json package-lock.json ./temp/dev/
COPY apps/server/package.json ./temp/dev/apps/server/package.json
RUN cd /temp/dev && npm install

# Install production dependencies only
RUN mkdir -p /temp/prod
COPY package.json package-lock.json /temp/prod/
COPY apps/server/package.json /temp/prod/apps/server/
RUN cd /temp/prod && npm install --production

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

# Change user to bun which is assumed to have necessary permissions
USER bun
# run
CMD ["bun", "run", "apps/server/app.js"]
