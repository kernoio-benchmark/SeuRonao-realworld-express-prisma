# Build stage
FROM node:18.19.0-bookworm-slim AS build

# Install OpenSSL for Prisma
RUN apt-get update && apt-get install -y --no-install-recommends openssl && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy dependency manifests first for better caching
COPY package*.json ./

# Install all dependencies (including devDependencies for build)
RUN npm ci

# Copy prisma schema
COPY prisma ./prisma

# Generate Prisma Client
RUN npx prisma generate

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Production stage
FROM node:18.19.0-bookworm-slim

# Install curl for healthcheck probes and OpenSSL for Prisma
RUN apt-get update && apt-get install -y --no-install-recommends curl openssl && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser -m appuser

WORKDIR /app

# Copy dependency manifests
COPY package*.json ./

# Install only production dependencies
RUN npm ci --omit=dev

# Copy prisma schema
COPY prisma ./prisma

# Generate Prisma Client for production
RUN npx prisma generate

# Copy built application from build stage
COPY --chown=appuser:appuser --from=build /app/dist ./dist

# Copy .env file if it exists (for production configuration)
COPY --chown=appuser:appuser .env* ./

# Change ownership of the app directory
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Expose port (default Express port)
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
