# Java8函数接口



## 概述

函数式编程作为Java8的新特性，以其便捷易读的方式操作集合，使得代码易于维护，更加可靠和更不容易出错。

函数式编程是对行为进行抽象化，函数接口是只有一个抽象方法的接口，用作Lambda表达式的类型，使用不可变值和函数对一个值进行处理映射成另一个值。



## 函数接口

| 接口                 | 参数   | 返回类型 | 示例                 |
| -------------------- | ------ | -------- | -------------------- |
| Predicate<T>         | T      | boolean  | 这张唱片已经发行了吗 |
| Consumer<T>          | T      | void     | 输出一个值           |
| Function<T, R>       | T      | R        | 获得Artist对象的名字 |
| Supplier<T>          | None   | T        | 工厂方法             |
| UnaryOperator<T>     | T      | T        | 逻辑非               |
| BinaryOperator<T, T> | (T, T) | T        | 求两个数的乘积       |



## Predicate

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * 
     */
    boolean test(T t);

    /**
     * 
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * 
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * 
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * 
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

Predicate接口的核心是

```
boolean test(T t);
```

接收一个对象，返回一个布尔值。

**test(T t)**

判断给定的值是否大于0

```java
 Predicate<Integer> predicate = x -> x > 0;
 System.out.println(predicate.test(100)); // 结果为true
```

**and(Predicate<? super T> other)**

**or(Predicate<? super T> other)**

判断给定的值是否是大于100的偶数

```java
Predicate<Integer> predicate = x -> x > 100;
predicate = predicate.and(x -> x % 2 == 0 );
System.out.println(predicate.test(98)); // false
System.out.println(predicate.test(102)); // true
System.out.println(predicate.test(103)); // false

or(Predicate<? super T> other)同理，逻辑或
```

**negate()**

计算一批用户中年龄大于22岁的用户的数量

```java
Predicate<Person> personPredicate = x -> x.age > 22;
 System.out.println(
                Stream.of(new Person(21,"zhangsan"),new Person(22,"lisi"),new Person(23,"wangwu"),
                        new Person(24,"wangwu"),new Person(25,"lisi"),new Person(26,"zhangsan"))
                        .filter(personPredicate.negate())
                        .count()
        ); // 结果为4
```

**isEqual(Object targetRef)**

计算给定的一批用户中一样的用户

```java
Person person = new Person(22,"lisi");
Predicate<Person> predicate =  Predicate.isEqual(person);
System.out.println(
                Stream.of(new Person(21,"zhangsan"),new Person(22,"lisi"),new Person(23,"wangwu"),
                        new Person(24,"wangwu"),new Person(22,"lisi"),new Person(26,"zhangsan"))
                        .filter(predicate)
                        .count()
        ); // 结果为2
```

##### 与Predicate<T>相关的接口

- BiPredicate<T, U>

  针对两个参数,看两个参数是否符合某个条件表达式

- DoublePredicate

  看一个double类型的值是否符合某个条件表达式

- IntPredicate

  看一个int类型的值是否符合某个条件表达式

- LongPredicate

  看一个long类型的值是否符合某个条件表达式



#### Optional

| 方法名          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| get             | 获取Value的值，如果Value值是空值，则会抛出NoSuchElementException异常；因此返回的Value值无需再做空值判断，只要没有抛出异常，都会是非空值。 |
| isPresent       | Value是否为空值的判断；                                      |
| ifPresent       | 当Value不为空时，执行传入的Consumer；                        |
| ifPresentOrElse | Value不为空时，执行传入的Consumer；否则执行传入的Runnable对象； |
| filter          | 当Value为空或者传入的Predicate对象调用test(value)返回False时，返回Empty对象；否则返回当前的Optional对象 |
| map             | 一对一转换：当Value为空时返回Empty对象，否则返回传入的Function执行apply(value)后的结果组装的Optional对象； |
| flatMap         | 一对多转换：当Value为空时返回Empty对象，否则传入的Function执行apply(value)后返回的结果（其返回结果直接是Optional对象） |
| or              | 如果Value不为空，则返回当前的Optional对象；否则，返回传入的Supplier生成的Optional对象； |
| stream          | 如果Value为空，返回Stream对象的Empty值；否则返回Stream.of(value)的Stream对象； |
| orElse          | Value不为空则返回Value，否则返回传入的值；                   |
| orElseGet       | Value不为空则返回Value，否则返回传入的Supplier生成的值；     |
| orElseThrow     | Value不为空则返回Value，否则抛出Supplier中生成的异常对象；   |

