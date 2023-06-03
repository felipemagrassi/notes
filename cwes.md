# Software Architecture - 2023-05-16 21:20:34.792123326 -0300 -03 m=+206.953824897

tags: #fleeting #full-cycle

## Technical Architect

The technical architect is someone who is specialized in a kind of software.

*   Elastic Architect
*   Salesforce Architect
*   AWS Architect
*   Oracle Architect

## Corporative Architect

The corporative architect is someone who is specialized in a kind of business.
Evaluating new technologies and new business models.

## Solution Architect

The solution architect is someone capable of designing a solution for a business.

*   Draws a lot
    *   C4
    *   UML
    *   BPMN
    *   Archimate
    *   SysML
    *   DFD
    *   ERD

## Software Architect

The software architect is responsible for the design of the software.

*   Orchestrate the development team.
*   Define the architecture of the software.
*   Good practices.

## Software Architecture vs Software Design

Software architecture has a broader scope than software design.
The main focus of the software architecture is to assure that the
business requirements, system attributes, and constraints are met.

Any design decision that does not have a relationship with any of
these three aspects is not a software architecture decision.

## Software sustentability

Software developmenting is expensive, most of the softwares
must pay for itself. Bad software architecture can lead to
impossible maintenance, and the software will be replaced
by a new one with new costs and new risks.

## Software Pillars

*   Structure
    *   Easy maintenability
*   Components
    *   Easy to reuse
*   Relationships
*   Governance

## Architecture Requirements (AR)

**How these requirements impact the architecture of the project?**

*   Performance
    *   SLA....
*   Data Persistence
    *   Compliance
    *   Cloud Provider
*   Escalability
    *   Load balancer configuration
    *   Autoscaling
*   Security
    *   PCI
    *   Cryptography
    *   Authentication
*   Legal
    *   GDPR
    *   Privacy
*   Audit
    *   Logs
    *   Metrics

## Architectural characteristics

### Operational characteristics

Architectural characteristics are characteristics that impact the operational
side of the software.

*   Availability
*   Disaster recovery
*   Performance
*   Backup
    *   Backup testing
    *   Backup strategy
*   Security and compliance
*   Robustness
*   Scalability

### Structural characteristics

Design characteristics are characteristics that impact the design of the software.

*   Configurable
    *   ENV
    *   Config files
*   Extensible
    *   Low refactoring
    *   Low coupling
    *   Anti-corruption layer
    *   Adapters
*   Easy installation
    *   Dockerization
*   Componentization
    *   Microservices
    *   Monolith
*   Internationalization
*   Easy maintance
    *   Solid
    *   Adapters
*   Portability
    *   Cloud
    *   On-premise
*   Support
    *   Logs
    *   Metrics
    *   Monitoring

### Crosscutting characteristics

Crosscutting characteristics are characteristics that impact in daily design
decisions.

*   Accessibility
*   Data retention and recovery
*   Authentication and authorization
*   Legal and regulatory compliance
*   Privacy
*   Security
*   Usability

## Architecture checklist

*   Performance
*   Scalability
*   Resilience

### Software Performance

Time and resources needed to complete a task

#### Metrics

*   Latency or response time
    *   measured in milliseconds
    *   affected by processing time, network latency, and queueing time
*   Throughput or request rate
    *   measured in requests per second

Software Scalability != Software Performance

#### Checklist for performance

*   \[] Inefficient Processing
*   \[] Limited computing resources
*   \[] Blocking I/O
*   \[] Serialized resource access

#### Ways to improve performance

*   Parallelism and concurrency
*   Algorithmic improvements
*   Hardware upgrades
*   Database optimizations
*   Caching

#### Vertical vs Horizontal Scaling

*   Vertical scaling

    *   Increase the capacity of a single server
    *   More expensive
    *   More complex
    *   More difficult to scale

*   Horizontal scaling

    *   Increase the number of servers
    *   Cheaper
    *   Easier to scale

#### Concurrency vs Parallelism

Concurrency is about dealing with lots of things at once.
Parallelism is about doing lots of things at once.

#### Caching

*   Useful caching options:
    *   Edge caching
    *   CDN
    *   Static assets
    *   Web pages
    *   Database queries
    *   API responses
    *   Heavy algorithms
    *   Objects

#### Exclusive caching vs Shared caching

*   Exclusive caching

    *   Cache is only used by a single user
    *   Cache is not shared
    *   Cache is not shared between users
    *   Cache is not shared between processes
    *   Cache is not shared between servers

*   Shared caching

    *   Cache is shared between users
    *   Cache is shared between processes
    *   Cache is shared between servers

Exclusively caching is not a good idea, because it can lead to
inconsistent data.

#### Edge computing

Edge computing is a distributed computing paradigm that brings computation and
data storage closer to the location where it is needed.

*   Cache closer to the user
*   Reduce latency and improve performance

***

### Software Scalability

Software scalability is the ability of a system to handle a growing amount of work
by adding resources to the system.

#### Scalability vs Performance

Performance focus on reducing the latency and increasing the throughput.
Scalability focus on increasing the throughput through the addition of resources.

#### Scalling strategies

Machines can be discarded or added to the system, without affecting the
system's behavior.

*   No disk usage
*   Application server vs assets server
*   Centralized Cache
*   Centralized Session
*   File Upload

#### Database Scalability

*   Increasing computational resources
*   CQRS
*   Sharding
*   Serverless database
*   Indexing
*   APM monitoring
*   Explain query

#### Reverse proxying

A reverse proxy is a type of proxy server that retrieves
resources on behalf of a client from one or more servers.
These resources are then returned to the client, appearing as
if they originated from the proxy server itself.

*   Nginx
*   HAProxy
*   Traefik

***

### Software Resilience

Software resilience is the ability of a system to recover from
unexpected events.

Not being able to recover from unexpected events can lead to
lost data, lost money, and lost customers.

*   SPF: single point of failure
    *   What happens if the system fails?

#### Protect and be protected

*   A system must have mechanisms to protect itself from unexpected
    events, and must have mechanisms to protect other systems from
    unexpected events.

*   A system can not be selfish to keep sending requests to a
    unavailable system.

*   Slow requests can lead to a cascading failure.

#### Health Check

A health check is a mechanism to check if a system is healthy.

A health check can be a simple ping, or a more complex check
that verifies if the system is able to process requests.
A good health check can be used to detect problems before
they start to affect the system.

#### Rate Limiting

Limit the number of requests that a system throughputs can handle.
Protect the system from a cascading failure or uncontrolled loops.

#### Circuit Breaker

A circuit breaker is a mechanism that protects a system from
unexpected events.

It works like a switch, when the system is healthy, the switch
is closed, and the system can process requests.
When the system is unhealthy, the switch is open, and the system
can not process requests.

After the circuit is open, from time to time, the system will
try to close the circuit, and start processing requests again.
This is called a half-open state.

#### API gateway

An API gateway is a system that acts as a proxy between the
client and the backend systems.

Validating requests, rate limiting, circuit breaker, and
health check are some of the features that an API gateway

#### Service Mesh

*   Control network traffic
*   Decouple network traffic from application logic
*   mTLS
*   Abstract network logic from developers

#### Asynchronous communication

*   Stop losing data
*   Message queue
*   Event sourcing
*   Broker to store messages

#### Retry

*   Backoff algorithm
    *   Exponential backoff vs linear backoff
    *   Jitter (small random delay)

#### Kafka

*   Broker for storing messages
*   Ack 0: No confirmation of message delivery
*   Ack 1: Leader confirmation of message delivery
*   Ack all or -1: Leader and replicas confirmation of message delivery
