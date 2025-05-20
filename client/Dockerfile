# Stage 1: Builder - Production-ready build
FROM node:16-alpine AS builder
WORKDIR /usr/src/app

# 1. Install ALL dependencies (needed for build)
COPY package*.json ./
RUN npm ci

# 2. Copy source and build (separate layer)
COPY . .
RUN npm run build

# Stage 2: Runtime - Minimal secure image with serve
FROM alpine:3.16
WORKDIR /app

# Install runtime with security hardening
RUN apk add --no-cache \
    nodejs~=16 \
    npm~=8 \
    tini=~0.19 && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup -h /app -D -s /bin/false && \
    chmod 750 /app && \
    npm install -g serve

# Copy only the build output
COPY --from=builder --chown=appuser:appgroup \
    /usr/src/app/build ./build

# Production runtime configuration
USER appuser
ENV NODE_ENV=production \
    PORT=3000 \
    NODE_OPTIONS="--enable-source-maps --max-old-space-size=512"
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s \
    --start-period=10s \
    --retries=3 \
    CMD wget -q --spider http://localhost:$PORT || exit 1

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["serve", "-s", "build", "-l", "3000"]