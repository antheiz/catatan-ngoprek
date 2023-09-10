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
    - 
* Step 6 (optional) -  Create container for PGAdmin
    - docker run --name pgadmin -p 5050:80 -e "PGADMIN_DEFAULT_EMAIL=admin@foo.com" -e "PGADMIN_DEFAULT_PASSWORD=rahasia" -d dpage/pgadmin4
