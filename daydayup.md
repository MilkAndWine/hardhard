# 20191022
## 一、subString()方法到底做了什么，不同版本的JDK中是否有区别，为什么
+ substring(int beginIndex, int endIndex)方法将返回一个字符串的子串，从beginIndex开始，到endIndex-1

+ JDK 6中的subString()方法,一个新的字符串被创建，这个字符串的字符数组还是指向原来那个数组的内存堆，不同的是offset和count的属性值不同

![](http://www.programcreek.com/wp-content/uploads/2013/09/string-substring-jdk6-650x389.jpeg)


```

	//JDK 6
	String(int offset, int count, char value[]) {
		this.value = value;
		this.offset = offset;
		this.count = count;
	}
 
	public String substring(int beginIndex, int endIndex){
		//check boundary
		return  new String(offset + beginIndex, endIndex - beginIndex, value);
	}

```

+ JDK 7中的subString()创建了新的字符数组
![](http://www.programcreek.com/wp-content/uploads/2013/09/string-substring-jdk71-650x389.jpeg)

```

	//JDK 7
	public String(char value[], int offset, int count) {
		//check boundary
		this.value = Arrays.copyOfRange(value, offset, offset + count);
	}	

 
	public String substring(int beginIndex, int endIndex){
		//check boundary
		int subLen = endIndex - beginIndex;
		return new String(value, beginIndex, subLen);
	}

```

## 二、如何理解String的intern方法，不同版本的JDK有何不同，为什么

+ 在JDK1.6及以前，intern()方法：

>当字符串在常量池存在时，则返回常量池中的字符串；当字符串在常量池不存在时，则在常量池中拷贝一份，然后再返回常量池中的字符串。

+ 此时再来看看JDK1.6以后的intern()方法：

>当字符串在常量池存在时，则返回常量池中的字符串；当字符串在常量池不存在时，则把堆内存中此对象引用添加到常量池中，然后再返回此引用。

+ JDK1.6及以前是将堆内存的对象拷贝一份到常量池中，JDK1.6以后是将此对象的引用放入常量池中。 因此，JDK1.6以后的intern()方法可以有效的减少内存的占用，提高运行时的性能。

[详情参考](https://blog.csdn.net/Game_Zmh/article/details/101701708)

## 三、什么是Java中整型的缓存机制

在Java 5中，在Integer的操作上引入了一个新功能来节省内存和提高性能。整型对象通过使用相同的对象引用实现了缓存和重用。

适用于整数值区间-128 至 +127。

只适用于自动装箱。使用构造函数创建对象不适用。

```

    public static void main(String... strings) {
        Integer integer1 = 3;
        Integer integer2 = 3;
        if (integer1 == integer2)
            System.out.println("integer1 == integer2");
        else
            System.out.println("integer1 != integer2");
 
        Integer integer3 = 300;
        Integer integer4 = 300;
        if (integer3 == integer4)
            System.out.println("integer3 == integer4");
        else
            System.out.println("integer3 != integer4");

    }
		
```

输出结果
>integer1 == integer2
>
>integer3 != integer4



# 20191021
## 一、Java中的char是否可以存储中文字符

可以，char是用unicode编码，占两个字节，如果某个特殊的汉字没有被包含在unicode编码字符集中，那么，这个char型变量中就不能存储这个特殊汉字

## 二、String s =new  String("xc")，定义了几个对象
一个或两个
在字符串池中找得到“xc”这个对象，则在堆里创建一个对象供引用
若在字符串内存池中找不到“xc”这个对象，则在堆和字符串池分别创建一个对象

## 三、String有没有长度限制，为什么，如果有，超过限制会发生什么
有
根据String的源码public String(char value[], int offset, int count)的定义，count是int类型的，最长的长度为 2^32，也就是4G。
不过，我们在编写源代码的时候，如果使用 Sting str = "aaaa";的形式定义一个字符串，那么双引号里面的ASCII字符最多只能有 65534 个。
>因为在class文件的规范中， CONSTANT_Utf8_info表中使用一个16位的无符号整数来记录字符串的长度的，最多能表示 65536个字节，而java class 文件是使用一种变体UTF-8格式来存放字符的，null值使用两个字节来表示，因此只剩下 65536－ 2 ＝ 65534个字节。也正是变体UTF-8的原因，如果字符串中含有中文等非ASCII字符，那么双引号中字符的数量会更少（一个中文字符占用三个字节）。

如果超出这个数量，在编译的时候编译器会报错


## 四、String,StringBuilder,StringBuffer之间的区别与联系
+ String类中对字符串的保存格式为

>private final char value[];


它被final所修饰，所以它是不可变的，从它被创建直至被销毁，它的字符序列都没有改变，我们对它的一系列操作都是通过创建新的String对象来完成的。
+ StringBuilder、StringBuffer两个类都继承自抽象类AbstractStringBuilder，它们对字符串的保存格式为

>char[] value;


所以它们是可变的，我们可以通过它提供的方法（如append(),insert()等）通过直接修改字符序列来完成对字符串的操作，StringBuffer支持和StringBuilder所有相同的操作。

+ （可变性）当我们不对存入的字符串序列进行变化时用String，而如果经常要对存储的序列进行变化时应使用StringBuffer或StringBuilder。
+ （安全性与效率）当只在单线程中使用时，选用效率更高的StringBuilder，而如果应用于多线程，应选用更安全的StringBuffer

![](https://img-blog.csdn.net/20180909103024199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p4MjAxNTIxNjg1Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
