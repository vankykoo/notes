## 一、linux上，MySQL忘记密码

1. 修改`/etc/my.cnf`文件，在[mysqld]中加上`skip-grant-tables`，保存退出。

2. 重启mysqld

3. mysql -u root -p登录mysql，不用输密码直接回车。

4. ```mysql
   use mysql
   update user set authentication_string=password('你要修改的密码') where user='root';
   //刷新系统权限表
   flush privileges;
   ```

5. 如果想让外部也能用这个账号密码登陆，执行`update user set host='%' where user='root123';`

6. 退出mysql，注释掉`skip-grant-tables`，重启即可。



## 二、密码等级太低，报错

可以降低密码等级限制：

先登录mysql：

```mysql
show variables like '%validate_password%'; # 查看密码策略
set global validate_password_policy=LOW; # 修改密码策略等级为LOW
set global validate_password_length=4; # 密码的最小长度
set global validate_password_mixed_case_count=0; # 设置密码中至少要包含0个大写字母和小写字母
set global validate_password_number_count=0; # 设置密码中至少要包含0个数字
set global validate_password_special_char_count=0; # 设置密码中至少要包含0个特殊字符
```



## 三、配置主从服务

可以看[Linux系统中MySQL数据库主从搭建（步骤详细、小白也能轻松完成） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/476752527)



