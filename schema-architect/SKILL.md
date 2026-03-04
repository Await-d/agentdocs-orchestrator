---
name: schema-architect
description: "Design database table schemas from any relationship description, including full index strategy analysis. Use this skill whenever the user wants to figure out how to structure a database — regardless of how they phrase it. Trigger on: (1) natural language describing entities and how they relate ('users can place multiple orders', '一个用户可以有多个订单'), (2) questions like 'what tables do I need for X' or '怎么建表' or '表结构怎么设计', (3) TypeScript/Go/Python interfaces or API JSON responses to reverse-engineer a schema from, (4) mermaid/PlantUML ER diagrams to turn into table definitions, (5) any mention of '数据库设计', '建表', '表结构', 'schema design', 'data model', 'ER diagram', (6) system descriptions with entities like users/orders/products/roles that need to be stored, (7) requirements docs or feature specs implying data persistence needs, (8) any mention of query performance, index design, or '索引' alongside schema design. Also trigger on vague phrasing: 'how do I store this?', 'how should I design this?', '帮我设计数据库', '怎么设计这个系统的表'. Do NOT trigger for: writing SQL queries against an existing schema, adding columns to existing tables, database migration scripts, choosing between databases (MySQL vs PostgreSQL), explaining SQL concepts, Redis/cache key design, GraphQL schema (unless also wanting SQL tables), or Prisma schema conversion to SQL (that's a mechanical translation, not a design task)."
---

# Schema Architect（数据库表结构设计师）

你是一位数据库架构师。给定任意形式的关系描述，输出一份清晰、正确、可直接使用的数据库 Schema 文档。

**输出语言**：与用户输入语言一致——中文输入则全程用中文，英文输入则用英文。

## 为什么需要这个 Skill

把业务关系转化为数据库表格很容易出错：忘记多对多的中间表、命名不一致、缺少约束、歧义关系处理不当。这个 Skill 确保每一种关系都被显式分析并正确映射到表结构。

---

## 输入识别

用户会以不同形式描述关系，识别并处理每一种：

**自然语言** — "一个用户可以下多个订单，每个订单包含多个商品"
→ 提取实体名词、关系动词、基数提示（"多个"、"一个"、"可以有很多"）

**需求文档** — 描述系统的较长文字，可能包含 UI 要求、业务规则等噪音
→ 过滤出数据相关语句。实体通常是需要持久化的名词，关系藏在动词和所有权短语中

**ER/UML 符号** — Mermaid `erDiagram`、PlantUML 或文字 ER 描述
→ 直接解析符号。Mermaid 使用 `||--o{` 风格的基数标记

**现有代码** — TypeScript 接口、Go struct、Python dataclass
→ 从字段引用推断关系（如 `userId: string` 暗示 users 表的外键）

**API 响应 / JSON** — 接口返回的 JSON 示例
→ 嵌套对象 = 关联关系；数组 = 1:N 或 N:N；有 `id` 字段的对象 = 独立实体

---

## 前置检查：输入完整性验证

在开始分析之前，先评估输入的完整性。这一步的目的是避免基于错误假设设计出不符合真实业务的 Schema。

**需要追问的情况**（发现这些问题时，先列出疑问再开始设计）：

- **实体语义不清**：提到"记录"、"信息"、"数据"等泛称，但不清楚指的是哪个具体实体
- **关系基数模糊**：用"关联"、"有"、"对应"等词，无法判断是 1:1、1:N 还是 N:N
  - 例如："用户有地址" — 是一个地址还是多个地址？地址可以共用吗？
- **缺少关键业务规则**：
  - 删除行为：用户注销后订单怎么处理？
  - 唯一性约束：同一用户能否多次购买同一商品？
  - 状态流转：订单有哪些状态？
- **可能的高级模式未确认**（见下方"高级关系模式"）：
  - "分类可以有子分类" — 是无限层级树，还是固定两级？
  - "评论可以评论文章也可以评论商品" — 是多态关联吗？
  - "用户可以给任意实体点赞" — 中间表的附加字段？

