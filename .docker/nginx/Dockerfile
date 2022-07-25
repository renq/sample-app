ARG APP_IMAGE
FROM ${APP_IMAGE} as app_image

FROM nginx:1.21.1-alpine

COPY .docker/nginx/vhost.conf /etc/nginx/conf.d/default.conf
COPY --from=app_image /var/www/application/public /var/www/application/public
RUN rm /var/www/application/public/*.php
