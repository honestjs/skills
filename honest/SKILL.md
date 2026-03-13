---
name: honest
description: Build and maintain Honest.js (HonestJS) applications - a Nest-style framework on Hono. Use when the user works with Honest, HonestJS, honestjs, or when building TypeScript web apps with controllers, modules, dependency injection, guards, pipes, or filters on Hono. Covers CLI scaffolding, Application.create, decorators, routing, DI, application context (registry), plugins, @honestjs/middleware, @honestjs/pipes, @honestjs/class-validator-pipe, @honestjs/rpc-plugin, @honestjs/api-docs-plugin, and http-essentials.
---

# Honest Skill

Build Honest.js apps with CLI, decorators, modules, and DI. Honest is a
TypeScript-first web framework on Hono with a Nest-like architecture.

## Setup

### Using Honest CLI

```bash
bun add -g @honestjs/cli
honestjs new my-project   # aliases: honest, hnjs
cd my-project
bun dev
```

### Manual setup

```bash
bun add honestjs hono reflect-metadata
```

Bootstrap (entry file):

```typescript
import "reflect-metadata";
import { Application } from "honestjs";
import { AppModule } from "./app.module";

const { app, hono } = await Application.create(AppModule, {
  routing: { prefix: "api", version: 1 },
});

export default hono;
```

Always import `reflect-metadata` once before any Honest decorators. Export the
`hono` instance for your server (e.g. Cloudflare Workers, Node, Bun).

## CLI

- **New project:** `honestjs new <project-name>` — options: `-t|--template`,
  `-p|--package-manager`, `--typescript`, `--eslint`, `--prettier`, `--docker`,
  `--git`, `--install`, `-y|--yes`
- **List templates:** `honestjs list` — `-j|--json`, `-c|--category`, `-t|--tag`
- **Info:** `honestjs info`
- **Generate:** `honestjs generate <schematic> <name>` (alias `g`) — schematics:
  `controller`|`c`, `service`|`s`, `module`|`m`, `view`|`v`, `middleware`|`c-m`,
  `guard`|`c-g`, `filter`|`c-f`, `pipe`|`c-p`. Options: `-p|--path`, `--flat`,
  `--skip-import`, `--export`

Prefer `honestjs new` for new apps and `honestjs generate` for adding
controllers, services, modules, guards, pipes, or filters.

## Application and routing

- **Create app:** `Application.create(RootModule, options)` returns
  `{ app, hono }`.
- **Routing:**
  `routing: { prefix?: string, version?: number | VERSION_NEUTRAL | number[] }`
  — e.g. `prefix: 'api'`, `version: 1` → `/api/v1/...`.
- **Global components:**
  `components: { middleware?, guards?, pipes?, filters? }` — applied to all
  routes.
- **Plugins:** `plugins?: PluginEntry[]` — each entry can be a plain plugin or
  `{ plugin, preProcessors?, postProcessors? }`. Processors receive
  `(app, hono, ctx)` (ctx = app context). Order: preProcessors →
  `beforeModulesRegistered`; `afterModulesRegistered` → postProcessors.
- **Custom handlers:** `onError?`, `notFound?` on options.
- **Hono access:** `app.getApp()` for the underlying Hono instance;
  `app.getRoutes()` for route info.
- **Application context (registry):** `app.getContext()` — app-scoped key-value
  store for the whole app (bootstrap, services, any code with `app`). Use
  `get<T>(key)`, `set<T>(key, value)`, `has(key)`, `delete(key)`, `keys()`.
  Namespace keys (e.g. `app.config`, `rpc.artifact`, `openapi.spec`). Use for pipeline/config or
  shared data that outlives a request. **Not** Hono request context: that is
  per-request and injected via `@Ctx()` (request, response, env, request-scoped
  variables).

## Modules and DI

- **Module:** `@Module({ controllers?, services?, imports? })` — list
  controller/service classes and imported modules.
- **Service:** `@Service()` — marks a class as injectable singleton; inject via
  constructor in controllers or other services.

```typescript
@Module({
  controllers: [UsersController],
  services: [UsersService],
  imports: [OtherModule],
})
class UsersModule {}
```

## Controllers and routes

- **Controller:** `@Controller(route?)` — base path for all handlers in the
  class.
- **HTTP methods:** `@Get(path?)`, `@Post(path?)`, `@Put(path?)`,
  `@Delete(path?)`, `@Patch(path?)`, `@Options(path?)`, `@All(path?)` — path is
  optional (default `''`).

**Parameter decorators:** use on handler arguments.

| Decorator                        | Purpose                                               |
| -------------------------------- | ----------------------------------------------------- |
| `@Body(key?)`                    | Request body (JSON); optional key to get one property |
| `@Param(name?)`                  | Route param(s)                                        |
| `@Query(name?)`                  | Query param(s)                                        |
| `@Header(name?)`                 | Header(s)                                             |
| `@Req()` / `@Request()`          | Hono request                                          |
| `@Res()` / `@Response()`         | Hono response                                         |
| `@Ctx()` / `@Context()`          | Hono context                                          |
| `@Var(name)` / `@Variable(name)` | Context variable                                      |

Example:

```typescript
@Controller("users")
class UsersController {
  @Get(":id")
  getOne(@Param("id") id: string, @Ctx() ctx: Context) {
    return ctx.json({ id });
  }

  @Post()
  async create(@Body() body: CreateUserDto) {
    return { created: body };
  }
}
```

