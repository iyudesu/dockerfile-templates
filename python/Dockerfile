# ------------------- Stage 1: Build Stage ------------------------------
FROM public.ecr.aws/docker/library/python:3.14.3-slim AS builder

# Set the working directory to /app
WORKDIR /app

# Run on virtual env
RUN python -m venv /venv
ENV PATH="/venv/bin:$PATH"

# Copy the contents of the backend directory into the container at /app
COPY requirements.txt .

# Install dependencies specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Copy application source into builder
COPY app.py .

# ------------------- Stage 2: Final Stage ------------------------------

# Use a slim Python image as the final base image
FROM public.ecr.aws/docker/library/python:3.14.3-slim AS runner

# Create non-root user (CIS Docker Benchmark compliant)
ARG APP_USER=appuser
ARG APP_UID=10001
ARG APP_GID=10001

RUN groupadd --gid ${APP_GID} ${APP_USER} && \
    useradd --uid ${APP_UID} --gid ${APP_GID} \
    --create-home --shell /usr/sbin/nologin ${APP_USER}

# Set the working directory to /app
WORKDIR /app

# Copy the built dependencies from the virtual env in previous stage
COPY --from=builder /venv /venv
ENV PATH="/venv/bin:$PATH"

# Copy only the application file from builder
COPY --from=builder /app/app.py /app/app.py

# Create directory
RUN mkdir -p /tmp/ready && \
    # chown -R ${APP_UID}:${APP_GID} /tmp/ready # set owner
    chmod -R 440 /tmp/ready

USER ${APP_USER}

# Expose port 8080 for the Flask application
EXPOSE 8080

# Define the default command to run the application
CMD ["python", "app.py"]
