# NestJS Interview Map

---

## 1. Node.js Runtime

### V8
- V8 是 Google 開發的 JavaScript 引擎，負責將 JS 編譯為機器碼（JIT compilation）
- 使用 Hidden Class + Inline Cache 優化物件存取
- 採用 Mark-and-Sweep Garbage Collection

📚 Source:
- https://v8.dev/docs
- https://nodejs.org/en/docs/guides/

---

### Event Loop
- Node.js 使用單執行緒 event loop
- 透過非阻塞 I/O 提升併發能力
- Event Loop phases:
  - timers
  - pending callbacks
  - idle, prepare
  - poll
  - check
  - close callbacks
- Microtask (Promise, process.nextTick) 優先於 macrotask

📚 Source:
- https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop

---

### libuv
- Node.js 的跨平台 I/O 函式庫
- 負責：
  - File system async
  - Network I/O
  - Thread pool 管理
- 底層使用 epoll (Linux)、kqueue (macOS)、IOCP (Windows)

📚 Source:
- https://libuv.org/
- Node.js 官方文件

---

### Thread Pool
- 預設 4 個 threads（可透過 UV_THREADPOOL_SIZE 調整）
- 用於：
  - fs
  - crypto
  - dns
  - zlib

📚 Source:
- Node.js docs - libuv threadpool

---

### Async Model
- 非阻塞 I/O
- Callback → Promise → async/await
- 透過事件驅動架構避免 thread-per-request

📚 Source:
- Node.js 官方文件

---

## 2. NestJS Core Architecture

### Module
- 應用程式的組織單位
- 使用 @Module decorator
- 管理 providers、controllers、imports、exports

📚 Source:
- https://docs.nestjs.com/modules

---

### Controller
- 負責處理 HTTP request
- 使用 @Controller, @Get, @Post 等 decorator

📚 Source:
- https://docs.nestjs.com/controllers

---

### Provider
- 可被注入的服務（Service）
- 透過 DI 容器管理生命周期

📚 Source:
- https://docs.nestjs.com/providers

---

### Dependency Injection
- 基於 TypeScript metadata + reflect-metadata
- Constructor-based injection
- 支援 Singleton / Request / Transient scope

📚 Source:
- https://docs.nestjs.com/fundamentals/custom-providers

---

### Decorator
- 使用 TypeScript decorator
- 本質為 metadata 註冊
- 透過 Reflect API 讀取

📚 Source:
- https://www.typescriptlang.org/docs/handbook/decorators.html

---

## 3. Execution Flow

Request Flow:
Middleware → Guard → Interceptor (before) → Pipe → Controller → Service → Interceptor (after) → Exception Filter

---

### Middleware
- Express 層級
- 用於 logging、修改 request

📚 Source:
- https://docs.nestjs.com/middleware

---

### Guard
- 控制是否進入 route
- 常用於 Authorization

📚 Source:
- https://docs.nestjs.com/guards

---

### Interceptor
- 可攔截 request / response
- 用於 logging、transform response

📚 Source:
- https://docs.nestjs.com/interceptors

---

### Pipe
- 資料轉換與驗證
- 常搭配 class-validator

📚 Source:
- https://docs.nestjs.com/pipes

---

### Exception Filter
- 全域錯誤處理
- 可自定義錯誤格式

📚 Source:
- https://docs.nestjs.com/exception-filters

---

## 4. Advanced Concepts

### Custom Provider
- useClass
- useValue
- useFactory
- useExisting

📚 Source:
- NestJS docs - custom providers

---

### Dynamic Module
- 透過 static register() 動態設定 module
- 常用於 DB 或第三方整合

📚 Source:
- https://docs.nestjs.com/fundamentals/dynamic-modules

---

### Request Scope
- 每 request 建立新 instance
- 會影響效能

📚 Source:
- NestJS docs - injection scopes

---

### Circular Dependency
- 使用 forwardRef()
- 或重構架構

📚 Source:
- NestJS docs - circular dependency

---

### Reflect Metadata
- NestJS DI 基礎
- 透過 reflect-metadata 讀取 constructor 參數型別

📚 Source:
- reflect-metadata GitHub

---

## 5. Performance & Production

### Clustering
- 使用 Node.js cluster module
- 或 PM2 cluster mode

📚 Source:
- Node.js cluster docs

---

### PM2
- Process manager
- 支援 cluster、auto restart

📚 Source:
- https://pm2.keymetrics.io/

---

### Caching
- 使用 CacheModule
- Redis 常見

📚 Source:
- NestJS caching docs

---

### Rate Limit
- @nestjs/throttler

📚 Source:
- NestJS throttler

---

### Logging
- 使用 Logger service
- 或整合 Winston

📚 Source:
- NestJS logger docs

---

### Memory Leak
- 常見原因：
  - 未清除 event listener
  - Global cache
- 使用 heap snapshot 分析

📚 Source:
- Node.js memory debugging docs

---

## 6. Database Layer

### TypeORM vs Prisma
TypeORM:
- ORM
- Decorator-based

Prisma:
- Schema-first
- 型別安全
- 效能較佳

📚 Source:
- 官方文件

---

### Transaction
- TypeORM QueryRunner
- Prisma $transaction()

📚 Source:
- ORM 官方文件

---

### Repository Pattern
- 封裝資料存取邏輯
- 提升可測試性

---

### N+1 Problem
- GraphQL 常見
- 使用 DataLoader 解決

📚 Source:
- GraphQL docs

---

## 7. Security

### JWT
- Stateless authentication
- 使用 @nestjs/jwt

---

### Passport
- Strategy-based authentication
- 支援 local / jwt / oauth

---

### CORS
- 控制跨域

---

### Helmet
- HTTP header security

📚 Source:
- NestJS security docs

---

## 8. Testing

### Unit Test
- Jest
- 測試 service

---

### e2e
- Supertest
- 測試完整 request flow

---

### Mocking Provider
- 使用 TestingModule
- overrideProvider()

📚 Source:
- https://docs.nestjs.com/fundamentals/testing