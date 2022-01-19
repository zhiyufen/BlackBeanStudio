## Robolectric学习笔记

[TOC]

Junit 属于 JVM 平台上的单元测试框架，无法提供 Android 运行时环境。如果在单元测试中涉及到 Android 特性，Junit 则无法实现。

因此在Android上，是启动 Android 模拟器(或真机)进行测试(Android Test)。但是在模拟器上运行测试用例是非常低效的，构建、安装、启动，每个步骤都异常耗时，为了解决这一问题，Robolectric 通过 mock Android 运行时环境，使得单元测试可以在 JVM 环境上运行。

而Robolectric测试在一个沙箱中运行，每个测试都为其配置Android环境，将每个测试之间进行隔离，并使用测试API扩展Androd框架，这些API可以对Android框架的行为和断言状态的可见性提供微小的控制。

注： 有些Android组件的常规行为，这Robolectric也不太好测试，比如硬件传感器之类的；

Robolectric主要适用于UI的测试，比如Activity，Fragment，一些页面操作的测试场景，采用Shadow的方式对Android中的组件进行模拟测试，从而实现Android单元测试Robolectric正好弥补了Mockito的不足，两者结合使用是最完美的。

### 配置

```groovy
android {
    testOptions {
        unitTests {
            includeAndroidResources = true
        }
    }
}

dependencies {
    testImplementation "org.robolectric:robolectric:4.3"
}
```

根据需要，可继承比， 进行一些全局的配置：

```java
public class MyTestRunner extends RobolectricTestRunner {
    //跑Robolectric里对应的SDK
    private static final int DEFAULT_SDK = 28;

	//一般不用这段代码
    static {
        System.setProperty("robolectric.dependency.repo.id", "xxxx");
        System.setProperty("robolectric.dependency.repo.url", "http://xxxx");
    }

    public MyTestRunner(Class<?> testClass) throws InitializationError {
        super(testClass);
    }

    /**
     * Enables a per-test setUp / tearDown hook.
     */
    public static class BaseTestLifecycle extends DefaultTestLifecycle {
        @Override
        public void beforeTest(Method method) {
            //日志输出
            ShadowLog.stream = System.out;
            super.beforeTest(method);
        }

        @Override
        public void afterTest(Method method) {
            super.afterTest(method);
        }
    }

    @Override
    protected Class<? extends TestLifecycle> getTestLifecycleClass() {
        return BaseTestLifecycle.class;
    }

    @Override
    protected Config buildGlobalConfig() {
        return new Config.Builder()
            //也可换成自己的Application,进行一些全局化配置
            .setApplication(android.app.Application.class)
            .setSdk(DEFAULT_SDK)
            .build();
    }
}
```



### Shadow

Shadow是Robolectric的核心。Robolectric里，定义了大量对应android源码的shadow类。shadow类，如其名“影子”。当一个android类的方法被调用的时候，Robolectric就会尝试寻找该类相应的影子类，替代对应的android类，执行相应的方法。

```java
public class Person {
        public String sayHello() {
            return "I'm myself!";
        }
}

@Implements(Person.class)
public class ShadowPerson {
    @Implementation
    public String sayHello() {
        return "I'm shadow!";
    }
}

@RunWith(RobolectricTestRunner.class)
@Config(shadows = {ShadowPerson.class})
public class ShadowTest {
    @Test
    public void sayHello() {
        Person person = new Person();
        assertEquals("I'm shadow!", person.sayHello());
    }
}
```

注： Shadow是一个强大功能，利用它，也可解决Mockito私有方法无法Mock等问题， 但一般使用powermock；

### UI测试

这块暂时不写，网上有很多，针对我们app来说，这块暂不需要写；

#### Activity

略

#### Service

略

#### BroadcastReceiver

略

