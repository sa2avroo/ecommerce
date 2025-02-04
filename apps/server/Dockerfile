# Base image with Node.js installed
FROM node:16-alpine AS base
LABEL authors="SHAKIL"

# Install additional dependencies if needed
RUN apk add --no-cache libc6-compat

# Set the working directory
WORKDIR /usr/src/app

# Install development dependencies
FROM base AS install
RUN mkdir -p /temp/dev
COPY . ./temp/dev/
WORKDIR /temp/dev
RUN npm ci --legacy-peer-deps

# Install production dependencies
FROM base AS install-prod
RUN mkdir -p /temp/prod
COPY . /temp/prod/
WORKDIR /temp/prod
RUN npm ci --only=production

# Copy node_modules from temp directory (for development)
FROM base AS prerelease
COPY --from=install /temp/dev/node_modules node_modules
COPY . .

# Optional: run tests and build (assuming you are using NX or a similar tool for your build process)
ENV NODE_ENV=production
RUN npx nx run server:build

# Final production image
FROM base AS release
COPY --from=install-prod /temp/prod/node_modules node_modules
COPY --from=prerelease /usr/src/app/apps/server/dist ./dist
COPY --from=prerelease /usr/src/app/apps/server/package.json ./

# Set environment variables
ENV NODE_ENV=production
EXPOSE 5500

# Start the app in production
CMD ["node", "apps/server/dist/app.js"]
