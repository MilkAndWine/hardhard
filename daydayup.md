# 20191025
## 一、Enumeration和Iterator接口的区别

Enumeration 接口的作用与 Iterator 接口类似，但只提供了遍历 Vector 和 Hashtable 类型集合元素的功能，不支持元素的移除操作。 Iterator 接口添加了一个可选的移除操作，并使用较短的方法名。新的实现应该优先考虑使用 Iterator 接口而不是 Enumeration 接口。

+ 区别：
> Enumeration速度是Iterator的2倍，同时占用更少的内存
> 
> Iterator远远比Enumeration安全，因为其他线程不能够修改正在被iterator遍历的集合里面的对象
> 
> Iterator允许调用者删除底层集合里面的元素，这对Enumeration来说是不可能的。


```

	package java.util;

	public interface Enumeration<E> {
    	boolean hasMoreElements();
    	E nextElement();
	}
	public interface Iterator<E> {
    	boolean hasNext();
    	E next();
    	void remove();
	}
```

## 二、Iterator和ListIterator之间有什么区别

terator和ListIterator主要区别在以下方面：

> 1.使用范围不同，Iterator可以应用于所有的集合，Set、List和Map和这些集合的子类型。而ListIterator只能用于List及其子类型。
> 
> 2. ListIterator有add()方法，可以向List中添加对象，而Iterator不能
> 
> 3. ListIterator和Iterator都有hasNext()和next()方法，可以实现顺序向后遍历，但是ListIterator有hasPrevious()和previous()方法，可以实现逆向（顺序向前）遍历。Iterator就不可以。
> 
> 4. ListIterator可以定位当前的索引位置，nextIndex()和previousIndex()可以实现。Iterator没有此功能。
> 
> 5. 都可实现删除对象，但是ListIterator可以实现对象的修改，set()方法可以实现。Iierator仅能遍历，不能修改。

## 三、什么是fail-fast ，什么是fail-safe ，有什么区别

+ 快速失败（fail-fast)


	在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了增加删除修改等操作，则会抛出Concurrent Modification Exception。

> 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

+ 安全失败（fail-safe）

	采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历

> 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

> 缺点：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。


# 20191024
## 一、什么是synchronizedList,他和Vector有什么区别
+ Vector是java.util包中的一个类。 SynchronizedList是java.util.Collections中的一个静态内部类。

+ 数据增长区别

> 从内部实现机制来讲ArrayList和Vector都是使用数组(Array)来控制集合中的对象。当你向这两种类型中增加元素的时候，如果元素的数目超出了内部数组目前的长度它们都需要扩展内部数组的长度，Vector缺省情况下自动增长原来一倍的数组长度，ArrayList是原来的50%,所以最后你获得的这个集合所占的空间总是比你实际需要的要大。所以如果你要在集合中保存大量的数据那么使用Vector有一些优势，因为你可以通过设置集合的初始化大小来避免不必要的资源开销。


+ SynchronizedList有很好的扩展和兼容功能。他可以将所有的List的子类转成线程安全的类。

+  Vector使用同步方法实现，synchronizedList使用同步代码块实现。
> (因为SynchronizedList只是使用同步代码块包裹了ArrayList的方法，而ArrayList和Vector中同名方法的方法体内容并无太大差异，所以在锁定范围和锁的作用域上两者并无却别。)
> 
> 使用SynchronizedList的时候，进行遍历时其中有listIterator和listIterator(int index)并没有做同步处理,要手动进行同步处理,但是Vector却对该方法加了方法锁。

+ SynchronizedList可以指定锁定的对象。SynchronizedList的同步代码块锁定的是mutex对象，Vector锁定的是this对象。

> 同步代码块和同步方法的区别 1.同步代码块在锁定的范围上可能比同步方法要小，一般来说锁的范围大小和性能是成反比的。 2.同步块可以更加精确的控制锁的作用域（锁的作用域就是从锁被获取到其被释放的时间），同步方法的锁的作用域就是整个方法。 3.静态代码块可以选择对哪个对象加锁，但是静态方法只能给this对象加锁。

## 二、通过Array.asList获得的List有何特点，使用时应该注意什么
+ 当数字为基本数据类型时，打印的都是地址值,由于泛型的类型不能为基本类型，而int[] i是整体作为一个对象进行转换。转换后也是一个对象

```

	public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```
+ 通过 Arrays.asList(strArray) 方式,将数组转换List后，不能对List增删，只能查改，否则抛异常。

## 三、Java中的Collection如何迭代
+ 当使用Iterator对集合元素进行迭代

Iterator并不是把集合元素本身传给了迭代变量，而是把集合元素的值传给了迭代变量，所以修改迭代变量的值对集合本身没有任何改变。

