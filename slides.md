# Landing

Hello. Thank you for comming to my talk. 

I am Marin. I am a Senior Developer and a Team Lead at **minus5** here in Zagreb. 

Today I would like to share with you the experience we had in dealing with microservices. 

I am really happy and excited to be able to share it with you and I hope that you will enjoy this talk as much as I will. 

Estimated time for this talk is 40 minutes so feel free to sit back and relax.

# 3 answers

In this talk I will try to give you answers to these questions:

- Do you need microservices?
- If you start using microservices which problems should you expect?
- How did we solve those problems?

# SuperSport

### SuperSport
Our most significat client (and at the same time our partner) is SuperSport betting company. SuperSport is the largest betting company in Croatia (20TB monthly data transfer, 9M monthly business transactions). It started 12 years ago with several bet-shops. It was the first company in Croatia to introduce betting machines in public places (10 years ago) and also the first one to introduce online betting on the first day it became legally possible (7 years ago). Today SuperSport holds the dominant position in the betting industry in Croatia.

### Minus5 and SuperSport

We have created from scratch almost everything SuperSport has related to techology including web site (desktop and mobile), mobile applications for andriod and iOS, betting machines, betshop cash registers, various betting displays in the betshops, bookmaker interfaces, financial reports, administration tools... well, basically everythig related to technology they ever had. They are business oriented and we are technology oriented. We have grown to have their absolute trust in desinging and implementig everty product almost independently for them which is something that both sides enjoy and profit from.

### Pictures

(Picutres) This is our betting website, our mobile apps, andriod and iphone, ...

# SS cont.

... our betting machines and out betshops. As you can see I am reffering to theses products as ours as we have grown really intimate and emotional relation to them.

# Examples on github

### Find and run examples

Since I am a developer I had the need to to leave you with someting you can easily try yourself on your local machine. So I have created a couple of examples that go along with this talk. You can find them here. I have tested them on mac and on ubuntu and everything worked as expected so if you arre interested feel free to try the out. But please don't do it during this talk, I don't want to lose your attention.

### Strip and build up

I have designed the examples by looking at our system any by trying to strip it down to its most basic elements. And that is the first example. In the following examples I add the components and tools that improve the system bit by bit and then analyze what happend, what we got from it and where to go next.

### Written in Go

If you do take a look at the examples (later) you will find that they are written in Go. Go is the language we use for, well more than 90% of services in our backend. We have been using is since the very beginnings, like X years ago, and we have found it to be very appripriate for backend tasks. Also, somehow plenty of other tools that we use have also been written in Go. It always depends on the context of course but for us at this moment Go is the "way to go".

# Microservices

The first answer I promised to give today is: Do you need microservices. So lets see what they are and why would you ever use them.

### The term

The term microservices is not new and everyone seems to now what it means by now, but for me suprisingly it turned out to be a bit different than I initially percieved.

### Definition from microservices.io

I really like the description of the term I took from the site microservices.io which states that it...

> is an architectural style that structures an application as a collection of loosely coupled services, which implement business capabilities. The microservice architecture enables the continuous delivery/deployment of large, complex applications. It also enables an organisation to evolve its technology stack.

So, in case you missed it the crucial point is that microservices are: 

- **not coupled**, meaning that each service is autonomous to some degree,
- that they **implement business capabilities**, which means NOT organizing stuff around technical features but around business feratures,
- that they **enable continioius delivery/deployment**, meaning that developers are able to **make changes to single service and deploy** them without messing with other services in production, which enables faster delivery of peatures to production,
- and finaly it enables organisation to **evolve its techology stack**, since you can easily switch techologies in one service without affection others.

Each of those four crucial points is not about techology but about somethig completely on the other side of the spectrum, like autonomy, business, continuos delivery, evolving tech stack and agility. In case you thought microservices are about some technical aspects you were wrong, they are much more abstract term which talks more about organizing your system and your teams than the techology itself.

### MS vs Monolith

Microservices are usually described as an alterantive to the monolith application architecure. On one side of the spectrum there is a single application that handles everything and on the other there is a number of autonomus services working on separate tasks.

### Autonomy and Databases

When we say that service is autonomous we mean that service owns all of the state it immediately depends on and manages. That is why I have drawn all those little databases attached to each service. Each service can read only his database, it is not allowed in any way to make queries on the other services database. This restriction enables me to modify the database of my service without fear of breaking some other service which was coupled to my private database. That's where the agiliy and continuous delivery come from - from the abiliy to change things without fear. Or with less fear at least. For the same reason, service should not depend on any common data store. If I drew the common database down here that would give them all possiblilty to read each others data and we would be back to square one regardless of this application division to smaller chunks.

