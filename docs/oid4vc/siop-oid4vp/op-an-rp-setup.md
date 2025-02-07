---
title: OP and RP setup and interactions
description: 'This chapter is a walk-through for using the library using the high-level OP and RP classes. To keep
it simple, the examples work without hosting partial request/response related objects using HTTP endpoints. They are passed by value, inlined in the respective payloads versus passed by reference.'
---

---

<Tip>
The examples use Ethereum (ethr) DIDs, but these could be other DIDs as well. The creation of DIDs is out of scope. We
provide an [ethereum DID example](ethr-dids-testnet.md), if you want to test it yourself without having DIDs currently.
You could also use the actual example keys and DIDs, as they are valid Ethr Ropsten testnet keys.
</Tip>

### Relying Party and SIOP should have keys and DIDs

This library does not provide methods for signing and verifying tokens and authorization requests. Verification and Signing functionality must be externally provided.

### Setting up the Relying Party (RP)

The Relying Party, typically a web app, but can also be something else, like a mobile app.
The consumer of this library must provide means for creating and verifying JWT to the RP class instance.
This library provides adapters for creating and verifying did, jwk, and x5c protected JWT`s.

Both the actual JWT request and the
registration metadata will be sent as part of the Auth Request since we pass them by value instead of by reference where
we would have to host the data at the reference URL. The redirect URL means that the OP will need to deliver the
auth response at the URL specified by the RP. We also populated the RP with a `PresentationDefinition` claim,
meaning we expect the OP to send in a Verifiable Presentation that matches our definition.
You can pass where you expect this presentation_definition to end up via the required `location` property.
This is either a top-level vp_token or it becomes part of the id_token.

```typescript
// The relying party (web) private key and DID and DID key (public key)

const EXAMPLE_REDIRECT_URL = 'https://acme.com/hello'

function verifyJwtCallback(): VerifyJwtCallback {
  return async (jwtVerifier, jwt) => {
    if (jwtVerifier.method === 'did') {
      // verify didJwt's
    } else if (jwtVerifier.method === 'x5c') {
      // verify x5c certificate protected jwt's
    } else if (jwtVerifier.method === 'jwk') {
      // verify jwk certificate protected jwt's
    } else if (jwtVerifier.method === 'custom') {
      // Only called if based on the jwt the verification method could not be determined
      throw new Error(`Unsupported JWT verifier method ${jwtIssuer.method}`)
    }
  }
}

function createJwtCallback(): CreateJwtCallback {
  return async (jwtIssuer, jwt) => {
    if (jwtIssuer.method === 'did') {
      // create didJwt
    } else if (jwtIssuer.method === 'x5c') {
      // create x5c certificate protected jwt
    } else if (jwtIssuer.method === 'jwk') {
      // create a jwk certificate protected jwt
    } else if (jwtIssuer.method === 'custom') {
      // Only called if no or a Custom jwtIssuer was passed to the respective methods
      throw new Error(`Unsupported JWT issuer method ${jwtIssuer.method}`)
    }
  }
}

const rp = RP.builder()
  .redirect(EXAMPLE_REDIRECT_URL)
  .requestBy(PassBy.VALUE)
  .withPresentationVerification(presentationVerificationCallback)
  .withCreateJwtCallback(createJwtCallback)
  .withVerifyJwtCallback(verifyJwtCallback)
  .withRevocationVerification(RevocationVerification.NEVER)
  .withClientMetadata({
    idTokenSigningAlgValuesSupported: [SigningAlgo.EDDSA],
    requestObjectSigningAlgValuesSupported: [SigningAlgo.EDDSA, SigningAlgo.ES256],
    responseTypesSupported: [ResponseType.ID_TOKEN],
    vpFormatsSupported: { jwt_vc: { alg: [SigningAlgo.EDDSA] } },
    scopesSupported: [Scope.OPENID_DIDAUTHN, Scope.OPENID],
    subjectTypesSupported: [SubjectType.PAIRWISE],
    subjectSyntaxTypesSupported: ['did', 'did:ethr'],
    passBY: PassBy.VALUE,
  })
  .addPresentationDefinitionClaim({
    definition: {
      input_descriptors: [
        {
          schema: [
            {
              uri: 'https://did.itsourweb.org:3000/smartcredential/Ontario-Health-Insurance-Plan',
            },
          ],
        },
      ],
    },
    location: PresentationLocation.VP_TOKEN, // Toplevel vp_token response expected. This also can be ID_TOKEN
  })
  .build()
