# 知识库目录结构与写作规范

采用“时间记录、主题知识、博客文章”三条主线。目录决定一篇内容的主要归属，标签补充交叉主题，普通 Markdown 链接连接记录与知识。保留现有文章和图片路径，新内容遵循以下规范。

## 一 目标目录

下图包含未来写作示例，并不表示已经创建了这些记录。当前入口包括根目录导航、AI 协作规范、建设与目录文档、内容检查记录、六类模板、日常记录与主题知识入口，以及现有文章的[分类索引](../posts/README.md)。日期、主题、新资源和 `site/` 在有实际内容或开始建站时再创建。

```text
my-blog/
├── AGENTS.md                     # 所有 AI 的共同工作约定
├── README.md
├── docs/
│   ├── knowledge-base-plan.md
│   ├── directory-design.md
│   └── content-audit.md          # 内容与资源检查及待复核项
├── templates/
│   ├── daily.md
│   ├── weekly.md
│   ├── note.md
│   ├── solution.md
│   ├── term.md
│   └── post.md
├── journal/
│   ├── README.md
│   ├── daily/
│   │   └── 2026/11/2026-11-09.md
│   └── weekly/
│       └── 2026/2026-W46.md
├── knowledge/
│   ├── README.md
│   ├── notes/
│   │   └── backend/001-Java线程池.md
│   ├── solutions/
│   │   └── middleware/001-Redis热点缓存失效排查.md
│   └── glossary/
│       └── ai/001-检索增强生成.md
├── posts/
│   ├── README.md                 # 文章分类索引
│   ├── interview/                 # 现有路径保留
│   │   ├── java/
│   │   ├── front-end/
│   │   ├── database/
│   │   ├── middleware/
│   │   ├── architecture/
│   │   └── ai/
│   ├── learning/                  # 现有路径保留
│   ├── my-projects/               # 现有路径保留
│   └── open-source-contributions/ # 有首篇内容时创建
├── assets/
│   └── images/
│       ├── interview/             # 现有图片路径保留
│       ├── learning/              # 现有图片路径保留
│       └── redis-hot-key-expiry/  # 新图片按内容短名组织
├── private/                      # 本地原始材料，已被 Git 忽略
│   ├── journal/
│   └── assets/
├── site/                         # 后续博客展示层
└── .gitignore
```

## 二 内容放在哪里

| 内容 | 位置与类型 | 写作范围 |
| --- | --- | --- |
| 每日记录 | `journal/daily/`，`type: daily` | 进展、结果、阻塞、收获、下一步，详细技术过程通过链接引用 |
| 每周复盘 | `journal/weekly/`，`type: weekly` | 本周成果、关键收获、遗留问题、知识整理、下周重点 |
| 专题笔记 | `knowledge/notes/<topic>/`，`type: note` | 一个相对独立的原理或用法，包含取舍和适用边界 |
| 问题方案 | `knowledge/solutions/<topic>/`，`type: solution` | 现象、环境、证据、根因、处理、验证与预防 |
| 术语卡片 | `knowledge/glossary/<topic>/`，`type: term` | 中文名、英文名、缩写、准确含义、例子和易混概念 |
| 博客文章 | `posts/<category>/`，`type: post` | 面向读者的完整表达，包含必要上下文和可复用结论 |

知识主题优先使用 `backend`、`frontend`、`database`、`middleware`、`architecture`、`ai`、`devops`、`tools`。有内容再创建目录；跨主题内容选择最主要的一个目录，其余用标签表达。现有 `posts/interview/java/` 和 `front-end/` 等路径继续保留，不为统一命名而移动。

文章同时使用 `category` 表达阅读分类，取值为 `interview`、`learning`、`my-projects` 或 `open-source-contributions`；`topic` 表达上面的技术主题。例如 MyBatis-Plus 属于 `category: interview`、`topic: backend`，原有 `posts/interview/middleware/` 路径为兼容旧链接保留。分类与主题以元数据为准，文件位置不替代技术归属。

