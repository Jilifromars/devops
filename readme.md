# Docker
## Database
***Why should we run the container with a flag -e to give the environment variables ?***

Pour éviter de déclarer des données sensibles comme des mots de passe (secrets) dans le Dockerfile.

***Why do we need a volume to be attached to our postgres container ?***

Cela permet la persistance des données lorsque l'on à besoin de re-build le containeur.

**Document your database container essentials:**

    FROM postgres:11.6-alpine 
    
    ENV POSTGRES_DB=db \ 
     POSTGRES_USER=usr \
     POSTGRES_PASSWORD=pwd

Commandes :

    sudo docker build -t db 
    sudo docker run --name=db -v data:/var/lib/postgresql/data --name database


## Backend API
***Why do we need a multistage build ? And explain each steps of this dockerfile***

  Il y à 2 étapes : premièrement il faut build le container puis le run. La première phase nécessite maven et la deuxième phase nécessite l'environnement d'exécution Java. 
On obtient des images plus petites avec qui n'utilisent que le nécessaire pour leur exécution.

**Dockerfile :**

     # Build
    FROM maven:3.6.3-jdk-11 AS myapp-build
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY pom.xml .
    COPY src ./src
    RUN mvn package -DskipTests
    # Run
    FROM openjdk:11-jre
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    
    COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
    
    ENTRYPOINT java -jar myapp.jar
## Httpd 
***Why do we need a reverse proxy ?***

On utilise un reverse proxy pour limiter les accès internet à notre application. Le proxy va permettre d'autoriser ou non des requêtes en faisant le liens ressources - client.


# CI/CD
***What are testcontainers ?***
Testcontainers est une librairie Java qui permet les tests unitaires. Ces containeurs sont l'avantage d'être légers.

***For what purpose do we need to push docker images ?***
Cela permet de versionner nos images pour récupérer une version antérieure si nécessaire.

**main.yaml :**
```name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - main
  pull_request:

env:
  GITHUB_TOKEN: $(secrets.GITHUB_TOKEN)
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

jobs:
  test-backend:
    runs-on: ubuntu-18.04
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3
      #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'

      #finally build your app with the latest command
      - name: Build and test with Maven
        working-directory: ./Backend_API/simple-api-compose/
        #run: mvn clean verify
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=Jilifromars_devops -Dsonar.organization=jilifromars -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{secrets.SONAR_TOKEN }} --file ./pom.xml

      # define job to build and publish docker image
  build-and-push-docker-image:
    # run only when code is compiling and tests are passing
    needs: test-backend
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Login to DockerHub
      run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{secrets.DOCKERHUB_TOKEN }}
    
    - name: Build image and push backend
      uses: docker/build-push-action@v2
      with:
        context: ./Backend_API/simple-api-compose/
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:simple-api
        push: ${{ github.ref == 'refs/heads/main' }}

    #database
    - name: Build image and push database
      uses: docker/build-push-action@v2
      with:
        context: db
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:database
        push: ${{ github.ref == 'refs/heads/main' }}

    #http
    - name: Build image and push httpd
      uses: docker/build-push-action@v2
      with:
        context: http
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:httpd
        push: ${{ github.ref == 'refs/heads/main' }}```
       
TP # Docker
## Database
***Why should we run the container with a flag -e to give the environment variables ?***

Pour éviter de déclarer des données sensibles comme des mots de passe (secrets) dans le Dockerfile.

***Why do we need a volume to be attached to our postgres container ?***

Cela permet la persistance des données lorsque l'on à besoin de re-build le containeur.

**Document your database container essentials:**

    FROM postgres:11.6-alpine 
    
    ENV POSTGRES_DB=db \ 
     POSTGRES_USER=usr \
     POSTGRES_PASSWORD=pwd

Commandes :

    sudo docker build -t db 
    sudo docker run --name=db -v data:/var/lib/postgresql/data --name database


## Backend API
***Why do we need a multistage build ? And explain each steps of this dockerfile***

  Il y à 2 étapes : premièrement il faut build le container puis le run. La première phase nécessite maven et la deuxième phase nécessite l'environnement d'exécution Java. 
On obtient des images plus petites avec qui n'utilisent que le nécessaire pour leur exécution.

**Dockerfile :**

     # Build
    FROM maven:3.6.3-jdk-11 AS myapp-build
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY pom.xml .
    COPY src ./src
    RUN mvn package -DskipTests
    # Run
    FROM openjdk:11-jre
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    
    COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
    
    ENTRYPOINT java -jar myapp.jar
## Httpd 
***Why do we need a reverse proxy ?***

On utilise un reverse proxy pour limiter les accès internet à notre application. Le proxy va permettre d'autoriser ou non des requêtes en faisant le liens ressources - client.

# A FINALISER


# CI/CD
***What are testcontainers ?***
Testcontainers est une librairie Java qui permet les tests unitaires. Ces containeurs sont l'avantage d'être légers.

***For what purpose do we need to push docker images ?***
Cela permet de versionner nos images pour récupérer une version antérieure si nécessaire.

**main.yaml :**
```name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - main
  pull_request:

env:
  GITHUB_TOKEN: $(secrets.GITHUB_TOKEN)
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

jobs:
  test-backend:
    runs-on: ubuntu-18.04
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3
      #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'

      #finally build your app with the latest command
      - name: Build and test with Maven
        working-directory: ./Backend_API/simple-api-compose/
        #run: mvn clean verify
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=Jilifromars_devops -Dsonar.organization=jilifromars -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{secrets.SONAR_TOKEN }} --file ./pom.xml

      # define job to build and publish docker image
  build-and-push-docker-image:
    # run only when code is compiling and tests are passing
    needs: test-backend
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Login to DockerHub
      run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{secrets.DOCKERHUB_TOKEN }}
    
    - name: Build image and push backend
      uses: docker/build-push-action@v2
      with:
        context: ./Backend_API/simple-api-compose/
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:simple-api
        push: ${{ github.ref == 'refs/heads/main' }}

    #database
    - name: Build image and push database
      uses: docker/build-push-action@v2
      with:
        context: db
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:database
        push: ${{ github.ref == 'refs/heads/main' }}

    #http
    - name: Build image and push httpd
      uses: docker/build-push-action@v2
      with:
        context: http
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:httpd
        push: ${{ github.ref == 'refs/heads/main' }}``` 