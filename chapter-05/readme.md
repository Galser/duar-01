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

```
 docker run --rm -ti ubuntu:latest /bin/bash
root@dc94c9b854ff:/# mount
overlay on / type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/ZV5NYPZDIISPTVCZPWNAZSRFVG:/var/lib/docker/overlay2/l/A6JSEYQVW5UOJEDYUKVHD7JJAA,upperdir=/var/lib/docker/overlay2/9fc6cf761233bdd5aec9f8953b19664942d28b1ebc4e48c19e474454a6a5e660/diff,workdir=/var/lib/docker/overlay2/9fc6cf761233bdd5aec9f8953b19664942d28b1ebc4e48c19e474454a6a5e660/work)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
tmpfs on /sys/fs/cgroup type tmpfs (rw,nosuid,nodev,noexec,relatime,mode=755,inode64)
openrc on /sys/fs/cgroup/openrc type cgroup (ro,nosuid,nodev,noexec,relatime,release_agent=/lib/rc/sh/cgroup-release-agent.sh,name=openrc)
cpuset on /sys/fs/cgroup/cpuset type cgroup (ro,nosuid,nodev,noexec,relatime,cpuset)
cpu on /sys/fs/cgroup/cpu type cgroup (ro,nosuid,nodev,noexec,relatime,cpu)
cpuacct on /sys/fs/cgroup/cpuacct type cgroup (ro,nosuid,nodev,noexec,relatime,cpuacct)
blkio on /sys/fs/cgroup/blkio type cgroup (ro,nosuid,nodev,noexec,relatime,blkio)
memory on /sys/fs/cgroup/memory type cgroup (ro,nosuid,nodev,noexec,relatime,memory)
devices on /sys/fs/cgroup/devices type cgroup (ro,nosuid,nodev,noexec,relatime,devices)
freezer on /sys/fs/cgroup/freezer type cgroup (ro,nosuid,nodev,noexec,relatime,freezer)
net_cls on /sys/fs/cgroup/net_cls type cgroup (ro,nosuid,nodev,noexec,relatime,net_cls)
perf_event on /sys/fs/cgroup/perf_event type cgroup (ro,nosuid,nodev,noexec,relatime,perf_event)
net_prio on /sys/fs/cgroup/net_prio type cgroup (ro,nosuid,nodev,noexec,relatime,net_prio)
hugetlb on /sys/fs/cgroup/hugetlb type cgroup (ro,nosuid,nodev,noexec,relatime,hugetlb)
pids on /sys/fs/cgroup/pids type cgroup (ro,nosuid,nodev,noexec,relatime,pids)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k,inode64)
/dev/disk/by-label/data-volume on /etc/resolv.conf type ext4 (rw,relatime)
/dev/disk/by-label/data-volume on /etc/hostname type ext4 (rw,relatime)
/dev/disk/by-label/data-volume on /etc/hosts type ext4 (rw,relatime)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
proc on /proc/bus type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/fs type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/irq type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/sys type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/sysrq-trigger type proc (ro,nosuid,nodev,noexec,relatime)
tmpfs on /proc/acpi type tmpfs (ro,relatime,inode64)
tmpfs on /proc/keys type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
tmpfs on /proc/timer_list type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
tmpfs on /proc/scsi type tmpfs (ro,relatime,inode64)
tmpfs on /sys/firmware type tmpfs (ro,relatime,inode64)
root@dc94c9b854ff:/# hostname -f
dc94c9b854ff
root@dc94c9b854ff:/#
exit
```


When you see any examples with a prompt that looks something like root@hashID, it means that you are running a command within the container instead of on the Docker host. Note that there are occasions when a container will have been configured with a different hostname instead (e.g., using --name on the CLI), but in the default case, it’s a hash.
There are quite a few bind mounts in a container, but in this case we are interested in this one:

```
/dev/disk/by-label/data-volume on /etc/hosts type ext4 (rw,relatime)
```
While the device number will be different for each container, the part we care about is that the mount point is /etc/hostname. This links the container’s /etc/hostname to the hostname file that Docker has prepared for the container, which by default contains the container’s ID and is not fully qualified with a domain name.
We can check this in the container by running `hostname`


To set the hostname specifically, we can use the --hostname argument to pass in a more specific value.
    $ docker run --rm -ti --hostname="mycontainer.example.com" \
        ubuntu:latest /bin/bash


### Domain Name Service

Just like /etc/hostname, the resolv.conf file that configured Domain Name Service (DNS) resolution is managed via a bind mount between the host and container.

```
	/dev/disk/by-label/data-volume on /etc/resolv.conf type ext4 (rw,relatime)    
```

By default, this is an exact copy of the Docker host’s resolv.conf file. If you didn’t want this, you could use a combination of the --dns and --dns-search arguments to over‐ ride this behavior in the container:

```
    $ docker run --rm -ti --dns=8.8.8.8 --dns=8.8.4.4 --dns-search=example1.com \
        --dns-search=example2.com ubuntu:latest /bin/bash
```

If you want to leave the search domain completely unset, then use `--dns-search=.`

Within the container, you would still see a bind mount, but the file contents would no longer reflect the host’s `resolv.conf`; instead, it would now look like this:

```
cat /etc/resolv.conf
search example1.com example2.com
nameserver 8.8.8.8
nameserver 8.8.4.4
```

vs what I have in system : 

```
cat /etc/resolv.conf
#
# macOS Notice
#
# This file is not consulted for DNS hostname resolution, address
# resolution, or the DNS query routing mechanism used by most
# processes on this system.
#
# To view the DNS configuration used by this system, use:
#   scutil --dns
#
# SEE ALSO
#   dns-sd(1), scutil(8)
#
# This file is automatically generated.
#
search home
nameserver fe80::729f:2dff:fe9a:baae%en0
nameserver 192.168.2.254
```


### MAC address

Another important piece of information that you can configure is the media access control (MAC) address for the container.
Without any configuration, a container will receive a calculated MAC address that starts with the 02:42:ac:11 prefix.
If you need to specifically set this to a value, you can do so by running something similar to this:

```
    $ docker run --rm -ti --mac-address="a2:11:aa:22:bb:33" ubuntu:latest /bin/bash
```


It is possible to cause ARP contention on your network if two systems advertise the same MAC address. If you have a strong need to do this, try to keep your locally administered address ranges within some of the official ranges, like x2-xx-xx-xx-xx-xx, x6-xx-xx-xx-xx- xx, xA-xx-xx-xx-xx-xx, and xE-xx-xx-xx-xx-xx (with x being any valid hexidecimal character).
