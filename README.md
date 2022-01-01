## Outline

* [Setup](#setup)
  * [Add API Keys and Database DSN](#add-api-keys-and-database-dsn)
  * [MEGA QUERY](#mega-query)
* [Project Structure](#project-structure)
  * [Entry Point](#entry-point)
  * [StepZen Configuration](#stepzen-configuration)
* [Develop Using Mock JSON Data](#develop-using-mock-json-data)
  * [Test Interface](#test-interface)
  * [getTest Query](#gettest-query)
  * [Run test query](#run-test-query)
* [How to Connect to a REST Service](#how-to-connect-to-a-rest-service)
  * [Breed Type](#breed-type)
  * [Breed by Id Query](#breed-by-id-query)
  * [Run Breed Test Query](#run-breed-test-query)
* [How to Connect to a GraphQL API](#how-to-connect-to-a-graphql-api)
  * [Character and Characters Type](#character-and-characters-type)
  * [Characters Query](#characters-query)
  * [Run Characters Test Query](#run-characters-test-query)
* [How to Introspect a MySQL Database](#how-to-introspect-a-mysql-database)
  * [Getting Started with MySQL on Railway](#getting-started-with-mysql-on-railway)
  * [Seed your database](#seed-your-database)
  * [Run stepzen import mysql](#run-stepzen-import-mysql)
  * [Enter your Database Credentials](#enter-your-database-credentials)
  * [Countries Type](#countries-type)
  * [Countries Query](#countries-query)
  * [Run Countries Test Query](#run-countries-test-query)

## Setup

Before this project will work, you need a MySQL database and an API key for the Cat API.

### Add API Keys and Database DSN

Rename `example.config.yaml` to `config.yaml`.

```bash
mv example.config.yaml config.yaml
```

Include a MySQL database DSN (if needed you can get one [here](https://dev.new/)) and an API key from the [Cat API](https://thecatapi.com/).

```yaml
configurationset:
  - configuration:
      name: mysql_config
      dsn: YOUR_DATABASE_DSN
  - configuration:
      name: cat_config
      Authorization: Apikey YOUR_API_KEY
```

Deploy your StepZen endpoint with `stepzen start`. Make sure you have a [StepZen](https://stepzen.com) account and the [StepZen CLI](https://stepzen.com/docs/cli).

```bash
stepzen start
```

Open [localhost:5001](http://localhost:5001) and run the following GraphQL query.

```graphql
query HELLO_WORLD {
  getTest {
    String
  }
}
```

### MEGA QUERY

```graphql
query MEGA_QUERY {
  getTest {
    Date
    DateTime
    Float
    Int
    JSON
    JSON2
    String
  }
  
  breedById(id: "abys") {
    id
    life_span
    name
    temperament
    origin
  }
  
  characters {
    results {
      id
      name
      image
    }
  }
  
  getCountriesList {
    GDPUSD
    country_name
    id
    isoCode
  }
}
```

## Project Structure

```
├── mysql
│   └── mysql.graphql
├── schema
│   ├── breed.graphql
│   ├── characters.graphql
│   └── test.graphql
├── .gitignore
├── example.config.yaml
├── index.graphql
├── README.md
└── stepzen.config.js
```

### Entry Point

```graphql
# index.graphql

schema
  @sdl(
    files: [
      "mysql/mysql.graphql"
      "schema/breed.graphql"
      "schema/characters.graphql"
      "schema/test.graphql"
    ]
  ) {
  query: Query
}
```

### StepZen Configuration

```json
{
  "endpoint": "api/stepzen-101"
}
```

## Develop Using Mock JSON Data

Generate a flat JSON object containing a range of common data types. If you have an interface and define a query that returns that interface you can execute the query and get mock data back with types including:

* `String`
* `Int`
* `Float`
* `JSON`
* `Date`
* `DateTime`

### Test Interface

Our schema creates a `Test` interface that includes a named type for each specified type.

```graphql
# schema/test.graphql

interface Test {
  String: String!
  Int: Int!
  Float: Float!
  JSON: JSON!
  JSON2: JSON!
  Date: Date!
  DateTime: DateTime!
}
```

### getTest Query

The `getTest` query returns a single `Test` object.

```graphql
# schema/test.graphql

type Query {
  getTest: Test
}
```

### Run test query

The `MockJSON` query runs the `getTest` query and returns a `Test` with all of the specified types.

```graphql
query MOCK_JSON {
  getTest {
    Date
    DateTime
    Float
    Int
    JSON
    JSON2
    String
  }
}
```

## How to Connect to a REST Service

`@rest` is a custom StepZen directive that connects you to a REST API. It supports PUT, POST and GET http methods. We'll be using the [Cat API](https://thecatapi.com/) to get a [breed by ID](https://docs.thecatapi.com/api-reference/breeds/breeds-list).

### Breed Type

Our schema creates a `Breed` type with `id`, `name`, `temperament`, and `life_span`.

```graphql
# schema/breed.graphql

type Breed {
  id: String!
  name: String!
  temperament: String!
  life_span: String!
  origin: String!
}
```

### Breed by Id Query

**`endpoint`** tells StepZen what endpoint to call. You can see the variable `$id` is set with a dollar sign in the url. **`configuration`** refers to your configuation setting in config.yaml. More on that below.

```graphql
# schema/breed.graphql

type Query {
  breedById(id: String!): [Breed]
    @rest(
      endpoint: "https://api.thecatapi.com/v1/breeds/search?q=$id"
      configuration: "cat_config"
    )
}
```

### Run Breed Test Query

```graphql
query BREED_QUERY {
  breedById(id: "abys") {
    id
    life_span
    name
    temperament
    origin
  }
}
```

## How to Connect to a GraphQL API

Connect to the public [Rick and Morty GraphQL API](https://rickandmortyapi.com/graphql) with the `@graphql` directive.

### Character and Characters Type

The `Character` object has three types:

* `id` with the type `ID`
* `name` for the character's name with the type `String`
* `image` of the character with the type `String`. The `image` is a URL to a `.jpeg` file contained within the string.

To match the Rick and Morty API schema, return a `results` array containing the `Character` objects and call the type `Characters`.

```graphql
# schema/characters.graphql

type Character {
  id: ID
  name: String
  image: String
}

type Characters {
  results: [Character]
}
```

### Characters Query

Our `Query` type has a `characters` query that returns the `Characters`, as specified in our previous types. The `@graphql` directive takes the `endpoint` of the GraphQL API, which in this case is `https://rickandmortyapi.com/graphql` with the URL placed in between quotation marks.

```graphql
# schema/characters.graphql

type Query {
  characters: Characters
    @graphql(
      endpoint: "https://rickandmortyapi.com/graphql"
    )
}
```

### Run Characters Test Query

To test our new endpoint, run the following query called `CHARACTERS_QUERY`. This executes the `characters` query and returns an array of `Character` objects with their `id`, `name`, and `image`.

```graphql
query CHARACTERS_QUERY {
  characters {
    results {
      id
      name
      image
    }
  }
}
```

## How to Introspect a MySQL Database

By introspecting your database, StepZen enables you to import custom schemas for your deployed MySQL databases.

### Getting Started with MySQL on Railway

Click [dev.new](https://dev.new/) and choose "Provision MySQL". After the database is setup click **"MySQL"** on the left and then choose **"Connect"**. Copy and paste the MySQL client command.

```bash
mysql -hcontainers-us-west-22.railway.app \
  -uroot -xxxx \
  --port 7309 \
  --protocol=TCP \
  railway
```

Your own password will be included in place of `xxxx`. Download MySQL with `brew install mysql`.

### Seed your database

Use the following SQL command to create a `countries` table in your database.

```sql
CREATE TABLE countries (
  id varchar(255),
  country_name varchar(255),
  isoCode varchar(255),
  GDPUSD int
);
```

Insert three countries into your newly created table.

```sql
INSERT INTO countries (
  id, country_name, isoCode, GDPUSD
)
VALUES (
  "Q889",
  "Afghanistan",
  "AFN",
  19.29
),
(
  "Q953",
  "Zambia",
  "ZMW",
  23.31
),
(
  "Q664",
  "New Zealand",
  "NZD",
  206.9
);
```

Verify that the seed command worked with a `SELECT` query.

```sql
SELECT * FROM countries;
```

### Run stepzen import mysql

```bash
stepzen import mysql
```

### Enter your Database Credentials

We need to include both the `host` and `port` for the first question `What is your host.` For example, if you are using Railway you will enter something like the following:

```
? What is your host? containers-us-west-11.railway.app:7199
? What is your database name? railway
? What is the username? root
? What is the password? [hidden]
```

### Countries Type

```graphql
# mysql/mysql.graphql

type Countries {
  GDPUSD: Int
  country_name: String!
  id: String
  isoCode: String
}
```

### Countries Query

The query type `getCountriesList` returns all the information on the countries in that table. It effectively performs a `SELECT * FROM countries` SQL command using StepZen's custom `@dbquery` directive for connecting to a database.

```graphql
# mysql/mysql.graphql

type Query {
  getCountriesList: [Countries]
    @dbquery(
      type: "mysql",
      table: "countries",
      configuration: "mysql_config"
    )
}
```

### Run Countries Test Query

```graphql
query COUNTRIES_QUERY {
  getCountriesList {
    GDPUSD
    country_name
    id
    isoCode
  }
}
```