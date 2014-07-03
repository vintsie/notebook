>开篇总是要说些什么，人这一辈子总会遇到很多不顺心的事情，可能来自各个方面。遇到难事时，就怕一蹶不振，最好应该做的是能抓住命运赐予的时机完善自己，做些自己感兴趣的和以前没有机会去做的事情。嗯，不多说了，下边正式开始记录我的拾遗学习过程。

###ArrayList最多能存放多少个Object
我是在看到 *ArrayList* 的成员变量 *MAX_ARRAY_SIZE* 时，发现这是自己以前忽略的东西，觉得有必要Mark一下，所以就采用了这样提问的方式，或许可以让我加深一下印象吧，又或许这个可以是三两好友茶余饭后的小谈资。以前或许就没有仔细去想，或者说自己没有仔细地去研究JDK的源码，一直让自己有个错觉，总觉得 *ArrayList* 的最大长度是 *Integer.MAX_VALUE* 。知道刚才看到 *MAX_ARRAY_SIZE* 的时候才知道自己的想法是那么地想当然，应了那句口头禅：学术应谨慎，且行且珍惜。

以下，引用（刚开始用CSDN的博客，不知道引用格式应该使用哪种模板，如果能支持Markdown，那便是极好的）JDK中的源码+注释，再次加深一下自己的印象：

```java
/** 
 * The maximum size of array to allocate. 
 * Some VMs reserve some header words in an array. 
 * Attempts to allocate larger arrays may result in 
 * OutOfMemoryError: Requested array size exceeds VM limit 
 */  
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; 
```
自己英文水平有限，将就能看懂JDK注释。这段注释的意思大概是：VM为数组可分配的最大长度是*Integer.MAX_VALUE-8*。有些虚拟机在数组会有一些预留字符，如果尝试去分配比这个常量更长的数组空间，可能会导致内存溢出的异常。

###最小化ArrayList实例的存储空间
如果有面试官问你 *ArrayList* 对象的 *Capacity* 和 *Size* 有什么区别，这两个值在什么情况下会相等，思考一下，你会怎么回答？这一小节，讲讲ArrayList中我另一个不太常用的方法 *trimToSize()* 。飞再多的口水，也不如看源码来的实在和过瘾，上源码。

```java
/** 
 * Trims the capacity of this <tt>ArrayList</tt> instance to be the 
 * list's current size.  An application can use this operation to minimize 
 * the storage of an <tt>ArrayList</tt> instance. 
 */  
public void trimToSize() {  
    modCount++;  
    int oldCapacity = elementData.length;  
    if (size < oldCapacity) {  
        elementData = Arrays.copyOf(elementData, size);  
    }  
}
```
从源码的注释可以看出，这个方法的用意是调整 *ArrayList* 实例的Capacity大小为其Size。呼，这一句话说的好曲折，更直观的表达是<code>Set ArrayList.Capacity = ArrayList.size()</code>。 *ArrayList* 对象的Capacity就是其成员变量elementData的长度，所以可以认为这个方法就是为了调整成员变量 *elementData* ，即数据容器的长度，从而优化内存存储。

当然，这个方法我觉得绝对有其特定的使用场景，估计使用场景也非常有限。不知道我这么说是否合理和准确。下面我说一下我这么说的理由，从源码里可以看到使用字符串拷贝的操作，而且还有可能会涉及反射操作，这些“耗时”的操作，无疑会使其使用场景得到限制。但是，有句话叫存在即合理，在JDK版本的升级和演变的过程中，这个方法依然得到了保留，我觉得它还是有存在的价值，在某些场景下它的确能优化程序的内存使用，比如有一些需要暂驻内存的数据，完全可以使用这个方法来优化内存占用嘛。

###纠正自己对toArray的理解
曾几何时，写过一段让自己很后悔的代码。把 *ArrayList* 转换成数据，然后修改数组中的元素，误以为可以直接修改 *ArrayList* 中的数据，现在想想还是挺可笑的。在使用 *ArrayList* 时，心里一定要默默地告诉自己， *ArrayList* 用到了很多<code>System.arraycopy</code>的操作，在使用有关API的时候，一定三思三思再三思。

toArray有个方法可以让我们传入一个数组，JDK中的源码如下

```java
public <T> T[] toArray(T[] a) {  
    if (a.length < size)  
        // Make a new array of a's runtime type, but my contents:  
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());  
    System.arraycopy(elementData, 0, a, 0, size);  
    if (a.length > size)  
        a[size] = null;  
    return a;  
}  
```
这个方法，我觉得需要注意两个地方
* 目标数组的长度。从代码里不难看出，如果入参a的长度小于当前List的长度，那么JVM会根据入参的运行时类型自动创建一个新的Array，然后把源List的数据拷贝到新的数组中去，那么返回的对象跟入参a就没多少关系了。所以不要天真的总是直接拿a继续往下操作。

* 刚才提到了运行时类型，是的，因为泛型是一种编译期约束机制，如果跳过编译期的检查，拷贝会不会出现问题呢，下边会给出测试代码。

看下边的这段代码：

```java
/**
 * 测试ArrayList对象方法-toArray(T[] d)
 * 代码功能顺序：先使用反射越过编译器的检验机制，然后调用
 * ArrayList对象的toArray(T[] d)方法，尝试拷贝Array的数据。
 * author:caiwm (vin.tsie@hotmail.com)
 *
 * @throws Exception
 */
public void testArrayListToArray() throws Exception {
    List<String> list = new ArrayList<String>(2 << 3);

    Method addMethod = list.getClass().getDeclaredMethod("add", Object.class);
    addMethod.invoke(list, "Hello"); // 正常情况 String
    addMethod.invoke(list, Long.MAX_VALUE); // 下边两行代码是绕过编译器存放整形
    addMethod.invoke(list, Integer.MIN_VALUE);

    for (Object obj : list) {
        System.out.println(obj.getClass().getName() + "|" + obj.toString());
    }
    // 代码写到这里，还没有涉及toArray接口，只是用代码阐述了一下泛型的机制，并且做到了
    // 越过编译器使用集合类

    String[] dArray = new String[]{"", ""};
    String[] tempArray = list.toArray(dArray);
    // 上边的这行代码能被成功执行吗？ 
    // 如果你真的试过，就能够知道程序会抛出异常java.lang.ArrayStoreException
    // 而且，就算拷贝成功了，dArray和tempArray也绝不是同一个实例。
}
```
程序的结果总是让人很焦躁，世界上总是会有一些事情的发生不符合自己的预期。我想这是否是Java引入泛型机制的原因之一呢？由于我没有经历过JDK1.5的时代，我没有深切地体会到没有泛型的世界，Coder们的生活是怎样的。不管怎么说，既然Java推出了泛型的机制，对我们来说总是有帮助的，把一些不必要的麻烦挡在Runtime之前，犹未晚矣。


