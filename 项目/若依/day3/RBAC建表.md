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

