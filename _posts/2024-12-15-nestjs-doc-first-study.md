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
- Built-in installed eslint & prettier (from CLI scaffolding only)

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

service: has method logics e.g. get data
controller: has routes
spec: unit test
module: root
main: entry (creates NestJS instance, start HTTP server, insert to port)

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

Note: Pass a type to create method to access either Express / Fastify API

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
Or manual script calls
```bash
# Lint and autofix with eslint
$ npm run lint

# Format with prettier
$ npm run format
```

## Controllers

What does it do?
- Routes (end points)

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

Shortcut: Make 1 controller
```bash
nest g controller [name]
```

Why decorator?
- Decorator binds class with metadata
  - Lets Nest bind request to controller (routing map)

Note: You can make your own decorators

Anatomy:
- `@Controller()`: defines controller
- **Path prefix**: group routes, e.g. `cats` for /cats related endpoints
- `@Get`: Maps req -> endpoint route -> handler, e.g. `@Get('breed')` maps req -> `GET /cats/breed` -> `findAll()` handler
- `findAll()`: the function method

`GET /cats` res 200
1. Can you built-in standard res (recommended) (Gets to use NestJS features like intercceptios, `@HttpCode()` / `@Header()`)
2. Can use lib res obj (e.g. Express) (`@Res()` or just `@Response()`)
  - Import `@types/express` when using them
  - Use `res.json()` or `res.send()`

1. Default NestJS built-in methods
  - JavaScript obj / array are serialized to JSON in res
  - JavaScript primitives are as is in res
  - 200 default
    - POST 201 default
   
Request obj
- Client req uses choosen platform's (Express)
- Inject req with `@Req()` decorator to handler signature
- Req has access to
  - query string (made of query params = `?search=John&sort=asc`) (`@Query()`)
  - route param (`user/id`) (`@Param()`)
  - header (`@Header()`)
  - body (`@Body()`)
```ts
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```

Note: In order to take advantage of express typings (as in the request: Request parameter example above), install @types/express package

This is a post handler, POST records

```ts
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

Here are the decorators you use to define endpoints
- @Get()
- @Post()
- @Put()
- @Delete()
- @Patch()
- @Options()
- @Head()
- @All() - this one maps to all HTTP methods

Can you pattern in routes (e.g. anything that starts with `ab` and ends with `cd`)
```ts
@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```

Status code
All defaults 200, Post 201
Use `@HttpCode()` to be explicit (import from `@nestjs/common`) (or lib specific like Express)
```ts
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

Custom header res, use `@Header()` (or lib sprcific like Express res.header())
```ts
@Post()
@Header('Cache-Control', 'no-store')
create() {
  return 'This action adds a new cat';
}
```

Redirect res to URL, use `@Redirect()` (or lib sprcific like Express res.redirect())
```ts
@Get()
@Redirect('https://nestjs.com', 301)
```

Not too interested in redirect, so I skip from here

Route param (user/id)
**Route with param should be declared AFTER statics**
Use `route param token` in `path`, e.g. use route param token in `@Get()`
- To access it in handler use `@Param()` in method signature (import `Param` from the `@nestjs/common`)
```ts
@Get(':id')
findOne(@Param() params: any): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
```
- Or you can be more specific which token you want
```ts
@Get(':id')
findOne(@Param('id') id: string): string {
  return `This action returns a #${id} cat`;
}
```

There is subdomain mapping also for endpoints but I skip that too since I am not interested


Note: NestJS have connection pool database, singleton services with global state
- Fully safe in NestJS app

Asynchronicity
- NestJS supports this
- Return `Promise`, returns deferred value that NestJS resolves for you
```ts
@Get()
async findAll(): Promise<any[]> {
  return [];
}
```
- Handlers return RxJS observable streams. Nest auto sub to source & take the last emitted value when stream is completed (use the above or this one its up to you)
```ts
@Get()
findAll(): Observable<any[]> {
  return of([]);
}
```

Request payloads
Accepting client param, use `@Body()`
1. Define Data Transfer Object (DTO): the HOW data is sent over network (TypeScript only)
2. Use
  - TypeScript interface
  - Classes (recommended: ES6 standard and preserved as entities in compiled JavaScript, can refer at runtime! Important for `Pipes` feature later)
    - `Pipes` feature like `ValidationPipe`, filters illegal props outside DTO (strip / throw, you pick later)
```ts
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```
3. Use DTO in handler signature
```ts
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
```

Full resource sample
```ts
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

