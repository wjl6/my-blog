---
title: "003-Agent.md"
date: 2026-01-06 20:52:18
tags: []
---

## 1. dify、扣子、hiagent他们有什么共同点和不同点？
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


## 2. 如何将本地API接口封装位MCP服务？

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















