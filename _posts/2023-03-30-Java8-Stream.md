---
layout: post
title: "crack Java 8 Stream"
date: 2023-03-30
description: "Java Stream knowledge"
tag: Java
---

## Stream

<img src="/images/Full-Stack/JavaCore/StreamApi.jpg">

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35fa8057a9ba45ee8c627fb29deeaee5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?">

Stream is used to process a series of operations on a collection in a pipeline. The pipeline consists of a source, zero or more intermediate operations, and a terminal operation. Stream is a one-time object that cannot be reused after the terminal operation is executed.

### Stream Creation

Create Stream using `java.util.Collection.stream()`

```Java
List<String> list = Arrays.asList("a", "b", "c");
// create a stream
Stream<String> stream = list.stream();
// creae a parallel stream
Stream<String> parallelStream = list.parallelStream();
```

### Exercising Code

```Java
class Person {
    private String name; // 姓名
    private int salary; // 薪资
    private String sex; //性别
    private String area; // 地区

    // 构造方法
    public Person(String name, int salary,String sex,String area) {
        this.name = name;
        this.salary = salary;
        this.sex = sex;
        this.area = area;
    }
    // 省略了get和set，请自行添加


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getSalary() {
        return salary;
    }

    public void setSalary(int salary) {
        this.salary = salary;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getArea() {
        return area;
    }

    public void setArea(String area) {
        this.area = area;
    }

    @Override
    public String toString() {
        return this.name + " " +this.salary + " " + this.sex + " " + this.area;
    }
}

    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, "male", "New York"));
    personList.add(new Person("Jack", 7000, "male", "Washington"));
    personList.add(new Person("Lily", 7800, "female", "Washington"));
    personList.add(new Person("Anni", 8200, "female", "New York"));
    personList.add(new Person("Owen", 9500, "male", "New York"));
    personList.add(new Person("Alisa", 7900, "female", "New York"));
```

### traverse and match (foreach/find/match)

Stream support traverse and match operations. elements are `Optional` inside the stream.

<img src="https://pic1.zhimg.com/80/v2-906e6e20b69232c86a1e7d276115d530_720w.webp">

```Java
public class StreamTest {
  public static void main(String[] args) {
        List<Integer> list = Arrays.asList(7, 6, 9, 3, 8, 2, 1);

        // traverse and print
        list.stream().filter(x -> x > 6).forEach(System.out::println);
        // match the first element
        Optional<Integer> findFirst = list.stream().filter(x -> x > 6).findFirst();
        // match any element
        Optional<Integer> findAny = list.parallelStream().filter(x -> x > 6).findAny();
        // whether contains element larger than 6
        boolean anyMatch = list.stream().anyMatch(x -> x < 6);
        System.out.println("match the first element: " + findFirst.get());
        System.out.println("match any element: " + findAny.get());
        System.out.println("whether contains element larger than 6：" + anyMatch);
    }
}
```

- `forEach`：Traverse the stream and execute the specified operation for each element in the stream.
- `findFirst`：`(Optional)` Find the first element in the stream.
- `findAny`：`(Optional)` Find any element in the stream. (randomly selected)
- `anyMatch`：`(Boolean value)` Whether any element in the stream matches the specified predicate, if any, returns true.

### filter (filter)

<img src="https://pic3.zhimg.com/80/v2-55190a2c00db4516541e20069012f4e6_720w.webp">

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Integer> list = Arrays.asList(6, 7, 3, 8, 1, 2, 9);
    Stream<Integer> stream = list.stream();
    stream.filter(x -> x > 7).forEach(System.out::println);
  }
}
```

- `filter`：(accept Predicate) Filter out the elements that meet the specified conditions in the stream and return a new stream.
- `distinct`：remove duplicate elements in the stream and return a new stream.
- `limit`：`limit(2)` (keep the first two element, just like SQL TOP)
- `skip`：`skip(2)` (skip the first two element)

Filter the person whose salary is greater than 8000, return the name of the person, and save it in the list.

```Java
List<String> newPersonList = personList.stream()
    .filter(person -> person.getSalary() > 8000)
    .map(Person::getName)
    .collect(Collectors.toList());

