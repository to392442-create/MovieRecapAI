# Render bridge: upload this file renamed to Dockerfile alongside MovieRecapAI-source.zip
FROM node:22-bookworm-slim AS frontend
WORKDIR /build
RUN corepack enable && corepack prepare pnpm@10.18.0 --activate \
    && apt-get update \
    && apt-get install -y --no-install-recommends unzip \
    && rm -rf /var/lib/apt/lists/*
COPY MovieRecapAI-source.zip /tmp/MovieRecapAI-source.zip
RUN unzip -q /tmp/MovieRecapAI-source.zip -d /build \
    && cd /build/MovieRecapAI \
    && CI=1 pnpm install --frozen-lockfile \
    && pnpm build:static

FROM python:3.12-slim-bookworm
WORKDIR /app
RUN apt-get update \
    && apt-get install -y --no-install-recommends ffmpeg fonts-noto-core unzip \
    && rm -rf /var/lib/apt/lists/*
COPY MovieRecapAI-source.zip /tmp/MovieRecapAI-source.zip
RUN unzip -q /tmp/MovieRecapAI-source.zip -d /tmp/source \
    && cp -a /tmp/source/MovieRecapAI/backend /app/backend \
    && cp -a /tmp/source/MovieRecapAI/Dockerfile /app/source.Dockerfile \
    && rm -rf /tmp/source /tmp/MovieRecapAI-source.zip
RUN pip install --no-cache-dir -r /app/backend/requirements.txt
COPY --from=frontend /build/MovieRecapAI/dist/public /app/dist/public
ENV PYTHONUNBUFFERED=1
ENV APP_ENV=production
ENV PORT=3000
ENV MOVIERECAP_DATA_DIR=/app/data
EXPOSE 3000
CMD ["sh", "-c", "uvicorn backend.main:app --host 0.0.0.0 --port ${PORT}"]
