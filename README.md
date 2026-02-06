# stamp-webservices-docker
Docker composer container files for host the application in docker

Managing Configuration files to deploy to the server can be done by placing files in the config folder.  This is copied to /tmp/config on the container and after
extracting the archive, will copy the structure over in-place to the application root. 


Building a Docker Image

Using Podman run the following from the dockerfiles folder to create the docker image

```
podman build -t jenkins-node-jdk:nodejs-24-jdk-21 -f Dockerfile-nodejs24-jdk21
```