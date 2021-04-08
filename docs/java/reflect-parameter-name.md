# Java 获取参数名称

JDK8 版本 javac 新增了 `-parameters` 参数用于把参数名记录到 class 文件中

> -parameters 参数默认关闭

```java
public class GetRuntimeParameterName {

    public void createUser(String name, int age, int version) {
        //
    }

    public static void main(String[] args) throws Exception {
        for (Method m : GetRuntimeParameterName.class.getMethods()) {
            System.out.println("----------------------------------------");
            System.out.println("   method: " + m.getName());
            System.out.println("   return: " + m.getReturnType().getName());
            for (Parameter p : m.getParameters()) {
                System.out.println("parameter: " + p.getType().getName() + ", " + p.getName());
            }
        }
    }
}
```

spring-boot 的 `spring-boot-starter-parent` 的 `pom.xml` 中的 `pluginManagement` 将 `maven-compiler-plugin` 的 `parameters` 设置为 `true`

这样 spring-boot 工程的 parameters 默认就是 true 了

```xml
    <pluginManagement>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <configuration>
            <parameters>true</parameters>
          </configuration>
        </plugin>
    </pluginManagement>
```

开启 `parameters` 后， `mybatis` 中也能正确获取参数名称，这样子 `Mapper` 中就可以减少 `@Param` 注解的编写了

Spring 中除了利用了 JDK8 新增的 parameters 参数，还提供了一个方法，使用 ASM 通过 class 字节码文件中的 LocalVariableTable 获取参数名称

```java
public class DefaultParameterNameDiscoverer extends PrioritizedParameterNameDiscoverer {

	public DefaultParameterNameDiscoverer() {
		// TODO Remove this conditional inclusion when upgrading to Kotlin 1.5, see https://youtrack.jetbrains.com/issue/KT-44594
		if (KotlinDetector.isKotlinReflectPresent() && !NativeDetector.inNativeImage()) {
			addDiscoverer(new KotlinReflectionParameterNameDiscoverer());
		}
        // JDK8 新增的 parameters 参数
		addDiscoverer(new StandardReflectionParameterNameDiscoverer());
        // ASM 获取 LocalVariableTable 参数
		addDiscoverer(new LocalVariableTableParameterNameDiscoverer());
	}

}
```