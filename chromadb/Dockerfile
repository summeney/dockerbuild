FROM python:3.11-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    CHROMA_DB_PATH=/data/chroma \
    CHROMA_SERVER_HOST=0.0.0.0 \
    CHROMA_SERVER_PORT=8000

WORKDIR /app

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends build-essential curl; \
    pip install --upgrade pip; \
    pip install "chromadb[server]==0.5.3"; \
    apt-get purge -y --auto-remove build-essential; \
    rm -rf /var/lib/apt/lists/*

VOLUME ["/data/chroma"]

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=15s --retries=3 \
    CMD curl -f http://127.0.0.1:${CHROMA_SERVER_PORT}/api/v1/heartbeat || exit 1

CMD chroma run --host ${CHROMA_SERVER_HOST} --port ${CHROMA_SERVER_PORT} --path ${CHROMA_DB_PATH}

