# Building a Simple App with Docker 
This project uses Docker to build a simple app to prove out an MVP (minimum viable product).


Build the container
``` bash 
docker build -t simple-task-manager .
```

Run Container
``` bash 
docker run -dp 3000:3000 simple-task-manager
```

Remember the -d and -p flags? We're running the new container in "detached" mode (in the background) and creating
a mapping between the host's port 3000 to the container's port 3000. Without the port mapping, we wouldn't be able
to access the application.

Stop and remove
``` bash
docker ps
docker rm -f <the-container-id>
```
Pushing  Image
Login to Docker Hub
``` bash
docker login -u YOUR-USER-NAME
```

Change name
``` bash
docker tag getting-started YOUR-USER-NAME/simple-task-manager
```

push
``` bash
docker push YOUR-USER-NAME/simple-task-manager
```

[Play with Docker](https://labs.play-with-docker.com/)


In this section, we learned how to share our images by pushing them to a registry. We then went to a brand 
new instance and were able to run the freshly pushed image. This is quite common in CI pipelines, where the 
pipeline will create the image and push it to a registry and then the production environment can use the latest version of the image.


file system 
``` bash
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

``` bash
docker exec <container-id> cat /data.txt
```

Named volumes
``` bash
docker volume create todo-db
```
Mount volume
``` bash
docker run -dp 3000:3000 -v todo-db:/etc/todos simple-task-manager
```

Diving into our Volume

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
  

Check the logs with
``` bash
docker logs -f <container-id>
```


Using bind mounts is very common for local development setups. The advantage is that the dev machine doesn't need to
have all of the build tools and environments installed. With a single docker run command, the dev environment is pulled
and ready to go. We'll talk about Docker Compose in a future step, as this will help simplify our commands
(we're already getting a lot of flags).

Multi-Container Apps

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

Start a new container using the [nicolaka/netshoot](https://github.com/nicolaka/netshoot) image
``` bash
docker run -it --network todo-app nicolaka/netshoot
```
Inside the container, we're going to use the dig command, which is a useful DNS tool.
We're going to look up the IP address for the hostname mysql.

``` bash
dig mysql
```

let's start our dev-ready container!
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
