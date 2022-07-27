
# ADR 010: DID Resolver

## Status

| Category | Status |
| :--- | :--- |
| **Authors** | Renata Toktar|
| **ADR Stage** | DRAFT |
| **Implementation Status** | Not Implemented |
| **Start Date** | 22 February 2022 |

## Summary

This Architecture Decision Record (ADR) defines the architecture of two DID resolvers: 

- A **full** cheqd DID Resolver, as a library written in Go, or as a standalone web service (the goal of which is to generate a spec compliant DIDDoc, based on the [cheqd DID method](adr-002-cheqd-did-method.md)), through communicating with a cheqd node at the gPRC endpoint; and
- A [Universal Resolver Driver](https://github.com/decentralized-identity/universal-resolver) (the goal of this, as a **proxy resolver**, is to **relay requests to the full DID resolver**, and to provide greater accessibility to third parties resolving cheqd DIDs). 

This ADR will also address the following architectural considerations:

- cheqd full DID Resolver has to marshal/unmarshal protobuf object into a resealable JSON DID Document.
- cheqd full DID Resolver will use Golang programming language.
- cheqd full DID resolver can be set up locally on client side, or used as a hosted web service.
- cheqd full DID resolver will use a framework like Goji or Echo, because it handles HTTP status codes etc., that are part of DID resolution, handling and catching errors.
- cheqd Universal Resolver Driver will use Node.js ( +itty-router ) proxy.

## Context

DID resolution is the process of **resolving a DID to fetch a DID document** by using the "Read" operation of the applicable DID method, in our case, the [cheqd DID method](adr-002-cheqd-did-method.md).

All conforming DID resolvers implement the functions below, which have the following abstract forms:

    resolve(did, resolutionOptions) → 
    « didResolutionMetadata, didDocument, didDocumentMetadata »

    resolveRepresentation(did, resolutionOptions) → 
    « didResolutionMetadata, didDocumentStream, didDocumentMetadata »
    
These two functions enable:

- **Resolve function**: resolution of a DID to a [map](https://www.w3.org/TR/did-core/#did-resolution), with the relevant DID Document, DID Document Metadata and DID Resolution Metadata populated;
- **resolveRepresentation function**: the resolution of a byte stream format of a DID (in say JSON, JSON-LD or CBOR) back into the [map](https://www.w3.org/TR/did-core/#did-resolution) structure. 

Since cheqd uses the Cosmos SDK and is built within the Cosmos ecosystem, data written to the ledger uses the serialisation method Google Protocol Buffers (protobuf). Therefore, in order to conform with the correct syntax and structure for the map produced after resolution, cheqd's DID Resolver will implement a resolveRepresentation function that is able to take output in protobuf and convert it into a corresponding, conformant map in JSON, according to the [DID core specification](https://www.w3.org/TR/did-core/).

## Overall Architecture of DID Resolver(s)

Both implementations of the cheqd DID resolver are decoupled from the cheqd network, which means updating the resolver does not require updating the application on the node side. This avoids having to go through an on-ledger governance vote, and voting period to make a change. In addition, the separation of the system into microservices provides more flexibility to third parties in how they choose to resolve cheqd DIDs.

The overall DID resolver architecture is detailed by the following flow:

![cheqd did resolver](../../assets/adr-010-did-resolver/universal-resolver-sequence-diagram.png)
[Figure 1: cheqd resolution sequence diagram.](../../assets/adr-010-did-resolver/universal-resolver-sequence-diagram.puml)

cheqd DID Document resolution is built to be lightweight and simple. Instead of needing to handle requests and threads in parallel, the resolover is built to handle all threads concurrently. This design principle will reduce the risk of large quantities of threads and requests blocking the efficiency of the service.

![cheqd did resolver class diagram](../../assets/adr-010-did-resolver/resolver-class-diagram.png)
[Figure 2: cheqd protobuf -> JSON marshalling.](../../assets/adr-010-did-resolver/resolver-class-diagram.puml)

## Resolution rules

Cheqd Resolver compiles with the rules defined in [Decentralized Identifier Resolution (DID Resolution) v0.2](https://w3c-ccg.github.io/did-resolution). This section clarifies and expands some descriptions specific to Cheqd.

### Supported types

[RFC 9110 HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html#name-accept) says:

> The "Accept" header field can be used by user agents to specify their preferences regarding response media types.

It means that a response "Content-Type" header field should contain one of types from a request "Accept" header field. 

In the same time, [Resolution specification](https://w3c-ccg.github.io/did-resolution/#bindings-https) requires: 

> The HTTP response body MUST contain the didDocumentStream, in the representation corresponding to the Accept HTTP header.
 
There for `ContentType` from HTTP response body should be the same with HTTP response header `Content-Type` and should be one of types from `Accept` request HTTP header.
[8.1 HTTP(S) Binding](https://w3c-ccg.github.io/did-resolution/#bindings-https) has been used for defining a list of available types for DID resolution. 

* Accept request HTTP header contains `application/did+ld+json`
  * Response HTTP header: `Content-Type: application/did+ld+json`
  * Response HTTP body
    * didDocument / contentStream contains `@context` section;
    * didResolutionMetadata / dereferencingMetadata `ContentType` field is `application/did+ld+json`;
* Accept request HTTP header contains `application/ld+json;profile="https://w3id.org/did-resolution"`
  * Response HTTP header: `Content-Type: application/ld+json;profile="https://w3id.org/did-resolution`
  * Response HTTP body
    * didDocument / contentStream contains `@context` section;
    * didResolutionMetadata / dereferencingMetadata `ContentType` field is `application/ld+json;profile="https://w3id.org/did-resolution`;
* Accept request HTTP header contains `application/did+json`
  * Response HTTP header: `Content-Type: application/did+json`
  * Response HTTP body
    * didDocument / contentStream DOES NOT contain `@context` section;
    * didResolutionMetadata / dereferencingMetadata `ContentType` field is `application/did+json`;
* Accept request HTTP header contains `*/*` 
  * Response HTTP header: `Content-Type: application/did+ld+json`
  * Response HTTP body
    * didDocument / contentStream contains `@context` section;
    * didResolutionMetadata / dereferencingMetadata `ContentType` field is `application/did+ld+json`;
* Accept request HTTP header does not contain any of the types above
  * Response HTTP header: `Content-Type: application/json`
  * Response HTTP body
    * didDocument / contentStream is `[]`;
    * didResolutionMetadata / dereferencingMetadata contains a property error with value representationNotSupported
    * didResolutionMetadata / dereferencingMetadata `ContentType` field is `application/json`;

### Errors

DID resolution result always has a following format: `( didResolutionMetadata, didDocument, didDocumentMetadata )`
If resolution was unsuccessful, DID resolver returns the following result:

* didResolutionMetadata contains `"error" : "<Error message>"`
* didDocument: null
* didDocumentMetadata: `[]`


DID dereferencing result always has a following format: `( dereferencingMetadata, contentStream, contentMetadata )`

* dereferencingMetadata contains `"error" : "<Error message>"`
* contentStream: null
* contentMetadata: `[]`

#### Error list

* **invalidDidUrl**
  * response status code 400
* **notFound** -> response status code 404
* **representationNotSupported** -> response status code 406
* **didDocumentMetadata** property deactivated with value true -> response status code 410
* **internalError** -> response status code 500 (but still return ( dereferencingMetadata, contentStream, contentMetadata ) or ( didResolutionMetadata, didDocument, didDocumentMetadata ))

### Resource resolution

    /1.0/identifiers/did:cheqd:testnet:DAzMQo4MDMxCjgwM/resources/

    Return collection resources metadata (without data)

    /1.0/identifiers/did:cheqd:testnet:DAzMQo4MDMxCjgwM/resources/44547089-170b-4f5a-bcbc-06e46e0089e4

    Return resources data
    ContentType = MediaType

    /1.0/identifiers/did:cheqd:testnet:DAzMQo4MDMxCjgwM/resources/44547089-170b-4f5a-bcbc-06e46e0089e4/matadata

    Return collection resources metadata  (without data)



## cheqd full DID resolver

There are two ways to use the full cheqd DID Resolver to return compliant DID Documents: 
- As a library (go module), and
- As a standalone web service (either hosted by cheqd or integrated directly on client side).

### 1. Go module (library)

In the first case, the Go module can be imported simply into a client's own libraries by using the following:

```golang
import (
     "github.com/cheqd/cheqd-did-resolver/services"
)
```
The flow for DID resolution is illustrated in the third "Client <-> Ledger" section from [figure 1](#cheqd-did-resolver--universal-resolver-driver).

The flow can be summarised as follows:
- A client sends a request directly to cheqd Node through the Cosmos SDK gRPC API.
- A DID Doc is returned in protobuf which the client application can format to resolvable DID Document or DID fragment in JSON format.
- This makes the DID resolution compliant with [W3C DID Core](https://www.w3.org/TR/did-core/).

#### Pros

- There are no third party services required as the client is able to import the library into their own internal software. The client can also set up their own node if they are concerned about any security risks with trusting the cheqd node for resolution.

#### Cons

- No Universal Resolver with other DID methods.
- The client application needs to use Golang which is not particularly common.

### 2. Standalone Web Service (hosted by cheqd)

cheqd provides its own web service with a full DID resolver. If a client doesn't need to use the full breadth of the did methods within Universal Resolver, then requests can be sent directly to the cheqd web service. 

In this case:
- A client launches the application with the command.

```bash
docker compose up
```

or launch light version on theirs Cloudflare 

or send requests directly to the cheqd web service

- A client sends a request to cheqd DID Resolver web service.
- cheqd DID Resolver retrieves DID Doc in protobuf format from the ledger through Cosmos SDK gRPC API.
- cheqd DID Resolver generates and sends a response for the client request, based on received DID Doc, in JSON format.

#### Pros

- Through routing requests directly to cheqd's Resolver, there is a shorter trust chain. This means that the client only needs to rely on the cheqd web service and the cheqd node, which has fewer security risks than routing through a Universal Resolver.

#### Cons

- Using a standalone web service does not have the same breadth as the Universal Resolver in terms of support for other DID methods.
- The security risks are not completely eliminated, and the client still needs to trust two services that they do not directly control.

### 3. Standalone Web Service (fully set up on a client side)

If a client chooses, it may be in their interest to set up the full cheqd resolver web service on their own client side. 

```bash
docker compose up
```

This will offer full oversight of the DID resolver, and as such, a higher level of security.

#### Pros

- The trust-chain is much shorter than relying on the web service hosted by cheqd, or on a Universal Resolver.

#### Cons

- The client will need to set up additional services which is more complex.
- If the client is security conscious, it is likely it would also want to set up its own node in addition to the Resolver to ensure oversight over requests to the gRPC endpoint.
 
**Note.** While it is possible to set up both the Universal Resolver and the full cheqd resolver on the client side **without** setting up a node, we have not expressely considered this option in this ADR, since we believe it is unlikely to occur in practice. 


## Universal Resolver Driver

A Universal Resolver Driver can be set up in a full or light mode. It is the same application with Cheqd DID resolver.


### 1. Universal resolver on DIF side

The Decentralised Identity Foundation (DIF) has a publically accessible Universal Resolver, which can be found at https://dev.uniresolver.io

The flow of resolving a DID via the Universal Resolver on DIFs side is shown in the "Universal Resolver" section from [figure 1](#cheqd-did-resolver--universal-resolver-driver) shows this flow.

To summarise the flow:
- A client just sends a request to https://dev.uniresolver.io. 
- The Universal Resolver on DIF servers uses the cheqd Universal Resolver Driver to redirect client request to the full cheqd DID Resolver.
- cheqd full DID Resolver gets DID Doc in protobuf format from the ledger through the Cosmos SDK gRPC API.
- cheqd full DID Resolver generates a response for the client's request based on received DID Doc.
- cheqd full DID Resolver sends a response to the client through the cheqd Universal Resolver Driver and Universal Resolver itself.

#### Pros

- The resolution endpoint can be utilised without additional library dependencies and without setting up additional services.

#### Cons

- https://dev.uniresolver.io can be used only for development, but can't be use in production goals.
- The Universal Resolver route hosted by DIF has the longest trust chain, since the client must trust both DIF as a hosting provider, cheqd as a hosting provider as well as cheqd's node.

### 2. Universal resolver on a client side

There is also an option of using the Universal Resolver on the client side, which is a similar flow to [Universal resolver on DIF side](#1-universal-resolver-on-dif-side). 

Here, however, the client sets up Universal Resolver with drivers in their own environment.

#### Pros

- This can be used in production.
- Trust in DIF servers is not needed.

#### Cons

- Setting up Universal Resolver and Drivers are needed which takes effort on the client side.
- The client still needs to trust both DIF as well as cheqd from a security standpoint.

### 3. Universal Resolver, cheqd full DID resolver and cheqd Node on a client side

This final option includes both the Universal Resolver in combination with the full cheqd DID resolver. This option may be beneficial for a client which wants the full breadth of resolution options offered by the Universal Resolver, and is highly security conscious, and as such, routes resolution requests through their own infrastrcuture rather than relying on cheqd's hosted web service or the cheqd node. 

The client will need to set up:
- Universal Resolver,
- Universal Resolver Drivers,
- cheqd resolver web service,
- cheqd node.

#### Pros

- This is the combination for maxmimum trust and security for DID resolution.

#### Cons

- This may be seen as overkill, given the number of services the client will need to set up on their own side. 

## Decision

This ADR will add a new application/library for did-resolution based on the DID resultion options listed in the sections above. 

## References

- [W3C Decentralized Identifiers (DIDs)](https://www.w3.org/TR/did-core/) specification
  - [DID Core Specification Test Suite](https://w3c.github.io/did-test-suite/)

## Unresolved questions

- Should the web service find another node for the request if it is not possible to connect to the cheqd node? Should the web service need to have a pool of trusted nodes for routing requests?
- How should synchronous response to client requests be handled, if needed in the future?
