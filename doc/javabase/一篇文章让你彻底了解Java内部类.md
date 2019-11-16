
## 一篇文章让你彻底了解Java内部类

为什么需要内部类？
一般来说，内部类继承自某个类或实现某个接口，内部类的代码操作创建其的外围类的对象。所以你可以认为内部类提供了某种进入其外围类的窗口。

内部类的优雅之处：
每个内部类都能独立的继承一个（接口的）实现，无论外部类是否已经继承了一个（接口的）实现，对内部类都没有影响。

内部类主要有以下几类：
+ 成员内部类
+ 局部内部类
+ 静态内部类
+ 匿名内部类

注意:

1. 定义了成员内部类后，必须使用外部类对象来创建内部类对象，而不能直接去 new 一个内部类对象，
即：内部类 对象名 = 外部类对象.new 内部类( );
2. 外部类是不能直接使用内部类的成员和方法滴，可先创建内部类的对象，然后通过内部类的对象来访问其成员变量和方法。
3. 可先创建内部类的对象，然后通过内部类的对象来访问其成员变量和方法。HelloWorld.this.name
## 成员内部类

作为外部类的一个成员存在，与外部类的属性、方法并列。

	public class Outer {

		private static int i = 1;
		private int j = 10;
		private int k = 20;


		public static void outerF1() {
		}

		/**
		 * 外部类的静态方法访问成员内部类，与在外部类外部访问成员内部类一样
		 */
		public static void outerF4() {
			//step1 建立外部类对象
			Outer out = new Outer();
			//step2 根据外部类对象建立内部类对象
			Inner inner = out.new Inner();
			//step3 访问内部类的方法
			inner.innerF1();
		}

		public static void main(String[] args) {

			/*
			 * outerF4();该语句的输出结果和下面三条语句的输出结果一样
			 *如果要直接创建内部类的对象，不能想当然地认为只需加上外围类Outer的名字，
			 *就可以按照通常的样子生成内部类的对象，而是必须使用此外围类的一个对象来
			 *创建其内部类的一个对象：
			 *Outer.Inner outin = out.new Inner()
			 *因此，除非你已经有了外围类的一个对象，否则不可能生成内部类的对象。因为此
			 *内部类的对象会悄悄地链接到创建它的外围类的对象。如果你用的是静态的内部类，
			 *那就不需要对其外围类对象的引用。
			 */
			Outer out = new Outer();
			Outer.Inner outin = out.new Inner();
			outin.innerF1();
		}

		public void outerF2() {
		}

		/**
		 * 外部类的非静态方法访问成员内部类
		 */
		public void outerF3() {
			Inner inner = new Inner();
			inner.innerF1();
		}

		/**
		 * 成员内部类中，不能定义静态成员
		 * 成员内部类中，可以访问外部类的所有成员
		 */
		class Inner {
			// static int innerI = 100;内部类中不允许定义静态变量
			// 内部类和外部类的实例变量可以共存
			int j = 100;
			int innerI = 1;


			void innerF1() {
				System.out.println(i);
				//在内部类中访问内部类自己的变量直接用变量名
				System.out.println(j);
				//在内部类中访问内部类自己的变量也可以用this.变量名
				System.out.println(this.j);
				//在内部类中访问外部类中与内部类同名的实例变量用外部类名.this.变量名
				System.out.println(Outer.this.j);
				//如果内部类中没有与外部类同名的变量，则可以直接用变量名访问外部类变量
				System.out.println(k);
				outerF1();
				outerF2();
			}
		}
	}

>注意：内部类是一个编译时的概念，一旦编译成功，就会成为完全不同的两类。
对于一个名为outer的外部类和其内部定义的名为inner的内部类。编译完成后出现outer.class和outer$inner.class两类。

## 局部内部类

在方法中定义的内部类称为局部内部类。与局部变量类似，局部内部类不能有访问说明符，因为它不是外围类的一部分，但是它可以访问当前代码块内的常量，和此外围类所有的成员。


	public class Outer {

		private int s = 100;
		private int outI = 1;

		public static void main(String[] args) {
			// 访问局部内部类必须先有外部类对象
			Outer out = new Outer();
			out.f(3);
		}

		public void f(final int k) {
			final int s = 200;
			int i = 1;
			final int j = 10;


			/**
			 * 定义在方法内部
			 */
			class Inner {
				// 可以定义与外部类同名的变量
				int s = 300;
				int innerI = 100;

				// static int m = 20; 不可以定义静态变量
				Inner(int k) {
					innerF(k);
				}
				void innerF(int k) {
					// java如果内部类没有与外部类同名的变量，在内部类中可以直接访问外部类的实例变量
					System.out.println(outI);
					// 可以访问外部类的局部变量(即方法内的变量)，但是变量必须是final的
					System.out.println(j);
					//System.out.println(i);
					// 如果内部类中有与外部类同名的变量，直接用变量名访问的是内部类的变量
					System.out.println(s);
					// 用this.变量名访问的也是内部类变量
					System.out.println(this.s);
					// 用外部类名.this.内部类变量名访问的是外部类变量
					System.out.println(Outer.this.s);
				}
			}
			new Inner(k);
		}
	}


