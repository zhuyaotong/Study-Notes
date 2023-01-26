
# 目录

- [1. Java 并发编程实战](#1-java-并发编程实战)
  
  - [1.1 可见性、原子性和有序性](#11-可见性原子性和有序性)

# 1. Java 并发编程实战

  并发编程可以总结为三个核心问题：分工、同步、互斥。
  所谓互斥，指的是同一时刻，只允许一个线程访问共享变量。

  并发编程全景图：

  <img src='https://static001.geekbang.org/resource/image/11/65/11e0c64618c04edba52619f41aaa3565.png' width=900 alt='并发编程全景图'> </img></div>

  ## 1.1 可见性、原子性和有序性

   - 互斥锁

      - Java 语言提供的锁技术：synchronized

          - 锁是一种通用的技术方案，Java 语言提供的 synchronized 关键字，就是锁的一种实现。synchronized 关键字可以用来修饰方法，也可以用来修饰代码块，它的使用示例基本上都是下面这个样子：

            ```java
              class X {
                // 修饰非静态方法
                synchronized void foo() {
                    // 临界区
                }

                // 修饰静态方法
                synchronized static void bar() {
                    // 临界区
                }

                // 修饰代码块
                Object obj = new Object();

                void baz() {
                    synchronized (obj) {
                        // 临界区
                    }
                }
              }
            ```
            
            对于上面的例子，synchronized 修饰静态方法相当于:

            ```java               
              class X {
                // 修饰静态方法
                synchronized(X.class) static void bar() {
                  // 临界区
                }
              }
            ```

            修饰非静态方法，相当于：

            ```java
              class X {
                // 修饰非静态方法
                synchronized(this) void foo() {
                  // 临界区
                }
              }
            ```
            
            用 synchronized 解决 count+=1 问题：

            ```java
              class SafeCalc {
                long value = 0L;
                long get() {
                  return value;
                }
                synchronized void addOne() {
                  value += 1;
                }
              }
            ```

            ```java
              class SafeCalc {
                long value = 0L;
                synchronized long get() {
                  return value;
                }
                synchronized void addOne() {
                  value += 1;
                }
              }
            ```

            <table align="center">
              <tr>
                <td align="center" style="width: 800px;">
                  <img src="https://static001.geekbang.org/resource/image/26/f6/26a84ffe2b4a6ae67c8093d29473e1f6.png" style="width: 1142px;"><br>
                  <sub>保护临界区 get() 和 addOne() 的示意图</sub>
                  <br>
                </td>
              </tr>
            </table>

          
      - 如何用一把锁保护多个资源？

        - 保护没有关联关系的多个资源

          相关的示例代码如下，账户类 Account 有两个成员变量，分别是账户余额 balance 和账户密码 password。取款 withdraw() 和查看余额 getBalance() 操作会访问账户余额 balance，我们创建一个 final 对象 balLock 作为锁（类比球赛门票）；而更改密码 updatePassword() 和查看密码 getPassword() 操作会修改账户密码 password，我们创建一个 final 对象 pwLock 作为锁（类比电影票）。不同的资源用不同的锁保护，各自管各自的，很简单。

          ```java
            class Account {
            // 锁：保护账户余额
            private final Object balLock
                    = new Object();
            // 账户余额  
            private Integer balance;
            // 锁：保护账户密码
            private final Object pwLock
                    = new Object();
            // 账户密码
            private String password;

            // 取款
            void withdraw(Integer amt) {
                synchronized (balLock) {
                    if (this.balance > amt) {
                        this.balance -= amt;
                    }
                }
            }

            // 查看余额
            Integer getBalance() {
                synchronized (balLock) {
                    return balance;
                }
            }

            // 更改密码
            void updatePassword(String pw) {
                synchronized (pwLock) {
                    this.password = pw;
                }
            }

            // 查看密码
            String getPassword() {
                synchronized (pwLock) {
                    return password;
                }
            }
          }
          ```

          当然，我们也可以用一把互斥锁来保护多个资源，例如我们可以用 this 这一把锁来管理账户类里所有的资源：账户余额和用户密码。具体实现很简单，示例程序中所有的方法都增加同步关键字 synchronized 就可以了，这里我就不一一展示了。

          但是用一把锁有个问题，就是性能太差，会导致取款、查看余额、修改密码、查看密码这四个操作都是串行的。而我们用两把锁，取款和修改密码是可以并行的。用不同的锁对受保护资源进行精细化管理，能够提升性能。这种锁还有个名字，叫细粒度锁。

        - 保护有关联关系的多个资源

          我们声明了个账户类：Account，该类有一个成员变量余额：balance，还有一个用于转账的方法：transfer()，然后怎么保证转账操作 transfer() 没有并发问题呢？

          ```java
            class Account {
              private int balance;
              // 转账
              void transfer(
                  Account target, int amt){
                if (this.balance > amt) {
                  this.balance -= amt;
                  target.balance += amt;
                }
              } 
            }
          ```

          相信你的直觉会告诉你这样的解决方案：用户 synchronized 关键字修饰一下 transfer() 方法就可以了，于是你很快就完成了相关的代码，如下所示。

          ```java 
            class Account {
              private int balance;
              // 转账
              synchronized void transfer(
                  Account target, int amt){
                if (this.balance > amt) {
                  this.balance -= amt;
                  target.balance += amt;
                }
              } 
            }
          ```

          在这段代码中，临界区内有两个资源，分别是转出账户的余额 this.balance 和转入账户的余额 target.balance，并且用的是一把锁 this，符合我们前面提到的，多个资源可以用一把锁来保护，这看上去完全正确呀。真的是这样吗？可惜，这个方案仅仅是看似正确，为什么呢？
          
          问题就出在 this 这把锁上，this 这把锁可以保护自己的余额 this.balance，却保护不了别人的余额 target.balance，就像你不能用自家的锁来保护别人家的资产，也不能用自己的票来保护别人的座位一样。

          
          <table align="center">
            <tr>
              <td align="center" style="width: 800px;">
                <img src="https://static001.geekbang.org/resource/image/1b/d8/1ba92a09d1a55a6a1636318f30c155d8.png?wh=1142*640" style="width: 1142px;"><br>
                <sub>用锁 this 保护 this.balance 和 target.balance 的示意图</sub>
                <br>
              </td>
            </tr>
          </table>


      - 使用锁的正确姿势

        示例代码如下，我们把 Account 默认构造函数变为 private，同时增加一个带 Object lock 参数的构造函数，创建 Account 对象时，传入相同的 lock，这样所有的 Account 对象都会共享这个 lock 了。

        ```java
          class Account {
            private Object lock;
            private int balance;

            private Account() {
            }

            // 创建Account时传入同一个lock对象
            public Account(Object lock) {
                this.lock = lock;
            }

            // 转账
            void transfer(Account target, int amt) {
                // 此处检查所有对象共享的锁
                synchronized (lock) {
                    if (this.balance > amt) {
                        this.balance -= amt;
                        target.balance += amt;
                    }
                }
            }
          }
        ```

        这个办法确实能解决问题，但是有点小瑕疵，它要求在创建 Account 对象的时候必须传入同一个对象，如果创建 Account 对象时，传入的 lock 不是同一个对象，那可就惨了，会出现锁自家门来保护他家资产的荒唐事。在真实的项目场景中，创建 Account 对象的代码很可能分散在多个工程中，传入共享的 lock 真的很难。

        所以，上面的方案缺乏实践的可行性，我们需要更好的方案。还真有，就是用 Account.class 作为共享的锁。Account.class 是所有 Account 对象共享的，而且这个对象是 Java 虚拟机在加载 Account 类的时候创建的，所以我们不用担心它的唯一性。使用 Account.class 作为共享的锁，我们就无需在创建 Account 对象时传入了，代码更简单。

        ```java
          class Account {
            private int balance;

            // 转账
            void transfer(Account target, int amt) {
                synchronized (Account.class) {
                    if (this.balance > amt) {
                        this.balance -= amt;
                        target.balance += amt;
                    }
                }
            }
          }
        ```

        下面这幅图很直观地展示了我们是如何使用共享的锁 Account.class 来保护不同对象的临界区的。

        <table>
          <tr>
            <td align="center" style="width: 800px;">
              <img src="https://static001.geekbang.org/resource/image/52/7c/527cd65f747abac3f23390663748da7c.png" style="width: 1142px;"><br>
              <br>
            </td>
          </tr>
        </table>

        总结:
          相信你看完这篇文章后，对如何保护多个资源已经很有心得了，关键是要分析多个资源之间的关系。如果资源之间没有关系，很好处理，每个资源一把锁就可以了。如果资源之间有关联关系，就要选择一个粒度更大的锁，这个锁应该能够覆盖所有相关的资源。除此之外，还要梳理出有哪些访问路径，所有的访问路径都要设置合适的锁，这个过程可以类比一下门票管理。

          

