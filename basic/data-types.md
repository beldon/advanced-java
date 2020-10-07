## 原生基本数据类型

Java 提供八种数据类型，其中六种数字类型（四种整数型，两种浮点型），一种字符类型，一种布尔型。



| 类型    | 长度                                               | 值                                                           | 默认值   |
| :------ | :------------------------------------------------- | :----------------------------------------------------------- | -------- |
| byte    | 8 bit                                              | -128（-2^7） ~  127（2^7-1）<br />Byte.MIN_VALUE ~ Byte.MAX_VALUE | 0        |
| short   | 16 bit                                             | -32768（-2^15）~ 32767（2^15 - 1）<br />Short.MIN_VALUE ~ Short.MAX_VALUE | 0        |
| int     | 32 bit                                             | -2^31~ 2^31 - 1<br />Integer.MIN_VALUE ~ Integer.MAX_VALUE   | 0        |
| long    | 64 bit                                             | -2^63~2^63 -1<br />Long..MIN_VALUE ~ Long..MAX_VALUE         | 0L       |
| float   | 1bit(符号位(S))+8bit(指数位(E)),23bit(尾数位(M))   | 1.17549435E-38f ~ 3.4028235e+38f<br />Float.MIN_VALUE ~ Float.MAX_VALUE | 0.0F     |
| double  | 1bit(符号位(S))+11bit(指数位(E)),,52bit(尾数位(M)) | 4.9e-324 ~ 1.7976931348623157e+308<br />Character.MIN_VALUE ~ Character.MAX_VALUE | 0.0D     |
| boolean | 没有明确指出boolean的大小                          | false\|true                                                  | false    |
| char    | 16bit                                              | \u0000(0) ~ \uffff(65,535)<br />Character.MIN_VALUE ~ Character.MAX_VALUE | '\u0000' |

> [!NOTE]
> boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

[Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

