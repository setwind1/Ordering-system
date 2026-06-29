点餐系统系统设计说明书

版本： v3.0.0  

文档日期： 2026年6月15日  

项目说明： 开源SaaS多门店扫码点餐系统，支持堂食/外卖/自提，集成部分微信生态

---

1. 项目概述

1.1 项目定位

点餐系统是一套基于 Java 17 + Spring Boot 3.2 + Vue 3 + uni-app 构建的完整餐饮数字化解决方案。系统面向中小型餐饮商户，提供从顾客扫码点餐、在线支付、后厨出单到商户后台管理的全链路服务。

1.2 核心业务目标

- 多门店 SaaS 架构：一套系统支撑多租户、多门店独立运营
- 全渠道覆盖：微信小程序、H5 网页、收银端三端统一
- 堂食/外卖/自提：三种就餐模式无缝切换
- 微信生态集成：公众号模板消息、小程序登录、微信支付整合

1.3 目标用户

  用户角色 	说明                    
  超级管理员	平台运营方，管理所有租户、门店和系统配置  
  门店管理员	各门店的实际运营者，管理本店商品、订单、数据
  门店员工 	收银、出餐等日常操作            
  普通顾客 	通过小程序/H5 扫码点餐的最终消费者   

---

2. 系统架构

2.1 整体架构分层

    ┌─────────────────────────────────────────────────────────────┐
    │                    用户接入层                                 │
    │   微信小程序    H5网页    PC管理后台    POS收银端              │
    └──────────────────────┬──────────────────────────────────────┘
                           │ HTTP / WebSocket
    ┌──────────────────────▼──────────────────────────────────────┐
    │                    API 网关层                                 │
    │          Spring Security OAuth2 + Knife4j 文档               │
    │          统一认证 · 限流熔断 · 日志记录 · 跨域处理            │
    └──────────────────────┬──────────────────────────────────────┘
                           │
    ┌──────────────────────▼──────────────────────────────────────┐
    │                    业务服务层                                 │
    │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
    │  │ 系统管理  │ │ 商品中心  │ │ 订单中心  │ │ 支付中心  │       │
    │  ├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤       │
    │  │ 门店管理  │ │ 会员中心  │ │ 营销中心  │ │ 积分中心  │       │
    │  ├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤       │
    │  │ 公众号管理│ │ 消息通知  │ │ 物流快递  │ │ 基础设施  │       │
    │  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
    └──────────────────────┬──────────────────────────────────────┘
                           │
    ┌──────────────────────┬──────────────────────────────────────┐
    │                    技术支撑层                                 │
    │  MyBatis-Plus · Redis · RocketMQ · Quartz · SkyWalking      │
    │  多租户 · 数据权限 · 动态数据源 · 分布式锁 · 文件存储         │
    └──────────────────────┬──────────────────────────────────────┘
                           │
    ┌──────────────────────▼──────────────────────────────────────┐
    │                     数据存储层                                │
    │          MySQL 8.0           Redis 6+         MinIO          │
    │        (关系数据)          (缓存/会话)      (文件存储)        │
    └─────────────────────────────────────────────────────────────┘

2.2 前端架构

    ┌──────────────────────────────────────────────────────────────┐
    │                       前端架构图                              │
    ├──────────────────────────────────────────────────────────────┤
    │                    PC 管理后台 (yshop-drink-vue3)              │
    │    Vue 3.5 + TypeScript + Vite 5 + Element Plus 2.11        │
    │    Pinia(状态管理) · Vue Router · Axios · ECharts            │
    │    i18n国际化 · UnoCSS · wangeditor · form-create            │
    ├──────────────────────────────────────────────────────────────┤
    │                移动端 (yshop-drink-uniapp-vue3)               │
    │    uni-app(Vue3) · uview-plus UI · Pinia · 多端编译          │
    │    微信小程序 · H5 · 跨平台统一代码                           │
    └──────────────────────────────────────────────────────────────┘

2.3 后端模块架构

