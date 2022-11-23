# Colocando em prática

## Aula 1 - Instalando framework em um container

  - [Dockerfile Laravel](https://github.com/marciafc/fc-devops-docker/blob/main/laravel/Dockerfile)

## Aula 2 - Ativando entrypoint e command

  - [Dockerfile Laravel](https://github.com/marciafc/fc-devops-docker/blob/main/laravel/Dockerfile)

```
$ git clone https://github.com/marciafc/fc-devops-docker.git
$ cd laravel/

================

// Imagem base
FROM php:7.4-cli

// Setar diretório
WORKDIR /var/www

// Instalações necessárias - ideal é fazer isso baseado em contexto (por causa das layers)
RUN apt-get update && \
    apt-get install libzip-dev -y && \
    docker-php-ext-install zip

// Instalações necessárias - agora é o composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');"

// Instalações necessárias - agora é o Laravel
RUN php composer.phar create-project --prefer-dist laravel/laravel laravel

// Subir servidor web pelo Laravel (em modo desenvolvimento)
ENTRYPOINT [ "php","laravel/artisan","serve" ]

// O servidor de desenvolvimento Laravel está liberado para rodar apenas no 127.0.0.1:8000 do container
// Como está acessando de fora (nao está acessando 127.0.0.1), não vai rolar.
// É necessário liberar 127.0.0.1 do container para qlq host
// # php artisan serve --help
CMD [ "--host=0.0.0.0" ]

================

// Gerar imagem
$ docker build -t marciafcinfo/laravel:latest .

// Criar container
$ docker run --rm -d --name laravel -p 8000:8000 marciafcinfo/laravel

// Visualizar log
$ docker logs laravel

Starting Laravel development server: http://0.0.0.0:8000

================

// Navegador 
localhost:8000


// Alterando a porta, usando o parâmetro do CMD
$ docker run --rm -d --name laravel -p 8001:8001 marciafcinfo/laravel --host=0.0.0.0 --port=8001

$ docker logs laravel

Starting Laravel development server: http://0.0.0.0:8001

// Navegador 
localhost:8001

$ docker login
$ docker push marciafcinfo/laravel

```

## Aula 3 - Criando aplicação Node.js sem o Node

  - Desenvolver na máquina sem a necessidade de ter o node instalado: ao alterar na maq, altera no container  
  
  - Compartilhar a pasta do computador com o container rodando
  
    - altera no computador e vemos o resultado no container

```
// Criar diretório onde vai ficar a app
$ cd
$ mkdir myapp
$ cd myapp/

// Criando container com nodejs, mapeando do host Docker para o container 
// O que for criado no container, será exibido na máq e vice-versa
$ docker run --rm -it -v $(pwd)/:/usr/src/app -p 3000:3000 node:15 bash

# ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var

# cd /usr/src/app

// Criando app, instalando express no container
# npm init
# npm install express --save


// Na raiz desse projeto, no host Docker, criar arquivo index.js:

const express = require('express')
const app = express()
const port = 3000

app.get('/', (req,res) => {
    res.send('<h1>Full Cycle 3.0</h1>')
})

app.listen(port, ()=> {
    console.log('Rodando na porta ' + port)
})      

// Executar a app no container
# node index.js

// Abrir no navegador
http://localhost:3000

```

## Aula 4 - Gerando imagem da aplicação Node.js

  - Criar imagem da aplicação no DockerHub
  
1. Criar Dockerfile no diretório onde está a app da "Aula 3 - Criando aplicação Node.js sem o Node"

```
FROM node:15

WORKDIR /usr/src/app

COPY . .

EXPOSE 3000

CMD ["node","index.js"]

```

2. Criar imagem e container

  - ANTES: encerrar container rodando na porta 3000 da aula 3 antes de rodar este passo

```
$ docker build -t marciafcinfo/hello-express .
$ docker run -p 3000:3000 marciafcinfo/hello-express:latest

localhost:3000/

```

3. Subir imagem para o Docker Hub

```
$ docker push marciafcinfo/hello-express
```

  - Podemos ter um Dockerfile por ambiente ('dev' ou 'prod')
  
=== Dockerfile ===

```
FROM node:15

WORKDIR /usr/src/app

EXPOSE 3000

CMD ["node","index.js"]

```

=== Dockerfile.prod ===

```
FROM node:15

WORKDIR /usr/src/app

COPY . .

EXPOSE 3000

CMD ["node","index.js"]

```
  - Como build o Dockerfile.prod?
  
    - Exemplo qdo tem diretório: ``` docker build -t marciafcinfo/hello-express node/. -f node/Dockerfile.prod ``` 
  
    - ``` docker build -t marciafcinfo/hello-express . -f Dockerfile.prod ``` 
