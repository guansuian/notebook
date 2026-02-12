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
