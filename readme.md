# Getting Started Tutorial ToDo App

#### Build image in Dockerfile
```bash
docker build -t docker-todo .
```
#### Start container
```bash
docker run -dp 3000:3000 --name docker-todo docker-todo
```

## Sharing image and container

#### Tag image
```bash
docker tag docker-todo danydodson/docker-todo
```
#### Push image to docker registry
```bash
docker push danydodson/docker-todo
```

## Persist data for todo app

#### Create a named volume
docker volume create docker-todo-db
```
#### Run todo app container with -v to attach named docker-todo-db volume
```bash
docker run -dp 3000:3000 -v docker-todo-db:/etc/todos --name docker-todo docker-todo
```
#### Inspect the named volume
```bash
docker volume inspect docker-todo-db
```
#### Run todo app container with -w (working dir) -v to bind mount (local:container)
```bash
  docker run -dp 3000:3000 \
    --name docker-todo \
    -w /app \
    -v ${PWD}:/app \
    -v docker-todo-db:/etc/todos \
    node:12-alpine \
    sh -c "yarn install && yarn run dev"
```
#### Watch container logs
```bash
docker logs -f <container-id>
```

## Multi-Container apps

#### Create the network for containers to use
```bash
docker network create docker-todo-network
```
#### Start a mysql container and attach network
```bash
docker run -d \
  --name docker-todo-mysql \
  --network docker-todo-network --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```
#### Start mysql container and login to mysql using env `secret`
```bash
docker exec -it <mysql-container-id> mysql -p
```
#### Show dbs
```sql
mysql> SHOW DATABASES;
```

## Connecting to MySQL

#### Use (nicolaka/netshoot) tools for troubleshooting or debugging networking issues
```bash
docker run -it --network todo-network --name nicolaka-netshoot nicolaka/netshoot
```
#### Command dig mysql run a DNS tool
```bash
dig mysql
```
## Run our app with MySQL
```bash
docker run -dp 3000:3000 \
  --name todo-app-with-mysql \
  -w /app -v ${PWD}:/app \
  --network todo-network \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```
#### Connect to mysql db container and start cli
```bash
docker exec -ti <mysql-container-id> mysql -p todos
```
#### Command to display todos
```sql
select * from todo_items;
```

## Run using docker compose

#### Convert the command used to define app to docker-compose.yml
- app service definition command
```bash
docker run -dp 3000:3000 \
  -w /app -v ${PWD}:/app \
  --network todo-app \
  --name todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```
- app service definition in docker-compose
```yml
version: "3.7"

services:
  app:
  image: node:12-alpine
  command: sh -c "yarn install && yarn run dev"
  ports:
    - 3000:3000
  working_dir: /app
  volumes:
    - ./:/app
  environment:
    MYSQL_HOST: mysql
    MYSQL_USER: root
    MYSQL_PASSWORD: secret
    MYSQL_DB: todos

```
- mysql service definition command
```bash
docker run -d \
  --network todo-network --network-alias mysql \
  --name todo-mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```
- mysql service definition in docker-compose
```yml
version: "3.7"

services:
  mysql:
    image: mysql:5.7
    volumes:
      - todo-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:

```
#### complete docker-compose.yml
```yml
version: "3.7"

services:
  todo-app:
    image: node:12-alpine
    container_name: todo-app
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: todo-mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  todo-mysql:
    image: mysql:5.7
    container_name: todo-mysql
    volumes:
      - todo-db-vol:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-db-vol:
```
