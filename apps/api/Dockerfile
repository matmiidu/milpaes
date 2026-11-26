FROM python:3.12-slim AS base

RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:${PATH}"

WORKDIR /app

# Capa de dependencias por separado del código: cachea mientras no cambien
# pyproject.toml/uv.lock, que es la mayoría de las veces.
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project

COPY . .
RUN uv sync --frozen --no-dev

RUN useradd --create-home appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# Binarios del venv directo: uv quedó instalado en /root/.local/bin, que
# appuser no puede leer (Render cortaba con "Exited with status 127").
CMD ["sh", "-c", ".venv/bin/alembic upgrade head && .venv/bin/uvicorn paes_api.main:app --host 0.0.0.0 --port ${PORT:-8000}"]
