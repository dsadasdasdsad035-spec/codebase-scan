# 架构说明：商城商品中心后端 (backend-package-yudao-module-product)

入口 ID：backend-package-yudao-module-product
入口类型：backend_package
源码范围：ruoyi-vue-pro/yudao-module-mall/yudao-module-product/**
证据：evidence/backend-package-yudao-module-product/{inventory,nodes,typecards}.json
配套前端：yudao-ui-admin-vue3 / src/views/mall/product

---

## 1. 入口职责 (Responsibilities)

本入口为"商城"模块下"商品中心"的后端 Spring Boot 模块，对应包前缀 `cn.iocoder.yudao.module.product`。共 7 大子域、128 个 Java 源文件、2132 个原生节点、488 个可分析符号（覆盖 25 个语义分类）。

| 子域 | 路径前缀 | 核心文件 | 职责 |
|---|---|---|---|
| 商品 SPU/SKU | `controller/admin/spu/`、`service/spu/`、`dal/dataobject/spu/` | ProductSpuController、ProductSpuServiceImpl、ProductSpuDO、ProductSkuDO | 商品发布、5-Tab 状态流转、SKU 编辑、价格汇总 |
| 商品分类 | `controller/admin/category/`、`service/category/`、`dal/dataobject/category/` | ProductCategoryController、ProductCategoryServiceImpl、ProductCategoryDO | 树形分类 CRUD、层级校验、级联删除保护 |
| 商品品牌 | `controller/admin/brand/`、`service/brand/`、`dal/dataobject/brand/` | ProductBrandController、ProductBrandServiceImpl、ProductBrandDO | 品牌 CRUD、名称唯一性校验 |
| 商品属性 | `controller/admin/property/`、`service/property/`、`dal/dataobject/property/` | ProductPropertyController、ProductPropertyServiceImpl、ProductPropertyDO、ProductPropertyValueDO | 属性项与属性值两级管理 |
| 商品评价 | `controller/admin/comment/`、`service/comment/`、`dal/dataobject/comment/` | ProductCommentController、ProductCommentServiceImpl、ProductCommentDO | 评价查询、可见性切换、商家回复 |
| 商品收藏 | `controller/app/favorite/`、`service/favorite/`、`dal/dataobject/favorite/` | AppFavoriteController、ProductFavoriteServiceImpl、ProductFavoriteDO | 用户收藏/取消收藏 |
| 浏览历史 | `controller/app/history/`、`service/history/`、`dal/dataobject/history/` | AppProductBrowseHistoryController、ProductBrowseHistoryServiceImpl、ProductBrowseHistoryDO | 用户浏览足迹记录与查询 |
| 跨模块 RPC | `api/{category,comment,sku,spu,property}/` | ProductCategoryApi、ProductCommentApi、ProductSkuApi、ProductSpuApi 及其 Impl | 对外提供商品/分类/评价/SKU 校验与查询能力 |
| 对象转换 | `convert/{brand,category,comment,favorite,sku,spu}/` | ProductSpuConvert 等 MapStruct 接口 | DO/VO/DTO 之间的字段映射 |

## 2. 模块依赖 (Dependencies)

### 2.1 框架与基础设施
- Spring Boot + Spring MVC（`@RestController`、`@RequestMapping`、`@Validated`）
- MyBatis-Plus（`BaseMapper`、`@TableName`、`@KeySequence`、`JacksonTypeHandler`、`IntegerListTypeHandler`）
- Spring Security（`@PreAuthorize("@ss.hasPermission('product:*')")`）
- Spring 事务（`@Transactional(rollbackFor = Exception.class)`）
- Lombok（`@Data`、`@Builder`、`@EqualsAndHashCode(callSuper = true)`、`@ToString(callSuper = true)`）
- MapStruct（`ProductSpuConvert.INSTANCE.convert(...)`）
- Hutool（`cn.hutool.core.collection.CollUtil`、`cn.hutool.core.util.ObjectUtil`）
- Guava（`com.google.common.collect.Maps.newLinkedHashMapWithExpectedSize`）
- Swagger v3（`@Tag`、`@Operation`、`@Parameter`）
- Excel 导出（`cn.iocoder.yudao.framework.excel.core.util.ExcelUtils`）
- API 日志（`@ApiAccessLog(operateType = EXPORT)`）

### 2.2 内部业务依赖（同模块）
- `cn.iocoder.yudao.framework.common.pojo.{CommonResult,PageResult}` — 统一返回结构
- `cn.iocoder.yudao.framework.common.util.collection.CollectionUtils` — 集合转换工具
- `cn.iocoder.yudao.framework.common.util.object.BeanUtils` — Bean 拷贝
- `cn.iocoder.yudao.framework.common.exception.util.ServiceExceptionUtil.exception` — 业务异常抛出
- `cn.iocoder.yudao.framework.common.enums.CommonStatusEnum` — 通用状态枚举（0=禁用，1=启用）
- `cn.iocoder.yudao.framework.mybatis.core.dataobject.BaseDO` — DO 基类（id/createTime/updateTime/createBy/updateBy/deleted）
- `cn.iocoder.yudao.framework.mybatis.core.query.LambdaQueryWrapperX` — 链式查询构造

### 2.3 跨模块依赖（通过 Spring `@Lazy` 解循环）
- `ProductCategoryService` ↔ `ProductSpuService`：分类删除校验 SPU 数量、SPU 保存校验分类层级
- `ProductBrandService` ← `ProductSpuService`：SPU 保存校验品牌有效性
- `ProductSkuService` ← `ProductSpuService`：SPU 保存/更新时同步 SKU、删除 SPU 时级联删除 SKU

### 2.4 跨模块 RPC 暴露
- `ProductCategoryApi.validateCategoryList(ids)` — 校验商品分类有效性（被 yudao-module-promotion 等模块的 `validateProductScope` 等调用）
- `ProductCommentApi.createComment(reqDTO)` — 创建商品评价（被 yudao-module-trade 等订单模块的 TradeOrderUpdateServiceImpl 调用）
- `ProductSkuApi.getSku / getSkuList / updateSkuStock` — SKU 查询与库存更新（被 yudao-module-trade 订单创建与取消流程调用）
- `ProductSpuApi.getSpu / getSpuList / getSpuMap / validateSpuList` — SPU 校验与查询（被 yudao-module-promotion、yudao-module-trade、yudao-module-bpm 等业务模块调用）

### 2.5 跨模块 RPC 消费（来自其他模块）
- `MemberUserApi` — 获取用户昵称/头像（评价创建、收藏、浏览历史场景）
- `DictDataApi` — 字典项查询（评价状态、SPU 状态映射）

## 3. 分层架构 (Layered Architecture)

入口采用标准的"四层架构 + 跨模块 RPC"模式：

```
┌─────────────────────────────────────────────────────────┐
│  Controller  layer (controller/admin/, controller/app/) │
│  - REST 端点定义、@PreAuthorize 权限、参数校验、Bean 转换  │
└───────────────────────────┬─────────────────────────────┘
                            │ 依赖 Service 接口
┌───────────────────────────▼─────────────────────────────┐
│  Service     layer (service/ 接口 + *Impl 实现)          │
│  - 业务编排、事务管理、跨子域 RPC 串联、参数与权限校验       │
└───────────────────────────┬─────────────────────────────┘
                            │ 依赖 Mapper 接口
┌───────────────────────────▼─────────────────────────────┐
│  Repository  layer (dal/mysql/*Mapper.java)              │
│  - MyBatis-Plus 持久化、动态 SQL、分页、复杂统计           │
└───────────────────────────┬─────────────────────────────┘
                            │ 映射
┌───────────────────────────▼─────────────────────────────┐
│  Entity      layer (dal/dataobject/*DO.java)             │
│  - 数据库表结构、表名、字段映射、TypeHandler              │
└─────────────────────────────────────────────────────────┘
                        辅助：Convert（DO↔VO↔DTO）、api/（RPC 暴露）
```

### 3.1 包结构
```
cn.iocoder.yudao.module.product
├── controller
│   ├── admin/                      # 管理后台 REST 控制器（8 个 Controller）
│   │   ├── brand/                  # 商品品牌
│   │   ├── category/               # 商品分类
│   │   ├── comment/                # 商品评价
│   │   ├── favorite/               # 商品收藏
│   │   ├── history/                # 浏览历史
│   │   ├── property/               # 商品属性（项+值）
│   │   ├── spu/                    # 商品 SPU
│   │   └── vo/                     # 请求/响应 VO
│   └── app/                        # 用户端 REST 控制器（5 个 Controller）
│       ├── category/               # 分类列表（App 端）
│       ├── comment/                # 评价提交/查询（App 端）
│       ├── favorite/               # 收藏（App 端）
│       ├── history/                # 浏览历史（App 端）
│       ├── spu/                    # SPU 列表/详情（App 端）
│       └── vo/                     # App 端 VO
├── service                         # 业务服务
│   ├── brand/                      # ProductBrandService + Impl
│   ├── category/                   # ProductCategoryService + Impl
│   ├── comment/                    # ProductCommentService + Impl
│   ├── favorite/                   # ProductFavoriteService + Impl
│   ├── history/                    # ProductBrowseHistoryService + Impl
│   ├── property/                   # ProductPropertyService + ProductPropertyValueService + Impl
│   ├── sku/                        # ProductSkuService + Impl
│   └── spu/                        # ProductSpuService + Impl
├── dal                             # 数据访问层
│   ├── dataobject/                 # DO 实体（9 个）
│   │   ├── brand/ProductBrandDO
│   │   ├── category/ProductCategoryDO
│   │   ├── comment/ProductCommentDO
│   │   ├── favorite/ProductFavoriteDO
│   │   ├── history/ProductBrowseHistoryDO
│   │   ├── property/ProductPropertyDO、ProductPropertyValueDO
│   │   ├── sku/ProductSkuDO
│   │   └── spu/ProductSpuDO
│   └── mysql/                      # Mapper 接口（9 个）
├── api                             # 跨模块 RPC 暴露
│   ├── category/                   # ProductCategoryApi + Impl
│   ├── comment/                    # ProductCommentApi + Impl
│   ├── property/                   # ProductPropertyValueDetailRespDTO
│   ├── sku/                        # ProductSkuApi + Impl + ProductSkuRespDTO + ProductSkuUpdateStockReqDTO
│   └── spu/                        # ProductSpuApi + Impl + ProductSpuRespDTO
├── convert                         # MapStruct 对象转换器
│   ├── brand/ProductBrandConvert
│   ├── comment/ProductCommentConvert
│   ├── favorite/ProductFavoriteConvert
│   ├── sku/ProductSkuConvert
│   └── spu/ProductSpuConvert
├── enums                           # 枚举与错误码
│   ├── DictTypeConstants           # 字典类型常量
│   ├── ErrorCodeConstants          # 业务错误码（8 组共 24 条）
│   ├── ProductConstants            # 模块常量
│   ├── comment/                    # ProductCommentAuditStatusEnum、ProductCommentScoresEnum
│   └── spu/ProductSpuStatusEnum    # SPU 销售状态
└── framework                       # 模块配置
    └── web/config/ProductWebConfiguration
```

## 4. 设计模式与横切关注点

### 4.1 业务校验分层
- Controller 层：使用 `@Valid` + Jakarta Validation 注解（`@NotNull`、`@Size` 等）做参数格式校验
- Service 层：业务规则校验（存在性、唯一性、状态合法性、层级合法性）
- 数据库层：唯一索引作为最后一道防线

### 4.2 循环依赖处理
- `ProductCategoryService` ↔ `ProductSpuService`：使用 `@Lazy` 注解打破循环
- 实际调用时机是分层触发的，不会发生初始化时的循环调用

### 4.3 价格单位
- 所有价格/积分/佣金钱段字段统一使用"分"为单位（`Integer`），避免浮点精度问题
- 前端展示时通过 `fenToYuan` / `formatToFraction` 等工具进行分↔元换算

### 4.4 库存扣减
- 库存变更通过 `ProductSpuService.updateSpuStock(Map<Long, Integer>)` 批量增量更新
- SKU 库存独立维护，SPU 库存是所有 SKU 库存的 sum

### 4.5 跨模块幂等性
- 商品评价 `createComment` 校验 `COMMENT_ORDER_EXISTS`，避免同一订单重复评价
- 商品收藏 `createFavorite` 校验 `FAVORITE_EXISTS`，避免重复收藏

### 4.6 状态机
- SPU 状态由 `ProductSpuStatusEnum` 定义：RECYCLE(-1) → DISABLE(0) → ENABLE(1)
- 状态变更通过 `updateSpuStatus` 端点切换，删除仅允许 RECYCLE 状态
- 评价可见性由 `visible` 布尔字段 + `replyStatus` 字段共同控制

## 5. 集成状态 (Integration Status)

**已接入主流程**：
- 8 个管理后台 Controller 共提供 53 个 HTTP 端点，覆盖商品中心 5 大子域的 CRUD 全部场景
- 5 个用户端 Controller 共提供 12 个 HTTP 端点，覆盖 App 端商品浏览、评价、收藏、足迹场景
- 4 个跨模块 RPC（Api + ApiImpl）共 20 个 RPC 方法，已被 yudao-module-promotion、yudao-module-trade 等业务模块消费
- 5 个对象转换器（MapStruct）覆盖 9 类实体的 DO↔VO↔DTO 互转
- 9 个 MyBatis-Plus Mapper 提供 40 个查询/更新方法，支持分页、复杂统计、增量更新

**模块解耦**：
- 通过 `api/` 包向其他模块暴露能力，避免直接 service 依赖
- 通过 `convert/` 隔离数据传输对象与持久化对象
- 通过 `enums/ErrorCodeConstants` 集中管理错误码（1-008-000-000 段）

**外部依赖**：
- yudao-framework（CommonResult、PageResult、BaseDO、ErrorCode 等基础设施）
- yudao-module-system（DictDataApi、MemberUserApi 等系统模块）
- yudao-module-promotion、yudao-module-trade、yudao-module-bpm 等业务模块通过 RPC 消费

## 6. source_nodes 追溯

- 8 个 controller 类节点 ID（详见 controller_method 与 endpoint 节点）
- 5 个 app_controller 类节点 ID
- 9 个 service 类与 9 个 service_contract 接口节点 ID
- 9 个 repository（Mapper）节点 ID
- 4 个 rpc_api 与 4 个 rpc_contract 节点 ID
- 5 个 convert 节点 ID
- 9 个 entity（DO）节点 ID
- 3 个 enum 节点 ID
- 共 488 个可分析符号，覆盖 7 大子域
- 54 个 HTTP endpoint 节点、20 个 RPC method 节点
