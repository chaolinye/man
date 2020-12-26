# 反射

## 解析类和 Field 上的 Annotation

定义两个 Annotation

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Element {
    String value();
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Property {
    String value() default "";
}
```

使用这两个 Annotation 装饰某些类

```java
@Element("title")
public class Title {
    @Property(value = "")
    private String value;
}

```

获取被 `@Element` 装饰的类上 `@Element` 和 `@Property` 的值

> 主要使用了 Spring 的两个工具类: 
> - `ClassPathScanningCandidateComponentProvider`: 获取类路径下被某些 Annotation 装饰的类，通过可获取这些 Annotation 的值
> - `AnnotatedElementUtils`: 获取 Annotation 的值

```java
public class ElementMetadataReader {
    public List<Map<String, Object>> getElementMetadataList() {
        ClassPathScanningCandidateComponentProvider provider = new ClassPathScanningCandidateComponentProvider(false);
        provider.addIncludeFilter(new AnnotationTypeFilter(Element.class));
        Set<BeanDefinition> candidateComponents = provider.findCandidateComponents("org.leaf.demo.metadata.element");
        return candidateComponents.stream().map(this::extractElementMetadata).collect(Collectors.toList());
    }

    private Map<String, Object> extractElementMetadata(BeanDefinition comp) {
        if (!(comp instanceof ScannedGenericBeanDefinition)) {
            throw new RuntimeException("extract element metadata error");
        }
        ScannedGenericBeanDefinition scannedComp = (ScannedGenericBeanDefinition) comp;
        Map<String, Object> result = scannedComp.getMetadata().getAnnotationAttributes(Element.class.getName());
        result.put("properties", extractElementProperties(scannedComp));
        return result;
    }

    private List<Map<String, Object>> extractElementProperties(ScannedGenericBeanDefinition comp) {
        List<Map<String, Object>> properties = new ArrayList<>();
        Class<?> aClass = null;
        try {
            aClass = Class.forName(comp.getBeanClassName());
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }

        for (Field field : aClass.getDeclaredFields()) {
            Map<String, Object> property = new HashMap<>();
            properties.add(property);
            property.put("name", field.getName());
            AnnotationAttributes mergedAnnotationAttributes = AnnotatedElementUtils.getMergedAnnotationAttributes(field, Property.class);
            if (mergedAnnotationAttributes == null) {
                continue;
            }
            property.putAll(mergedAnnotationAttributes);
        }
        return properties;
    }
}
```