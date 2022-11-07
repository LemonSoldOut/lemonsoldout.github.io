# Bean 到底是什么?
- Java Spring 中的 `bean` 和 Java 本身的对象一样，拥有 `private` 自身属性和对应的 `setXxx()`, `getXxx()` 方法

> User.java

```java
public class User{
	private String name;
	private int age;
	public String getName(){
		return name;
	}
	public String getAge(){
		return age;
	}
	public void setName(String name){
		this.name = name;
	}
	public void setAge(int age){
		this.age = age;
	}
}
```

- 按照我们原来写 Java 的思想来看，想要创建一个 User 类的对象只需要返回一个 `new User()`
    - 也就是通过默认的无参构造器创建即可

```java
User u1 = new User();
u1.setAge(18);
u1.setName("lbw");
```


- 那么，`bean` 是如何初始化，使用与销毁的呢?

# 通过 `<config>.xml` 文件创建并管理 Bean
1. `idea` IDE 集成环境软件下创建 `Maven` 项目
2. 在项目主目录下存在一个默认 `pom.xml` 项目依赖配置文件，写入相关依赖
    - 依赖包可能过时，酌情考虑引入
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>Spring01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.13</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.3.13</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.9.1</version>

        </dependency>
    </dependencies>

</project>
```

3. 新建一个 `Student` 类
```java
package com.test.bean;

/**
 * @author lemon
 * @date 2022-09-04
 * @description Student 类
 */
