---
title: Introduction
description: 'OpenID for Verifiable Credential Issuance - Issuer'
---

  <img className="block dark:hidden" src="/logo/light.svg" alt="Sphereon" />
  <img className="hidden dark:block" src="/logo/dark.svg" alt="Sphereon" />

[![CI](https://github.com/Sphereon-Opensource/openid4vci-client/actions/workflows/main.yml/badge.svg)](https://github.com/Sphereon-Opensource/openid4vci-client/actions/workflows/main.yml) [![codecov](https://codecov.io/gh/Sphereon-Opensource/openid4vci-client/branch/develop/graph/badge.svg)](https://codecov.io/gh/Sphereon-Opensource/openid4vci-client) [![NPM Version](https://img.shields.io/npm/v/@sphereon/oid4vci-client.svg)](https://npm.im/@sphereon/oid4vci-client)

**_WARNING: This readme and code base is a copy of the Sphereon Github repository. The code as such will not be maintained in this repository. If you have a question, need
support, or a suggest go to the [Sphereon Github](https://github.com/Sphereon-OpenSource) please_**

# Background

The OID4VCI issuer is used in issuer type applications, where an organization is issuing the credential(s)

# Flows

The spec lists 2 flows. Currently only one is supported!

## Authorized Code Flow

This flow isn't supported yet!

## Pre-authorized Code Flow

The pre-authorized code flow assumes the user is using an out-of-band mechanism outside the issuance flow to
authenticate first.

The below diagram shows the steps involved in the pre-authorized code flow. Note that wallet inner functionalities (like
saving VCs) are out of scope for this library.

![Flow diagram](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/Sphereon-Opensource/OID4VCI-client/develop/docs/preauthorized-code-flow.puml)
