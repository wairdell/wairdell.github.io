---
title: 单元测试之Mockito
date: 2018-08-22 16:51:20
tags:
- Mockito
- 单元测试
categories:
- 技术
---
## Mockito 是什么
在我们做单元测试中，被测试的类肯定或多或少的依赖于其他的类，但有些类我们并不能轻松的实例化。如 android 的 MVP 设计模式中，当我们做 Presenter 的测试时，Presenter 会在很多地方依赖于 View 对象的方法，而 View 一般的实现是 *Acitivity* 或 *Fragment* 的，其中又有很多的实现是依赖于用户的输入的，这时候进行单元测试是一件比较复杂的事情。

而使用 Mockito 就能轻松的帮你解决上面的问题。Mock 最大的功能是帮你把单元测试的耦合分解开，如果你的代码对另一个类或者接口有依赖，它能够帮你模拟这些依赖，并帮你验证所调用的依赖的行为。

## Mockito 的简单使用

首先需要在 `build.gradle` 文件中添加相关依赖
```
	testCompile "org.mockito:mockito-core:2.11.0"
	androidTestImplementation "org.mockito:mockito-android:2.11.0" //如果不做 android 的单元测试，则不需要依赖
```

代码示例
我们假设需要 Mockito 替换下面这个类的依赖
```java
public interface LoginView {
	String getInputUserName();//本来由 activity 实现返回用户输入的用户名
	String getInputPassword();//本来由 activity 实现返回用户输入的密码
	void showErrorDialog(String msg);
}
```

1. Mock 对象的四种方式
	- 普通方法
	```java
		public class MockitoTest {

			@Test
			public void testIsNotNull(){
				LoginView loginView = mock(LoginView.class); 
				assertNotNull(loginView);
			}
		}
	```
	- 注解方法
	```java
		public class MockitoAnnotationsTest {

			@Mock
			LoginView loginView;

			@Before
			public void setup(){
				MockitoAnnotations.initMocks(this);
			}

			@Test
			public void testIsNotNull(){
				assertNotNull(loginView);
			}

		}
	```
	- 运行器方法
	```java
		@RunWith(MockitoJUnitRunner.class) 
		public class MockitoJUnitRunnerTest {

			@Mock 
			LoginView loginView;

			@Test
			public void testIsNotNull(){
				assertNotNull(loginView);
			}

		}
	```
	- MockitoRule 方法
	```java
		public class MockitoRuleTest {

			@Mock
			Person mPerson;

			@Rule
			public MockitoRule mockitoRule = MockitoJUnit.rule();

			@Test
			public void testIsNotNull(){
				assertNotNull(mPerson);
			}

		}
	```
2. 拦截方法的行为
Mock 创建出来的对象其方法返回值都是初始值(跟 Java 在类中声明的字段规则一样，如 `int` 为0, `boolean` 为 false, `Object` 为 null)，我们可以通过下面相关方法改变这一行为
```java
/**
     * 设置方法返回的值
     */
    @Test
    public void testReturn(){
        when(loginView.getInputUserName()).thenReturn("wairdell");
        assertEquals(loginView.getInputUserName(), "wairdell");
        doReturn("aaaaaa").when(loginView).getInputPassword();
        assertEquals(loginView.getInputPassword(), "aaaaaa");
        when(loginView.getInputUserName()).thenAnswer(new Answer<String>() {
            @Override
            public String answer(InvocationOnMock invocation) throws Throwable {
                //invocation.getArguments() 如果方法有参数可以通过这个方法拿到调用此方法的参数
                return "被我拦截了";
            }
        });
        assertEquals(loginView.getInputUserName(), "被我拦截了");
        when(loginView.getInputPassword()).thenCallRealMethod(); //调用方法的真是实现，LoginView是个接口，没有默认的实现，这个会报错
        assertEquals(loginView.getInputPassword(), null);
    }


    /**
     * 设置方法抛出异常
     */
    @Test(expected = IllegalArgumentException.class)
    public void testThrow() {
        when(loginView.getInputUserName()).thenThrow(new IllegalArgumentException());
        loginView.getInputUserName();
        doThrow(new IllegalArgumentException()).when(loginView).getInputPassword();
        loginView.getInputPassword();
    }

```
3. 验证行为
	- `after(long millis)` 在给定的时间后进行验证
	- `timeout(long millis)` 	验证方法执行是否超时
	- `atLeast(int minNumberOfInvocations)` 至少进行n次验证
	- `atMost(int maxNumberOfInvocations)` 至多进行n次验证
	- `description(String description)` 验证失败时输出的内容
	- `times(int wantedNumberOfInvocations)` 验证调用方法的次数
	- `never()` 验证交互没有发生,相当于times(0)
	- `only()` 验证方法只被调用一次，相当于times(1)
	- `inOrder` 验证顺序
	
4. 参数匹配器
	
	对于有参数的方法，我们需要参数匹配器来生效方法行为的范围。ArgumentMatchers 中，提供了一组不同类型的操作。如：`any(Class)`，`anyObject()`，`anyVararg()`，`anyChar()`，`anyInt`，`anyBoolean()`，`anyCollectionOf(Class)`, `contains(substring)`，`argThat(matcher)` 等。
	```java
		@Test
		public void testPrams() {
			//匹配包含 hidden 字符串参数 包含则不做任何事
			doNothing().when(loginView).showErrorDialog(contains("hidden"));

			//匹配 长度小于6 字符串参数，小于6则抛出异常
			doThrow(new IllegalArgumentException()).when(loginView).showErrorDialog(argThat(new ArgumentMatcher<String>() {
				@Override
				public boolean matches(String argument) {
					return argument.length() < 6;
				}
			}));
			//匹配所有的字符串参数
			doAnswer(new Answer() {
				@Override
				public Object answer(InvocationOnMock invocation) throws Throwable {
					return null;
				}
			}).when(loginView).showErrorDialog(anyString());
		}
	```
		
## 参考
- [Mockito 简明教程](https://blog.csdn.net/kkkloveyou/article/details/50695517?locationNum=15)
- [Android单元测试(二)：Mockito框架的使用](https://blog.csdn.net/qq_17766199/article/details/78450007)
- [Mockito 中文文档](https://github.com/hehonghui/mockito-doc-zh)