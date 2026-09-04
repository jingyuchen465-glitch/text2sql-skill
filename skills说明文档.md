## 1. 核心开发（必装）

### `layered-architecture` — 分层架构规范

**作用：** 强制代码按层组织，防止"大泥球"代码

plain

```plain
Controller 层  →  只处理 HTTP 请求/响应，不碰业务逻辑
     ↓
Service 层     →  核心业务逻辑，不直接操作数据库
     ↓
Repository 层  →  数据访问，只负责 CRUD
```

**AI 会帮你：**

- 拒绝在 Controller 里写 SQL
- 拒绝在 Service 里返回 HTTP 状态码
- 每层职责清晰，代码易维护

------

### `rest-api-conventions` — REST API 规范

**作用：** 统一 API 设计标准，前后端协作更顺畅

**包含规范：**

表格



| 方面      | 规范                                                         |
| :-------- | :----------------------------------------------------------- |
| URL 设计  | `/api/v1/users`（复数名词）                                  |
| HTTP 方法 | GET 查、POST 增、PUT 全改、PATCH 局部改、DELETE 删           |
| 响应格式  | `{ "data": ..., "message": "...", "timestamp": ... }`        |
| 错误码    | 400 参数错误、401 未认证、403 无权限、404 不存在、500 服务器错误 |
| 分页      | `?page=0&size=20&sort=name,asc`                              |

**AI 会帮你：**

- 生成符合 REST 规范的端点
- 统一返回 JSON 结构
- 自动处理异常和错误响应

------

### `spring-data-jpa` — JPA 数据访问规范

**作用：** 规范数据库操作，避免性能陷阱

**包含内容：**

- **实体规范** — 主键策略、字段映射、关联关系
- **N+1 问题预防** — 自动使用 `JOIN FETCH` 或 `@EntityGraph`
- **投影查询** — 只查需要的字段，减少数据传输
- **键集分页** — 大数据量分页不跳页、不慢

**AI 会帮你：**

- 正确写 `@OneToMany`、`@ManyToOne` 关联
- 自动生成高效的查询方法
- 避免常见的 JPA 性能坑

------

## 2. 安全（必装）

### `spring-security-jwt` — JWT 认证授权

**作用：** 实现安全的登录认证和权限控制

**包含内容：**

- **JWT Token** — 生成、验证、刷新
- **过滤器链** — 每个请求自动验 Token
- **RBAC** — 基于角色的权限控制（ADMIN、USER 等）
- **Token 轮换** — 自动刷新，提升安全性

**AI 会帮你：**

- 搭建完整的登录/注册接口
- 配置 Spring Security 过滤器
- 给接口加 `@PreAuthorize("hasRole('ADMIN')")` 权限注解

------

## 3. 测试

### `java-springboot-testing` — Spring Boot 测试

**作用：** 写高质量的自动化测试

**包含内容：**

表格



| 技术               | 用途                                                        |
| :----------------- | :---------------------------------------------------------- |
| **测试切片**       | `@WebMvcTest`、`@DataJpaTest` — 只加载需要的 Bean，测试更快 |
| **MockMvcTester**  | 模拟 HTTP 请求，测试 Controller 不启动服务器                |
| **Testcontainers** | 用 Docker 容器跑真实数据库测试，不是内存数据库              |
| **AssertJ**        | 流式断言，测试代码更易读                                    |

**AI 会帮你：**

- 给每个 Controller/Service 写单元测试
- 用 Testcontainers 做集成测试
- 生成覆盖率高的测试用例

------

## 4. AI 集成（进阶）

### `spring-ai-integration` — Spring AI 集成

**作用：** 在 Spring Boot 里接入大模型能力

**包含内容：**

- **ChatClient** — 调用 GPT/Claude 等大模型
- **RAG 管道** — Retrieval-Augmented Generation，让 AI 基于你的文档回答
- **向量数据库** — 存储文档向量，实现语义搜索

**AI 会帮你：**

- 搭建智能问答接口
- 实现"基于公司知识库的客服机器人"
- 代码自动生成、智能审查等功能