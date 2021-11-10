## 1. Relationship between Iterator and Inhanced For

1.首先增强for循环和iterator遍历的效果是一样的，也就说
增强for循环的内部也就是调用iteratoer实现的，但是增强for循环 有些缺点，例如不能在增强循环里动态的删除集合内容，不能获取下标等。
2.ArrayList由于使用数组实现，因此下标明确，最好使用普通循环。
3.而对于 LinkedList 由于获取一个元素，要从头开始向后找，因此建议使用 增强for循环，也就是iterator。
以下例子证明第一点

```
①   public static void removeEventsDemo1(List<Integer> lst)
  {
     for (Integer x : lst)
        if (x % 2 == 0)
          lst.remove(x);
     

     System.out.println(lst); 

  }
```

```
②   public static void removeEventsDemo2(List<Integer> lst)
  {
     Iterator<Integer> itr = lst.iterator();
     while (itr.hasNext())
        if (itr.next() % 2 == 0)
          itr.remove();

     System.out.println(lst); 
  }
```

①在运行时抛出异常，②正常
原因分析：因为增强的for循环内部就是调用iterator实现的，在遍历的时候就将list转化为了迭代器，当迭代器被创建之后，如果从结构上对列表修改除非通过迭代器自身的remove、add方法，其他任何时间任何方式的修改，迭代器都会抛出ConcurrentModificationException异常。

## 2. 比较器compareTo和compare()的lambda用法

List的sort()方法通过改写compare()实现升序、降序

必须明确两点：

compare(Integer e1, Integer e2){}返回的只有-1，0，1这些具体值，函数自己并不会对list进行排序。对list排序是因为有sort()对返回的值进行处理，规则为**：1表示不交换位置，0表示相等时不交换，-1表示交换位置。**
compare(Integer e1, Integer e2){}中，**e1代表的是List容器中的后一个元素，e2代表的是List容器中的前一个元素。**

```
//实现Comparator进行降序排序  
  Collections.sort(list, new Comparator<Object>(){   
  	@Override   
  	public int compare(Object o1, Object o2) {
  		//升序排序  return ((Student) o1).getAge() - ((Student) o2).getAge();   
  		return ((Student) o2).getAge() - ((Student) o1).getAge();   
  	}  
  });
```




字符串的compareTo的用法：

返回参与比较的前后两个字符串的asc码的差值，如果两个字符串首字母不同，则该方法返回首字母的asc码的差值。

```
　　　　 String a1 = "a";
        String a2 = "c";        
        System.out.println(a1.compareTo(a2));//结果为-2
```

参与比较的两个字符串如果首字符相同，则比较下一个字符，直到有不同的为止，返回该不同的字符的asc码差值。

```
　　　　 String a1 = "aa";
        String a2 = "ad";        
        System.out.println(a1.compareTo(a2));//结果为-3
```

如果两个字符串不一样长，可以参与比较的字符又完全一样，则返回两个字符串的长度差值。

```
　　　　 String a1 = "aa";
        String a2 = "aa12345678";        
        System.out.println(a1.compareTo(a2));//结果为-8
```

Integer比较用compareTo()时：返回为正数表示a1>a2, 返回为负数表示a1<a2, 返回为0表示a1==a2。
    

```
  Integer x = 5;
      System.out.println(x.compareTo(3));  //1
      System.out.println(x.compareTo(5));  //0
      System.out.println(x.compareTo(8));  //-1

```

实际应用：
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

解答：通过在排序时传入一个自定义的 Comparator 实现，重新定义 String 列表内的排序方法，若拼接 s1 + s2 > s2 + s1，那么显然应该把 s2 在拼接时放在前面，以此类推，将整个 String 列表排序后再拼接起来。

比较list中每两两元素的组合，组合得到的数越小，越是将其放在前面，代码如下：

