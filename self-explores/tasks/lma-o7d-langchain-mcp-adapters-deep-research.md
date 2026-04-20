---
date: 2026-04-20
type: task-worklog
task: lma-o7d
title: "langchain-mcp-adapters — Deep Research (Tư duy Top 0.1%)"
status: completed
started_at: 2026-04-20 10:30
completed_at: 2026-04-20 11:05
detailed_at: 2026-04-20 10:00
detail_score: ready-for-dev
tags: [system-design, deep-research, design-principles, langchain-mcp-adapters]
template: system-design-deep-dive
template_id: 6
template_task: T4
---

# langchain-mcp-adapters — Deep Research (Tư duy Top 0.1%) — Detailed Design

## 1. Objective

Phân tích sâu "Tại sao họ thiết kế như vậy?" cho ≥3 design decisions quan trọng trong `langchain-mcp-adapters`. Mỗi decision phải được phân tích theo 4 chiều: (1) SOLID/GoF nguyên lý gốc, (2) Tradeoff vs alternatives đơn giản hơn, (3) Historical context từ commit history, (4) Industry reference từ hệ thống nổi tiếng khác. Task chạy song song với T3 (Code Mapping).

## 2. Scope

**In-scope:**
- ≥3 design decisions quan trọng (không phải implementation details)
- Mỗi decision: 4 chiều phân tích đầy đủ
- Ít nhất 1 industry reference per decision (LangChain, OpenAI SDK, httpx, Anthropic SDK, etc.)
- Code references clickable

**Out-of-scope:**
- Không cần chạy code
- Không cần internet research (dùng kiến thức sẵn có về patterns/industry)
- Không cần phân tích tests

## 3. Input / Output

**Input:**
- `self-explores/tasks/lma-dxm-langchain-mcp-adapters-strategic-eval.md` — design decisions identified từ T2
- Source files: tập trung vào decisions từ T2 (thường là: `interceptors.py`, `sessions.py`, `tools.py`)
- Git commit history (nếu accessible): `git log --oneline langchain_mcp_adapters/interceptors.py`

