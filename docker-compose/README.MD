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
  
    - docker-compose.yaml parte 'db: ...'
  

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

// Configurando o MySQL
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


## Aula 4 - Errata - Criando banco de dados MySQL

A partir da imagem da versão MySQL 5.7: não usar MYSQL_USER=root (já é default)


## Aula 5 - Configurando app node com docker-compose

  - Projeto [https://github.com/marciafc/fc-devops-docker/](https://github.com/marciafc/fc-devops-docker/)
  
    - docker-compose.yaml parte 'app:...'
  
```
version: '3'

services: 

// Configurando o Node.js
  app:
    build: 
// pasta node é o contexto	
// Não precisa definir o 'dockerfile' pq já vai pegar o default
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
      - ./mysql:/var/lib/mysql
	  
    environment: 
      - MYSQL_DATABASE=nodedb
      - MYSQL_ROOT_PASSWORD=root
    networks: 
      - node-network

networks: 
  node-network:
    driver: bridge      

```  

  - Acessar container node.js

```  
$ docker-compose up -d --build

$ docker exec -it app bash  
```  


## Aula 6 - Node vs MySQL


```  
$ docker-compose up -d --build

// Aplicação Node.js
$ docker exec -it app bash  

// Em outra aba, acessar o mysql
$ docker exec -it db bash

bash-4.2# mysql -uroot -p

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| nodedb             |
| performance_schema |
| sys                |
+--------------------+

mysql> use nodedb;

mysql> create table people(id int not null auto_increment, name varchar(255), primary key(id));

mysql> desc people;

```  

Na aba que está o Node.js

```  
// Instalar o MySQL
/usr/src/app# npm install mysql --save

```  

Criar arquivo [index.js](https://github.com/marciafc/fc-devops-docker/blob/main/node/index.js)

```  
const express = require('express')
const app = express()
const port = 3000

const config = {
//host: 'db' -> é o nome do container
    host: 'db',
    user: 'root',
    password: 'root',
    database:'nodedb'
};

const mysql = require('mysql')
const connection = mysql.createConnection(config)

// Insert
const sql = `INSERT INTO people(name) values('Wesley')`
connection.query(sql)
connection.end()


app.get('/', (req,res) => {
    res.send('<h1>Full Cycle</h1>')
})

app.listen(port, ()=> {
    console.log('Rodando na porta ' + port)
})
```

Rodar a app Node.js

```
/usr/src/app# node index.js 
```

No container do mysql, rodar select

```
mysql> select * from people;
+----+--------+
| id | name   |
+----+--------+
|  1 | Wesley |
+----+--------+

```