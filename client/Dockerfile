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

# Install dependencies + SSH setup
RUN apk add --no-cache nodejs openssh && \
    adduser -D myuser -s /bin/sh -h /app && \
    mkdir -p /app/.ssh && \
    chown -R myuser:myuser /app && \
    chmod 700 /app/.ssh

# Copy pre-authorized key (build-time only)
COPY --chown=myuser:myuser ./docker/authorized_keys /app/.ssh/authorized_keys
RUN chmod 600 /app/.ssh/authorized_keys

# Copy application files
COPY --from=builder --chown=myuser:myuser /usr/src/app ./

USER myuser
EXPOSE 3000 22
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:5000/health || exit 1

# Start both SSH and Node
CMD ["sh", "-c", "sudo /usr/sbin/sshd -D & node server.js"]