# TIL(2017-11-09) Java Excel, Guava-1

## 用 Apache POI 写 Excel

#### 文档

- https://poi.apache.org/

#### 定义

- HSSF 开头的是 Excel 2003 相关的
- XSSF 相关的是 Excel 2007 相关的
- WorkBook 对应一个 Excel 文档
- Sheet 对应一张表
- Row 对应行
- Cell 对应一格

#### Maven

```xml
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi-ooxml</artifactId>
  <version>3.15</version>
</dependency>
```

*poi-ooxml 是 XSSF 的包（2007），poi 是 HSSF 的包（2003）*

```java

public class ExcelTest {

    private static final String FILE_NAME = "MyFirstExcel.xls";
    private static final List<String> FIELDS = Arrays.asList("a", "b", "c", "d", "e");
    private Random random = new Random();

    @Test
    public void writeExcel() {
        List<Map<String, String>> lines = Lists.newArrayList();
        for (int i = 0; i < 100; i++) {
            lines.add(genContent());
        }

        HSSFWorkbook workbook = new HSSFWorkbook();
        HSSFSheet sheet = workbook.createSheet();

        int rowNum = 0;
        int colNum = 0;
        Row row = sheet.createRow(rowNum++);
        for (String field : FIELDS) {
            Cell cell = row.createCell(colNum++);
            cell.setCellValue(field);
        }
      
        for (Map<String, String> line : lines) {
            row = sheet.createRow(rowNum++);
            colNum = 0;
            for (String field : FIELDS) {
                Cell cell = row.createCell(colNum++);
                cell.setCellValue(line.get(field));
            }
        }

        try {
            FileOutputStream outputStream = new FileOutputStream(FILE_NAME);
            workbook.write(outputStream);
            workbook.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    private Map<String, String> genContent() {
        Map<String, String> line = Maps.newHashMap();
        FIELDS.forEach(field -> line.put(field, String.valueOf(random.nextLong())));
        return line;
    }
}
```



## Guava 学习-1

Google 的 Guava 是很强大的 Java 工具库。

> collections, caching, primitives support, concurrency libraries, common annotations, string processing, I/O, and so forth.

#### 文档

- github: https://github.com/google/guava
- User Guide: https://github.com/google/guava/wiki

#### Guava Optional 与 Java Optional

