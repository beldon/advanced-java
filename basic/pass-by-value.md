> [!TIP]
> Java is always **pass-by-value**. <br/>
> Java中的传递，是值传递，而这个值，实际上是对象的引用。


## 概念

- 值传递（pass by value）  

在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。

- 引用传递（pass by reference）  

在调用函数时将实际参数的地址直接传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。


- 共享传递

是指在调用函数时，传递给函数的是实参的地址的拷贝（如果实参在栈中，则直接拷贝该值）。在函数内部对参数进行操作时，需要先拷贝的地址寻找到具体的值，再进行操作。如果该值在栈中，那么因为是直接拷贝的值，所以函数内部对参数进行操作不会对外部变量产生影响。如果原来拷贝的是原值在堆中的地址，那么需要先根据该地址找到堆中对应的位置，再进行操作。因为传递的是地址的拷贝所以函数内对值的操作对外部变量是可见的。

- 形式参数

是在定义函数名和函数体的时候使用的参数,目的是用来接收调用该函数时传入的参数。

- 实际参数

在调用有参函数时，主调函数和被调函数之间有数据传递关系。在主调函数中调用一个函数时，函数名后面括号中的参数称为“实际参数”。

> [!TIP]
> 实际参数是调用有参方法的时候真正传递的内容，而形式参数是用于接收实参内容的参数。

```java
public static void main(String[] args) {
    ParamTest pt = new ParamTest();
    pt.sout("demo");//实际参数为 demo
}
public void sout(String name) { //形式参数为 name
    System.out.println(name);
}
```


[Is Java “pass-by-reference” or “pass-by-value”?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)


[Java 到底是值传递还是引用传递？](https://www.zhihu.com/question/31203609)