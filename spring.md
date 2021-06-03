- 什么是注解

  - Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能
  - 注解本质上继承 Annotation 接口,我们可以通过反射获取注解的相关信息,从而做些逻辑操作
  - springboot里面大量使用了注解，@Controller 、@RestController 、@Service、 @Autowire 等

- 什么是Spring框架

  - 什么是Spring：轻量级的 DI / IoC 和 AOP 容器的开源框架

  - https://spring.io/projects/spring-framework

  - bean

  - 有啥好处：

    - 管理创建和组装对象之间的依赖关系, 加了spring注解的类会自动创建一个实例，加到IOC容器里面，然后看哪里需要它，就自动赋值过去

      - 使用前：手工创建

      ```
      Controller -> Service -> Dao
      ```

      ```
      UserControoler
      ```

      ```
      private UserService userService = new UserService();
      ```

      - 使用后：Spring创建，自动注入

      ```
      Controller -> Service -> Dao
      ```

      ```
      UserControoler
      ```

      ```
      @Autowire
      ```

      ```
      private UserService userService 
      ```

- CDN ： 内容分发网络

