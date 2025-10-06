# Stage 1: build with node
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files first to leverage layer cache
COPY package*.json ./
RUN npm ci

COPY . .
# Build Vite production assets (assumes output dir is dist)
RUN npm run build

# Stage 2: serve static with nginx
FROM nginx:1.27-alpine AS runtime
# Clear default content
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder /app/dist /usr/share/nginx/html
# Replace default config so SPA routing works
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
