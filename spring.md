
# 目录

- IoC 容器（控制反转容器（依赖注入））

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

