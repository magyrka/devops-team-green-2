
[![version](https://img.shields.io/badge/Ubuntu-20.04-brown)](https://semver.org)
[![version](https://img.shields.io/badge/NodeJS-14-green)](https://semver.org)
[![version](https://img.shields.io/badge/JAVA-11-green)](https://semver.org)
[![version](https://img.shields.io/badge/Posrgres-11-blue)](https://semver.org)
[![version](https://img.shields.io/badge/Redis-latest-red)](https://semver.org)
[![version](https://img.shields.io/badge/MongoDB-latest-green)](https://semver.org)
[![version](https://img.shields.io/badge/Tomcat-9.0.5-yellow)](https://semver.org)


# Class schedule
## Quick start ⚡
- [Application Diagram](#application-architecture-in-diagram)
- [For Developers](#instructions-for-developers-how-to-run-project-locally)
- [Run in Production](#instructions-how-to-deploy-project-in-production-or-stage)
- [Partners](#partners-)

## General info
This repository contains a source code of the Class Schedule Project.

The main goal of the project is designing a website where the university or institute staff will be able to create, store and display their training schedules.

Link to the development version of the site: https://develop-softserve.herokuapp.com/

## Application Architecture in Diagram
![schedule_application_architecturE.png](screenshots%2Fschedule_application_architecturE.png)

------------------------------------------
# Instructions for Developers (how to run Project locally)
Assuming that all commands will be running in shell from working directory and DockerCLI is installed
## Clone repository locally from Git
In order to create a local copy of the project you need:
1. Download and install the last version of Git https://git-scm.com/downloads
2. Open a terminal and go to the directory where you want to clone the files. 
3. Run the following command. Git automatically creates a folder with the repository name and downloads the files there.
```shell
git clone https://github.com/magyrka/devops-team-green && cd devops-team-green/
```
4. Enter your username and password if GitLab requests.

### Creating Your .env File with variables
```dotenv
FRONTEND_PORT=3000
BACKEND_PORT=8080
REDIS_PORT=6379
MONGO_PORT=27017
PG_PORT=5432

PG_PASSWORD=
DB_NAME=schedule
PG_USER=schedule
PG_DUMP_FILE='backup/2023-09-07.dump'

MONGO_DB_NAME=schedules
MONGO_CONTAINER_NAME=mongo_container
PG_CONTAINER_NAME=pg_container
REDIS_CONTAINER_NAME=redis_container
TOM_CONTAINER_NAME=tomcat_container

IP_SUBNET='192.168.100.0/28'
IP_PG='192.168.100.2'
IP_REDIS='192.168.100.3'
IP_MONGO='192.168.100.4'
IP_TOMCAT='192.168.100.5'
```

### Export variables from .env 
```shell
source .env
```

### Create docker network
```shell
docker network create --driver bridge --subnet $IP_SUBNET schedule_network
```

## Database PostgresQL
1. Run this simple commands to Start version 14 of PostgreSQL in Docker container
```shell
docker run -d --name $PG_CONTAINER_NAME \
	--network schedule_network \
	--ip $IP_PG \
	-v postgres-data:/var/lib/postgresql/data \
	-e POSTGRES_PASSWORD=$PG_PASSWORD \
	-e POSTGRES_DB=$DB_NAME \
	-e POSTGRES_USER=$PG_USER \
	-p 5432:5432 postgres:14
```
2. Restore PG data from file
```shell
docker cp $PG_DUMP_FILE $PG_CONTAINER_NAME:/tmp/backup.dump
docker exec -it $PG_CONTAINER_NAME psql -U $PG_USER -d $DB_NAME -f /tmp/backup.dump
docker exec -it $PG_CONTAINER_NAME psql -U $PG_USER -d $DB_NAME -c "CREATE DATABASE ${DB_NAME}_test WITH OWNER $PG_USER"
```
3. Configure connection url in `src/main/resources/hibernate.properties` and `src/test/resources/hibernate.properties` files:
```text
hibernate.connection.url=jdbc:postgresql://${IP_PG}:${PG_PORT}/schedule
```
## Database Redis
1. Start the latest version of Redis in Docker container   
```shell
docker volume create redis-data
docker run -d --network schedule_network \
   --name $REDIS_CONTAINER_NAME \
   --ip $IP_REDIS \
   -v redis-data:/data \
   -p $REDIS_PORT:$REDIS_PORT redis 
```
2. Configure connection url in `src/main/resources/cache.properties` file:
```text
redis.address = redis://${IP_REDIS}:${REDIS_PORT}
```

## Starting backend server using IntelliJ IDEA and Tomcat
1. Download and install the Ultimate version of IntelliJ IDEA (alternatively you can use a trial or EAP version) https://www.jetbrains.com/idea/download
2. Download and install Tomcat 9.0.50 https://tomcat.apache.org/download-90.cgi
3. Start the IDE and open class_schedule.backend project from the folder where you previously download it.
4. Make sure Tomcat and TomEE Integration is checked (`File –>> Settings –>> Plugins`).
5. `Run –>> Edit Configurations…`
6. Clicks `+` icon, select `Tomcat Server –>> Local`
7. Clicks on “Server” tab, then press `Configure...` button and select the directory with Tomcat server
8. Clicks on “Deployment” tab, then press `+` icon to select an artifact to deploy, and select `Gradle:com.softserve:class_schedule.war`
9. Press OK to save the configuration
10. `Run –>> Run 'Tomcat 9.0.50'` to start the backend server

## (Optionally) Run TomCat App in Docker container
```shell
docker build -t tom_app_img .
docker run -d --network schedule_network \
    --name $TOM_CONTAINER_NAME \
    --ip $IP_TOMCAT \
    -p $BACKEND_PORT:$BACKEND_PORT tom_app_img
```
------------------------------------------
# Instructions how to deploy Project in Production or Stage

## For deploying this application  run this simple commands (on VM or Remote Server):
### Clone repository
```shell
git clone https://github.com/magyrka/devops-team-green && cd devops-team-green/
```

### Run Docker compose file to start app
```shell
docker compose up -d -f docker-compose.prod.yaml
```
---------------------------------------
# Partners ❤️
## In the list, you can find collaborators, who was working in this project
- Developer Team:
   - Frontend team (incognito)
   - Backend team (incognito)
- Devops Team:
   - Vitaliy Kostyreva 
   - Vladyslav Manjirka
   - Valentyn Stratji
   - Liza Voievutska (Network support specialist)
   - Roman Hirnyak