# Coupling

### Not techical

So, coupling is number 1 reason for having microservices. Notice that this is again not techical reason. It is not about performance or scaling the system for larger traffic.

### Greg Young

I run into a beautiful question from Greg Young's talk which goes like this: If you can't build a monolith what makes you think putting network in the middle will help?

Plenty of software systems end up stuck in the monolith application where nobody is able to change anything since every change reflects on something else in the system, every change breaks something else. So it is best to leave everything as it is. This hugely influences the business as the competition takes bigger and bigger portions of the market until the business together with the monolith application dies out. So engineers search for the solution for their monolith problems and usually find the answer in microservice arhitecrture. 

And than this question pops up.

Indeed, why would putting the network between system components ever improve the system?

### Answer: Developer Dicipline and Physical boundaries

The answer is a bit sad (for me at least) and it is "**developer discipline**". If you don't put physical boundaries between two software components (like modules or database tables) they are verly likely to become coupled very soon. So developers usually think "let me just pick this data from this database and I'm on my way". Everything seems to work, the application is shipped to production and everybody is happy. And it can go this way for quite some time until it all becomes to coupled to make any further changes.

### There has to be some coupling

The individual coupling is never problem per se, but the bunch of couplings in the long run are. Of course, there has to be some coupling in the system, somebody has to actually fetch the data from the database. But coupling is something that should be handled with extreme caution. And in that sense, microsevices do seem to help.

# Latency

### Database queries

Coupling may bring another side-effect which emerges in production environments: latency. Executing database queries on multiple rows/tables within single transaction triggers locking mechanisms which may cut down the overall performance. 

### Eventual consistency

Microservices usually trade database consistency for eventual consistency to improve latency.

Consider the following workflow for handling a betting ticket request: (picture)

This workflow spans four disjoint services: tickets, odds, accounts and authorization. Doing everyting within single database transaction would lock up all services until the ticket is placed in the database. By separating those entities into autonomous services we have improved the latency and reduced data consistency level.

What could possibly go wrong? (dramatic pause)

### Distributed saga

Well, a lot can go wrong. A common pattern for handling problems that emerge from service decoupling is distributed saga pattern. In the basic terms is says is that you should: 

- make every action idempotent, which means that you can repeat every action as many times as you want without changing the outcome,
- for every action make reversible action, so you can cancel every step if you decide to,
- make order of actions irrelevant, so you can first cancel some action and than do the action later with exactly the same outcome

### Compensate manually

So, we have given up the good old database consistency and now we are doing everything we can to compensate. Which is kind of hard to get used to because it requires additional effort from developers. And they cannont help but think "omg, why are we doing this, this would be a one-liner in SQL". So you constantly have to remind yourself why are you doing this to maintain your sanity.

### Warning 1: Unexpected Extra Work

The bottom line is - microservices may bring along a lot of extra work outside of the core domain and lot of edge cases you have to handle manually.

# Technical debt

### No quick fixes

If you listened carefully you might have noticed that now it is not possible for one service to reach into another services database. So what happens it your manager comes in one day and asks you to make a quick fix by grabbing that data from another database?

Decoupling of services make quick-fixes a bit harder. The interface each service has is strictly defined and can not be violated. A service is not able to reach out to another's data storage; that communication must be explicitly defined.

From a business point of view it is very nice to have ocasional quick-fix and to leave the actual full-size fixes for later. Such full-size fixes tend to fall out of focus since everything seems to work nicely and there is no evident business reason to do them. This slowly accumulates technical debt which can be very harmful for business in the long run.

Just to be clear, that is exactly what we wanted to achieve in the first place by using microservices: to restrict accidental or non-accidental coupling!

# Scalability

Scalability is obviously a huge, huge benefit you get from doing microservices. In a sentence: it is easier to scale a distributed system then monolith.

# REST

Hopefuly that shuold be enough to give you the idea what are we trying to solve and get us on the same page. Now it's time to move on to the examples I promised.

### Components

We start with 3 services: `Sensor`, `Worker` and `App`.

`Sensor` generates random numbers and exposes them at HTTP interface. 

`Worker` is able to receive some sensor data ad perform some heavy computation on it. In this showcase it only squares received numbers and returns them as a result. It also exposes this functionality on HTTP interface.

`App` orchestrates the whole process. It asks `Sensor` for some data, sends the data to the `Worker`, receives the result and writes it to log. 

In given example `App` has the responsibility of process **orchestration** by leading it through series of actions:

