## 原生基本数据类型

Java 提供八种数据类型，其中六种数字类型（四种整数型，两种浮点型），一种字符类型，一种布尔型。

| 类型      | 长度                                        | 值                                                                           | 默认值      |
|:------- |:----------------------------------------- |:--------------------------------------------------------------------------- | -------- |
| byte    | 8 bit                                     | -128（-2^7） ~  127（2^7-1）<br />Byte.MIN_VALUE ~ Byte.MAX_VALUE               | 0        |
| short   | 16 bit                                    | -32768（-2^15）~ 32767（2^15 - 1）<br />Short.MIN_VALUE ~ Short.MAX_VALUE       | 0        |
| int     | 32 bit<br />Integer.Size                  | -2^31~ 2^31 - 1<br />Integer.MIN_VALUE ~ Integer.MAX_VALUE                  | 0        |
| long    | 64 bit                                    | -2^63~2^63 -1<br />Long..MIN_VALUE ~ Long..MAX_VALUE                        | 0L       |
| float   | 1bit(符号位(S))+8bit(指数位(E)),23bit(尾数位(M))   | 1.17549435E-38f ~ 3.4028235e+38f<br />Float.MIN_VALUE ~ Float.MAX_VALUE     | 0.0F     |
| double  | 1bit(符号位(S))+11bit(指数位(E)),,52bit(尾数位(M)) | 4.9e-324 ~ 1.7976931348623157e+308<br />Double.MIN_VALUE ~ Double.MAX_VALUE | 0.0D     |
| boolean | 没有明确指出boolean的大小                          | false\|true                                                                 | false    |
| char    | 16bit                                     | \u0000(0) ~ \uffff(65,535)<br />Character.MIN_VALUE ~ Character.MAX_VALUE   | '\u0000' |

> [!NOTE]
> boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

[Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

## 包装类型

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

| 基本类型    | 包装类型      |
| ------- | --------- |
| byte    | Byte      |
| short   | Short     |
| int     | Integer   |
| long    | Long      |
| float   | Float     |
| double  | Double    |
| boolean | Boolean   |
| char    | Character |

通过 xxx.valueOf 来进行装箱，如 Integer.valueOf(2)

通过 xxxValue() 来进行拆箱，如 X.intValue()

[Autoboxing and Unboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)

## 缓冲池

Integer 是有缓冲池的，是有 Integer 的静态内部类 IntegerCache 实现，IntegerCache 有 `Integer[] cache` 数据来缓存。默认范围是 -128 ~127。

IntegerCache 默认低位是 **-128**，高位可以通过 **java.lang.Integer.IntegerCache.high** 配置来更改。

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer[] cache;
    static Integer[] archivedCache;
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                h = Math.max(parseInt(integerCacheHighPropValue), 127);
                // Maximum array size is Integer.MAX_VALUE
                   h = Math.min(h, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;
            // Load IntegerCache.archivedCache from archive, if possible
            VM.initializeFromArchive(IntegerCache.class);
            int size = (high - low) + 1;
            // Use the archived cache if it exists and is large enough
            if (archivedCache == null || size > archivedCache.length) {
                Integer[] c = new Integer[size];
                int j = low;
                for(int i = 0; i < c.length; i++) {
                   c[i] = new Integer(j++);
            }
            archivedCache = c;
        }
        cache = archivedCache;
        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }
    private IntegerCache() {}
}
```

new Integer(100) 与 Integer.valueOf(100) 的区别在于：

- new Integer(100) 每次都会新建一个对象；
- Integer.valueOf(100) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

```java
Integer a = new Integer(100);
Integer b = new Integer(100);
System.out.println(a == b);    // false
Integer c = Integer.valueOf(100);
Integer d = Integer.valueOf(100);
System.out.println(c == d);   // true,同一个对象
```

在 jdk1.9 之后，**new Integer()** 已经标记为 deprecated 了，官方还是建议用 valueOf 来进行装箱。