
# 内容

  - 非ASCII字符

    > Tip: 在使用Unicode转义符或是一些实际的Unicode字符时，建议做些注释给出解释，这有助于别人阅读和理解。

    例如：

    ```java
      String unitAbbrev = "μs";                                 | 赞，即使没有注释也非常清晰
      String unitAbbrev = "\u03bcs"; // "μs"                    | 允许，但没有理由要这样做
      String unitAbbrev = "\u03bcs"; // Greek letter mu, "s"    | 允许，但这样做显得笨拙还容易出错
      String unitAbbrev = "\u03bcs";                            | 很糟，读者根本看不出这是什么
      return '\ufeff' + content; // byte order mark             | Good，对于非打印字符，使用转义，并在必要时写上注释
    ```

  - switch语句

    在一个switch块内，每个语句组要么通过break, continue, return或抛出异常来终止，要么通过一条注释来说明程序将继续执行到下一个语句组， 任何能表达这个意思的注释都是OK的(典型的是用// fall through)。这个特殊的注释并不需要在最后一个语句组(一般是default)中出现。示例：

    ```java
      switch (input) {
        case 1:
        case 2:
          prepareOneOrTwo();
          // fall through
        case 3:
          handleOneTwoOrThree();
          break;
        default:
          handleLargeNumber(input);
      }
    ```

    default的情况要写出来，每个switch语句都包含一个default语句组，即使它什么代码也不包含。

  - 常量名

    常量名命名模式为CONSTANT_CASE，全部字母大写，用下划线分隔单词。那，到底什么算是一个常量？

    每个常量都是一个静态final字段，但不是所有静态final字段都是常量。在决定一个字段是否是一个常量时， 考虑它是否真的感觉像是一个常量。例如，如果任何一个该实例的观测状态是可变的，则它几乎肯定不会是一个常量。 只是永远不打算改变对象一般是不够的，它要真的一直不变才能将它示为常量。

    ```java
      // Constants
      static final int NUMBER = 5;
      static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
      static final Joiner COMMA_JOINER = Joiner.on(',');  // because Joiner is immutable
      static final SomeMutableType[] EMPTY_ARRAY = {};
      enum SomeEnum { ENUM_CONSTANT }

      // Not constants
      static String nonFinal = "non-final";
      final String nonStatic = "non-static";
      static final Set<String> mutableCollection = new HashSet<String>();
      static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
      static final Logger logger = Logger.getLogger(MyClass.getName());
      static final String[] nonEmptyArray = {"these", "can", "change"};
    ```

  - 捕获的异常：不能忽视

    除了下面的例子，对捕获的异常不做响应是极少正确的。(典型的响应方式是打印日志，或者如果它被认为是不可能的，则把它当作一个AssertionError重新抛出。)

    如果它确实是不需要在catch块中做任何响应，需要做注释加以说明(如下面的例子)。

    ```java
      try {
        int i = Integer.parseInt(response);
        return handleNumericResponse(i);
      } catch (NumberFormatException ok) {
        // it's not numeric; that's fine, just continue
      }
      return handleTextResponse(response);
    ```

    例外：在测试中，如果一个捕获的异常被命名为expected，则它可以被不加注释地忽略。下面是一种非常常见的情形，用以确保所测试的方法会抛出一个期望中的异常， 因此在这里就没有必要加注释。

    ```java
      try {
        emptyStack.pop();
        fail();
      } catch (NoSuchElementException expected) {
      }
    ```

  - 静态成员：使用类进行调用

    使用类名调用静态的类成员，而不是具体某个对象或表达式。

    ```java
      Foo aFoo = ...;
      Foo.aStaticMethod(); // good
      aFoo.aStaticMethod(); // bad
      somethingThatYieldsAFoo().aStaticMethod(); // very bad
    ```


