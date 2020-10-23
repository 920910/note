Groovy支持List,Map,Ranges.大多数都是基于java类型，并使用Groovy开发工具包中的其他方法进行修饰
1. List
```groovy
	// 您可以按如下方式创建列表。请注意，[]是空List表达式。
	// 每个List表达式都创建了java.util.List的实现
	def list = [5, 6, 7, 8]
	assert list.get(2) == 7
	assert list[2] == 7
	assert list instanceof java.util.List

	def emptyList = []
	assert emptyList.size() == 0
	emptyList.add(5)
	assert emptyList.size() == 1
	// 通过构造器获取list
	def list1 = ['a', 'b', 'c']
	//construct a new list, seeded with the same items as in list1
	def list2 = new ArrayList<String>(list1)

	assert list2 == list1 // == checks that each corresponding element is the same

	// clone() can also be called
	def list3 = list1.clone()
	assert list3 == list1
	// List的常规操作
	def list = [5, 6, 7, 8]
	assert list.size() == 4
	assert list.getClass() == ArrayList     // the specific kind of list being used

	assert list[2] == 7                     // indexing starts at 0
	assert list.getAt(2) == 7               // equivalent method to subscript operator []
	assert list.get(2) == 7                 // alternative method

	list[2] = 9
	assert list == [5, 6, 9, 8,]           // trailing comma OK

	list.putAt(2, 10)                       // equivalent method to [] when value being changed
	assert list == [5, 6, 10, 8]
	assert list.set(2, 11) == 10            // alternative method that returns old value
	assert list == [5, 6, 11, 8]

	assert ['a', 1, 'a', 'a', 2.5, 2.5f, 2.5d, 'hello', 7g, null, 9 as byte]
	//objects can be of different types; duplicates allowed

	assert [1, 2, 3, 4, 5][-1] == 5             // use negative indices to count from the end
	assert [1, 2, 3, 4, 5][-2] == 4
	assert [1, 2, 3, 4, 5].getAt(-2) == 4       // getAt() available with negative index...
	try {
	    [1, 2, 3, 4, 5].get(-2)                 // but negative index not allowed with get()
	    assert false
	} catch (e) {
	    assert e instanceof IndexOutOfBoundsException
	}
```
2. List作为布尔表达式
```groovy
	assert ![]             // an empty list evaluates as false

	//all other lists, irrespective of contents, evaluate as true
	assert [1] && ['a'] && [0] && [0.0] && [false] && [null]
```
3. List的遍历
```groovy
	// 迭代列表的元素通常是调用each和eachWithIndex方法,这些方法在列表的每个项目上执行代码:
	[1, 2, 3].each {
	    println "Item: $it" // `it` is an implicit parameter corresponding to the current element
	}
	['a', 'b', 'c'].eachWithIndex { it, i -> // `it` is the current element, while `i` is the index
	    println "$i: $it"
	}
	// 除了迭代之外，通过将每个元素转换为其他元素来创建新列表通常很有用。由于collect方法，这个操作（通常称为映射）在Groovy中完成
	assert [1, 2, 3].collect { it * 2 } == [2, 4, 6]

	// shortcut syntax instead of collect
	assert [1, 2, 3]*.multiply(2) == [1, 2, 3].collect { it.multiply(2) }

	def list = [0]
	// it is possible to give `collect` the list which collects the elements
	assert [1, 2, 3].collect(list) { it * 2 } == [0, 2, 4, 6]
	assert list == [0, 2, 4, 6]
```
4. List操作
```groovy
	// 过滤和搜索
	assert [1, 2, 3].find { it > 1 } == 2           // find 1st element matching criteria
	assert [1, 2, 3].findAll { it > 1 } == [2, 3]   // find all elements matching critieria
	assert ['a', 'b', 'c', 'd', 'e'].findIndexOf {      // find index of 1st element matching criteria
	    it in ['c', 'e', 'g']
	} == 2

	assert ['a', 'b', 'c', 'd', 'c'].indexOf('c') == 2  // index returned
	assert ['a', 'b', 'c', 'd', 'c'].indexOf('z') == -1 // index -1 means value not in list
	assert ['a', 'b', 'c', 'd', 'c'].lastIndexOf('c') == 4

	assert [1, 2, 3].every { it < 5 }               // returns true if all elements match the predicate
	assert ![1, 2, 3].every { it < 3 }
	assert [1, 2, 3].any { it > 2 }                 // returns true if any element matches the predicate
	assert ![1, 2, 3].any { it > 3 }

	assert [1, 2, 3, 4, 5, 6].sum() == 21                // sum anything with a plus() method
	assert ['a', 'b', 'c', 'd', 'e'].sum {
	    it == 'a' ? 1 : it == 'b' ? 2 : it == 'c' ? 3 : it == 'd' ? 4 : it == 'e' ? 5 : 0
	    // custom value to use in sum
	} == 15
	assert ['a', 'b', 'c', 'd', 'e'].sum { ((char) it) - ((char) 'a') } == 10
	assert ['a', 'b', 'c', 'd', 'e'].sum() == 'abcde'
	assert [['a', 'b'], ['c', 'd']].sum() == ['a', 'b', 'c', 'd']

	// an initial value can be provided
	assert [].sum(1000) == 1000
	assert [1, 2, 3].sum(1000) == 1006

	assert [1, 2, 3].join('-') == '1-2-3'           // String joining
	assert [1, 2, 3].inject('counting: ') {
	    str, item -> str + item                     // reduce operation
	} == 'counting: 123'
	assert [1, 2, 3].inject(0) { count, item ->
	    count + item
	} == 6	
	// 这里有一个惯用的Groovy代码，用于查找集合中的最大值和最小值
	def list = [9, 4, 2, 10, 5]
	assert list.max() == 10
	assert list.min() == 2

	// we can also compare single characters, as anything comparable
	assert ['x', 'y', 'a', 'z'].min() == 'a'

	// we can use a closure to specify the sorting behaviour
	def list2 = ['abc', 'z', 'xyzuvw', 'Hello', '321']
	assert list2.max { it.size() } == 'xyzuvw'
	assert list2.min { it.size() } == 'z'
	// 除了闭包之外，您还可以使用Comparator来定义比较条件
	Comparator mc = { a, b -> a == b ? 0 : (a < b ? -1 : 1) }

	def list = [7, 4, 9, -6, -1, 11, 2, 3, -9, 5, -13]
	assert list.max(mc) == 11
	assert list.min(mc) == -13

	Comparator mc2 = { a, b -> a == b ? 0 : (Math.abs(a) < Math.abs(b)) ? -1 : 1 }


	assert list.max(mc2) == -13
	assert list.min(mc2) == -1

	assert list.max { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -13
	assert list.min { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -1

	// 添加和删除元素
	// 我们可以使用[]分配一个新的空列表，然后使用<<来添加元素
	def list = []
	assert list.empty

	list << 5
	assert list.size() == 1

	list << 7 << 'i' << 11
	assert list == [5, 7, 'i', 11]

	list << ['m', 'o']
	assert list == [5, 7, 'i', 11, ['m', 'o']]

	//first item in chain of << is target list
	assert ([1, 2] << 3 << [4, 5] << 6) == [1, 2, 3, [4, 5], 6]

	//using leftShift is equivalent to using <<
	assert ([1, 2, 3] << 4) == ([1, 2, 3].leftShift(4))
	// 其他添加方法
	assert [1, 2] + 3 + [4, 5] + 6 == [1, 2, 3, 4, 5, 6]
	// equivalent to calling the `plus` method
	assert [1, 2].plus(3).plus([4, 5]).plus(6) == [1, 2, 3, 4, 5, 6]

	def a = [1, 2, 3]
	a += 4      // creates a new list and assigns it to `a`
	a += [5, 6]
	assert a == [1, 2, 3, 4, 5, 6]

	assert [1, *[222, 333], 456] == [1, 222, 333, 456]
	assert [*[1, 2, 3]] == [1, 2, 3]
	assert [1, [2, 3, [4, 5], 6], 7, [8, 9]].flatten() == [1, 2, 3, 4, 5, 6, 7, 8, 9]

	def list = [1, 2]
	list.add(3)
	list.addAll([5, 4])
	assert list == [1, 2, 3, 5, 4]

	list = [1, 2]
	list.add(1, 3) // add 3 just before index 1
	assert list == [1, 3, 2]

	list.addAll(2, [5, 4]) //add [5,4] just before index 2
	assert list == [1, 3, 5, 4, 2]

	list = ['a', 'b', 'z', 'e', 'u', 'v', 'g']
	list[8] = 'x' // the [] operator is growing the list as needed
	// nulls inserted if required
	assert list == ['a', 'b', 'z', 'e', 'u', 'v', 'g', null, 'x']
	// 也可以通过将其索引传递给remove方法来删除元素，在这种情况下，列表会发生改变
	def list = ['a','b','c','d','e','f','b','b','a']
	assert list.remove(2) == 'c'        // remove the third element, and return it
	assert list == ['a','b','d','e','f','b','b','a']
	// 如果您只想删除列表中具有相同值的第一个元素，而不是删除所有元素，则可以调用remove方法传递值
	def list= ['a','b','c','b','b']
	assert list.remove('c')             // remove 'c', and return true because element removed
	assert list.remove('b')             // remove first 'b', and return true because element removed

	assert ! list.remove('z')           // return false because no elements removed
	assert list == ['a','b','b']
	// 如您所见，有两种可用的删除方法。 取一个整数并通过索引删除元素，另一个取消匹配传递值的第一个元素。 那么当我们有一个整数列表时我们应该怎么办？ 在这种情况下，您可能希望使用removeAt通过其索引删除元素，并使用removeElement删除与值匹配的第一个元素。
	def list = [1,2,3,4,5,6,2,2,1]

	assert list.remove(2) == 3          // this removes the element at index 2, and returns it
	assert list == [1,2,4,5,6,2,2,1]

	assert list.removeElement(2)        // remove first 2 and return true
	assert list == [1,4,5,6,2,2,1]

	assert ! list.removeElement(8)      // return false because 8 is not in the list
	assert list == [1,4,5,6,2,2,1]

	assert list.removeAt(1) == 4        // remove element at index 1, and return it
	assert list == [1,5,6,2,2,1]
	// 当然，removeAt和removeElement将适用于任何类型的列表
	// 删除所有元素
	def list= ['a',2,'c',4]
	list.clear()
	assert list == []
	// 集合操作
	assert 'a' in ['a','b','c']             // returns true if an element belongs to the list
	assert ['a','b','c'].contains('a')      // equivalent to the `contains` method in Java
	assert [1,3,4].containsAll([1,4])       // `containsAll` will check that all elements are found

	assert [1,2,3,3,3,3,4,5].count(3) == 4  // count the number of elements which have some value
	assert [1,2,3,3,3,3,4,5].count {
	    it%2==0                             // count the number of elements which match the predicate
	} == 2

	assert [1,2,4,6,8,10,12].intersect([1,3,6,9,12]) == [1,6,12]

	assert [1,2,3].disjoint( [4,6,9] )
	assert ![1,2,3].disjoint( [2,4,6] )	
	//排序
	assert [6, 3, 9, 2, 7, 1, 5].sort() == [1, 2, 3, 5, 6, 7, 9]

	def list = ['abc', 'z', 'xyzuvw', 'Hello', '321']
	assert list.sort {
	    it.size()
	} == ['z', 'abc', '321', 'Hello', 'xyzuvw']

	def list2 = [7, 4, -6, -1, 11, 2, 3, -9, 5, -13]
	assert list2.sort { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } ==
	        [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

	Comparator mc = { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 }

	// JDK 8+ only
	// list2.sort(mc)
	// assert list2 == [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

	def list3 = [6, -3, 9, 2, -7, 1, 5]

	Collections.sort(list3)
	assert list3 == [-7, -3, 1, 2, 5, 6, 9]

	Collections.sort(list3, mc)
	assert list3 == [1, 2, -3, 5, 6, -7, 9]
	// 重复元素
	assert [1, 2, 3] * 3 == [1, 2, 3, 1, 2, 3, 1, 2, 3]
	assert [1, 2, 3].multiply(2) == [1, 2, 3, 1, 2, 3]
	assert Collections.nCopies(3, 'b') == ['b', 'b', 'b']

	// nCopies from the JDK has different semantics than multiply for lists
	assert Collections.nCopies(2, [1, 2]) == [[1, 2], [1, 2]] //not [1,2,1,2]
```