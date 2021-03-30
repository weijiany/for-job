### 基础

1. int float short double long char 占字节数？

    - int: 4
    - float: 4
    - short: 2
    - double: 8
    - long: 8
    - char: 2

2. int 范围？float 范围？

    int: -2^(31) ~ 2^(31) - 1

    float: 2^(-149) ~ 2^(128) - 1

3. hashcode 和 equals 的关系

    hashcode: 用来计算 obj 的 hash  值，可以用来做 obj 的唯一值

    equals: 用来比较两个对象

4. java 异常体系？RuntimeException Exception Error 的区别。

    Throwable -> Error: 一般是 jvm 层级的错误，仅靠程序本身不能预防和恢复
    
    Throwable -> Exception -> RuntimeException 
    - Exception: 非运行是一场，需要显示抛出该异常，指可预见的异常，例如：FileNotFoundException
    - RuntimeException: 运行时异常，不可预见，一般都是程序运行错误才会出现，NullPointerException, IllegalArgumentException

5. lambda 表达式中使用外部变量，为什么要 final？

    lambda 表达式最终会生成一个 class 文件，对于外部变量则通过`构造器`传入，为了使 lamnbda 和方法中的变量值保持一致，所以会在语言层面声明为 final。

6. Collection 有什么子接口、有哪些具体的实现

    接口: Set, List
    Set: HashSet, TreeSet, LinkedHashSet
    List: ArrayList, LinkedList,

7. 简单介绍下 ArrayList 怎么实现，加操作、取值操作，什么时候扩容？
    - 实现: 基于数组实现，数组默认长度 10，也可以在创建对象的时候，通过构造器传入。
    - add: 
        - modCount ++; modify count 修改次数，当使用迭代器对 ArrayList 进行遍历的时候，会检查 modCount 和迭代器中 expectedModCount 的值，不相同报错。这样为了保证使用迭代器进行遍历的时候，list 不会被改变。
        - 把传入的值放在当前位置，长度 + 1
    - 扩容: 如果 list 中存储对象达到上限，进行扩容。如果当前数组长度小于 10 每次增长 5，否则每次增长 数组长度的一半


------------

# IOC

BeanDefinitionReader 用来从读取不同类型的配置文件
BeanDefinition bean 的定义
实例化 bean => 在 IOC 中创建对应实例，在堆上开辟一片内存空间
初始化 bean => 给属性赋值，调用初始化方法

BeanFactory 用来实例化 bean
BeanFactoryPostProcessor 增强器，用来处理 bean 定义信息
BeanPostProcessor 处理 bean 对象，例如：AbstractAutoProxyCreator.createProxy => JdkDynamicAopProxy/ObjenesisCglibAopProxy

xml => BeanDefinitionReader => BeanDefinitoin => 被所有 BeanFactoryPostProcessor 处理 => BeanFactory => 实例化 Bean => 填充属性 => BeanPostProcessor: postProcessBeforeInitialization => 初始化 Bean 执行 init 方法 => BeanPostProcessor: postProcessAfterInitialization => 完整 bean 对象

Bean 声明周期

> * Bean factory implementations should support the standard bean lifecycle interfaces
> * as far as possible. The full set of initialization methods and their standard order is:
> * 
> * BeanNameAware's {@code setBeanName}
> * BeanClassLoaderAware's {@code setBeanClassLoader}
> * BeanFactoryAware's {@code setBeanFactory}
> * EnvironmentAware's {@code setEnvironment}
> * EmbeddedValueResolverAware's {@code setEmbeddedValueResolver}
> * ResourceLoaderAware's {@code setResourceLoader}
> * (only applicable when running in an application context)
> * ApplicationEventPublisherAware's {@code setApplicationEventPublisher}
> * (only applicable when running in an application context)
> * MessageSourceAware's {@code setMessageSource}
> * (only applicable when running in an application context)
> * ApplicationContextAware's {@code setApplicationContext}
> * (only applicable when running in an application context)
> * ServletContextAware's {@code setServletContext}
> * (only applicable when running in a web application context)
> * {@code postProcessBeforeInitialization} methods of BeanPostProcessors
> * InitializingBean's {@code afterPropertiesSet}
> * a custom init-method definition
> * {@code postProcessAfterInitialization} methods of BeanPostProcessors
> * l>

Environment 接口：为了在容器创建对象时方便加载系统环境变量和 properties 属性，把这两种资源加载到 StandardEnvironment 对象中

在 bean 对象创建过程中，想详细了解每一个步骤完成的进度，可以使用`观察者模式：监听器，监听事件`

BeanPostProcessor 实现的功能: AOP

BeanFactoryPostProcessor 实现的功能: 
    - Placeholder，替换 ${},
        - ConfigurationClassPostProcessor 处理注解

Bean 创建过程：AbstractApplicationContext.class => refresh => finishBeanFactoryInitialization => beanFactory.preInstantiateSingletons => getBean => doGetBean => createBean => doCreateBean => addSingletonFactory(创建代理对象，把 ObjectFactory 放到三级缓存中) => populateBean => autowireByName => getBean

一级缓存：`Map<String, Object> singletonObjects`

二级缓存：`Map<String, Object> earlySingletonObjects`

三级缓存：`Map<String, ObjectFactory<?>> singletonFactories`，添加对象到三级缓存：`addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));`

1. 创建 bean，为什么需要清除二级和三级缓存中的对象？

   因为每次 getBean() 都是从一级缓存中获取，如果一级缓存中存在，就不会再去 二级和三级缓存拿了。

2. 如果只有一级缓存，能否解决循环依赖问题？

   不能，一级缓存放的是成品对象，二级缓存放的是半成品对象，如果只有一级缓存，那么可能会获取到半成品对象，可能出现空指针

3. 三级缓存的作用？

   为了解决 AOP 过程中 AOP 动态代理。

   例子：

   设：A <= B 意为 A 依赖 B，现在有三个类 A、B、C 均被 AOP 增强

   A <= B

   B <= A，B <= C

   C <= A

   加载开始：实例化 A(只有在触发 BeanPostProcessor 才知道 A 是代理对象)，把 A 放到三级缓存中，设置 B；实例化 B，由于 A 已经初始化过，设置属性 A：从三级缓存中获取 A 的 ObjectFactory，触发 ObjectFactory getEarlyBeanReference，在这个方法中会创建对应对象的代理对象(或者原对象)，并把对象放到二级缓存中，设置 C；实例化 C，设置 A，在二级缓存中获取 A**(如果没有三级缓存的话，由于有 AOP 代理，所以第一次与第二次拿到的对象不一致)**

4. 一个对象要被代理，是否需要创建当前对象的普通对象？

需要

------------

# AOP(AbstractAutoProxyCreator)



