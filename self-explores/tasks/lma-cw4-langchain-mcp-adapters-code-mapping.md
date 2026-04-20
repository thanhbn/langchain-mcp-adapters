---
date: 2026-04-20
type: task-worklog
task: lma-cw4
title: "langchain-mcp-adapters — Code Mapping (Truy vết thực tế)"
status: completed
started_at: 2026-04-20 11:30
completed_at: 2026-04-20 12:00
detailed_at: 2026-04-20 10:00
detail_score: ready-for-dev
tags: [system-design, code-mapping, leverage-points, langchain-mcp-adapters]
template: system-design-deep-dive
template_id: 6
template_task: T3
---

# langchain-mcp-adapters — Code Mapping (Truy vết thực tế) — Detailed Design

## 1. Objective

Truy vết từng Core Component và Leverage Point đã xác định ở T2 đến file:line cụ thể. Trích đoạn "code tinh hoa" (50-100 dòng) thể hiện nguyên lý thiết kế rõ nhất và giải thích **TẠI SAO** (không phải WHAT). Mọi reference phải là clickable markdown links. Task này chạy song song với T4 (Deep Research).

## 2. Scope

**In-scope:**
- Map từng Core Component/Leverage Point từ T2 đến file:line
- Trích ≥3 đoạn "code tinh hoa" với giải thích tại sao (nguyên lý thiết kế)
- Tạo mapping table đầy đủ với clickable links
- Giải thích "code tinh hoa" theo góc "tại sao đoạn này đặc biệt" (không phải "đoạn này làm gì")

**Out-of-scope:**
- Không trace tests (chỉ trace source code)
- Không cần chạy code
- Không cần internet/API

## 3. Input / Output

**Input:**
- `self-explores/tasks/lma-dxm-langchain-mcp-adapters-strategic-eval.md` — danh sách Core Components + Leverage Points từ T2
- Source files: tất cả 7 modules trong `langchain_mcp_adapters/`

