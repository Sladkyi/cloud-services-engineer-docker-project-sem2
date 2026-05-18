FROM node:18-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund

COPY . .

ARG VUE_APP_API_URL=/api
ENV VUE_APP_API_URL=${VUE_APP_API_URL}

RUN npx vue-cli-service build --public-path ''

FROM nginxinc/nginx-unprivileged:1.27-alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080
