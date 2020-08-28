# Schema Registry DID Method Specification v0.1
## About

This Document describes the specification of a DID method used to identify and address schema definitions in a schema registry. It provides information about the method-sepcific DID syntax and exemplary CRUD operations as well as security and privacy considerations.

This specification conforms to the requirements in the [DID specification](https://w3c.github.io/did-core/) currently published by the W3C Credentials Community Group. For more information please refer to the link above. 


## Abstract

The Schema Registry DID Method aims to provide unique and universal identification for schemas in multiple formats hosted on multiple storage mechanisms or networks.

This first version will focus on [JSON Schema](https://json-schema.org/) and [XML Schema Definition/XSD](https://www.w3.org/TR/xmlschema11-1/) schemas stored using the IPFS (InterPlanetary File System) protocol, but the DID method and concept is open for contributions and extensions regarding further schema formats and storage/network options.


## Specification
### Method-specific DID Syntax
The namestring that identifies this DID method is: ```schema```

The according ABNF Scheme is defined as follows: 
````
schema-did = "did:schema:" storage-network [ ":" schema-type ] ":" schema-hash
storage-network = "public-ipfs" / "evan-ipfs"
schema-type = "json-schema" / "xsd"
schema-hash = 1* hash-char
hash-char = a-z / A-Z / 0-9 / "." / "-" / "_"
````

#### Syntax Explanation

The DID comprises of multiple elements separated by colons (`:`):

1. The general DID method prefix `did:schema`.
1. A mandatory identifier for the storage network, chosen from a list of supported storage mechanisms and networks. Currently, the public IPFS network as well as Evan IPFS (core network) are supported.
1. An optional identifier for the type or format of the schema, chosen from a list of supported types. Currently, the JSON Schema and XSD formats are supported.
1. The mandatory schema-hash identifying the schema file in the given storage network. Represented as a string.

#### Example of a Schema DID

The following DID represents an XSD definition stored on the public IPFS network with the IPFS hash `QmUQAxKQ5sbWWrcBZzwkThktfUGZvuPQyTrqMzb3mZnLE5`

````
did:schema:public-ipfs:xsd:QmUQAxKQ5sbWWrcBZzwkThktfUGZvuPQyTrqMzb3mZnLE5
````

The same schema could be addressed without the type, however in this case the caller would need to determine the format of the schema themselves:

````
did:schema:public-ipfs:QmUQAxKQ5sbWWrcBZzwkThktfUGZvuPQyTrqMzb3mZnLE5
````

### CRUD Operations

#### Reference Implementation

The CRUD operations described in the following were implemented in a TypeScript library available in GitHub repository https://github.com/51nodes/schema-registry-did-crud and as an npm package https://www.npmjs.com/package/@51nodes/decentralized-schema-registry.

There is also a (universal resolver compliant) DID Resolver implementation based on Nest.js and the reference library available in GitHub repository https://github.com/51nodes/schema-registry-did-resolver.

#### Create

A schema DID is implicitly created by storing a respective schema file to one of the supported storage networks and making sure it remains stored on a permanent basis. For example, the provider of a JSON schema could store the JSON string containing the full schema definition as a file in the public IPFS network, receive the IPFS hash of this file and set up a pinning service (see e.g. https://docs.ipfs.io/concepts/persistence/) for this hash that makes sure the file is not removed from the storage.

The provider can then assemble the schema DID from the combination of storage network, schema type and schema hash as described in the previous chapter. It is assumed that the provider stores this information for later usage and distribution, since there is no registry or catalog of existing schema DIDs.

Creating a schema can be performed with the reference library using the function `registerSchema`.

#### Read/Resolve

Essentially, a schema DID resolves to the schema file stored in the underlying storage network. So the resolution of the schema DID consists of

* verifying the existence and availability of the schema file corresponding to the schema hash in the storage network,
* optionally validating the schema type (if present as a type element in the DID),
* and dynamically assembling a DID document including a service endpoint for the download of the schema file.

If the schema hash does not exist or the type validation fails, the resolver must treat the DID as nonexistent.

A DID document of the DID `did:schema:evan-ipfs:xsd:QmUQAxKQ5sbWWrcBZzwkThktfUGZvuPQyTrqMzb3mZnLE5` would look like the following:

````json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:schema:evan-ipfs:xsd:QmUQAxKQ5sbWWrcBZzwkThktfUGZvuPQyTrqMzb3mZnLE5",
  "service": [
    {
      "id": "did:schema:evan-ipfs:xsd:QmUQAxKQ5sbWWrcBZzwkThktfUGZvuPQyTrqMzb3mZnLE5#get",
      "type": "GetSchemaService",
      "serviceEndpoint": "https://ipfs.evan.network/ipfs/QmUQAxKQ5sbWWrcBZzwkThktfUGZvuPQyTrqMzb3mZnLE5"
    }
  ]
}
````

The service endpoint with the fragment `#get` provides a URL to download the schema from the Evan IPFS.

Alternatively, the schema could be loaded using the reference library function `getSchema(did)`.

#### Update

A schema DID is implicitly updated by updating the corresponding file in the underlying storage network. It is assumed, though, that the schema hash provides a reference to an immutable set of data. This is the case in the IPFS networks where any change in a schema file would result in a new hash and thus a new DID. That's why the reference library currently does not offer a distinct update operation and it is required to call the `registerSchema` operation again if the content of a schema is updated.

#### Delete

A schema DID is implicitly deleted by deleting the corresponding file from the underlying storage network. On IPFS networks, there is however no guaranteed way that a resource is actually deleted, since it may still be stored on some of the distributed nodes.

Since the explicit deletion of a schema is not considered a mandatory requirement, the delete operation is currently not supported in the reference library.

## Security Considerations

The basic purpose of this DID method is to store, identify and resolve schema definitions in a decentralized way. This is not considered to be a security critical mechanism. However, along the process of storing and accessing data, users of this DID method must take the usual common sense precautions to protect their accounts (e.g. to be used for pinning in IPFS networks) and do not expose any credentials or other security sensitive information in schema content.

In addition, each Storage network has its own security aspects:
### Public IPFS 
One of the security considerations is evading. A node can easily generate a completely new [node identity](https://medium.com/textileio/how-ipfs-peer-nodes-identify-each-other-on-the-distributed-web-8b5b6476aa5e) making it impossible to exclude dishonest nodes, which could serve content that does not match the requested [CID](https://docs.ipfs.io/concepts/content-addressing/).

Another consideration is using public or private [IPFS gateways](https://docs.ipfs.io/concepts/ipfs-gateway/#overview). A gateway can be malicious or have vulnerabilities that would allow a [man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) to intercept and manipulate communication between the client and the gateway.

Therefore, recipients should verify that the received content matches the requested CID. In case of adding a new schema to IPFS through a gateway, the resulting CID should also be verified to match the added content.

A [DDoS attack](https://de.wikipedia.org/wiki/Denial_of_Service) on the public IPFS network is difficult because of its distributed and peer-to-peer architecture. The content is spread over several nodes and can be accessed through the distributed storage network.

### evan.network IPFS
Since the IPFS of evan.network is a permissioned IPFS cluster accessed through a centrally managed endpoint, evading or man-in-the-middle-attacks should not be a security concern. However, the permissioned network is currently small and a DDos attack on the network is possible.

## Privacy Considerations

Due to the fact that schemas are technical definitions of data structures that should never involve any personal information, privacy should not be much of a concern. In all supported storage networks, the data is stored anonymously and can not be traced back to any specific individual. It is the responsibility of the party providing the schema to make sure that no privacy sensitive data is contained in the schema content.

Each storage network also has its own privacy considerations:
### Public IPFS
With a slight modification to the IPFS node, clients requesting content can be tracked and information like the IP address, node id or region could be collected and correlated. That happens because when a user running a node requests content from the network, each connected node receives a message asking for that content. Even if the user is not running an own node but uses a gateway, the provider of the gateway could also track the user's request and content.

### evan.Network IPFS
Because evan.network IPFS is permissioned, accessing the content for non participating users is only possible through gateways which are provided by evan.network or known node operators. Therefore, the privacy of user request data is the responsibility of the gateway provider.