同一概念的原理放专题笔记，某次排障过程放问题方案，简短定义放术语卡片，互相引用。已有卡片覆盖相同内容时直接更新；仅出现不同版本、场景或根因且确实需要分开时，才创建新条目。零散想法先放日周记录的待整理区，不预建大量空目录。

## 三 文件与日期命名

- 新增文章、专题笔记、问题方案和术语卡片采用 `三位编号-中文主题.md`，例如 `001-Java线程池.md`。编号在同一目录内递增，保留技术名的准确大小写，避免空格；不为插入新内容重新编号或批量移动存量文件。
- `README.md`、`AGENTS.md`、配置及工具约定的固定文件名保留各自约定，日报和周报继续按日期命名。
- 日报使用 `journal/daily/YYYY/MM/YYYY-MM-DD.md`。
- 周报使用 `journal/weekly/周所属年份/YYYY-Www.md`，周一到周日为一周，采用 ISO 周编号。跨年周使用周所属的 ISO 年，并在正文注明起止日期；例如 2027 年 1 月 1 日属于 `2026/2026-W53.md`。
- 专题、方案和术语使用准确的中文主题，例如 `001-Java线程池.md`、`001-Redis热点缓存失效排查.md`、`001-检索增强生成.md`。
- 博客文章沿用所在分类的编号和中文主题格式，例如 `posts/learning/002-从MySQL锁等待超时排查到binlog容量失控.md`；时间归档交给元数据，不增加年份目录。
- 文件名与公开地址分开维护，`slug` 继续使用唯一、稳定的英文短名，不随文件改名自动变化。图片资源目录和文件仍按资源短名组织，避免把内容编号当作资源身份。
- 新记录日期按北京时间填写。`date` 为首次记录或文章日期，存量日期时间完整保留，不丢弃时分秒、不补猜测的时区；`updated` 为最近实质整理日期，不因修正空格反复更新，也不代表技术事实已经核验。

## 四 元数据约定

新内容在文件第一行使用 YAML front matter。复制模板后替换日期和占位标题，不能把 `YYYY-MM-DD` 当成真实日期保存。

```yaml
---
title: "Redis 热点缓存失效问题复盘"
date: "2026-11-09"
updated: "2026-11-09"
type: solution
tags: [redis, cache]
status: draft
visibility: private
---
```

| 字段 | 规则 |
| --- | --- |
| `title` | 必填。可读的中文标题，不包含文件编号或 `.md` |
| `date` | 必填。新内容用带引号的 `YYYY-MM-DD`，存量完整保留带引号的原始日期时间，后续通过接入层处理 |
| `updated` | 可选。发生实质整理时填写带引号的 `YYYY-MM-DD`，不覆盖原始 `date`，不代表技术事实已核验 |
| `type` | 必填。`daily`、`weekly`、`note`、`solution`、`term`、`post` 之一 |
| `category` | `post` 必填。`interview`、`learning`、`my-projects`、`open-source-contributions` 之一 |
| `topic` | `post` 必填，其他知识内容可选。使用约定的技术主题；复制文章模板时必须填写 |
| `tags` | 必填。数组，初稿允许为空，准备公开时补齐准确标签 |
| `status` | 必填。`draft` 为整理中，`ready` 为已核对可发布，`published` 为已在博客发布 |
| `visibility` | 必填。`private` 或 `public`，模板默认 `private` |
| `slug` | 准备公开时必填。全站唯一的英文短名，作为稳定地址标识 |
| `summary` | 准备公开时必填。用一两句话说明问题与结论，供列表、搜索、订阅展示 |
| `aliases` | 术语可选。中文名、英文全称、常见缩写等检索别名数组 |

