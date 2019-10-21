#20191021
##一、Java中的char是否可以存储中文字符
可以，char是用unicode编码，占两个字节，如果某个特殊的汉字没有被包含在unicode编码字符集中，那么，这个char型变量中就不能存储这个特殊汉字

##二、String s =new  String("xc")，定义了几个对象
一个或两个
在字符串池中找得到“xc”这个对象，则在堆里创建一个对象供引用
若在字符串内存池中找不到“xc”这个对象，则在堆和字符串池分别创建一个对象

##三、String有没有长度限制，为什么，如果有，超过限制会发生什么
有
根据String的源码public String(char value[], int offset, int count)的定义，count是int类型的，最长的长度为 2^32，也就是4G。
不过，我们在编写源代码的时候，如果使用 Sting str = "aaaa";的形式定义一个字符串，那么双引号里面的ASCII字符最多只能有 65534 个。
>因为在class文件的规范中， CONSTANT_Utf8_info表中使用一个16位的无符号整数来记录字符串的长度的，最多能表示 65536个字节，而java class 文件是使用一种变体UTF-8格式来存放字符的，null值使用两个字节来表示，因此只剩下 65536－ 2 ＝ 65534个字节。也正是变体UTF-8的原因，如果字符串中含有中文等非ASCII字符，那么双引号中字符的数量会更少（一个中文字符占用三个字节）。

如果超出这个数量，在编译的时候编译器会报错


##四、String,StringBuilder,StringBuffer之间的区别与联系
+ String类中对字符串的保存格式为
>private final char value[];


它被final所修饰，所以它是不可变的，从它被创建直至被销毁，它的字符序列都没有改变，我们对它的一系列操作都是通过创建新的String对象来完成的。
+ StringBuilder、StringBuffer两个类都继承自抽象类AbstractStringBuilder，它们对字符串的保存格式为
>char[] value;


所以它们是可变的，我们可以通过它提供的方法（如append(),insert()等）通过直接修改字符序列来完成对字符串的操作，StringBuffer支持和StringBuilder所有相同的操作。

+ （可变性）当我们不对存入的字符串序列进行变化时用String，而如果经常要对存储的序列进行变化时应使用StringBuffer或StringBuilder。
+ （安全性与效率）当只在单线程中使用时，选用效率更高的StringBuilder，而如果应用于多线程，应选用更安全的StringBuffer

![](https://img-blog.csdn.net/20180909103024199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p4MjAxNTIxNjg1Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
