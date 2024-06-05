# 中断 和 LockSupport

## 一、中断

在Java中，线程中断（Thread Interruption）是一种用于协作地停止线程执行的方法。线程中断机制使得一个线程可以请求另一个线程停止它正在执行的任务。

### 1. 中断线程的基本方法
Java提供了几种用于中断线程的方法：
- **`interrupt()`**: 用于中断线程。这个方法会设置线程的中断状态，即将线程的中断标志设为 `true`。
- **`isInterrupted()`**: 用于检查线程是否被中断。它会返回线程的中断状态，但不会清除中断标志。
- **`interrupted()`**: 一个静态方法，用于检查当前线程是否被中断，并清除中断标志。如果线程被中断，它会返回 `true`，并清除中断状态。

### 2. 中断的工作机制
当线程调用 `interrupt()` 方法时，具体的行为取决于线程的状态：
- 如果线程处于 **阻塞状态**（例如 `sleep()`、`wait()`、`join()` 等方法），会抛出 `InterruptedException` 异常，并清除线程的中断状态。
- 如果线程处于 **运行状态**，则仅仅设置线程的中断标志位【不会停止】，线程可以通过检查这个标志位来决定是否终止执行。

### 3. 处理中断
通常，在编写线程任务时需要处理可能的中断，以实现良好的线程协作。以下是一个示例：

```java
public class InterruptExample implements Runnable {
    @Override
    public void run() {
        try {
            while (!Thread.currentThread().isInterrupted()) {
                System.out.println("Thread is running...");
                // 模拟一些工作
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("Thread was interrupted during sleep.");
            // 重设中断状态
            Thread.currentThread().interrupt();
        }
        System.out.println("Thread exiting.");
    }

    public static void main(String[] args) {
        Thread thread = new Thread(new InterruptExample());
        thread.start();

        // 运行一段时间后中断线程
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread.interrupt();
    }
}
```

### 4. 处理中断的最佳实践
- **循环检查中断状态**: 在线程执行循环任务时，定期检查中断状态，以便线程能及时响应中断请求。
- **清理资源**: 当线程检测到中断时，应该进行必要的资源清理操作，以确保程序状态的一致性。
- **重设中断状态**: 捕获 `InterruptedException` 异常后，可以根据需要重设中断状态（调用 `Thread.currentThread().interrupt()`），以便调用者知道线程被中断了。

### 5. 中断的应用场景
- **优雅停止线程**: 在线程需要被停止时，使用中断机制是一种优雅的方式，而不是直接使用 `stop()` 方法（已被废弃）。
- **资源管理**: 在阻塞操作（如I/O操作）中，响应中断可以帮助程序更好地管理资源。
- **协作取消**: 在需要多个线程协作完成任务时，通过中断机制可以有效地实现任务取消。

### 总结
Java的线程中断机制提供了一种协作式的线程停止方法，通过使用 `interrupt()` 方法可以设置线程的中断标志，线程可以通过检查中断标志或捕获 `InterruptedException` 来响应中断请求。处理线程中断时应遵循最佳实践，确保资源的正确释放和程序的正常运行。



## 二、LockSupport

`LockSupport`是Java中的一个工具类，提供了一组基本的线程阻塞和唤醒机制。与`ReentrantLock`和`synchronized`相比，`LockSupport`提供了更底层和灵活的线程控制方法。以下是对`LockSupport`与`synchronized`和`ReentrantLock`的对比及其使用的详细介绍。

### 1.  synchronized

- **基本概念**：`synchronized`关键字用于修饰方法或代码块，以确保同一时刻只有一个线程可以执行被同步的代码。
- **使用方式**：
  ```java
  public synchronized void syncMethod() {
      // synchronized method
  }

  public void syncBlock() {
      synchronized (this) {
          // synchronized block
      }
  }
  ```
- **特性**：
  - 自动加锁和解锁，简洁明了。
  - 适用于简单的互斥需求。
  - 等待和通知机制通过`wait`, `notify`, `notifyAll`实现。

