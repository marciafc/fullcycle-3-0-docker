# Iniciando com Docker

## Aula 1

```
docker ps

docker run hello-world

docker ps -a

```

## Aula 2

```
docker run -it ubuntu bash

docker run -i -t ubuntu:latest bash

ls

uname -a

docker ps -a


CONTAINER ID        IMAGE                                             COMMAND                  CREATED             STATUS                         PORTS                      NAMES
6a6c0bbeda07        ubuntu:latest                                     "bash"                   18 minutes ago      Exited (0) 13 seconds ago                                 trusting_curran


docker start trusting_curran


// Qdo encerrar o Ubuntu (control D ou exit), automaticamente remove o container
docker run -it --rm  ubuntu:latest bash

```

## Aula 3 - Publicando portas com nginx

```
// web server -> nginx
docker run nginx

docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
cf04c91ab7c8        nginx               "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        80/tcp              flamboyant_yonath


// redirecionamento de porta
// 8080 da minha máquina para a 80 da nginx
docker run -p 8082:80 nginx

http://localhost:8082/

docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
ac0f22d0ad9b        nginx               "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        0.0.0.0:8082->80/tcp   nifty_kalam

// -d detached = desatachando o terminal do processo do nginx do processo do container
docker run -d -p 8082:80 nginx

```

## Aula 4 - Removendo containers

```
$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
9ffe5319d069        nginx               "/docker-entrypoint.…"   3 hours ago         Up 3 hours          0.0.0.0:81->80/tcp     elastic_shannon
0ffd3bd2d99c        nginx               "/docker-entrypoint.…"   3 hours ago         Up 3 hours          0.0.0.0:8082->80/tcp   elastic_sutherland

$ docker stop 9f

9f

$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
0ffd3bd2d99c        nginx               "/docker-entrypoint.…"   3 hours ago         Up 3 hours          0.0.0.0:8082->80/tcp   elastic_sutherland


$ docker start 9f

// Remover container
$ docker rm 324568629a06

// Remover container pelo nome do container
$ docker rm funny_snyder

// Remover container com force
$ docker rm elastic_shannon -f

```

## Aula 5 - Acessando e alterando arquivos de um container

- [exec command](https://docs.docker.com/engine/reference/commandline/exec/)

```
// --name  -> nomear container 
$ docker run -d --name nginx -p 8082:80 nginx

// Matando container pelo nome
$ docker rm nginx -f


$ docker run -d --name nginx -p 8082:80 nginx
http://localhost:8082/

// Acessar o container -> exec e o nome do container
$ docker exec nginx ls

// Executar o bash -> vai executar e sair
$ docker exec nginx bash


// Executar o bash de modo iterativo e com tty
$ docker exec -it nginx bash

// Acessar arquivos do nginx
root@b22dad2ac89d:/# cd /usr/share/nginx/html/

# ls
50x.html  index.html

// similar
http://localhost:8082/index.html
http://localhost:8082

// Baixar as imagens, todos os arquivos que pode instalar
# apt-get update

// Instalar editor vim
# apt-get install vim

// Editar index com o vim
# vim index.html

```

### Editar com o vim

  1. Modo de edição -> apertar tecla i

  2. Salvar -> esc para sair do modo de edição e digitar :w

  3. http://localhost:8082/

  4. Sair do vim  :q

  - Sair sem salvar :q!

  - Sair salvando  :wq


## Aula 6 - Iniciando com bind mounts

  - [bind mounts](https://docs.docker.com/storage/bind-mounts/)

  - Mapeia volume do computador para o container 
	
  - Se o container morrer, o arquivo é mantido no computador

```
$ cd Downloads/
$ mkdir html
$ cd html/
$ touch index.html

$ nano index.html
<h1>Full Cycle</h1>

$ cat index.html

===========

-v  --> para montar volume de uma pasta q está no computador p/ pasta no container
  ~/Downloads/html/       --> path do computador
  /usr/share/nginx/html/  --> path onde ficam os arquivos do nginx

===========
$ docker run -d --name nginx -p 8082:80 -v ~/Downloads/html/:/usr/share/nginx/html/ nginx

http://localhost:8082/

$ docker rm 01 -f

===========

// Diretório atual
$ echo $(pwd)

/home/marcia-castagna/Downloads/html

===========

// -v é um comando bem antigo, usar --mount, source é o computador e target o container
$ docker run -d --name nginx -p 8082:80 --mount type=bind,source="$(pwd)",target=/usr/share/nginx/html nginx

===========

Diferença de usar -v e --mount?

O mount é mais explícito

O -v caso não exista o diretório no computador, irá criar... o mount não cria

===========


```

## Aula 7 - Trabalhando com volumes

  - Melhor performance

  - Quando não tem controle das pastas, onde quer montar

  - Mapear volume para diversos containers ao mesmo tempo

```

$ docker volume

Usage:	docker volume COMMAND

Manage volumes

Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused local volumes
  rm          Remove one or more volumes


// Criar volume 
$ docker volume create meuvolume
meuvolume

$ docker volume ls
DRIVER              VOLUME NAME
local               meuvolume

// Detalhes do volume
$ docker volume inspect meuvolume
[
    {
        "CreatedAt": "2022-11-18T02:23:09-03:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/meuvolume/_data",
        "Name": "meuvolume",
        "Options": {},
        "Scope": "local"
    }
]

// Criando container com nome 'nginx' com volume
$ docker run --name nginx -d --mount type=volume,source=meuvolume,target=/app nginx

// Criar arquivo dentro do container
$ docker exec -it nginx bash
# cd app/
# touch oi
# exit

// Criando outro container, o 'nginx2' com volume
$ docker run --name nginx2 -d --mount type=volume,source=meuvolume,target=/app nginx

// Acessando o container 'nginx2'
$ docker exec -it nginx2 bash

// O arquivo 'oi' criado no primeiro container está aí
# cd app/
# ls
oi

// Se criar um novo arquivo no segundo container, o 'nginx2'
//   ele também estará no primeiro, o 'nginx'
# touch oi2
# exit
$ docker exec -it nginx bash
# cd app/
# ls
oi  oi2

// Acessando em um terceiro container (o 'nginx3') com o -v e o nome do volume diretamente
$ docker run --name nginx3 -d -v meuvolume:/app nginx
$ docker exec -it nginx3 bash
# cd app/
# ls
oi  oi2

// Apagar volumes não utilizados
$ docker volume prune

```