`ready` 表示内容已核对，不代表已经部署；发布后才改为 `published`。已发布文章继续使用相同 `slug`，重要技术结论尚未核实的修改先保留在草稿中。需要撤下文章时移出公开集合，并在后续发布中检查页面与索引是否移除。

这些字段是本仓库的写作协议，博客框架不会自动理解全部字段。以后需要明确将 `date`、`summary`、`slug` 等映射为所选框架使用的字段，并实现状态筛选。`visibility: private` 不会隐藏 Git 文件、禁止他人读取，也不能代替脱敏。

### 标签与检索

每篇内容通常使用 2 至 5 个准确标签，例如 `[java, spring-boot]`、`[redis, cache]`、`[ai, rag]`。初稿不强求数量，不使用“学习”“其他”“技术”等泛标签。

同一标签统一小写，例如 `spring-boot`、`typescript`、`postgresql`。新增标签前先搜索已有写法，中文别名可在正文或术语 `aliases` 中出现。文件夹负责主要分类，`type` 表达内容类型，标签只补充技术主题。

当前可直接使用编辑器全局搜索标题、中文术语、缩写、标签和报错关键词。后续博客再基于统一元数据生成标签和归档页面，不要求日常手工维护全量文章清单。

## 五 正文与资源规则

每篇正文用一个一级标题，与 `title` 保持一致；二级标题组织主要内容。开头先交代问题或核心结论，代码块注明语言，示例标明适用版本与必要前提。模板中没有实际内容的可选栏目直接删除。

关联内容使用相对路径和 `.md` 后缀，例如从 `knowledge/notes/backend/` 引用 AI 术语：

```markdown
[检索增强生成](../../glossary/ai/001-检索增强生成.md)
```

只有目标文件真实存在时才添加正式链接；未完成的知识写入待整理项。当前选择这种写法便于仓库阅读，未来建站需在接入层处理目标路由，不假设所有框架都自动重写链接。

新图片放在 `assets/images/<内容短名>/`，例如 `assets/images/redis-hot-key-expiry/request-flow.png`。从 `knowledge/solutions/middleware/001-Redis热点缓存失效排查.md` 引用时使用：

```markdown
![请求流程](../../../assets/images/redis-hot-key-expiry/request-flow.png)
```

图片使用有意义的文件名与替代文字，减少难以检索的纯截图。现有图片路径保持不变。未脱敏截图和日志附件放在本地 `private/assets/`；公开内容不能依赖私有文件或不可访问的内网链接。

新增截图必须使用隔离演示环境与完全模拟数据，并注明“模拟数据”或“演示环境”；不能对真实业务截图简单遮挡后当作模拟图。还需检查地址栏、本机路径、终端、通知、二维码和图片元信息。历史图片的出处与使用条件不明时保留待复核状态，不能去水印或当作原创，详见 [AI 协作规范](../AGENTS.md)。

## 六 发布边界与后续迁移

建站阶段以显式允许的目录或候选清单选取正文，再同时检查 `visibility: public` 与 `status: ready/published`。缺少字段、字段值无效或仍有占位符的内容排除；路径、字段、唯一 `slug` 和摘要均通过核对后再进入公开集合。

- 首批只接入人工选定的 `posts/` 内容；公开知识页或周记另行加入，日报默认不发布。
- `docs/`、`templates/`、`private/`、各目录 `README.md` 和未来 `site/` 的工程文件不作为博客正文。
- 正文、站内链接、搜索、标签、归档、RSS 和站点地图使用同一公开集合，公开页不得链接到未发布笔记。
- 资源按已公开正文的实际引用复制，先核对资源可公开性，不把整个仓库或整个图片目录直接作为公开静态目录。
- 原有文章补齐字段后才进入候选清单。移动文件需检查相对链接、图片和公开地址；已公开地址保持稳定，必要时设置重定向。

当前阶段整理内容、索引与规范，上述站点规则在以后建站时实现。当前原始工作材料的存放边界以既有 `private/` 忽略规则和仓库访问权限为准。
