# mybatis plus时间字段自动注入

1. 加入处理器

   ```java
   @Component
   @Slf4j
   public class MyMetaObjectHandler implements MetaObjectHandler {

       @Override
       public void insertFill(MetaObject metaObject) {
           log.info("autoFill insert...");
           this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
       }

       @Override
       public void updateFill(MetaObject metaObject) {
           log.info("autoFill update...");
           this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
       }
   }
   ```

   ​

2. 在实体类的属性上添加注解：

   ```java
   @TableField(value = "create_time", fill = FieldFill.INSERT)
   private LocalDateTime createTime;
   ```

### 踩坑

使用自动填充的时候，如果填充的实体类中的填充自动有值，则按照该值进行填充，如果为空，才按照填充规则进行填充。