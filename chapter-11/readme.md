# Advanced Topics

## Containers in Detail
Though we usually talk about Linux containers as a single entity, they are, in fact, implemented through several separate mechanisms built into the Linux kernel that all work together: control groups (cgroups), namespaces, and SELinux or AppArmor, all of which serve to contain the process. cgroups provide for resource limits, namespa‐ ces allow for processes to use identically named resources and isolate them from each other’s view of the system, and SELinux or AppArmor provides strong security isolation.

Control groups, or cgroups for short, allow you to set limits on resources for processes and their children. This is the mechanism that Docker uses to control limits on mem‐ ory, swap, CPU, and storage and network I/O resources. cgroups are built into the Linux kernel and originally shipped back in 2007 in Linux 2.6.24.

The official kernel documentation defines them as “a mechanism for aggregating/partitioning sets of tasks, and all their future children, into hierarchical groups with specialized behav‐ ior.” It’s important to note that this **setting applies to a process and all of the children that descend from it. That’s exactly how containers are structured.** 

cgroups hooks exposed by Docker via the Remote API. This allows you to control memory, swap, and disk usage. But there are lots of other things you can limit with cgroups, including the number of I/O operations per second (iops) a container can have, for example. You might find that in your environ‐ ment you need to use some of these levers to keep your containers under control, and there are a few ways you can go about doing that. By their nature, cgroups need to do a lot of accounting of resources used by each group. That means that when you’re using them, the kernel has a lot of interesting statistics about how much CPU, RAM, disk I/O, and so on your processes are using. So Docker uses cgroups not just to limit resources but also to report on them. These are many of the metrics you see, for example, in the output of docker stats.

## The /sys filesystem

The primary way to control cgroups in a fine-grained manner, even if you configured them with Docker, is to manage them yourself. This is the most powerful method because changes don’t just happen at creation time—they can be done on the fly.
On systems with systemd, there are command-line tools like systemctl that you can use to do this. But since cgroups are built into the kernel, the method that works everywhere is to talk to the kernel directly via the /sys filesystem. If you’re not familiar with /sys, it’s a filesystem that directly exposes a number of kernel settings and out‐ puts. You can use it with simple command-line tools to tell the kernel how to behave in a number of ways.
Note that this method of configuring cgroups controls for containers works only directly on the Docker server, so it is not available remotely via any API. If you use this method, you’ll need to figure out how to script this for your own environment.


Some tests : 

First we need to get the long ID of the container, and then we need to find it in the /sys filesystem. Here’s what that looks like:

```    
    $ docker ps --no-trunc
    CONTAINER ID IMAGE        COMMAND       CREATED     STATUS    NAMES
    dcbbaa763... 0415448f2cc2 "supervisord" 3 weeks ago Up 2 days romantic_morse
```

 Let’s inspect the CPU shares for this container. Remember that we set these earlier via the Docker command-line tool. But for a normal container where no settings were passed, this setting is the default:

```
    $ cat /sys/fs/cgroup/cpu/docker/dcbbaa763daf/cpu.shares
    1024
```    

1024 CPU shares means we are not limited at all. Let’s tell the kernel that this con‐ tainer should be limited to half that:

```
    $ echo 512 > /sys/fs/cgroup/cpu/docker/dcbbaa763daf/cpu.shares
    $ cat /sys/fs/cgroup/cpu/docker/dcbbaa763daf/cpu.shares
    512
```

In production you should not use this method to adjust cgroups on the fly, but we are demonstrating it here so that you understand the underlying mechanics that make all of this work. Take a look at docker container update if you’d like to adjust these on a run‐ ning container. You might also find the --cgroup-parent option to docker run interesting.



## Namespaces

