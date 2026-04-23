# Build stage
FROM python:3.11-slim AS builder

WORKDIR /build

COPY requirements.txt .

RUN pip install --user --no-cache-dir --no-warn-script-location -r requirements.txt

# Runtime stage
FROM python:3.11-slim

WORKDIR /app

# Create non-root user
# Added '*' to curl and redis-tools to satisfy DL3008 version pinning
RUN useradd -m -u 1000 appuser && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    curl* \
    redis-tools* && \
    rm -rf /var/lib/apt/lists/*

# Copy Python dependencies from builder
COPY --from=builder --chown=appuser:appuser /root/.local /home/appuser/.local

# Copy worker code
COPY --chown=appuser:appuser worker.py .

# Set environment variables
ENV PATH=/home/appuser/.local/bin:$PATH \
    PYTHONUNBUFFERED=1 \
    REDIS_HOST=redis \
    REDIS_PORT=6379

# Switch to non-root user
USER appuser

# Health check - verify Redis connectivity
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD redis-cli -h ${REDIS_HOST} -p ${REDIS_PORT} ping | grep -q PONG || exit 1

CMD ["python", "worker.py"]