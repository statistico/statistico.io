---
title: "Technology Overview"
date: 2023-11-08T14:10:54Z
draft: false
author: "Joe Sweeny"
tags: ["Business", "Technology"]
image: "/img/aws.jpg"
---
The Statistico technology ecosystem is deployed within AWS and consists of:

- A suite of containerised API based microservice applications served via an application load balancer.
- Small executable functions deployed using serverless technologies.
- Asynchronous background processes using an event driven architecture.
- A web visualisation platform deployed using S3 and Cloudfront.

Each microservice application is backed by an RDS instance using the Postgres engine and Elasticache is used 
as the temporary data store. Lambda is utilised to execute functions on a schedule for background processes such as
odds collection, odds compilation and trade placing/resolution. For asynchronous data handling SNS and SQS are used to 
distribute messages to multiple consuming applications using Lambda event source mapping to trigger functions.

## Technology Decisions
As mentioned in [What is Statistico?]({{< ref "/content/blog/20230109-what-is-statistico.md" >}} "What is Statistico?"), 
Statistico's origin is based upon as a passion project that provided an exposure to new programming languages and exploring 
new technological concepts. A lot of thought and consideration was put into the languages, architecture and protocols that Statistico
is built upon. The high level concepts and key decision drivers were:

- PHP was the server side language I had the most commercial experience in. 
- I wanted to explore new languages. 
- The languages chosen had to be high performing and relevant i.e. by learning these languages it provided career growth opportunities. 
- I had extensive experience in RESTful API development. Should I look to explore another communication framework?

### Well Scoped Applications

Prior to any development, I had the ideology in my mind that I would build a suite of independently deployable applications
with very well-defined scope and bounded context. Building small and well scoped applications would provide me with the
opportunity to build applications in different programming languages, allowing me to choose the correct tool for 
the job and would also allow me to pivot to a new programming language with minimal uplift should I wish to explore another
language or deem there is a better tool for the job.

### Speed Is Key

I had a desire to build each application with speed in mind. The element of speed was a key driver due to:

- The amount of fixture, goals, cards, corners and team statistical data being handled.
- Placing trades with live sports betting exchanges ensuring liquidity in the markets.
- Multiple asynchronous processes running concurrently and continuously throughout the day.

