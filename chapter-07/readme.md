## Process Output 

One of the first things you’ll want to know when debugging a container is what is running inside it. Docker has a built-in command for doing that: docker top

To run docker top, we need to pass it the ID of our container, so we get that from docker ps. We then pass that to docker top and get a nice listing of what is running inside our container, ordered by PID just as we’d expect from Linux ps output.
There are some oddities here, though. The primary one is namespacing of user IDs and filesystems.
For example, a user might exist in a container’s /etc/passwd directory, but that same user might not exist simultaneously on the host machine, or have an entirely different user ID on the host machine versus inside the container. In the case where that user is running a process in a container, the ps output on the host machine will show a numeric ID rather than a username. In some cases, two containers might have users squatting on the same numeric ID, or mapping to an ID that is a completely different user on the host system. For example, if haproxy were installed on the host system, it would be possible for nginx in the container to appear to be running as the haproxy user on the host.

```
 docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED        STATUS                  PORTS       NAMES
0d0568f01711   mongo-with-check:3.2   "/opt/bitnami/script…"   44 hours ago   Up 44 hours (healthy)   27017/tcp   mongo-hc

➜  duar-01 git:(chapter-07) docker top 0d0568f01711
PID                 USER                TIME                COMMAND
4182                1001                34:46               {mongod} /usr/bin/qemu-x86_64 /opt/bitnami/mongodb/bin/mongod /opt/bitnami/mongodb/bin/mongod --config=/opt/bitnami/mongodb/conf/mongodb.conf
```

Likewise, because the process has a different view of the filesystem, paths that are shown in the ps output are relative to the container and not the host. In these cases, knowing it is in a container is a big win.

For the case where colima pid  is equal to "34205" ( or for Linux we may use "pstree -p `pidof dockerd` ")

```
 pstree 34205
-+- 34205 andrii /opt/homebrew/bin/limactl hostagent --pidfile /Users/andrii/.lima/colima/ha.pid --socket /Users/andrii/.lima/colima
 |-+- 34229 andrii /Users/andrii/.colima/_wrapper/3a9197e1ca3cd2da076da2b473d7a7eb118e2cca/bin/qemu-system-aarch64 -m 2048 -cpu host
 | \--- 34233 andrii qemu-system-aarch64 -m 2048 -cpu host -machine virt,accel=hvf,highmem=off -smp 2,sockets=1,cores=2,threads=1 -d
 |--- 34425 andrii ssh -F /dev/null -o IdentityFile="/Users/andrii/.lima/_config/user" -o IdentityFile="/Users/andrii/.ssh/cloud" -o
 |--- 34426 andrii /usr/libexec/sftp-server -e -d /Users/andrii
 |--- 34429 andrii ssh -F /dev/null -o IdentityFile="/Users/andrii/.lima/_config/user" -o IdentityFile="/Users/andrii/.ssh/cloud" -o
 \--- 34430 andrii /usr/libexec/sftp-server -e -d /tmp/colima
```

## Controlling processes

Keep in mind that containers are supposed to look to the outside world like a single bundle. If you need to kill off something inside the container other than for debug‐ ging purposes, it’s best to replace the whole container. 


### WHAT???? 

> Because containers work just like any other process, it’s important to understand how they can interact with your application in a less helpful way."


What this suppose to mean? 


There are some special needs in a container for processes that spawn background children—that is, anything that forks and daemonizes so the parent no longer manages the child process lifecy‐ cle. Jenkins build containers are one common example where people see this go wrong. When daemons fork into the background, they become children of PID 1 on Unix systems. Process 1 is special and is usually an init process of some kind.
PID 1 is responsible for making sure that children are reaped. In your container, by default your main process will be PID 1. Since you probably won’t be handling reap‐ ing of children from your application, you can end up with zombie processes in your container. There are a few solutions to this problem. The first is to run an init system in the container of your own choosing—one that is capable of handling PID 1 responsibilities. s6, runit, and others described in the preceding Note can be easily used inside the container.
But Docker itself provides an even simpler option that solves just this one case without taking on all the capabilities of a full init system. If you provide the --init flag to docker run, Docker will launch a very small init process based on the tini project that will act as PID 1 inside the container on startup. Whatever you specify in your Dockerfile as the CMD is passed to tini and otherwise works in the same way you would expect. It does, however, replace anything you might have in the ENTRYPOINT section of your Dockerfile.


