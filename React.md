
# Dockerizing React App



## Development Environment

Create a docker file in root directory of project with filename Dockerfile

```bash
FROM node:12-alpine
USER node
RUN mkdir /home/node/code/
WORKDIR /home/node/code/
COPY --chown=node:node package.json package-lock.json ./
RUN npm ci
COPY --chown=node:node . .
CMD ["npm", "start"]
```
From node:12-alpine : Use node container from docker hub with version 12 and based on alpine distribution. Alpine is lightweight distribution with less memory, better performance, security and maintainability.

USER node : Create a user node & add in group node

RUN mkdir /home/node/code : Create a directory with user role node

WORKDIR /home/node/code : Set working directory inside the container

COPY --chown=node:node package.json package-lock.json ./: Copy package.json & package-lock.json in working directory inside container. Cache npm packages which helps skip installing packages in subsequent docker container builds.

npm ci : Bypass package.json to install packages from lockfile package-lock.json which is faster than npm install

COPY --chown=node:node . . : Copy our project in container working directory

CMD ["npm", "start"] : Execute command on running the docker

##Build image from a Dockerfile

Go to your terminal and run the following command
```
docker build -t react-app .
```
-t: Creates named tag. If we simply execute docker build . , it will generate random string as container id.

Running docker image
```
docker run -it --rm -v ${PWD}:/home/node/code -v /home/node/code/node_modules -p 3000:3000 -e CHOKIDAR_USEPOLLING=true react-app
```
The development environment should run in http://localhost:3000 in the browser.

-it : Run docker in interactive mode

-v : Mount our project source in container and persist data changes using container volumes. You can find more information about volumes here.

``` Note: Don't use volumes in production build. Instead copy all the dependencies to docker image using copy command. Create separate dockerfiles for different images for production eg- postgresql. Copy all dependencies to that image. ```

Use volumes in development: 
Volumes are the preferred mechanism for persisting data generated by and used by Docker containers. While bind mounts…
read more here : https://docs.docker.com/storage/volumes/

-p : map port between the application and container

CHOKIDAR_USEPOLLING : Use polling to watch for file changes in create react app

Running docker image with compose
Compose is a tool for defining and running multi-container docker applications. It provides simpler and more efficient way for running our docker images.

Create a compose yml file with filename docker-compose.yml

version: "3.7"
services:
  react:
    container_name: react
    build: .
ports:
      - "3000:3000"
    volumes:
      - .:/home/node/code
      - /home/node/code/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true
      - CI=true
Running docker image using docker compose

docker-compose up
### Force building docker image
docker-compose up -d --build

# Production Environment
Create a different docker file with filename prod.Dockerfile

We are using multi staged docker containers for build and production environment. We can use nginx container to serve production files.

```bash
# Production environment
FROM node:12-alpine as build
USER node
RUN mkdir /home/node/code/
WORKDIR /home/node/code/
COPY --chown=node:node package.json package-lock.json ./
RUN npm ci
COPY --chown=node:node . .
RUN npm run build
FROM nginx:1.17
COPY --from=build /home/node/code/build /usr/share/nginx/html
```

Running docker image
```
docker build -t react-app -f ./prod.Dockerfile . 
docker run -p 8080:80 react-app
```
The production environment should run in http://localhost:8080 in the browser.

Running docker image with compose
Create a compose yml file with filenamedocker-compose.prod.yml

```bash
version: "3.7"
services:
  prod:
    container_name: prod
    build:
      context: .
      dockerfile: prod.Dockerfile
    ports:
      - "8080:80"
```

Running docker image using docker compose
```
docker-compose -f ${PWD}/docker-compose.prod.yml up -d
```
References:
https://btholt.github.io/complete-intro-to-containers