ARG PHP_IMAGE
FROM $PHP_IMAGE

WORKDIR /var/www/html

COPY wait-for /usr/local/bin

ARG ADDITIONAL_EXTENSIONS
RUN rm $PHP_INI_DIR/php.ini-development \
  && mv $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini \
  && apk --no-cache add --virtual .build-deps $PHPIZE_DEPS \
  && pecl install redis \
  && docker-php-ext-enable redis \
  && docker-php-ext-install bcmath pdo_mysql $ADDITIONAL_EXTENSIONS \
  && apk del .build-deps
