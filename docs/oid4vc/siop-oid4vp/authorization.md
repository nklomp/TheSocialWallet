---
title: Authorization
---
## AuthorizationRequest class

In the previous chapter we have seen the highlevel OP and RP classes. These classes use the Auth Request and
Response objects explained in this chapter and the next chapter. If you want you can do most interactions using these
classes at a lower level. This however means you will not get automatic resolution of values passed by reference like
for instance request and registration data.

### createURI

Create a signed URL encoded URI with a signed SIOP Auth Request

#### Data Interface

```typescript
interface AuthorizationRequestURI extends SIOPURI {
    jwt?: string;                                    // The JWT when requestBy was set to mode Reference, undefined if the mode is Value
    requestOpts: AuthorizationRequestOpts;          // The supplied request opts as passed in to the method
    requestPayload: AuthorizationRequestPayload;    // The json payload that ends up signed in the JWT
}

export type SIOPURI = {
    encodedUri: string;                  // The encode JWT as URI
    encodingFormat: UrlEncodingFormat;   // The encoding format used
};

// https://openid.net/specs/openid-connect-self-issued-v2-1_0.html#section-8
export interface AuthorizationRequestOpts {
    authorizationEndpoint?: string;
    redirectUri: string;                // The redirect URI
    requestBy: ObjectBy;                // Whether the request is returned by value in the URI or retrieved by reference at the provided URL
    signature: InternalSignature | ExternalSignature | NoSignature; // Whether no withSignature is being used, internal (access to private key), or external (hosted using authentication)
    checkLinkedDomain?: CheckLinkedDomain; // determines how we'll handle the linked domains for this RP
    responseMode?: ResponseMode;        // How the URI should be returned. This is not being used by the library itself, allows an implementor to make a decision
    responseContext?: ResponseContext;  // Defines the context of these opts. Either RP side or OP side
    responseTypesSupported?: ResponseType[];
    claims?: ClaimOpts;                 // The claims, uncluding presentation definitions
    registration: RequestRegistrationOpts; // Registration metadata options
    nonce?: string;                     // An optional nonce, will be generated if not provided
    state?: string;                     // An optional state, will be generated if not provided
    scopesSupported?: Scope[];
    subjectTypesSupported?: SubjectType[];
    requestObjectSigningAlgValuesSupported?: SigningAlgo[];
    revocationVerificationCallback?: RevocationVerificationCallback;
    // slint-disable-next-line @typescript-eslint/no-explicit-any
    // [x: string]: any;
}

static async createURI(opts: SIOP.AuthorizationRequestOpts): Promise<SIOP.AuthorizationRequestURI>
```

#### Usage

