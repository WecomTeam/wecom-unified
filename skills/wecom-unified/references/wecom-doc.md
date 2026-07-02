# 企业微信文档管理

> 公共概念与规则请参考 [wecom-shared.md](wecom-shared.md)
>
> 相关品类入口：
> - 表格（在线表格）→ [wecom-sheet.md](wecom-sheet.md)
> - 智能表格 → [wecom-smartsheet.md](wecom-smartsheet.md)
> - 智能文档（智能主页）→ [wecom-smartpage.md](wecom-smartpage.md)

管理企业微信文档的创建、读取和编辑。文档接口支持通过 `docid` 或 `url` 二选一定位文档。

## 调用方式

通过 `wecom-cli` 调用，品类为 `doc`：

```bash
wecom-cli doc <tool_name> '<json_params>'
```

---

## URL 品类识别

企业微信文档有四种品类，**URL 格式不同，读取内容所用的接口也不同**，切勿混用：

| URL 模式 | 品类 | 入口文件 |
|---|---|---|
| `https://doc.weixin.qq.com/doc/*` | **文档**（doc_type=3） | 本文件 |
| `https://doc.weixin.qq.com/sheet/*` | **表格 / 在线表格** | [wecom-sheet.md](wecom-sheet.md) |
| `https://doc.weixin.qq.com/smartsheet/*` | **智能表格**（doc_type=10） | [wecom-smartsheet.md](wecom-smartsheet.md) |
| `https://doc.weixin.qq.com/smartpage/*` | **智能文档**（原名智能主页） | [wecom-smartpage.md](wecom-smartpage.md) |

**判断规则**：URL 路径以 `/doc/*` 开头 → 文档 → 使用本文件接口。

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

## 接口列表

### get_doc_content

获取文档的完整内容数据，统一以 Markdown 格式返回。采用**异步轮询机制**：首次调用无需传 `task_id`，接口返回 `task_id`；若 `task_done` 为 false，需携带该 `task_id` 再次调用，直到 `task_done` 为 true 时返回完整内容。

- 首次调用（不传 task_id）：
```bash
wecom-cli doc get_doc_content '{"docid": "DOCID", "type": 2}'
```
- 轮询（携带上次返回的 task_id）：
```bash
wecom-cli doc get_doc_content '{"docid": "DOCID", "type": 2, "task_id": "xxx"}'
```
- 通过 URL 读取文档：
```bash
wecom-cli doc get_doc_content '{"url": "https://doc.weixin.qq.com/doc/xxx", "type": 2}'
```

### create_doc

新建文档（doc_type=3）。创建成功返回 url 和 docid。

```bash
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "项目周报"}'
```

参见 [API 详情](wecom-doc-create-doc.md)。

### edit_doc_content

用 Markdown 内容覆写文档正文。`content_type` 固定为 `1`（Markdown）。

```bash
wecom-cli doc edit_doc_content '{"docid": "DOCID", "content": "# 标题\n\n正文内容", "content_type": 1}'
```

参见 [API 详情](wecom-doc-edit-doc-content.md)。

---
