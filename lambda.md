# 深入理解Lambda表达式

## 概述

今天在Pluralsight看了一个讲Java Lambda 表达式的视频教程，觉得很好，自己研究并记录分享一下以飨读者。

因为Java8已经出来好久了，Lambda已经被大量使用了，所以这里只是分享一下对其的思考和总结,不准备过多讲解其用法，目的是使我们对其有更加深刻的理解。

## 匿名类到Lambda表达式

我们知道，只有函数接口才可以使用Lambda表达式。

```
函数接口：只有一个abstract的方法的接口
```

那我们怎么将实现了函数接口的匿名类转换成Lambda表达式呢？我们以code来说话：

### 示例1，抽象方法无入参，无返回值

我们以Runnable函数接口为例

```
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```


匿名类：

```
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("hello thread");
    }
};
new Thread(runable).start();
```

Lambda 表达式

将方抽象方法括号及其中参数拷贝出来并加上->
()->
将抽象方法方法体拷贝出来，如果其中只有一句代码则{} 可以省略

```
	()->{ System.out.println("hello thread");}

    省略括号后
    ()->System.out.println("hello thread")
    
    new Thread(() -> System.out.println("hello thread")).start();    


```



### 示例2，抽象方法带入参，带返回值的情形

例如用于比较的Comparator函数接口

```
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    ...
 }
```


匿名类

```
 Comparator<Integer> comparator = new Comparator<Integer>() {
     @Override
     public int compare(Integer o1, Integer o2) {
         return Integer.compare(o2, o1);
     }
 };
```


转换为Lambda表达式

将方抽象方法括号及其中参数拷贝出来并加上->
(Integer o1, Integer o2) ->
将抽象方法方法体拷贝出来放在->后面

```
(Integer o1, Integer o2) ->`{
    return Integer.compare(o2, o1);
}
```

简化
参数类型可以省略，如果方法体只有一句代码则{}和 return 关键字可以省略

```
(o1, o2)-> Integer.compare(o2, o1)
```


不是说越简单越好，如果你发现带上类型可读性更好，那就带上，完全没有问题。

不知道你们平时在开发中是否遇到过特别复杂函数接口，手写lambda表达式变得很困难，不好理解。此时反而利用IDE的提示功能直接new匿名类对象反而更方便，然后写完了再使用IDE一键将其转换为Lambda表达式。

## Method reference（方法引用）

Lambda的一种简写方式，无他。其可以替换单方法的Lambda表达式。
详情请参考：秒懂Java之方法引用（method reference）

**灵魂拷问**

**Lambda 有类型吗？有的话是什么类型？**

**Lambda可以赋值给变量吗？可以当方法的入参和返回值吗？**

**Lambda是对象吗？有的话我们可以在代码中引用它吗？**

第一问：
Java是强类型语言，在Java中任何事物都有类型，Lambda也不例外，它的类型就是其对应的函数接口。

第二问：
Lambda可以赋值给变量，而且可以作为方法的参数及返回值

第三问：
Lambda是个对象吗？这块情况比较就比较复杂了，我只能将自己的调查和理解呈上，至于更深刻的机制需要你去研究。不过我这里说的，对于普通程序员已经足够了，我个人觉得这部分很有意思，如果你也想知道Java编译器到底如何处理Lambda表达式这种语法糖的话，应该接着往下看。

首先我们来看一段代码

```
package top.ss007;
...
public class Student {
    Runnable runnable = () -> {
    };
    Comparator<Integer> comparator = (o1, o2) -> 1;

    Runnable runnableAnon = new Runnable() {
        @Override
        public void run() {
        }
    };

}
```


定义了一个Student类，里面声明了3个field，两个赋值为Lambda表达式，一个赋值为匿名内部类对象。

我们使用Java编译器javac将其编译为class文件

```
javac -g  Student.java


-g 保留所有调试信息
```

执行上述命令后生成了Student.class和Student$1.class两个文件

然后我们使用字节码查看器ByteCodeViewer查看一下这两个文件，从命名上就可以看出Student$1.class是Student类的内部类.

我们先看使用匿名内部类实现函数接口的部分：

```
Runnable runnableAnon = new Runnable() {
    @Override
    public void run() {
    }
};
```


我们知道Javac 会为其生成一个实现了Runnable接口的Student的内部类，这块我们就不看字节码了直接看生成的代码类

```
class Student$1 implements Runnable {
   // $FF: synthetic field
   final Student this$0;

