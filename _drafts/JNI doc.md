来源：
http://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html

Introduction
----
####  Chapter 1:
This chapter introduces the Java Native Interface (JNI). The JNI is a native programming interface. It allows Java code that runs inside a Java Virtual Machine (VM) to interoperate with applications and libraries written in other programming languages, such as C, C++, and assembly.

本章介绍Java本地接口(Java Native Interface, JNI)。JNI是一种本地编程接口，它使得运行在Java虚拟中的Java代码可以与由其他语言（例如C, C++, 以及汇编语言）编写的库或者程序进行交互。

The most important benefit of the JNI is that it imposes no restrictions on the implementation of the underlying Java VM. Therefore, Java VM vendors can add support for the JNI without affecting other parts of the VM. Programmers can write one version of a native application or library and expect it to work with all Java VMs supporting the JNI.

JNI最大的好处在于，它没有强制要求底层Java虚拟机的实现方式。因此，Java虚拟机供应商可以在不影响虚拟机其他部分的条件下，添加对JNI的支持。开发者可以编写一个本地版本的程序或者库，并期望在所有支持JNI的虚拟机上运行。

This chapter covers the following topics:

    Java Native Interface Overview
    Historical Background
        JDK 1.0 Native Method Interface
        Java Runtime Interface
        Raw Native Interface and Java/COM Interface
    Objectives
    Java Native Interface Approach
    Programming to the JNI
    Changes

本章主要包含以下内容：
    
-   JNI概述
-   历史背景
-   原本地接口以及Java/COM 接口
-   目标
-   JNI实现方法
-   使用JNI编程
-   变化


##### Java Native Interface Overview

While you can write applications entirely in Java, there are situations where Java alone does not meet the needs of your application. Programmers use the JNI to write Java native methods to handle those situations when an application cannot be written entirely in Java.

##### JNI概述
虽然可以完全用Java编写程序，但是有些情况下，仅仅使用Java无法满足程序的需求。当程序无法完全使用Java编写时，开发者通过使用JNI来编写Java本地方法来处理这些情况。


The following examples illustrate when you need to use Java native methods:

    The standard Java class library does not support the platform-dependent features needed by the application.
    You already have a library written in another language, and wish to make it accessible to Java code through the JNI.
    You want to implement a small portion of time-critical code in a lower-level language such as assembly.

下面的粒子说明了在何时使用Java本地方法：

    标准的Java类库不支持部分应用程序依赖的与平台相关的特性。
    已经使用其他语言编写了一个库，期望Java代码能够通过JNI调用。
    计划使用低级语言（如汇编）实现一小部分对时间敏感的代码。

By programming through the JNI, you can use native methods to:

    Create, inspect, and update Java objects (including arrays and strings).
    Call Java methods.
    Catch and throw exceptions.
    Load classes and obtain class information.
    Perform runtime type checking.

通过使用JNI编程，可以使用本地方法实现：

    创建，检查，更新Java对象（包括数组和字符串）。
    调用Java方法。
    捕获/抛出异常。
    加载类，获取类的信息。
    执行运行时类型检查。

You can also use the JNI with the Invocation API to enable an arbitrary native application to embed the Java VM. This allows programmers to easily make their existing applications Java-enabled without having to link with the VM source code.

也可以使用JNI调用API将任意的本地程序嵌入到Java虚拟机中。这样允许开发者在不需要链接虚拟机源码的情况下，轻易地将现有的程序支持Java。

##### Historical Background

VMs from different vendors offer different native method interfaces. These different interfaces force programmers to produce, maintain, and distribute multiple versions of native method libraries on a given platform.

We briefly examine some of the native method interfaces, such as:

-    JDK 1.0 native method interface
-    Netscape's Java Runtime Interface
-    Microsoft's Raw Native Interface and Java/COM interface

#####历史背景
不同的厂商实现的虚拟机提供不同的本地方法接口。这些不同的接口限制开发者需要为每一个平台编写、维护和发布不同版本的本地方法库。