```typescript
const EXAMPLE_REDIRECT_URL = 'https://acme.com/hello'
const EXAMPLE_REFERENCE_URL = 'https://rp.acme.com/siop/jwts'
const HEX_KEY = 'f857544a9d1097e242ff0b287a7e6e90f19cf973efe2317f2a4678739664420f'
const DID = 'did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0'
const KID = 'did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0#keys-1'

const opts: AuthorizationRequestOpts = {
  checkLinkedDomain: CheckLinkedDomain.NEVER,
  requestObjectSigningAlgValuesSupported: [SigningAlgo.EDDSA, SigningAlgo.ES256],
  redirectUri: EXAMPLE_REDIRECT_URL,
  requestBy: {
    type: PassBy.VALUE,
  },
  signature: {
    hexPrivateKey: HEX_KEY,
    did: DID,
    kid: KID,
  },
  registration: {
    idTokenSigningAlgValuesSupported: [SigningAlgo.EDDSA, SigningAlgo.ES256],
    requestObjectSigningAlgValuesSupported: [SigningAlgo.EDDSA, SigningAlgo.ES256],
    responseTypesSupported: [ResponseType.ID_TOKEN],
    scopesSupported: [Scope.OPENID_DIDAUTHN, Scope.OPENID],
    subjectSyntaxTypesSupported: ['did:ethr:', SubjectIdentifierType.DID],
    subjectTypesSupported: [SubjectType.PAIRWISE],
    vpFormatsSupported: {
      ldp_vc: {
        proof_type: [IProofType.EcdsaSecp256k1Signature2019, IProofType.EcdsaSecp256k1Signature2019],
      },
    },
    registrationBy: {
      type: PassBy.VALUE,
    },
  },
}

AuthorizationRequest.createURI(opts).then((uri) => console.log(uri.encodedUri))

// Output:
// openid://
//      ?response_type=id_token
//      &scope=openid
//      &client_id=did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0
//      &redirect_uri=https://acme.com/hello&iss=did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0
//      &response_mode=post
//      &response_context=rp
//      &nonce=HxhBU9jBRVP51Z6J0eQ5AxeKoWK9ChApWRrumIqnixc
//      &state=cbde3cdc5389f3be94063be3
//      &registration={
//          "id_token_signing_alg_values_supported":["EdDSA","ES256"],
//          "request_object_signing_alg_values_supported":["EdDSA","ES256"],
//          "response_types_supported":["id_token"],
//          "scopes_supported":["openid did_authn","openid"],
//          "subject_types_supported":["pairwise"],
//          "subject_syntax_types_supported":["did:ethr:","did"],
//          "vp_formats":{
//              "ldp_vc":{
//                  "proof_type":["EcdsaSecp256k1Signature2019","EcdsaSecp256k1Signature2019"]
//              }
//          }
//      }
//      &request=eyJhbGciOiJFUzI1NksiLCJraWQiOiJkaWQ6ZXRocjoweDAxMDZhMmU5ODViMUUxRGU5QjVkZGI0YUY2ZEM5ZTkyOEY0ZTk5RDAja2V5cy0xIiwidHlwIjoiSldUIn0.eyJpYXQiOjE2NjQ0Mzk3MzMsImV4cCI6MTY2NDQ0MDMzMywicmVzcG9uc2VfdHlwZSI6ImlkX3Rva2VuIiwic2NvcGUiOiJvcGVuaWQiLCJjbGllbnRfaWQiOiJkaWQ6ZXRocjoweDAxMDZhMmU5ODViMUUxRGU5QjVkZGI0YUY2ZEM5ZTkyOEY0ZTk5RDAiLCJyZWRpcmVjdF91cmkiOiJodHRwczovL2FjbWUuY29tL2hlbGxvIiwiaXNzIjoiZGlkOmV0aHI6MHgwMTA2YTJlOTg1YjFFMURlOUI1ZGRiNGFGNmRDOWU5MjhGNGU5OUQwIiwicmVzcG9uc2VfbW9kZSI6InBvc3QiLCJyZXNwb25zZV9jb250ZXh0IjoicnAiLCJub25jZSI6Ikh4aEJVOWpCUlZQNTFaNkowZVE1QXhlS29XSzlDaEFwV1JydW1JcW5peGMiLCJzdGF0ZSI6ImNiZGUzY2RjNTM4OWYzYmU5NDA2M2JlMyIsInJlZ2lzdHJhdGlvbiI6eyJpZF90b2tlbl9zaWduaW5nX2FsZ192YWx1ZXNfc3VwcG9ydGVkIjpbIkVkRFNBIiwiRVMyNTYiXSwicmVxdWVzdF9vYmplY3Rfc2lnbmluZ19hbGdfdmFsdWVzX3N1cHBvcnRlZCI6WyJFZERTQSIsIkVTMjU2Il0sInJlc3BvbnNlX3R5cGVzX3N1cHBvcnRlZCI6WyJpZF90b2tlbiJdLCJzY29wZXNfc3VwcG9ydGVkIjpbIm9wZW5pZCBkaWRfYXV0aG4iLCJvcGVuaWQiXSwic3ViamVjdF90eXBlc19zdXBwb3J0ZWQiOlsicGFpcndpc2UiXSwic3ViamVjdF9zeW50YXhfdHlwZXNfc3VwcG9ydGVkIjpbImRpZDpldGhyOiIsImRpZCJdLCJ2cF9mb3JtYXRzIjp7ImxkcF92YyI6eyJwcm9vZl90eXBlIjpbIkVjZHNhU2VjcDI1NmsxU2lnbmF0dXJlMjAxOSIsIkVjZHNhU2VjcDI1NmsxU2lnbmF0dXJlMjAxOSJdfX19fQ.owSdQP3ZfOyHryCIO86zB5qenzd5l2AUcEZhA3TvlUWNDJyhhzIgZmBgzV4OMilczr2AJss5HGqxHPmBRTaHcQ
```

