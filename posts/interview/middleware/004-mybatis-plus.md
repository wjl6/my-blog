---
title: "004-mybatis-plus.md"
date: 2026-01-08 14:08:53
tags: []
---

## 1. mybatis-plus中SQL查询的过程？

MP 基于 MyBatis 原生内核做上层封装，查询过程本质是「**MP 动态构建 SQL → 交给 MyBatis 执行引擎处理 → 结果映射返回**」，全程兼容 MyBatis 逻辑，同时通过内置 CRUD、条件构造器等简化开发，无需手动编写基础 SQL。

```text
┌───────────────────────┐
│  调用MP查询方法        │
└───────────────┬───────┘
                ▼
┌───────────────────────┐
│  解析查询条件/实体映射  │
└───────────────┬───────┘
                ▼
┌───────────────────────┐
│  动态构建SQL语句       │
└───────────────┬───────┘
                ▼
┌───────────────────────┐
│  生成MyBatis可执行的   │
│  MappedStatement      │
└───────────────┬───────┘
                ▼
┌───────────────────────┐
│  MyBatis执行SQL        │
│  （预处理/执行）       │
└───────────────┬───────┘
                ▼
┌───────────────────────┐
│  结果集映射为Java对象  │
└───────────────┬───────┘
                ▼
┌───────────────────────┐
│  MP封装结果返回        │
└───────────────────────┘
```
### 步骤 1：调用 MP 查询方法
用户通过 BaseMapper 内置方法（如 selectById/selectList）或自定义方法发起查询，这是流程的起点：
- **内置方法**：MP 预定义的 CRUD 方法（无需写 SQL），如 selectById（单条）、selectList（列表）、selectPage（分页）；
- **自定义方法**：兼容 MyBatis 原生的注解 / XML SQL（如 @Select("SELECT * FROM user WHERE age > #{age}")）。

### 步骤 2：解析查询条件/实体映射

MP 接收到查询请求后，首先做「条件解析 + 实体与数据库的映射解析」：
- 条件解析：
  - 若用 QueryWrapper/LambdaQueryWrapper，解析其中的条件（如 eq("age",18) → age=?、like("user_name","张") → user_name LIKE ?）；
  - 处理逻辑删除：若实体加了 @TableLogic，自动拼接 deleted=0（未删除）到 WHERE 条件；
  - 处理主键策略：解析 @TableId 注解，确定主键字段（如 id）。
- 实体映射解析：
  - 解析 @TableName 确定数据库表名（如 User 类 → user 表）；
  - 解析 @TableField 完成「实体属性 ↔ 数据库字段」映射（如 userName → user_name）；
  - 确定要查询的字段：默认查实体所有映射字段，也可通过 wrapper.select("id","user_name") 指定字段。

### 步骤 3：动态构建 SQL 语句（MP 核心）
这是 MP 最核心的环节 ——自动拼接符合数据库语法的 SQL，无需手动编写：

示例：selectList(Wrapper) 构建 SQL 的过程
```sql
// 代码中的条件构造器
QueryWrapper<User> wrapper = new QueryWrapper<User>()
    .eq("age", 18)        // WHERE age = ?
    .like("user_name", "张") // AND user_name LIKE ?
    .orderByDesc("id");   // ORDER BY id DESC

// MP 最终构建的完整 SQL（MySQL 语法）
SELECT id, user_name, age, email 
FROM user 
WHERE deleted = 0 
  AND age = ? 
  AND user_name LIKE ? 
ORDER BY id DESC
```
- SELECT 子句：默认包含实体所有映射字段，可通过 select() 手动指定；
- FROM 子句：来自 @TableName 注解；
- WHERE 子句：拼接 Wrapper 条件 + 逻辑删除条件；
- ORDER BY/GROUP BY：来自 Wrapper 的 orderBy/groupBy 方法。


### 步骤 4：生成 MyBatis 可执行的 **MappedStatement**
  - MyBatis 执行 SQL 的核心是 MappedStatement（包含 SQL 语句、参数映射、结果映射等），MP 会：
  把构建好的 SQL、参数映射（如 age=? 对应 Integer 类型）、结果映射（字段→属性）封装成 MappedStatement；
  - 将 MappedStatement 注册到 MyBatis 的 Configuration 中（相当于 MyBatis 原生 XML 解析后的效果）；
  - 这一步是 MP 兼容 MyBatis 的关键 ——MP 动态生成 MappedStatement，替代了手动编写 XML 的工作。


### 步骤 5：MyBatis 执行 SQL（原生内核）
MP 底层完全复用 MyBatis 的执行引擎，这一步是 MyBatis 的核心逻辑：
- 创建 SqlSession：通过 SqlSessionFactory 创建 SqlSession（默认 DefaultSqlSession），绑定执行环境（数据源、事务）；
- 参数预处理：将 Wrapper 中的参数（如 18、"张"）绑定到 SQL 的 ? 占位符，防止 SQL 注入；
- 执行 SQL：
  - 通过 Executor（执行器，默认 SimpleExecutor）获取 JDBC Connection；
  - 用 PreparedStatement 执行 SQL，获取数据库返回的 ResultSet（结果集）；
  - 分页查询特殊处理：先执行 SELECT COUNT(*) 获取总条数，再执行 SELECT ... LIMIT ? OFFSET ? 获取分页数据。

### 步骤 6：结果集映射为 Java 对象
MyBatis 将数据库返回的 ResultSet 映射为实体对象：
- 根据步骤 2 解析的「属性 - 字段」映射关系，将数据库字段（如 user_name）赋值给实体属性（如 userName）；
- 处理类型转换（如数据库 INT → Java Integer、VARCHAR → String）；
- 单条记录 → 单个实体对象，多条记录 → List 集合。

### 步骤 7：MP 封装结果返回
MP 对 MyBatis 返回的结果做轻量封装后返回给用户：
- selectById：返回单个 User 对象（无数据则 null）；
- selectList：返回 List<User>；
- selectPage：返回 Page<User>（包含 records（当前页数据）、total（总条数）、size（每页条数））。























