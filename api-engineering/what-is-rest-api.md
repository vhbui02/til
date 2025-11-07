# What Is REST API

<!-- tl;dr starts -->

REST (or **RE**presentational State Transfer) is an _architectural style_ for **distributed hypermedia systems**. [**Roy Fielding**](https://en.wikipedia.org/wiki/Roy_Fielding) first presented it in 2000 in his [dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)

<!-- tl;dr ends -->

## Misconception clarification

REST is NOT a protocol or a standard, it's _an architectural style_, which is something more high-level. API developers can implement REST in a variety of ways.

An architectural style has its _guiding principles and constraints_. A service interface is referred as **RESTful** if those principles are satisfied.

A Web Service API (note: avoid using the term "Web API" to refer to REST APIs, as it may also refer to browser functionality interfaces) conforming to the REST architectural style (follows its guiding principles and constraints) is called **REST API** (or _RESTful_ API).

## The SIX Guiding Principles of REST

_Pros:_

- Simplicity
- Scalability
- Statelessness

### 1. Uniform Interface

REST defines a consistent and uniform interface for interaction between Clients and Servers.

#### 1.1. Identification of resources

_Definition:_ The interface must uniquely identify each resource involved in the interaction between the client and the server.

> _Q:_ What is a resource, or more specifically, a REST resource?
>
> _A:_ Any information that can be named is a resource. It can be a document, an image, a temporal service (e.g. today's weather in Ha Noi), a collection of other resources, a non-virtual object (e.g. a person), ...
>
> A _resource_ is conceptual mapping to a set of entities, not the entity that corresponds to the mapping at any particular point in time.

_Implementation:_ **URI** (Uniform Resource Identifiers).

#### 1.2. Manipulation of resources through representation

_Definition:_ The resources should have uniform representations in the server response. API consumers should use these representations to modify the resource state in the server.

> _Q:_ What is a resource's representation?
>
> _A:_ A concept entity with 3 components
>
> - Data
> - Metadata about Data
> - Hypermedia links helping the clients transition to the next desired state.

_Implementation:_ **MIME type** (or IANA media type) - the data format of a representation. It identifies a specification, this specification defines how the client should process this representation.

    ```
    application/json
    application/pdf
    # ...
    ```

**REST API's resource model**: A REST API consists of a collection of interlinked resources.

#### 1.3. Self-descriptive messages

_Definition:_ Each resource representation should carry enough information to describe how to process the message. It should also provide information on the additional actions that the client can perform on the resources.

_Layman's term:_ the client doesn't need to know **what's the resource**, it only needs to know the **media type** associated with the resource. Every media type defines a _processing model_.

_Implementation_: The process of clicking an HTML anchor tag with an `href=` attribute that points to another REST API, which when clicked will automatically send an HTTP GET request.

#### 1.4. Hypermedia as the engine of application state (HATEOAS)

_Definition:_ The client should have only the initial URI of the application. The client application should dynamically drive all other resources and interactions with the use of hyperlinks.

> _Q:_ What is Hypertext, Hypedlinks and Hypermedia?
>
> _A:_
>
> - Hypertext: Text-based content that contains links to other text. A classic example is HTML with its anchor tags.
> - Hyperlinks: The anchor tags.
> - Hypermedia: Links now not only associate with text but also with images, audio, video, interactive elements, ...

_Implementation_:

- For `application/html`, its content includes a lot of anchor tags.
- For `application/json`, its content includes links to sub-collection resources, other collection resources, ...
- etc.

### 2. Client-Server

It's a design pattern that enforces **separation of concerns**. Clients do not need to know about the Servers evolution, and neither does the Servers.

This design pattern can improve:

- Portability of UI across multiple platforms (e.g. cross-platform cross-engine browsers)
- Scalability by simplifying server components.

> **NOTE:** while the client and the server evolve, we have to make sure the interface/contract between the client does not break. The contract can be seen in things like browser compatibility for specific HTML, CSS, JS versions.

### 3. Stateless

Simple term: each request from the client to the server must contain **ALL** of the information necessary for the server to understand and complete the request.

The server can't take advantage of any previously stored context information on the server.

Later, some practical challenges come up: Authentication, User Preferences (Theme, ...) Session context, ... scientists have to create Cookies to mitigate the cons of statelessness.

### 4. Cacheable

A response should implicitly or explicitly label itself as cacheable or non-cacheable. If it's cacheable, the client application should reuse the response data later for equivalent requests. This should happen in a specified period, after that the client application must fetch new reponse data.

### 5. Layered System

This architectural style should be composed of hierarchical layers, each layer can't see beyond the immediate layer that it's interacting with. E.g. Three-tier architecture.

### 6. Code on Demand (optional)

Client application functionality should be extended by downloading and executing code in the form of scripts. This is commonly implemented using JavaScript.

## References

- [April 1, 2025, Lokesh Gupta's "What is REST?"](https://restfulapi.net/)