```

	 public static void main(String[] args){
        Collection books = new HashSet(); 
        books.add("语文");
        books.add("数学");
        books.add("英语");
        //打印结果为[语文, 英语, 数学]
        System.out.println(books);
        Iterator it = books.iterator();
        while(it.hasNext()){
            String book= (String)it.next();
            if("语文".equals(book)){
                it.remove();
            }
            //对book变量赋值，不会改变集合元素本身
            book = "测试字段";
           
        }
        //打印结果为[英语, 数学]
        System.out.println(books);
    }


```

>当使用Iterator来迭代访问Collection集合元素时，Collection集合里的元素不能被改变，只有通过Iterator的remove方法来删除上一次next方法返回的集合元素才可以。否则将会引发Java.util.ConcurrentModificationException异常。

```

	public class Test {
    
    	public static void main(String[] args){
      		Collection books = new HashSet();
       		books.add("语文");
        	books.add("数学");
        	books.add("英语");
        	Iterator it  =  books.iterator();
        	while(it.hasNext()){
            	String book = (String)it.next();
            	if(book.equals("英语")){
                	//这样做就会抛Java.util.ConcurrentModificationException
                	books.remove(book);
            }
        }
    }
}
```
>
> boolean hasNext();   如果被迭代的集合元素还没有被遍历，则返回true。
> 
> Object next();    返回集合里下一个元素。
> 
> void remove();   删除集合里上一次next返回的元素。

+ 使用foreach循环遍历集合元素

除了可以使用Iterator类迭代访问Collection集合里的元素外，也可以使用foreach循环来迭代访问集合元素，而且更加便捷如下：

```

	public class Test {
    
    public static void main(String[] args){
        Collection books = new HashSet();
        books.add("语文");
        books.add("数学");
        books.add("英语");
        for(Object o : books){
            String book = (String)o;
            if(book.equals("语文")){
                //这行代码将会引发java.util.ConcurrentModificationException异常
                //books.remove(book);
            }
            System.out.println(o);
        }
    }
}

```

如上所示，同样，当使用foreach循环迭代访问集合元素时，该集合也不能被改变，否则将引发ConcurrentModificationException异常。


# 20191023
## 一、Collection和Collections有什么区别
+ Collection是集合类的上级接口，继承与他有关的接口主要有list和set
+ Collections是针对集合类的一个帮助类，他提供一系列的静态方法实现对各种集合的搜索、排序、线程安全等操作，
> 排序(sort)、混排（Shuffling）、反转（Reverse）、替换所有的元素（fill）、拷贝（copy）、返回Collections中最小元素（min）、返回Collections中最大元素（max）、返回指定源列表中最后一次出现指定目标列表的起始位置（lastIndexOfSubList）、返回指定源列表中第一次出现指定目标列表的起始位置（IndexOfSubList）、根据指定的距离循环移动指定列表中的元素（Rotate）

## 二、Set是如何保证元素不重复的
在Java的Set体系中，根据实现方式不同主要分为两大类。HashSet和TreeSet。
+ HashSet是哈希表实现的,HashSet中的数据是无序的，可以放入null，但只能放入一个null，两者中的值都不能重复，就如数据库中唯一约束

> 因为HashSet底层是用HashMap存储数据的。当向HashSet中添加元素的时候，首先计算元素的hashcode值，然后用这个（元素的hashcode）%（HashMap集合的大小）+1计算出这个元素的存储位置，如果这个位置位空，就将元素添加进去；如果不为空，则用equals方法比较元素是否相等，相等就不添加，否则找一个空位添加。


+  HashSet源码中的的几个构造器：

```

	private transient HashMap<E,Object> map;

    	private static final Object PRESENT = new Object();//注意这个变量，待会会用到
　　

　　		//这个是经常使用到的构造器，可以发现无论是哪一个构造器，HashSet的底层实现都是创建了一个HashMap
   		public HashSet() {
       		map = new HashMap<>();
   		}
    	public HashSet(Collection<? extends E> c) {
        	map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
       		addAll(c);
    	}

    
   		 public HashSet(int initialCapacity, float loadFactor) {
        	map = new HashMap<>(initialCapacity, loadFactor);
    	}		

    	public HashSet(int initialCapacity) {
        	map = new HashMap<>(initialCapacity);
    	}

```

+ 再来看一下HashSet的add方法的源码：

```
	public boolean add(E e) {
　　　　　//PRESENT是一个虚拟值，是为了HashMap实现HashSet的一个假设的值
        return map.put(e, PRESENT)==null;
    }
```

> 当HashSet进行add操作时，其实是将要add的元素作为map的key，将PRESENT这个对象作为map的value，存入map中，并且需要注意到add方法的返回值是一个boolean。
因为HashSet的底层是由HashMap实现的，而HashSet的add方法，是将元素作为map的key进行存储的，map的key是不会重复的，所以HashSet中的元素也不会重复。


