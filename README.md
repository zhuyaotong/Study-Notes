# study-notes
笔记...

- [面向对象设计与实现](#面向对象设计与实现)

- [代码整洁之道](#代码整洁之道)
  - [取名](#取名)
  - [函数](#函数)
  - [注释](#注释)
  - [对象和数据结构](#对象和数据结构)

# 面向对象设计与实现
  
  - 划分职责识别类
  
  - 定义属性与方法

  - 定义类之间的交互关系

  - 组装类并提供执行入口

# 代码整洁之道

  - ## 取名
    
    - 使用描述性的名称  
      
      - 别害怕长名称，长而具有描述性的名称，要比短而令人费解的名称好，要比描述性的长注释好。

  - ## 函数
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

  - ## 注释
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


  - ## 对象和数据结构

    - 对象：暴露行为（接口），隐藏数据（私有变量）。
    
    - 数据结构：没有明显的行为（接口），暴露数据。如DTO（Data Transfer Objects）、Entity。

    
