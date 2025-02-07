# The social wallet Project

The Social Wallet project, aligned with the Trustchain initiative, aims to address digital inclusion for marginalized communities. Prior to its launch
in Weert, the project focused on strategic development, prioritizing user-friendly design and robust privacy and security measures.
Engaging with stakeholders, including local authorities and potential users, was key to ensure the platform's relevance.
The project's objective is to empower vulnerable individuals through digital tools and access to essential services, employing privacy-preserving
technology for data security.
The upcoming launch in Weert is pivotal for real-world application and assessment. Post-launch, plans include scaling, enhancing user experience, and
developing sustainable models.
The project's alignment with Trustchain's objectives encompasses empowering citizens, safeguarding data privacy, and fostering digital inclusion and
sustainability.

## 1. About

The Social Wallet project's key achievement lies in its innovative use of identity technology to streamline and secure the distribution of social
benefits through decentralized digital identities.
By leveraging decentralized identifiers (DIDs) and verifiable credentials (VCs), the project has significantly enhanced privacy, security, and user
control over personal data, marking a substantial improvement over traditional, centralized systems.
Impact:
• Improved Accessibility: Made social benefits more accessible and inclusive, especially for marginalized communities.
• Increased Trust and Transparency: Fostered greater trust between entities and individuals, thanks to transparent and tamper-proof transactions.
• Enhanced Security and Privacy: Ensured robust data protection, reducing the risk of fraud and unauthorized access.
Potential Future Developments:
• Scalability Enhancements: Further research and development to ensure the platform can accommodate a growing number of users and transactions without
compromising performance.
• Cross-Border Utility: Expansion of the platform's utility to enable cross-border recognition and exchange of verifiable credentials, broadening its
applicability.
• Integration with Public Services: Collaboration with government and non-profit organizations to extend the platform's capabilities to other public
services, enhancing overall societal welfare.
The TrustChain project not only represents a significant leap forward in the management and distribution of social benefits but also sets the stage
for broader adoption and integration of decentralized digital identity technologies across various sectors.

You can find a demo of the solution at https://www.youtube.com/watch?v=6ZjOMY3Wuzo

# 2. Repositories

Following software code is developed and enhanced so far:

