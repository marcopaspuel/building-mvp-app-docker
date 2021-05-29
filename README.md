# building-mvp-app-docker
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