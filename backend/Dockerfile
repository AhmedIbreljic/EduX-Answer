# Build stage for installing dependencies and building any necessary extensions
FROM python:3.11.7-slim-bookworm as builder

# Set environment variables to reduce bloat
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Default DANSWER_VERSION, typically overriden during builds by GitHub Actions.
ARG DANSWER_VERSION=0.3-dev
ENV DANSWER_VERSION=${DANSWER_VERSION}

# Install system dependencies
RUN apt-get update && apt-get install -y \
    cmake \
    curl \
    zip \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Install Python dependencies
COPY ./requirements/default.txt /tmp/requirements.txt
RUN pip install --no-cache-dir --upgrade -r /tmp/requirements.txt \
    && pip uninstall -y py \
    && playwright install chromium \
    && playwright install-deps chromium

# Final stage
FROM python:3.11.7-slim-bookworm

# Copy over only the necessary dependencies
COPY --from=builder /usr/local /usr/local

# Cleanup for CVEs and size reduction
RUN apt-get update && apt-get remove -y --allow-remove-essential \
    perl-base \
    xserver-common \
    xvfb \
    cmake \
    libldap-2.5-0 \
    libldap-2.5-0 \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -f /usr/local/lib/python3.11/site-packages/tornado/test/test.key

# Set up application files
WORKDIR /app
COPY ./danswer /app/danswer
COPY ./shared_models /app/shared_models
COPY ./alembic /app/alembic
COPY ./alembic.ini /app
COPY supervisord.conf /usr/etc/supervisord.conf

ENV PYTHONPATH /app

# Default command which does nothing
# This container is used by api server and background which specify their own CMD
CMD ["tail", "-f", "/dev/null"]
