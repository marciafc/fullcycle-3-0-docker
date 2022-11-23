# Otimizando imagens


## Aula 1 - Otimização utilizando Multistage Building

  - Alpine linux menor
  
  - Multistage Building reduz tamanho da imagem
  
  - [Dockerfile.prod](https://github.com/marciafc/fc-devops-docker/blob/main/laravel/Dockerfile.prod)
  
```
FROM php:7.4-cli AS builder
WORKDIR /var/www
RUN apt-get update && \
    apt-get install libzip-dev -y && \
    docker-php-ext-install zip
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');"
RUN php composer.phar create-project --prefer-dist laravel/laravel laravel


// outro stage
FROM php:7.4-fpm-alpine
WORKDIR /var/www
RUN rm -rf /var/www/html

// Vai copiar o passo anterior (AS builder) que gerou insumos em /var/www/laravel para /var/www
// O ponto "." no final, é o WORKDIR, que é o /var/www
COPY --from=builder /var/www/laravel .

RUN ln -s public html
RUN chown -R www-data:www-data /var/www
EXPOSE 9000
CMD [ "php-fpm" ]
```

  - Criar imagem
  
    - Imagem com Multistage Building (prod) e SEM Multistage Building (normal)
  
```
// -f laravel/Dockerfile.prod -> informar -f qdo mudar o nome padrão do Dockerfile (Dockerfile.prod, no caso)
$ docker build -t marciafcinfo/laravel:prod   laravel -f laravel/Dockerfile.prod

$ docker build -t marciafcinfo/laravel:normal laravel -f laravel/Dockerfile

$ docker images | grep laravel

```
  - Diferença no tamanho da imagem de 553MB para 140MB

```
$ docker images | grep laravel

marciafcinfo/laravel         prod                8d5d3d36c878        7 minutes ago       140MB
marciafcinfo/laravel         normal              dd88ef478402        25 hours ago        553MB

```

## Aula 2 - Nginx como proxy reverso








