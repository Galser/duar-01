##  Working with Docker Containers

### Docker run

`docker` run is really a convenience command that wraps two separate steps into one. The first thing it does is create a container from the underlying image. We can accomplish this separately using the `docker create` command. The second thing docker run does is execute the container, which we can also do separately with the `docker start` command.

### Name

By default, Docker randomly names your container by combining an adjective with the name of a famous person. This results in names like ecstatic-babbage and serene- albattani. If you want to give your container a specific name, you can use the --name argument.

```
    $ docker create --name="awesome-service" ubuntu:latest sleep 120
```

After creating this container, you could then start it by using the `docker start awesome-service`. It will automatically exit after 120 seconds, but you could stop it before then by running `docker stop awesome-service.`

### Labels 

Labels are key/value pairs that can be applied to Docker images and containers as metadata. When new Docker containers are created, they automatically inherit all the labels from their parent image.

It is also possible to add new labels to the containers so that you can apply metadata that might be specific to that single container.

```
    docker run -d --name has-some-labels -l deployer=Ahmed -l tester=Asako \
      ubuntu:latest sleep 1000
```

You can then search for and filter containers based on this metadata, using com‐ mands like docker ps.

```
    $ docker ps -a -f label=deployer=Ahmed
CONTAINER ID   IMAGE           COMMAND        CREATED         STATUS         PORTS     NAMES
341858837ec5   ubuntu:latest   "sleep 1000"   9 seconds ago   Up 8 seconds             has-some-labels
```

You can use the docker inspect command on the container to see all the labels that a container has.

```
    $ docker inspect  341858837ec5 |
    ...
          "Labels": {
        "deployer": "Ahmed",
        "org.opencontainers.image.ref.name": "ubuntu",
        "org.opencontainers.image.version": "22.04",
        "tester": "Asako"
      }
    ...
```

 Note that this container runs the command sleep 1000, so after 1,000 seconds it will stop running.


 ### Hostname

By default, when you start a container, Docker copies certain system files on the host, including /etc/hostname, into the container’s configuration directory on the host,2 and then uses a bind mount to link that copy of the file into the container. We can launch a default container with no special configuration like this:

```
    $ docker run --rm -ti ubuntu:latest /bin/bash
```

This command uses the docker run command, which runs docker create and docker start in the background. Since we want to be able to interact with the con‐ tainer that we are going to create for demonstration purposes, we pass in a few useful arguments. The --rm argument tells Docker to delete the container when it exits, the -t argument tells Docker to allocate a pseudo-TTY, and the -i argument tells Docker
that this is going to be an interactive session, and we want to keep STDIN open. The final argument in the command is the executable that we want to run within the con‐ tainer, which in this case is the ever-useful `/bin/bash` .

 