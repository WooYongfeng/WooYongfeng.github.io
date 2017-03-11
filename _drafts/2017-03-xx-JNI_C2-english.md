---
layout: post
title: template page
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---

Design Overview
Chapter   2 
Chapter 2 设计概要

This chapter focuses on major design issues in the JNI. Most design issues in this section are related to native methods. The design of the Invocation API is covered in Chapter 5.

本章关注于JNI的主要设计问题。本章中的设计问题大多都与本地方法相关。调用API的设计将在第五章介绍。

This chapter covers the following topics:

 -    JNI Interface Functions and Pointers
 -    Compiling, Loading and Linking Native Methods
    -        Resolving Native Method Names
    -        Native Method Arguments
 -    Referencing Java Objects
    -        Global and Local References
    -        Implementing Local References
 -    Accessing Java Objects
    -        Accessing Primitive Arrays
    -        Accessing Fields and Methods
 -    Java Exceptions
    -        Exceptions and Error Codes
    -        Asynchronous Exceptions
    -        Exception Handling

本章包含以下内容：

 -    JNI Interface Functions and Pointers
 -    Compiling, Loading and Linking Native Methods
    -        Resolving Native Method Names
    -        Native Method Arguments
 -    Referencing Java Objects
    -        Global and Local References
    -        Implementing Local References
 -    Accessing Java Objects
    -        Accessing Primitive Arrays
    -        Accessing Fields and Methods
 -    Java Exceptions
    -        Exceptions and Error Codes
    -        Asynchronous Exceptions
    -        Exception Handling

----
#####JNI Interface Functions and Pointers

Native code accesses Java VM features by calling JNI functions. JNI functions are available through an interface pointer. An interface pointer is a pointer to a pointer. This pointer points to an array of pointers, each of which points to an interface function. Every interface function is at a predefined offset inside the array. Figure 2-1 illustrates the organization of an interface pointer.

----
<h5 id ='1'>JNI接口函数和指针</h5>
 本地代码通过调用JNI函数来访问Java虚拟机的特性。JNI函数可以通过接口指针获得。该指针指向了一个指针的数组，数组中的每一个指针都指向一个接口函数。每一个接口函数位于数组中预先定义的偏移量中。图2-1解释了一个接口指针的组织方式。
The previous context describes this image.

 
Figure 2-1 Interface Pointer
![jni_doc_c2_designa.gif](\images\blocks\JNI_doc\jni_doc_c2_designa.gif)

图 2-1 接口指针

The JNI interface is organized like a C++ virtual function table or a COM interface. The advantage to using an interface table, rather than hard-wired function entries, is that the JNI name space becomes separate from the native code. A VM can easily provide multiple versions of JNI function tables. For example, the VM may support two JNI function tables:

 - one performs thorough illegal argument checks, and is suitable for debugging;

 - the other performs the minimal amount of checking required by the JNI specification, and is therefore more efficient.

The JNI interface pointer is only valid in the current thread. A native method, therefore, must not pass the interface pointer from one thread to another. A VM implementing the JNI may allocate and store thread-local data in the area pointed to by the JNI interface pointer.

Native methods receive the JNI interface pointer as an argument. The VM is guaranteed to pass the same interface pointer to a native method when it makes multiple calls to the native method from the same Java thread. However, a native method can be called from different Java threads, and therefore may receive different JNI interface pointers.

JNI接口的组织方式类似于C++虚函数表或者COM接口。相比使用硬接入函数入口，接口表的优势在于将JNI命名空间与本地代码独立。虚拟机可以很容易的提供多种版本的JNI函数表。例如，一个虚拟机可能支持如下两种JNI函数表:

 - 一个执行所有非法参数检查，适合于debugging。
 - 另一个只执行JNI标准要求的最精简的检查，因此更加高效。

JNI接口指针只在当前线程中有效。因此，本地方法一定不要在不同的线程中传递接口指针。实现了JNI的虚拟机可能会在JNI指针指向的空间分配和存储线程相关的数据。