## Network Inspection

Compared to process inspection, debugging containerized applications at the net‐ work level can be more complicated. 


```
  $ netstat -anp
    Active Internet connections (servers and established)
Proto ... Local Address
tcp   ... 10.0.3.1:53
tcp   ... 0.0.0.0:22
tcp6  ... :::23235
tcp6  ... :::2375
Foreign Address State  PID/Program name
tcp6  ... :::4243
tcp6  ... fe80::389a:46ff:fe92:53 :::*
tcp6  ... :::22
udp   ... 10.0.3.1:53
udp   ... 0.0.0.0:67
udp   ... 0.0.0.0:68
udp6  ... fe80::389a:46ff:fe92:53 :::*
0.0.0.0:*
0.0.0.0:*
:::*
:::*
:::*
LISTEN 23861/dnsmasq
LISTEN 902/sshd
LISTEN 24053/docker-proxy
LISTEN 954/docker
LISTEN 954/docker
LISTEN 23861/dnsmasq
LISTEN 902/sshd
       23861/dnsmasq
       23861/dnsmasq
       880/dhclient3
       23861/dnsmasq
:::*
0.0.0.0:*
0.0.0.0:*
0.0.0.0:*
```

We see the same output, but notice what is bound to the port: docker-proxy. That’s because, in its default configuration, Docker actually has a proxy written in Go that sits between all of the containers and the outside world. That means that when we look at output, we see only docker-proxy. Notice that there is no clue here about which container that docker-proxy is handling. Luckily, docker ps shows us which containers are bound to which ports, so this isn’t a big deal. 

docker compose or other tools - can though change those networks . 


## Image inspections

```
 docker history 6b06eafb1dd2
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
6b06eafb1dd2   44 hours ago    /bin/bash -o pipefail -c #(nop)  HEALTHCHECK…   0B
b6b9de260984   44 hours ago    /bin/bash -o pipefail -c #(nop) COPY file:4c…   192B
9317984d7b90   15 months ago   /bin/bash -o pipefail -c #(nop)  CMD ["/opt/…   0B
<missing>      15 months ago   /bin/bash -o pipefail -c #(nop)  ENTRYPOINT …   0B
<missing>      15 months ago   /bin/bash -o pipefail -c #(nop)  USER 1001      0B
<missing>      15 months ago   /bin/bash -o pipefail -c #(nop)  EXPOSE 27017   0B
<missing>      15 months ago   /bin/bash -o pipefail -c #(nop)  ENV APP_VER…   0B
<missing>      15 months ago   /bin/bash -o pipefail -c /opt/bitnami/script…   1.03kB
<missing>      15 months ago   /bin/bash -o pipefail -c #(nop) COPY dir:481…   66.5kB
<missing>      15 months ago   /bin/bash -o pipefail -c chmod g+rwX /opt/bi…   0B
<missing>      15 months ago   /bin/bash -o pipefail -c apt-get update && a…   0B
<missing>      15 months ago   /bin/bash -o pipefail -c . /opt/bitnami/scri…   2.3MB
<missing>      15 months ago   /bin/bash -o pipefail -c . /opt/bitnami/scri…   339MB
<missing>      15 months ago   /bin/bash -o pipefail -c . /opt/bitnami/scri…   2.6MB
<missing>      15 months ago   /bin/bash -o pipefail -c . /opt/bitnami/scri…   2.05MB
<missing>      15 months ago   /bin/bash -o pipefail -c . /opt/bitnami/scri…   10.2MB
<missing>      15 months ago   /bin/bash -o pipefail -c . /opt/bitnami/scri…   160MB
<missing>      15 months ago   /bin/bash -o pipefail -c install_packages ac…   14.4MB
<missing>      15 months ago   /bin/bash -o pipefail -c #(nop)  SHELL [/bin…   0B
<missing>      15 months ago   /bin/sh -c #(nop) COPY dir:183139dde6a77af32…   65.8kB
<missing>      15 months ago   /bin/sh -c #(nop)  ENV HOME=/ OS_ARCH=amd64 …   0B
<missing>      15 months ago                                                   67.5MB    from Bitnami with love
```

