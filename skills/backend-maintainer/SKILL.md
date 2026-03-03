---
name: backend-maintainer
description: 统一约束 Java 后端代码规范。只要用户请求涉及任何代码生成、代码修改、重构、补丁、数据库脚本变更、接口变更、DTO/Entity/Mapper/Service/Controller 相关开发任务，必须立即启用本 skill 并按规范执行。
version: 1.0.0
owner: jbkc
---

# Guard

## 目标
- 将 JBKC 项目规范作为单一执行基线。
- 在实现需求时优先保证分层边界、接口契约、数据访问方式和可维护性。

## 使用时机
- 用户要求“按规范开发/修复/重构/生成代码”。
- 涉及 Controller、DTO、Entity、MapStruct、Flyway、Redis、MinIO、分页、参数校验。
- 涉及 PostgreSQL 迁移脚本编写与命名。

## 强制触发
- 只要任务包含“生成代码”或“修改代码”（包括新增、删除、重构、修复、迁移），无条件触发本 skill。
- 仅在纯聊天、纯概念解释且不落地改代码时可不触发。

## 强制规范

### 1. DTO 与接口边界
- 必须为所有 DTO 类、字段、方法编写规范 JavaDoc。
- 禁止在 DTO 类中使用 Swagger 注解（`@ApiModel`、`@ApiModelProperty` 等）。
- Controller 返回值必须统一为 `com.lw.common.pojos.ApiResponse`，禁止直接返回其他类型。
- Controller 输入参数若字段数大于等于 2，必须采用 DTO 入参。
- DTO 必须放在 `com.*.*.model` 包下。
- 禁止 Controller 直接接受 `Map`、`String` 等松散结构作为复杂输入。
- 禁止将数据库实体类直接返回给前端，必须返回 DTO。

### 2. 包结构与职责
- 常量统一放在 `com.*.*.constanas`。
- 枚举统一放在 `com.*.*.enums`。
- `com.*.*.entity` 只能存放 MyBatis-Plus 自动生成的数据库实体类，禁止手写业务代码。

### 3. MyBatis-Plus 数据访问
- 单表场景（CRUD/分页/条件查询/条件更新/逻辑删除）禁止写 XML、禁止手写 SQL。
- 统一使用 MyBatis-Plus：`LambdaQueryWrapper`、`LambdaUpdateWrapper`、`lambdaQuery()`、`lambdaUpdate()`、MyBatis-Plus 分页能力。
- 只有确因性能或功能无法满足时，才允许 SQL/XML；需要在输出中说明理由。

### 4. 对象转换
- 必须使用 MapStruct。
- 转换接口统一放在 `com.*.*.mapstruct`。
- 禁止通过 Spring 注入转换器（不得使用 `componentModel = "spring"`、`@Autowired`、`@Resource`）。
- 必须通过 `Mappers.getMapper(XxxConverter.class)` 获取静态单例 `INSTANCE`。
- 严禁使用 `BeanUtils`（包括 `copyProperties`）进行属性拷贝。
- 可参考 `com.*.*.mapstruct.OperationLogConverter`。

### 5. 分页规范
- 有分页请求时，请求 DTO 直接继承 `GeneralPageReq`。

### 6. Flyway（PostgreSQL）
- 迁移脚本目录必须为：`resources/flyway/pg/`。
- 扫描路径固定：`classpath:flyway/pg`。
- 命名必须为：`V-版本号-描述.sql`。
- 示例：`V日期时间-init_schema.sql`、`V日期时间_index.sql`。
- 命名校验开启：`validateMigrationNaming(true)`。
- 禁止乱序执行：`outOfOrder(false)`。
- 首次接入允许基线：`baselineOnMigrate(true)`。
- 禁止清库：`cleanDisabled(true)`。
- 版本记录表：`schema_version`。
- `CREATE TABLE` 必须带 `IF NOT EXISTS`。
- 变更脚本使用 `ALTER TABLE`，并保证幂等。

### 7. 项目通用字段
- 通用字段：`createdBy`、`updatedBy`、`isDeleted`。
- 逻辑删除语义：DB `0=未删, 1=已删`；Java `false=未删, true=已删`。
- 租户字段：实体 `tenantId` 对应 DB `tenant_id`，映射与 SQL 必须一致。

### 8. MinIO 规范
- 必须统一通过 `com.lw.mini_io.service.MinioService`。
- 对象名统一使用 `com.lw.mini_io.utils.MinioObjectNameGenerator.generateObjectName(businessPrefix, fileExtension)`。
- 禁止绕过以上规范直接使用 MinIO SDK 或其他封装。

### 9. 代码生成与基础设施
- 涉及表结构/实体/Mapper/Service/XML 缺失时，对话中不直接手写生成结果。
- 必须使用生成器：`lw-jbkc/src/test/java/generator/JbkcCodeGenerator.main()`。
- Redis 禁止二次封装 `RedisUtil/RedisService` 等。
- 必须直接使用 Redisson 原生 API（`RedissonClient + getBucket/getMap/getLock...`）。
- 测试统一使用 Spring Boot 单元测试体系。

### 10. 接口参数校验
- 事务场景或 POST 请求输入 DTO 时，必须执行参数校验。
- 标准注解：`jakarta.validation.constraints.*`。
- 自定义注解：`com.lw.validator.annotations.*`。
- 可用注解：`@EnumListSubset`、`@ComplexCustomValidLogic`、`@MultiFieldLogic`、`@NotBlankElements`。
- 请求参数应使用 `@Valid`。
- 非代理模式调用函数时使用 `com.lw.validator.ValidationSupport`。
- 响应仍统一使用 `ApiResponse<T>`。
- 全局异常处理、校验失败、序列化时间、Long 类型等在 `com.lw.spring.configs` 已处理，不重复实现。

## 执行流程
1. 先识别任务涉及的层（Controller/Service/Mapper/DTO/DB脚本/对象存储/缓存）。
2. 逐条匹配“强制规范”，先列风险点再实施修改。
3. 输出时必须给出“本次命中的规范项”和“未命中的规范项（若不适用需说明原因）”。
4. 若发现请求与规范冲突，优先遵循本 skill，并给出替代实现。

## 输出模板
每次响应按以下结构组织：
1. 目标理解（1-3 句）
2. 实施方案（按文件或模块）
3. 规范命中清单（逐条勾选）
4. 风险与兼容性
5. 验收步骤（含测试建议）

## 完成标准
- 无 DTO Swagger 注解。
- Controller 全量使用 `ApiResponse`。
- 无 Entity 对外直出。
- 单表操作全部为 MyBatis-Plus Lambda 风格。
- 无 `BeanUtils.copyProperties`。
- 分页 DTO 继承 `GeneralPageReq`。
- Flyway 脚本位置、命名、执行策略、幂等性全部满足。
- Redis/MinIO 使用方式符合规范。
- 校验注解与 `@Valid` 使用正确。
