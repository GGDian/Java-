* Integer、Long、Short、Byte 缓存 -127 ~ 128
  * Integer 可通过设置 java.lang.Integer.IntegerCache.high 扩大缓存区间
* Character 缓存了 0 ~ 127
* Float 和 Double 没有缓存的意义  
* 之所以 Integer、Long、String 这些类的对象可以缓存，是因为它们是**不可变类**



* 基础类型包装类的缓存池使用一个**数组**进行缓存
* 而 String 类型，JVM 内部使用 HashTable 进行缓存，数组长度 60013，可以通过 **-XX:StringTableSize=N** 设置长度；并且不可以动态扩容；但可以进行 rehash



### 创建和回收

* 使用双引号来表示一个字符串的时候，这个字符串就会进入到 String Pool 中；**前提是这个类已经加载到 JVM 中的类**
*  String#intern()
* 只要 String Pool 中的 String 对象对于 GC Roots 来说不可达，那么它们就是可以被回收的。
* 如果 Pool 中对象过多，可能导致 YGC 变长，因为 YGC 的时候，需要扫描 String Pool



### String Pool 的实现

* Java 6，可以通过 -XX:MaxPermSize 可设置永久代大小，但不管我们设置多少，这个值都是固定的；即使用 String.intern 过多，可能会抛出 OutOfMemoryError
* Java 7，将 String Pool 移到了堆中