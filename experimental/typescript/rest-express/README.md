# REST API Example

This example shows how to implement a **REST API with TypeScript** using [Express](https://expressjs.com/) and [Prisma Client](https://www.prisma.io/docs/concepts/components/prisma-client).

## How to use

### 1. Download example & install dependencies

Clone this repository:

```
git clone git@github.com:prisma/prisma-examples.git --depth=1
```

Install Node dependencies:

```
cd prisma-examples/experimental/typescript/rest-express
npm install
```

Note that this also generates Prisma Client JS into `node_modules/@prisma/client` via a `postinstall` hook of the `@prisma/client` package from your `package.json`.

### 2. Migrate your database schema

Perform an initial schema migration against your database using the following commands:

```
npx prisma migrate save --name 'init' --experimental
npx prisma migrate up --experimental
```

The first step will save the migration in the `prisma/migrations` folder. The second step will execute the migrations.

> **Note**: You're using [npx](https://github.com/npm/npx) to run Prisma 2 CLI that's listed as a development dependency in [`package.json`](./package.json). Alternatively, you can install the CLI globally using `npm install -g @prisma/cli`. When using Yarn, you can run: `yarn prisma dev`.

<Details>
<Summary><b>Alternative: </b>Connect to your own database</Summary>

Prisma supports MySQL and PostgreSQL at the moment. If you would like to connect to your own database, you can do so by specifying a different data source in the [Prisma schema file](prisma/schema.prisma).

For a MySQL provider:

```prisma
datasource mysql {
    provider = "mysql"
    url      = "mysql://johndoe:secret42@localhost:3306/mydatabase"
}
```

*OR*

For a PostgreSQL provider:

```prisma
datasource postgresql {
  provider = "postgresql"
  url      = "postgresql://johndoe:secret42@localhost:5432/mydatabase?schema=public"
}
```

> **Note**: In the above example connection strings, `johndoe` would be the username to your database, `secret42` the password, `mydatabase` the name of your database, and `public` the [PostgreSQL schema](https://www.postgresql.org/docs/9.1/ddl-schemas.html).

</Details>

### 3. Generate Prisma Client

Run the following command to generate your Prisma Client API:

```
npx prisma generate
```

This command updated the Prisma Client API in `node_modules/@prisma/client`.

### 4. Seed the database with test data

The `seed` script from `package.json` contains some code to seed the database with test data. Execute it with the following command:

```
npm run seed
```

### 5. Start the REST API server

```
npm run dev
```

The server is now running on `http://localhost:3000`. You can send the API requests implemented in `index.js`, e.g. [`http://localhost:3000/feed`](http://localhost:3000/feed).

## Using the REST API

You can access the REST API of the server using the following endpoints:

### `GET`

- `/post/:id`: Fetch a single post by its `id`
- `/feed`: Fetch all _published_ posts
- `/filterPosts?searchString={searchString}`: Filter posts by `title` or `content`

### `POST`

- `/post`: Create a new post
  - Body:
    - `title: String` (required): The title of the post
    - `content: String` (optional): The content of the post
    - `authorEmail: String` (required): The email of the user that creates the post
- `/user`: Create a new user
  - Body:
    - `email: String` (required): The email address of the user
    - `name: String` (optional): The name of the user

### `PUT`

- `/publish/:id`: Publish a post by its `id`

### `DELETE`

- `/post/:id`: Delete a post by its `id`


## Evolving the app with Prisma Migrate

Evolving the application typically requires four subsequent steps:

1. Update your Prisma schema
1. Save and migrate the database schema using Prisma Migrate
1. Generate Prisma Client to match the new schema with `prisma generate`
1. Use the updated Prisma Client in your application code

For the following example scenario, assume you want to add a "profile" feature to the app where users can create a single profile (1:1 relationship) and write a short bio about themselves and add a link to an image.

### 1. Update your Prisma schema

The first step would be to add a new model to the Prisma schema, e.g. called `Profile`. Prisma Migrate will create a corresponding table in the database.

To define a `1:1` relationship between `User` and `Profile`, you need to specify the relation field on both models as follows:

Add the Profile model with a `user` field of type `User`:

```prisma
model Profile {
  id       Int    @id @default(autoincrement())
  user     User
  bio      String
  imageURL String?
}
```

Add the `Profile` relation field to the `User` model:

```diff
model User {
   id      Int      @id @default(autoincrement())
   email   String   @unique
   name    String?
   posts   Post[]
+  profile Profile?
}
```

### 2. Save and run the migration

#### Save the migration

To migrate the database schema you first need to save the migration as follows:

```
npx prisma migrate save --name 'add-profile' --experimental
```

The CLI will output the planned changes and save the migration files in `prisma/migrations/20200313000000-add-profile/` (they contain details about required migrations steps and SQL operations).

#### Run the migration

Now that the migration has been saved you can run the migration as follows:

```
npx prisma migrate up --experimental
```

This will actually perform the schema migration against the database.

Assuming everything went right, the CLI should output:

```
🚀    Done with 1 migration
```

### 3. Generate Prisma Client

With the updated Prisma schema, you can now also update the Prisma Client API with the following command:

```
npx prisma generate
```

This command updated the Prisma Client API in `node_modules/@prisma/client`.

## Next steps

- Read the holistic, step-by-step [Prisma Framework tutorial](https://github.com/prisma/prisma2/blob/master/docs/tutorial.md)
- Check out the [Prisma Framework docs](https://github.com/prisma/prisma2) (e.g. for [data modeling](https://github.com/prisma/prisma2/blob/master/docs/data-modeling.md), [relations](https://github.com/prisma/prisma2/blob/master/docs/relations.md) or the [Prisma Client API](https://github.com/prisma/prisma2/tree/master/docs/prisma-client-js/api.md))
- Share your feedback in the [`prisma2-preview`](https://prisma.slack.com/messages/CKQTGR6T0/) channel on the [Prisma Slack](https://slack.prisma.io/)
- Create issues and ask questions on [GitHub](https://github.com/prisma/prisma2/)
- Track Prisma 2's progress on [`isprisma2ready.com`](https://isprisma2ready.com)