简要的分析一些本地方法接口，例如：

- JDK 1.0 本地方法接口
- Netscape的Java运行时接口
- Microsoft的原本地接口和Java/COM接口

##### JDK 1.0 Native Method Interface

JDK 1.0 was shipped with a native method interface. Unfortunately, there were two major reasons that this interface was unsuitable for adoption by other Java VMs.

First, the native code accessed fields in Java objects as members of C structures. However, the Java Language Specification does not define how objects are laid out in memory. If a Java VM lays out objects differently in memory, then the programmer would have to recompile the native method libraries.

Second, JDK 1.0's native method interface relied on a conservative garbage collector. The unrestricted use of the unhand macro, for example, made it necessary to conservatively scan the native stack.

#####JDK 1.0 本地方法接口
JDK 1.0封装了本地方法接口。但是，该接口不适用于其他Java虚拟机，原因如下：

首先，Java对象中的代码被当做C结构体中的成员进行访问。然而，Java语言规范没有规定如何在内存中管理对象。如果一个Java虚拟机以不同的方式将对象进行存储，那么开发者必须重新编译本地方法库。

其次，JDK 1.0的本地方法接口依赖于保守型垃圾回收器。例如，无限制的使用unhand 宏，将需要保守地扫描本地栈。

##### Java Runtime Interface

Netscape had proposed the Java Runtime Interface (JRI), a general interface for services provided in the Java virtual machine. JRI was designed with portability in mind---it makes few assumptions about the implementation details in the underlying Java VM. The JRI addressed a wide range of issues, including native methods, debugging, reflection, embedding (invocation), and so on.

#####Java运行时接口
Netscape曾经提出了Java运行时接口(Java Runtime Interface, JRI), 一种访问Java虚拟机提供服务的通用接口。JRI的设计采用了可移植性的思想，对Java虚拟机的实现细节几乎不做要求。JRI解决了大量的问题，包括本地方法，调试，反射，嵌入式调用）等。

##### Raw Native Interface and Java/COM Interface

The Microsoft Java VM supports two native method interfaces. At the low level, it provides an efficient Raw Native Interface (RNI). The RNI offers a high degree of source-level backward compatibility with the JDK’s native method interface, although it has one major difference. Instead of relying on conservative garbage collection, the native code must use RNI functions to interact explicitly with the garbage collector.

At a higher level, Microsoft's Java/COM interface offers a language-independent standard binary interface to the Java VM. Java code can use a COM object as if it were a Java object. A Java class can also be exposed to the rest of the system as a COM class.

#####原本地接口和Java/COM接口
微软的Java虚拟机支持两种本地方法接口。在底层中，它提供了一种高效的原本地接口(Raw Native Interface, RNI)。RNI提供了与JDK的本地方法在代码级别有较高的向后兼容性，虽然它有一个主要的区别。本地代码必须通过RNI方法与垃圾回收器进行显示的交互，而不是依赖于保守型垃圾回收机制。
在上层中，微软的Java/COM接口为Java虚拟机提供了一种语言无关的标准二进制接口。Java代码把COM对象当做Java对象来使用。一个Java类也可以被作为COM类暴露给系统的其他部分。

##### Objectives

We believe that a uniform, well-thought-out standard interface offers the following benefits for everyone:

-    Each VM vendor can support a larger body of native code.
-    Tool builders will not have to maintain different kinds of native method interfaces.
-    Application programmers will be able to write one version of their native code and this version will run on different VMs.

The best way to achieve a standard native method interface is to involve all parties with an interest in Java VMs. Therefore we organized a series of discussions among the Java licensees on the design of a uniform native method interface. It is clear from the discussions that the standard native method interface must satisfy the following requirements:

