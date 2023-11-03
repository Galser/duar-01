## The Path to Production Containers


Large scale production envs

At the most basic level, a production story must encompass three things:
• It must be a repeatable process. Each time you invoke it, it needs to do the same thing.
• It needs to handle configuration for you. You must be able to define your application’s configuration in a particular environment and then guarantee that it will ship that configuration on each deployment.
• It must deliver an executable artifact that can be started.

huh??? 

> The next level of configuration is the configuration directly applied to the application. We talked earlier about this in detail. Docker’s native mechanism is to use environ‐ ment variables, and this works across all of the modern platforms. Some systems, notably, make it easier to rely on more traditional configuration files. We find that this can really impact the observability of the application and discourage you from relying on that crutch.


There is one more Docker-specific step. We want to take our passed build and push that image to our registry. The registry is the interchange point between builds and deployments. It also allows us to share the image with other builds that might be stacked on top of it. But for now, let’s just think of it as the place where we put and tag successful builds. Our build script will now do a docker tag to give the image the right build tag(s), including latest, and then a docker push to push the build to the registry.
