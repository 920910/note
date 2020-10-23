Groovy提供了许多辅助方法来处理I/O.虽然您可以使用Groovy中的标准Java代码来处理这些代码，但Groovy提供了更方便的方法来处理文件，流，读取器......
* [java.io.File](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/File.html)
* [java.io.InputStream](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html)
* [java.io.OutputStream](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/OutputStream.html)
* [java.io.Reader](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Reader.html)
* [java.io.Writer](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Writer.html)
* [java.nio.file.Path](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/nio/file/Path.html)
1. 读文件
 ```groovy
 	new File(baseDir, 'haiku.txt').eachLine { line ->
    	println line
	}
	//eachLine方法是由Groovy自动添加到File类的方法,并且有许多变体,例如,如果您需要知道行号,则可以使用此变体:
	new File(baseDir, 'haiku.txt').eachLine { line, nb ->
    	println "Line $nb: $line"
	}
	// 如果由于某种原因在eachLine主体中抛出异常,则该方法确保正确关闭资源。对于Groovy添加的所有I/O资源方法都是如此。
	// 如果您需要将文本文件的行收集到列表中,您可以执行以下操作
	def list = new File(baseDir, 'haiku.txt').collect {it}
	// 甚至可以利用as运算符将文件的内容转换为行数组:
	def array = new File(baseDir, 'haiku.txt') as String[]
	// 将文件转换成byte数组
	byte[] contents = file.bytes
	// 从文件中获取输入流
	def is = new File(baseDir,'haiku.txt').newInputStream()
	// do something ...
	is.close()
	// 但是你可以看到它需要你处理关闭输入流。在Groovy中,使用withInputStream更方便,它将为您处理流的关闭
	new File(baseDir,'haiku.txt').withInputStream { stream ->
   		 // do something ...
	}
 ```
 2. 写文件
  ```groovy
	new File(baseDir,'haiku.txt').withWriter('utf-8') { writer ->
    	writer.writeLine 'Into the ancient pond'
    	writer.writeLine 'A frog jumps'
    	writer.writeLine 'Water’s sound!'
	}
	// 也可以使用<<操作符，例如
	new File(baseDir,'haiku.txt') << '''Into the ancient pond
	A frog jumps
	Water’s sound!'''
	// 向文中写入字节
	file.bytes = [66,22,11]
	// 从文件中获取输出流
	def os = new File(baseDir,'data.bin').newOutputStream()
	// do something ...
	os.close()
	// 但是你可以看到它需要你处理关闭输出流。在Groovy中,使用withOutputStream更方便,它将为您处理流的关闭
	new File(baseDir,'data.bin').withOutputStream { stream ->
   		 // do something ...
	}
  ```
  3. 遍历文件树
  ```txt
  	// 在脚本上下文中,遍历文件树以查找某些特定文件并对其执行某些操作是一项常见任务。 Groovy提供了多种方法来完成此任务。例如,您可以对目录的所有文件执行某些操作:
  	// 在目录中找到的每个文件上执行闭包代码
  	dir.eachFile { file ->                      
   	 	println file.name
	}
	// 对与指定模式匹配的目录中的文件执行闭包代码
	dir.eachFileMatch(~/.*\.txt/) { file ->     
   		 println file.name
	}
	// 通常,您将不得不处理更深层次的文件,在这种情况下,您可以使用eachFileRecurse
	// 以递归方式在目录中找到的每个文件或目录上执行闭包代码
	dir.eachFileRecurse { file ->                      
    	println file.name
	}
	// 仅对文件执行闭包代码，但递归执行
	dir.eachFileRecurse(FileType.FILES) { file ->      
   	 println file.name
	}
	// 对于更复杂的遍历技术,您可以使用traverse遍历方法,该方法要求您设置一个特殊标志,指示如何处理遍历
	dir.traverse { file ->
    	if (file.directory && file.name=='bin') {
    		// 如果当前文件是目录且其名称为bin，则停止遍历
     	   FileVisitResult.TERMINATE                   
   		 } else {
      	  println file.name
      		  // 否则打印文件名并继续
     	   FileVisitResult.CONTINUE                    
   		 }

	}
  ```
  4. 数据和对象
  ```groovy
  	// 在Java中,分别使用java.io.DataOutputStream和java.io.DataInputStream类来序列化和反序列化数据并不罕见.Groovy将使它更容易处理它们.例如,您可以将数据序列化为文件并使用以下代码对其进行反序列化
  	boolean b = true
	String message = 'Hello from Groovy'
	// Serialize data into a file
	file.withDataOutputStream { out ->
    	out.writeBoolean(b)
    	out.writeUTF(message)
	}
	// ...
	// Then read it back
	file.withDataInputStream { input ->
    	assert input.readBoolean() == b
    	assert input.readUTF() == message
	}
	// 同样,如果要序列化的数据实现Serializable接口,则可以继续使用对象输出流，如下所示
	Person p = new Person(name:'Bob', age:76)
	// Serialize data into a file
	file.withObjectOutputStream { out ->
  	 	 out.writeObject(p)
	}
	// ...
	// Then read it back
	file.withObjectInputStream { input ->
   		 def p2 = input.readObject()
   		 assert p2.name == p.name
   		 assert p2.age == p.age
	}
  ```
  5. 执行外部进程
  ```groovy
  	// 执行命令行命令
  	// 在外部进程中执行ls命令
  	def process = "ls -l".execute()             
  	// 使用命令的输出并检索文本
	println "Found text ${process.text}" 
	// execute方法返回一个java.lang.Process的实例并允处理输入/输出/错误流以及检查进程的退出值等
	// 在外部进程中执行ls命令
	def process = "ls -l".execute()   
	// 遍历流中的每一行          
	process.in.eachLine { line ->               
		// 打印行
    	println line                            
	}
	// 值得注意的是,对应于命令的标准输出的输入流.out将引用一个流,可以将数据发送到进程（其标准输入
	// 请记住,许多命令都是shell内置函数,需要特殊处理。因此,如果您想在Windows计算机上的目录中列出文件并写入
	def process = "dir".execute()
	println "${process.text}"
	// 你会收到一个IOException,说不能运行程序"dir":CreateProcess error = 2,系统找不到指定的文件。这是因为dir内置于Windows shell（cmd.exe）无法作为简单的可执行文件运行。 相反,你需要写
	def process = "cmd /c dir".execute()
	println "${process.text}"
	// 此外,由于此功能目前使用java.lang.Process不安全,因此必须考虑该类的缺陷。
	// Groovy提供了更加便捷的处理方式
	//这里也是使用StringBuffer，InputStream，OutputStream等的consumeProcessOutput的变种......
	def p = "rm -f foo.tmp".execute([], tmpDir)
	p.consumeProcessOutput()
	p.waitFor()
	// 另外，这些是一个pipeTo命令（映射到|允许重载），它允许一个进程的输出流被输入另一个进程的输入流。
	proc1 = 'ls'.execute()
	proc2 = 'tr -d o'.execute()
	proc3 = 'tr -d e'.execute()
	proc4 = 'tr -d i'.execute()
	proc1 | proc2 | proc3 | proc4
	proc4.waitFor()
	if (proc4.exitValue()) {
  	 	 println proc4.err.text
	} else {
  	  	println proc4.text
	}
	// 处理异常
	def sout = new StringBuilder()
	def serr = new StringBuilder()
	proc2 = 'tr -d o'.execute()
	proc3 = 'tr -d e'.execute()
	proc4 = 'tr -d i'.execute()
	proc4.consumeProcessOutput(sout, serr)
	proc2 | proc3 | proc4
	[proc2, proc3].each { it.consumeProcessErrorStream(serr) }
	proc2.withWriter { writer ->
   		 writer << 'testfile.groovy'
	}
	proc4.waitForOrKill(1000)
	println "Standard output: $sout"
	println "Standard error: $serr
  ```



