# RBAC 模型数据库表参考

## 1、用户表

```sql
CREATE TABLE `user` (
 `id` bigint NOT NULL AUTO_INCREMENT COMMENT '用户ID',
 `username` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '用户账号',
 `nickname` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '用户昵称',
 `email` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '用户邮箱',
 `mobile` varchar(11) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '手机号码',
 `sex` int DEFAULT '0' COMMENT '用户性别（0男 1女 2未知）',
 `avatar` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '头像地址',
 `password` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT '' COMMENT '密码',
 `status` int DEFAULT '0' COMMENT '帐号状态（0正常 1停用）',
 `creator` bigint DEFAULT '1' COMMENT '创建者',
 `create_time` datetime DEFAULT NULL COMMENT '创建时间',
 `updater` bigint DEFAULT '1' COMMENT '更新者',
 `update_time` datetime DEFAULT NULL COMMENT '更新时间',
 `remark` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '备注',
 `deleted` tinyint DEFAULT '0',
 PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4
```



## 2、角色表

```sql
CREATE TABLE `role` (
 `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '角色id',
 `role_label` VARCHAR(255) DEFAULT NULL COMMENT '角色标识',
 `role_name` VARCHAR(255) DEFAULT NULL COMMENT '角色名字',
 `sort` INT DEFAULT NULL COMMENT '排序',
 `status` INT DEFAULT NULL COMMENT '状态：0：可用，1：不可用',
 `deleted` INT DEFAULT NULL COMMENT '是否删除：0: 未删除，1：已删除',
 `remark` VARCHAR(255) DEFAULT NULL COMMENT '备注',
 `create_time` DATETIME DEFAULT NULL COMMENT '创建时间',
 `update_time` DATETIME DEFAULT NULL COMMENT '修改时间',
 PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;
```



## 3、权限表

```sql
CREATE TABLE `permission` (
 `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
 `parent_id` BIGINT NOT NULL DEFAULT '0' COMMENT '父id',
 `permission_name` VARCHAR(255) DEFAULT NULL COMMENT '菜单名',
 `path` VARCHAR(255) NOT NULL DEFAULT '/' COMMENT '匹配路径',
 `sort` INT DEFAULT '0' COMMENT '排序',
 `permission_type` INT DEFAULT NULL COMMENT '类型：0，目录，1菜单，2：按钮',
 `perms` VARCHAR(255) DEFAULT NULL COMMENT '权限标识',
 `icon` VARCHAR(255) DEFAULT NULL COMMENT '图标',
 `deleted` INT DEFAULT NULL COMMENT '是否删除',
 `create_time` DATETIME DEFAULT NULL COMMENT '创建时间',
 `update_time` DATETIME DEFAULT NULL COMMENT '修改时间',
 PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4;
```



## 4、用户-角色 映射表

```sql
CREATE TABLE `user_role` (
 `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键id',
 `user_id` BIGINT NOT NULL COMMENT '用户id',
 `role_id` BIGINT NOT NULL COMMENT '角色id',
 PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4;
```



## 5、角色-权限映射表

```sql
CREATE TABLE `user_role` (
 `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键id',
 `user_id` BIGINT NOT NULL COMMENT '用户id',
 `role_id` BIGINT NOT NULL COMMENT '角色id',
 PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4;
```