**Output (ghi vào ## Worklog):**
- Mapping table: Component/Point → File:Line (clickable)
- ≥3 code snippets trích dẫn (trong code blocks) với giải thích TẠI SAO
- Tổng kết: "Đoạn code nào đại diện tinh hoa nhất của codebase này?"

## 4. Dependencies

- `lma-dxm` phải xong trước (cần danh sách Core Components + Leverage Points từ T2)
- **Có thể chạy song song với `lma-o7d` (T4 Deep Research)** — không depend nhau

## 5. Flow xử lý

### Step 1: Đọc T2 output và lập danh sách cần map (~5 phút)

Đọc lma-dxm worklog, extract:
- Core Components list (≥2 modules)
- Leverage Points list (≥2 items)

Tạo checklist làm việc:
```
[ ] Core Component: client.py → cần map đến class/method cụ thể
[ ] Core Component: sessions.py → cần map đến create_session()
[ ] Leverage Point: interceptors.py → cần map đến Protocol definition
[ ] Leverage Point: ... → ...
```

**Verify:** Có checklist ≥4 items trước khi bắt đầu scan.

### Step 2: Scan từng module — entry points (~15 phút)

Với mỗi module cần map:

```bash
grep -n "^class\|^def\|^async def" langchain_mcp_adapters/client.py
grep -n "^class\|^def\|^async def" langchain_mcp_adapters/sessions.py
grep -n "^class\|^def\|^async def" langchain_mcp_adapters/tools.py
grep -n "^class\|^def\|^async def" langchain_mcp_adapters/interceptors.py
```

Đọc class definitions + docstrings để hiểu purpose.

**Verify:** Biết line numbers chính xác cho mỗi class/function cần map.

### Step 3: Map từng item đến file:line cụ thể (~20 phút)

Với mỗi item trong checklist:
1. Xác định file chính
2. Tìm class/function line number
3. Đọc 5-10 dòng xung quanh để confirm đây là "đúng chỗ"
4. Ghi vào mapping table với clickable link

Format:
```markdown
| Core Component | File | Line | Nguyên lý |
|---|---|---|---|
| MultiServerMCPClient | [`client.py:50`](../../langchain_mcp_adapters/client.py#L50) | 50 | Orchestrator pattern |
| create_session factory | [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) | 399 | Factory Method |
| ToolCallInterceptor Protocol | [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) | 112 | Decorator/Chain |
```

**Verify:** Mỗi link trỏ đến đúng class/function (không phải random line).

### Step 4: Chọn và trích "code tinh hoa" (~20 phút)

Tiêu chí chọn "code tinh hoa":
- Đoạn code thể hiện design decision rõ nhất (tại sao KHÔNG làm cách khác)
- Thường là: Protocol definitions, Factory functions, Conversion functions

Candidates dựa trên codebase:
- `_build_interceptor_chain()` trong [`tools.py:201`](../../langchain_mcp_adapters/tools.py#L201) — chain pattern
- `ToolCallInterceptor` Protocol trong [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) — duck typing
- `create_session()` dispatch logic trong [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) — Factory pattern

Với mỗi đoạn được chọn:
1. Trích trong code block (đủ context, 20-80 dòng)
2. Giải thích TẠI SAO (nguyên lý, tradeoff, alternative không được chọn)

**Format giải thích:**
```
**Tại sao đoạn này là "code tinh hoa":**
- Nguyên lý: {SOLID principle hoặc GoF pattern}
- Thay vì làm X (cách đơn giản hơn), tác giả làm Y vì: {tradeoff}
- Nếu thay đổi dòng N: {consequence}
```

**Verify:** ≥3 đoạn được trích với giải thích "tại sao" (không phải "what it does").

### Step 5: Format và save vào worklog (~5 phút)

Append vào ## Worklog phần:
1. Mapping table (bảng)
2. ≥3 "code tinh hoa" sections
3. Tổng kết: "Đoạn code nào đại diện tinh hoa nhất?"

**Verify:** Chạy grep để đảm bảo 0 plain text file refs:
```bash
grep -P '[a-z_]+\.py:\d+' self-explores/tasks/lma-cw4-*.md | grep -v "^\s*#\|```" | grep -v '\[`'
```
Nếu có kết quả → còn plain text refs chưa convert.

## 6. Edge Cases & Error Handling

| Case | Trigger | Expected | Recovery |
|------|---------|----------|---------|
| T2 không có findings | lma-dxm worklog trống hoặc chưa done | Không có gì để map | Dừng, report blocking dependency |
| Line numbers lệch | PR mới merge giữa T2 và T3 | Links bị stale | Re-verify bằng grep trước khi save |
| "Code tinh hoa" quá obvious | Đoạn code chỉ là boilerplate | Không có insight | Chọn đoạn khác, focus vào conversion logic |
| Trùng mapping với T4 | Cùng module nhưng góc nhìn khác | OK | Ghi rõ "xem T4 để biết tại sao" |

## 7. Acceptance Criteria

- **Happy 1:** Given T2 findings list, When map từng item, Then 100% Core Components và Leverage Points có ít nhất 1 file:line
- **Happy 2:** Given ≥3 code snippets, When đọc giải thích, Then mỗi giải thích nói "tại sao không làm X" (tradeoff visible)
- **Happy 3:** Given mapping table, When grep plain text refs, Then 0 kết quả (100% clickable)
- **Negative:** Given đoạn code chỉ là boilerplate, Then KHÔNG force-fit thành "tinh hoa" — honest về quality

## 8. Technical Notes

- Known entry points cần map (từ grep output):
  - [`tools.py:70`](../../langchain_mcp_adapters/tools.py#L70) — `MCPToolArtifact` TypedDict
  - [`tools.py:201`](../../langchain_mcp_adapters/tools.py#L201) — `_build_interceptor_chain()`
  - [`tools.py:272`](../../langchain_mcp_adapters/tools.py#L272) — `convert_mcp_tool_to_langchain_tool()`
  - [`tools.py:436`](../../langchain_mcp_adapters/tools.py#L436) — `load_mcp_tools()`
  - [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) — `create_session()` dispatch
  - [`interceptors.py:52`](../../langchain_mcp_adapters/interceptors.py#L52) — `MCPToolCallRequest` dataclass
  - [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) — `ToolCallInterceptor` Protocol
- Relative path từ worklog file: `../../langchain_mcp_adapters/` → `#LN` anchor
- GitHub renders clickable links nếu dùng đúng format: `` [`file.py:N`](path#LN) ``

## 9. Risks

- Line numbers thay đổi → link stale. Mitigation: map đến class-level (stable) hơn function-body (unstable)
- "Code tinh hoa" subjective → anchor vào nguyên lý cụ thể (SOLID, GoF) để tránh bias

## Worklog

### [Step 1] Checklist từ T2

Danh sách items cần map (từ lma-dxm findings):

**Core Components (Trục 1):**
- [x] `tools.py` — `convert_mcp_tool_to_langchain_tool()` + `_convert_call_tool_result()` + `_list_all_tools()`
- [x] `sessions.py` — `create_session()` transport factory

**Leverage Points (Trục 2):**
- [x] `interceptors.py` — `ToolCallInterceptor` Protocol (30-line definition)
- [x] `interceptors.py` — `MCPToolCallRequest` dataclass với `.override()` immutable pattern
- [x] `tools.py` — `_build_interceptor_chain()` (32-line composition engine)
- [x] `sessions.py` — `create_session()` dispatcher + `session_kwargs` callback injection (L427-436)

**Extensibility (Trục 3):**
- [x] `interceptors.py` — `@runtime_checkable Protocol` (OCP extension point)
- [x] `callbacks.py` — `Callbacks.to_mcp_format()` (Observer pattern, context injection)

---

### [Step 2+3] Mapping Table — Core Components & Leverage Points → File:Line

| Loại | Component/Point | File:Line | Nguyên lý |
|------|-----------------|-----------|-----------|
| Core | `MultiServerMCPClient` (orchestrator) | [`client.py:45`](../../langchain_mcp_adapters/client.py#L45) | Orchestrator = thin coordinator, NOT resource owner |
| Core | `convert_mcp_tool_to_langchain_tool()` | [`tools.py:272`](../../langchain_mcp_adapters/tools.py#L272) | Bridge Pattern — MCP ↔ LangChain |
| Core | `_convert_call_tool_result()` | [`tools.py:135`](../../langchain_mcp_adapters/tools.py#L135) | Content type dispatch, gatekeeper result |
| Core | `_convert_mcp_content_to_lc_block()` | [`tools.py:84`](../../langchain_mcp_adapters/tools.py#L84) | Structural dispatch, 5 content types |
| Core | `_list_all_tools()` pagination | [`tools.py:235`](../../langchain_mcp_adapters/tools.py#L235) | Cursor-based pagination, no assumption on list size |
| Core | `create_session()` factory gate | [`sessions.py:399`](../../langchain_mcp_adapters/sessions.py#L399) | Factory Method — single transport entry point |
| Leverage | `ToolCallInterceptor` Protocol | [`interceptors.py:112`](../../langchain_mcp_adapters/interceptors.py#L112) | DIP + ISP — duck typing, zero-import coupling |
| Leverage | `MCPToolCallRequest` dataclass | [`interceptors.py:52`](../../langchain_mcp_adapters/interceptors.py#L52) | Immutable value object với `.override()` |
| Leverage | `_MCPToolCallRequestOverrides` TypedDict | [`interceptors.py:39`](../../langchain_mcp_adapters/interceptors.py#L39) | Type-safe partial overrides (total=False) |
| Leverage | `_build_interceptor_chain()` | [`tools.py:201`](../../langchain_mcp_adapters/tools.py#L201) | Onion composition + Python closure default-arg trick |
| Leverage | callback injection in `create_session()` | [`sessions.py:427`](../../langchain_mcp_adapters/sessions.py#L427) | DRY injection point — 1 place sets ALL session callbacks |
| Leverage | `execute_tool()` inner fn (header mutation) | [`tools.py:329`](../../langchain_mcp_adapters/tools.py#L329) | Session-on-demand + header merging per call |
| Extensibility | `Callbacks.to_mcp_format()` | [`callbacks.py:104`](../../langchain_mcp_adapters/callbacks.py#L104) | Observer adapter — LangChain context → MCP SDK closures |
| Extensibility | `LoggingMessageCallback` Protocol | [`callbacks.py:37`](../../langchain_mcp_adapters/callbacks.py#L37) | Optional hook, duck typing |
| Extensibility | `ProgressCallback` Protocol | [`callbacks.py:53`](../../langchain_mcp_adapters/callbacks.py#L53) | Optional hook |
| Extensibility | `ElicitationCallback` Protocol | [`callbacks.py:71`](../../langchain_mcp_adapters/callbacks.py#L71) | Optional hook — human-in-loop |
| Scale | `asyncio.gather(*load_mcp_tool_tasks)` | [`client.py:197`](../../langchain_mcp_adapters/client.py#L197) | All-or-fail parallel gather (scale bottleneck) |
| Scale | `create_session()` inside `execute_tool()` | [`tools.py:372`](../../langchain_mcp_adapters/tools.py#L372) | New session per call = no pooling (scale bottleneck) |

---

### [Step 4] Code Tinh Hoa — 3 Đoạn Chính

---

#### 🏆 Code Tinh Hoa #1: `_build_interceptor_chain()` — Onion Composition với Python Closure Trick

**File:** [`tools.py:201-232`](../../langchain_mcp_adapters/tools.py#L201-L232)

```python
def _build_interceptor_chain(
    base_handler: Callable[[MCPToolCallRequest], Awaitable[MCPToolCallResult]],
    tool_interceptors: list[ToolCallInterceptor] | None,
) -> Callable[[MCPToolCallRequest], Awaitable[MCPToolCallResult]]:
    handler = base_handler

    if tool_interceptors:
        for interceptor in reversed(tool_interceptors):
            current_handler = handler

            async def wrapped_handler(
                req: MCPToolCallRequest,
                _interceptor: ToolCallInterceptor = interceptor,
                _handler: Callable[
                    [MCPToolCallRequest], Awaitable[MCPToolCallResult]
                ] = current_handler,
            ) -> MCPToolCallResult:
                return await _interceptor(req, _handler)

            handler = wrapped_handler

    return handler
```

**Tại sao đoạn này là "code tinh hoa":**

- **Nguyên lý:** OCP (Open for extension) + Python closure semantics
- **Pattern:** Onion/Middleware — handlers wrap nhau như củ hành, outermost = first in list
- **Vấn đề ẩn:** Python closures có "late binding" gotcha — nếu viết `return await interceptor(req, handler)` trực tiếp, **tất cả closures sẽ dùng cùng `interceptor`** = giá trị cuối vòng lặp. Đoạn code này giải quyết bằng cách capture vào default args (`_interceptor=interceptor, _handler=current_handler`). Default args trong Python được evaluate **tại thời điểm định nghĩa function**, không phải lúc gọi.
- **Thay vì làm:** Dùng `functools.partial()` hoặc `lambda` — nhưng default-arg trick ngắn hơn và idiom hơn trong Python async context
- **Nếu xóa `reversed()`:** First interceptor = innermost (đảo execution order), user code sẽ bị ngược behavior hoàn toàn
- **Nếu bỏ default args:** Late binding bug — toàn bộ chain sẽ gọi interceptor cuối cùng trong list, bỏ qua các interceptor trước

---

#### 🏆 Code Tinh Hoa #2: `ToolCallInterceptor` + `MCPToolCallRequest` — Protocol API với Immutable Request

**File:** [`interceptors.py:39-141`](../../langchain_mcp_adapters/interceptors.py#L39-L141)

```python
class _MCPToolCallRequestOverrides(TypedDict, total=False):
    name: NotRequired[str]
    args: NotRequired[dict[str, Any]]
    headers: NotRequired[dict[str, Any] | None]


@dataclass
class MCPToolCallRequest:
    name: str
    args: dict[str, Any]
    server_name: str           # Context: read-only
    headers: dict[str, Any] | None = None  # Modifiable
    runtime: object | None = None          # Context: read-only

    def override(
        self, **overrides: Unpack[_MCPToolCallRequestOverrides]
    ) -> MCPToolCallRequest:
        return replace(self, **overrides)


@runtime_checkable
class ToolCallInterceptor(Protocol):
    async def __call__(
        self,
        request: MCPToolCallRequest,
        handler: Callable[[MCPToolCallRequest], Awaitable[MCPToolCallResult]],
    ) -> MCPToolCallResult:
        ...
```

**Tại sao đoạn này là "code tinh hoa":**

- **Nguyên lý:** DIP (Dependency Inversion) + ISP (Interface Segregation) + Immutable Value Object
- **`@runtime_checkable Protocol`** thay vì ABC: User implement interceptor mà KHÔNG import từ `langchain_mcp_adapters`. Bất kỳ class nào có `async def __call__(self, request, handler)` đều là interceptor hợp lệ. Đây là structural subtyping (PEP 544).
- **`server_name` vs `headers` phân loại rõ:** `server_name` = context, read-only (vì interceptor không nên redirect sang server khác); `headers` = modifiable (interceptor có thể inject auth headers). Phân biệt này được encode vào `_MCPToolCallRequestOverrides` TypedDict: `server_name` KHÔNG có trong `TypedDict` → TypeScript sẽ báo lỗi compile-time nếu cố override.
- **`.override()` immutable pattern:** `dataclasses.replace()` tạo copy mới thay vì mutate in-place. Multiple interceptors trong chain đều nhận request độc lập — không có shared mutable state bug.
- **Thay vì làm:** Dùng `dict` hoặc mutable object — nhưng sẽ bị bug khi interceptors share reference và mutate nhau

---

#### 🏆 Code Tinh Hoa #3: `Callbacks.to_mcp_format()` — Context Injection via Closure Adapter

**File:** [`callbacks.py:104-141`](../../langchain_mcp_adapters/callbacks.py#L104-L141)

```python
def to_mcp_format(self, *, context: CallbackContext) -> _MCPCallbacks:
    if (on_logging_message := self.on_logging_message) is not None:

        async def mcp_logging_callback(
            params: LoggingMessageNotificationParams,
        ) -> None:
            await on_logging_message(params, context)
    else:
        mcp_logging_callback = None

    if (on_progress := self.on_progress) is not None:

        async def mcp_progress_callback(
            progress: float, total: float | None, message: str | None
        ) -> None:
            await on_progress(progress, total, message, context)
    else:
        mcp_progress_callback = None

    if (on_elicitation := self.on_elicitation) is not None:

        async def mcp_elicitation_callback(
            mcp_context: MCPRequestContext,
            params: ElicitRequestParams,
        ) -> MCPElicitResult:
            return await on_elicitation(mcp_context, params, context)
    else:
        mcp_elicitation_callback = None

    return _MCPCallbacks(
        logging_callback=mcp_logging_callback,
        progress_callback=mcp_progress_callback,
        elicitation_callback=mcp_elicitation_callback,
    )
```

**Tại sao đoạn này là "code tinh hoa":**

- **Nguyên lý:** Adapter Pattern + Closure-based Context Propagation
- **Vấn đề:** MCP SDK callbacks nhận các argument cố định (theo protocol MCP). LangChain cần inject thêm `CallbackContext(server_name, tool_name)` — nhưng không thể thay đổi signature MCP callback vì đó là external interface.
- **Giải pháp:** Wrap mỗi user callback trong 1 closure mới, signature = MCP SDK expected, nhưng captures `context` từ outer scope và pass thêm vào user callback. MCP SDK chỉ thấy `mcp_logging_callback(params)` — đúng signature — còn user code nhận `(params, context)` — đúng cái cần.
- **Walrus operator (`:=`)**: `if (on_logging_message := self.on_logging_message) is not None:` — capture vào local variable trước khi create closure. Tránh closure capturing `self.on_logging_message` (attribute access trong closure có thể bị garbage collect hoặc rebind).
- **Thay vì làm:** Subclass MCP SDK callbacks hay dùng inheritance. Nhưng MCP SDK types là external — subclassing = tight coupling + version fragility. Closure adapter = zero dependency thay đổi.
- **`_MCPCallbacks` vs `Callbacks`:** Public API (`Callbacks`) dùng LangChain-style Protocols với extra context. Internal (`_MCPCallbacks`) dùng MCP SDK types. `to_mcp_format()` là "translation boundary" — chỉ 1 nơi trong toàn codebase biết cả 2 formats.

---

#### 🎖️ Code Tinh Hoa Bonus #4: `_convert_mcp_content_to_lc_block()` — Exhaustive Type Dispatch

**File:** [`tools.py:84-132`](../../langchain_mcp_adapters/tools.py#L84-L132)

```python
def _convert_mcp_content_to_lc_block(  # noqa: PLR0911
    content: ContentBlock,
) -> ToolMessageContentBlock:
    if isinstance(content, TextContent):
        return create_text_block(text=content.text)

    if isinstance(content, ImageContent):
        return create_image_block(base64=content.data, mime_type=content.mimeType)

    if isinstance(content, AudioContent):
        msg = (
            "AudioContent conversion to LangChain content blocks is not yet "
            f"supported. Received audio with mime type: {content.mimeType}"
        )
        raise NotImplementedError(msg)

    if isinstance(content, ResourceLink):
        mime_type = content.mimeType or None
        if mime_type and mime_type.startswith("image/"):
            return create_image_block(url=str(content.uri), mime_type=mime_type)
        return create_file_block(url=str(content.uri), mime_type=mime_type)

    if isinstance(content, EmbeddedResource):
        resource = content.resource
        if isinstance(resource, TextResourceContents):
            return create_text_block(text=resource.text)
        if isinstance(resource, BlobResourceContents):
            mime_type = resource.mimeType or None
            if mime_type and mime_type.startswith("image/"):
                return create_image_block(base64=resource.blob, mime_type=mime_type)
            return create_file_block(base64=resource.blob, mime_type=mime_type)
        msg = f"Unknown embedded resource type: {type(resource).__name__}"
        raise ValueError(msg)

    msg = f"Unknown MCP content type: {type(content).__name__}"
    raise ValueError(msg)
```

**Tại sao đoạn này là "code tinh hoa":**

- **Nguyên lý:** Exhaustive dispatch + Fail-fast at boundary
- **`# noqa: PLR0911`** (too many return statements): Comment này là signal quan trọng. Tác giả biết đây vi phạm linting convention, nhưng chủ động suppress vì **exhaustive matching** quan trọng hơn style compliance. Đây không phải sloppy code — đây là conscious tradeoff được document.
- **AudioContent: `NotImplementedError`** thay vì silent skip: Nếu bỏ qua `AudioContent`, user sẽ nhận empty result mà không biết tại sao. `NotImplementedError` với message cụ thể = honest "not yet supported" thay vì silent corruption.
- **Nested dispatch cho `EmbeddedResource`:** Resource có thể là `TextResourceContents` hoặc `BlobResourceContents` → lại dispatch tiếp. Và trong Blob, còn dispatch tiếp trên mime_type để chọn `image` hay `file`. 3 levels of dispatch = mirror cấu trúc của MCP spec.
- **Thay vì làm:** Dùng dict lookup hoặc `match/case` — nhưng `isinstance()` chain explicit hơn, không cần import extras, và mypy hiểu được narrowing.
- **Nếu thêm content type mới vào MCP spec:** Phải thêm `elif` mới + fallthrough `raise ValueError` cuối đảm bảo unknown types fail loud.

---

### [Step 5] Tổng kết — Đoạn code nào đại diện tinh hoa nhất?

**Winner: `_build_interceptor_chain()` tại [`tools.py:201-232`](../../langchain_mcp_adapters/tools.py#L201-L232)**

Lý do:
1. **Density cao:** 32 lines, nhưng giải quyết 2 vấn đề non-trivial cùng lúc: middleware composition pattern + Python closure late-binding gotcha
2. **Leverage tối đa:** 32 lines này chi phối 100% tool call traffic qua toàn bộ system
3. **Subtlety cao:** Ai không biết Python closure semantics sẽ viết code "trông đúng" nhưng bị bug. Default-arg trick là pattern cần biết, không phải pattern tự nghĩ ra được

**Runner-up: `ToolCallInterceptor` Protocol tại [`interceptors.py:112-141`](../../langchain_mcp_adapters/interceptors.py#L112-L141)**

Lý do: Đây là extension API của library — quyết định này (Protocol vs ABC) ảnh hưởng đến **mọi user** của library. 30 lines define how the whole ecosystem integrates.

**Honorable mention: `Callbacks.to_mcp_format()` tại [`callbacks.py:104-141`](../../langchain_mcp_adapters/callbacks.py#L104-L141)**

Là "translation boundary" pattern ít người biết — elegantly giải quyết signature mismatch giữa MCP SDK và LangChain API mà không coupling vào nhau.

---

### Acceptance Criteria — Verification

```bash
# Verify 0 plain text refs (không có file.py:N ngoài code blocks và bảng markdown)
grep -P '[a-z_]+\.py:\d+' self-explores/tasks/lma-cw4-langchain-mcp-adapters-code-mapping.md \
  | grep -v '^\s*#\|```\|`\[`\|\[`' | grep -v '^\-\-\-'
```

- [x] Happy 1: 100% Core Components và Leverage Points từ T2 đã mapped (18 rows trong bảng)
- [x] Happy 2: ≥3 đoạn code tinh hoa với giải thích "tại sao không làm X" (4 đoạn: 3 chính + 1 bonus)
- [x] Happy 3: 100% file:line references dùng clickable markdown links format `` [`file.py:N`](path#LN) ``
- [x] Negative: Không force-fit boilerplate — `prompts.py` và `resources.py` (thin wrappers) không có trong mapping table
