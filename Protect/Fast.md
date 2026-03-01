# FastAPI Interview Map

---

## 1. Python Runtime & ASGI Foundation

### CPython & GIL
- FastAPI 基於 CPython
- GIL（Global Interpreter Lock）確保同時間只有一個 thread 執行 Python bytecode
- 適合 I/O-bound，不適合 CPU-bound

📚 Source:
- https://docs.python.org/3/glossary.html#term-global-interpreter-lock
- Python 官方文件

---

### Asyncio Event Loop
- Python 使用 asyncio event loop
- 基於協程（coroutine）
- 使用 async / await
- 任務切換發生在 await I/O 時

📚 Source:
- https://docs.python.org/3/library/asyncio.html

---

### ASGI
- Asynchronous Server Gateway Interface
- 取代 WSGI
- 支援 async request handling
- FastAPI 建立於 Starlette（ASGI framework）

📚 Source:
- https://asgi.readthedocs.io/
- https://www.starlette.io/

---

### Uvicorn
- ASGI server
- 使用 uvloop + httptools
- 高效能事件驅動

📚 Source:
- https://www.uvicorn.org/

---

## 2. FastAPI Core Architecture

### Path Operation
- 使用 decorator 定義 API
- @app.get(), @app.post()
- 自動綁定 path + method

📚 Source:
- https://fastapi.tiangolo.com/tutorial/path-operation-decorators/

---

### Dependency Injection
- 使用 Depends()
- 函式式 DI（非 class-based）
- 支援 nested dependencies

📚 Source:
- https://fastapi.tiangolo.com/tutorial/dependencies/

---

### Request Validation
- 使用 Pydantic
- 自動資料驗證與型別轉換

📚 Source:
- https://docs.pydantic.dev/

---

### Response Model
- 使用 response_model
- 自動過濾回傳欄位
- 強制輸出格式

📚 Source:
- FastAPI 官方文件

---

## 3. Execution Flow

Request Flow:
ASGI Server → Middleware → Dependency Injection → Path Operation → Response Model → JSON Serialization

---

### Middleware
- 基於 Starlette
- 用於 logging、CORS

📚 Source:
- https://fastapi.tiangolo.com/tutorial/middleware/

---

### Exception Handling
- HTTPException
- 可自定義 exception handler

📚 Source:
- https://fastapi.tiangolo.com/tutorial/handling-errors/

---

### Background Tasks
- 使用 BackgroundTasks
- 適合輕量 async 任務

📚 Source:
- FastAPI 官方文件

---

## 4. Advanced Concepts

### Async vs Sync Endpoint
- async def → 非阻塞 I/O
- def → threadpool 執行

📚 Source:
- FastAPI async docs

---

### Lifespan Events
- startup
- shutdown
- 用於初始化 DB、Cache

📚 Source:
- FastAPI lifespan events

---

### WebSocket
- 支援 WebSocket endpoint
- 基於 ASGI

📚 Source:
- FastAPI WebSocket docs

---

### Custom Dependency
- 可建立 reusable dependency
- 可控制 scope

📚 Source:
- FastAPI dependency docs

---

## 5. Performance & Production

### Uvicorn + Gunicorn
- Gunicorn 管理多 worker
- Uvicorn worker 執行 async app

📚 Source:
- Uvicorn + Gunicorn 官方文件

---

### Workers
- 多 process 模式
- 避免 GIL 限制

📚 Source:
- Gunicorn docs

---

### Caching
- Redis
- fastapi-cache

📚 Source:
- fastapi-cache GitHub

---

### Rate Limit
- slowapi
- Redis-based

📚 Source:
- slowapi GitHub

---

### Memory & CPU-bound Task
- CPU-bound 任務建議使用:
  - Celery
  - RQ
  - Multiprocessing

📚 Source:
- Celery 官方文件

---

## 6. Database Layer

### SQLAlchemy
- ORM
- 支援 async engine

📚 Source:
- https://docs.sqlalchemy.org/

---

### Tortoise ORM
- Async ORM
- 類似 Django ORM

📚 Source:
- Tortoise ORM docs

---

### Transaction
- 使用 session.begin()
- async with session.begin()

📚 Source:
- SQLAlchemy docs

---

### N+1 Problem
- 使用 eager loading
- selectinload()

📚 Source:
- SQLAlchemy loading docs

---

## 7. Security

### OAuth2
- OAuth2PasswordBearer
- Token-based authentication

📚 Source:
- FastAPI security docs

---

### JWT
- python-jose
- PyJWT

📚 Source:
- FastAPI security tutorial

---

### CORS
- CORSMiddleware

📚 Source:
- FastAPI middleware docs

---

### Password Hashing
- passlib
- bcrypt

📚 Source:
- FastAPI security docs

---

## 8. Testing

### TestClient
- 基於 Starlette
- 使用 pytest

📚 Source:
- FastAPI testing docs

---

### Dependency Override
- app.dependency_overrides

📚 Source:
- FastAPI testing docs

---

### e2e Testing
- HTTPX
- pytest-asyncio

📚 Source:
- HTTPX docs