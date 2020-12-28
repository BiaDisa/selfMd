*note from [Java四种引用类型](https://www.cnblogs.com/liyutian/p/9690974.html)*



JDK1.2后，JAVA有4种引用类型：

- **强引用（Strong Reference）**
- **软引用（Soft Reference）**
- **弱引用（Weak Reference）**
- **虚引用（Phantom Reference）**



## 强引用



Java中默认声明的就是强引用，比如：

```
Object obj = new Object(); //只要obj还指向Object对象，Object对象就不会被回收
obj = null;  //手动置null
```

只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足时，JVM也会直接抛出OutOfMemoryError，不会去回收。

如果想中断强引用与对象之间的联系，可以显示的将强引用赋值为null，这样一来，JVM就可以适时的回收对象了



## 软引用

软引用是用来描述一些非必需但仍有用的对象。**在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常**。这种特性常常被用来实现缓存技术，比如网页缓存，图片缓存等。
在 JDK1.2 之后，用java.lang.ref.SoftReference类来表示软引用。

下面以一个例子来进一步说明强引用和软引用的区别：

在运行下面的Java代码之前，需要先配置参数 -Xms2M -Xmx3M，将 JVM 的初始内存设为2M，最大可用内存为 3M。

首先先来测试一下强引用，在限制了 JVM 内存的前提下，下面的代码运行正常

```
public class TestOOM {
	
	public static void main(String[] args) {
		 testStrongReference();
	}
	private static void testStrongReference() {
		// 当 new byte为 1M 时，程序运行正常
		byte[] buff = new byte[1024 * 1024 * 1];
	}
}
```

但是如果我们将

```
byte[] buff = new byte[1024 * 1024 * 1];
```

替换为创建一个大小为 2M 的字节数组

```
byte[] buff = new byte[1024 * 1024 * 2];
```

则内存不够使用，程序直接报错，强引用并不会被回收
![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194052676-1646914311.png)

接着来看一下软引用会有什么不一样，在下面的示例中连续创建了 10 个大小为 1M 的字节数组，并赋值给了软引用，然后循环遍历将这些对象打印出来。

```
public class TestOOM {
	private static List<Object> list = new ArrayList<>();
	public static void main(String[] args) {
	     testSoftReference();
	}
	private static void testSoftReference() {
		for (int i = 0; i < 10; i++) {
			byte[] buff = new byte[1024 * 1024];
			SoftReference<byte[]> sr = new SoftReference<>(buff);
			list.add(sr);
		}
		
		System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((SoftReference) list.get(i)).get();
			System.out.println(obj);
		}
		
	}
	
}
```

打印结果：
![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194016719-117632363.png)

我们发现无论循环创建多少个软引用对象，打印结果总是只有最后一个对象被保留，其他的obj全都被置空回收了。
这里就说明了在内存不足的情况下，软引用将会被自动回收。
值得注意的一点 , 即使有 byte[] buff 引用指向对象, 且 buff 是一个strong reference, 但是 SoftReference sr 指向的对象仍然被回收了，这是因为Java的编译器发现了在之后的代码中, buff 已经没有被使用了, 所以自动进行了优化。
如果我们将上面示例稍微修改一下：

```
	private static void testSoftReference() {
		byte[] buff = null;

		for (int i = 0; i < 10; i++) {
			buff = new byte[1024 * 1024];
			SoftReference<byte[]> sr = new SoftReference<>(buff);
			list.add(sr);
		}

        System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((SoftReference) list.get(i)).get();
			System.out.println(obj);
		}

		System.out.println("buff: " + buff.toString());
	}
```

则 buff 会因为强引用的存在，而无法被垃圾回收，从而抛出OOM的错误。
![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194030314-105853688.png)

如果一个对象惟一剩下的引用是软引用，那么该对象是软可及的（softly reachable）。垃圾收集器并不像其收集弱可及的对象一样尽量地收集软可及的对象，相反，它只在**真正 “需要” 内存时才收集软可及的对象**。



## 弱引用

弱引用的引用强度比软引用要更弱一些，**无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收**。在 JDK1.2 之后，用 java.lang.ref.WeakReference 来表示弱引用。
我们以与软引用同样的方式来测试一下弱引用：

```
	private static void testWeakReference() {
		for (int i = 0; i < 10; i++) {
			byte[] buff = new byte[1024 * 1024];
			WeakReference<byte[]> sr = new WeakReference<>(buff);
			list.add(sr);
		}
		
		System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((WeakReference) list.get(i)).get();
			System.out.println(obj);
		}
	}
```

打印结果：
![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194112309-477100844.png)

可以发现所有被弱引用关联的对象都被垃圾回收了。



## 虚引用

虚引用是最弱的一种引用关系，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它随时可能会被回收，在 JDK1.2 之后，用 PhantomReference 类来表示，通过查看这个类的源码，发现它只有一个构造函数和一个 get() 方法，而且它的 get() 方法仅仅是返回一个null，也就是说将永远无法通过虚引用来获取对象，虚引用必须要和 ReferenceQueue 引用队列一起使用。

```
public class PhantomReference<T> extends Reference<T> {
    /**
     * Returns this reference object's referent.  Because the referent of a
     * phantom reference is always inaccessible, this method always returns
     * <code>null</code>.
     *
     * @return  <code>null</code>
     */
    public T get() {
        return null;
    }
    public PhantomReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }
}
```

## 引用队列（ReferenceQueue）

引用队列可以与软引用、弱引用以及虚引用一起配合使用，当垃圾回收器准备回收一个对象时，如果发现它还有引用，那么就会在回收对象之前，把这个引用加入到与之关联的引用队列中去。程序可以通过判断引用队列中是否已经加入了引用，来判断被引用的对象是否将要被垃圾回收，这样就可以在对象被回收之前采取一些必要的措施。

与软引用、弱引用不同，虚引用必须和引用队列一起使用。