##静态内部类(嵌套类)：


**注意：前两种内部类与变量类似，所以可以对照参考变量
**
>如果你不需要内部类对象与其外围类对象之间有联系，那你可以将内部类声明为static。这通常称为嵌套类（nested class）。想要理解static应用于内部类时的含义，你就必须记住，普通的内部类对象隐含地保存了一个引用，指向创建它的外围类对象。然而，当内部类是static的时，就不是这样了。嵌套类意味着：

+ 要创建嵌套类的对象，并不需要其外围类的对象。
+ 不能从嵌套类的对象中访问非静态的外围类对象。

单例模式:由于静态内部类的加载机制,决定了他可以使用来处理单例模式


	public class Outer {
		private static int i = 1;
		private int j = 10;

		public static void outerF1() {
		}

		public static void main(String[] args) {
			new Outer().outerF3();
		}

		public void outerF2() {
		}

		public void outerF3() {
			// 外部类访问内部类的静态成员：内部类.静态成员
			System.out.println(Inner.inner_i);
			Inner.innerF1();
			// 外部类访问内部类的非静态成员:实例化内部类即可
			Inner inner = new Inner();
			inner.innerF2();
		}

		/**
		 * 静态内部类可以用public,protected,private修饰
		 * 静态内部类中可以定义静态或者非静态的成员
		 */
		static class Inner {
			static int inner_i = 100;
			int innerJ = 200;

			static void innerF1() {
				// 静态内部类只能访问外部类的静态成员(包括静态变量和静态方法)
				System.out.println("Outer.i" + i);
				outerF1();
			}


			void innerF2() {
				// 静态内部类不能访问外部类的非静态成员(包括非静态变量和非静态方法)
				// System.out.println("Outer.i"+j);
				// outerF2();
			}
		}
	}

