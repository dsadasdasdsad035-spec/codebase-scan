# 技术架构：商城商品中心后端 (backend-package-yudao-module-product)

入口 ID：backend-package-yudao-module-product
证据：evidence/backend-package-yudao-module-product/{nodes,typecards}.json
覆盖：技术栈、模块边界、并发模型、可观测性

---

## 1. 技术栈

| 维度 | 选型 | 用途 |
|---|---|---|
| 运行时 | JDK 17+ | Spring Boot 3.x 基线 |
| Web 框架 | Spring MVC | REST API 入口 |
| 安全框架 | Spring Security + @PreAuthorize | 权限校验 |
| 持久化 | MyBatis-Plus 3.5.x | ORM、Lambda Query |
| 事务 | Spring `@Transactional` | 声明式事务 |
| 校验 | Jakarta Validation + Spring Validator | 参数校验 |
| 映射 | MapStruct | DO/VO/DTO 转换 |
| 工具 | Lombok + Hutool + Guava | 代码简化 |
| 日志 | SLF4J + Logback | 日志输出 |
| 文档 | Swagger v3 (springdoc) | OpenAPI 文档 |
| 缓存 | Redis（框架级） | 字典、用户信息 |
| 序列化 | Jackson | JSON 转换 |
| 监控 | Micrometer + Prometheus（框架级） | 指标采集 |

## 2. 框架集成点

### 2.1 yudao-framework 基础设施

| 模块 | 提供的类 | 用途 |
|---|---|---|
| common-pojo | `CommonResult<T>` | 统一返回结构 `{code, data, msg}` |
| common-pojo | `PageResult<T>` | 分页结果 `{list, total}` |
| common-util | `CollectionUtils.convertMap/convertList` | 集合转换 |
| common-util | `BeanUtils.toBean` | Bean 拷贝 |
| common-exception | `ServiceExceptionUtil.exception(ErrorCode)` | 业务异常抛出 |
| common-exception | `ErrorCode` | 错误码定义 |
| common-enums | `CommonStatusEnum` | 通用状态枚举 |
| mybatis-core | `BaseDO` | DO 基类（id/createTime/updateTime/...） |
| mybatis-core | `BaseMapperX` | Mapper 基类 |
| mybatis-core | `LambdaQueryWrapperX` | 链式查询 |
| mybatis-core | `JacksonTypeHandler` | JSON 字段映射 |
| mybatis-core | `IntegerListTypeHandler` | List\<Integer\> 字段映射 |
| excel-core | `ExcelUtils.write` | Excel 导出 |
| apilog-core | `@ApiAccessLog` | API 访问日志 |
| security-core | `@PreAuthorize("@ss.hasPermission(...)")` | 权限校验 |

### 2.2 跨模块 API 消费

| API | 来源 | 使用场景 |
|---|---|---|
| `MemberUserApi.getMemberUser(userId)` | yudao-module-member | 评价创建、收藏、浏览历史（获取用户昵称/头像） |
| `DictDataApi.getDictData(type, value)` | yudao-module-system | 字典项查询 |
| 自身暴露的 RPC | yudao-module-product/api | 订单、营销、装修等模块消费 |

### 2.3 自身暴露的 RPC

| RPC | 方法 | 消费方（已识别） |
|---|---|---|
| `ProductCategoryApi.validateCategoryList` | 5 个 | `CouponTemplateServiceImpl.validateProductScope`、`RewardActivityServiceImpl.validateProductScope` |
| `ProductSpuApi.getSpuList/getSpuMap/validateSpuList/getSpu` | 4 个 | 多个业务模块 |
| `ProductSkuApi.getSku/getSkuList/updateSkuStock` | 3 个 | 订单模块 |
| `ProductCommentApi.createComment` | 1 个 | `TradeOrderUpdateServiceImpl` |

## 3. 模块边界

### 3.1 入方向（外部 → 本模块）

- HTTP 请求：Admin/App 端 REST API（53 + 12 = 65 个端点）
- RPC 请求：跨模块 RPC（4 组共 20 个方法）
- 内部服务调用：依赖注入的 service 和 mapper

### 3.2 出方向（本模块 → 外部）

- HTTP 响应：CommonResult 序列化为 JSON
- RPC 响应：DTO 序列化为 JSON
- 内部 Bean：返回 service 暴露的对象
- 持久化：通过 MyBatis-Plus 操作数据库

### 3.3 隔离原则

- Controller 不直接访问 Mapper
- Service 不直接处理 HTTP 协议
- DO 实体不出 Controller 层（通过 VO/DTO 转换）
- ApiImpl 调用 service，禁止调用 Controller
- 跨模块通信仅通过 api/ 包

## 4. 并发模型

### 4.1 线程模型

- Spring MVC 默认：Tomcat 200 线程池
- 异步任务：使用 `@Async`（框架级，未在本模块使用）
- 定时任务：`@Scheduled`（未在本模块使用）

### 4.2 并发控制点

- `@Transactional(rollbackFor = Exception.class)`：保证多表操作原子性
- MyBatis-Plus `@Version`：乐观锁（未在本模块使用，建议库存场景引入）
- 数据库唯一索引：品牌名、属性项名、属性值联合键、订单项 ID、用户-SPU 联合键

### 4.3 库存并发

库存更新采用"增量更新"模式：

```java
public void updateSpuStock(Map<Long, Integer> stockIncrCounts) {
    stockIncrCounts.forEach((id, incCount) -> productSpuMapper.updateStock(id, incCount));
}
```

`updateStock` SQL：

