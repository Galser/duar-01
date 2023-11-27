# Container Platform Design

We will explore two open documents, “The Twelve-Factor App” and the “The Reactive Manifesto” and discuss how they relate to Docker and to building robust container platforms. 


## The Twelve Factors 

I. Codebase
One codebase tracked in revision control, many deploys

II. Dependencies
Explicitly declare and isolate dependencies

III. Config
Store config in the environment

IV. Backing services
Treat backing services as attached resources

V. Build, release, run
Strictly separate build and run stages

VI. Processes
Execute the app as one or more stateless processes

VII. Port binding
Export services via port binding

VIII. Concurrency
Scale out via the process model

IX. Disposability
Maximize robustness with fast startup and graceful shutdown

X. Dev/prod parity
Keep development, staging, and production as similar as possible

XI. Logs
Treat logs as event streams

XII. Admin processes
Run admin/management tasks as one-off processes


> HUh - 
If you need a process for managing secrets that need to be provided to your containers, you might want to look into the documentation for the docker secret command and HashiCorp’s Vault. :-) 

## The Reactive Manifesto
Riding alongside “The Twelve-Factor App,” another pertinent document was released in July of 2013 by Typesafe cofounder and CTO Jonas Bonér, entitled: [“The Reactive Manifesto.”](https://www.reactivemanifesto.org/)  Jonas originally worked with a small group of contributors to solidify a manifesto that discusses how the expectations for application resiliency have evolved over the last few years, and how applications should be engineered to react in a pre‐ dictable manner to various forms of interaction, including events, users, load, and failures.
“The Reactive Manifesto” states that “reactive systems” are responsive, resilient, elas‐ tic, and message-driven.

### Responsive
The system responds in a timely manner if at all possible.
In general, this means that the application should respond to requests very quickly. Users simply don’t want to wait, and there is almost never a good reason to make them. If you have a containerized service that renders large PDF files, design it so that it immediately responds with a job submitted message so that users can go about their day, and then provide a message or banner that informs them when the job is finished and where they can download the resulting PDF.

### Resilient
The system stays responsive in the face of failure.
When your application fails for any reason, the situation will always be worse if it becomes unresponsive. It is much better to handle the failure gracefully, and dynami‐ cally reduce the application’s functionality or even display a simple but clear problem message to the user while reporting the issue internally.

### Elastic
The system stays responsive under varying workload.
With Docker, you achieve this by dynamically deploying and decommissioning con‐ tainers as requirements and load fluctuate so that your application is always able to handle server requests quickly, without deploying a lot of underutilized resources.

### Message-Driven
Reactive systems rely on asynchronous message passing to establish a boundary between components that ensures loose coupling, isolation, and location transparency.
Although not directly addressed by Docker, the idea here is that there are times when an application can become busy or unavailable. If you utilize asynchronous message passing between your services, you can help ensure that your services will not lose requests and that they will be processed as soon as possible.



and... end of the book
