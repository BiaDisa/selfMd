### 静态内部类是共享的吗？

- 静态内部类的**非静态变量**在每个父对象中都有一个独立副本，生命周期内是**隔离的**。
- 静态内部类的**静态变量**是共享的。

```java
public class StaticInnerClassTest {

    public void init(){
        InnerStaticTest.setI(10);
    }

    public void print(){
        System.out.println(InnerStaticTest.getI());
    }

    @Data
    private static class InnerStaticTest{
        private static int i ;

        public static int getI() {
            return i;
        }

        public static void setI(int i) {
            InnerStaticTest.i = i;
        }
    }

    public static void main(String[] args) {
        StaticInnerClassTest test1 = new StaticInnerClassTest();
        test1.init();
        test1.print();
        StaticInnerClassTest test2 = new StaticInnerClassTest();
        test2.print();
    }
}
```

console输出：10 10