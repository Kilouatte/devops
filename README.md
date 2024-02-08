# TP1 : devops
## Q1.2 : Why do we need a multistage build? And explain each step of this dockerfile.

We use a multistage build to : </br>
- reduce the image size, we copy only necessary files or artifacts into the last stage. </br>
- isolate the build dependencies, all the lib required during the build phase are isolated in the intermediate stage. So the last image include runtime depencencies of the application and nothing else.</br>
- separe the different phases of the build process. It makes it easir to manage and we can understand the Dockerfile in each stage.

# Postgresql Setup

## How to build docker image:
`docker build -t Kilouatte/database`

## Hwo to creare a network :
`docker network create app-network`

## How to run the container with arguments -d to run in background and --net to connect to the network:
`docker run -d --net=app-network --name=database Kilouatte/database`
### We had --name=database to give a name to the container

## How to run the adminer container with arguments -p to expose the port and --net to connect to the network:
`docker run -p "8090:8080" --net=app-network --name=adminer -d adminer`

## How to mount a volume to the container with arguments -v to mount the volume and --net to connect to the network:
`docker run -d -v data:/var/lib/postgresql/data --net=app-network --name=db Kilouatte/database`

## Dockerfile configuration :

```bash
FROM postgres:14.1-alpine

COPY ./createSchema.sql /docker-entrypoint-initdb.d/
COPY ./insertData.sql /docker-entrypoint-initdb.d/
```
### The first line of the Dockerfile specifies the base image you are using to build the new image. In this case, we are using the official postgres image from Docker Hub.
### The second line copies the createSchema.sql file from the current directory to the /docker-entrypoint-initdb.d/ directory in the image. This directory is used by the official Postgres image to initialize the database with custom scripts. The scripts are executed in alphabetical order, so we can use numbers to control the order of execution.
### The third line as the second one copies the insertData.sql file from the current directory to the /docker-entrypoint-initdb.d/ directory in the image.

# Now for the backend server

## How to build backend docker image:

`docker build -t Kilouatte/backend`

## How to run the container with arguments -p to expose the port and --net to connect to the network:
`docker run -p "8080:8080" --name=backend --net=app-network Kilouatte/backend`
### We had --name=database to give a name to the container

## Backend Dockerfile configuration :

```bash
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```
### summary of the Dockerfile :
- The first block of the Dockerfile is the build stage. It uses the official Maven image to build the application. It copies the pom.xml and src directories to the /opt/myapp directory in the image. Then it runs the mvn package command to build the application. The -DskipTests option is used to skip running the tests during the build process.
- The second block of the Dockerfile is the run stage. It uses the official Amazon Corretto image as the base image. It copies the JAR file from the build stage to the /opt/myapp directory in the image. Then it sets the entrypoint to java -jar myapp.jar to run the application when the container starts.


# Now for the httpd: Apache

## How to build httpd docker image:
`docker build -t Kilouatte/apache`

## How to run the container with arguments -dit to run in background and -p to expose the port and --net to connect to the network:
`docker run -dit -p "8000:80" --net=app-network --name=apache Kilouatte/apache`
### We had --name=database to give a name to the container

## Httpd Dockerfile configuration :

```bash
FROM httpd:2.4
COPY ./ /usr/local/apache2/htdocs/
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf

```
### summary of the Dockerfile :
- The first line of the Dockerfile specifies the base image you are using to build the new image.
- The second line copies the content of the current directory to the /usr/local/apache2/htdocs/ directory in the image. This is the default directory where the Apache HTTP Server serves files from.
- The third line copies the httpd.conf file from the current directory to the /usr/local/apache2/conf/httpd.conf file in the image. This is the main configuration file for the Apache HTTP Server.


## Answer to the question 1.2:
### Why do we need a multistage build? And explain each step of this dockerfile.

We use a multistage build to : </br>
- reduce the image size, we copy only necessary files or artifacts into the last stage. </br>
- isolate the build dependencies, all the lib required during the build phase are isolated in the intermediate stage. So the last image include runtime dependencies of the application and nothing else.</br>
- spare the different phases of the build process. It makes it easier to manage and we can understand the Dockerfile in each stage.

### Document docker-compose most important commands.

## Docker Compose ymal file :
```yaml
version: '3.8'

services:
    backend:
        #build: ./Java/api/simple-api-student-main/
        env_file: .env
        networks:
          - app-network
        depends_on:
          - database
        image: kilouatt/backend:1.0

    database:
        #build: ./Database
        volumes:
          - database:/var/lib/postgresql/data
        networks:
          - app-network
        env_file: .env
        image: kilouatt/database:1.0

    httpd:
        #build: ./Http
        ports:
          - 80:80
        networks:
          - app-network
        depends_on:
          - backend
        image: kilouatt/httpd:1.0

networks:
    app-network:


volumes:
    database:

```
### summary of the docker-compose file :
- `version: '3.8'`-> The version key specifies the version of the Docker Compose file format.
- `services:` -> The services key is used to define the services that make up the application. In this case, we have three services: backend, database, and httpd.
- `backend:` -> The backend service is used to define the configuration for the backend server.
- `env_file: .env` -> The env_file key is used to specify the path to an environment file that contains environment variables for the service. Here it is log for the database.
- `networks:` -> The networks key is used to specify the networks that the service should be connected to. In this case, the backend and database services are connected to the app-network network.
- `depends_on:` -> The depends_on key is used to specify the services that the current service depends on.
- `image:` -> The image key is used to specify the name of the image to use for the service. In this case, the backend service uses the kilouatt/backend:1.0 image.
### For image, we choose between build and image. If we use build, we have to specify the path to the Dockerfile and the context in which the build should be run. If we use image, we have to specify the name of the image to use for the service, we can find it on docker hub.
- `volumes:` -> The volumes key is used to specify the volumes that should be mounted for the service. In this case, we use a named volume called database to persist the data for the database service.
### For the volumes, if we use a name, compose will create a volume with this name. If we use a path, compose will mount the volume to the path.
- `ports` -> The ports key is used to specify the ports that should be exposed for the service. In this case, the httpd service exposes port 80 on the host machine.
### For the two last arguments networks and volumes, we specify the name of the network or volume to use for the service.