参考 [Optional, Guava and Java 8](http://onelineatatime.io/optional-guava-and-java-8/)。

简单说：

- 两个 Optional 都是表示可能为 null，Java 的 Optioanl 是 Java 8 引入的
- Guava 的 Optional 是抽象类，有 Absent 和 Present 的实现；Java 的则是一个 final class。
- `Absent` 只有一个 JVM 只有一个（单例），表示什么都没有，`Present` 则是有内容。
- Optional 的 isPresent, get 和 Java 的一样，Optional 的 or 对应 Java 8 的 orElseGet， transform 对应 map，不过  jdk 里还有 filter 和 flatMap。
- JDK 的 Optional 不是 Serializable，可能是 JDK 大佬不希望我们用 Optional 在 RMI 中。

Optional 提高可读性，我个人比较倾向于用 JDK 的，毕竟原生。只在 JDK < 8 时再用 Guava。

#### Precondton：类似 assert，一套封装的条件校验异常机制

具体的一些方法见 [文档](https://github.com/google/guava/wiki/PreconditionsExplained)

#### 通用 Object 方法

 各种高版本 JDK 语法的替代方案。

写 compareTo 时，可以用 Guava 的 ComparisonChain 来搞，

```java
   public int compareTo(Foo that) {
     return ComparisonChain.start()
         .compare(this.aString, that.aString)
         .compare(this.anInt, that.anInt)
         .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
         .result();
   }
```

不过如果是 JDK8，就直接用 `java.util.Collections#sort(java.util.List<T>, java.util.Comparator<? super T>)` 就行了。

```java
    public void test1() {
        List<String> strings = Arrays.asList("a", "aaa", "aba", "b");
        Collections.sort(strings, Comparator.comparing(String::length));
        Collections.sort(strings, Comparator.comparing(String::length).reversed());
        Collections.sort(strings, Comparator.comparing(String::length, (a, b) -> b - a));
        Collections.sort(strings, Comparator.comparing(a -> a, String.CASE_INSENSITIVE_ORDER));
        Collections.sort(strings, Comparator.naturalOrder());
        Collections.sort(strings, Comparator.reverseOrder());
    }
```

#### Multi- Collection

**Multiset**: 可以放多个 value 的 set，其实主要是用来 count 集合中有几个 value

**Multimap**: 可以放多个 value 的 map，当成 `Map<T, Collection<T>>` 就好了，虽然里面的方法有点不一样。

**BiMap**: 双向 Map，kv 需要一对一否则会报错。

**Table**: 看成二维的表。 两层的 Map，如 `Map<A, Map<B, C>>`，不过可以取一行或一列。

**RangeSet**: 以区间为对象进行一层封装，可以把各种区间塞进去，然后看看一个数是否符合区间。

**RangeMap**: 类是 RangeSet，不过在区间上会有值，对区间的操作是会更新值的。

#### Immutable Collections

和 JDK 的 `Collections.unmodifiableXXX` 差不多，不过他们说 Guava 的 ImmutableXXX 会更轻便、安全、高效。

#### Collection Utilities

Collection 工具是日常中用得比较多的工具，很实用。

JDK 有自带的 Collections 工具，Guava 更是扩展了它们。 *（Apache common 里也有很不错的工具）*

**常用工具**: Lists, Sets, Maps,

**Iterables**: Guava 除了对常见容器提供工具类，还对 Iterable 提供了工具类，叫 Iteratles

##### 有意思的摘要：

**Maps.uniqueIndex**:  一个 Iterable，用里面每个元素的一个唯一属性作为 key 转成 Map。（不是唯一属性会报错），返回的是一个 ImmutableMap。（用 stream 也能实现这个效果，不过 stream 的 toMap 返回的是可变 map）

```java
    @Test
    public void collectionTest() {
        List<String> strings = Lists.newArrayList("a", "bb", "ccc");
        ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings, String::length);
        System.out.println(stringsByIndex); // {1=a, 2=bb, 3=ccc}
      
        Map<Integer, String> collect = strings.stream().collect(Collectors.toMap(String::length, i -> i));
        collect.put(4, "aaaa");
        System.out.println(collect);
    }
```

**Multimaps.index**: 和 `Maps.uniqueIndex` 类似，不过是允许属性重复，它会把属性重复的放到一个 List 中。同样可以用 stream 实现。

```java
    @Test
    public void collectionTest() {
        ImmutableSet<String> digits = ImmutableSet.of(
                "zero", "one", "two", "three", "four",
                "five", "six", "seven", "eight", "nine");
        ImmutableListMultimap<Integer, String> digitsByLength = Multimaps.index(digits, String::length);
        System.out.println(digitsByLength);
        /*
         * digitsByLength maps:
         *  3 => {"one", "two", "six"}
         *  4 => {"zero", "four", "five", "nine"}
         *  5 => {"three", "seven", "eight"}
         */

        Map<Integer, List<String>> collect1 = digits.stream().collect(Collectors.groupingBy(String::length));
        System.out.println(collect1);
    }
```

虽然 Guava 提供了一些有用的工具类，不过在 JDK 7, 8 的一些特性和增强下，也显得不是那么好用，有不少方法都可以用 Stream 来更灵活的实现。

不过 Guava 的一些 Immutable 的容器用来做 final 变量还不错，构建语法也很友好，下次研究看看 Guava Immutable 和 JDK 的 Unmodify 容器到底有什么区别。

#### Graphs 图类

Guava 支持一些图的构建和操作，支持有向、无向、带权、网络等各种图，需要用到图时，用这个会很方便。

支持 **equals** 操作，比较图是否相同。

百万节点以上的图不适用。

很遗憾，google 并没有多少用 Graph 做最短路算法之类的资料，感觉这个类比较冷，毕竟是 Guava 20 版本才出的。而已看他们的 mail list 的讨论，Graph 的定位是只是实现一个图 API，算法什么的不会支持太多，如果需要便捷的算法，可以用 JUNG 或 JGraphT。











