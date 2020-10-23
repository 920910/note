Ranges允许您创建顺序值列表,这些可以用作List,因为Range扩展了java.util.List。
使用..表示法定义的范围是包含的(即列表包含from和to值)
使用..<表示法定义的范围是半开的,它们包括第一个值但不包括最后一个值
```groovy
	// an inclusive range
	def range = 5..8
	assert range.size() == 4
	assert range.get(2) == 7
	assert range[2] == 7
	assert range instanceof java.util.List
	assert range.contains(5)
	assert range.contains(8)

	// lets use a half-open range
	range = 5..<8
	assert range.size() == 3
	assert range.get(2) == 7
	assert range[2] == 7
	assert range instanceof java.util.List
	assert range.contains(5)
	assert !range.contains(8)

	//get the end points of the range without using indexes
	range = 1..10
	assert range.from == 1
	assert range.to == 10
	// Range可以用于任何实现java.lang.Comparable以进行比较的Java对象,并且还具有方法next()和previous()以返回范围中的下一个/上一个项目。例如,您可以创建一系列String元素：
	// an inclusive range
	def range = 'a'..'d'
	assert range.size() == 4
	assert range.get(2) == 'c'
	assert range[2] == 'c'
	assert range instanceof java.util.List
	assert range.contains('a')
	assert range.contains('d')
	assert !range.contains('e')
	// 可以用在for循环的迭代中
	for (i in 1..10) {
	    println "Hello ${i}"
	}
	// 更具Groovy风格的写法
	(1..10).each { i ->
	    println "Hello ${i}"
	}
	// 在switch中使用
	switch (years) {
	    case 1..10: interestRate = 0.076; break;
	    case 11..25: interestRate = 0.052; break;
	    default: interestRate = 0.037;
	}
```