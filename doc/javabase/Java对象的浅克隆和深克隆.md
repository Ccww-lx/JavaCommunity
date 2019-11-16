## Java对象的浅克隆和深克隆

 ### 为什么需要克隆
在实际编程过程中，我们常常要遇到这种情况：有一个对象A，在某一时刻A中已经包含了一些有效值，此时可能会需要一个和A完全相同新对象B，并且此后对B任何改动都不会影响到A中的值，也就是说，A与B是两个独立的对象，但B的初始值是由A对象确定的。在Java语言中，用简单的赋值语句是不能满足这种需求的，要满足这种需求有很多途径。
 
### 克隆的实现方式
### 一、浅度克隆
浅度克隆对于要克隆的对象，对于其基本数据类型的属性，复制一份给新产生的对象，对于非基本数据类型的属性，仅仅复制一份引用给新产生的对象，即新产生的对象和原始对象中的非基本数据类型的属性都指向的是同一个对象。

**浅度克隆步骤：**
1. 实现java.lang.Cloneable接口
要clone的类为什么还要实现Cloneable接口呢？Cloneable接口是一个标识接口，不包含任何方法的！这个标识仅仅是针对Object类中clone()方法的，如果clone类没有实现Cloneable接口，并调用了Object的 clone()方法（也就是调用了super.Clone()方法），那么Object的clone()方法就会抛出 CloneNotSupportedException异常。
2. 重写java.lang.Object.clone()方法
JDK API的说明文档解释这个方法将返回Object对象的一个拷贝。要说明的有两点：一是拷贝对象返回的是一个新对象，而不是一个引用。二是拷贝对象与用new操作符返回的新对象的区别就是这个拷贝已经包含了一些原来对象的信息，而不是对象的初始信息。

 观察一下Object类的clone()方法是一个native方法，native方法的效率一般来说都是远高于java中的非native方法。这也解释了为什么要用Object中clone()方法而不是先new一个类，然后把原始对象中的信息赋到新对象中，虽然这也实现了clone功能。Object类中的clone()还是一个protected属性的方法，重写之后要把clone()方法的属性设置为public。
 
 Object类中clone()方法产生的效果是：先在内存中开辟一块和原始对象一样的空间，然后原样拷贝原始对象中的内容。对基本数据类型，这样的操作是没有问题的，但对非基本类型变量，我们知道它们保存的仅仅是对象的引用，这也导致clone后的非基本类型变量和原始对象中相应的变量指向的是同一个对象。
 
**Java代码实例：**

	public class Product implements Cloneable {   
		private String name;   

		public Object clone() {   
			try {   
				return super.clone();   
			} catch (CloneNotSupportedException e) {   
				return null;   
			}   
		}   
	}  
 
 ### 二、深度克隆
在浅度克隆的基础上，对于要克隆的对象中的非基本数据类型的属性对应的类，也实现克隆，这样对于非基本数据类型的属性，复制的不是一份引用，即新产生的对象和原始对象中的非基本数据类型的属性指向的不是同一个对象

**深度克隆步骤：**
要克隆的类和类中所有非基本数据类型的属性对应的类
1. 都实现java.lang.Cloneable接口
2. 都重写java.lang.Object.clone()方法
 
**Java代码实例：** 

	public class Attribute implements Cloneable {   
		private String no;   

		public Object clone() {   
			try {   
				return super.clone();   
			} catch (CloneNotSupportedException e) {   
				return null;   
			}   
		}   
	}   

	public class Product implements Cloneable {   
		private String name;   

		private Attribute attribute;   

		public Object clone() {   
			try {   
				return super.clone();   
			} catch (CloneNotSupportedException e) {   
				return null;   
			}   
		}   
	}   
 
 
### 三、使用对象序列化和反序列化实现深度克隆
**所谓对象序列化就是将对象的状态转换成字节流，以后可以通过这些值再生成相同状态的对象。**
 
对象的序列化还有另一个容易被大家忽略的功能就是对象复制（Clone），Java中通过Clone机制可以复制大部分的对象，但是众所周知，Clone有深度Clone和浅度Clone，如果你的对象非常非常复杂，并且想实现深层 Clone，如果使用序列化，不会超过10行代码就可以解决。
 
