# 嵌套泛型反序列化

对于嵌套泛型的反序列化，常见的 JSON 序列化库都提供了 `TypeReference` 的方案

下面是个 Jackson 反序列化嵌套泛型的例子

```java
public class NestedGenericTests {
    private String userResponseString;
    private ObjectMapper objectMapper;

    @BeforeEach
    public void init() throws JsonProcessingException {
        objectMapper = new ObjectMapper();

        User user = new User(3L, "zhangsan");
        Response<List<User>> userResponse = new Response<>(Collections.singletonList(user));
        userResponseString = objectMapper.writeValueAsString(userResponse);
    }

    @Test
    public void testCommonClass() throws Exception {
        Response<List<User>> userResponse = objectMapper.readValue(userResponseString, Response.class);
        // 由于泛型擦除机制，运行时无法获取到 Response 的泛型，无法正确反序列化
        Assertions.assertThrows(ClassCastException.class, () -> {
            userResponse.getData().get(0).getName();
        });
    }

    @Test
    public void testTypeReference() throws Exception {
        // 这里构建了一个内部类，编译时会生成一个内部类 class 文件，class 文件中会在 Signature 中记录具体的泛型类型
        Response<List<User>> userResponse = objectMapper.readValue(userResponseString, new TypeReference<Response<List<User>>>() {
        });
        Assertions.assertEquals("zhangsan", userResponse.getData().get(0).getName());
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    static class User {
        private Long id;
        private String name;
    }

    @Getter
    @Setter
    static class Response<T> {
        private int retCode;
        private String errorMsg;
        private T data;

        public Response() {}

        public Response(T data) {
            this.data = data;
            this.retCode = 200;
        }

    }
}
```

`TypeReference` 的实现如下

```java
public abstract class TypeReference<T> implements Comparable<TypeReference<T>>
{
    protected final Type _type;
    
    protected TypeReference()
    {
        Type superClass = getClass().getGenericSuperclass();
        if (superClass instanceof Class<?>) { // sanity check, should never happen
            throw new IllegalArgumentException("Internal error: TypeReference constructed without actual type information");
        }
        /* 22-Dec-2008, tatu: Not sure if this case is safe -- I suspect
         *   it is possible to make it fail?
         *   But let's deal with specific
         *   case when we know an actual use case, and thereby suitable
         *   workarounds for valid case(s) and/or error to throw
         *   on invalid one(s).
         */
        _type = ((ParameterizedType) superClass).getActualTypeArguments()[0];
    }

    public Type getType() { return _type; }
}
```

通过 `Class.getGenericSuperclass().getActualTypeArguments()` 可以获取父类泛型的具体类型

```java
    class SubClass extends ArrayList<String> {}

    @Test
    public void testGenericSuperClass() {
        Assertions.assertEquals(String.class,
                ((ParameterizedType) SubClass.class.getGenericSuperclass()).getActualTypeArguments()[0]);
    }
}
```

## 泛型 Signature

反编译前面编译的 `SubClass.class` 文件

`javap -v -l -c 'NestedGenericTests$SubClass.class'`

```java
Classfile /Users/chaolinye/Project/opensource/springmvc-demo/target/test-classes/org/leaf/NestedGenericTests$SubClass.class
  Last modified 2020-12-26; size 596 bytes
  MD5 checksum 174f22e29efafcb919000f619e902341
  Compiled from "NestedGenericTests.java"
class org.leaf.NestedGenericTests$SubClass extends java.util.ArrayList<java.lang.String>
  minor version: 0
  major version: 52
  flags: ACC_SUPER
Constant pool:
   #1 = Fieldref           #3.#21         // org/leaf/NestedGenericTests$SubClass.this$0:Lorg/leaf/NestedGenericTests;
   #2 = Methodref          #4.#22         // java/util/ArrayList."<init>":()V
   #3 = Class              #24            // org/leaf/NestedGenericTests$SubClass
   #4 = Class              #25            // java/util/ArrayList
   #5 = Utf8               this$0
   #6 = Utf8               Lorg/leaf/NestedGenericTests;
   #7 = Utf8               <init>
   #8 = Utf8               (Lorg/leaf/NestedGenericTests;)V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               SubClass
  #14 = Utf8               InnerClasses
  #15 = Utf8               Lorg/leaf/NestedGenericTests$SubClass;
  #16 = Utf8               MethodParameters
  #17 = Utf8               Signature
  #18 = Utf8               Ljava/util/ArrayList<Ljava/lang/String;>;
  #19 = Utf8               SourceFile
  #20 = Utf8               NestedGenericTests.java
  #21 = NameAndType        #5:#6          // this$0:Lorg/leaf/NestedGenericTests;
  #22 = NameAndType        #7:#26         // "<init>":()V
  #23 = Class              #27            // org/leaf/NestedGenericTests
  #24 = Utf8               org/leaf/NestedGenericTests$SubClass
  #25 = Utf8               java/util/ArrayList
  #26 = Utf8               ()V
  #27 = Utf8               org/leaf/NestedGenericTests
{
  final org.leaf.NestedGenericTests this$0;
    descriptor: Lorg/leaf/NestedGenericTests;
    flags: ACC_FINAL, ACC_SYNTHETIC

  org.leaf.NestedGenericTests$SubClass(org.leaf.NestedGenericTests);
    descriptor: (Lorg/leaf/NestedGenericTests;)V
    flags:
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #1                  // Field this$0:Lorg/leaf/NestedGenericTests;
         5: aload_0
         6: invokespecial #2                  // Method java/util/ArrayList."<init>":()V
         9: return
      LineNumberTable:
        line 75: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      10     0  this   Lorg/leaf/NestedGenericTests$SubClass;
            0      10     1 this$0   Lorg/leaf/NestedGenericTests;
    MethodParameters:
      Name                           Flags
      this$0                         final mandated
}
Signature: #18                          // Ljava/util/ArrayList<Ljava/lang/String;>;
SourceFile: "NestedGenericTests.java"
InnerClasses:
     #13= #3 of #23; //SubClass=class org/leaf/NestedGenericTests$SubClass of class org/leaf/NestedGenericTests
```

可见，class 文件在 Signature 中记录了泛型父类的具体泛型类型，说明运行时确实可以获取到泛型类型

## 总结

!> Java 的泛型擦除针对方法体内的泛型对象

!> 具体泛型类型的类的方法，可以在运行时获取到具体的泛型类型

## 学习资料

- [fastJson反序列化处理泛型 我能从中学到什么](https://juejin.cn/post/6844903908184162312)
- [json 反序列化多层嵌套泛型类与 java 中的Type类型笔记](https://cloud.tencent.com/developer/article/1460768)