# Services 101

# It's all about Coupling.

![coupling](./coupling.png)

Eric Evans [DDD & Microservices: At Last, Some Boundaries!](https://www.youtube.com/watch?v=yPvef9R3k-M&index=18&list=PLinPBP3n4t5t9R7zF1fR8Ck3G3dC9BOmr)

Services (microservices) kao rjesenje ...? 

Doug McIlroy, the inventor of Unix pipes:
"(i) Make each program do one thing well. To do a new job, build afresh rather than complicate old programs by adding new features."

Unix philosophy: 
"1. Rule of Modularity: Write simple parts connected by clean interfaces."



# Code is a liability

![Code is liability](./code_is_a_liability.png)

# Services definition (~2004)

services are

![definition](./services_definition.png)

services are not

![service is not](./service_is_not.png)


# Small vs large

... kako 'prodavati' services

![small vs large](./small_vs_large.png)


# Paterns 

(nuzno da bi services arhitektura uspjela):

Central/common:
 
* communication patterns
  * sync
  * async 
* service discovery
* deployment system
* logging 
* monitoring system 


# Antipaterns 

(micorservice can go wrong):

* consistency (vs eventual consistency)
* synchronous communication
* shared libraries
* shared database



# Resources

Youtube [Arhitecture](https://www.youtube.com/playlist?list=PLinPBP3n4t5t9R7zF1fR8Ck3G3dC9BOmr) playlist

tl;dr
* Chad Fowler: [Kill "Microservices" before its too late](https://youtu.be/-UKEPd2ipEk?t=49)
* Clemes Vasters: [Messaging and Microservices](https://www.youtube.com/watch?v=rXi5CLjIQ9k)


books:
* Eric Evans: [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215/ref=asap_bc?ie=UTF8)
* Jez Humble: [Continuous Delivery](https://www.amazon.com/Continuous-Delivery-Deployment-Automation-Addison-Wesley/dp/0321601912/ref=sr_1_sc_1?s=books&ie=UTF8&qid=1493892201&sr=1-1-spell&keywords=contionus+delivery)
* [Enterprise Integration Patterns](https://www.amazon.com/Enterprise-Integration-Patterns-Designing-Deploying/dp/0321200683/ref=sr_1_1?s=books&ie=UTF8&qid=1493892281&sr=1-1&keywords=enterprise+integration+patterns)
* ers: [The Art of UNIX Programming](http://www.faqs.org/docs/artu/ch01s06.html#id2877537)