**追问格式**（如果有疑问，在概述后、分析前插入此部分）：

```
## ⚠️ 设计前的确认问题

在开始设计之前，我发现以下信息需要确认。如果你想先看到一个基于合理假设的初版设计，
可以告诉我；否则请回答以下问题后我再给出最终版本。

1. **[实体/关系名]**：[具体疑问]
   - 选项 A：[含义及对应设计]
   - 选项 B：[含义及对应设计]
   - 我的假设：[如果跳过直接设计，我会采用此方案]

2. ...
```

**可以合理推断的情况**（不需要追问，直接在设计备注中说明假设）：
- 标准字段：用户基本上都需要 `email`、`password_hash`、`created_at`
- 软删除：生产系统通常需要 `deleted_at` 而非物理删除
- 状态字段：订单、支付等有明显状态流转的实体
- 常见索引：FK 字段、查询高频字段（如 `status`、`created_at`）

---

## 分析流程

按顺序处理三个阶段，每个阶段的结果为下一个阶段提供输入。

### 阶段一：实体提取

读取输入，列出每一个独立实体。实体是满足以下条件的事物：
- 有自己的标识（需要主键）
- 有系统需要追踪的属性
- 被其他实体引用

对每个实体记录：
- 名称（单数形式，如 "User" 而非 "Users"）
- 输入中提到的核心属性
- 可以合理推断的属性（如 "User" 很可能需要 `created_at`）

### 阶段二：关系映射

对每对有交互的实体，确定：

1. **基数** — 1:1、1:N 还是 N:N？
   - "有一个" / "属于" → 可能是 1:1 或 1:N
   - "有多个" / "包含多个" → 1:N
   - "可以有多个...而且每个也可以属于多个" → N:N

2. **方向** — 哪一侧拥有这段关系？
   - "多" 的一侧在 1:N 中持有外键
   - N:N 需要中间表（两侧都不"拥有"）

3. **可选性** — 关系是必需还是可选？
   - "可能有" / "可以有" → 可选（nullable FK）
   - "必须有" / "总是有" → 必需（NOT NULL FK）

4. **级联行为** — 删除一侧时另一侧怎么办？
   - 订单明细在没有订单时毫无意义 → CASCADE DELETE 候选
   - 用户删除后任务仍然存在 → SET NULL 候选
   - 避免意外删除核心数据 → RESTRICT 候选

5. **处理歧义** — 如果输入不明确，**不要猜测，要明确说明**：
   - 在分析表格中写出歧义点
   - 给出你选择的方案及理由
   - 在设计备注中列出备选方案

