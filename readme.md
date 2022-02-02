TP part 01 - Docker

Database

Why should we run the container with a flag -e to give the
environment variables ?

when we talk about /docker-entrypoint-initdb.d it means inside the container so we have to copy your directory and the container’s directory?

Why do we need a volume to be attached to our postgres container ?


1-1 Document your database container essentials: commands and
Dockerfile

commands : 


Creating a network :
sudo docker network create my-net
Building database image and container :
sudo docker build -t db .
sudo docker run --name=db-container -p 5432:5432 --network=my-net -t db
Building API image and container :
sudo docker build -t api .
sudo docker run --name=api-container -p 8080:8080 --network=my-net -t api

Creating the image : 
    sudo docker build -t bd .
Building the container : 
    sudo docker run --name=db -v data:/var/lib/postgresql/data -t devops
Delete the container :
    sudo docker rm db

Backend API

1-2 Why do we need a multistage build ? And explain each steps of
this dockerfile

How to access the database container from your backend application? Use the deprecated
--link or create a docker network.

2-2 Document your Github Actions configurations
Secured variables, why ?

Why did we put needs: build-and-test-backend on this job?