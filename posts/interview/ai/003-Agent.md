---
title: "Agent 与 RAG 文档切分实践"
date: "2026-01-06 20:52:18"
updated: "2026-11-09"
type: post
category: interview
topic: ai
tags: [agent, rag, mcp, milvus]
slug: "agent-rag-document-chunking"
summary: "整理智能体平台比较、API 服务适配思路与 RAG 文档切分策略，并以小说内容为示例说明结构切分、语义处理和向量入库。"
status: draft
visibility: private
---

# Agent 与 RAG 文档切分实践

## 1. Dify、扣子、HiAgent 有什么共同点和不同点？
| 维度         | Dify                                  | 扣子（Coze）| HiAgent                                |
|--------------|---------------------------------------|---------------------------------------|----------------------------------------|
| 面向人群     | 开发者                                | 普通用户 / 运营                       | 企业                                   |
| 是否开源     | ✅ 是                                 | ❌ 否                                 | ❌ 否                                   |
| 私有化部署   | ✅ 强                                 | ❌ 几乎无                             | ✅ 必须                                 |
| 低代码       | 中                                    | 强                                    | 强                                     |
| 工程可控性   | ⭐⭐⭐⭐                                | ⭐⭐                                   | ⭐⭐⭐⭐⭐                                 |
| 企业治理     | 中                                    | 弱                                    | 极强                                   |
| 内容 & 分发  | 弱                                    | 极强                                  | 弱                                     |
| 典型场景     | AI SaaS / 内部系统                    | 聊天机器人 / 玩法                     | 数字员工 / 企业流程                    |

## 2. 如何将本地 API 接口封装为 MCP 服务？

核心是让你的 **API 接口适配 MCP 协议规范，对外提供标准化的 RPC 调用能力。**

### 一、核心前提与技术选型
1. 核心思路  
MCP 本质是基于 HTTP/JSON 或 gRPC 的标准化协议，封装 Java 接口为 MCP 服务的核心是：
- 定义符合 MCP 规范的接口（入参 / 出参、请求方法）；
- 搭建 MCP 服务端框架，将本地 Java 接口映射为 MCP 接口；
- 暴露 HTTP/gRPC 端口，让外部通过 MCP 协议调用你的 Java 接口。
2. 推荐技术栈（适配 Java 生态）  

| 组件         | 作用                          | 选择理由                                   |
|--------------|-------------------------------|--------------------------------------------|
| Spring Boot  | 快速搭建 Java 服务            | 生态完善，易集成各类插件                   |
| Spring MVC   | 处理 HTTP 请求（MCP 基础版）| 适配 MCP 的 HTTP/JSON 协议                 |
| gRPC         | 处理高性能 RPC（MCP 进阶版）| 符合 MCP 高性能调用需求                     |
| MCP SDK      | 简化 MCP 协议适配             | 避免手动写协议解析逻辑                     |

> 注：目前主流的 MCP 实现以 Python 居多，Java 生态可优先基于「HTTP/JSON 版 MCP 规范」封装，降低复杂度。

### 二、分步实现：将本地 Java 接口封装为 MCP 服务
假设你有一个本地 Java 接口 UserService，功能是根据用户 ID 查询用户信息，我们以此为例完成封装。
#### 步骤 1：定义本地 Java 接口（待封装的核心逻辑）
先准备好你的业务接口和实现类：
```java
// 1. 定义用户实体
public class User {
    private Long id;
    private String name;
    private Integer age;
    // 省略 getter/setter/构造方法
}

// 2. 本地业务接口
public interface UserService {
    // 核心方法：根据ID查用户
    User getUserById(Long id);
}

// 3. 接口实现类
@Service
public class UserServiceImpl implements UserService {
    @Override
    public User getUserById(Long id) {
        // 模拟本地业务逻辑（实际可能是查数据库/调用其他本地接口）
        return new User(id, "张三", 25);
    }
}
```
#### 步骤 2：引入 MCP 相关依赖（Spring Boot 项目）
在 pom.xml 中添加核心依赖（以 HTTP/JSON 版 MCP 为例）：
```xml
<dependencies>
    <!-- Spring Boot Web 核心（处理 HTTP 请求） -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- JSON 序列化（适配 MCP 协议的 JSON 格式） -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    <!-- 可选：MCP 官方 Java SDK（若有，优先用） -->
    <!-- <dependency>
        <groupId>com.modelcontext</groupId>
        <artifactId>mcp-sdk-java</artifactId>
        <version>最新版本</version>
    </dependency> -->
</dependencies>
```

