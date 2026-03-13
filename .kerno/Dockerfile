# Multi-stage Dockerfile for NestJS Prisma Application

# Stage 1: Build
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including devDependencies for build)
# Using --legacy-peer-deps to handle peer dependency conflicts
RUN npm install --legacy-peer-deps

# Copy prisma schema
COPY prisma ./prisma

# Generate Prisma Client
RUN npx prisma generate

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Stage 2: Production
FROM node:18-alpine AS production

# Set working directory
WORKDIR /app

# Set NODE_ENV to production
ENV NODE_ENV=production

# Copy package files
COPY package*.json ./

# Install only production dependencies
# Using --legacy-peer-deps to handle peer dependency conflicts
RUN npm install --only=production --legacy-peer-deps && \
    npm cache clean --force

# Copy prisma schema
COPY prisma ./prisma

# Copy generated Prisma Client from builder stage
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma
COPY --from=builder /app/node_modules/@prisma/client ./node_modules/@prisma/client

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Copy environment file template (should be overridden at runtime)
COPY .env.prod* ./

# Expose application port (default 3000, can be overridden)
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD node -e "require('http').get('http://localhost:${APP_PORT:-3000}/api', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Run migrations and start the application
CMD ["sh", "-c", "npx prisma migrate deploy && npm run start:prod"]
