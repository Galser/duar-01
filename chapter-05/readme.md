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



### Storage Volumes

There are times when the default disk space allocated to a container, or the container’s ephemeral nature, is not appropriate for the job at hand, so you’ll need storage that can persist between container deployments.
Mounting storage from the Docker host is not generally advisable because it ties your container to a particular Docker host for its persistent state. But for cases like temporary cache files or other semi-ephemeral states, it can make sense.
For times like this, you can leverage the -v command to mount directories and indi‐ vidual files from the host server into the container. The following example mounts /Users/.../duar-01/mnttest from host into /data  inside the container


```
 mkdir mnttest
 cp /etc/man.conf mnttest
 ✗ ls -l mnttest
total 8
-r--r--r--@ 1 andrii  staff  2274 Sep 22 11:42 man.conf
 ✗ ls /Users/.../duar-01/mnttest
man.conf
 ✗ docker run --rm -ti -v /Users/.../duar-01/mnttest:/data ubuntu:latest /bin/bash
root@96a5a2619001:/# mount | grep data
:/Users/andrii on /data type fuse.sshfs (rw,nosuid,nodev,relatime,user_id=501,group_id=1000,allow_other)
/dev/disk/by-label/data-volume on /etc/resolv.conf type ext4 (rw,relatime)
/dev/disk/by-label/data-volume on /etc/hostname type ext4 (rw,relatime)
/dev/disk/by-label/data-volume on /etc/hosts type ext4 (rw,relatime)
root@96a5a2619001:/# ls /data
man.conf
root@96a5a2619001:/# cat /data/man.conf
#
# This file is read by man to configure the default manpath (also used
# when MANPATH contains an empty substring), and to indicate support for
# a given locale.  The following configuration variables are supported:
...
```

### SELinux and Volume Mounts

If you have SELinux enabled on your Docker host, you may get a “Permission Denied” error when trying to mount a volume into your container. You can handle this by using one of the z options to the Docker command for mounting volumes:
• The lowercase z option indicates that the bind mount content is shared among multiple containers.
• The uppercase Z option indicates that the bind mount content is private and unshared.
If you are going to share a volume between containers, you can use the z option to the volume mount:

```    
    docker run -v /app/dhcpd/etc:/etc/dhcpd:z dhcpd
```

However, the best option is actually the Z option to the volume mount command, which will set the directory with the exact MCS label (e.g., chcon ... -l s0:c1,c2) that the container will be using. This provides for the best security and will allow only a single container to mount the volume:


```
    docker run -v /app/dhcpd/etc:/etc/dhcpd:Z dhcpd
```

Use extreme caution with the z options. Bind-mounting a sys‐ tem directory such as /etc or /var with the Z option will very likely render your system inoperable and require you to relabel the host machine manually.



### Read-only volumes

It is possible to tell Docker that the root volume of your container should be mounted read-only so that processes within the container cannot write anything to the root filesystem. This prevents things like logfiles, which a developer may be unaware of, from filling up the container’s allocated disk in production. When it’s used in con‐ junction with a mounted volume, you can ensure that data is written only into expected locations.
In the previous example, we could accomplish this simply by adding --read- only=true to the command.

```
    $ docker run --rm -ti --read-only=true -v /mnt/session_data:/data \
        ubuntu:latest /bin/bash
```        

Output : 

```
docker run --rm --read-only=true -ti -v /Users/.../duar-01/mnttest:/data ubuntu:latest /bin/bash
root@93579ff20ace:/# mount
overlay on / type overlay (ro,relatime,lowerdir=/var/lib/docker/overlay2/l/JUXVJWFJSYCNPWEAT4SUAIHKW6:/var/lib/docker/overlay2/l/A6JSEYQVW5UOJEDYUKVHD7JJAA,upperdir=/var/lib/docker/overlay2/9a023f42c916821e53f33fd018e347eed29eff9ba3cbe78dcf6ba6965eb15c89/diff,workdir=/var/lib/docker/overlay2/9a023f42c916821e53f33fd018e347eed29eff9ba3cbe78dcf6ba6965eb15c89/work)
...
tmpfs on /sys/firmware type tmpfs (ro,relatime,inode64)
root@93579ff20ace:/# ls -l
total 52
lrwxrwxrwx   1 root root       7 Aug 16 02:25 bin -> usr/bin
drwxr-xr-x   2 root root    4096 Apr 18  2022 boot
drwxr-xr-x   1  501 dialout   96 Sep 22 09:42 data
....
root@93579ff20ace:/# cd /
root@93579ff20ace:/# mkdir test
mkdir: cannot create directory 'test': Read-only file system
```

