# Stage 1: Build (Node.js)
FROM node:16-alpine AS builder
WORKDIR /usr/src/app

# Install dependencies separately for better caching
COPY package*.json ./
RUN npm ci --omit=dev  
COPY . .

# Stage 2: Runtime (Minimal Alpine)
FROM alpine:3.16
WORKDIR /app

# Install only necessary runtime dependencies
RUN apk add --no-cache nodejs && \
    adduser -D myuser && \
    chown -R myuser:myuser /app

# Copy built files from builder
COPY --from=builder --chown=myuser:myuser /usr/src/app ./

# Runtime configuration
USER myuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s \
    CMD curl -f http://localhost:3000 || exit 1
CMD ["npm", "start"]