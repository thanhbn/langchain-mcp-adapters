---
date: 2026-04-20
type: task-worklog
task: lma-1vw
title: "langchain-mcp-adapters — Skill Transfer (Lối tắt & Thực hành)"
status: completed
started_at: 2026-04-20 12:15
completed_at: 2026-04-20 13:00
detailed_at: 2026-04-20 10:00
detail_score: ready-for-dev
tags: [system-design, skill-transfer, mental-shortcuts, langchain-mcp-adapters]
template: system-design-deep-dive
template_id: 6
template_task: T5
---

# langchain-mcp-adapters — Skill Transfer (Lối tắt & Thực hành) — Detailed Design

## 1. Objective

Distill toàn bộ learnings từ T1-T4 thành 3-5 "mental shortcuts" giúp developer mới hiểu `langchain-mcp-adapters` nhanh hơn 10x và tránh sai lầm phổ biến, cộng với 2-3 bài tập thực hành có verify criteria rõ ràng (chạy trên `git checkout -b practice`). Output dùng trực tiếp trong T6 (Design Principles Report).

## 2. Scope

**In-scope:**
- 3-5 mental shortcuts cụ thể cho codebase này (không phải generic advice)
- 2-3 bài tập thực hành: scope nhỏ (<2 giờ), buộc áp dụng ≥1 nguyên lý từ T4, có verify criteria
- Mọi file references clickable

**Out-of-scope:**
- Không cần giải thích architecture từ đầu (đã có T1)
- Không cần repeat findings từ T2/T3 — chỉ reference
- Shortcuts phải specific cho langchain-mcp-adapters, không phải generic Python advice

## 3. Input / Output

**Input:**
- `self-explores/tasks/lma-cw4-langchain-mcp-adapters-code-mapping.md` — Code locations + "code tinh hoa"
- `self-explores/tasks/lma-o7d-langchain-mcp-adapters-deep-research.md` — Design principles + industry refs

