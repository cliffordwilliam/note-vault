# Prisma

This is my notes on the official doc

[Ref](https://www.prisma.io)

## What is Prisma

Company that provides tools to work with databases
- Connection pooling
- Caching
- More...

I am interested in their open source Node.js and TypeScript ORM
- Readable data model
- Automated migrations
- Type-safety
- Auto-completion

[Ref](https://www.prisma.io/orm)

## Setup (relational PosgreSQL)

[Ref](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql)

Learn how to
1. Create a new Node.js from scratch
2. Connecting Prisma ORM to your database
3. Generating a Prisma Client for database access

We will use
- Prisma CLI
- Prisma Client
- Prisma Migrate

Req
- Node.js	18.8 / 20.9 / 22.11
- PostgreSQL server running locally

For more detailed system requirements, read this
[Ref](https://www.prisma.io/docs/orm/reference/system-requirements)

Install Prisma CLI as a development dependency
```bash
npm install prisma --save-dev
```

Use Prisma CLI to
- Create `.env` file in root
  - Database connection
- Create `prisma` dir
  - `schema.prisma`
    - Database connection
    - Schema models
```bash
npx prisma init
```

Note, .env should have
```.env
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

Here is PostgreSQL `connection URL` format
```bash
postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=SCHEMA
```
- `USER`: The name of your database user
- `PASSWORD`: The password for your database user
- `HOST`: The name of your host name (for the local environment, it is localhost)
- `PORT`: The port where your database server is running (typically 5432 for PostgreSQL)
- `DATABASE`: The name of the database
- `SCHEMA`: The name of the schema inside the database

## Connect your database

Set `url` with `connection URL` from `.env` in Prisma schema
```bash
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

## Using Prisma Migrate

Migrate: Use schema to make tables

Define model in Prisma schema
```prisma
model Post {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  title     String   @db.VarChar(255)
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
}

model Profile {
  id     Int     @id @default(autoincrement())
  bio    String?
  user   User    @relation(fields: [userId], references: [id])
  userId Int     @unique
}

model User {
  id      Int      @id @default(autoincrement())
  email   String   @unique
  name    String?
  posts   Post[]
  profile Profile?
}
```

Use Prisma CLI to
1. Create migration file
2. Runs migration file
```bash
npx prisma migrate dev --name init
```

Note: When still in development stage, just use push, skip migration

Note: `generate` is called under the hood by default, after running `prisma migrate dev`. If `prisma-client-js` generator is defined in your schema, this will check if `@prisma/client` is installed and install it if it's missing.

## Install Prisma Client

Install `@prisma/client` with npm as dependency
```bash
npm install @prisma/client
```
This install command also runs `prisma generate`
- Generate Prisma Client with schema

Protocol
1. Update Prisma schema
2. `prisma migrate dev` / `prisma db push` to update database (these 2 commands runs `generate` automatically for you!)
3. At this point Prisma Client is regenerated!

## Querying the database

You ask question with the Prisma Client

Note: DO NOT DO IT THIS WAY, MAKE SINGLETON OF THE CLIENT INSTANCE INSTEAD!

This example is for testing only, create an instance for 1 question to ask for all `User` from database and print it! Should be empty since we did not seed.
```js
const { PrismaClient } = require('@prisma/client')

const prisma = new PrismaClient()

async function main() {
  // ... you will write your Prisma Client queries here
  const allUsers = await prisma.user.findMany()
  console.log(allUsers)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

## Write data into database

Here are queries (questions) to write new records (rows) in the following tables
- `Post`
- `User`
- `Profile`
  - Using nested write query, since `User` is owned by `Post` and `Profile`
  - Use `include` option in `findMany` to include the `User` owners objects
```js
async function main() {
  await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@prisma.io',
      posts: {
        create: { title: 'Hello World' },
      },
      profile: {
        create: { bio: 'I like turtles' },
      },
    },
  })

  const allUsers = await prisma.user.findMany({
    include: {
      posts: true,
      profile: true,
    },
  })
  console.dir(allUsers, { depth: null })
}
```

The expected output
```bash
[
  {
    email: 'alice@prisma.io',
    id: 1,
    name: 'Alice',
    posts: [
      {
        content: null,
        createdAt: 2020-03-21T16:45:01.246Z,
        updatedAt: 2020-03-21T16:45:01.246Z,
        id: 1,
        published: false,
        title: 'Hello World',
        authorId: 1,
      }
    ],
    profile: {
      bio: 'I like turtles',
      id: 1,
      userId: 1,
    }
  }
]
```

User
| id | email             | name    |
|----|-------------------|---------|
| 1  | "alice@prisma.io" | "Alice" |

Post
| id | createdAt                | updatedAt                | title         | content | published | authorId |
|----|--------------------------|--------------------------|---------------|---------|-----------|----------|
| 1  | 2020-03-21T16:45:01.246Z | 2020-03-21T16:45:01.246Z | "Hello World" | null    | false     | 1        |

Profile
| id | bio              | userId |
|----|------------------|--------|
| 1  | "I like turtles" | 1      |

---

This is how to update
```js
async function main() {
  const post = await prisma.post.update({
    where: { id: 1 },
    data: { published: true },
  })
  console.log(post)
}
```

The expected output
```bash
{
  id: 1,
  title: 'Hello World',
  content: null,
  published: true,
  authorId: 1
}
```

Post
| id | createdAt                | updatedAt                | title         | content | published | authorId |
|----|--------------------------|--------------------------|---------------|---------|-----------|----------|
| 1  | 2020-03-21T16:45:01.246Z | 2020-03-21T16:45:01.246Z | "Hello World" | null    | true      | 1        |

## Next steps

- Explore Prisma Client API?
- Build app with Prisma ORM?
  - Laterst tutorials about Prisma ORM
- Prisma Studio?
  - Visual editor for your database data, run `npx prisma studio` to run it on a port
- Prisma Optimize provides recommendations to make queries faster!
- Prisma ORM example
  - Next.js + REST API
  - Next.js + GraphQL API
  - GraphQL server + `@apollo/server`
  - Simple REST API with Express.JS
  - Simple gRPC API

[Ref](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases/next-steps)