JNI接口指针将被本地方法作为参数接收。在同一个Java线程中多次调用一个本地方法时，虚拟机保证将同一个接口指针传递给该本地方法。然而，一个本地方法可以在不同的Java线程中被调用，因此可能会接收到不同的JNI接口指针。

----

#####Compiling, Loading and Linking Native Methods

Since the Java VM is multithreaded, native libraries should also be compiled and linked with multithread aware native compilers. For example, the -mt flag should be used for C++ code compiled with the Sun Studio compiler. For code complied with the GNU gcc compiler, the flags -D_REENTRANT or -D_POSIX_C_SOURCE should be used. For more information please refer to the native compiler documentation.

----
<h5 id ='2'>编译，加载和链接本地方法</h5>
由于Java虚拟机是多线程的，本地库也应该使用支持多线程的本地编译器编译和链接。例如，使用 Sun Studio编译器编译C++代码时，应该使用 ```-mt``` 标志。而对于使用GNU gcc编译器编译时，应使用```-D_REENTRANT``` 或者 ```-D_POSIX_C_SOURCE``` 标志。更多的信息请查阅相关本地编译器的文档。


Native methods are loaded with the System.loadLibrary method. In the following example, the class initialization method loads a platform-specific native library in which the native method f is defined:

```
package pkg;  

class Cls { 

     native double f(int i, String s); 

     static { 

         System.loadLibrary(“pkg_Cls”); 

     } 

} 

```

使用方法```System.loadLibrary ```来加载本地方法。在下面的示例中，类的初始化方法加载与平台相关的本地库，方法 f 定义在该库中。

```
package pkg;  

class Cls { 

     native double f(int i, String s); 

     static { 

         System.loadLibrary(“pkg_Cls”); 

     } 

} 

```

The argument to System.loadLibrary is a library name chosen arbitrarily by the programmer. The system follows a standard, but platform-specific, approach to convert the library name to a native library name. For example, a Solaris system converts the name pkg_Cls to libpkg_Cls.so, while a Win32 system converts the same pkg_Cls name to pkg_Cls.dll.



The programmer may use a single library to store all the native methods needed by any number of classes, as long as these classes are to be loaded with the same class loader. The VM internally maintains a list of loaded native libraries for each class loader. Vendors should choose native library names that minimize the chance of name clashes.

If the underlying operating system does not support dynamic linking, all native methods must be prelinked with the VM. In this case, the VM completes the System.loadLibrary call without actually loading the library.

The programmer can also call the JNI function RegisterNatives() to register the native methods associated with a class. The RegisterNatives() function is particularly useful with statically linked functions.
#####Resolving Native Method Names

Dynamic linkers resolve entries based on their names. A native method name is concatenated from the following components:

 -   the prefix Java_

 -  a mangled fully-qualified class name

 -   an underscore (“_”) separator

 -   a mangled method name

 -   for overloaded native methods, two underscores (“__”) followed by the mangled argument signature

The VM checks for a method name match for methods that reside in the native library. The VM looks first for the short name; that is, the name without the argument signature. It then looks for the long name, which is the name with the argument signature. Programmers need to use the long name only when a native method is overloaded with another native method. However, this is not a problem if the native method has the same name as a nonnative method. A nonnative method (a Java method) does not reside in the native library.

In the following example, the native method g does not have to be linked using the long name because the other method g is not a native method, and thus is not in the native library.

```
class Cls1 { 

  int g(int i); 

  native int g(double d); 

} 
```

We adopted a simple name-mangling scheme to ensure that all Unicode characters translate into valid C function names. We use the underscore (“_”) character as the substitute for the slash (“/”) in fully qualified class names. Since a name or type descriptor never begins with a number, we can use _0, ..., _9 for escape sequences, as Table 2-1 illustrates:

 
Table 2-1 Unicode Character Translation

