## Docker Compose


Sic! For some compose features on MacOS with Colima one may need to 

1. Specify env var for DOCKER_HOST , as Colima has non-standart socker location : 

```
export DOCKER_HOST="unix://$HOME/.colima/docker.sock"
```


2. Install docker compose, as defualt installation contains empty wrapper. For example with Brew


```
brew install docker-compose
```

