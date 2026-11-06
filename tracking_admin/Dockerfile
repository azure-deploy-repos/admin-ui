
# =====================================================
# Build Arguments
# =====================================================

ARG NODE_VERSION=20
ARG APP_NAME=booktrack-frontend
ARG APP_VERSION=1.0.0

# =====================================================
# Stage 1: Build React Application
# =====================================================

FROM node:${NODE_VERSION}-alpine AS builder

ARG APP_NAME
ARG APP_VERSION

# Vite environment variables (public build-time values)
ARG VITE_AUTH_API_URL
ARG VITE_PRODUCT_API_URL
ARG VITE_ORDER_API_URL
ARG VITE_TRACKING_API_URL
ARG VITE_USER_API_URL

ENV VITE_AUTH_API_URL=${VITE_AUTH_API_URL} \
    VITE_PRODUCT_API_URL=${VITE_PRODUCT_API_URL} \
    VITE_ORDER_API_URL=${VITE_ORDER_API_URL} \
    VITE_TRACKING_API_URL=${VITE_TRACKING_API_URL} \
    VITE_USER_API_URL=${VITE_USER_API_URL}

WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy application
COPY . .

# Build production application
RUN npm run build

# =====================================================
# Stage 2: Production Nginx Runtime
# =====================================================

FROM nginxinc/nginx-unprivileged:stable-alpine AS runtime

ARG APP_NAME
ARG APP_VERSION

LABEL application="${APP_NAME}" \
      version="${APP_VERSION}"

# Copy production build
COPY --from=builder --chown=101:101 /app/dist /usr/share/nginx/html

# Copy Nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# =====================================================
# Runtime Security
# =====================================================

USER 101:101

# Nginx unprivileged port
EXPOSE 8080

# =====================================================
# Health Check
# =====================================================

HEALTHCHECK --interval=30s \
    --timeout=5s \
    --start-period=15s \
    --retries=3 \
    CMD ["wget", "-q", "-O", "/dev/null", "http://127.0.0.1:8080/"]

# =====================================================
# Start Nginx Without Shell
# =====================================================

CMD ["nginx", "-g", "daemon off;"]
