











## 7、Bean的自动装配

+ 自动装配是Spring满足Bean依赖的一种方式！
+ Spring会在上下文中自动寻找，并自动给Bean装配属性！



在Spring中有三种装配方式：

1. 在xml中显式配置
2. 在Java中显式配置
3. 隐式 自动装配



**自动装配**

+ byname：需要保证所有bean的id唯一
+ bytype：需要每个类型只有一个bean，可以不配bean id



**使用注解自动装配**

1. 导入约束
2. 配置注解的支持 <context:annotation-config/>（IDEA写这个会自动导入约束）

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
</beans>
```

+ Autowired自动根据type装配，type相同再根据name装配（Java注解Resource）
+ 可以添加Qualifier过滤根据name装配



**Autowired和Resource的区别：**

Autowired首先通过Type装配；Resource优先使用Name装配；









