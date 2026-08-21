# 企业微信会话与消息读取（chat）

`chat` 资源用于**只读**地查询会话与消息：列出机器人所在、近期有消息的群聊会话，以及拉取指定会话在时间窗内的消息记录。本资源**只负责读取，不发送消息**；发送请走 `message` 域（见 [wecomcli-message.md](wecomcli-message.md)）。

> 注意：本资源是 `wecom-cli` 中**未被早期文档覆盖**的部分（原 SKILL 业务域概览未列 `chat`）。以下行为均经实机验证：7 天时间窗限制、bot 视角范围、以及 `853006` 权限分水岭。

## 子资源

| 子资源 | 命令 | 作用 |
|---|---|---|
| `groups` | `wecom-cli chat groups list` | 按时间范围分页获取有消息的会话列表（**目前仅群聊**） |
| `messages` | `wecom-cli chat messages list` | 按时间范围分页拉取**指定会话**的消息列表 |

## 适用范围

### 适用

- 列出机器人所在、且在查询时间窗内有消息活动的群聊（`chat groups list`）。
- 拉取某群聊/单聊在 7 天窗内、机器人即时收到的消息（`chat messages list`）。
- 结合两者：先 `groups list` 取 `chat_id`，再 `messages list --chat-id` 取该会话消息。

### 不适用 / 限制

- **不能列出授权人全部群**：只返回 bot 自身所在、近期有消息的群，看不到授权人其他未加 bot 的群。
- **不能回溯任意历史**：受 7 天时间窗限制；更深的历史（数月/年）以及 bot 不在的群，会触发企业权限限制（见下文 `853006`）。
- **不含单聊会话列表**：`groups list` 当前仅群聊；单聊消息可经 `messages list --chat-id <对方userid>` 直接拉（需对方在机器人可接收范围）。

## 一、列出群聊会话 — `chat groups list`

### 命令

```bash
wecom-cli chat groups list \
  --begin-time "YYYY-MM-DD HH:MM:SS" \
  --end-time "YYYY-MM-DD HH:MM:SS" \
  --page-count 100 \
  -o groups.ndjson
```

### 参数

| 参数 | 必填 | 说明 |
|---|:---:|---|
| `--begin-time` | 是 | 查询开始时间，格式 `YYYY-MM-DD HH:MM:SS`，**仅支持最近 7 天** |
| `--end-time` | 是 | 查询结束时间，格式同上，须晚于 `begin-time` |
| `--cursor` | 否 | 翻页游标，传上一次返回的 `next_cursor`；不传则从最新一页开始 |

### 通用选项

| 选项 | 说明 |
|---|---|
| `--page-count <N>` | 启用自动分页，连续拉取 N 页，输出 NDJSON（每行一条 JSON） |
| `--page-delay <MS>` | 分页请求间隔毫秒，默认 100 |
| `-o, --output <FILE>` | 将响应体写入文件 |
| `--output-dir <DIR>` | 将响应与附件写入目录 |
| `--dry-run` | 仅在本地校验请求，不实际发送 |

### 返回

| 字段 | 类型 | 说明 |
|---|---|---|
| `chats` | array | 会话列表，按最后消息时间从新到旧；具体数量以实际回包为准 |
| `chats[].chat_id` | string | 群会话 ID（内部标识，禁止面向用户展示） |
| `chats[].chat_name` | string | 群名称（展示用） |
| `chats[].last_msg_time` | string | 最后一条消息时间，`YYYY-MM-DD HH:MM:SS` |
| `chats[].msg_count` | integer | 该时间窗内消息条数 |
| `chats_count` | integer | `chats` 数组元素数量 |
| `has_more` | bool | 是否还有下一页 |

### 关键约束

- **7 天时间窗**：`begin-time` 不能早于「当前时间往前 7 天」，否则报 `850016 begin_time不能早于7天前`；`end-time` 须晚于 `begin-time`。建议把 `begin-time` 设为「当前时间 − 约 6 天」并留余量，避免边界被拒。
- **仅群聊 + 需有消息**：只返回窗内有消息的群。新 bot 往往不在任何群、或窗内无活动 → `chats_count=0`。把 bot 加进群并在窗内发一条消息后，下一次查询才会出现。
- **bot 视角**：返回的群是「bot 所在且近期有消息」的交集，不是授权人全部群。