-    Binary compatibility - The primary goal is binary compatibility of native method libraries across all Java VM implementations on a given platform. Programmers should maintain only one version of their native method libraries for a given platform.
-    Efficiency - To support time-critical code, the native method interface must impose little overhead. All known techniques to ensure VM-independence (and thus binary compatibility) carry a certain amount of overhead. We must somehow strike a compromise between efficiency and VM-independence.
-    Functionality - The interface must expose enough Java VM internals to allow native methods to accomplish useful tasks.

#####目标
JNI设计者认为，一种统一的、经过周详考虑的标准接口可以提供如下好处：

- 每一个虚拟机厂商可以支持大量的本地代码。
- 工具开发者不需要维护各种各样的本地方法接口。
- 程序开发者将可以只编写一个版本的本地代码，并且可以在所有的虚拟机上运行。

制定一个标准本地方法接口的最佳方法是考虑所有的对Java虚拟机有兴趣的机构。因此，我们在Java授权商之间组织了一系列的会议，研讨关于统一的本地方法接口的设计。从讨论的结果可以发现，标准的本地方法接口必须满足如下需求：

- 二进制兼容性；主要目标在一个给定平台上所有Java虚拟机实现中，是本地方法库的二进制兼容性。针对一个特定的平台，开发者应该对本地方法库只需要维护一个版本。
- 高效性；为了支持对时间敏感的代码，本地方法接口必须占用很小的开销。为了确保虚拟机无关性（从而二进制兼容性），所有已知的技术都将会带来一定的开销。我们必须在效率和虚拟机无关性中间达成某种妥协。
- 功能性；接口必须能够暴露足够多的Java虚拟机的内部细节，以允许本地方法完成有意义的任务。

##### Java Native Interface Approach

We hoped to adopt one of the existing approaches as the standard interface, because this would have imposed the least burden on programmers who had to learn multiple interfaces in different VMs. Unfortunately, no existing solutions are completely satisfactory in achieving our goals.

Netscape’s JRI is the closest to what we envision as a portable native method interface, and was used as the starting point of our design. Readers familiar with the JRI will notice the similarities in the API naming convention, the use of method and field IDs, the use of local and global references, and so on. Despite our best efforts, however, the JNI is not binary-compatible with the JRI, although a VM can support both the JRI and the JNI.

##### JNI使用方法
我们希望采纳一种现有的方法来作为标准接口，因为这样可以给已经熟悉多种虚拟机接口的开发者增加最少的负担。不幸的是，没有一种已有的方法能够完全满足我们的目标。

Netscape提出的JRI是最接近我们所期望的一种可移植的本地方法接口，因此被用来作为我们设计的起点。熟悉JRI的读者将会发现在API命名约定，方法和字段的ID，局部和全局引用等方面的相似之处。虽然我们尽了最大的努力，但是，JNI与JRI不是二进制兼容的，即使一个虚拟机可以同时支持JRI和JNI。

Microsoft’s RNI was an improvement over JDK 1.0 because it solved the problem of native methods working with a nonconservative garbage collector. The RNI, however, was not suitable as a VM-independent native method interface. Like the JDK, RNI native methods access Java objects as C structures, leading to two problems:

-    RNI exposed the layout of internal Java objects to native code.
-    Direct access of Java objects as C structures makes it impossible to efficiently incorporate “write barriers,” which are necessary in advanced garbage collection algorithms.

Microsoft的RNI在JDK 1.0 基础上提出改进，因为它解决了本地方法使用非保守型垃圾回收器的问题。然而，RNI并不适合作为一种与虚拟机无关的本地方法接口。就像JDK一样，RNI本地方法把Java对象当做C结构体来访问，导致了两个问题：

- RNI 把Java对象的内部结构暴露给了本地代码。
- 把Java对象当做C结构体直接访问，使得无法与“写入屏障(write barriers)”高效协作，而这一点对高级垃圾回收算法是至关重要的。

As a binary standard, COM ensures complete binary compatibility across different VMs. Invoking a COM method requires only an indirect call, which carries little overhead. In addition, COM objects are a great improvement over dynamic-link libraries in solving versioning problems.