- initiating the process every second
- asking the `Sensor` for new data
- asking the `Worker` to perform calculations
- logging the result to log

### Orchestration

Very important aspect of the orchestrator is that it concentrates the overall **process workflow** on **single place**: in its source code. With implementation details delegated further (to other services) process workflow becomes much easier to grasp.

### Second attempt

`App` initiates the cycle every second with hope that there is some new data available on `Sensor`. If the data is unevenly distributed through time (it almost always is) it could emit ten events in one second and than have a period of silence several minutes long. It would be much better to push the data from `Sensor` to `App` the moment it becomes available.

For that purpose we might try to rearrange the layout to allow `Sensor` to initiate the process. In that case the workflow might look like this:

- data becomes available on `Sensor`
- `Sensor` sends the data to `Worker`
- `Worker` crunches the data and sends the result to `App`
- `App` logs the result

This layout has some significant improvements: 

- it starts to work instantly when the data is available on `Sensor`
- it works only when there is data available

However, this layout is in many aspects a step backwards:

- `Sensor` has now become aware that there are other services in the system (introduced **coupling**)
- `Sensor` has become responsible to deliver the data to the `Worker`
- the **overall process workflow** of the system **is scattered around** number of services

### Surprise: not possible without drawbacks

So it actually came to me as a surprise that I was not able to build even a simple system such as this one withot significant drawbacks. For a long time I believed that the whole system could work just by using HTTP requests. So I was a bit disappointed with the result. Nontheless, it is still a very interesting observation that building system with only synchronous requests is very limitig factor.

# Messaging

So it is not much of a surprise that the next step is the introduction of asynchronous messaging. 

Every microservice attaches itself to the message queueing service and uses it as it's only interface to other services. In our system we are using [NSQ](http://nsq.io) distributed messaging system. 

This is a second example in our examples repository.

### Routing intro

Services are not in any way aware of other services in the system. Message producers are unable to choose which service should receive messages being send. Instead they use various routing algorithms.

# Routing

### Topics and channels

Message routing in NSQ is organised around *topics* and *channels*:

- publisher pushes the message to some named **topic** 
- each interested consumer opens his own named **channel** on that topic
- every channel will receive a copy of every message published on topic

### Pub/sub and load balancing

Routing mechanism we just described is equivalent to classic **pub/sub** mechanism (`Worker` subscribes to `Sensor` messages).

The other common pattern is to distribute messages evenly between several service instances, know as **load balancing**. That is also very simple to achive by attaching to exisitig channel. 

So if I open a channel whith a brand new name I will get all messages (thus implementig pub/sub) and I open a channel with existing name I will share messages with some othere services (thus implementing load balancing).

### Not statically configured

It is important to notice that NSQ routing is **not statically configured**. It is up to services to decide which topics and channels to open. This way **message routing is completely managed by services**.

### Anti patterns 

There are many other routing patterns available. However, one should be aware that complex routing algorithms have been recognised as an anti-pattern in microservice architecture. Messaging component should be kept as dumb as possible; smart parts of the application should be moved to the endpoints. If message routing is too smart it becomes too hard for developers to track down what is happening in the system and we lose the agiliy again. Which defeats the whole purpose of decoupling.

# Messaging edge cases

### Stale

What happens if some consumer goes down and **misses some messages**? Messages will wait in the queue and be delivered when service comes up again. In most cases that is a wonderful feature but it might have some dangerous side effects. The effects of stale messages have to be **considered separately depending on the context**.

### Missed

What happens when some messages have not been handled properly? One common solution is to allow consumer to ask the publisher to **replay the data missed**. For such purposes publishers should have additional interface for data replay queries. Consumer should be able:

- to detect that he missed some messages
- to send query to the publisher to replay data missed
- to handle messages idempotently
- to make **full state recovery** when too much data has been missed

### Order

Also, one has to be aware that messages may arrive in **diferent order** than being sent. How do we deal with that?

### Warning 2: Messaging edge cases

All of these problems are outside of the domain you are dealing with. They are not in any way related to betting or sport or odds or bookmakers. But now your developers are handling bunch of edge cases which came together with separation of single monolith to multiple services. The system is distributed and you constantly find yourself designing your services around those edge cases caused by architecture and not the root domain problems.

# NSQadmin

This is a screenshot of admin interface for NSQ messaging also know as nsqadmin. I took it from our production system. I contains a lot of information but lets take it step by step.

On the top of the page is the name of the topic, Betradar. Betradar is our business partner which provides us a stream of live odds. So somehow underneath we are connecting to their servers, take the information from them and strem them into our system as a stream of messages.

