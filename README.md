# Schema Registry DID Method Specification v0.01
## About

This Document describes the specification of a schema DID. It provides information about the method-sepcific did syntax and exemplary CRUD operations aswell as security and privacy considerations.

This specification conforms to the requirements in the [DID specification](https://w3c.github.io/did-core/) currently published by the W3C Credentials Community Group. For more information please refer to the link above. 

This DID Method will be registered in the [DID Method Registry](https://w3c-ccg.github.io/did-method-registry/#the-registry).

## Abstract

The Schema Registry DID Method aims to provide unique and universal identification for schemas in multiple formats hosted on multiple networks.

This first version will only have support for JSON/XSD schemas hosted on IPFS networks, but feel free to contribute and extend either the supported schema formats or the supported storage/network options.

## Status of This Document

Currently this document is not a W3C Standard. It is a draft and will be updated in future.


## Specification
### Method-specific DID Syntax
The namestring that identifies this DID method is: ```schema```

The according ABNF Scheme is defined as follows: 
````
schmema-did = "did:schema:" ipfs-network ":" type-hint ":" schema-type ":" ipfs-hash 
ipfs-network = 
type-hint =
schema-type = 
ipfs-hash =
````

Example of a Schema DID - TODO: place example here

TODO: place syntax explanation here

### CRUD Operations
#### Create
#### Read/Resolve
#### Update
#### Delete
## Security Considerations
## Privacy Considerations