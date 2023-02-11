
# Java 开发手册

  - [1. 命名风格](#1-命名风格)

  - [2. 常量定义](#2-常量定义)

  - [3. OOP规约](#3-oop规约)

  - [4. 日期时间](#4-日期时间)

  - [5. 集合处理](#5-集合处理)

  - [6. MySQL数据库](#6-mysql数据库)

  - [7. 单元测试](#7-单元测试)

  - [8. 异常处理](#8-异常处理)

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