// output : [Tom, Anni, Owen]
```

### Aggregate (count/max/min)

just like SQL, aggregate functions are used to calculate the sum, average, maximum, minimum, etc. of the elements in the stream.

<img src="https://pic2.zhimg.com/80/v2-0c74341dcc479827c37bbc376758ef35_720w.webp">

1. get the longest string in the list

```Java

public class StreamTest {
  public static void main(String[] args) {
    List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");

    Optional<String> max = list.stream().max(Comparator.comparing(String::length));

    // or in this way
    Optional<String> max = list.stream().max((o1, o2) -> o1.length() - o2.length());
    // output : weoujgsd
    // [pot, adnm, admmt, xbangd, weoujgsd] it will sort the list in ascending order and get the last element
    // for min, it will get the first element
  }
}
```

2. get the maximum int of the list

```Java
    List<Integer> list = Arrays.asList(7, 6, 9, 4, 11, 6);
    Optional<Integer> max = list.stream().max(Integer::compareTo);
    // output : 11
    // or in this way
    Optional<Integer> max = list.stream().max((o1, o2) -> o1 - o2);
```

3. get the maximum salary of the person

```Java
    Optional<Person> maxSalary = personList.stream().max(Comparator.comparingInt(Person::getSalary));
    System.out.println(maxSalary.get());
    // or
    Optional<Person> maxSalary = personList.stream().max((p1, p2) -> p1.getSalary() - p2.getSalary());
```

4. count the number of elements larger than 7

```Java
    List<Integer> list = Arrays.asList(7, 6, 9, 4, 11, 6);
    long count = list.stream().filter(x -> x > 7).count();
    System.out.println(count);
```

### Map (map/flatMap) (peek)

<img src="https://pic3.zhimg.com/80/v2-56cad49b35f264e8a71517c226caaa02_720w.webp">

1. convert all element to upper cases

```Java

    String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
    List<String> stringList = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());
    System.out.println(stringList); // [ABCD, BCDD, DEFDE, FTR]
```

2. Each person's salary increase 1000 (modify the original list, we can use peek)

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, 23, "male", "New York"));
    personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
    personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
    personList.add(new Person("Anni", 8200, 24, "female", "New York"));
    personList.add(new Person("Owen", 9500, 25, "male", "New York"));
    personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

    // 不改变原来员工集合的方式
    List<Person> personListNew = personList.stream().map(person -> {
      Person personNew = new Person(person.getName(), 0, 0, null, null);
      personNew.setSalary(person.getSalary() + 10000);
      return personNew;
    }).collect(Collectors.toList());
    System.out.println("一次改动前：" + personList.get(0).getName() + "-->" + personList.get(0).getSalary());
    System.out.println("一次改动后：" + personListNew.get(0).getName() + "-->" + personListNew.get(0).getSalary());

    // 改变原来员工集合的方式 (not recommended)
    List<Person> personListNew2 = personList.stream().map(person -> {
      person.setSalary(person.getSalary() + 1000);
      return person;
    }).collect(Collectors.toList());
    // or use peek
    List<Person> personListNew2 = personList.stream().peek(person -> person.setSalary(person.getSalary() + 1000))
        .collect(Collectors.toList());
    System.out.println("二次改动前：" + personList.get(0).getName() + "-->" + personListNew.get(0).getSalary());
    System.out.println("二次改动后：" + personListNew2.get(0).getName() + "-->" + personListNew.get(0).getSalary());
  }
}
```

3. concatenate the two list

```Java
    List<String> list = Arrays.asList("m,k,l,a", "1,3,5,7");
    List<String> newList = list.stream().flatMap(s -> {
      // Stream<String[]>
      String[] split = s.split(",");
      Stream<String> stream = Arrays.stream(split);
      return stream;
    }).collect(Collectors.toList());
    System.out.println(newList);
```