Make dircotry test at the end to confirm.

### Resource Quotas

When people discuss the types of problems they must often cope with when working in the cloud, the “noisy neighbor” is often near the top of the list. The basic problem this term refers to is that other applications running on the same physical system as yours can have a noticeable impact on your performance and resource availability. There is an important caveat here. While Docker supports various resource limits, you must have these capabilities enabled in your kernel in order for Docker to take advantage of them.



> Constraints are normally applied at the time of container creation. If you need to change them, you can use the docker container update command or deploy a new container with the adjustments.


### CPU shares
Docker has several ways to limit CPU usage by applications in containers.  Docker assigns the number 1024 to represent the full pool. By config‐ uring a container’s CPU shares, you can dictate how much time the container gets to use the CPU for. If you want the container to be able to use at most half of the com‐ puting power of the system, then you would allocate it 512 shares. Note that these are not exclusive shares, meaning that assigning all 1024 shares to a container does not
prevent all other containers from running. Rather, it’s a hint to the scheduler about how long each container should be able to run each time it’s scheduled. 

For the following examples, we’ll use a new Docker image that contains the stress command for pushing a system to its limits. When we run stress without any cgroup constraints, it will use as many resources as we tell it to. The following command creates a load average of around 5 by creating two CPU-bound processes, one I/O-bound process, and two memory allocation pro‐ cesses. Note that in the following code, we are running on a system with two CPUs.
    $ docker run --rm -ti progrium/stress \
      --cpu 2 --io 1 --vm 2 --vm-bytes 128M --timeout 120s



### CPU pinning
It is also possible to pin a container to one or more CPU cores. This means that work for this container will be scheduled only on the cores that have been assigned to this container. 


### Simplifying CPU qutoas
While CPU shares were the original mechanism in Docker for managing CPU limits, Docker has evolved a great deal since and one of the ways that it now makes users’ lives easier is by greatly simplifying how CPU quotas can be set. Instead of trying to set CPU shares and quotas correctly, you can now simply tell Docker how much CPU you would like to be available to your container, and it will do the math required to set the underlying cgroups correctly.
The --cpus command can be set to a floating-point number between 0.01 and the number of CPU cores on the Docker server.


### Memory
We can control how much memory a container can access in a manner similar to constraining the CPU. There is, however, one fundamental difference: while con‐ straining the CPU only impacts the application’s priority for CPU time, the memory limit is a hard limit. Even on an unconstrained system with 96 GB of free memory, if we tell a container that it may have access only to 24 GB, then it will only ever get to use 24 GB regardless of the free memory on the system. 


Example of memoery limit : 


```
docker run --rm -ti --memory 100m progrium/stress --cpu 2 --io 1 --vm 2 \
        --vm-bytes 128M --timeout 120s
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
stress: info: [1] dispatching hogs: 2 cpu, 1 io, 2 vm, 0 hdd
stress: dbug: [1] using backoff sleep of 15000us
stress: dbug: [1] setting timeout to 120s
stress: dbug: [1] --> hogcpu worker 2 [8] forked
stress: dbug: [1] --> hogio worker 1 [10] forked
stress: dbug: [1] --> hogvm worker 2 [11] forked
stress: dbug: [1] using backoff sleep of 6000us
stress: dbug: [1] setting timeout to 120s
stress: dbug: [1] --> hogcpu worker 1 [12] forked
stress: dbug: [1] --> hogvm worker 1 [13] forked
stress: dbug: [13] allocating 134217728 bytes ...
stress: dbug: [13] touching bytes in strides of 4096 bytes ...
stress: FAIL: [1] (416) <-- worker 13 got signal 9
stress: WARN: [1] (418) now reaping child worker processes
stress: dbug: [1] <-- worker 10 reaped
stress: dbug: [1] <-- worker 8 reaped
stress: dbug: [1] <-- worker 11 reaped
stress: dbug: [1] <-- worker 12 reaped
stress: FAIL: [1] (452) failed run completed in 0s
```