#### 步骤 3：定义 MCP 协议适配层（核心）
MCP 协议要求请求 / 响应有固定格式，我们需要封装「MCP 请求对象」「MCP 响应对象」，并编写 Controller 映射本地接口：
- 3.1 定义 MCP 标准请求 / 响应对象
```java
// MCP 请求对象（符合 MCP 协议规范）
public class MCPRequest {
    // MCP 协议固定字段：方法名（对应本地接口方法）
    private String method;
    // MCP 协议固定字段：方法入参（JSON 格式）
    private Map<String, Object> params;
    // 省略 getter/setter
}

// MCP 响应对象（符合 MCP 协议规范）
public class MCPResponse<T> {
    // 状态码：0=成功，非0=失败
    private Integer code;
    // 提示信息
    private String msg;
    // 响应数据（本地接口返回结果）
    private T data;
    // 省略 getter/setter
}
```
- 编写 MCP 服务端 Controller（映射本地接口）
```java
@RestController
@RequestMapping("/mcp") // MCP 服务根路径
public class MCPController {

    // 注入本地业务接口
    @Autowired
    private UserService userService;

    // MCP 接口入口：接收 MCP 协议请求，调用本地接口
    @PostMapping("/invoke")
    public MCPResponse<?> invoke(@RequestBody MCPRequest request) {
        try {
            // 1. 解析 MCP 请求：获取方法名和入参
            String method = request.getMethod();
            Map<String, Object> params = request.getParams();

            // 2. 根据方法名路由到本地接口
            Object result = switch (method) {
                case "getUserById" -> {
                    // 解析入参：获取用户 ID
                    Long userId = Long.parseLong(params.get("id").toString());
                    // 调用本地接口
                    yield userService.getUserById(userId);
                }
                // 扩展：添加其他本地接口方法的路由
                default -> throw new IllegalArgumentException("不支持的 MCP 方法：" + method);
            };

            // 3. 封装 MCP 成功响应
            MCPResponse<Object> response = new MCPResponse<>();
            response.setCode(0);
            response.setMsg("success");
            response.setData(result);
            return response;

        } catch (Exception e) {
            // 4. 封装 MCP 失败响应
            MCPResponse<Object> errorResponse = new MCPResponse<>();
            errorResponse.setCode(500);
            errorResponse.setMsg("调用失败：" + e.getMessage());
            errorResponse.setData(null);
            return errorResponse;
        }
    }
}
```
#### 步骤 4：配置并启动 MCP 服务

---

## 3. RAG 文本如何切分？

切分直接影响三件事：
- 召回准不准（向量检索）
- 上下文是否完整（回答是否“断片”）
- Token 成本 & 延迟

切得不好，典型症状是：
- 回答像“只看了半页说明书”
- 明明文档里有答案，却总是“答非所问”
- 模型胡编（因为上下文不够）

### 文本切分的 4 种常见方式
#### 1️⃣ 固定长度切分（最基础，不推荐单独用）
做法
- 每 N 个字符 / Token 切一段
- 可加 overlap（重叠）
```text
chunk_size = 500 tokens
chunk_overlap = 50
```
优点
- 简单、快
- 各种框架都支持（LangChain、LlamaIndex）

致命问题
- 语义被硬切
- 标题和内容分家
- 问“怎么配置 X”，返回的是“注意事项的后半段”

> 👉 只适合 demo，不适合生产

#### 2️⃣ 按结构切分（文档级必用）
适合：
- Markdown
- 技术文档
- 接口文档
- 博客

切分依据
- 标题（# ## ###）
- 段落
- 列表
- 代码块

示例（Markdown）：
```markdown
## 数据库配置
这里是数据库配置说明...

### MySQL
...

### PostgreSQL
...
```
👉 切分单位：

- \## 数据库配置 作为一个 chunk

- \### MySQL / ### PostgreSQL 各自独立

好处

- 语义完整
- 非常适合知识库问答

