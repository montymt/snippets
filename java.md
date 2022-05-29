# java

## basic

标识符：`0-9a-zA-Z_$`

基本类型：byte，short，int，long，float，double, boolean，char

byte：-128 ~ 127

long：`long num = 123L`

float: `float f = 3.141592f`，有效位数7-8位，double为15-16位

char: `char c = 'a'`，与short一样占2字节

变量需要先申明并初始化再使用

byte，short，char运算，需要int接收

强制类型转换采取截断操作，精度会损失

加运算，如果有String则是连接，char默认按数值运算

String转int，`Integer.parseInt()`

负数：用补码存储，即反码加1，最高位为1，-128的补码：`1000 0000`

取余：结果的符号与被模数相同

`++`，`+=`等运算符不会改变原数据类型

左移，相当于乘以2的N次幂，右移，相当于除以2的N次幂，最高位保持不变，负数空缺位补1，无符号右移，`>>>`，空缺位补0

按位取反`~`，逻辑取反`!`，逻辑运算符：`&&`短路与，`&`非短路与

`> < >= <=`，只能用在数值类型之间比较

获取用户输入：

```java
import java.util.Scanner;
Scanner scan = new Scanner(System.in);
String name = scan.next();
```

switch语句条件类型：byte，char，short，int，String，Enum，匹配后继续向下执行，不判断条件，需要加break跳出，case中只能是常量

for加标签：`labelName: for(...`，可在break或continue中使用标签结束指定的循环：`break labelName;`

数组：

```java
int[] scoreArr = {89, 67, 88};
int[] scoreOri = Arrays.copyOf(scoreArr);
Arrays.sort(scoreArr);
System.out.println(Arrays.toString(scoreArr));
String[] nameArr = new String[3];//null
```
## oop