```
//保证nums排序顺序为：s1s2 < s2s1
public String minNumber(int[] nums) {
    List<String> list = new ArrayList<>(nums.length);
    for (int e : nums) {
        list.add(String.valueOf(e));
    }
    //s1+s2 < s2+s1, 则返回1
    //1表示不交换位置，0表示相等时不交换，-1表示交换位置
    list.sort((s1, s2) -> (s1 + s2).compareTo(s2 + s1));
    StringBuilder result = new StringBuilder();
    list.forEach(e -> result.append(e));
    return result.toString();
}
```



## 3. Why Comparator is a functional interface?

在了解这个问题之前，首先要知道什么是函数式接口

函数式接口(Functional Interface)就是接口里面只可以有一个抽象的方法，但是可以有多个非抽象方法。
在注解@FuctionalInterface（自动检测是否为函数式接口）的javadoc中如下说明

```
 Conceptually, a functional interface has exactly one abstract method.  Since {@linkplain java.lang.reflect.Method#isDefault() 
 default methods} have an implementation, they are not abstract.  If an interface declares an abstract method overriding one of the public methods of {@code java.lang.Object}, that also does <em>not</em> count toward the interface's abstract method count since any implementation of the interface will have an
 implementation from {@code java.lang.Object} or elsewhere.
```

意思就是：

```
**1.函数式接口只会有一个抽象方法**
**2.default方法不属于抽象方法**
**3.接口重写了Object的公共方法也不算入内**
```

所以虽然在Comparator看起来有两个抽象的方法。

```
int compare(T o1, T o2);
boolean equals(Object obj);
```

但是因为equals是Object的方法，所以不算抽象方法，所以Comparator是函数式接口。

版权声明：本文为CSDN博主「划船全靠浪i」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_47235839/article/details/120641273

## 4. Difference between Comparable & Comparator

Java 中为我们提供了两种比较机制：Comparable 和 Comparator，他们之间有什么区别呢？今天来了解一下。

### **Comparable 自然排序**

Comparable 在 java.lang 包下，是一个接口，内部只有一个方法 compareTo()：

```
public interface Comparable<T> {
    public int compareTo(T o);
```


Comparable 可以让实现它的类的对象进行比较，具体的比较规则是按照 compareTo 方法中的规则进行。这种顺序称为 自然顺序。

compareTo 方法的返回值有三种情况：

```
e1.compareTo(e2) > 0 即 e1 > e2
e1.compareTo(e2) = 0 即 e1 = e2
e1.compareTo(e2) < 0 即 e1 < e2
```

**注意：**

1.由于 null 不是一个类，也不是一个对象，因此在重写 compareTo 方法时应该注意 e.compareTo(null) 的情况，即使 e.equals(null) 返回 false，compareTo 方法也应该主动抛出一个空指针异常 NullPointerException。
2.Comparable 实现类重写 compareTo **方法时一般要求 e1.compareTo(e2) == 0 的结果要和 e1.equals(e2) 一致**。这样将来使用 SortedSet 等根据类的自然排序进行排序的集合容器时可以保证保存的数据的顺序和想象中一致。
有人可能好奇上面的第二点如果违反了会怎样呢？

**举个例子，如果你往一个 SortedSet 中先后添加两个对象 a 和 b，a b 满足 (!a.equals(b) && a.compareTo(b) == 0)，同时也没有另外指定个 Comparator，那当你添加完 a 再添加 b 时会添加失败返回 false, SortedSet 的 size 也不会增加，因为在 SortedSet 看来它们是相同的，而 SortedSet 中是不允许重复的。**

实际上所有实现了 Comparable 接口的 Java 核心类的结果都和 equlas 方法保持一致。 
实现了 Comparable 接口的 List 或则数组可以使用 Collections.sort() 或者 Arrays.sort() 方法进行排序。

实现了 Comparable 接口的对象才能够直接被用作 SortedMap (SortedSet) 的 key，要不然得在外边指定 Comparator 排序规则。

