---
title: "003-ElasticSearch.md"
date: 2025-12-27 18:34:01
tags: []
---

### 1. 为什么要用 ES？不用 MySQL 行不行？

ES 适合：
- 全文检索
- 多条件组合搜索
- 高并发读

MySQL：
- 精确查询
- 事务

👉 **ES ≠ 数据库替代品**

---

### 2. ES 的核心概念？

| ES | 类比数据库 |
|----|----|
| Index | Database |
| Type（已废弃） | Table |
| Document | Row |
| Field | Column |

---

### 3. ES 为什么快？

- 倒排索引
- 分片并行
- 内存 + 磁盘混合

---

### 4. 倒排索引是什么？

- 词 → 文档列表
- 非逐行扫描
- 适合全文搜索

---

### 5. ES 分片与副本的作用？

- 分片（Shard）：提高并发
- 副本（Replica）：高可用 + 读性能

---

### 6. ES 写入流程？

1. 写入内存 Buffer
2. Refresh（生成 segment）
3. Flush（落盘）

---

### 7. ES 如何保证数据不丢？

- 副本机制
- translog
- 合理 refresh 策略

---

### 8. ES 查询优化手段？

- 避免深度分页
- 使用 filter 替代 query
- 合理分片数量
- 关闭无用字段索引

---


