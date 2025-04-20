# Stage 1: Builder - Production-ready build
FROM node:16-alpine AS builder
WORKDIR /usr/src/app

# 1. Install ONLY production dependencies (cached layer)
COPY package*.json ./
RUN npm ci --omit=dev --ignore-scripts

# 2. Copy source and build (separate layer)
COPY . .
RUN npm run build && \
    npm prune --production && \
    rm -rf /usr/src/app/.npm /usr/src/app/.cache

# Stage 2: Runtime - Minimal secure image
FROM alpine:3.16
WORKDIR /app

# Install runtime with security hardening
RUN apk add --no-cache \
    nodejs~=16 \
    tini=~0.19 && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup -h /app -D -s /bin/false && \
    chmod 750 /app

# Copy only necessary production artifacts
COPY --from=builder --chown=appuser:appgroup \
    /usr/src/app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup \
    /usr/src/app/package*.json ./
COPY --from=builder --chown=appuser:appgroup \
    /usr/src/app/build ./

# Production runtime configuration
USER appuser
ENV NODE_ENV=production \
    PORT=3000 \
    NODE_OPTIONS="--enable-source-maps --max-old-space-size=512"
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s \
    --start-period=10s \
    --retries=3 \
    CMD curl -f http://localhost:$PORT/health || exit 1
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]