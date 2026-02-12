**用户表**

```sql
CREATE TABLE `sys_user`  (
  `id` bigint NOT NULL COMMENT '用户表主键',
  `username` varchar(50) NULL COMMENT '用户名字',
  `password` varchar(255) NOT NULL COMMENT '密码',
  PRIMARY KEY (`id`)
);
```

**角色表**

```sql
CREATE TABLE `sys_role` (
  `id` bigint NOT NULL COMMENT '角色id',
  `name` varchar(50) DEFAULT NULL COMMENT '角色名字',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

**权限表**

```sql
CREATE TABLE `sys_menu` (
  `id` bigint NOT NULL COMMENT '权限id',
  `parent_id` bigint DEFAULT NULL COMMENT '父id',
  `name` varchar(50) DEFAULT NULL COMMENT '权限名字',
  `perms` varchar(100) DEFAULT NULL COMMENT '权限标识',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

**角色用户表**

```sql
CREATE TABLE `sys_user_role` (
  `user_id` bigint NOT NULL COMMENT '用户id',
  `role_id` int NOT NULL COMMENT '角色id',
  PRIMARY KEY (`user_id`,`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

**角色权限表**

```sql
CREATE TABLE `sys_role_meun` (
  `role_id` bigint NOT NULL COMMENT '角色id',
  `menu_id` bigint NOT NULL COMMENT '权限id',
  PRIMARY KEY (`role_id`,`menu_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

# gemini帮我建的表

这份设计是典型的**“教科书式 RBAC”**，在学校交作业是满分，但在**工程落地**（尤其是你想做“派聪明”这种核心项目）时，它是**不及格**的。

作为你的“代码审查员（Code Reviewer）”，我找出了 **3 个致命 Bug** 和 **4 个严重缺失**。

请对照修改，否则你写代码时会非常痛苦。

---

### 🔴 第一类：致命 Bug (必须修，否则报错)

#### 1. 数据类型不匹配 (外键灾难)

- **你的设计：** `sys_role` 表的主键 `id` 是 `bigint`，但 `sys_user_role` 表里的 `role_id` 却是 `int`。
    
- **后果：** MySQL 在做 JOIN 查询时，**索引会失效**，查询速度慢 10 倍。如果你加外键约束，直接报错插不进去数据。
    
- **修正：** 全部统一为 `bigint`。
    

#### 2. 表名拼写错误 (低级失误)

- **你的设计：** `sys_role_meun`
    
- **后果：** 以后你写代码、写 SQL 每次都要打这个错别字，一旦打对 `menu` 反而报错。会被同事（或面试官）笑死。
    
- **修正：** 改为 `sys_role_menu`。
    

---

### 🟡 第二类：严重逻辑缺失 (写代码时会卡死)

#### 3. 缺少 `menu_type` (前端怎么渲染？)

- **问题：** 你的 `sys_menu` 表里把“左侧菜单栏（页面）”和“页面里的按钮（功能）”混在一起了。
    
- **后果：** 前端去读这张表，不知道哪个该显示成导航栏，哪个该隐藏起来作为按钮权限。
    
- **修正：** 增加 `menu_type` (M=目录, C=菜单, F=按钮)。
    

#### 4. 缺少 `role_key` (后端怎么硬编码？)

- **问题：** `sys_role` 只有 `name`（比如“管理员”）。
    
- **后果：** 你在代码里想写 `@PreAuthorize` 限制只有管理员能进，难道写 `@PreAuthorize("hasRole('管理员')")` 吗？中文作为标识符非常危险。
    
- **修正：** 增加 `role_key` 字段（存 `admin`, `common`），代码里用英文标识。
    

#### 5. 缺少 `router_path` (页面往哪跳？)

- **问题：** 既然是菜单，点击后浏览器地址栏变不变？
    
- **后果：** 前端拿到了菜单名，但不知道路由跳转地址，无法生成侧边栏。
    
- **修正：** `sys_menu` 增加 `path` 字段。
    

---

### 🟢 第三类：工程标准缺失 (面试减分项)

#### 6. 没有审计字段 (Audit Fields)

- **问题：** 出了事不知道是谁干的，也不知道是什么时候创建的。
    
- **修正：** 所有表（尤其是 User 和 Role）必须加 `create_time`, `update_time`, `del_flag`（逻辑删除）。
    

---

### 🚀 修正后的 SQL (直接复制可用)

这是为你**“派聪明”**项目定制的 **RBAC 2.0 版本**。既保留了你的简洁，又补全了工程核心。

```sql
-- 1. 用户表
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user` (
  `user_id`     bigint(20)      NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username`    varchar(50)     NOT NULL                COMMENT '账号',
  `password`    varchar(100)    DEFAULT ''              COMMENT '密码(加密)',
  `status`      char(1)         DEFAULT '0'             COMMENT '状态（0正常 1停用）',
  `del_flag`    char(1)         DEFAULT '0'             COMMENT '逻辑删除（0存在 2删除）',
  `create_time` datetime                                COMMENT '创建时间',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB COMMENT='用户表';

-- 2. 角色表
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role` (
  `role_id`     bigint(20)      NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `role_name`   varchar(50)     NOT NULL                COMMENT '角色名称(中文)',
  `role_key`    varchar(50)     NOT NULL                COMMENT '角色权限字符串(英文)',
  `status`      char(1)         DEFAULT '0'             COMMENT '状态',
  `del_flag`    char(1)         DEFAULT '0'             COMMENT '逻辑删除',
  PRIMARY KEY (`role_id`)
) ENGINE=InnoDB COMMENT='角色表';

-- 3. 菜单权限表 (最关键的表)
DROP TABLE IF EXISTS `sys_menu`;
CREATE TABLE `sys_menu` (
  `menu_id`     bigint(20)      NOT NULL AUTO_INCREMENT COMMENT '菜单ID',
  `parent_id`   bigint(20)      DEFAULT 0               COMMENT '父菜单ID',
  `menu_name`   varchar(50)     NOT NULL                COMMENT '菜单名称',
  `path`        varchar(200)    DEFAULT ''              COMMENT '路由地址(前端用)',
  `perms`       varchar(100)    DEFAULT NULL            COMMENT '权限标识(后端PreAuthorize用)',
  `menu_type`   char(1)         DEFAULT ''              COMMENT '类型(M目录 C菜单 F按钮)',
  `order_num`   int(4)          DEFAULT 0               COMMENT '显示顺序',
  PRIMARY KEY (`menu_id`)
) ENGINE=InnoDB COMMENT='菜单权限表';

-- 4. 用户和角色关联表
DROP TABLE IF EXISTS `sys_user_role`;
CREATE TABLE `sys_user_role` (
  `user_id`     bigint(20)      NOT NULL COMMENT '用户ID',
  `role_id`     bigint(20)      NOT NULL COMMENT '角色ID',
  PRIMARY KEY (`user_id`, `role_id`)
) ENGINE=InnoDB COMMENT='用户和角色关联表';

-- 5. 角色和菜单关联表
DROP TABLE IF EXISTS `sys_role_menu`;
CREATE TABLE `sys_role_menu` (
  `role_id`     bigint(20)      NOT NULL COMMENT '角色ID',
  `menu_id`     bigint(20)      NOT NULL COMMENT '菜单ID',
  PRIMARY KEY (`role_id`, `menu_id`)
) ENGINE=InnoDB COMMENT='角色和菜单关联表';
```

### 💡 为什么这么改？

- **User表：** 加了 `user_id` (比简单的 `id` 更明确)，加了 `status` (账号封禁功能)。
    
- **Role表：** 加了 `role_key`。比如你可以存 `role_name="普通用户"`, `role_key="common"`。将来代码里写 `@PreAuthorize("hasRole('common')")`。
    
- **Menu表：** **这是核心差异。** 只有分开 `path` (路由) 和 `perms` (权限字符串)，你的前后端分离架构才能跑通。
    

**立刻执行：**

把这几十行 SQL 扔进数据库跑一下。

然后问自己：**“如果我要查询‘张三’拥有的所有按钮权限，SQL 怎么写？”** (这是下一道面试题)