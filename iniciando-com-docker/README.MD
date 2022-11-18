# Iniciando com Docker

- Aula 1

```
docker ps

docker run hello-world

docker ps -a

```

- Aula 2

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

- Aula 3 - Publicando portas com nginx

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

