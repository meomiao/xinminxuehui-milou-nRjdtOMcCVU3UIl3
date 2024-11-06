
![](https://files.mdnice.com/user/39258/5935697b-6c9c-4c2a-b805-7121e54673ce.jpg)



> 本博客所有文章除特别声明外，均采用[CC BY\-NC\-SA 4\.0](https://github.com)许可协议。转载请注明来自 [唯你](https://github.com)


# 使用场景


JAVA 与 Rust 互操作让 Rust 可以背靠 Java 大生态来做更多事情，而 Java 也可以享受 Rust 语言特性的内存安全，所有权机制，无畏并发。


互操作的典型场景包括：


* 性能优化：利用 Rust 处理计算密集型任务，提高 Java 应用的整体性能。
* 系统级编程：结合 Rust 的底层控制能力与 Java 的高级抽象，实现更高效的系统交互。
* 跨平台开发：使用 Rust 编写核心逻辑，通过 JNI 在不同平台上与 Java 交互，实现高效跨平台开发。
* 安全关键应用：在金融、医疗等领域，利用 Rust 处理敏感数据和核心功能，保证高度安全性。
* 实时系统：在游戏引擎、音频处理等延迟敏感的应用中，使用 Rust 处理时间关键部分。


# 背景知识


## **JNI**


![image.png](https://17fc544.webp.li/image.png)


* 全称 Java Native Interface，它允许 Java 代码与其他语言（如 C 或 C\+\+）编写的应用程序进行互操作。
* [JNI Specification](https://github.com "JNI Specification")：这是 JNI 的官方规范，详细描述了 JNI 的使用方法、接口和功能。


## **Java 虚拟机（JVM）**


![image 1.png](https://17fc544.webp.li/image%201.png)


JNI 是 Java 虚拟机的一部分，JVM 在启动时为每个线程创建一个 JNI 环境。JNI 环境包括指向 JVM 内部数据结构的指针，这些数据结构用于存储 Java 对象、方法和字段的信息。


## JNIEnv (JNI 环境)


![image 2.png](https://17fc544.webp.li/image%202.png)


* `JNIEnv`是一个指向结构体的指针，代表当前线程的 JNI 环境。它包含所有 JNI 相关函数的指针，让你能在本地代码中使用这些函数。每个线程都有自己独立的`JNIEnv`，所以不能在不同线程间传递这个指针。
* 可以将`JNIEnv`视为一个"翻译器"。当 Rust 代码需要与 Java 交互时，它通过这个"翻译器"发送请求，当调用 Java 方法或获取 Java 对象的属性。每个线程都拥有自己独立的"翻译器"，这确保了各线程与 Java 交互时的独立性。


## 另外


* 当 Java 代码调用本地方法时，JVM 会加载相应的本地库并创建一个`JNIEnv`指针。
* 本地代码可以使用这个指针访问 JNI 提供的函数，进行 Java 对象的操作。
* 每个线程有独立`JNIEnv`，保证线程安全。新线程需调用`AttachCurrentThread`获取对应`JNIEnv`。
* JNI 提供了数据类型转换机制，实现 Java 与 C/C\+\+之间的数据传递。


在 Rust 生态中使用 jni 0\.21\.1 库可以实现与 Java 代码的交互。


# JNI 0\.21\.1 简介


该项目为 Rust 提供了完整的 JNI 绑定，允许：


* 使用 Rust 代码与 Java 库进行交互，调用 Java 方法和访问 Java 对象。
* 从 Rust 代码中使用 Java 类和接口。
* 实现跨语言的高效数据交换。
* 利用 Rust 的性能优势和 Java 的成熟生态系统


跨平台 UI 框架 Flutter 源码中的 MethodChannel 实现了 Dart 与 Android 层的通信，其底层 C\+\+也是通过 JNI 调用插件中的 onMethodCall 来实现的。这与上述 jni 0\.21\.1 采用了相同的思路，但存在以下不同点：


* 语言特性和类型安全：
Rust 的`jni`库提供了一种更安全的方式来处理 Java 对象和方法调用。它利用 Rust 的所有权系统来减少潜在的内存错误，使得在 Rust 中使用 JNI 时更易于管理资源和避免常见错误。
* 多平台支持：jni 0\.21\.1 提供了更广泛的跨平台支持。


# 如何运行示例



> 示例源码请[阅读原文](https://github.com "阅读原文"):[veee加速器](https://liuyunzhuge.com)，见原文底部**源码获取**


在 Windows 11 环境下运行示例时，笔者遇到了两个问题：


1. Windows 自带的 PowerShell 无法直接执行 Makefile
2. 由于 Rust 配置了特定目标平台，出现了不明原因的编译错误


以下是解决这些问题的方法：


## 在 MinGW\-w64 中执行 Makefile


1. 确保已在 MinGW\-w64 环境中安装 `mingw32-make` 工具（通常随 MinGW\-w64 一起安装）
2. 打开 MinGW\-w64 命令行
3. 导航至 Makefile 所在目录
4. 执行以下命令



```
//当前示例中是makefile，直接执行mingw32-make -f makefile即可
mingw32-make -f YourMakefileName

```

## 确认当前 Rust 环境


举例来说，笔者在 C:\\Users\\xxx.cargo 目录下配置了 config.toml 文件：



```
[build]
target = "aarch64-linux-android"

```

这导致在 Windows 上使用 mingw32\-make 来编译针对 Android 平台的 Rust .so 文件，造成了混乱并引发了莫名其妙的编译错误。**解决方法是删除不必要的 config.toml 文件，确保当前运行环境与目标平台（如 Windows）一致。**。


输出结果：



```
Hello, josh!
[B@2f92e0f4
factCallback: res = 720
counterCallback: count = 1
counterCallback: count = 2
counterCallback: count = 3
counterCallback: count = 4
counterCallback: count = 5
Invoking asyncComputation (thread id = 1)
asyncCallback: thread id = 23, progress = 0%
asyncCallback: thread id = 23, progress = 10%
asyncCallback: thread id = 23, progress = 20%
asyncCallback: thread id = 23, progress = 30%
asyncCallback: thread id = 23, progress = 40%
asyncCallback: thread id = 23, progress = 50%
asyncCallback: thread id = 23, progress = 60%
asyncCallback: thread id = 23, progress = 70%
asyncCallback: thread id = 23, progress = 80%
asyncCallback: thread id = 23, progress = 90%
asyncCallback: thread id = 23, progress = 100%

```

# 示例简析


让我们深入分析示例中 asyncComputation 的流程。其核心目的是在 Rust 端执行一个异步计算，同时 Rust 端会调用 Java 端来报告计算进度。


![image.png](https://17fc544.webp.li/20241106070312.png)


整体流程图说明：


1. Java 的 main()方法调用 asyncComputation()。
2. asyncComputation()通过 JNI 调用 Rust 的 Java\_HelloWorld\_asyncComputation()函数。
3. Rust 函数创建一个新线程来执行异步计算。
4. 在新线程中，Rust 执行计算并周期性地调用 Java 的 asyncCallback()方法报告进度。
5. 当 Rust 完成计算后，控制权返回到 Java 的主线程。


这个过程展示了 Java 调用 Rust（步骤 1）和 Rust 回调 Java（步骤 4）的双向交互。


## 源码如下


![image 3.png](https://17fc544.webp.li/image%203.png)


# 1 将一个 HelloWorld 类实例传递给 Rust 端，这对应下方 Rust 侧实现中 \#3 处的 callback 对象


![image 4.png](https://17fc544.webp.li/image%204.png)


### Rust 实现说明


1. `JNIEnv` 参数：详见上述相关概念中的解释。
2. `JClass`：代表调用此本地方法的 Java 类引用，主要用于访问类级别的静态方法和字段。
3. `callback`：Java 中新创建的 HelloWorld 对象实例。
4. 获取 JVM 对象：因为 `env` 对象不支持线程间传递和共享（仅实现了 Send），而 JVM 支持。通过 JVM 对象可在线程间传递，并最终获得 `env`。
5. 创建全局引用：获取 HelloWorld() 实例对象的全局引用，防止被垃圾回收。
6. 线程安全：每个线程都有自己的 `JNIEnv`，确保线程安全。在新线程中需调用 `AttachCurrentThread` 获取对应的 `JNIEnv`。
7. 反向调用 Java：通过 `env` 反向调用 Java 代码。调用对象是新创建的 HelloWorld 实例，回调方法是其中的 asyncCallback。在 JNI 中，`"(I)V"` 是方法签名，描述了 Java 方法的参数和返回类型。`"(I)V"` 表示接受一个整数参数并返回 `void` 的方法。在这里，asyncCallback 方法接收一个整数（`progress`）作为参数，无返回值。


# 总结


1. Java 与 Rust 互操作让两种语言优势互补，提高性能和安全性，适用于多种场景如性能优化、系统级编程和跨平台开发。
2. JNI（Java Native Interface）是实现 Java 与 Rust 互操作的关键技术，允许 Java 代码与其他语言编写的应用程序进行交互。
3. 通过示例分析，我们了解了 Java 调用 Rust 函数和 Rust 回调 Java 方法的双向交互过程，展示了两种语言之间的无缝协作。


# 参考链接


[https://github.com/jni\-rs/jni\-rs](https://github.com "https://github.com/jni-rs/jni-rs")


[JNI APIs and Developer Guides](https://github.com "JNI APIs and Developer Guides")


[Leveraging Rust in our high\-performance Java database](https://github.com "Leveraging Rust in our high-performance Java database")


