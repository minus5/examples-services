# REST

We start with dummy system which consists of 3 services: *sensor*, *worker* and *app* ([example 1](https://github.com/minus5/examples-services/tree/master/01-http)).

<img src="./images/rest.png" height=300/>

*Sensor* generates random numbers and exposes them at HTTP interface. 

*Worker* is able to recieve some sensor data ad perform some heavy computation on it. In this showcase it only squares received numbers and returns them as a result. It also exposes this functionality on HTTP interface.

*App* orchestrates the whole process. It asks sensor for some data, sends the data to the worker, receives the result from the worker and writes it to log. 

## Building and running

We should be able to build and start the whole system as easy as with single monolith application. In system with multiple services we must have **automated building and running**. Without it development process tends to become slow and painful.

We can start our example by separately building and starting each service. For example, to build and run sensor service one sholud do:

```
$ cd ./01-http/sensor
$ go build
$ ./sensor
Started sensor at http://localhost:9001.
```

Since the procedure is similar for each service we can automate it. To do so we write a simple *Ruby* script (using [thor](https://github.com/erikhuda/thor) task runner):

```
desc "binary", "Build single go binary"
def binary(name)
  puts "#{name}: building binary"
  path_cmd(name, "go build")
end

desc "binary_all", "Build all go binaries"
def binary_all()
  binary('sensor')
  binary('worker')
  binary('app')
end
```

We use another task runner ([goreman](https://github.com/mattn/goreman)) to run and stop all the services at the same time. We instruct goreman which services to run by listing them in the [Procfile](https://github.com/minus5/examples-services/blob/master/01-http/Procfile):

```
sensor: ./sensor/sensor 2>&1 
worker: ./worker/worker 2>&1 
app: ./app/app 2>&1
```

Now we are able to build and start everything with two commands:

```
./build.rb binary_all
goreman start
```

## Orchestration

In given example *app* service carries the responsibility of process **orchestration** by leading it through series of actions:

- initiating the process every second
- asking the sensor for new sensor data
- asking the worker to perform calculations
- logging the result to log

Very important aspect of the orchestrator is that it keeps the overal **process workflow** on **single place**: in its source code. That source code resembles pseudo code with implementation of workfow actions delegated further to other services in the system.

## REST: second attempt

In the example orchestrator initiates the cycle every second with hope that there is some new data available on sensor. If the data on sensor is unevenly distrubuted throuh time (it almost always it) it could bring us ten events in one second and than a period of silence several minutes long. It would be much better to push the data from sensor to app the second it becomes available.

For that purpose we might try to rearrange the layout to allow *sensor* to initiate the the process. In that case the workflow might look like this:

- data becomes available on sensor
- sensor sends the data to worker
- worker crunches the data and sends the result to app
- app logs the result to log

<img src="./images/rest2.png" height=120/>

This layout has some significant improvents: 

- it starts to work instantly when the data is available on sensor
- it works only when there is data available

However, this layout is in many aspects a step backwards:

- sensor has now become aware that there are other services in the system (introduced coupling)
- sensor has become responsible to deliver the data to the worker
- the **overal workflow** of the system **is scattered around** number of services

# Messaging (NSQ)

In the [second example](https://github.com/minus5/examples-services/tree/master/02-nsq) we replace HTTP communication with **asynchronous messaging**. Every microservice attaches itself to the message queueing service and uses it as its only interface to other system components. In our system we are using [NSQ](http://nsq.io) distributed messaging system. 

<img src="./images/nsq.png" height=220/>

Services are not in any way aware of other services in the system. For that reason they are unable to choose which service should receive the message being send. For that purposes messaging systems use various **routing** algorithms.

Message routing in NSQ is organized around "topics" and "channels":

- publisher publishes the message to some named **topic** 
- each interested consumer opens his own named **channel** on that topic
- every channel will receive a copy of every message published on topic

Routing meshanism we just described is actualy as a classic **pub/sub** mechanism (worker subscribes to sensor messages).

The other common pattern is to distribute messages evenly between several service instances (**load balancing**). For example, if we introduce another instance of worker service all messages will be evenly distributed between all available workers (image from [NSQ docs](http://nsq.io/overview/design.html)). 

<img src="./images/nsq.gif" height=320/>

## Routing antipatterns 

NSQ supports those two routing patterns. Other Message Queuing Systems support plenty of others. However, one should be aware that complex routing algoritms are an **antipattern** in microservice architecture. Messaging component should be kept as *dumb* as possible; *smart* parts of tha application shoud be moved to the endpoints ([pictures by Martin Fowler](https://www.youtube.com/watch?v=wgdBVIX9ifA)).

<img src="./images/nsq-smarts1.png" height=270/>
<img src="./images/nsq-smarts2.png" height=270/>

## Distributed messaging

Crucial propery of NSQ is that it is **distributed** system. This prevents it from becoming single point of failure. Basic components of nsq system are:

- nsqd - messaging node
- nsqlookupd - discovery node (messaging DNS)
- nsqadmin - administration GUI

One can **simoultanously** use mutliple instances of **any** node type. 

When multiple nsqd instances are available in the system they all register at nsqlookupd. Publisher can publish messages to any nsqd node to any topic. Subscriber should ask nsqlookupd for list of nsqd nodes that have some messages from some topic and connect to them directly.

## Impact of messaging

Messaging has impact on many other aspects of the system:

- decoupling
- event driven design (event sourcing, replay)
- asynchronicity (better support for bursts of data)
- persistence (disaster recovery)
- routing (event broadcast, load balancing)

# Example 3: Service discovery (Consul)

Service discovery is the automatic detection of devices and services on a computer network (from [wikipedia](https://en.wikipedia.org/wiki/Service_discovery)). In our system we are using [Consul](https://www.consul.io/) for service discovery.

Usually there are plenty of components that are not able to communicate using messaging (databases, key-value storages, proxies, external web services...). Service discovery helps us locate those services in the system **by their name**.

Every service in the system registers itself to counsul by sending him its:

- name (e.g. mongodb)
- location (IP, port)
- health_check location

Consul will periodically poll each registered service for its health_check and inform other services about any changes in the service infrastucture. 

The only thing that remains to be manually configured within each service is a list of Consul locations; all other infrastructure information is obtained from Consul.

There are several ways to resolve some service location via Consul:

- setting up Consul as DNS (query DNS for mongo.service.sd)
- polling Consul by HTTP requests
- permanent TCP connection to Consul

It is very common to write wrappers for HTTP requests that will make service discovery by Consul transparent for the developers.

Some other neat features of Consul are:

- alerting (build upon health_check)
- leader election
- multi datacenters

Consul-template
---

[Consul-template](https://github.com/hashicorp/consul-template) is a small tool that facilitates configuration of services with current information from Consul. 

For example, to reconfigure Rails application with current service locations we can generate its config.yml from Consul template:

```
# Excerpt from Rails config.yml, ask Consul for current location of statsd
{{range service "statsd|passing,warning"}}
  statsd:
    server:    {{.Address}}
    port:      {{.Port}}
{{end}}
```

Within the container that runs this Rails application we run consul-template as background job:

```
consul-template -template \
  "/templates/config.local.yml:/apps/backend/config.local.yml:touch /apps/backend/tmp/restart.txt"
```

Whenever status of statsd service changes on Consul it will trigger rendering of *config.yml* and gracefully restart Rails application.

We have been using consul-template with various applications, both third party (haproxy, nginx, nsq, keepalived) and our custom (Rails, Node, ...). For Golang applications we use our [custom library](https://github.com/minus5/svckit/tree/master/dcy) that maintains constant connection with Consul without need for restarting the service.

#Containerization (Docker)

Adding new modules to the monolith application rarely impacts the development environment or production infrastucture. We want be able to instantiate new services just as easily. That's what Docker is here for.

In [example 4](https://github.com/minus5/examples-services/tree/master/04-docker) we setup our system infrastructure using [Docker](https://www.docker.com/). Each service gets its own docker container with its own OS and environment. Services are deployed to production by instantiating containers on docker hosts. 

Here are some basic terms used in Docker ecosystem:

- Dockerfile - a receipt for building image
- image - a blueprint for creating containers
- container - image mounted on a Docker host (does the actual work)
- [registry](https://hub.docker.com/_/registry/) - a repository for storing Docker images
- [Docker Hub](https://hub.docker.com/) - public Docker registry

## Dockerfile

Docker file is a receipt for building single Docker image. For each service (service, worker, app) we define [separate receipt](https://github.com/minus5/examples-services/tree/master/04-docker/images). Here is an example of receipt for our sensor service:

```
FROM gliderlabs/alpine:3.4     # start from existing image (download it from Docker hub)
COPY sensor /bin               # add my binary into image
WORKDIR bin                    # position myself into directory
ENTRYPOINT ["sensor"]          # when starting container start my binary
```

## Docker image

Docker image is created from Dockerfile receipt.

```
$ docker build -t sensor .
```

After createing the image we should be able to see it by listing all available images. Here we see that now we have locally available alpine and sensor image.

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sensor              latest              8d1f3a5ccb5c        5 seconds ago       11.7MB
gliderlabs/alpine   3.4                 bce0a5935f2d        13 days ago         4.81MB
```


## Docker container

Containers are created by mounting images on Docker host:

```
$ docker create sensor
$ dokcer ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cad9a64773b3        sensor              "sensor"            47 seconds ago      Created                                 elated_jackson
```

Containers are components that actually run our services. Once the container is mounted we use Docker commands to control it (start/stop).

## Docker Compose

[Docker-compose](https://docs.docker.com/compose/) is a tool enables you to define a set of containers that should be simoultanously started on a single host. Additionaly you can define the environment for every container (env variables, open ports, mounted volumes, ...).

In our example we [define](https://github.com/minus5/examples-services/blob/master/04-docker/datacenters/dev/host1/docker-compose.yml) that we want to have separate containers for:

- sensor
- worker
- app
- nsqd
- nsqlookupd
- nsqadmin
- consul

Also, in the same file for each container we define some enviroment varialbles, open ports and so on. This way we can use the same images on multiple stages (production, staging, development...) with some modifications in corresponding docker-compose files.

Now we can start **the whole system** on our local Docker host with **a single command**:

```
docker-compose up
```

## Docker Machine

By using [docker-machine](https://docs.docker.com/machine/) we can execute docker commands on remote hosts. This way we can easily control docker containers on remote hosts (i.e. in production).

## Infrastructure as a source

Very important aspect of containerization is that it allows us to have the **complete infrastructure** defined **in source** in a single place (single git repository). 

Our infrastructure is hierarchicaly organized in a tree structure:

- list of **datacenters** (dev, supersport, aws)
- each datacenter has a list of Docker **hosts** 
- each host has a list of running **containers** and their environment (*docker-compose.yml*)

## Continuous integration

In our *dev* datacenter we have several containers that manage the CI process.

**Builder** container reacts on every git commit made, builds a new image and pushes it to private Docker repository. We have defined several builders for different languages (Golang, JS, Rails).

**Deployer** awaits for remotely dispatched deploy commands and executes them on remote Docker hosts. It also commits every change to the infrastructure repository (every deploy is a change int the infrastructure). Deployer performs the deployment by running docker-machine commands on remote hosts.

