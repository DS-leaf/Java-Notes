# BeanFactory
[toc]

## 快速入门1 - Bean的简单创建和使用
**前置：** 创建一个maven项目
**1. 导入依赖**
```xml
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.3.18</version>
    </dependency>
```

**2. 创建接口**
```java
package jmu.yw.service;

public interface UserService {

}
```

**3. 创建接口实现类**
```java
package jmu.yw.service.impl;

import jmu.yw.service.UserService;

public class UserServiceImpl implements UserService {

}
```

**4. 配置SpringConfig**
在resource下创建Spring Config并配置Bean
![创建SpringConfig](/assets/创建SpringConfig.png)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="jmu.yw.service.impl.UserServiceImpl"></bean>

</beans>
```

### 测试
```java
    public static void main(String[] args) {
        //创建工厂对象
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        //创建读取器（xml文件）
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
        //读取配置文件给工厂
        reader.loadBeanDefinitions("beans.xml");
        //根据id获取Bean实例对象
        UserService userService = (UserService) beanFactory.getBean("userService");
        System.out.println(userService);
    }
```