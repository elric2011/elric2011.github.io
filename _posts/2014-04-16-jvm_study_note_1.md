---
layout: post
title: "《深入理解Java虚拟机》学习笔记（一）"
permalink: /a/jvm_study_note_1.html
description: ""
category: 读书笔记
tags: [深入理解Java虚拟机, Java, JVM]
---
{% include JB/setup %}

Java内存区域的划分
-----

根据存放数据的不同，JVM的内存空间分为程序计数器、虚拟机栈、本地方法栈、堆、方法区、运行时常量池、直接内存（在Java运行时内存空间之外），本节总结了这几个区域的作用、特点，并给出了部分区域内存溢出的简单用例。

> 备注：代码可从 [https://github.com/elric2011/jvm-study](https://github.com/elric2011/jvm-study) 获取

### 程序计数器 ###

程序计数器可理解为当前线程所执行**字节码**的行号指示器，用于告诉字节码解释器下一条需要执行的字节码指令

程序的逻辑跳转（分支、循环、跳转、异常处理、线程恢复等）都是基于程序计数器实现的

#### 特点： ####

* 线程私有
* 当线程执行到Native Method时，改计数器值会置为空
* 规范中没有规定该区域的OutOfMemoryError情况

### 虚拟机栈 ###

虚拟机执行Java方法（字节码）时，用于存放局部变量表、操作数表、方法出口、动态链接等信息的区域。

每一个Java方法被调用到方法调用完成，就对应着一个栈帧在虚拟机栈中的压栈到出栈。

一个方法的局部变量表（存放基础类型和引用类型）大小在编译阶段就可知，进入方法时分配给这块的内存大小是确定。

#### 特点： ####

* 线程私有的
* 当栈深度超出虚拟机允许的深度时，抛出StackOverflowError
* 如果栈总空间可动态扩展，当扩展时无法申请到足够内存，抛出OutOfMemoryError

#### 代码示例 ####

栈溢出示例：

    /**
     * VM Args: -Xss128K
     * @author elric.wang
     */
    public class VMStackSOF {
    
        private int counter = 1;
    
        private void stackLeak() {
            long a = 0;
            counter++;
            stackLeak();
        }
    
        public static void main(String[] args) {
            VMStackSOF sut = new VMStackSOF();
            try {
                sut.stackLeak();
            } catch (Throwable t) {
                t.printStackTrace();
                System.out.println("StackLength:" + sut.counter);
            }
        }
    }

问题：多线程程序，总的栈空间多大？

### 本地方法栈 ###

概念与虚拟机栈相同，区别是为虚拟机执行Native Method服务

#### 特点： ####

* 线程私有
* 当栈深度超出虚拟机允许的深度时，抛出StackOverflowError
* 如果栈总空间可动态扩展，当扩展时无法申请到足够内存，抛出OutOfMemoryError
* 某些虚拟机实现直接将其与虚拟机栈合二为一（HotSpot）

### Java堆 ###

存放对象示例和数组，几乎所有的对象示例都存储在这块区域，所以一般在虚拟机运行时内存中占最大一块比例

#### 特点： ####

* 线程间共享
* 垃圾回收器主要工作于该区域
* 虚拟机实现时可固定大小，也可以是动态扩展
* 如果堆中没有足够空间完成实例分配，且无法继续动态扩展时，抛出OutOfMemoryError

#### 代码示例 ####

堆溢出示例：

    /**
     * VM Args: -Xmx100M -Xms100M
     * @author elric.wang
     */
    public class HeapOOM {
    
        public static void main(String[] args) {
            List<HeapOOM> list = new ArrayList<HeapOOM>();
            while (true) {
                list.add(new HeapOOM());
            }
        }
    }

### 方法区 ###

用于存放虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。HotSpot实现中该区域也叫“永久区”，即虚拟机将GC分代收集扩展到该区域。

#### 特点： ####

* 线程间共享
* 当方法区无法满足分配需求时，抛出OutOfMemoryError

### 运行时常量池 ###

该区域是方法区的一部分，用于存放编译时生成的字面量和符号引用以及运行时产生的常量（例如String.intern()方法产生新的字符串常量）。

#### 特点： ####

* 方法区一部分，无法申请到内存空间时抛出OutOfMemoryError

#### 代码示例 ####

运行时常量池溢出示例：

    /**
     * VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
     * @author elric.wang
     */
    public class RuntimeConstPoolOOM {
    
        public static void main(String[] args) {
            List<String> list = new ArrayList<String>();
            int i = 0;
            while (true) {
                list.add(("do something to make this string longer" + i++).intern());
            }
        }
    }

### 直接内存区 ###

Java在引入NIO后，增加了管道和缓冲区，允许通过Native函数库直接分配堆外内存，以便提升数据交换性能（否则需要在Java堆和Native堆来回复制数据）

#### 特点： ####

* 不属于虚拟机运行时内存区
* 受操作系统内存限制，当无法分配到足够的内存时，抛出OutOfMemoryError

JVM内存分配相关参数
----

| 参数 | 用途 | 示例 | 备注 |
|-----|-----|-----|-----|
| -Xmx | 设置堆最大空间 | -Xmx256M | |
| -Xms | 设置堆最小空间 | -Xms128M | |
| -Xss | 设置每个线程虚拟机栈空间大小 | -Xss128K | |
| -Xoss | 设置每个线程本地方栈空间大小 | -Xoss128K | HotSpot改参数无效 |
| -Xmn | 设置年轻态堆空间大小 | -Xmn128M | eden+2survivor |
| -XX:PermSize | 设置持久代(perm gen)初始值 | -XX:PermSize=64M | 即方法区 |
| -XX:MaxPermSize | 设置持久代最大空间 | -XX:MaxPermSize=64M | |