虽然Java的序列化非常简单、强大，但是要用好，还有很多地方需要注意。比如曾经序列化了一个对象，可由于某种原因，该类做了一点点改动，然后重新被编译，那么这时反序列化刚才的对象，将会出现异常。 你可以通过添加serialVersionUID属性来解决这个问题。如果你的类是个单例（Singleton）类，虽然我们多线程下的并发问题来控制单例，但是，是否允许用户通过序列化机制或者克隆来复制该类，如果不允许你需要谨慎对待该类的实现（让此类不用实现Serializable接口或Externalizable接口和Cloneable接口就行了）。
  
**Java代码实例：**
	public class Attribute {   
		private String no;   
	}   

	public class Product {   
		private String name;   

		private Attribute attribute;   

		public Product clone() {   
			ByteArrayOutputStream byteOut = null;   
			ObjectOutputStream objOut = null;   
			ByteArrayInputStream byteIn = null;   
			ObjectInputStream objIn = null;   

			try {   
	// 将该对象序列化成流,因为写在流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。所以利用这个特性可以实现对象的深拷贝
				byteOut = new ByteArrayOutputStream();    
				objOut = new ObjectOutputStream(byteOut);    
				objOut.writeObject(this);   
	  // 将流序列化成对象
				byteIn = new ByteArrayInputStream(byteOut.toByteArray());   
				objIn = new ObjectInputStream(byteIn);   

				return (ContretePrototype) objIn.readObject();   
			} catch (IOException e) {   
				throw new RuntimeException("Clone Object failed in IO.",e);      
			} catch (ClassNotFoundException e) {   
				throw new RuntimeException("Class not found.",e);      
			} finally{   
				try{   
					byteIn = null;   
					byteOut = null;   
					if(objOut != null) objOut.close();      
					if(objIn != null) objIn.close();      
				}catch(IOException e){      
				}      
			}   
		}   
	} 
	
**或者：**

	public static <T extends Serializable> T copy(T input) {
		ByteArrayOutputStream baos = null;
		ObjectOutputStream oos = null;
		ByteArrayInputStream bis = null;
		ObjectInputStream ois = null;
		try {
			baos = new ByteArrayOutputStream();
			oos = new ObjectOutputStream(baos);
			oos.writeObject(input);
			oos.flush();

			byte[] bytes = baos.toByteArray();
			bis = new ByteArrayInputStream(bytes);
			ois = new ObjectInputStream(bis);
			Object result = ois.readObject();
			return (T) result;
		} catch (IOException e) {
			throw new IllegalArgumentException("Object can't be copied", e);
		} catch (ClassNotFoundException e) {
			throw new IllegalArgumentException("Unable to reconstruct serialized object due to invalid class definition", e);
		} finally {
			closeQuietly(oos);
			closeQuietly(baos);
			closeQuietly(bis);
			closeQuietly(ois);
		}
	}

**也可以用json等其他序列化技术实现深度复制；**

## 四、各框架Bean复制
Bean复制的几种框架中（Apache BeanUtils、PropertyUtils,Spring BeanUtils,Cglib BeanCopier）都是相当于克隆中的浅克隆。

1)spring包和Apache中的 BeanUtils采用反射实现

>Spring： void copyProperties(Object source, Object target,String[] ignoreProperties)
		Apache：void  copyProperties(Object dest, Object orig)

2)cglib包中的  Beancopier采用动态字节码实现
   >cglib: BeanCopier create(Class source, Class target,boolean useConverter)
   
  例如：
  
        BeanCopier copier =BeanCopier.create(stuSource.getClass(), stuTarget.getClass(), false);
        copier.copy(stuSource, stuTarget, null);

公司内部对用到的bean属性复制做了下性能分析：

|框架|类|性能|
|-|-|-|
|cglib  |  BeanCopier  | 15ms|
|Spring | BeanUtil    |  4031ms|
|apache |BeanUtils  |    18514ms|

>文章链接：`https://blog.csdn.net/caomiao2006/article/details/52590622`
