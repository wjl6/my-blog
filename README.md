# 我的技术知识库

记录工作进展、学习收获、技术难题及解决方案、专业术语，逐步整理为可检索、可复用的个人知识，并沉淀为个人博客文章。

内容按三条主线组织：工作记录按时间查，长期知识按主题查，博客文章按阅读场景查。已有面试、学习和项目文章继续保留原路径。

## 开始使用

所有 AI 助手和自动化编辑任务开始前必须阅读 [AGENTS.md](AGENTS.md)，其中统一规定隐私、模拟截图、写作、修改范围和交付要求。

1. 阅读[知识库建设计划](docs/knowledge-base-plan.md)，选择每天简记或每周集中整理的节奏。
2. 对照[目录结构与写作规范](docs/directory-design.md)，确定内容归属、命名和元数据。
3. 从下表复制模板，填写真实内容；没有内容的栏目可以删除。

| 要记录什么 | 入口 | 模板 |
| --- | --- | --- |
| 今天做了什么、遇到什么阻塞、学到什么 | [日常记录](journal/README.md) | [每日记录](templates/daily.md) |
| 本周成果、关键收获、遗留问题、下周计划 | [日常记录](journal/README.md) | [每周复盘](templates/weekly.md) |
| 某项技术的原理、用法、取舍和边界 | [主题知识](knowledge/README.md) | [专题笔记](templates/note.md) |
| 某个问题如何定位、解决和验证 | [主题知识](knowledge/README.md) | [问题与方案](templates/solution.md) |
| 专业术语的中英文、含义和例子 | [主题知识](knowledge/README.md) | [术语卡片](templates/term.md) |
| 面向读者的教程、技术文章、项目经验 | [已有文章](posts/) | [博客文章](templates/post.md) |

## 仓库结构

下面展示当前已建立的入口。日期目录、知识主题目录及文章文件在首次写作时按需创建，完整示例见[目录设计](docs/directory-design.md)。

```text
.
├── AGENTS.md                     # AI 协作与内容开发规范
├── README.md
├── docs/                         # 建设计划、目录规范和内容检查记录
├── templates/                    # 六类写作模板
├── journal/                      # 每日记录和每周复盘入口
│   └── README.md
├── knowledge/                    # 专题笔记、问题方案和术语入口
│   ├── README.md
│   └── notes/tools/              # 工程实践与知识表达
├── posts/                        # 现有文章及后续博客文章
│   ├── README.md                 # 文章分类与标签索引
│   ├── interview/                # 面试知识整理
│   ├── learning/                 # 教程与学习文章
│   └── my-projects/              # 个人项目实践
├── assets/
│   └── images/                   # 文章图片
└── .gitignore
```

`posts/open-source-contributions/` 留作开源贡献文章分类，有第一篇文章时再创建。

## 写作约定

- 日报和周报记录结果与链接，原理、排障过程、术语各维护一份正文。
- 新内容使用标准 Markdown、相对链接和统一元数据，目录只决定主归属，标签用于跨主题检索。
- 可跟踪目录中的内容应先完成脱敏。原始工作日志及附件放在本地 `private/`，该路径已有忽略规则，需要单独备份。
- `visibility` 和 `status` 是内容约定，不控制 Git 仓库访问权限；未来建站时需实现显式发布筛选。

## 本地保留与版本管理

| 内容 | 存放位置 | 版本管理 |
| --- | --- | --- |
| 原始工作日志、未脱敏记录 | `private/journal/` | 仅本地保存，单独备份 |
| 真实截图、原始日志附件 | `private/assets/` | 仅本地保存，单独备份 |
| 私有导出文件、临时资料 | `private/exports/` | 仅本地保存，单独备份 |
| 含真实配置或凭证的环境文件 | `.env`、`.env.*` | 已忽略，不进入版本库 |
| 环境配置示例 | `.env.example`、`.env.*.example`、`.env.sample`、`.env.template` | 可以纳入版本管理，只允许占位符 |
| AI 规范、计划、模板、索引与脱敏知识 | `AGENTS.md`、`docs/`、`templates/`、内容目录 | 应纳入版本管理，方便持续维护与协作 |

`journal/`、`knowledge/` 和 `posts/` 不整体忽略，可进入版本库的内容必须先脱敏。`.gitignore` 不会取消已经跟踪的文件，也不会清除 Git 历史；原始材料应从创建时就放入 `private/`，不要先写入文章再依赖忽略规则隐藏。

## 已有内容

- [MySQL 锁等待与 binlog 容量排查](posts/learning/002-%E4%BB%8EMySQL%E9%94%81%E7%AD%89%E5%BE%85%E8%B6%85%E6%97%B6%E6%8E%92%E6%9F%A5%E5%88%B0binlog%E5%AE%B9%E9%87%8F%E5%A4%B1%E6%8E%A7.md)：分层取证、安全处置、恢复验收与容量治理。
- [2026 年工程实践总结](knowledge/notes/tools/history-conversation-insights.md)：需求验收、数据链路、AI 任务、数据库与邮件排查、分页和移动端交互中的工程方法。
- [全部文章与技术主题索引](posts/README.md)：按分类和技术主题阅读，包含统一标签。
- [面试知识](posts/interview/)：Java、前端、数据库、中间件、架构、AI。
- [学习文章](posts/learning/)：技术学习与操作教程。
- [个人项目](posts/my-projects/)：项目设计、实现经验和解决方案。
- [GitHub Profile](https://github.com/wjl6)

当前文章已补齐分类、主题、标签和元数据，默认保持 `draft/private`，本次整理不等同于技术事实已审核。图片、外链及待人工复核事项见[内容检查记录](docs/content-audit.md)。

