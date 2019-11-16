## 必须理解的Java 类的实例化顺序

### 类实例化过程
我们每次使用类时都要将其实例话，也就是 new。

**每一次的 new 都经历了** 

1. 加载: 将 Class 文件读入内存，并为之创建一个 java.lang.Class 对象
2. 连接: 为静态域分配内存
3. 初始化: 初始化超类，执行 static
4. 实例化: 创建一个 Object 对象

**每一次 new 一个对象时都会（都是先父类再子类）**

+ 如果是第一次 new ，则按顺序执行静态代码块和静态变量（凌驾于所有动态代码块和构造器之上）
+ 按顺序执行动态代码块和动态变量（非 static 的变量都是动态变量）
+ 构造器


	class Father{
	  static {     // 1
		System.out.println("Father static block");
	  }

	  {     // 5
		System.out.println("Father not static block");
	  }

	  public Father(){     // 7
		System.out.println("Father Constructor");
	  }

	  public static int i=0;     // 2
	  public int j=0;     // 6
	}
	public class Test extends Father{
	  public static int i=0;     // 3
	  public int j=0;     // 8

	  public Test(){     // 10
		System.out.println("Test Constructor");
	  }

	  {     // 9
		System.out.println("Test not static block");
	  }

	  static {     // 4
		System.out.println("Test static block");
	  }

	  public static void main(String[] args) throws Exception {
		new Test();
	  }
	}

**输出**

>Father static block
Test static block
Father not static block
Father Constructor
Test not static block
Test Constructor

这里可以看到，不管父类和子类的 static 先执行，然后是父类的动态代码块和构造器，接下来才是子类的动态代码块和构造器。

**至于静态变量如何确定执行**

	class Father{
	  static {
		System.out.println("Father static block");
	  }

	  {
		System.out.println("Father not static block");
	  }

	  public Father(){
		System.out.println("Father Constructor");
	  }

	  public static Father father=new Father(); // 这里要执行一次动态代码块和构造器
	  public int j=0;
	}
	public class Test extends Father{
	  public static void main(String[] args) throws Exception {
		new Father(); // 这里也要执行一次动态代码块和构造器
	  }
	}

**输出**

>Father static block
Father not static block
Father Constructor
Father not static block
Father Constructor

但这里有一个注意点，如果先初始化类的实例会如何

	class Father{
	  public static Father father = new Father();
	  public int j=0;
	  static {
		System.out.println("Father static block");
	  }

	  {
		System.out.println("Father not static block");
	  }

	  public Father(){
		System.out.println("Father Constructor");
	  }

	}
	public class Test extends Father{
	  public static void main(String[] args) throws Exception {
		new Father();
	  }
	}

**输出**

>Father not static block
Father Constructor
Father static block
Father not static block
Father Constructor

在 public static Father father=new Father(); 时，静态代码块虽还未执行，但可以看作它已经加载被确定执行，所以这里并没有执行静态代码块。

那么动态变量就很容易理解了，但是会死循环(~_~;)

	class Father {
	  public Father father = new Father();
	}

	public class Test extends Father {
	  public static void main(String[] args) throws Exception {
		new Father();
	  }
	}

### 类的初始化
我们每一次的 new 都是创建一个 Object 对象，也就是 Class 类中的 newInstance() 需要 Class 对象的引用去调用，如：Test.class.newInstance();

但是 Class 可以在不实例化类的情况下将类初始化，也就是 .class 或者 Class.formName();

	public class Test {
	  static {
		System.out.println("static");
	  }
	  {
		System.out.println("not static");
	  }
	  public static void main(String[] args) throws Exception {
		Class test=Test.class;
	  }
	}


**输出**

>static

此时类只初始化，执行 static ，之后的实例化都不会再执行 static。


>文章链接:`https://blog.csdn.net/Vencc__/article/details/52222628`
