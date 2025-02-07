---
title: Introduction
description: 'OpenID for Verifiable Credentials'
---

  <img className="block dark:hidden" src="/logo/light.svg" alt="Sphereon" />
  <img className="hidden dark:block" src="/logo/dark.svg" alt="Sphereon" />
**_WARNING: This readme and code base is a copy of the Sphereon Github repository. The code as such will not be maintained in this repository. If you have a question, need
support, or a suggest go to the [Sphereon Github](https://github.com/Sphereon-OpenSource) please_**

# Background

This software has a client and issuer package to request and receive Verifiable Credentials using
the [OpenID for Verifiable Credential Issuance](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) (
OpenID4VCI) specification for receiving Verifiable Credentials as a holder/subject. In addition the monorepo contains a package
for requesting the presentation of Verifiable Credentials and Verifying these presentations [OpenID for Verifiable Presentations](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html) (
OpenID4VP)

OpenID4VCI defines an API designated as Credential Endpoint that is used to issue verifiable credentials and
corresponding OAuth 2.0 based authorization mechanisms (see [RFC6749]) that a Wallet uses to obtain authorization to
receive verifiable credentials. W3C formats as well as other Credential formats are supported. This allows existing
OAuth 2.0 deployments and OpenID Connect OPs (see [OpenID.Core]) to extend their service and become Credential Issuers.
It also allows new applications built using Verifiable Credentials to utilize OAuth 2.0 as an integration and
interoperability layer. This package provides holder/wallet support to interact with OpenID4VCI capable Issuer systems.

In addition to the client and issuer, there is also a common package, which has all the types and payloads shared between the client and issuer.

## OpenID for VCI Client

The OpenID4VCI client is typically used in wallet type of applications, where the user is receiving the credential(s). More info can be found in the client [README](./packages/client/README.md)

## OpenID for VCI Issuer

The OpenID4VCI issuer is used in issuer type applications, where an organization is issuing the credential(s). More info can be found in the issuer [README](./packages/issuer/README.md). 
Please note that the Issuer is a library. It has some examples on how to run it with REST endpoints. If you are however looking for a full solution we suggest our [SSI SDK](https://github.com/Sphereon-Opensource/ssi-sdk) or the [demo](https://github.com/Sphereon-Opensource/OID4VC-demo)

## OpenID for Verifiable Presentations

The SIOP-OpenID4VP package is used in wallet type applications and verifier type of applications. Meaning it provides both Wallet (OpenId Provider) and Verifier (Relying Party) functionality. More info can be found in the siop-oid4vp package [README](./packages/siop-oid4vp/README.md)

# OpenID for VCI Flows

The spec lists 2 flows:

## Authorized Code Flow

This flow is supported but might need more work, so you might run into issues trying to use it.

## Pre-authorized Code Flow

The pre-authorized code flow assumes that the user is using an out of bound mechanism outside the issuance flow to
authenticate first.

The below diagram shows the steps involved in the pre-authorized code flow. Note that inner wallet functionalities (like
saving VCs) are out of scope for this library.
![Flow diagram](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/Sphereon-Opensource/OID4VCI-client/develop/docs/preauthorized-code-flow.puml)

# OpenID for VP Flows

Visit the [README](./packages/siop-oid4vp/README.md) for more information.
