## 1. 组件注册概述

组件注册通俗的讲就是往 `BeanFactory` 中添加 `Bean` 对象。组件注册的常见方式有如下五种：

- - 在 `Xml` 文件中配置 `Bean` 信息

- - 使用 `@Configuration` 与 `@Bean`

- - 使用 `@ComponentScan` 与组件注解（`@Controller`、`@Service`、`@Repository`、`@Component`）

- - 使用 `@Configuration` 与 `@Import` 注解，其中 `@Import` 的 `value` 属性可以有如下取值：

- - - Class
    - ImportSelector
    - ImportBeanDefinitionRegistrar

- - 使用 `FactoryBean` 接口的实现类进行 `Bean` 的注册，其中 `FactoryBean` 接口中提供了三个方法：

- - - getObject()
    - getObjectType()
    - isSingleton()



## 2. 五种注册方式



### 2.1 使用 Xml 注册 Bean

创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

在 `beans.xml` 配置文件中添加 `Person` 的配置信息

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--
        特别注意：
            1、默认情况下，创建出来的 bean 对象是 singleton 的
            2、容器启动时，会直接创建所有 scope 为 singleton 的 bean
    -->
    <bean id="person" class="org.example.entity.Person">
        <property name="name" value="张三"></property>
        <property name="age" value="18"></property>
    </bean>
</beans>
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);
    }
}
```

测试结果为：

```
Person(name=张三, age=18)
```



### 2.2 使用 @Bean 注册 Bean

创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建配置类 `MainConfig` 对象

```
@Configuration
public class MainConfig {
    /**
     *  特别注意：
     *      1、默认情况下，方法名称就是 Bean 的名称；
     *      2、如果想要指定 Bean 的名称，可以通过 @Bean 的 value 属性进行指定；
     *      3、默认请求下，这个 Bean 对象的属性值为默认值，除非我们为其设置了属性值；
     *      4、可以指定初始化和销毁方法：
     *          @Bean 的 initMethod 属性用于指定初始化方法
     *          @Bean 的 destroyMethod 属性用于指定销毁方法
     */
    @Bean(value = "person")
    public Person person() {
        return new Person();
    }
}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);
    }
}
```

测试结果为：

```
Person(name=张三, age=18)
```



### 2.3 使用组件注解注册

创建 `PersonController` 对象

```
/**
 * 特别说明：
 *      1、默认情况下，类名首字母小写即为 Bean 的名称，例如 personController
 *      2、可以通过 @Controller 的 value 属性指定 Bean 名称
 */
@Controller(value = "personController")
public class PersonController {
    
}
```

创建配置类 `MainConfig` 对象

```
/**
 * 特别说明：
 *      1、@ComponentScan 注解可以同时使用多个，用于扫描多个路径
 */
@ComponentScan(value = "org.example.controller")
//@ComponentScan(value = "org.example.service.impl")
//@ComponentScan(value = "org.example.dao")
@Configuration
public class MainConfig {

}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        PersonController personController = (PersonController) applicationContext.getBean("personController");
        System.out.println(personController);
    }
}
```

测试结果为：

```
org.example.controller.PersonController@48aaecc3
```



### 2.4 使用 @Import 注解注册



#### 2.4.1 方式一：@Import 里添加 Class 对象

创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建配置类 `MainConfig` 对象

```
@Configuration
@Import({Person.class})
public class MainConfig {

}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = (Person) applicationContext.getBean("org.example.entity.Person");
        System.out.println(person);
    }
}
```

测试结果为：

```
Person(name=null, age=null)
```



#### 2.4.2 方式二：@Import 里添加 ImportSelector 对象

创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建 `ImportSelector` 接口的实现类

```
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 需要往容器中添加哪些 Bean 对象，就往这个数组中添加该对象的全限定类名
        return new String[]{"org.example.entity.Person"};
    }
}
```

创建配置类 `MainConfig` 对象

```
@Configuration
@Import(value = {MyImportSelector.class})
public class MainConfig {

}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = (Person) applicationContext.getBean("org.example.entity.Person");
        System.out.println(student);
    }
}
```

测试结果为：

```
Person(name=null, age=null)
```



#### 2.4.3 方式三：@Import 里添加 ImportBeanDefinitionRegistrar 对象

创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建 `ImportBeanDefinitionRegistrar` 接口的实现类

```
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        BeanDefinition personDefinition = new RootBeanDefinition(Person.class);
        // 要注册什么Bean就往BeanDefinitionRegistry中添加该Bean的BeanDefinition
        registry.registerBeanDefinition("person",personDefinition);
    }
}
```

创建配置类 `MainConfig` 对象

```
@Configuration
@Import(value = {MyImportBeanDefinitionRegistrar.class})
public class MainConfig {

}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = (Person) applicationContext.getBean("person");
        // Person(name=null, age=null)
        System.out.println(person);
    }
}
```

测试结果为：

```
Person(name=null, age=null)
```



### 2.5 使用 FactoryBean 注册

创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建 `FactoryBean` 接口的实现类

```
public class PersonFactoryBean implements FactoryBean<Person> {
    @Override
    public Person getObject() throws Exception {
        // 添加对象
        return new Person();
    }

