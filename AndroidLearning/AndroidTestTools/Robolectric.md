# Robolectric

## 介绍
在 Android 模拟器或设备上运行测试很慢！构建、部署和启动应用程序通常需要一分钟或更长时间。那不是做TDD的方法。一定会有更好的办法。Robolectric是一个为 Android 带来快速可靠的单元测试的框架。测试在您工作站上的 JVM 内运行几秒钟。

### 测试 API 和隔离
与传统的基于模拟器的 Android 测试不同，Robolectric 测试在沙箱内运行，允许将 Android 环境精确配置为每个测试所需的条件，将每个测试与其邻居隔离，并使用提供精细控制的测试 API 扩展 Android 框架Android 框架的行为和断言状态的可见性。
虽然大部分 Android 框架将在 Robolectric 测试中按预期工作，但一些 Android 组件的常规行为并不能很好地转化为单元测试：需要模拟硬件传感器，系统服务需要加载测试夹具数据。在这些情况下，Robolectric 提供了适用于大多数单元测试场景的测试替身。

### 在模拟器之外运行测试
Robolectric 允许您在工作站上运行测试，或者在常规 JVM 中的持续集成环境中运行测试，而无需模拟器。因此，不需要索引、打包和在模拟器上安装这些步骤，将测试周期从几分钟缩短到几秒钟，这样您就可以快速迭代并自信地重构您的代码。

### SDK、资源和本地方法模拟
Robolectric 处理视图的膨胀、资源加载以及许多其他在 Android 设备上以本机 C 代码实现的内容。这允许测试完成您可以在真实设备上完成的大多数事情。为特定的 SDK 方法提供您自己的实现也很容易，因此您可以模拟错误条件或真实世界的传感器行为等。

### 无需模拟框架
Robolectric 的另一种方法是使用Mockito等模拟框架或模拟 Android SDK。虽然这是一种有效的方法，但它通常会产生本质上是应用程序代码的反向实现的测试。
Robolectric 允许更接近黑盒测试的测试风格，使测试更有效地进行重构，并允许测试专注于应用程序的行为而不是 Android 的实现。如果愿意，您仍然可以将模拟框架与 Robolectric 一起使用。

## Android Studio 添加依赖

``` build.gradle
android {
  testOptions {
    unitTests {
      includeAndroidResources = true
    }
  }
}

dependencies {
  testImplementation 'junit:junit:4.13.2'
  testImplementation 'org.robolectric:robolectric:4.9'
}

```

将此行添加到您的gradle.properties（Android Studio 3.3+ 不再需要）

``` gradle.properties
android.enableUnitTestBinaryResources=true

```

## 一个简单示例
编写一个测试，断言当用户单击按钮时，应用程序会启动 LoginActivity。

``` java 
public class WelcomeActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.welcome_activity);

        final View button = findViewById(R.id.login);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                startActivity(new Intent(WelcomeActivity.this, LoginActivity.class));
            }
        });
    }
}

```

为了对此进行测试，我们可以检查当用户单击“登录”按钮时，我们启动了正确的意图。因为 Robolectric 是一个单元测试框架，**LoginActivity 实际上不会启动**，但我们可以检查 WelcomeActivity 是否触发了正确的意图：

``` java
@RunWith(RobolectricTestRunner.class)
public class WelcomeActivityTest {

    @Test
    public void clickingLogin_shouldStartLoginActivity() {
        try (ActivityController<WelcomeActivity> controller = Robolectric.buildActivity(WelcomeActivity.class)) {
            controller.setup(); // Moves Activity to RESUMED state
            WelcomeActivity activity = controller.get();

            activity.findViewById(R.id.login).performClick();
            Intent expectedIntent = new Intent(activity, LoginActivity.class);
            Intent actual = shadowOf(RuntimeEnvironment.application).getNextStartedActivity();
            assertEquals(expectedIntent.getComponent(), actual.getComponent());
        }
    }
}

```


