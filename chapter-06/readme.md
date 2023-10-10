## Chapter 06, Exploring Docker


### Version and such

version 


```
docker version


Client:
 Cloud integration: v1.0.33
 Version:           24.0.2
 API version:       1.42 (downgraded from 1.43)
 Go version:        go1.20.4
 Git commit:        cb74dfc
 Built:             Thu May 25 21:51:16 2023
 OS/Arch:           darwin/arm64
 Context:           colima

Server:
 Engine:
  Version:          23.0.6
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.20.4
  Git commit:       9dbdbd4b6d7681bd18c897a6ba0376073c2a72ff
  Built:            Fri May 12 13:54:36 2023
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          v1.7.0
  GitCommit:        1fbd70374134b891f97ce19c70b6e50c7b9f4e0d
 runc:
  Version:          1.1.7
  GitCommit:        860f061b76bb4fc671f0f9e900f7d80ff93d4eb7
 docker-init:
  Version:          0.19.0
```

info 

```
 docker info
Client:
 Version:    24.0.2
 Context:    colima
 Debug Mode: false

Server:
 Containers: 12
  Running: 0
  Paused: 0
  Stopped: 12
 Images: 41
 Server Version: 23.0.6
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 1fbd70374134b891f97ce19c70b6e50c7b9f4e0d
 runc version: 860f061b76bb4fc671f0f9e900f7d80ff93d4eb7
 init version:
 Security Options:
  seccomp
   Profile: builtin
 Kernel Version: 6.1.29-0-virt
 Operating System: Alpine Linux v3.18
 OSType: linux
 Architecture: aarch64
 CPUs: 2
 Total Memory: 1.925GiB
 Name: colima
 ID: 5aa83b8a-1740-4cc4-ae17-764fa1bec923
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
 ```

### Downloading Image Updates

We’re going to use an Ubuntu base image for the following examples. Even if you have already grabbed the Ubuntu base image once, you can pull it again and it will automatically pick up any updates that have been published since you last ran it.


```
docker pull ubuntu:latest
latest: Pulling from library/ubuntu
Digest: sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054
Status: Image is up to date for ubuntu:latest
docker.io/library/ubuntu:latest
```

That command pulled down only the layers that have changed since we last ran the command.

It’s good to remember that even though you pulled latest, Docker won’t automatically keep the local image up to date for you. 

In addition to referring to items in the registry by the latest tag or another version number tag, you can also refer to them by their content-addressable tag. These look like sha256:2f9a...82cf (where we’ve shortened a very long ID). These are actually generated as a hashed sum of the contents of the image and are a very precise identi‐ fier. This is by far the safest way to refer to Docker images where you need to make sure you are getting the exact version you expected, because these can’t be moved like a version tag. The syntax for pulling them from the registry is very similar, but note the @ in the tag.
    

```
    docker pull ubuntu@sha256:2f9a...82cf
```


### Exploring containers

Let’s get a container running with just an interactive bash shell so we can take a look around. We’ll do that, as we did before, by just running something like:


```
    $ docker run -i -t ubuntu:16.04 /bin/bash
```

That will run an Ubuntu 16.04 LTS container with the bash shell as the top-level pro‐ cess. By specifying the 16.04 tag, we can be sure to get a particular version of the image. So, when we start that container, what processes are running?

```
docker run -i -t ubuntu:16.04 /bin/bash
Unable to find image 'ubuntu:16.04' locally
16.04: Pulling from library/ubuntu
828b35a09f0b: Pull complete
66927c6d1d3d: Pull complete
000560be9165: Pull complete
6225a0253717: Pull complete
Digest: sha256:1f1a2d56de1d604801a9671f301190704c25d604a416f59e03c04f5c6ffee0d6
Status: Downloaded newer image for ubuntu:16.04
root@04e41339d9ac:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 12:00 pts/0    00:00:00 /bin/bash
root        11     1  0 12:00 pts/0    00:00:00 ps -ef
```

Docker containers don’t, by default, start anything in the back‐ ground like a full virtual machine would. They’re a lot lighter weight than that and therefore don’t start an init system. You can, of course, run a whole init system if you need to, or Docker’s own tini init system, but you have to ask for it. 


### docker exec

 To get inside the container with docker exec. The command line for that, unsurprisingly, looks a lot like the command line for docker run. We request an interactive session and a pseudo-TTY with the -i and -t flags:
 
```
 docker exec -i -t 589f2ad30138 /bin/bash
    root@589f2ad30138:/#
```

### nsenter

