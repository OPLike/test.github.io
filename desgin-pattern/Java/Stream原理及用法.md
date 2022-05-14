# Stream原理及用法





## Stream操作分类

|  |      |      |
| -------- | ---------- | ---- |
| 中间操作 | 无状态     | unordered() filter() map() mapToInt() mapToLong() mapToDouble() flatMap() flatMapToInt() flatMapToLong() flatMapToDouble() peek() |
| 中间操作 | 有状态     | distinct() sorted() limit() skip() |
| 结束操作 | 非短路操作 | forEach() forEachOrdered() toArray() reduce() collect() max() min() count() |
| 结束操作 | 短路操作   | anyMatch() allMatch() noneMatch() findFirst() findAny() |

## 中间操作

| 方法       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| map        | 通过一个 Function 把一个元素类型为 T 的流转换成元素类型为 R 的流。包括mapToInt、mapToLong、mapToDouble |
| flatMap    | 通过一个 Function 把一个元素类型为 T 的流中的每个元素转换成一个元素类型为 R 的流，再把这些转换之后的流合并。 |
| filter     | 过滤流中的元素，只保留满足由 Predicate 所指定的条件的元素。  |
| distinct   | 使用 equals 方法来删除流中的重复元素。                       |
| limit      | 截断流使其最多只包含指定数量的元素。                         |
| skip       | 返回一个新的流，并跳过原始流中的前 N 个元素。                |
| sorted     | 对流进行排序。                                               |
| peek       | 返回的流与原始流相同。当原始流中的元素被消费时，会首先调用 peek 方法中指定的 Consumer 实现对元素进行处理。 |
| dropWhile  | 从原始流起始位置开始删除满足指定 Predicate 的元素，直到遇到第一个不满足 Predicate 的元素。 |
| takeWhile  | 从原始流起始位置开始保留满足指定 Predicate 的元素，直到遇到第一个不满足 Predicate 的元素。 |
| sequential | 返回一个相等的串行的Stream对象，如果原Stream对象已经是串行就可能会返回原对象 |
| parallel   | 返回一个相等的并行的Stream对象，如果原Stream对象已经是并行的就会返回原对象 |
| unordered  | 返回一个不关心顺序的Stream对象，如果原对象已经是这类型的对象就会返回原对象 |
| onClose    | 返回一个相等的Steam对象，同时新的Stream对象在执行Close方法时会调用传入的Runnable对象 |
| close      | 关闭Stream对象                                               |

## 终端操作

| 方法           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| iterator       | 返回Stream中所有对象的迭代器;                                |
| spliterator    | 返回对所有对象进行的spliterator对象                          |
| forEach        | 对所有元素进行迭代处理，无返回值                             |
| forEachOrdered | 按Stream的Encounter所决定的序列进行迭代处理，无返回值        |
| toArray        | 返回所有元素的数组                                           |
| reduce         | 使用一个初始化的值，与Stream中的元素一一做传入的二合运算后返回最终的值。每与一个元素做运算后的结果，再与下一个元素做运算。它不保证会按序列执行整个过程。 |
| collect        | 根据传入参数做相关汇聚计算                                   |
| min            | 返回所有元素中最小值的Optional对象；如果Stream中无任何元素，那么返回的Optional对象为Empty |
| max            | 与Min相反                                                    |
| count          | 所有元素个数                                                 |
| anyMatch       | 只要其中有一个元素满足传入的Predicate时返回True，否则返回False |
| allMatch       | 所有元素均满足传入的Predicate时返回True，否则False           |
| noneMatch      | 所有元素均不满足传入的Predicate时返回True，否则False         |
| findFirst      | 返回第一个元素的Optioanl对象；如果无元素返回的是空的Optional； 如果Stream是无序的，那么任何元素都可能被返回。 |
| findAny        | 返回任意一个元素的Optional对象，如果无元素返回的是空的Optioanl。 |
| isParallel     | 判断是否当前Stream对象是并行的                               |



## Stream高级用法

#### 1.BigDecimal属性求和、差、最大最小值

```java
// 和
BigDecimal sumValue = statResultList.stream().map(StatResult::getSumValue).reduce(BigDecimal.ZERO, BigDecimal::add);
// 差
BigDecimal diffValue = statResultList.stream().map(StatResult::getDiffValue).reduce(BigDecimal.ZERO, BigDecimal::subtract);
// 最大值
BigDecimal maxValue = statResultList.stream().map(StatResult::getMaxValue).reduce(BigDecimal.ZERO, BigDecimal::max);
// 最小值
BigDecimal minValue = statResultList.stream().map(StatResult::getMinValue).reduce(maxValue, BigDecimal::min);

map是一个对于流对象的中间操作，通过给定的方法，它能够把流对象中的每一个元素对应到另外一个对象上
reduce是一个终结操作，它能够通过某一个方法，对元素进行削减操作。该操作的结果会放在一个Optional变量里返回。可以利用它来实现很多聚合方法比如count,max,min等。
这里利用了reduce的第二个方法重载T reduce(T identity, BinaryOperator accumulator);
第一个参数是我们给出的初值，第二个参数是累加器，可以自己用实现接口完成想要的操作，这里使用Bigdecimal的add方法
最后reduce会返回计算后的结果
```



#### 2.int属性的和、最大值、最小值、平均数、总个数

```java
// 和
int total = list.stream().mapToInt(User::getAge).sum();
int ageSum = userList.stream().collect(Collectors.summingInt(User::getAge));
// 最大值
int intmax = listUsers.stream().mapToInt(Users::getAge).max().getAsInt();
// 最小值
int intmin = listUsers.stream().mapToInt(Users::getAge).min().getAsInt();
// 平均值
double avg = listUsers.stream().mapToDouble(Users::getAge).average().getAsDouble();
```



#### 3.包含多个列表的流

假设有一个包含多个列表的流，想要得到所有列表的序列

```java
// 将每个列表转换成Stream对象，其余部分由flatMap方法处理。
// flatMap方法的相关函数接口和map的方法一样，都是Function接口，只是返回值为Stream类型。
List<Integer> list = Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4)).flatMap(Collection::stream).collect(Collectors.toList());
等价于
List<Integer> list = Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4)).flatMap(n -> n.stream()).collect(Collectors.toList());

```



#### 4.获取最大最小的对象流

```java
List<Track> tracks = asList(new Track("Bakai", 524), new Track("Violets for Your Furs", 378), new Track("Time Was", 451));
Track shortestTrack = tracks.stream().min(Comparator.comparing(track -> track.getLength())).get(); 

查找Stream中的最大最小元素，首先要考虑排序指标（示例中track的length属性）
```



#### 5.根据对象某一字段进行去重、排序、分组

```java
// 根据Student对象的age进行去重
studentList = studentList.stream().collect(Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(Student :: getAge))), ArrayList::new));

// 将对象 List 根据的某一字段过滤、排序
List<String> newList = objectList.stream().filter(object -> "Value".equals(object.getVar()))
                                          .sorted(Comparator.comparing(Object::getVar))
                                          .collect(Collectors.toList());

// 将对象 List 根据的某一字段分组
Map<String,List<Object>> objectMap = objectList.stream().collect(Collectors.groupingBy(Object::getVar));
```













