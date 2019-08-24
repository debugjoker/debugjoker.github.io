---
layout: post
title: ' 剖析Arrays.asList方法'
tags: [code]
---

使用Arrays.asList()的原因无非是想将数组或一些元素转为集合,而你得到的集合并不一定是你想要的那个集合。

而一开始asList的设计时用于打印数组而设计的，但jdk1.5开始，有了另一个比较更方便的打印函数Arrays.toString(),于是打印不再使用asList()，而asList()恰巧可用于将数组转为集合。

**一、 错误用法**

如果你这样使用过，那你可要注意了。

**1、错误一** 

将基本类型数组作为asList的参数

```java
int[] arr = {1,2,3};
List list = Arrays.asList(arr);
System.out.println(list.size());// 输出结果为1
```

**2、错误二** 

将数组作为asList参数后，修改数组或List

```java
String[] arr = {"Java", "核心" , "技术"};
List list = Arrays.asList(arr);
arr[1] = "SpringBoot";
list.set(2, "学习");
System.out.println(Arrays.toString(arr));// 输出结果为[Java, SpringBoot, 学习]
System.out.println(list.toString());// 输出结果为[Java, SpringBoot, 学习]
```

**3、错误三** 

数组转换为集合后，进行增删元素

```java
String[] arr = {"Java", "核心" , "技术"};
List list = Arrays.asList(arr);
list.add("SpringBoot");
list.remove("核心");
System.out.println(list.toString());// 抛出异常
```

**二、深入探究**

我们通过asList()源码可发现其原因

```java
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
// 这里的ArrayList是java.util.Arrays.ArrayList不是我们常用的java.util.ArrayList
```

继续点进去查看源码，看看java.util.Arrays.ArrayList和java.util.ArrayList两者的区别

```java
public class Arrays {
    private static class ArrayList<E> extends AbstractList<E>
            implements RandomAccess, java.io.Serializable
        {
            private static final long serialVersionUID = -2764017481108945198L;
            private final E[] a;

            ArrayList(E[] array) {
                a = Objects.requireNonNull(array);
            }

            @Override
            public int size() {
                return a.length;
            }
        ｝
}
```



**三、不同之处**

+ 从上述源码可以看出java.util.Arrays.ArrayList实际上是个静态内部类，并没有完全实现List的方法，而ArrayList是直接实现List接口的所有方法。

+ 长度不同 和 实现的方法不同

  Arrays.ArrayList是一个定长集合，因为它没有重写add,remove方法，所以一旦初始化元素后，集合的size就是不可变的。

+ 参数赋值方式不同

  java.util.Arrays.ArrayList将外部数组的引用直接通过“=”赋予内部的泛型数组，所以本质指向同一个数组。java.util.ArrayList是将其他集合转为数组后copy到自己内部的数组的。

  ```java
  // java.util.Arrays.ArrayList
  ArrayList(E[] array) {
      a = Objects.requireNonNull(array);
  }
  
  // java.util.ArrayList
  public ArrayList(Collection<? extends E> c) {
      elementData = c.toArray();// 转换为Object数组
      if ((size = elementData.length) != 0) {
          // c.toArray might (incorrectly) not return Object[] (see 6260652)
          if (elementData.getClass() != Object[].class)
              elementData = Arrays.copyOf(elementData, size, Object[].class);// 数组拷贝
      } else {
          // replace with empty array.
          this.elementData = EMPTY_ELEMENTDATA;
      }
  }
  ```

  

**四、问题分析**

**1、错误一**

由于Arrays.ArrayList参数为可变长泛型，而基本类型是无法泛型化的，所以它把int[] arr数组当成了一个泛型对象，所以集合中最终只有一个元素arr。

**2、错误二**

由于asList产生的集合元素是**数组的引用**，所以当外部数组或集合改变时，数组和集合会同步变化。

**3、错误三**

由于asList产生的集合并没有重写add,remove等方法，所以它会调用父类AbstractList的方法，而父类的方法中抛出的却是异常信息。

**五、问题解决**

```java
// 如果项目中使用了Spring，则可以使用org.springframework.util.CollectionUtils工具类解决
int[] a = {1,2,3};
List list = CollectionUtils.arrayToList(a);
System.out.println(list);
```

最简单的方法遍历数组转换成ArrayList

```java
int[] a = {1,2,3};
List list = new ArrayList();
for (int i : a) {
    list.add(i);
}
System.out.println(list);
```

使用工具类，看似优雅实则还是遍历数组

```java
List<String> list = new ArrayList();
String[] strings = {"Welcome", "to", "China!"};
Collections.addAll(list,strings);
System.out.println(list);
```

```java
// Collections.addAll()方法源码使用的还是遍历
public static <T> boolean addAll(Collection<? super T> c, T... elements) {
    boolean result = false;
    for (T element : elements)
        result |= c.add(element);
    return result;
}
```

使用Java8流处理

```java
// 如果项目中使用的是Java8则可以使用流Stream来处理
int[] a = {1,2,3};
List list = Arrays.stream(a).boxed().collect(Collectors.toList());
System.out.println(list);
```

