# base defines a base stage that uses the official python runtime base image
FROM python:3.11-slim@sha256:b7a9e5f6a93d6d4a31f129bbd4bdee69efccb6ded0fb230d78aafaf37af98377 AS base

# Add curl for healthcheck
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY requirements.txt ./requirements.txt

RUN pip install --no-cache-dir --upgrade \
    "jaraco.context>=6.1.0" \
    "wheel>=0.46.2"

RUN pip install --no-cache-dir -r requirements.txt
# dev defines a stage for development, where it'll watch for filesystem changes
FROM base AS dev
RUN pip install watchdog

RUN useradd --system --create-home --shell /usr/sbin/nologin voteuser
USER voteuser

ENV FLASK_ENV=development
CMD ["python", "app.py"]

# final defines the stage that will bundle the application for production
FROM base AS final

COPY . .

EXPOSE 80

RUN useradd --system --create-home --shell /usr/sbin/nologin voteuser
USER voteuser

CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