Part of the core util-linux package from kernel.org is nsenter, short for “Name‐ space Enter,” which allows you to enter any Linux namespace. In Chapter 11, we’ll go into more detail on namespaces. But they are the core of what makes a container a container. Using nsenter, therefore, we can get into a Docker container from the server itself, even in situations where the dockerd server is not responding and we can’t use docker exec. nsenter can also be used to manipulate things in a container as root on the server that would otherwise be prevented by docker exec, for exam‐ ple. This can be really useful when you are debugging. Most of the time, docker exec is all you need, but you should have nsenter in your tool belt.
Most Linux distributions ship with the util-linux package that contains nsenter. But not all of them ship with one that is new enough to have nsenter itself installed, because it’s a recent addition to the package. Ubuntu 14.04, for example, still has util-linux from 2012. Ubuntu 16.04, however, has the newest package. If you are on a distribution that does not have it, the easiest way to get ahold of nsenter is to install it via a third-party Docker container.

This works by pulling a Docker image from the Docker Hub registry and then run‐ ning a specially crafted Docker container that will install the nsenter command-line tool into /usr/local/bin. This might seem strange at first, but it’s a clever way to allow you to install nsenter to any Docker server remotely using nothing more than the docker command.


MacOs `nsenter`

```
docker run -it --privileged --pid=host debian nsenter
```

This code shows how we install nsenter to /usr/local/bin on your Docker server:

```
    $ docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter
    Unable to find image 'jpetazzo/nsenter' locally
    Pulling repository jpetazzo/nsenter
    9e4ef84f476a: Download complete
    511136ea3c5a: Download complete
    71d9d77ae89e: Download complete
    Status: Downloaded newer image for jpetazzo/nsenter:latest
    Installing nsenter to /target
    Installing docker-enter to /target
```

You should be very careful about doing this! It’s always a good idea to check out what you are running, and particularly what you are exposing part of your filesystem to, before you run a third-party container on your system. With -v, we’re telling Docker to expose the host’s /usr/local/bin directory into the running container as /target. When the container starts, it is then copying an exe‐ cutable into that directory on our host’s filesystem. In Chapter 11, we will discuss some security frameworks and commands that can be leveraged to prevent potentially nefarious container activities.

Note - the above command will work **ONLY on Linux** 


MacOS test : 

```
 ~ docker exec -i -t 2a0e0fdc804d /bin/bash
root@2a0e0fdc804d:/# exit
➜  ~ PID=$(docker inspect --format \{{.State.Pid\}} 2a0e0fdc804d)
➜  ~ docker inspect --format \{{.State.Pid\}} 2a0e0fdc804d
3431
➜  ~ echo $PID
3431
➜  ~ sudo nsenter --target $PID --mount --uts --ipc --net --pid
Password:
sudo: nsenter: command not found
➜  ~ docker run -it --privileged --pid=host debian nsenter --target $PID --mount --uts --ipc --net --pid
# who
# ps
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
   22 pts/0    00:00:00 sh
   26 pts/0    00:00:00 ps
# host
-sh: 3: host: not found
# hostname
2a0e0fdc804d
```


### Logging


There are some caveats that apply to all of the logging drivers. For example, Docker supports only one at a time. This means that you can use the syslog or gelf logging driver, but not along with the json-file driver. Unless you run json-file or journald, you will lose the ability to use the docker logs command! T

One final caveat to be aware of regarding most of the logging plug-ins: they are block‐ ing by default, which means that logging back-pressure can cause issues with your application. You can change this behavior by setting --log-opt mode=non-blocking and then setting the maximum buffer size for logs to something like --log-opt max- buffer-size=4m. Once these are set, the application will no longer block when that buffer fills up. Instead, the oldest loglines in memory will be dropped. Again, reliabil‐ ity needs to be weighed here against your business need to receive all logs.



Finally, while you really should be capturing your logs somewhere, there are rare sit‐ uations where you simply don’t want any logging. You can turn them off completely using the --log-driver=none switch.

### Monitoring

Docker supports container health checks and some nice, basic reporting capabilities via docker stats and docker events.

- CLI

```
docker stats ef83a0135684
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT   MEM %     NET I/O       BLOCK I/O   PIDS
ef83a0135684   has-some-labels   0.00%     692KiB / 1.925GiB   0.03%     1.09kB / 0B   0B / 0B     1
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT   MEM %     NET I/O       BLOCK I/O   PIDS
```

Stats API endpoint