1. [Web wallet and management platform](https://github.com/Sphereon-Opensource/web-wallet): The backend platform allowing for management, verification and issuance of credentials
2. [Edge wallet for citizens](https://github.com/Sphereon-Opensource/mobile-wallet): Repository with the mobile edge wallet for citizens and vendors
3. [SSI-SDK](https://github.com/Sphereon-Opensource/SSI-SDK): Lower level SDK modules

**_WARNING: This readme and code base is a copy of the Sphereon Github repository. The code as such will not be maintained in this repository. If you have a question, need
support, or a suggest go to the [Sphereon Github](https://github.com/Sphereon-OpenSource) please_**

# 3. Installation and running the solution

## Mobile wallet

![mobile-wallet.png](mobile-wallet.png)

_Image: Wallet designed as part of this project_

For the mobile wallet you can just go to the Android PlayStore and iOS app store and search for `Sphereon wallet`. More information for the mobile wallet can be found in
the [README.md](mobile-wallet/README.md)

## Web wallet

### Tools (pnpm)

We use pnpm as package manager. Install it with the following command:

```shell
npm install --global pnpm
```

<b>NOTE</b>: After installation be sure to close the terminal window. If installed from your IDE, be sure to close the
IDE and start it (do not restart, as it might not pick up the latest environment variables)

Go to the [web-wallet](./web-wallet) folder root and execute the following command:

```shell
npm install
```

This will install all the dependencies of the agent and web wallet frontend.

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

### Postgresql Database

We use postgresl as a database. You can install it from https://www.postgresql.org/ Installation of Postgresql is out of scope of this document.
Make sure that port 5432 is available and that any exceptions needed are created in a firewall.

### OpenID Connect capable server

You will need an OpenID Connect capable Authorization Server. The web wallet is using this for authentication of the users.
One of the most well-known solutions for this is [Kecloak](https://www.keycloak.org/). How to install/configure KeyCloak or any other OIDC solution is out of scope for this
document.

### Agent

Copy the .env.example to .example.local. The file itself contains information about the environment variable you can use.

See the example below to connect with a postgresql DB named `vc-sphereon-db` on localhost, with username `user` and
password `password`

```properties
# The database connection name
DB_CONNECTION_NAME=default
# Whether to enable the cache for the DB.
DB_CACHE_ENABLED=true
# The Database type. Currently only sqlite and postgresql are supported
DB_TYPE=postgres
# The URL of the database. Either use the URL (more flexible), or you can also use DB_HOST and DB_PORT for postgres
# In case of sqlite, this should be the path, like 'database/agent_default.sqlite'
# For postgres you can also include username and password: postgresql://user:password:5432/vc-issuer-db?sslmode=prefer
# When using a postgresl url, the sslmode param options can be found here: https://www.postgresql.org/docs/12/libpq-connect.html
DB_URL="postgresql://user:password@localhost:5432/vc-sphereon-db?sslmode=prefer"
# The encryption key to use for the database for encrypted fields, like private keys. Needs to be unique per environment.
# Key needs to be in hex with length 64.
DB_ENCRYPTION_KEY=29739248cad1bd1a0fc4d9b75cd4d2990de535baf5caadfdf8d8f86664aa830c
```

More info on the agent setup can be found in [agent.md](docs/agent.md), [agent-setup.md](docs/agent-setup.md) and [agent-instances.md](docs/agent-instances.md)

### Webwallet frontend

Set the `NEXTAUTH_URL` environment variable in the `.env.local` file or use the shell. The value is the application url:

```shell
export NEXTAUTH_URL=<protocol://domain:port/path>

// example:
NEXTAUTH_URL=http://localhost:3001/
```

**OIDC settings**
See above for the OpenID Connect setup. You will need an OIDC server, with a client

Set the below values in `.env.local`:

```properties
# OpenID Connect values needed. Replace with your values
OIDC_ISSUER=https://auth01.test.sphereon.com/auth/realms/example 
OIDC_CLIENT_ID=your-configured-clientid
OIDC_CLIENT_SECRET=Your configured secret
```

You also need to make sure that next to a client, you also have an account/user on the OIDC server that you can use to login to the wallet. No roles or other setup is needed. As
long as both the client and user are part of the same tenant/realm.

#### Starting the Web-Wallet

First you should copy the `.env.example` file to `.env.local` and make the necessary changes as also described above.

The Web-Wallet can be started by running the following command.

Development mode

```shell
pnpm dev
```

Production mode

Create a production build

```shell
pnpm build
```

Start the server

```shell
pnpm start
```

Now you can go to http://localhost:3001

#### Local Development Setup for Web-Wallet with Supabase

The current wallet is using Supabase in certain areas. We are planning to remove that in the short term.
Supabase has two options to work with it locally. You can either use a CLI tool that they have provided or use a
dockerized
solution to do so.

- [Supabase CLI](https://supabase.com/docs/guides/cli/local-development)
- [Supabase self-hosting with docker](https://supabase.com/docs/guides/self-hosting/docker)

Following the guides above, you can connect and use your local supabase instance.

# 4. Additional documentation

The latest documentation including more information than you can find in this repository is available at https://docs.sphereon.com This includes examples and REST API Postman
collections, swagger/OpenAPI links and live API interactions.

In this repo:

All document can be found [here](./docs). The individual `web-wallet` [README.md](./web-wallet/README.md) and [docs](./web-wallet/docs) contains more information as well. Lastly
there is the agent [README.md](./web-wallet/packages/agent/README.md) and web frontend [README.md](./web-wallet/packages/web-wallet/README.md).

## Agent
1. [Agent info](./docs/agent.md)
2. [Agent setup](./docs/agent-setup.md)
3. [Agent instances](./docs/agent-instances.md)

## OID4VC
1. [Introduction](./docs/oid4vc/introduction.md)
2. [Credential Branding in Verifiable Credentials](./docs/oid4vc/oid4vci/credential-branding.md)
3. [OID4VCI Issuer](./docs/oid4vc/oid4vci/issuer/REST.md)

## Wallet
1. [Mobile Wallet README](./mobile-wallet/README.md)
2. [Mobile Wallet documents](./mobile-wallet/docs)
3. [Web Wallet README](./web-wallet/README.md)


# 5. About The Social Wallet project and Sphereon

Sphereon has received funding from the NGI TrustChain project.

<img src="https://github.com/CreatorCredentials/.github/assets/14913025/f53962c2-7c4c-4312-a0b2-1485de3e60e5" width="250"/>
The development work and innovation process has been funded by European Union’s Horizon 2020 research and innovation programme under grant agreement No. 101093274
The social wallet components are released as an Open-Source Libraries in Github.
[https://trustchain.ngi.eu/the-social-wallet/creatorcredentials](https://trustchain.ngi.eu/the-social-wallet/)

Sphereon is a Dutch technology provider specializing in Digital Identity and Signatures services with a focus on Open-Source.
For more info visit [Sphereon website](https://sphereon.com)
