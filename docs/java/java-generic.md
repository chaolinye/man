# 泛型

## 通配符

泛型的集合无法向上转型。

```java
class Fruit {}

class Apple extends Fruit {}

class Jonathan extends Apple {}

class Orange extends Fruit {}

List<Fruit> flist = new ArrayList<Apple>(); // 编译报错
```

主要的问题点：Apple 的集合只能存储 Apple 及其子类（Jonathan），但是如果赋值给基类 Fruit 引用，那么就可能存入 Fruit 及其其它子类（Orange），导致非法。

值得一提，派生类的数组取可以赋值给基类的引用，编译会通过，但是如果存入非法的值会运行时报错。

```java
Fruit[] fruit = new Apple[10];
fruit[0] = new Apple(); // OK
fruit[1] = new Jonathan(); // OK
// Runtime type is Apple[], not Fruit[] or Orange[]:
try {
    // Compiler allows you to add Fruit:
    fruit[0] = new Fruit(); // ArrayStoreException
} catch (Exception e) {
    System.out.println(e);
}
try {
    // Compiler allows you to add Oranges:
    fruit[0] = new Orange(); // ArrayStoreException
} catch (Exception e) {
    System.out.println(e);
}
```

这是运行时的数组机制知道它处理的是 `Apple[]`，因此会在向数组中放置异构类型时抛出异常。

但是 Java 的泛型是擦除式泛型，运行时无法知道其类型，所以编译时直接不给赋值。

为了能够赋值，Java 引入了通配符

```java
// Wildcards allow covariance:
List<? extends Fruit> flist = new ArrayList<Apple>();
// Compile Error: can't add any type of object:
// flist.add(new Apple());
// flist.add(new Fruit());
// flist.add(new Object());
flist.add(null); // Legal but uninteresting
// We know it returns at least Fruit:
Fruit f = flist.get(0);
```

`? extends Fruit` 表示这是 Fruit 或者其某个派生类的集合，但由于不知道其具体类型，编译器会限制这种集合**不能存入数据，只能读取数据**。

为了能够存入数据，又引入了 `? super` 用法

```java
void writeTo(List<? super Apple> apples) {
    apples.add(new Apple());
    apples.add(new Jonathan());
    // apples.add(new Fruit()); // Error
}
```

`? super Apple` 表示这是 Apple 或者其某个父类的集合，这是存入 Apple 及其子类一定是合法，但是由于不知道其具体类型，也就不知道用什么类型的引用读取数据。因此编译器会限制这种集合**只能存入数据，不能读取数据**。