public class Student {
    String name;
    int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

```

4. `src/main/resources`目录下自定义 `<config>.xml` 文件，配置并注册 `Student` 作为 `Spring` 框架管理的 `bean` 
   - 利用 Bean 创建对象是通过 Java 反射机制生成的
   - Bean 配置中，name 或 id 属性全局唯一，不可重复

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--id 和 name 效果相同-->
    <bean name="student" class="com.test.bean.Student">
</beans>
```

![name or id 重复问题](https://fastly.jsdelivr.net/gh/lemonsoldout/pictures@main/1662601382782a2e11d46630b4bac31383d625085fee0.png)

> Exception in thread "main" org.springframework.beans.factory.parsing.BeanDefinitionParsingException: Configuration problem: **Bean name 'student' is already used in this element Offending resource: class path resource [config.xml]**

5. 创建 `<Main>.java` 带有 `main` 方法作为 Java 程序的主入口
   - `idea` 自动识别并显示配置应用程序上下文，点击创建即可
```java
package com.test;

import com.test.bean.Student;
import org.springframework.context.support.ClassPathXmlApplicationContext;


/**
 * @author lemon
 * @date 2022-09-04
 * @description Spring 01 - 配置 bean 和编写第一个 demo
 */
public class SpringMain {
    public static void main(String[] args) {
        /*
            配置文件名不能写错
            报错: Caused by: java.io.FileNotFoundException: class path resource [cnfig.xml] cannot be opened because it does not exist
            context 现在就是一个容器
         */

        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("config.xml");

        Student s1 = new Student();
        Student stu = context.getBean(Student.class);//通过类型获取 bean
        Student stu1 = (Student) context.getBean("student");//通过名字直接获取 bean
        System.out.println("Bean stu:\t" + stu);
        System.out.println("new S1:\t" + s1);
        
        
        //注意容器也要关闭
        context.close();
    }
}

```

- 可以发现通过 `getBean()` 获得的两个对象都指向同一个内存地址
```java
Student stu1 = (Student) context.getBean("student");//通过名字直接获取 bean
Student stu2 = (Student) context.getBean("student");
System.out.println("Bean stu1:\t" + stu1);
System.out.println("Bean stu2:\t" + stu2);
```

```java
Bean stu1:	com.test.bean.Student@56ac3a89
Bean stu2:	com.test.bean.Student@56ac3a89
```

- 相同地址也就意味着是同一个对象
- 单例模式(与之相对的就是原型模式 也可以称之为多例模式)
- 若要生成多个对象，可在 bean 配置中修改 scope 值，默认为 `singleton`, 可以修改成 `prototype`

![Bean 的单例和原型模式](https://fastly.jsdelivr.net/gh/lemonsoldout/pictures@main/166260173878840366355795b3c8347e818bef91afde4.png)

- 修改完成之后，每次生成的对象就会指向不同的内存地址了，也就是所有的对象互不相同

```java
Bean stu1:	com.test.bean.Student@5700d6b1
Bean stu2:	com.test.bean.Student@6fd02e5
```

- 单例 vs 原型
> Student.java
```java
public Student(){
    System.out.println("Student 无参构造函数被创建...");
}

```

```java
Student 无参构造函数被创建...
Bean stu1:	com.test.bean.Student@56ef9176
Bean stu2:	com.test.bean.Student@56ef9176

---------------------------------------------

Student 无参构造函数被创建...
Student 无参构造函数被创建...
Bean stu1:	com.test.bean.Student@5700d6b1
Bean stu2:	com.test.bean.Student@6fd02e5
```

# Bean 的声明周期管理
> Student.java

```java
public void init(){
    System.out.println("Student 初始化...");
}

public Student(){
    System.out.println("Student 无参构造函数被创建...");
}

public void destroy(){
    System.out.println("Student 摧毁...");
}
```

> config.xml

```xml
<bean name="student" class="com.test.bean.Student" init-method="init" destroy-method="destroy"/>
```

> output
```java
Student 无参构造函数被创建...
Student 初始化...
Bean stu1:	com.test.bean.Student@72d818d1
Student 摧毁...
```
`

- `bean` 的注册顺序

```java
package com.test.bean;

public class Class {
    String name;
    int classNo;

    public Class(){
        System.out.println("Class 无参构造函数创建...");
    }
}
```

```xml
<bean name="student" class="com.test.bean.Student"/>
<bean name="class" class="com.test.bean.Class"/>
```

```java
Student 无参构造函数被创建...
Class 无参构造函数创建...
Bean stu1:	com.test.bean.Student@27c20538
```

- `bean` 加载顺序取决于配置文件的 `bean` 写入顺序，若要人为控制先手顺序，需要添加 `depends-on`
- 自定义 bean 加载顺序，`Student` `bean` 必须等待 `Class` `bean` 创建完成之后才会被创建

```xml
<bean name="student" class="com.test.bean.Student" depends-on="class"/>
```

```java
Class 无参构造函数创建...
Student 无参构造函数被创建...
Bean stu1:	com.test.bean.Student@59494225
```

- 给 `bean` 中的对象属性赋值
- 若属性没有 `set` 和 `get` 方法会异常
> Caused by: org.springframework.beans.NotWritablePropertyException: Invalid property 'name' of bean class [com.test.bean.Student]: Bean property 'name' is not writable or has an invalid setter method. Does the parameter type of the setter match the return type of the getter?

```xml
<bean name="student" class="com.test.bean.Student">
    <property name="name" value="张三"/>
    <property name="age" value="18"/>
</bean>
```

```java
Student 无参构造函数被创建...
Bean stu1:	com.test.bean.Student@23e028a9
Bean stu1 name:	张三
Bean stu1 age:	18
```

- 上面是属性赋值，那么对象呢?

```java
Class class1;

public Class getClass1() {
    return class1;
}

public void setClass1(Class class1) {
    this.class1 = class1;
}
```

```xml
<bean name="student" class="com.test.bean.Student">
    <property name="name" value="张三"/>
    <property name="age" value="18"/>
    <property name="class1" ref="class"/>
</bean>

<bean name="class" class="com.test.bean.Class"/>
```

- 当 `student` 设置原型模式时，每个不同的 `student` `bean` 中的 `class` 对象都会引用同一个 `class` `bean`
- 当 `class` 设置为原型模式时，`student` `bean` 中的 `class` 对象会引用不同的 `class` `bean`

# 如何在bean中配置属性
> Student.java

```java
List<String> list;

public List<String> getList() {
    return list;
}

public void setList(List<String> list) {
    this.list = list;
}
```

> config.xml

```xml
<bean name="student" class="com.test.bean.Student">
    <property name="name" value="张三"/>
    <property name="age" value="18"/>
    <property name="list">
        <list>
            <value type="double">100.0</value>
            <value type="double">101.0</value>
            <value type="double">102.0</value>
        </list>
    </property>
</bean>
```

> output

```java
Student 无参构造函数被创建...
Class 无参构造函数创建...
Bean stu1:	com.test.bean.Student@cd2dae5
stu1 list:	[100.0, 101.0, 102.0]
```

- `list` 和 `array` 效果相同

```xml
<property name="str">
    <array>
        <value type="double">10.0</value>
        <value type="double">11.0</value>
        <value type="double">12.0</value>
    </array>
 </property>
```

```xml
Student 无参构造函数被创建...
Class 无参构造函数创建...
Bean stu1:	com.test.bean.Student@cd2dae5
stu1 list:	[100.0, 101.0, 102.0]
stu1 list:	[10.0, 11.0, 12.0]
```

- Map

```xml
<property name="map">
    <map>
        <entry key="张三" value="99"/>
        <entry key="李四" value="23"/>
        <entry key="王五" value="66"/>
    </map>
</property>
```

```java
Student 无参构造函数被创建...
Class 无参构造函数创建...
Bean stu1:	com.test.bean.Student@79be0360
stu1 list:	[100.0, 101.0, 102.0]
stu1 array:	[10.0, 11.0, 12.0]
stu1 map:	{张三=99, 李四=23, 王五=66}
```

# 自动装配
- 之前是手动写入 class1 属性值

![autowrie 自动装配](https://fastly.jsdelivr.net/gh/lemonsoldout/pictures@main/1662602778782e306a9f993ececedadd0633b6cc8811c.png)

```xml
<bean name="student" class="com.test.bean.Student" autowire="byType">
```

- 当选择`byName` 时，下面 `bean` 的 `name` 要和 `set` 的 `name` 相同才能装配成功
- `card` 属性会被自动选择类型为 `Card` 的 `bean` 然后匹配到 `Student.java` 中的 `Card` `card`

----------
- 不使用 set 方法创建对象，指定一个有参构造方法创建对象

```java
public Student(String name){
    this.name = name;
    }
    
public Student(String name, int age){
    this.name = name;
    this.age = age;
}
```

```xml
<constructor-arg type="java.lang.String" value="王五"/>
```

```xml
<constructor-arg type="java.lang.String" index="0"/>
```

- index=0 表示只有一个参数，即使用第一个方法配置
- 如果只有第一个配置 name 的方法，那么不能直接使用 name="age" 配置除非再写一个相同的 age 方法

![](https://fastly.jsdelivr.net/gh/lemonsoldout/pictures@main/16626029717910608d973080429ef5c86ec531aed5585.png)

![](https://fastly.jsdelivr.net/gh/lemonsoldout/pictures@main/1662602995782b4f3fcb96cd5c61e2135037f4fefc75e.png)