   Student$1(Student this$0) {
      this.this$0 = this$0;
   }

   public void run() {
   }
}
```


可见，非静态的内部类是会持有其包含类的一个引用的。

接下来看一下其具体的赋值字节码

```
public Student() { // <init> //()V
    <localVar:index=0 , name=this , desc=Ltop/ss007/Student;, sig=null, start=L1, end=L2>

    L1 {
        aload0 // reference to self
        invokespecial java/lang/Object.<init>()V
    }

...

    L5 {
        aload0 // reference to self
        new top/ss007/Student$1
        dup
        aload0 // reference to self
        invokespecial top/ss007/Student$1.<init>(Ltop/ss007/Student;)V
        putfield top/ss007/Student.runnableAnon:java.lang.Runnable
        return
    }

}

上面是Student类的构造函数的字节码
```

L1{} 里面是调用Object的构造函数的代码，又一次印证了Object是所有类型的基类。

L5{} 的代码是我们要关注的，通过new top/ss007/Student$1 new 一个Student$1的对象，初始化后赋值给field runableAnon.

通过上面的调查我们已经清楚了javac是如何处理使用匿名内部类实现函数接口的方式，接下来让我看一下Lambda表达式是如何处理的。

查看Student的字节码文件可以发现如下两段代码

 

```
private static synthetic lambda$new$1(java.lang.Integer arg0, java.lang.Integer arg1) { //(Ljava/lang/Integer;Ljava/lang/Integer;)I
     <localVar:index=0 , name=o1 , desc=Ljava/lang/Integer;, sig=null, start=L1, end=L2>
     <localVar:index=1 , name=o2 , desc=Ljava/lang/Integer;, sig=null, start=L1, end=L2>

     L1 {
         iconst_1
         ireturn
     }
     L2 {
     }

 }

 private static synthetic lambda$new$0() { //()V
     L1 {
         return
     }
 }