Inside each container, you see a filesystem, network interfaces, disks, and other resources that all appear to be unique to the container despite sharing the kernel with all the other processes on the system. The primary network interface on the actual machine, for example, is a single shared resource. But inside your container it will look like it has an entire network interface to itself. This is a really useful abstraction: it’s what makes your container feel like a machine all by itself. The way this is imple‐ mented in the kernel is with namespaces. Namespaces take a traditionally global resource and present the container with its own unique and unshared version of that resource.
Unlike cgroups, namespaces are not something that you can see reflected in the kernel filesystems, like /sys and /proc.

Rather than just having a single namespace, however, containers have a namespace on each of the six types of resources that are currently namespaced in the kernel: mounts, UTS, IPC, PID, network, and user namespaces. Essentially when you talk about a container, you’re talking about a number of different namespaces that Docker sets up on your behalf. So what do they all do?


### Exploring namespaces

One of the easiest to demonstrate namespaces is the UTS namespace, so let’s use docker exec to get a shell in a container and take a look. From within the Docker server, run the following:

```
# hostname
andrii-NT9F3W7MC7
... ✗ docker exec -i -t 72179e8d7d5b /bin/bash -l
root@72179e8d7d5b:/# hostname
72179e8d7d5b
root@72179e8d7d5b:/#
```

Although docker exec will work from a remote system, here we ssh into the Docker server itself in order to demonstrate that the hostname of the server is different from inside the container. There are also easier ways to obtain some of the information we’re getting here. But the idea is to explore how namespaces work, not to pick the most ideal path to gather the information.
That docker exec command line gets us an interactive process (-i), allocates a pseudo-TTY (-t), and then executes /bin/bash while executing all the normal login process in the bash shell (-l). Once we have a terminal open inside the container’s namespace, we ask for the hostname and get back the container ID. That’s the default hostname for a Docker container unless you tell Docker to name it otherwise. This is a pretty simple example, but it should clearly show that we’re not in the same name‐ space as the host.

## Privileged Containers
There are times when you need your container to have special kernel capabilities that would normally be denied to the container. These could include mounting a USB drive, modifying the network configuration, or creating a new Unix device.


The easiest way to significantly expand a container’s privileges is by launch‐ ing it with the --privileged=true argument. 

> Example - changing MAC-address of netwrok adapter

The problem with using the --privileged=true argument is that you are giving your container very broad privileges, and in most cases you likely need only one or two kernel capabilities to get the job done.

To change the MAC address, the only kernel capability we actually need is CAP_NET_ADMIN. Instead of giving our container the full set of privileges, we can give it this one privilege by launching our Docker container with the --cap-add argu‐ ment, as shown here:

```
    $ docker run -ti --rm --cap-add=NET_ADMIN ubuntu /bin/bash
```


It is also possible to remove specific capabilities from a container. I imagine for a moment that your security team requires that tcpdump be disabled in all containers, and when you test some of your containers, you find that tcpdump is installed and can easily be run.


```
    $ docker run -ti --rm spkane/train-os:latest tcpdump -i eth0
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
    26:52:10.182287 IP6 :: > ff03::1:ff3e:f0e4: ICMP6, neighbor solicitation, ...
    26:52:10.199257 ARP, Request who-has 1.qarestr.sub-172-14-0.myvzw.com     ...
    26:52:10.199312 ARP, Reply 1.qarestr.sub-172-14-0.myvzw.com is-at         ...
    26:52:10.199328 IP 204117a59e09.38725 > 192.168.62.4.domain: 52244+ PTR?  ...
    26:52:11.223184 IP6 fe83::8c11:30ff:fe5e:f0e4 > ff02::16: HBH ICMP6,      ...
    ...

```

You could remove tcpdump from your images, but there is very little preventing someone from reinstalling it. The most effective way to solve this problem is to deter‐ mine what capability tcpdump needs to operate and remove that from the container. In this case, you can do so by adding --cap-drop=NET_RAW to your docker run com‐ mand.


```
$ docker run -ti --rm --cap-drop=NET_RAW
    spkane/train-os:latest tcpdump -i eth0
    tcpdump: eth0: You don't have permission to capture on that device
    (socket: Operation not permitted)
```

