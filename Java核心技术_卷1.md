# Java 基础知识
## 一、当前进展
除了第10、11章关于图形界面相关的，其他的已读一遍，入门 Java 基础知识。

## 二、读书心得
- Java 开发依赖虚拟机运行环境，每个文件一个公开的类
- 每一个可定义到一个包内，同一个包可以相互访问，非同一个包访问需要导入包
- 函数内部可以改变输入参数的值，类似C++的引用传值，但不是引用传值
- Java 只允许单继承，但可以实现多个接口
- 每一个公开类都可以实现一个静态的 main() 函数，用于运行当前类的成员函数
- Java 的 JDK 包含大量的已实现的包，用户可以很方便的使用和实现自己的程序
- Java 不采用解释性编译器，使用**即时编译器**

## 三、Java 并发编程
### 1. 创建线程
#### 1.1 继承 Thread 类，重写 run() 方法

**不推荐使用**

``` java
public class FirstThreadTest extends Thread {
    int i = 0;
    // 重写run方法，run方法的方法体就是现场执行体  
    public void run() {
        for (; i < 100; i++) {
            System.out.println(getName() + "  " + i);
        }
    }
}

// 使用
FirstThreadTest thred1 = new FirstThreadTest();
thred1.start();
```

#### 1.2 实现接口 Runnable

实现接口可以使用，但是比较麻烦，推荐使用 lambda 表达式，只有一个函数的接口推荐使用 lambda 表达式。

```java
// 实现接口
public class RunnableThreadTest implements Runnable {
    private int i;
    public void run() {
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}

// lambda 表达式
Runnable rtt = () -> {  
    for (int i = 0; i < 100; i++) {
        System.out.println(Thread.currentThread().getName() + " " + i);
    }
}

// 使用
RunnableThreadTest rtt = new RunnableThreadTest();
new Thread(rtt, "新线程1").start();

```

#### 1.3 通过 Callable 接口和 Future 包装器创建线程

Callable 接口搭配 uture 包装器，可以得到线程的返回值，可以抛出异常
Runnable 接口不能返回值，也不能抛出异常

``` java
public class CallableThreadTest implements Callable<Integer> {
    // 重写 call() 函数，可有返回值 Runnable 没有返回值
    @Override
    public Integer call() throws Exception {
        int i = 0;
        for (; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
        return i;
    }

    // 使用，需要和 FutureTask 包装器一起使用，获取返回值
    CallableThreadTest ctt = new CallableThreadTest();
    FutureTask<Integer> ft = new FutureTask<>(ctt);

    new Thread(ft, "有返回值的线程").start();
    try {
        System.out.println("子线程的返回值：" + ft.get());
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}

```

### 2. 线程同步

#### 2.1 锁对象
1. synchronized 对象
1.1 可以自动释放，不管是正常执行，还是抛出异常
1.2 独占式锁
1.3 可重入，可以继续调用 synchronized 管理的函数
1.4 直接唤醒全部线程组

2. ReentrantLock
1.1 可重入，含有计数器
1.2 手动释放和加锁
1.3 绑定 Condition（条件）类，可以分组唤醒多个线程

#### 2.2 原子对象
对一个小的对象加锁来保证同步性，太耗费资源了。解决方法是借助硬件实现原子变量来管控较小的临界对象。

原子变量类可以分为 4 组：
1. 基本类型
1.1 AtomicBoolean - 布尔类型原子类
1.2 AtomicInteger - 整型原子类
1.3 AtomicLong - 长整型原子类
2. 引用类型
2.1 AtomicReference - 引用类型原子类
2.1 AtomicMarkableReference - 带有标记位的引用类型原子类
2.3 AtomicStampedReference - 带有版本号的引用类型原子类
3. 数组类型
3.1 AtomicIntegerArray - 整形数组原子类
3.2 AtomicLongArray - 长整型数组原子类
3.3 AtomicReferenceArray - 引用类型数组原子类
4. 属性更新器类型
4.1 AtomicIntegerFieldUpdater - 整型字段的原子更新器。
4.2 AtomicLongFieldUpdater - 长整型字段的原子更新器。
4.3 AtomicReferenceFieldUpdater - 原子更新引用类型里的字段。



