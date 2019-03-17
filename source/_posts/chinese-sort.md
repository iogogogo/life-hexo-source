---
title: Comparable和Comparator的区别
date: 2019-02-17 11:27:48
tags: Java
---

## **Comparable**

Comparable可以认为是一个**内比较器**，实现了Comparable接口的类有一个特点，就是这些类是可以和自己比较的，至于具体和另一个实现了Comparable接口的类如何比较，则依赖compareTo方法的实现，compareTo方法也被称为**自然比较方法**。如果开发者add进入一个Collection的对象想要Collections的sort方法帮你自动进行排序的话，那么这个对象必须实现Comparable接口。compareTo方法的返回值是int，有三种情况：

1、比较者大于被比较者（也就是compareTo方法里面的对象），那么返回正整数

2、比较者等于被比较者，那么返回0

3、比较者小于被比较者，那么返回负整数

```java
public class Domain implements Comparable<Domain> {
    private String str;

    public Domain(String str) {
        this.str = str;
    }

    public int compareTo(Domain domain) {
        if (this.str.compareTo(domain.str) > 0)
            return 1;
        else if (this.str.compareTo(domain.str) == 0)
            return 0;
        else
            return -1;
    }

    public String getStr() {
        return str;
    }
}

public static void main(String[] args) {
    Domain d1 = new Domain("c");
    Domain d2 = new Domain("c");
    Domain d3 = new Domain("b");
    Domain d4 = new Domain("d");
    System.out.println(d1.compareTo(d2));
    System.out.println(d1.compareTo(d3));
    System.out.println(d1.compareTo(d4));
}
// 输出结果
0
1
-1
```

## Comparator

Comparator可以认为是是一个**外比较器**，个人认为有两种情况可以使用实现Comparator接口的方式：

1、一个对象不支持自己和自己比较（没有实现Comparable接口），但是又想对两个对象进行比较

2、一个对象实现了Comparable接口，但是开发者认为compareTo方法中的比较方式并不是自己想要的那种比较方式

Comparator接口里面有一个compare方法，方法有两个参数T o1和T o2，是泛型的表示方式，分别表示待比较的两个对象，方法返回值和Comparable接口一样是int，有三种情况：

1、o1大于o2，返回正整数

2、o1等于o2，返回0

3、o1小于o3，返回负整数

```java
public class DomainComparator implements Comparator<Domain> {
    public int compare(Domain domain1, Domain domain2) {
        if (domain1.getStr().compareTo(domain2.getStr()) > 0)
            return 1;
        else if (domain1.getStr().compareTo(domain2.getStr()) == 0)
            return 0;
        else
            return -1;
    }
}

public static void main(String[] args) {
    Domain d1 = new Domain("c");
    Domain d2 = new Domain("c");
    Domain d3 = new Domain("b");
    Domain d4 = new Domain("d");
    DomainComparator dc = new DomainComparator();
    System.out.println(dc.compare(d1, d2));
    System.out.println(dc.compare(d1, d3));
    System.out.println(dc.compare(d1, d4));
}
// 输出结果
0
1
-1
```

因为泛型指定死了，所以实现Comparator接口的实现类只能是两个相同的对象（不能一个Domain、一个String）进行比较了，因此实现Comparator接口的实现类一般都会以"待比较的实体类+Comparator"来命名

## 总结

1、如果实现类没有实现Comparable接口，又想对两个类进行比较（或者实现类实现了Comparable接口，但是对compareTo方法内的比较算法不满意），那么可以实现Comparator接口，自定义一个比较器，写比较算法

2、实现Comparable接口的方式比实现Comparator接口的耦合性 要强一些，如果要修改比较算法，要修改Comparable接口的实现类，而实现Comparator的类是在外部进行比较的，不需要对实现类有任何修 改。从这个角度说，其实有些不太好，尤其在我们将实现类的.class文件打成一个.jar文件提供给开发者使用的时候。实际上实现Comparator 接口的方式后面会写到就是一种典型的**策略模式**。



## 中文实现排序

#### treeMap 方式

```java

@org.junit.Test
public void test() {
    CollatorComparator comparator = new CollatorComparator();
    TreeMap<String, Object> sortMap = new TreeMap<>(comparator);
    sortMap.put("广州", "500");
    sortMap.put("安徽", "100000000000");
    sortMap.put("中山", "9000");
    sortMap.put("潮州", "8000");
    //注意：每次对TreeMap进行put()时，TreeMap都会自动调用它的compare(key,Entry.key)
    //按照key进行排序
    System.out.println(sortMap);
}

class CollatorComparator implements Comparator<Object> {
    Collator collator = Collator.getInstance();

    @Override
    public int compare(Object element1, Object element2) {
        CollationKey key1 = collator.getCollationKey(element1.toString());
        CollationKey key2 = collator.getCollationKey(element2.toString());
        return key1.compareTo(key2);
    }
}

```

#### Comparator 方式
```java
@org.junit.Test
public void test() {
    Comparator<Object> com = Collator.getInstance(java.util.Locale.CHINA);
    String[] newArray = {"中山", "汕头", "广州", "$", "安庆", "1", "阳江", "z", "南京", "武汉", "北京", "安阳", "北方"};
    List<String> list = Arrays.asList(newArray);
    list.sort(com);
    for (String i : list) {
        System.out.print(i + "  ");
    }
}
```