```

### OpenID Provider (OP)

The OP, typically a useragent together with a mobile phone in a cross device flow is accessing a protected resource at the RP, or needs to sent
in Verifiable Presentations. The consumer of the library must provide means for creating and verifying JWT to the OP class instance.
This library provides adapters for creating and verifying did, jwk, and x5c protected JWT`s.

```typescript
const op = OP.builder()
  .withExpiresIn(6000)
  .addDidMethod('ethr')
  .withCreateJwtCallback(createJwtCallback)
  .withVerifyJwtCallback(verifyJwtCallback)
  .withClientMetadata({
    authorizationEndpoint: 'www.myauthorizationendpoint.com',
    idTokenSigningAlgValuesSupported: [SigningAlgo.EDDSA],
    issuer: ResponseIss.SELF_ISSUED_V2,
    requestObjectSigningAlgValuesSupported: [SigningAlgo.EDDSA, SigningAlgo.ES256],
    responseTypesSupported: [ResponseType.ID_TOKEN],
    vpFormats: { jwt_vc: { alg: [SigningAlgo.EDDSA] } },
    scopesSupported: [Scope.OPENID_DIDAUTHN, Scope.OPENID],
    subjectTypesSupported: [SubjectType.PAIRWISE],
    subjectSyntaxTypesSupported: ['did:ethr'],
    passBy: PassBy.VALUE,
  })
  .build()
```

### RP creates the Auth Request

The Relying Party creates the Auth Request. This could have been triggered by the OP accessing a URL, or clicking a button
for instance. The Created SIOP V2 Auth Request could also be displayed as a QR code for cross-device flows. In the below text we are
leaving the transport out of scope.

Given we already have configured the RP itself, all we need to provide is a nonce and state for this request. These will
be communicated throughout the process. The RP definitely needs to keep track of these values for later usage. If no
nonce and state are provided then the createAuthorizationRequest method will automatically provide values for these and
return them in the object that is returned from the method.

Next to the nonce we could also pass in claim options, for instance to specify a Presentation Definition. We have
already configured the RP itself to have a Presentation Definition, so we can omit it in the request creation, as the RP
class will take care of that on every Auth Request creation.
When creating signed objects on the OP and RP side, a jwtIssuer can be specified.
These adapters provide information about how the jwt will be signed later and metadata to set certain fields in the JWT,
This means that the JWT only needs to be signed and not necessarily modified by the consumer of this library.
If the jwtIssuer is omitted the createJwtCallback will be called with method 'custom' indicating that it's up to the consumer
to populate required fields before the JWT is signed.

```typescript
const authRequest = await rp.createAuthorizationRequest({
  correlationId: '1',
  nonce: 'qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg',
  state: 'b32f0087fc9816eb813fd11f',
  jwtIssuer: { method: 'did', didUrl: 'did:key:v4zagSPkqFJxuNWu#zUC74VEqqhEHQc', alg: SigningAlgo.EDDSA },
})

console.log(`nonce: ${authRequest.requestOpts.nonce}, state: ${authRequest.requestOpts.state}`)
// nonce: qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg, state: b32f0087fc9816eb813fd11f

console.log(await authRequest.uri().then((uri) => uri.encodedUri))
// openid://?response_type=id_token&scope=openid&client_id=did.......&jwt=ey..........
```

#### Optional: OP Auth Request Payload parsing access

The OP class has a method that both parses the Auth Request URI as it was created by the RP, but it als
resolves both the JWT and the Registration values from the Auth Request Payload. Both values can be either
passed by value in the Auth Request, meaning they are present in the request, or passed by reference, meaning
they are hosted by the OP. In the latter case the values have to be retrieved from an https endpoint. The parseAuthorizationRequestURI takes
care of both values and returns the Auth Request Payload for easy access, the resolved signed JWT as well as
the resolved registration metadata of the RP. Please note that the Auth Request Payload that is also returned
is the original payload from the URI, so it will not contain the resolved JWT nor Registration if the OP passed one of
them by reference instead of value. Only the direct access to jwt and registration in the Parsed Auth Request
URI are guaranteed to be resolved.

---

**NOTE**

Please note that the parsing also automatically happens when calling the verifyAuthorizationRequest method with a URI
as input argument. This method allows for manual parsing if needed.

---

```typescript
const parsedReqURI = op.parseAuthorizationRequestURI(reqURI.encodedUri)

console.log(parsedReqURI.requestPayload.request)
// ey....... , but could be empty if the OP would have passed the request by reference usiing request_uri!

console.log(parsedReqURI.jwt)
// ey....... , always resolved even if the OP would have passed the request by reference!
```

