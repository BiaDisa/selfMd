- 在项目中原本预计增加一个新的feature：使用nio代替原有的Stream的方式进行下载，在其中使用了DirectBuffer。DirectBuffer与Buffer类相比，申请了堆外内存，**据说**在读写速度上更快。[参照](https://www.jianshu.com/p/007052ee3773)

- 但根据查阅资料，由于DirectBuffer的申请速度较慢，因此使用了全局静态变量进行初始化。部分代码如下：

  ````java
  public class HttpIOUtils {
      
     private static ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);
      
     ....
         
     public static File downloadInputStream(InputStream inputStream,String suffix) {
          File tmpSource;
          try {
              tmpSource = generateRandomTmpFile(suffix);
              ReadableByteChannel ins = Channels.newChannel(inputStream);
              FileChannel fos = new FileOutputStream(tmpSource,true).getChannel();
              while ((ins.read(directBuffer)) != -1) {
                  directBuffer.flip();
                  fos.write(directBuffer);
                  directBuffer.clear();
              }
              directBuffer.clear();
              inputStream.close();
              ins.close();
              fos.close();
              return tmpSource;
          } catch (IOException e) {
              e.printStackTrace();
          }
          return null;
      }
  ````

  



代码审查时，发现DirectBuffer是**虚引用**，并且根据上述代码发现了如下问题：



- DirectBuffer被ByteBuffer类型的变量引用，是哪种引用类型，还是虚引用吗？
- 如果仍然是虚引用，在静态方法中被全局变量引用的情况会不会被放入GC队列？
- 如果确实在任何时间节点都会被放入引用队列（*ReferenceQueue*），那么在重新分配时，使用哪种锁，如果不加锁，会发生什么情况？
- 因为是全局静态变量，这个DirectBuffer会不会存在并发的问题？（A、B线程一起进入静态方法，A线程中的变量写到B线程里了）
- 如果要做成全局并且池化，该如何去实现？
- 如果docker镜像分配的内存=jvm配置最大内存，那么堆外内存可以正常申请吗？
- 如果可以申请，堆内外内存读写差距大吗？



note-这部分其实和JVM垃圾回收机制有关。



**PS：可以使用netty封装好的IO做操作，不仅支持池化，而且效率更高。** [参考](https://www.juejin.im/post/6861560044690800648)



---------------------------------------------



#### 全局变量引用是否会被放入引用队列等待GC？

其实是会的。

根据(StackOverflow上的讨论)

---------------------------------------

#### 使用中的对象是否会被GC？

~~不可能。NT问题。~~



----------------------------------

#### 对象重分配时如果不加锁，在并发环境下是否能保证happens-before语义？

显然不行。

---------------------------------------------

#### JVM内存=物理内存时，堆外内存可以正常申请吗？

[参见这篇文章后半段](https://www.jianshu.com/p/007052ee3773)

> 

如果无法分配，会尝试最多gc9次，如果依然无法分配，便抛出OOM异常：

> OutOfMemoryError("Direct buffer memory”)

---------------------------------------------------

### 引用类型

参考资料：

[引用类型](https://www.cnblogs.com/liyutian/p/9690974.html)

[的详细介绍](https://blog.csdn.net/firebolt100/article/details/82660333)

>![RefType](C:\Users\86134\Desktop\md\note\pic\RefType.png)



上面的图概括如下：

#### StrongReference

我们发现在类包（**Refernce包**）中我们并没有发现 StrongReference 类型，原因是我们平时写的代码基本上都是 StrongReference 。我们最常的创建对象方式就是 new 一个对象，然后将其赋值给一个声明为这个对象的类型及其父类的引用。如果对象有一个 StrongReference ，那么这个对象将不会被gc回收。

##### 举例

```java
HelloWorld hello = new HelloWorld();
```

这里 hello 就是一个 HelloWorld 对象的 StrongReference。

#### SoftReference

如果一个对象没有 StrongReference 但存在一个 SoftReference ，那么 gc 将会在虚拟机需要释放一些内存的时候回收这个对象。可以通过对对象的 SoftReference 调用 get() 方法获取该对象。如果这个对象没有被 gc 回收，则返回此对象，否则返回 null 。

//todo-代码介绍

>创建方式：new SoftReference<Type>(valObject);

#### WeakReference

如果一个对象没有 StrongReference 但有存在一个 WeakReference ，那么 gc 将会在下一次运行时对其进行回收，哪怕虚拟机的内存还足够多。

//todo-代码介绍

>创建方式：new WeakReference<Type>(valObject);

#### PhantomReference 与 FinalReference

如果某个对象没有以上这些类型的引用，那么它可能有一个 PhantomReference 。PhantomReference 不能用于直接访问对象。调用 get() 方法都会返回 null 。

//todo-代码介绍

> 无法显式创建。





*//todo-验证*



####  ReferenceQueue

> ![RefernceQueue官方解释](C:\Users\86134\Desktop\md\note\pic\RefernceQueue官方解释.jpg)

加入到这个队列中的对象，在**适当的可达性改变被检测到**时，被系统GC。

//todo-介绍

--------------------------------------



### 池化实现



可以使用现成的组件**commons-pool2**



[java-commons-pool2](https://blog.csdn.net/u_ascend/article/details/80594306)



##### maven依赖：

``````````````````````
		<dependency>    

<groupId>org.apache.commons</groupId>    

<artifactId>commons-pool2</artifactId>  

<version>2.5.0</version> 

		</dependency>
``````````````````````



​		

##### 使用例：

- 1.[commons-pool2](https://www.cnblogs.com/lighten/p/7375611.html)

- 2.netty=pooled相关类

  ##### 池化相关概念

  - //todo
  - netty的池化技术

----------------------------------------------------------



##### 全局静态变量的多线程安全问题

##### sample:

上述代码中如下改动：

​			通过休眠随机数ns进行模拟上下文切换的动作。

``````````````````java
public static File downloadInputStreamVer2(InputStream inputStream,String suffix,boolean debug) {
        File tmpDest = null;
        try {

            tmpDest = generateRandomTmpFile(suffix);
            ReadableByteChannel ins = Channels.newChannel(inputStream);
            FileChannel fos = new FileOutputStream(tmpDest,true).getChannel();
            while ((ins.read(buffer)) != -1) {
                buffer.flip();
                if(debug){
                    Thread.sleep(new Random().nextInt(10));
                }
                fos.write(buffer);
                buffer.clear();
            }
            buffer.clear();
            inputStream.close();
            ins.close();
            fos.close();
            return tmpDest;
        } catch (IOException|InterruptedException e) {
            if(null != tmpDest && tmpDest.exists()){
                delayDeleteFileQueue.offer(tmpDest.getAbsolutePath());
                GlobalThreadPool.execute(HttpIOUtils::deleteFile);
            }
            e.printStackTrace();
        }
        return null;
    }
``````````````````

同时通过两个线程的调用，模拟上下文切换：

``````````````````````````JAVA
 public static void main(String[] args){
        GlobalThreadPool.execute(()->threadTestA());
        GlobalThreadPool.execute(()->threadTestB());
    }


    private static void threadTestA(){
        File in = new File(dir + "\\后台篇.pdf");
        System.out.println(in.getAbsolutePath());
        InputStream is = HttpIOUtils.getStreamFromFile(in);
        HttpIOUtils.downloadInputStreamVer2(is,"pdf",true);
    }

    private static void threadTestB(){
        File in = new File(dir + "\\bit.md");
        System.out.println(in.getAbsolutePath());
        InputStream is = HttpIOUtils.getStreamFromFile(in);
        HttpIOUtils.downloadInputStreamVer2(is,"md",true);
    }
``````````````````````````



原始文件大小（下图中的**后台篇.pdf**）：



![image-20200924144027870](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20200924144027870.png)



执行后出错，原因为：buffer为空，原因是在某次交叉读写顺序不同。

并且，buffer在读取后，需要调用如下函数：

`````````
buffer.flip();
`````````



才会使ByteBuffer从读转化为写，此时使用该buffer继续执行读操作时会报错。

说明buffer是没有类似内存屏障的隐式语义的，如果要以全局变量的形式调用，则需要在使用时额外上锁。

-----------------------------------



### 堆内外内存读写差距



使用**JMH**进行markbench的测试，依据[知乎上莫枢大神的回答](https://www.zhihu.com/question/58735131/answer/307771944)，简单的main方法循环效果并不好。




JMH在JDK9之前，需要导入依赖如下：

```````````````xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.23</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.23</version>
</dependency>

```````````````

类头加入：

`````````````````````java
@BenchmarkMode({Mode.Throughput})
@State(value = Scope.Thread)
@Timeout(time = 10,timeUnit = TimeUnit.MINUTES)
@Warmup(iterations = 3, time = 30,timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 3, time = 1,timeUnit = TimeUnit.MINUTES)
@OutputTimeUnit(TimeUnit.SECONDS)
@Threads(8)
`````````````````````

并为测试方法上注解：

````````````````````````
@Benchmark
````````````````````````



测试代码如下：

``````````````````````````java
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 1, time = 1)
@Measurement(iterations = 1, time = 2)
@Threads(4)
@Fork(1)
@Timeout(time = 10,timeUnit = TimeUnit.SECONDS)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
.............................
    
public class FileNIO {
    private static final String dir = "C:\\Users\\86134\\Desktop\\md\\note";

    public static void main(String[] args) throws RunnerException {
        System.gc();
        Options opt = new OptionsBuilder()
                .include(FileNIO.class.getSimpleName())
                .result("result.json")
                .threads(8)
                .forks(3)
                .resultFormat(ResultFormatType.JSON).build();
        new Runner(opt).run();
        System.gc();
    }

}
``````````````````````````

> 
>
> 基准测试过程中还发现了一个很有趣的现象，在正确关闭所有相关流以及对应对象后，上述的f.delete()即使调用成功了，也并不代表着第一时间删除了该文件，而是在一段时间后删除，因此在目标文件夹会有一大堆等删除的文件，如果线程忙没有成功删除，则该文件很有可能不会被删除掉。
>
> 

​	将buffer/byte数组大小设置为4096，根据[stackoverflow上关于NIO性能的讨论](https://stackoverflow.com/questions/34493320/how-does-buffer-size-affect-nio-channel-performance?r=SearchResults)，*在hadoop中*，

>1. 大于8kb（8*1024b）的buffer，会被分割成若干8kb的块，进行IO操作。
>2. 这样做的目的是，***This is to avoid jdk from creating many direct buffers as the size of buffer increases.***
>3. 这说明8kb以下的buffer**很可能**并不会被JDK底层自动分成若干块。

测试参数为：

```````````````````java
# VM options: -XX:MaxDirectMemorySize=500M -javaagent:C:\solidBackup\work-tools\idea\IntelliJ IDEA 2019.2.4\lib\idea_rt.jar=64061:C:\solidBackup\work-tools\idea\IntelliJ IDEA 2019.2.4\bin -Dfile.encoding=UTF-8
# Warmup: 3 iterations, 30 s each
# Measurement: 3 iterations, 1 min each
# Timeout: 10 min per iteration
# Threads: 8 threads, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: test.func.FileNIO.alloc
```````````````````



结果为：

![image-20200929183340042](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20200929183340042.png)

意外的是使用堆外分配的要略慢于直接在堆内分配的。

一种解释为:

> 



以及另外一篇关于**FileInputStream**的[讨论](https://stackoverflow.com/questions/236861/how-do-you-determine-the-ideal-buffer-size-when-using-fileinputstream)：





