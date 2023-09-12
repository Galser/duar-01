## Optimizing build sepped ( opt for Docker cache)

Given Dockerfile : 

```
FROM fedora
RUN dnf install -y httpd && \
dnf clean all
RUN mkdir /var/www && \
        mkdir /var/www/html
ADD index.html /var/www/html
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

+ [index.html]

Let's time our build : 


```
time docker build --no-cache .

Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM fedora
 ---> f838063d816c
Step 2/5 : RUN dnf install -y httpd && dnf clean all
 ---> Running in 5be7957a767f

...
Complete!
42 files removed
Removing intermediate container 5be7957a767f
 ---> 59ff253ccd48
Step 3/5 : RUN mkdir -p /var/www/html
 ---> Running in ed42838868ab
Removing intermediate container ed42838868ab
 ---> d2aef076ad27
Step 4/5 : ADD index.html /var/www/html
 ---> ca71329249a1
Step 5/5 : CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
 ---> Running in 32e82c894160
Removing intermediate container 32e82c894160
 ---> 09342bd412b9
Successfully built 09342bd412b9
docker build --no-cache .  0.05s user 0.08s system 0% cpu 53.277 total
```

Rebuild right away but using cache : 

```
time docker build .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM fedora
 ---> f838063d816c
Step 2/5 : RUN dnf install -y httpd && dnf clean all
 ---> Using cache
 ---> 59ff253ccd48
Step 3/5 : RUN mkdir -p /var/www/html
 ---> Using cache
 ---> d2aef076ad27
Step 4/5 : ADD index.html /var/www/html
 ---> Using cache
 ---> ca71329249a1
Step 5/5 : CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
 ---> Using cache
 ---> 09342bd412b9
Successfully built 09342bd412b9
docker build .  0.05s user 0.07s system 39% cpu 0.286 total
````

So we have 53.277 seconds  total vs 0.286 seconds total

Now, let’s make a small improvement to the index.html file so that it looks like this:
    
```
    <html>
      <head>
<title>My custom Web Site</title> </head>
<body>
<div align="center">
<p>Welcome to my custom Web Site!!!</p> </div>
      </body>
    </html>
```

and then let’s time the rebuild again:

```
time docker build .

Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM fedora
 ---> f838063d816c
Step 2/5 : RUN dnf install -y httpd && dnf clean all
 ---> Using cache
 ---> 59ff253ccd48
Step 3/5 : RUN mkdir -p /var/www/html
 ---> Using cache
 ---> d2aef076ad27
Step 4/5 : ADD index.html /var/www/html
 ---> 54926d75ef0c
Step 5/5 : CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
 ---> Running in 9d78cc2c1d98
Removing intermediate container 9d78cc2c1d98
 ---> 3111df76a6b3
Successfully built 3111df76a6b3
docker build .  0.05s user 0.08s system 16% cpu 0.794 total
```

If you look at the output carefully, you will see that the cache was used for most of the build. It wasn’t until Docker step 4, when Docker needed to copy index.html, that the cache was invalidated and the layers had to be recreated. Because the cache could be used for most of the build, the build still did not exceed a second.


Now let's change order 

But what would happen if you changed the order of the commands in the Dockerfile so that they looked like this:
FROM fedora
RUN mkdir /var/www && \
        mkdir /var/www/html
ADD index.html /var/www/html RUN dnf install -y httpd && \
dnf clean all
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
Let’s quickly time another test build without the cache to get a baseline:


    $ time docker build --no-cache .
    Sending build context to Docker daemon  3.072kB
    ...
    Successfully built c949ed0f036e
    real 0m48.824s
    user 0m0.076s
    sys  0m0.066s
In this case, the build took 48 seconds to complete. The difference in time from the very first test is entirely due to fluctuating network speeds and has nothing to do with the changes that you have made to the Dockerfile.

But now, if we change index.html - cache is of no us, as adding INDEX.HTML comes BEFORE heavy-install :-(

