# ADR 008: cheqd DIDDoc resources: Schemas and Credential Definitions

## Status

| Category                  | Status                                           |
| ------------------------- | ------------------------------------------------ |
| **Authors**               | Renata Toktar, Alexander Kolesov, Ankur Banerjee |
| **ADR Stage**             | DRAFT                                            |
| **Implementation Status** | Draft                                            |
| **Start Date**            | 2021-09-23                                       |

## Summary

This ADR defines the cheqd DID method and describes the identity entities, queries, and transaction types for the cheqd network: a purpose-built self-sovereign identity (SSI) network based on the [Cosmos blockchain framework](https://github.com/cosmos/cosmos-sdk).

This ADR will define how Verifiable Credential schemas can be represented through the use of a DID URL, which when dereferenced, fetches the credential schemas a resource.
The identity entities and transactions for the cheqd network are designed to support usage scenarios and functionality currently supported by [Hyperledger Indy](https://github.com/hyperledger/indy-node).

## Context

Hyperledger Indy is a verifiable data registry (VDR) built for DIDs with a strong focus on privacy-preserving techniques. It is one of the most widely-adopted SSI blockchain ledgers. Most notably, Indy is used by the [Sovrin Network](https://sovrin.org/overview/).

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

## Decision

### DID Resources

#### Assumptions

* Immutability:
  * Resources are immutable, so can't be updated/removed;
* Limitations
  * Resource size is now limited by maximum tx/block size;

#### Future improvements

* Limitations
  * Introduce module level resource size limit that can be changed by voting

#### 'Resources' module on ledger

A new module will be created: `resource`.

#### Dependencies

* It will have `cheqd` module as a dependency.
  * Will be used for DIDs existence checks.
  * Will be used for authentication

### Types

#### Resource

* **Collection ID: UUID ➝** (did:cheqd:...:)**`UUID` (supplied client-side)**
* **ID: UUID ➝ specific to resource, also effectively a version number (supplied client-side)**
* **Name: String (e.g., `CL-Schema1` (supplied client-side)**
* **ResourceType (`CL-Schema`/`JSONSchema2020`) (supplied client-side)**
* **MimeType: (`application/json`/`image`/`application/octet-stream`/`text/plain`) (supplied client-side)**
* **Data: Byte\[\] (supplied client-side)**
* Created: XMLDatetime (computed ledger-side)
* Checksum: SHA-256 (computed ledger-side)
* previousVersionId: `null` if first, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)
* nextVersionId: `null` if first/latest, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)

Example:

```jsonc
{
  "collection_id":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":               "CL-Schema1",
  "resource_type":      "CL-Schema",
  "mime_type":          "application/json"
  "data":               <json string '{\"attrNames\":[\"last_name\",\"first_name\"]}` in bytes>,
  "created":            "2022-04-20T20:19:19Z",
  "checksum":           "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
  "previous_version_id: null,
  "next_version_id:     null
}
```

#### ResourcePreview

* **Collection ID: UUID ➝** (did:cheqd:...:)**`UUID` (supplied client-side)**
* **ID: UUID ➝ specific to resource, also effectively a version number (supplied client-side)**
* **Name: String (e.g., `CL-Schema1` (supplied client-side)**
* **ResourceType (`CL-Schema`/`JSONSchema2020`) (supplied client-side)**
* **MimeType: (`application/json`/`image`/`application/octet-stream`/`text/plain`) (supplied client-side)**
* Created: XMLDatetime (computed ledger-side)
* Checksum: SHA-256 (computed ledger-side)
* previousVersionId: `null` if first, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)
* nextVersionId: `null` if first/latest, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)

Example:

```jsonc
{
  "collection_id":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":               "CL-Schema1",
  "resource_type":      "CL-Schema",
  "mime_type":          "application/json"
  "created":            "2022-04-20T20:19:19Z",
  "checksum":           "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
  "previous_version_id: null,
  "next_version_id:     null
}
```

#### MsgCreateResource

* **Collection ID: UUID ➝** (did:cheqd:...:)`<identifier>` (supplied client-side)** - a parent DIDDoc identifier from `id` field
* **ID: UUID ➝ specific to resource, also effectively a version number (supplied client-side)**
* **Name: String (e.g., `CL-Schema1` (supplied client-side)**
* **ResourceType (`CL-Schema`/`JSONSchema2020`) (supplied client-side)**
* **MimeType: (`application/json`/`image`/`application/octet-stream`/`text/plain`) (supplied client-side)**
* **Data: Byte\[\] (supplied client-side)**

Example:

```jsonc
{
  "collection_id":  "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":             "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":           "CL-Schema1",
  "resource_type":  "CL-Schema",
  "mime_type":      "application/json"
  "data":           <json string '{\"attrNames\":[\"last_name\",\"first_name\"]}` in bytes>
}
```

#### MsgCreateResourceResponse

* Resource: [Resource](#resource)

Example:

```jsonc
{ "resource":  <Resource> }
```

#### QueryGetCollectionResourcesRequest

* Collection ID: String - an identifier of linked DIDDoc
  
Example:

```jsonc
{ "collection_id": "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY" }
```

#### QueryGetCollectionResourcesResponse

* Resources: [ResourcePreview\[\]](#resourcepreview)

Example:

```jsonc
{ "resources":  [<ResourcePreview1>, <ResourcePreview2>] }
```

#### QueryGetResourceRequest

* Collection ID: String - an identifier of linked DIDDoc
* ID: String - unique resource id

Example:

```jsonc
{ 
  "collection_id": "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id": "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096"
}
```

#### QueryGetResourceResponse

* Resource: [Resource](#resource)

Example:

```jsonc
{ "resource":  <Resource> }
```

#### QueryGetAllResourceVersionsRequest

* Collection ID: String - an identifier of linked DIDDoc
* Name: String
* ResourceType: String
* MimeType: String

Example:

```jsonc
{ 
  "collection_id":  "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "name":           "CL-Schema1",
  "resource_type":  "CL-Schema",
  "mime_type":      "application/json"
}
```

#### QueryGetAllResourceVersionsResponse

* Resources: [ResourcePreview\[\]](#resourcepreview)

Example:

```jsonc
{ "resources":  [<ResourcePreview1>, <ResourcePreview2>] }
```

### State

* `resources:<collection-id>:<resource-id>` ➝ **Resource**
  * `<collection-id>` is the last part of DID. It can be UUID, Indy-style or whatever is allowed by ledger. It allows us to evolve over time more easily.
  * `<resource-id>` is a unique resource identifier on UUID format

### Transactions

#### CreateResource

* Input:
  * [MsgCreateResource](#msgcreateresource)

* Output:
  * [MsgCreateResourceResponse](#msgcreateresourceresponse)

* Processing logic:
  * Check that associated DIDDoc exists;
  * Authenticate request the same way as DIDDoc creation and updating;
  * Validate properties;
  * Validate **data** for specific resource types (CL Schema);
  * Validate that **ID** is unique;
  * Set **created** date time;
  * Set `previousVersion` and `nextVersion` if this is a new version (a resource with the same name, type ans resource-type exists);
  * Compute **checksum**;
  * Persist the **resource** in state;

CLI Example:

```jsonc
cheqd-noded tx resource create-resource "{
                                          \"collection_id\":  \"zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY\",
                                          \"id\":             \"9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096\",
                                          \"name\":           \"CL-Schema1\",
                                          \"resource_type\":  \"CL-Schema\",
                                          \"mime_type\":      \"application/json\"
                                         }" file_with_resource.data\
                                          --private-key <private-identity-key-by-collection-

