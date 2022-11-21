# Trabalhando com imagens

## Aula 1 - Entendendo imagens e DockerHub

  - [Docker Hub](https://hub.docker.com/)
  
    - Container registry do Docker
	
	- [Documentation DockerHub](https://docs.docker.com/docker-hub/)

```
// Listas as imagens
$ docker images


// Baixar imagem do php
$ docker pull php


// Remover uma imagem
$ docker rmi php:latest

```

## Aula 2 - Criando primeira imagem com Dockerfile
 
  - docker build
  
  - RUN
  
```
// Rodar esse comando na pasta aula2, onde tem um Dockerfile
// Cria a imagem com nome 'marciafcinfo/nginx-com-vim' e tag 'latest'
$ docker build -t marciafcinfo/nginx-com-vim:latest .


// Rodando e acessando o container (imagem nginx com o vim instalado)
$ docker run -it marciafcinfo/nginx-com-vim bash
/# vim oi

```

## Aula 3 - Avançando com Dockerfile
 
  - WORKDIR
  
  - COPY
  
```
// Rodar esse comando na pasta aula3, onde tem um Dockerfile
$ docker build -t marciafcinfo/nginx-com-vim:latest .


// Ao executar, já está dentro da pasta app
$ docker run -it marciafcinfo/nginx-com-vim bash

/app# cd /usr/share/nginx/html/
/usr/share/nginx/html# ls

50x.html  index.html


/usr/share/nginx/html# cat index.html 

<h1>Full Cycle</h1>


```

## Aula 4 - ENTRYPOINT vs CMD
 
  - CMD: o comando é substituído pelo que é passado por linha de comando
  
  - Listar id dos containers ativos e inativos
  
  - Remover containers ativos e inativos
  
```
// Rodar esse comando na pasta aula4, onde tem um Dockerfile
$ docker build -t marciafcinfo/hello .

// Exibe 'Hello World'
$ docker run --rm marciafcinfo/hello:latest 
Hello World

// Listar id dos containers ativos e inativos
$ docker ps -a -q

// Remover containers ativos e inativos
$ docker rm $(docker ps -a -q) -f

// Ignora o 'Hello World' do Dockerfile e exibe 'oi'
$ docker run --rm marciafcinfo/hello echo "oi"
oi

// Ignora o 'Hello World' do Dockerfile e executa o bash
$ docker run --rm marciafcinfo/hello bash

```

  - ENTRYPOINT: é sempre FIXO (é um comando fixo), já o CMD é um comando VARIÁVEL (entra como parâmetro do ENTRYPOINT)
  
    - ENTRYPOINT ["echo", "Hello "]

    - CMD ["World"]  

```
// Rodar esse comando na pasta aula4a, onde tem um Dockerfile
$ docker build -t marciafcinfo/hello .


// Exibe o 'Hello ' do ENTRYPOINT e o 'World' do CMD
$ docker run --rm marciafcinfo/hello
Hello  World


// Exibe o 'Hello ' do ENTRYPOINT e o valor informado na linha de comando no lugar do valor default do CMD
// Qdo não informa nada na linha de comando (sem parâmetros), usa o default do CMD no Dockerfile
$ docker run --rm marciafcinfo/hello Marcia
Hello  Marcia

```

## Aula 5 - Docker entrypoint exec

  - Para conhecer o entrypoint da imagem, clicar na tag da imagem no Docker Hub
  
    - Exemplo: [https://hub.docker.com/_/nginx](https://hub.docker.com/_/nginx) e escolher "1.23.2, mainline, 1, 1.23, latest"
	
	- [Dockerfile](https://github.com/nginxinc/docker-nginx/blob/fef51235521d1cdf8b05d8cb1378a526d2abf421/mainline/debian/Dockerfile)
	  
	  - ENTRYPOINT ["/docker-entrypoint.sh"]

      - CMD ["nginx", "-g", "daemon off;"]

  - ENV: variáveis de ambiente

```
// Quando roda a imagem do nginx, o ENTRYPOINT será executado e o CMD (comando variável) será substituído pelo bash
$ docker run --rm nginx bash

// Verificando se o arquivo do ENTRYPOINT, o 'docker-entrypoint.sh' existe
$ docker run --rm -it nginx bash
/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

// O que tem por padrão no 'docker-entrypoint.sh'?
/# cat docker-entrypoint.sh

#!/bin/sh
# vim:sw=4:ts=4:et

set -e
...
...
...
exec "$@"


Na última linha do arquivo 'docker-entrypoint.sh' tem 'exec "$@"' -> ele executa o que vem como argumento por linha de comando,
substituindo o que está no CMD:
/# ./docker-entrypoint.sh echo 'Hello'
Hello

```

## Aula 6 - Publicando imagem no DockerHub

```
// Criar Dockerfile com esse conteúdo
// https://github.com/marciafc/fc-devops-docker/blob/main/nginx/Dockerfile
// Executar na pasta 'aula6' -> Dockerfile

// Criando imagem
$ docker build -t marciafcinfo/nginx-fullcycle .

// Executar imagem
$ docker run --rm -d -p 8081:80 marciafcinfo/nginx-fullcycle

// Navegador
http://localhost:8081/


$ docker images

REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
marciafcinfo/nginx-fullcycle                latest              b800c22f2406        2 minutes ago       142MB

// login
$ docker login

// Subindo imagem para o DockerHub
$ docker push marciafcinfo/nginx-fullcycle

// Esta imagem no DockerHub
https://hub.docker.com/repository/docker/marciafcinfo/nginx-fullcycle

// Logout
$ docker logout

```