#### OP Auth Request verification

The Auth Request from the RP in the form of a URI or JWT string needs to be verified by the OP. The
verifyAuthorizationRequest method of the OP class takes care of this. As input it expects either the URI or the JWT
string. IF a JWT is supplied it will use the JWT directly, if a URI is provided it
will internally parse the URI and extract/resolve the JWT before passing it to the provided verifyJwtCallback.
The jwtVerifier in the verifyJwtCallback is augmented with metadata to simplify jwt verification for each adapter.
The options can contain an optional nonce, which means the
request will be checked against the supplied nonce, otherwise the supplied nonce is only checked for presence. Normally
the OP doesn't know the nonce beforehand, so this option can be left out.

The verified Auth Request object returned again contains the Auth Request payload, and the issuer.

---

**NOTE**

In the below example we directly access requestURI.encodedUri, in a real world scenario the RP and OP don't have access
to shared objects. Normally you would have received the openid:// URI as a string, which you can also directly pass into
the verifyAuthorizationRequest or parse methods of the OP class. The method accepts both a JWT or an openid:// URI as
input

---

```typescript
const verifiedReq = op.verifyAuthorizationRequest(reqURI.encodedUri) // When an HTTP endpoint is used this would be the uri found in the body
// const verifiedReq = op.verifyAuthorizationRequest(parsedReqURI.jwt); // If we have parsed the URI using the above optional parsing

console.log(`RP DID: ${verifiedReq.issuer}`)
// RP DID: did:ethr:ropsten:0x028360fb95417724cb7dd2ff217b15d6f17fc45e0ffc1b3dce6c2b8dd1e704fa98
```

### OP Presentation Exchange