因此自己定义的类如果想要使用有序的集合类，需要实现 Comparable 接口，比如：


    public class BookBean implements Serializable, Comparable {
    private String name;
    private int count
    public BookBean(String name, int count) {
        this.name = name;
        this.count = count;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getCount() {
        return count;
    }
    
    public void setCount(int count) {
        this.count = count;
    }
    
    /**
     * 重写 equals
     * @param o
     * @return
     */
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof BookBean)) return false;
    
        BookBean bean = (BookBean) o;
    
        if (getCount() != bean.getCount()) return false;
        return getName().equals(bean.getName());
    
    }
    
    /**
     * 重写 hashCode 的计算方法
     * 根据所有属性进行 迭代计算，避免重复
     * 计算 hashCode 时 计算因子 31 见得很多，是一个质数，不能再被除
     * @return
     */
    @Override
    public int hashCode() {
        //调用 String 的 hashCode(), 唯一表示一个字符串内容
        int result = getName().hashCode();
        //乘以 31, 再加上 count
        result = 31 * result + getCount();
        return result;
    }
    
    @Override
    public String toString() {
        return "BookBean{" +
                "name='" + name + '\'' +
                ", count=" + count +
                '}';
    }
    
    /**
     * 当向 TreeSet 中添加 BookBean 时，会调用这个方法进行排序
     * @param another
     * @return
     */
    @Override
    public int compareTo(Object another) {
        if (another instanceof BookBean){
            BookBean anotherBook = (BookBean) another;
            int result;
    
            //比如这里按照书价排序
            result = getCount() - anotherBook.getCount();     
    
          //或者按照 String 的比较顺序
          //result = getName().compareTo(anotherBook.getName());
    
            if (result == 0){   //当书价一致时，再对比书名。 保证所有属性比较一遍
                result = getName().compareTo(anotherBook.getName());
            }
            return result;
        }
        // 一样就返回 0
        return 0;
    }
    上述代码还重写了 equlas(), hashCode() 方法，自定义的类将来可能会进行比较时，建议重写这些方法。
感谢 @li1019865596 指出，这里我想表达的是在有些场景下 equals 和 compareTo 结果要保持一致，这时候不重写 equals，使用 Object.equals 方法得到的结果会有问题，比如说 HashMap.put() 方法，会先调用 key 的 equals 方法进行比较，然后才调用 compareTo。

后面重写 compareTo 时，要判断某个相同时对比下一个属性，把所有属性都比较一次。

Comparable 接口属于 Java 集合框架的一部分。

### Comparator 定制排序

Comparator 在 java.util 包下，也是一个接口，JDK 1.8 以前只有两个方法：

public interface Comparator<T> {

    public int compare(T lhs, T rhs);
    
    public boolean equals(Object object);
}
JDK 1.8 以后又新增了很多方法：

基本上都是跟 Function 相关的，这里暂不介绍 1.8 新增的。

从上面内容可知使用自然排序需要类实现 Comparable，并且在内部重写 comparaTo 方法。

而 Comparator 则是在外部制定排序规则，然后作为排序策略参数传递给某些类，比如 Collections.sort(), Arrays.sort(), 或者一些内部有序的集合（比如 SortedSet，SortedMap 等）。

```
使用方式主要分三步：

创建一个 Comparator 接口的实现类，并赋值给一个对象 
在 compare 方法中针对自定义类写排序规则
将 Comparator 对象作为参数传递给 排序类的某个方法
向排序类中添加 compare 方法中使用的自定义类
```

举个例子：

        // 1.创建一个实现 Comparator 接口的对象
        Comparator comparator = new Comparator() {
            @Override
            public int compare(Object object1, Object object2) {
                if (object1 instanceof NewBookBean && object2 instanceof NewBookBean){
                    NewBookBean newBookBean = (NewBookBean) object1;
                    NewBookBean newBookBean1 = (NewBookBean) object2;
                    //具体比较方法参照 自然排序的 compareTo 方法，这里只举个栗子
                    return newBookBean.getCount() - newBookBean1.getCount();
                }
                return 0;
            }
        };
    
        //2.将此对象作为形参传递给 TreeSet 的构造器中
        TreeSet treeSet = new TreeSet(comparator);
    
        //3.向 TreeSet 中添加 步骤 1 中 compare 方法中设计的类的对象
        treeSet.add(new NewBookBean("A",34));
        treeSet.add(new NewBookBean("S",1));
        treeSet.add( new NewBookBean("V",46));
        treeSet.add( new NewBookBean("Q",26));