#### 3️⃣ 语义切分（智能体推荐）
**核心思想：不看长度，看“语义是否换话题了”**
常见实现方式：
- 基于句子
- 滑动窗口 + embedding 相似度
- Topic change detection

简单理解
- 连续句子 embedding 相似度下降 → 切
- 语义发生明显转折 → 切

效果
- 一段话只讲一件事
- 特别适合 FAQ、制度、说明文

#### 4️⃣ 场景化切分（智能体才会用）
这是很多人没意识到的一层 👇
> 同一份文档，切法应该随“使用场景”变化

举例：用户手册

| 场景       | 切分策略              |
| -------- | ----------------- |
| FAQ 问答   | 一问一答一 chunk       |
| 操作步骤     | 一个步骤链一个 chunk     |
| API 查询   | 一个接口一个 chunk      |
| Agent 执行 | 一个“可行动单元”一个 chunk |

👉 对智能体来说：
> 一个 chunk = 一个“可被引用 / 推理 / 执行”的知识单元

### 智能体常用的“黄金切分策略”

#### ✅ 推荐组合
```text
1. 先按文档结构切（标题 / 段落）
2. 每块限制在 300~800 tokens
3. 超长块 → 再做语义切分
4. 保留 10%~20% overlap
5. 给每个 chunk 补充元信息
```

#### 在 RAG / Agent 中怎么选 chunk 大小？
| 场景         | chunk 大小       |
| ---------- | -------------- |
| FAQ / 制度   | 200~400 tokens |
| 技术文档       | 400~800 tokens |
| API 文档     | 300~600 tokens |
| Agent 工具说明 | 一个“动作”一个 chunk |

⚠️ chunk 太大
- 检索命中率下降
- token 成本爆炸

⚠️ chunk 太小
- 信息碎片化
- 模型胡补上下文

### 最容易忽略的 3 个点

#### ❌ 1. 不保留标题

标题本身是超级强语义锚点
👉 建议：标题内容拼进 chunk 正文

#### ❌ 2. 不区分“知识块”和“执行块”
Agent 场景下要区分：
- 描述性文本（解释）
- 操作性文本（能执行）

这俩切法不一样。
#### ❌ 3. 只切一次
正确流程是：
```text
原始文档
  ↓
结构切分
  ↓
语义细分
  ↓
场景二次切分
```

---

## 4. 以《凡人修仙传》小说为例，如何实现文档切分及向量化？

### 目标定义（先说清楚要干嘛）
目标是：
> 用 Milvus 存储《凡人修仙传》的最终向量，用于 RAG 问答 / 人物关系理解 / 剧情回溯

不是全文检索，而是：
- “韩立第一次筑基是在哪一卷？”
- “南宫婉和韩立的关系发展脉络”
- “血色试炼发生了什么？”

👉 这决定了切分策略
### 总体切分流水线
```text
原始小说文本
  ↓
结构切分（卷 / 章）
  ↓
语义细分（事件 / 情节）
  ↓
场景二次切分（RAG 可用单元）
  ↓
Milvus 向量入库
```
#### 一、原始文档阶段（人工 + 工具）
📌 输入
- 《凡人修仙传》TXT / EPUB
- 全文约 700~800 万字

👤 人工要做的事（必须）   
✅ 人工确认这些规则：
- 一共多少卷（如：凡人篇、灵界篇）
- 章节分隔格式（如：第XXX章）
- 是否包含作者感言、广告、重复章节（要剔除）

👉 这一步必须人工
> 小说结构不统一，靠程序猜，必翻车

🛠 工具做的事
- 清洗文本
- 去广告
- 统一编码
- 标准化章节标题

产出：
```json
{
  "volume": "凡人篇",
  "chapter": "第001章 山边小村",
  "raw_text": "韩立出生在一个普通农家..."
}
```

#### 二、结构切分（强结构，工具完成）
🎯 目标  
> 不碰语义，只切“天然边界”  

切分层级  
```markdown
书
 └─ 卷
     └─ 章
```
🛠 工具自动完成
- 正则匹配：
  - 第.*章
  - 第.*卷
存储示例（关系型数据库）
```text
document
  └─ volume
      └─ chapter
```
```text
chapter
--------
id
volume_id
chapter_title
chapter_index
content
```
👉 到这一步为止：  
- 还没有任何向量  
- 不进 Milvus

