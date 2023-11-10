## Docker at Scale

The major public cloud providers have all worked to support Linux containers natively in their offerings. Some of the biggest efforts to implement Docker contain‐ ers in the public cloud include:
• Amazon Elastic Container Service
• Google Kubernetes Engine
• Azure Container Service
• Red Hat OpenShift

+ originally Windows-based Azure : 
• Azure Container Service (Azure supports both Windows and Linux operating systems)


Some tools :

- Centurion, Docker Swarm, Amazon ECS and Fargate


**There are now two things called “Swarm,” both of which come from Docker, Inc.**
The original standalone **Docker Swarm** is officially Docker Swarm, but there is a second “Swarm,” which is more specifically called **Swarm mode**. This is actually built into the Docker Engine. The built-in Swarm mode is a lot more capable than the original Docker Swarm and is intended to replace it entirely. Swarm mode has the major advantage of not requiring you to install anything separately. You already have Docker Swarm Mode 
this on your Docker box! This is the Docker Swarm we’ll focus on here. Hopefully now that you know there are two different Docker Swarms, you won’t get confused by contradictory information on the internet.
The idea behind Docker Swarm is to present a single interface to the docker client tool, but have that interface be backed by a whole cluster rather than a single Docker daemon. Swarm is primarily aimed at managing clustered computing resources via the Docker tools. 


**Fargate** is simply a marketing label Amazon uses for the new fea‐ ture of ECS that makes it possible for AWS to automatically man‐ age all the nodes in your container cluster so that you can focus on deploying your service.



## AWS EC2/Fargate test on MacOS ( Colima )


- Install AWS Cli
```
brew update
brew install awscli
```

- Configrue AWS ( Doormat !) .  either do env vatrs or config file 
```
export AWS_ACCESS_KEY_ID=ASIAXLCPKSCKZLEDOXUJ
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

- Test it : 

```
aws iam list-users
{
    "Users": []
}
(END)
````

- Create new Cluster : 

```
aws ecs create-cluster --cluster-name fargate-testing --region eu-central-1
```
and I got output :

```JSON
{
    "cluster": {
        "clusterArn": "arn:aws:ecs:eu-central-1:504825024661:cluster/fargate-testing",
        "clusterName": "fargate-testing",
        "status": "ACTIVE",
        "registeredContainerInstancesCount": 0,
        "runningTasksCount": 0,
        "pendingTasksCount": 0,
        "activeServicesCount": 0,
        "statistics": [],
        "tags": [],
        "settings": [
            {
                "name": "containerInsights",
                "value": "disabled"
            }
        ],
        "capacityProviders": [],
        "defaultCapacityProviderStrategy": []
    }
}
```


Now that our container cluster is set up, we need to start putting it to work. To do this, we need to create at least one task definition. The Amazon Elastic Container Ser‐ vice defines the term task definition as a list of containers grouped together.
To create your first task definition, open up your favorite editor, copy in the following JSON, and then save it as webgame-task.json in your current directory, as shown here:

```JSON
{
  "containerDefinitions": [
    {
      "name": "web-game",
      "image": "spkane/quantum-game",
      "cpu": 0,
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "environment": [],
      "mountPoints": [],
      "volumesFrom": []
    }
  ],
  "family": "fargate-game",
  "networkMode": "awsvpc",
  "volumes": [],
  "placementConstraints": [],
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "cpu": "256",
  "memory": "512"
}
```

To upload this task definition to Amazon, you will need to run a command similar to what is shown here:


```
aws ecs register-task-definition --cli-input-json file://./webgame-task.json --region eu-central-1
```

The output of the command for my case is : 

```JSON
{
    "taskDefinition": {
        "taskDefinitionArn": "arn:aws:ecs:eu-central-1:504825024661:task-definition/fargate-game:1",
        "containerDefinitions": [
            {
                "name": "web-game",
                "image": "spkane/quantum-game",
                "cpu": 0,
                "portMappings": [
                    {
                        "containerPort": 8080,
                        "hostPort": 8080,
                        "protocol": "tcp"
                    }
                ],
                "essential": true,
                "environment": [],
                "mountPoints": [],
                "volumesFrom": []
            }
        ],
        "family": "fargate-game",
        "networkMode": "awsvpc",
        "revision": 1,
        "volumes": [],
        "status": "ACTIVE",
        "requiresAttributes": [
            {
                "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
            },
            {
                "name": "ecs.capability.task-eni"
            }
        ],
        "placementConstraints": [],
        "compatibilities": [
            "EC2",
            "FARGATE"
        ],
        "requiresCompatibilities": [
            "FARGATE"
        ],
        "cpu": "256",
        "memory": "512",
        "registeredAt": "2023-11-07T15:35:42.094000+01:00",
        "registeredBy": "arn:aws:sts::504825024661:assumed-role/aws_andrii_test-developer/andrii@hashicorp.com"
    }
}
```

Let's check. We can then list all of our task definitions by running the following: : 

```BASH
aws ecs list-task-definitions
```

Output : 

```JSON
{
    "taskDefinitionArns": [
        "arn:aws:ecs:eu-central-1:504825024661:task-definition/fargate-game:1"
    ]
}
```

Now you are ready to create your first task in your cluster. You do so by running a command like the one shown next. The count argument in the command allows you to define how many copies of this task you want deployed into your cluster. For this job, one is enough.
You will need to modify the following command to reference a valid subnet ID and security-group ID from your AWS VPC. You should be able to find these in the AWS console or by using the AWS CLI commands aws ec2 describe-subnets and aws ec2 describe-security-groups. You can also tell AWS to assign your tasks a public IP address by using a network configuration similar to this:


```BASH
aws ecs create-service --cluster fargate-testing --service-name \
        fargate-game-service --task-definition fargate-game:1 --desired-count 1 \
        --launch-type "FARGATE" --network-configuration \
        "awsvpcConfiguration={subnets=[subnet-0cdf5829905c4bc9e],\
        securityGroups=[sg-0f07c7dd21cc1e434]}"

```


```JSON

{
    "service": {
        "serviceArn": "arn:aws:ecs:eu-central-1:504825024661:service/fargate-testing/fargate-game-service",
        "serviceName": "fargate-game-service",
        "clusterArn": "arn:aws:ecs:eu-central-1:504825024661:cluster/fargate-testing",
        "loadBalancers": [],
        ......
        
```

Let's list our services : 

```BASH
aws ecs list-services --cluster fargate-testing --region eu-central-1
```

Output
```JSON
{
    "serviceArns": [
        "arn:aws:ecs:eu-central-1:504825024661:service/fargate-testing/fargate-game-service"
    ]
}
```

> Note : Game links in book are not valid anymore 

Task stopped at: 2023-11-07T14:55:16.006Z
CannotPullContainerError: pull image manifest has been retried 5 time(s): failed to resolve ref docker.io/spkane/quantum-game:latest: failed to do request: Head "https://registry-1.docker.io/v2/spkane/quantum-game/manifests/latest": dial tcp 34.194.164.123:443: i/o timeout


Dont't forget to stop and delete the tasks. 