### verifyJWT

Verifies a SIOP Auth Request JWT. Throws an error if the verifation fails. Returns the verified JWT and
metadata if the verification succeeds

#### Data Interface

```typescript
export interface VerifiedAuthorizationRequestWithJWT extends VerifiedJWT {
    payload: AuthorizationRequestPayload;       // The unsigned Auth Request payload
    presentationDefinitions?: PresentationDefinitionWithLocation[]; // The optional presentation definition objects that the RP requests
    verifyOpts: VerifyAuthorizationRequestOpts; // The verification options for the Auth Request
}

export interface VerifiedJWT {
    payload: Partial<JWTPayload>;            // The JWT payload
    didResolutionResult: DIDResolutionResult;// DID resolution result including DID document
    issuer: string;                          // The issuer (did) of the JWT
    signer: VerificationMethod;              // The matching verification method from the DID that was used to sign
    jwt: string;                             // The JWT
}

export interface VerifyAuthorizationRequestOpts {
    verification: Verification
    nonce?: string; // If provided the nonce in the request needs to match
    verifyCallback?: VerifyCallback;
}

export interface DIDResolutionResult {
    didResolutionMetadata: DIDResolutionMetadata // Did resolver metadata
    didDocument: DIDDocument                     // The DID document
    didDocumentMetadata: DIDDocumentMetadata     // DID document metadata
}

export interface DIDDocument {              // Standard DID Document, see DID spec for explanation
    '@context'?: 'https://www.w3.org/ns/did/v1' | string | string[]
    id: string
    alsoKnownAs?: string[]
    controller?: string | string[]
    verificationMethod?: VerificationMethod[]
    authentication?: (string | VerificationMethod)[]
    assertionMethod?: (string | VerificationMethod)[]
    keyAgreement?: (string | VerificationMethod)[]
    capabilityInvocation?: (string | VerificationMethod)[]
    capabilityDelegation?: (string | VerificationMethod)[]
    service?: ServiceEndpoint[]
}

static async verifyJWT(jwt:string, opts: SIOP.VerifyAuthorizationRequestOpts): Promise<SIOP.VerifiedAuthorizationRequestWithJWT>
```

#### Usage

```typescript
const verifyOpts: VerifyAuthorizationRequestOpts = {
  verification: {
    resolveOpts: {
      subjectSyntaxTypesSupported: ['did:ethr'],
    },
  },
}
const jwt = 'ey..........' // JWT created by RP
AuthorizationRequest.verifyJWT(jwt).then((req) => {
  console.log(`issuer: ${req.issuer}`)
  console.log(JSON.stringify(req.signer))
})
// issuer: "did:ethr:0x56C4b92D4a6083Fcee825893A29023cDdfff5c66"
// "signer": {
//      "id": "did:ethr:0x56C4b92D4a6083Fcee825893A29023cDdfff5c66#controller",
//      "type": "EcdsaSecp256k1RecoveryMethod2020",
//      "controller": "did:ethr:0x56C4b92D4a6083Fcee825893A29023cDdfff5c66",
//      "blockchainAccountId": "0x56C4b92D4a6083Fcee825893A29023cDdfff5c66@eip155:1"
// }
```

## AuthorizationResponse class

### createJwtFromRequestJWT

Creates an AuthorizationResponse object from the OP side, using the AuthorizationRequest of the RP and its
verification as input together with settings from the OP. The Auth Response contains the ID token as well as
optional Verifiable Presentations conforming to the Submission Requirements sent by the RP.

#### Data interface