作为一个标准，COM保证在所有的虚拟机上完全二进制兼容。调用一个COM方法只需要一个间接调用，这将会带来一点开销。此外，COM对象在动态链接库的版本问题解决上进行了较大幅度的改进。
The use of COM as the standard Java native method interface, however, is hampered by a few factors:

-    First, the Java/COM interface lacks certain desired functions, such as accessing private fields and raising general exceptions.
-    Second, the Java/COM interface automatically provides the standard IUnknown and IDispatch COM interfaces for Java objects, so that native code can access public methods and fields. Unfortunately, the IDispatch interface does not deal with overloaded Java methods and is case-insensitive in matching method names. Furthermore, all Java methods exposed through the IDispatch interface are wrapped to perform dynamic type checking and coercion. This is because the IDispatch interface is designed with weakly-typed languages (such as Basic) in mind.
-    Third, instead of dealing with individual low-level functions, COM is designed to allow software components (including full-fledged applications) to work together. We believe that it is not appropriate to treat all Java classes or low-level native methods as software components.
-    Fourth, the immediate adoption of COM is hampered by the lack of its support on UNIX platforms.

但是，COM作为一个标准的Java本地方法接口而言，被以下几个问题所阻碍：

- 首先，Java/COM接口缺少必须的方法，例如访问私有字段以及抛出通用异常。
- 其次，Java/COM接口自动为Java对象提供了标准的IUnknown和IDispatch COM接口，所以本地方法可以访问public的方法和字段。不幸的是，IDispatch接口不能够处理Java方法重载的问题，而且它在方法名称匹配时对大小写不敏感。此外，所有通过IDispatch接口暴露的Java方法，被打包执行动态类型检查和强制转换。因为IDispatch接口融合了弱类型语言（如 Basic）的思想。
- 再次，COM的设计是用来解决软件模块级别（包括成熟的软件）间协作的问题，而非处理个别低级的方法。我们认为将所有的Java类或者低级的本地方法当做软件模块来对待是不合理的。
- 最后，由于COM不支持UNIX平台，所以阻碍了它被立即采用。

Although Java objects are not exposed to the native code as COM objects, the JNI interface itself is binary-compatible with COM. JNI uses the same jump table structure and calling convention that COM does. This means that, as soon as cross-platform support for COM is available, the JNI can become a COM interface to the Java VM.

尽管Java对象并没有作为COM对象暴露给本地代码，JNI本身与COM是二进制兼容的。JNI采用了和COM相同的跳表结构和调用约定。这意味着，如果支持跨平台的COM，JNI能够成为Java虚拟机的COM接口。

JNI is not believed to be the only native method interface supported by a given Java VM. A standard interface benefits programmers, who would like to load their native code libraries into different Java VMs. In some cases, the programmer may have to use a lower-level, VM-specific interface to achieve top efficiency. In other cases, the programmer might use a higher-level interface to build software components. Indeed, as the Java environment and component software technologies become more mature, native methods will gradually lose their significance.

JNI并不是Java虚拟机所支持的唯一本地方法接口。那些喜欢将本地代码库加载到不同的Java虚拟机的开发者，将从标准接口获益。

##### Programming to the JNI

Native method programmers should program to the JNI. Programming to the JNI insulates you from unknowns, such as the vendor’s VM that the end user might be running. By conforming to the JNI standard, you will give a native library the best chance to run in a given Java VM.

If you are implementing a Java VM, you should implement the JNI. JNI has been time tested and ensured to not impose any overhead or restrictions on your VM implementation, including object representation, garbage collection scheme, and so on. Please send us your feedback if you run into any problems we may have overlooked.

##### Changes

As of Java SE 6.0, the deprecated structures JDK1_1InitArgs and JDK1_1AttachArgs have been removed, instead JavaVMInitArgs and JavaVMAttachArgs are to be used.
Contents | Previous | Next 	
