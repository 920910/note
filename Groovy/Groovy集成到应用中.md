1. Eval
```java
	// 只支持单行命令
	 System.out.println(Eval.me("\"foo\".toUpperCase()"));

     System.out.println(Eval.me("33*3"));
     System.out.println(Eval.x(4, "2*x"));
     System.out.println(Eval.me("k", 4, "2*k"));
     System.out.println(Eval.xy(4, 5, "x*y"));
     System.out.println(Eval.xyz(4, 5, 6, "x*y+z"));

```
2. GroovyShell
 * 多行数据	
```java
	// groovy.lang.GroovyShell类是评估脚本的首选方法，它能够缓存生成的脚本实例。 虽然Eval类返回已编译脚本的执行结果，但GroovyShell类提供了更多选项
	GroovyShell shell = new GroovyShell();
    System.out.println(shell.evaluate("3*5"));
    System.out.println(shell.evaluate(new StringReader("3*5")));
    Script script = shell.parse("3*5");
    System.out.println(script instanceof groovy.lang.Script);
    System.out.println(script.run());	
```
 * 在脚本和应用中传值
```java
 	// 通过setProperties写入数据
  	Binding sharedData = new Binding();
  	GroovyShell shell = new GroovyShell(sharedData);
    Date now = new Date();
    sharedData.setProperty("date", now);

    shell.evaluate("println(date)");
    // 通过脚本写入数据
    Binding sharedData = new Binding();
    GroovyShell shell = new GroovyShell(sharedData);
    shell.evaluate("date=new Date()");

    shell.evaluate("println(date)");
    // 重要的是要了解如果要写入绑定，则需要使用未声明的变量。使用def或类似下面示例中的显式类型会失败，因为您将创建一个局部变量
     Binding sharedData = new Binding();
     GroovyShell shell = new GroovyShell(sharedData);
     shell.evaluate("int foo=123");

     try {
         System.out.println(sharedData.getProperty("foo"));;
     } catch (MissingPropertyException e) {
         System.out.println("foo is defined as a local variable");
     }
     // 在多线程环境中使用共享数据时必须非常小心。传递给GroovyShell的Binding实例不是线程安全的，并且由所有脚本共享。
     // 通过脚本实例和Binding实例实现数据共享
     GroovyShell shell = new GroovyShell();
     Map<String, Integer> map1 = new HashMap<>(1);
     Map<String, Integer> map2 = new HashMap<>(1);
     map1.put("x", 3);
     map2.put("x", 4);
     Binding b1 = new Binding(map1);
     Binding b2 = new Binding(map2);
     Script script = shell.parse("x = 2*x");
     script.setBinding(b1);
     script.run();
     script.setBinding(b2);
     script.run();
     System.out.println((Integer) b1.getProperty("x") == 6);
     System.out.println((Integer) b2.getProperty("x") == 8);
     System.out.println(b1 != b2);
    // 但是,您必须知道您仍在共享相同的脚本实例。因此,如果您有两个线程处理同一个脚本,则无法使用此技术。在这种情况下,您必须确保创建两个不同的脚本实例
     GroovyShell shell = new GroovyShell();
     Map<String, Integer> map1 = new HashMap<>(1);
     Map<String, Integer> map2 = new HashMap<>(1);
     map1.put("x", 3);
     map2.put("x", 4);
     Binding b1 = new Binding(map1);
     Binding b2 = new Binding(map2);
     Script script1 = shell.parse("x = 2*x");
     Script script2 = shell.parse("x = 2*x");
     System.out.println(script1 != script2);
     script1.setBinding(b1);
     script2.setBinding(b2);
     Thread t1 = new Thread(() -> script1.run());
     Thread t2 = new Thread(() -> script2.run());
     t1.start();
     t2.start();
     t1.join();
     t2.join();
     System.out.println((Integer) b1.getProperty("x") == 6);
     System.out.println((Integer) b2.getProperty("x") == 8);
     System.out.println(b1 != b2);
    // 如果你需要这里的线程安全，建议直接使用GroovyClassLoader
```
 * 自定义脚本类
```java
    // 我们已经看到parse方法返回一个groovy.lang.Script的实例，但是可以使用自定义类，因为它扩展了Script本身。它可用于为脚本提供其他行为，如下例所示：

```
 * GroovyClassLoader
```groovy
    // 在上一节中，我们已经证明GroovyShell是一个执行脚本的简单工具，但它使编译除脚本之外的任何东西变得复杂。 在内部，它使用了groovy.lang.GroovyClassLoader，它是运行时编译和加载类的核心。
    def gcl = new GroovyClassLoader()
    def clazz = gcl.parseClass('class Foo { void doIt() { println "ok" } }')
    assert clazz.name == 'Foo'
    def o = clazz.newInstance()
    o.doIt()
    // GroovyClassLoader保留了它创建的所有类的引用,因此很容易造成内存泄漏。特别是,如果您执行两次相同的脚本,如果它是一个String，那么您将获得两个不同的类!
    // 原因是GroovyClassLoader不跟踪源文本。如果要拥有相同的实例,则源必须是文件,如下例所示
    def gcl = new GroovyClassLoader()
    def clazz1 = gcl.parseClass(file)                                           
    def clazz2 = gcl.parseClass(new File(file.absolutePath))                    
    assert clazz1.name == 'Foo'                                                 
    assert clazz2.name == 'Foo'
    assert clazz1 == clazz2   
```
 * GroovyScriptEngine
```groovy
    // groovy.util.GroovyScriptEngine类为依赖脚本重新加载和脚本依赖性的应用程序提供了灵活的基础。虽然GroovyShell专注于独立脚本和`GroovyClassLoader处理任何Groovy类的动态编译和加载，但GroovyScriptEngine将在GroovyClassLoader之上添加一个层来处理脚本依赖和重新加载。
    // ReloadingTest.groovy
    class Greeter {
        String sayHello() {
            def greet = "Hello, world!"
            greet
        }
    }

    new Greeter()
    // Test.groovy
    def binding = new Binding()
    def engine = new GroovyScriptEngine([tmpDir.toURI().toURL()] as URL[])          

    while (true) {
        def greeter = engine.run('ReloadingTest.groovy', binding)                   
        println greeter.sayHello()                                                  
        Thread.sleep(1000)
    }
    // 修改def greet = "Hello, world!"则会直接改变输出
```


