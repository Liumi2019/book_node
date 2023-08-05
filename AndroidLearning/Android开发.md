# Android 开发

## 构键工具

### 1. Ninja

Ninja 是一个专注于速度的小型构建系统。它与其他构建系统在两个主要方面不同：
1. Ninja的输入文件被设计为由更高级的构建系统生成。
2. Ninja被设计为尽可能快地运行构建, 其他构建系统基于高级语言，而Ninja基于汇编。

Ninja基于汇编，专注于速度，不支持分支、循环等流程控制，也不支持逻辑运算，但它允许以其它语言如来维护这些复杂的编译流程和逻辑。例如，我们可以采用Makefile, go, python等等来维护编译的流程和逻辑
可以看到，ninja的构建文件，书写起来是不很方便的，所以，我们需要一些ninja构建文件的生成器。这些生成器就是一些元构建系统(meta-build system)，例如Blueprint、CMake等等。Ninja的基于底层实现使其非常适合嵌入到这些功能更强大的构建系统中。

### 2. Blueprint

Blueprint 是一个元构建系统，该系统读取Blueprint文件来描述需要构建的模块，并生成一个Ninja清单来描述需要运行的命令及其依赖项。 在大多数构建系统使用内置规则或特定领域语言来描述将模块描述转换为构建规则的逻辑。Blueprint将其委托给使用Go编写的针对每个项目的构建逻辑。 对于大型，异构项目，这允许以高级语言维护固有、复杂的构建逻辑，同时仍可以通过修改易于理解的Blueprint文件来对单个模块进行简单更改。
前面在Ninja的构建文件一节我们提到了，Blueprint是ninja构建文件的生成器。android编译系统soong集成了Blueprint，Blueprint可将我们编写的android.bp解析生成一个ninja构建文件。我们在编译一个模块时，只需要将这个模块的android.bp文件配置好，编译系统会自动为这个模块生成ninja清单，最终使用ninja来调用gcc、clang、java、dex、aapt2等等命令来构建模块。

### 3. kati
kati 是Google开发的一个实验性的GNU make clone，kati的主要目标是加快Android的增量构建。目前，kati本身并不能提供更快的构建。 而是将Makefile转换为ninja。一开始kati是用go语言开发的，但作者发现使用go写的有性能问题，后来作者又用C++进行了重写，也就是kati变成了ckati(作者的原文描述可以查看android源码目录下的：build/kati/INTERNALS.md)。后续提到kati时，如不特别指出，即是指ckati。
简单点说，kati就是一个转换工具，它可以将Makefile和.mk文件转换为ninja。
android源码目录下的：prebuilts/build-tools下有预置的kati，Android 7.0及以上版本，编译源码时会自动使用kati。大多数情况下，我们不会直接使用kati，但如果你还想了解更多kati的相关信息，可以访问：https://github.com/google/kati。也可以查看android源码目录下的 build/kati/INTERNALS.md 和 build/kati/README.md。

### 4. soong
在android 6.0版本之前，编译android源码采用的是基于make的编译系统(make-based build system)，也就是android的各个库、APK等等目标文件都是采用make来构建的。 但是，由于make在编译时表现出效率不够高、增量编译速度慢等问题，Google在android 7.0版本引进了编译速度更快的soong来替代make。
Soong集成了Ninja, 而Ninja专注于速度，没有条件或流程控制语句，也不支持逻辑运算。但它允许以其它语言如来维护这些复杂的编译流程和逻辑。例如，我们可以继续采用makefile, 或者采用go语言来维护编译流程和逻辑。上面已经提到了Ninja，Blueprint， kati等等好几种工具，为了完整、快速的构建一个android系统，就需要一个“管家”来协调这些工具。例如，将.bp转换成ninja时使用Blueprint, 将Makefile转换成ninja时使用kati。这个选择转换工具、选择解析框架、解析维护构建逻辑的“管家”就是soong。
编译android源码时，soong也会被自动使用，我们可以和原来一样：
首先，source build/envsetup.sh。
然后，lunch选择target。
最后，使用m、mm、mmm或者make来编译指定的模块或者整个系统。
使用soong构建项目，主要工作是编写bp文件。关于bp文件我们会在接下来的第5节中详细介绍。