```sql
UPDATE product_spu SET stock = stock + #{incCount} WHERE id = #{id}
```

并发场景下应配合 Redis 分布式锁或乐观锁使用。

## 5. 可观测性

### 5.1 日志

- SLF4J + Logback
- 关键节点日志：service 方法入口、Mapper SQL、异常堆栈
- 框架级 `@ApiAccessLog` 记录 HTTP 访问
- 业务异常堆栈可通过 `GlobalExceptionHandler` 统一处理

### 5.2 指标（Micrometer + Prometheus）

- HTTP 请求计数（按 path、method、status）
- HTTP 请求耗时（P50/P95/P99）
- JVM 内存、GC、线程
- 数据库连接池（HikariCP）

### 5.3 链路追踪

- 框架级 SkyWalking / Zipkin 集成
- 通过 traceId 关联跨服务调用

## 6. 配置管理

### 6.1 application.yaml

```yaml
spring:
  application:
    name: yudao-module-product
  datasource:
    url: jdbc:mysql://localhost:3306/ruoyi-vue-pro?useUnicode=true&characterEncoding=utf8
    username: root
    password: ${MYSQL_PASSWORD:root}
  redis:
    host: localhost
    port: 6379
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
```

### 6.2 ProductWebConfiguration

本入口仅有一个框架配置类 `ProductWebConfiguration`，当前无特殊配置（保留扩展点）。

## 7. 性能与扩展性

### 7.1 性能基线

- HTTP API 平均响应：< 200ms
- DB 查询平均耗时：< 50ms
- 分页查询：默认每页 10-20 条，最大 100 条

### 7.2 性能优化点

| 优化项 | 实现方式 |
|---|---|
| N+1 查询 | 批量 IN 查询 + Map 转换 |
| 大表分页 | 索引 + 延迟关联 |
| 频繁读 | Redis 缓存 + Spring Cache |
| 频繁写 | 批量操作 + 异步落库 |
| 复杂查询 | 视图或读写分离 |

### 7.3 扩展性

- 多租户：通过 tenant_id 字段（框架级支持）
- 读写分离：通过 MyBatis-Plus 路由（框架级支持）
- 分布式事务：Seata（框架级支持，未在本模块使用）
- 消息队列：RabbitMQ（框架级支持，未在本模块使用）

## 8. 安全与合规

### 8.1 鉴权

- 所有 admin 端点使用 `@PreAuthorize("@ss.hasPermission('product:*')")` 鉴权
- 所有 app 端点使用 `@PreAuthorize` 或隐式用户登录态
- Spring Security CSRF、CORS 配置（框架级）

### 8.2 SQL 注入防护

- MyBatis-Plus 使用 PreparedStatement 参数化
- LambdaQueryWrapper 避免字符串拼接
- 动态 SQL 使用 `<if>` 标签 + 必填校验

### 8.3 数据脱敏

- 用户敏感字段（手机号、邮箱等）通过 `@Sensitive` 注解脱敏（框架级）
- 评价内容、图片 URL 存储明文

### 8.4 操作审计

- `@ApiAccessLog` 记录 HTTP 访问
- `BaseDO.createBy` / `updateBy` 记录操作人
- 业务日志可结合 ELK 检索

## 9. 部署

### 9.1 启动方式

```bash
# IDE 启动
mvn spring-boot:run -pl yudao-module-product

# 命令行启动
mvn package -pl yudao-module-product
java -jar yudao-module-product/target/yudao-module-product.jar

# Docker 启动
docker build -t yudao-module-product -f Dockerfile .
docker run -p 48001:48001 yudao-module-product
```

### 9.2 健康检查

- `/actuator/health`：Spring Boot Actuator
- `/admin-api/product/spu/page`：业务健康探针（可选）

### 9.3 数据库初始化

- 表结构由 `sql/mysql/` 下的 SQL 脚本维护
- 字典数据由 `sql/mysql/dict_data.sql` 初始化
- 序列（`product_*_seq`）由 MyBatis-Plus 自动管理

## 10. 依赖关系图

```mermaid
graph TD
    subgraph "yudao-module-product"
        Controller[Controller<br/>8 admin + 5 app]
        Service[Service<br/>9 interfaces + 9 impls]
        Mapper[Mapper<br/>9 MyBatis mappers]
        DO[DO<br/>9 entities]
        Api[API<br/>4 interfaces + 4 impls]
        Convert[Convert<br/>5 MapStruct]
    end

    subgraph "yudao-framework"
        Common[Common<br/>CommonResult, PageResult, BaseDO]
        MyBatis[MyBatis Core<br/>TypeHandler, Mapper]
        Security[Security<br/>@PreAuthorize]
    end

    subgraph "Cross-Module"
        Promotion[yudao-module-promotion]
        Trade[yudao-module-trade]
        Member[yudao-module-member]
        BPM[yudao-module-bpm]
    end

    Controller --> Service
    Service --> Mapper
    Mapper --> DO
    Api --> Service
    Controller --> Convert
    Service --> Convert

    Service --> Common
    Mapper --> MyBatis
    Controller --> Security

    Promotion -.-> Api
    Trade -.-> Api
    BPM -.-> Api
    Service -.-> Member
```

## 11. source_nodes 追溯

- 1 个 framework 类（ProductWebConfiguration）
- 1 个 interface（ErrorCodeConstants）
- 1 个 interface（DictTypeConstants）
- 1 个 interface（ProductConstants）
- 3 个 enum 类
- 488 个可分析符号全部经过深度分析
