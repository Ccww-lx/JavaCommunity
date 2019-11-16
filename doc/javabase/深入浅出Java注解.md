## Java对象的浅克隆和深克隆

### 什么是注解
注解对于开发人员来讲既熟悉又陌生，熟悉是因为只要你是做开发，都会用到注解（常见的@Override）；陌生是因为即使不使用注解也照常能够进行开发；注解不是必须的，但了解注解有助于我们深入理解某些第三方框架（比如Android Support Annotations、JUnit、xUtils、ActiveAndroid等），提高工作效率。

Java注解又称为标注，是Java从1.5开始支持加入源码的特殊语法元数据；Java中的类、方法、变量、参数、包都可以被注解。这里提到的元数据是描述数据的数据，结合实例来说明：

	<string name="app_name">AnnotionDemo</string>
	
这里的"app_name"就是描述数据"AnnotionDemo"的数据，这是在配置文件中写的，注解是在源码中写的，如下所示：

	@Override
	protected void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main_layout);
		new Thread(new Runnable(){
			@Override
			public void run(){
				setTextInOtherThread();
			}
		}).start();
	}
	
在上面的代码中，在MainActivity.java中复写了父类Activity.java的onCreate方法，使用到了@Override注解。但即使不加上@Override注解标记代码，程序也能够正常运行。那这里的@Override注解有什么用呢？使用它有什么好处？事实上，@Override是告诉编译器这个方法是一个重写方法，如果父类中不存在该方法，编译器会报错，提示该方法不是父类中的方法。如果不小心拼写错误，将onCreate写成了onCreat，而且没有使用@Override注解，程序依然能够编译通过，但运行结果和期望的大不相同。从示例可以看出，注解有助于阅读代码。

使用注解很简单，根据注解类的@Target所修饰的对象范围，可以在类、方法、变量、参数、包中使用“@+注解类名+[属性值]”的方式使用注解。比如：

	@UiThread
	private void setTextInOtherThread(@StringRes int resId){
		TextView threadTxtView = (TextView)MainActivity.this.findViewById(R.id.threadTxtViewId);
		threadTxtView.setText(resId);
	}
	
特别说明：

注解仅仅是元数据，和业务逻辑无关，所以当你查看注解类时，发现里面没有任何逻辑处理；

javadoc中的@author、@version、@param、@return、@deprecated、@hide、@throws、@exception、@see是标记，并不是注解；

### 注解的作用
+ 格式检查：告诉编译器信息，比如被@Override标记的方法如果不是父类的某个方法，IDE会报错；

+ 减少配置：运行时动态处理，得到注解信息，实现代替配置文件的功能；

+ 减少重复工作：比如第三方框架xUtils，通过注解@ViewInject减少对findViewById的调用，类似的还有（JUnit、ActiveAndroid等）；

### 注解是如何工作的？
注解仅仅是元数据，和业务逻辑无关，所以当你查看注解类时，发现里面没有任何逻辑处理，eg：

	@Target(ElementType.FIELD)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface ViewInject {

		int value();

		/* parent view id */
		int parentId() default 0;
	}
	
如果注解不包含业务逻辑处理，必然有人来实现这些逻辑。注解的逻辑实现是元数据的用户来处理的，注解仅仅提供它定义的属性（类/方法/变量/参数/包）的信息，注解的用户来读取这些信息并实现必要的逻辑。当使用java中的注解时（比如@Override、@Deprecated、@SuppressWarnings）JVM就是用户，它在字节码层面工作。如果是自定义的注解，比如第三方框架ActiveAndroid，它的用户是每个使用注解的类，所有使用注解的类都需要继承Model.java，在Model.java的构造方法中通过反射来获取注解类中的每个属性：

	public TableInfo(Class<? extends Model> type) {
		mType = type;

		final Table tableAnnotation = type.getAnnotation(Table.class);

		if (tableAnnotation != null) {
			mTableName = tableAnnotation.name();
			mIdName = tableAnnotation.id();
		}
		else {
			mTableName = type.getSimpleName();
		}

		// Manually add the id column since it is not declared like the other columns.
		Field idField = getIdField(type);
		mColumnNames.put(idField, mIdName);

		List<Field> fields = new LinkedList<Field>(ReflectionUtils.getDeclaredColumnFields(type));
		Collections.reverse(fields);

		for (Field field : fields) {
			if (field.isAnnotationPresent(Column.class)) {
				final Column columnAnnotation = field.getAnnotation(Column.class);
				String columnName = columnAnnotation.name();
				if (TextUtils.isEmpty(columnName)) {
					columnName = field.getName();
				}

				mColumnNames.put(field, columnName);
			}
		}

	}
	
### 注解和配置文件的区别
通过上面的描述可以发现，其实注解干的很多事情，通过配置文件也可以干，比如为类设置配置属性；但注解和配置文件是有很多区别的，在实际编程过程中，注解和配置文件配合使用在工作效率、低耦合、可拓展性方面才会达到权衡。

**配置文件：** 

使用场合：
+ 外部依赖的配置，比如build.gradle中的依赖配置；
+ 同一项目团队内部达成一致的时候；
+ 非代码类的资源文件（比如图片、布局、数据、签名文件等）；

优点：
+ 降低耦合，配置集中，容易扩展，比如Android应用多语言支持；
+ 对象之间的关系一目了然，比如strings.xml；
+ xml配置文件比注解功能齐全，支持的类型更多，比如drawable、style等；

缺点：
+ 繁琐；
+ 类型不安全，比如R.java中的都是资源ID，用TextView的setText方法时传入int值时无法检测出该值是否为资源ID，但@StringRes可以；

注解：

使用场合：
+ 动态配置信息；
+ 代为实现程序逻辑（比如xUtils中的@ViewInject代为实现findViewById）；
+ 代码格式检查，比如Override、Deprecated、NonNull、StringRes等，便于IDE能够检查出代码错误；

优点：
+ 在class文件中，提高程序的内聚性；
+ 减少重复工作，提高开发效率，比如findViewById。

缺点：
+ 如果对annotation进行修改，需要重新编译整个工程；
+ 业务类之间的关系不如XML配置那样一目了然；
+ 程序中过多的annotation，对于代码的简洁度有一定影响；
+ 扩展性较差；


>文章链接：https://www.jianshu.com/p/5cac4cb9be54