### 静态内部类和成员内部类的区别
>生成一个静态内部类不需要外部类成员
静态内部类的对象可以直接生成：
Outer.Inner in = new Outer.Inner();
而不需要通过生成外部类对象来生成。这样实际上使静态内部类成为了一个顶级类
(正常情况下，你不能在接口内部放置任何代码，但嵌套类可以作为接口的一部分，因为它是static 的。只是将嵌套类置于接口的命名空间内，这并不违反接口的规则）

## 匿名内部类(from thinking in java 3th)

匿名内部类就是没有名字的内部类。

### 什么情况下需要使用匿名内部类？
如果满足下面的一些条件，使用匿名内部类是比较合适的：

+ 只用到类的一个实例。
+ 类在定义后马上用到。
+ 类非常小（SUN推荐是在4行代码以下）
+ 给类命名并不会导致你的代码更容易被理解。

在使用匿名内部类时，要记住以下几个原则：

+ 匿名内部类一般不能有构造方法。
+ 匿名内部类不能定义任何静态成员、方法和类。
+ 匿名内部类不能是public,protected,private,static。
+ 	只能创建匿名内部类的一个实例。
+ 一个匿名内部类一定是在new的后面，用其隐含实现一个接口或实现一个类。
+ 因匿名内部类为局部内部类，所以局部内部类的所有限制都对其生效。

下面的例子看起来有点奇怪：

	// 在方法中返回一个匿名内部类
	public class Parcel6 {
		public static void main(String[] args) {
			Parcel6 p = new Parcel6();
			Contents c = p.cont();
		}

		public Contents cont() {
			return new Contents() {
				private int i = 11;


				public int value() {
					return i;
				}
			}; // 在这里需要一个分号
		}
	}

cont()方法将下面两个动作合并在一起：返回值的生成，与表示这个返回值的类的定义！
　　进一步说，这个类是匿名的，它没有名字。更糟的是，看起来是你正要创建一个Contents对象：

	return new Contents()

但是，在到达语句结束的分号之前，你却说：“等一等，我想在这里插入一个类的定义”:

	return new Contents() {
		private int i = 11;

		public int value() {
			return i;
		}
	};


这种奇怪的语法指的是：“创建一个继承自Contents的匿名类的对象。”通过new 表达式返回的引用被自动向上转型为对Contents的引用。匿名内部类的语法是下面例子的简略形式：

	class MyContents implements Contents {
		private int i = 11;

		public int value() {
			return i;
		}
	}
	return new MyContents();

在这个匿名内部类中，使用了缺省的构造器来生成Contents。下面的代码展示的是，如果你的基类需要一个有参数的构造器，应该怎么办：

	public class Parcel7 {
		public static void main(String[] args) {
			Parcel7 p = new Parcel7();
			Wrapping w = p.wrap(10);
		}

		public Wrapping wrap(int x) {
			// Base constructor call:
			// Pass constructor argument.
			return new Wrapping(x) { 
				public int value() {
					return super.value() * 47;
				}
			}; // Semicolon required
		}
	}


只需简单地传递合适的参数给基类的构造器即可，这里是将x 传进new Wrapping(x)。在匿名内部类末尾的分号，并不是用来标记此内部类结束（C++中是那样）。实际上，它标记的是表达式的结束，只不过这个表达式正巧包含了内部类罢了。因此，这与别的地方使用的分号是一致的。

如果在匿名类中定义成员变量，你同样能够对其执行初始化操作：

	public class Parcel8 {
		public static void main(String[] args) {
			Parcel8 p = new Parcel8();
			Destination d = p.dest("Tanzania");
		}

		// Argument must be final to use inside
		// anonymous inner class:
		public Destination dest(final String dest) {
			return new Destination() {
				private String label = dest;

				public String readLabel() {
					return label;
				}
			};
		}
	}


如果你有一个匿名内部类，它要使用一个在它的外部定义的对象，编译器会要求其参数引用是final 型的，就像dest()中的参数。如果你忘记了，会得到一个编译期错误信息。如果只是简单地给一个成员变量赋值，那么此例中的方法就可以了。但是，如果你想做一些类似构造器的行为，该怎么办呢？在匿名类中不可能有已命名的构造器（因为它根本没名字！），但通过实例初始化，你就能够达到为匿名内部类“制作”一个构造器的效果。像这样做：

	abstract class Base {
		public Base(int i) {
			System.out.println("Base constructor, i = " + i);
		}

		public abstract void f();
	}


	public class AnonymousConstructor {
		public static Base getBase(int i) {
			return new Base(i) {
				{
					System.out.println("Inside instance initializer");
				}

				public void f() {
					System.out.println("In anonymous f()");
				}
			};
		}

		public static void main(String[] args) {
			Base base = getBase(47);
			base.f();
		}
	}

在此例中，不要求变量i 一定是final 的。因为i 被传递给匿名类的基类的构造器，它并不会在匿名类内部被直接使用。下例是带实例初始化的“parcel”形式。注意dest()的参数必须是final，因为它们是在匿名类内被使用的。


	public class Parcel9 {
		public Destinationdest(final String dest, final float price) {
			return new Destination() {
				private int cost;
				private String label = dest;

				// Instance initialization for each object:
				{
					cost = Math.round(price);
					if (cost > 100)
						System.out.println("Over budget!");
				}

				public String readLabel() {
					return label;
				}
			};
		}

		public static void main(String[] args) {
			Parcel9 p = new Parcel9();
			Destination d = p.dest("Tanzania", 101.395F);
		}
	}

在实例初始化的部分，你可以看到有一段代码，那原本是不能作为成员变量初始化的一部分而执行的（就是if 语句）。所以对于匿名类而言，实例初始化的实际效果就是构造器。当然它受到了限制：你不能重载实例初始化，所以你只能有一个构造器。

### 从多层嵌套类中访问外部
一个内部类被嵌套多少层并不重要，它能透明地访问所有它所嵌入的外围类的所有成员，如下所示：


	class MNA {
		private void f() {
		}

		class A {
			private void g() {
			}

			public class B {
				void h() {
					g();
					f();
				}
			}
		}
	}

	public class MultiNestingAccess {
		public static void main(String[] args) {
			MNA mna = new MNA();
			MNA.A mnaa = mna.new A();
			MNA.A.B mnaab = mnaa.new B();
			mnaab.h();
		}
	}


可以看到在MNA.A.B中，调用方法g()和f()不需要任何条件（即使它们被定义为private）。这个例子同时展示了如何从不同的类里面创建多层嵌套的内部类对象的基本语法。“.new”语法能产生正确的作用域，所以你不必在调用构造器时限定类名。

### 内部类的重载问题
如果你创建了一个内部类，然后继承其外围类并重新定义此内部类时，会发生什么呢？也就是说，内部类可以被重载吗？这看起来似乎是个很有用的点子，但是“重载”内部类就好像它是外围类的一个方法，其实并不起什么作用：


	class Egg {
		private Yolk y;


		public Egg() {
			System.out.println("New Egg()");
			y = new Yolk();
		}

		protected class Yolk {
			public Yolk() {
				System.out.println("Egg.Yolk()");
			}
		}
	}


	public class BigEgg extends Egg {
		public static void main(String[] args) {
			new BigEgg();
		}

		public class Yolk {
			public Yolk() {
				System.out.println("BigEgg.Yolk()");
			}
		}
	}

输出结果为：

>New Egg()
Egg.Yolk()

缺省的构造器是编译器自动生成的，这里是调用基类的缺省构造器。你可能认为既然创建了BigEgg 的对象，那么所使用的应该是被“重载”过的Yolk，但你可以从输出中看到实际情况并不是这样的。

这个例子说明，当你继承了某个外围类的时候，内部类并没有发生什么特别神奇的变化。这两个内部类是完全独立的两个实体，各自在自己的命名空间内。当然，明确地继承某个内部类也是可以的：

	class Egg2 {
		private Yolk y = new Yolk();


		public Egg2() {
			System.out.println("New Egg2()");
		}

		public void insertYolk(Yolk yy) {
			y = yy;
		}

		public void g() {
			y.f();
		}

		protected class Yolk {
			public Yolk() {
				System.out.println("Egg2.Yolk()");
			}


			public void f() {
				System.out.println("Egg2.Yolk.f()");
			}
		}
	}


	public class BigEgg2 extends Egg2 {
		public BigEgg2() {
			insertYolk(new Yolk());
		}

		public static void main(String[] args) {
			Egg2 e2 = new BigEgg2();
			e2.g();
		}

		public class Yolk extends Egg2.Yolk {
			public Yolk() {
				System.out.println("BigEgg2.Yolk()");
			}


			public void f() {
				System.out.println("BigEgg2.Yolk.f()");
			}
		}
	}

输出结果为：

>Egg2.Yolk()
New Egg2()
Egg2.Yolk()
BigEgg2.Yolk()
BigEgg2.Yolk.f()

现在BigEgg2.Yolk 通过extends Egg2.Yolk 明确地继承了此内部类，并且重载了其中的方法。Egg2 的insertYolk()方法使得BigEgg2 将它自己的Yolk 对象向上转型，然后传递给引用y。所以当g()调用y.f()时，重载后的新版的f()被执行。第二次调用Egg2.Yolk()是BigEgg2.Yolk 的构造器调用了其基类的构造器。可以看到在调用g()的时候，新版的f()被调用了。

### 内部类的继承问题（thinking in java 3th p294）
因为内部类的构造器要用到其外围类对象的引用，所以在你继承一个内部类的时候，事情变得有点复杂。问题在于，那个“秘密的”外围类对象的引用必须被初始化，而在被继承的类中并不存在要联接的缺省对象。要解决这个问题，需使用专门的语法来明确说清它们之间的关联：

	class WithInner {
		class Inner {
			Inner() {
				System.out.println("this is a constructor in WithInner.Inner");
			}

			;
		}
	}


	public class InheritInner extends WithInner.Inner {
		// ! InheritInner() {} // Won't compile
		InheritInner(WithInner wi) {
			wi.super();
			System.out.println("this is a constructor in InheritInner");
		}


		public static void main(String[] args) {
			WithInner wi = new WithInner();
			InheritInner ii = new InheritInner(wi);
		}
	}


输出结果为：

>this is a constructor in WithInner.Inner
this is a constructor in InheritInner

可以看到，InheritInner 只继承自内部类，而不是外围类。但是当要生成一个构造器时，缺省的构造器并不算好，而且你不能只是传递一个指向外围类对象的引用。此外，你必须在构造器内使用如下语法：

	enclosingClassReference.super();

### 关于Java回调函数
在Java中，通常就是编写另外一个类或类库的人规定一个接口，然后你来实现这个接口，然后把这个接口的一个对象作为参数传给别人的程序，别人的程序必要时就会通过那个接口来调用你编写的函数,执行后续的一些方法,。


	public class CallBack {

		public static void main(String[] args) {
			CallBack callBack = new CallBack();
			callBack.toDoSomethings(100, new CallBackInterface() {
				public void execute() {
					System.out.println("我的请求处理成功了");
				}
			});

		}

		public void toDoSomethings(int a, CallBackInterface callBackInterface) {
			long start = System.currentTimeMillis();
			if (a > 100) {
				callBackInterface.execute();
			} else {
				System.out.println("a < 100 不需要执行回调方法");
			}
			long end = System.currentTimeMillis();
			System.out.println("该接口回调时间 : " + (end - start));
		}
	}
	public interface CallBackInterface {

		void execute();
	}

Java回调的实现,是不是就是基于匿名内部类实现的呢?答案是肯定的.

Java里的回调,可以说是匿名内部类精彩表演,优美的编码风格,真是让人陶醉~


>文章链接:`https://blog.csdn.net/sinat_34344123/article/details/81942427`