### 5. androidmk
soong中还集成了一个非常有用的工具androidmk。androidmk可以将android.mk转换成android.bp。我们可以使用androidmk来转换代码中已有的android.mk，以此来减少重写android.bp的工作量。
在现有版本，由于kati工具的存在，继续使用android.mk也是没有问题的。但是，也许在将来的某一天，Google可能会不再支持Makefile。因此，建议新增的模块采用android.bp，存量的android.mk，在条件允许的情况下，也应当尽量转换成android.bp。

### 6. bpfmt
为了方便格式化，Soong还包含了一个用于格式化Android.bp文件的工具bpfmt，类似于gofmt。 

ninja，Blueprint，kati，androidmk与Soong的关系和作用
kati可以将Android.mk文件转换成ninja文件。
androidmk可以将Android.mk文件转换成Android.bp文件
Blueprint可以将Android.bp文件转换成ninja文件。
Blueprint，kati，androidmk由Soong调用和协调，一起合作完成android源码的构建。

### 7. Android.bp

#### 7.1 bp文件的命名与文件格式
soong的编译配置文件以.bp结尾，通常命名为Android.bp，但也有少数情况不以Android.bp命名。例如：frameworks/rs/support.bp。与Makefile一样，使用soong编译前，会遍历所有以bp为后缀名的文件。因此，soong的编译配置文件只要以.bp结尾即可。

#### 7.2 模块（module）
bp文件中的模块（module） 以模块类型(module type)开头，后面跟着一系列的属性(property)。每个模块都必须具有一个属性名为name的属性，并且name的属性值在所有Android.bp文件中必须是唯一的。bp文件的内容与JSON、Bazel BUILD很像，模块的格式为：

``` shell
[module type] {
    name: "[name value]",
    [property1 name]："[property1 value]",
    [property2 name]："[property2 value]",
}
```

常见的模块类型(module type)有：
cc_library,
cc_library_headers,
cc_library_shared,
cc_library_static,
android_app,
android_app_certificate,
java_library,
java_library_static,
java_sdk_library等等。

#### 7.3 文件列表
一个属性的属性值为一系列文件时，属性值也可以采用全局匹配模式和输出路径扩展。

#### 7.4 类型和变量
Android.bp文件中，变量和属性是强类型的，变量是基于首次分配动态地创建的，属性是根据模块类型静态地创建的。
支持的类型有：
Bool (true or false)
Integers (int)
Strings ("string")
Lists of strings (["string1", "string2"])
Maps ({key1: "value1", key2: ["value2"]})

#### 7.5 Android.mk和Android.bp的区别
Android.mk文件通常可以包含多个同名模块（例如，用于库的静态（static）和共享（shared）版本，用于不同主机（host）的版本，用于不同设备（device）版本）。

Android.bp文件的每个模块都需要唯一的名称，但是单个模块可以构建为多个变体。例如，通过添加host_supported：true。 包含多个同名模块的Android.mk被androidmk转换成Android.bp之后，将生成多个冲突的模块，这些模块必须手动处理，将同名模块改为一个具有多个变体的模块，这些在target：{android：{}，host：{}}块内的变体可以有区别，例如，引用不同的源文件，不同架构的共享库文件等等。

Soong故意不支持Android.bp文件中的大多数条件。 Google建议从构建中删除大多数条件。Soong将本地支持的大多数条件将转换为map属性。 构建模块时，将选择map中的属性之一，并且将其值追加到模块顶层的同名属性中。

### 8. 结语
Google在Android N就引入了Soong，目的是替换make。但是，很遗憾的是截止到Android Q，甚至Android R，它们的源码中都还存在大量的Makefile。也许make就是构建语言里的C语言，无论世界如何发展变化，它的生命力依然旺盛。未来的很长一段时间内Android.bp和Android.mk可能会一直共存，一起完成Android系统源码的构建。这并不代表我们可以不用去学习Android.bp，技术总是不断的发展，我们则需要不断的学习才能适应技术的发展。

