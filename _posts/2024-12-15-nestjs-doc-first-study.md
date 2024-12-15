# NestJS

This is my notes from the official doc

[Ref](https://nestjs.com/)

## What is NestJS?

Node.js web app framework (open source MIT)

## Introduction

NestJS uses many frameworks, one of them is `Express` by default

Why not just use `Express`?
- NestJS provides abstraction (opinionated)! (like Next.js to React)
- NestJS provides **architecture**! (similar to Angular's)
- Built-in TypeScript
- Built-in code eslint & prettier (from CLI scaffolding only)

Note:
- eslint -> linter -> no unused imports, ...
- prettier -> formatter -> remove trailling commas, ...

## Installation

Req
- Node.js (version >= 16)

Use `Nest CLI` or clone a `starter project`

Scaffold project with Nest CLI (recommended)

## First steps

Here we are making CRUD app (TypeScript)

Setup new NestJS project
- Create src dir
  - `app.controller.spec.ts`
  - `app.controller.ts`
  - `app.module.ts`
  - `app.service.ts`
  - `main.ts`
```bash
npm i -g @nestjs/cli
nest new my-project
```

service: has methods
controller: has routes
spec: unit test
module: root
main: entry (creates NestJS instance, HTTP server, insert to port)

main.ts
```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

Run the app
```bash
npm run start
```

Try to request here `http://localhost:3000/`

Run the app with deamon (like nodemon)
```bash
npm run start:dev
```

Has built-in linting and formatting
Has IDE integration with official extensions
Or manual calls
```bash
# Lint and autofix with eslint
$ npm run lint

# Format with prettier
$ npm run format
```

## Controllers

TODO: Read this!

What does it do?
- The router (end points)

Shortcut: CRUD controller + validation
```bash
nest g resource [name]
```
```ts
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

Why decorators?
- Decorators associate classes with required metadata
  - Allows Nest to create a routing map (tie requests to the corresponding controllers)

Explanations:
- @Controller() defines controller
- 'cats': **path prefix**, set this controller to handle
  - cats/id
  - cats/login