By using both the --cap-add and --cap-drop arguments to docker run, you can finely control your container’s Linux kernel capabilities.


## Secure Computing Mode

When Linux kernel version 2.6.12 was released in 2005, it included a new security feature called Secure Computing Mode, or seccomp for short. This feature enables a process to make a one-way transition into a special state, where it will only be allowed to make the system calls exit(), sigreturn(), and read() or write() to already- open file descriptors.
An extension to seccomp, called seccomp-bpf, utilizes the Linux version of Berkeley Packet Filter (bpf) rules to allow you to create a policy that will provide an explicit list of system calls that a process can utilize while running under Secure Computing Mode. The Docker support for Secure Computing Mode utilizes seccomp-bpf so that users can create profiles that give them very fine-grained control of what kernel sys‐ tem calls their containerized processes are allowed to make.

Let's use `strace` to actually see what calls are used : 

``` 
 docker run -ti --rm spkane/train-os:latest whoami
Unable to find image 'spkane/train-os:latest' locally
latest: Pulling from spkane/train-os
a35d67b9c848: Pull complete
66702060bc41: Pull complete
46a5f8f4db9a: Pull complete
7267d4acb0f3: Pull complete
Digest: sha256:597dba0ca0055c05b9a37e0c78204c112829edf0968ff8a42b64dcce77716eff
Status: Downloaded newer image for spkane/train-os:latest
root
```

And now `strace` : 

