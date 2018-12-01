---
title: Java中Iterator的fast-fail分析
key: 20170502
tags: java
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---
`Java`中的`Iterator`非常方便地为所有的数据源提供了一个统一的数据读取(删除)的接口，但是新手通常在使用的时候容易报如下错误`ConcurrentModificationException`，原因是在使用迭代器时候底层数据被修改，最常见于数据源不是线程安全的类，如`HashMap & ArrayList`等。
<!--more-->
## 为什么要有fast-fail ##

### 一个案例 ###

来一个新手容易犯错的例子：

```
String[] stringArray = {"a","b","c","d"};
List<String> strings = Arrays.asList(stringArray);
Iterator<String> iterator = strings.iterator();
while (iterator.hasNext()) {    
  if(iterator.next().equals("c")) {        
    strings.remove("c");    
  }
}
```

更加常见的是在foreach(**本质一样，都是调用Iterator时，操作了原始的strings**)语句中：

```
for(String s : strings) {    
  if(s.equals("c")) {        
    strings.remove("c");
  }
}
```

### 产生原因 ###

`Java`中的集合类(数据源)分为两种类型：线程安全，位于`java.util.concurrent`命名目录下，如`CopyOnWriteArrayList`；线程不安全：位于`java.util`目录下,如`ArrayList,HashMap`。所谓线程安全是在多线程环境下，这个类还能表现出和行为规范一致的结果，是否文绉绉的...自己google吧。

那既然我们可以有线程安全的集合替代品，那么为什么还要存在`ArrayList`等呢？因为线程安全的类通常需要通过各种手段去保持对数据访问的同步，所以通常来说效率会比较差。而如果使用者清楚自身使用场景不存在并发的场景，那么使用非线程安全的集合类在速度上有很大的优势。

如果开发者在使用时没有注意，将非线程安全的集合类用在了并发的场景下，比如线程A获取了`ArrayList`的`iterator`,然后线程B通过调用`ArrayList.add()`修改了ArrayList的数据，此时就有可能会抛出`ConcurrentModificationException`，注意，这里是**有可能**。那为啥上面的例子里面也会报这个错误呢？上面并不存在并发的情况，搂一眼源码吧。

### Iterator源码分析 ###

集合类中的`fast-fail`实现方式都差不多，我们以最简单的`ArrayList`为例吧。
 `ArrayList`中会持有一个变量，声明为:
 `protected transient int modCount = 0;`记录的是我们对`ArrayList`修改的次数，比如我们调用 `add(),remove()`等改变数据的操作时，会将`modCount++`。

我们通过`ArrayList.iterator()`返回的是一个实现了`Iterator`接口的`ArrayListIterator`：

```
private class ArrayListIterator implements Iterator<E> {
  
    //省略部分代码.......
    //初始化时，直接给expectedModCount赋ArrayList的修改次数
    private int expectedModCount = modCount;

    @SuppressWarnings("unchecked") public E next() {
           ............
        ArrayList<E> ourList = ArrayList.this;
        //简单比较一下当前iterator初始化时ArrayList.modCount的值
        //和现在的值是否一致，如果不相等，认为在获取了当前iterator之后
        //有别的位置(有可能是别的线程)修改了ArrayList，直接抛异常
        if (ourList.modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
          ............
    }
}
```

原理很简单，构建`Iterator`时将当前`ArrayList`的`modCount`存起来，以后每一次`next()`时，判断`ArrayList`的`modCount`值是否有变化，如果有，则是在这个过程中有代码改变了数据(前面已经提及，只有调用`add() remove()`等才会去修改`modCount`的值)。
 这也说明了为什么在例子里面我们并不是并发的场景也报错，因为我们调用`ArrayList.remove()`时改变了`modCount`的值。

但是这个东西意义有多大呢？在我看来它有点**画蛇添足的嫌疑**。因为在真正的并发场景下，这个`fast-fail`机制并不能真正即使发现另外线程访问并修改`ArrayList`中的数据。原因如下：

1. 再看看`modCount`的定义`protected transient int modCount = 0;`。你没有看错，它就是一个普通的变量，那么在并发场景下由于共享对象的不可见性，有可能别的线程修改了`ArrayList`中的`modCount`，而`iterator`所在的线程却并没有读取到这个更新。`HashMap`在1.6以前确实是用了`volatile`来修饰了`modCount`来保证各个线程直接对`modCount`的可见性，但是在1.7里面把这个修饰去掉了，而且认为这是一个bug-->[Java7去掉volatitle](https://link.jianshu.com?t=http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6625725),可悲啊。。。原因嘛，就是JDK的开发者认为为了这么个破事而需要使用`volatitle`简直浪费效率。
2. 就算是使用`volatitle`就完事大吉了吗？nono，举个最简单的例子，线程A获取了一个集合类的`Iterator`,线程B调用了集合类的`add()`,在`add()`还没有执行到`modCount++`时，线程A获取执行，并执行结束。在这种场景下，执行结果并不确定。对于`ArrayList`的`Iterator`来说，有可能会报一个数组越界的异常...

## 总结 ##

`fast-fail`是JDK为了**提示**开发者将非线程安全的类使用到并发的场景下时，抛出一个异常，及早发现代码中的问题。但正如本文前面所述，这种机制却不能绝对正确地给出提示，而且老的JDK版本为了更好地支持这个机制还付出了一定的效率代价。

`fast-fail`存在的唯一价值可能就是**给新手制造一些迷惑，给他深入探索的动力...嘿嘿**

补充：

很多网上资料说在使用`Iterator`时是不能修改数据的，这样也并不完全准确。即便是支持`fast-fail`的`Iterator`本身也提供了`remove()`来删除当前遍历到的元素，例如：`ArrayListIterator中的remove()`，前面举的栗子改成如下即可：

```
while (iterator.hasNext()) {    
  if(iterator.next().equals("c")) {        
    iterator.remove("c");    
  }
}
```




Java中Iterator的fast-fail分析  https://www.jianshu.com/p/d9ee4b075d5a


    