The /stats/ endpoint that we’ll hit on the API will continue to stream stats to us as long as we keep the connection open. 

Example for MacOS Colima :

- Start container
```
docker run -d ubuntu:latest sleep 1000
b6a02dc6645859539d60fa868b71da17fb400d21c33c8168dcae35064fcbaba7
➜  chapter-06 git:(chapter-06) ✗ docker ps
CONTAINER ID   IMAGE           COMMAND        CREATED          STATUS          PORTS     NAMES
b6a02dc66458   ubuntu:latest   "sleep 1000"   34 seconds ago   Up 32 seconds             nifty_kapitsa
ef83a0135684   ubuntu:latest   "sleep 1000"   4 minutes ago    Up 4 minutes              has-some-labels
```

- Get stats via API
```
curl --unix-socket ///Users/andrii/.colima/default/docker.sock \
        http://v1/containers/b6a02dc66458/stats
{"read":"2023-10-10T11:37:04.377567793Z","preread":"0001-01-01T00:00:00Z","pids_stats":{"current":1},"blkio_stats":{"io_service_bytes_recursive":[],"io_serviced_recursive":[],"io_queue_recursive":[],"io_service_time_recursive":[],"io_wait_time_recursive":[],"io_merged_recursive":[],"io_time_recursive":[],"sectors_recursive":[]},"num_procs":0,"storage_stats":{},"cpu_stats":{"cpu_usage":{"total_usage":15701581,"percpu_usage":[12905875,2795706],"usage_in_kernelmode":10000000,"usage_in_usermode":0},"system_cpu_usage":46136140000000,"online_cpus":2,"throttling_data":{"periods":0,"throttled_periods":0,"throttled_time":0}},"precpu_stats":{"cpu_usage":{"total_usage":0,"usage_in_kernelmode":0,"usage_in_usermode":0},"throttling_data":{"periods":0,"throttled_periods":0,"throttled_time":0}},"memory_stats":{"usage":450560,"max_usage":6381568,"stats":{"active_anon":86016,"active_file":0,"cache":0,"dirty":0,"hierarchical_memory_limit":9223372036854771712,"hierarchical_memsw_limit":9223372036854771712,"inactive_anon":4096,"inactive_file":0,"mapped_file":0,"pgfault":987,"pgmajfault":0,"pgpgin":515,"pgpgout":493,"rss":90112,"rss_huge":0,"total_active_anon":86016,"total_active_file":0,"total_cache":0,"total_dirty":0,"total_inactive_anon":4096,"total_inactive_file":0,"total_mapped_file":0,"total_pgfault":987,"total_pgmajfault":0,"total_pgpgin":515,"total_pgpgout":493,"total_rss":90112,"total_rss_huge":0,"total_unevictable":0,"total_writeback":0,"unevictable":0,"writeback":0},"limit":2066771968},"name":"/nifty_kapitsa","id":"b6a02dc6645859539d60fa868b71da17fb400d21c33c8168dcae35064fcbaba7","networks":{"eth0":{"rx_bytes":796,"rx_packets":10,"rx_errors":0,"rx_dropped":0,"tx_bytes":0,"tx_packets":0,"tx_errors":0,"tx_dropped":0}}}
{"read":"2023-10-10T11:37:05.38712971Z","preread":"2023-10-10T11:37:04.377567793Z","pids_stats":{"current":1},"blkio_stats":{"io_service_bytes_recursive":[],"io_serviced_recursive":[],"io_queue_recursive":[],"io_service_time_recursive":[],"io_wait_time_recursive":[],"io_merged_recursive":[],"io_time_recursive":[],"sectors_recursive":[]},"num_procs":0,"storage_stats":{},"cpu_stats":{"cpu_usage":{"total_usage":15701581,"percpu_usage":[12905875,2795706],"usage_in_kernelmode":10000000,"usage_in_usermode":0},"system_cpu_usage":46138140000000,"online_cpus":2,"throttling_data":{"periods":0,"throttled_periods":0,"throttled_time":0}},"precpu_stats":{"cpu_usage":{"total_usage":15701581,"percpu_usage":[12905875,2795706],"usage_in_kernelmode":10000000,"usage_in_usermode":0},"system_cpu_usage":46136140000000,"online_cpus":2,"throttling_data":{"periods":0,"throttled_periods":0,"throttled_time":0}},"memory_stats":{"usage":450560,"max_usage":6381568,"stats":{"active_anon":86016,"active_file":0,"cache":0,"dirty":0,"hierarchical_memory_limit":9223372036854771712,"hierarchical_memsw_limit":9223372036854771712,"inactive_anon":4096,"inactive_file":0,"mapped_file":0,"pgfault":987,"pgmajfault":0,"pgpgin":515,"pgpgout":493,"rss":90112,"rss_huge":0,"total_active_anon":86016,"total_active_file":0,"total_cache":0,"total_dirty":0,"total_inactive_anon":4096,"total_inactive_file":0,"total_mapped_file":0,"total_pgfault":987,"total_pgmajfault":0,"total_pgpgin":515,"total_pgpgout":493,"total_rss":90112,"total_rss_huge":0,"total_unevictable":0,"total_writeback":0,"unevictable":0,"writeback":0},"limit":2066771968},"name":"/nifty_kapitsa","id":"b6a02dc6645859539d60fa868b71da17fb400d21c33c8168dcae35064fcbaba7","networks":{"eth0":{"rx_bytes":866,"rx_packets":11,"rx_errors":0,"rx_dropped":0,"tx_bytes":0,"tx_packets":0,"tx_errors":0,"tx_dropped":0}}}
```

