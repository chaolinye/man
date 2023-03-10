# Java Debug

## 查看某个类来自于哪个 class 文件路径

常用于排查类加载问题。

```java
System.out.println(Xxx.class.getProtectionDomain().getCodeSource().getLocation().toString());
```