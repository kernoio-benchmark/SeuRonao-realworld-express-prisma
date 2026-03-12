# Multi-stage Dockerfile for Node.js/TypeScript/Prisma application

# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Install OpenSSL for Prisma
RUN apk add --no-cache openssl

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev dependencies for build)
RUN npm ci

# Copy prisma schema
COPY prisma ./prisma/

# Generate Prisma Client
RUN npx prisma generate

# Copy source code and configuration
COPY tsconfig.json ./
COPY src ./src

# Build TypeScript application
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Install OpenSSL for Prisma
RUN apk add --no-cache openssl

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm ci --omit=dev && \
    npm cache clean --force

# Copy Prisma schema and generate client for production
COPY prisma ./prisma/
RUN npx prisma generate

# Copy compiled application from builder
COPY --from=builder /app/dist ./dist

# Create .env file placeholder (should be mounted or provided at runtime)
RUN touch .env

# Expose port (adjust if needed based on application configuration)
EXPOSE 3000

# Run the application
CMD ["node", "dist/server.js"]
