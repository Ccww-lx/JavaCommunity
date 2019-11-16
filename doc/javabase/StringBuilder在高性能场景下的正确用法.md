## StringBuilder在高性能场景下的正确用法

### 初始长度很重要
`StringBuilder` 的内部有一个`char[]`，不同的`append()`就是不断的往`char[]`里填东西的过程。

`new StringBuilder()`时`char[]`的默认长度为16，超过就用`System.arraycopy`成倍复制扩容。

这样一来有数组拷贝的成本，二来原来的`char[]`也白白浪费了，要被`GC`掉。

所以，合理设置一个初始值是很重要的。

一种长度设置的思路，在`append()`的时候，不急着往`char[]`里塞东西，而是先拿一个`String[]`把它们都存起来，到了最后才把所有`String`的`length`加起来，构造一个合理长度的`StringBuilder`。

### Liferay的StringBundler类
Liferay的StringBundler类提供了另一个长度设置的思路，它在append()的时候，不急着往char[]里塞东西，而是先拿一个String[]把它们都存起来，到了最后才把所有String的length加起来，构造一个合理长度的StringBuilder。


### 浪费一倍的char[]
因为：

  	return new String(value, 0, count);

String的构造函数会用System.arraycopy()复制一次传入的char[]来保证安全性及不可变性，这样StringBuilder里的char[]就白白牺牲掉了。

为了不浪费这些char[]，可以重用StringBuilder。

### 重用StringBuilder 
	public StringBuilder getStringBuilder() {
		sb.setLength(0);
		return sb;
	}
为了避免并发冲突，这个Holder一般设为ThreadLocal。

### +和StringBuilder
	String str = "hello " + user.getName();
这一句经过javac编译后的效果，的确等价于使用StringBuilder，但没有设定长度。

但是，如果像下面这样：

	String str = "hello ";
	str = str + user.getName();
每一条语句，都会生成一个新的StringBuilder，这样这里就有了两个StringBuilder，性能就完全不一样了。

保险起见，还是继续自己用StringBuilder并设定好长度。

	private static final ThreadLocal<StringBuilderHelper> threadLocalStringBuilderHolder = new ThreadLocal<StringBuilderHelper>() {
		protected StringBuilderHelper initialValue() {
			return new StringBuilderHelper(256);
		}
	}

	StringBuilder sb = threadLocalStringBuilderHolder.get().resetAndGetStringBuilder();
	
**StringBuidlerHolder**

	public class StringBuilderHolder {
		private final StringBuilder sb;

		public StringBuilderHolder(int capacity) {
			sb = new StringBuidler(capacity);
		}

		public StringBuilder resetAndGetStringBuilder() {
			sb.setLength(0);
			return sb;
		}
	}
	

>文章链接:`http://calvin1978.blogcn.com/articles/stringbuilder.html`