### Health-checks 

To help remove the complexity and standardize on a universal interface, Docker has added a health-check mechanism. Following the shipping container metaphor, Docker containers should really look the same to the outside world no matter what is inside the container, so Docker’s health-check mechanism not only standardizes health checking for containers, but also maintains the isolation between what is inside the container and what it looks like on the outside. This means that containers from Docker Hub or other shared repositories can implement a standardized health- checking mechanism and it will work in any other Docker environment designed to run production containers.
Health checks are a build-time configuration item and are created with a check defi‐ nition in the Dockerfile. This directive tells the Docker daemon what command it can run inside the container to ensure the container is in a healthy state. As long as the command exits with a code of zero (0), Docker will consider the container to be healthy. Any other exit code will indicate to Docker that the container is not in a healthy state, at which point appropriate action can be taken by a scheduler or moni‐ toring system.
We will be using the following project to explore Docker Compose in a few chapters. But, for the moment, it includes a useful example of Docker health checks. Go ahead and pull down a copy of the code and then navigate into the rocketchat-hubot-demo/ mongodb/docker/ directory:

$ git clone https://github.com/spkane/rocketchat-hubot-demo.git \
        --config core.autocrlf=input
$ cd rocketchat-hubot-demo/mongodb/docker

In this directory, you will see a Dockerfile and a script called docker-healthcheck. If
you view the Dockerfile, this is all that you will see: FROM mongo:3.2

    COPY docker-healthcheck /usr/local/bin/
    HEALTHCHECK CMD ["docker-healthcheck"]

It is very short because we are basing this on the upstream Mongo image, and our image inherits a lot of things from that including the entry point, default command, and port to expose:

ENTRYPOINT ["docker-entrypoint.sh"] EXPOSE 27017
CMD ["mongod"]

So, in our Dockerfile we are only adding a single script that can health-check our con‐ tainer, and defining a health-check command that runs that script.

Let's build it : 

```
docker build -t mongo-with-check:3.2  .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM docker.io/bitnami/mongodb:4.4
4.4: Pulling from bitnami/mongodb
fb7104aeaae7: Pull complete
e26dd7af2d9c: Pull complete
ba6183ce7f3b: Pull complete
3a5f1a3b97b3: Pull complete
2090d56b0713: Pull complete
d4f232eb3fab: Pull complete
5e40ac1fbeb7: Pull complete
a297f00fc847: Pull complete
69007df9d9b1: Pull complete
57fbe5f31afd: Pull complete
5fe0ef0f6380: Pull complete
171c11113e79: Pull complete
d81acf1c0f2d: Pull complete
Digest: sha256:916202d7af766dd88c2fff63bf711162c9d708ac7a3ffccd2aa812e3f03ae209
Status: Downloaded newer image for bitnami/mongodb:4.4
 ---> 9317984d7b90
Step 2/3 : COPY docker-healthcheck /usr/local/bin/
 ---> b6b9de260984
Step 3/3 : HEALTHCHECK CMD ["docker-healthcheck"]
 ---> [Warning] The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
 ---> Running in 81e1b4ce623c
Removing intermediate container 81e1b4ce623c
 ---> 6b06eafb1dd2
Successfully built 6b06eafb1dd2
Successfully tagged mongo-with-check:3.2
```

Run : 

```
➜  docker git:(main) docker run -d --name mongo-hc mongo-with-check:3.2
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
0d0568f01711277677b4d7f5bdc049d588ab4cdf996b3f4670208032eb3dbbfa
```

