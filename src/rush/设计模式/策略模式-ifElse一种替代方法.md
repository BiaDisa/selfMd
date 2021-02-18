
# 策略模式

##### 简单策略模式的总结：

​	通过内外一致的参数，对一系列继承某个父类的方法进行区分和调用，这个区分筛选的过程可以使用map进行（钩子方法）。

##### 策略模式的本质：

1. 接口继承
2. 动态绑定s

## 笔记：



#### 1.阿里技术团队-初识

url : https://juejin.cn/post/6897011052601409549 



> 阿里开发规约-编程规约-控制语句-第六条 ：超过 3 层的 if-else 的逻辑判断代码可以使用卫语句、策略模式、状态模式等来实现。相信大家都见过这种代码：

```Java
if (conditionA) {
    逻辑1
} else if (conditionB) {
    逻辑2
} else if (conditionC) {
    逻辑3
} else {
    逻辑4
}

```

<center><b>代码1</b></center>

 

> 这种代码虽然写起来简单，但是很明显违反了面向对象的 2 个基本原则：
>
> - 单一职责原则（一个类应该只有一个发生变化的原因）：因为之后修改任何一个逻辑，当前类都会被修改
> - 开闭原则（对扩展开放，对修改关闭）：如果此时需要添加（删除）某个逻辑，那么不可避免的要修改原来的代码
>
> 因为违反了以上两个原则，尤其是当 if-else 块中的代码量比较大时，后续代码的扩展和维护就会逐渐变得非常困难且容易出错，使用卫语句也同样避免不了以上两个问题。因此根据我的经验，得出一个我个人认为比较好的实践：
>
> - if-else 不超过 2 层，块中代码 1~5 行，直接写到块中，否则封装为方法
> - if-else 超过 2 层，但块中的代码不超过 3 行，尽量使用卫语句<sup>【1】</sup>
> - if-else 超过 2 层，且块中代码超过 3 行，尽量使用策略模式

​			【1】：卫语句：如下述 中的：

> 

````````````
if(A == 0){}...else if(A == 1){}....
````````````

​					可以修改成

``````````````````java
if(){

}
doSomething();
``````````````````

​				简单地说，*优化后精简的if语句，把多层嵌套的if语句转化为单层的if语句* 的if语句，被称为卫语句。



##### 小tips

一个方便注入的技巧：

```````````java
private ApplicationContext appContext;

 @Override
    public void afterPropertiesSet() {
        // 将 Spring 容器中所有的 FormSubmitHandler 注册到 FORM_SUBMIT_HANDLER_MAP
        appContext.getBeansOfType(FormSubmitHandler.class)
                  .values()
                  .forEach(handler -> FORM_SUBMIT_HANDLER_MAP.put(handler.getSubmitType(), handler));
    }

    @Override
    public void setApplicationContext(@NonNull ApplicationContext applicationContext) {
        appContext = applicationContext;
    }

```````````

>我们让 FormSubmitHandlerFactory 实现 InitializingBean ,ApplicationContextAware接口，在 afterPropertiesSet 方法中，基于 Spring 容器将所有 FormSubmitHandler 自动注册到 FORM_SUBMIT_HANDLER_MAP，从而 Spring 容器启动完成后， getHandler 方法可以直接通过 submitType 来获取对应的表单提交处理器。



 #### 2.运用其他的特性，实现低代码的策略模式

url ： https://juejin.cn/post/6898151607444504584

思路：通过jdk8的lambda表达式中的：

- Predicate来替换if-else的判断句。
- Function/Consumer来替换if-else判断为true后执行的语句

后，

- 通过Pair（或其他一对一的映射关系对象），拼装Predicate和Function的方法。

- 通过List容器收集需要的Pair对象

来替换普通的策略模式中，大量需要通过继承来实现的类，降低编码量。

- 优点
  - 一个工厂类中可以通过简洁的P-F/C的方法，替代需要背后大量类实现的策略模式。
- 缺点
  - 如果Function/Consumer逻辑较为复杂，若不将执行逻辑抽出为方法，实际上因为lambda自带的括号等，看着会很杂乱。
  - 通过这个方式实现的虽然不用大量的类作为策略的具体实现，但也不能使用Spring的管理，需要手动注册策略实现。



## 一个简单的实现demo：

声明一个接口对



``````
if(){}else{}...
``````

中涉及到的，需要的分支方法进行抽象：

```java
public interface BaseStrategy {

    //返回本策略方法的类型用于区分,也可以用枚举做进一步的信息同步记录
    String getType();

    //参数的处理,参数也可以使用继承等方式进行拓展
    Object process(Request request);


}
```

2. 对方法进行录入：

``````````````java
public class InsertStrategy implements BaseStrategy {

    private final String type = "INSERT";

    @Override
    public String getType() {
        return type;
    }

    @Override
    public Object process(Request request) {
        System.out.println(type);
        return type;
    }
}

public class UpdateStrategy implements BaseStrategy {

    private final static String type = "Update";

    @Override
    public String getType() {
        return type;
    }

    @Override
    public Object process(Request request) {
        System.out.println(type);
        return type;
    }
}
``````````````

3. 通过工厂方法，对分散的方法进行规约与统一处理：

``````````java
@Component
public class StrategyFactory implements InitializingBean, ApplicationContextAware {

    private HashMap<String,BaseStrategy> strategyDic = new HashMap<>();

    private ApplicationContext appContext;

    @Override
    public void afterPropertiesSet() throws Exception {
        appContext.getBeansOfType(BaseStrategy.class)
                .values()
                .forEach(handler -> strategyDic.put(handler.getType(), handler));
    }

    @Override
    public void setApplicationContext(@NonNull ApplicationContext applicationContext) throws BeansException {
        appContext = applicationContext;
    }
    
    public Object process(Request request){
        BaseStrategy cur = strategyDic.get(request.getSubmitType());
        if(null == cur){
            return "暂无法支持的类型";
        }
        return cur.process(request);
    }
}
``````````

这部分中使用了上面文章中的技巧：

1. 通过继承ApplicationContextAware来注入appContext；
2. 通过继承InitializingBean来自动对策略字典进行初始化。这样的好处是在后续代码策略增加时不需要再考虑到这部分的代码修改了。



### 可以改进的地方：

1. 工厂方法需要new/注入来创建反直觉，是否可以在修改部分代码，不涉及核心变更（工厂方法写好后，在不需要其他方法的前提下不需要再修改该方法）的情况下，修改成静态工厂的调用方式？
2. 该方式需要通过参数来进行mapping取出对应的方法，因此在入参和策略类上都带着对应的type，有没有更优雅的实现方式？
   - answer：type是作为索引使用的，如果不使用索引，可以使用上述提到的Predicate+List进行判别，两者在效率上有差距。

