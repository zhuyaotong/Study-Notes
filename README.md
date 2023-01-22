
# study-notes

- [0. 面向对象设计与实现](#0-面向对象设计与实现)

  - [0.1 设计](#01-设计)

  - [0.2 特性](#02-特性)

- [1. 代码整洁之道](#1-代码整洁之道)
  
  - [1.1 取名](#11-取名)
  
  - [1.2 函数](#12-函数)
  
  - [1.3 注释](#13-注释)
  
  - [1.4 对象和数据结构](#14-对象和数据结构)

  - [1.5 单元测试](#15-单元测试)

  - [1.6 类](#16-类)

  - [1.7 系统](#17-系统)

  - [1.8 并发编程](#18-并发编程)

- [2. 重构](#2-重构)
  
  - [2.1 函数](#21-函数)

- [3. 高效学习](#3-高效学习)
  - [3.1 如何学习和阅读代码](#31-如何学习和阅读代码)
  
# 0. 面向对象设计与实现
  
  - ## 0.1 设计

    - 划分职责识别类
    
    - 定义属性与方法

    - 定义类之间的交互关系

    - 组装类并提供执行入口

  - ## 0.2 特性

    - 面向对象的编程有三大特性：封装、继承和多态。
  
    - 两个面向对象的核心理念：
      
      - “Program to an ‘interface’, not an ‘implementation’.”
      
        - 使用者不需要知道数据类型、结构、算法的细节。
        
        - 使用者不需要知道实现细节，只需要知道提供的接口。
        
        - 利于抽象、封装、动态绑定、多态。
        
        - 符合面向对象的特质和理念。

      - “Favor ‘object composition’ over ‘class inheritance’.”
        
          - 继承需要给子类暴露一些父类的设计和实现细节。
        
          - 父类实现的改变会造成子类也需要改变。
        
          - 我们以为继承主要是为了代码重用，但实际上在子类中需要重新实现很多父类的方法。
        
          - 继承更多的应该是为了多态。

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

# 2. 重构

  - ## 2.1 函数

    - Extract Method (提炼函数)

      - 你有一段代码可以被组织在一起并独立出来，将这段代码放进一个独立函数中，并让函数名称解释该函数的用途。

        ```java
        void printOwing(double amount) {
            printBanner();

            // print details
            System.out.println("name:" + _name);
            System.out.println("amount" + _amount);
        }
        ```

        ```java
        void printOwing(double amount) {
            printBanner();
            printDetails(amount);
        }

        // 提炼函数
        void printDetails(double amount) {
            System.out.println("name" + _name);
            System.out.println("amount" + amount);
        }
        ```

        注意目标代码中的局部变量。如果需要对局部变量进行再次赋值并使用，提炼出的函数可以使用参数以及返回值。

    - Replace Nested Conditional with Guard Clauses（以卫语句取代嵌套表达式）

      ```java
        double getPayAmount() {
            double result;
            if (_isDead) {
                result = deadAmount();
            }
            else {
                if (_isSeparted) {
                    result = separatedAmount();
                }
                else {
                    if (_isRetired) result = retiredAmount();
                    else result = normalPayAmount();
                }
            }
        }
      ```

      使用卫语句：

      ```java
        double getPayAmount() {
            if (_isDead) return deadAmount();
            if (_isSeparated) return separatedAmount();
            if (_isRetired) return retiredAmount();
            return normalPayAmount();
        }
      ```

      条件表达式通常有两种表现形式，一种是所有分支都是正常行为，另一种是只有一种是正常行为，其他都是特殊情况。
      对于第一种就应该使用if-else语句，而对于第二种就应当使用卫语句。卫语句通常从函数中返回或抛异常。

    - 引入参数对象：

      ```javascript
        function amountInvoiced(startDate, endDate)

        // ====>

        function amountInvoiced(dateRange)
      ```

    - 以对象取代基本类型：

      ```javascript
        order.filter(o => 'high' === o.priority
          || 'rush' === o.priority)

        // =====>

        order.filter(o => o.priority.hightThen(new Priority('normal')))
      ```


    - 拆分循环：

      ```javascript
        let averageAge = 0
        let totalSalary = 0
        for (const p of people) {
          averageAge += p.age
          totalSalary = p.salary
        }
        averageAge /= people.length

        // ======>

        let averageAge = 0
        for (const p of people) {
          averageAge += p.age
        }
        averageAge /= people.length

        let totalSalary = 0
        for (const p of people) {
          totalSalary = p.salary
        }
      ```

      也许上面的代码对于很多人来说都是一种坏味道, 甚至我现在也这样觉得, 因为理论上多执行了一次循环, 但是往往在很多的场景中, 循环并不是性能的瓶颈, 分开书写之后能带来更好的维护性和可读性, 也能方便的拆分代码逻辑。

    -  以多态取代条件表达式：

        ```javascript
          switch(bird.type) {
            case 'a':
              return 'a type'
            case 'b':
              return 'b type'
            default:
              return 'default'
          }

          // ====>

          class AType {
            getType() {
              return 'a type'
            }
          }

          class BType {
            getType() {
              return 'b type'
            }
          }
        ```

    - 将查询函数和修改函数分离：
      
      ```javascript
        function getTotalOutstandingAndSendBill() {
          const result = customer.invoices.reduce((total, each) => each.amount + total, 0)
          sendBill()
          return result;
        }

        // ====>

        function totalOutstanding() {
          return customer.invoices.reduce((total, each) => each.amount + total, 0)
        }
        function sendBill() {
          sendBill()
        }
      ```

      > 原书中有一句话我印象深刻, 明确表现出有副作用与无副作用两种函数之间的差异, 是个很好的想法。
      任何有返回值的函数, 都不应该有看得到的副作用–命令与查询分离。

# 3. 高效学习
  
  ## 3.1 如何学习和阅读代码
   
   - 代 码 => What, How & Details
   
   - 文档 / 书 => What, How & Why 

     **可见，代码并不会告诉你 Why，看代码只能靠猜测或推导来估计 Why，是揣测，不准确，所以会有很多误解。而且，我们每个人都知道，Why 是能让人一通百通的东西，也是能让人醍醐灌顶的东西。**

   - 如何阅读源代码

     1. 首先，在阅读代码之前，我建议你需要有下面的这些前提再去阅读代码，这样你读起代码来会很顺畅。
    
        1. 基础知识。相关的语言和基础技术的知识。

        1. 软件功能。你先要知道这个软件完成的是什么样的功能，有哪些特性，哪些配置项。你先要读一遍用户手册，然后让软件跑起来，自己先用一下感受一下。

        1. 相关文档。读一下相关的内部文档，Readme 也好，Release Notes 也好，    Design 也好，Wiki 也好，这些文档可以让你明白整个软件的方方面面。如果你的软件没有文档，那么，你只能指望这个软件的原作者还在，而且他还乐于交流。
      

        1. 代码的组织结构。也就是代码目录中每个目录是什么样的功能，每个文档是干什么的。如果你要读的程序是在某种标准的框架下组织的，比如：Java 的 Spring 框架，那么恭喜你，这些代码不难读了。

     2. 接下来，你要了解这个软件的代码是由哪些部分构成的，我在这里给你一个列表，供你参考。

        1. 接口抽象定义。任何代码都会有很多接口或抽象定义，其描述了代码需要处理的数据结构或者业务实体，以及它们之间的关系，理清楚这些关系是非常重要的。

        1. 模块粘合层。我们的代码有很多都是用来粘合代码的，比如中间件（middleware）、Promises 模式、回调（Callback）、代理委托、依赖注入等。这些代码模块间的粘合技术是非常重要的，因为它们会把本来平铺直述的代码给分裂开来，让你不容易看明白它们的关系。

        1. 业务流程。这是代码运行的过程。一开始，我们不要进入细节，但需要在高层搞清楚整个业务的流程是什么样的，在这个流程中，数据是怎么被传递和处理的。一般来说，我们需要画程序流程图或者时序处理图。

        1. 具体实现。了解上述的三个方面的内容，相信你对整个代码的框架和逻辑已经有了总体认识。这个时候，你就可以深入细节，开始阅读具体实现的代码了。对于代码的具体实现，一般来说，你需要知道下面一些事实，这样有助于你在阅读代码时找到重点。

            - 代码逻辑。代码有两种逻辑，一种是业务逻辑，这种逻辑是真正的业务处理逻辑；另一种是控制逻辑，这种逻辑只是用控制程序流转的，不是业务逻辑。比如：flag 之类的控制变量，多线程处理的代码，异步控制的代码，远程通讯的代码，对象序列化反序列化的代码等。这两种逻辑你要分开，很多代码之所以混乱就是把这两种逻辑混在一起了（详情参看《编程范式游记》）。

            - 出错处理。根据二八原则，20% 的代码是正常的逻辑，80% 的代码是在处理各种错误，所以，你在读代码的时候，完全可以把处理错误的代码全部删除掉，这样就会留下比较干净和简单的正常逻辑的代码。排除干扰因素，可以更高效地读代码。

            - 数据处理。只要你认真观察，就会发现，我们好多代码就是在那里倒腾数据。比如 DAO、DTO，比如 JSON、XML，这些代码冗长无聊，不是主要逻辑，可以不理。

            - 重要的算法。一般来说，我们的代码里会有很多重要的算法，我说的并不一定是什么排序或是搜索算法，可能会是一些其它的核心算法，比如一些索引表的算法，全局唯一 ID 的算法、信息推荐的算法、统计算法、通读算法（如 Gossip）等。这些比较核心的算法可能会非常难读，但它们往往是最有技术含量的部分。

            - 底层交互。有一些代码是和底层系统的交互，一般来说是和操作系统或是 JVM 的交互。因此，读这些代码通常需要一定的底层技术知识，不然，很难读懂。

            
              总结一下，阅读代码的方法如下：

              - 一般采用自顶向下，从总体到细节的“剥洋葱皮”的读法。

              - 画图是必要的，程序流程图，调用时序图，模块组织图……
              
              - 代码逻辑归一下类，排除杂音，主要逻辑才会更清楚。
              
              - debug 跟踪一下代码是了解代码在执行中发生了什么的最好方式。


    