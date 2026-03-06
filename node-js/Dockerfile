# # Install dependencies only when needed
# FROM public.ecr.aws/docker/library/node:23.11.0-alpine3.21 AS builder

# # Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
# RUN apk add --no-cache libc6-compat

# WORKDIR /app

# # Copy config before dependency installation
# COPY package*.json ./

# # Install dependencies
# RUN npm ci

# # Copy the rest of the application files
# COPY . .

# # Second image, copy all the files and run next
# FROM public.ecr.aws/docker/library/node:23.11.0-alpine3.21 AS runner

# # Set the working directory
# WORKDIR /app

# # Update libraries
# RUN apk update

# # Copy config
# COPY --from=builder /app ./

# # Create directory
# RUN mkdir /tmp/ready

# # Expose port 8080 for the application
# EXPOSE 8080

# # Command to run the application
# ENTRYPOINT ["node","app.js"]

############################
# 1️⃣ Builder Stage
############################
FROM public.ecr.aws/docker/library/node:23.11.0-alpine3.21 AS builder

# Set build arguments (can be overridden at build time)
ARG NODE_ENV=production
ARG VERSION=1.0.0
ARG COMMIT=unknown
ARG APP_HOME=/app

# Environment variables
ENV NODE_ENV=${NODE_ENV}

# Install required packages
RUN apk add --no-cache libc6-compat

WORKDIR ${APP_HOME}

# Copy only package files first (better layer caching)
COPY package*.json ./

# Install only production dependencies
RUN npm ci --omit=dev && \
    npm cache clean --force

# Copy application source
COPY . .

############################
# 2️⃣ Runtime Stage
############################
FROM public.ecr.aws/docker/library/node:23.11.0-alpine3.21 AS runner

# Set runtime argument variables
ARG NODE_ENV=production 
ARG PORT=8080 
ARG APP_HOME=/app

# Create non-root user and group
RUN addgroup nodegroup && \
    adduser -D nodeuser -G nodegroup

# Set working directory
WORKDIR ${APP_HOME}

# Copy built app from builder
COPY --from=builder /app ./

# Remove unnecessary files
RUN rm -rf \
    /root/.npm \
    /tmp/* \
    /var/cache/apk/*

# Create required directory with correct permissions
RUN mkdir -p /tmp/ready \
    # chown -R nodeuser:nodegroup /tmp/ready # Set owner
    && chmod -R 440 /tmp/ready

# Switch to non-root user
USER nodeuser

# Expose application port
EXPOSE ${PORT}

# Use CMD instead of ENTRYPOINT for better override flexibility
CMD ["node", "app.js"]