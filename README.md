# Minimal Ethereum mainnet squid

This is a minimal runnable template of a squid run against an Ethereum mainnet. It does not index any data. To index Ethereum transaction and event data, modify `src/processor.ts` and subscribe to EVM logs and transactions. 

For a step-by-step migration guide from TheGraph, see [the dedicated docs page](https://docs.subsquid.io/migrate/migrate-subgraph/).

For full details on how to transform the data and serve with a GraphQL API, consult [the docs](https://docs.subsquid.io). 


## Quickstart

```bash
# 1. Install dependencies
npm ci

# 2. Compile typescript files
make build

# 3. Start target Postgres database and detach
make up

# 4. Start the processor
make process

# 5. The command above will block the terminal
#    being busy with fetching the chain data, 
#    transforming and storing it in the target database.
#
#    To start the graphql server open the separate terminal
#    and run. The GraphQL playground will be available at localhost:4350graphl
make serve
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
`squid-typeorm-migration(1)` tool provides several commands to drive the process.
It is all [TypeORM](https://typeorm.io/#/migrations) under the hood.

```bash
# Connect to database, analyze its state and generate migration to match the target schema.
# The target schema is derived from entity classes generated earlier.
# Don't forget to compile your entity classes beforehand!
npx squid-typeorm-migration generate

# Create template file for custom database changes
npx squid-typeorm-migration create

# Apply database migrations from `db/migrations`
npx squid-typeorm-migration apply

# Revert the last performed migration
npx squid-typeorm-migration revert   
```

### 4. Import ABI contract and generate interfaces to decode events

It is necessary to import the respective ABI definition to decode EVM logs. 

To generate a type-safe facade class to decode EVM logs, use `squid-evm-typegen(1)`. For example, for a ERC721 contract use

```bash
npx squid-evm-typegen --abi src/abi/ERC721.json --output src/abi/ERC721.ts
```


## Project conventions

Squid tools assume a certain project layout.

* All compiled js files must reside in `lib` and all TypeScript sources in `src`.
The layout of `lib` must reflect `src`.
* All TypeORM classes must be exported by `src/model/index.ts` (`lib/model` module).
* Database schema must be defined in `schema.graphql`.
* Database migrations must reside in `db/migrations` and must be plain js files.
* `sqd(1)` and `squid-*(1)` executables consult `.env` file for a number of environment variables.

## GraphQL server extensions

It is possible to extend `squid-graphql-server(1)` with custom
[type-graphql](https://typegraphql.com) resolvers and to add request validation. See [the docs](https://docs.subsquid.io/develop-a-squid/graphql-api/custom-resolvers/) for more details.


## Disclaimer

This is alpha-quality software. The Squid SDK may introduce breaking changes in future versions.
