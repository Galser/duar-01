# duar-01
DUAR - Docker Up And Running - 01. Follow-along exercises from the book


>> note - I am runniong Docker via Colima on MacOS 

# Docker-node-hello

Steps : 

- git clone https://github.com/spkane/docker-node-hello.git \
        --config core.autocrlf=input
- cd docker-node-hello
- docker build -t example/docker-node-hello:latest .

Output:
```
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  9.728kB
Step 1/15 : FROM docker.io/node:18.13.0
18.13.0: Pulling from library/node
c345c9e441f5: Pull complete
7b716680367d: Pull complete
0855378f8903: Pull complete
4bfb8dc93d41: Pull complete
fb726ea60d28: Pull complete
02f41717b6ae: Pull complete
6d99896e8af9: Pull complete
40cff91b82ae: Pull complete
5301fac16292: Pull complete
Digest: sha256:d871edd5b68105ebcbfcde3fe8c79d24cbdbb30430d9bd6251c57c56c7bd7646
Status: Downloaded newer image for node:18.13.0
 ---> b0cef62e0901
Step 2/15 : ARG email="anna@example.com"
 ---> Running in f835656868d4
Removing intermediate container f835656868d4
 ---> 37b68867a20a
Step 3/15 : LABEL "maintainer"=$email
 ---> Running in ffd0f1b47bca
Removing intermediate container ffd0f1b47bca
 ---> 5f4294748416
Step 4/15 : LABEL "rating"="Five Stars" "class"="First Class"
 ---> Running in ba529f7ce42f
Removing intermediate container ba529f7ce42f
 ---> 584905c0c382
Step 5/15 : USER root
 ---> Running in fdcf942a0be4
Removing intermediate container fdcf942a0be4
 ---> d3a37653196b
Step 6/15 : ENV AP /data/app
 ---> Running in 65cb85554000
Removing intermediate container 65cb85554000
 ---> 59533537d23a
Step 7/15 : ENV SCPATH /etc/supervisor/conf.d
 ---> Running in e9634efca745
Removing intermediate container e9634efca745
 ---> 14713ba595f9
Step 8/15 : RUN apt-get -y update
 ---> Running in b0dd57d3cb3f
Get:1 http://deb.debian.org/debian bullseye InRelease [116 kB]
Get:2 http://deb.debian.org/debian-security bullseye-security InRelease [48.4 kB]
Get:3 http://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]
Get:4 http://deb.debian.org/debian bullseye/main arm64 Packages [8071 kB]
Get:5 http://deb.debian.org/debian-security bullseye-security/main arm64 Packages [242 kB]
Get:6 http://deb.debian.org/debian bullseye-updates/main arm64 Packages [14.9 kB]
Fetched 8535 kB in 3s (2931 kB/s)
Reading package lists...
Removing intermediate container b0dd57d3cb3f
 ---> d36e6b0c1bc7
Step 9/15 : RUN apt-get -y install supervisor
 ---> Running in 4969495932e5
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  python3-pkg-resources
Suggested packages:
  python3-setuptools supervisor-doc
The following NEW packages will be installed:
  python3-pkg-resources supervisor
0 upgraded, 2 newly installed, 0 to remove and 71 not upgraded.
Need to get 499 kB of archives.
After this operation, 2320 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main arm64 python3-pkg-resources all 52.0.0-4 [190 kB]
Get:2 http://deb.debian.org/debian bullseye/main arm64 supervisor all 4.2.2-2 [309 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 499 kB in 0s (1894 kB/s)
Selecting previously unselected package python3-pkg-resources.
(Reading database ... 22781 files and directories currently installed.)
Preparing to unpack .../python3-pkg-resources_52.0.0-4_all.deb ...
Unpacking python3-pkg-resources (52.0.0-4) ...
Selecting previously unselected package supervisor.
Preparing to unpack .../supervisor_4.2.2-2_all.deb ...
Unpacking supervisor (4.2.2-2) ...
Setting up python3-pkg-resources (52.0.0-4) ...
Setting up supervisor (4.2.2-2) ...
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Removing intermediate container 4969495932e5
 ---> 0fb2eeb6dbdf
Step 10/15 : RUN mkdir -p /var/log/supervisor
 ---> Running in cd14ebb98b54
Removing intermediate container cd14ebb98b54
 ---> e3dc927384db
Step 11/15 : COPY ./supervisord/conf.d/* $SCPATH/
 ---> 234c0311057b
Step 12/15 : COPY *.js* $AP/
 ---> 24bde9eb6198
Step 13/15 : WORKDIR $AP
# duar-01
 ---> Running in 7d4e6f073c00
Removing intermediate container 7d4e6f073c00
 ---> 6cebe226d123
Step 14/15 : RUN npm install
 ---> Running in a6ea84655483
npm WARN EBADENGINE Unsupported engine {
npm WARN EBADENGINE   package: 'formidable@1.0.13',
npm WARN EBADENGINE   required: { node: '<0.9.0' },
npm WARN EBADENGINE   current: { node: 'v18.13.0', npm: '8.19.3' }
npm WARN EBADENGINE }
npm WARN deprecated mkdirp@0.3.4: Legacy versions of mkdirp are no longer supported. Please update to mkdirp 1.x. (Note that the API surface has changed to use Promises in 1.x.)
npm WARN deprecated formidable@1.0.13: Please upgrade to latest, formidable@v2 or formidable@v3! Check these notes: https://bit.ly/2ZEqIau
npm WARN deprecated connect@2.7.9: connect 2.x series is deprecated

added 18 packages, and audited 19 packages in 3s

7 vulnerabilities (2 moderate, 5 high)

To address all issues, run:
  npm audit fix --force

Run `npm audit` for details.
npm notice
npm notice New major version of npm available! 8.19.3 -> 10.0.0
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v10.0.0>
npm notice Run `npm install -g npm@10.0.0` to update!
npm notice
Removing intermediate container a6ea84655483
 ---> ef353ccee4a4
Step 15/15 : CMD ["supervisord", "-n"]
 ---> Running in 9a0ebcd7b9a1
Removing intermediate container 9a0ebcd7b9a1
 ---> cbc04243a50d
Successfully built cbc04243a50d
Successfully tagged example/docker-node-hello:latest
```

