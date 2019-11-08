#20191031
## 一、什么是跳表(SkipList)
跳表是一种可以快速查找，以空间换时间的数据结构，类似于平衡数。跳表是基于链表，插入和删除的效率较高，链表的每个结点不仅记录next结点位置，还可以按照level层级分别记录后继第level个结点。在高并发环境下，平衡树需要一个全局锁来保证整个树的线程安全，而跳表只需要局部锁来控制，查询的时间复杂度为O(longn)  
Skip list让已排序的数据分布在多层链表中，以0-1随机数决定一个数据的向上攀升与否，通过“空间来换取时间”的一个算法，在每个节点中增加了向前的指针，在插入、删除、查找时可以忽略一些不可能涉及到的结点，从而提高了效率。

结构原理图：
![](https://img-blog.csdn.net/2018072721393750?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTQ2MjA0Nw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## 二、什么是ConcurrentSkipListMap，和ConcurrentHashMap有什么区别
ConcurrentSkipListMap提供了一种线程安全的并发访问的排序映射表。内部是SkipList（跳表）结构实现，利用底层的插入、删除的CAS原子性操作，通过死循环不断获取最新的结点指针来保证不会出现竞态条件。在理论上能够在O(log(n))时间内完成查找、插入、删除操作。调用ConcurrentSkipListMap的size时，由于多个线程可以同时对映射表进行操作，所以映射表需要遍历整个链表才能返回元素个数，这个操作是个O(log(n))的操作

ConcurrentHashMap采取了“锁分段”技术来细化锁的粒度：把整个map划分为一系列被成为segment的组成单元，一个segment相当于一个小的hashtable。这样，加锁的对象就从整个map变成了一个更小的范围——一个segment。  
ConcurrentHashMap线程安全并且提高性能原因就在于：对map中的读是并发的，无需加锁；只有在put、remove操作时才加锁，而加锁仅是对需要操作的segment加锁，不会影响其他segment的读写，由此，不同的segment之间可以并发使用，极大地提高了性能。

JDK1.8对ConCurrentHashMap的改动：
> 1.取消segments字段，直接采用transient volatile HashEntry<K,V>[] table保存数据，采用table数组元素作为锁，从而实现了对每一行数据进行加锁，进一步减少并发冲突的概率。  
> 2. 把Table数组＋单向链表的数据结构 变成为  Table数组 ＋ 单向链表 ＋ 红黑树的结构。


#20191030
## 一、介绍下CopyOnWriteArrayList,和普通的ArrayList存在哪些区别，以及，什么是CopyOnWrite
CopyOnWriteArrayList相当于线程安全的ArrayList，它实现了List接口。CopyOnWriteArrayList是支持高并发的。

+ CopyOnWriteArrayList  
> 先copy出一个容器(可以简称副本)，再往新的容器里添加这个新的数据，最后把新的容器的引用地址赋值给了之前那个旧的的容器地址，但是在添加这个数据的期间，其他线程如果要去读取数据，仍然是读取到旧的容器里的数据。可避免在操作集合添加的同时读取会报修改异常，适合并发迭代操作，但是添加操作多时，效率低。

+ CopyOnWrite：
> 线程安全，写时复制， 在往集合中添加数据的时候，先拷贝存储的数组，然后添加元素到拷贝好的数组中，然后用现在的数组去替换成员变量的数组（就是get等读取操作读取的数组）。这个机制和读写锁是一样的，但是比读写锁有改进的地方，那就是读取的时候可以写入的 ，这样省去了读写之间的竞争。适用多读取，少添加。


# 20191029
## 一、HashMap和TreeMap的区别是什么？（需了解红黑树）
1.HashMap是以数组方式存储key/value，基于哈希表实现，允许null作为key和value；TreeMap：基于红黑二叉树的NavigableMap的实现，不允许null
2.HashMap是通过hashcode()对其内容进行快速查找的；HashMap中的元素是没有顺序的；TreeMap中所有的元素都是有某一固定顺序的，如果需要得到一个有序的结果，就应该使用TreeMap；  
3.HashMap和TreeMap都不是线程安全的；  
4.HashMap继承AbstractMap类；覆盖了hashcode() 和equals() 方法，以确保两个相等的映射返回相同的哈希值；TreeMap继承SortedMap类，保持键的有序顺序；  
5.使用HashMap要求添加的键类明确定义了hashcode() 和equals() （可以重写该方法），可以调优初始容量和负载因子优化HashMap的空间使用；存入TreeMap的元素应当实现Comparable接口或者实现Comparator接口，会按照排序后的顺序迭代元素，两个相比较的key不得抛出classCastException。  
6、HashMap：适用于Map插入，删除，定位元素；
TreeMap：适用于按自然顺序或自定义顺序遍历键（key）；

## 二、所有类都可以作为Map的key么？有什么需要注意的
可以，但是需要重载hashCode()和equals()两个方法才能实现自定义键在HashMap中的查找

```

	public class Person {
 
  		private String id;
 
  		public Person(String id) {
    		this.id = id;
  		}
 
  		@Override
  		public boolean equals(Object o) {
    		if (this == o) return true;
    		if (o == null || getClass() != o.getClass()) return false;
 
    		Person person = (Person) o;
 
    		if (id != null ? !id.equals(person.id) : person.id != null) return false;
 
    		return true;
  		}
 
  		@Override
  		public int hashCode() {
    		return id != null ? id.hashCode() : 0;
  		}
	}

```

[示例参考](https://blog.csdn.net/lzhcoder/article/details/84840418)

## 三、了解Java的并发编程包么，并发集合类是什么，有哪些
J.U.C并发包，即java.util.concurrent包，是JDK的核心工具包，是JDK1.5之后，由 Doug Lea实现并引入
整个java.util.concurrent包，按照功能可以大致划分如下：
> juc-locks 锁框架  
> juc-atomic 原子类框架  
> juc-sync 同步器框架  
> juc-collections 集合框架  
> juc-executors 执行器框架

+ 并发集合类：
> List和Set实现类包括CopyOnWriteArrayList, CopyOnWriteArraySet和ConcurrentSkipListSet  
> Map的实现类包括: ConcurrentHashMap和ConcurrentSkipListMap  
> Queue的实现类包括: ArrayBlockingQueue(是数组实现的线程安全的有界的阻塞队列), LinkedBlockingQueue(单向链表实现的(指定大小)阻塞队列，该队列按 FIFO（先进先出）排序元素), LinkedBlockingDeque(双向链表实现的(指定大小)双向并发阻塞队列，该阻塞队列同时支持FIFO和FILO两种操作方式), ConcurrentLinkedQueue(是单向链表实现的无界队列，该队列按 FIFO（先进先出）排序元素)和ConcurrentLinkedDeque(是双向链表实现的无界队列，该队列同时支持FIFO和FILO两种操作方式)


# 20191028
## 一、HashMap和ConcurrentHashMap的区别
+ HashMap
> 底层数组+链表实现，可以存储null键和null值，线程不安全  
> 初始size为16，扩容：newsize = oldsize*2，size一定为2的n次幂  
> 扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入  
> 插入元素后才判断该不该扩容，有可能无效扩容（插入后如果扩容，如果没有再次插入，就会产生无效扩容）  
> 当Map中元素总数超过Entry数组的75%，触发扩容操作，为了减少链表长度，元素分配更均匀  
> 计算index方法：index = hash & (tab.length – 1)

+ ConcurrentHashMap

> 底层采用分段的数组+链表实现，线程安全  
> 通过把整个Map分为N个Segment，可以提供相同的线程安全，但是效率提升N倍，默认提升16倍。(读操作不加锁，由于HashEntry的value变量是 volatile的，也能保证读取到最新的值。)  
>ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术  
> 有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁  
> 扩容：段内扩容（段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免无效扩容

补充：
> HashMap的初始值还要考虑加载因子:  
> 哈希冲突：若干Key的哈希值按数组大小取模后，如果落在同一个数组下标上，将组成一条Entry链，对Key的查找需要遍历Entry链上的每个元素执行equals()比较。  
> 加载因子：为了降低哈希冲突的概率，默认当HashMap中的键值对达到数组大小的75%时，即会触发扩容。因此，如果预估容量是100，即需要设定100/0.75＝134的数组大小。  
> 空间换时间：如果希望加快Key查找的时间，还可以进一步降低加载因子，加大初始大小，以降低哈希冲突的概率。

 
## 二、同样是线程安全的MAP,HashTable和ConcurrentHashMap之间有什么区别

+ HashTable
> 底层数组+链表实现，无论key还是value都不能为null，线程安全，实现线程安全的方式是在修改数据时锁住整个HashTable，效率低，ConcurrentHashMap做了相关优化  
> 初始size为11，扩容：newsize = olesize*2+1  
> 计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length
 
ConcurrentHashMap参考20191028中的一。
  
> 区别：Hashtable的synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术

## 三、hashCode()和equals()方法的作用，二者有什么关系
+ hashcode()

	获得哈希码也称为散列码，哈希码的作用是确定该对象在哈希表中的索引位置

+ equals() : 

	它的作用也是判断两个对象是否相等。但它一般有两种使用情况：
> 情况1，类没有覆盖equals()方法。则通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。(== : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象。)  
> 情况2，类覆盖了equals()方法。我们用equals()方法来两个对象的内容相等；若它们的内容相等，则返回true(即，认为这两个对象相等)，反之为false。

+ 关系：
> 1.equal()相等的两个对象他们的hashCode()肯定相等，也就是用equal()对比是绝对可靠的。  
> 2.hashCode()相等的两个对象他们的equal()不一定相等，也就是hashCode()不是绝对可靠的。


# 20191027
## 一、通过给定集合得到一个synchronized的集合
使用Collections.synchronizedCollection(Collection c)根据指定集合来获取一个synchronized（线程安全的）集合。

## 二、Java中的Map主要有哪几种，之间有什么区别
常用的map实现类主要有HashMap、HashTable、TreeMap、ConcurrentHashMap、LinkedHashMap、weakHashMap等等。
区别：
> HashMap使用位桶和链表实现（最近的jdk1.8改用红黑树存储而非链表），它是线程不安全的Map，方法上都没有synchronize关键字修饰  
> hashTable是线程安全的一个map实现类，它实现线程安全的方法是在各个方法上添加了synchronize关键字。(但是现在已经不再推荐使用HashTable了)  
> ConcurrentHashMap这个map实现类是在jdk1.5中加入的，其在jdk1.6/1.7中的主要实现原理是segment段锁，它不再使用和HashTable一样的synchronize一样的关键字对整个方法进行枷锁，而是转而利用segment段落锁来对其进行加锁，以保证Map的多线程安全。在JAVA的jdk1.8中则对ConcurrentHashMap又再次进行了大的修改，取消了segment段锁字段，采用了CAS+Synchronize技术来保障线程安全。底层采用数组+链表+红黑树的存储结构，也就是和HashMap一样。  
> TreeMap会对Key进行排序，使用了TreeMap存储键值对，再使用iterator进行输出时，会发现其默认采用key由小到大的顺序输出键值对，如果想要按照其他的方式来排序，需要重写也就是override 它的compartor接口。  
> LinkedHashMap它的特点主要在于linked，底层用的是链表来进行的存储。是先进先出FIFIO上，LinkedHashMap主要依靠双向链表和hash表来实现的。

+ 补充ConcurrentHashMap：

	底层采用数组+链表+红黑树的存储结构，也就是和HashMap一样。这里注意Node其实就是保存一个键值对的最基本的对象。其中Value和next都是使用的volatile关键字进行了修饰，以确保线程安全。


```

	class Node<K,V> implements Map.Entry<K,V>{
		final int hash;
		fianl K key;
		volatile V val;
		volatile Node<K,V> next;
		...
	}
```

> 为什么这里会用volatile进行修饰，主要有两个用处：  
> 1、令这个被修饰的变量的更新具有可见性，一旦该变量遭到了修改，其他线程立马就会知道，立马放弃自己在自己工作内存中持有的该变量值，转而重新去主内存中获取到最新的该变量值。  
> 2、产生了内存屏障，这里volatile可以保证CPU在执行代码时保证，所有被volatile中被修饰的之前的一定在之前被执行，也就是所谓的“指令重排序”。

> 在插入元素时，会首先进行CAS判定，如果OK就插入其中，并将size+1，但是如果失败了，就会通过自旋锁自旋后再次尝试插入，直到成功。  
> 所谓的CAS也就是compare And Swap，即在更改前先对内存中的变量值和你指定的那个变量值进行比较，如果相同这说明在这期间没有被修改过，则可以进行修改，而如果不一样了，则就要停止修改，否则就会覆盖掉其他的参数。即内存值a，旧值b，和要修改的值c，如果这里a=b，那么就可以进行更新，就可以将内存值a修改成c。否则就要终止该更新操作。 

同hashMap一样，在JDK1.8中，如果链表中存储的Entry超过了8个则就会自动转换链表为红黑树，提高查询效率。

## 三、遍历map的几种方式

```

    public static void main(String[] args) {
        Map<String, String> map=new HashMap<String, String>();
        map.put("张三1", "男");
        map.put("张三2", "男");
        map.put("张三3", "男");
        map.put("张三4", "男");
        map.put("张三5", "男");
        
        //第一种遍历map的方法，通过加强for循环map.keySet()，然后通过键key获取到value值
        for(String s:map.keySet()){
            System.out.println("key : "+s+" value : "+map.get(s));
        }
        
        
        //第二种只遍历键或者值，通过加强for循环
        for(String s1:map.keySet()){//遍历map的键
            System.out.println("键key ："+s1);
        }
        for(String s2:map.values()){//遍历map的值
            System.out.println("值value ："+s2);
        }
           
        
        //第三种方式Map.Entry<String, String>的加强for循环遍历输出键key和值value
        for(Map.Entry<String, String> entry : map.entrySet()){
            System.out.println("键 key ："+entry.getKey()+" 值value ："+entry.getValue());
        }
       
        
        //第四种Iterator遍历获取，然后获取到Map.Entry<String, String>，再得到getKey()和getValue()
        Iterator<Map.Entry<String, String>> it=map.entrySet().iterator();
        while(it.hasNext()){
            Map.Entry<String, String> entry=it.next();
            System.out.println("键key ："+entry.getKey()+" value ："+entry.getValue());
        }

		// Lambda 获取key and value
 		public void testLambda() {
    		map.forEach((key, value) -> {
     		 System.out.println(key + ":" + value);
    		});
  		}
       
    }
    
    
```


# 20191026
## 一、如何在遍历的同时删除ArrayList中的元素
+ 1.每次list删除元素后，后面的元素都要往前移动一位，就相当于i多加了1，remove后继续遍历就会错过一个元素，所以就需要代码中的i--，抵消remove后，后面元素前移一位的影响*/

```

	 for(int i = 0; i < list.size(); i++){
		System.out.println(i);
            if(list.get(i).equals("C")){
                list.remove(list.get(i));
                i--;
            }
	}
```

+ 2.按索引从大到小，这样remove方法的删除元素导致的后面的元素往前移动一位,对遍历就没有影响了

```

	for(int i= list.size()-1; i > -1; i--){
		if(list.get(i).equals("C")){
                list.remove(list.get(i));
            }
	}

```

+ 3.和1相似，不同的是，是用Itr对象，内含两个指针cursor和lastRet实现的，如果调用remove，相当于cursor回退了一位，和第一种的i--思想有些像

```

	Iterator<String> iterator = list.iterator();
	while(iterator.hasNext()){
		String s = iterator.next();
		if(s.equals("C")){
			iterator.remove();
		}
	}

```

## 二、如何对一组对象进行排序
需要去重写一个compareTo方法，重写这个方法的时候应该要继承一个接口Comparable。

```

	class Person3 implements Comparable<Person3>{
		private String name;
		private int age;
		private double score;
	
		public Person3(String name, int age, double score{
			super();
			this.name = name;
			this.age = age;
			this.score = score;
		}		
	}

	Arrays.sort(array, new Comparator<Person3>(){
			//传入的时候由用户自己选
			@Override
			public int compare(Person3 o1, Person3 o2) {
				// TODO Auto-generated method stub
				int age1 = o1.getAge();
				int age2 = o2.getAge();
				return age1 > age2 ? age1:(age1==age2) ? 0:-1;
				
			}
		});
```

## 三、Comparable和Comparator有何区别

+ 实现了Comparable接口的类有一个特点，就是这些类是可以和自己比较的，至于具体和另一个实现了Comparable接口的类如何比较，则依赖compareTo方法的实现。compareTo方法的返回值是int，有三种情况：

>1、比较者大于被比较者（也就是compareTo方法里面的对象），那么返回正整数  
>2、比较者等于被比较者，那么返回0  
>3、比较者小于被比较者，那么返回负整数

```

	 public interface Comparable<T> {
   		 public int compareTo(T o);
 	}

```

+ Comparator可以认为是是一个外比较器，有两种情况可以使用实现Comparator接口的方式：
> 1、一个对象不支持自己和自己比较（没有实现Comparable接口），但是又想对两个对象进行比较。  
> 2、一个对象实现了Comparable接口，但是开发者认为compareTo方法中的比较方式并不是自己想要的那种比较方式。


> Comparator接口里面有一个compare方法，方法有两个参数T o1和T o2，是泛型的表示方式，分别表示待比较的两个对象，方法返回值和> > Comparable接口一样是int，有三种情况：1、o1大于o2，返回正整数,2、o1等于o2，返回0,3、o1小于o3，返回负整数


```

	public interface Comparator<T> {
   		int compare(T o1, T o2);
    	boolean equals(Object obj);
	}
```

## 四、Java中的集合使用泛型有哪些好处
+ Java泛型（generics）

	是JDK5中引入的一个新特性，泛型提供了编译时类型安全监测机制，该机制允许我们在编译时检测到非法的类型数据结构；
	泛型的本质就是参数化类型，也就是所操作的数据类型被指定为一个参数；

+ 泛型的好处

> 1.类型安全。泛型的主要目的就是提高Java程序的类型安全。通过知道使用泛型定义的变量的类型限制，编译器可以在一个高得多的程度上验证类型假设。没有泛型，这些假设只能我们自己记或者代码注释；  
> 2.消除强制类型转换。泛型一个附带好处是，消除代码中许多强制类型的转换。减少代码出错率，更好阅读；  
> 3.潜在的性能收益。可以带来更好的优化可能。在泛型的初始实现中，编译器强制类型转换（没有泛型的话，程序员会指定这些强制类型转换，）插入生成的字节码中。但是更多类型信息可用于编译器这一事实，为以后的JVM可以带来更好的优化。由于泛型的实现方式，支持泛型几乎不需要JVM或类文件更改，所有工作都在编译器中完成，编译器生成的类没有泛型（和强制类型转换），只是来确保数据类型安全；

## 五、当一个集合被作为参数传递给一个函数时，如何才能确保函数不能修改它
在作为参数传递之前，可以使用Collections.unmodifiableCollection(Collection c)方法创建一个只读集合，这将确保改变集合的任何操作都会抛出UnsupportedOperationException。



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
> 2.ListIterator有add()方法，可以向List中添加对象，而Iterator不能
> 
> 3.ListIterator和Iterator都有hasNext()和next()方法，可以实现顺序向后遍历，但是ListIterator有hasPrevious()和previous()方法，可以实现逆向（顺序向前）遍历。Iterator就不可以。
> 
> 4.ListIterator可以定位当前的索引位置，nextIndex()和previousIndex()可以实现。Iterator没有此功能。
> 
> 5.都可实现删除对象，但是ListIterator可以实现对象的修改，set()方法可以实现。Iierator仅能遍历，不能修改。

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