## 二、拉取指定会话消息 — `chat messages list`

### 前置条件

调用前必须先取得可信的 `chat_id`：优先来自**本次** `chat groups list` 返回的 `chats[].chat_id`；单聊则传对方成员的 `userid`。不要直接使用历史上下文保存的 `chat_id` 或用户手填的 ID（这些行为在其他域被明确禁止；读取场景同样应取自当次返回）。

### 命令

```bash
wecom-cli chat messages list \
  --chat-id "<本次 groups list 返回的 chat_id>" \
  --begin-time "YYYY-MM-DD HH:MM:SS" \
  --end-time "YYYY-MM-DD HH:MM:SS" \
  --page-count 100 \
  -o msgs.ndjson
```

### 参数

| 参数 | 必填 | 说明 |
|---|:---:|---|
| `--chat-id` | 是 | 会话 ID：单聊传对方成员 `userid`，群聊传群会话 ID（由框架解密为 `{id, type}`） |
| `--begin-time` | 是 | 拉取开始时间，格式 `YYYY-MM-DD HH:MM:SS`，**仅支持最近 7 天** |
| `--end-time` | 是 | 拉取结束时间，格式同上，须晚于 `begin-time` |
| `--cursor` | 否 | 翻页游标，传上一次返回的 `next_cursor`；不传则从最新一页开始 |

通用选项同 `groups list`（`--page-count` / `--page-delay` / `-o` / `--dry-run` 等）。

### 返回

| 字段 | 类型 | 说明 |
|---|---|---|
| `messages` | array | 消息列表，按时间从新到旧；具体数量以实际回包为准 |
| `messages[].userid` | string | 发送者成员 ID（内部标识，禁止展示） |
| `messages[].user_name` | string | 发送者显示名（展示用） |
| `messages[].send_time` | string | 发送时间，`YYYY-MM-DD HH:MM:SS` |
| `messages[].msg_type` | string | 消息类型（如 `text` / `image` 等） |
| `messages[].text.content` | string | 文本消息正文；非文本消息该字段可能缺失，附件类以 `media_id` 等形式返回 |
| `messages_count` | integer | `messages` 数组元素数量 |
| `has_more` | bool | 是否还有下一页 |

### 关键约束

- **7 天时间窗**：与 `groups list` 一致，`begin-time` 不能早于 7 天前，否则报 `850016`。
- **权限分水岭 `853006`**：若目标群 bot 不在即时接收范围、或请求落到「会话内容存档」历史回溯路径，会返回 `errcode: 853006 — this tool is not available for your corporation`。含义是**企业未开通「会话内容存档」权限**，属于企业微信后台功能开通问题（通常需企业认证 + 付费 + 合规双重同意），**客户端参数无法绕过**。

  - 实测结论：`bot 在群内 + 7 天窗内 + 即时收到的消息` → 可读（返回正常）；`主动回溯历史 / bot 不在的群` → 可能触发 `853006`。
  - 与「发送消息」是两个独立权限平面：bot 能发文（普通机器人能力），不代表能读历史（需会话存檔增值能力）。
- 图像/文件等消息可能仅返回媒体标识，需结合 `media` 域（[wecomcli-media.md](wecomcli-media.md)）下载。

## 通用约束（与 message 域一致）

- `chat_id` / `userid` / `media_id` 等内部标识**禁止面向用户展示**，回覆一律用可读名称（`chat_name` / `user_name` 等）。详见 SKILL.md「通用输出约束」。
- 接口失败时如实转达错误码与含义，**不使用 curl / Python 等方式绕过 `wecom-cli`**。
- 不编造消息 ID 或会话 ID。
- 读到的消息仅作展示/分析，不静默截断；如需更长历史请提示用户该接口受 7 天与 `853006` 限制。
