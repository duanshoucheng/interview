当然需要[官网镇贴](https://developer.android.google.cn/ndk/guides/concepts.html)。
Android NDK 是一组允许您将 C 或 C++（“原生代码”）嵌入到 Android 应用中的工具，使用场景：
- 在平台之间移植其应用。
- 重复使用现有库，或者提供其自己的库供重复使用。
- 在某些情况下提高性能，特别是像游戏这种计算密集型应用
 
名词解析：
- ndk-build：ndk-build 脚本用于在 NDK 中心启动构建脚本。使用 ndk-build 编译原生（.so、.a）库。
- Java 原生接口 (JNI)：JNI 是 Java 和 C++ 组件用以互相沟通的接口。 本指南假设您具备 JNI 知识；如需了解相关信息，请查阅 Java 原生接口规范。
- 原生共享库：NDK 从原生源代码构建这些库或 .so 文件。
- 原生静态库：NDK 也可构建静态库或 .a 文件，您可以关联到其他库。
