
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

  - [1.9 日志](#19-日志)

- [2. 重构](#2-重构)
  
  - [2.1 函数](#21-函数)

- [3. 高效学习](#3-高效学习)
  - [3.1 如何学习和阅读代码](#31-如何学习和阅读代码)

- [4. 规范](#4-规范)

  - [4.1 日志规范](#41-日志规范)

- [5. Java 并发编程实战](./concurrency.md)

- [6. Spring](./spring.md)

  
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
      
      - 别害怕长名称，长而具有描述性的名称，要比短而令人费解的名称好，要比描述性的长注释好。通过阅读名字，就可以知道它为什么存在，它是什么，它做什么，它怎么用。
        
        ```java
          // before
          int d; // elapsed time in days
          // after
          int elapsedTimeInDays;
        ```

        ```java
          // before
          public List<int[]> getThem() {
              List<int[]> list1 = new ArrayList<int[]>();
              for (int[] x : theList){
                  if (x[0] == 4)
                  list1.add(x);
              }
              return list1;
          }
          // after
          public List<Cell> getFlaggedCells() {
              List<Cell> flaggedCells = new ArrayList<Cell>();
              for (Cell cell : gameBoard){
                  if (cell.isFlagged())
                      flaggedCells.add(cell);
              }
              return flaggedCells;
          }
        ```
      
      - 做有意义的区分
        - 尽量不以数字系列命名，如下：
          
          ```java
            public static void copyChars(char a1[], char a2[]) {
                for (int i = 0; i < a1.length; i++) {
                    a2[i] = a1[i];
                }
            }
          ```

          可以修改如下:

          ```java
            public static void copyChars(char source[], char destination[]) {
              for (int i = 0; i < source.length; i++) {
                  destination[i] = source[i];
              }
            }
          ```

      - 使用读的出的名字

        - 编程是种社会活动。一个可以读的名字，能使人的记忆和交流更方便。

          ```java
            // before
            class DtaRcrd102 {
                private Date genymdhms;
                private Date modymdhms;
                private final String pszqint = "102";
                /* ... */
            };

            // after
            class Customer {
                private Date generationTimestamp;
                private Date modificationTimestamp;;
                private final String recordId = "102";
                /* ... */
            };
          ```

      - 使用可搜索的名字

        > 尽量不要用单字母和数字作为名字，很难被搜索。如英文字母e，是最常用的字母。
        单字母可以用于局部变量。换而言之，名字的长短最好与它作用范围的大小成正比。
        如果变量在代码的多处被使用，应该取一个好搜索的名字。

        ```java
          // before
          for (int j=0; j<34; j++) {
              s += (t[j]*4)/5;
          }
          // after
          int realDaysPerIdealDay = 4;
          const int WORK_DAYS_PER_WEEK = 5;
          int sum = 0;
          for (int j=0; j < NUMBER_OF_TASKS; j++) {
              int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
              int realTaskWeeks = (realdays / WORK_DAYS_PER_WEEK);
              sum += realTaskWeeks;
          }
        ```

  - ## 1.2 函数

    - 方法名
      
      - 方法名应该是个动词。对属性访问器、修改器和断言，应该加上get、set、is前缀。

        ```java
          string name = employee.getName();
          customer.setName("mike");
          if (paycheck.isPosted())...
        ```

      - 重载构造器时，使用描述参数的静态工厂方法。

        ```java
          // before
          Complex fulcrumPoint = new Complex(23.0);
          // after, this is better
          Complex fulcrumPoint = Complex.FromRealNumber(23.0);
        ```

    - 只做一件事
      
      - 如果函数只是做了该函数名下同一抽象层上的步骤，则函数只做了一件事。
      
      - 要判断函数是否不止做了一件事，就是要看是否能再拆出一个函数。

    - 添加有意义的语境

      需要用良好命名的类、函数或者名称空间，来为读者提供语境。如果没有，就只能加前缀了。

      没语境的代码：

      ```java
        private void printGuessStatistics(char candidate, int count) {
          String number;
          String verb;
          String pluralModifier;
          if (count == 0) {
              number = "no";
              verb = "are";
              pluralModifier = "s";
          } else if (count == 1) {
              number = "1";
              verb = "is";
              pluralModifier = "";
          } else {
              number = Integer.toString(count);
              verb = "are";
              pluralModifier = "s";
          }
          String guessMessage = String.format(
              "There %s %s %s%s", verb, number, candidate, pluralModifier
              );
          print(guessMessage);
        }
      ```

      有语境的代码：

      ```java
        public class GuessStatisticsMessage {
          private String number;
          private String verb;
          private String pluralModifier;
          
          public String make(char candidate, int count) {
              createPluralDependentMessageParts(count);
              return String.format(
                      "There %s %s %s%s",
                      verb, number, candidate, pluralModifier );
          }
          
          private void createPluralDependentMessageParts(int count) {
              if (count == 0) {
                  thereAreNoLetters();
              } else if (count == 1) {
                  thereIsOneLetter();
              } else {
                  thereAreManyLetters(count);
              }
          }
          
          private void thereAreManyLetters(int count) {
              number = Integer.toString(count);
              verb = "are";
              pluralModifier = "s";
          }
          
          private void thereIsOneLetter() {
              number = "1";
              verb = "is";
              pluralModifier = "";
          }
          
          private void thereAreNoLetters() {
              number = "no";
              verb = "are";
              pluralModifier = "s";
          }
        }
      ```
    
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

        ```java
          public void delete(Page page) {
              try {
                  deletePageAndAllReferences(page);
              }
              catch (Exception e) {
                  logError(e);
              }
          }
          private void deletePageAndAllReferences(Page page) throws Exception {
              deletePage(page);
              registry.deleteReference(page.name);
              configKeys.deleteKey(page.name.makeKey());
          }
          private void logError(Exception e) {
              logger.log(e.getMessage());
          }
        ```

      - 给出异常发生的环境说明
        
        应创建信息充分的错误信息，并和异常一起传递出去。在消息中，包括失败的操作和失败类型。如果你的应用程序有日志系统，传递足够的信息给catch块，并记录下来。
          
      - 依调用者需要定义异常类

        ```java
          // bad
          ACMEPort port = new ACMEPort(12);
          try {
              port.open();
          } catch(DeviceResponseException e) {
              reportPortError(e);
              logger.log("Device response exception",e);
          } catch(ATM1212UnlockedException e) {
              reportPortError(e);
              logger.log("Unlock exception",e);
          } catch(GMXError e) {
              reportPortError(e);
              logger.log("Device response exception",e);
          } finally {
              // .....
          }
        ```

        通过打包调用API，确保它返回通过用异常类型，从而简化代码：

        ```java
          // good
          LocalPort port = new LocalPort(12);
          try {
              port.open();
          } catch(PortDeviceFailure e) {
              reportError(e);
              logger.log(e.getMessage(),e);
          } finally {
              // .....
          }

          public class LocalPort{
            private ACMEPort innerPort;
            
            public LocalPort(int portNumber){
                innerPort = new ACMEPort(portNumber);
            }
            
            public open() {
              try {
                  innerPort.open();
              } catch(DeviceResponseException e) {
                      // 自定义的异常类
                  throw new PortDeviceFailure(e);
              } catch(ATM1212UnlockedException e) {
                  throw new PortDeviceFailure(e);
              } catch(GMXError e) {
                  throw new PortDeviceFailure(e);
              }
            }
          }
        ```

        将第三方API打包是个良好的实践手段。当你打包一个第三方API，你就降低了对它的依赖。

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

    - 短小

      ```java
        public static String testableHtml(PageData pageData, boolean includeSuiteSetup) throws Exception {
              WikiPage wikiPage = pageData.getWikiPage();
              StringBuffer buffer = new StringBuffer();
              if (pageData.hasAttribute("Test")) {
                  if (includeSuiteSetup) {
                      WikiPage suiteSetup =
                          PageCrawlerImpl.getInheritedPage(
                          SuiteResponder.SUITE_SETUP_NAME, wikiPage
                      );
                      if (suiteSetup != null) {
                          WikiPagePath pagePath =
                              suiteSetup.getPageCrawler().getFullPath(suiteSetup);
                          String pagePathName = PathParser.render(pagePath);
                          buffer.append("!include -setup .")
                              .append(pagePathName)
                              .append("\n");
                      }
                  }
                  WikiPage setup =
                  PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
                  if (setup != null) {
                      WikiPagePath setupPath =
                          wikiPage.getPageCrawler().getFullPath(setup);
                      String setupPathName = PathParser.render(setupPath);
                      buffer.append("!include -setup .")
                          .append(setupPathName)
                          .append("\n");
                  }
              }
              buffer.append(pageData.getContent());
              if (pageData.hasAttribute("Test")) {
                  WikiPage teardown =
                  PageCrawlerImpl.getInheritedPage("TearDown", wikiPage);
                  if (teardown != null) {
                      WikiPagePath tearDownPath =
                          wikiPage.getPageCrawler().getFullPath(teardown);
                      String tearDownPathName = PathParser.render(tearDownPath);
                      buffer.append("\n")
                          .append("!include -teardown .")
                          .append(tearDownPathName)
                          .append("\n");
                  }
                  if (includeSuiteSetup) {
                      WikiPage suiteTeardown =
                          PageCrawlerImpl.getInheritedPage(
                          SuiteResponder.SUITE_TEARDOWN_NAME,
                          wikiPage
                      );
                      if (suiteTeardown != null) {
                          WikiPagePath pagePath =
                              suiteTeardown.getPageCrawler().getFullPath (suiteTeardown);
                          String pagePathName = PathParser.render(pagePath);
                          buffer.append("!include -teardown .")
                              .append(pagePathName)
                              .append("\n");
                      }
                  }
              }
              pageData.setContent(buffer.toString());
              return pageData.getHtml();
        }
      ```

      这容易看懂吗？来改善一次：

      ```java
        public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
            boolean isTestPage = pageData.hasAttribute("Test");
            if (isTestPage) {
                WikiPage testPage = pageData.getWikiPage();
                StringBuffer newPageContent = new StringBuffer();
                includeSetupPages(testPage, newPageContent, isSuite);
                newPageContent.append(pageData.getContent());
                includeTeardownPages(testPage, newPageContent, isSuite);
                pageData.setContent(newPageContent.toString());
            }
            return pageData.getHtml();
        }
      ```

      这个好点了。但还是太长了，可以再改善一下：

      ```java
        public static String renderPageWithSetupsAndTeardowns(
          PageData pageData, boolean isSuite) throws Exception {
          if (isTestPage(pageData))
              includeSetupAndTeardownPages(pageData, isSuite);
          return pageData.getHtml();
        }
      ```
    
    - switch语句
      
      switch天生要做许多事。

      ```java
        public Money calculatePay(Employee e)
          throws InvalidEmployeeType {
              switch (e.type) {
              case COMMISSIONED:
                  return calculateCommissionedPay(e);
              case HOURLY:
                  return calculateHourlyPay(e);
              case SALARIED:
                  return calculateSalariedPay(e);
              default:
                  throw new InvalidEmployeeType(e.type);
          }
        }
      ```

      以上代码有四个问题：

      1. 太长，而且出现新雇员类型时，它会变更长
      2. 它做了很多事
      3. 它违背了单一权责原则（Single Responsibility Principle，SRP）
      4. 它违反了开放闭合原则（Open Closed Principle，OCP），每次添加新类型时，就要修改它。
      
      这个问题的解决方案，就是将switch埋到抽象工厂下，不让任何人看到。

      ```java
        public abstract class Employee {
          public abstract boolean isPayday();
          public abstract Money calculatePay();
          public abstract void deliverPay(Money pay);
        }
        -----------------
        public interface EmployeeFactory {
            public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
        }
        -----------------
        public class EmployeeFactoryImpl implements EmployeeFactory {
            public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
                switch (r.type) {
                case COMMISSIONED:
                    return new CommissionedEmployee(r) ;
                case HOURLY:
                    return new HourlyEmployee(r);
                case SALARIED:
                    return new SalariedEmploye(r);
                default:
                    throw new InvalidEmployeeType(r.type);
                }
            }
        }
      ```

    - 无副作用

      尽量不要在函数中，更改参数的值。

        ```java
          public class UserValidator {
            private Cryptographer cryptographer;
            public boolean checkPassword(String userName, String password) {
                User user = UserGateway.findByName(userName);
                if (user != User.NULL) {
                    String codedPhrase = user.getPhraseEncodedByPassword();
                    String phrase = cryptographer.decrypt(codedPhrase, password);
                    if ("Valid Password".equals(phrase)) {
                        Session.initialize();
                        return true;
                    }
                }
                return false;
            }
          }
        ```

        Session.initialize(); 就是这一行产生的副作用。

    - 分割指令与询问

      函数要么做什么事情，要么回答什么问题。不要混淆。

      ```java
        // 如果有这样子一个函数
        public boolean set(String attribute, String value);
        // 被这样子调用，是很让人疑惑的
        if (set("username", "unclebob"))...
        // 应该这样子写：
        if (attributeExists("username")) {
            setAttribute("username", "unclebob");
            ...
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

    - 数据抽象
      
      - 具象点

      ```java
        public class Point {
          public double x;
          public double y;
        }
      ```

      -  抽象点

        ```java
          public interface Point {
            double getX();
            double getY();
            void setCartesian(double x, double y);
            double getR();
            double getTheta();
            void setPolar(double r, double theta);
          }
        ```

        抽象点的好处在于，它不仅仅是在变量之上放了一个函数层那么简单。

        隐藏关乎抽象。

        也就是为了让用户不需要了解数据的实现就能操作数据本体。

  - ## 1.5 单元测试

    - 每个测试一个断言
  
      - 每个测试函数有且仅有一个断言语句，每个测试函数中只测试一个概念。 

    - 整洁的测试依赖于FIRST规则

      - fast: 测试代码应该能够快速运行，因为我们需要频繁运行它。

      - independent: 测试应该相互独立，某个测试不应该依赖上一个测试的结果，测试可以以任何顺序进行。

      - repeatable: 测试应可以在任何环境中通过。

      - self-validating: 测试应该有bool值输出，不应通过查看日志来确认测试结果，不应手工对比两个文本文件确认测试结果。

      - timely: 及时编写测试代码。单元测试应该在生产代码之前编写，否则生产代码会变得难以测试。

    - 整洁的测试

      三个要素：可读性，可读性和可读性。

      ```java
        public void testGetPageHieratchyAsXml() throws Exception {
            crawler.addPage(root, PathParser.parse("PageOne"));
            crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
            crawler.addPage(root, PathParser.parse("PageTwo"));
            request.setResource("root");
            request.addInput("type", "pages");
            Responder responder = new SerializedPageResponder();
            SimpleResponse response =
                (SimpleResponse) responder.makeResponse(
                                    new FitNesseContext(root), request);
            String xml = response.getContent();
            assertEquals("text/xml", response.getContentType());
            assertSubString("<name>PageOne</name>", xml);
            assertSubString("<name>PageTwo</name>", xml);
            assertSubString("<name>ChildOne</name>", xml);
        }

        public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception {
            WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
            crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
            crawler.addPage(root, PathParser.parse("PageTwo"));
            PageData data = pageOne.getData();
            WikiPageProperties properties = data.getProperties();
            WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
            symLinks.set("SymPage", "PageTwo");
            pageOne.commit(data);
            request.setResource("root");
            request.addInput("type", "pages");
            Responder responder = new SerializedPageResponder();
            SimpleResponse response =
                (SimpleResponse) responder.makeResponse(
                                        new FitNesseContext(root), request);
            String xml = response.getContent();
            assertEquals("text/xml", response.getContentType());
            assertSubString("<name>PageOne</name>", xml);
            assertSubString("<name>PageTwo</name>", xml);
            assertSubString("<name>ChildOne</name>", xml);
            assertNotSubString("SymPage", xml);
        }

        public void testGetDataAsHtml() throws Exception {
            crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");
            request.setResource("TestPageOne");
            request.addInput("type", "data");
            Responder responder = new SerializedPageResponder();
            SimpleResponse response =
                (SimpleResponse) responder.makeResponse(
                                    new FitNesseContext(root), request);
            String xml = response.getContent();
            assertEquals("text/xml", response.getContentType());
            assertSubString("test page", xml);
            assertSubString("<Test", xml);
        }
      ```

      重构后：

      ```java
        public void testGetPageHierarchyAsXml() throws Exception {
            makePages("PageOne", "PageOne.ChildOne", "PageTwo");
            submitRequest("root", "type:pages");
            assertResponseIsXML();
            assertResponseContains(
                "<name>PageOne</name>", "<name>PageTwo</name>"，"<name>ChildOne</name>"
            );
        }
        public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
            WikiPage page = makePage("PageOne");
            makePages("PageOne.ChildOne", "PageTwo");
            addLinkTo(page, "PageTwo", "SymPage");
            submitRequest("root", "type:pages");
            assertResponseIsXML();
            assertResponseContains(
                "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
            );
            assertResponseDoesNotContain("SymPage");
        }
        public void testGetDataAsXml() throws Exception {
            makePageWithContent("TestPageOne", "test page");
            submitRequest("TestPageOne", "type:data");
            assertResponseIsXML();
            assertResponseContains("test page", "<Test");
        }
      ```

      **重构后，这些测试呈现了构造-操作-检验模式。每个测试都该这样子。**

  - ## 1.6 类

    - 类应该短小

    - 为修改而组织

      - 类应当对扩展开放，对修改封闭（开放闭合原则）。
      
      - 在理想系统中，我们通过扩展系统而非修改现有代码来添加新特性。

  - ## 1.7 系统

    - 将系统的构造与使用分开

      - 依赖注入（DI），控制反转（IoC）是分离构造与使用的强大机制。

      - 有时应用程序也需要负责确定何时创建对象，我们可以使用抽象工厂模式让应用自行控制何时创建对象，但构造能力在工厂实现类里，与使用部分分开。

      - 将关注的方面分离开，是软件技艺中最古老也是最重要的设计技巧。
        然而大多数程序，都没有做分离处理，如下:

        ```java
          public Service getService() {
            if (service == null)
                service = new MyServiceImpl(...); // Good enough default for most cases?
            return service;
          }
        ```

        这段代码虽然是所谓的延时构造、赋值，有一些好处。但是我们也得到 MyServiceImpl 构造函数所需要的一切的硬编码依赖。




  - ## 1.8 并发编程
    
    - 防御并发代码问题的原则与技巧

      - 遵循单一职责原则。分离并发代码与非并发代码。
      
      - 限制临界区数量、限制对共享数据的访问。
      
      - 避免使用共享数据，使用对象的副本。
      
      - 线程尽可能地独立，不与其他线程共享数据。

        ```java
          public class X {
            private int lastIdUsed;
            public int getNextId() {
                return ++lastIdUsed;
            }
          }
        ```

        如果两个线程中共享这个实例对象，那么getNextId返回的结果就有可能不是我们想要的。


        
  - ## 1.9 日志

    - 学习log4j

      - 日志等级

        1. TRACE 在线调试

            默认情况下，即不打印到终端也不输出到文件。对程序的运行效率几乎不产生影响。

        1. DEBUG 终端查看、在线调试

            默认情况下，打印到终端输出，但是不会归档到日志文件。因此，一般用于开发者在程序当前启动窗口上，查看日志的流水信息。

        1. INFO 报告程序进度和状态信息

            一般这种情况都是一次性的，不会大量反复输出。
          
        1. WARNING 警告信息

            程序处理中遇到非法数据或者某种可能的错误。该错误是一过性的、可恢复的，不会影响程序的继续运行，程序仍处在正常状态。

        1. ERROR 状态错误

            该错误发生后程序仍然可以运行，但是极有可能运行在某种非正常的状态下，导致无法完成全部既定的功能。

        1. FATAL 致命错误

            表明程序遇到了致命的错误，必须马上终止运行。

        > Log4j建议只使用四个级别，优先级从高到低分别是 
          ERROR > WARN > INFO > DEBUG

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


      
# 4. 规范

  - ## 4.1 日志规范

    - 日志工具
      
      项目中仅使用抽象日志框架来处理日志，**不得**直接使用Log4J或LogBack，建议使用[Slf4J](http://www.slf4j.org/)。具体的日志框架实现，建议使用[LogBack](http://logback.qos.ch/)。

            
      在Maven项目的依赖中，如果间接依赖了不同的日志框架，需要exclude掉，或者它们仅能出现在test的scope中。

      以使用Slf4j和LogBack为例，如果有第三方库依赖了Apache Commons Logging或者Log4J框架，则可进行如下配置：

      1. 添加Slf4J对其他框架的支持
          ```xml
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>jcl-over-slf4j</artifactId>
                <version>1.7.5</version>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>log4j-over-slf4j</artifactId>
                <version>1.7.5</version>
            </dependency>
          ```
        
                
      2. 排除掉第三方库中的依赖，以Apache Commons Logging为例，在`<dependency>`中添加如下内容：

      ```xml
      <exclusions>
          <exclusion>
              <groupId>commons-logging</groupId>
              <artifactId>commons-logging</artifactId>
          </exclusion>
      </exclusions>
      ```


    - Logger

      在使用`Logger`时，需要用如下方式在类中定义Logger对象：

      ```java
          private static final Logger logger = LoggerFactory.getLogger(xxx.class);
      ```

      如果有确定的日志名，则可以直接指定日志名：

      ```java
          private static final Logger logger = LoggerFactory.getLogger("日志名");
      ```
            
      输出日志时，**不得**使用字符串拼接的方式，即：

      ```java
          logger.info("foo" + bar);
      ```

      需要使用以下方式，避免不必要的`toString`和字符串拼接操作：

      ```java
          logger.info("foo{}", bar);
      ```

      Slf4J的方法支持可变参数，可以添加任意数量的{}，且无需在输出日志前进行isXXXEnabled判断。

      如果使用Apache Commons Logging或者Log4J，需要在输出之前进行判断，例如：

      ```java
          if (logger.isInfoEnabled()) {
              logger.info("foo");
          }
      ```

      建议对`ERROR`和`WARN`级别以外的日志都增加此判断。

      为了保证异常信息能够输出完整的异常堆栈，必须使用支持Throwable参数的方法输出异常，同时选择`ERROR`级别输出：

      ```java
          logger.error("处理XXX业务时发生异常", e);
      ```

      使用`+`拼接描述和异常对象，这样会丢失异常堆栈，因此**严禁**使用以下方法：

      ```java
          logger.error("处理XXX业务时发生异常" + e);
      ```


    - 日志文件

        - 日志分为以下几类：

          1. 业务日志
          2. 错误日志，统一将`ERROR`级别的日志汇总到一个日志中
          3. 摘要日志，以`-digest`作为主文件名后缀
          4. 统计日志，以`-stat`作为主文件名后缀   

        
        - 日志文件可以有两种扩展名：

          1. xxx.log
          2. xxx.Nd.log，`N`表示天数，用来标识日志的保存天数


          **未标识保存天数的日志文件，运维可以按需要任意清除。**

        - 日志必须按照一定规则进行滚动，以避免单个日志文件过大，影响性能。

          日志可以按天或者按小时滚动，滚动后的日志文件后缀名结尾方式如下：

          1. 按天滚动，xxx.log.20131213
          2. 按小时滚动，xxx.log.20131213_12

          综上，一个按天滚动、需要保存7天的摘要日志完整文件名是这样的：

          ```
            xxx-digest.7d.log.20131213
          ```
          
    - 日志格式

        日志需要遵循一定的格式，方便人与机器阅读和分析。日志一般由以下几部分组成：

        1. 时间，一般精确到毫秒
        2. 日志级别，即`ERROR`、`WARN`、`INFO`、`DEBUG`、`TRACE`
        3. 唯一ID，一般是贯穿所有上下游系统的唯一ID，如果没有的话也可以是本系统内部的标识
        4. 线程，输出日志的线程名称
        5. 日志名称，一般是类名（可以做些限制，避免过长），也可以是特定的名称
        6. 具体内容

 

    - 业务日志

            
      业务日志的格式建议如下：

          时间 级别 [唯一ID][线程] 日志名 - 内容

      例如：

          2013-12-09 18:17:30,110 WARN [1049754451][http-nio-8081-exec-6] c.b.c.b.s.i.XXXServiceImpl - 请求XXX业务处理，传入的参数为空。


    - 摘要日志

            
      大多数摘要日志都是按类型输出到不同文件中的，因此文件就代表了含义，不需要日志名。格式建议如下：

          时间 级别 [唯一ID][线程] - 内容

      不同的类型，内容有所不同，一般摘要可以有以下几类：

      1. 数据层访问摘要，文件名一般为`xxx-dal-digest.log`
      2. 调用下游服务摘要，文件名一般为`xxx-sal-digest.log`
      3. 对上游提供服务摘要，文件名一般为`xxx-service-digest.log`
      4. Web页面访问摘要，文件名一般为`xxx-page-digest.log`

      一般的摘要日志内容中要包含以下内容：

      1. 记录的对象
      2. 结果，成功或失败
      3. 耗时
      4. 参数，用`,`分隔，可选
      5. 返回值，用`,`分隔，可选

      因此，完整的摘要日志格式会是这样的：

          时间 级别 [唯一ID][线程] - [(对象,结果,耗时)(参数)(返回值)]

      忽略的内容用`-`填充。

      如果摘要是由系统框架级统一输出的，选择同样的级别输出，则可以忽略“级别”一项。

      - 数据层访问摘要

        
        每次数据库操作建议都记录摘要，以便后续分析系统处理请求过程中的瓶颈，借业务判断数据库状态等等。

              2013-12-09 15:40:55,092 INFO  [1049754451][http-nio-8081-exec-6] - [(UserDao.findByNameAndType, Y, 10ms)(foo,bar)(-)]


      - 对上游提供服务摘要

                
        对上游提供服务的摘要中，最好能够记录下是哪个上游系统发起的请求，因此需要增加一项。

        以REST风格的服务为例，其实质相当于处理了一次HTTP请求，可以采用如下格式：

            时间 级别 [唯一ID][线程][上游系统] - [(HTTP方法,请求URL,对应处理方法,结果,总耗时,业务处理耗时,结果渲染耗时)(参数)(HTTP结果响应码,返回值)]

        例如：

            2013-12-09 15:40:55,578 INFO  [1049754451][http-nio-8081-exec-6][systemA] - [(POST,/user,UserController.find,Y,3ms,2ms,1ms)(foo,bar)(200,0)]

        RPC风格的服务，则可以使用如下格式：

            时间 级别 [唯一ID][线程][上游系统] - [(处理方法,结果,耗时)(参数)(返回值)]
  

    - 统计日志
            
      统计日志时对某些资源一段时间内使用情况的汇总，因此与具体的请求或线程无关，格式较为简单：

          时间 - [(统计项1)(统计项2)]

      例如对系统LOAD和内存的统计：

          2013-12-13 16:08:05,532 - [(2.15)(367,64400,0,0,2486)]


    - 日志内容
          
      本节主要描述业务日志的内容，一般根据业务内容进行定义，但需要遵循以下要求：

      1. 日志内容要完整，至少要能通过日志了解业务的情况。
      2. 日志文件需要合理的拆分，**不能**全部放置于一个文件中。
      3. 日志文件内容尽量不要重复输出，避免浪费存储空间。
      4. 日志要有合理的日志级别，生产环境一般只打开`INFO`及以上的日志级别。
          1. 正常的业务日志记录，使用`INFO`级别。
          2. 可接受的或者可预见的业务错误，使用`WARN`级别。
          3. 系统异常或者严重的业务错误，使用`ERROR`级别。
          4. 仅供调试使用的日志，使用`DEBUG`级别。

      出于安全角度的考虑，以下内容严禁**完整**出现在日志中，可以选择截取部分显示：

      1. 身份证号码
      2. 手机号码
      3. 银行卡号码

      关于信用卡，以下内容**严禁**出现在日志中：

      1. 信用卡有效期
      2. 信用卡CVV
