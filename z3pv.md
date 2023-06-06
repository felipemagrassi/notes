# System communications - 2023-06-02 23:03:12.185583668 -0300 -03 m=+0.013861547

<!--toc:start-->
- [System communications - 2023-06-02 23:03:12.185583668 -0300 -03 m=+0.013861547](#system-communications-2023-06-02-230312185583668-0300-03-m0013861547)
  - [Rest](#rest)
    - [Maturity Model](#maturity-model)
  - [GraphQL](#graphql)
  - [gRPC](#grpc)
  - [System Discovery and Consul](#system-discovery-and-consul)
<!--toc:end-->

tags: #fleeting #full-cycle #communication

***

## Rest

REST is a HTTP-based protocol for data exchange between systems.

*   Stateless
*   Cacheable
*   Simple
*   Client-Server

### Maturity Model

*   Level 0: The Swamp of POX
  * Transactional
*   Level 1: Resources Usage
  * HTTP Verbs
  * URI
  * Actions
*   Level 2: HTTP Verbs and representations
  * Read with GET
  * Create with POST
  * Update with PUT
  * Delete with DELETE
  * Partial Update with PATCH
*   Level 3: Hypermedia Controls
  * HATEOAS
  * Use links to navigate through the API


### Good API Characteristics 

- Use unique URI for each resource
- Use all HTTP verbs to represent the actions, including caching
- Use HTTP status codes to represent the result of the action
- Related link should be provided in the response

### Good API Patterns

#### HAL 

Media type: `application/hal+json`

```json
{
    "_links": {
        "self": { "href": "/orders" },
        "curies": [{ "name": "ea", "href": "http://example.com/docs/rels/{rel}", "templated": true }],
        "next": { "href": "/orders?page=2" },
        "ea:find": {
            "href": "/orders{?id}",
            "templated": true
        },
        "ea:admin": [{
            "href": "/admins/2",
            "title": "Fred"
        }, {
            "href": "/admins/5",
            "title": "Kate"
        }]
    },
    "currentlyProcessing": 14,
    "shippedToday": 20,
    "_embedded": {
        "ea:order": [{
            "_links": {
                "self": { "href": "/orders/123" },
                "ea:basket": { "href": "/baskets/98712" },
                "ea:customer": { "href": "/customers/7809" }
            },
            "total": 30.00,
            "currency": "USD",
            "status": "shipped"
        }, {
            "_links": {
                "self": { "href": "/orders/124" },
                "ea:basket": { "href": "/baskets/97213" },
                "ea:customer": { "href": "/customers/12369" }
            },
            "total": 20.00,
            "currency": "USD",
            "status": "processing"
        }]
    }
}
```

*  `_links`: Links to navigate through the API
  * `self`: Link to the current resource
*  `_embedded`: Embedded resources


#### Siren

#### Collection+JSON

### Method Negotiation

Negotiation that tells the client, which methods are allowed for a given resource.

```http
OPTIONS /orders HTTP/1.
```

returns:

```http
HTTP/1.1 200 OK
Allow: GET, POST, HEAD, OPTIONS
```

### Content Negotiation

Negotiation that tells the client, which content types are allowed for a given resource.

```http
GET /orders HTTP/1.1
Accept: application/json
```

returns:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

or 
  
```http
HTTP/1.1 406 Not Acceptable
Content-Type: application/json
```


## GraphQL

Usage in BFFs - Backend for Frontends

Allows the client to query only the data it needs.

### Types vs Inputs

Types are used to represent the data that is returned from the server. 
Inputs are used to represent the data that is sent to the server.

### Mutations vs Queries

Mutations are used to change the data on the server.
Queries are used to retrieve data from the server.

### Resolvers

When a query is executed, the GraphQL engine will call the resolvers to retrieve the data.





## gRPC

## System Discovery and Consul
