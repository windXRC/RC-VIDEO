# Spring基础

### 1. 控制反转(IOC)

定义： 把对象的创建、赋值、管理工作交给外部容器实现，对象的创建通过外部资源实现。 

#### 1.1 IOC示例

##### Servlet的创建过程：

* 创建类继承HttpServlet
* web.xml注册servlet
* Servlet是tomcat服务器创建的，tomcat也称为容器

> tomcat容器中的对象：Servlet对象，Listener，Filter对象

#### 1.2 依赖注入

只需要只提供对象名称即可，对象的创建、赋值、查找都交给spring容器实现

#### 1.3 bean创建方式

```java
    <bean id="myTest" class="example.service.MyTest" scope="prototype">
        <!-- bean 配置 -->
    </bean>
```

#### 1.4 DI属性注入

##### 1.4.1 set注入

```java
	public class MyTest {
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    }

    <bean id="myTest" class="example.service.MyTest">
        <property name="name" value="zhangsan"></property>
        //调用MyTest类中的setName方法,name属性可无
    </bean>
```

##### 1.4.2 引用类型的set注入

```java
    <bean id="student" class="example.service.Student">
        <property name="name" value="lisi" />
    </bean>

    <bean id="school" class="example.service.School">
        <property name="name" value="changde" />
        <property name="student" ref="student" />//引用student对象
    </bean>
```

##### 1.4.3 构造注入

 ```java
     <bean id="student" class="example.service.Student">
         <constructor-arg name="name" value="zhangsan" />
         <constructor-arg name="age" value="123" />
     </bean>
 ```

##### 1.4.4 应用类型自动注入

```java
    //引用类型的属性名称和spring容器中<bean>的id名称一样，且数据类型一致
    <bean id="school" class="example.service.School" autowire="byName" >
        <property name="name" value="changde" />
    </bean>
    
    //引用类型的数据类型和spring容器中<bean>的class属性是同源关系，同一个bean.xml中，符合byType条件的bean应当只有一个，注入父子类时应当特别注意
    <bean id="school" class="example.service.School" autowire="byType" >
        <property name="name" value="changde" />
    </bean>
```

#### 1.5 spring注解

##### 1.5.1 组件扫描器

```java
    <!--指定多个包的三种方式-->
    <!--第一种：使用多组扫描器-->
    <context:component-scan base-package="example.Controller" />
    <context:component-scan base-package="example.Util" />
    <!--第二种：分隔符（;或,）-->
    <context:component-scan base-package="example.Controller;example.Util" />
    <!--第三种：指定父包-->
    <context:component-scan base-package="example" /> 
```

##### 1.5.2 @Component

创建对象，等价于<bean>的功能，默认名称小写。

##### 1.5.3 @Respository

放在dao的实现类上，表示创建dao对象，dao对象可访问数据库。

##### 1.5.4 @Service

放在service的实现类上，service对象做业务处理，可以有事务等功能。

##### 1.5.5 @Controller

放在控制器类上，创建控制器对象的，能够接受用户提交的参数，显示请求的处理结果。

##### 1.5.6 @Value

属性赋值，无需set方法，也可用于set方法之上。

```java
    @Value("zhangsan")
    private String name;

    @Value("24")
    private int number;

    // 引用类型赋值
    // @Autowired: 通过注解给引用类型赋值，使用自动注入的原理，默认使用byType自动注入
    @Autowired
    private School school;

    // byName自动注入
    @Autowired
    @Qualifier("school") //@Qualifier注解属性值不能为空

    //引用类型是否存在，默认为true，为true时，若引用类型不存在则会抛出异常，bean注入失败
    @Autowired(required = true)

    //设置为false时，若引用类型不存在则为对象为null
    @Autowired(required = false)


```

##### 1.5.7 @Resource

jdk中的注解，spring框架提供了对这个注解的功能支持，可以使用他给引用类型赋值，使用的也是自动注入的原理，默认是byName，若byName赋值失败，再使用byType。

```java
    @Resource
    private School school;

    @Resource(name = "school") // 只使用byName
    private School school;
```

> 代码经常改动使用配置文件，不经常改动使用注解，生产效率方式更高，注解为主，配置文件为辅

### 2. 面向切面编程（AOP）

#### 2.1 动态代理

使用jdk中的Proxy，Method，InvocationHandler创建代理对象。动态代理类要求要求目标类必须实现接口。

#### 2.2 aspectJ

定义：aspectJ是一个开源的专门做aop的框架。spring中集成了aspectJ框架。

1. 使用xml配置文件，可以用来配置全局事务
2. 使用注解，项目中一般都使用注解

##### 2.2.1 注解

@Before

@AfterReturning

@Around

@AfterThrowing

@After

##### 2.2.2 Aspect切入点表达式

execution（访问权限、**方法返回值、方法声明（参数）**、异常类型）

| 符号 |                             意义                             |
| :--: | :----------------------------------------------------------: |
|  *   |                       0或多个任意字符                        |
|  ..  | 用在方法参数中，表示任意多个参数；用在包名后，表示当前包及其子包路径 |
|  +   | 用在类名后，表示当前类及其子类；用在接口后，表示当前接口及其实现类 |

举例：

* execution(public * * (..))

指定切入点为任意公共方法

* execution(* set*(..))

指定切入点为任何一个以“set”开始的方法

* execution(* com.xyz.\*.\*(..))

指定切入点为com.xyz包里的任意所有的所有方法

* execution(* com.xyz..\*.\*(..))

指定切入点为com.xyz包及其子包里的所有类的任意所有

* execution(* *..xyz.\*.\*(..))

指定切入点为所有包下的xyz子包里的所有类的所有方法