```
➜  duar-01 git:(chapter-11) ✗ docker run -ti --rm spkane/train-os:latest strace whoami
execve("/usr/bin/whoami", ["whoami"], 0xffffd33ea7e0 /* 7 vars */) = 0
brk(NULL)                               = 0xaaab16e65000
faccessat(AT_FDCWD, "/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=9264, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 9264, PROT_READ, MAP_PRIVATE, 3, 0) = 0xffff829f0000
close(3)                                = 0
openat(AT_FDCWD, "/lib64/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0\267\0\1\0\0\0\200L\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0\2H\220\210\226\205\36F\n\304L\204Fpc`"..., 68, 768) = 68
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=2017256, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xffff829ee000
mmap(NULL, 1616352, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xffff8282d000
mprotect(0xffff82993000, 94208, PROT_NONE) = 0
mmap(0xffff829aa000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x16d000) = 0xffff829aa000
mmap(0xffff829b0000, 31200, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xffff829b0000
close(3)                                = 0
mprotect(0xffff829aa000, 12288, PROT_READ) = 0
mprotect(0xaaaae037f000, 4096, PROT_READ) = 0
mprotect(0xffff829f6000, 8192, PROT_READ) = 0
munmap(0xffff829f0000, 9264)            = 0
brk(NULL)                               = 0xaaab16e65000
brk(0xaaab16e86000)                     = 0xaaab16e86000
geteuid()                               = 0
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 3
connect(3, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
close(3)                                = 0
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 3
connect(3, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
close(3)                                = 0
newfstatat(AT_FDCWD, "/etc/nsswitch.conf", {st_mode=S_IFREG|0644, st_size=2150, ...}, 0) = 0
newfstatat(AT_FDCWD, "/", {st_mode=S_IFDIR|0755, st_size=4096, ...}, 0) = 0
openat(AT_FDCWD, "/etc/nsswitch.conf", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=2150, ...}, AT_EMPTY_PATH) = 0
read(3, "#\n# /etc/nsswitch.conf\n#\n# Name "..., 4096) = 2150
read(3, "", 4096)                       = 0
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=2150, ...}, AT_EMPTY_PATH) = 0
close(3)                                = 0
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=9264, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 9264, PROT_READ, MAP_PRIVATE, 3, 0) = 0xffff829f0000
close(3)                                = 0
openat(AT_FDCWD, "/lib64/libnss_sss.so.2", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0\267\0\1\0\0\0`\27\0\0\0\0\0\0"..., 832) = 832
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=70112, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 131688, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xffff8280c000
mprotect(0xffff82815000, 90112, PROT_NONE) = 0
mmap(0xffff8282b000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0xf000) = 0xffff8282b000
close(3)                                = 0
mprotect(0xffff8282b000, 4096, PROT_READ) = 0
munmap(0xffff829f0000, 9264)            = 0
openat(AT_FDCWD, "/var/lib/sss/mc/passwd", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/var/lib/sss/mc/passwd", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
getpid()                                = 9
getpid()                                = 9
socket(AF_UNIX, SOCK_STREAM, 0)         = 3
fcntl(3, F_GETFL)                       = 0x2 (flags O_RDWR)
fcntl(3, F_SETFL, O_RDWR|O_NONBLOCK)    = 0
fcntl(3, F_GETFD)                       = 0
fcntl(3, F_SETFD, FD_CLOEXEC)           = 0
connect(3, {sa_family=AF_UNIX, sun_path="/var/lib/sss/pipes/nss"}, 110) = -1 ENOENT (No such file or directory)
close(3)                                = 0
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=9264, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 9264, PROT_READ, MAP_PRIVATE, 3, 0) = 0xffff829f0000
close(3)                                = 0
openat(AT_FDCWD, "/lib64/libnss_files.so.2", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0\267\0\1\0\0\0\340(\0\0\0\0\0\0"..., 832) = 832
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=70624, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 156616, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xffff827e5000
mprotect(0xffff827f0000, 81920, PROT_NONE) = 0
mmap(0xffff82804000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0xf000) = 0xffff82804000
mmap(0xffff82805000, 25544, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xffff82805000
close(3)                                = 0
mprotect(0xffff82804000, 4096, PROT_READ) = 0
munmap(0xffff829f0000, 9264)            = 0
openat(AT_FDCWD, "/etc/passwd", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=1064, ...}, AT_EMPTY_PATH) = 0
lseek(3, 0, SEEK_SET)                   = 0
read(3, "root:x:0:0:root:/root:/bin/bash\n"..., 4096) = 1064
close(3)                                = 0
newfstatat(1, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}, AT_EMPTY_PATH) = 0
write(1, "root\n", 5root
)                   = 5
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```

On Linux it fails with : 


```
strace: ptrace(PTRACE_TRACEME, ...): Operation not permitted
    +++ exited with 1 +++
```

As default security profile does not allow such things.  You could potentially fix this by giving your container the process-tracing-related capabilities, like this:

```
$ docker run -ti --rm --cap-add=SYS_PTRACE spkane/train-os:latest \
        strace whoami   
execve("/usr/bin/whoami", ["whoami"], [/* 4 vars */]) = 0
brk(NULL) = 0x136f000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, ... access("/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file ... open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
...
write(1, "root\n", 5root
) =5
close(1)
munmap(0x7f854286b000, 4096)
close(2)
exit_group(0)
+++ exited with 0 +++        
```

But you can also solve this problem by using a seccomp profile. Unlike seccomp, -- cap-add will enable a whole set of system calls, and you may not need them all. With a seccomp profile, however, you can be very specific about exactly what you want enabled or disabled.
If we take a look at the default seccomp profile, we’ll see something like this: 

```
{
...
...
] },
], "syscalls": [
{
"defaultAction": "SCMP_ACT_ERRNO", "archMap": [
{
}, {
"names": [ "accept",
"accept4",
"write",
"writev"
],
"action": "SCMP_ACT_ALLOW", "args": [],
"comment": "",
"includes": {}, "excludes": {}
= 0 = 0 = 0 = ?
"architecture": "SCMP_ARCH_X86_64", "subArchitectures": [
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X32"
...
```

You could completely disable the default Secure Computing Mode profile by setting --securityopt=seccomp:unconfined, however running a container unconfined is a very bad idea in general, and is probably only useful when you are trying to figure out exactly what system calls you need to define in your profile.


        