+ TreeSet是二差树实现的,Treeset中的数据是自动排好序的，不允许放入null值
> 1. 先查看根节点是否存在，如果不存在就直接吧这个节点放在根节点上。
> 2. 如果根节点存在就依顺序向下查找，如果找到对应的节点，就把该节点的值替换。
> 3. 如果遍历到了叶子节点仍然没有命中，那么就向叶子节点插入一个子节点，小就在左边大就在右边。

> 因为TreeSet插入的值都是空对象，只有key是有效的，key又是相等就覆盖，所以不会重复

## 三、Java中的List有几种实现，各有什么不同
三种，ArrayList、LinkList、Vector

+ ArrayList：
底层数据结构使数组结构，查询速度快，增删改慢，

+ LinkList：底层使用链表结构，增删速度快，查询稍慢；

+ Vector：底层是数组结构，Vector是线程同步的，所以它也是线程安全的。而ArratList是线程异步的，不安全。如果不考虑安全因素，一般用Arralist效率比较高；

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


# 20191020

## 一、String类能不能被继承，为什么，这种设计有什么好处

不能，String 类是被final关键字修饰的，被final修饰的类不能被继承。
好处：主要是为了 “ 效率 ” 和 “ 安全性 ”， 若 String 允许被继承, 由于它的高度被使用率, 可能会降低程序的性能，所以 String 被定义成 final。

## 二、什么是Static Nested Class，什么是Inner Class，二者有什么不同

Inner Class（内部类）定义在类中的类。 (一般是JAVA的说法)

Nested Class（嵌套类）是静态（static）内部类。（一般是C++的说法）
内部类：就是在一个类的内部定义的类，

> 非静态内部类中不能定义静态成员（静态对象是默认加载，那么静态内部类应该先于外部类被加载到内存中）

> 内部类可以直接访问外部类中的成员变量

> 内部类可以定义在外部类的方法外面，也可以定义在外部类的方法体中
> 
> Static Nested Class它可以定义成public、protected、默认的、private等多种类型，而普通类只能定义成public和默认的这两种类型。
> 
> 在外面引用Static Nested Class类的名称为“外部类名.内部类名”。
> 
> 在外面不需要创建外部类的实例对象，就可以直接创建Static Nested Class，例如，假设Inner是定义在Outer类中的Static Nested Class，那么可以使用如下语句创建Inner类：
Outer.Inner inner = new Outer.Inner();

> 由于static Nested Class不依赖于外部类的实例对象，所以，static Nested Class能访问外部类的非static成员变量。
> 
> 当在外部类中访问Static Nested Class时，可以直接使用Static Nested Class的名字，而不需要加上外部类的名字了，在Static Nested Class中也可以直接引用外部类的static的成员变量，不需要加上外部类的名字。

## 三、Java中有几种基本数据类型，如何分类的，String是基本数据类型么

八种,string不是基本数据类型
>布尔型：boolean  
>整型：byte，short，int，long  
>浮点型：float，double  
>字符型：char

## 四、整型的几种中，各个类型的取值范围是多少，如何计算的，超出范围会发生什么
  

|数据类型   | 范围  |  字节数 |
| ------------ | ------------ | ------------ |
|  byte | -2^7 ~ 2^7(-128~127) | 1个字节
|  short | -2^15 ~ 2^15-1(-32768~32767) | 2个字节
|  int | -2^31 ~ 2^31-1(-2147483648~2147483647) | 4个字节
|  long | -2^63 ~ 2^63-1(-9223372036854774808~9223372036854774807) | 8个字节



超出范围会发生数据溢出，eg.最大值（2147483647）+1会变成最小值（-2147483648）

## 三、什么是浮点型，什么是单精度和双精度，为什么代码中不要用浮点数表示金额，那应该用什么表示金额

单精度浮点型float，用32位存储，1位为符号位, 指数8位, 尾数23位，即：float的精度是23位，能精确表达23位的数，超过就被截取。

双精度浮点型double，用64位存储，1位符号位,11位指数,52位尾数，即：double的精度是52位，能精确表达52位的数，超过就被截取。

+ 精度丢失问题  

	从上面我们可以知道，float的精度是23位，double精度是63位。在存储或运算过程中，当超出精度时，超出部分会被截掉，由此就会造成误差。
    
+ 进制转换误差  

	在计算机实际处理和运算过程中，浮点数本质上以二进制形式存在的。而十进制的0.1在二进制下将是一个无限循环小数，这就会导致误差的现。  

	如果一个小数不是2的负整数次幂，用浮点数表示必然产生浮点误差。
	换言之：A进制下的有限小数，转换到B进制下极有可能是无限小数，误差也由此产生。
	浮点数不精确的根本原因在于：尾数部分的位数是固定的，一旦需要表示的数字的精度高于浮点数的精度，那么必然产生误差！
   
解决这个问题的方法是BigDecimal的类，这个类可以表示任意精度的数字，其原理是：用字符串存储数字，转换为数组来模拟大数，实现两个数组的数学运算并将结果返回。