Binding controller (no need if you had made this with CLI generate! That thing auto bind as in auto edit your import code!)
Need to tell NestJS that we have made this controller
- modules owns controllers
  - For this example we add this controller to root module
```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}
```
Attach metadata with`@Module()` to class (define module)

There is library approach to handle res, I am not interested so I skip that from here

## Providers

What is provider?
- Controller pass complex logic to providers
- Class to be injected as dependency
  - Service
  - Repository
  - Factory
  - Helper
 
This service is for (this is used by controller)
- Data storage
- Retrieval 
```ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```
This has
- 1 prop
- 2 methods

Cats interface being used in example
```ts
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

Anatomy
- `@Injectable()`: attaches metadata, declare this class is managed by Nest IoC container

Shortcut: Make 1 service
```bash
nest g service cats
```

Use service in controller via dependency injection, use it to get data
- Inject via constructor (private) (declare and init in 1 line)
- Dependencies are resolved via type
  - Resolve `catsService` by returning instance `CatsService` (if singleton, return existing instance)
```ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

Scopes
- Providers have scope (lifetime) sync with app lifecycle
- On app bootstrap
  - Every dependencies (providers) must be resolved (instantiated) - then on shut down also all must be destroyed

Note: There are ways to make it request scoped but I am skipping that, not interested

Custom providers
NestJS has inversion of control (IoC) container
- Resolves relations between providers
- You can define provider
  - Use values
  - Classes
  - Async / sync factories
 
Optional providers
If class have optional dependency (ok if its not resolved)
Not interested, skip

Property based injection
The ones above are constructor injection, you can also do property based injection
Not interested, skip

Provider registration
- Now, we have provider (servide) & consumer (controller)
- Register service (in app.module root for now) with NestJS (so it can inject / resolve dependencies)

Project dir
.
└── src/
    ├── cats/
    │   ├── dto/
    │   │   └── create-cat.dto.ts
    │   ├── interfaces/
    │   │   └── cat.interface.ts
    │   ├── cats.controller.ts
    │   └── cats.service.ts
    ├── app.module.ts
    └── main.ts

Manual instantiation
- Nest handle dependencies resolutions
- You can manually get / instantiate providers!
  - Not interested, skip
 
## Module
- Modules are by defaults
  - Singletons!
  - Shared! (reuse in other modules)
- You define module with class + `@Module()`, that decorator adds metadata to that class
- App needs at least 1 module (root module)
  - Root module: starting point for app graph (node tree)
    - NestJS use app graph: its data structure to resolve relations and dependencies (modules & providers)
   
Recommended: use module to organize components! Encapsulate features!
- A module prop
  - providers
  - controllers
  - imports: lists of other modules (can import their exported providers here!)
  - exports: lists of my providers that others can import (token or provider itself)