```typescript
export interface AuthorizationResponseOpts {
    redirectUri?: string; // It's typically comes from the request opts as a measure to prevent hijacking.
    registration: ResponseRegistrationOpts;      // Registration options
    checkLinkedDomain?: CheckLinkedDomain; // When the link domain should be checked
    presentationVerificationCallback?: PresentationVerificationCallback; // Callback function to verify the presentations
    signature: InternalSignature | ExternalSignature; // Using an internal/private key withSignature, or hosted withSignature
    nonce?: string;                              // Allows to override the nonce, otherwise the nonce of the request will be used
    state?: string;                              // Allows to override the state, otherwise the state of the request will be used
    responseMode?: ResponseMode;                 // Response mode should be form in case a mobile device is being used together with a browser
    did: string;                                 // The DID of the OP
    vp?: VerifiablePresentationResponseOpts[];   // Verifiable Presentations with location and format
    expiresIn?: number;                          // Expiration
}

export interface VerifiablePresentationResponseOpts extends VerifiablePresentationPayload {
    location: PresentationLocation;
}

export enum PresentationLocation {
    VP_TOKEN = 'vp_token', // VP will be the toplevel vp_token
    ID_TOKEN = 'id_token', // VP will be part of the id_token in the verifiable_presentations location
}

export interface VerifyAuthorizationRequestOpts {
    verification: Verification
    nonce?: string;                                              // If provided the nonce in the request needs to match
    verifyCallback?: VerifyCallback                              // Callback function to verify the domain linkage credential
}

export interface AuthorizationResponsePayload extends JWTPayload {
    iss: ResponseIss.SELF_ISSUED_V2 | string;                      // The SIOP V2 spec mentions this is required
    sub: string;                                                   // did (or thumbprint of sub_jwk key when type is jkt)
    sub_jwk?: JWK;                                                  // JWK containing DID key if subtype is did, or thumbprint if it is JKT
    aud: string;                                                   // redirect_uri from request
    exp: number;                                                  // expiration time
    iat: number;                                                  // issued at
    state: string;                                                 // The state which should match the AuthRequest state
    nonce: string;                                                 // The nonce which should match the AuthRequest nonce
    did: string;                                                   // The DID of the OP
    registration?: DiscoveryMetadataPayload;                       // The registration metadata from the OP
    registration_uri?: string;                                     // The URI of the registration metadata if it is returned by reference/URL
    verifiable_presentations?: VerifiablePresentationPayload[];    // Verifiable Presentations
    vp_token?: VerifiablePresentationPayload;
}

export interface AuthorizationResponseWithJWT {
    jwt: string;                                 // The signed Response JWT
    nonce: string;                               // The nonce which should match the nonce from the request
    state: string;                               // The state which should match the state from the request
    payload: AuthorizationResponsePayload;      // The unsigned payload object
    verifyOpts?: VerifyAuthorizationRequestOpts;// The Auth Request verification parameters that were used
    responseOpts: AuthorizationResponseOpts;    // The Auth Response options used during generation of the Response
}

static async createJWTFromRequestJWT(requestJwt: string, responseOpts: SIOP.AuthorizationResponseOpts, verifyOpts: SIOP.VerifyAuthorizationRequestOpts): Promise<SIOP.AuthorizationResponseWithJWT>
```

#### Usage

```typescript
const responseOpts: AuthorizationResponseOpts = {
  checkLinkedDomain: CheckLinkedDomain.NEVER,
  redirectUri: 'https://acme.com/hello',
  registration: {
    authorizationEndpoint: 'www.myauthorizationendpoint.com',
    idTokenSigningAlgValuesSupported: [SigningAlgo.EDDSA, SigningAlgo.ES256],
    issuer: ResponseIss.SELF_ISSUED_V2,
    responseTypesSupported: [ResponseType.ID_TOKEN],
    subjectSyntaxTypesSupported: ['did:ethr:'],
    vpFormats: {
      ldp_vc: {
        proof_type: [IProofType.EcdsaSecp256k1Signature2019, IProofType.EcdsaSecp256k1Signature2019],
      },
    },
    registrationBy: {
      type: PassBy.REFERENCE,
      referenceUri: 'https://rp.acme.com/siop/jwts',
    },
  },
  signature: {
    did: 'did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0',
    hexPrivateKey: 'f857544a9d1097e242ff0b287a7e6e90f19cf973efe2317f2a4678739664420f',
    kid: 'did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0#controller',
  },
  did: 'did:ethr:0x0106a2e985b1E1De9B5ddb4aF6dC9e928F4e99D0',
  responseMode: ResponseMode.POST,
}
```