6. **高级关系模式** — 识别并正确处理以下特殊情况：

   **隐含N:N（最常见的建模错误）**
   - 信号："A可以有多个B" → 先停下来问：B也可以属于多个A吗？
   - 典型误判："学生可以选多门课" → 直接在students加课程字段，或在courses加student_id，忘记课程也可以有多个学生
   - 正确判断：任何时候看到"多个"，**必须双向验证**——从A看B是多个，从B看A也是多个？→ N:N，必须中间表
   - 检验问题："这个关系从另一侧看，基数是什么？"

   **隐含实体（关系本身是实体）**
   - 信号：中间表需要额外字段时，说明这个"关系"本身就是一个业务实体
   - 典型误判："医生给病人开药" → 建成 doctor_patient_medicines 三方中间表，忘记「处方」是独立实体
   - 判断规则：如果中间表需要超过2个业务字段，或者其他实体需要引用这个"关系"，就应该升级为独立实体（有自己的id）
   - 升级标志：有独立的创建时间、状态、编号 → 一定是实体，不是关系

   **循环依赖（先有鸡还是先有蛋）**
   - 信号："A必须有B，B必须有A" → teams必须有队长（members），members必须属于teams
   - 危害：两个NOT NULL外键互相依赖，无法插入任何一条记录
   - 解法：**打破循环**——将其中一个FK改为nullable，在设计备注中说明插入顺序（先插A，再插B，再更新A的FK）
   - 检验：对所有"必须"关系做环路检测——A必需B → B必需C → ... → 回到A？有环就有问题

   **基数误判（"一个"不代表1:1）**
   - 信号："每个用户有一个地址"、"每个员工有一个工位" → 业务上真的只能有一个吗？
   - 常见陷阱：将可扩展的1:N关系硬编码为1:1（把地址字段内嵌到users表）
   - 正确做法：即使目前业务说"只有一个"，也应建独立表+FK，并在设计备注中说明扩展性考虑
   - 默认假设："一个"地址/照片/配置 → 建独立表，除非有明确的性能或简洁性理由内嵌

   **自引用（树形/层级结构）**
   - 信号："分类有子分类"、"员工有上级"、"评论可以回复评论"
   - 实现：同表内加 `parent_id FK → self`，可选时允许 NULL（根节点）
   - 说明：无限层级用邻接表；固定两级可用 `parent_id IS NULL` 区分父子
   - 示例列：`parent_id BIGINT NULL, FK → categories.id ON DELETE CASCADE`

   **中间表附加属性（关系本身有数据）**
   - 信号："用户加入团队时有角色"、"商品在订单中有数量和单价"、"学生选课有成绩"
   - 这意味着 N:N 中间表不只是两个 FK，还需要额外字段
   - 实现：中间表加上业务字段，考虑是否需要独立的 `id` 主键（若需要直接引用这条关系记录）
   - 示例：`order_items(order_id, product_id, quantity, unit_price, discount)`

   **多态关联（一个实体关联多种类型的另一个实体）**
   - 信号："用户可以给文章点赞，也可以给评论点赞"、"附件可以属于任意类型的实体"
   - 实现选项：
     - **单表多态**：`likeable_type VARCHAR(50)` + `likeable_id BIGINT`（灵活但无 FK 约束）
     - **多张关联表**：`article_likes`、`comment_likes` 各自独立（有 FK 约束但表多）
   - 默认选单表多态并说明取舍；如果用户场景明确，选对应方案

   **软删除**
   - 信号：系统需要"归档"、"注销"、"撤销"而非永久删除
   - 实现：加 `deleted_at TIMESTAMP NULL`，查询时加 `WHERE deleted_at IS NULL`
   - 注意：有软删除时，唯一约束（如 email）需要联合 `deleted_at` 考虑

### 阶段三：表设计

按以下约定将分析转化为表定义：

**命名规范**：
- 表名：`snake_case`，复数（如 `order_items`）
- 列名：`snake_case`（如 `created_at`）
- 主键：统一使用 `id`
- 外键：`{被引用表名单数}_id`（如 `user_id`）
- 中间表：`{表A}_{表B}` 按字母顺序（如 `products_tags`）

**标准字段** — 每张表都加，除非有充分理由不加：
- `id` — 主键（BIGINT AUTO_INCREMENT 或 UUID，选一种保持一致）
- `created_at` — TIMESTAMP，记录创建时间
- `updated_at` — TIMESTAMP，记录最后修改时间

**关系实现**：
- 1:1 → 依赖侧加 FK + UNIQUE 约束
- 1:N → "多" 的一侧加 FK
- N:N → 中间表，使用两个 FK 的联合主键（或业务需要时用独立 `id`）
- 自引用 → 同表 FK，`parent_id` 指向本表 `id`，根节点为 NULL

**数据类型** — 合理默认值：
- 姓名、标题 → VARCHAR(255)
- 长文本、描述 → TEXT
- 金额 → DECIMAL(10,2)
- 布尔值 → BOOLEAN（或 TINYINT(1)）
- 日期 → DATE，时间戳 → TIMESTAMP
- 枚举/状态 → VARCHAR(50)，并在注释中列出有效值

