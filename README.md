# study-notes
笔记...

- [0. 面向对象设计与实现](#0-面向对象设计与实现)

- [1. 代码整洁之道](#1-代码整洁之道)
  
  - [1.1 取名](#11-取名)
  
  - [1.2 函数](#12-函数)
  
  - [1.3 注释](#13-注释)
  
  - [1.4 对象和数据结构](#14-对象和数据结构)

  - [1.5 单元测试](#15-单元测试)

  - [1.6 类](#16-类)

  - [1.7 系统](#17-系统)

  - [1.8 并发编程](#18-并发编程)

# 0. 面向对象设计与实现
  
  - 划分职责识别类
  
  - 定义属性与方法

  - 定义类之间的交互关系

  - 组装类并提供执行入口

# 1. 代码整洁之道

  - ## 1.1 取名
    
    - 使用描述性的名称  
      
      - 别害怕长名称，长而具有描述性的名称，要比短而令人费解的名称好，要比描述性的长注释好。

  - ## 1.2 函数
    - 只做一件事
      
      - 如果函数只是做了该函数名下同一抽象层上的步骤，则函数只做了一件事。
      
      - 要判断函数是否不止做了一件事，就是要看是否能再拆出一个函数。
    
    - 参数
      
      - 如果函数需要两个、三个或者三个以上参数，就说明其中一些参数应该封装成类了。
      
      - 将参数的顺序编码进函数名，减轻记忆参数顺序的负担，例如，assertExpectedEqualsActual(expected, actual)。

    - 读写分离
      
      - if (set("username", "unclebob")) { ... } 的含义模糊不清。应该改为:
        
        ```java
          if (attributeExists("username")) { 
              setAttribute("username", "unclebob");
              ...
          }
        ```

    - 使用异常机制
      
      ```java
        try {
            deletePage();
            xxx();
            yyy();
        } catch (Exception e) {
            log(e->getMessage());
        }
      ```

      - try/catch代码块丑陋不堪，所以最好把try和catch代码块的主体抽离出来，单独形成函数。
        
        ```java
          try {
              do();
          } catch (Exception e) {
              handle();
          }
        ```

    - 消除重复

      - 两个方法提取共性到新方法中，新方法分解到另外的类里，从而提升其可见性。
        模板方法模式是消除重复的通用技巧。

        ```java
          // 原始逻辑
          public class VacationPolicy() {
              public void accrueUSDivisionVacation() {
                  //do x;
                  //do US y;
                  //do z;
              }
              public void accrueEUDivisionVacation() {
                  //do x;
                  //do EU y;
                  //do z;
              }
          }

          // 模板方法模式重构之后
          abstract public class VacationPolicy {
              public void accrueVacation() {
                  x();
                  y();
                  z();
              }
              private void x() {
                  //do x;
              }
              abstract protected void y() {
              
              }
              private void z() {
                  //do z;
              }
          }
          public class USVacationPolicy extends VacationPolicy {
              protected void y() {
                  //do US y;
              }
          }
          public class EUVacationPolicy extends VacationPolicy {
              protected void y() {
                  //do EU y;
              }
          }

        ```

  - ## 1.3 注释
    - 用代码来阐述
      - 创建一个与注释所言同一事物的函数即可。
        
        ```java
          // check to see if the employee is eligible for full benefits
          if ((employee.falgs & HOURLY_FLAG) && (employee.age > 65))
          
          // 下面这个更好
          if (employee.isEligibleForFullBenefits())
        ```

    - 能用函数或者变量表示就别用注释:

      ```java
      // does the module from the global list <mod> 
      // depend on the subsystem we are part of?
      if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem())

      // 可以改为
      ArrayList moduleDependees = smodule.getDependSubsystems();
      String ourSubSystem = subSysMod.getSubSystem();
      if (moduleDependees.contains(ourSubSystem))
      ```


  - ## 1.4 对象和数据结构

    - 对象：暴露行为（接口），隐藏数据（私有变量）。
    
    - 数据结构：没有明显的行为（接口），暴露数据。如DTO（Data Transfer Objects）、Entity。


  - ## 1.5 单元测试

    - 每个测试一个断言
  
      - 每个测试函数有且仅有一个断言语句，每个测试函数中只测试一个概念。 

    - 整洁的测试依赖于FIRST规则

      - fast: 测试代码应该能够快速运行，因为我们需要频繁运行它。

      - independent: 测试应该相互独立，某个测试不应该依赖上一个测试的结果，测试可以以任何顺序进行。

      - repeatable: 测试应可以在任何环境中通过。

      - self-validating: 测试应该有bool值输出，不应通过查看日志来确认测试结果，不应手工对比两个文本文件确认测试结果。

      - timely: 及时编写测试代码。单元测试应该在生产代码之前编写，否则生产代码会变得难以测试。

  - ## 1.6 类

    - 类应该短小

    - 为修改而组织

      - 类应当对扩展开放，对修改封闭（开放闭合原则）。
      
      - 在理想系统中，我们通过扩展系统而非修改现有代码来添加新特性。

  - ## 1.7 系统

    - 将系统的构造与使用分开

      - 依赖注入（DI），控制反转（IoC）是分离构造与使用的强大机制。

      - 有时应用程序也需要负责确定何时创建对象，我们可以使用抽象工厂模式让应用自行控制何时创建对象，但构造能力在工厂实现类里，与使用部分分开。


  - ## 1.8 并发编程
    
    - 防御并发代码问题的原则与技巧

      - 遵循单一职责原则。分离并发代码与非并发代码。
      
      - 限制临界区数量、限制对共享数据的访问。
      
      - 避免使用共享数据，使用对象的副本。
      
      - 线程尽可能地独立，不与其他线程共享数据。

      