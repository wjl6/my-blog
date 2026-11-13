# 文章分类与主题索引

共 26 篇文章：面试知识 20 篇、学习教程 5 篇、项目实践 1 篇。按元数据的 `category` 和 `topic` 组织，文件及图片原路径保持不变。

当前均为 `draft/private`，表示尚未完成技术事实与公开内容复核；日期显示原始记录日期，完整时间保留在各篇元数据中。`updated` 记录本次整理时间，不表示文中步骤已重新执行。

统一规范见[目录设计](../docs/directory-design.md)与 [AI 协作要求](../AGENTS.md)，图片、外链和技术疑点见[内容检查记录](../docs/content-audit.md)。

## 面试知识

### 后端开发

| 文章 | 原始日期 | 标签 |
| --- | --- | --- |
| [Java 基础面试知识](interview/java/001-java%E5%9F%BA%E7%A1%80.md) | 2025-12-26 | java, jvm, collections, concurrency, io |
| [Java 进阶面试知识](interview/java/002-java%E8%BF%9B%E9%98%B6.md) | 2025-12-27 | java, thread-pool, annotations, reflection, dynamic-proxy |
| [Spring 与 Spring Boot 面试知识](interview/java/003-spring%20boot.md) | 2025-12-26 | java, spring, spring-boot, dependency-injection, transaction |
| [Spring Cloud 与微服务面试知识](interview/java/004-spring%20cloud.md) | 2025-12-26 | spring-cloud, microservices, service-discovery, circuit-breaker |
| [MyBatis-Plus 查询执行流程整理](interview/middleware/004-mybatis-plus.md) | 2026-01-08 | java, mybatis, mybatis-plus, sql, orm |

### 前端开发

| 文章 | 原始日期 | 标签 |
| --- | --- | --- |
| [Vue 核心原理与面试知识](interview/front-end/001-vue.md) | 2026-01-08 | vue, javascript, reactivity |
| [TypeScript 与 ECMAScript 面试知识](interview/front-end/002-ts.md) | 2026-01-11 | typescript, javascript, ecmascript |

### 数据库

| 文章 | 原始日期 | 标签 |
| --- | --- | --- |
| [MySQL 基础面试知识整理](interview/database/001-MySQL-%E5%9F%BA%E7%A1%80.md) | 2025-12-28 | mysql, sql, innodb, index, transaction |
| [MySQL 进阶面试与性能优化整理](interview/database/002-MySQL-%E8%BF%9B%E9%98%B6.md) | 2025-12-26 | mysql, sql-optimization, sharding, data-migration, locking |
| [PostgreSQL 面试知识整理](interview/database/003-PostgreSQL.md) | 2025-12-29 | postgresql, mvcc, vacuum, index, sql-optimization |

### 中间件与服务调用

| 文章 | 原始日期 | 标签 |
| --- | --- | --- |
| [消息队列与 RabbitMQ 面试知识整理](interview/middleware/001-mq.md) | 2025-12-26 | message-queue, rabbitmq, idempotency, message-reliability |
| [Redis 缓存与分布式锁面试整理](interview/middleware/002-Redis.md) | 2025-12-26 | redis, cache, distributed-lock, redisson |
| [Elasticsearch 检索与集群面试整理](interview/middleware/003-ElasticSearch.md) | 2025-12-27 | elasticsearch, full-text-search, inverted-index, search-after |
| [Dubbo 与 Feign 服务调用对比](interview/middleware/005-Dubbo.md) | 2026-01-08 | java, dubbo, rpc, feign, microservices |

### 系统架构

| 文章 | 原始日期 | 标签 |
| --- | --- | --- |
| [高并发系统设计面试知识](interview/architecture/001-%E9%AB%98%E5%B9%B6%E5%8F%91%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.md) | 2025-12-28 | high-concurrency, system-design, rate-limiting, idempotency, distributed-lock |
| [分布式事务与一致性面试知识](interview/architecture/002-%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1.md) | 2025-12-28 | distributed-transactions, consistency, tcc, saga, snowflake |

### AI 与大模型

| 文章 | 原始日期 | 标签 |
| --- | --- | --- |
| [LangChain 与 RAG 面试知识](interview/ai/001-LangChain.md) | 2025-12-26 | langchain, rag, llm, agent |
| [Milvus 向量检索面试知识](interview/ai/002-Milvus.md) | 2025-12-30 | milvus, vector-search, rag |
| [Agent 与 RAG 文档切分实践](interview/ai/003-Agent.md) | 2026-01-06 | agent, rag, mcp, milvus |
| [大模型参数高效微调与 PEFT](interview/ai/004-%E5%A4%A7%E6%A8%A1%E5%9E%8B.md) | 2026-01-22 | llm, peft, lora, fine-tuning |

MyBatis-Plus 按技术内容归入后端主题，文件继续保留在原 `interview/middleware/` 目录，以兼容历史路径。

## 学习教程

| 文章 | 原始日期 | 主题与标签 |
| --- | --- | --- |
| [Redmi AC2100 刷入 Breed 教程](learning/001-Redmi%E7%BA%A2%E7%B1%B3%E8%B7%AF%E7%94%B1%E5%99%A8AC2100%E5%88%B7Breed%E6%95%99%E7%A8%8B.md) | 2023-08-10 | tools / redmi-ac2100, router, breed, ssh |
| [从 MySQL 锁等待超时排查到 binlog 容量失控](learning/002-%E4%BB%8EMySQL%E9%94%81%E7%AD%89%E5%BE%85%E8%B6%85%E6%97%B6%E6%8E%92%E6%9F%A5%E5%88%B0binlog%E5%AE%B9%E9%87%8F%E5%A4%B1%E6%8E%A7.md) | 2026-11-10 | database / mysql, innodb, locking, binlog, incident-response |
| [MySQL 异地灾备方案设计与恢复演练](learning/003-MySQL%E5%BC%82%E5%9C%B0%E7%81%BE%E5%A4%87%E6%96%B9%E6%A1%88%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%81%A2%E5%A4%8D%E6%BC%94%E7%BB%83.md) | 2026-11-11 | database / mysql, backup, disaster-recovery, replication |
| [从指标采集到告警闭环的服务监控方案](learning/004-%E4%BB%8E%E6%8C%87%E6%A0%87%E9%87%87%E9%9B%86%E5%88%B0%E5%91%8A%E8%AD%A6%E9%97%AD%E7%8E%AF%E7%9A%84%E6%9C%8D%E5%8A%A1%E7%9B%91%E6%8E%A7%E6%96%B9%E6%A1%88.md) | 2026-11-11 | devops / prometheus, grafana, monitoring, alerting, mysql |
| [AI 文档处理的阶段契约与链路一致性](learning/005-AI%E6%96%87%E6%A1%A3%E5%A4%84%E7%90%86%E7%9A%84%E9%98%B6%E6%AE%B5%E5%A5%91%E7%BA%A6%E4%B8%8E%E9%93%BE%E8%B7%AF%E4%B8%80%E8%87%B4%E6%80%A7.md) | 2026-11-12 | ai / ai, pdf, ocr, data-pipeline, validation |

## 项目实践

| 文章 | 原始日期 | 主题与标签 |
| --- | --- | --- |
| [基于 LibreOffice 的 Word 转 PDF HTTP 服务](my-projects/001-libreoffice-pdf-service.md) | 2025-12-25 | tools / libreoffice, pdf, docker, http |

## 索引维护

新增文章或修改标题、分类、主题、标签时同步更新本索引。标签使用小写英文，具体命名优先复用本表已有写法。开源贡献目录在出现首篇真实文章时创建，再增加相应分类。
