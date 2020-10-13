
> [!TIP]
> 在不确定方法需要处理的对象的数量时可以使用可变长参数，会使得方法调用更简单，无需手动创建数组 new T[]{…}


> [!DANGER]
> 不定长参数在使用数组传参时，一定要注意**不能是基本类型数据的数组**，只能是**对象类型数据的数组**，比如基本数据类型的包装类。

## 概念
Variable Argument (Varargs)，不定长参数，或者叫可变参数，是 Java 5 中提供的，允许中调用方法时传入不定长度的参数。不定长参数是 Java 的一个语法糖，本质上还是基于数组的实现。

```java
void method1(String... args);
//等同于
void method2(String[] args);
```

```java
//方法签名 
([Ljava/lang/String;)V // public void method1(String[] args)
```

## 定义方法

在定义方法时，在最后一个形参后加上 **...** ，就表示该形参可以接收多个参数值，多个参数值会被当作数组传入。

> [!WARNING]
> - 可变参数只能作为最后一个参数。
> - 函数最多只能有一个可变参数。
> - Java 的可变参数，会被编译器编译成一个数组。
> - 变长参数在编译为字节码后，在方法签名中就是以数组形态出现的。这两个方法的签名是一致的，不能作为方法的重载。如果同时出现，是不能编译通过的。可变参数可以兼容数组，反之则不成立。

```java
void foo(String... args){};
void foo(String[] args){};
```
上面代码上编译不通过的。

- 使用方式

```java
public void foo(String...varargs){}
foo("arg1", "arg2", "arg3");
//上述过程和下面的调用是等价的
foo(new String[]{"arg1", "arg2", "arg3"});
```

## 方法重载

> [!TIP]
> 发生与不定长参数重载时，会优先匹配固定参数。

调用一个被重载的方法时，如果此调用既能够和固定参数的重载方法匹配，也能够与可变长参数的重载方法匹配，则选择固定参数的方法

```java
public class Varargs {
    public static void test(String... args) {
        System.out.println("version 1");
    }
    public static void test(String arg1, String arg2) {
        System.out.println("version 2");
    }
    public static void main(String[] args) {
        test("a","b");//version 2 优先匹配固定参数的重载方法
        test();//version 1
    }
}
```

> [!TIP]
> 匹配到多个可变参数时会报错

```java
public class Varargs {
    public static void test(String... args) {
        System.out.println("version 1");
    }
    public static void test(String arg1, String... arg2) {
        System.out.println("version 2");
    }
    public static void main(String[] args) {
        test("a","b");//Compile error
    }
}
```

> [!DANGER]
> 别让 null 值和空值威胁到变长方法

```java
public class Client {  
     public void methodA(String str,Integer... is){       
     }  

     public void methodA(String str,String... strs){          
     }  

     public static void main(String[] args) {  
           Client client = new Client();  
           client.methodA("China", 0);  
           client.methodA("China", "People");  
           client.methodA("China");  //compile error
           client.methodA("China",null);  //compile error
     }  
}
```
可修改如下:

```java
public static void main(String[] args) {  
     Client client = new Client();  
     String[] strs = null;  
     client.methodA("China",strs);  
}
```

让编译器知道这个null值是String类型的，编译即可顺利通过，也就减少了错误的发生。

## 方法重写

> [!DANGER]
> 避免带有变长参数的方法重载

即便编译器可以按照优先匹配固定参数的方式确定具体的调用方法，但在阅读代码的依然容易掉入陷阱。要慎重考虑变长参数的方法重载。

```java
public class VarArgsTest2 {
    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        // 向上转型
        Base base = new Sub();
        base.print("hello");
        // 不转型
        Sub sub = new Sub();
        sub.print("hello");//compile error,Required type: String[]
    }
}
// 基类
static class Base {
    void print(String... args) {
        System.out.println("Base......test");
    }
}
// 子类，覆写父类方法
static class Sub extends Base {
    @Override
    void print(String[] args) {
        System.out.println("Sub......test");
    }
}
```

base 对象把子类对象 sub 做了向上转型，形参列表是由父类决定的，当然能通过。而看看子类直接调用的情况，这时编译器看到子类覆写了父类的 print 方法，
因此肯定使用子类重新定义的 print 方法，尽管参数列表不匹配也不会跑到父类再去匹配下，因为找到了就不再找了，因此有了类型不匹配的错误。

## 会出现的问题

使用 Object… 作为变长参数：

```java
public void foo(Object... args) {
    System.out.println(args.length);
}

foo(new String[]{"arg1", "arg2", "arg3"}); //3
foo(100, new String[]{"arg1", "arg1"}); //2

foo(new Integer[]{1, 2, 3}); //3
foo(100, new Integer[]{1, 2, 3}); //2
foo(1, 2, 3); //3
foo(new int[]{1, 2, 3}); //1
```

int[] 无法转型为 Object[], 因而被当作一个单纯的数组对象 ; Integer[] 可以转型为 Object[], 可以作为一个对象数组。