    @Override
    public Class<?> getObjectType() {
        // 指定类型
        return Person.class;
    }

    @Override
    public boolean isSingleton() {
        // 是否单利
        return true;
    }
}
```

创建配置类 `MainConfig` 对象

```
@Configuration
public class MainConfig {
    /**
     *  特别注意：
     *      此处虽然注册的是PersonFactoryBean对象，且默认该对象的名称为personFactoryBean，
     *      但是当我们使用getBean("personFactoryBean")获取对象时，获取到的是Person对象，即
     *      获取到的是PersonFactoryBean.getObject()中的对象。
     */
    @Bean
    public PersonFactoryBean personFactoryBean() {
        return new PersonFactoryBean();
    }
}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        System.out.println(applicationContext.getBean("personFactoryBean"));
        System.out.println(applicationContext.getBean("personFactoryBean").getClass());
    }
}
```

测试结果为：

```
Person(name=null, age=null)
class org.example.entity.Person
```



## 3. 其他相关注解



### 3.1 设置组件的作用域

在 `Spring` 中 `Bean` 的作用域默认是单例的，当 `IOC` 容器启动的时候，会为所有单例的 `Bean` 进行实例化（在堆中开辟了一块内存，属性值为默认值）。



如果有需要，可以通过 `@Scope` 注解指定 `Bean` 的作用域，该注解的取值范围如下：

```
//singleton：单实例的 --> 容器启动时直接实例化该作用域的Bean对象，每次获取时从容器中获取到的是同一个对象

//prototype：多实例的 --> 容器启动时不会实例化该作用域的Bean对象，而是在每次获取的时候才进行实例化，每次获取到的都是新创建的对象

//request：同一次请求创建一个实例

//session：同一个session创建的一个实例
```

接下来将创建一个 Person 与 Animal 对象进行测试

创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建 `Animal` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建配置类 `MainConfig` 对象

```
@Configuration
public class MainConfig {
    @Scope(value = "singleton")
    @Bean
    public Person person() {
        System.out.println("实例化（在堆中开辟了一块内存，属性值为默认值） Person 对象");
        return new Person();
    }
    
    @Scope(value = "prototype")
    @Bean
    public Animal animal() {
        System.out.println("实例化（在堆中开辟了一块内存，属性值为默认值） Animal 对象");
        return new Animal();
    }
}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        System.out.println("容器创建完成");
        Person person = applicationContext.getBean("person", Person.class);
        Animal animal = applicationContext.getBean("animal", Animal.class);
    }
}
```

测试结果如下，可以发现单例的 `Bean` 在容器创建时就会被实例化，而多例的 `Bean` 只有在获取的时候才会被实例化

```
实例化（在堆中开辟了一块内存，属性值为默认值） Person 对象
容器创建完成
实例化（在堆中开辟了一块内存，属性值为默认值） Animal 对象
```



### 3.2 设置组件加载模式为懒加载

懒加载：是专门针对于单例的 `Bean` 的操作

- - - 单实例的 `Bean`：默认是在容器启动的时候创建对象；
    - 懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）`Bean` 的时候创建对象；



创建 `Person` 对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;  
}
```

创建配置类 MainConfig 对象

```
@Configuration
public class MainConfig {
    /**
     *  作用域为单例的 Bean，默认是在容器启动的时候创建实例对象。我们可以在注册 Bean 的时候使用 @Lazy 指定该 Bean 进行懒加载，
     *  那么该 Bean 只有当被获取的时候才进行实例化操作。
     */
    @Lazy
    @Bean
    public Person person() {
        System.out.println("实例化（在堆中开辟了一块内存，属性值为默认值） Person 对象");
        return new Person();
    }
}
```

从容器中获取 `Bean` 对象

```
public class TestSpringAnnotation {
    @Test
    public void test() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        System.out.println("容器创建完成");
        // 单例的 Bean 标注了 @Lazy 注解之后，实例化时机为：第一次获取 Bean 的时候
        Person person = applicationContext.getBean("person", Person.class);
    }
}
```

测试结果

```
容器创建完成
实例化（在堆中开辟了一块内存，属性值为默认值） Person 对象
```



### 3.3 设置组件的注册条件

我们知道，默认情况下：单例的 `Bean` 在容器启动的时候就会实例化，而多例的 `Bean` 只有当被获取的时候才会实例化。那么能不能实现某个 Bean 符合某个条件才会被实例化。