项目采用多模块 Maven 项目结构，分为三个层次：

    yshop-drink-boot3/
    ├── yshop-dependencies/          # 依赖版本管理中心 (BOM)
    ├── yshop-framework/            # 框架基础设施层
    │   ├── yshop-common/           #   公共工具类、常量、枚举
    │   ├── ...spring-boot-starter-* #  15个技术Starter组件
    │   └── ...
    ├── yshop-module-system/        # 系统管理模块
    │   ├── system-api/             #   API定义、DTO、Feign接口
    │   └── system-biz/             #   业务实现层
    ├── yshop-module-mall/          # 核心商城模块
    │   ├── product-api/biz         #   商品中心
    │   ├── order-api/biz           #   订单中心
    │   ├── shop-api/biz            #   门店中心
    │   └── store-api/biz           #   店铺配置
    ├── yshop-module-member/        # 会员模块
    ├── yshop-module-marketing/     # 营销模块（优惠券）
    ├── yshop-module-pay/           # 支付模块
    ├── yshop-module-score/         # 积分商城模块
    ├── yshop-module-mp/            # 微信公众号模块
    ├── yshop-module-message/       # 消息通知模块
    ├── yshop-module-express/       # 物流快递模块
    ├── yshop-module-infra/         # 基础设施模块
    └── yshop-server/               # Spring Boot 启动入口

api/biz 拆分模式

每个业务模块拆分为 -api 和 -biz 两个子模块：

- api 层：定义对外暴露的接口、DTO、枚举、Feign 客户端
- biz 层：实现业务逻辑、数据库操作、Controller、Service

这种拆分使得模块间依赖只面向 API 接口，实现松耦合。

---

3. 技术栈

3.1 后端技术栈

  技术                         	版本   	用途        
  Java                       	17   	开发语言      
  Spring Boot                	3.2.2	微服务基础框架   
  Spring Security            	6.x  	认证与授权     
  Spring OAuth2              	6.x  	令牌认证机制    
  MyBatis-Plus               	3.5.x	ORM 框架    
  Dynamic Datasource         	4.x  	多数据源/读写分离 
  MySQL                      	8.0  	关系数据库     
  Redis                      	6+   	缓存/分布式锁/会话
  Redisson                   	3.x  	分布式锁实现    
  RocketMQ / Kafka / RabbitMQ	-    	消息队列（可选）  
  Quartz                     	2.3  	分布式任务调度   
  Knife4j                    	4.x  	API 文档生成  
  EasyExcel                  	3.x  	Excel 导入导出
  SkyWalking                 	8.x  	分布式链路追踪   
  Spring Boot Admin          	3.x  	应用监控      
  MapStruct                  	1.5  	对象映射      
  Lombok                     	1.18 	代码简化      
  Druid                      	1.2  	数据库连接池    
  Jackson                    	2.x  	JSON 序列化  

3.2 前端技术栈

  技术          	版本  	用途          
  Vue         	3.5 	前端框架        
  TypeScript  	5.x 	类型安全        
  Vite        	5.x 	构建工具        
  Element Plus	2.11	UI 组件库      
  Pinia       	2.x 	状态管理        
  Vue Router  	4.x 	路由管理        
  ECharts     	5.x 	数据可视化       
  Vue I18n    	9.x 	国际化         
  Axios       	1.x 	HTTP 客户端    
  UnoCSS      	-   	原子化 CSS     
  uni-app     	3.x 	跨端开发框架      
  uview-plus  	3.x 	uni-app UI 库

3.3 部署运行环境

  组件     	版本要求
  JDK    	17+ 
  MySQL  	8.0+
  Redis  	6+  
  Maven  	3.8+
  Node.js	16+ 
  pnpm   	8.6+

---

4. 核心业务流程

4.1 堂食点餐流程

    顾客扫码 ─→ 进入门店主页 ─→ 浏览菜单 ─→ 选规格/加购物车
                                                  │
                              ┌───────────────────┘
                              ▼
                        提交订单 ─→ 选择支付方式
                              │
                        ┌─────┴─────┐
                        ▼           ▼
                    微信支付      余额支付
                        │           │
                        └─────┬─────┘
                              ▼
                        支付成功 ─→ 打印小票
                              │
                              ▼
                        后厨接单 ─→ 制作 ─→ 出餐

4.2 外卖/自提流程

    顾客选餐 ─→ 提交订单 ─→ 支付 ─→ 商家接单
                                        │
                               ┌────────┴────────┐
                               ▼                 ▼
                            自提取餐            配送发货
                               │                 │
                               ▼                 ▼
                            核销取餐          物流跟踪

