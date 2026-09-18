# syntax=docker/dockerfile:1
# --- Build stage: install ALL deps (build tools live in devDependencies) ---
FROM node:20-bookworm-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm pkg delete scripts.prepare scripts.postinstall >/dev/null 2>&1 || true && \
    if [ -f package-lock.json ]; then npm ci || npm ci --legacy-peer-deps; else npm install; fi
COPY . .
RUN npm run postinstall --if-present
# Vite inlines this into the bundle at BUILD time — CI passes the real value.
ARG VITE_API_URL=""
ENV VITE_API_URL=${VITE_API_URL}
RUN npm run build

# --- Runtime: unprivileged nginx (non-root uid 101, listens on 8080) ---
FROM nginxinc/nginx-unprivileged:1.27-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -q --spider http://localhost:8080/healthz || exit 1
CMD ["nginx", "-g", "daemon off;"]
