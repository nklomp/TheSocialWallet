**_WARNING: This readme and code base is a copy of the Sphereon Github repository. The code as such will not be maintained in this repository. If you have a question, need
support, or a suggest go to the [Sphereon Github](https://github.com/Sphereon-OpenSource) please_**

# OID4VCI REST API

The Sphereon agent can expose a management REST API for 1 or more OpenID for Verifiable Credentials Issuance instances


# Full OpenAPI
The OpenAPI description of our OID4VCI agent can be found [on our Swaggerhub here](https://app.swaggerhub.com/apis/SphereonInt/OID4VCI)

There are 3 endpoint types (tags) of which only the `Backend` endpoints are important to create an integration into your solution. The other endpoints are used by wallets, which need to conform to the OID4VCI specification and thus should automatically be compatible with our implementation (provided that the current draft specification versions are supported by both the wallet and our issuer)


# Explanation
These are requests to Issue credentials using the OpenID for Verifiable Credentials Issuance specification
The first two GET requests are merely there for information purposes, so you can see what type of metadata is being hosted by an OpenID for Verifiable Credential Issuer and its OAuth2 Authorization Server
You start with creating a credential offer URI, supplying it a pre-authorized-code from your backend. We suggest to make these values random.
Then you start polling the issuance status endpoint, until the credential is issued or an error has occurred.
Lastly you can delete the credential
