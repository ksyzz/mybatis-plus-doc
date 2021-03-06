# 分页插件

- 自定义查询语句分页（自己写sql/mapper）
- spring 注入 mybatis 配置分页插件

```xml
<plugins>
    <!--
     | 分页插件配置
     | 插件提供二种方言选择：1、默认方言 2、自定义方言实现类，两者均未配置则抛出异常！
     | overflowCurrent 溢出总页数，设置第一页 默认false
     | optimizeType Count优化方式 （ 版本 2.0.9 改为使用 jsqlparser 不需要配置 ）
     | -->
    <!-- 注意!! 如果要支持二级缓存分页使用类 CachePaginationInterceptor 默认、建议如下！！ -->
    <plugin interceptor="com.baomidou.mybatisplus.plugins.PaginationInterceptor">
        <property name="sqlParser" ref="自定义解析类、可以没有" />
        <property name="localPage" value="默认 false 改为 true 开启了 pageHeper 支持、可以没有" />
        <property name="dialectClazz" value="自定义方言类、可以没有" />
    </plugin>
</plugins>
```

```java
//Spring boot方式
@EnableTransactionManagement
@Configuration
@MapperScan("com.baomidou.cloud.service.*.mapper*")
public class MybatisPlusConfig {

    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}

```

# 方式一 、传参区分模式【推荐】 

[测试用例](https://gitee.com/baomidou/mybatis-plus/blob/dev/mybatis-plus-core/src/test/java/com/baomidou/mybatisplus/test/h2/H2Pagination4SqlTest.java)

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
    // 不进行 count sql 优化，解决 MP 无法自动优化 SQL 问题
    // page.setOptimizeCountSql(false);
    // 不查询总记录数
    // page.setSearchCount(false);
    return page.setRecords(userMapper.selectUserList(page, state));
}
```

- UserMapper.xml 等同于编写一个普通 list 查询，mybatis-plus 自动替你分页

```xml
<select id="selectUserList" resultType="User">
    SELECT * FROM user WHERE state=#{state}
</select>
```

# 方式二、ThreadLocal 模式【容易出错，不推荐】

- PageHelper 使用方式如下：
```java
// 开启分页
PageHelper.startPage(1, 2);
List<User> data = userService.findAll(params);
// 获取总条数
int total = PageHelper.getTotal();
// 获取总条数，并释放资源
int total = PageHelper.freeTotal();
```

# JSON 序列化移除 transient 修饰的 Page 无关紧要的返回属性
```
// Jackson 方式
objectMapper.configure(MapperFeature.PROPAGATE_TRANSIENT_MARKER, true);
```

