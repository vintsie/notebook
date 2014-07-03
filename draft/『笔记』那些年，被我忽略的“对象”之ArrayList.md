>开篇总是要说些什么，人这一辈子总会遇到很多不顺心的事情，可能来自各个方面。遇到难事时，就怕一蹶不振，最好应该做的是能抓住命运赐予的时机完善自己，做些自己感兴趣的和以前没有机会去做的事情。嗯，不多说了，下边正式开始记录我的拾遗学习过程。

###ArrayList最多能存放多少个Object
我是在看到*ArrayList*的成员变量*MAX_ARRAY_SIZE*时，发现这是自己以前忽略的东西，觉得有必要Mark一下，所以就采用了这样提问的方式，或许可以让我加深一下印象吧，又或许这个可以是三两好友茶余饭后的小谈资。以前或许就没有仔细去想，或者说自己没有仔细地去研究JDK的源码，一直让自己有个错觉，总觉得*ArrayList*的最大长度是*Integer.MAX_VALUE*。知道刚才看到*MAX_ARRAY_SIZE*的时候才知道自己的想法是那么地想当然，应了那句口头禅：学术应谨慎，且行且珍惜。

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

