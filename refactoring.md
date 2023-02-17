# 2. 重构

  - [2.1 Extract Method](#21-extract-method)


## 2.1 Extract Method

  Problem:
  ```java
    void printOwing() {
      printBanner();

      // Print details.
      System.out.println("name: " + name);
      System.out.println("amount: " + getOutstanding());
    }
  ```

  Solution:
  ```java
    void printOwing() {
      printBanner();
      printDetails(getOutstanding());
    }

    void printDetails(double outstanding) {
      System.out.println("name: " + name);
      System.out.println("amount: " + outstanding);
    }
  ```