#### 三、语义细分（最关键的一步）
🎯 目标
> “一章”拆成 若干个“情节块”

举个真实例子（非常重要）  
原章节（简化）
```markdown
第45章 血色试炼

1. 韩立进入秘境
2. 遇到其他修士
3. 发现灵草
4. 遭遇追杀
5. 成功逃脱
```
❌ 错误切法（很多人会这么干）
> 1000 字一切

问题：
- 一个 chunk 里混了 进入秘境 + 被追杀
- 问“血色试炼有什么危险” → 回答一半

✅ 正确语义切分（事件级）  
语义 chunk 示例
```json
{
  "event": "进入血色试炼秘境",
  "content": "韩立被传送进血色试炼..."
}

{
  "event": "遭遇同门修士",
  "content": "他在秘境中遇到了几名..."
}

{
  "event": "被追杀逃生",
  "content": "韩立因争夺灵草被追杀..."
}
```
👤 人工要做什么？   
这里是关键分水岭 👇   

人工配置的不是“切”，而是：   
✅ 定义“什么算一个事件”
- 战斗 = 一个事件
- 人物关系变化 = 一个事件
- 剧情转折 = 一个事件

你只需要给规则，不用一章一章切。

🛠 工具做什么？
- 句子分割
- embedding 相似度滑动窗口
- 语义突变检测（topic shift）

👉 自动判断：
> “这里是不是换了一件事”

产出表：
```text
semantic_chunk
---------------
id
chapter_id
event_type      -- 战斗 / 修炼 / 人物 / 剧情推进
content
token_size
```

#### 四、场景二次切分（决定进不进 Milvus，最关键一步）
> 这一步 100% 为 RAG 服务

🎯 不是所有 semantic chunk 都要进 Milvus

举例：哪些该进？
✅ 适合进 Milvus：
- 重要剧情
- 人物关系变化
- 世界观设定
- 修炼体系说明

❌ 不适合进：
- 水文日常
- 重复心理描写
- 过渡段

👤 人工要做什么？  
只做“策略配置”，不碰数据   
比如：
```text
scene_rules:
  rag:
    include_event_types:
      - 战斗
      - 修炼突破
      - 重要剧情
      - 人物关系
    max_tokens: 600
```
👉 你是在定义价值标准

🛠 工具自动完成
- 合并 / 拆分 semantic chunk
- 控制 token 大小
- 补充上下文（人物名、章节名）

最终 RAG chunk 示例
```json
{
  "content": "在血色试炼中，韩立因争夺灵草遭遇追杀...",
  "metadata": {
    "book": "凡人修仙传",
    "volume": "凡人篇",
    "chapter": "第45章",
    "characters": ["韩立"],
    "event": "血色试炼追杀"
  }
}
```

#### 五、Milvus 入库（工具完成）
content 和 metadata 中的核心语义字段整合为一段完整文本后再向量化
```text
内容：在血色试炼中，韩立因争夺灵草遭遇追杀... | 书籍：凡人修仙传 | 卷：凡人篇 | 章节：第45章 | 人物：韩立 | 事件：血色试炼追杀
```
这样做的优势是：**向量中同时包含 “情节内容” 和 “元数据上下文”**，检索时既可以搜 “韩立 血色试炼 追杀”，也可以搜 “凡人修仙传 第 45 章 灵草”，召回精度更高。

Collection 设计示例
```text
collection: fanren_rag
fields:
- id
- embedding (vector)
- content
- volume
- chapter
- characters
- event
```
👤 人工要配置的只有这些：
- embedding 模型
- 向量维度
- 相似度度量（cosine）

### 完整流程责任划分总结（重点）

| 阶段   | 人工做什么      | 工具做什么              |
| ---- | ---------- | ------------------ |
| 原始文档 | 确认结构、清洗规则  | 文本解析               |
| 结构切分 | 定义“卷/章”规则  | 正则切分               |
| 语义细分 | 定义“事件”标准   | 语义检测               |
| 场景切分 | 定义“什么值得进库” | chunk 生成           |
| 向量存储 | 模型/索引配置    | embedding + Milvus |

---