4.3 拼桌/多人点餐

    顾客A扫码 ─→ 创建桌台会话
    顾客B扫码 ─→ 加入同一桌台（WebSocket实时同步）
        │
        ├── 各自加菜到共享购物车
        ├── 各自提交个人订单
        └── 独立支付

---

5. 数据库设计

5.1 设计规范

- 字符集：utf8mb4 统一支持全 Unicode（含表情）
- 存储引擎：InnoDB
- 逻辑删除：所有业务表均包含 deleted（bit(1)）字段
- 审计字段：creator、create_time、updater、update_time
- 多租户：tenant_id（bigint）字段标识租户

5.2 核心数据模型

5.2.1 门店管理

    yshop_store_shop (门店)
    ├── id              bigint         主键
    ├── name            varchar(100)   门店名称
    ├── image           varchar(255)   门店图片
    ├── phone           varchar(20)    联系电话
    ├── address         varchar(255)   详细地址
    ├── latitude        decimal        纬度
    ├── longitude       decimal        经度
    ├── open_time       varchar(20)    营业开始时间
    ├── close_time      varchar(20)    营业结束时间
    ├── delivery_switch tinyint(1)     外卖开关
    ├── delivery_amount decimal(8,2)   起送金额
    ├── delivery_fee    decimal(8,2)   配送费
    ├── printer_config  json           云打印配置
    ├── tenant_id       bigint         租户ID
    ├── deleted         bit(1)         逻辑删除
    └── create_time     datetime       创建时间

5.2.2 商品管理

    yshop_store_product (商品)
    ├── id                  bigint          主键
    ├── image               varchar(255)    商品主图
    ├── images              text            商品轮播图
    ├── slider_image        varchar(255)    商品视频
    ├── store_name          varchar(100)    商品名称
    ├── store_info          varchar(255)    商品简介
    ├── keyword             varchar(255)    关键词
    ├── cate_id             bigint          分类ID
    ├── brand_id            bigint          品牌ID
    ├── price               decimal(8,2)    价格
    ├── ot_price            decimal(8,2)    原价
    ├── cost_price          decimal(8,2)    成本价
    ├── stock               int             总库存
    ├── sales               int             销量
    ├── is_show             tinyint(1)      上架状态
    ├── spec_type           tinyint(1)      规格类型(0=单规格/1=多规格)
    ├── temp_id             bigint          运费模板ID
    ├── give_integral       decimal(10,2)   赠送积分
    ├── sort                int             排序
    ├── content             longtext        商品详情(富文本)
    ├── tenant_id           bigint          租户ID
    ├── deleted             bit(1)          逻辑删除
    └── create_time         datetime        创建时间
    
    yshop_store_product_attr (商品属性名)
    ├── id                  bigint          主键
    ├── product_id          bigint          商品ID
    ├── attr_name           varchar(32)     属性名(如"口味")
    └── ...
    
    yshop_store_product_attr_value (商品属性值)
    ├── id                  bigint          主键
    ├── product_id          bigint          商品ID
    ├── attr_name           varchar(32)     属性名
    ├── attr_value          varchar(32)     属性值(如"辣/不辣")
    └── ...
    
    yshop_store_product_attr_result (SKU结果)
    ├── id                  bigint          主键
    ├── product_id          bigint          商品ID
    ├── value               varchar(255)    SKU组合(如"辣;大杯")
    ├── detail              varchar(255)    SKU图片
    ├── price               decimal(8,2)    SKU价格
    ├── stock               int             SKU库存
    ├── sales               int             SKU销量
    ├── cost                decimal(8,2)    SKU成本
    ├── bar_code            varchar(50)     条码
    └── ...

