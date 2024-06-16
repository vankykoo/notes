# java 对象内存布局与对象头

## 一、创建对象时内存布局

在Java中，当我们创建一个对象时，该对象在JVM（Java虚拟机）中的内存布局可以分为几个重要的部分。这些部分主要包括对象头（Object Header）、实例数据（Instance Data）和对齐填充（Padding）。以下是对这些部分的详细介绍：

1. **对象头（Object Header）**：
   对象头在JVM中占据的内存包括以下两个部分：
   - **Mark Word**：用于存储对象的**哈希码（hash code）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID**等信息。Mark Word 是一个可变数据结构，其具体内容在不同状态下是不同的。
   - **Class Metadata Address**：一个指向对象的类元数据的指针。通过这个指针，JVM可以知道对象是哪个类的实例，并从类元数据中获取对象的类型信息。

2. **实例数据（Instance Data）**：
   实例数据是对象的实际数据字段。包括父类继承下来的字段以及对象自己定义的字段。实例数据按照字段声明的顺序存储，但具体的顺序可能会因为字段对齐而有所调整。

3. **对齐填充（Padding）**：
   JVM的对象内存布局要求对象的起始地址必须是8字节的整数倍（某些JVM实现可能要求是16字节）。因此，在对象头和实例数据之后，可能会存在一些填充字节，以确保下一个对象的起始地址符合对齐要求。

具体来说，对象的内存布局可以表示如下：

```
| Object Header (Mark Word + Class Metadata Address) | Instance Data (Fields) | Padding |
```

### 示例

假设我们有一个简单的Java类如下：

```java
public class Example {
    int a;       // 4 bytes
    long b;      // 8 bytes
    byte c;      // 1 byte
}
```

在默认情况下（未考虑压缩指针的情况下），对象的内存布局可能如下：

1. **Object Header**：
   - Mark Word：8字节
   - Class Metadata Address：8字节
     共16字节。

2. **Instance Data**：
   - `int a`：4字节
   - `long b`：8字节
   - `byte c`：1字节
     为了对齐，下一个字段将从8的倍数位置开始，因此需要填充7字节。
     共20字节（4+8+1+7）。

3. **Padding**：
   由于对象头加上实例数据已经是36字节，符合8字节的对齐要求，因此不需要额外的填充。

最终对象的内存布局为：

```
| Mark Word (8 bytes) | Class Metadata Address (8 bytes) | int a (4 bytes) | long b (8 bytes) | byte c (1 byte) | Padding (7 bytes) |
```

因此，这个对象在JVM中的内存占用是36字节。需要注意的是，不同JVM实现以及不同配置（如启用压缩指针）可能会对具体的内存布局产生影响。

### 内存布局总结

- **对象头**：包含Mark Word和类元数据指针，占用固定大小。
- **实例数据**：包含对象的字段，根据字段类型和对齐要求排列。
- **对齐填充**：用于满足对象的对齐要求。

了解对象的内存布局有助于优化Java程序的性能，特别是在高性能应用中。