# Docker-compose

## Aula 1 - Iniciando com Docker-compose (DC)

  - https://github.com/marciafc/fc-devops-docker/blob/main/docker-compose.laravel.yaml
  
```
version: '3'

services: 

  laravel:
    image: wesleywillians/laravel:prod
    container_name: laravel
    networks: 
      - laranet

  nginx:
    image: wesleywillians/nginx:prod
    container_name: nginx
    networks: 
      - laranet
    ports: 
      - "8080:80"

networks: 
  laranet:
    driver: bridge    


=================

$ docker-compose up

```

## Aula 2 - Buildando imagens com DC 

  - Usar 'build' no DC e não imagem fixa
  
  - [https://github.com/marciafc/fc-devops-docker/blob/main/docker-compose.laravel.yaml](https://github.com/marciafc/fc-devops-docker/blob/main/docker-compose.laravel.yaml)
  
```
version: '3'

services: 

  laravel:
  
// Qual pasta está o Dockerfile e nome do arquivo  
    build: 
      context: ./laravel
      dockerfile: Dockerfile.prod
	  
// é possível deixar o nome apenas como referência (irá gerar img com esse nome)	
// ou não informar
    image: wesleywillians/laravel:prod
	
    container_name: laravel
    networks: 
      - laranet

  nginx:
  
// Qual pasta está o Dockerfile e nome do arquivo  
    build: 
      context: ./nginx
      dockerfile: Dockerfile.prod
	  
// é possível deixar o nome apenas como referência (irá gerar img com esse nome)		
// ou não informar	  
    image: wesleywillians/nginx:prod
	
    container_name: nginx
    networks: 
      - laranet
    ports: 
      - "8080:80"

networks: 
  laranet:
    driver: bridge    
	
	
=================

// -d não trava terminal (detached)
$ docker-compose up -d	

// rebuilda as imagens (atualiza alterações nas imagens)
// Usar sempre que alterar os Dockerfile
$ docker-compose up -d	--build

$ docker-compose ps

$ docker-compose down

```

## Aula 3 - Criando banco de dados MySQL

  - Projeto [https://github.com/marciafc/fc-devops-docker/](https://github.com/marciafc/fc-devops-docker/)
  
    - docker-compose.yaml
  

```
version: '3'

services: 

  app:
    build: 
      context: node
    container_name: app
    entrypoint: dockerize -wait tcp://db:3306 -timeout 20s docker-entrypoint.sh
    networks: 
      - node-network
    volumes: 
      - ./node:/usr/src/app
    tty: true
    ports: 
      - "3000:3000"
    depends_on: 
       - db

  db:
    image: mysql:5.7
    command: --innodb-use-native-aio=0
    container_name: db
    restart: always
    tty: true
    volumes: 
// pasta 'mysql' do host mapeada para 'var/lib/mysql' do container	  
      - ./mysql:/var/lib/mysql
	  
    environment: 
	
// MySQL 5.7 MYSQL_USER=root não é mais aceita (já é default)
      - MYSQL_DATABASE=nodedb
      - MYSQL_ROOT_PASSWORD=root
    networks: 
      - node-network

networks: 
  node-network:
    driver: bridge      

```

Rodar o DC na pasta raiz (onde está o arquivo docker-compose.yaml)

```
// Criar os containers
$ docker-compose up -d	

$ docker ps

// Logs do mysql (container db)
$ docker logs db



```

