# ADR 002: cheqd DID method

## Status

| Category | Status |
| :--- | :--- |
| **Authors** | Ankur Banerjee, Alexandr Kolesov, Alex Tweeddale, Brent Zundel, Renata Toktar, Richard Esplin   |
| **ADR Stage** | ACCEPTED |
| **Implementation Status** | Implemented |
| **Start Date** | 2021-09-23 |
| **Last Updated** | 2022-06-24 |

## Summary

This ADR defines the cheqd DID method and describes the identity entities, queries, and transaction types for the cheqd network: a purpose-built self-sovereign identity (SSI) network based on the [Cosmos blockchain framework](https://github.com/cosmos/cosmos-sdk).

[Decentralized identifiers](https://www.w3.org/TR/did-core) (DIDs) are a type of identifier that enables verifiable, decentralized digital identity. A DID refers to any subject (for example, a person, organization, thing, data model, abstract entity, and so on) as determined by the controller of the DID.

## Context

### Rationale for baselining against Hyperledger Indy

Hyperledger Indy is a verifiable data registry (VDR) built for DIDs with a strong focus on privacy-preserving techniques. It is one of the most widely-adopted SSI blockchain ledgers. Most notably, Indy is used by the [Sovrin Network](https://sovrin.org/overview/).

The Sovrin Foundation initiated a project called [`libsovtoken`](https://github.com/sovrin-foundation/libsovtoken) in 2018 to create a native token for Hyperledger Indy. `libsovtoken` was intended to be a payment handler library that could work with `libindy` and be merged upstream. This native token would allow interactions on Hyperledger Indy networks (such as Sovrin) to be paid for using tokens.

Due to challenges the project ran into, the `libsovtoken` codebase saw its [last official release in August 2019](https://github.com/sovrin-foundation/libsovtoken/releases/tag/v1.0.1).

### Rationale for using the Cosmos blockchain framework for cheqd

The cheqd network aims to support similar use cases for SSI as seen on Hyperledger Indy networks, with a similar focus on privacy-resspecting techniques.

Since the core of Hyperledger Indy's architecture was designed before the [W3C DID specification](https://www.w3.org/TR/did-core/) started to be defined, the [Indy DID Method](https://hyperledger.github.io/indy-did-method/) (`did:indy`) has aspects that are not fully-compliant with latest specifications.

However, the [rationale for why the cheqd team chose the Cosmos blockchain framework instead of Hyperledger Indy](https://blog.cheqd.io/why-cheqd-has-joined-the-cosmos-4db8845722c5) were primarily down to the following reasons:

1. **Hyperledger Indy is a permissioned ledger**: Indy networks are permissioned networks where the ability to have write capability is restricted to a limited number of nodes. Governance of such a permissioned network is therefore also not decentralised.
2. **Limitations of Hyperledger Indy's consensus mechanism**: Linked to the permissioned nature of Indy are the drawbacks of its [Plenum Byzantine Fault Tolerant (BFT) consensus](https://github.com/hyperledger/indy-plenum) mechanism, which effectively limits the number of nodes with write capability to approximately 25 nodes. This limit is due to limited transactions per second (TPS) for an Indy network with a large number of nodes, rather than a hard cap implemented in the consensus protocol.
3. **Wider ecosystem for token functionality outside of Hyperledger Indy**: Due to its origins as an identity-specific ledger, Indy does not have a fully-featured token implementation with sophisticated capabilities. Moreover, this also impacts end-user options for ecosystem services such as token wallets, cryptocurrency exchanges, custodianship services etc that would be necessary to make a viable, enterprise-ready SSI ledger with token functionality.

By selecting the Cosmos blockchain framework, the maintainers of the cheqd project aim to address the limitations of Hyperledger Indy outlined above. However, with an eye towards interoperability, the cheqd project aims to use [Hyperledger Aries](https://wiki.hyperledger.org/display/ARIES/Hyperledger+Aries) for ledger-related peer-to-peer interactions.

### Identity-domain transaction types in Hyperledger Indy

Our aim is to support the functionality enabled by [identity-domain transactions in by Hyperledger Indy](https://github.com/hyperledger/indy-node/blob/master/docs/source/transactions.md) into `cheqd-node`. This will partly enable the goal of allowing use cases of existing SSI networks on Hyperledger Indy to be supported by the cheqd network.

The following identity-domain transactions from Indy were considered:

1. `NYM`: Equivalent to "DIDs" on other networks
2. `ATTRIB`: Payload for DID Document generation
3. `SCHEMA`: Schema used by a credential
4. `CRED_DEF`: Credential definition by an issuer for a particular schema
5. `REVOC_REG_DEF`: Credential revocation registry definition
6. `REVOC_REG_ENTRY`: Credential revocation registry entry

Revocation registries for credentials are not covered under the scope of this ADR. This topic is discussed separately in [ADR 007: **Revocation registry**](adr-007-revocation-registry.md) as there is ongoing research by the cheqd project on how to improve the privacy and scalability of credential revocations.

Schemas and Credential Definitions are also not covered under the scope of this ADR as there are ongoing discussions on how to implement this in a W3C standards-compatible format.

## cheqd DID method (`did:cheqd`)

### DID Method Name

The `method-name` for the **cheqd DID Method** will be identified by the string `cheqd`.

A DID that uses the cheqd DID method MUST begin with the prefix `did:cheqd`. This prefix string MUST be in lowercase. The remainder of the DID, after the prefix, is as follows:

### Method Specific Identifier

The cheqd DID method's method-specific identifier (`method-specific-id`) is made up of the **`namespace`** component. The **`namespace`** is defined as a string that identifies the cheqd network (e.g., "mainnet", "testnet") where the DID reference is stored. Different cheqd networks may be differentiated based on whether they are production vs non-production, governance frameworks in use, participants involved in running nodes, etc.

The `namespace` associated with a certain network/ledger is stored in the genesis file on the node and cannot be changed by validators vote. A namespace is optional and can be omitted.

#### UUID-style Unique Identifiers

A `did:cheqd` DID **must** be unique by having the `unique-id` component defined as a [Universally Unique Identifier (UUID)](https://en.wikipedia.org/wiki/Universally_unique_identifier) generated by the creator(s) of the DID. UUIDs are the preferred unique identifier format for cheqd network.

Usage of UUID-style identifiers significantly simplifies the generation and implementation of unique identifiers, [with extremely-low probability of a collission where the same UUID](http://www.h2database.com/html/advanced.html#uuid) is generated independently.

Any client application can generate these UUIDs using their own preferred implementation in any programming language, as opposed to the method-specific logic required for Indy-style DID identifiers.

#### Indy-style Unique Identifiers

Alternatively, [the `unique-id` can also be generated similar to the `did:indy method`](https://hyperledger.github.io/indy-did-method/#indy-did-method-identifiers) from the initial public key of the DID (e.g., base58 encoding of the first 16 bytes of the SHA256 of the first Verification Method `Ed25519` public key). This `unique-id` format is referred to as the "Indy-style" unique identifier in our documentation.

In addition, Unique Identifiers can be up to 32 base58 characters long.

Support for Indy-style unique identifiers makes compatibility with Indy-based client SDKs, such as those based on [Hyperledger Aries](https://www.hyperledger.org/use/aries).

#### Namespace

If no `namespace` is specified, it assumed to be default `namespace` for the network/ledger the request is targetted at. This will generally be `mainnet` for the primary production cheqd network.

### Syntax for `did:cheqd` method

The cheqd DID method ABNF to conform with [DID syntax guidelines](https://www.w3.org/TR/did-core/#did-syntax) is as follows:

```abnf
cheqd-did       = "did:cheqd:" [namespace]
namespace       = 1*namespace-char ":" unique-id
namespace-char  = ALPHA / DIGIT
unique-id       = 16*id-char / 32*id-char / UUID
id-char         = ALPHA / DIGIT
```

#### ABNF syntax for UUID-style identifiers

Where the [formal definition of UUIDs is represented using this ABNF](https://datatracker.ietf.org/doc/html/rfc4122#section-3):

```abnf
UUID                   = time-low "-" time-mid "-"
                         time-high-and-version "-"
                         clock-seq-and-reserved
                         clock-seq-low "-" node
time-low               = 4hexOctet
time-mid               = 2hexOctet
time-high-and-version  = 2hexOctet
clock-seq-and-reserved = hexOctet
clock-seq-low          = hexOctet
node                   = 6hexOctet
hexOctet               = hexDigit hexDigit
hexDigit =
      "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9" /
      "a" / "b" / "c" / "d" / "e" / "f" /
      "A" / "B" / "C" / "D" / "E" / "F"
```

### Examples of `did:cheqd` identifiers

#### Indy-style

A DID written to the cheqd "mainnet" ledger `namespace` with a 32-character Indy-style identifier:

```text
did:cheqd:mainnet:zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY
```

A DID written to the cheqd "testnet" ledger `namespace` with a 16-character Indy-style identifier:

```text
did:cheqd:testnet:7Tqg6BwSSWapxgUD
```

An Indy-style DID where no namespace is defined, where the `namespace` would default to the one defined on ledger where it's published (typically, `mainnet`):

```text
did:cheqd:6cgbu8ZPoWTnR5Rv
```

#### UUID-style

A UUID-style DID on cheqd "mainnet" `namespace`:

```text
did:cheqd:mainnet:de9786cd-ec53-458c-857c-9342cf264f80
```

A UUID-style DID where no namespace is defined, where the `namespace` would default to the one defined on ledger where it's published (typically, `mainnet`):

```text
did:cheqd:de9786cd-ec53-458c-857c-9342cf264f80
```

## DID Documents ("DIDDocs") on cheqd

A DID Document ("DIDDoc") associated with a cheqd DID is a set of data describing a DID subject. The [representation of a DIDDoc when requested for production](https://www.w3.org/TR/did-core/#representations) from a DID on cheqd networks MUST meet the DID Core specifications.

### Elements of a DIDDoc

The following elements are needed for a W3C specification compliant DIDDoc representation:

1. **`id`**: Target DID with cheqd DID Method prefix `did:cheqd:<namespace>:` and a `unique-id` identifier.
2. **`controller`** (optional): A list of fully qualified DID strings or one string. Contains one or more DIDs who can update this DIDdoc. All DIDs must exist.
3. **`verificationMethod`** (optional): A list of Verification Methods
4. **`authentication`** (optional): A list of strings with key aliases or IDs
5. **`assertionMethod`** (optional): A list of strings with key aliases or IDs
6. **`capabilityInvocation`** (optional): A list of strings with key aliases or IDs
7. **`capabilityDelegation`** (optional): A list of strings with key aliases or IDs
8. **`keyAgreement`** (optional): A list of strings with key aliases or IDs
9. **`service`** (optional): A set of Service Endpoint maps
10. **`alsoKnownAs`** (optional): A list of strings. A DID subject can have multiple identifiers for different purposes, or at different times. The assertion that two or more DIDs refer to the same DID subject can be made using the `alsoKnownAs` property.
11. **`@context`** (optional): A list of strings with links or JSONs for
describing specifications that this DID Document is following to.

#### Example of DIDDoc representation

```jsonc
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2",
  "verificationMethod": [
    {
      "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2#authKey1",
      "type": "Ed25519VerificationKey2020", // external (property value)
      "controller": "did:cheqd:mainnet:N22N22N22KY2Dyvmuu2",
      "publicKeyMultibase": "zAKJP3f7BD6W4iWEQ9jwndVTCBq8ua2Utt8EEjJ6Vxsf"
    },
    {
      "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2#capabilityInvocationKey",
      "type": "Ed25519VerificationKey2020", // external (property value)
      "controller": "did:cheqd:mainnet:N22N22N22KY2Dyvmuu2",
      "publicKeyMultibase": "z4BWwfeqdp1obQptLLMvPNgBw48p7og1ie6Hf9p5nTpNN"
    }
  ],
  "authentication": ["did:cheqd:mainnet:N22N22KY2Dyvmuu2#authKey1"],
  "capabilityInvocation": ["did:cheqd:mainnet:N22N22KY2Dyvmuu2#capabilityInvocationKey"],
}
```

#### State format for DIDDocs on ledger

`"diddoc:<id>" -> {DIDDoc, DidDocumentMetadata, txHash, txTimestamp }`

`didDocumentMetadata` is created by the node after transaction ordering and before adding it to a state.

### DID Document metadata

Each DID Document MUST have a metadata section when a representation is produced. It can have the following properties:

1. **`created`** (string): Formatted as an XML Datetime normalized to UTC 00:00:00 and without sub-second decimal precision, e.g., `2020-12-20T19:17:47Z`.
2. **`updated`** (string): The value of the property MUST follow the same
formatting rules as the created property. The `updated` field is `null` if an Update operation has never been performed on the DID document. If an updated property exists, it can be the same value as the created property when the difference between the two timestamps is less than one second.
3. **`deactivated`** (string): If DID has been deactivated, DID document metadata MUST include this property with the boolean value `true`. By default this is set to `false`.
4. **`versionId`** (string): Contains transaction hash of the current DIDDoc version.
5. **`resources`** (list of resources metadata referred to as [Resource previews](adr-008-ledger-resources.md)| *optional*). Cannot be changed by CreateDID or UpdateDID transactions. cheqd ledger stores only the resource identifiers in the DID Doc metadata. The remainder of the resources' metadata is added when a DID is resolved.

#### Example of DIDDoc metadata

```jsonc
{
  "created": "2020-12-20T19:17:47Z",
  "updated": "2020-12-20T19:19:47Z",
  "deactivated": false,
  "versionId": "1B3B00849B4D50E8FCCF50193E35FD6CA5FD4686ED6AD8F847AC8C5E466CFD3E",
  "linkedResourceMetadata": [
       {
        "resourceURI":          "did:cheqd:mainnet:N22KY2Dyvmuu2PyyqSFKue/resources/9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
        "resourceCollectionId": "N22KY2Dyvmuu2PyyqSFKue",
        "resourceId":            "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
        "resourceName":         "PassportSchema",
        "resourceType":         "CL-Schema",
        "mediaType":            "application/json",
        "created":              "2022-04-20T20:19:19Z",
        "checksum":             "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
        "previousVersionId":     null,
        "nextVersionId":         null
      }
  ]
}
```

### Verification method

Verification methods are used to define how to authenticate / authorise interactions with a DID subject or delegates. Verification method is an OPTIONAL property.

1. **`id`** (string): A string with format `did:cheqd:<namespace>#<key-alias>`
2. **`controller`**: A string with fully qualified DID. DID must exist.
3. **`type`** (string)
4. **`publicKeyJwk`** (`map[string,string]`, optional): A map representing a JSON Web Key that conforms to [RFC7517](https://tools.ietf.org/html/rfc7517). See definition of `publicKeyJwk` for additional constraints.
5. **`publicKeyMultibase`** (optional): A base58-encoded string that conforms to a [MULTIBASE](https://datatracker.ietf.org/doc/html/draft-multiformats-multibase-03)
encoded public key.

**Note**: A single verification method entry cannot contain both `publicKeyJwk` and `publicKeyMultibase`, but must contain at least one of them.

#### Example of Verification method in a DIDDoc

```jsonc
{
  "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2#key-0",
  "type": "JsonWebKey2020",
  "controller": "did:cheqd:mainnet:N22N22KY2Dyvmuu2",
  "publicKeyJwk": {
    "kty": "OKP",
    // external (property name)
    "crv": "Ed25519",
    // external (property name)
    "x": "VCpo2LMLhn6iWku8MKvSLg2ZAoC-nlOyPVQaO3FxVeQ"
    // external (property name)
  }
}
```

### Services

Services can be defined in a DIDDoc to express means of communicating with the DID subject or associated entities.

1. **`id`** (string): The value of the `id` property for a Service MUST be a URI conforming to [RFC3986](https://www.rfc-editor.org/rfc/rfc3986). A conforming producer MUST NOT produce multiple service entries with the same ID. A conforming consumer MUST produce an error if it detects multiple service entries with the same ID. It has a follow formats: `<DIDDoc-id>#<service-alias>` or `#<service-alias>`.
2. **`type`** (string): The service type and its associated properties SHOULD be registered in the [DID Specification Registries](https://www.w3.org/TR/did-spec-registries/)
3. **`serviceEndpoint`** (strings): A string that conforms to the rules of [RFC3986](https://www.rfc-editor.org/rfc/rfc3986) for URIs, a map, or a set composed of a one or more strings that conform to the rules of
[RFC3986](https://www.rfc-editor.org/rfc/rfc3986) for URIs and/or maps.

#### Example of Service in a DIDDoc

```jsonc
{
  "id":"did:cheqd:mainnet:N22N22KY2Dyvmuu2#linked-domain",
  "type": "LinkedDomains",
  "serviceEndpoint": "https://bar.example.com"
}
```

## DID transactions

### Create DID

This operation creates a new DID using the `did:cheqd` method along with associated DID Document representation.

- **`signatures`**: `CreateDidRequest` should be signed by all `controller` private keys. This field contains a `dict` structure with the key URI from `DIDDoc.authentication`, as well as signature values.
- **`id`**: Fully qualified DID of type `did:cheqd:<namespace>`.
- **`controller, verificationMethod, authentication, assertionMethod, capabilityInvocation, capabilityDelegation, keyAgreement, service, alsoKnownAs, context`**: Optional parameters in accordance with DID Core specification properties.

#### Client request format for create DID

```jsonc
WriteRequest (CreateDidRequest(id, controller, verificationMethod, authentication, assertionMethod, capabilityInvocation, capabilityDelegation, keyAgreement, service, alsoKnownAs, context), signatures)
```

#### Example of a create DID client request

```jsonc
WriteRequest{
  "data": 
    "CreateDidRequest" {   
      "context": [
          "https://www.w3.org/ns/did/v1",
          "https://w3id.org/security/suites/ed25519-2020/v1"
      ],
      "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2",
      "controller": ["did:cheqd:mainnet:N22N22KY2Dyvmuu2"],
      "verificationMethod": [
        {
          "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2#authKey1",
          "type": "Ed25519VerificationKey2020", // external (property value)
          "controller": "did:cheqd:mainnet:N22N22KY2Dyvmuu2",
          "publicKeyMultibase": "z4BWwfeqdp1obQptLLMvPNgBw48p7og1ie6Hf9p5nTpNN"
        }
      ],
      "authentication": ["did:cheqd:mainnet:N22N22KY2Dyvmuu2#authKey1"],
  },
  "signatures": {
      "Verification Method URI": "<signature>"
      // Multiple verification methods and corresponding signatures can be added here
  }
}
```

### Update DID

This operation updates the DID Document associated with an existing DID of type `did:cheqd:<namespace>`.

- **`signatures`**: `UpdateDidRequest` should be signed by all `controller` private keys. This field contains a `dict` structure with the key URI from `DIDDoc.authentication`, as well as signature values.
- **`id`**: Fully qualified DID of type `did:cheqd:<namespace>`.
- **`versionId`**: Transaction hash of the previous DIDDoc version. This is necessary to provide replay protection. The previous DIDDoc `versionId` can fetched using a get DID query.
- **`controller, verificationMethod, authentication, assertionMethod, capabilityInvocation, capabilityDelegation, keyAgreement, service, alsoKnownAs, context`**: Optional parameters in accordance with DID Core specification properties.

#### Client request format for update DID

```jsonc
WriteRequest(UpdateDidRequest(id, controller, verificationMethod, authentication, assertionMethod, capabilityInvocation, capabilityDelegation, keyAgreement, service, alsoKnownAs, context, versionId), signatures)
```

#### Example of an update DID client request

```jsonc
WriteRequest{
  "data": 
    "UpdateDidRequest" {   
      "context": [
          "https://www.w3.org/ns/did/v1",
          "https://w3id.org/security/suites/ed25519-2020/v1"
      ],
      "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2",
      "controller": ["did:cheqd:mainnet:N22N22KY2Dyvmuu2"],
      "verificationMethod": [
        {
          "id": "did:cheqd:mainnet:N22N22KY2Dyvmuu2#capabilityInvocationKey",
          "type": "Ed25519VerificationKey2020", // external (property value)
          "controller": "did:cheqd:mainnet:N22N22N22KY2Dyvmuu2",
          "publicKeyMultibase": "z4BWwfeqdp1obQptLLMvPNgBw48p7og1ie6Hf9p5nTpNN"
        }
      ],
      "authentication": ["did:cheqd:mainnet:N22N22KY2Dyvmuu2#authKey1"],
      "versionId": "1B3B00849B4D50E8FCCF50193E35FD6CA5FD4686ED6AD8F847AC8C5E466CFD3E"
  },
  "signatures": {
      "Verification Method URI": "<signature>"
      // Multiple verification methods and corresponding signatures can be added here
  }
}
```

### Get/Resolve DID

DIDDocs associated with a DID of type `did:cheqd:<namespace>` can be resolved using the `GetDid` query to fetch a response from the ledger. The response contains:

- **`did`**: DIDDoc associated with the specified DID in a W3C specification compliant [DIDDoc structure](#did-documents-diddocs).
- **`metadata`**: Contains the MUST have [DIDDoc metadata](#diddoc-metadata) associated with a DIDDOc.

#### Client request format for get/resolve DID

DID resolution requests can be sent to the Tendermint RPC interface for a node by passing the fully-qualified DID.

```jsonc
QueryGetDidResponse(did)
```

#### Example of an get/resolve DID client request

The response is returned as a [Protobuf](https://developers.google.com/protocol-buffers/docs/overview), which can be converted to JSON client-side.

```jsonc
{
  "did":{
    "id":"did:cheqd:mainnet:2PRyVHmkXQnQzJQK",
    "controller":[
        "did:cheqd:mainnet:2PRyVHmkXQnQzJQK"
    ],
    "verification_method":[
        {
          "id":"did:cheqd:mainnet:2PRyVHmkXQnQzJQK#verkey",
          "type":"Ed25519VerificationKey2020",
          "controller":"did:cheqd:mainnet:2PRyVHmkXQnQzJQK",
          "public_key_multibase":"zkqa2HyagzfMAq42H5f9u3UMwnSBPQx2QfrSyXbUPxMn"
        }
    ],
    "authentication":[
        "did:cheqd:mainnet:2PRyVHmkXQnQzJQK#verkey"
    ]
  },
  "metadata":{
    "created":"2022-04-20T20:19:19Z",
    "updated":"2022-04-20T20:19:19Z",
    "deactivated":false,
    "version_id":"1B3B00849B4D50E8FCCF50193E35FD6CA5FD4686ED6AD8F847AC8C5E466CFD3E"
  }
}
```

## Considerations

### Security considerations

1. For creating a new DID or update the DIDDoc associated with an existing DID, the requested should be signed by all `controller` signatures.
   1. To update a DIDDoc fragment without a `controller` (any field except `VerificationMethods`), the request MUST be signed by the DID's `controller`(s).
   2. To update a DIDDoc fragment that has its own `controller`(s), the request MUST be signed by the DID's `controller`(s) **and** the DIDDoc fragment's `controller`(s).
2. Changing the `controller`(s) associated with  a DID requires a list of signatures as before for changing any field.

### Privacy Considerations

One of the key design decisions in the cheqd DID method is to use separate sets of key pairs for Cosmos / node layer transactions and identity payloads.

Keypairs and accounts on the Cosmos node layer are public, and can be crawled/explored by inspecting transactions on a node or through a block explorer. This therefore poses a privacy risk through correlation, if the identity payloads were signed using the same keys.

By splitting the keys/accounts for the two layers, we account for identity payloads being signed using keys that can be kept off-ledger.

This also allows for uses cases where the key owners/controllers at the identity layer are different than the key/account owners at the Cosmos node layer. In essence, that this allows is it removes the need for a DID's controllers to also have an account on the cheqd network ledger.

Further discussion on how these boundaries are separated in implementation, with one specific implementation library, is described in [ADR 003: **Command Line Interface (CLI) tools**](adr-003-cli-tools.md)

### Changes from Indy entities and transactions

#### Rename `NYM` transactions to `DID` transactions

[**NYM** is the term used by Hyperledger Indy](https://hyperledger-indy.readthedocs.io/projects/node/en/latest/transactions.html#nym) for DIDs. cheqd uses the term `DID` instead of `NYM` in transactions, which should make it easier to understand the context of a transaction easier by bringing it closer to W3C DID terminology used by the rest of the SSI ecosystem.

#### Removing `role` field from DID transactions

Hyperledger Indy is a public-permissioned distributed ledger and therefore use the `role` field to distinguish transactions from different types of nodes. As cheqd networks are public-permissionless, the `role` scope has been removed.

#### `ATTRIB` transactions dropped

`ATTRIB` was originally used in Hyperledger Indy to add document content similar to DID Documents (DIDDocs). The cheqd DID method replaces this by implementing DIDDocs for most transaction types.

#### Included resource metadata within DIDDoc metadata

To support the [cheqd Resource Module](adr-008-ledger-resources.md), cheqd ledger includes a reference to resource previews within the DIDDoc metadata.

## Decision

Identity entities and transactions for the cheqd network may differ in name from those in Hyperledger Indy, but aim enable equivalent support for privacy-respecting SSI use cases.

The differences stem primarily from aiming to achieve better compliance with the W3C [DID Core](https://www.w3.org/TR/did-core/) specification and architectural differences between Hyperledger Indy and [Cosmos SDK](https://github.com/cosmos/cosmos-sdk) (used to build `cheqd-node`).

With better compliance against the DID Core specification, the goal of the **cheqd DID method** is to maximise interoperability with compatible third-party software librarires, tools and projects in the SSI ecosystem.

## Consequences

### Backward Compatibility

- The cheqd DID method does not aim to be 1:1 compatible in API methods with `did:indy`. It makes opinionated choices to not implement certain transaction types, which in our analysis have been superseded by new developments in the W3C DID Core specification.
- Support for Indy-style DID unique identifiers is intended to provide a compatibility mode with existing Indy-based client applications.
- `cheqd-node` [release v0.1.19](https://github.com/cheqd/cheqd-node/releases/tag/v0.1.19) and earlier had a transaction type called `NYM` which would allow writing/reading a unique identifier on ledger. However, this `NYM` state was not fully defined as a DID method and did not store DID Documents associated with a DID. This `NYM` transaction type is deprecated and the data written to cheqd testnet with legacy states will not be retained.

### Positive

- Design decisions defined in this ADR aim to make the cheqd DID method close to compliance with the W3C DID Core specification.
- Usage of UUID-style identifiers significantly simplifies the generation and implementation of unique identifiers, since any client application can generate these UUIDs using their own preferred implementation in any programming language, as opposed to the method-specific logic required for Indy-style DID identifiers.
- As the client/peer-to-peer exchange layer (at least in the implementation provided by [VDR Tools SDK](https://gitlab.com/evernym/verity/vdr-tools)) is built on a library that supports Hyperledger Aries, extending Aries implementations to other W3C compliant DID methods should become simpler for the SSI ecosystem.

### Negative

- Trying to maintain backwards-compatibility with Hyperledger Indy APIs means some functionality will not be available for legacy client applications which cannot support non-Indy DID Document elements, e.g., multiple verification methods in the same DIDDoc.

### Neutral

- DID transaction operations at the moment must be assembled using a client-side library with DID specification identity standards support, and then wrapped up inside a Cosmos transaction that is sent to the Tendermint RPC or Cosmos SDK gRPC interface. We aim to build client apps/SDKs that can be used by developers to make the process of interacting with the ledger simpler.

## References

- [Hyperledger Indy](https://wiki.hyperledger.org/display/indy) official project background on Hyperledger Foundation wiki
  - [`indy-node`](https://github.com/hyperledger/indy-node) GitHub repository: Server-side blockchain node for Indy ([documentation](https://hyperledger-indy.readthedocs.io/projects/node/en/latest/index.html))
  - [`indy-plenum`](https://github.com/hyperledger/indy-plenum) GitHub repository: Plenum Byzantine Fault Tolerant consensus protocol; used by `indy-node` ([documentation](https://hyperledger-indy.readthedocs.io/projects/plenum/en/latest/index.html))
  - [Indy DID method](https://hyperledger.github.io/indy-did-method/) (`did:indy`)
  - [Indy identity-domain transactions](https://github.com/hyperledger/indy-node/blob/master/docs/source/transactions.md)
- [Hyperledger Aries](https://wiki.hyperledger.org/display/ARIES/Hyperledger+Aries) official project background on Hyperledger Foundation wiki
  - [`aries`](https://github.com/hyperledger/aries) GitHub repository: Provides links to implementations in various programming languages
  - [`aries-rfcs`](https://github.com/hyperledger/aries-rfcs) GitHub repository: Contains Requests for Comment (RFCs) that define the Aries protocol behaviour
- [W3C Decentralized Identifiers (DIDs)](https://www.w3.org/TR/did-core/) specification
  - [DID Core Specification Test Suite](https://w3c.github.io/did-test-suite/)
- [Cosmos blockchain framework](https://cosmos.network/) official project website
  - [`cosmos-sdk`](https://github.com/cosmos/cosmos-sdk) GitHub repository ([documentation](https://docs.cosmos.network/))
- [Sovrin Foundation](https://sovrin.org/)
  - [Sovrin Networks](https://sovrin.org/overview/)
  - [`libsovtoken`](https://github.com/sovrin-foundation/libsovtoken): Sovrin Network token library
  - [Sovrin Ledger token plugin](https://github.com/sovrin-foundation/token-plugin)
