
# 6. Spring
- [6.1 IoC 容器（控制反转容器（依赖注入））](#61-ioc-容器控制反转容器依赖注入)

- [6.2 Spring常用注解](#62-spring常用注解)

- [6.3 20%的业务代码的Spring声明式事务，可能都没处理正确](#63-20的业务代码的spring声明式事务可能都没处理正确)

## 6.1 IoC 容器（控制反转容器（依赖注入））

  控制反转（IOC，Inverse Of Control），即把创建对象的权利交给框架，也就是指将对象的创建、对象的存储、对象的管理交给了Spring容器。Spring容器是Spring中的一个核心模块，用于管理对象，底层可以理解为是一个Map集合。

  - 依赖注入（DI）

    在软件工程中，依赖注入（dependency injection，缩写为 DI）是一种软件设计模式，也是实现控制反转的其中一种技术。这种模式能让一个对象接收它所依赖的其他对象。“依赖”是指接收方所需的对象。“注入”是指将“依赖”传递给接收方的过程。在“注入”之后，接收方才会调用该“依赖”。

    Spring 中触发 IoC 容器“依赖注入” 的方式有两种，一个是应用程序通过 getBean()方法 向容器索要 bean 实例时触发依赖注入；另一个是提前给 bean 配置了 lazy-init 属性为 false，Spring 在 IoC 容器初始化会自动调用此 bean 的 getBean() 方法，提前完成依赖注入。总的来说，想提高运行时获取 bean 的效率，可以考虑配置此属性。

    首先看一下 AbstractBeanFactory 中的 getBean() 系列方法及 doGetBean() 具体实现：

    ```java
      public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {

        //---------------------------------------------------------------------
        // BeanFactory 接口的实现，下列的 getBean() 方法不论是哪种重载，最后都会走
        // doGetBean(final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly) 的具体实现
        //---------------------------------------------------------------------

        // 获取 IoC容器 中指定名称的 bean
        public Object getBean(String name) throws BeansException {
            return doGetBean(name, null, null, false);
        }

        // 获取 IoC容器 中指定名称和类型的 bean
        public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
            return doGetBean(name, requiredType, null, false);
        }

        // 获取 IoC容器 中指定名称和参数的 bean
        public Object getBean(String name, Object... args) throws BeansException {
            return doGetBean(name, null, args, false);
        }

        // 获取 IoC容器 中指定名称、类型和参数的 bean
        public <T> T getBean(String name, Class<T> requiredType, Object... args) throws BeansException {
            return doGetBean(name, requiredType, args, false);
        }

        // 真正实现向 IoC容器 获取 bean 的功能，也是触发 依赖注入(DI) 的地方
        @SuppressWarnings("unchecked")
        protected <T> T doGetBean(final String name, final Class<T> requiredType, final Object[] args,
                boolean typeCheckOnly) throws BeansException {

            // 根据用户给定的名称(也可能是别名alias) 获取 IoC容器 中与 BeanDefinition 唯一对应的 beanName
            final String beanName = transformedBeanName(name);
            Object bean;

            // 根据 beanName 查看缓存中是否有已实例化的 单例bean，对于 单例bean，整个 IoC容器 只创建一次
            Object sharedInstance = getSingleton(beanName);
            if (sharedInstance != null && args == null) {
                if (logger.isDebugEnabled()) {
                    if (isSingletonCurrentlyInCreation(beanName)) {
                        logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
                                "' that is not fully initialized yet - a consequence of a circular reference");
                    }
                    else {
                        logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
                    }
                }
                // 获取给定 bean 的实例对象，主要是完成 FactoryBean 的相关处理
                // 注意：BeanFactory 是一个 IoC容器，它保存了 bean 的基本配置信息。
                // 而 FactoryBean 是 IoC容器 中一种特殊的 bean，它能够实例化 bean对象，注意两者之间的区别
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
            }
            // 如果应用程序要获取的 bean 还未创建
            else {
                if (isPrototypeCurrentlyInCreation(beanName)) {
                    throw new BeanCurrentlyInCreationException(beanName);
                }

                // 获取当前容器的父容器
                BeanFactory parentBeanFactory = getParentBeanFactory();
                // 如果当前容器中没有指定的 bean，且当前容器的父容器不为空
                // 则从父容器中去找，如果父容器也没有，则沿着当前容器的继承体系一直向上查找
                if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
                    // 根据用户传入的 name(有可能是别名alias)，获取唯一标识的 beanName
                    String nameToLookup = originalBeanName(name);
                    if (args != null) {
                        // 委派父级容器根据指定名称和显式的参数查找
                        return (T) parentBeanFactory.getBean(nameToLookup, args);
                    }
                    else {
                        // 委派父容器根据指定名称和类型查找
                        return parentBeanFactory.getBean(nameToLookup, requiredType);
                    }
                }

                // 创建的 bean 是否需要进行类型验证，一般不需要
                if (!typeCheckOnly) {
                    // 向容器标记指定的 bean 已经被创建
                    markBeanAsCreated(beanName);
                }

                try {
                    // 根据 beanName 获取对应的 RootBeanDefinition
                    final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
                    checkMergedBeanDefinition(mbd, beanName, args);

                    // 获取当前 bean 所依赖bean 的 beanName，下面的 getBean(dependsOnBean) 方法会触发
                    // getBean() 的递归调用，直到取到一个不依赖任何其它 bean 的 bean 为止。
                    // 比如：beanA 依赖了 beanB，而 beanB 依赖了 beanC，那么在实例化 beanA 时会先实例化
                    // beanC，然后实例化 beanB 并将 beanC 注入进去，最后实例化 beanA 时将 beanB 注入
                    String[] dependsOn = mbd.getDependsOn();
                    if (dependsOn != null) {
                        for (String dependsOnBean : dependsOn) {
                            // 递归调用 getBean() 方法，从末级节点依次实例化 依赖的bean
                            getBean(dependsOnBean);
                            // 把 当前bean 直接依赖的bean 进行注册
                            //（也就是通过 setter 或构造方法将依赖的 bean 赋值给当前 bean 对应的属性）
                            registerDependentBean(dependsOnBean, beanName);
                        }
                    }

                    // 如果当前 bean 是单例的
                    if (mbd.isSingleton()) {
                        // 这里使用了一个匿名内部类，创建 bean实例对象
                        sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
                            public Object getObject() throws BeansException {
                                try {
                                    // 根据给定的 beanName 及 RootBeanDefinition对象，创建 bean 实例对象
                                    return createBean(beanName, mbd, args);
                                }
                                catch (BeansException ex) {
                                    destroySingleton(beanName);
                                    throw ex;
                                }
                            }
                        });
                        // 获取给定 bean 的实例对象
                        bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                    }

                    // 创建原型模式的 bean 实例对象
                    else if (mbd.isPrototype()) {
                        // 原型模式 (Prototype) 每次都会创建一个新的对象
                        Object prototypeInstance = null;
                        try {
                            // 回调 beforePrototypeCreation() 方法，默认的功能是注册当前创建的原型对象
                            beforePrototypeCreation(beanName);
                            // 创建指定 bean 对象实例
                            prototypeInstance = createBean(beanName, mbd, args);
                        }
                        finally {
                            // 回调 afterPrototypeCreation() 方法，默认的功能是告诉 IoC容器
                            // 指定 bean 的原型对象不再创建了
                            afterPrototypeCreation(beanName);
                        }
                        // 获取给定 bean 的实例对象
                        bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
                    }

                    // 要创建的 bean 既不是单例模式，也不是原型模式，则根据该 bean元素 在配置文件中
                    // 配置的生命周期范围，选择实例化 bean 的合适方法，这种在 Web 应用程序中
                    // 比较常用，如：request、session、application 等的生命周期
                    else {
                        // 获取此 bean 生命周期的范围
                        String scopeName = mbd.getScope();
                        final Scope scope = this.scopes.get(scopeName);
                        // bean 定义资源中没有配置生命周期范围，则该 bean 的配置不合法
                        if (scope == null) {
                            throw new IllegalStateException("No Scope registered for scope '" + scopeName + "'");
                        }
                        try {
                            // 这里又使用了一个 ObjectFactory 的匿名内部类，获取一个指定生命周期范围的实例
                            Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                                public Object getObject() throws BeansException {
                                    beforePrototypeCreation(beanName);
                                    try {
                                        return createBean(beanName, mbd, args);
                                    }
                                    finally {
                                        afterPrototypeCreation(beanName);
                                    }
                                }
                            });
                            // 获取给定 bean 的实例对象
                            bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                        }
                        catch (IllegalStateException ex) {
                            throw new BeanCreationException(beanName,
                                    "Scope '" + scopeName + "' is not active for the current thread; " +
                                    "consider defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                                    ex);
                        }
                    }
                }
                catch (BeansException ex) {
                    cleanupAfterBeanCreationFailure(beanName);
                    throw ex;
                }
            }

            // 对要返回的 bean实例对象 进行非空验证和类型检查，如果没问题就返回这个已经完成 依赖注入的bean
            if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
                try {
                    return getTypeConverter().convertIfNecessary(bean, requiredType);
                }
                catch (TypeMismatchException ex) {
                    if (logger.isDebugEnabled()) {
                        logger.debug("Failed to convert bean '" + name + "' to required type [" +
                                ClassUtils.getQualifiedName(requiredType) + "]", ex);
                    }
                    throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
                }
            }
            return (T) bean;
          }
      }
    ```

    总的来说，getBean() 方法是依赖注入的起点，之后会调用 createBean()，根据之前解析生成的 BeanDefinition 对象 生成 bean 对象。


## 6.2 Spring常用注解

- 用于注册bean对象注解

    - @Component

        > 作用：调用无参构造创建一个bean对象，并把对象存入Spring的IoC容器，交由Spring容器进行管理。相当于在xml中配置一个Bean。<br>
        > 属性：value：指定Bean的id。如果不指定value属性，默认Bean的id是当前类的类名，首字母小写。

        - 子注解

            以下三个注解是从@Component派生出来的，它们的作用与@Component是一样的，Spring增加这三个注解是为了在语义行区分MVC三层架构对象。

            ```java
                @Component：用于注册非表现层、业务层、持久层的对象
                @Controller：用于注册表现层对象  
                @Service：用于注册业务层对象  
                @Reposity：用于注册持久层对象
            ```


    - @Bean

        > 作用：用于把当前方法的返回值（对象）作为Bean对象存入Spring的IoC容器中（注册Bean）<br>
        > 属性：name/value：用于指定Bean的id。当不写时，默认值是当前方法的名称。注意：当我们使用注解配置方法时，如果方法有参数，Spring框架会去容器中查找有没有可用的Bean对象，查找的方式和Autowired注解的方式是一样的。

        案例：

        ```java
            @Configuration
            public class JdbcConfig {
                /**
                * 用于创建QueryRunner对象
                * @param dataSource
                * @return
                */
                @Bean(value = "queryRunner")
                public QueryRunner createQueryRunner(DataSource dataSource) {
                    return new QueryRunner(dataSource);
                }

                /**
                * 创建数据源对象
                * @return
                */
                @Bean(value = "dataSource")
                public DataSource createDataSource() {
                    try {
                        ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
                        comboPooledDataSource.setDriverClass("com.mysql.jdbc.Driver");
                        comboPooledDataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydatabase");
                        comboPooledDataSource.setUser("root");
                        comboPooledDataSource.setPassword("root");
                        return comboPooledDataSource;
                    }catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                }
            }
        ```

- 用于依赖注入的注解

    - @Autowired
        
        > 作用：@Autowire和@Resource都是Spring支持的注解形式动态装配Bean的方式。Autowire默认按照类型(byType)装配，如果想要按照名称(byName)装配，需结合@Qualifier注解使用。<br>
        > 属性：required：@Autowire注解默认情况下要求依赖对象必须存在。如果不存在，则在注入的时候会抛出异常。如果允许依赖对象为null，需设置required属性为false。

        案例：
        ```java
            @Autowired /* Autowire默认按照类型(byType)装配 */
            // @Autowired(required = false) 
            /* @Autowire注解默认情况下要求依赖对象必须存在。如果不存在，则在注入的时候会抛出异常。如果允许依赖对象为null，需设置required属性为false。 */
            // @Qualifier("userService") /* 按照名称(byName)装配 */
            private UserService userService;
        ```

        
    - @Qualifier

        > 作用：@Autowired是按照名称(byName)装配，使用了@Qualifier注解之后就变成了按照名称(byName)装配。它在给字段注入时不能独立使用，必须和@Autowire一起使用；但是给方法参数注入时，可以独立使用。<br>
        > 属性：value：用于指定要注入的bean的id，其中，该属性可以省略不写。

        案例：
        ```java
            @Autowired
            @Qualifier(value="userService") 
            //@Qualifier("userService")     //value属性可以省略不写
            private UserService userService;
        ```	

    - @Resource
        
        > 作用：@Autowire和@Resource都是Spring支持的注解形式动态装配bean的方式。@Resource默认按照名称(byName)装配，名称可以通过name属性指定。如果没有指定name，则注解在字段上时，默认取（name=字段名称）装配。如果注解在setter方法上时，默认取（name=属性名称）装配。<br>
        
        > 属性：name：用于指定要注入的bean的id<br>type：用于指定要注入的bean的type

        装配顺序：

        1. 如果同时指定name和type属性，则找到唯一匹配的bean装配，未找到则抛异常；
        2. 如果指定name属性，则按照名称(byName)装配，未找到则抛异常；
        3. 如果指定type属性，则按照类型(byType)装配，未找到或者找到多个则抛异常；
        4. 既未指定name属性，又未指定type属性，则按照名称(byName)装配；如果未找到，则按照类型(byType)装配。

        案例：
        ```java
            @Resource
            @Resource(name = "userService", type = UserService.class)
            private UserService userService;
        ```	

    - @Value

        > 作用：通过@Value可以将外部的值动态注入到Bean中，可以为基本类型数据和String类型数据的变量注入数据

        案例：
        ```java
            // 1.基本类型数据和String类型数据的变量注入数据
            @Value("tom") 
            private String name;
            @Value("18") 
            private Integer age;

            /*

                // 2.从properties配置文件中获取数据并设置到成员变量中
                // 2.1jdbcConfig.properties配置文件定义如下
                jdbc.driver = com.mysql.jdbc.Driver  
                jdbc.url = jdbc:mysql://localhost:3306/eesy  
                jdbc.username = root  
                jdbc.password = root

            */

            // 2.2获取数据如下
            @Value("${jdbc.driver}")  
            private String driver;

            @Value("${jdbc.url}")  
            private String url;  
            
            @Value("${jdbc.username}")  
            private String username;  
            
            @Value("${jdbc.password}")  
            private String password;
        ```	

- 生命周期相关的注解

    - @PostConstruct

        > 作用：指定初始化方法

        案例：
        ```java
            @PostConstruct  
            public void init() {  
                System.out.println("初始化方法执行");  
            }
        ```	

    - @PreDestroy

        
        > 作用：指定销毁方法

        案例：
        ```java
            @PreDestroy  
            public void destroy() {  
                System.out.println("销毁方法执行");  
            }
        ```	


## 6.3 20%的业务代码的Spring声明式事务，可能都没处理正确

### 6.3.1 小心 Spring 的事务可能没有生效
@Transactional 生效原则 1，除非特殊配置（比如使用 AspectJ 静态织入实现 AOP），否则只有定义在 public 方法上的 @Transactional 才能生效。原因是，Spring 默认通过动态代理的方式实现 AOP，对目标方法进行增强，private 方法无法代理到，Spring 自然也无法动态增强事务处理逻辑。<br>
```java
    @Service
    @Slf4j
    public class UserService {
        @Autowired
        private UserRepository userRepository;

        //一个公共方法供Controller调用，内部调用事务性的私有方法
        public int createUserWrong1(String name) {
            try {
                this.createUserPrivate(new UserEntity(name));
            } catch (Exception ex) {
                log.error("create user failed because {}", ex.getMessage());
            }
            return userRepository.findByName(name).size();
        }

        //标记了@Transactional的private方法
        @Transactional
        private void createUserPrivate(UserEntity entity) {
            userRepository.save(entity);
            if (entity.getName().contains("test"))
                throw new RuntimeException("invalid username!");
        }

        //根据用户名查询用户数
        public int getUserCount(String name) {
            return userRepository.findByName(name).size();
        }
    }
```

@Transactional 生效原则 2，必须通过代理过的类从外部调用目标方法才能生效。<br>
```java
    public int createUserWrong2(String name) {
        try {
            this.createUserPublic(new UserEntity(name));
        } catch (Exception ex) {
            log.error("create user failed because {}", ex.getMessage());
        }
    return userRepository.findByName(name).size();
    }

    //标记了@Transactional的public方法
    @Transactional
    public void createUserPublic(UserEntity entity) {
        userRepository.save(entity);
        if (entity.getName().contains("test"))
            throw new RuntimeException("invalid username!");
    }

    
    @GetMapping("right2")
    public int right2(@RequestParam("name") String name) {
        try {
            userService.createUserPublic(new UserEntity(name));
        } catch (Exception ex) {
            log.error("create user failed because {}", ex.getMessage());
        }
        return userService.getUserCount(name);
    }
```

### 6.3.2 事务即便生效也不一定能回滚

第一，只有异常传播出了标记了 @Transactional 注解的方法，事务才能回滚。在 Spring 的 TransactionAspectSupport 里有个 invokeWithinTransaction 方法，里面就是处理事务的逻辑。可以看到，只有捕获到异常才能进行后续事务处理：
```java
    try {
        // This is an around advice: Invoke the next interceptor in the chain.
        // This will normally result in a target object being invoked.
        retVal = invocation.proceedWithInvocation();
    }
    catch (Throwable ex) {
        // target invocation exception
        completeTransactionAfterThrowing(txInfo, ex);
        throw ex;
    }
    finally {
        cleanupTransactionInfo(txInfo);
    }
```

第二，默认情况下，出现 RuntimeException（非受检异常）或 Error 的时候，Spring 才会回滚事务。<br>
打开 Spring 的 DefaultTransactionAttribute 类能看到如下代码块，可以发现相关证据，通过注释也能看到 Spring 这么做的原因，大概的意思是受检异常一般是业务异常，或者说是类似另一种方法的返回值，出现这样的异常可能业务还能完成，所以不会主动回滚；而 Error 或 RuntimeException 代表了非预期的结果，应该回滚：
```java
    /**
    * The default behavior is as with EJB: rollback on unchecked exception
    * ({@link RuntimeException}), assuming an unexpected outcome outside of any
    * business rules. Additionally, we also attempt to rollback on {@link Error} which
    * is clearly an unexpected outcome as well. By contrast, a checked exception is
    * considered a business exception and therefore a regular expected outcome of the
    * transactional business method, i.e. a kind of alternative return value which
    * still allows for regular completion of resource operations.
    * <p>This is largely consistent with TransactionTemplate's default behavior,
    * except that TransactionTemplate also rolls back on undeclared checked exceptions
    * (a corner case). For declarative transactions, we expect checked exceptions to be
    * intentionally declared as business exceptions, leading to a commit by default.
    * @see org.springframework.transaction.support.TransactionTemplate#execute
    */
    @Override
    public boolean rollbackOn(Throwable ex) {
        return (ex instanceof RuntimeException || ex instanceof Error);
    }
```
接下来，我和你分享 2 个反例。<br>
```java
    @Service
    @Slf4j
    public class UserService {
        @Autowired
        private UserRepository userRepository;
        
        //异常无法传播出方法，导致事务无法回滚
        @Transactional
        public void createUserWrong1(String name) {
            try {
                userRepository.save(new UserEntity(name));
                throw new RuntimeException("error");
            } catch (Exception ex) {
                log.error("create user failed", ex);
            }
        }

        //即使出了受检异常也无法让事务回滚
        @Transactional
        public void createUserWrong2(String name) throws IOException {
            userRepository.save(new UserEntity(name));
            otherTask();
        }

        //因为文件不存在，一定会抛出一个IOException
        private void otherTask() throws IOException {
            Files.readAllLines(Paths.get("file-that-not-exist"));
        }
    }
```
第一，如果你希望自己捕获异常进行处理的话，也没关系，可以手动设置让当前事务处于回滚状态：
```java
    @Transactional
    public void createUserRight1(String name) {
        try {
            userRepository.save(new UserEntity(name));
            throw new RuntimeException("error");
        } catch (Exception ex) {
            log.error("create user failed", ex);
            TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
        }
    }
```
第二，在注解中声明，期望遇到所有的 Exception 都回滚事务（来突破默认不回滚受检异常的限制）：
```java
    @Transactional(rollbackFor = Exception.class)
    public void createUserRight2(String name) throws IOException {
        userRepository.save(new UserEntity(name));
        otherTask();
    }
```

### 6.3.2 请确认事务传播配置是否符合自己的业务逻辑

让子逻辑在独立事务中运行，也就是改一下 SubUserService 注册子用户的方法，为注解加上 propagation = Propagation.REQUIRES_NEW 来设置 REQUIRES_NEW 方式的事务传播策略，也就是执行到这个方法时需要开启新的事务，并挂起当前事务：
```java
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createSubUserWithExceptionRight(UserEntity entity) {
        log.info("createSubUserWithExceptionRight start");
        userRepository.save(entity);
        throw new RuntimeException("invalid status");
    }

    
    @Transactional
    public void createUserRight(UserEntity entity) {
        createMainUser(entity);
        try{
            subUserService.createSubUserWithExceptionRight(entity);
        } catch (Exception ex) {
            // 捕获异常，防止主方法回滚
            log.error("create sub user error:{}", ex.getMessage());
        }
    }
```