## Pipeline

Apply at class or method level:

- **UseMiddleware(...middleware)** — runs before the handler.
- **UseGuards(...guards)** — determines if the request is allowed.
- **UsePipes(...pipes)** — transform input before the handler.
- **UseFilters(...filters)** — handle exceptions for the controller/method.

Same pattern as Nest: class-level applies to all methods; method-level overrides
or adds.

## Ecosystem

### @honestjs/rpc-plugin

```bash
bun add @honestjs/rpc-plugin
```

- **Typed RPC client** — analyzes HonestJS controllers and generates a type-safe
  TypeScript client. Register: `plugins: [RPCPlugin]` or
  `plugins: [new RPCPlugin(options)]`.
- **Options:** `controllerPattern` (glob for controller files), `tsConfigPath`,
  `outputDir` (default `./generated/rpc`), `generateOnInit` (default `true`),
  `generators` (optional array of custom generators).
  Use `controllerPattern` if controllers live outside the default
  `src/modules/*/*.controller.ts`.
- **Generators:** when `generators` is omitted, plugin uses built-in
  `TypeScriptClientGenerator`; when defined, only provided generators run.
- **Generated client:** `ApiClient` with controller-namespaced methods; call
  with `{ params?, query?, body?, headers? }`. Use
  `new ApiClient(baseUrl, { fetchFn? })` for custom fetch (testing, retries,
  interceptors). `setDefaultHeaders()` for auth. Errors via `ApiError`.
- **Manual generation:** `generateOnInit: false` then
  `await rpcPlugin.analyze()` when needed. Controllers should use `@Body()`,
  `@Param()`, `@Query()` with typed DTOs/interfaces for best client inference.
- **OpenAPI/Swagger:** RPC plugin does not generate OpenAPI specs. It publishes
  routes/schemas artifact to app context (default key: `rpc.artifact`). API Docs
  plugin defaults to that key — use `new ApiDocsPlugin()` with RPC, or pass
  `artifact` for a custom key or direct `{ routes, schemas }` object.

### @honestjs/api-docs-plugin

```bash
bun add @honestjs/api-docs-plugin
```

- **OpenAPI + Swagger UI** — generates OpenAPI spec from an artifact and serves
  JSON + Swagger UI. Register: `plugins: [new ApiDocsPlugin()]` or
  `plugins: [new ApiDocsPlugin(options)]`. With RPC, `artifact` defaults to
  `'rpc.artifact'` so no options are needed.
- **Artifact source:** `artifact` is optional (default: **context key**
  `'rpc.artifact'`). Can pass another context key or a **direct object**
  `{ routes, schemas }`. Put the producer plugin (e.g. RPCPlugin) **before**
  ApiDocsPlugin in the plugins array when using a context key.
- **Options:** `title`, `version`, `description`, `servers` (OpenAPI metadata);
  `openApiRoute` (default `/openapi.json`), `uiRoute` (default `/docs`),
  `uiTitle` (default `'API Docs'`), `reloadOnRequest` (default `false`).
- **Programmatic:** `fromArtifactSync(artifact, options)` and `write(spec, path)`
  for generating spec files; types: `OpenApiArtifactInput`, `OpenApiDocument`,
  `OpenApiGenerationOptions`.

### @honestjs/middleware

```bash
bun add @honestjs/middleware
```

- **Application config:**
  `components: { middleware: [new LoggerMiddleware(), new CorsMiddleware({ origin: '...' }), new SecureHeadersMiddleware()] }`.
- **Wrap Hono middleware:** `new HonoMiddleware(poweredBy())` or
  `new HonoMiddleware(async (c, next) => { ... })`.

### @honestjs/pipes

```bash
bun add @honestjs/pipes
```

- **PrimitiveValidationPipe** — validates and transforms primitive types
  (String, Number, Boolean) on route params/query/body. Register globally:
  `components: { pipes: [new PrimitiveValidationPipe()] }`.

### @honestjs/class-validator-pipe

```bash
bun add @honestjs/class-validator-pipe
```

- **ClassValidatorPipe** — validates and transforms DTOs with class-validator
  and class-transformer. Define DTOs with decorators (`@IsString()`,
  `@IsEmail()`, `@MinLength()`, `@IsOptional()`, etc.), then register:
  `components: { pipes: [new ClassValidatorPipe()] }`. Use `@Body()` with the
  DTO type in handlers.

### http-essentials

```bash
bun add http-essentials
```

- **Status/phrase:** `HttpStatus.OK`, `HttpPhrase.NOT_FOUND`,
  `httpPhraseByStatus`, `httpStatusByPhrase`.
- **Exceptions:** `NotFoundException`, `BadRequestException`,
  `UnauthorizedException`, etc. — throw in handlers or use in filters; include
  default messages and custom message arg.

Use in guards and exception filters for consistent HTTP responses.

## Guidelines

- Use `honestjs new` for new projects; use `honestjs generate` for controllers,
  services, modules, guards, pipes, filters.
- Always `import 'reflect-metadata'` once at entry before any Honest code.
- Export `hono` from the entry used by your server (Worker, Node, Bun).
- Honest is pre-v1: API may change; avoid relying on undocumented behavior.
- For Hono-specific behavior (middleware, adapters), use `app.getApp()` and the
  Hono API.