In the first table we can see that we actually have two sources of Betradar messages. But if we look a little closer we see that the first source is the only one sending the messages, the other one is not there but not doing any actual work. So this one might be a backup instance waiting to take over the job or the stale one ready to be terminated.

Also we see that since the beginning of the procces this producer hac produced 11 million messages.

In the second table we see a list of channels open on that topic. From the names of the channles we can usualy deduce what application is using it. That is developers are behaving nicely. 

The most important one here `tecajna` service. Other services are:
- nsq_to_log which are soring nsq messages to logs obviously, 
- nsq_to_ws which replicate those messages to our stqging areas, namely staging 2 and staging 3. So this way we are supporting our staging areas with the same live fedd which or production side has. Which is very nice bonus feature of messaging.

So here we have 2 producers of messages (or 1 to be honest) and we have 4 consumers of messages which all receive a copy of every message. So this is an example of pub/sub. If we started another instance of tecajna service they would automatically start to load-balance messages.

Depth 0 is telling us that there are no messages waiting in the queue, in-flight 0 tells us that there are no messages being processed at the moment, and ewe can also see that some messages have been requeued and that some messages have been "timed out" whic indicates that there have been some problems with those messages in the past. 

Also, on top of the page there is a small control panel by which we can empty queue when someting goes terribly wrong. Or to pause message delivery when doing some potentialy risky stuff like moving containers from one host to another or something like that.

So we can get a lot of information about the health of our system just by looking at this dashboard. It is usualy very helpful to check this dashboard when trying to understand what is going on in the system.

# Service Discovery

### REST is still alive

Even though messaging is the preferred way to communicate we still have a lot of communication done using REST interfaces. Also, there are some components in the system that are not able to communicate using messaging. For example there are databeses, key-value storages, web proxies, externale web services and so on. 

So we have reduced coupling to some extent but there is stil a lot of coupling to be resolved. 

### Classic solution: confing and env

So I have a service that needs to makes some HTTP requests to some other service. How do I explain to him where to do it? The classic solution is to use some config files or to read configuration from environment.

But now we have a system that is very dynamic, new containers are put to production or removed from production every now and then. We dont want to reconfigure every service manually. Instead we use **Service dicovery**.

### Service Discovery

So we introduce another component into the system that has the responsibility of registering every service in the system as soon as it appears in the system and deregistering them as soon as they disappear.

The tool we use for that is named Consul. This is how it works.

### Service registration

Every service in the system **registers itself** to Counsul by sending him its:

- name (e.g. *mongodb*)
- location (IP, port)
- *health_check* endpoint

Consul will periodically poll *health_check* endpoint of each registered service and inform other services about any changes in the service infrastructure. 

# Service resolution

There are several ways to resolve service location via Consul.

The first option is to setup Conusul as DNS resolver. So other services are configured to make requests to some DNS names, like for example mongo.service.sd, and will receive the actual IP and port of the sercvice.

The other option is to ask Consul the actual service location before making each reuqest. But this ends up as to much unnessecary queries being made. They all get the same response. So you can cache your requests and so on.

Consul also supports events. Service can attach itself to Consul and listen to its events. Wheneves someting changes in the system infrastructure Consul emits events and application react accordingly. So services don't have to poll anymore.

For third-party applications, where you cannot change the way they work there is a solution to reconfigure thir config files and to gracefully restart them. We have been doing this with nginx config files, Rails config files, proxies, and meny others. For example we create nginx config file as a template that is filled with values from Consul and gracefully reload nginx on every relevant change on Counsul. 

# Consul admin

This is another screenshot from our production. This is admin interface for Consul. On the upper right corner there is a dropdown where you can select the datacenter you are exploring. I have selected our s2 datacenter. You can see that we have some other interesting datacenters like for example several staging datacenters. 

On the left side is the list of the services in s2 datacenter registered to Consul. Also, you can see that everything is green which means that every service is running as expected. Consult know that by periodically polling health_check for every service. So for example we can see that for selecetd service dib there are 4 healt checks passing. And so on.

On the right side we see that there are 2 instances of `dbi` service started, one on node app1 and the other on node app2. They are also green so we are all clear, everything is working as expected.

In case something was wrong we would see which services are down and what health_checks are not passing. Of course, now we can attach some notifiers to Consul to alert us whenever something goes wrong.

# Alerting in Slack

And this is how it looks like. This is screenshot from our alerts room in Slack to which we send all notifications from consul.

This first message is alerting our developers that something is wrong with http_to_nsq service. Also we can see the details of its health_checks. So here it says, service is down because it can no longer access some API, namely 5-lotatoe.suopersport.local.ping. 

