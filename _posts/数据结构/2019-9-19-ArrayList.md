# ArrayList

ArrayList是List接口的实现类，它是支持根据需要而动态增长的数组。java中标准数组是定长的，在数组被创建之后，它们不能被加长或缩短。这就意味着在创建数组时需要知道数组的所需长度，但有时我们需要动态从程序中获取数组长度。ArrayList 就是为此而生的。

### ArrayList 的底层实现

List的底层实现主要是数组，很多的操作都是从数组演变而来。list每次在调用add()方法添加元素时，arraylist都需要对这个list的容量进行一个判断。如果容量够，直接添加，否则需要进行扩容。在jdk1.8 arraylist这个类中，扩容调用的是grow()方法，通过grow()方法中调用的Arrays.copyof()方法进行对原数组的复制，在通过调用System.arraycopy()方法进行复制，达到扩容的目的


##### 代码详解：

我们从开始定义一个 arraylist 实例开始

```java
List list = new ArrayList();
```

```java
//定义数组的默认容量
private static final int DEFAULT_CAPACITY = 10;
//当arraylist 定义初始容量为0时，为此定义一个空实例
private static final Object[] EMPTY_ELEMENTDATA = {};
//用于定义默认大小的空实例，....
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//存储ArrayList元素的数组缓冲区，ArrayList的容量是这个数组缓冲区的长度，当给ArrayList添加第一个元素时，elementData 的容量被扩展为 DEFAULT_CAPACITY
transient Object[] elementData;
//ArrayList的大小(它包含的元素的数量)，注意size的初始值并没有明确指定，默认为 0
private int size;

//构造具有指定初始容量的空列表
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
//我们在创建ArrayList实例时其实是创建一个数组，构造一个初始容量为10的空列表,这里其实只是将DEFAULTCAPACITY_EMPTY_ELEMENTDATA 数组赋值给 elementData数组，并没有定义初始容量，而在我们向elementData数组中添加一个元素调用 add方法时，会给elementData数组赋一个初始容量
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

//将指定的元素追加到list的末尾,这里调用了ensureCapacityInternal方法
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //注意这里是size++，先赋值后运算
    elementData[size++] = e;
    return true;
}
//确认内部的容量
private void ensureCapacityInternal(int minCapacity) {
    //如果是个空数组则最小容量取默认容量与minCapacity之间的最大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
//确认扩展容量
private void ensureExplicitCapacity(int minCapacity) {
    //记录list 被改变的次数，初始值为0
    modCount++;
    // 如果最小需要空间比elementData的内存空间要大，则需要扩容
    if (minCapacity - elementData.length > 0)
        //扩容
        grow(minCapacity);
}

//定义数组得最大值：2147483639
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//扩展容量的方法
private void grow(int minCapacity) {
    //得到原数组的长度
    int oldCapacity = elementData.length;
    // 扩容至原来的1.5倍，>> 转二进制向右边移动 相当于 /2
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //判断新数组的容量够不够，够了就直接使用这个长度创建新数组，不够就将数组长度设置为需要的长度
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //若新创建的容量大于数组的最大值，调用hugeCapacity方法，否则将创建新的数组
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

//若新创建的容量大于数组的最大值，则将返回Integer的最大值（2147483647），否则返回数组的最大值（2147483639）
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}


```

上述代码给出了ArrayList 在创建新的实例、扩容时的过程，而在定义elementData时用transient所修饰，具体原因我们继续分析

### ArrayList中elementData为什么被transient修饰？

[Java中transient关键字的详细总结](https://blog.csdn.net/u012723673/article/details/80699029)

我们了解了 transient 的用法后，也就知道此问的原因了，在ArrayList中，ArrayList在序列化的时候会调用writeObject，直接将size和element写入ObjectOutputStream；

反序列化时调用readObject，从ObjectInputStream获取size和element，再恢复到elementData。

而定义的elementData是一个缓存数组，它通常会预留一些容量，等容量不足时再扩充容量，那么有些空间可能就没有实际存储元素，采用上述的方式来实现序列化时，就可以保证**只序列化实际存储的那些元素，而不是整个数组**，从而节省空间和时间。