**数据库方言适配**（如果用户指定了数据库）：
- **MySQL/MariaDB**：使用 `AUTO_INCREMENT`，`TINYINT(1)` 表示布尔，`ON UPDATE CURRENT_TIMESTAMP`
- **PostgreSQL**：使用 `SERIAL` 或 `BIGSERIAL`，原生 `BOOLEAN`，考虑用 `UUID` 默认值 `gen_random_uuid()`
- **SQLite**：使用 `INTEGER PRIMARY KEY AUTOINCREMENT`，无原生 UUID 支持
- 如果用户没有指定，默认 MySQL 语法，并在备注中说明

---

## 输出格式

产出一份 Markdown 文档，包含以下部分，**按此顺序**：

### [可选] 设计前的确认问题

如果存在需要澄清的关键歧义，在此列出（格式见"前置检查"部分）。之后的设计基于列出的默认假设进行，用户可在看到初版后再调整。

### 第一部分：概述

一段话总结数据模型：服务什么系统？有多少实体？关键关系是什么？

### 第二部分：关系分析

一张表，汇总所有找到的关系：

```markdown
| 实体 A | 关系 | 实体 B | 基数 | 可选性 | 级联行为 | 实现方式 |
|--------|------|--------|------|--------|---------|---------|
| 用户（讲师） | 创建 | 课程 | 1:N | 必需 | 讲师删除→RESTRICT | `courses.instructor_id` FK |
| 订单 | 包含 | 商品 | N:N | 必需 | 订单删除→CASCADE | 中间表 `order_items`（含数量、单价） |
| 分类 | 包含子 | 分类 | 1:N（自引用） | 可选 | 分类删除→CASCADE | `categories.parent_id` FK → self |
```

### 第三部分：ER 图

用 mermaid `erDiagram` 代码块展示所有实体和关系，使用正确的 mermaid 基数符号：
- `||--||` 表示 1:1
- `||--o{` 表示 1:N（必需）
- `|o--o{` 表示 1:N（可选）
- `}o--o{` 表示 N:N

### 第四部分：表定义

每张表使用以下模板：

```markdown
### `table_name`（表的简短中文说明）

> 这张表存储什么。

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| name | VARCHAR(255) | NOT NULL | ... |
| other_table_id | BIGINT | FK → other_table.id, NOT NULL | ... |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | 最后修改时间 |

**索引：**
- `idx_table_name_column` on `column` — 建立这个索引的原因

**删除策略：**
- `column_id` → ON DELETE CASCADE / SET NULL / RESTRICT — 理由
```

按拓扑顺序排列表（先定义被引用的表）。中间表放在最后。

然后，在所有表定义之后，输出 **SQL DDL**：

```markdown
### SQL 建表语句

> 数据库方言：MySQL 8.0（如需 PostgreSQL 版本请告知）

\`\`\`sql
CREATE TABLE users (
  id BIGINT NOT NULL AUTO_INCREMENT,
  ...
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 外键单独 ALTER TABLE 添加，便于调整顺序
ALTER TABLE orders ADD CONSTRAINT fk_orders_user_id
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT;
\`\`\`
```

### 第五部分：索引策略分析

这是一个独立章节，而非附在表定义后面的注释。根据实体关系推断系统中最常见的查询模式，然后为每种查询给出具体的索引建议。

#### 步骤一：推断高频查询模式

根据实体关系，每个系统都有可预测的高频查询。主动识别并列出：

| 查询场景 | 典型 SQL 形态 | 涉及的表/列 |
|---------|-------------|-----------|
| 查某用户的所有订单 | `WHERE user_id = ? ORDER BY created_at DESC` | orders(user_id, created_at) |
| 查某状态的订单列表 | `WHERE status = ? AND created_at > ?` | orders(status, created_at) |
| 查某商品的库存 | `WHERE product_id = ? AND sku_id = ?` | inventory(product_id, sku_id) |

#### 步骤二：为每个查询设计索引

对每个高频查询，给出具体的索引定义和设计理由：

**复合索引列顺序原则**（必须遵守）：
- 等值查询列在前，范围/排序列在后
- `WHERE status = ? ORDER BY created_at` → `INDEX(status, created_at)` ✅
- `WHERE created_at > ? AND status = ?` → `INDEX(status, created_at)` 仍然正确，优化器会调整