其实可以看到，Comparator 的使用是一种策略模式，不熟悉策略模式的同学可以点这里查看： 策略模式：网络小说的固定套路 了解。

排序类中持有一个 Comparator 接口的引用：

```
Comparator<? super K> comparator;
```

而我们可以传入各种自定义排序规则的 Comparator 实现类，对同样的类制定不同的排序策略。

### 总结

Java 中的两种排序方式：

**Comparable 自然排序。（实体类实现）**
**Comparator 是定制排序。（无法修改实体类时，直接在调用方创建）**
**同时存在时采用 Comparator（定制排序）的规则进行比较。**

**对于一些普通的数据类型（比如 String, Integer, Double…），它们默认实现了Comparable 接口，实现了 compareTo 方法，我们可以直接使用。**

**而对于一些自定义类，它们可能在不同情况下需要实现不同的比较策略，我们可以新创建 Comparator 接口，然后使用特定的 Comparator 实现进行比较。**

这就是 Comparable 和 Comparator 的区别。
————————————————
版权声明：本文为CSDN博主「拭心」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u011240877/article/details/53399019



## 5.Difference between Hashtable & HashMap

HashMap与Hashtable的区别是面试中经常遇到的一个问题。这个问题看似简单，但如果深究进去，也能了解到不少知识。本文对两者从来源，特性，算法等多个方面进行对比总结。力争多角度，全方位的展示二者的不同，做到此问题的终结版。

### 1 作者

Hashtable的作者：