## Network will be used to connect the different containers together. It will allow the containers to communicate with each other. So we specify the same name for every container.

## We have a .env file to store the environment variables to connect to the database :

```bash
POSTGRES_DB=db
POSTGRES_USER=usr
POSTGRES_PASSWORD=pwd
```

# Now for the docker publish :

### How to publish the images to DockerHub :
- First we have to login to DockerHub with the command `docker login`
- Then we have to tag the image with the command `docker tag tp1-db kilouatt/httpd:1.0`
- Finally, we have to push the image to DockerHub with the command `docker push kilouatt/httpd:1.0`

### We will do the same for the 3 containers : database, backend and httpd.

### Comments about published images :
- The images are published on DockerHub and can be pulled by anyone who has access to the internet.
- The images are tagged with the version number 1.0 to indicate that they are the first version of the images.
- We can push different versions of the images to DockerHub to keep track of the changes and to allow users to choose which version they want to use.

# TP2 : GitActions:

```xml
<dependencies>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>jdbc</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```
## Q2.1 : What are testcontainers?
- Testcontainers is a Java library that provides lightweight, throwaway instances of common databases or anything else that can run in a Docker container. It is used to write integration tests for applications.

## Github actions configuration

### Sonar Cloud github action

File commented : [sonar.yml](.github/workflows/sonar.yml)

### Docker github action

File commented : [docker.yml](.github/workflows/docker.yml)


# TP 3 - Commands_Ansible

## Useful commands
### Ansible with python 3.11
`python3.11 -m venv ansible`

### Activate the virtual environment
`source ansible/bin/activate`

### Check python version 
`python --version`

### Install ansible
`python3 -m pip install ansible`

### Check ansible version
`ansible --version`

### Ugrade ansible
`python3 -m pip install --upgrade ansible`  

### We give the read only permission to the key
`chmod 400 ./id_rsa`

### We connect to the server.
`ssh -i ~/.ssh/id_rsa_takima cento@julien.perbet.takima.cloud`
#### We need to validate the fingerprint

# TP 3 - Ansible
## setup.yml configuration
```yaml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ~/.ssh/id_rsa_takima
  children:
    prod:
      hosts: julien.perbet.takima.cloud
```
### After this command `ansible all -i inventories/setup.yml -m ping` we have :


### We use `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"` to get the distribution of the server.

### Now we delete apache with `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become`.

## playbook.yml first configuration
```yaml
- hosts: all
  gather_facts: false
  become: true

  tasks:
   - name: Test connection
     ping:
```

### We use `ansible-playbook -i inventories/setup.yml playbook.yml` to test the connection.

## playbook.yml second configuration
```yaml
- hosts: all
  gather_facts: false
  become: true

# Install Docker
  tasks:

  - name: Install device-mapper-persistent-data
    yum:
      name: device-mapper-persistent-data
      state: latest

  - name: Install lvm2
    yum:
      name: lvm2
      state: latest

  - name: add repo docker
    command:
      cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

  - name: Install Docker
    yum:
      name: docker-ce
      state: present

  - name: Install python3
    yum:
      name: python3
      state: present

  - name: Install docker with Python 3
    pip:
      name: docker
      executable: pip3
    vars:
      ansible_python_interpreter: /usr/bin/python3

  - name: Make sure Docker is running
    service: name=docker state=started
    tags: docker
```
- We install everything we need to install docker.
- After this we can delete the docker installation since it is already installed.

#### We use `ansible-playbook -i inventories/setup.yml playbook.yml` to install docker.

### We do `ansible-galaxy init roles/docker` to create the role docker.
#### We run again `ansible-playbook -i inventories/setup.yml playbook.yml`.

## We create a secret  encrypted file to store all the sensitive data.
### And our command is ` ansible-env % ansible-playbook -i inventories/setup.yml -e @secret.yml --ask-vault-password  playbook.yml`

# Playbook commented : [playbook.yml](tp3/ansible-env/playbook.yml)
## Others files are in : 
- [Roles](tp3/ansible-env/roles)
- [Inventory/Setup.yml](tp3/ansible-env/inventories/setup/yml)
- [httpd.conf](tp1/Http/httpd.conf)


# WARNING : LOAD BALANCING NOT WORKING, ILYES AND I WORKED ON IT VERY LATE AND IT DIDN'T WORKD FOR SOME REASON.
# I'M NOT SURE EVERYTHING TURNED BACK AS IT SHOULD BE, BUT I DID ALL THE CHECKS !







