FROM ghcr.io/astral-sh/uv:0.9.2-python3.12-bookworm-slim AS base

WORKDIR /code

COPY pyproject.toml uv.lock ./

# ===== development =====
FROM base AS dev
RUN uv sync --frozen
COPY ./app /code/app
CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]

# ===== production =====
FROM base AS prod
RUN uv sync --frozen --no-dev
COPY ./app /code/app
ENV PATH="/code/.venv/bin:$PATH"
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80", "--workers", "4"]