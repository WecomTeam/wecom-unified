# 企业微信智能文档（智能主页）管理

> 公共概念与规则请参考 [wecom-shared.md](wecom-shared.md)
>
> 相关品类入口：
> - 文档 → [wecom-doc.md](wecom-doc.md)
> - 表格（在线表格）→ [wecom-sheet.md](wecom-sheet.md)
> - 智能表格 → [wecom-smartsheet.md](wecom-smartsheet.md)

管理企业微信**智能文档**（原名「智能主页」，`/smartpage/*`）的创建与导出。智能文档接口支持通过 `docid` 或 `url` 二选一定位。

> ⚠️ **重要触发规则**：**只有**当用户明确提到「**智能文档**」或「**智能主页**」时，才使用本文件中的接口（`smartpage_*` 系列）。其他所有涉及「文档」的场景（如"创建文档"、"写个文档"、"帮我建个文档"等），一律使用企微文档接口，详见 [wecom-doc.md](wecom-doc.md)。

## 调用方式

通过 `wecom-cli` 调用，品类为 `doc`：

```bash
wecom-cli doc <tool_name> '<json_params>'
```

---

## URL 品类识别

| URL 模式 | 品类 | 入口文件 |
|---|---|---|
| `https://doc.weixin.qq.com/doc/*` | **文档**（doc_type=3） | [wecom-doc.md](wecom-doc.md) |
| `https://doc.weixin.qq.com/sheet/*` | **表格 / 在线表格** | [wecom-sheet.md](wecom-sheet.md) |
| `https://doc.weixin.qq.com/smartsheet/*` | **智能表格**（doc_type=10） | [wecom-smartsheet.md](wecom-smartsheet.md) |
| `https://doc.weixin.qq.com/smartpage/*` | **智能文档**（原名智能主页） | 本文件 |

**判断规则**：URL 路径以 `/smartpage/*` 开头 → 智能文档 → 使用本文件接口。

## 返回格式说明

所有接口返回 JSON 对象，包含以下公共字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `errcode` | integer | 返回码，`0` 表示成功，非 `0` 表示失败 |
| `errmsg` | string | 错误信息，成功时为 `"ok"` |

当 `errcode` 不为 `0` 时，说明接口调用失败，可重试 1 次；若仍失败，将 `errcode` 和 `errmsg` 展示给用户。

### 特殊错误码

| errcode | errmsg | 含义 | 处理方式 |
|---------|--------|------|----------|
| `851002` | `incompatible doc type` | 文档品类与所调用的接口不匹配 | 根据文档 URL 重新确认品类（参见上方「URL 品类识别」表），然后使用该品类对应的正确接口重试 |

---

## 适用场景

1. 将本地 Markdown 文件创建为智能文档
2. 异步导出智能文档内容为 Markdown

---

## 接口列表

### smartpage_create

创建智能文档（原名智能主页），支持传入标题和多个子页面。每个子页面可指定标题、内容类型和本地文件路径。创建成功返回 docid 和 url。

> ⚠️ **特殊语法**：此命令必须使用 `+smartpage_create`（带 `+` 前缀），加号不可省略；该 `+` 仅适用于此命令，不要泛化到其他 `doc` 子命令。

```bash
wecom-cli doc +smartpage_create '{"title": "项目概览", "pages": [{"page_title": "需求文档", "content_type": 1, "page_filepath": "/path/to/requirements.md"}]}'
```

**注意**：
- `content_type` **必须与文件实际内容匹配**：`.md` 文件或包含 Markdown 语法的内容必须传 `1`（Markdown），仅纯文本才传 `0`。绝大多数场景应传 `1`
- docid 仅在创建时返回，需妥善保存
- 每个子页面的 Markdown 文件大小不得超过 **10MB**，超过会导致创建失败。如果文件过大，需先拆分为多个子页面再创建

参见 [API 详情](wecom-doc-smartpage-create.md)。

### smartpage_export_task

发起智能文档内容导出任务（异步）。传入 docid（或 url）和 content_type，返回 task_id。这是异步导出的第一步，需配合 `smartpage_get_export_result` 轮询获取导出结果。

- 通过 docid：
```bash
wecom-cli doc smartpage_export_task '{"docid": "DOCID", "content_type": 1}'
```
- 或通过 URL：
```bash
wecom-cli doc smartpage_export_task '{"url": "https://doc.weixin.qq.com/smartpage/xxx", "content_type": 1}'
```

参见 [API 详情](wecom-doc-smartpage-export.md)。

### smartpage_get_export_result

查询智能文档导出任务进度。传入 task_id 进行轮询，当 `task_done` 为 `true` 时返回 `content`（导出的完整文档内容）。

```bash
wecom-cli doc smartpage_get_export_result '{"task_id": "TASK_ID"}'
```

当 `task_done` 为 `true` 时，`content` 字段即为导出的 Markdown 内容。

参见 [API 详情](wecom-doc-smartpage-export.md)。

---

## 典型工作流

> **关键提示**：只有用户明确提到「智能文档」或「智能主页」时才走本文件流程，其他文档场景一律使用 [wecom-doc.md](wecom-doc.md)。

1. **创建智能文档**（⚠️ 命令必须带 `+` 前缀，不可省略） →
```bash
wecom-cli doc +smartpage_create '{"title": "标题", "pages": [{"page_title": "子页面", "content_type": 1, "page_filepath": "/path/to/file.md"}]}'
```
   保存返回的 docid
2. **获取智能文档内容**（URL 含 `/smartpage/`，异步两步）：
   - **第一步**：发起导出任务 →
```bash
wecom-cli doc smartpage_export_task '{"docid": "DOCID", "content_type": 1}'
```
   获取 `task_id`
   - **第二步**：轮询导出结果 →
```bash
wecom-cli doc smartpage_get_export_result '{"task_id": "TASK_ID"}'
```
   若 `task_done` 为 `false` 则继续轮询，直到 `task_done` 为 `true`，返回的 `content` 字段即为 Markdown 内容