![这里写图片描述](https://img-blog.csdn.net/20180306020620854?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

HashMap的作者：

![这里写图片描述](https://img-blog.csdn.net/20180306020640697?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


Hash Map的作者比Hashtable的作者多了著名顶顶的并发大神Doug Lea。他写了util.concurrent包。著有并发编程圣经Concurrent Programming in Java: Design Principles and Patterns 一书。他的个人主页： http://g.oswego.edu/

Josh Bloch 为领导了众多Java平台特性的设计和实现，其中包括Java Collection框架、java.math包以及assert机制。著有 Effective Java 一书。

Arthur van Hoff最早任职于硅谷的Sun Microsystems公司，从事Java程序语言的早期开发工作。设计并实现了JDK 1.0的许多方面，包括Java编译器、Java调试器、许多标准Java类以及HotJava浏览器。随后创立了多家成功的企业，其中包括Marimba（1999年IPO）、Strangeberry（后被TiVo收购）、ZING（后被Dell收购）和Ellerdale（后被Flipboard收购）。Java命名来源有这么一种说法，来源于开发人员名字的组合：James Gosling、Arthur Van Hoff和Andy Bechtolsheim首字母的缩写。

Neal Gafter是Java SE 4和5语言增强的主要设计者和实现者，他的Java闭包实现赢得了OpenJDK创新者挑战赛的大奖。他也在继续参与SE 7和8的语言发展。之前Neal在为Google的在线日历工作，也曾经是C++标准委员会的一员，并曾在Sun微系统公司，MicroTec研究院和德州仪器领导开发C和C++编译器。如今Neal在微软开发.NET平台编程语言。Neal是《Java Puzzlers：Traps, Pitfalls and Corner Cases》（Addison Wesley，2005）一书的合作者。他拥有罗彻斯特大学计算机科学的博士学位。

可见这些作者都是java乃至整个it领域大名鼎鼎的人物。也只有这些大师级人物才能写出HashMap这么大道至简的数据类型了。

### 2 产生时间

Hashtable是java一开始发布时就提供的键值映射的数据结构，而HashMap产生于JDK1.2。虽然Hashtable比HashMap出现的早一些，但是现在Hashtable基本上已经被弃用了。而HashMap已经成为应用最为广泛的一种数据类型了。造成这样的原因一方面是因为Hashtable是线程安全的，效率比较低。另一方面可能是因为Hashtable没有遵循驼峰命名法吧。。。

### 3 继承的父类不同

HashMap和Hashtable不仅作者不同，而且连父类也是不一样的。HashMap是继承自AbstractMap类，而HashTable是继承自Dictionary类。不过它们都实现了同时实现了map、Cloneable（可复制）、Serializable（可序列化）这三个接口



![这里写图片描述](https://img-blog.csdn.net/20180306020714182?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Dictionary类是一个已经被废弃的类（见其源码中的注释）。父类都被废弃，自然而然也没人用它的子类Hashtable了。

![这里写图片描述](https://img-blog.csdn.net/20180306020658482?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

NOTE: This class is obsolete. New implementations should
implement the Map interface, rather than extending this class.

### 4 对外提供的接口不同

Hashtable比HashMap多提供了elments() 和contains() 两个方法。

elments() 方法继承自Hashtable的父类Dictionnary。elements() 方法用于返回此Hashtable中的value的枚举。

contains()方法判断该Hashtable是否包含传入的value。它的作用与containsValue()一致。事实上，contansValue() 就只是调用了一下contains() 方法。

![这里写图片描述](https://img-blog.csdn.net/20180306020727138?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 5 对Null key 和Null value的支持不同

Hashtable既不支持Null key也不支持Null value。Hashtable的put()方法的注释中有说明。

![这里写图片描述](https://img-blog.csdn.net/20180306020744222?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


当key为Null时，调用put() 方法，运行到下面这一步就会抛出空指针异常。因为拿一个Null值去调用方法了。

![这里写图片描述](https://img-blog.csdn.net/20180306020754168?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

当value为null值时，Hashtable对其做了限制，运行到下面这步也会抛出空指针异常。

![这里写图片描述](https://img-blog.csdn.net/20180306020802695?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。

### 6 线程安全性不同

Hashtable是线程安全的，它的每个方法中都加入了Synchronize方法。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步

HashMap不是线程安全的，在多线程并发的环境下，可能会产生死锁等问题。具体的原因在下一篇文章中会详细进行分析。使用HashMap时就必须要自己增加同步处理，

虽然HashMap不是线程安全的，但是它的效率会比Hashtable要好很多。这样设计是合理的。在我们的日常使用当中，大部分时间是单线程操作的。HashMap把这部分操作解放出来了。当需要多线程操作的时候可以使用线程安全的ConcurrentHashMap。ConcurrentHashMap虽然也是线程安全的，但是它的效率比Hashtable要高好多倍。因为ConcurrentHashMap使用了分段锁，并不对整个数据进行锁定。

### 7 遍历方式的内部实现上不同

Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。

HashMap的Iterator是fail-fast迭代器。当有其它线程改变了HashMap的结构（增加，删除，修改元素），将会抛出ConcurrentModificationException。不过，通过Iterator的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。

JDK8之前的版本中，Hashtable是没有fast-fail机制的。在JDK8及以后的版本中 ，HashTable也是使用fast-fail的， 源码如下：

![这里写图片描述](https://img-blog.csdn.net/20180306020819368?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

modCount的使用类似于并发编程中的CAS（Compare and Swap）技术。我们可以看到这个方法中，每次在发生增删改的时候都会出现modCount++的动作。而modcount可以理解为是当前hashtable的状态。每发生一次操作，状态就向前走一步。设置这个状态，主要是由于hashtable等容器类在迭代时，判断数据是否过时时使用的。尽管hashtable采用了原生的同步锁来保护数据安全。但是在出现迭代数据的时候，则无法保证边迭代，边正确操作。于是使用这个值来标记状态。一旦在迭代的过程中状态发生了改变，则会快速抛出一个异常，终止迭代行为。

### 8 初始容量大小和每次扩充容量大小的不同

Hashtable默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。

创建时，如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小。也就是说Hashtable会尽量使用素数、奇数。而HashMap则总是使用2的幂作为哈希表的大小。

之所以会有这样的不同，是因为Hashtable和HashMap设计时的侧重点不同。Hashtable的侧重点是哈希的结果更加均匀，使得哈希冲突减少。当哈希表的大小为素数时，简单的取模哈希的结果会更加均匀。而HashMap则更加关注hash的计算效率问题。在取模计算时，如果模数是2的幂，那么我们可以直接使用位运算来得到结果，效率要大大高于做除法。HashMap为了加快hash的速度，将哈希表的大小固定为了2的幂。当然这引入了哈希分布不均匀的问题，所以HashMap为解决这问题，又对hash算法做了一些改动。这从而导致了Hashtable和HashMap的计算hash值的方法不同

### 9 计算hash值的方法不同

为了得到元素的位置，首先需要根据元素的 KEY计算出一个hash值，然后再用这个hash值来计算得到最终的位置。

Hashtable直接使用对象的hashCode。hashCode是JDK根据对象的地址或者字符串或者数字算出来的int类型的数值。然后再使用除留余数发来获得最终的位置。

![这里写图片描述](https://img-blog.csdn.net/20180306020836521?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


Hashtable在计算元素的位置时需要进行一次除法运算，而除法运算是比较耗时的。
HashMap为了提高计算效率，将哈希表的大小固定为了2的幂，这样在取模预算时，不需要做除法，只需要做位运算。位运算比除法的效率要高很多。

HashMap的效率虽然提高了，但是hash冲突却也增加了。因为它得出的hash值的低位相同的概率比较高，而计算位运算

为了解决这个问题，HashMap重新根据hashcode计算hash值后，又对hash值做了一些运算来打散数据。使得取得的位置更加分散，从而减少了hash冲突。当然了，为了高效，HashMap只做了一些简单的位处理。从而不至于把使用2 的幂次方带来的效率提升给抵消掉。

![这里写图片描述](https://img-blog.csdn.net/20180306020849468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

附上关于这个问题的说明：
Joshua Bloch: The downside of using a power-of-two is that the resulting hash table is very sensitive to the quality of the hash function (hashCode). It is imperative that any change in the input must affect the low order bits of the hash value. (Ideally, it should affect all bits of the hash value with equal likelihood.) Because we have no assurance that this is true, we put in a secondary (or “defensive”) hash function when we switched to the power-of-two hash table. This hash function is applied to the results of hashCode before masking off the low order bits. Its job is to scatter the information over all the bits, and in particular, into the low order bits. Of course it has to run very fast, or you lose the benefit of switching to the power-of-two-sized table. The original secondary hash function in 1.4 turned out to be insufficient. We knew that this was a theoretical possibility, but we thought that it didn’t affect any practical data sets. We were wrong. The replacement secondary hash function (which I developed with the aid of a computer) has strong statistical properties that pretty much guarantee good bucket distribution.

————————————————
版权声明：本文为CSDN博主「hahahaha233」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/wangxing233/article/details/79452946





## 6. Collection Frame

![img](https://sethxiong.oss-cn-shenzhen.aliyuncs.com/uPic/collections%20framework%20overview.png)

![image-20211110122536192](/Users/dyq/Library/Application Support/typora-user-images/image-20211110122536192.png)

![image-20211110122558664](/Users/dyq/Library/Application Support/typora-user-images/image-20211110122558664.png)

![image-20211110122611878](/Users/dyq/Library/Application Support/typora-user-images/image-20211110122611878.png)

![image-20211110122646598](/Users/dyq/Library/Application Support/typora-user-images/image-20211110122646598.png)

![image-20211110122655030](/Users/dyq/Library/Application Support/typora-user-images/image-20211110122655030.png)