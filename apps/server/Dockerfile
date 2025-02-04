# Base image with Node.js installed
FROM node:16-alpine AS base
LABEL authors="SHAKIL"

# Install additional dependencies if needed
RUN apk add --no-cache libc6-compat

# Set the working directory
WORKDIR /usr/src/app

# Copy root package.json and package-lock.json
COPY package.json package-lock.json ./

# Install root dependencies (includes shared libs)
RUN npm ci --legacy-peer-deps

# Install development dependencies for server app
FROM base AS install
RUN mkdir -p /temp/dev
COPY apps/server/package.json ./apps/server/package.json
COPY package-lock.json ./package-lock.json
WORKDIR /usr/src/app
RUN npm install --legacy-peer-deps

# Copy all source code into the container
COPY . .

# Optional: Run any build or tests (e.g., using nx or other tool)
RUN npx nx run server:build

# Copy the compiled/dist directory to the final image
FROM base AS release
COPY --from=install /usr/src/app/apps/server/dist ./dist
COPY --from=install /usr/src/app/node_modules ./node_modules
COPY --from=install /usr/src/app/apps/server/package.json ./apps/server/package.json

# Set environment variables
ENV NODE_ENV=production
EXPOSE 5500

# Start the app in production
CMD ["node", "apps/server/dist/app.js"]