5.2.3 订单管理

    yshop_store_order (订单主表)
    ├── id                  bigint          主键
    ├── order_id            varchar(32)     订单号
    ├── uid                 bigint          用户ID
    ├── real_name           varchar(32)     联系人
    ├── user_phone          varchar(18)     联系电话
    ├── user_address        varchar(255)    地址
    ├── total_num           int             商品总数
    ├── total_price         decimal(8,2)    总价
    ├── total_postage       decimal(8,2)    运费
    ├── pay_price           decimal(8,2)    实付金额
    ├── pay_postage         decimal(8,2)    配送费
    ├── deduction_price     decimal(8,2)    抵扣金额
    ├── coupon_id           bigint          优惠券ID
    ├── coupon_price        decimal(8,2)    优惠券减免
    ├── paid                tinyint(1)      支付状态
    ├── pay_time            datetime        支付时间
    ├── pay_type            varchar(32)     支付方式(wechat/balance)
    ├── trade_no            varchar(100)    微信订单号
    ├── status              tinyint(1)      订单状态
    ├── refund_status       tinyint(1)      退款状态
    ├── refund_amount       decimal(8,2)    退款金额
    ├── verify_code         varchar(10)     核销码
    ├── order_type          varchar(32)     订单类型(takein/takeout)
    ├── table_id            bigint          桌号
    ├── mark                text            备注
    ├── remark              varchar(500)    商家备注
    ├── clerk_id            bigint          核销员
    ├── tenant_id           bigint          租户ID
    ├── deleted             bit(1)          逻辑删除
    └── create_time         datetime        创建时间
    
    yshop_store_order_cart_info (订单商品明细)
    ├── id                  bigint          主键
    ├── order_id            varchar(32)     订单号
    ├── product_id          bigint          商品ID
    ├── product_name        varchar(100)    商品名称
    ├── image               varchar(255)    商品图片
    ├── sku                 varchar(255)    规格说明
    ├── price               decimal(8,2)    单价
    ├── cost_price          decimal(8,2)    成本价
    ├── pay_price           decimal(8,2)    实际支付价
    ├── cart_num             int             数量
    ├── weight              decimal(8,2)    重量
    ├── is_remain           tinyint(1)      是否已退款
    ├── is_reply            tinyint(1)      是否已评价
    ├── tenant_id           bigint          租户ID
    └── ...

5.2.4 用户/会员管理

    yshop_user (用户表)
    ├── uid                 bigint          主键
    ├── phone               varchar(16)     手机号
    ├── nickname            varchar(64)     昵称
    ├── avatar              varchar(255)    头像
    ├── sex                 tinyint(1)      性别
    ├── birthday            date            生日
    ├── now_money           decimal(10,2)   余额
    ├── integral            decimal(10,2)   积分
    ├── openid              varchar(100)    微信OpenID
    ├── routine_openid      varchar(100)    小程序OpenID
    ├── unionid             varchar(100)    微信UnionID
    ├── level               tinyint(1)      等级
    ├── is_promoter         tinyint(1)      推广员
    ├── tenant_id           bigint          租户ID
    ├── deleted             bit(1)          逻辑删除
    └── create_time         datetime        注册时间

5.2.5 核心 E-R 关系

    门店 (Shop) 1 ── N 商品 (Product)
    商品 (Product) 1 ── N SKU (AttrResult)
    商品 (Product) N ── 1 分类 (Category)
    
    用户 (User) 1 ── N 订单 (Order)
    订单 (Order) 1 ── N 订单明细 (CartInfo)
    订单 (Order) N ── 1 优惠券 (CouponUser)
    
    用户 (User) N ── N 优惠券 (Coupon) [通过 CouponUser 关联]
    用户 (User) 1 ── N 地址 (Address)
    
    门店 (Shop) N ── 1 租户 (Tenant)  [多租户隔离]

5.3 数据库分表策略

- 按租户隔离：所有业务表通过 tenant_id 实现数据隔离（共享数据库模式）
- 逻辑删除：所有业务表使用 deleted 字段标记，避免物理删除造成的数据丢失
- 时间分区：日志类表（system_login_log、system_operate_log、infra_api_access_log）可按月分区

5.4 缓存设计

  缓存对象 	Redis 数据结构   	过期策略    	说明                 
  用户会话 	String       	TTL 30天 	OAuth2 Access Token
  商品详情 	String(JSON) 	TTL 1小时 	热点商品缓存             
  门店信息 	String(JSON) 	TTL 30分钟	门店基础数据             
  分类树  	String(JSON) 	TTL 1小时 	商品分类层级             
  分布式锁 	Redisson Lock	自动释放    	订单防重复提交            
  短信验证码	String       	TTL 5分钟 	登录/注册验证码           
  字典数据 	String(JSON) 	TTL 24小时	系统字典缓存             

---

6. API 设计

6.1 API 命名规范

采用 RESTful 风格，URL 路径按访问端分类：

  前缀                            	面向    	示例                                   
  /admin-api/{module}/{resource}	PC管理后台	/admin-api/product/store-product/page
  /app-api/{module}/{resource}  	移动端   	/app-api/product/{id}                

6.2 接口安全设计

