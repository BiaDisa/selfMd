### 引子

@Value注入值时，只需要在变量前进行声明

`````````````java
    @Value("redis.env")
    private static String env;
`````````````

即可。但当注入的值为静态值时，用这种方法注入的值会是空的。需要对**非静态方法**进行标注后再在方法内进行绑定，同时必须让对应的方法被Spring托管才会生效：

````````````java
@Component
public class HttpUtils {
    ...............
	private static boolean isProxy;
    @Value("${isProxy}")
    public void setProxy(boolean proxy) {
        HttpUtils.isProxy = proxy;
    }
}
````````````

因此有如下问题：

1. @Value是如何工作的？
2. 有和代理模式一样的问题，是否和Spring中常用的方法一样，是通过Cglib/JVM的代理模式进行代理的？



### @Value