Note:
- One lego block can be simple, or complex and use other blocks (just like my game primitives and actors! But only 1 scene)
- But this is like the default Next.js pattern, where each route has a dir to hold its features. Then other route can import from other route
- Module is a component (app domain) (lego block), has
  - Controller (routing) (e.g. GET /cats/breed/1, POST /...)
  - Providers (logic) (findAll()...) (aka module's public API)

So keep 1 feature in 1 module in 1 dir
- Manage complexity

Create CatModule
- Register this module to root module
```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

Shortcut: Make 1 module
```bash
nest g module cats
```

The project dir now
.
└── src/
    ├── cats/
    │   ├── dto/
    │   │   └── create-cat.dto.ts
    │   ├── interfaces/
    │   │   └── cat.interface.ts
    │   ├── cats.controller.ts
    │   ├── cats.service.ts
    │   └── cats.module.ts
    ├── app.module.ts
    └── main.ts

Share CatsService with other modules
- Export it
  - Add to its module exports list
    - GOOD: If others import this module, they have access to this service! (same instance too)
      - So make prisma module, that others import, that way the service instance is autoload!  
    - BAD: Do not register a service in each module one by one, that creates multi instances! hogs mem
```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

Module re-exporting
If module import a provider, that same module can export it again from itself
Not interested, skip

Dependency injection
Module cannot be injected as provider (only to be registered to root module)
Can inject provider to module for **configuration purposes**
```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

Global modules
- Can make modules global everywhere accessible
- Make sets of providers be available everywhere (e.g. helpers, database connections)
- Register global modules once only by root module
- **Modules that needs `CatsService` does not need to import `CatsModule` in their module import array**
```ts
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

Note to self worth trying in future: Try this way, just have 1 module for routing the whole back-end, then have features dirs that holds services, then each endpoint can reuse said services when needed
- We can adjust a bit, so have 1 root module, that have smaller other modules just for the sake of end point organization
  - But the features are all going to be global modules (use module to organize similar feature), does not do routing at all, where they are meant to be used by any end points as needed
  - Or maybe each feature module is not global, that way it is easy to see which endpoint use which module right

Dynamic modules
Create module where provider are
- Registered dynamically
- Configure dynamically
- Example, DatabaseModule (example of feature module, has connection feature and others who import me can use it)
- Use `forRoot()` to return dynamic module, either
  - Sync
  - Async (Promise)
    - The prop returned (from dynamic module, dynamic repository providers) extend base module (@Module() only have connection provider)
```ts
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
  exports: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

Can make the above global if you want
```ts
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

This is how you import it (use this module feature in an end point dedicated module)
```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

I am not doing this but just in case, this is how to re export it
```ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```

## Middleware

This is a function
- Called first, before router (any other modules's controller!)
- Middleware has access to req and res just like controller router endpoint
- Also have `next()`
- Same as Express middleware

Note
- With Express platform
  - `json`, `urlencoded` & `body-parser` are default middleware
  - can customize it with `MiddlewareConsumer`
    - turn off global middleware, set `bodyParser` to false in `NestFactory.create()`

What does it do?
- Run any code
- Change req and res between client and module's controllers
- Can end req res cycle
- Call next middleware in stack
- **If this middleware func does not end req res cycle, must call `next()` to pass to next middleware stack, or req hangs.**

How to use it?
- Function
- Class + `@Injectable`, implement the `NestMiddleware` interface
```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

Dependency injection
Fully supports dependency injection like providers & controllers
- Can inject dependencies in the same module via constructor

Applying middleware
Apply in module using `configure()` (can be async inside it), module with interface must use `NestModule` interface
Set up `LoggerMiddleware` in `AppModule` level
- for `/cats` route in `CatsController`
```ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```

Restrict to a req method
- Pass obj with router path and method to `forRoutes()`
```ts
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```
Note, the path value can use wilcard pattern
- match
  - abcd
  - ab ... cd
```ts

forRoutes({
  path: 'ab*cd',
  method: RequestMethod.ALL,
});

```

Middleware consumer
Helper class, has methods to manage middleware, chainable, `fourRoutes()` can take
- single str
- multi str
- `RouteInfo` obj
- controller class
- multi controller class

Most often you pass multi controller with comma delimiters
```ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes(CatsController);
  }
}
```
Note
- `apply()` may take single / multi middlewares

Excluding routes
Use `exclude()`, it takes
- single str
- multi str
- `RouteInfo`
- wildcard also
```ts
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST },
    'cats/(.*)',
  )
  .forRoutes(CatsController);
```
Here, this middleware is binded to all route in `CatsController` except the one in `exclude()`

Functional middleware
if middleware does not have
- many methods
- dependencies
  - then just use functional middleware (recommended)
```ts
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```

this is how to use it in root `AppModule`
```ts
consumer
  .apply(logger)
  .forRoutes(CatsController);
```

Multiple middleware
need sequential middleware? just list with comma delimiter
```ts
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);
```

Global middleware
Bind 1 middleware to all routes? use `use()` from `INestApplication`
```ts
const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(process.env.PORT ?? 3000);
```

Note (not sure what this is but might be important)
Accessing the DI container in global middleware is not possible
- use `functional middleware` in `app.use()`
- or use class middleware and consume it with `.forRoutes('*')` with `AppModule`

Exception filters
This catch all unhandled exceptions in app
- sends user friendly res, using `global exception filter`, handles
  - `HttpException` and its sub class
    - Exception unrecognized? (not `HttpException` or its sub class)
      - built-in generates JSON res
```ts
{
  "statusCode": 500,
  "message": "Internal server error"
}
```
Note
- Partial support for `http-errors`, throw exceptions with `statusCode` + `message` will be used instead of `InternalServerErrorException`
TODO
