# syntax=docker/dockerfile:1

ARG PYTHON_VERSION=3.12.3
FROM python:${PYTHON_VERSION}-slim AS base

ENV PYTHONUNBUFFERED=ENV
ENV PYTHONDONTWRITEBYTECODE=1

COPY . /app

WORKDIR /app

RUN pip install --no-cache-dir -e .

EXPOSE 5000

CMD ["python", "run.py"]
