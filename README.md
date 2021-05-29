## Building a Simple App with Docker 
This project uses Docker to build a simple app to prove out an MVP (minimum viable product).

### Introduction
This project builds a simple task manager using Node.js and Docker. This project we build to demonstrate teh capabilities
of docker and docker-compose to develop, build and package application.

#### Build the Image
To build the container run the following command in the root directory of the project. 

``` bash 
docker build -t simple-task-manager .
```
Run container
``` bash 
docker run -dp 3000:3000 simple-task-manager
```

- `-d` and `-p` flags - Run in detached (background) mode and create a port mapping

Stop and remove
``` bash
docker ps
docker rm -f <the-container-id>
```

#### Pushing  Image to Docker Hub
Login to Docker Hub
``` bash
docker login -u YOUR-USER-NAME
```

Change name
``` bash
docker tag getting-started YOUR-USER-NAME/simple-task-manager
```

Push
``` bash
docker push YOUR-USER-NAME/simple-task-manager
```

This is quite common in CI pipelines, where the pipeline will create the image and push it to a registry and then
the production environment can use the latest version of the image.


If you would like to test the image on a different environment you can use [Play with Docker](https://labs.play-with-docker.com/).



### File System 
Understanding the file system of docker:
``` bash
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

``` bash
docker exec <container-id> cat /data.txt
```

#### Named Volumes

Create a named volume
``` bash
docker volume create todo-db
```

Mount named volume when running the container
``` bash
docker run -dp 3000:3000 -v todo-db:/etc/todos simple-task-manager
```

Diving into the Volume

``` bash
docker volume inspect todo-db
```

### Using Bind Mounts

Bind mounts and named volumes are the two main types of volumes that come with the Docker engine. However,
additional volume drivers are available to support other use cases
([SFTP](https://github.com/vieux/docker-volume-sshfs),
[Ceph](https://ceph.com/geen-categorie/getting-started-with-the-docker-rbd-volume-plugin/),
[NetApp](https://netappdvp.readthedocs.io/en/stable/),
[S3](https://github.com/elementar/docker-s3-volume), and more).

Starting a Dev-Mode Container

``` bash
docker run -dp 3000:3000 \
    -w /app -v "$(pwd):/app" \
    node:12-alpine \
    sh -c "yarn install && yarn run dev"
```

- `-dp 3000:3000` - same as before. Run in detached (background) mode and create a port mapping
- `-w /app` - sets the "working directory" or the current directory that the command will run from
- `-v "$(pwd):/app"` - bind mount the current directory from the host in the container into the /app directory
- `node:12-alpine` - the image to use. Note that this is the base image for our app from the Dockerfile
- `sh -c "yarn install && yarn run dev"` - the command. We're starting a shell using
  `sh` (alpine doesn't have `bash`) and running `yarn install` to install all dependencies and then running
  `yarn run dev`. If we look in the `package.json`,  we'll see that the `dev` script is starting `nodemon`.
  
#### Logs
Check the logs with
``` bash
docker logs -f <container-id>
```

#### Multi-Container Apps

Create the network.
``` bash
docker network create todo-app
```

Start a MySQL container and attach it to the network.
``` bash
docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
```

Verify database is running
``` bash
docker exec -it <mysql-container-id> mysql -p
```


#### Troubleshoot network problems with netshoot
Start a new container using the [nicolaka/netshoot](https://github.com/nicolaka/netshoot) image:
``` bash
docker run -it --network todo-app nicolaka/netshoot
```
Inside the container, we're going to use the dig command, which is a useful DNS tool.
We're going to look up the IP address for the hostname mysql.

``` bash
dig mysql
```

Let's start our dev-ready container!
``` bash
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

### Using Docker Compose
``` bash
docker-compose version
```

``` bash
docker-compose up -d
```
``` bash
docker-compose logs -f
```

``` bash
docker-compose down
```
``` bash
docker-compose down --volumes
``` 
