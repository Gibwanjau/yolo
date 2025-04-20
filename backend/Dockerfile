# Stage 1: Builder - Install and build
FROM node:16-alpine AS builder
WORKDIR /usr/src/app

# 1. Install dependencies (layer cached separately)
COPY package*.json ./
RUN npm ci --omit=dev

# 2. Copy source and build (separate layer)
COPY . .
RUN npm run build && \
    npm prune --production  # Remove dev dependencies

# Stage 2: Runtime - Minimal production image
FROM alpine:3.16
WORKDIR /app

# Install only necessary runtime deps
RUN apk add --no-cache nodejs && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup -h /app -D && \
    chmod 750 /app

# Copy only production artifacts
COPY --from=builder --chown=appuser:appgroup \
    /usr/src/app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup \
    /usr/src/app/package*.json ./
COPY --from=builder --chown=appuser:appgroup \
    /usr/src/app/build ./

# Runtime config
USER appuser
ENV NODE_ENV=production \
    PORT=5000
EXPOSE 5000
HEALTHCHECK --interval=30s --timeout=5s \
    CMD curl -f http://localhost:$PORT/health || exit 1
CMD ["node", "server.js"]