
# Java 开发手册

  - [1. 命名风格](#1-命名风格)

  - [2. 常量定义](#2-常量定义)

  - [3. OOP规约](#3-oop规约)

  - [4. 日期时间](#4-日期时间)

  - [5. 集合处理](#5-集合处理)

  - [6. MySQL数据库](#6-mysql数据库)

  - [7. 单元测试](#7-单元测试)

  - [8. 异常处理](#8-异常处理)

  - [9. 日志规约](#9-日志规约)

  - [10. 并发处理](#10-并发处理)

  - [11. 控制语句](#11-控制语句)

  ## 1. 命名风格
    
  - 【强制】避免在子父类的成员变量之间、或者不同代码块的局部变量之间采用完全相同的命名，使可理
       解性降低。

       说明：子类、父类成员变量名相同，即使是 public 也是能够通过编译，而局部变量在同一方法内的不同代码块中同名
       也是合法的，但是要避免使用。对于非 setter / getter 的参数名称也要避免与成员变量名称相同。
       反例：

       ```java
        public class ConfusingName {
            protected int stock;
            protected String alibaba;

            // 非 setter/getter 的参数名称，不允许与本类成员变量同名
            public void access(String alibaba) {
                if (condition) {
                    final int money = 666;
                    // ...
                }
                for (int i = 0; i < 10; i++) {
                    // 在同一方法体中，不允许与其它代码块中的 money 命名相同
                    final int money = 15978;
                    // ...
                }
            }
        }

        class Son extends ConfusingName {
            // 不允许与父类的成员变量名称相同
            private int stock;
        }
       ```

  ## 2. 常量定义
  
  - 【强制】不允许任何魔法值（即未经预先定义的常量）直接出现在代码中。

      反例：

      ```java
        // 开发者 A 定义了缓存的 key。
        String key = "Id#taobao_" + tradeId;
        cache.put(key, value);
        // 开发者 B 使用缓存时直接复制少了下划线，即 key 是"Id#taobao" + tradeId，导致出现故障。
        String key = "Id#taobao" + tradeId;
        cache.get(key);
      ```

  - 【推荐】如果变量值仅在一个固定范围内变化用 enum 类型来定义。
          说明：如果存在名称之外的延伸属性应使用 enum 类型，下面正例中的数字就是延伸信息，表示一年中的第几个季节。
          正例：
        
      ```java
        public enum SeasonEnum {
          SPRING(1), SUMMER(2), AUTUMN(3), WINTER(4);
          private int seq;

          SeasonEnum(int seq) {
              this.seq = seq;
          }

          public int getSeq() {
              return seq;
          }
        }
      ```

  ## 3. OOP规约
  
  - 【强制】浮点数之间的等值判断，基本数据类型不能使用 == 进行比较，包装数据类型不能使用 equals
      进行判断。
      说明：浮点数采用“尾数+阶码”的编码方式，类似于科学计数法的“有效数字+指数”的表示方式。二进制无法精确表
      示大部分的十进制小数，具体原理参考《码出高效》。
      反例：

      ```java
        float a = 1.0F - 0.9F;
        float b = 0.9F - 0.8F;
        if (a == b) {
            // 预期进入此代码块，执行其它业务逻辑
            // 但事实上 a == b 的结果为 false
            System.out.println("a == b");
        }
        Float x = Float.valueOf(a);
        Float y = Float.valueOf(b);
        if (x.equals(y)) {
            // 预期进入此代码块，执行其它业务逻辑
            // 但事实上 equals 的结果为 false
            System.out.println("x.equals(y)");
        }
      ```

      正例：<br>
      (1)指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。

      ```java
        float a = 1.0F - 0.9F;
        float b = 0.9F - 0.8F;
        float diff = 1e-6F;
        if (Math.abs(a - b) < diff) {
            System.out.println("true");
        }
      ```

      (2)使用 BigDecimal 来定义值，再进行浮点数的运算操作。

      ```java
        BigDecimal a = new BigDecimal("1.0");
        BigDecimal b = new BigDecimal("0.9");
        BigDecimal c = new BigDecimal("0.8");
        BigDecimal x = a.subtract(b);
        BigDecimal y = b.subtract(c);
        if (x.compareTo(y) == 0) {
            System.out.println("true");
        }
      ```

  - 【推荐】使用索引访问用 String 的 split 方法得到的数组时，需做最后一个分隔符后有无内容的检查，
      否则会有抛 IndexOutOfBoundsException 的风险。

    ```java
      String str = "a,b,c,,";
      String[] ary = str.split(",");
      // 预期大于 3，结果等于 3
      System.out.println(ary.length);
    ```

  - 【推荐】当一个类有多个构造方法，或者多个同名方法，这些方法应该按顺序放置在一起，便于阅读。

    ```java
      public int method(int param);
      protected double method(int param1, int param2);
      private void method();
    ```

  - 【推荐】setter 方法中，参数名称与类成员变量名称一致，this.成员名=参数名。在 getter / setter 方
      法中，不要增加业务逻辑，增加排查问题的难度。<br>
      反例：

      ```java
        public Integer getData() {
            if (condition) {
                return this.data + 100;
            } else {
                return this.data - 100;
            }
        }
      ```

  - 【推荐】循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展。<br>

    反例：
    ```java
      String str = "start";
      for (int i = 0; i < 100; i++) {
          str = str + "hello";
      }
    ```

  ## 4. 日期时间
  
  - 【强制】禁止在程序中写死一年为 365 天，避免在公历闰年时出现日期转换错误或程序逻辑错误。

      ```java
        // 获取今年的天数
        int daysOfThisYear = LocalDate.now().lengthOfYear();
        System.out.println(daysOfThisYear);
        // 获取指定某年的天数
        LocalDate.of(2011, 1, 1).lengthOfYear();
      ```

      反例：

      ```java
        // 第一种情况：在闰年 366 天时，出现数组越界异常
        int[] dayArray = new int[365];
        // 第二种情况：一年有效期的会员制，2020 年 1 月 26 日注册，硬编码 365 返回的却是 2021 年 1 月 25 日
        Calendar calendar = Calendar.getInstance();
        calendar.set(2020, 1, 26);
        calendar.add(Calendar.DATE, 365);
      ```

  ## 5. 集合处理
  
   - 【强制】判断所有集合内部的元素是否为空，使用 isEmpty() 方法，而不是 size() == 0 的方式。
        说明：在某些集合中，前者的时间复杂度为 O(1)，而且可读性更好。

        ```java
          Map<String, Object> map = new HashMap<>(16);
          if (map.isEmpty()) {
              System.out.println("no element in this map.");
          }
        ```

  - 【强制】在使用 java.util.stream.Collectors 类的 toMap() 方法转为 Map 集合时，一定要使用参数类型
  为 BinaryOperator，参数名为 mergeFunction 的方法，否则当出现相同 key 时会抛出
  IllegalStateException 异常。
  说明：参数 mergeFunction 的作用是当出现 key 重复时，自定义对 value 的处理策略。
  正例：<br>
    ```java
      List<Pair<String, Double>> pairArrayList = new ArrayList<>(3);
      pairArrayList.add(new Pair<>("version", 12.10));
      pairArrayList.add(new Pair<>("version", 12.19));
      pairArrayList.add(new Pair<>("version", 6.28));
      // 生成的 map 集合中只有一个键值对：{version=6.28}
      Map<String, Double> map = pairArrayList.stream()
        .collect(Collectors.toMap(Pair::getKey, Pair::getValue, (v1, v2) -> v2));
    ```

    反例：<br>
    ```java
      String[] departments = new String[]{"RDC", "RDC", "KKB"};
      // 抛出 IllegalStateException 异常
      Map<Integer, String> map = Arrays.stream(departments)
        .collect(Collectors.toMap(String::hashCode, str -> str));
    ```

  - 【强制】在使用 java.util.stream.Collectors 类的 toMap() 方法转为 Map 集合时，一定要注意当 value
    为 null 时会抛 NPE 异常。<br>
    说明：在 java.util.HashMap 的 merge 方法里会进行如下的判断：<br>
    ```java
      if (value == null || remappingFunction == null)
        throw new NullPointerException();
    ```

    反例：<br>
    ```java
      List<Pair<String, Double>> pairArrayList = new ArrayList<>(2);
      pairArrayList.add(new Pair<>("version1", 8.3));
      pairArrayList.add(new Pair<>("version2", null));
      // 抛出 NullPointerException 异常
      Map<String, Double> map = pairArrayList.stream()
        .collect(Collectors.toMap(Pair::getKey, Pair::getValue, (v1, v2) -> v2));
    ```

  - 【强制】使用集合转数组的方法，必须使用集合的 toArray(T[] array)，传入的是类型完全一致、长度为
    0 的空数组。
    反例：直接使用 toArray 无参方法存在问题，此方法返回值只能是 Object[]类，若强转其它类型数组将出现
    ClassCastException 错误。<br>
    正例：<br>
    ```java
      List<String> list = new ArrayList<>(2);
      list.add("guan");
      list.add("bao");
      String[] array = list.toArray(new String[0]);
    ```
    说明：<br>
      使用 toArray 带参方法，数组空间大小的 length：<br>
        1）等于 0，动态创建与 size 相同的数组，性能最好。<br>
        2）大于 0 但小于 size，重新创建大小等于 size 的数组，增加 GC 负担。<br>
        3）等于 size，在高并发情况下，数组创建完成之后，size 正在变大的情况下，负面影响与 2 相同。<br>
        4）大于 size，空间浪费，且在 size 处插入 null 值，存在 NPE 隐患。

  - 【强制】使用工具类 Arrays.asList() 把数组转换成集合时，不能使用其修改集合相关的方法，它的 add
    / remove / clear 方法会抛出 UnsupportedOperationException 异常。
    说明：asList 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。Arrays.asList 体现的是适配器模式，只
    是转换接口，后台的数据仍是数组。

    ```java
      String[] str = new String[]{ "yang", "guan", "bao" };
      List list = Arrays.asList(str);
    ```
    第一种情况：list.add("yangguanbao"); 运行时异常。<br>
    第二种情况：str[0] = "change"; list 中的元素也会随之修改，反之亦然。

  - 【强制】在无泛型限制定义的集合赋值给泛型限制的集合时，在使用集合元素时，需要进行
    instanceof 判断，避免抛出 ClassCastException 异常。
    说明：毕竟泛型是在 JDK5 后才出现，考虑到向前兼容，编译器是允许非泛型集合与泛型集合互相赋值。
    反例：<br>
    ```java
      List<String> generics = null;
      List notGenerics = new ArrayList(10);
      notGenerics.add(new Object());
      notGenerics.add(new Integer(1));
      generics = notGenerics;
      // 此处抛出 ClassCastException 异常
      String string = generics.get(0);
    ```

  - 【强制】不要在 foreach 循环里进行元素的 remove / add 操作。remove 元素请使用 iterator 方式，
    如果并发操作，需要对 iterator 对象加锁。<br>
    正例：<br>
    ```java
      List<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String item = iterator.next();
            if (删除元素的条件) {
                iterator.remove();
            }
        }
    ```
    反例：<br>
    ```java
      for (String item : list) {
            if ("2".equals(item)) {
                list.remove(item);
            }
        }
    ```

  - 【推荐】高度注意 Map 类集合 K / V 能不能存储 null 值的情况，如下表格：<br>
        
    | 集合类  | Key | Value | Super | 说明 |
    |---------|----------|---------|----------------------------------------------------------|-|
    | Hashtable     |    不允许为 null    |    不允许为 null    | Dictionary | 线程安全 |
    | TreeMap    |    不允许为 null    |    允许为 null    | AbstractMap | 线程不安全 |
    | ConcurrentHashMap  |    不允许为 null    |    不允许为 null    | AbstractMap | 锁分段技术（JDK8:CAS） |
    | HashMap     |    允许为 null    |    允许为 null    | AbstractMap | 线程不安全 |

    反例：由于 HashMap 的干扰，很多人认为 ConcurrentHashMap 是可以置入 null 值，而事实上，存储 null 值时会抛
    出 NPE 异常。

  ## 6. MySQL数据库

  - 【参考】合适的字符存储长度，不但节约数据库表空间、节约索引存储，更重要的是提升检索速度。<br>
      正例：无符号值可以避免误存负数，且扩大了表示范围：
    
    | 对象 | 年龄区间 | 类型 | 字节 | 表示范围 |
    |:----:|:-------:|:---:|:----:|:--------|
    | 人     | 150 岁之内 | tinyint unsigned | 1 | 无符号值：0 到 255 |
    | 龟     | 数百岁 | smallint unsigned | 2 | 无符号值：0 到 65535 |
    | 恐龙化石     | 数千万年 | int unsigned | 4 | 无符号值：0 到约43亿 |
    | 太阳 | 约50亿年 | bigint unsigned | 8 | 无符号值：0 到约 10 的 19 次方 |

  - 【推荐】利用延迟关联或者子查询优化超多分页场景。
    说明：MySQL 并不是跳过 offset 行，而是取 offset+N 行，然后返回放弃前 offset 行，返回 N 行，那当 offset 特别大
    的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行 SQL 改写。<br>
    正例：先快速定位需要获取的 id 段，然后再关联：

    ```sql
      SELECT t1.* FROM 表 1 as t1 , (select id from 表 1 where 条件 LIMIT 100000 , 20) as t2 where t1.id = t2.id
    ```

  - 【推荐】建组合索引的时候，区分度最高的在最左边。
    正例：如果 where a = ? and b = ?，a 列的几乎接近于唯一值，那么只需要单建 idx_a 索引即可。
    说明：存在非等号和等号混合判断条件时，在建索引时，请把等号条件的列前置。如：where c > ? and d = ? 那么即使
    c 的区分度更高，也必须把 d 放在索引的最前列，即建立组合索引 idx_d_c。

  - 【强制】不要使用 count(列名) 或 count(常量) 来替代 count(*)，count(*) 是 SQL92 定义的标准统计行
    数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。
    说明：count(*) 会统计值为 NULL 的行，而 count(列名) 不会统计此列为 NULL 值的行。

  - 【强制】对于数据库中表记录的查询和变更，只要涉及多个表，都需要在列名前加表的别名（或表名）进
    行限定。
    说明：对多表进行查询记录、更新记录、删除记录时，如果对操作列没有限定表的别名（或表名），并且操作列在多个
    表中存在时，就会抛异常。
    正例：
    
    ```sql 
    select t1.name from first_table as t1 , second_table as t2 where t1.id = t2.id; 
    ```

    反例：在某业务中，由于多表关联查询语句没有加表的别名（或表名）的限制，正常运行两年后，最近在某个表中增加
    一个同名字段，在预发布环境做数据库变更后，线上查询语句出现出 1052 异常：
    Column 'name' infield list is ambiguous。

  - 【推荐】SQL 语句中表的别名前加 as，并且以 t1、t2、t3、...的顺序依次命名。
      说明：<br>
      1）别名可以是表的简称，或者是依照表在 SQL 语句中出现的顺序，以 t1、t2、t3 的方式命名。<br>
      2）别名前加 as 使别名更容易识别。<br>
      正例：
      ``` sql
        select t1.name from first_table as t1 , second_table as t2 where t1.id = t2.id;
      ```

  ## 7. 单元测试

  - 【推荐】对于数据库相关的查询，更新，删除等操作，不能假设数据库里的数据是存在的，或者直接操作数据库把数据插入进去，请使用程序插入或者导入数据的方式来准备数据。
  反例：删除某一行数据的单元测试，在数据库中，先直接手动增加一行作为删除目标，但是这一行新增数据并不符合业务插入规则，导致测试结果异常。

  - 【强制】单元测试是可以重复执行的，不能受到外界环境的影响。
  说明：单元测试通常会被放到持续集成中，每次有代码 push 时单元测试都会被执行。如果单测对外部环境（网络、服
  务、中间件等）有依赖，容易导致持续集成机制的不可用。
  正例：为了不受外界环境影响，要求设计代码时就把 SUT（Software under test）的依赖改成注入，在测试时用 Spring
  这样的 DI 框架注入一个本地（内存）实现或者 Mock 实现。

  - 【推荐】单测的基本目标：语句覆盖率达到 70%；核心模块的语句覆盖率和分支覆盖率都要达到 100%
  说明：在工程规约的应用分层中提到的 DAO 层，Manager 层，可重用度高的 Service，都应该进行单元测试。

  - 【推荐】编写单元测试代码遵守 BCDE 原则，以保证被测试模块的交付质量。<br>
  ⚫ B：Border，边界值测试，包括循环边界、特殊取值、特殊时间点、数据顺序等。<br>
  ⚫ C：Correct，正确的输入，并得到预期的结果。<br>
  ⚫ D：Design，与设计文档相结合，来编写单元测试。<br>
  ⚫ E：Error，强制错误信息输入（如：非法数据、异常流程、业务允许外等），并得到预期的结果。

  ## 8. 异常处理   

  - 【强制】Java 类库中定义的可以通过预检查方式规避的 RuntimeException 异常不应该通过 catch 的方
      式来处理，比如：NullPointerException，IndexOutOfBoundsException 等等。
      说明：无法通过预检查的异常除外，比如，在解析字符串形式的数字时，可能存在数字格式错误，不得不通过 catch
      NumberFormatException 来实现。

    正例：if (obj != null) {...}<br>
    反例：try { obj.method(); } catch (NullPointerException e) {…}

  - 强制】finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch。
    说明：如果 JDK7，可以使用 try-with-resources 方式。

  - 【强制】不要在 finally 块中使用 return
    说明：try 块中的 return 语句执行成功后，并不马上返回，而是继续执行 finally 块中的语句，如果此处存在 return 语句，
    则会在此直接返回，无情丢弃掉 try 块中的返回点。<br>
    反例：<br>
    ```java
      private int x = 0;
      public int checkReturn() {
        try {
          // x 等于 1，此处不返回
          return ++x;
        } finally {
          // 返回的结果是 2
          return ++x;
        }
      }
    ```

  ## 9. 日志规约

  - 【强制】应用中不可直接使用日志系统（Log4j、Logback）中的 API，而应依赖使用日志框架（SLF4J、
    JCL—Jakarta Commons Logging）中的 API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。
    说明：日志框架（SLF4J、JCL--Jakarta Commons Logging）的使用方式（推荐使用 SLF4J）

    使用 SLF4J：<br>
    ```java
      import org.slf4j.Logger;
      import org.slf4j.LoggerFactory;
      private static final Logger logger = LoggerFactory.getLogger(Test.class);
    ```
    
    使用 JCL：<br>
    ```java
      import org.apache.commons.logging.Log;
      import org.apache.commons.logging.LogFactory;
      private static final Log log = LogFactory.getLog(Test.class);
    ```
    
  - 【强制】日志文件至少保存 15 天，因为有些异常具备以“周”为频次发生的特点。对于当天日志，以
    “应用名.log”来保存，保存在/{统一目录}/{应用名}/logs/目录下，过往日志格式为：
    {logname}.log.{保存日期}，日期格式：yyyy-MM-dd
    正例：以 mppserver 应用为例，日志保存/home/admin/mppserver/logs/mppserver.log，历史日志名称
    为 mppserver.log.2021-11-28  

  - 【强制】在日志输出时，字符串变量之间的拼接使用占位符的方式。
  说明：因为 String 字符串的拼接会使用 StringBuilder 的 append() 方式，有一定的性能损耗。使用占位符仅是替换动作，可以有效提升性能。
  正例：<br>
    ```java
      logger.debug("Processing trade with id : {} and symbol : {}", id, symbol);
    ```
  
  - 【强制】对于 trace / debug / info 级别的日志输出，必须进行日志级别的开关判断：
  说明：虽然在 debug(参数) 的方法体内第一行代码 isDisabled(Level.DEBUG_INT) 为真时（Slf4j 的常见实现 Log4j 和
  Logback），就直接 return，但是参数可能会进行字符串拼接运算。此外，如果 debug(getName()) 这种参数内有
  getName() 方法调用，无谓浪费方法调用的开销。
  正例：<br>
    ```java
      // 如果判断为真，那么可以输出 trace 和 debug 级别的日志
      if (logger.isDebugEnabled()) {
        logger.debug("Current ID is: {} and name is: {}", id, getName());
      }
    ```

  - 【强制】异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么通过关键字
  throws 往上抛出。
  正例：<br>
    ```java
      logger.error("inputParams: {} and errorMessage: {}", 各类参数或者对象 toString(), e.getMessage(), e);
    ```

## 10. 并发处理

  - 【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。<br>
    正例：自定义线程工厂，并且根据外部特征进行分组，比如，来自同一机房的调用，把机房编号赋值给
    whatFeatureOfGroup：
    ```java
      public class UserThreadFactory implements ThreadFactory {
        private final String namePrefix;
        private final AtomicInteger nextId = new AtomicInteger(1);
        // 定义线程组名称，在利用 jstack 来排查问题时，非常有帮助
        UserThreadFactory(String whatFeatureOfGroup) {
          namePrefix = "FromUserThreadFactory's " + whatFeatureOfGroup + "-Worker-";
        }
        @Override
        public Thread newThread(Runnable task) {
          String name = namePrefix + nextId.getAndIncrement();
          Thread thread = new Thread(null, task, name, 0, false);
          System.out.println(thread.getName());
          return thread;
        }
      }
    ```
  
  - 【强制】在使用阻塞等待获取锁的方式中，必须在 try 代码块之外，并且在加锁方法与 try 代码块之间没
    有任何可能抛出异常的方法调用，避免加锁成功后，在 finally 中无法解锁。
    说明一：在 lock 方法与 try 代码块之间的方法调用抛出异常，无法解锁，造成其它线程无法成功获取锁。
    说明二：如果 lock 方法在 try 代码块之内，可能由于其它方法抛出异常，导致在 finally 代码块中，unlock 对未加锁的对
    象解锁，它会调用 AQS 的 tryRelease 方法（取决于具体实现类），抛出 IllegalMonitorStateException 异常。
    说明三：在 Lock 对象的 lock 方法实现中可能抛出 unchecked 异常，产生的后果与说明二相同。<br>
    正例： <br>
    ```java
      Lock lock = new XxxLock();
      // ...
      lock.lock();
      try {
        doSomething();
        doOthers();
      } finally {
        lock.unlock();
      }
    ```

    反例：
    ```java
      Lock lock = new XxxLock();
      // ...
      try {
        // 如果此处抛出异常，则直接执行 finally 代码块
        doSomething();
        // 无论加锁是否成功，finally 代码块都会执行
        lock.lock();
        doOthers();
      } finally {
        lock.unlock();
      }
    ```

  - 【强制】在使用尝试机制来获取锁的方式中，进入业务代码块之前，必须先判断当前线程是否持有锁。
    锁的释放规则与锁的阻塞等待方式相同。
    说明：Lock 对象的 unlock 方法在执行时，它会调用 AQS 的 tryRelease 方法（取决于具体实现类），如果当前线程不
    持有锁，则抛出 IllegalMonitorStateException 异常。
    正例：<br>
    ```java
      Lock lock = new XxxLock();
      // ...
      boolean isLocked = lock.tryLock();
      if (isLocked) {
        try {
          doSomething();
          doOthers();
        } finally {
          lock.unlock();
        }
      }
    ```

  - 【推荐】通过双重检查锁（double-checked locking），实现延迟初始化需要将目标属性声明为
    volatile 型，（比如修改 helper 的属性声明为 private volatile Helper helper = null;）。
    正例：
    ```java
      public class LazyInitDemo {
        private volatile Helper helper = null;
        public Helper getHelper() {
          if (helper == null) {
            synchronized(this) {
              if (helper == null) {
                helper = new Helper();
              }
            }
          }
          return helper;
        }
        // other methods and fields...
      }
    ```

  - 【参考】volatile 解决多线程内存不可见问题对于一写多读，是可以解决变量同步问题，但是如果多
    写，同样无法解决线程安全问题。
    说明：如果是 count++操作，使用如下类实现：
    AtomicInteger count = new AtomicInteger();
    count.addAndGet(1);
    如果是 JDK8，推荐使用 LongAdder 对象，比 AtomicLong 性能更好（减少乐观锁的重试次数）。

## 11. 控制语句

  - 【强制】当 switch 括号内的变量类型为 String 并且此变量为外部参数时，必须先进行 null 判断。<br>
    反例：如下的代码输出是什么？
    ```java
      public class SwitchString {
        public static void main(String[] args) {
            method(null);
        }

        public static void method(String param) {
            switch (param) {
                // 肯定不是进入这里
                case "sth":
                    System.out.println("it's sth");
                    break;
                // 也不是进入这里
                case "null":
                    System.out.println("it's null");
                    break;
                // 也不是进入这里
                default:
                    System.out.println("default");
            }
        }
    }
    ```
  
  - 【强制】三目运算符 condition ? 表达式 1：表达式 2 中，高度注意表达式 1 和 2 在类型对齐时，可能
    抛出因自动拆箱导致的 NPE 异常。
    说明：以下两种场景会触发类型对齐的拆箱操作：<br>
    1）表达式 1 或 表达式 2 的值只要有一个是原始类型。<br>
    2）表达式 1 或 表达式 2 的值的类型不一致，会强制拆箱升级成表示范围更大的那个类型。<br>
    反例：
    ```java
      Integer a = 1;
      Integer b = 2;
      Integer c = null;
      Boolean flag = false;
      // a*b 的结果是 int 类型，那么 c 会强制拆箱成 int 类型，抛出 NPE 异常
      Integer result = (flag ? a * b : c);
    ```

  - 【推荐】表达异常的分支时，少用 if-else 方式，这种方式可以改写成：
      ```java
        if (condition) {
          ...
          return obj;
        }
        // 接着写 else 的业务逻辑代码;
      ```
      说明：如果非使用 if()...else if()...else...方式表达逻辑，避免后续代码维护困难，请勿超过 3 层。<br>
      正例：超过 3 层的 if-else 的逻辑判断代码可以使用卫语句、策略模式、状态模式等来实现，其中卫语句示例如下：
      ```java
        public void findBoyfriend(Man man) {
          if (man.isUgly()) {
              System.out.println("本姑娘是外貌协会的资深会员");
              return;
          }
          if (man.isPoor()) {
              System.out.println("贫贱夫妻百事哀");
              return;
          }
          if (man.isBadTemper()) {
              System.out.println("银河有多远，你就给我滚多远");
              return;
          }
          System.out.println("可以先交往一段时间看看");
        }
      ```

  - 【推荐】避免采用取反逻辑运算符。<br>
      说明：取反逻辑不利于快速理解，并且取反逻辑写法一般都存在对应的正向逻辑写法。<br>
      正例：使用 if(x < 628) 来表达 x 小于 628。<br>
      反例：使用 if(!(x >= 628)) 来表达 x 小于 628。