### Reduce (reduce) Hard

<img src="https://pic1.zhimg.com/80/v2-8fdfe2b031a9e2c25883066129cd0908_720w.webp">

1. get the sum/max/product of list

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Integer> list = Arrays.asList(1, 3, 2, 8, 11, 4);
    // sum1
    Optional<Integer> sum = list.stream().reduce((x, y) -> x + y);
    // sum2
    Optional<Integer> sum2 = list.stream().reduce(Integer::sum);
    // sum3
    Integer sum3 = list.stream().reduce(0, Integer::sum);

    // product
    Optional<Integer> product = list.stream().reduce((x, y) -> x * y);

    // max1
    Optional<Integer> max = list.stream().reduce((x, y) -> x > y ? x : y);
    // max2
    Integer max2 = list.stream().reduce(1, Integer::max);

    System.out.println("list sum：" + sum.get() + "," + sum2.get() + "," + sum3);
    System.out.println("list product：" + product.get());
    System.out.println("list sum：" + max.get() + "," + max2);

    // another way is map the list to Integer. IntegerStream has sum/max/min/average
    // for statistics using collect would be better, this is just for learning
    Integer sum = list.stream().mapToInt(Integer::intValue).sum();
    Integer max = list.stream().mapToInt(Integer::intValue).max().getAsInt();
    Integer min = list.stream().mapToInt(Integer::intValue).min().getAsInt();
    Double average = list.stream().mapToInt(Integer::intValue).average().getAsDouble();
  }
}
```

2. get the sum of the person's salary | get the maximum salary of the person

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, 23, "male", "New York"));
    personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
    personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
    personList.add(new Person("Anni", 8200, 24, "female", "New York"));
    personList.add(new Person("Owen", 9500, 25, "male", "New York"));
    personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

    // The first 0 in the reduce function is the initial value, used to store the sum of the salary.
    // sum1：
    Optional<Integer> sumSalary = personList.stream().map(Person::getSalary).reduce(Integer::sum);
    // sum2：
    Integer sumSalary2 = personList.stream().reduce(0, (sum, p) -> sum += p.getSalary(),
        (sum1, sum2) -> sum1 + sum2);
    // sum3：
    Integer sumSalary3 = personList.stream().reduce(0, (sum, p) -> sum += p.getSalary(), Integer::sum);

    // max1：
    Integer maxSalary = personList.stream().reduce(0, (max, p) -> max > p.getSalary() ? max : p.getSalary(), Integer::max);
    // max2：
    Integer maxSalary2 = personList.stream().reduce(0, (max, p) -> max > p.getSalary() ? max : p.getSalary(), (max1, max2) -> max1 > max2 ? max1 : max2);
    // max3: get the person object
    Optional<Person> a = personList.stream().reduce(BinaryOperator.maxBy((Comparator.comparingInt(Person::getSalary))));

    System.out.println("sum of salary：" + sumSalary.get() + "," + sumSalary2 + "," + sumSalary3);
    System.out.println("maximum of salary：" + maxSalary + "," + maxSalary2);
  }
}
```

### Collect (collect) most important

collect the elements in the stream into a collection or other data structure.

most dependent on `java.util.stream.Collectors`

