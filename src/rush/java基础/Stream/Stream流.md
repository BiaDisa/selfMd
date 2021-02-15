## Stream API

base on JDK1.8





### 常见开启stream的方法

- Collection类族：

  - stream()/parallelStream() 
    - 调用的方法为：

		> ​	StreamSupport.stream(spliterator(), true/false)
		
		​	通过streamSupport开启一个流。
	
- 数组类：

  - Arrays.stream()方法
    - 调用的方法为内部重载的若干方法。



### Stream方法的工作结构

- 最上层结构：

![image-20210215111649653](C:\Users\BiaDisa\AppData\Roaming\Typora\typora-user-images\image-20210215111649653.png)

- 通过类中的方法，可以得知：stream是在Pipeline和Stream两套继承体系下去实现的。



- BaseStream定义了：
  - 迭代器（iterator()）与分离器（Spliterator()）
  - 流的一些执行形式变换（顺序seq，并行par，无序unordered)
  - 以及创建一个可关闭的流（onClose)。通过继承的autoClosable的close方法，可以将通过onClose创建的流关闭。
  - 没有提供具体的方法实现。
  - 相关类：Iterator，Spliterator
- stream中定义了： 具体执行的常用的方法，例如filter，foreach等。
  - 方法介绍-todo
  - 相关类：
    - Predicate
    - Function
    - Comparator
    - Consumer
    - Optional
    - BinaryOperator
    - Supplier
    - 。。。



- PipelineHelper
  - 根据注解，这个类的设计目的：通过这个类的API能够获取到streamOp的所有相关信息。
  - 相关类：
    - StreamShape，描述Stream的类型：double，long，int，ref；
    - Sink



- AbstractPipeline

  - 根据文档所述:

    - 这个类是Stream的核心与首要实现方式：

    - >```
      >* Abstract base class for "pipeline" classes, which are the core
      >* implementations of the Stream interface and its primitive specializations.
      >* Manages construction and evaluation of stream pipelines.
      >```

      

    - 其他类型强相关的方法与实现类，都是在该抽象类的基础上进行了类型相关的操作。

      

    - 当后续接上新的中间操作，或者已经执行了终止操作，流便已经被视为已消费，不能继续进行其他的中间操作或者终止操作了。

    - >```
      >After chaining a new intermediate operation, or executing a terminal
      >* operation, the stream is considered to be consumed, and no more intermediate
      >* or terminal operations are permitted on this stream instance.
      >```
    
    
    
    - 执行顺序以及并行/串行流的执行过程：
    
    - >```
      >* For sequential streams, and parallel streams without
      >* stateful intermediate operations, parallel streams, pipeline evaluation 
      >* is done in a single
      >* pass that "jams" all the operations together.  For parallel streams with
      >* stateful operations, execution is divided into segments, where each
      >* stateful operations marks the end of a segment, and each segment is
      >* evaluated separately and the result used as the input to the next
      >* segment.  In all cases, the source data is not consumed until a terminal
      >* operation begins.
      >```
    
    - 串行流在遇到终止操作，并行流在遇到有状态的中间操作前，数据源是不会被消费的。
    
    - 相关类：
    
      - Node
    
    - 类方法描述-todo
    
      



#### 几个重要概念

- 



#### BaseStream提供的方法