- 认证方式：Bearer Token（OAuth2 JWT）
- 接口鉴权：Spring Security + @PreAuthorize 注解，角色-菜单-权限三级管控
- 防重复提交：基于 Redis 的幂等性校验注解 @Idempotent
- 频率限制：基于分布式限流注解 @RateLimiter
- XSS 过滤：全局 XSS 过滤器，对请求参数进行转义
- 参数校验：@Valid + 分组校验 + 自定义校验器

6.3 统一响应格式

    {
      "code": 0,
      "msg": "success",
      "data": { ... }
    }

- code == 0 表示成功，非零值表示具体错误码
- 错误码在 system_error_code 表中统一管理

6.4 端点分组

  分组      	功能描述         	典型端点                           
  认证中心    	登录/登出/注册/刷新令牌	/auth/login、/auth/logout       
  商品中心    	商品CRUD、分类、评价 	/product/*、/category/*         
  订单中心    	下单/支付/退款/核销  	/order/*                       
  门店中心    	门店信息/营业设置    	/store/*                       
  会员中心    	用户信息/积分/地址   	/member/*、/address/*           
  支付中心    	支付下单/回调      	/pay/*                         
  营销中心    	优惠券管理/发放     	/coupon/*                      
  积分中心    	积分商品/兑换订单    	/score-product/*、/score-order/*
  公众号（未完善）	菜单/素材/自动回复   	/mp/*                          
  基础设施    	文件/定时任务/日志   	/infra/file/*、/infra/job/*     

---

7. 安全设计

7.1 认证与授权

    ┌──────────┐    ┌──────────────┐    ┌─────────────┐
    │  客户端   │───▶│  OAuth2 认证  │───▶│  令牌签发    │
    │          │◀───│  服务器       │◀───│  (JWT)      │
    └──────────┘    └──────────────┘    └─────────────┘
                          │                      │
                          ▼                      ▼
                  ┌──────────────┐    ┌──────────────────┐
                  │  密码/短信    │    │  Token 校验/刷新   │
                  │  微信登录     │    │  权限校验          │
                  └──────────────┘    └──────────────────┘

- 多端多因子认证：支持账号密码、短信验证码、微信授权码等多种方式
- 双令牌机制：Access Token（短期）+ Refresh Token（长期）
- 权限模型：用户 → 角色（多对多）→ 菜单/权限（多对多），基于RBAC

7.2 数据安全

- 密码加密：BCrypt 哈希存储
- 传输加密：HTTPS 全链路
- 参数过滤：HTML 标签转义防止 XSS
- SQL 注入防护：MyBatis-Plus 参数绑定
- 数据隔离：多租户自动拼接 tenant_id 过滤条件

7.3 操作审计

- 登录日志：记录每次登录的 IP、设备、时间
- 操作日志：记录重要业务操作的操作人、操作内容、请求参数
- API 日志：记录所有 API 调用的耗时、状态码、异常信息

---

8. 基础设施设计

8.1 多租户架构

采用共享数据库、共享Schema、租户ID隔离的模式：

- 所有业务表包含 tenant_id 字段
- 通过 MyBatis-Plus 拦截器自动注入租户过滤条件
- 支持部分表忽略租户过滤（如系统配置表、字典表）
- 租户套餐（system_tenant_package）控制各租户的功能权限和资源限制

8.2 文件存储

支持两种存储模式：

- 本地存储：开发环境使用，文件存储于服务器磁盘
- MinIO 对象存储：生产环境推荐，兼容 S3 API，支持分布式部署

8.3 消息队列

可配置的消息中间件，用于以下异步场景：

  场景          	消息内容       	建议队列    
  订单状态变更通知    	订单ID + 新状态 	RocketMQ
  云打印         	订单ID + 门店ID	RocketMQ
  WebSocket 推送	消息体        	RocketMQ
  短信/邮件发送     	模板ID + 参数  	RocketMQ

8.4 定时任务

基于 Quartz 的分布式任务调度，支持：

- 任务动态管理（新增、修改、暂停、删除）
- Cron 表达式配置
- 执行日志记录
- 失败重试机制

典型定时任务：

  任务       	说明       	频率   
  自动取消未支付订单	超时未支付自动取消	每1分钟 
  自动确认收货   	发货后超时自动确认	每30分钟
  优惠券过期处理  	批量更新过期优惠券	每天凌晨 
  数据统计     	生成日报/月报  	每天凌晨 

8.5 WebSocket 推送

- 用于拼桌场景的实时数据同步
- 基于消息队列广播的统一推送通道
- 支持按门店/桌号维度定向推送

---

9. 部署架构

9.1 推荐部署方案（生产环境）

                                   ┌─────────────────┐
                                   │    Nginx 反向代理  │
                                   │  SSL终止 / 负载均衡 │
                                   └────────┬────────┘
                                            │
                        ┌───────────────────┴───────────────────┐
                        │                                       │
                ┌───────▼───────┐                       ┌───────▼───────┐
                │  管理后台      │                       │  移动端资源    │
                │  (静态资源)    │                       │  (静态资源)    │
                └───────────────┘                       └───────────────┘
                        │                                       │
                        └───────────────────┬───────────────────┘
                                            │
                                ┌───────────▼───────────┐
                                │   Spring Boot 集群     │
                                │    (多实例部署)         │
                                └───────────┬───────────┘
                                            │
                        ┌───────────────────┼───────────────────┐
                        │                   │                   │
                ┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
                │    MySQL 8.0   │   │   Redis 6+    │   │   MinIO       │
                │   (主从+读写分离)│   │  (主从/Sentinel)│   │  (对象存储)   │
                └───────────────┘   └───────────────┘   └───────────────┘

9.2 Docker Compose 快速部署

项目根目录提供 Docker Compose 配置，一键启动所有依赖服务：

    services:
      mysql:     # MySQL 8.0 数据库
      redis:     # Redis 6+ 缓存
      app:       # Spring Boot 应用
      vue:       # Nginx 托管前端静态资源

---

10. 前端设计

10.1 管理后台架构

    ┌─────────────────────────────────────────────────────────────┐
    │                    管理后台 (Vue 3 + Element Plus)            │
    ├─────────────────────────────────────────────────────────────┤
    │  Layout ▶ 侧边栏 · 顶栏 · 标签页 · 面包屑 · 用户信息          │
    │  Router ▶ 按模块组织路由                                     │
    │  Store  ▶ Pinia 模块化状态管理                                │
    │  API    ▶ Axios 封装 · 请求拦截 · 响应拦截 · 错误处理         │
    │  Hooks  ▶ 可组合逻辑复用                                     │
    │  Directives ▶ v-permission(权限) · v-copy(复制)              │
    │  Components ▶ 通用组件库(上传/富文本/二维码/地图/图表)        │
    │  Styles ▶ 全局样式 · UnoCSS 原子化                           │
    │  Locales ▶ i18n 国际化(中/英)                                │
    │  Plugins ▶ ECharts · 地图 · 引导教程                        │
    └─────────────────────────────────────────────────────────────┘

10.2 移动端页面结构

    ┌─────────────────────────────────────────────────────────────┐
    │              移动端 (uni-app + uview-plus)                    │
    ├─────────────────────────────────────────────────────────────┤
    │  Pages:                                                     │
    │  ├── 首页 (门店信息/公告/分类/推荐)                            │
    │  ├── 菜单 (商品列表/规格选择/加入购物车)                       │
    │  ├── 购物车 (数量调整/清空/结算)                               │
    │  ├── 订单 (下单/支付/催单/核销)                               │
    │  └── 我的 (个人信息/地址/优惠券/积分/订单历史)                  │
    │  Components:                                                │
    │  ├── 商品卡片 · 数量选择器 · 规格弹窗                         │
    │  ├── 购物车组件 · 地址选择器 · 支付组件                       │
    │  └── 搜索栏 · 轮播图 · 公告栏                                │
    └─────────────────────────────────────────────────────────────┘

---

11. 集成设计

11.1 微信生态集成

  功能          	说明                   	涉及模块   
  微信小程序登录     	OAuth2 授权码模式获取 OpenID	member 
  微信支付        	JSAPI 调起支付 / 退款      	pay    
  公众号模板消息（未完善）	订单状态变更通知             	message
  公众号菜单管理（未完善）	自定义菜单配置              	mp     
  公众号素材管理（未完善）	图文消息管理               	mp     
  公众号自动回复（未完善）	关键词/关注自动回复           	mp     

11.2 云打印集成

支持多家云打印服务商：

- 订单支付后自动推送打印任务
- 按门店配置打印机
- 支持打印联数设置

11.3 物流快递集成

- 对接快递鸟（KdNiao）和快递100（Kd100）API
- 实时物流轨迹查询
- 电子面单打印

---

12. 项目目录引用

    yshop-drink-master/
    ├── README.md                          # 项目说明文档
    ├── LICENSE                            # MIT 开源许可
    │
    ├── yshop-drink-boot3/                 # 后端 Java 项目
    │   ├── pom.xml                        # 根 POM，模块声明
    │   ├── yshop-dependencies/            # 依赖版本管理
    │   ├── yshop-framework/               # 框架基础设施 (15 个 Starter)
    │   ├── yshop-module-system/           # 系统管理模块
    │   ├── yshop-module-mall/             # 商城核心模块
    │   │   ├── product-api/biz            #   商品中心
    │   │   ├── order-api/biz              #   订单中心
    │   │   ├── shop-api/biz               #   门店中心
    │   │   └── store-api/biz              #   店铺配置
    │   ├── yshop-module-member/           # 会员模块
    │   ├── yshop-module-marketing/        # 营销模块
    │   ├── yshop-module-pay/              # 支付模块
    │   ├── yshop-module-score/            # 积分模块
    │   ├── yshop-module-mp/               # 公众号模块
    │   ├── yshop-module-message/          # 消息模块
    │   ├── yshop-module-express/          # 物流模块
    │   ├── yshop-module-infra/            # 基础设施模块
    │   ├── yshop-server/                  # Spring Boot 启动入口
    │   └── sql/                           # 数据库初始化脚本
    │
    ├── yshop-drink-vue3/                  # PC 管理后台前端
    │   ├── src/
    │   │   ├── api/                       #   接口层
    │   │   ├── views/                     #   页面组件
    │   │   ├── router/                    #   路由
    │   │   ├── store/                     #   状态管理
    │   │   ├── components/                #   公共组件
    │   │   ├── hooks/                     #   组合式函数
    │   │   ├── layouts/                   #   布局组件
    │   │   ├── locales/                   #   国际化
    │   │   ├── styles/                    #   全局样式
    │   │   ├── utils/                     #   工具函数
    │   │   ├── directives/                #   自定义指令
    │   │   └── plugins/                   #   插件
    │   └── ...
    │
    └── yshop-drink-uniapp-vue3/           # 移动端前端
        ├── pages/                         #   页面
        │   ├── index/                     #     首页
        │   ├── menu/                      #     菜单
        │   ├── order/                     #     订单
        │   ├── cart/                      #     购物车
        │   └── mine/                      #     个人中心
        ├── api/                           #   接口层
        ├── components/                    #   组件
        ├── store/                         #   状态管理
        ├── utils/                         #   工具函数
        ├── hooks/                         #   组合式函数
        ├── static/                        #   静态资源
        └── config/                        #   配置

---

13. 质量属性

13.1 可扩展性

- 数据源扩展：基于 dynamic-datasource 支持读写分离、多主多从
- 消息队列：RocketMQ / Kafka / RabbitMQ 灵活切换
- 代码生成：内置代码生成器，通过配置表即可快速生成完整 CRUD 模块

13.2 可维护性

- 模块化解耦：业务模块通过 API 接口依赖，可独立开发测试
- 统一异常处理：全局异常处理器 + 错误码管理
- 监控体系：SkyWalking 链路追踪 + Spring Boot Admin 健康检查
- 日志规范：Logback 配置 + API 访问/错误日志

13.3 可用性

- 服务集群：Spring Boot 应用支持水平扩展
- 缓存保护：Redis 缓存降低数据库压力
- 限流保护：接口级别限流，防止突发流量打垮服务
- 幂等性：关键接口防重复提交

---

14. 开发环境配置

14.1 后端启动

    cd yshop-drink-boot3
    # 导入 SQL 脚本到 MySQL
    mysql -u root -p < sql/yixiang-drink-open.sql
    
    # 修改 application-local.yaml 中的数据库/Redis 配置
    
    # 启动服务
    mvn clean package -DskipTests
    java -jar yshop-server/target/yshop-server.jar --spring.profiles.active=local

14.2 管理后台启动

    cd yshop-drink-vue3
    pnpm install
    pnpm dev

14.3 移动端启动

使用 HBuilderX 打开 yshop-drink-uniapp-vue3 目录，运行到微信小程序或浏览器。

---