The Verified Request object created in the previous step contains a `presentationDefinitions` array property in case the
OP wants to receive a Verifiable Presentation according to
the [OpenID Connect for Verifiable Presentations (OIDC4VP)](https://openid.net/specs/openid-connect-4-verifiable-presentations-1_0.html)
specification. If this is the case we need to select credentials and create a Verifiable Presentation. If the OP doesn't
need to receive a Verifiable Presentation, meaning the presentationDefinitions property is undefined or empty, you can
continue to the next chapter and create the Auth Response immediately.

See the below sub flow for Presentation Exchange to explain the process:

![PE Flow diagram](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/Sphereon-Opensource/did-auth-siop/develop/docs/presentation-exchange.puml)

#### Create PresentationExchange object

If the `presentationDefinitions` array property is present it means the op.verifyAuthorizationRequest already has
established that the Presentation Definition(s) itself were valid and present. It has populated the
presentationDefinitions array for you. If the definition was not valid, the verify method would have thrown an error,
which means you should never continue the authentication flow!

Now we have to create a `PresentationExchange` object and pass in both the available Verifiable Credentials (typically
from your wallet) and the holder DID.

---

**NOTE**

The verifiable credentials you pass in to the PresentationExchange methods do not get sent to the RP. Only the
submissionFrom method creates a VP, which you should manually add as an option to the createAuthorizationResponse
method.

---

```typescript
import { PresentationExchange } from './PresentationExchange'
import { PresentationDefinition } from '@sphereon/pe-models'

const verifiableCredentials: VerifiableCredential[] = [VC1, VC2, VC3] // This typically comes from your wallet
const presentationDefs: PresentationDefinition[] = verifiedReq.presentationDefinitions

if (presentationDefs) {
  const pex = new PresentationExchange({
    did: op.authResponseOpts.did,
    allVerifiableCredentials: verifiableCredentials,
  })
}
```

#### Filter Credentials that match the Presentation Definition

Now we need to filter the VCs from all the available VCs to an array that matches the Presentation Definition(s) from
the RP. If the OP, or rather the PresentationExchange instance doesn't have all credentials to satisfy the Presentation
Definition from the OP, the method will throw an error. Do not try to authenticate in that case!

The selectVerifiableCredentialsForSubmission method returns the filtered VCs. These VCs can satisfy the submission
requirements from the Presentation Definition. You have to do a manual selection yourself (see note below).

---

**NOTE**

You can have multiple VCs that match a single definition. That can be because the OP uses a definition that wants to
receive multiple different VCs as part of the Verifiable Presentation, but it can also be that you have multiple VCs
that match a single constraint from a single definition. Lastly there can be multiple definitions. You always have to do
a final manual selection of VCs from your application (outside of the scope of this library).

---

```typescript
// We are only checking the first definition to not make the example too complex
const checked = await pex.selectVerifiableCredentialsForSubmission(presentationDefs[0])
// Has errors if the Presentation Definition has requirements we cannot satisfy.
if (checked.errors) {
  // error handling here
}
const matches: SubmissionRequirementMatch = checked.matches

// Returns the filtered credentials that do match
```

#### Application specific selection and approval

The previous step has filtered the VCs for you into the matches constant. But the user really has to acknowledge that
he/she will be sending in a VP containing the VCs. As mentioned above the selected VCs might still need more filtering
by the user. This part is out of the scope of this library as it is application specific. For more info also see
the [PEX library](https://github.com/Sphereon-Opensource/pex).

In the code examples we will use 'userSelectedCredentials' as variable for the outcome of this process.

```typescript
// Your application process here, resulting in:
import { IVerifiableCredential } from '@sphereon/pex'

const userSelectedCredentials: VerifiableCredential[] // Your selected credentials
```

#### Create the Verifiable Presentation from the user selected VCs

Now that we have the final selection of VCs, the Presentation Exchange class will create the Verifiable Presentation for
you. You can optionally sign the Verifiable Presentation, which is out of the scope of this library. As long as the VP
contains VCs which as subject has the same DID as the OP, the RP can know that the VPs are valid, simply by the fact
that withSignature of the resulting Auth Response is signed by the private key belonging to the OP and the VP.

---

**NOTE**

We do not support signed selective disclosure yet. The VP will only contain attributes that are requested if the
Presentation Definition wanted to limit disclosure. You need BBS+ signatures for instance to sign a VP with selective
disclosure. Unsigned selective disclosure is possible, where the RP relies on the Auth Response being signed
as long as the VP subject DIDs match the OP DID.

---

```typescript
// We are only creating a presentation out of the first definition to keep the example simple
const verifiablePresentation = await pex.submissionFrom(presentationDefs[0], userSelectedCredentials)

// Optionally sign the verifiable presentation here (outside of SIOP library scope)
```

#### End of Presentation Exchange

Once the VP is returned it means we have gone through the Presentation Exchange process as defined
in [OpenID Connect for Verifiable Presentations (OIDC4VP)](https://openid.net/specs/openid-connect-4-verifiable-presentations-1_0.html)
. We can now continue to the regular flow of creating the Auth Response below, all we have to do is pass the
VP in as an option.

### OP creates the Auth Response using the Verified Request

Using the Verified Request object we got back from the op.verifyAuthorizationRequest method, we can now start to create
the Auth Response. If we were in the Presentation Exchange flow because the request contained a Presentation
Definition we now need to pass in the Verifiable Presentations using the vp option. If there was no Presentation
Definition, do not supply a Verifiable Presentation! The method will check for these constraints.

```typescript
import { PresentationLocation, VerifiablePresentationTypeFormat } from './SIOP.types'

// Example with Verifiabl Presentation in linked data proof format and as part of the vp_token
const vpOpt = {
  format: VerifiablePresentationTypeFormat.LDP_VP,
  presentation: verifiablePresentation,
  location: PresentationLocation.VP_TOKEN,
}

const authRespWithJWT = await op.createAuthorizationResponse(verifiedReq, { vp: [vpOpt] })

// Without Verifiable Presentation
// const authRespWithJWT = await op.createAuthorizationResponse(verifiedReq);
```

### OP submits the Auth Response to the RP

We are now ready to submit the Auth Response to the RP. The OP class has the submitAuthorizationResponse
method which accepts the response object. It will automatically submit to the correct location as specified by the RP in
its request. It expects a response in the 200 range. You get access to the HTTP response from the fetch API as a return
value.

```typescript
// Example with Verifiable Presentation
const response = await op.submitAuthorizationResponse(authRespWithJWT)
```

### RP verifies the Auth Response

```typescript
const verifiedAuthResponseWithJWT = await rp.verifyAuthorizationResponseJwt(authRespWithJWT.jwt, {
  audience: EXAMPLE_REDIRECT_URL,
})

expect(verifiedAuthResponseWithJWT.jwt).toBeDefined()
expect(verifiedAuthResponseWithJWT.payload.state).toMatch('b32f0087fc9816eb813fd11f')
expect(verifiedAuthResponseWithJWT.payload.nonce).toMatch('qBrR7mqnY3Qr49dAZycPF8FzgE83m6H0c2l0bzP4xSg')
```