#### Usage

```typescript
const EXAMPLE_REDIRECT_URL = 'https://acme.com/hello'
const NONCE = '5c1d29c1-cf7d-4e14-9305-9db46d8c1916'
const verifyOpts: VerifyAuthorizationResponseOpts = {
  audience: 'https://rp.acme.com/siop/jwts',
  nonce: NONCE,
}

verifyJWT('ey......', verifyOpts).then((jwt) => {
  console.log(`nonce: ${jwt.payload.nonce}`)
  // output: nonce: 5c1d29c1-cf7d-4e14-9305-9db46d8c1916
})
```

### Verify Revocation

Verifies whether a verifiable credential contained verifiable presentation is revoked

#### Data Interface

```typescript
export type RevocationVerificationCallback = (
  vc: W3CVerifiableCredential, // The Verifiable Credential to be checked
  type: VerifiableCredentialTypeFormat, // Whether it is a LDP or JWT Verifiable Credential
) => Promise<IRevocationVerificationStatus>
```

```typescript
export interface IRevocationVerificationStatus {
  status: RevocationStatus // Valid or invalid
  error?: string
}
```

```typescript
export enum RevocationVerification {
  NEVER = 'never', // We don't want to verify revocation
  IF_PRESENT = 'if_present', // If credentialStatus is present, did-auth-siop will verify revocation. If present and not valid an exception is thrown
  ALWAYS = 'always', // We'll always check the revocation, if not present or not valid, throws an exception
}
```

#### Usage

```typescript
  const verifyRevocation = async (
    vc: W3CVerifiableCredential,
    type: VerifiableCredentialTypeFormat
):Promise<IRevocationVerificationStatus> => {
  // Logic to verify the credential status
  ...
  return { status, error }
};
```

```typescript
import { verifyRevocation } from './Revocation'

const rp = RP.builder()
  .withRevocationVerification(RevocationVerification.ALWAYS)
  .withRevocationVerificationCallback((vc, type) => verifyRevocation(vc, type))
```

### Verify Presentation Callback

The callback function to verify the verifiable presentation

#### Data interface

```typescript
export type PresentationVerificationCallback = (args: IVerifiablePresentation) => Promise<PresentationVerificationResult>
```

```typescript
export type IVerifiablePresentation = IPresentation & IHasProof
```

```typescript
export type PresentationVerificationResult = { verified: boolean }
```

#### Usage

JsonLD

```typescript
import { PresentationVerificationResult } from './SIOP.types'

const verifyPresentation = async (vp: IVerifiablePresentation): Promise<PresentationVerificationResult> => {
  const keyPair = await Ed25519VerificationKey2020.from(VC_KEY_PAIR)
  const suite = new Ed25519Signature2020({ key: keyPair })
  suite.verificationMethod = keyPair.id
  // If the credentials are not verified individually by the library,
  // it needs to be implemented. In this example, the library does it.
  const { verified } = await vc.verify({ presentation: vp, suite, challenge: 'challenge', documentLoader: new DocumentLoader().getLoader() })
  return Promise.resolve({ verified })
}
```

or

JWT

```typescript
import { IVerifiablePresentation } from '@sphereon/ssi-types'

const verifyPresentation = async (vp: IVerifiablePresentation): Promise<PresentationVerificationResult> => {
  // If the credentials are not verified individually by the library,
  // it needs to be implemented. In this example, the library does it.
  await verifyCredentialJWT(jwtVc, getResolver({ subjectSyntaxTypesSupported: ['did:key:'] }))
  return Promise.resolve({ verified: true })
}
```

```typescript
const rp = RP.builder()
      .withPresentationVerification((args) => verifyPresentation(args))
      ...
```