#### toList/toSet/toMap

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Integer> list = Arrays.asList(1, 6, 3, 4, 6, 7, 9, 6, 20);
    List<Integer> listNew = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());
    Set<Integer> set = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toSet());

    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, 23, "male", "New York"));
    personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
    personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
    personList.add(new Person("Anni", 8200, 24, "female", "New York"));

    Map<?, Person> map = personList.stream().filter(p -> p.getSalary() > 8000).collect(Collectors.toMap(Person::getName, p -> p));

    System.out.println("toList:" + listNew);
    System.out.println("toSet:" + set);
    System.out.println("toMap:" + map);
  }
}
```

#### statistics (count/average/sum/max/min)

Collectors provides a variety of statistical methods:

- count : count()
- average : averagingInt/averagingLong/averagingDouble
- max/min : maxBy/minBy
- sum : summingInt/summingLong/summingDouble
- calculate all the statistics : summarizingInt/summarizingLong/summarizingDouble

1. statistics of the person's number, avg salary，sum of salary，max salary.

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, 23, "male", "New York"));
    personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
    personList.add(new Person("Lily", 7800, 21, "female", "Washington"));

    // count
    Long count = personList.stream().collect(Collectors.counting());
    // avg salary
    Double average = personList.stream().collect(Collectors.averagingDouble(Person::getSalary));
    // max salary
    Optional<Integer> max = personList.stream().map(Person::getSalary).collect(Collectors.maxBy(Integer::compare));
    // sum salary , for easy to use, map to int and call sum()
    Integer sum = personList.stream().collect(Collectors.summingInt(Person::getSalary));
    // SummaryStatistics : count/average/sum/max/min
    DoubleSummaryStatistics collect = personList.stream().collect(Collectors.summarizingDouble(Person::getSalary));

    System.out.println("员工总数：" + count);
    System.out.println("员工平均工资：" + average);
    System.out.println("员工工资总和：" + sum);
    System.out.println("员工工资所有统计：" + collect);
  }
}
```

#### groupingBy/partitioningBy (mapping)

- groupingBy : cut the stream into multiple `Maps`
- partitioningBy : cut the stream into two `Maps` (salary > 8000 and salary <= 8000)

<img src="https://pic2.zhimg.com/80/v2-cf16b4beb10bbcd990152729f8b070f1_720w.webp">

1. partition the person into two groups (salary > 8000 and salary <= 8000) || group the person by sex and area

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, "male", "New York"));
    personList.add(new Person("Jack", 7000, "male", "Washington"));
    personList.add(new Person("Lily", 7800, "female", "Washington"));
    personList.add(new Person("Anni", 8200, "female", "New York"));
    personList.add(new Person("Owen", 9500, "male", "New York"));
    personList.add(new Person("Alisa", 7900, "female", "New York"));

    // 将员工按薪资是否高于8000分组
        Map<Boolean, List<Person>> part = personList.stream().collect(Collectors.partitioningBy(x -> x.getSalary() > 8000));
        // 将员工按性别分组
        Map<String, List<Person>> group = personList.stream().collect(Collectors.groupingBy(Person::getSex));
        // 将员工按性别分组, return the name of the person
        Map<String, List<String>> group2 = personList.stream().collect(Collectors.groupingBy(Person::getSex, Collectors.mapping(Person::getName, Collectors.toList())));
        // 将员工先按性别分组，再按地区分组
        Map<String, Map<String, List<Person>>> group2 = personList.stream().collect(Collectors.groupingBy(Person::getSex, Collectors.groupingBy(Person::getArea)));
        System.out.println("员工按薪资是否大于8000分组情况：" + part);
        System.out.println("员工按性别分组情况：" + group);
        System.out.println("员工按性别、地区：" + group2);
  }
}
// {female={New York=[Anni 8200 female New York, Alisa 7900 female New York], Washington=[Lily 7800 female Washington]}, male={New York=[Tom 8900 male New York, Owen 9500 male New York], Washington=[Jack 7000 male Washington]}}
```

#### joining

join the elements in the stream into a string

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, 23, "male", "New York"));
    personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
    personList.add(new Person("Lily", 7800, 21, "female", "Washington"));

    String names = personList.stream().map(p -> p.getName()).collect(Collectors.joining(","));
    System.out.println("all employee：" + names);
    List<String> list = Arrays.asList("A", "B", "C");
    String string = list.stream().collect(Collectors.joining("-"));
    System.out.println("concatenated string：" + string);
  }
}
```

#### Reducing

