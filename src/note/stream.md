### stream

stream类包结构

![1593587595795](C:\Users\86134\Desktop\md\note\pic\1593587595795.png)





#### stream若干组件

consumer:

supplier:





#### stream类组

​		常用的stream对象，基本都是来自list.stream方法创建的。stream类在最终的实现上，都是以pipeLine的形式进行的。<stackTrace0

##### 	baseStream



#### pipeline

##### AbstractPipeline

在实现形式上类似链表，有一个**指向链表头部**的引用，以及一个**prev**引用，以及一个指向**下一个pipeline**的引用。

​	![1593589508081](C:\Users\86134\Desktop\md\note\pic\1593589508081.png)

组件中有一个spliterator。

![1593589675332](C:\Users\86134\Desktop\md\note\pic\1593589675332.png)

<stackTrace1

#### Spliterator

用于遍历与分割对象。



## Collector

5个组件：

![1594292367155](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1594292367155.png)



## Ops

#### 1.DistinctOps

​	返回一个**ReferencePipeline**对象，并不进行实质操作。

​	该**ReferencePipeline**对象中定义了若干方法。

![1594623089774](C:\Users\86134\Desktop\md\note\pic\distinctOps.png)







## 具体流程

stream中实际为pipeLine

