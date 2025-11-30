FROM php:8.2-cli AS builder

WORKDIR /app
COPY . .

FROM php:8.2-cli-alpine

RUN apk add --no-cache bash

RUN docker-php-ext-install pdo pdo_mysql

WORKDIR /app

COPY --from=builder /app /app

RUN chmod +x /app/wait-for-it.sh

CMD ["php", "-S", "0.0.0.0:8000"]
