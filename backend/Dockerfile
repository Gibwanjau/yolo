# Stage 1: Builder - Install and build
# Changed to smaller package
FROM node:16-alpine AS builder
WORKDIR /usr/src/app

# Install production dependencies only
COPY package*.json ./
RUN npm ci --omit=dev

# Copy source and build
COPY . .
RUN npm run build

# Stage 2: Runtime - Minimal production image
FROM alpine:3.16
WORKDIR /app

# Install runtime dependencies
RUN apk add --no-cache nodejs && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup -h /app -D && \
    chown -R appuser:appgroup /app

# Copy built application from builder
COPY --from=builder --chown=appuser:appgroup /usr/src/app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /usr/src/app/package*.json ./
COPY --from=builder --chown=appuser:appgroup /usr/src/app/build ./

# Runtime configuration
USER appuser
ENV NODE_ENV=production
EXPOSE 5000
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:5000/health || exit 1
CMD ["node", "server.js"]