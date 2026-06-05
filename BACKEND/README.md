# Ferramentas Dockerizadas para Backend
Esta sessão fornecerá explicação e containers prontos para utilização das ferramentas: Java, Php e Node.

## Java
O Java é uma linguagem de programação multiplataforma e orientada a objetos que opera sob o famoso princípio "escreva uma vez, rode em qualquer lugar" (write once, run anywhere). Por conta de sua alta segurança e escalabilidade, ela é amplamente utilizada no desenvolvimento de sistemas corporativos robustos, aplicativos Android e APIs de grande porte no backend.

```bash
FROM docker.io/library/eclipse-temurin:25-jdk-alpine AS builder
WORKDIR /build
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN chmod +x mvnw
RUN ./mvnw dependency:go-offline
COPY src ./src
RUN ./mvnw clean package -DskipTests

FROM docker.io/library/eclipse-temurin:25-jre-alpine
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Php
O PHP é uma das linguagens de script mais utilizadas no mundo para o desenvolvimento web, sendo executada do lado do servidor para gerar conteúdo dinâmico em sites e aplicações. Sua grande popularidade se deve à facilidade de integração com diversos bancos de dados e à robustez com que sustenta desde pequenos blogs até grandes plataformas globais.

```bash
FROM php:8.2-cli-alpine AS builder
WORKDIR /var/www
RUN apk --no-cache git unzip libzip-dev icu-dev libpng-dev
RUN docker-php-ext-install zip intl gd
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-interaction --no-plugins --no-scripts
COPY . .

FROM php:8.2-fpm-alpine AS production
WORKDIR /var/www
RUN apk add --no-cache icu-dev libzip-dev libpng-dev && docker-php-ext-install intl zip pdo_mysql gd && opcache
COPY docker/php/opcache.ini /usr/local/etc/php/conf.d/opcache.ini
COPY --from=builder --chown=www-data:www-data /var/www /var/www
USER www-data
EXPOSE 9000
CMD ["php-fpm]
```

## Node
O Node.js é um ambiente de execução JavaScript assíncrono e orientado a eventos, projetado para criar aplicações de rede escaláveis e de alta performance. Baseado no motor V8 do Google Chrome, ele permite que desenvolvedores utilizem a mesma linguagem tanto no front-end quanto no back-end de forma eficiente.

```bash
FROM node:20-alpine AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build
RUN npm prune --production && npm cache clean --force

FROM node:20-alpine AS runner
ENV NODE_ENV=production
USER node
COPY --from=builder --chown=node:node /app/package*.json ./
COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --from=builder --chown=node:node /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/index.js"]
```