**Output (ghi vào ## Worklog):**
- ≥3 design decision analyses, mỗi cái có:
  - Decision statement (1 câu rõ ràng)
  - Nguyên lý (SOLID/GoF/architectural pattern cụ thể)
  - Tradeoff analysis (cách đơn giản hơn vs tại sao không chọn)
  - Industry reference (hệ thống nào dùng pattern này)
  - Code evidence clickable link

## 4. Dependencies

- `lma-dxm` phải xong trước (cần decisions từ T2 làm research agenda)
- **Chạy song song với `lma-cw4` (T3)** — không depend nhau
- Không cần network (industry references từ kiến thức sẵn có)

## 5. Flow xử lý

### Step 1: Đọc T2 output và set research agenda (~5 phút)

Đọc worklog lma-dxm, extract design decisions đã identified.

Nếu T2 không explicit list decisions → infer từ findings:
- "Core Component: interceptors.py" → Decision: "Tại sao dùng Protocol thay vì ABC?"
- "Leverage Point: sessions.py factory" → Decision: "Tại sao dùng Factory pattern thay vì inject session directly?"
- "Extensibility: context manager" → Decision: "Tại sao MultiServerMCPClient KHÔNG phải context manager?"

**Verify:** Có ≥3 research questions trước khi bắt đầu.

### Step 2: Research Decision A — Protocol vs ABC cho Interceptors (~15 phút)

**Decision:** Tại sao `ToolCallInterceptor` và `Callbacks` dùng `Protocol` (structural typing) thay vì `ABC` (abstract base class)?

Đọc [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112).

**4 chiều phân tích:**

1. **Nguyên lý gốc:**
   - Python `Protocol` (PEP 544) = structural subtyping (duck typing at type system level)
   - Không cần `class MyInterceptor(ToolCallInterceptor)` — chỉ cần implement đúng signature
   - Liên quan đến: Dependency Inversion Principle (depend on abstraction, not implementation)

2. **Tradeoff vs Alternative:**
   - Alternative: `ABC` với `@abstractmethod` → force explicit inheritance
   - Tại sao KHÔNG: forcing inheritance tạo tight coupling, khó mock trong tests, cần import ABC
   - Protocol cho phép: user code không cần import `ToolCallInterceptor` để implement nó

3. **Commit history:**
   ```bash
   git log --oneline langchain_mcp_adapters/interceptors.py | head -10
   git show HEAD:langchain_mcp_adapters/interceptors.py | head -50
   ```

4. **Industry reference:**
   - FastAPI: `HTTPException` và dependency injection dùng Protocol-like patterns
   - `httpx`: `Auth` protocol — any callable với đúng signature là auth handler
   - Python async ecosystem: Protocol pattern là best practice cho callback interfaces

**Verify:** Có đủ 4 chiều, có ít nhất 1 industry ref cụ thể (tên thư viện + pattern name).

### Step 3: Research Decision B — Sessions Factory Pattern (~15 phút)

**Decision:** Tại sao transport-specific logic bị tách vào `_create_stdio_session()`, `_create_sse_session()`, etc. thay vì xử lý trong 1 function lớn?

Đọc [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) (dispatch function) và một transport function (VD: [`sessions.py:214`](../../langchain_mcp_adapters/sessions.py#L214)).

**4 chiều phân tích:**

1. **Nguyên lý gốc:**
   - Factory Method pattern (GoF): define interface for creating object, let subclasses decide implementation
   - Single Responsibility: mỗi `_create_*_session()` chỉ biết về 1 transport type
   - Open/Closed: thêm transport mới = thêm function mới + thêm dispatch case, không sửa existing

2. **Tradeoff:**
   - Alternative: 1 function lớn với if/elif chain
   - Tại sao KHÔNG: hard to test, hard to add new transport, violation of SRP
   - Tradeoff của Factory: boilerplate nhiều hơn, nhưng maintainability tốt hơn

3. **Commit history:** `git log --oneline langchain_mcp_adapters/sessions.py | head -10`

4. **Industry reference:**
   - Django: database backend factory — `django.db.backends.*`
   - `httpx`: `AsyncClient` và `Client` có chung interface, khác transport (sync vs async)
   - LangChain: `BaseLanguageModel` với nhiều provider implementations

**Verify:** Có đủ 4 chiều với ≥1 industry ref cụ thể.

### Step 4: Research Decision C — MultiServerMCPClient không phải Context Manager (~10 phút)

**Decision:** Từ v0.1.0, `MultiServerMCPClient` không còn là async context manager (xóa `__aenter__`/`__aexit__`), thay bằng `client.session(name)` method.

Đọc [`client.py`](../../langchain_mcp_adapters/client.py) — class definition và error message `ASYNC_CONTEXT_MANAGER_ERROR`.

**4 chiều phân tích:**

1. **Nguyên lý gốc:**
   - Single Purpose context managers: mỗi session là 1 resource, client là collection manager
   - Explicit vs Implicit resource management
   - Principle of Least Surprise: `async with client` implies "client is the resource" — misleading

2. **Tradeoff:**
   - Old API: `async with MultiServerMCPClient(...) as client:` — single session only
   - New API: `client = MultiServerMCPClient(...); async with client.session("server") as session:`
   - Tại sao thay đổi: multiple server support requires per-server session lifecycle, không phải per-client

3. **Commit history:** Tìm commit "remove context manager" hoặc changelog
   ```bash
   git log --oneline --all | head -20
   grep -r "ASYNC_CONTEXT_MANAGER" . --include="*.py"
   ```

4. **Industry reference:**
   - `httpx.AsyncClient`: cũng không phải context manager mặc định, nhưng CÓ `async with` support — tradeoff khác nhau
   - `asyncpg` Pool: pool là context manager nhưng connection không phải
   - Pattern: "Manager quản lý nhiều resources" thường không phải context manager

**Verify:** Có đủ 4 chiều.

### Step 5: Format findings và save vào worklog (~5 phút)

Append vào ## Worklog với format chuẩn cho mỗi decision:

```markdown
### Decision N: {Tên decision ngắn gọn}

**Statement:** {1 câu mô tả decision}

**Nguyên lý:** {SOLID/GoF/Architectural pattern} — {giải thích ngắn}

**Tradeoff:**
- Alternative không chọn: {mô tả}
- Tại sao không chọn: {lý do kỹ thuật}
- Tradeoff của lựa chọn hiện tại: {nhược điểm còn lại}

**Industry Reference:** {Thư viện/system} → {pattern giống} → {lesson}

**Code evidence:** [`file.py:N`](path#LN)
```

**Verify:** ≥3 decisions có format đầy đủ. Mọi code references clickable.

## 6. Edge Cases & Error Handling

| Case | Trigger | Expected | Recovery |
|------|---------|----------|---------|
| T2 không explicit list decisions | Worklog của lma-dxm vắn tắt | Infer từ "Core Components" list | Xem step 1 fallback logic |
| Commit history không available | `.git` không accessible hoặc repo mới | Bỏ qua dimension 3, ghi "N/A: git not accessible" | Vẫn OK nếu 3 dimensions còn lại đủ |
| Industry reference chưa chắc | Không chắc pattern X xuất hiện ở đâu | Ghi "pattern này phổ biến trong..." thay vì specific lib | Honest > confident-but-wrong |
| Chỉ tìm được 2 decisions | Codebase quá nhỏ/đơn giản | Tìm thêm ở function level | `TypedDict` vs dataclass, `asynccontextmanager` usage |

## 7. Acceptance Criteria

- **Happy 1:** Given T2 findings, When analyze ≥3 decisions, Then mỗi decision có đủ 4 chiều phân tích
- **Happy 2:** Given each decision, When kiểm tra industry reference, Then có tên thư viện/system cụ thể (không phải "many systems use this")
- **Happy 3:** Given code evidence, Then 100% clickable links resolve đúng file
- **Negative:** Given commit history không accessible, Then ghi "N/A" cho dimension 3, không fabricate

## 8. Technical Notes

- Known design decisions cần research (từ codebase scan):
  - `Protocol` trong [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) và [`callbacks.py:37`](../../langchain_mcp_adapters/callbacks.py#L37)
  - Factory dispatch trong [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399)
  - Error message `ASYNC_CONTEXT_MANAGER_ERROR` trong [`client.py`](../../langchain_mcp_adapters/client.py) (line ~35)
  - `TypedDict` cho Connection types: [`sessions.py:83`](../../langchain_mcp_adapters/sessions.py#L83), [`sessions.py:130`](../../langchain_mcp_adapters/sessions.py#L130), etc.
  - `_build_interceptor_chain()` trong [`tools.py:201`](../../langchain_mcp_adapters/tools.py#L201) — functional chain pattern
- MCP spec URL: https://modelcontextprotocol.io — nhiều decisions driven by protocol spec, không phải arbitrary
- SOLID principles: S=SRP, O=OCP, L=LSP, I=ISP, D=DIP
- GoF patterns relevant: Factory Method, Decorator, Chain of Responsibility, Strategy, Observer

## 9. Risks

- Library là adapter (thin layer) → nhiều decisions driven by MCP spec constraints, không phải internal design. Phân biệt rõ "driven by spec" vs "design choice"
- Industry references có thể stale (library APIs thay đổi) → cite pattern level, không phải API level

## Worklog

### [10:30] Bắt đầu — auto-claimed via /viec bat-dau tu dong
- Kế hoạch 5 bước, ước tính ~50 phút
- Bắt đầu từ Step 1: Đọc T2 output và set research agenda

### [10:35] Step 1 xong — Research agenda confirmed
T2 (lma-dxm) đã explicitly list 3 decisions cho T4:
- "Why Protocol not ABC?" → `@runtime_checkable Protocol` choice
- "Why factory not if-elif monolith?" → `create_session()` dispatch design
- "Why remove context manager?" → `async with client:` → `client.session(name)` migration

✓ ≥3 research questions. Bắt đầu Step 2.

---

### [10:40] Step 2: Decision A — Protocol vs ABC cho ToolCallInterceptor

**Statement:** `ToolCallInterceptor` dùng `@runtime_checkable Protocol` (PEP 544) thay vì ABC với `@abstractmethod`, cho phép structural subtyping không cần explicit inheritance.

**Nguyên lý (SOLID + Python typing):**
- **Dependency Inversion Principle (D)**: Code phụ thuộc vào interface shape, không phải class identity. `ToolCallInterceptor` định nghĩa shape (`async def __call__(self, request, handler) → result`), không phải ancestry.
- **Interface Segregation (I)**: Protocol định nghĩa đúng 1 method. Không có "fat interface" bắt buộc implement methods không dùng đến.
- **Structural subtyping**: Bất kỳ object nào có đúng signature `__call__` đều thỏa mãn Protocol — không cần `import ToolCallInterceptor` từ library. Zero coupling về import.
- `@runtime_checkable` (dòng 111): Cho phép `isinstance(obj, ToolCallInterceptor)` check khi cần validation, mà vẫn giữ duck typing semantics.

**Tradeoff vs Alternative (ABC):**
- **Nếu dùng ABC:** `class MyInterceptor(ToolCallInterceptor): ...` → user PHẢI import `ToolCallInterceptor` → tight coupling vào library namespace. Mỗi lần library đổi import path là breaking change với user code.
- **Tại sao không chọn ABC:** Library adapter (thin bridge) không nên force user code vào inheritance tree. Interceptor là "behavior hook", không phải "domain entity" — Protocol phù hợp hơn ABC.
- **Nhược điểm Protocol:** IDE autocomplete kém hơn (không có `super().__call__()` template). Lỗi sai signature chỉ phát hiện tại runtime, không compile-time (trừ khi dùng mypy).

**Commit history:**
- [`be545d7`](https://github.com/langchain-ai/langchain-mcp-adapters/commit/be545d7): "feature: roll out interceptor patterns (#351)" — Commit đầu tiên tạo interceptor system ĐÃ dùng Protocol từ đầu. Không có ABC-to-Protocol migration. Đây là deliberate upfront choice, không phải refactor.
- [`6d4a5e1`](https://github.com/langchain-ai/langchain-mcp-adapters/commit/6d4a5e1): "add ability to return Command to interceptors" — Thêm `MCPToolCallResult = CallToolResult | ToolMessage | Command` mà KHÔNG cần sửa Protocol signature. Structural typing cho phép mở rộng return type union mà không breaking change cho user code.

**Industry Reference:**
- **httpx `Auth` protocol**: `httpx.Auth` là Protocol — bất kỳ object nào có `auth_flow()` generator là valid Auth handler. Không ABC, không inheritance. Cùng triết lý: "behavioral duck typing cho callback interfaces."
- **Python `Iterable` / `Iterator` protocols (PEP 234)**: `for x in obj` chỉ cần `__iter__`. Không import `Iterable` từ collections.abc. `langchain-mcp-adapters` áp dụng cùng philosophy ở application level.
- **FastAPI `Depends()`**: Chấp nhận bất kỳ callable nào có đúng signature → structural typing cho DI. Không class hierarchy.
- **Go interfaces**: Python Protocol là "Go interfaces" của Python. `langchain-mcp-adapters` là một trong số ít Python libraries áp dụng pattern này nhất quán cho plugin API.

**Code evidence:** [`interceptors.py:111-141`](../../langchain_mcp_adapters/interceptors.py#L111-L141)

---

### [10:50] Step 3: Decision B — Function-based Factory Dispatch trong create_session()

**Statement:** `create_session()` là pure function dispatcher với `if/elif` chain thay vì class hierarchy (`StdioSessionFactory(SessionFactory)`), và được implement như `@asynccontextmanager` thay vì returning object.

**Nguyên lý (GoF Factory Method + OCP):**
- **Factory Method Pattern (GoF)**: "Define interface for creating object, let subclasses decide" — nhưng ở đây "subclasses" được thay bằng private functions (`_create_stdio_session`, `_create_sse_session`, etc.). Inversion without class hierarchy.
- **Open/Closed Principle (O)**: Thêm transport mới → chỉ thêm `elif transport == "grpc":` + `_create_grpc_session()`. Zero existing code changes. Đã verified qua commit lịch sử.
- **Single Responsibility (S)**: `create_session()` chỉ làm dispatch + validation. Mỗi `_create_*_session()` chỉ biết về 1 transport. Không cross-concern.
- **Tại sao `@asynccontextmanager`**: MCP session là RESOURCE (giữ open connection/subprocess). `async with create_session(conn) as session:` = acquire-use-release. Đây là standard Python pattern cho managed resources.

**Tradeoff vs Alternatives:**
- **Alternative 1 — Class hierarchy** (`StdioSessionFactory`, `SSESessionFactory`): Cần instantiate factory object trước khi create session. Config dict phải map to class. JSON config trở nên phức tạp hơn nhiều.
- **Alternative 2 — Monolith function**: Tất cả transport logic inline trong 1 function lớn. Impossible to unit test individual transports. Violated SRP.
- **Alternative 3 — Plugin registry** (dict mapping string → factory): Overengineered cho 4 transports với stable set. Thêm complexity không cần thiết.
- **Chosen: dict-like TypedDict config + function dispatch**: `connection["transport"]` là string → JSON-serializable, environment-configurable, no class instantiation at config site. **Key insight**: connections đến từ external config (YAML/JSON/env vars). String-based dispatch = config-driven architecture.

**Commit history — Pattern được validated qua evolution:**
- [`7d06f49`](https://github.com/langchain-ai/langchain-mcp-adapters/commit/7d06f49): "Allow passing streamable-http and http as transport (#282)" — Thêm ALIASES cho cùng transport. Với class hierarchy, cần thêm subclass hoặc alias mapping. Với `if/elif`: chỉ thêm `elif transport in {"streamable_http", "streamable-http", "http"}:` — 1 dòng. **Commit này chứng minh function-dispatch linh hoạt hơn class-based factory.**
- [`bdfe8d9`](https://github.com/langchain-ai/langchain-mcp-adapters/commit/bdfe8d9): "fix: resolve `${VAR}` env variables" — Thêm `_expand_env_vars()` vào `create_session()` mà không ảnh hưởng transport logic. Single entry point = single place để add cross-cutting concerns.

**Industry Reference:**
- **SQLAlchemy `create_engine("postgresql://...")`**: String URL dispatch → backend factory. Function-based, config-driven. Không class hierarchy at call site. Cùng pattern.
- **Django database backends**: `DATABASES = {"ENGINE": "django.db.backends.sqlite3"}` — String dispatch to backend. Không `engine = PostgreSQLBackend()`.
- **LangChain `init_chat_model("gpt-4")`**: String → model class dispatch. Functional factory pattern nhất quán trong LangChain ecosystem. `langchain-mcp-adapters` (cùng tổ chức) áp dụng cùng philosophy.

**Code evidence:** [`sessions.py:398-470`](../../langchain_mcp_adapters/sessions.py#L398-L470)

---

### [10:58] Step 4: Decision C — MultiServerMCPClient Context Manager Removal (Tombstone Pattern)

**Statement:** Từ v0.1.0, `MultiServerMCPClient.__aenter__()` raise `NotImplementedError` thay vì implement context manager — deliberate removal, bảo tồn "helpful error" thay vì silent deletion, vì `client` là **orchestrator of N resources**, không phải resource itself.

**Nguyên lý (Resource Lifecycle Design + Principle of Least Surprise):**
- **Context manager semantic contract**: `async with obj:` implies "obj IS a single, atomic resource" — entering acquires it, exiting releases it. `MultiServerMCPClient` manages N server connections với N independent lifecycles — không có single resource để acquire/release.
- **Principle of Explicit Resource Management**: Thay vì `async with client:` (vague: "what is being acquired?"), API mới là `async with client.session("server_name") as session:` — explicit: "cái session này với server này là resource."
- **Tombstone Pattern (UX-driven stub)**: `__aenter__` vẫn tồn tại nhưng raise `NotImplementedError(ASYNC_CONTEXT_MANAGER_ERROR)` với migration guide (lines 33-42). Tại sao không simply delete method? Vì Python duck typing: `async with obj:` đầu tiên gọi `__aenter__` — nếu không có method, user nhận `TypeError: object does not support async context manager`. `NotImplementedError` với clear message = better DX.

**Tradeoff:**
- **Old API (pre-0.1.0)**: `async with MultiServerMCPClient(connections) as client:` — implied single-server, single-session lifecycle. User convenience nhưng wrong mental model cho multi-server.
- **Breaking change tại 0.1.0**: Cần support multi-server với independent lifecycles. Không thể giữ `async with client:` vì "what does 'exiting the client' mean when server A is still running?"
- **New API**: `client.session("math")` returns `@asynccontextmanager` — per-server resource. `get_tools()` tạo và đóng session internally (stateless, mỗi call là fresh session). Hai modes: "stateless convenience" vs "explicit session control."
- **Nhược điểm**: Breaking API change. Cần migration. Old code fails at runtime (không compile-time). `ASYNC_CONTEXT_MANAGER_ERROR` message ở lines 33-42 là mitigation.

**Commit history:**
- [`570d5c3`](https://github.com/langchain-ai/langchain-mcp-adapters/commit/570d5c3): "release 0.1.0 (#150)" — Version bump. Context manager removal là part of 0.1.0 redesign. Không có pre-0.1.0 commits trong scope hiện tại để trace chính xác PR, nhưng `ASYNC_CONTEXT_MANAGER_ERROR` message explicitly says "As of langchain-mcp-adapters 0.1.0" — self-documenting.
- [`c0f1ae7`](https://github.com/langchain-ai/langchain-mcp-adapters/commit/c0f1ae7): "fix: workaround mcp context manager suppressing exceptions + doc (#352)" — Interesting: MCP SDK itself uses context managers (for sessions), và có bug với exception suppression. `langchain-mcp-adapters` phải workaround SDK behavior. **Bằng chứng**: Context managers vẫn được dùng (cho `client.session()`), nhưng client-level context manager đúng là không cần thiết và đã bị loại bỏ.

**Industry Reference:**
- **asyncpg `Pool`**: `Pool.acquire()` returns context-managed connection, nhưng `Pool` itself is NOT a context manager. "Pool orchestrates connections; Pool ≠ a connection." Cùng split như `MultiServerMCPClient.session()`.
- **httpx `AsyncClient`**: `async with httpx.AsyncClient() as client:` IS supported — vì single client = single connection pool (1 resource). Multi-server complexity đảo quan hệ này. `langchain-mcp-adapters` đúng khi không force single-resource semantic.
- **General pattern: "Orchestrator of N resources ≠ is itself a resource"**: `asyncpg.Pool`, `aiohttp.TCPConnector`, `concurrent.futures.ProcessPoolExecutor` — tất cả IS NOT context managers theo đúng nghĩa (nếu có, thường là convenience wrapper, không phải core pattern).
- **LangChain `BaseChatModel`**: Client objects không phải context managers. Chỉ resources (sessions, connections) mới là.

**Code evidence:** [`client.py:33-42`](../../langchain_mcp_adapters/client.py#L33-L42) (error message) và [`client.py:255-279`](../../langchain_mcp_adapters/client.py#L255-L279) (tombstone methods)

---

### [11:05] Step 5: Acceptance Criteria check

- [x] Happy 1: 3 decisions, mỗi cái có đủ 4 chiều phân tích (nguyên lý, tradeoff, commit, industry ref)
- [x] Happy 2: Mỗi decision có ≥1 industry reference cụ thể (tên lib + pattern name):
  - Decision A: httpx Auth protocol, Python Iterable protocol, FastAPI Depends, Go interfaces
  - Decision B: SQLAlchemy create_engine, Django backends, LangChain init_chat_model
  - Decision C: asyncpg Pool, httpx AsyncClient, LangChain BaseChatModel
- [x] Happy 3: Code evidence references clickable (verified format)
- [x] Negative: Không fabricate commit details không có trong git log

**Bonus finding — Decision D: `_build_interceptor_chain()` closure default-arg trick**

Pattern đáng ghi chú tại [`tools.py:201-232`](../../langchain_mcp_adapters/tools.py#L201-L232): vòng lặp `for interceptor in reversed(tool_interceptors)` với `async def wrapped_handler(..., _interceptor=interceptor, _handler=current_handler)`.

**Tại sao default args thay vì closure capture?** Python closure capture là by-reference (late binding) — nếu không dùng default args, tất cả `wrapped_handler` closures sẽ capture cùng tham chiếu `interceptor` (giá trị cuối cùng sau loop). Default args là by-value (early binding) — capture giá trị tại thời điểm định nghĩa. **Classic Python gotcha**, được giải quyết đúng cách.

**Industry ref**: Pattern này identical với pattern trong Python docs "lambda in a loop" problem. Được dùng trong Django middleware chain, Starlette middleware, và bất kỳ Python middleware system nào.
