# Multi-stage Dockerfile for Node.js/TypeScript/Prisma application

# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy dependency manifests
COPY package.json package-lock.json* ./

# Install dependencies
RUN npm ci

# Copy source files
COPY tsconfig.json ./
COPY prisma ./prisma
COPY src ./src

# Generate Prisma Client
RUN npx prisma generate

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine

# Install OpenSSL for Prisma
RUN apk add --no-cache openssl

WORKDIR /app

# Set environment to production
ENV NODE_ENV=production

# Copy package manifests
COPY package.json package-lock.json* ./

# Copy node_modules from builder
COPY --from=builder /app/node_modules ./node_modules

# Copy Prisma schema
COPY prisma ./prisma

# Copy compiled application
COPY --from=builder /app/dist ./dist

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

# Run as non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "dist/server.js"]