### 2. ReentrantLock

- **基本概念**：`ReentrantLock`是`Lock`接口的一个实现类，提供了显式的锁操作，更加灵活和功能丰富。
- **使用方式**：
  ```java
  private final ReentrantLock lock = new ReentrantLock();

  public void lockMethod() {
      lock.lock();
      try {
          // critical section
      } finally {
          lock.unlock();
      }
  }
  ```
- **特性**：
  - 支持公平锁和非公平锁。
  - 提供了条件变量（`Condition`），可以更灵活地进行线程间的等待和通知。
  - 可以响应中断和超时。

### 3. LockSupport

- **基本概念**：`LockSupport`提供了一组底层的线程阻塞和唤醒工具方法，直接使用基于许可的机制来控制线程的阻塞和唤醒。
- **使用方式**：
  ```java
  import java.util.concurrent.locks.LockSupport;

  public class LockSupportExample {
      public static void main(String[] args) {
          Thread thread = new Thread(() -> {
              System.out.println("Thread is going to park");
              LockSupport.park();
              System.out.println("Thread is unparked");
          });

          thread.start();

          try {
              Thread.sleep(2000); // Simulate some work
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
          }

          System.out.println("Main thread is going to unpark the thread");
          LockSupport.unpark(thread);
      }
  }
  ```
- **特性**：
  - 提供`park`和`unpark`方法用于阻塞和唤醒线程。
  - 基于许可（permit）的机制，线程可以通过`unpark`方法获得许可，再调用`park`方法时如果有许可就不会阻塞。
  - 不需要与特定的锁或条件变量关联，使用更灵活。

### 4. 对比总结

1. **锁的管理**：
   - `synchronized`：隐式锁管理，由JVM自动处理，简单易用。
   - `ReentrantLock`：显式锁管理，提供更多控制和功能，如公平锁、条件变量等。
   - `LockSupport`：不直接管理锁，而是提供底层的阻塞和唤醒机制，灵活但需要更细致的控制。

2. **等待和通知机制**：
   - `synchronized`：使用`wait`, `notify`, `notifyAll`进行线程等待和通知。
   - `ReentrantLock`：通过`Condition`对象的`await`, `signal`, `signalAll`进行更灵活的线程等待和通知。
   - `LockSupport`：直接使用`park`和`unpark`进行线程阻塞和唤醒，没有条件变量的概念。

3. **灵活性和复杂度**：
   - `synchronized`：简单易用，但功能较少，适用于简单的互斥和同步场景。
   - `ReentrantLock`：功能强大，灵活性高，但使用复杂，需要显式的锁操作和异常处理。
   - `LockSupport`：最灵活，提供最低层次的线程控制，但也最复杂，适用于需要精细控制线程状态的高级场景。

### 5. 示例应用

#### `synchronized` 示例

```java
public class SynchronizedExample {
    private final Object lock = new Object();

    public void synchronizedMethod() {
        synchronized (lock) {
            // critical section
        }
    }
}
```

#### `ReentrantLock` 示例

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final Lock lock = new ReentrantLock();

    public void reentrantLockMethod() {
        lock.lock();
        try {
            // critical section
        } finally {
            lock.unlock();
        }
    }
}
```

#### `LockSupport` 示例

```java
import java.util.concurrent.locks.LockSupport;

public class LockSupportExample {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("Thread is going to park");
            LockSupport.park();
            System.out.println("Thread is unparked");
        });

        thread.start();

        try {
            Thread.sleep(2000); // Simulate some work
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        System.out.println("Main thread is going to unpark the thread");
        LockSupport.unpark(thread);
    }
}
```

通过这些对比和示例，可以看出`LockSupport`提供了更底层、更灵活的线程阻塞和唤醒机制，而`synchronized`和`ReentrantLock`则提供了更高层次、更易于使用的锁和同步控制。根据具体需求和场景，选择合适的工具来实现线程同步和并发控制。

















 

