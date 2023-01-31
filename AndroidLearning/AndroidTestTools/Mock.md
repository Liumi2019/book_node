# Mock

## Mock 介绍

Mock简单来理解，就是在测试过程中，对于某些不容易构造或者不容易获取的对象，用一个虚拟的对象来创建以便测试。而这个虚拟的对象就是mock对象。mock对象就是真实对象在调试期间的代替品。有时也将Mock服务称之为测试服务替身，或者测试服务档板。

## Mockito 介绍
[Mockito](https://site.mockito.org/)它是目前 Java 最多人使用的 Mocking framework。

### 1. 添加依赖
在项目中添加依赖项，build.gradle 添加：

``` build.gradle
repositories {
    mavenCentral()
}
dependencies {
    testCompile "org.mockito:mockito-core:+"
    androidTestCompile "org.mockito:mockito-android:+"
}
```

### 2. 基本使用

1. 使用 mock  

``` java
import static org.mockito.Mockito.*;

// mock creation
List mockedList = mock(List.class);
// or even simpler with Mockito 4.10.0+
// List mockedList = mock();

// using mock object - it does not throw any "unexpected interaction" exception
mockedList.add("one");
mockedList.clear();

// selective, explicit, highly readable verification
verify(mockedList).add("one");
verify(mockedList).clear();

// 打桩 thenReturn() 根据输入参数（mockedList.get(0)）返回对应的值（first）
// stubbing appears before the actual execution
when(mockedList.get(0)).thenReturn("first");

// the following prints "first"
System.out.println(mockedList.get(0));

// the following prints "null" because get(999) was not stubbed
System.out.println(mockedList.get(999));

```

1. 使用 spy
创建真实对象的间谍。当您使用间谍时，将调用真正的方法（除非方法被打桩）。

``` java

List list = new LinkedList();
List spy = spy(list);

//optionally, you can stub out some methods:
when(spy.size()).thenReturn(100);  // 被打桩，spy.size() 返回固定值 100

//using the spy calls *real* methods
spy.add("one");
spy.add("two");

//prints "one" - the first element of a list
System.out.println(spy.get(0));

//size() method was stubbed - 100 is printed
System.out.println(spy.size()); // 固定值 100

//optionally, you can verify
verify(spy).add("one");
verify(spy).add("two");

```


