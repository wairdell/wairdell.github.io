---
title: Android APT的使用
date: 2018-08-20 14:19:08
tags:
- Android
- Apt
- annotationProcessor
- 注解处理器
categories:
- 技术
---
## Apt 介绍
APT(Annotation Processing Tool)是一种处理注释的工具,它对源代码文件进行检测找出其中的 Annotation，根据注解自动生成代码。 Annotation 处理器在处理 Annotation 时可以根据源文件中的 Annotation 生成额外的源文件和其它的文件(文件具体内容由 Annotation 处理器的编写者决定),APT 还会编译生成的源文件和原来的源文件，将它们一起生成 class 文件。我们 Android 常用的库如 Dagger2, ButterKnife, EventBus3 等都使用了 APT 的技术。

本篇通过一个简单的 Android 路由的项目来介绍 APT 相关内容和进行 APT 开发的大致流程。
## 使用 APT 开发 Android 路由框架
1. ### 首先我们创建一个名字为 route-annotation 的 module，定义 APT 需要处理的注解。
```java
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.TYPE)
public @interface Route {
    String address() default "";
}
```

2. ### 在创建一个名字为 route-compiler 的 module 实现一个注解处理器。

	- #### 在 module 对应的 build.gradle 里添加一下依赖
	```
	implementation project(':route-annotation')
	implementation 'com.squareup:javapoet:1.11.1'
	implementation 'com.google.auto.service:auto-service:1.0-rc2'
	```

	- #### 下面就来创建我们的处理器 RouteProcessor

	我们需要创建一个名为 RouteProcessor 的类继承 **AbstractProcessor**，并添加三个注解 **@AutoService**、 **@SupportedAnnotationTypes**、 **@SupportedSourceVersion**
		* @SupportedAnnotationTypes 表示我们需要处理哪些注解
		* @SupportedSourceVersion 表示我们生成的代码是 Java 的哪个版本

	然后我们一般需要重写 AbstractProcessor 的两个方法 **init(ProcessingEnvironment processingEnv)** 和 **process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment)**,通过 init 方法的 processingEnv 我们可以得到一些辅助类, 通过 process 的 roundEnvironment 我们可以得到被注解对象(Element)的集合,相关代码如下
	
	```java
	@AutoService(Processor.class)
	@SupportedAnnotationTypes("com.wairdell.route.annotation.Route")
	@SupportedSourceVersion(SourceVersion.RELEASE_7)
	public class RouteProcesser extends AbstractProcessor {


		private Filer mFiler; //文件相关的辅助类
		private Elements mElementUtils; //元素相关的辅助类
		private Messager mMessager; //日志相关的辅助类

		@Override
		public synchronized void init(ProcessingEnvironment processingEnv) {
			super.init(processingEnv);
			mFiler = processingEnv.getFiler();
			mElementUtils = processingEnv.getElementUtils();
			mMessager = processingEnv.getMessager();
		}

		@Override
		public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
			Set<? extends Element> elements = roundEnvironment.getElementsAnnotatedWith(Route.class);
			List<RouteAnnotation> annotationList = new ArrayList<>();
			for (Element element : elements) {
				annotationList.add(new RouteAnnotation(element));
			}
			JavaFile javaFile = new RouteGenerater(annotationList, mMessager).generateJavaFile();
			try {
				javaFile.writeTo(mFiler);
			} catch (IOException e) {
				e.printStackTrace();
				return false;
			}
			return true;
		}
	}
	```
	这里介绍下 Element 的概念, 注解处理器会把我们注解的方法、字段、类等等封装成一个 Elment 的对象，
		- 类会封装成 TypeElement
		- 方法会被封装成 ExecuteableElement
		- 字段会被封装成 VariableElement
	
	然后我们可能通过这个对象获取被注解对象的信息(如方法的参数和返回值、字段的类型和名字等等)，并且这些 Element 元素还相当于 XML 中的 DOM 树,通过相关方法可以访问它的父元素或者子元素。
		- element.getEnclosingElement();// 获取父元素
		- element.getEnclosedElements();// 获取子元素
	
	如下面这段代码，通过 Element 获取需要的信息
	```java
		public class RouteAnnotation {

			private final String address;
			private final String className;
			private final String classSimpleName;
			private final TypeMirror typeMirror;

			public RouteAnnotation(Element element) {
				TypeElement typeElement = (TypeElement) element; //因为我们声明的注解Route是作用在类上，这里把Element的强转为TypeElement
				className = typeElement.getQualifiedName().toString();
				classSimpleName = typeElement.getSimpleName().toString(); //得到被Route注解声明类的名字
				typeMirror = typeElement.asType(); //得到被Route注解声明类的类型
				address = element.getAnnotation(Route.class).value(); //得到Route注解时里面的value
			}

			public String getAddress() {
				return address;
			}

			public TypeMirror getClassType() {
				return typeMirror;
			}

			public String getClassSimpleName() {
				return classSimpleName;
			}
		}

	```
	- #### 我们通过注解对象的信息用 **[javapoet](https://github.com/square/javapoet)** 这个库来生成相关的代码
	```java
		public class RouteGenerater {

			private List<RouteAnnotation> routeAnnotationList;

			private Messager messager;

			public static ClassName CONTEXT_CLASS = ClassName.get("android.content", "Context");

			public static ClassName INTENT_CLASS = ClassName.get("android.content", "Intent");

			public static ClassName BUNDLE_CLASS = ClassName.get("android.os", "Bundle");

			public static ClassName LOG_CLASS = ClassName.get("android.util", "Log");


			public RouteGenerater(List<RouteAnnotation> routeAnnotationList, Messager messager) {
				this.routeAnnotationList = routeAnnotationList;
				this.messager = messager;
			}

			public JavaFile generateJavaFile() {
				//创建一个名为RouteCenter类
				TypeSpec.Builder classBuilder = TypeSpec
						.classBuilder("RouteCenter")
						.addModifiers(Modifier.PUBLIC);

				String addressParam = "address";
				String contextParam = "context";
				String extrasParam = "extras";

				MethodSpec.Builder methodBuilder = MethodSpec
						.methodBuilder("startActivity") //方法名为startActivity
						.addModifiers(Modifier.PUBLIC) //添加修饰符 public
						.addModifiers(Modifier.STATIC) //添加修饰符 static
						.addParameter(CONTEXT_CLASS, contextParam) //添加 类型为android.content.Context 名字为context 的参数
						.addParameter(String.class, addressParam) //添加 类型为java.lang.String 名字为 address 的参数
						.addParameter(BUNDLE_CLASS, extrasParam) //添加 类型为android.os.Bundle 名字为 extras 的参数
						.returns(Void.TYPE); //添加返回值
				String intentVariable = "intent";
				//开始声明方法内的代码
				CodeBlock.Builder codeBuilder = CodeBlock
						.builder()
						.addStatement("$T $N = new $T()", INTENT_CLASS, intentVariable, INTENT_CLASS) //添加一个局部变量 intent
						.beginControlFlow("switch($N)", addressParam); //添加一个switch语句
				String caseName;
				String address;
				//for循环往switch添加case
				for (RouteAnnotation routeAnnotation : routeAnnotationList) {
					address = routeAnnotation.getAddress();
					//如果Route注解里没有填入值，则用类名作为case 条件
					caseName = (address != null && !"".equals(address)) ? address : routeAnnotation.getClassSimpleName();
					//添加代码 intent.setClass(context, class) class是被Route注解声明的Activity
					codeBuilder.
							addStatement("case $S:\n$N.setClass($N, $T.class);\nbreak", caseName, intentVariable, contextParam, routeAnnotation.getClassType());

				}
				//添加代码 没找到路由打印日志
				codeBuilder.addStatement("default:\n$T.e($S, $S + \" \" + $N);\nbreak", LOG_CLASS, "RouteCenter", "not found Activity by route address", addressParam).endControlFlow();
				//添加代码intent.putExtras(extras);
				codeBuilder.beginControlFlow("if($N != null)", extrasParam).addStatement("$N.putExtras($N)", intentVariable, extrasParam).endControlFlow();
				//添加代码context.startActivity(intent)
				codeBuilder.addStatement("$N.startActivity($N)", contextParam, intentVariable);


				methodBuilder.addCode(codeBuilder.build()); //往方法里添加代码
				classBuilder.addMethod(methodBuilder.build()); //往类里添加代码

				//最后通过包名和类生成 Java 文件
				return JavaFile.builder("com.wairdell.route", classBuilder.build()).build();
			}

		}
	```
3. ### 最后我在需要是用 route 的 module 使用我们上面写好的库
	- 在 build.gradle 增加相关依赖
	```
		implementation project(':route-annotation')
		annotationProcessor project(':route-compiler')
	```
	- 在需要的被路由的 Activity 声明 @Route 的注解
	```java
		@Route("login")
		public class LoginActivity extends AppCompatActivity {

			@Override
			protected void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);
				setContentView(R.layout.activity_login);
			}
		}
	```
	- Make Project 一下就能在 *build/generated/source/apt/debug* 下面看到生成的类了

## 项目结构和地址

{% asset_img QQ截图20180821150543.png %}
项目地址: [https://github.com/wairdell/RouteDemo](https://github.com/wairdell/RouteDemo)
	
## 参考
- [Android 利用 APT 技术在编译期生成代码](https://brucezz.itscoder.com/use-apt-in-android)
- [java annotation processing tools(APT)实例解析](https://blog.csdn.net/white__cat/article/details/40681421)
