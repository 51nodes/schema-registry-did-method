# Schema Registry DID Method Specification v0.01
## About

This Document describes the specification of a DID method used to identify and address schema definitions in a schema registry. It provides information about the method-sepcific DID syntax and exemplary CRUD operations as well as security and privacy considerations.

This specification conforms to the requirements in the [DID specification](https://w3c.github.io/did-core/) currently published by the W3C Credentials Community Group. For more information please refer to the link above. 

This DID Method will be registered in the [DID Method Registry](https://w3c-ccg.github.io/did-method-registry/#the-registry).

## Abstract

The Schema Registry DID Method aims to provide unique and universal identification for schemas in multiple formats hosted on multiple storage mechanisms or networks.

This first version will focus on [JSON Schema](https://json-schema.org/) and [XML Schema Definition/XSD](https://www.w3.org/TR/xmlschema11-1/) schemas stored using the IPFS (InterPlanetary File System) protocol, but the DID method and concept is open for contributions and extensions regarding further schema formats and storage/network options.

## Status of This Document

Currently this document is not a W3C Standard. It is a draft and will be updated in future.


## Specification
### Method-specific DID Syntax
The namestring that identifies this DID method is: ```schema```

The according ABNF Scheme is defined as follows: 
````
schema-did = "did:schema:" storage-network [ ":type-hint=" schema-type ] ":" schema-hash 
storage-network = "ipfs-public" / "ipfs-evan" / "ipfs-evan-test"
schema-type = "json-schema" / "xsd"
schema-hash = "0x"n*HEXDIG
````

#### Syntax Explanation

The DID comprises of multiple elements separated by colons (`:`):

1. The general DID method prefix `did:schema`.
1. A mandatory identifier for the storage network, chosen from a list of supported storage mechanisms and networks. Currently, the public IPFS network as well as Evan IPFS (core and testcore) are supported.
1. An optional identifier for the type or format of the schema, chosen from a list of supported types. Currently, the JSON Schema and XSD formats are supported.
1. The mandatory hash identifying the schema file in the given storage network. Represented as a hexadecimal / base16 string with the prefix `0x`.

#### Example of a Schema DID

The following DID represents a JSON Schema definition stored on the public IPFS network with the hash `0x64EC88CA00B268E5BA1A35678A1B5316D212F4F366B2477232534A8AECA37F3C`:

````
did:schema:ipfs-public:type-hint=json-schema:0x64EC88CA00B268E5BA1A35678A1B5316D212F4F366B2477232534A8AECA37F3C
````

The same schema could be addressed without the type hint, however in this case the caller would need to determine the format of the schema themselves:

````
did:schema:ipfs-public:0x64EC88CA00B268E5BA1A35678A1B5316D212F4F366B2477232534A8AECA37F3C
````

### CRUD Operations
#### Create

A schema DID is implicitly created by storing a respective schema file to one of the supported storage networks and making sure it remains stored on a permanent basis. For example, the provider of a JSON schema could store the JSON string containing the full schema definition as a file in the public IPFS network, receive the IPFS hash of this file and set up a pinning service (see e.g. https://docs.ipfs.io/concepts/persistence/) for this hash that makes sure the file is not removed from the storage.

The provider can then assemble the schema DID from the combination of storage network, schema type and schema hash as described in the previous chapter. It is assumed that the provider stores this information for later usage and distribution, since there is no registry or catalog of existing schema DIDs.

#### Read/Resolve

Essentially, a schema DID resolves to the schema file stored in the underlying storage network. So the resolution of the schema DID consists of

* verifying the existence and availability of the schema file corresponding to the hash in the storage network,
* optionally validating the schema type (if present as a type hint in the DID),
* and dynamically assembling a DID document including a service endpoint for the download of the schema file.

If the hash does not exist or the type validation fails, the resolver must treat the DID as nonexistent.

TODO add example DID document

TODO mentiond and link reference implementation

TODO how to download schema directly e.g. via DID path or DID fragment? 

#### Update

Since the schema definition is supposed to be immutable and any change in a schema file would result in a new hash and thus a new DID, the update operation is not required.

#### Delete

A schema DID is implicitly deleted by deleting the corresponding file from the underlying storage network.

## Security Considerations

## Privacy Considerations

Due to the fact that schemas are technical definitions of data structures that should never involve any personal information, privacy should not be much of a concern. In all supported storage networks, the data is stored anonymously and can not be traced back to any specific individual. It is the responsibility of the party providing the schema to make sure that no privacy sensitive data is contained in the schema content.