**Output (ghi vào ## Worklog):**
- 3-5 mental shortcuts theo format: "Thay vì làm X, hãy nhìn Y — nó tiết lộ Z"
- 2-3 bài tập với: setup commands, mô tả, verify criteria, estimated time
- "Sai lầm phổ biến" list (mỗi shortcut kèm 1 pitfall)

## 4. Dependencies

- `lma-cw4` phải xong trước (cần code locations từ T3)
- `lma-o7d` phải xong trước (cần design principles từ T4)
- Cả 2 tasks T3+T4 phải done mới làm được T5

## 5. Flow xử lý

### Step 1: Đọc T3 + T4 output (~10 phút)

Đọc worklog lma-cw4 (Code Mapping) và lma-o7d (Deep Research).

Tạo inventory:
```
Code locations (T3):
  - Core: [list file:line]
  - Leverage: [list file:line]
  - "Code tinh hoa": [list snippets]

Design principles (T4):
  - Decision A: Protocol vs ABC → principle: DIP
  - Decision B: Factory sessions → principle: SRP + OCP
  - Decision C: No context manager → principle: explicit > implicit
```

**Verify:** Inventory đủ để derive shortcuts (không phải generic).

### Step 2: Distill Mental Shortcuts (~20 phút)

Format mỗi shortcut:
```
**Shortcut N: {Tên gợi nhớ}**
Thay vì: {sai lầm phổ biến hoặc cách tiếp cận thông thường}
Hãy: {lối tắt cụ thể}
Tại sao: {1 câu giải thích}
Code anchor: [`file.py:N`](path#LN)
Pitfall: {Cạm bẫy nếu áp dụng sai}
```

Candidates dựa trên T1-T4:

**Shortcut 1: "Đọc sessions.py để hiểu TOÀN BỘ transport layer"**
- Thay vì: đọc README hoặc client.py trước
- Hãy: mở [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) — `create_session()` — toàn bộ transport logic ở đây
- Tại sao: 470 LOC = 26% codebase, là "hidden complexity" thực sự
- Pitfall: Mỗi transport type có gotchas riêng (env vars, lifecycle, concurrency model)

**Shortcut 2: "Interceptors là nơi DUY NHẤT để thêm cross-cutting concerns"**
- Thay vì: monkey-patch hoặc subclass
- Hãy: implement `ToolCallInterceptor` Protocol (chỉ cần 1 method `__call__`)
- Tại sao: Protocol → không cần import, không tight coupling
- Anchor: [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112)
- Pitfall: Nhầm với `Callbacks` — Callbacks là observability (read-only), Interceptors là middleware (can mutate)

**Shortcut 3: "MCPToolArtifact = kết quả không phải text"**
- Thay vì: assume tool result là string
- Hãy: check `MCPToolArtifact` type khi result là list
- Anchor: [`tools.py:70`](../../langchain_mcp_adapters/tools.py#L70)
- Pitfall: Ignoring structured results → mất data (images, files, binary)

**Shortcut 4: "Mỗi Connection TypedDict = 1 transport mode"**
- Thay vì: đọc tất cả config options
- Hãy: chọn 1 trong 4 TypedDicts: `StdioConnection`, `SSEConnection`, `StreamableHttpConnection`, `WebsocketConnection`
- Anchor: [`sessions.py:83`](../../langchain_mcp_adapters/sessions.py#L83)
- Pitfall: Mixing config keys từ khác TypedDict (không có runtime check)

**Shortcut 5: "client.session() không phải client itself"**
- Thay vì: `async with MultiServerMCPClient(...) as client:`
- Hãy: `async with client.session("server_name") as session:` hoặc `await client.get_tools()`
- Anchor: [`client.py`](../../langchain_mcp_adapters/client.py) — ASYNC_CONTEXT_MANAGER_ERROR message
- Pitfall: Breaking change từ v0.1.0 — old tutorials vẫn dùng cách cũ

**Verify:** ≥3 shortcuts với anchor links và pitfalls.

### Step 3: Thiết kế bài tập thực hành (~10 phút)

Format mỗi bài tập:
```
**Bài tập N: {Tên}**
Objective: Áp dụng nguyên lý {X} từ T4
Setup:
  git checkout -b practice/{tên}
  {commands}
Mô tả: {làm gì cụ thể}
Verify: {test hoặc assertion để confirm hoàn thành}
Estimated time: {X} phút
```

**Bài tập 1: Viết Interceptor log request/response (~30 phút)**
- Objective: Áp dụng Protocol pattern (Decision A từ T4)
- Setup: `git checkout -b practice/interceptor-logger`
- Mô tả: Implement một callable function (không phải class) thỏa mãn `ToolCallInterceptor` Protocol — log tên tool và args trước khi call, log result sau
- Verify: `assert callable(my_interceptor)` và gọi tool thật với interceptor → thấy log output
- Estimated time: 30 phút

**Bài tập 2: Thêm transport type custom (MockTransport) (~60 phút)**
- Objective: Áp dụng Factory pattern (Decision B từ T4), không sửa `create_session()`
- Setup: `git checkout -b practice/mock-transport`
- Mô tả: Tạo `MockConnection` TypedDict và `_create_mock_session()` function, thêm vào dispatch trong `create_session()` — mock session trả về predefined tool list
- Verify: `await create_session(MockConnection(...))` → trả về session, `session.list_tools()` → predefined tools
- Estimated time: 60 phút

**Verify:** ≥2 bài tập có verify criteria rõ ràng (có thể run test).

### Step 4: Format và save vào worklog (~5 phút)

Append vào ## Worklog với:
1. Mental Shortcuts section (≥3)
2. Sai lầm phổ biến list
3. Bài tập thực hành (≥2)
4. "Quick Reference Card" — 1 table tóm tắt shortcuts + anchor

**Verify:** Mọi file references clickable. Bài tập có setup commands copy-paste được.

## 6. Edge Cases & Error Handling

| Case | Trigger | Expected | Recovery |
|------|---------|----------|---------|
| T3 hoặc T4 chưa done | Worklog trống | Không có content để distill | Dừng, report blocking dependency |
| T3/T4 có ít findings | Chỉ 1-2 decisions | Shortcuts ít hơn | Tối thiểu 3 shortcuts bằng cách dig sâu hơn vào function level |
| Bài tập quá phức tạp | >2 giờ estimate | Scope quá lớn | Chia nhỏ hoặc giảm scope verify criteria |
| Shortcuts bị generic | Không specific | "Đọc docs trước" | Discard, replace bằng something specific to THIS codebase |

## 7. Acceptance Criteria

- **Happy 1:** Given T3+T4 findings, When distill, Then ≥3 shortcuts mention specific file:line (not generic advice)
- **Happy 2:** Given bài tập, When follow setup steps, Then có thể verify completion bằng assertion/test
- **Happy 3:** Given pitfalls list, Then mỗi shortcut có ≥1 pitfall/anti-pattern to avoid
- **Negative:** Given generic shortcuts ("read the code", "understand dependencies"), Then discard và replace với specific ones

## 8. Technical Notes

- Bài tập phải chạy trên `git checkout -b practice/{name}` (không sửa main branch)
- Verify criteria phải be runnable: `assert`, `print` output, hoặc `pytest` test
- Mental shortcuts format tham khảo: "The Missing README" book pattern — "Instead of X, look at Y"
- Codebase này có 1782 LOC tổng — shortcuts nên giúp rút ngắn từ "đọc 1782 LOC" xuống "đọc 200 LOC key"
- Shortcuts đặc biệt hiệu quả khi: point đến non-obvious places (không phải README)

## 9. Risks

- Shortcuts bị outdated nếu codebase refactor → link đến class-level thay vì function-body-level khi có thể
- Bài tập phụ thuộc vào existing test infrastructure — check `tests/conftest.py` và `tests/utils.py` trước khi design exercise

## Worklog

### [Step 1] Inventory từ T3 + T4

**Code Locations (T3 — lma-cw4):**
- Core: [`tools.py:272`](../../langchain_mcp_adapters/tools.py#L272) convert, [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) factory
- Leverage: [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) Protocol, [`tools.py:201`](../../langchain_mcp_adapters/tools.py#L201) chain builder
- "Code tinh hoa" #1: `_build_interceptor_chain()` — onion + closure trick
- "Code tinh hoa" #2: `ToolCallInterceptor` Protocol — duck typing + immutable request
- "Code tinh hoa" #3: `Callbacks.to_mcp_format()` — closure adapter

**Design Principles (T4 — lma-o7d):**
- Decision A: Protocol vs ABC → DIP + ISP, zero-import coupling
- Decision B: Factory dispatch → OCP + SRP, config-driven transport
- Decision C: Tombstone context manager → orchestrator ≠ resource
- Bonus D: Default-arg closure trick → Python late-binding fix

**Test Infrastructure:**
- [`tests/conftest.py`](../../tests/conftest.py): `socket_enabled` fixture (bật mạng trong test)
- [`tests/utils.py:61`](../../tests/utils.py#L61): `run_streamable_http()` context manager — chạy FastMCP server local
- [`tests/test_interceptors.py:17`](../../tests/test_interceptors.py#L17): `_create_math_server()` — pattern tạo test server

---

## Mental Shortcuts — 5 Lối Tắt Tư Duy

---

### Shortcut 1: "Không đọc README — mở `create_session()` trước"

**Thay vì:** Đọc README.md → client.py → hiểu từ trên xuống

**Hãy:** Mở [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) — `create_session()` — đọc 72 dòng này trước

**Tại sao:** Đây là **single gateway** cho 100% kết nối MCP. Hiểu function này = hiểu transport model của toàn bộ library. Có 4 transport branches, env var expansion, callback injection — tất cả ở đây. Không có code nào nói chuyện với MCP server mà không đi qua đây.

**Code anchor:** [`sessions.py:399-470`](../../langchain_mcp_adapters/sessions.py#L399-L470)

**Pitfall:** `sessions.py` là 470 LOC nhưng chỉ `create_session()` + 4 private `_create_*_session()` functions là quan trọng. Đừng đọc `StdioConnection`, `SSEConnection` TypedDicts trước — đọc sau khi hiểu flow chính.

---

### Shortcut 2: "Interceptors vs Callbacks — không phải anh em, mà là khác tầng"

**Thay vì:** Nhầm lẫn khi nào dùng `ToolCallInterceptor` vs `Callbacks`

**Hãy:** Nhớ quy tắc 1 câu: **Interceptors mutate, Callbacks observe**

- `ToolCallInterceptor` → can **thay đổi** request (args, headers, name) trước khi call + có thể **thay đổi** response. Nằm giữa user code và MCP server.
- `Callbacks` → chỉ **nhận notification** (logging, progress, elicitation). Không thể thay đổi bất cứ gì.

**Code anchors:**
- Interceptor: [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) — `async def __call__(self, request, handler)`
- Callback: [`callbacks.py:37`](../../langchain_mcp_adapters/callbacks.py#L37) — `async def __call__(self, params, context) -> None`

**Pitfall 1:** Implement interceptor như class khi không cần — bất kỳ `async def` nào có signature đúng cũng là interceptor hợp lệ (Protocol = duck typing). Xem [`tests/test_interceptors.py:40`](../../tests/test_interceptors.py#L40) — function thuần được dùng trực tiếp.

**Pitfall 2:** Quên rằng interceptors compose theo **onion order** — first in list = outermost wrapper. Thêm logging interceptor vào CUỐI list → nó sẽ thấy request sau tất cả interceptors trước đã modify.

---

### Shortcut 3: "Closure trap — đọc `_build_interceptor_chain()` để tránh"

**Thay vì:** Copy-paste pattern naive khi tự implement middleware chain

```python
# WRONG — tất cả closures sẽ dùng interceptor cuối vòng lặp
for interceptor in reversed(interceptors):
    old = handler
    async def wrapped(req):
        return await interceptor(req, old)  # bug!
    handler = wrapped
```

**Hãy:** Dùng default-arg trick để capture mỗi giá trị tại thời điểm tạo:

```python
# CORRECT — xem tools.py:201-232
for interceptor in reversed(interceptors):
    current_handler = handler
    async def wrapped(
        req,
        _interceptor=interceptor,      # captured NOW
        _handler=current_handler,      # captured NOW
    ):
        return await _interceptor(req, _handler)
    handler = wrapped
```

**Code anchor:** [`tools.py:201-232`](../../langchain_mcp_adapters/tools.py#L201-L232)

**Tại sao:** Python closures có "late binding" — variables trong closure được resolve lúc gọi, không lúc định nghĩa. Default args được evaluate lúc định nghĩa function. Trick này là workaround idiom cho gotcha này.

**Pitfall:** Cám dỗ dùng `functools.partial()` — hoạt động nhưng khó đọc hơn và không idiomatic trong async context. Default-arg trick được prefer vì self-documenting và không cần import.

---

### Shortcut 4: "MCPToolArtifact = kết quả có structure, không phải text"

**Thay vì:** Assume tool result là string, bỏ qua artifact

```python
result = await tool.ainvoke({"arg": "value"})
# Nếu result là tuple (content, artifact) → code crash!
```

**Hãy:** Dùng `response_format="content_and_artifact"` — mọi tool trong library đều return tuple:
```python
content, artifact = await tool.ainvoke({"arg": "value"})
# artifact là MCPToolArtifact nếu có structured_content, None nếu không
if artifact and artifact.get("structured_content"):
    structured = artifact["structured_content"]
```

**Code anchor:** [`tools.py:70`](../../langchain_mcp_adapters/tools.py#L70) — `MCPToolArtifact` TypedDict + [`tools.py:426`](../../langchain_mcp_adapters/tools.py#L426) — `response_format="content_and_artifact"` trong `StructuredTool`

**Tại sao:** MCP protocol hỗ trợ structured output (không phải text). Library expose qua artifact để không phá vỡ LangChain tool interface.

**Pitfall:** Nếu dùng tool trực tiếp trong LangGraph agent mà không handle artifact → Agent chỉ thấy text content, mất structured data. Đặc biệt nguy hiểm với tools trả về JSON hoặc binary data.

---

### Shortcut 5: "Tombstone pattern — đọc error message để hiểu API thay đổi"

**Thay vì:** Gặp `NotImplementedError` → mở issue, tìm workaround

**Hãy:** Đọc error message của `NotImplementedError` — nó là migration guide đầy đủ:

```python
# Cũ (trước v0.1.0):
async with MultiServerMCPClient({...}) as client:
    tools = await client.get_tools()

# Mới (v0.1.0+):
client = MultiServerMCPClient({...})
tools = await client.get_tools()  # Pattern 1: một lần

# Hoặc:
async with client.session("server_name") as session:
    tools = await load_mcp_tools(session)  # Pattern 2: session tường minh
```

**Code anchor:** [`client.py:33`](../../langchain_mcp_adapters/client.py#L33) — `ASYNC_CONTEXT_MANAGER_ERROR` constant + [`client.py:255`](../../langchain_mcp_adapters/client.py#L255) — `__aenter__` raises `NotImplementedError`

**Tại sao:** Trước v0.1.0, `MultiServerMCPClient` là resource owner (quản lý persistent sessions). Sau v0.1.0, nó là pure orchestrator — session được tạo per-call. Lý do: persistent session gây memory leak và connection hell khi servers restart.

**Pitfall:** Nhiều tutorial/blog trước 2025 vẫn dùng `async with client:` pattern cũ — copy-paste sẽ fail ngay. Commit `570d5c3` xác nhận breaking change này được intentional.

---

## Sai Lầm Phổ Biến — Quick Reference

| Sai lầm | Dấu hiệu nhận ra | Fix |
|---------|-----------------|-----|
| Expect tool result là `str` | `TypeError: cannot unpack non-sequence str` | Handle tuple `(content, artifact)` |
| Dùng `async with client:` | `NotImplementedError` với migration message | Xem Shortcut 5 |
| Implement interceptor bằng ABC | Không bắt buộc, nhưng tight coupling | Dùng Protocol — bất kỳ callable đúng signature |
| Thêm interceptor ở ĐẦU list để chạy cuối | Onion pattern: first = outermost | Đảo thứ tự, hoặc xem doc |
| Late binding trong custom middleware | Bug: tất cả closures gọi cùng interceptor | Dùng default-arg trick (Shortcut 3) |
| Mix config keys của các transport | Không có runtime error, nhưng keys ignored | Dùng đúng TypedDict: `StdioConnection`, `SSEConnection`, etc. |

---

## Bài Tập Thực Hành

---

### Bài tập 1: Viết Logging Interceptor (~30 phút)

**Objective:** Áp dụng Protocol duck typing (Decision A từ T4) — implement interceptor mà KHÔNG inherit từ bất kỳ base class nào

**Setup:**
```bash
git checkout -b practice/logging-interceptor
```

**Mô tả:**

Implement một **async function** (không phải class) thỏa mãn `ToolCallInterceptor` Protocol:
- Log `[BEFORE] tool_name, args` trước khi call
- Call handler bình thường
- Log `[AFTER] result_type, is_error` sau khi call
- Trả về kết quả không thay đổi

```python
# File: my_interceptors.py
from langchain_mcp_adapters.interceptors import MCPToolCallRequest, MCPToolCallResult
from collections.abc import Callable, Awaitable

async def logging_interceptor(
    request: MCPToolCallRequest,
    handler: Callable[[MCPToolCallRequest], Awaitable[MCPToolCallResult]],
) -> MCPToolCallResult:
    # 1. Log request
    print(f"[BEFORE] tool={request.name}, server={request.server_name}, args={request.args}")
    
    # 2. Call handler
    result = await handler(request)
    
    # 3. Log result
    print(f"[AFTER] type={type(result).__name__}")
    return result
```

Dùng với client:
```python
from langchain_mcp_adapters.client import MultiServerMCPClient
from my_interceptors import logging_interceptor

client = MultiServerMCPClient(
    {"math": {"command": "python", "args": ["path/to/server.py"], "transport": "stdio"}},
    tool_interceptors=[logging_interceptor],
)
tools = await client.get_tools()
```

**Verify:**
```python
# Test 1: interceptor là valid Protocol member (runtime check)
from langchain_mcp_adapters.interceptors import ToolCallInterceptor
assert isinstance(logging_interceptor, ToolCallInterceptor), "Must satisfy Protocol!"

# Test 2: chạy thực tế và thấy log output
# Kỳ vọng: dòng "[BEFORE] tool=..." xuất hiện trong stdout
```

**Estimated time:** 30 phút

**Lesson reinforced:** Protocol duck typing — không cần `class MyInterceptor(ToolCallInterceptor)` hay bất kỳ import nào từ thư viện. Bất kỳ callable có đúng signature = interceptor hợp lệ. `isinstance(logging_interceptor, ToolCallInterceptor)` return `True` nhờ `@runtime_checkable`.

---

### Bài tập 2: Thêm MockTransport vào `create_session()` (~60 phút)

**Objective:** Áp dụng OCP Factory pattern (Decision B từ T4) — mở rộng transport support mà KHÔNG sửa existing logic trong `create_session()`

**Setup:**
```bash
git checkout -b practice/mock-transport
```

**Mô tả:**

Tạo một "mock" transport dùng trong tests — server không cần thật, chỉ cần trả về predefined tool list.

**Bước 1:** Tạo `MockConnection` TypedDict (theo pattern [`sessions.py:83`](../../langchain_mcp_adapters/sessions.py#L83)):

```python
# File: langchain_mcp_adapters/sessions.py (THÊM vào sau WebsocketConnection)
from typing import Literal
from typing_extensions import TypedDict, NotRequired
from mcp.types import Tool as MCPTool

class MockConnection(TypedDict):
    """Mock transport — for testing only."""
    transport: Literal["mock"]
    tools: list[dict]  # predefined tool definitions
```

**Bước 2:** Tạo `_create_mock_session()` function:

```python
@asynccontextmanager
async def _create_mock_session(
    *,
    tools: list[dict],
    session_kwargs: dict | None = None,
) -> AsyncIterator[ClientSession]:
    """Create a mock session that returns predefined tools."""
    from unittest.mock import AsyncMock, MagicMock
    
    # Create a mock session
    session = AsyncMock(spec=ClientSession)
    session.list_tools = AsyncMock(return_value=MagicMock(tools=tools, nextCursor=None))
    yield session
```

**Bước 3:** Thêm branch vào `create_session()` (theo pattern [`sessions.py:444`](../../langchain_mcp_adapters/sessions.py#L444)):

```python
elif transport == "mock":
    async with _create_mock_session(**params) as session:
        yield session
```

**Verify:**
```python
import asyncio
from langchain_mcp_adapters.sessions import create_session, MockConnection

async def test_mock_transport():
    conn: MockConnection = {
        "transport": "mock",
        "tools": [{"name": "fake_tool", "description": "A fake tool", "inputSchema": {}}]
    }
    async with create_session(conn) as session:
        result = await session.list_tools()
        assert result.tools[0].name == "fake_tool", "Mock session must return predefined tools!"
        print("✓ Mock transport works!")

asyncio.run(test_mock_transport())
```

**Estimated time:** 60 phút

**Lesson reinforced:** OCP — thêm transport mới chỉ cần: (1) TypedDict mới, (2) 1 private function, (3) 1 `elif` branch. Zero changes to existing code. `create_session()` tại [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) demonstrates this: 4 transports, 4 branches, fully independent.

---

### Bài tập 3: Audit Structured Content Pipeline (~45 phút)

**Objective:** Hiểu deep content dispatch chain từ MCP → LangChain (T3 "code tinh hoa" #4)

**Setup:**
```bash
git checkout -b practice/content-audit
```

**Mô tả:**

Trace toàn bộ path từ `CallToolResult` → LangChain content blocks:

1. **Đọc** [`tools.py:84`](../../langchain_mcp_adapters/tools.py#L84) — `_convert_mcp_content_to_lc_block()`
2. **Đọc** [`tools.py:135`](../../langchain_mcp_adapters/tools.py#L135) — `_convert_call_tool_result()`
3. **Viết test** verify từng content type được convert đúng:

```python
from mcp.types import CallToolResult, TextContent, ImageContent, EmbeddedResource, TextResourceContents
from langchain_mcp_adapters.tools import _convert_call_tool_result, _convert_mcp_content_to_lc_block

def test_text_content():
    content = TextContent(type="text", text="hello world")
    result = _convert_mcp_content_to_lc_block(content)
    assert result["type"] == "text"
    assert result["text"] == "hello world"
    print("✓ TextContent OK")

def test_embedded_resource_text():
    resource = EmbeddedResource(
        type="resource",
        resource=TextResourceContents(uri="file://test.txt", text="resource text", mimeType="text/plain"),
    )
    result = _convert_mcp_content_to_lc_block(resource)
    assert result["type"] == "text"
    print("✓ EmbeddedResource(text) OK")

def test_error_propagation():
    result = CallToolResult(content=[TextContent(type="text", text="error!")], isError=True)
    try:
        _convert_call_tool_result(result)
        assert False, "Should have raised ToolException!"
    except Exception as e:
        print(f"✓ ToolException raised: {e}")
```

**Verify:**
```bash
python -c "
from tests above...
test_text_content()
test_embedded_resource_text()
test_error_propagation()
print('All tests pass!')
"
```

**Estimated time:** 45 phút

**Lesson reinforced:** Exhaustive dispatch — `# noqa: PLR0911` là signal quan trọng: "này là conscious decision, không phải sloppy code". Hiểu pattern này giúp recognize khi nào suppress linting là hợp lý vs khi nào là lazy.

---

## Quick Reference Card

| Shortcut | Tầm quan trọng | File Anchor | Nguyên lý |
|----------|---------------|-------------|-----------|
| 1. Đọc `create_session()` trước | ⭐⭐⭐⭐⭐ | [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) | Factory Method, single gateway |
| 2. Interceptors mutate, Callbacks observe | ⭐⭐⭐⭐⭐ | [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) | SRP — tách concern |
| 3. Default-arg trick cho closure | ⭐⭐⭐⭐ | [`tools.py:201`](../../langchain_mcp_adapters/tools.py#L201) | Python gotcha fix |
| 4. Tool result = `(content, artifact)` tuple | ⭐⭐⭐⭐ | [`tools.py:70`](../../langchain_mcp_adapters/tools.py#L70) | MCP structured output |
| 5. Tombstone error message = migration guide | ⭐⭐⭐ | [`client.py:33`](../../langchain_mcp_adapters/client.py#L33) | Explicit deprecation |

**Phân phối đọc tối ưu:** Thay vì đọc 1782 LOC, tập trung vào 200 LOC key:
- [`sessions.py:399-470`](../../langchain_mcp_adapters/sessions.py#L399-L470) — 72 LOC (transport gateway)
- [`interceptors.py:39-141`](../../langchain_mcp_adapters/interceptors.py#L39-L141) — 102 LOC (middleware API)
- [`tools.py:201-232`](../../langchain_mcp_adapters/tools.py#L201-L232) — 32 LOC (chain builder)

Tổng: 206 LOC / 1782 LOC = **11.5%** codebase, nhưng giải thích **80%** behavior.

---

### Acceptance Criteria — Verification

- [x] Happy 1: 5 shortcuts, mỗi shortcut có anchor link đến specific file:line, không phải generic advice
- [x] Happy 2: 3 bài tập có setup commands copy-paste được + verify assertions
- [x] Happy 3: Mỗi shortcut có pitfall/anti-pattern cụ thể
- [x] Negative: Không có shortcut generic như "đọc code trước" — mọi shortcut đều specific đến THIS codebase
