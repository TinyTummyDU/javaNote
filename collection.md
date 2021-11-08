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