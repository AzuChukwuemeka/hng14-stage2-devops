# Build stage
FROM node:18-alpine AS builder

WORKDIR /build

COPY package.json package-lock.json* ./

RUN npm ci --only=production

# Runtime stage
FROM node:18-alpine

WORKDIR /app

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser && \
    apk add --no-cache curl

# Copy node modules from builder
COPY --from=builder --chown=appuser:appuser /build/node_modules ./node_modules

# Copy application code
COPY --chown=appuser:appuser app.js .
COPY --chown=appuser:appuser views ./views

# Set environment variables
ENV NODE_ENV=production \
    API_URL=http://api:8000

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000 || exit 1

EXPOSE 3000

CMD ["node", "app.js"]
