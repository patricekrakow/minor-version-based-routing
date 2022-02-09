# Minor Version-Based Routing

Within an API-Centric Service Mesh, API endpoint requests MUST be routed to service instances implementing a compatible version of the API.

## Abstract

As a _producer_ (or a _provider_), everything starts with designing a beautiful API, a beautiful interface that your _consumers_ will love to use, expressed in a _OpenAPI document_ that conforms to the _OpenAPI Specification (OAS)_. After (and only after) this (API-)first step, the producer/provider can start coding the implementation of the different API endpoints - uniquely identified by the triplet (HTTP method, hostname, URL path template) - that make the API.

When these API (endpoints) are exposed to the outside world, usually via an API Gateway, there is a clear distinction between the API, being solely an interface, and the implementation. When you call an API endpoint to get a response to your request, you have absolutely no clue about the way the underlying service is implemented. That clear distinction tends to disappear when the API endpoint is exposed within the internal network of the company. Usually, the _consumers_ is calling directly the service as she/he has direct access to the internal address of the service. The fact that the network connectivity is direct, without the additional network hop through the API Gateway, is the good part. The fact that the producer/provider loses the control she/he can have on the routing of the API endpoint requests, in order to perform, for instance, canary release, is the bad part.

An **_API-Centric Service_** Mesh, based on layer 7 traffic routing capabilities of modern service meshes, aims to unify the Developer eXperience (from both producer and consumer point of view) of API Management whenever the APIs are used externally, internally, or both.

However, given that (1) the implementation of API endpoints can (should) evolve in a backwards compatible manner and given that (2) you may have different backwards compatible versions of implementations of a particular API endpoint, it is crucial to route the API endpoint requests to service instances that implement a compatible version of the API endpoint, according to the subscriptions of the _consumers_.

Let's illustrate that point with the following scenario:

1. The `Alpha API`, containing the `API endpoint X`, has been designed by the `Producer P1` and marked with the (semantic) version `1.0.0`.
2. The `Producer P1` deploys several instances of the `Service S1` which implements the `API endpoint X` as defined the version `1.0.0` of the `Alpha API`.
3. The `Consumer C1` successfully creates a subscription for the `Application A1` to the `API endpoint X` as defined the version `1.0.0` of the `Alpha API`.
4. The `Producer P1` designs a new backward compatible change of the `API endpoint X` of her/his `Alpha API` which is then marked with the (semantic) version `1.1.0`.
5. The `Producer P1` deploys several instances of the `Service S2` which implements the `API endpoint X` as defined the version `1.1.0` of the `Alpha API`.
6. The requests from the `Application A1` to the `API endpoint X` can be routed to either `Service S1` or `Service S2` instances as the change introduced was backward compatible.
7. The `Consumer C2` successfully creates a subscription for the `Application A2` to the `API endpoint X`  **as defined the version `1.1.0` of the `Alpha API`**.
8. The requests from the `Application A2` to the `API endpoint X` MUST be routed exclusively to `Service S2` instances, as the `Service S1` instances do not implement new features introduced by the version `1.1.0` of the `Alpha API` that might be needed by the `Application A2`.

Therefore it is clear that, within the service mesh, the client router MUST route API endpoint requests checking that the **minor version of the API from the subscription is equal or lower than the minor version of the API from the implementation**, while the major versions MUST match exactly of course. It is this check that we propose to call the **_Minor Version-Based Routing_**.
