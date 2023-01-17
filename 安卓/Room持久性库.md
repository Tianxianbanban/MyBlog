## Room持久性库



### **Room是什么？**

+ [官方-Room持久性库](https://developer.android.com/topic/libraries/architecture/room)



### **Room如何使用？**

**基本使用Demo**

+ [官方-使用Room将数据保存到本地数据库](https://developer.android.com/training/data-storage/room)

+ [LiveData + ViewModel + Room (Google 官文)+Demo](https://juejin.im/post/5a1f983cf265da43252913f8#comment)

+ [Android Jetpack Room的详细教程](https://juejin.im/post/5ebac942e51d454dea6fe3e9)

  

**官方教程**

+ 使用实体定义数据

+ 定义对象之间的关系

+ 在数据库中创建视图

+ 使用DAO访问数据

  + @ColumnInfo

  + @Dao

  + @Database

  + @DatabaseView

  + @Delete

  + @Embedded

  + @Entity

  + @ForeignKey

  + @Fts3

  + @Fts4

  + @Ignore

  + @Index 

  + @Insert

  + @OnConflictStrategy

    冲突策略，比如插入数据的时候如果发生冲突，是选择跳过还是回退等。

  + @PrimaryKey

  + @Query

  + @RawQuery

  + @Relation

  + @SkipQueryVerification

  + @Transaction

    那么里面的注解就会被当成事务提交，而不是等很多语句执行完，再来当做事务提交。

  + @TypeConverter

    类型转换。

  + @TypeConverters

  + @Update

    更新操作。

+ 预填充数据库

+ 迁移数据库

+ 测试和调试数据库

+ 引用复杂数据



### **数据库相关内容**

+ [廖雪峰-外键](https://www.liaoxuefeng.com/wiki/1177760294764384/1218728424164736)

