## 你可能不知道的Java Integer的缓存策略

本文将介绍 Java 中 Integer 缓存的相关知识。这是 Java 5 中引入的一个有助于节省内存、提高性能的特性。首先看一个使用 Integer 的示例代码，展示了 Integer 的缓存行为。接着我们将学习这种实现的原因和目的。你可以先猜猜下面 Java 程序的输出结果。很明显，这里有一些小陷阱，这也是我们写这篇文章的原因。

	public class JavaIntegerCache {
		public static void main(String[] args) {
			Integer integer1=3;
			Integer integer2=3;

			if(integer1==integer2){
				System.out.println("integer1==integer2");
			}else{
				System.out.println("integer1!=integer2");
			}<br>
			Integer integer3=300;
			Integer integer4=300;

			if(integer3==integer4){
				System.out.println("integer3==integer4");
			}else{
				System.out.println("integer3!=integer4");
			}
		}
	}
 大多数人都认为上面的两个判断的结果都是 false。虽然它们的值相等，但由于比较的是对象，而对象的引用不一样，所以会认为两个 if 判断都是 false 的。在 Java 中，== 比较的是对象引用，而 equals 比较的是值。因此，在这个例子中，不同的对象有不同的引用，所以在进行比较的时候都应该返回 false。但是奇怪的是，这里两个相似的 if 条件判断却返回不同的布尔值。  

下面是上面代码真正的输出结果，


	integer1==integer2
	integer3!=integer4
 

Java 中 Integer 缓存实现

在 Java 5 中，为 Integer 的操作引入了一个新的特性，用来节省内存和提高性能。整型对象在内部实现中通过使用相同的对象引用实现了缓存和重用。

上面的规则适用于整数区间 -128 到 +127。

这种 Integer 缓存策略仅在自动装箱（autoboxing）的时候有用，使用构造器创建的 Integer 对象不能被缓存。

Java 编译器把原始类型自动转换为封装类的过程称为自动装箱（autoboxing），这相当于调用 valueOf 方法.

我们来看看 valueOf 的源码。

	public static Integer valueOf(int i) {
		if (i >= IntegerCache.low && i <= IntegerCache.high)
			return IntegerCache.cache[i + (-IntegerCache.low)];
		return new Integer(i);
	}
 

在创建新的 Integer 对象之前会先在 IntegerCache.cache 中查找。有一个专门的 Java 类来负责 Integer 的缓存。

IntegerCache 类

IntegerCache 是 Integer 类中一个私有的静态类。我们来看看这个类，有比较详细的文档，可以提供我们很多信息。

	/**
	 * Cache to support the object identity semantics of autoboxing for values between
	 * -128 and 127 (inclusive) as required by JLS.
	 *
	 * The cache is initialized on first usage.  The size of the cache
	 * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
	 * During VM initialization, java.lang.Integer.IntegerCache.high property
	 * may be set and saved in the private system properties in the
	 * sun.misc.VM class.
	 */

	private static class IntegerCache {
		static final int low = -128;
		static final int high;
		static final Integer cache[];

		static {
			// high value may be configured by property
			int h = 127;
			String integerCacheHighPropValue =
				sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
			if (integerCacheHighPropValue != null) {
				try {
					int i = parseInt(integerCacheHighPropValue);
					i = Math.max(i, 127);
					// Maximum array size is Integer.MAX_VALUE
					h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
				} catch( NumberFormatException nfe) {
					// If the property cannot be parsed into an int, ignore it.
				}
			}
			high = h;

			cache = new Integer[(high - low) + 1];
			int j = low;
			for(int k = 0; k < cache.length; k++)
				cache[k] = new Integer(j++);

			// range [-128, 127] must be interned (JLS7 5.1.7)
			assert IntegerCache.high >= 127;
		}

		private IntegerCache() {}
	}


>Javadoc 详细的说明这个类是用来实现缓存支持，并支持 -128 到 127 之间的自动装箱过程。最大值 127 可以通过 JVM 的启动参数 -XX:AutoBoxCacheMax=size 修改。 缓存通过一个 for 循环实现。从小到大的创建尽可能多的整数并存储在一个名为 cache 的整数数组中。这个缓存会在 Integer 类第一次被使用的时候被初始化出来。以后，就可以使用缓存中包含的实例对象，而不是创建一个新的实例(在自动装箱的情况下)。

>实际上在 Java 5 中引入这个特性的时候，范围是固定的 -128 至 +127。后来在 Java 6 中，最大值映射到 java.lang.Integer.IntegerCache.high，可以使用 JVM 的启动参数设置最大值。这使我们可以根据应用程序的实际情况灵活地调整来提高性能。是什么原因选择这个 -128 到 127 这个范围呢？因为这个范围的整数值是使用最广泛的。 在程序中第一次使用 Integer 的时候也需要一定的额外时间来初始化这个缓存。

这种缓存行为不仅适用于Integer对象。我们针对所有整数类型的类都有类似的缓存机制。

有 ByteCache 用于缓存 Byte 对象

有 ShortCache 用于缓存 Short 对象

有 LongCache 用于缓存 Long 对象

有 CharacterCache 用于缓存 Character 对象

Byte，Short，Long 有固定范围: -128 到 127。对于 Character, 范围是 0 到 127。除了 Integer 可以通过参数改变范围外，其它的都不行。

>文章链接:`http://www.importnew.com/18884.html`
