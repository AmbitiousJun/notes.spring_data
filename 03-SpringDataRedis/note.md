# Spring-Data-Redis

## 1. 入门案例

### 1.1 创建SpringBoot工程，引入相关依赖

```xml
<dependencies>
    <!--jedis-->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <!--移除掉lettuce依赖，使用jedis-->
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!--测试-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
    <!--json-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

### 1.2 配置Redis

- 在配置文件application.yml中配置连接信息

```yaml
spring:
  redis:
    connect-timeout: 20000  # 超时时间
    host: 192.168.40.130  # 主机地址
    port: 6379  # 端口号
    jedis:
      pool:
        max-idle: 20  # 最大空闲连接数
        min-idle: 10  # 最小空闲连接数
```

- 创建配置类，实例化一个RedisTemplate对象

```java
@Configuration
public class RedisConfig {

    // jedis连接工厂
    @Autowired
    private JedisConnectionFactory connectionFactory;

    /**
     * redis模板
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        return template;
    }
}
```

### 1.3 测试

```java
@SpringBootTest
public class RedisTest {

    @Autowired
    public RedisTemplate redisTemplate;

    @Test
    public void testSave() {
        ValueOperations operations = redisTemplate.opsForValue();
        operations.set("name", "Ambitious");
    }
}
```

运行测试之后可以看到，数据被成功存储到redis服务器中，但是数据全都变成二进制的形式

这是因为SpringDataRedis在将数据存储到Redis之前，会使用序列化器将数据序列化之后再进行存储

而默认使用的jdk序列化器就是将数据序列化成二进制的形式

![](./assets/入门案例-运行结果1.png)

### 1.4 设置序列化器

在实例化RestTemplate时可以设置序列化器

```java
@Bean
public RedisTemplate<String, Object> redisTemplate() {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new StringRedisSerializer());
    return template;
}
```

重新运行测试，再保存一条数据

```java
@Test
public void testSave() {
    ValueOperations operations = redisTemplate.opsForValue();
    operations.set("name1", "Ambitious1");
}
```

![](./assets/入门案例-运行结果2.png)

## 2. 操作String类型

使用**ValueOperations**接口的实现，完成String类型的操作

### 2.1 设置一个String类型的值

```java
void set(K key, V value);
```

```java
@Test
public void testSave() {
    operations.set("name1", "Ambitious1");
}
```

### 2.2 设置一个String类型的值，并指定有效时间

- 设定时间到达之后这个值会自动消失
- `long timeout`：过期时间（数值）
- `TimeUnit unit`：过期时间（单位）

```java
void set(K key, V value, long timeout, TimeUnit unit);
```

```java
@Test
public void testSave1() {
    // 设置数据10秒后过期
    operations.set("name2", "Ambitious2", 10, TimeUnit.SECONDS);
}
```

### 2.3 当一个key存在时，不执行操作；反之进行存储

```java
Boolean setIfAbsent(K key, V value);
```

```java
@Test
public void testSave2() {
    Boolean isSuccess = operations.setIfAbsent("name2", "ambitious2");
    System.out.println(isSuccess);
}
```

### 2.4 当一个key存在时，往其值末尾进行补充；反之执行保存操作

返回的整型值是执行完成之后，当前key对应的value有多少个字符

```java
Integer append(K key, String value);
```

### 2.5 一次性存储多个值

```java
void multiSet(Map<? extends K, ? extends V> map);
```

```java
@Test
public void testSave4() {
    Map<String, String> map = new HashMap<>();
    map.put("name4", "嘻嘻嘻");
    map.put("name5", "啊啊啊");
    map.put("name6", "呵呵呵");
    operations.multiSet(map);
}
```

### 2.6 String类型的其他操作

```java
@Test
public void testOther() {
    // 获取某个key对应的值
    String name = operations.get("name1");
    System.out.println(name);

    // 获取某个key的值，并截取子串，范围是闭区间[start, end]
    String name1 = operations.get("name1", 5, 7);
    System.out.println(name1);

    // 批量获取值
    List<String> keys = new ArrayList<>(Arrays.asList("name2", "name3", "name4"));
    List<String> values = operations.multiGet(keys);
    for (String value : values) {
        System.out.println(value);
    }

    // 自增
    operations.set("age", "18");
    // 自增1
    operations.increment("age");
    System.out.println(operations.get("age"));
    // 自定义自增大小
    operations.increment("age", 20);
    System.out.println(operations.get("age"));
    // 自减
    operations.decrement("age");
    System.out.println(operations.get("age"));

    // 删除
    redisTemplate.delete("age");
    System.out.println(operations.get("age"));
}
```

