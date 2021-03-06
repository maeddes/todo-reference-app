## 1. Clone and get started with the repo

```bash
git clone https://github.com/maeddes/todo-reference-app.git
```

Note the various branches

```bash
git branch
  db_profiles
  in_memory
* master
```

The application consists of 3 main components
- UI (todoui)
- Main app (todobackend)
- DB (postgresdb)

todoui and todobackend are Spring Boot apps with the following external configuration possibilies:

todoui:
```properties
backend.host=${BACKEND_HOST:todobackend}
backend.port=${BACKEND_PORT:8090}
backend.url=http://${backend.host}:${backend.port}
```

todobackend (in case of 'prod' profile in db_profiles branch)
```properties
spring.datasource.url= jdbc:postgresql://${POSTGRES_HOST:postgresdb}:5432/mydb 
```

## 2. Run in standalone in-memory mode

```bash
git checkout in_memory
mvn -f todobackend spring-boot:run
mvn -f todoui spring-boot:run
```

backend shoud start on port 8080 by default, ui on port 8090

can be specified explicitly through backend.endpoint variable, e.g.

```bash
mvn -f todobackend spring-boot:run -Dserver.port=8081
mvn -f todoui spring-boot:run -Dbackend.endpoint='http://localhost:8081' -Dserver.port=8082
```

## 3. Run in multiple database mode

### A) Run with H2 in-memory database

```bash
git checkout db_profiles
mvn -f todobackend spring-boot:run -Dspring.profiles.active=dev 
mvn -f todoui spring-boot:run 
```

### B) Run with dockerized Postgres DB

```bash
git checkout db_profiles
docker run --name postgresdb -p 5432:5432 -e POSTGRES_PASSWORD=password -e POSTGRES_USER=matthias -e POSTGRES_DB=mydb -d postgres:latest 
mvn -f todobackend spring-boot:run -Dspring.profiles.active=prod -DPOSTGRES_HOST=localhost
mvn -f todoui spring-boot:run -DBACKEND_HOST=localhost
```

## 4. Run all components containerized

Build the applciation jars and container images

```bash
mvn -f todobackend clean install
mvn -f todoui clean install
docker build -f Dockerfile-todobackend -t maeddes/todobackend:v1 .
docker build -f Dockerfile-todoui -t maeddes/todoui:v1 .
```

### A) Use Docker Links - (THIS FEATURE IS DEPRECATED!)

In case it is not running start the DB as described in 3B.

Run the container images and set the links to dependent containers.

```bash
docker run --name todobackend -p 8080:8080 -e SPRING_PROFILES_ACTIVE='prod' -e SPRING_DATASOURCE_URL='jdbc:postgresql://db:5432/mydb' --link=postgresdb:db maeddes/todobackend:v1
```

OR

```bash
docker run --name todobackend -p 8080:8080 -e SPRING_PROFILES_ACTIVE='prod' --link=postgresdb:postgresdb maeddes/todobackend:v1
```

```bash
docker run --name todoui -p 8090:8090 -e BACKEND_ENDPOINT='http://todobackend:8080' --link=todobackend:todobackend maeddes/todoui:v1
```

### B) Use Docker Network

clear all containers

```bash
docker network create todonet

docker run --net todonet --name postgresdb -p 5432:5432 -e POSTGRES_PASSWORD=password -e POSTGRES_USER=matthias -e POSTGRES_DB=mydb -d postgres:latest

docker run --net todonet --name todobackend -p 8080:8080 -e SPRING_PROFILES_ACTIVE='prod' maeddes/todobackend:v1
```

or (alternative version)
```bash
docker run --net todonet --name todobackend -p 8080:8080 -e SPRING_PROFILES_ACTIVE='prod' -e SPRING_DATASOURCE_URL='jdbc:postgresql://postgresdb.todonet:5432/mydb'  maeddes/todobackend:v1
```

```bash
docker run --net todonet --name todoui -p 8090:8090 -e BACKEND_ENDPOINT='http://todobackend.todonet:8080' maeddes/todoui:v1

docker network inspect todonet
```

## 5. Use docker-compose

clear all containers

```bash
docker-compose up
```

(uses docker-compose.yml)

## 6. Kubernetes



