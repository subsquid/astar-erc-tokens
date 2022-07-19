# squid-template

This is a sample [squid](https://subsquid.io) project to demonstrate its structure and conventions.
It accumulates [Moonsama](https://moonsama.com/) token transfers over the [Moonriver network](https://moonbeam.network/networks/moonriver/) and serves them via graphql API.

This sample project is worthy of notice, because it showcases the native support for EVM logs of the Subsquid SDK.

For more info consult [FAQ](./FAQ.md).

## Prerequisites

* node 16.x
* docker

## Quickly running the sample

```bash
# 1. Install dependencies
npm ci

# 2. Compile typescript files
npm run build

# 3. Start target Postgres database
docker compose up -d

# 4. Apply database migrations from db/migrations
npx sqd db create
npx sqd db migrate

# 5. Now start the processor
node -r dotenv/config lib/processor.js

# 6. The above command will block the terminal
#    being busy with fetching the chain data, 
#    transforming and storing it in the target database.
#
#    To start the graphql server open the separate terminal
#    and run
npx squid-graphql-server
```

## Dev flow

### 1. Define database schema

Start development by defining the schema of the target database via `schema.graphql`.
Schema definition consists of regular graphql type declarations annotated with custom directives.
Full description of `schema.graphql` dialect is available [here](https://docs.subsquid.io/schema-spec).

### 2. Generate TypeORM classes

Mapping developers use TypeORM [EntityManager](https://typeorm.io/#/working-with-entity-manager)
to interact with target database during data processing. All necessary entity classes are
generated by the squid framework from `schema.graphql`. This is done by running `npx sqd codegen`
command.

### 3. Generate database migration

All database changes are applied through migration files located at `db/migrations`.
`sqd(1)` tool provides several commands to drive the process.
It is all [TypeORM](https://typeorm.io/#/migrations) under the hood.

```bash
# Connect to database, analyze its state and generate migration to match the target schema.
# The target schema is derived from entity classes generated earlier.
npx sqd db create-migration

# Create template file for custom database changes
npx sqd db new-migration

# Apply database migrations from `db/migrations`
npx sqd db migrate

# Revert the last performed migration
npx sqd db revert

# DROP DATABASE
npx sqd db drop

# CREATE DATABASE
npx sqd db create            
```

### 4. Import ABI contract and set up interfaces to decode events

In order to be able to process EVM logs, it is necessary to import the respective ABI definition. In the case of this project, this has been done via the [`src/abis/ERC721.json`](src/abis/ERC721.json) file.

Furthermore, it is necessary to decode logs and this is shown in [`src/abis/erc721.ts`](src/abis/erc721.ts). The `events` dictionary define there maps the event name at the center of this project to the function used to decode it.

## Project conventions

Squid tools assume a certain project layout.

* All compiled js files must reside in `lib` and all TypeScript sources in `src`.
The layout of `lib` must reflect `src`.
* All TypeORM classes must be exported by `src/model/index.ts` (`lib/model` module).
* Database schema must be defined in `schema.graphql`.
* Database migrations must reside in `db/migrations` and must be plain js files.
* `sqd(1)` and `squid-*(1)` executables consult `.env` file for a number of environment variables.

## Graphql server extensions

It is possible to extend `squid-graphql-server(1)` with custom
[type-graphql](https://typegraphql.com) resolvers and to add request validation.
More details will be added later.

## Disclaimer

This is alpha-quality software. Expect some bugs and incompatible changes in coming weeks.