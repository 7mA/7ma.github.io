---
title: Spring入门笔记
permalink: Spring_beginning/
date: 2017-03-31 10:29:36
tags:
- Spring
- Java
- Legacy
categories:
- 技术笔记
---
最近实验室的微服务项目开坑了，我所在的小组负责SpringCloud的学习和应用。虽然Spring框架并不是一个崭新的框架了，但是自己并没有接触过，所以特地花了三个晚上补了一下Spring的知识。

<!--more-->

---

## 0.感谢
> - 本文的大部分代码与思路均来自 **[CSDN专栏：spring快速入门](http://blog.csdn.net/column/details/13907.html "CSDN专栏：spring快速入门")** 。专栏的内容浅显易懂，比较容易上手。本文在该专栏的基础上加了一些个人理解与具体操作时需要注意的地方。
> - 感谢每天督促我看督促我上组会汇报~~自己却去忙毕论组会后还让我给他讲~~的组内某学长。

---

## 1.基础概念与准备

Spring是一个于2003年兴起的轻量级的开源Java开发框架，由Rod Johnson创建。其核心思想是**控制反转（IoC）**，以帮助分离项目组件之间的依赖关系。官网：[https://spring.io/](https://spring.io/ "https://spring.io/")

> 本文内容基于以下环境：
> 	
> - **Spring Framework 4.3.7 RELEASE**
> - **Eclipse Neon.3 Release (4.6.3)**
> - **JRE 1.8.0_121**

关于**Spring库**，可以选择手动下载添加，也可以选择用**Maven**自动添加。因为我在第一步构建Spring项目时对这些jar包不是很熟悉，所以采用了手动添加的方式。

手动下载地址：[http://repo.spring.io/release/org/springframework/spring/](http://repo.spring.io/release/org/springframework/spring/ "http://repo.spring.io/release/org/springframework/spring/")

另外，在Eclipse搭建Spring项目更好的方式是使用Spring官方推荐的Eclipse插件：**STS**。这个插件本身就是Spring的产品之一。只要你的Eclipse集成了Maven，就可以直接搭建你想要的Spring项目。下载时要注意版本要求要与你的Eclipse一致。下载地址：[http://spring.io/tools/sts/all](http://spring.io/tools/sts/all "http://spring.io/tools/sts/all")

---

## 2.控制反转(IoC)
>接下来采用一个简单的例子理解控制反转。

让我们回忆一下Java里接口的概念。

```java
public class TestDemo {
	public static void main(String[] args) {
		Teacher person = new Teacher();
		System.out.println(person.sayhello()); //Hello, I'm a teacher.
	}
}
```

但是联系一下现实，`sayhello()`这个方法并不是只有`Teacher`类的对象才具有的，`Student`、`Officer`也可以有。换句话说，这个方法的抽象等级不够准确。我们应该把`sayhello()`这个方法放到更高一级的类当中，只不过方法的具体实现方式有区别（Hello, I'm a student之类的）。这样我们就可以采用接口的概念，修改上述代码。

```Java
public class TestDemo {
	public static void main(String[] args) {
		Person person = new Teacher();
		System.out.println(person.sayhello()); //Hello, I'm a teacher.
	}
}

public class Teacher implements Person {
	@Override
	public String sayhello(){
		return "Hello, I am a teacher";
  	}
}
```

即便是这样，我们仍需要在主函数中指定Person接口到底由哪个类来实现。这样Hardcoding的感觉还是比较重。那么该怎么办呢？

我们可以采用Spring的方式解决这个问题。我们在同一个Package下建一个spring配置文件`spring-cfg.xml`，内容如下。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="thePerson" class="com.7ma.spring.demo.Teacher">
  </bean>

</beans>
```

然后修改主函数如下，注意`getBean()`的第一个参数要和xml文件中对应`<Bean>`标签的`id`相同。

```Java
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestDemo {
    public TestDemo() {
    }

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:com/7ma/spring/demo/spring-cfg.xml");
        Person person = (Person)context.getBean("thePerson", Person.class);
        System.out.println(person.sayhello()); //Hello, I'm a teacher.
        context.close();
    }
}
```
这样主函数里并没有指定`Teacher`类来实现`Person`接口，但是却可以得到`Teacher`类的输出结果了。

> 如果这一步出现以下错误：
> `Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/commons/logging/LogFactory`，
> 则说明缺少**commons-logging**的jar包，自行下载添加即可。

可以看到Spring的生效流程如下:
1. 装载spring配置文件（`ClassPathXmlApplicationContext`）
2. 从返回的context中检索相应的spring bean（`context.getBean()`）
3. 执行spring bean
4. 关闭context（` context.close()`）

所谓控制反转（IoC），对于软件来说，即是某一接口具体实现类的选择控制权从调用类中移除，转交给第三方决定。Spring就是这样的一个第三方的容器，它通过**配置文件**或**注解**描述类和类之间的依赖关系（上文就是以配置文件为例）。举一个现实一点的例子，我们假设调用类就像剧本，剧作家在写剧本时会写到各种各样的角色（接口），但是剧作家在写的过程中并不会考虑由哪个演员（类）来演，找演员的工作通常是交给制片方来做。而这个制片方的角色在我们的例子中就由Spring来担任了。

此外，关于Spring还有一个名词：**依赖注入（Dependence Injection）**经常会被提到。其定义可以看作与IoC等价，都是指Spring这种分离接口与具体实现的思想。至于为什么会有两个相同含义的不同说法，主要是因为历史遗留问题，感兴趣可以自己Google一下。

---
## 3.构造注入
> 上文讲述了IoC的基本概念。下面我们就利用IoC的思想实现Spring的**注入**操作。除了上面向调用类注入具体实现类的例子，Spring的注入还有很多种，诸如构造注入、值注入等。如上节所述，注入可以通过配置文件和注解两种方式。下面以用**xml配置文件**进行**构造注入**为例，简单看一下注入的大体思路。

首先定义一个`TeachingService`的接口和一个`SwimmingTeachingService`的类，用后者实现前者：

```Java
public interface TeachingService {
    public String provideTeachingService();
}

public class SwimmingTeachingService implements TeachingService {
    @Override
    public String provideTeachingService(){
        return "I provide service of teaching swimming.";
    }
}
```

之后我们将修改`Person`和`Teacher`的定义：

```Java
public interface Person {
    public String sayhello();
    public String provideTeachingService();
}

public class Teacher implements Person{
    private TeachingService service;
    public Teacher(TeachingService theService){
        this.service=theService;
    }
    public String sayhello(){
        return "Hello, I am a teacher";
    }
    @Override
    public String provideTeachingService(){
        return service.provideTeachingService();
    }
}
```

配置文件如下，注意`<constructor-arg>`的`ref`值要和对应`<Bean>`标签的`id`相同：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="thePerson" class="com.7ma.spring.demo.Teacher">
        <constructor-arg ref="theService"></constructor-arg>
    </bean>

    <bean id="theService" class="com.7ma.spring.demo.SwimmingTeachingService">
    </bean>

</beans>
```

TestDemo：
```Java
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestDemo {
    public TestDemo() {
    }

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:com/7ma/spring/demo/spring-cfg.xml");
        Person person = (Person)context.getBean("thePerson", Person.class);
        System.out.println(person.sayhello());
        System.out.print(person.provideTeachingService());
        context.close();
    }
}
```

可以看到在`Teacher`类的构造函数中，我们定义了私有成员`service`。而`TeachingService`是一个接口，所以构造函数的参数一定是一个能实现`TeachingService`的类的对象。可以看到Spring不仅支持在xml文件中指定调用类中实现接口的类，还支持在构造函数中所需对象参数的类，这就是**构造注入**。构造注入的应用范围不仅仅是对象参数，值参数也是可以的，我们只需要在`<Constructor-arg>`标签中指定参数的值就可以了。

---

**Reference Source:**
1. [CSDN专栏：spring快速入门](http://blog.csdn.net/column/details/13907.html "CSDN专栏：spring快速入门")
2. [透透彻彻IoC（你没有理由不懂！）](http://stamen.iteye.com/blog/1489223/ "透透彻彻IoC（你没有理由不懂！）")
