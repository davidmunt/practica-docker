## Servicio de MySQL

Requisitos de la practica:

- Utilitzarà la imatge docker oficial de MySQL
- El contenidor associat es denominarà mysql_contenidor
- Serà el primer servei a arrancar
- La primera tasca que farà res més arrancar serà crear les taules necessàries (script en la tasca).
- Has de crear un volum per a que no es perden les dades de la base de dades
- Tots els fitxers de configuració necessaris residiran en una carpeta en el projecte

Para implementarlo, añadí el siguiente bloque al archivo docker-compose.yml:

```yml
version: "3.9"

services:
  mysql:
    image: mysql:8.0
    container_name: mysql_contenidor
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: elementos
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
      - ./mysql-init:/docker-entrypoint-initdb.d

volumes:
  db_data:
```

## Servicio de backend

Requisitos:

- S'ha de construir una imatge (Dockerfile).
- No arrancarà fins que el servei de MySQL no estiga preparat completament (wait-for-it)
- El contenidor associat es denominarà backend_contenidor
- Executarà com a primer comando res més arrancar el comando necesari per posar en marcha el servidor

En la carpeta backend, en el archivo Dockerfile añado:

```Dockerfile
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
```

Y por ultimo en el archivo docker-compose.yml añado esto al final del todo:

```yml
backend:
  build: ./backend
  container_name: backend_contenidor
  depends_on:
    - mysql
  ports:
    - "8000:8000"
  command: ["./wait-for-it.sh", "mysql:3306", "--", "php", "-S", "0.0.0.0:8000"]
```

## Servicio de frontend

Requisitos:

- Arrancarà després del servei de backend
- El contenidor associat es denominarà frontend_contenidor
- Executarà com a primer comando res més arrancar: npm run serve

Para esto en la carpeta de frontend, en el archivo Dockerfile añado:

```Dockerfile
FROM node:18 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .

FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app /app

EXPOSE 8080

CMD ["npm", "run", "serve"]
```

Y por ultimo en el archivo docker-compose.yml añado esto al final del todo:

```yml
frontend:
  build: ./frontend
  container_name: frontend_contenidor
  depends_on:
    - backend
  ports:
    - "8080:8080"
  command: ["npm", "run", "serve"]
```

Requisitos:

- Arrancarà després del servei de backend
- El contenidor associat es denominarà frontend_contenidor
- Executarà com a primer comando res més arrancar: npm run serve

Añado este codigo en el docker-compose.yml:

```yml
phpmyadmin:
  image: phpmyadmin/phpmyadmin
  container_name: adminMySQL_contenidor
  depends_on:
    - mysql
  environment:
    PMA_HOST: mysql
    PMA_PORT: 3306
  ports:
    - "8081:80"
```

## Puesta en marcha del proyecto

Por ultimo, para construir las imágenes de los servicios:

```sh
docker compose build
```

Para iniciar todos los contenedores:

```sh
docker compose up
```

Una vez arranque todo el proyecto, los servicios estarán disponibles en:

Frontend:
[http://localhost:8080](http://localhost:8080)

Backend:
[http://localhost:8000](http://localhost:8000)

phpMyAdmin:
[http://localhost:8081](http://localhost:8081)
