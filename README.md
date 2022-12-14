# Docker

  - [Certificado conclusão](https://fullcycle.com.br/certificado/9851fc2b-8067-49f2-962b-815fcdd348a5/)

  - [Código-fonte](https://github.com/marciafc/fc-devops-docker)
  
    - Laravel
	
	- nginx
	
	- Node.js
  
  - [Iniciando com Docker](iniciando-com-docker/README.MD)
  
    - Publicando portas
	
	- Removendo containers
	
	- Acessando e alterando arquivos de um container
	
	- Iniciando com bind mounts
	
	- Trabalhando com volumes
	
  - [Trabalhando com imagens](trabalhando-com-imagens/README.MD)
  
    - Docker Hub
	
	  - [Documentation](https://docs.docker.com/docker-hub/)
	
	- Container registry
	
	- Criar imagem com Dockerfile
	
	- ENTRYPOINT vs CMD
	
	  - ENTRYPOINT é um comando fixo
	  
	  - CMD comando variável - usado como parâmetro do ENTRYPOINT
	
	- Entendendo melhor o ENTRYPOINT + EXEC (exec "$@"), variável de ambiente (ENV)
	
	- Publicar imagem no DockerHub
  
  - [Networks](networks/README.MD)  
  
    - docker network => inspect, create, connect	    
	
    - Tipos de redes
	
	- Bridge - Comunicação entre containers
	
	- Host - Junta a rede do Docker host com a rede do container
	
	- Acessar o host Docker de dentro do conteiner Docker
	
  - [Colocando em prática](colocando-em-pratica/README.MD)
  
    - Exemplo prático de criação de imagem e container até o push no DockerHub

      - docker build
  
      - docker run
  
      - docker logs	  
  
      - Alterar porta via CMD
	
	- Exemplo de app Nodejs sem possuir o Node instalado no Docker host
	
	  - Nodejs e Express instalados apenas no container, sem instalar na máquina
	  
	  - Gerar imagem da aplicação e subir para o DockerHub
	  
      - Criar e executar Dockerfile por ambiente (dev ou prod)	  
	  
  - [Otimizando imagens](otimizando-imagens/README.MD)
  
    - Otimização utilizando Multistage Building (prod X normal)
  
      - docker build -t marciafcinfo/laravel:prod   laravel -f laravel/Dockerfile.prod

      - docker build -t marciafcinfo/laravel:normal laravel -f laravel/Dockerfile
	
	  - docker images | grep laravel
	  
    - Nginx como proxy reverso
	
	  - Nginx -> laravel
	  
  - [Docker-compose](docker-compose/README.MD)	  
  
    - docker-compose.yaml com imagem fixa
	
	- docker-compose.yaml com imagem a ser "buildada"
	
	- docker-compose.yaml com MySQL e Node.js
	
	- Dependência entre containers com dockerize

  - [Desafios](desafios/README.MD)
  
    - [Desafio Go](https://github.com/marciafc/desafio-docker-fullcycle#desafio-go)
	
	- Nginx com Node.js
  
## Material extra
  
  - [Descomplicando o Docker - LINUXtips](https://github.com/badtuxx/DescomplicandoDocker)
  
    - [Livro](https://livro.descomplicandodocker.com.br/chapters/chapter_00.html)

  - [Docker para desenvolvedores - Gomex](https://github.com/gomex/docker-para-desenvolvedores)
  
  - [Remove All Containers and Images in Docker](https://www.geeksforgeeks.org/remove-all-containers-and-images-in-docker/#:~:text=Remove%20all%20Containers%3A%20To%20remove,containers%20in%20the%20docker%2Dmachine.)
  
    - Remove todos containers ``` $ docker rm $(docker ps -aq)```
	
	- Remove todas imagens ``` $ docker rmi $(docker images -q) ```
	