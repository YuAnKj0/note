# @Autowired与@Resource与@Reference注解的区别

## 1、Spring自动装配byName和byType区别

### 1.1 byName:

byName会搜索整个配置文件中的bean,如果有相同名称的bean则自动配置，否则显示异常

### 1.2 byType:

byType会搜索整个配置文件中的bean，如果有相同类型的bean则自动配置，否则显示异常

## 2、@Autowired

@Autowired按byType自动注入，是由j2ee提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired。

## 3、@Resource

@Resource默认按byName自动注入，是由j2ee提供的注解，需要导入包javax.annotation.Resource。

### 3.1、@Resource装配属性

@Resource有两个重要的属性name和type，

1. 如果同时制订了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
2. 如果指定了name，则从上下文中找到名称（id）匹配的bean进行装配，找不到则抛出异常
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个都会抛出异常
4. 如果既没指定name，也没指定type，则自动按照byName方式进行装配，如果没有匹配，则回退为一个原始类型进行装配，如果匹配则自动装配

### 3.2、@Resource和@Autowired相互转换

​	以下两段代码是等价的：

#### 	@Autowired

```java
public class TestServiceImpl{
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao;
}
```

#### 	@Resource

```java
public class TestServiceImpl {
    @Resource(name="userDao")
    private UserDao userDao; // 用于字段上
}
```

## 4、@Reference

@Reference是dubbo的注解，它注入的是分布式的远程服务对象，需要dubbo配置使用。在微服务中，工程项目会分为多个模块（maven工程），每一个模块相当于一个服务，一个服务调用另一个服务的功能需要@Reference注解。
