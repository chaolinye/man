# 深拷贝

Java 深拷贝主要有三种方法

- 自定义 `clone` 方法

- Java 序列化和反序列化，可以使用 `apache-common` 的 `SerializationUtils#clone` 工具方法

- JSON 序列化和反序列化

!> 根据测试，在数据较大时，JSON 序列化和反序列化性能更好

```java
public class DeepCopyTests {

    ObjectMapper objectMapper;

    User user;

    @BeforeEach
    public void initData() {
        objectMapper = new ObjectMapper();
        Address address = new Address("guangdong", "shenzhen");
        user = new User(1, "zhangsan", address);
    }

    @Test
    public void testSerializationUtils() {
        StopWatch sw = new StopWatch("testSerializationUtils");
        sw.start();
        User userClone = (User) SerializationUtils.deserialize(SerializationUtils.serialize(user));
        sw.stop();
        System.out.println(sw.prettyPrint());
        Assertions.assertEquals(userClone, user);
    }

    @Test
    public void testJackson() throws JsonProcessingException {
        StopWatch sw = new StopWatch("testJackson");
        sw.start();
        User userClone = objectMapper.readValue(objectMapper.writeValueAsString(user), User.class);
        sw.stop();
        System.out.println(sw.prettyPrint());
        Assertions.assertEquals(userClone, user);
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class User implements Serializable {
        private Integer id;
        private String name;
        private Address address;
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Address implements Serializable {
        private String province;
        private String city;
    }
}
```

## References

- [How to Make a Deep Copy of an Object in Java](https://www.baeldung.com/java-deep-copy)