# Microservices introduction course

This document is a notebook for Microservices introduction course by Sean Campbell on Udemy:

* [Microservices: Designing Highly Scalable Systems](https://udemy.com/course/introduction-to-microservices/learn)

Enroll to the course for more information, explanation and examples.

## Principles of Microservices

* Microservices should not share code/data
* Unnecessary coupling between services should be avoided
* Independence over code re-usability
* There should be no single point of failure (point that will cause inavailability of more than one service when broken)
* Each service should be responsible for single function
* Microservices should use event/message bus to communicate between them (direct communication not allowed)

## Benefits and anti-patterns

### Benefits from microservices architecture

* Improves modularity
* Reduces complexity (smaller, easier code per microservice)
* Easier functionality update without impact (or with low impact) on rest of the system
* Better team collaboration organisation (when working on one bigger system)
* Continuous delivery and deployment of system in "divide and conquer" maner
* Each service (so functionality) can be deployed independently
* Highly scalable architecture
* Possible deployment on many different environments (cloud, on-premise)
* Faster ramp-up of new team members

### Anti-patterns and false opinions on microservices

* Everything in microservices but one big database
* Microservices as a remedy for poor development and deployment processes
* In microservices architecture development teams do not have to cooperate
* Making key focus on technology behind microservice

## Microservices architecture

* Building blocks:
  * Clients (web app, mobile app and so) - communicates with API gateway
  * API gateway - communicates with Clients (outer world) and services (inner world) including authentication service
  * Services - communicate with API server and Event bus (does not concern auth. service)
* RESTful API
  * Microservices are mostly REST API servers (provides functionality through REST API)
  * Realize CRUD operations (Create, Read, Update, Delete)
* API operations (HTTP methods)
  * POST - create new object
  * GET - read existing object(s)
  * PUT - update existing object (full)
  * PATCH - partially update existing object
  * DELETE - delete object

As POST can also return data it should be used instead of GET when complex data description or some sensitive data are part of read request.

### HTTP status codes

* 1xx - Informational
* 2xx - Successfull execution (with or without data returned)
* 3xx - Redirections
* 4xx - Client request errors (invalid, non authorized, forbidden, not found)
* 5xx - Servers side errors

## Data management

### CQRS - Command Query Responsibillity Segregation

* Commands (operations that alters data like PATCH or DELETE) and Queries (operations that return states like GET) should be handled separately
* why CQRS:
  * Data are more frequently queried than modified
  * Separation of C and Q allows to optimize each of them separately
  * It safer to have separate models for commands and queries (data contention)
  * Differences between read and write representation of data (?)
  * Separation allows easier security rules implementation
* Also databases for Commands and Queries should be separated.

### Event sourcing

An approach where all changes are stored as a sequence of events (in event store). It's at the same time logs of performed changes (history) and data for describing current state.

Event store is a "write database" while separate "read database" stores current state for queries.

One event store can be used to feed more than one databse.

## Tools apps and libraries related to Microservice architecture (some examples)

### Container technologies

* Docker
* CoreOS rkt
* LXC containers
* OpenVZ
* containderd

### Orchestration technologies

* K8s
* Docker Swarm
* OpenShift
* CoreOS Fleet
* some proprietary (Amazon, Azure)

### Service discovery

* etcd
* Consul
* Apache Zookeper
* Smart Stack
* SkyDNS

### API Gateways

* Kong
* Ambassador
* Ocelot
* Tyk
* KrakenD

### Event bus technologies/tools

* Apache Kafka
* RabbitMQ
* some proprietary (Amazon, Azure, Google Cloud)

### Logging tools

* Fluentd
* Graylog
* Kibana
* Logstash
* Bunyan
* Suro

### Monitoring tools

* Grafana
* Prometheus
* cAdvisor
* Riemann

### Documentation tools

* Swagger UI
* Apiary
* Readme.io
* Slate

### Testing tools

* Postman
* Hoverfly
* Pact
* Gatling
* Insomnia (not from course, self-added)

## Moving from monolith into Microservices guide

List of step-by-step actions for application (system/solution) redesign/refactoring:

* Identify all features in existing monolith
* Define microservices based on the list above (having all principles in mind)
* Select non-critical microservice(s) for proof of concept implementation
* Focus on REST API side first (before implementing event bus)
* Implement API gateway
* Use existing database(s) until full functionality converted to independent microservices
* Identify processes that are now handled by more than one API
* For each microservice define events that will trigger actions in other microservices
* Implement event bus. Start event based communication between two non-critical microservices
* Decompose existing database(s) into DB per service