### 参考
1. https://www.jianshu.com/p/f69d1c381182
2. [Soong 构建系统 —— Android 官网](https://source.android.com/docs/setup/build?hl=zh-cn)

## AndroidManifest.xml 文件
每一个 app 都应该包含一个 AndroidManifest.xml 文件，是 app 的配置信息。配置信息如下：
1. 提供 app 包名，在 android 系统里为一个标识
2. 为主件提供信息，配置启动、响应方式等
3. 配置 app 可访问的权限信息
4. 配置 app 的版本号信息

### 文件结构
1. \<manifest\> 元素
这是文件的根节点。它必须要包含\<application\>元素，并且指明 xmlns:android 和 package 属性。

2. xmlns:android
这个属性定义了Android命名空间。必须设置成"http://schemas.android.com/apk/res/android"。不要手动修改。

3. package
这是一个完整的Java语言风格包名。包名由英文字母（大小写均可）、数字和下划线组成。每个独立的名字必须以字母开头。
构建APK的时候，构建系统使用这个属性来做两件事：
3.1 生成 R.java 类时用这个名字作为命名空间（用于访问APP的资源）比如：package 被设置成 com.sample.teapot，那么生成的R类就是：com.sample.teapot.R
3.2 用来生成在manifest文件中定义的类的完整类名。比如package被设置成com.sample.teapot，并且activity元素被声明成 \<activity android:name=".MainActivity"\>，完整的类名就是 com.sample.teapot.MainActivity。包名也代表着唯一的application ID，用来发布应用。但是，要注意的一点是：在APK构建过程的最后一步，package 名会被 build.gradle 文件中的 applicationId 属性取代。如果这两个属性值一样，那么万事大吉，如果不一样，那就要小心了。

4. android:versionCode
内部的版本号。用来表明哪个版本更新。这个数字不会显示给用户。显示给用户的是versionName。这个数字必须是整数。不能用16进制，也就是说不接受"0x1"这种参数

5. android:versionName
显示给用户看的版本号。

6. \<uses-feature\> 元素
Google Play利用这个元素的值从不符合应用需要的设备上将应用过滤。这东西的作用是将APP所依赖的硬件或者软件条件告诉别人。它说明了APP的哪些功能可以随设备的变化而变化。使用的时候要注意，必须在单独的 \<uses-feature\> 元素中指定每个功能，如果要多个功能，需要多个 \<uses-feture\> 元素。

7. \<application\> 元素
此元素描述了应用的配置。这是一个必备的元素，它包含了很多子元素来描述应用的组件，它的属性影响到所有的子组件。许多属性（例如icon、label、permission、process、taskAffinity和allowTaskReparenting）都可以设置成默认值。

8. \<activity\> 元素
该元素声明一个实现应用可视化界面的Activity（Activity类子类）。这是 \<application\> 元素中必要的子元素。所有Activity都必须由清单文件中的 \<activity\> 元素表示。任何未在该处声明的Activity对系统都不可见，并且永远不会被执行。

## Android 测试

1. 在 Android Studio 中测试
对于基本测试需求，Android Studio 提供的一些功能可帮助您在该 IDE 中创建、运行和查看测试结果。使用 Android Studio 时，您可以通过指向和点击应用源代码中的某些代码来创建和运行针对特定类或方法的测试，使用菜单配置多个测试设备，并与测试矩阵工具窗口交互以直观呈现测试结果。如需详细了解如何使用 Android Studio 创建和管理测试，[请参阅在 Android Studio 中测试](https://developer.android.com/studio/test/test-in-android-studio)

2. 从命令行运行测试
若想实现更精细的控制，您可以通过命令行运行测试。命令行测试提供了一种简便的方法，可单独针对模块或 build 变体，或者针对二者的组合运行测试。通过 Android 调试桥 (adb) shell 运行测试，可以对要运行的测试进行最大程度的自定义。
从命令行运行测试在持续集成系统上也很有用，[从命令行进行测试](https://developer.android.com/studio/test/command-line)。

3. 高级测试
对于高级测试需求，您可能需要替换默认设置、配置 Gradle 选项或重构代码，以便分别在各自的模块中运行测试。如需详细了解如何针对特殊用例设置测试配置，请参阅[高级测试设置](https://developer.android.com/studio/test/advanced-test-setup).

### 在 Android Studio 中测试

#### 测试相关的工具、框架

1. JUnit 针对 Java 语言的测试，不依赖 Android API （本地测试）
2. Mockito、Mockk、Robolectric 部分依赖 Android API 测试使用 （本地测试）
3. Robolectric 模拟 Android 运行环境，实现在 JVM 测试 Java 代码 （本地测试）
3. Espresso Android UI 测试 （依赖设备测试）
4. JaCoCo 代码覆盖率统计

#### 测试类别

1. 本地单元测试
位于 module-name/src/test/java/，这些测试在计算机的本地 Java 虚拟机 (JVM) 上运行。纯java代码，不存在对android库的依赖，可以进行JUnit单元测试。
部分依赖，代码实现依赖注入，该类需要依赖Context等android对象的依赖，可以通过 Mock 或 Robolectric 等其它第三方框架实现JUnit单元测试或使用androidJunitRunner进行单元测试，借助模拟器模拟 Android 环境。

2. 插桩测试
位于 module-name/src/androidTest/java/ 在硬件设备或模拟器上运行。他们有权访问 Instrumentation API，可让您访问 Context 类等信息，并且允许您通过测试代码来控制受测应用。插桩测试内置于单独的 APK 中，因此它们都有自己的 AndroidManifest.xml 文件。此文件是自动生成的，但您可以在 $module-name/src/androidTest/AndroidManifest.xml 创建自己的版本，该版本将与生成的清单合并。在编写集成和功能界面测试来自动执行用户交互时，或者当您的测试具有您无法为其创建测试替身的 Android 依赖项时，可以使用插桩测试。
强依赖关系，如在Activity，Service等组件中的方法，其特点是大部分为private方法，并且与其生命周期相关，无法直接进行单元测试，可以进行Ecspreso等UI测试。

##### 1. 本地单元测试（JUnit 和 GoogleTest）

单元测试应该关注正常情况和边缘情况。边缘案例是人类测试人员和大型测试不太可能捕捉到的不常见场景。示例包括以下内容：
- 使用负数、零和边界条件的数学运算。
- 所有可能的网络连接错误。
- 损坏的数据，例如格式错误的 JSON。
- 保存到文件时模拟完整存储。
- 在进程中间重新创建的对象（例如设备旋转时的活动）。

要避免的单元测试，一些单元测试应该避免，因为它们的价值很低：
- 验证框架或库的正确操作的测试，而不是您的代码。
- 活动、片段或服务等框架入口点不应该有业务逻辑，因此单元测试不应该是优先事项。活动的单元测试没有什么价值，因为它们将主要覆盖框架代码并且需要更复杂的设置。UI 测试等插桩测试可以覆盖这些类。

1. 添加测试依赖项

build.gradle 添加：

``` build.gradle
dependencies {
  // Required -- JUnit 4 framework
  testImplementation "junit:junit:$jUnitVersion"
  // Optional -- Robolectric environment
  testImplementation "androidx.test:core:$androidXTestVersion"
  // Optional -- Mockito framework
  testImplementation "org.mockito:mockito-core:$mockitoVersion"
  // Optional -- mockito-kotlin
  testImplementation "org.mockito.kotlin:mockito-kotlin:$mockitoKotlinVersion"
  // Optional -- Mockk framework
  testImplementation "io.mockk:mockk:$mockkVersion"
}
```
测试使用断言库，例如junit.Assert、Hamcrest或 Truth。

``` java
import org.junit.Test;

import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;

class EmailValidatorTest {

  @Test  // 添加 @Test 
  public void emailValidator_CorrectEmailSimple_ReturnsTrue() {
    assertTrue(EmailValidator.isValidEmail("name@email.com"));
  }

}

```

2. 测试 Context 需要模拟器
测试中有部分依赖 Android 库，需要添加模拟器来测试。使用 Mockable Android 库和模拟框架（例如 Mockito或MockK），您可以在单元测试中对 Android 类的模拟行为进行编程。

##### 2. 插桩测试

添加 AndroidX 测试 API，包括：
- JUnit 4 测试AndroidJUnitRunner运行器
- 功能性 UI 测试（例如Espresso、UI Automator和Compose 测试）的 API 。

添加依赖项：
``` build.gradle
dependencies {
    androidTestImplementation "androidx.test:runner:$androidXTestVersion"
    androidTestImplementation "androidx.test:rules:$androidXTestVersion"
    // Optional -- UI testing with Espresso
    androidTestImplementation "androidx.test.espresso:espresso-core:$espressoVersion"
    // Optional -- UI testing with UI Automator
    androidTestImplementation "androidx.test.uiautomator:uiautomator:$uiAutomatorVersion"
    // Optional -- UI testing with Compose
    androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
}
```

### Benchmark（基准测试）

指标测试通常用于执行硬件抽象层 (HAL) 或直接与底层系统服务交互。

## 系统组件测试

### 测试类型
1. 插桩测试 
2. GoogleTest 测试
3. Junit 测试

### 简单配置

测试例子，系统组件 Android.bp 配置文件：

``` bp
android_test {
    name: "HelloWorldTests",
    srcs: ["src/**/*.java"],
    sdk_version: "current",
    static_libs: ["android-support-test"],
    certificate: "platform",
    test_suites: ["device-tests"],
}

```

请注意，开头的 **android_test** 声明表示这是一个测试。相反，如果包含 **android_app**，就表示这是一个 build 软件包。例如，在本例中，生成的测试 APK 将命名为 HelloWorldTests.apk。此外，此设置还可以为模块定义 make 目标名称，以便您可以使用 make [options] <HelloWorldTests> 构建测试模块及其所有依赖项。

### 插桩测试
插桩测试在硬件设备或模拟器上运行。他们有权访问 Instrumentation API，可让您访问 Context 类等信息，并且允许您通过测试代码来控制受测应用。插桩测试内置于单独的 APK 中，因此它们都有自己的 AndroidManifest.xml 文件。此文件是自动生成的，但您可以在 $module-name/src/androidTest/AndroidManifest.xml 创建自己的版本，该版本将与生成的清单合并。在编写集成和功能界面测试来自动执行用户交互时，或者当您的测试具有您无法为其创建测试替身的 Android 依赖项时，可以使用插桩测试。在平台测试中使用插桩测试的方式存在一些差异。

如果要设置一个新的测试模块，请在上述某个位置按照 AndroidManifest.xml 和 Android.mk 的设置进行操作

#### 添加新测试
通常，您的团队已经建立了用于签入代码的位置和用于添加测试的位置的既定模式。大多数团队拥有一个单独的 git 存储库，或者与其他团队共享一个，但有一个包含组件源代码的专用子目录。
假设您的组件源的根位置在 \<component source root> ，大多数组件下面都有src和tests文件夹，以及一些额外的文件，例如Android.mk （或分解成额外的.mk文件），清单文件AndroidManifest.xml和测试配置文件“AndroidTest.xml”。
由于您正在添加一个全新的测试，您可能需要在组件src旁边创建tests目录，并用内容填充它。
在某些情况下，由于需要将不同的测试套件打包到单独的 apk 中，您的团队可能有更多的目录结构正在tests中。在这种情况下，您需要在tests下创建一个新的子目录。
无论结构如何，您最终都会使用与示例 gerrit 更改中的instrumentation目录中的文件类似的文件填充tests目录或新创建的子目录。以下部分将详细解释每个文件。

### JAR 测试

``` bp
java_test_host {
    name: "HelloWorldHostTest",

    test_suites: ["general-tests"],

    srcs: ["test/**/*.java"],

    static_libs: [
        "junit",
        "mockito",
    ],
}

```

开头的 java_test_host 声明表示这是一个 JAR 主机测试。如果指定了 java_test_host 模块类型（在代码块的开头），则需要 name 设置。此设置会为模块命名，并且生成的 JAR 将与模块名称相同，不过带有 .jar 后缀。在本例中，生成的测试 JAR 将命名为 HelloWorldHostTest.jar。此外，此设置还可以为模块定义 make 目标名称，以便您可以使用 make [options] \<HelloWorldHostTest> 构建测试模块及其所有依赖项。

### Test Mapping

Test Mapping 是一种基于 Gerrit 的方法，让开发者能够直接在 Android 源代码树中创建提交前规则和提交后规则，并将要测试的分支和设备的决策留给测试基础架构本身。Test Mapping 定义是名为 TEST_MAPPING 的 JSON 文件，该文件可放置在任何源目录中。

Atest 可以使用 TEST_MAPPING 文件在相关目录中运行提交前测试。借助 Test Mapping，您只需在 Android 源代码树中进行简单的更改，即可将同一组测试添加到提交前检查。

Test Mapping 组通过测试组进行测试。测试组的名称可以是任何字符串。例如，presubmit 可用于在验证更改时运行的测试组。postsubmit 测试可在更改合并后用于验证 build。

### Atest

Atest 是一个命令行工具，允许用户在本地构建、安装和运行 Android 测试，从而大大加快测试重新运行的速度，而无需了解Trade Federation 测试工具命令行选项。本页介绍如何使用 Atest 运行 Android 测试。






