# Multi-stage Dockerfile for Node.js/TypeScript/Prisma application

# Stage 1: Build stage
FROM node:18-alpine AS builder

# Install build dependencies
RUN apk add --no-cache openssl

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev dependencies for build)
RUN npm ci

# Copy application source
COPY . .

# Generate Prisma Client
RUN npx prisma generate

# Build TypeScript application
RUN npm run build

# Stage 2: Production stage
FROM node:18-alpine AS production

# Install runtime dependencies
RUN apk add --no-cache openssl

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy Prisma schema for runtime
COPY prisma ./prisma

# Generate Prisma Client in production
RUN npx prisma generate

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Copy .env file if it exists (optional, can be mounted at runtime)
COPY .env* ./

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port (default for Express apps)
EXPOSE 3000

# Set NODE_ENV to production
ENV NODE_ENV=production

# Start the application
CMD ["node", "dist/server.js"]