```



其中有两个使用了synthetic的方法，说明这两个方法是编译器帮我们生成的。
lambda$new$0 对应的是

```
()->{}
```


lambda$new$1 对应的是

```
(o1, o2) -> 1;
```


赋值语句位于Student的构造函数中，如下所示：

```
public Student() { // <init> //()V
    <localVar:index=0 , name=this , desc=Ltop/ss007/Student;, sig=null, start=L1, end=L2>
...
    L3 {
        aload0 // reference to self
        invokedynamic java/lang/invoke/LambdaMetafactory.metafactory(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite; : run()Ljava/lang/Runnable; ()V top/ss007/Student.lambda$new$0()V (6) ()V
        putfield top/ss007/Student.runnable:java.lang.Runnable
    }
    L4 {
        aload0 // reference to self
        invokedynamic java/lang/invoke/LambdaMetafactory.metafactory(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite; : compare()Ljava/util/Comparator; (Ljava/lang/Object;Ljava/lang/Object;)I top/ss007/Student.lambda$new$1(Ljava/lang/Integer;Ljava/lang/Integer;)I (6) (Ljava/lang/Integer;Ljava/lang/Integer;)I
        putfield top/ss007/Student.comparator:java.util.Comparator
    }
...
}
```


L3{} 块内是

```
Runnable runnable = () -> {   };

```


的处理代码

L4{} 块内是

```
Comparator<Integer> comparator = (o1, o2) -> 1;
```


的处理代码

其中最为关键的就是

```
invokedynamic java/lang/invoke/LambdaMetafactory.metafactory(
		Ljava/lang/invoke/MethodHandles$Lookup;
		Ljava/lang/String;
		Ljava/lang/invoke/MethodType;
		Ljava/lang/invoke/MethodType;
		Ljava/lang/invoke/MethodHandle;
		Ljava/lang/invoke/MethodType;)
Ljava/lang/invoke/CallSite; : run()Ljava/lang/Runnable; ()V top/ss007/Student.lambda$new$0()V (6) ()V
```


上面的代码的作用是什么呢？
总的来说，其是JVM在Runtime时创建() -> { } 对应的类，即在运行时创建一个实现了Runable接口的类(字节码)。

我们看到此处使用了invokedynamic ,只是为了动态的生成其对应类的字节码，但是Lambda方法的执行仍然使用的是 invokevirtual或者invokeinterface, 在首次调用时，生成对应类的字节码，然后就缓存起来，下次使用缓存。

那么是时候回答一下开头的问题了：每一个Lambda表达式是否对应一个对象？
答案是：是的！但是，注意但是，每个Lambda是对应一个对象，但是有可能出现多对一的情况。
例如Lambda1，Lambda2 结构相同，那么他们在JVM中就有可能对应同一个对象

## 总结

写一篇不误人子弟的文章实在是太费时间了，因为你要不断的确认自己写的是不是真的可以实现，而不是自己的想当然。

技术真的是没有止境，应尽早确定自己的方向，少年加油。
————————————————
版权声明：本文为CSDN博主「ShuSheng007」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/ShuSheng0007/article/details/106878442



# Method reference 详解

## 概述

如果你对将Lambda表达式转换成对应的方法引用有疑惑的话，本文你值得一看。

方法引用（MethodReference）是Lambda表达式的另一种格式，在某些场景下可以提高代码的可读性，那么如何将一个Lambda表达式替换成MethodReference呢？有的同学说了，可以使用IDE协助转换，我只能说你太机智了，那这篇文章不是为你准备的。

## 使用条件

只可以替换单方法的Lambda表达式

什么意思呢 ？

例如下面这个Lambda表达式就不可以使用方法引用替换，因为其不是单方法的，有好几行呢。

```
Predicate<Integer> p2 = integer -> {
     System.out.println("中国股市是吃屎长大的");
     return TestUtil.isBiggerThan3(integer);
 };

```


下面这个就可以使用方法引用替换了



```
Predicate<Integer> p2 = integer -> TestUtil.isBiggerThan3(integer);


```



## 使用场景

当使用方法引用替换Lambda表达式具有更好的可读性时，考虑使用。

如何使用

曾几何时，我对方法引用的理解很模糊，一般是使用IDE协助转换，但是转换后的写法很多我都不明白：

Lambda的参数哪去了？
为什么::前面有的是类名称，有的是实例对象呢？
不是只有静态方法::前面才使用类名吗，怎么有的实例方法也使用类名啊？
为什么 ClassName::new 有时要求无参构造器，有时又要求有参构造器呢？构造器参数由什么决定呢？

结论其实显而易见了，我对方法引用的理解根本就是一团浆糊！如果你也有上面的疑问，也许应该接着往下看。

## 方法引用的类型

解决纷繁复杂信息的最好方式就是分类，这里也不例外。
方法引用可以分为分四类，只要掌握了类型区别一切就变得易如反掌了。
为行文方便，这里先列出要使用的类：

TestUtil里面有一个静态方法，一个实例方法。Student是一个普通实体类，具体如下代码所示

```
public class MethodReference {
	...
  
  //示例类
  public static class TestUtil {
        public static boolean isBiggerThan3(int input) {
            return input > 3;
        }

        public void printDetail(Student student) {
            System.out.println(student.toString());
        }
    }

  public static class Student {
        private String name;
        private int age;

        public Student(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getStatus(String thing) {
            return String.format("%d岁的%s正在%s", age, name, thing);
        }

        @Override
        public String toString() {
            return "Student{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}



```



### 调用类的静态方法

Lambda表达式的那个单方法是某个类的静态方法

有如下格式，args是参数，可以是多个，例如(a1,a2,a3)

lambda：

```
(args) -> Class.staticMethod(args)


```

method reference

```
Class::staticMethod


```

符合上面形式的调用，不管有多少参数，都省略掉，编译器自动会帮我们传入

实例：

```
public void testStaticMethodRef() {
    //匿名内部类形式
    Predicate<Integer> p1 = new Predicate<Integer>() {
        @Override
        public boolean test(Integer integer) {
            return TestUtil.isBiggerThan3(integer);
        }
    };
    //lambda表达式形式
    Predicate<Integer> p2 = integer -> TestUtil.isBiggerThan3(integer);
    //MethodReference形式
    Predicate<Integer> p3 = TestUtil::isBiggerThan3;

    Stream.of(1, 2, 3, 4, 5).filter(p3).forEach(System.out::println);
}


```


其中isBiggerThan3 是TestUtil类的static方法。从上面的代码你可以清晰的看到，方法从匿名类到Lambda再到方法引用的演变。

###  调用传入的实例的方法

lambda：

```
(obj, args) -> obj.instanceMethod(args)
```


method reference

```
ObjectType::instanceMethod
```


看到我们lambda的入参obj了吗？它是一个类型，假设为ObjectType，的实例对象。然后再看lambda表达式，是在调用此实例obj的方法。这种类型的lambda就可以写成上面的形式了，看起来和静态方法那个一样。

我们来举个栗子：

```
public void testInstanceMethodRef1() {
   //匿名类
    BiFunction<Student, String, String> f1 = new BiFunction<Student, String, String>() {
        @Override
        public String apply(Student student, String s) {
            return student.getStatus(s);
        }
    };
    
    //lambda
    BiFunction<Student, String, String> f2 = (student, s) -> student.getStatus(s);
    
	//method reference
    BiFunction<Student, String, String> f3 = Student::getStatus;

    System.out.println(getStudentStatus(new Student("erGouWang", 18), "study", f3));
}
private String getStudentStatus(Student student, String action, BiFunction<Student, String, String> biFunction) {
    return biFunction.apply(student, action);
}


```

### 调用已经存在的实例的方法

lambda：

```
(args) -> obj.instanceMethod(args)
```


method reference

```
obj::instanceMethod
```


我们观察一下我们lambda表达式，发现obj对象不是当做参数传入的，而是已经存在的，所以写成方法引用时就是实例::方法.

举个栗子：

```
public void testInstanceMethodRef2() {
    TestUtil utilObj = new TestUtil();
	//匿名类
    Consumer<Student> c1 = new Consumer<Student>() {
        @Override
        public void accept(Student student) {
            utilObj.printDetail(student);
        }
    };
    //Lambda表达式
    Consumer<Student> c2 = student -> utilObj.printDetail(student);
	//方法引用
    Consumer<Student> c3 = utilObj::printDetail;
	//使用
    consumeStudent(new Student("erGouWang", 18), c3);
}

private void consumeStudent(Student student, Consumer<Student> consumer) {
    consumer.accept(student);
}


```

可见utilObj对象是我们提前new出来的，是已经存在了的对象，不是Lambda的入参。

### 调用类的构造函数

lambda：

```
(args) -> new ClassName(args)
```


method reference

```
ClassName::new
```


当lambda中的单方法是调用某个类的构造函数，我们就可以将其写成如上形式的方法引用

举个栗子

```
public void testConstructorMethodRef() {
    BiFunction<String, Integer, Student> s1 = new BiFunction<String, Integer, Student>() {
        @Override
        public Student apply(String name, Integer age) {
            return new Student(name, age);
        }
    };
	//lambda表达式
    BiFunction<String, Integer, Student> s2 = (name, age) -> new Student(name, age);
	//对应的方法引用
    BiFunction<String, Integer, Student> s3 = Student::new;
	//使用
    System.out.println(getStudent("cuiHuaNiu", 20, s3).toString());
}

private Student getStudent(String name, int age, BiFunction<String, Integer, Student> biFunction) {
    return biFunction.apply(name, age);
}



```

上面代码值得注意的就是，Student类必须有一个与lambda入参相匹配的构造函数。例如此例中，(name, age) -> new Student(name, age); 需要两个入参的构造函数，为什么呢？

因为我们的lambda表达式的类型是 BiFunction，而其正是通过两个入参来构建一个返回的。其签名如下：

```
@FunctionalInterface
public interface BiFunction<T, U, R> {
	  R apply(T t, U u);
	  ...
}


```


通过入参(t,u)来生成R类型的一个值。

我们可以写一个如下的方法引用

```
Function<String, Student> s4 = Student::new;
```


但是IDE就会报错，提示我们的Student类没有对应的构造函数，我们必须添加一个如下签名的构造函数才可以

```
public Student(String name) {
    this.name = name;
}


```



## 总结

熟悉了以上四种类型后，方法引用再也难不住你了！留个作业：

```
Consumer<String> consumer1 = new Consumer<String>() {
    @Override
    public void accept(String s) {
        System.out.println(s);
    }
};
//lambda表达式
Consumer<String> consumer2 = ;
//方法引用
Consumer<String> consumer3 = ;
```


上面代码中的consumer2和consumer3分别是什么呢？其属于方法引用的哪一类呢？请在评论区提交答案。

古之立大事者，不惟有超世之才，亦必有坚忍不拔之志。——苏轼

【版权申明】非商业目的注明出处可自由转载
博文地址：https://blog.csdn.net/ShuSheng0007/article/details/107562812
出自：shusheng007