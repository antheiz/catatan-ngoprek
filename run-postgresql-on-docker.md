### Run PostgreSQL on Docker
--- 

* Pull image postgres from [Docker Hub](https://hub.docker.com/_/postgres)
    - docker pull postgres   

* Run your images and create your container name 
    - docker run -d -e POSTGRES_PASSWORD=salupa --name postgres -p 5432:5432 postgres
    
* Enter the postgres bash system
    -  docker exec -it postgres bash

* Enter to postgres cli
    - psql -h localhost -U postgres
