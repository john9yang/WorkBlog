# Java运行时数据区域

## 程序计数器

当前线程所执行的字节码的行号指示器，线程私有的，每个线程都有自己的程序计数器。

## Java虚拟机栈

线程私有的。局部变量存放了基本数据类型(boolean, byte, char, short, int, float, long, double), 对象引用，returnAddress(指向了一条字节码指令的地址)。

## 本地方法栈

## Java堆 {id="java_1"}

各个线程共享的区域。

## 方法区

各个线程共享的内存区域。

## 运行时常量

## 直接内存


