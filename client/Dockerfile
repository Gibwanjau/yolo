# Stage 1: Build (Node.js)
FROM node:16-alpine AS builder
WORKDIR /usr/src/app

# Install production dependencies only (better caching)
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .

# Stage 2: Runtime (Minimal Alpine)
FROM alpine:3.16
WORKDIR /app

# Install only necessary runtime dependencies
RUN apk add --no-cache nodejs && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup -h /app -D && \
    chown -R appuser:appgroup /app

# Copy built application from builder
COPY --from=builder --chown=appuser:appgroup /usr/src/app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /usr/src/app/package*.json ./
COPY --from=builder --chown=appuser:appgroup /usr/src/app ./

# Runtime configuration
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "server.js"]