And - health check

```
➜  docker git:(main) docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                            PORTS       NAMES
0d0568f01711   mongo-with-check:3.2   "/opt/bitnami/script…"   8 seconds ago    Up 6 seconds (health: starting)   27017/tcp   mongo-hc
b6a02dc66458   ubuntu:latest          "sleep 1000"             10 minutes ago   Up 10 minutes                                 nifty_kapitsa
ef83a0135684   ubuntu:latest          "sleep 1000"             14 minutes ago   Up 14 minutes                                 has-some-labels
```

See "health:starting". Now, after a minute it looks like : 

```
docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED              STATUS                        PORTS       NAMES
0d0568f01711   mongo-with-check:3.2   "/opt/bitnami/script…"   About a minute ago   Up About a minute (healthy)   27017/tcp   mongo-hc
b6a02dc66458   ubuntu:latest          "sleep 1000"             11 minutes ago       Up 11 minutes                             nifty_kapitsa
ef83a0135684   ubuntu:latest          "sleep 1000"             16 minutes ago       Up 15 minutes

```


You can query this status directly, using the docker inspect command. 

```
$ docker inspect --format='{{.State.Health.Status}}' mongo-hc
healthy
```

or : 

```
    $ docker inspect --format='{{json .State.Health}}' mongo-hc | jq
    {
"Status": "healthy", "FailingStreak": 0, "Log": [
...
] }
```

### Docker Events

The dockerd daemon internally generates an events stream around the container life‐ cycle. This is how various parts of the system find out what is going on in other parts. You can also tap into this stream to see what lifecycle events are happening for con‐ tainers on your Docker server. This, as you probably expect by now, is implemented in the docker CLI tool as another command-line argument. When you run this com‐ mand, it will block and continually stream messages to you. Behind the scenes, this is a long-lived HTTP request to the Docker API that returns messages in JSON blobs as they occur. The docker CLI tool decodes them and prints some data to the terminal.
This events stream is useful in monitoring scenarios or in triggering additional actions, like wanting to be alerted when a job completes. For debugging purposes, it allows you to see when a container died even if Docker restarts it later. Down the road, this is a place where you might also find yourself directly implementing some tooling against the API. Here’s how we use it on the command line:
    
```
    $ docker events
    2018-02-18T14:00:39-08:00 1b3295bf300f: (from 0415448f2cc2) die
    2018-02-18T14:00:39-08:00 1b3295bf300f: (from 0415448f2cc2) stop
```    



### cAdvisor
docker stats and docker events are useful but don’t yet get us graphs to look at. And graphs are pretty helpful when we’re trying to see trends. Of course, other people have filled some of this gap. When you begin to explore the options for monitoring Docker, you will find that many of the major monitoring tools now provide some functionality to help you improve the visibility into your containers’ performance and ongoing state.

In addition to the commercial tooling provided by companies like DataDog, Ground‐ Work, and New Relic, there are plenty of options for free, open source tools like Prometheus or even Nagios. We’ll talk about Prometheus in the next section, but first we’ll start with a nice offering from Google. A few years ago they released their own internal container advisor as a well-maintained open source project on GitHub, called cAdvisor. Although cAdvisor can be run outside of Docker, by now you’re probably not surprised to hear that the easiest implementation of cAdvisor is to simply run it as a Docker container.
To install cAdvisor on most Linux systems, all you need to do is run this code:

```    
    $ docker run \
        --volume=/:/rootfs:ro \
        --volume=/var/run:/var/run:rw \
        --volume=/sys:/sys:ro \
        --volume=/var/lib/docker/:/var/lib/docker:ro \
        --publish=8080:8080 \
        --detach=true \
        --name=cadvisor \
        google/cadvisor:latest
    Unable to find image 'google/cadvisor:latest' locally
    Pulling repository google/cadvisor
    f0643dafd7f5: Download complete
    ...
    ba9b663a8908: Download complete
    Status: Downloaded newer image for google/cadvisor:latest
    f54e6bc0469f60fd74ddf30770039f1a7aa36a5eda6ef5100cddd9ad5fda350b
```

    
On RHEL and CentOS-based systems, you will need to add the fol‐ lowing line to the docker run command shown here: --volume=/ cgroup:/cgroup \.
Once you have done this, you will be able to navigate to your Docker host on port 8080 to see the cAdvisor web interface (i.e., http://172.17.42.10:8080/) and the various detailed charts it has for the host and individual containers 