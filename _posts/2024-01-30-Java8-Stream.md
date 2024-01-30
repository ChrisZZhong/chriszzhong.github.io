---
layout: post
title: "crack Java 8 Stream"
date: 2023-01-30
description: "Java Stream knowledge"
tag: Java
---

## Stream

<img src="/images/Full-Stack/JavaCore/StreamApi.jpg">

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

// [Tom, Anni, Owen]
```

### Aggregate (count/max/min/sum/average)

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

### Map (map/flatMap) (remember to return the object in map)

<img src="https://pic3.zhimg.com/80/v2-56cad49b35f264e8a71517c226caaa02_720w.webp">

1. convert all element to upper cases

```Java

    String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
    List<String> stringList = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());
    System.out.println(stringList); // [ABCD, BCDD, DEFDE, FTR]
```

2. Each person's salary increase 1000

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

    // 改变原来员工集合的方式
    List<Person> personListNew2 = personList.stream().map(person -> {
      person.setSalary(person.getSalary() + 1000);
      return person;
    }).collect(Collectors.toList());
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

### Reduce (reduce)

<img src="https://pic1.zhimg.com/80/v2-8fdfe2b031a9e2c25883066129cd0908_720w.webp">
