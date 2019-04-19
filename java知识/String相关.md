# String相关
## String可变么？

String是不可变对象，即对象一旦生成，就不能被更改。对String对象的改变会引发新的String对象的生成。每当我们创建字符串常量时，JVM会首先检查字符串常量池，如果该字符串已经存在常量池中，那么就直接返回常量池中的实例引用。如果字符串不存在常量池中，就会实例化该字符串并且将其放到常量池中。由于String字符串的不可变性我们可以十分肯定常量池中一定不存在两个相同的字符串。

那么来看一段代码：

```
String a = "abc";
String b = "abc";
String c = new String("abc");
```

A、b和字面上的abc都是指向JVM字符串常量池中的"abc"对象，他们指向同一个对象。而new关键字一定会产生一个对象abc（注意这个abc和上面的abc不同），同时这个对象是存储在堆中。
所以上面应该产生了两个对象：保存在栈中的c和保存堆中abc。但是在Java中根本就不存在两个完全一模一样的字符串对象。故堆中的abc应该是引用字符串常量池中abc。所以c、abc、池abc的关系应该是：c---> abc--->池abc。




## 你真的了解String的拼接么？

提到String拼接，有java基础的人都知道如下方法：

1. 加号 “+”
2. String contact() 方法
3. StringBuffer append() 方法
4. StringBuilder append() 方法

区别：StringBuffer是线程安全的类。StringBuild不是线程安全的类，在单线程中性能要比StringBuffrer高。

那么，问题来了其中的“+”是用什么方法实现的？

有过了解到人可能知道，是StringBuffer的append()方法。
即执行一次字符串“+”,相当于 str = new StringBuilder(str).append("a").toString();

那么问题又来了，怎么证明呢？

这就要用到javap命令了，此命令能反编译class文件，也可以查看字节码。具体方法如下：


```
javac Test.java
javap -verbose Test
```

JDK9之前：

```
0: ldc           #16                 // String www
       2: astore_1
       3: iconst_0
       4: istore_2
       5: goto          30
       8: new           #18                 // class java/lang/StringBuilder
      11: dup
      12: aload_1
      13: invokestatic  #20                 // Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
      16: invokespecial #26                 // Method java/lang/StringBuilder."":(Ljava/lang/String;)V
      19: iload_2
      20: invokevirtual #29                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      23: invokevirtual #33                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      26: astore_1
      27: iinc          2, 1
      30: iload_2
      31: bipush        10
      33: if_icmplt     8
      36: return
```

可以很明显看到是用StringBuffer实现。

JDK10中得到如下结果：

```
 		  0: ldc           #2                  // String abc
         2: astore_1
         3: ldc           #3                  // String 123
         5: astore_2
         6: aload_1
         7: aload_2
         8: invokedynamic #4,  0              // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
        13: astore_3
        14: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
        17: aload_3
        18: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        21: return
```




