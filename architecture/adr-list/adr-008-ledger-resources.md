# ADR 008: On-ledger Resources with DIDs

## Status

| Category | Status |
| :--- | :--- |
| **Authors** | Ankur Banerjee, Alexandr Kolesov, Alex Tweeddale, Renata Toktar   |
| **ADR Stage** | PROPOSED |
| **Implementation Status** | Not Implemented |
| **Start Date** | 2021-09-23 |
| **Last Updated** | 2022-06-24 |

## Summary

This ADR defines how on-ledger resources (e.g., text, JSON, images, etc) can be referenced using [a permanent and unique `did:cheqd` identifier](adr-002-cheqd-did-method.md), with create/update operations controlled using the specified DIDs.

## Context

### "Resources" in decentralised identity

[Trust over IP Foundation (ToIP)](https://trustoverip.org/) describes [how "resources" could be generically defined and accessed using DID URLs](https://wiki.trustoverip.org/display/HOME/DID+URL+Resource+Parameter+Specification). In a self-sovereign identity (SSI) ecosystem, such resources are often required in tandem with [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model/), which is a standard way of representing portable digital credentials that represent claims about its subjects and can be verified via digital proofs.

Common types of resources that might be required to issue and validate Verifiable Credentials are:

* **Schemas**: Describe [the fields and content types in a credential](https://w3c.github.io/vc-data-model/#data-schemas) in a machine-readable format. Prominent examples of this include [Schema.org](https://schema.org/docs/schemas.html), [Hyperledger Indy `SCHEMA` objects](https://hyperledger-indy.readthedocs.io/projects/node/en/latest/transactions.html#schema), etc.
* **Revocation status lists**: Allow recipients of a Verifiable Credential exchange to [check the revocation status of a credential](https://w3c.github.io/vc-data-model/#validity-checks) for validity. Prominent examples of this include the [W3C `Status List 2021`](https://w3c-ccg.github.io/vc-status-list-2021/) specification, [W3C `Revocation List 2020`](https://w3c-ccg.github.io/vc-status-rl-2020/), [Hyperledger Indy revocation registries](https://hyperledger-indy.readthedocs.io/projects/sdk/en/latest/docs/concepts/revocation/cred-revocation.html), etc.
* **Visual representations for Verifiable Credentials**: Although Verifiable Credentials can be exchanged digitally, in practice most identity wallets want to present "human-friendly" representations. This allows the credential representation to be shown according to the brand guidelines of the issuer, [internationalisation ("i18n") translations](https://en.wikipedia.org/wiki/Internationalization_and_localization), etc. Examples of this include the [Overlays Capture Architecture (OCA) specification](https://oca.colossi.network/), [Apple Wallet PassKit](https://developer.apple.com/documentation/walletpasses) ("`.pkpass`"), [Google Wallet Pass](https://developers.google.com/wallet/generic), etc.

![Mobile boarding passes for Apple Wallet](../../.gitbook/assets/mobile-boarding-pass.jpeg)
Such visual representations can also be used to quickly communicate information visually during identity exchanges, such as airline mobile boarding passes. In the [example above from British Airways](https://mediacentre.britishairways.com/pressrelease/details/86/2016-72/6130), the pass at the front is for a "Gold" loyalty status member, whereas the pass at the back is for a "standard" loyalty status member. This information can be represented in a Verifiable Credential, of course, but the example here uses the Apple Wallet / Google Wallet formats to overlay a richer, "human-friendly" display.

More broadly, there are other resources that might be relevant for issuers and verifiers in a self-sovereign identity exchange:

* **Documents related to SSI ecosystems**: [ToIP recommends making Governance Frameworks available through DID URLs](https://wiki.trustoverip.org/pages/viewpage.action?pageId=71241), which would typically be a text file, a [Markdown file](https://en.wikipedia.org/wiki/Markdown), PDF etc. This, for example, can enable parties building self-sovereign identity ecosystems to use DIDs to reference Governance Frameworks they conform to, at different levels of the technical stack. 
* **Logos**: Issuers may want to provide authorised image logos to display in relation to their DID or Verifiable Credentials. Examples of this include [key-publishing sites like Keybase.io](https://keybase.io/cheqd_identity) (which is used by [Cosmos SDK block explorers such as our own](https://explorer.cheqd.io/validators) to show logos for validators) and "[favicons](https://en.wikipedia.org/wiki/Favicon)" (commonly used to set the logo for websites in browser tabs).

### Rationale for storing resources on-ledger

Decentralized Identifiers (DIDs) are often stored on ledgers (e.g., [cheqd](adr-002-cheqd-did-method.md), [Hyperledger Indy](https://hyperledger.github.io/indy-did-method/#:~:text=The%20DID%20Indy%20method%20specification,ledgers%20are%20W3C%20standard%20DIDs.), distributed storage (e.g., [IPFS](https://ipfs.io/) in [Sidetree](https://identity.foundation/sidetree/spec/)), or non-ledger distributed systems (e.g., [KERI](https://keri.one/)).

#### Drawbacks of hosting resources on traditional web endpoints

DIDs *can* be stored on traditional centralised-storage endpoints (e.g., [`did:web`](https://w3c-ccg.github.io/did-method-web/), [`did:git`](https://github-did.com/)) but this comes with certain drawbacks:

1. **DIDs could be tampered by compromising the hosting provider**: DIDs and DID Documents ("DIDDocs") stored at a centralised web endpoint can be compromised and replaced by malicious actors.
2. **Hosting providers could unilaterally cease to host particular clients**: Hosting providers could terminate accounts due to factors such as non-payment of fees, violation of Terms of Service, etc.
3. **Single point-of-failure in resiliency**: Even for highly-trusted and sophisticated hosting providers who may not present a risk of infrastructure being compromised, a service outage at the hosting provider can make a DID anchored on their systems inaccessible.
   1. See [notable examples of service outages](https://totaluptime.com/notable-network-and-cloud-outages-of-2021/) from major cloud providers: [Amazon Web Services (AWS)](https://awsmaniac.com/aws-outages/), [Microsoft Azure](https://www.theregister.com/2018/09/17/azure_outage_report/), [Google Cloud](https://www.thousandeyes.com/blog/google-cloud-platform-outage-analysis), [Facebook / Meta](https://en.wikipedia.org/wiki/2021_Facebook_outage), [GitHub](https://github.blog/2022-03-23-an-update-on-recent-service-disruptions/), [Cloudflare](https://blog.cloudflare.com/cloudflare-outage-on-june-21-2022/)...
   ![Graph showing drop in Facebook traffic from their global service outage in 2021](../../.gitbook/assets/facebook-outage.png)
   1. In particular, the Facebook outage ([shown in the graph above](https://www.kentik.com/blog/facebooks-historic-outage-explained/)) also [took down apps that used "Login with Facebook"](https://web.archive.org/web/20211005032128/https://www.wired.com/story/why-facebook-instagram-whatsapp-went-down-outage/) functionality. This highlights the risks of "contagion impact" (e.g., [a *different* Facebook outage took down Spotify, TikTok, Pinterest](https://www.engadget.com/facebook-sdk-spotify-tinder-tiktok-ios-outage-125806814.html)) of centralised digital systems - even ones run by extremely-capable tech providers.
4. **Link rot**: "Link rot" happens when over time, URLs become inaccessible, either because the endpoint where the content was stored is no longer active, or the URL format itself changes. The graph below from [an analysis by *The New York Times* of linkrot](https://www.cjr.org/analysis/linkrot-content-drift-new-york-times.php) shows degradation over time of URLs.
  ![Linkrot analysis by New York Times](../../.gitbook/assets/linkrot.jpeg)

#### Risks applicable in the context of Verifiable Credentials

The issues highlighted above **a material difference to the longevity of Verifiable Credentials**. For example, a passport ([which typically have a 5-10 year validity]((https://en.wikipedia.org/wiki/Passport_validity))) issued as a Verifiable Credential anchored to a DID (regardless of whether the DID was on-ledger or not) might stop working if the credential schema, visual presentation format, or other necessary resources were stored off-ledger on traditional centralised storage.

Despite these issues, many self-sovereign identity (SSI) implementations - *even ones that use ledgers / distributed systems for DIDs* -  often utilise centralised storage. From the [W3C Verifiable Credential Implementation Guide](https://w3c.github.io/vc-imp-guide/#creating-new-credential-types):

> Example schema.org address with full URLs
>
> ```jsonc
> {
>   "@type": "http://schema.org/Person",
>   "http://schema.org/address": {
>     "@type": "http://schema.org/PostalAddress",
>     "http://schema.org/streetAddress": "123 Main St.",
>     "http://schema.org/addressLocality": "Blacksburg",
>     "http://schema.org/addressRegion": "VA",
>     "http://schema.org/postalCode": "24060",
>     "http://schema.org/addressCountry": "US"
>   }
> }
> ```

Using traditional web endpoints to store resources (such as schemas) that are critical for a Verifiable Credential to function undermines the benefits that persistently-accessible Decentralized Identifiers offer.

## Resources on cheqd ledger



### DID Resource Creation Flow

![Resource Creation Flow](../../assets/adr008-identity-resources-flow.png)

### Types

#### ResourceHeader

* **Collection ID: UUID ➝** (did:cheqd:...:)**`UUID` (supplied client-side)**
* **ID: UUID ➝ specific to resource, also effectively a version number (supplied client-side)**
* **Name: String (e.g., `CL-Schema1` (supplied client-side)**
* **ResourceType (supplied client-side)** Possible values:
  * `CL-Schema`
  * `JSONSchema2020`
* **MimeType: (`application/json`/`image`/`application/octet-stream`/`text/plain`) (supplied client-side)** Possible values:
  * `application/json`
  * `application/octet-stream`
  * `text/plain`
  * `image/apng`
  * `image/avif`
  * `image/gif`
  * `image/jpeg`
  * `image/heic`
  * `image/png`
  * `image/svg+xml`
  * `image/webp`
* Created: XMLDatetime (computed ledger-side)
* Checksum: SHA-256 (computed ledger-side)
* previousVersionId: `null` if first, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)
* nextVersionId: `null` if first/latest, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)

Example:

```jsonc
{
  "collectionId":   "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":           "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name": "CL-Schema1",
  "resourceType": "CL-Schema",
  "mimeType": "application/json",
  "created": "2022-04-20T20:19:19Z",
  "checksum":           "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
  "previousVersionId": null,
  "nextVersionId":     null
}
```

#### Resource

* **Header: ResourceHeader**
* **Data: Byte\[\] (supplied client-side)**

Example:

```jsonc
{
  "header": {
    "collectionId":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
    "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
    "name":               "CL-Schema1",
    "resourceType":      "CL-Schema",
    "mimeType":          "application/json"
    "created":            "2022-04-20T20:19:19Z",
    "checksum":           "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
    "previousVersionId: null,
    "nextVersionId:     null
  },
  "data":               <json string '{\"attrNames\":[\"last_name\",\"first_name\"]}` in bytes>,
}
```

<details>
<summary>ResourcePreview</summary>

#### ResourcePreview

* **Collection ID: UUID ➝** (did:cheqd:...:)**`UUID` (supplied client-side)**
* **ID: UUID ➝ specific to resource, also effectively a version number (supplied client-side)**
* **Name: String (e.g., `CL-Schema1` (supplied client-side)**
* **ResourceType (supplied client-side)** Possible values:
  * `CL-Schema`
  * `JSONSchema2020`
* **MimeType: (`application/json`/`image`/`application/octet-stream`/`text/plain`) (supplied client-side)** Possible values:
  * `application/json`
  * `application/octet-stream`
  * `text/plain`
  * `image/apng`
  * `image/avif`
  * `image/gif`
  * `image/jpeg`
  * `image/png`
  * `image/svg+xml`
  * `image/webp`
* Created: XMLDatetime (computed ledger-side)
* Checksum: SHA-256 (computed ledger-side)
* previousVersionId: `null` if first, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)
* nextVersionId: `null` if first/latest, otherwise ID as long as Name, ResourceType, and MimeType match previous version (computed ledger-side)

Example:

```jsonc
{
  "collectionId":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":               "CL-Schema1",
  "resourceType":      "CL-Schema",
  "mimeType":          "application/json"
  "created":            "2022-04-20T20:19:19Z",
  "checksum":           "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
  "previousVersionId: null,
  "nextVersionId:     null
}
```

</details>

<details>
<summary>MsgCreateResource</summary>

#### MsgCreateResource

* **Collection ID: UUID ➝** (did:cheqd:...:)`<identifier>` (supplied client-side)** - a parent DIDDoc identifier from `id` field
* **ID: UUID ➝ specific to resource, also effectively a version number (supplied client-side)**
* **Name: String (e.g., `CL-Schema1` (supplied client-side)**
* **ResourceType (supplied client-side)** Possible values:
  * `CL-Schema`
  * `JSONSchema2020`
* **MimeType: (`application/json`/`image`/`application/octet-stream`/`text/plain`) (supplied client-side)** Possible values:
  * `application/json`
  * `application/octet-stream`
  * `text/plain`
  * `image/apng`
  * `image/avif`
  * `image/gif`
  * `image/jpeg`
  * `image/png`
  * `image/svg+xml`
  * `image/webp`
* **Data: Byte\[\] (supplied client-side)**

Example:

```jsonc
{
  "collectionId":  "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id":             "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":           "CL-Schema1",
  "resourceType":  "CL-Schema",
  "mimeType":      "application/json"
  "data":           <json string '{\"attrNames\":[\"last_name\",\"first_name\"]}` in bytes>
}
```

</details>

<details>
<summary>MsgCreateResourceResponse</summary>

#### MsgCreateResourceResponse

* Resource: [Resource](#resource)

Example:

```jsonc
{ "resource":  <Resource> }
```

</details>

<details>
<summary>QueryGetCollectionResourcesRequest</summary>

#### QueryGetCollectionResourcesRequest

* Collection ID: String - an identifier of linked DIDDoc
  
Example:

```jsonc
{ "collectionId": "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY" }
```

</details>

<details>
<summary>QueryGetCollectionResourcesResponse</summary>

#### QueryGetCollectionResourcesResponse

* Resources: [ResourceHeader\[\]](#resourceheader)

Example:

```jsonc
{ "resources":  [<ResourceHeader1>, <ResourceHeader2>] }
```

</details>

<details>
<summary>QueryGetResourceRequest</summary>

#### QueryGetResourceRequest

* Collection ID: String - an identifier of linked DIDDoc
* ID: String - unique resource id

Example:

```jsonc
{ 
  "collectionId": "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "id": "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096"
}
```

</details>

<details>
<summary>QueryGetResourceResponse</summary>

#### QueryGetResourceResponse

* Resource: [Resource](#resource)

Example:

```jsonc
{ "resource":  <Resource> }
```

</details>

<details>
<summary>QueryGetAllResourceVersionsRequest</summary>

#### QueryGetAllResourceVersionsRequest

* Collection ID: String - an identifier of linked DIDDoc
* Name: String
* ResourceType: String
* MimeType: String

Example:

```jsonc
{ 
  "collectionId":  "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
  "name":           "CL-Schema1",
  "resourceType":  "CL-Schema",
  "mimeType":      "application/json"
}
```

</details>

<details>
<summary>QueryGetAllResourceVersionsResponse</summary>

#### QueryGetAllResourceVersionsResponse

* Resources: [ResourceHeader\[\]](#resourceheader)

Example:

```jsonc
{ "resources":  [<ResourceHeader1>, <ResourceHeader2>] }
```

</details>

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
                                          \"collectionId\":  \"zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY\",
                                          \"id\":             \"9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096\",
                                          \"name\":           \"CL-Schema1\",
                                          \"resourceType\":  \"CL-Schema\",
                                          \"mimeType\":      \"application/json\"
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
  * Returns only resource headers (without `data` field);
  
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
  * Returns only resource headers (without `data` field);
  
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

`CollectionId` field is an identifier of existing DIDDoc. There are no restrictions on the fields of this DIDDoc other than those described in [cheqd DID Method ADR](adr-002-cheqd-did-method.md) and [W3C DID specification](https://www.w3.org/TR/did-core/). DIDDoc must be located in the same ledger where the resource is created.
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
  "collectionId":      "N22KY2Dyvmuu2PyyqSFKue",
  "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
  "name":               "CL-Schema1",
  "resourceType":      "CL-Schema",
  "mimeType":          "application/json"
  "data":               <json string '{\"attrNames\":[\"last_name\",\"first_name\"]}` in bytes>,
  "created":            "2022-04-20T20:19:19Z",
  "checksum":           "a7c369ee9da8b25a2d6e93973fa8ca939b75abb6c39799d879a929ebea1adc0a",
  "previousVersionId: null,
  "nextVersionId:     null
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
    "header": {
      "collectionId":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
      "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
      "name":               "CL-Schema1",
      "resourceType":      "CL-Schema",
      "mimeType":          "application/json"
      ...
      "previousVersionId: "12d5a5f6-e72d-11ec-8fea-0242ac120002",
      "nextVersionId:     null
    },
    "data": ...
  }
  ```

Step 2. Client send request for creating a new resource with a transaction MsgCreateResource

  ```jsonc
  MsgCreateResource for creating Resource2
  {
    "collectionId":  "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
    "id":             "f47e4790-1b4b-4186-8357-da6199665236",
    "name":           "CL-Schema1",
    "resourceType":  "CL-Schema",
    "mimeType":      "application/json"
    "data":           ...
  }
  ```

Step 3. After the transaction applying

```jsonc
Resource1
{
  "header": {
    "collectionId":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
    "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
    "name":               "CL-Schema1",
    "resourceType":      "CL-Schema",
    "mimeType":          "application/json"
    ...
    "previousVersionId: "12d5a5f6-e72d-11ec-8fea-0242ac120002",
    "nextVersionId:     "f47e4790-1b4b-4186-8357-da6199665236"  // Resource2.id
  },
  "data": ...
}
```

```jsonc
Resource2
{
  "header": {
    "collectionId":      "zF7rhDBfUt9d1gJPjx7s1JXfUY7oVWkY",
    "id":                 "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096",
    "name":               "CL-Schema1",
    "resourceType":      "CL-Schema",
    "mimeType":          "application/json"
    ...
    "previousVersionId: "9cc97dc8-ab3a-4a2e-a18a-13f5a54e9096", // Resource1.id
    "nextVersionId:     null
  },
  "data": ...
}
```

## Decision

### Assumptions

* Immutability:
  * Resources are immutable, so can't be updated/removed;
* Limitations
  * Resource size is now limited by maximum tx/block size;

### Future improvements

* Limitations
  * Introduce module level resource size limit that can be changed by voting

### 'Resources' module on ledger

A new module will be created: `resource`.

### Dependencies

* It will have `cheqd` module as a dependency.
  * Will be used for DIDs existence checks.
  * Will be used for authentication

## References

* [W3C Decentralized Identifiers (DIDs)](https://www.w3.org/TR/did-core/) specification
  * [DID Core Specification Test Suite](https://w3c.github.io/did-test-suite/)
* [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model/) official specification
* [Cosmos blockchain framework](https://cosmos.network/) official project website
  * [`cosmos-sdk`](https://github.com/cosmos/cosmos-sdk) GitHub repository ([documentation](https://docs.cosmos.network/))
* [Hyperledger Indy](https://wiki.hyperledger.org/display/indy) official project background on Hyperledger Foundation wiki
  * [`indy-node`](https://github.com/hyperledger/indy-node) GitHub repository: Server-side blockchain node for Indy ([documentation](https://hyperledger-indy.readthedocs.io/projects/node/en/latest/index.html))
  * [`indy-plenum`](https://github.com/hyperledger/indy-plenum) GitHub repository: Plenum Byzantine Fault Tolerant consensus protocol; used by `indy-node` ([documentation](https://hyperledger-indy.readthedocs.io/projects/plenum/en/latest/index.html))
  * [Indy DID method](https://hyperledger.github.io/indy-did-method/) (`did:indy`)
  * [Indy identity-domain transactions](https://github.com/hyperledger/indy-node/blob/master/docs/source/transactions.md)
* [Hyperledger Aries](https://wiki.hyperledger.org/display/ARIES/Hyperledger+Aries) official project background on Hyperledger Foundation wiki
  * [`aries`](https://github.com/hyperledger/aries) GitHub repository: Provides links to implementations in various programming languages
  * [`aries-rfcs`](https://github.com/hyperledger/aries-rfcs) GitHub repository: Contains Requests for Comment (RFCs) that define the Aries protocol behaviour