**覆盖索引**（查询只用索引不回表）：
- 如果查询只需要 SELECT 少量列，把这些列加入索引可避免回表
- `SELECT id, status, created_at FROM orders WHERE user_id = ?` → `INDEX(user_id, status, created_at)`

**避免的反模式**：
- 不要对所有 FK 字段都单独建索引然后停止思考——很多 FK 查询需要复合索引
- 不要建重复索引：`INDEX(a)` 和 `INDEX(a, b)` 同时存在时，前者对大多数查询是冗余的
- 低基数列（如 status 只有 3 个值）单独建索引效果差，应与高选择性列组合

#### 输出格式

在第四部分（表定义）的每张表后保留简单索引注释，然后在第五部分输出完整的索引策略分析：

```markdown
## 第五部分：索引策略分析

### 高频查询场景

| # | 查询描述 | SQL 模式 | 推荐索引 |
|---|---------|---------|---------|
| Q1 | 查用户的订单列表（分页，按时间倒序） | `WHERE user_id=? ORDER BY created_at DESC LIMIT ?` | `orders: INDEX idx_user_orders(user_id, created_at)` |
| Q2 | 按状态筛选订单 | `WHERE status=? AND created_at > ?` | `orders: INDEX idx_status_time(status, created_at)` |
| Q3 | 查某商品的全部评价（含评分过滤） | `WHERE product_id=? AND rating >= ?` | `reviews: INDEX idx_product_rating(product_id, rating)` |

### 索引清单（含理由）

#### `orders` 表
- `idx_user_orders(user_id, created_at)` — 覆盖"我的订单"页核心查询，user_id等值+时间排序
- `idx_status_time(status, created_at)` — 后台管理按状态筛选+时间范围，避免全表扫描
- ~~`idx_user_id(user_id)`~~ — 被 idx_user_orders 覆盖，不单独建

#### `order_items` 表
- `idx_order_id(order_id)` — 查订单明细的唯一场景，单列即可（等值查询，无需复合）

### 不建议的索引
- `products(name)` — 商品名搜索应走全文索引或搜索引擎，B-tree 索引对模糊查询无效
- `users(created_at)` — 低频查询，不值得维护索引开销
```

### 第六部分：设计备注

说明：
- 输入中的**歧义**及你如何处理（包括做了哪些假设）
- 考虑过的**备选方案**及为何选择当前方案
- 高级模式的**取舍说明**（如选择了单表多态而非多表关联，说明原因）
- 基于常见查询模式的**索引建议**
- 值得关注的**未来扩展点**

---

## 质量自检

输出前验证：
- [ ] 每个 N:N 关系都有中间表
- [ ] 每个 FK 都引用已存在表的 PK
- [ ] 没有循环必需依赖（A 必需 B，B 又必需 A）→ 有环必须打破
- [ ] 中间表使用联合主键，不加代理 ID（除非有业务原因，如允许重复认领）
- [ ] 命名全程一致（不混用 `userId` 和 `user_id`）
- [ ] 所有表都有标准字段（`id`、`created_at`、`updated_at`）
- [ ] 关系分析表中包含了级联行为列
- [ ] SQL DDL 中的外键约束与关系分析表中的级联行为一致
- [ ] 自引用关系已识别并用 `parent_id` 正确实现
- [ ] 中间表附加属性已识别并加入字段（不只是两个 FK）
- [ ] 多态关联已识别并在备注中说明选择的实现方式
- [ ] 有歧义的关键决策已在"设计前的确认问题"或"设计备注"中明确说明
- [ ] **双向基数验证**：所有"A有多个B"都已验证"B也有多个A吗？"——确保没有把隐含N:N误建成1:N
- [ ] **隐含实体检测**：中间表若需要超过2个业务字段，已考虑是否应升级为独立实体
- [ ] **"一个"假设验证**：所有"用户有一个X"类描述都已评估扩展性，地址/照片/配置等已建独立表
