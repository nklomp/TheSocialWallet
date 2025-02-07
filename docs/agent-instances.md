**_WARNING: This readme and code base is a copy of the Sphereon Github repository. The code as such will not be maintained in this repository. If you have a question, need
support, or a suggest go to the [Sphereon Github](https://github.com/Sphereon-OpenSource) please_**

# Agent instances

The agent can be configured using several environment variables. Amongst these are variables to enable certain
functionalities of the agent. If you want to use Docker then there are 2 distinct agent versions you can run.
This document works based on the agents and code available in the [web wallet repository]([web wallet](https://github.com/Sphereon-Opensource/web-wallet). The agent for the web
wallet however can be used standalone as well. More info about the agent can be found [here](./agent.md)

- A standalone agent, to be used without the [web wallet](https://github.com/Sphereon-Opensource/web-wallet), only enabling REST APIs
- The web wallet agent, enabling certain features needed for the web wallet to run. This includes some additional services and depencies.


- The Sphereon **Standalone Agent**: This agent running on port 5010 by default, runs without a web-wallet, and
is responsible for issuance and optional
storage of Verifiable Credentials. Creating DIDs from the REST API is enabled on this agent. Resolution of DIDs will
use hybrid resolution, meaning any did:web will be resolved to the actual https endpoint, but it also resolved
non-published DIDs only available to the agent. The W3C VC API is available, and also support to act as a Relying Party.
- The **Wallet Agent**: This agent running on port 5010 by default, it can create and verify Verifiable
Credentials using a W3C VC API, or using OID4VC. The DIDs will be resolved in hybrid mode, meaning the agent will
first look
whether the DID is managed by the agent and then generate a DID resolution result from the database. If not managed by
the agent it will perform an external resolution call.

# Agent documentation

The [agent documentation](./agent.md) contains information about supported features, methods,
environment variables, as well as how to call the different REST API endpoints

# Building and testing

## Docker

Docker images are provided in the `docker` folder for both agent types. Please read

You can run `docker compose up` to run the agents in Docker.

The [docker](https://github.com/Sphereon-Opensource/web-wallet/tree/feature/surf/docker) directory contains several Docker files and docker compose files. Be aware that these are currently mutually
exclusive. In other words, by default you cannot run multiple docker compose files together. This mainly has to do with
the fact they are sharing the same ports and configuration variables. It would be relatively straightforward to adjust
these in case you do need to run multiple instances simultaneously.

The different docker(compose files):
- standalone-agent-full: This image allows you to run a standalone agent, without the web-wallet being involved. It has most features enabled by default. You can adjust them via the .env file

The below docker files are currently being worked on, so they might not work
- standalone-agent-verifier: This image allows you to run a standalone agent, without the web-wallet being involved, and focused on verifier/RP functionality.
- standalone-agent-issuer: This image allows you to run a standalone agent, without the web-wallet being involved, and focused on issuer/provider functionality.
- wallet-agent-full: This image runs the agent, including support for the web wallet. It requires a postgresql (only) database. Sqlite is not supported in this mode currently.
- wallet-frontend-only: This image runs the web wallet frontend only. Be aware that the frontend always needs the backend/agent to run as well


## OpenAPI

There is an [OpenAPI definition](https://app.swaggerhub.com/apis/SphereonInt/vc-wallet-api) for some wallet endpoints.
the [docs/openapi](./docs/openapi) folder.
You can use the definition to generate models for a target language of choice.
This folder also contains an [HTML documentation](./docs/openapi/index.html) export of the REST API endpoints and
models.

## From source
We recommend using this approach if you are testing/developing.

The below commands need to be run from the web-wallet folder

### Lerna

These module make use of Lerna for managing multiple packages. Lerna is a tool that optimizes the workflow around
managing multi-package repositories with git and pnpm.

### Build

The below command builds all packages for you using lerna

### Pnpm

To build the project [pnpm](https://www.npmjs.com/package/pnpm) is used. Do not confuse this package manager with the
more regular `npm`.

Install pnpm globally:

```shell
npm -g install pnpm
```

Install the dependencies of all the projects

```shell
pnpm install
```

Build the projects

```shell
pnpm build
```

#### Production commands

If you want to run this project in production, directly from the project, instead of using an NPM repo for this project,
follow the below steps.

- Build the project according to the above steps first. This is needed because you will need to create the `dist`
folders, and it needs the NodeJS and Typescript libraries during build.
- Remove the `node_modules` top-level folder, keep any `dist` folder, as that is where the built project is to be found.
You can also run the command below (ignore the error about node_modules missing at the end)

```shell
pnpm run clean:modules
```

- Install modules without dev dependencies and also do it offline, since everything should already be available

```shell
pnpm run install:prod
# The above is the same as pnpm install --prod --offline
```

- Running the production installation

```shell
pnpm run start:prod
```

### Utility scripts

There are other utility scripts that help with development.

* `pnpm fix:prettier` - runs `prettier` to fix code style.

### Publish

Please note that currently the packages are marked as internal. Meaning they will not be published to an NPM repository!

There are scripts that can publish the following versions:

* `latest`
* `next`
* `unstable`

```shell
pnpm publish:[version]
```
