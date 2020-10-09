### 什么是注解

Java官方文档上说，注解是元数据的一种形式，它提供不属于程序一部分的数据，注解对被注解的代码没有直接的影响。
**准确上说，注解只不过是一种特殊的注释而已，如果没有解析它的代码，它可能连注释都不如。**

### 主要用途

注解有很多种中途，其中包括：

- 提供编译器使用信息

编译器可以使用这些注解来检查错误或者禁止显示告警，如 @Override、@Deprecated、@SuppressWarnings。

- 编译或部署时处理

可以通过注解信息生产相关代码，如lombok的 @Data、@ToString 等注解。

- 运行时处理

在运行时处理的逻辑，如常用的Spring框架中的 @Service、@Component、@SpringBootApplication 等注解

### Java 内置的注解

Java一开始有定义了一套注解，共8个，3个在java.lang中，剩下4个在java.lang.annotation中

其中，作用于代码的注解有

- @Override - 检查该方法是否是重载方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
- @Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
- @SuppressWarnings - 指示编译器去忽略注解中声明的警告。

作用在其他注解的注解（或者说是**元注解**）有

- @Retention - 注解的声明周期，标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问，value为RetentionPolicy类型，有以下三种
   - RetentionPolicy.SOURCE：当前注解编译期可见，不会写入 class 文件
   - RetentionPolicy.CLASS：类加载阶段丢弃，会写入 class 文件
   - RetentionPolicy.RUNTIME：永久保存，可以反射获取
- @Target - 注解的作用目标，标记这个注解应该是哪种 Java 成员，value为ElementType数组，ElementType类型如下
   - ElementType.TYPE：允许被修饰的注解作用在类、接口和枚举上
   - ElementType.FIELD：允许作用在属性字段上
   - ElementType.METHOD：允许作用在方法上
   - ElementType.PARAMETER：允许作用在方法参数上
   - ElementType.CONSTRUCTOR：允许作用在构造器上
   - ElementType.LOCAL_VARIABLE：允许作用在本地局部变量上
   - ElementType.ANNOTATION_TYPE：允许作用在注解上
   - ElementType.PACKAGE：允许作用在包上
- @Inherited - 阐述了某个被标注的类型是被继承的。
- @Documented - 标记这些注解是否包含在用户文档中。

从java7开始，又额外添加了3个注解

- @SafeVarargs - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
- @FunctionalInterface - Java 8 开始支持，标识一个匿名函数或函数式接口。
- @Repeatable - Java 8 开始支持，标识某注解可以在同一个声明上使用多次。

### 自定义注解

自定义注解，是使用注解**元数据**来实现的，和java内置注解一样，是使用 @interface 关键字来声明的。
定义注解格式

```java
public @interface 注解名 {定义体}
```

如：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface DemoAnnotation {
  	String value(); //public 关键字可以去掉，默认是public的
    String name() default "write"; 
}
```

参数只有public或默认(default)这两个访问修饰符，例如 String value() 这里把方法设为 default 类型
注解参数可支持的数据类型有：

- 所有基本数据类型（int,float,boolean,byte,double,char,long,short)
- String类型
- Class类型
- enum类型
- Annotation类型
- 以上所有类型的数组


其中当value属性存在的时候，且只写value属性是，可以省略value，如spring的 @Component 注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```

使用的时候可以是 @Component、@Component("demo")、@Component(value="demo")；

### 注解与反射

通过过反射是可以获取相应的注解的，然后获取注解后根据注解或者注解里面的属性来进一步逻辑处理。
如Class类中提供了一下一些方法用于反射注解：

- getAnnotation：返回指定的注解
- isAnnotationPresent：判定当前元素是否被指定注解修饰
- getAnnotations：返回所有的注解
- getDeclaredAnnotation：返回本元素的指定注解
- getDeclaredAnnotations：返回本元素的所有注解，不包含父类继承而来的


相应的，Field、Method等类也有同样的方法来反射注解

### 注解使用案例

- 定义注解

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.TYPE})
public @interface CustomName {
    String value();
}
```


- 定义实体类



```java

@CustomName("person")
public class Person {
    @CustomName("user_name")
    private String userName;
    @CustomName("gender")
    private String sex;
    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getUserName() {
        return userName;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public String getSex() {
        return sex;
    }
}
```

- 反射使用



```java
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;

public class AnnotationDemo {
    public static void main(String[] args) throws Exception {
        Person person = new Person();
        person.setUserName("demo");
        person.setSex("boy");
        
        String personKeyName = "";
        CustomName annotation = Person.class.getAnnotation(CustomName.class);
        if (annotation != null) {
            personKeyName = annotation.value();
        }
        Map<String, Object> data = new HashMap<>();
        for (Field field : Person.class.getDeclaredFields()) {
            CustomName fieldAnnotation = field.getAnnotation(CustomName.class);
            String key;
            if (fieldAnnotation == null) {
                key = field.getName();
            } else {
                key = fieldAnnotation.value();
            }
            field.setAccessible(true);
            Object value = field.get(person);
            data.put(key, value);
        }

        System.out.println("person key name:" + personKeyName);
        for (Map.Entry<String, Object> entry : data.entrySet()) {
            System.out.println("key:" + entry.getKey() + " value:" + entry.getValue());
        }
    }
}
```

输出

```shell
person key name:person
key:gender value:boy
key:user_name value:demo
```

上面的例子至少个简单的注解使用，注解使用的场景还是比较多的，如一开始的用途例子中的 **lombok** 用于代码生成，spring中的 @Component 用于 bean 的生产，jpa中的 @Table 用于数据表的识别等等。

### 总结


- java使用 **@interface** 来定义注解
- 注解可以定义两个或多个参数和默认值，核心参数使用 **value** 名称
- 应当设置 **@Retention(RetentionPolicy.RUNTIME)** 便于运行期读取该 Annotation。
- 注解是可以通过反射来使用的