We see that this run quickly fails with the line:

```    
    stress: FAIL: [1] (452) failed run completed in 0s
```   

This is because the container tries to allocate more memory than it is allowed, and the Linux out-of-memory (OOM) killer is invoked and starts killing processes within the cgroup to reclaim memory. Since our container has only one running process, this kills the container.
Docker has features that allow you to tune and disable the Linux OOM killer by using the --oom-kill-disable and the --oom-score-adj arguments to docker run.


### Block I/O

Many containers are just stateless applications and won’t have a need for I/O restric‐ tions. But Docker also supports limiting block I/O in a few different ways via the cgroups mechanism. Acts a bit like CPU shares goes from 0 to 1000


### ulimits

Before Linux cgroups, there was another way to place a limit on the resources avail‐ able to a process: the application of user limits via the ulimit command. That mecha‐ nism is still available and still useful for all of the use cases where it was traditionally used.


### Auto-Restarting a Container
In many cases, we want our containers to restart if they exit. Some containers are very short-lived and come and go quickly. But for production applications, for instance, you expect them to be up and running at all times after you’ve told them to run. If you are running a more complex system, a scheduler may do this for you.
In the simple case, we can tell Docker to manage restarts on our behalf by passing the --restart argument to the docker run command. It takes four values: no, always, or on-failure, or unless-stopped. If restart is set to no, the container will never restart if it exits. If it is set to always, the container will restart whenever it exits, with no regard to the exit code. If restart is set to on-failure:3, then whenever the container exits with a nonzero exit code, Docker will try to restart the container three times before giving up. unless-stopped is the most command choice, and will restart the container unless it is intentionally stopped with something like docker stop.
We can see this in action by rerunning our last memory-constrained stress container without the --rm argument, but with the --restart argumen


### Pausing and Unpausing a Container
There are a few reasons why we might not want to completely stop our container. We might want to pause it, leave its resources allocated, and leave its entries in the pro‐ cess table. That could be because we’re taking a snapshot of its filesystem to create a new image, or just because we need some CPU on the host for a while. If you are used to normal Unix process handling, you might wonder how this actually works since containerized processes are just processes.
Pausing leverages the cgroups freezer, which essentially just prevents your process from being scheduled until you unfreeze it. This will prevent the container from doing anything while maintaining its overall state, including memory contents. Unlike stopping a container, where the processes are made aware that they are stop‐ ping via the SIGSTOP signal, pausing a container doesn’t send any information to the container about its state change. That’s an important distinction. Several Docker commands use pausing and unpausing internally as well. Here is how we pause a con‐ tainer:


```
docker pause 6b785f78b75e
```

If we look at the list of running containers, we will now see that the Redis container status is listed as (Paused).
    
```    
    # docker ps
    CONTAINER ID  IMAGE                   ...  STATUS                  ...
    6b785f78b75e  progrium/stress:latest  ...  Up 36 minutes (Paused)  ...
```

Attempting to use the container in this paused state would fail. It’s present, but noth‐ ing is running. We can now resume the container by using the docker unpause com‐ mand.

```
    $ docker unpause 6b785f78b75e
    6b785f78b75e
    $ docker ps
    CONTAINER ID  IMAGE                   ... STATUS ...
    6b785f78b75e  progrium/stress:latest  ... Up 38 minutes ...
```

It’s back to running, and docker ps correctly reflects the new state. Note that it shows “Up 38 minutes” now, because Docker still considers the container to be running even when it is paused.


.