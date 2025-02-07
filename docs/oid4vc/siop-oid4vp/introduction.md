---
title: Introduction
description: 'Self Issued OpenID Provider (SIOPv2) with OpenID4VP support'
---

An OpenID authentication library conforming to
the [Self Issued OpenID Provider v2 (SIOPv2)](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html)
and [OpenID for Verifiable Presentations (OpenID4VP)](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html)
as specified in the OpenID Connect working group.

## Introduction

[SIOP v2](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html) is an OpenID specification to allow End-users to act as OpenID Providers (OPs) themselves. Using
Self-Issued OPs, End-users can authenticate themselves and present claims directly to a Relying Party (RP),
typically a webapp, without involving a third-party Identity Provider. This makes the interactions fully self sovereign, as
it doesn't depend on any third parties and strictly happens peer 2 peer, yet still using well known constructs from the OpenID protocol.

Next to the user acting as an OpenID Provider, this library also has support for Verifiable Presentations using
the [Presentation Exchange](https://identity.foundation/presentation-exchange/) provided by
our [PEX](https://github.com/Sphereon-Opensource/pex) library. This means that the Relying Party can express submission
requirements in the form of Presentation Definitions, defining the Verifiable Credentials(s) types it would like to receive from the User/OP.
The OP then checks whether it has the credentials to support the Presentation Definition. Only if that is the case it will send the relevant (parts of
the) credentials as a Verifiable Presentation in the Authorization Response destined for the Webapp/Relying Party. The
relying party in turn checks validity of the Verifiable Presentation(s) as well as the match with the submission
requirements. Only if everything is verified successfully the RP serves the protected page(s). This means that the
authentication can be extended with claims about the authenticating entity, but it can also be used to easily consume
credentials from supporting applications, without having to setup DIDComm connections for instance. These credentials can either be self-asserted or from trusted 3rd party issuer.

The term Self-Issued comes from the fact that the End-users (OP) issue self-signed ID Tokens to prove validity of the
identifiers and claims. This is a trust model different from regular OpenID Connect where the OP is run by the
third party who issues ID Tokens on behalf of the End-user to the Relying Party upon the End-user's consent. This means
the End-User is in control about his/her data instead of the 3rd party OP.

## Demo: 

<iframe 
  src="https://player.vimeo.com/video/630104529?badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" 
  frameborder="0" 
  className="w-full aspect-video"
  allow="autoplay; fullscreen; picture-in-picture; clipboard-write" 
  title="OIDC with SIOPv2 and DIF Presentation Exchange">
</iframe>
and a more stripped down demo: 

<iframe
  className="w-full aspect-video"
  src="https://www.youtube.com/embed/cqoKuQWPj-s"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
></iframe>

## Active Development

_IMPORTANT:_

- _This software still is in an early development stage. As such you should expect breaking changes in APIs, we
  expect to keep that to a minimum though. Version 0.3.X has changed the external API, especially for Requests, Responses and slightly for the RP/OP classes._
- _The name of the package also changed from [@sphereon/did-auth-siop](https://www.npmjs.com/package/@sphereon/did-auth-siop) to [@sphereon/siopv2-oid4vp](https://www.npmjs.com/package/@sphereon/SIOP-OpenID4VP), to better reflect specification name changes_

## Functionality

This library supports:

- Generic methods to verify and create/sign Json Web Tokens (JWTs) as used in OpenID Connect, with adapter for Decentralized Identifiers (DIDs), JSON Web Keys (JWK), x509 certificates
- OP class to create Authorization Requests and verify Authorization Responses
- RP class to verify Authorization Requests and create Authorization Responses
- Verifiable Presentation and Presentation Exchange support on the RP and OP sides, according to the OpenID for Verifiable Presentations (OID4VP) and Presentation Exchange specifications
- SIOPv2 specification version discovery with support for the latest [development version (draft 11)](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html), [Implementers Draft 1](https://openid.net/specs/openid-connect-self-issued-v2-1_0-ID1.html) and the [JWT VC Presentation Interop Profile](https://identity.foundation/jwt-vc-presentation-profile/)


## Acknowledgements

This library has been partially sponsored by [Gimly](https://www.gimly.io/) as part of the [NGI Ontochain](https://ontochain.ngi.eu/) project. NGI Ontochain has received funding from the European Union’s Horizon 2020 research and innovation programme under grant agreement No 957338

<center><a href="https://www.gimly.io/"><img src="https://avatars.githubusercontent.com/u/64525639?s=200&v=4" alt="Gimly" height="80"/></a>&nbsp;<a href="https://ontochain.ngi.eu" target="_blank"><img src="https://ontochain.ngi.eu/sites/default/files/logo-ngi-ontochain-positive.png" height="100px" alt="ONTOCHAIN Logo"/></a>&nbsp;<img src="https://ontochain.ngi.eu/sites/default/files/images/EU_flag.png" height="80px" alt="European Union Flag"/></center>
