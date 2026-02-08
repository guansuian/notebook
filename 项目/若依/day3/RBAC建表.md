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