And with no truncate : 

```
➜  duar-01 git:(chapter-07) ✗ docker history 6b06eafb1dd2 --no-trunc
IMAGE                                                                     CREATED         CREATED BY                                                                                                                                                                                                                                                                                                                                                                                                           SIZE      COMMENT
sha256:6b06eafb1dd20d7588f6b83784e2dbf4f9ee05c9278d6a5397aaf1054ec83d77   44 hours ago    /bin/bash -o pipefail -c #(nop)  HEALTHCHECK &{["CMD" "docker-healthcheck"] "0s" "0s" "0s" '\x00'}                                                                                                                                                                                                                                                                                                                   0B
sha256:b6b9de260984bc9351d88946a99fd0bf86b1b0381ee217ea037593269bce7808   44 hours ago    /bin/bash -o pipefail -c #(nop) COPY file:4c9f6f9a624fb6444f7199a9dc5b91a165587631c14a16e9378161d4cd1769f7 in /usr/local/bin/                                                                                                                                                                                                                                                                                        192B
sha256:9317984d7b909791cc62168f3aa9099457649f4aa0989fbf1d5ab778566e8277   15 months ago   /bin/bash -o pipefail -c #(nop)  CMD ["/opt/bitnami/scripts/mongodb/run.sh"]
```

## Filesystem inspection

For a runnin container : `docker diff <CONTAINER_ID>`

```
docker diff 0d0568f01711
C /.mongodb
A /.mongodb/mongosh
A /.mongodb/mongosh/65253942bbccf82cef003025_log
A /.mongodb/mongosh/config
C /opt
C /opt/bitnami
C /opt/bitnami/mongodb
C /opt/bitnami/mongodb/tmp
A /opt/bitnami/mongodb/tmp/mongodb-27017.sock
A /opt/bitnami/mongodb/tmp/mongodb.pid
C /opt/bitnami/mongodb/conf
C /opt/bitnami/mongodb/conf/mongodb.conf
C /bitnami
C /bitnami/mongodb
C /bitnami/mongodb/data
A /bitnami/mongodb/data/db
A /bitnami/mongodb/data/db/collection-4-6450241117759633738.wt
A /bitnami/mongodb/data/db/journal
A /bitnami/mongodb/data/db/journal/WiredTigerPreplog.0000000001
A /bitnami/mongodb/data/db/journal/WiredTigerPreplog.0000000002
A /bitnami/mongodb/data/db/journal/WiredTigerLog.0000000002
A /bitnami/mongodb/data/db/mongod.lock
A /bitnami/mongodb/data/db/storage.bson
A /bitnami/mongodb/data/db/collection-2-6450241117759633738.wt
A /bitnami/mongodb/data/db/WiredTiger.wt
A /bitnami/mongodb/data/db/WiredTigerHS.wt
A /bitnami/mongodb/data/db/index-5-6450241117759633738.wt
A /bitnami/mongodb/data/db/index-6-6450241117759633738.wt
A /bitnami/mongodb/data/db/sizeStorer.wt
A /bitnami/mongodb/data/db/WiredTiger
A /bitnami/mongodb/data/db/diagnostic.data
A /bitnami/mongodb/data/db/diagnostic.data/metrics.2023-10-10T11-45-01Z-00000
A /bitnami/mongodb/data/db/diagnostic.data/metrics.2023-10-10T11-45-12Z-00000
A /bitnami/mongodb/data/db/diagnostic.data/metrics.interim
A /bitnami/mongodb/data/db/collection-0-6450241117759633738.wt
A /bitnami/mongodb/data/db/WiredTiger.turtle
A /bitnami/mongodb/data/db/_mdb_catalog.wt
A /bitnami/mongodb/data/db/index-1-6450241117759633738.wt
A /bitnami/mongodb/data/db/index-3-6450241117759633738.wt
A /bitnami/mongodb/data/db/WiredTiger.lock
```

Each line begins with either A or C, which are just shorthand for added or changed, respectively. 

