
---

# 团队开发规范V3.0

> 本文档用于统一团队在 Java 服务端开发中的编码规范、分层规范、Git 工作流规范以及数据库规范，确保协作流畅、代码高质量、项目可持续维护。

---

# 目录（TOC）

* [阅读须知](#阅读须知)
* [一、Java 项目规范](#一java-项目规范)

  * [1.1 项目命名规范](#11-项目命名规范)
  * [1.2 方法参数规范](#12-方法参数规范)
  * [1.3 代码目录结构](#13-代码目录结构)
  * [1.4 common 目录规范](#14-common-目录规范)
  * [1.5 module 目录规范](#15-module-目录规范)
* [二、MVC 层规范](#二mvc-层规范)

  * [2.1 分层结构](#21-分层结构)
  * [2.2 Controller 规范](#22-controller-规范)
  * [2.3 Service 规范](#23-service-规范)
  * [2.4 Manager 层规范](#24-manager-层规范)
  * [2.5 DAO 层规范](#25-dao-层规范)
  * [2.6 JavaBean 命名规范](#26-javabean-命名规范)
  * [2.7 Boolean 命名规范](#27-boolean-命名规范)
* [三、数据库规范](#三数据库规范)

  * [3.1 数据库命名](#31-数据库命名)
  * [3.2 表命名](#32-表命名)
  * [3.3 建表规范](#33-建表规范)
  * [3.4 索引规范](#34-索引规范)
* [四、Git 工作流规范](#四git-工作流规范)

  * [4.1 Git 提交信息规范](#41-git-提交信息规范)
  * [4.2 Git 分支命名规范](#42-git-分支命名规范)
  * [4.3 Git 同步代码规范](#43-git-同步代码规范)
* [五、DTO、VO、Entity 规范](#五dtovoeentity-规范)
* [六、大模块下项目管理办法](#六大模块下项目管理办法)
---

# 一、Java 项目规范

## 1.1 项目命名规范

全部采用：

* 小写字母
* 中划线连接

**示例（推荐）：**

```
mall-management-system
order-service-client
user-api
```

---

## 1.2 方法参数规范

每个方法 **最多 5 个参数**。超过 5 个必须封装为 DTO 或 Model。

理由：

* 避免调用者出错
* 避免多个相同类型参数依赖顺序
* controller/service 可读性提升

---

## 1.3 代码目录结构

```
src                               源码目录
|-- common                        通用类库
|-- config                        配置信息
|-- constant                      全局公共常量
|-- handler                       全局处理器
|-- interceptor                   全局拦截器
|-- listener                      全局监听器
|-- module                        业务模块（方便拆分微服务）
|   |-- employee                  员工模块
|   |-- role                      角色模块
|   |-- login                     登录模块
|-- third                         外部服务（redis/oss/微信 SDK 等）
|-- util                          全局工具类
|-- Application.java              启动类
```

---

## 1.4 common 目录规范

```
src
|-- common
|   |-- anno        通用注解（权限、登录等）
|   |-- constant    通用常量（如 ResponseCodeConst）
|   |-- domain      全局 JavaBean（BaseEntity、PageParamDTO）
|   |-- exception   全局异常类（BusinessException）
|   |-- json        Json 处理类
|   |-- swagger     Swagger 文档配置
|   |-- validator   通用校验器（CheckEnum、CheckBigDecimal）
```

---

## 1.5 module 目录规范

```
src
|-- module
|   |-- role
|       |-- RoleController.java
|       |-- RoleConst.java
|       |-- RoleService.java
|       |-- RoleDao.java
|       |-- domain
|           |-- RoleEntity.java
|           |-- RoleForm.java
|           |-- RoleVO.java
|   |-- employee
|   |-- login
|   |-- email
```

---

# 二、MVC 层规范

## 2.1 分层结构

* Controller
* Service
* Manager
* DAO

---

## 2.2 Controller 规范

### ① RequestMapping 只能写在方法上

避免 URL 隐藏在 class 注解里。

### ② 禁止 RESTful 强语义 URL

推荐：`/模块/子模块/动作`

示例：

```
POST /department/add
POST /department/update
GET  /department/delete/{id}
```

### ③ Swagger 必须带作者说明

```
@ApiOperation("更新部门 @author 卓大")
```

### ④ Controller 必须保持简洁

* 不写业务逻辑
* 不做复杂校验（仅限 @Valid）
* 不组装 VO

### ⑤ 只能在 Controller 获取当前用户

否则 ThreadLocal 会失效。

---

## 2.3 Service 规范

### ① 大型业务必须拆分 Service

例如订单：

* OrderQueryService
* OrderCreateService
* OrderDeliverService

### ② 谨慎使用 @Transactional

以减少长时间占用数据库连接。

建议：

* service 准备参数
* manager 执行事务

### ③ 类内方法互调不会触发事务

需：

```
@EnableAspectJAutoProxy(exposeProxy = true)
```

或拆分到 manager。

---

## 2.4 Manager 层规范

来源阿里手册：

* 封装第三方调用
* 下沉通用能力（缓存、中间件逻辑）
* DAO 组合复用

---

## 2.5 DAO 层规范

### ① 使用 Mybatis-plus

* 所有 Dao 必须继承 BaseMapper

### ② 禁止 Wrapper

避免 SQL 追踪困难。

### ③ XML 中禁止写死常量

必须使用参数传入。

### ④ join 必须使用完整表名

便于阅读与维护：

```
FROM t_notice
LEFT JOIN t_employee
```

---

## 2.6 JavaBean 命名规范

### ① 通用要求

* 不包含业务逻辑
* 所有基本类型必须用包装类型
* 必须使用 Lombok（如 @Data）
* 属性必须有多行注释
* 建议用 @Builder、@NoArgsConstructor

### ② 命名规范

| 类型        | 用途            |
| --------- | ------------- |
| XxxEntity | 数据库实体         |
| XxxDTO    | 前端请求 / 内部传输对象 |
| XxxVO     | 前端返回对象        |
| XxxForm   | 表单对象          |
| XxxBO     | 内部业务对象        |

---

## 2.7 Boolean 命名规范

禁止使用：

```
isDeleted
```

必须使用：

```
deletedFlag
onlineFlag
```

避免反序列化框架解析错误。

---

# 三、数据库规范

## 3.1 数据库命名

小写 + 下划线 + 环境标识：

```
smart_admin_v2_dev
smart_admin_v2_prod
```

---

## 3.2 表命名

全部小写 + 下划线 + `t_`：

```
t_employee
t_department
t_config
```

---

## 3.3 建表规范

必备字段：

* `id / module_id`
* `create_time`
* `update_time`

枚举字段必须写全含义：

```
sync_status TINYINT COMMENT '同步状态 0 未开始 1 同步中 2 成功 3 失败'
```

---

## 3.4 索引规范

遵循《阿里巴巴 Java 开发手册》。

---

# 四、Git 工作流规范

## 4.1 Git 提交信息规范

格式：

```
type: 描述
```

| 类型       | 用途        |
| -------- | --------- |
| feat     | 新功能       |
| fix      | 问题修复      |
| docs     | 文档更新      |
| style    | 格式化，不影响逻辑 |
| refactor | 重构        |
| test     | 测试相关      |
| chore    | 构建/工具链    |

---

## 4.2 Git 分支命名规范

| 类型       | 示例                     |
| -------- | ---------------------- |
| feature  | feature/user-auth      |
| fix      | fix/login-error        |
| refactor | refactor/login-service |
| release  | release/v1.0.0         |

---

## 4.3 Git 同步代码规范

> 尽量用 rebase 代替 merge

```bash
git checkout main
git pull --rebase

git fetch origin
git rebase origin/main
```

---

# 五、DTO、VO、Entity 规范

* Entity **只能用于数据库交互**
* DTO 用于 **接收前端参数**
* VO 用于 **返回前端**
* 即使结构一样，也必须分成 3 个类
  → 便于扩展、解耦、脱敏

---

# 六、大模块下项目管理办法

## 1. 包名规范

```
大模块名.小模块名.具体功能
```

示例：

```
model.fireinvestigation.photodossier.entity.PhotoDossier
```

---

## 2. 表名规范

```
tb_大模块前缀_表名
```

示例：

```
tb_fireinv_photo_dossier
```

实体仍保持：

```
PhotoDossier
```

---

## 3. 大模块前缀对照表

| 模块   | 大模块名              | 前缀      |
| ---- | ----------------- | ------- |
| 火灾调查 | fireinvestigation | fireinv |
| 执法监督 | fireenforcement   | fireenf |

---