In the next line, one minute later, you can see that I am telling everybody else that it is my fault and that I am doing someting and that I will fix it ASAP.

And then two minutes ahead we can see that we are back to normal state and that everything is working fine.

In this line we can see another type of alert. This one says that the queue deply for this nsq channel is in constant increase for last 8 checks. That can mean either that application is down and noone is consuming the messages or that application is consuming messages slower than they appear in the queue. So somebody should probably do sometihng about it. As far as we can see here nobody has reacted to this alert and it cleared out by itself. This probably means that we should adjust our parameters for alerting for this specific case.

# Containerisation

### Monolith and new modules

Adding new modules to the monolith application rarely has any impact on the development environment or on the production infrastructure. We want be able to instantiate new services just as easily.

### Docker

So this is how we do it using Docker. For each service we write a recipe which specifies requirements for running some service. For example we say we are going to need this version of Linux, this version of Ruby and so on. In this step we dont actually install anything, we just write what we are goinng to need in the text file. This is just a recipe.

The next step is to build image from the recipe. In this stem Docker takes the recipe, acually collects everytihn fro web, writes it down to image which is then ready to run our service.

In the final step we run this image. In other words we create running container with our service in it.

### Infrastructure as a code

Now, I wouldn't like to go into details of how Docker works. The reason why I am telling you this is that by using this we have all of our infrastructure written in source code. 

This is very beautiful feature. It is a bit hard to appreciate its full beauty if you have never went through agony of manualy installing OS and software requirements for every server you have.

# Containerisation features

### Decoupling levels by Greg Young

Containerisation adds another level of decoupling to services. Now each service can have its own OS, its own library versions and so on. So now I can change version of Ruby for one service, deploy it and be sure that other sercvices are left untact.

I could run all services in a single OS process. If I did this my costs would be low. But I would have no way of knowing that another process is not looking at my memory directly. On top of that, what happens when one service chrashes? It takes down the whole OS process down which takes down all services.

I could go to the next level. I could run one OS process per service on the same machine. Now I've got better failure isolation but my costs are higher because I have to maintain all these processes. Anoter benefit is that I can use standard OS tools to manage my processes. Like for example killing and restarting the problematic process. 

Another level of isolation is running Docker conatiner per service. Another level is running on multiple nodes.

Each step increases overal cost. But you don't have to make that decision up front. 

Another feature is that I can try the service in exactly same environment as in production and deploy the same image to production.

Another feauture is that you get the command line toolset for service management. So now you suddenly have tools to deploy sometihng to production, to restart service, to start or stop it. 

So all together you get very nice set of tools to use.

# Infrastructuire as code

So as I have mentioned now we have all out infrastructure in sorce code. To be more specific, we have a single git repository which describes everything.

I that repository first you will find is a list of datacenters we have. In each datacenters whe have defined a list of nodes we have.

On each node we have a list of list of Containers. 

So every change to infrastucture is commited to this git repository. So when we instruct our deployer application to deploy some image to some node it actually makes changes to our git repository and commits them automatically. So you can always be sure that the last version of the infrastructure is in git repo.

# Continuous integration

*Application builder* reacts on every commit in the source code and **builds application binaries**. We have separate container for each target technology: *Go* builder builds *Go* binaries, *Rails* builder precompiles web assets, *JS* builder builds web pages using *Webpack* , etc. 

*Image builder* puts together prepared binaries with *Dockerfiles*, **builds new Docker images**, tags them and pushes them to our local Docker registry.

*Deployer* **executes deploy commands** on remote Docker hosts (using *docker-machine*). It also commits every change to the infrastructure repository. Every deploy is a change in the system infrastructure. Reverting is equivalent to deployment of older image.

# Infrastructure examples

Picture below shows the overview of the final arhitecture. Two datacenters are shown, *dev* and *aws*. Each datacenter has several Docker hosts. *Dev* datacenter has two: *dev1* and *dev2*. Each Docker host has several containers running. Host *dev1* has eight.

What happens when new `Worker` is added to the system?

- new `Worker` container is created on Docker host
- `Registrator` gets the event from Docker and registers new `Worker` to `Consul`
- `Consul` informs all interested services that new `Worker` is available
- `Worker` asks `Consul` where he can find `nsqlookupd`
- `Worker` asks `nsqlookupd` where he can find `nsqd` with topic `sensor_data`
- `Worker` connects to `nsqd` and starts receiving messages


# Conclusion

We have reached the end of this talk. I hope I gave you the answers I promised:


- Do you need microservices?
- Which problems should you expect?
- How did we solve them?

# Thank you

