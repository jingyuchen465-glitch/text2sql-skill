***

name: {产品代号}-text-to-sql
description: 当用户询问智能货柜业务数据，需要生成 SQL、计算指标、查询结果或解释业务口径时使用。
--------------------------------------------------------

# 智能货柜 Text-to-SQL

## 适用场景

当用户询问智能货柜业务数据时使用本 Skill，例如：

- 订单、支付、退款、销售额、GMV、商品、购买、客单价、转化率

- 商品、品类、规格、条码、价格、毛利、销量、上下架

- 货柜、智能柜、设备、格子、货道、门锁、型号、点位、在线、离线、故障

- 库存、补货、盘点、缺货、断货、过期、损耗、入库、出库、上架

- 点位、网点、商圈、覆盖、地址、运营商、点位活跃

- 顾客、会员、用户、下单、复购、客群、画像、标签、生命周期

- 告警、异常、预警、掉线、工单、活动、营销、优惠、运营

如果问题只是通用 SQL 教学，或者和智能货柜业务数据无关，不使用本 Skill。

## 核心原则

通过 Bear MCP Resource 获取业务定义、指标口径、表目录和 DDL；基于这些上下文生成一条安全 SQL；需要执行时，只通过 Bear MCP 动态工具 `execute_text_to_sql_query` 执行。

## 权限上下文

`execute_text_to_sql_query` 不能从用户对话里接收角色编码、Bear 用户 ID 或智能货柜后端用户 ID。

该工具使用 Groovy binding 中可信的 `{产品代号}Id` 作为智能货柜后端用户 ID。后端收到后，会通过角色查询映射查询该用户角色，并在 `InternalTextToSqlController#executeSql` 中执行 Text-to-SQL 表权限、字段权限、SQL AST、EXPLAIN 和最大行数校验。

如果 `{产品代号}Id` 不存在或格式非法，工具必须拒绝执行。不要通过手动传角色、扩大权限或绕过权限校验来重试。

## 运行时 Prompt

交互式客户端可先渲染 Bear MCP Prompt：

```text
{产品代号}_text_to_sql
```

Prompt 用于提供当前问题、是否执行、最大返回行数和回答格式要求。本 Skill 用于规定完整方法和安全边界。

## Bear MCP 流程

1. 识别用户问题所属业务域：

   - `order`

   - `product`

   - `cabinet`

   - `inventory`

   - `site`

   - `customer`

   - `operation`

2. 读取对应业务 Resource：

   - `bear://{产品代号}/text-to-sql/business/{domain}`

3. 如果用户询问指标、KPI、趋势、数量、求和、平均、占比、排名或分布，先查询指标口径：

   - 优先使用 `search_text_to_sql_metrics`

   - 使用 text-to-sql 知识库中的指标文档作为口径依据

   - 新增指标时使用 `create_text_to_sql_metric`

4. 选表前先读取表目录：

   - `bear://{产品代号}/text-to-sql/table-catalog`

5. SQL 使用任何表或字段之前，必须读取对应 DDL Resource：

   - `bear://{产品代号}/text-to-sql/ddl/{table_name}`

6. 只有在 Bear MCP 上下文确认以下信息后才生成 SQL：

   - 表存在

   - 所有字段存在

   - 关联字段存在

   - 状态值或枚举值明确

   - 指标公式明确

   - 时间字段明确，或者已经向用户澄清

7. 需要执行 SQL 时，调用 `execute_text_to_sql_query`，参数为：

   - `sql`：一条只读 SELECT SQL

   - `table_ddl_context`：覆盖 SQL 中全部表和字段的 DDL 上下文

   - `business_domain`：已识别出的主业务域

   - `max_rows`：最大返回行数，最高 500

8. 如果执行结果返回权限不足，只说明被拒绝的表或字段，不要用更高角色重试。

## 业务域路由

| 业务域         | 用户提到这些内容时使用                    |
| ----------- | ------------------------------ |
| `order`     | 订单、支付、退款、销售额、GMV、客单价、转化率、复购订单  |
| `product`   | 商品、品类、规格、条码、价格、毛利、销量、上下架       |
| `cabinet`   | 货柜、智能柜、设备、格子、货道、门锁、型号、在线、离线、故障 |
| `inventory` | 库存、补货、盘点、缺货、断货、过期、损耗、入库、出库、上架  |
| `site`      | 点位、网点、商圈、覆盖、地址、运营商、点位活跃        |
| `customer`  | 顾客、会员、用户、下单、客群、画像、标签、生命周期      |
| `operation` | 告警、异常、预警、掉线、工单、活动、营销、优惠        |

如果一个问题涉及多个业务域，需要读取所有相关业务域的 Resource。

## 任务类型

| 类型                     | 含义                     | 必需上下文                   |
| ---------------------- | ---------------------- | ----------------------- |
| `detail_query`         | 明细、列表、记录、详情查询          | 业务定义 + 表目录 + DDL        |
| `metric_query`         | 数量、求和、平均、占比、排名、趋势等指标查询 | 业务定义 + 指标口径 + 表目录 + DDL |
| `definition_query`     | 解释业务术语或指标含义            | 业务定义或指标文档               |
| `clarification_needed` | 关键口径、时间字段、过滤条件不明确      | 向用户追问                   |

## 需要追问的情况

以下情况不要猜，先向用户确认：

- 指标定义缺失

- 时间字段不明确

- 有效订单、活跃设备、热销、缺货、过期等业务口径不明确

- 一个指标存在多个可能公式

- 用户请求敏感明细，但没有明确业务必要性

- Resource 中找不到需要的表或字段

## SQL 安全要求

只能生成一条只读 `SELECT` SQL。

禁止生成：`INSERT`、`UPDATE`、`DELETE`、`MERGE`、`DROP`、`ALTER`、`TRUNCATE`、`CREATE`、存储过程调用、多语句 SQL、包含分号的 SQL。

不要使用 `SELECT *`。

不要使用 {产品代号} DDL Resource 中不存在的表或字段。

不要默认查询手机号、密码、凭证类 token 等敏感字段。

不要自行假设金额单位换算。

## 推荐工具

| 工具                           | 用途                                 |
| ---------------------------- | ---------------------------------- |
| `render_prompt`              | 渲染 `{产品代号}_text_to_sql` 运行时 Prompt |
| `read_mcp_resource`          | 读取业务定义、表目录和 DDL Resource           |
| `search_text_to_sql_metrics` | 从 text-to-sql 知识库检索指标口径            |
| `execute_text_to_sql_query`  | 使用可信 `{产品代号}Id` 权限上下文执行最终 SQL      |
| `create_text_to_sql_metric`  | 新增或上传指标 JSON 文档                    |
| `list_mcp_resources`         | 查找当前可用 Resource                    |

## 最小工作模式

1. 判断业务域和任务类型。
2. 读取业务定义和相关指标口径。
3. 读取表目录和 DDL Resource。
4. 生成一条安全 SELECT SQL。
5. 只有用户要求查询结果，或 Prompt 明确允许执行时，才调用 `execute_text_to_sql_query`。
6. 返回指标依据、SQL、使用的 Resource、权限校验结果和查询结果。

