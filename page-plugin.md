# 分页插件

- 自定义查询语句分页（自己写sql/mapper）
- spring 注入 mybatis 配置分页插件

```xml
<plugins>
    <!--
     | 分页插件配置
     | 插件提供二种方言选择：1、默认方言 2、自定义方言实现类，两者均未配置则抛出异常！
     | overflowCurrent 溢出总页数，设置第一页 默认false
     | optimizeType Count优化方式 默认default
     | 1.支持 aliDruid 方式，需添加aliDruid依赖
     | 2.支持 jsqlparser 方式，需添加jsqlparser依赖
     | dialectType 数据库方言
     |             默认支持  mysql  oracle  hsql  sqlite  postgre  sqlserver
     | dialectClazz 方言实现类
     |              自定义需要实现 com.baomidou.mybatisplus.plugins.pagination.IDialect 接口
     | -->
    <!-- 注意!! 如果要支持二级缓存分页使用类 CachePaginationInterceptor 默认、建议如下！！ -->
    <!-- 配置方式一、使用 MybatisPlus 提供方言实现类 -->
    <plugin interceptor="com.baomidou.mybatisplus.plugins.PaginationInterceptor">
        <property name="dialectType" value="mysql" />
        <property name="optimizeType" value="aliDruid" />
    </plugin>
    <!-- 配置方式二、使用自定义方言实现类 -->
    <plugin interceptor="com.baomidou.mybatisplus.plugins.PaginationInterceptor">
        <property name="dialectClazz" value="xxx.dialect.XXDialect" />
        <property name="optimizeType" value="jsqlparser" />
    </plugin>
</plugins>
```

```java
//Spring boot方式
@Bean
public SqlSessionFactory sqlSessionFactory(){
  MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
  ...
  PaginationInterceptor pagination = new PaginationInterceptor();
  pagination.setOptimizeType("jsqlparser");
  sqlSessionFactory.setPlugins(new Interceptor[]{
      pagination
    });
  return sqlSessionFactory.getObject();
}
```

- UserMapper.java 方法内容

```java
public interface UserMapper{//可以继承或者不继承BaseMapper
    /**
     * <p>
     * 查询 : 根据state状态查询用户列表，分页显示
     * </p>
     *
     * @param page
     *            翻页对象，可以作为 xml 参数直接使用，传递参数 Page 即自动分页
     * @param state
     *            状态
     * @return
     */
    List<User> selectUserList(Pagination page, Integer state);
}
```

- UserServiceImpl.java 调用翻页方法，需要 page.setRecords 回传给页面

```java
public Page<User> selectUserPage(Page<User> page, Integer state) {
    page.setRecords(userMapper.selectUserList(page, state));
    return page;
}
```

- UserMapper.xml 等同于编写一个普通 list 查询，mybatis-plus 自动替你分页

```xml
<select id="selectUserList" resultType="User">
    SELECT * FROM user WHERE state=#{state}
</select>
```
