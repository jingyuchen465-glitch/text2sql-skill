# text2sql-skill

Text-to-SQL Agent Skill 仓库，用于基于 Bear MCP 生成安全的只读 SQL、计算业务指标并查询结果。

## 目录结构

| 路径 | 说明 |
| --- | --- |
| `{产品代号}-text-to-sql/` | 面向 **智能货柜业务** 的 Text-to-SQL Skill（本仓库的主要交付物）。`{产品代号}` 为占位符，请按实际产品名统一替换。 |
| `yanque-text-to-sql/` | 原始「燕雀」教育业务的 Text-to-SQL 模板（自定义前的基础版本，保留备查）。 |
| `skills说明文档.md` | 团队沉淀的开发规范（分层架构、REST、JPA、安全、测试、AI 集成等）。 |

## 智能货柜业务 Skill 概览

- **适用场景**：订单/支付/退款/GMV、商品品类、货柜设备、库存补货、点位网点、会员用户、运营告警等智能货柜业务数据问题。
- **业务域（domain）**：`order` / `product` / `cabinet` / `inventory` / `site` / `customer` / `operation`。
- **数据来源**：通过 Bear MCP Resource（`bear://{产品代号}/text-to-sql/...`）读取业务定义、指标口径、表目录和 DDL。
- **安全边界**：只允许单条只读 `SELECT`；禁止写操作与 DDL；执行前必须核对表目录与 DDL，敏感字段（手机号、token 等）默认不查询。

## 使用前必读

后端 Bear MCP 的智能货柜资源（业务域定义、table-catalog、DDL）需要在服务端创建后，Skill 中的资源 URI 才能生效。`{产品代号}` 需替换为真实产品代号，资源 URI 与权限上下文中的 `{产品代号}Id` 需与后端保持一致。