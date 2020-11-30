# Сборка multistage для storefront
FROM node:10 as storefront-builder
WORKDIR /app
COPY ./saleor-storefront/package.json ./
COPY ./saleor-storefront/package-lock.json ./
RUN npm install
COPY ./saleor-storefront/ .
ARG API_URI
ARG SENTRY_DSN
ARG SENTRY_APM
ARG DEMO_MODE
ARG GTM_ID
ENV API_URI ${API_URI}
RUN API_URI=${API_URI} npm run build

# Сборка multistage для dashboard
FROM node:10 as dashboard-builder
WORKDIR /app
COPY ./saleor-dashboard/package.json ./
COPY ./saleor-dashboard/package-lock.json ./
RUN npm install
COPY ./saleor-dashboard/ .
ARG APP_MOUNT_URI
ARG API_URI
ARG STATIC_URL
ENV API_URI ${API_URI}
ENV APP_MOUNT_URI ${APP_MOUNT_URI:-/dashboard/}
ENV STATIC_URL ${STATIC_URL:-/dashboard/}
RUN STATIC_URL=${STATIC_URL} API_URI=${API_URI} APP_MOUNT_URI=${APP_MOUNT_URI} npm run build


FROM nginx:stable
WORKDIR /app

RUN rm /etc/nginx/conf.d/default.conf
RUN rm /etc/nginx/nginx.conf

# Копирование конфигураций
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./nginx/saleor.conf /etc/nginx/saleor.conf

# Копирование сгенерированной статики из storefront-builder и dashboard-builder
COPY --from=storefront-builder /app/dist/ /app/
COPY --from=dashboard-builder /app/build/ /app/

# Выставляем права для nginx
RUN chown -R www-data:www-data /app/*
RUN chmod -R 0755 /app/*

EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