- docker run -d -p 8080:8080 example/docker-node-hello:latest

```
docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED         STATUS         PORTS                                       NAMES
48c58b8391b7   example/docker-node-hello:latest   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   stupefied_heyrovsky
```

Let's check it in browser :

<img width="272" alt="image" src="https://github.com/Galser/duar-01/assets/914404/3dd3dd17-dffd-4690-838c-635dc9cac632">


## Env vars at run 

```
docker run -d -p 8080:8080 -e WHO="Andrii" \
        example/docker-node-hello:latest
```

## Tagging before upload to public registry

- Post-build : 

```
docker tag example/docker-node-hello:latest \
        ${<myuser>}/docker-node-hello:latest
```

- or during built : 

```
docker build -t ${<myuser>}/docker-node-hello:latest .
```

but that presumed Linux + Bash Docker installation. 

So for modern M1/M2 hardware MacOS-based install, running Colima correct way is to defiend your won env variable and proceed with it : 

```
export DOCKER_HUB_USER=XUSERDOCKER
docker tag example/docker-node-hello:latest \
        ${DOCKER_HUB_USER}/docker-node-hello:latest
```

Let's check it : 

```
 docker images
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
example/docker-node-hello     latest    cbc04243a50d   23 hours ago   973MB
XUSERDOCKER/docker-node-hello   latest    cbc04243a50d   23 hours ago   973MB
node                          18.13.0   b0cef62e0901   8 months ago   945MB
```

## Pushin to a public Docker repo

Upload the image to the Docker repository by using the docker push command, classic instalaltion : 

```
$ docker push ${<myuser>}/docker-node-hello:latest
```

My option : 


```
docker push ${DOCKER_HUB_USER}/docker-node-hello:latest
The push refers to repository [docker.io/XUSERDOCKER/docker-node-hello]
728213049d97: Pushed
392ecd7d2512: Pushed
7c9112f9a9d7: Pushed
d9ac0a0c2340: Pushed
902e1d792777: Pushed
a69fed3eae3a: Mounted from library/node
cce7d50540ad: Mounted from library/node
0c176246eb1d: Mounted from library/node
1e80eaad2549: Mounted from library/node
15b65600aa80: Mounted from library/node
85f9ebffaf4d: Mounted from library/node
72235aad06ad: Mounted from library/node
5d37ad02a8e2: Mounted from library/node
ea8ab45f064e: Mounted from library/node
latest: digest: sha256:4afe2d1d903164fdc80d11d6c3a58480f690bab00af2c0a01ecf8028383d643f size: 3264
```

## Keeping Images Small 

Demo : 
```
docker run -d -p 8080:8080 adejonge/helloworld
```

Which ..fails on my arc, tested it in another place.\

But - interesting to look around another , classic minimal image : 

```
docker run -ti alpine:latest /bin/sh
```

Reminder , Since we are using Docker Community Edition, we need to use our nsenter trick to enter the SSH-less virtual machine and explore the filesystem.

```
$ docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

##  Layers are additive 
Something that is not apparent until you dig much deeper into how images are built is that the filesystem layers that make up your images are strictly additive in nature.


So image created from this Dockerfile : 

```
FROM fedora
RUN dnf install -y httpd && \
    dnf clean all
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```


Going to be smaller than : 

```
FROM fedora
RUN dnf install -y httpd
RUN dnf clean all
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

due to fact that DNF cache was already ADDED .. remmeber? Layers are additive. 


Here is how `docker history` for the optimized`image (frist one) would look like : 

```
docker history size3
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
0168584dae0b   30 seconds ago   /bin/sh -c #(nop)  CMD ["/usr/sbin/httpd" "-…   0B
f23e2484f5ee   30 seconds ago   /bin/sh -c dnf install -y httpd &&     dnf c…   79.6MB
f838063d816c   4 weeks ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      4 weeks ago      /bin/sh -c #(nop) ADD file:50785841137152691…   274MB
<missing>      10 months ago    /bin/sh -c #(nop)  ENV DISTTAG=f38container …   0B
<missing>      10 months ago    /bin/sh -c #(nop)  LABEL maintainer=Clement …   0B
```

## Optimizing build sepped ( opt for Docker cache)

Detailed : [cache-optimize-01/README.MD]



