# Java 中 List 和 数组之间的转换

## 一. 数组转为 LIST

```Java
Element[] array = {new Element(1),new Element(2),new Element(3)};
```

## 1. 最流行也是被最多人接受的答案

最普遍也是被最多人接受的答案如下：

```Java
ArrayList<Element> arrayList = new ArrayList<Element>(Arrays.asList(array));
```

首先，我们来看下ArrayList的构造方法的文档。
*ArrayList(Collection < ? extends E > c) : 构造一个包含特定容器的元素的列表，并且根据容器迭代器的顺序返回。*
所以构造方法所做的事情如下：  
1.将容器c转换为一个数组  
2.将数组拷贝到ArrayList中称为”elementData”的数组中  
ArrayList的构造方法的源码如下：

```Java
public ArrayList(Collection<? extends E> c) {
       elementData = c.toArray();
       size = elementData.length;

       if (elementData.getClass() != Object[].class)
             elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

## 2. 另外一个流行的答案

另外一个流行的答案是：

```Java
List<Element> list = Arrays.asList(array);
```

这不是最好的，因为**asList()**返回的列表的大小是固定的。事实上，返回的列表不是**java.util.ArrayList**，而是定义在**java.util.Arrays**中一个私有静态类。我们知道**ArrayList**的实现本质上是一个数组，而**asList()**返回的列表是由原始数组支持的固定大小的列表。这种情况下，如果添加或删除列表中的元素，程序会抛出异常**UnsupportedOperationException**。

## 3. 又一个解决方案

```Java
Element[] array = {new Element(1), new Element(2)};
List<element> list = new ArrayList<element>(array.length);
Collections.addAll(list, array);
```

## 二. LIST 转为数组

```Java
List<String> list = new ArrayList<>();
String[] array = list.toArray(new String[list.size()]);
```
