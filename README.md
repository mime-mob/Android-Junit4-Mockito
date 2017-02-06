# Android MVP Presenter层单元测试 简述 #

**测试工具**：

1. Junit4

2. Mockito

**选择这两个测试工具的理由**：

1. 当前Android钱包2.0项目框架影响

	当前采用MVP的框架，分M层，V层，P层，每一层都有对应的测试工具及其方法。

	这两个工具是针对于其中的P层，P层负责业务逻辑

	Junit主要用来测试方法返回值是否正确，Mockito功能较为强大，用来测试一些流程

2. 该测试与UI无关，非UI单元测试，可及时运行发现问题，不依赖于真实的网络环境等

## Junit 4 ##

### 注解 ###

- `@Before` 在所有的测试方法前都会运行
- `@After` 在所有的测试方法后都会运行
- `@Test` 标注方法为测试方法
- `@Test annotation` 验证方法会抛出某些异常 eg:@Test(expected = IllegalArgumentException.class) 如果测试方法没有抛出IllegalArgumentException则测试失败
- `@Ignore` 在运行所有测试方法时，忽略该注解标注的方法
- `@BeforeClass`用来标注在测试开始时运行
- `@AfterClass` 用来标注在测试结束时运行

### assert方法 ###

- `assertEquals(expected, actual)` 若两个参数值相同，则通过，若传入值为object，则对比方法为equals()
- `assertEquals(expected, actual, tolerance)` 此处的参数为double类型，第三个参数为运行偏差值，[因为计算机表示浮点型数据都会有一定的偏差](http://blog.csdn.net/misterliwei/article/details/5455666)，因此只要两数差值在允许偏差内，都算通过
- `assertTrue(boolean condition)` 验证condition值是否为true
- `assertFalse(boolean condition)` 验证condition值是否为false
- `assertNull(Object obj)` 验证obj是否为null
- `assertNotNull(Object obj)` 验证obj是否不为null
- `assertSame(expected, actual)` 验证两个参数是否为同一对象
- `assertNotSame(expected, actual)` 验证两参数是否不为同一对象
- `fail()` 让测试方法失败
- ps:上述方法都有一个重载方法，在参数列表第一个加String message作为测试方法失败后的提示语

## Mockito ##

### Mock ###

Mock的概念:所谓的mock就是创建一个类的虚假的对象，在测试环境中用来替换掉真是的对象以达到量达目的:

- 验证这个对象的某些方法调用情况，调用的多少次，参数是什么等
- 制定这个对象的某些方法的行为，返回特定的值，或者是执行特定的动作

要使用Mock，就要用到mock框架，其中，`Mockit`框架，是java界使用最广泛的一个mock框架。

### Mockito中的方法 ###

#### 1，验证方法的调用 ####

- 大部分时候我们会`static import Mockito`这个类的所有静态方法，就不用每次加`Mockito.`前缀了
- `Mockito.verify(objectToVerify).methodToVerify(arguements)`; objectToVerify和methodToVerify分别是你想要验证的对象和方法  `Mockito.verify()`参数必须是mock对象，Mockito只能验证mock对象方法的调用情况
- `Mockito.verify(objectToVerify， Mockito.times(n)).methodToVerify(arguements)` 是上述方法的重载，测试该方法是否调用的n次
- `atMost(count)`，`atLeast(count)`，`never()` 验证最多，最少，从来没有等
- Mockito提供一系列`any`方法，表示任何参数都行，如:`anyString()`,`anyInt()`,`anyDouble()`,`anyObject()`,`any(clazz)`表示任何属于clazz的对象，`anyCollection()`,`anyCollection(clazz)`,`anyList(Map, set)`, `anyListOf(clazz)`等等

#### 2，制定某个方法的返回值，或者执行特定的动作 ####

- `Mockito.when(mockObject.targetMethod(args)).thenReturn(desiredReturnValue);` 通过mock出的对象调用方法，返回特定的值desiredReturnValue
- 当有callback等时，需要mock对象执行特定的动作 用下述方法 `Mockito.doAnswer(desiredAnswer).when(mockObject).targetMethod(args);` 传给`doAnswer()`的是一个`Answer`对象
- `Mockito.doNothing()` 指定目标方法“什么都不做”
- `Mockito.doThrow(desiredException)` 指定目标方法“抛出一个异常”
- `Mockito.doCallRealMethod()` 让目标方法调用真实的逻辑


mock 与 spy 他们唯一区别是默认行为不一样:spy对象的方法默认调用真是的逻辑，mock对象的方法默认什么都不做或直接返回默认值


### Presenter层单元测试实践 ###

- Mocktio测试：当使用presenter实例调用方法，传入any参数，要使用eq()方法将any参数包裹起来，使用mock出来的对象再次传入该any参数时，不要用eq()方法包裹 (传入any参数与传入具体值使用方法不同，具体原因未知)


## 测试用例的运行 ##

- 图形化界面运行，右击测试方法(测试类，测试包)-->run，分别对应运行不同范围的测试用例 或者使用 快捷键 `Ctrl+Shift+R` 根据光标位置不同，运行不同范围的测试方法
- 通过命令行运行 `gradle testDebugUnitTest` 或 `gradle testReleaseUnitTest` 分别对应debug和release版本的unit testing 这种方式可以一次性运行所有测试类的所有测试方法

**测试内容**：

1. 方法
2. 流程

**方法**

1. 有返回值(包括处理之后，原有值改变的(如：list等))

	有返回值的方法，使用Junit4可以将预期的值与真实值做对比，从而验证方法是否正确

	`assertEquals(expected, actual);`

2. 无返回值

	无返回值的方法，没有返回值，结果可能是运行一个方法(保存数据，或变更UI)，此时，使用Mockito来进行测试

	Mockito可以使用Mock出来的对象调用想要测试的方法，如果该方法的结果是调用 一个result()的方法，那么Mockito内部会将调用这个方法的次数记录下来，而它提供了一个方法，我们可以预期这个方法会调用一次，来验证该方法逻辑是否无误(预期次数与Mockito内部记录的数据做对比)

	`Mockito.verify(objectToVerify).methodToVerify(arguements);`

**流程**

流程的测试其实与方法是相似的，只是可能会有多个方法串联，有依赖关系存在，此时，Mockito发挥主要作用

我们可以认为，**Junit4用来测试返回值，Mockito用来验证方法是否被调用**

**依赖流程测试**

**举个栗子**：

	a() {
		if(如调用了一个外部类的方法来判断) {
			print("11111");
		} else {
			print("22222");
		}
	}

上面简单代码会依赖于外部类的方法

其中a()方法中条件判断，我们并不能确定其值，而Mockito提供了一个方法，可以让我们指定某个方法的返回值

我们可以通过Mockito如下方法，指定这个外部类方法的返回值

`Mockito.when(mockObject.targetMethod(args)).thenReturn(desiredReturnValue);`

比如那个方法是d()方法，我们指定d()方法返回值为true，则最后结果为打印11111，反之则打印22222

**再举个栗子(接口调用依赖)**：

钱包2.0登录接口

我们要在getPublickKey()接口调用成功后，再去调用登录的接口，串行调用

接口的返回，都是以回调的形式，Mockito提供了一个对象ArgumentCaptor<>，专门来处理回调

Mockito可以通过captor对象指定对应的回调，如successed，failed

回归栗子：

getPublickKey()接口调用成功后，指定successed回调，其中会执行调用login()接口，再次指定其回调为successed，再用Mockito看是否调用其中应调用的方法，以此验证流程是否成功

文档参考：

1. [https://segmentfault.com/u/chriszou/articles](https://segmentfault.com/u/chriszou/articles)(有关单元测试的几篇)