Compared with the reduce method of Stream, the reducing method of Collectors is more flexible. It support custom operations.

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();
    personList.add(new Person("Tom", 8900, 23, "male", "New York"));
    personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
    personList.add(new Person("Lily", 7800, 21, "female", "Washington"));

    // 每个员工减去起征点后的薪资之和（这个例子并不严谨，但一时没想到好的例子）
    Integer sum = personList.stream().collect(Collectors.reducing(0, Person::getSalary, (i, j) -> (i + j - 5000)));
    System.out.println("员工扣税薪资总和：" + sum);

    // stream的reduce
    Optional<Integer> sum2 = personList.stream().map(Person::getSalary).reduce(Integer::sum);
    System.out.println("员工薪资总和：" + sum2.get());
  }
}
```

### Sort (sorted)

- sorted() : sort in ascending order
- sorted(Comparator.reverseOrder()) : sort in descending order
- sorted(Comparator com) : custom sort

1. sort person by salary (if salary is the same, sort by age)

```Java
public class StreamTest {
  public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();

    personList.add(new Person("Sherry", 9000, 24, "female", "New York"));
    personList.add(new Person("Tom", 8900, 22, "male", "Washington"));
    personList.add(new Person("Jack", 9000, 25, "male", "Washington"));
    personList.add(new Person("Lily", 8800, 26, "male", "New York"));
    personList.add(new Person("Alisa", 9000, 26, "female", "New York"));

    // 按工资升序排序（自然排序）
    List<String> newList = personList.stream().sorted(Comparator.comparing(Person::getSalary)).map(Person::getName)
        .collect(Collectors.toList());
    // 按工资倒序排序
    List<String> newList2 = personList.stream().sorted(Comparator.comparing(Person::getSalary).reversed())
        .map(Person::getName).collect(Collectors.toList());
    // 先按工资再按年龄升序排序
    List<String> newList3 = personList.stream()
        .sorted(Comparator.comparing(Person::getSalary).thenComparing(Person::getAge)).map(Person::getName)
        .collect(Collectors.toList());
    // 先按工资再按年龄自定义排序（降序）
    List<String> newList4 = personList.stream().sorted((p1, p2) -> {
      if (p1.getSalary() == p2.getSalary()) {
        return p2.getAge() - p1.getAge();
      } else {
        return p2.getSalary() - p1.getSalary();
      }
    }).map(Person::getName).collect(Collectors.toList());

    System.out.println("按工资升序排序：" + newList);
    System.out.println("按工资降序排序：" + newList2);
    System.out.println("先按工资再按年龄升序排序：" + newList3);
    System.out.println("先按工资再按年龄自定义降序排序：" + newList4);
  }
}
```

### distinct/limit/skip

<img src="https://pic4.zhimg.com/80/v2-60a0fe2ff3952b0c2ce4072f4f32c317_720w.webp">

<img src="https://pic2.zhimg.com/80/v2-00e33517160122f9869eafe8ef8daba5_720w.webp">

<img src="https://pic3.zhimg.com/80/v2-ca6bc341310807b7999d62c30d0912fa_720w.webp">

```Java
public class StreamTest {
  public static void main(String[] args) {
    String[] arr1 = { "a", "b", "c", "d" };
    String[] arr2 = { "d", "e", "f", "g" };

    Stream<String> stream1 = Stream.of(arr1);
    Stream<String> stream2 = Stream.of(arr2);
    // concat:合并两个流 distinct：去重
    List<String> newList = Stream.concat(stream1, stream2).distinct().collect(Collectors.toList());
    // limit：限制从流中获得前n个数据
    List<Integer> collect = Stream.iterate(1, x -> x + 2).limit(10).collect(Collectors.toList());
    // skip：跳过前n个数据
    List<Integer> collect2 = Stream.iterate(1, x -> x + 2).skip(1).limit(5).collect(Collectors.toList());

    System.out.println("流合并：" + newList);
    System.out.println("limit：" + collect);
    System.out.println("skip：" + collect2);
  }
}
```