```

### Queries

#### GetCollectionResources

* Input:
  * [QueryGetCollectionResourcesRequest](#querygetcollectionresourcesrequest)

* Output:

  * [QueryGetCollectionResourcesResponse](#querygetcollectionresourcesresponse)

* Processing logic:

  * Retrieves the whole resource collection for the specified DID;
  * Returns only resources preview without `data` field;
  
#### GetResource

* Input:
  * [QueryGetResourceRequest](#querygetresourcerequest)

* Output:

  * [QueryGetResourceResponse](#querygetresourceresponse)

* Processing logic:
  * Retrieves a specific resource by Collection-ID and resource ID;

#### GetAllResourceVersions

* Input:
  * [QueryAllResourceVersionsRequest](#querygetallresourceversionsrequest)

* Output:
  * [QueryAllResourceVersionsResponse](#querygetallresourceversionsresponse)

* Processing logic:
  * Retrieves all resource versions by collection id, resource name, resource type and mime type;
  * Returns only resources preview without `data` field;
  
### DID Resolver

We need to support resource resolution in the DID resolver.

#### Resource resolution

* Input DIDUrl:
  * `https://resolver.cheqd.net/1.0/identifiers/<did>/resources/<resource-id>`

* Output:
  * JSON encoded [QueryGetResourceResponse](#querygetresourcerequest)

* Processing logic:
  * Simply call [GetResource](#getresource) via GRPC

### Linked DIDDoc

`CollectionId` field is an identifier of existing DIDDoc. There are no restrictions on the fields of this DIDDoc other than those described in [Cheqd DID Method ADR](adr-002-cheqd-did-method.md) and [W3C DID specification](https://www.w3.org/TR/did-core/). DIDDock must be located in the same ledger where the resource is created.
A list of resources related to DIDDoc can be found in its metadata:

```jsonc
QueryGetDidResponse {
  "did": {
    "id": "did:cheqd:mainnet:N22KY2Dyvmuu2PyyqSFKue",
    ...
  },
  "metadata": {
    "created": "2020-12-20T19:17:47Z",
    "updated": "2020-12-20T19:19:47Z",
    "deactivated": false,
    "versionId": "1B3B00849B4D50E8FCCF50193E35FD6CA5FD4686ED6AD8F847AC8C5E466CFD3E",
    "resources": [
      {
        "resourceURI":      "did:cheqd:mainnet:N22KY2Dyvmuu2PyyqSFKue/resources/9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
        "resourceType":      "CL-Schema",
        "mimeType":          "application/json"
      }
    ]
  }
}
```

```jsonc
Resource {
  "collection_id":      "N22KY2Dyvmuu2PyyqSFKue",
  "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":               "CL-Schema1",
  "resource_type":      "CL-Schema",
  "mime_type":          "application/json"
  "data":               <json string '{\"attrNames\":[\"last_name\",\"first_name\"]}` in bytes>,
  "created":            "2022-04-20T20:19:19Z",
  "checksum":           "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
  "previous_version_id: null,
  "next_version_id:     null
}
```

### Resource versioning

Resource are immutable, but it is possible to create new versions of it under a new identifier(`id` field). When creating a resource whose fields `collectionId`, `name`, `resourceType`, `mimeType` match an existing resource:

* The latest version of the current resource will be added with a link to the new one. That is, field `nextVersionId` will contain the new resource identifier.
* A new resource with data from the transaction will be created with the previous version resource id in field `previousVersionId`.

Example:

Step 1. Resource exists in the ledger:

  ```jsonc
  Resource1
  {
    "collection_id":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
    "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
    "name":               "CL-Schema1",
    "resource_type":      "CL-Schema",
    "mime_type":          "application/json"
    ...
    "previous_version_id: "12d5a5f6-e72d-11ec-8fea-0242ac120002",
    "next_version_id:     null
  }
  ```

Step 2. Client send request for creating a new resource with a transaction MsgCreateResource

  ```jsonc
  MsgCreateResource for creating Resource2
  {
    "collection_id":  "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
    "id":             "f47e4790-1b4b-4186-8357-da6199665236",
    "name":           "CL-Schema1",
    "resource_type":  "CL-Schema",
    "mime_type":      "application/json"
    "data":           ...
  }
  ```

Step 3. After the transaction applying

```jsonc
Resource1
{
  "collection_id":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":               "CL-Schema1",
  "resource_type":      "CL-Schema",
  "mime_type":          "application/json"
  ...
  "previous_version_id: "12d5a5f6-e72d-11ec-8fea-0242ac120002",
  "next_version_id:     "f47e4790-1b4b-4186-8357-da6199665236"  // Resource2.id
}
```

```jsonc
Resource2
{
  "collection_id":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":               "CL-Schema1",
  "resource_type":      "CL-Schema",
  "mime_type":          "application/json"
  ...
  "previous_version_id: "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096", // Resource1.id
  "next_version_id:     null
}
```

### CL Schema

CL-Schema resource can be created via `CreateResource` transaction with the follow list of parameters:

MsgCreateResource:

* **Collection ID: UUID ➝** (did:cheqd:...:) ➝ Parent DID identifier without a prefix
* **ID: UUID ➝ specific to resource, also effectively a version number (supplied client-side)**
* **Name: String (e.g., `CL-Schema1` )** ➝ Schema name
* **ResourceType**  ➝ `CL-Schema`
* **MimeType**  ➝  `application/json`
* **Data: Byte\[\]** ➝ JSON string with the follow structure:
  * **attrNames**: Array of attribute name strings (125 attributes maximum)
  
  ```jsonc
  {
    "attrNames": ["undergrad", "last_name", "first_name", "birth_date", "postgrad", "expiry_date"]
  }
  ```

CLI Example:

```jsonc
cheqd-noded tx resource create-cl-schema <collection_id> <id> <name> <schema-data-json>
                                          --private-key <private-identity-key-by-collection-id-diddoc>

cheqd-noded tx resource create-cl-schema zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY\
                                         9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096\
                                         CLSchema1\
                                         "{\"attrNames\":[\"last_name\", \"first_name\"]}"\
                                          --private-key <private-identity-key-by-collection-id-diddoc>

```

### Credential Definition

[TODO: explain that a Cred Def is simply an additional property inside of
the Issuer's DID Doc]

Adds a Credential Definition (in particular, public key), which is created by an
Issuer and published for a particular Credential Schema.

It is not possible to update Credential Definitions. If a Credential Definition
needs to be evolved (for example, a key needs to be rotated), then a new
Credential Definition needs to be created for a new Issuer DIDdoc.
Credential Definitions is added to the ledger in as verification method for
Issuer DIDDoc

* **`id`**: DID as base58-encoded string for 16 or 32 byte DID value with Cheqd
DID Method prefix `did:cheqd:<namespace>:` and a resource
type at the end.
* **`value`** (dict): Dictionary with Credential Definition's data if
`signature_type` is `CL`:
  * **`primary`** (dict): Primary credential public key
  * **`revocation`** (dict, optional): Revocation credential public key
* **`schemaId`** (string): `id` of a Schema the credential definition is created
for.
* **`signatureType`** (string): Type of the credential definition (that is
credential signature). `CL-Sig-Cred_def` (Camenisch-Lysyanskaya) is the only
supported type now. Other signature types are being explored for future releases.
* **`tag`** (string, optional): A unique tag to have multiple public keys for
the same Schema and type issued by the same DID. A default tag `tag` will be
used if not specified.
* **`controller`**: DIDs list of strings or only one string of a credential
definition controller(s). All DIDs must exist.

`CRED_DEF` entity transaction format:

```jsonc
{
    "id": "<cred_def_url>",
    "type": "CL-CredDef",
    "controller": "did:cheqd:mainnet-1:123456789abcdefghi",
    "schemaId": "did:cheqd:mainnet-1:5ZTp9g4SP6t73rH2s8zgmtqdXyT?service=CL-Schema",
    "tag": "some_tag",
    "value": {
      "primary": "...",
      "revocation": "..."
    }
}
```

Don't store Schema DIDDoc in the State.

CredDef URL: `did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue`

CredDef Entity URL: `did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue?service=CL-CredDef`

`CRED_DEF` DID Document transaction format:

```jsonc
{
  "id": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue",
  "controller": "did:cheqd:mainnet-1:IK22KY2Dyvmuu2PyyqSFKu", // CredDef Issuer DID
  "service":[
    {
      "id": "cheqd-cred-def",
      "type": "CL-CredDef",
      "serviceEndpoint": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue?service=CL-CredDef"
    }
  ]
}
```

`CRED_DEF` state format:

`"credDef:<id>" -> {CredDefEntity, txHash, txTimestamp}`

**Note**: `CRED_DEF` **cannot** be updated.

### Rationale and Alternatives

#### Schema options not used

##### Schema. Option 2

Schema URL: `did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue#<schema_entity_id>`

`SCHEMA` DID Document transaction format:

```jsonc
{
  "id": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue",
  "schema":[
    {
      "id": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue#schema1",
      "type": "CL-Schema",
      "controller": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue",
      "value": {
                "version": "1.0",
                "name": "Degree",
                "attrNames": ["undergrad", "last_name", "first_name", "birth_date", "postgrad", "expiry_date"]
              },
    },
  ]
}
```

##### Schema. Option 3

Schema URL: `did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue`

`SCHEMA` DID Document transaction format:

```jsonc
{
  "id": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue",
  "schema": {
              "id": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue",
              "type": "CL-Schema",
              "controller": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue",
              "value": {
                "version": "1.0",
                "name": "Degree",
                "attrNames": ["undergrad", "last_name", "first_name", "birth_date", "postgrad", "expiry_date"]
              },
            },
}
```

##### Schema. Option 4

Schema URL: `did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue#<schema_entity_id>`

`SCHEMA` DID Document transaction format:

```jsonc
{
  "id": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue",
  "schema":[
              {
                "id": "cheqd-schema",
                "type": "CL-Schema",
                "schemaRef": "did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue?resource=true"
              }
          ]
}
```

`SCHEMA` State format:

* `"schema:<id>" -> {SchemaEntity, txHash, txTimestamp}`

`id` example: `did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue`

#### Cred Def options not used

##### Cred Def. Option 2

Store inside Issuer DID Document

CredDef URL: `did:cheqd:mainnet-1:N22KY2Dyvmuu2PyyqSFKue#<cred_def_entity_id>`

`CRED_DEF` DID Document transaction format:

```jsonc
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/jws-2020/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "verificationMethod": [
    {
      "id": "did:cheqd:mainnet-1:IK22KY2Dyvmuu2PyyqSFKu#creddef-1", // Cred Def ID
      "type": "CamLysCredDefJwk2021", // TODO: define and register this key type
      "controller": "did:cheqd:mainnet-1:IK22KY2Dyvmuu2PyyqSFKu"
      "publicKeyJwk":
      {
        [TODO: Define structure for CL JWK]
      }
    }
  ],
  "assertionMethod": [
    "#creddef-1"
  ]

}
```

`CRED_DEF` state format:

###### Positive

* Credential Definition is a set of Issuer keys. So storing them in Issuer's DIDDoc reasonable.

###### Negative

* Credential Definition name means that it contains more than just a key and `value` field
  provides this flexibility.
* Adding all Cred Defs to Issuer's DIDDoc makes it too large. For every DIDDoc or Cred Def request
  a client will receive the whole list of Issuer's Cred Defs.
* Impossible to put a few controllers for Cred Def.
* In theory, we need to make Credential Definitions mutable.

## References

* [Hyperledger Indy](https://wiki.hyperledger.org/display/indy) official project background on Hyperledger Foundation wiki
  * [`indy-node`](https://github.com/hyperledger/indy-node) GitHub repository: Server-side blockchain node for Indy ([documentation](https://hyperledger-indy.readthedocs.io/projects/node/en/latest/index.html))
  * [`indy-plenum`](https://github.com/hyperledger/indy-plenum) GitHub repository: Plenum Byzantine Fault Tolerant consensus protocol; used by `indy-node` ([documentation](https://hyperledger-indy.readthedocs.io/projects/plenum/en/latest/index.html))
  * [Indy DID method](https://hyperledger.github.io/indy-did-method/) (`did:indy`)
  * [Indy identity-domain transactions](https://github.com/hyperledger/indy-node/blob/master/docs/source/transactions.md)
* [Hyperledger Aries](https://wiki.hyperledger.org/display/ARIES/Hyperledger+Aries) official project background on Hyperledger Foundation wiki
  * [`aries`](https://github.com/hyperledger/aries) GitHub repository: Provides links to implementations in various programming languages
  * [`aries-rfcs`](https://github.com/hyperledger/aries-rfcs) GitHub repository: Contains Requests for Comment (RFCs) that define the Aries protocol behaviour
* [W3C Decentralized Identifiers (DIDs)](https://www.w3.org/TR/did-core/) specification
  * [DID Core Specification Test Suite](https://w3c.github.io/did-test-suite/)
* [Cosmos blockchain framework](https://cosmos.network/) official project website
  * [`cosmos-sdk`](https://github.com/cosmos/cosmos-sdk) GitHub repository ([documentation](https://docs.cosmos.network/))
* [Sovrin Foundation](https://sovrin.org/)
  * [Sovrin Networks](https://sovrin.org/overview/)
  * [`libsovtoken`](https://github.com/sovrin-foundation/libsovtoken): Sovrin Network token library
  * [Sovrin Ledger token plugin](https://github.com/sovrin-foundation/token-plugin)