|Escape Sequence |Denotes              |
|----------------|---------------------|
|_0XXXX          |a Unicode character XXXX.Note that lower case is used to represent non-ASCII Unicode characters, e.g., _0abcd as opposed to _0ABCD.|
| _1             | the character “_”    |
| _2             | the character “;” in signatures|
| _3             | the character “[“ in signatures|

 

Both the native methods and the interface APIs follow the standard library-calling convention on a given platform. For example, UNIX systems use the C calling convention, while Win32 systems use __stdcall.

-----

#####Native Method Arguments

The JNI interface pointer is the first argument to native methods. The JNI interface pointer is of type JNIEnv. The second argument differs depending on whether the native method is static or nonstatic. The second argument to a nonstatic native method is a reference to the object. The second argument to a static native method is a reference to its Java class.

The remaining arguments correspond to regular Java method arguments. The native method call passes its result back to the calling routine via the return value. Chapter 3 describes the mapping between Java and C types.

The following code example illustrates using a C function to implement the native method ```f ```. The native method ```f ```is declared as follows:

```
package pkg; 

class Cls {
    native double f(int i, String s);
    // ...
}
```

The C function with the long mangled name ```Java_pkg_Cls_f_ILjava_lang_String_2``` implements native method ```f```:

 

```
jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (
     JNIEnv *env,        /* interface pointer */
     jobject obj,        /* "this" pointer */
     jint i,             /* argument #1 */
     jstring s)          /* argument #2 */
{
     /* Obtain a C-copy of the Java string */
     const char *str = (*env)->GetStringUTFChars(env, s, 0);

     /* process the string */
     ...

     /* Now we are done with str */
     (*env)->ReleaseStringUTFChars(env, s, str);

     return ...
}
```

Note that we always manipulate Java objects using the interface pointer env. Using C++, you can write a slightly cleaner version of the code, as shown in the following code example:

```
extern "C" /* specify the C calling convention */ 

jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (

     JNIEnv *env,        /* interface pointer */
     jobject obj,        /* "this" pointer */
     jint i,             /* argument #1 */
     jstring s)          /* argument #2 */

{
     const char *str = env->GetStringUTFChars(s, 0);

     // ...

     env->ReleaseStringUTFChars(s, str);

     // return ...
}
```

With C++, the extra level of indirection and the interface pointer argument disappear from the source code. However, the underlying mechanism is exactly the same as with C. In C++, JNI functions are defined as inline member functions that expand to their C counterparts.

#####Referencing Java Objects

Primitive types, such as integers, characters, and so on, are copied between Java and native code. Arbitrary Java objects, on the other hand, are passed by reference. The VM must keep track of all objects that have been passed to the native code, so that these objects are not freed by the garbage collector. The native code, in turn, must have a way to inform the VM that it no longer needs the objects. In addition, the garbage collector must be able to move an object referred to by the native code.

#####Global and Local References

The JNI divides object references used by the native code into two categories: local and global references. Local references are valid for the duration of a native method call, and are automatically freed after the native method returns. Global references remain valid until they are explicitly freed.

Objects are passed to native methods as local references. All Java objects returned by JNI functions are local references. The JNI allows the programmer to create global references from local references. JNI functions that expect Java objects accept both global and local references. A native method may return a local or global reference to the VM as its result.

In most cases, the programmer should rely on the VM to free all local references after the native method returns. However, there are times when the programmer should explicitly free a local reference. Consider, for example, the following situations:

    A native method accesses a large Java object, thereby creating a local reference to the Java object. The native method then performs additional computation before returning to the caller. The local reference to the large Java object will prevent the object from being garbage collected, even if the object is no longer used in the remainder of the computation.

    A native method creates a large number of local references, although not all of them are used at the same time. Since the VM needs a certain amount of space to keep track of a local reference, creating too many local references may cause the system to run out of memory. For example, a native method loops through a large array of objects, retrieves the elements as local references, and operates on one element at each iteration. After each iteration, the programmer no longer needs the local reference to the array element.

The JNI allows the programmer to manually delete local references at any point within a native method. To ensure that programmers can manually free local references, JNI functions are not allowed to create extra local references, except for references they return as the result.

Local references are only valid in the thread in which they are created. The native code must not pass local references from one thread to another.

#####Implementing Local References

To implement local references, the Java VM creates a registry for each transition of control from Java to a native method. A registry maps nonmovable local references to Java objects, and keeps the objects from being garbage collected. All Java objects passed to the native method (including those that are returned as the results of JNI function calls) are automatically added to the registry. The registry is deleted after the native method returns, allowing all of its entries to be garbage collected.

There are different ways to implement a registry, such as using a table, a linked list, or a hash table. Although reference counting may be used to avoid duplicated entries in the registry, a JNI implementation is not obliged to detect and collapse duplicate entries.

Note that local references cannot be faithfully implemented by conservatively scanning the native stack. The native code may store local references into global or heap data structures.

#####Accessing Java Objects

The JNI provides a rich set of accessor functions on global and local references. This means that the same native method implementation works no matter how the VM represents Java objects internally. This is a crucial reason why the JNI can be supported by a wide variety of VM implementations.

The overhead of using accessor functions through opaque references is higher than that of direct access to C data structures. We believe that, in most cases, Java programmers use native methods to perform nontrivial tasks that overshadow the overhead of this interface.

#####Accessing Primitive Arrays

This overhead is not acceptable for large Java objects containing many primitive data types, such as integer arrays and strings. (Consider native methods that are used to perform vector and matrix calculations.) It would be grossly inefficient to iterate through a Java array and retrieve every element with a function call.

One solution introduces a notion of “pinning” so that the native method can ask the VM to pin down the contents of an array. The native method then receives a direct pointer to the elements. This approach, however, has two implications:

    The garbage collector must support pinning.

    The VM must lay out primitive arrays contiguously in memory. Although this is the most natural implementation for most primitive arrays, boolean arrays can be implemented as packed or unpacked. Therefore, native code that relies on the exact layout of boolean arrays will not be portable.

We adopt a compromise that overcomes both of the above problems.

First, we provide a set of functions to copy primitive array elements between a segment of a Java array and a native memory buffer. Use these functions if a native method needs access to only a small number of elements in a large array.

Second, programmers can use another set of functions to retrieve a pinned-down version of array elements. Keep in mind that these functions may require the Java VM to perform storage allocation and copying. Whether these functions in fact copy the array depends on the VM implementation, as follows:

    If the garbage collector supports pinning, and the layout of the array is the same as expected by the native method, then no copying is needed.

    Otherwise, the array is copied to a nonmovable memory block (for example, in the C heap) and the necessary format conversion is performed. A pointer to the copy is returned.

Lastly, the interface provides functions to inform the VM that the native code no longer needs to access the array elements. When you call these functions, the system either unpins the array, or it reconciles the original array with its non-movable copy and frees the copy.

Our approach provides flexibility. A garbage collector algorithm can make separate decisions about copying or pinning for each given array. For example, the garbage collector may copy small objects, but pin the larger objects.

A JNI implementation must ensure that native methods running in multiple threads can simultaneously access the same array. For example, the JNI may keep an internal counter for each pinned array so that one thread does not unpin an array that is also pinned by another thread. Note that the JNI does not need to lock primitive arrays for exclusive access by a native method. Simultaneously updating a Java array from different threads leads to nondeterministic results.

#####Accessing Fields and Methods

The JNI allows native code to access the fields and to call the methods of Java objects. The JNI identifies methods and fields by their symbolic names and type signatures. A two-step process factors out the cost of locating the field or method from its name and signature. For example, to call the method f in class cls, the native code first obtains a method ID, as follows:


jmethodID mid =      env->GetMethodID(cls, “f”, “(ILjava/lang/String;)D”); 

The native code can then use the method ID repeatedly without the cost of method lookup, as follows:


jdouble result = env->CallDoubleMethod(obj, mid, 10, str); 

A field or method ID does not prevent the VM from unloading the class from which the ID has been derived. After the class is unloaded, the method or field ID becomes invalid. The native code, therefore, must make sure to:

    keep a live reference to the underlying class, or

    recompute the method or field ID

if it intends to use a method or field ID for an extended period of time.

The JNI does not impose any restrictions on how field and method IDs are implemented internally.

#####Reporting Programming Errors

The JNI does not check for programming errors such as passing in NULL pointers or illegal argument types. Illegal argument types includes such things as using a normal Java object instead of a Java class object. The JNI does not check for these programming errors for the following reasons:

    Forcing JNI functions to check for all possible error conditions degrades the performance of normal (correct) native methods.

    In many cases, there is not enough runtime type information to perform such checking.

Most C library functions do not guard against programming errors. The printf() function, for example, usually causes a runtime error, rather than returning an error code, when it receives an invalid address. Forcing C library functions to check for all possible error conditions would likely result in such checks to be duplicated--once in the user code, and then again in the library.

The programmer must not pass illegal pointers or arguments of the wrong type to JNI functions. Doing so could result in arbitrary consequences, including a corrupted system state or VM crash.

#####Java Exceptions

The JNI allows native methods to raise arbitrary Java exceptions. The native code may also handle outstanding Java exceptions. The Java exceptions left unhandled are propagated back to the VM.

#####Exceptions and Error Codes

Certain JNI functions use the Java exception mechanism to report error conditions. In most cases, JNI functions report error conditions by returning an error code and throwing a Java exception. The error code is usually a special return value (such as NULL) that is outside of the range of normal return values. Therefore, the programmer can:

    quickly check the return value of the last JNI call to determine if an error has occurred, and

    call a function, ExceptionOccurred(), to obtain the exception object that contains a more detailed description of the error condition.

There are two cases where the programmer needs to check for exceptions without being able to first check an error code:

    The JNI functions that invoke a Java method return the result of the Java method. The programmer must call ExceptionOccurred() to check for possible exceptions that occurred during the execution of the Java method.

    Some of the JNI array access functions do not return an error code, but may throw an ArrayIndexOutOfBoundsException or ArrayStoreException.

In all other cases, a non-error return value guarantees that no exceptions have been thrown.

#####Asynchronous Exceptions

In cases of multiple threads, threads other than the current thread may post an asynchronous exception. An asynchronous exception does not immediately affect the execution of the native code in the current thread, until:

    the native code calls one of the JNI functions that could raise synchronous exceptions, or

    the native code uses ExceptionOccurred() to explicitly check for synchronous and asynchronous exceptions.

Note that only those JNI function that could potentially raise synchronous exceptions check for asynchronous exceptions.

Native methods should insert ExceptionOccurred()checks in necessary places (such as in a tight loop without other exception checks) to ensure that the current thread responds to asynchronous exceptions in a reasonable amount of time.

#####Exception Handling

There are two ways to handle an exception in native code:

    The native method can choose to return immediately, causing the exception to be thrown in the Java code that initiated the native method call.

    The native code can clear the exception by calling ExceptionClear(), and then execute its own exception-handling code.

After an exception has been raised, the native code must first clear the exception before making other JNI calls. When there is a pending exception, the JNI functions that are safe to call are:


  ExceptionOccurred()
  ExceptionDescribe()
  ExceptionClear()
  ExceptionCheck()
  ReleaseStringChars()
  ReleaseStringUTFChars()
  ReleaseStringCritical()
  Release<Type>ArrayElements()
  ReleasePrimitiveArrayCritical()
  DeleteLocalRef()
  DeleteGlobalRef()
  DeleteWeakGlobalRef()
  MonitorExit()
  PushLocalFrame()
  PopLocalFrame()


