# 企业微信表格（在线表格）管理

> 公共概念与规则请参考 [wecom-shared.md](wecom-shared.md)
>
> 相关品类入口：
> - 文档 → [wecom-doc.md](wecom-doc.md)
> - 智能表格 → [wecom-smartsheet.md](wecom-smartsheet.md)
> - 智能文档（智能主页）→ [wecom-smartpage.md](wecom-smartpage.md)

管理企业微信**表格 / 在线表格**（`/sheet/*`）的新建、内容读写以及子工作表管理。表格接口支持通过 `docid` 或 `url` 二选一定位。

>  **表格 ≠ 智能表格**：二者是不同品类（`/sheet/` vs `/smartsheet/`）。当用户说「表格」且 URL 为 `/sheet/*`，或没有提供 URL 但明显是普通在线表格场景时，走本文件接口。需要使用更结构化的字段/记录模型（字段类型、关联、视图等），属于「智能表格」场景，请改用 [wecom-smartsheet.md](wecom-smartsheet.md)。

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
| `https://doc.weixin.qq.com/sheet/*` | **表格 / 在线表格**（doc_type=4） | 本文件 |
| `https://doc.weixin.qq.com/smartsheet/*` | **智能表格**（doc_type=10） | [wecom-smartsheet.md](wecom-smartsheet.md) |
| `https://doc.weixin.qq.com/smartpage/*` | **智能文档**（原名智能主页） | [wecom-smartpage.md](wecom-smartpage.md) |

**判断规则**：URL 路径以 `/sheet/*` 开头 → 表格（在线表格） → 使用本文件接口。

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

## 接口路由表

> **硬规则**：第二列是 `references/xxx.md` 链接的，命中这一行后**先 `read` 对应 references 文件，再构造命令**。写入/读取子表数据前，先用 `sheet_get_info` 拿到目标子表的 `sheet_id`。

| 用户意图 | 参考位置                                                                 |
|---|----------------------------------------------------------------------|
| 新建空白在线表格 | 见下方「create_doc」                                                      |
| 读取在线表格完整内容（Markdown 概览） | 见下方「get_doc_content」                                                 |
| 读取在线表格基础信息与子表列表 | 见下方「sheet_get_info」                                                                  |
| 修改在线表格指定区域内容 | [wecom-sheet-update-range-data.md](wecom-sheet-update-range-data.md) |
| 在线表格末尾追加一行数据 | [wecom-sheet-append-data.md](wecom-sheet-append-data.md)             |
| 添加在线表格子工作表 | [wecom-sheet-add-sub.md](wecom-sheet-add-sub.md)                     |
| 删除在线表格子工作表 | [wecom-sheet-delete-sub.md](wecom-sheet-delete-sub.md)               |

---

## 接口列表

### create_doc

从零新建一篇企微**在线表格**：空白。创建成功后返回 `docid` 和 `url`。

```bash
wecom-cli doc create_doc '{"doc_type": 4, "doc_name": "项目排期表"}'
```

#### 关键参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `doc_type` | int | 是 | 固定传 `4`（在线表格） |
| `doc_name` | string | 是 | 表格标题 |

**注意事项**

- 本接口仅创建**空白**在线表格，不支持携带初始内容；如需写入数据，请在创建后先用 `sheet_get_info` 拿到子表 `sheet_id`，再通过 `sheet_update_range_data` / `sheet_append_data` 写入。

### get_doc_content

获取**表格（在线表格）**的完整内容数据，统一以 Markdown 格式返回。采用**异步轮询机制**：首次调用无需传 `task_id`，接口返回 `task_id`；若 `task_done` 为 false，需携带该 `task_id` 再次调用，直到 `task_done` 为 true 时返回完整内容。

#### 关键参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| docid | string | 与 url 二选一 | 文档的 docid |
| url | string | 与 docid 二选一 | 文档的访问链接 |
| type | integer | 是 | 内容返回格式，固定传 `2`（Markdown 格式） |
| task_id | string | 否 | 任务 ID，初次调用不填，后续轮询时填写上次返回的 task_id |

- 首次调用（不传 task_id）：
```bash
wecom-cli doc get_doc_content '{"docid": "DOCID", "type": 2}'
```
- 轮询（携带上次返回的 task_id）：
```bash
wecom-cli doc get_doc_content '{"docid": "DOCID", "type": 2, "task_id": "xxx"}'
```
- 通过 URL 读取表格（在线表格）：
```bash
wecom-cli doc get_doc_content '{"url": "https://doc.weixin.qq.com/sheet/xxx", "type": 2}'
```

### sheet_get_info

读取**在线表格**的基础信息，包括工作表列表、文档名称与访问链接。所有后续需要 `sheet_id` 的接口（`sheet_update_range_data` / `sheet_append_data` / `sheet_delete_sub` 等）的 `sheet_id` 都从本接口返回的 `sheets[]` 中取。

#### 命令

```bash
wecom-cli doc sheet_get_info '<JSON 参数>'
```

#### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `docid` | string | 与 `url` 二选一 | 在线表格的 docid |
| `url` | string | 与 `docid` 二选一 | 在线表格的访问链接 |

#### 返回字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `sheets` | array | 工作表列表；每项含 `sheet_id` / `title` / `row_count` / `column_count` / `data_range` 等基础信息 |
| `url` | string | 文档访问链接 |
| `name` | string | 文档名称 |

---

## 读 → 写 链路

当需要往具体子表写内容（`sheet_update_range_data` / `sheet_append_data`）时：

1. 先用 `get_doc_content` 读取并识别出各子表的标题与具体数据；
2. 再调用 `sheet_get_info` 拿到 `sheets[]` 中每个子表的 `sheet_id` 与行列数（`row_count` / `column_count`）；
3. 通过**子表标题与 `title` 匹配**确定目标 `sheet_id`，并结合行列数核对写入区域范围，从而打通「读 → 写」的完整链路。

---
