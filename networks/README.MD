# Networks

## Aula 1 - Entendendo [tipos de Network](https://docs.docker.com/network/)

  - [Bridge](https://docs.docker.com/network/bridge/) (bem comum seu uso, qdo não define a network é a default)
  
    - [Tutorial](https://docs.docker.com/network/network-tutorial-standalone/)
  
  - [Host](https://docs.docker.com/network/host/) (bem comum seu uso)
  
  - [Overlay](https://docs.docker.com/network/overlay/) (comum com Swarm)
  
  - [Macvlan](https://docs.docker.com/network/macvlan/)
  
  - [None](https://docs.docker.com/network/none/)
  
    - Isola um container, não vai estar em rede


## Aula 2 - Trabalhando com bridge

```

$ docker network 

Usage:	docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks
  
========================  
  
// Listar redes  
$ docker network ls

NETWORK ID          NAME                            DRIVER              SCOPE
ada773817be3        bridge                          bridge              local
bff369afdd52        elasticsearch7-docker_default   bridge              local
6bffa38fe0c9        host                            host                local
6c807046fa71        kafka_default                   bridge              local
5b9e316903c3        mongodb_default                 bridge              local
c9fcbbcf3f0a        none                            null                local
b4a9dd5c8569        redis_default                   bridge              local
  
========================  

// Remover networks não utilizadas
$ docker network prune

// Networks padrão
$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
ada773817be3        bridge              bridge              local
6bffa38fe0c9        host                host                local
c9fcbbcf3f0a        none                null                local

========================  

// Criação de dois containers
$ docker run -d -it --name ubuntu1 bash
$ docker run -d -it --name ubuntu2 bash

========================

// Inspecionando network
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        ...,
        "Containers": {
            "15582a867498bd2053682a27a55c7fcb4d68d1beee70d87c5fe7a4eb7db28158": {
                "Name": "ubuntu1",
                ...,
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            ...,
            "9f672828ddd772a6564cca29173907fba8704a0c6ebaa1c15987b93bc31de389": {
                "Name": "ubuntu2",
                ...,                
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {
          ...
        },
        "Labels": {}
    }
]

========================

// Rodando no modo attach o container 'ubuntu1', cujo ip é 172.17.0.3
$ docker attach ubuntu1

# ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var


// Visualizando IP do container, no caso, inet 172.17.0.3
# ip addr show

...
inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0


// Ping no outro container, o 'ubuntu2', cujo ip é 172.17.0.4
# ping 172.17.0.4

PING 172.17.0.4 (172.17.0.4): 56 data bytes
64 bytes from 172.17.0.4: seq=0 ttl=64 time=0.359 ms

========================

// Não está fazendo resolução de nome
# ping ubuntu2
ping: bad address 'ubuntu2'

========================

// Removendo containers ubuntu1 e ubuntu2
control D no ubuntu1

$ docker stop ubuntu2
ubuntu2

$ docker rm ubuntu1 ubuntu2

========================

// Criar uma rede bridge com nome 'minharede'
$ docker network create --driver bridge minharede


$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
5a11e287a92b        minharede           bridge              local
...


// Criar dois containers na rede 'minharede'
// -d -it é igual a -dit
$ docker run -dit --name ubuntu1 --network minharede bash
$ docker run -dit --name ubuntu2 --network minharede bash


// Entrar no primeiro container
$ docker exec -it ubuntu1 bash


// Agora consegue se comunicar com resolução de nomes
# ping ubuntu2

PING ubuntu2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.314 ms

// Criando outro container mas nao está na rede 'minharede'
$ docker run -dit --name ubuntu3 bash

// Não consegue fazer ping nos outros containers
$ docker exec -it ubuntu3 bash

# ping ubuntu2
ping: bad address 'ubuntu2'

# ping ubuntu1
ping: bad address 'ubuntu1'


// Conectar um container em uma network
// ubuntu3 agora estará conectado a 'minharede'
$ docker network connect minharede ubuntu3

// Acessando container 'ubuntu3'
$ docker exec -it ubuntu3 bash

// E agora consegue fazer ping nos outros containers que estão na 'minharede'
# ping ubuntu2
PING ubuntu2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.111 ms

# ping ubuntu1
PING ubuntu1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.314 ms

====================

// Inspecionando a rede criada -> 'minharede'
$ docker network inspect minharede 

[
    {
        "Name": "minharede",
		...,
        "Driver": "bridge",
        ...,
        "Containers": {
            "56baa56a937f461e8453317545c89f9bf1619f004bb047392eed41452c57bce2": {
                "Name": "ubuntu3",
           		...,
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "64346c0a8f82ec2e32e02889b26fd7ab824d39ae0a6c7a175a60af628ea8cd92": {
                "Name": "ubuntu2",
                ...,
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "65f511faa656f6814294447f5b6edd3bf6cfc454a28c406af2b4a21aa37c1284": {
                "Name": "ubuntu1",
                ...,
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

## Aula 3 - Trabalhando com host

  - Junta a rede do Docker host com a rede do container
  
    - ``` --network host ```
  
  - No MAC não funciona porque a máquina não é a máq virtual que está rodando o Docker 

```

// Ao acessar 'localhost' já funciona na porta 80
$ docker run --rm -d --name nginx --network host nginx

http://localhost/

http://localhost:80

Welcome to nginx!
...

```

## Aula 4 - Container acessando nossa máquina

  - "As of Docker version 18.03, you can use the host.docker.internal hostname to connect to your Docker host from inside a Docker container."
    - [https://medium.com/@TimvanBaarsen/how-to-connect-to-the-docker-host-from-inside-a-docker-container-112b4c71bc66](https://medium.com/@TimvanBaarsen/how-to-connect-to-the-docker-host-from-inside-a-docker-container-112b4c71bc66)

```
// Criando container com imagem do Ubuntu 20.04 LTS (Focal Fossa)
//   Precisei definir --add-host=host.docker.internal:host-gateway
//     https://stackoverflow.com/questions/31324981/how-to-access-host-port-from-docker-container/61424570#61424570
//   The magical IP number worked: `--add-host=host.docker.internal:172.17.0.1 `
//     https://stackoverflow.com/questions/48546124/what-is-linux-equivalent-of-host-docker-internal
$ docker run --rm -it --add-host=host.docker.internal:172.17.0.1 --name ubuntu-focal ubuntu:focal bash

// Instalando curl no container
# apt-get update \
     && apt-get -y install curl

// Subindo um servidor http com o python na porta 8000 no Docker host (minha máquina)
// Rodei o comando na pasta /fullcycle-3-0-docker/trabalhando-com-imagens/aula3/html, onde tem um index.html
$ python3 -m http.server 8000

// Navegador
http://localhost:8000/
Full Cycle

// Executar curl do container 'ubuntu-focal'
# curl http://host.docker.internal:8000
<h1>Full Cycle</h1>

```