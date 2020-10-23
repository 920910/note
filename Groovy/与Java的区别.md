* 默认导包
  ```groovy
  // groovy默认导入的包
	java.io.*

	java.lang.*

	java.math.BigDecimal

	java.math.BigInteger

	java.net.*

	java.util.*

	groovy.lang.*

	groovy.util.*
  ```
  * 重载
  ```grrovy
  // 在Groovy中方法是在运行时选择调用的，称为运行时调度或者多方法。这意味着方法的执行是基于运行时参数的类型而不是定义的类型，例如
  	int method(String arg) {
    return 1;
	}
	int method(Object arg) {
    	return 2;
	}
	Object o = "Object";
	int result = method(o);
	// 在java中result=2而在groovy中result=1
  ```
  * 数组初始化
  ```groovy
  // 在Groovy中{}块用于闭包，所以数组的初始化使用[],例如
  	int[] array = [1,2,3]
  ```
  * 包范围可见性
  ```grrovy
  // 在Groovy中，省略字段上的修饰符不会导致像Java一样的包私有字段,相反，它用于创建属性，即私有字段，关联的getter和关联的setter,若想创建包私有字段需要使用@PackageScope，例如
	class Person {
    	@PackageScope String name
	}
  ```
  * 自动资源管理块
  ```groovy
  // Groovy不支持Java7中的ARM(自动资源管理)块.相反Groovy提供了依赖闭包的各种方法,具有相同的效果，而更具惯用性，例如
  	Path file = Paths.get("/path/to/file");
	Charset charset = Charset.forName("UTF-8");
	try (BufferedReader reader = Files.newBufferedReader(file, charset)) {
    	String line;
    	while ((line = reader.readLine()) != null) {
        	System.out.println(line);
    	}

	} catch (IOException e) {
    	e.printStackTrace();
	}

	// 能够被重写成
	new File('/path/to/file').eachLine('UTF-8') {
 	  println it
	}
	// 或者更接近java的写法
	new File('/path/to/file').withReader('UTF-8') { reader ->
  	 reader.eachLine {
  	     println it
 	  }
	}
  ```
  * 内部类
  > 匿名内部类和嵌套类的实现遵循Java主导,但是你不应该拿出Java语言规范并不断地讨论不同的事情,它的实现有点像闭包，这有好有坏。如，访问私有字段和方法可能会成为一个问题，但另一方面，局部变量不一定是最终的。
  ```groovy
  	// 静态内部类，Groovy为静态内部类的使用提供了最好的支持。如果你绝对需要一个内部类，你应该把它变成一个静态类。
	class A {
    	static class B {}
	}

	new A.B()
	// 匿名内部类
	import java.util.concurrent.CountDownLatch
	import java.util.concurrent.TimeUnit

	CountDownLatch called = new CountDownLatch(1)

	Timer timer = new Timer()
	timer.schedule(new TimerTask() {
 	   void run() {
 	       called.countDown()
 	   }
	}, 0)

	assert called.await(10, TimeUnit.SECONDS)
	// 创建非静态内部类的实例
	// 在java中如下
	public class Y {
    	public class X {}
   		 public X foo() {
     		   return new X();
   	 	}
   		 public static X createX(Y y) {
     		   return y.new X();
   		 }
	}
	// Groovy不支持y.new X（）语法。相反，你必须编写新的X（y），如下面的代码所示：
	public class Y {
   		 public class X {}
    		public X foo() {
       		 return new X()
   		 }
  		  public static X createX(Y y) {
      		  return new X(y)
   		 }
		}
	// 但是请注意，Groovy支持使用一个参数调用方法而不提供参数。 然后该参数的值为null。 基本上相同的规则适用于调用构造函数。 例如，您可能会编写新的X()而不是新的X(this)。 由于这也可能是常规方式，我们尚未找到防止此问题的好方法。		
  ```
  * Lambda表达式
  ```groovy
  	// Java8支持Lambda表达式和方法引用
  	Runnable run = () -> System.out.println("Run");
	list.forEach(System.out::println);
	// Java8Lambda表达式可以或多或少地被视为匿名内部类。 Groovy不支持该语法，但有闭包：
	Runnable run = { println 'run' }
	list.each { println it } // or list.each(this.&println)
  ```
  * GStrings
  ```txt
  	由于双引号字符串文字被解释为GString值，如果使用Groovy和Java编译器编译包含美元字符的字符串文字的类，Groovy可能会因编译错误而失败或产生微妙的不同代码,所以对于$可能需要使用"\"进行转义[GString](https://blog.csdn.net/asty9000/article/details/80299204)
  ```
  * 字符串和字符文字
  ```groovy 
  	// Groovy中单引号文字用于String，而双引号结果用于String或GString，具体取决于文字中是否有插值。例如
  	assert 'c'.getClass()==String
	assert "c".getClass()==String
	assert "c${1}".getClass() in GString
	// 只有在分配给char类型的变量时，Groovy才会自动将单字符String转换为char。当使用char类型的参数调用方法时，我们需要显式转换或确保已提前转换值,例如
	char a='a'
	assert Character.digit(a, 16)==10 : 'But Groovy does boxing'
	assert Character.digit((char) 'a', 16)==10

	try {
 	 assert Character.digit('a', 16)==10
 	 assert false: 'Need explicit cast'
	} catch(MissingMethodException e) {
	}
	// Groovy支持两种类型的转换，在转换为char时和在转换多字符串时存在细微差别。 Groovy样式转换更宽松，将采用第一个字符，而C-style转换将失败，异常。 例如
	// for single char strings, both are the same
	assert ((char) "c").class==Character
	assert ("c" as char).class==Character

	// for multi char strings they are not
	try {
	  ((char) 'cx') == 'c'
	  assert false: 'will fail - not castable'
	} catch(GroovyCastException e) {
	}
	assert ('cx' as char) == 'c'
	assert 'cx'.asType(char) == 'c'
  ```
  * 基源和包装
  ```groovy
  	// 因为Groovy将Objects用于所有内容，所以它会自动对基元进行引用。 因此，它不遵循Java的扩展行为优先于自动装箱和自动拆箱。 这是一个使用int的例子
  	int i
	m(i)

	// method1
	void m(long l) {           
 	 println "in m(long)"
	}
	// method2
	void m(Integer i) {        
 	 println "in m(Integer)"
	}
	// method1是Java可以调用的方法，因为扩展优先于自动拆箱
	// method2是Groovy实际调用的方法，因为所有原始引用都使用它们的包装类
  ```
  * ==的行为
  ```txt
  	在Java中==表示基本类型或对象标识的相等性。 在Groovy中==转换为a.compareTo(b)== 0，如果它们是Comparable,则转换为a.equals(b), 为了检查对象是否相等使用is。例如。a.is(b)
  ```
  * 额外关键字
  ```groovy
  	as
  	def
  	in
  	trait
  ```