[Go](https://go.dev/) (or Golang) immediately caught my eye due to its inbuilt concurrency paradigm which met my need for speed. Concepts 
such as Goroutines and channels were an entirely new concept that I felt that I needed to explore. Add to this, Go's 
strong typing system it felt like a match made in heaven for the engineer who leans towards the *strong typed side of life*.

With Go as my initial language of choice I felt like a needed to explore options beyond the standard HTTP protocol and
RESTful interfaces as I had become more than accustomed to within my day to day employment. Writing OpenAPI documentation through to
devising databases schemas for clean RESTful APIs was my bread and butter. I needed something new and exciting that
broadened my horizons. 

The main communication channels within the Statistico ecosystem were due to be application to application. [gRPC](https://grpc.io/) 
felt like a match made in heaven with Go due to:

- gRPC request and response streaming fitting hand in glove with Go's channels and Goroutines for low latency and high performing applications. 
- Defining schemas in *.proto* files provided nice and clean application interfaces.
- Automatically generated idiomatic client and server stubs for each application in a variety of languages was a perfect fit for the microservice architecture that was being adopted.

### Right Tool For The Job

Determining the programming language for the odds compilation application was the next piece in the jigsaw. Having adopted
Go as my language of choice for the majority of the services I had outlined, unfortunately using Go for my odds compilation
service felt like *a square peg in a round holes*....let me explain. 

I planned to utilise machine learning algorithms (Generalised Linear Models and Poisson distribution calculations), 
Python and R trumps Go in every department. I opted for Python due to the low barrier to entry, a huge supportive community and great documentation. 

### The Icing On The Cake

With a suite of APIs designed and implemented, the icing (and the cherry) on the cake was producing a visual representation of
the odds and trade related data by building a clean and intuitive user interface. I opted for [Vite](https://vitejs.dev/), using React and Typescript, for a lightening
fast development experience that provided me with an option to move away from the defacto at the time, Create React App. 

In a surprising turn of events I took a leap into the unknown by using end to end gRPC using [protobuf-ts](https://github.com/timostamm/protobuf-ts)
and taking advantage of the [AWS application load balancer support for end to end HTTP/2 and gRPC](https://aws.amazon.com/blogs/aws/new-application-load-balancer-support-for-end-to-end-http-2-and-grpc/).

## Statistico Applications

The following diagram provides a high level overview of the Statistico applications, the application to application
communication channels and the programming language each application is written in.

![Statistico Applications](/img/statistico-applications.png?)

### Statistico Football Data
Written in Go and is responsible for fetching, storing and serving football related statistics such
as teams, players, fixtures, results, goals, card, corners, team based and player based statistics. Processes
are executed on a cron schedule at set periods throughout to day to continually ingest data.

### Statistico Ratings
Written in Go and is home to Statistico's unique team rating algorithm that helps power future odds' compilation. Using
team based football statistics, this application runs on a daily schedule to consume result based data to formalise rolling
team based attack and defence ratings. Statistico Ratings communicates with Statistico Football Data to ingest historical fixtures and statistics
to build fixture related team attack and defence ratings in order to compile odds.

### Statistico Odds Compiler
Written in Python, this application is the central node of everything Statistico. It uses machine learning algorithms and is responsible 
for compiling Statistico's own odds and probabilities for specific football matches and betting markets to be able to compare against the exchanges.
Statistico Odds Compiler communicates with Statistico Football Data to ingest historical fixtures and statistics and Statistico Ratings
for team attack and defence ratings per fixture.

### Statistico Odds Checker
Written in Go, this application runs on a frequent cron schedule fetching current odds from numerous well known exchanges and bookmakers
such as Betfair, Pinnacle and BetCRIS. Odds are continuously published into SNS to be consumed by numerous subscribing applications.

### Statistico Odds Warehouse
Written in Go, this application consumes published odds and stores them within a Postgres database for current and historical
reference. Statistico Odds Warehouse provides a programmatic interface allowing odds to be fetched and visualised within the
Statistico Web Platform.

### Statistico Trader
Written in Go, this application is the heartbeat of the Statistico philosophy. This application provides the creation of league
and market based strategies, when having a status set to live, consumes published odds and determines whether a trade should be placed.
If an inefficiency is exposed, a trade a placed using integrations with leading sports betting exchanges. Statistico Trader communicates with
Statistico Football Data for fixture related data and Statistico Odds Warehouse for the current odds' data to determine
if a trade should be placed.

### Statistico Envoy Proxy
Utilising [Envoy Proxy](https://www.envoyproxy.io/) this application provides a containerised solution that serves as a proxy
to translate HTTP/1.1 requests produced by the client (Statistico Web Application) into HTTP/2 calls that are expected
by the suite of gRPC based microservice applications.

### Statistico Web Platform
Written in React and Typescript while leveraging the Vite library, this application serves as a web based graphical interface
to, at the time of writing, provide an insight into current market odds and ongoing strategy performance.

#### Future blog posts will dive deeper into each of the above applications to provide a more granular overview.

## AWS Infrastructure
Statistico applications are deployed within the AWS ecosystem. The following diagram provides a high level overview of
the infrastructure components. This diagram provides an example of a container based and database backed microservice (such as [Statistico
Football Data](#statistico-football-data)):

![Statistico Infrastructure HLD](/img/statistico-aws-high-level-architecture.png)

**VPC**: Statistico infrastructure components are deployed with a VPC (Virtual Private Cloud) which is a virtual network
dedicated to the Statistico AWS account. It is logistically isolated from other virtual networks within the AWS Cloud.

**Load Balancer**: Is the single point of contact for clients, within the world of Statistico, serving requests from the 
Statistico Web Platform. The load balancer distributes incoming application traffic across multiple targets, Statistico's applications using ECS, 
in multiple availability zones. Using multiple targets across multiple AZ's increases the availability of Statistico applications.
An internet gateway is used to proxy ingress traffic from the web.

**Public Subnet**: 2 x public subnets are deployed within the VPC, each within a different availability zone. Public subnets
are accessible to the public internet.

**Private Subnet**: 2 x private subnets are deployed within the VPC, each within a different availability zone. Private subnets
do not have publicly accessible IP addresses and can only be reached via the public facing load balancer. To preserve
our security posture, application and database resources are deployed within the private subnets preventing direct
and unauthenticated / unauthorised access from the outside world.

**NAT Gateway**: A NAT gateway is deployed within each public subnet. A NAT Gateway is used a gateway to route traffic to the
outside world but prevent ingress traffic from the outside world. The collection of statistics and placing trades from private subnet
based applications will route traffic via the NAT gateway to perform the necessary actions with third party providers such as
Sportmonks and Betfair.

**S3**: Compiled Javascript/Typescript assets that contain application code is deployed to a privately accessible S3 bucket.

**Cloudfront**: Cloudfront is used to serve the Statistico Web Platform application over a secure network (using HTTPS) and at speed 
using edge based caching for a low latency. An S3 bucket policy providing access only to the Cloudfront distribution is configured for optimal security.

**ECR**: Applications such as [Statistico Football Data](#statistico-football-data) and [Statistico
Trader](#statistico-trader) composed of application code are deployed as a Docker images to ECR. Each application has
its own independent repository with versioned images per deploy.

**ECS**: A task definition, referencing an ECR based Docker image, is then
defined and an ECS service is deployed based on the task definition for each individual application. ECS is used to not only
serve these applications, auto-scaling policies are defined to allow each service to scale up and down upon demand with
ECS providing these orchestration capabilities out of the box. Statistico uses Fargate due to its ease of use and
offsetting the server management and patching to AWS.

**Target Groups**: The application load balancer has a URL configured target group configured to understand which application to forward each request to.

**Service Discovery**: AWS Service Discovery is used to provide each ECS service with a custom DNS name. AWS Cloud Map API
is used for HTTP and DNS namespaces providing ECS with a way automatically register itself with a predictable and friendly DNS name.

**RDS**: Each microservice is backed by an RDS instance to truly keep each microservice independent and scalable. RDS
instances are only accessible by their respective application / microservice with access locked down based on a security group
to security group configuration. The Postgres engine is used for each microservice currently within each Statistico microservice. 
RDS provides a managed service that allows Statistico to set up, operate and scale data persistence stores with ease.

**Elasticache**: Certain data points within Statistico are not as frequently changing as others so Elasticache (using the Redis engine) is used
for applications with these use cases. For example, the result of a football match does not change once finished. Elasticache
improves responses times and eases the load on persistent data stores such as RDS.

*To summarise, Statistico applications are deployed within private subnets to keep application logic and data stores secure.
Access to each application is routed via the publicly accessible load balancer with target group configuration providing the
load balancer with an understanding of which microservice to proxy each request to. Traffic routed from within to outside the network is
routed via the NAT Gateway.*

## Event Driven Architecture
The section above explains the architecture for outward facing consumer interfaced applications, the following describes 
in detail the background processes that powers Statistico using an event driven architecture. 

![Statistico Background HLD](/img/statistico-background-service-hld.png)

**Lambda**: Lambda functions are deployed to provide small and compact executables containing single context functions.
Deployed Lambda functions include:

- Stats fetching - [Statistico Football Data](#statistico-football-data)
- Odds fetching - [Statistico Odds Checker](#statistico-odds-checker)
- Odds consumption and data storage - [Statistico Odds Warehouse](#statistico-odds-warehouse)
- Odds consumption and trade placing - [Statistico Trader](#statistico-trader)
- Trade resolution - [Statistico Trader](#statistico-trader)

**Events Rule**: Event Rules are configured within Event Bridge to execute a Lambda functions on a pre-defined
schedule using cron expressions.

**SNS**: Executed cron scheduled Lambda functions push messages into SNS topics. SNS topics are used to provide a scalable
solution meaning additional consumers can be added as subscribers with minimal configuration as the Statistico ecosystem grows.

**SQS**: Each consuming application has an SQS subscription to the relevant SNS topic i.e. [Statistico Odds Warehouse](#statistico-odds-warehouse)
has an SQS subscription to the [Statistico Odds Checker](#statistico-odds-checker) SNS topic to consume and persist odds
data for current and historical context.

**Event Source Mapping**: Lambda event source mapping is used to configure a mapping between a Lambda function and an SQS
queue. This mapping allows messages to be consumed from the SQS queue as soon as a message is available to be consumed
and processed by the mapped Lambda function. This mapping provides an elegant, cost-effective and scalable message
processing solution.

## Summary
Statistico is built upon the philosophy of small single use functions and domain driven microservice applications,
deployed using serverless components, following industry best practices within AWS. 

gRPC is used for application to application
communication for low latency responses and bidirectional streaming. gRPC is also for used end to end, client to server communication using
HTTP/2 translation via a proxy layer. 

Microservices are built with the philosophy of being rebuilt in another language that
may be deemed a better tool for the job.

## References
- [What is Statistico?](https://statistico.io/blog/20230109-what-is-statistico/)
- [Statistico Web Platform](https://platform.statistico.io)
- [Statistico - GitHub](https://github.com/statistico)
- [gRPC](https://grpc.io/)
- [Envoy Proxy](https://www.envoyproxy.io/)
