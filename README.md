# 企业微信套件 Skill (WeCom-Unified)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> 💬 扫码加入企业微信交流群：
>
> <img src="https://wwcdn.weixin.qq.com/node/wework/images/202603241759.3fb01c32cc.png" alt="扫码入群交流" width="200" />

企业微信 CLI 套件，覆盖文档、智能表格、消息、日程、会议、待办、通讯录等业务功能。支持新建和读取文档/智能表格/智能文档、收发单聊和群聊消息、新建和获取日程、预约和修改会议、新建和更新待办、获取通讯录成员信息，以提高个人或小团队场景、企业场景下的办公效率。

## 功能范围

覆盖企业微信核心业务品类：

| 品类        | 能力                                                                          |
| ----------- | ----------------------------------------------------------------------------- |
| 📄 文档     | 文档创建、读取、编辑等；智能文档创建、读取  |
| 📊 智能表格 | 智能表格创建、子表与字段管理、记录增删改查等                |
| 💬 消息     | 会话列表查询、消息记录拉取（文本/图片/文件/语音/视频）、多媒体下载、发送文本等 |
| 👤 通讯录   | 获取可见范围成员列表、按姓名/别名搜索等                                       |
| ✅ 待办     | 创建/读取/更新/删除待办，变更用户处理状态等                                   |
| 🎥 会议     | 创建预约会议、取消会议、更新受邀成员、查询列表与详情等                        |
| 📅 日程     | 日程增删改查、参与人管理、多成员闲忙查询等                                    |

## 适用场景

### 企业场景

对于 10 人以上规模的企业，企业微信为 API 模式智能机器人提供了文档和待办CLI 能力，机器人可以创建及读取文档、创建并跟进待办，提高企业场景下的办公效率。

### 个人/小团队场景

对于 10 人及以下的个人/小团队，企业微信为 API 模式智能机器人提供了消息、文档、日程、会议、待办等 CLI 能力，以满足个人或小团队提效场景。


## 快速开始

### 前置条件

- 支持平台：macOS (x64/arm64)、Linux (x64/arm64) 及 Windows (x64)
- Node.js `>= 18`
- 企业微信账号

### 安装 Skill

支持 WorkBuddy、CodeBuddy、Claude Code 等平台，一行命令自动检测已安装的平台并完成安装：

```bash
npx skills add WecomTeam/wecom-unified -y -g
```

安装完成后，AI 编程助手将自动获得企业微信操作能力。首次使用时，Skill 会自动检查并安装所需的 CLI 工具，并引导完成凭证配置。

### 使用方式

安装 Skill 后，直接用自然语言向 AI 助手提出需求即可，AI 助手会自动调用对应的企业微信能力完成操作。

## 项目结构

```
wecom-unified/
├── LICENSE
├── README.md
└── skills/
    └── wecom-unified/
        ├── SKILL.md              # Skill 主文件（指令与配置）
        └── references/           # 各业务域参考文档
            ├── wecom-contact.md
            ├── wecom-msg.md
            ├── wecom-doc.md
            ├── wecom-schedule.md
            ├── wecom-meeting.md
            ├── wecom-todo.md
            ├── wecom-shared.md
            └── ...
```

## 许可证

本项目基于 [MIT 许可证](./LICENSE) 开源。
