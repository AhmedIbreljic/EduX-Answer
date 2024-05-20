# Use multi-stage builds to separate building and runtime environments
FROM python:3.11.7-slim-bookworm as builder

# Set build-time variable
ARG DANSWER_VERSION=0.3-dev

# Install system dependencies and perform cleanup in the same layer
RUN apt-get update && apt-get install -y \
    cmake \
    curl \
    zip \
    ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies, remove unneeded 'py' package, and install Playwright
COPY ./requirements/default.txt /tmp/requirements.txt
RUN pip install --no-cache-dir --upgrade -r /tmp/requirements.txt && \
    pip uninstall -y py && \
    playwright install chromium && \
    playwright install-deps chromium

# Start a new stage from a smaller base image
FROM python:3.11.7-slim-bookworm

# Copy installed packages from builder stage
COPY --from=builder /usr/local /usr/local

# Set environment variables
ENV DANSWER_VERSION=${DANSWER_VERSION}
ENV PYTHONPATH /app

# Remove unnecessary packages and files to reduce image size
RUN apt-get remove -y --allow-remove-essential perl-base xserver-common xvfb cmake && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* && \
    rm /usr/local/lib/python3.11/site-packages/tornado/test/test.key

# Set up application files
WORKDIR /app
COPY ./danswer /app/danswer
COPY ./shared_models /app/shared_models
COPY ./alembic /app/alembic
COPY ./alembic.ini /app/alembic.ini
COPY supervisord.conf /usr/etc/supervisord.conf

# Default command to keep the container running
CMD ["tail", "-f", "/dev/null"]
