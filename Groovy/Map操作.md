1. Map
```groovy
	// 在Groovy中，可以使用地图文字语法创建地图(也称为关联数组):[:]
	def map = [name: 'Gromit', likes: 'cheese', id: 1234]
	assert map.get('name') == 'Gromit'
	assert map.get('id') == 1234
	assert map['name'] == 'Gromit'
	assert map['id'] == 1234
	assert map instanceof java.util.Map

	def emptyMap = [:]
	assert emptyMap.size() == 0
	emptyMap.put("foo", 5)
	assert emptyMap.size() == 1
	assert emptyMap.get("foo") == 5
	// 默认情况下，映射键是字符串:[a:1]等同于['a':1]。如果您定义名为a的变量并且希望a的值成为Map中的键，则可能会造成混淆。 如果是这种情况,则必须通过添加括号来转义键，如下例所示
	def a = 'Bob'
	def ages = [a: 43]
	assert ages['Bob'] == null // `Bob` is not found
	assert ages['a'] == 43     // because `a` is a literal!

	ages = [(a): 43]            // now we escape `a` by using parenthesis
	assert ages['Bob'] == 43   // and the value is found!
	// 除了Map之外，还可以获取Map的新副本来克隆它
	def map = [
	        simple : 123,
	        complex: [a: 1, b: 2]
	]
	def map2 = map.clone()
	assert map2.get('simple') == map.get('simple')
	assert map2.get('complex') == map.get('complex')
	map2.get('complex').put('c', 3)
	assert map.get('complex').get('c') == 3
	// 生成的Map是原始Map的浅表副本,如上例所示。
	// 映射属性表示法
	// Map也像bean一样，所以你可以使用属性表示法来获取/设置Map中的项目，只要这些键是有效的Groovy标识符的字符串
	def map = [name: 'Gromit', likes: 'cheese', id: 1234]
	assert map.name == 'Gromit'     // can be used instead of map.get('name')
	assert map.id == 1234

	def emptyMap = [:]
	assert emptyMap.size() == 0
	emptyMap.foo = 5
	assert emptyMap.size() == 1
	assert emptyMap.foo == 5
	// 注意:按照设计map.foo将始终在地图中查找关键字foo。这意味着foo.class将在不包含类键的映射上返回null。如果你真的想知道这个类,那么你必须使用getClass()
	def map = [name: 'Gromit', likes: 'cheese', id: 1234]
	assert map.class == null
	assert map.get('class') == null
	assert map.getClass() == LinkedHashMap // this is probably what you want

	map = [1      : 'a',
	       (true) : 'p',
	       (false): 'q',
	       (null) : 'x',
	       'null' : 'z']
	assert map.containsKey(1) // 1 is not an identifier so used as is
	assert map.true == null
	assert map.false == null
	assert map.get(true) == 'p'
	assert map.get(false) == 'q'
	assert map.null == 'z'
	assert map.get(null) == 'x'
```
2. 遍历
```groovy
	// 像往常一样在Groovy开发工具包中,Map上的惯用迭代使用了each和eachWithIndex方法.值得注意的是,使用Map文字符号创建的Map是有序的,也就是说,如果您迭代Map元素,则可以保证元素将按照它们在Map中添加的顺序返回。
	def map = [
	        Bob  : 42,
	        Alice: 54,
	        Max  : 33
	]

	// `entry` is a map entry
	map.each { entry ->
	    println "Name: $entry.key Age: $entry.value"
	}

	// `entry` is a map entry, `i` the index in the map
	map.eachWithIndex { entry, i ->
	    println "$i - Name: $entry.key Age: $entry.value"
	}

	// Alternatively you can use key and value directly
	map.each { key, value ->
	    println "Name: $key Age: $value"
	}

	// Key, value and i as the index in the map
	map.eachWithIndex { key, value, i ->
	    println "$i - Name: $key Age: $value"
	}
```
3. Map操作
```groovy
	// 添加删除元素
	// 通过put或者putAll添加元素
	def defaults = [1: 'a', 2: 'b', 3: 'c', 4: 'd']
	def overrides = [2: 'z', 5: 'x', 13: 'x']

	def result = new LinkedHashMap(defaults)
	result.put(15, 't')
	result[17] = 'u'
	result.putAll(overrides)
	assert result == [1: 'a', 2: 'z', 3: 'c', 4: 'd', 5: 'x', 13: 'x', 15: 't', 17: 'u']
	// 删除所有元素
	def m = [1:'a', 2:'b']
	assert m.get(1) == 'a'
	m.clear()
	assert m == [:]
	// 使用map literal语法生成的Map使用对象equals和hashcode方法。这意味着您永远不应该使用哈希代码随时间变化的对象,否则您将无法获得相关的值。值得注意的是,您永远不应该使用GString作为映射的键，因为GString的哈希码与等效String的哈希码不同
	def key = 'some key'
	def map = [:]
	def gstringKey = "${key.toUpperCase()}"
	map.put(gstringKey,'value')
	assert map.get('SOME KEY') == null

	// 键 值 实体
	def map = [1:'a', 2:'b', 3:'c']

	def entries = map.entrySet()
	entries.each { entry ->
	  assert entry.key in [1,2,3]
	  assert entry.value in ['a','b','c']
	}

	def keys = map.keySet()
	assert keys == [1,2,3] as Set
	// 由于操作的成功直接取决于被操作的Map的类型,因此强烈建议不要对Map返回的值(无论是Map元素,键还是值)进行变换.特别是,Groovy依赖于JDK的集合,通常不能保证可以通过keySet,entrySet或值安全地操作集合。

	// 过滤和搜索
	def people = [
	    1: [name:'Bob', age: 32, gender: 'M'],
	    2: [name:'Johnny', age: 36, gender: 'M'],
	    3: [name:'Claire', age: 21, gender: 'F'],
	    4: [name:'Amy', age: 54, gender:'F']
	]

	def bob = people.find { it.value.name == 'Bob' } // find a single entry
	def females = people.findAll { it.value.gender == 'F' }

	// both return entries, but you can use collect to retrieve the ages for example
	def ageOfBob = bob.value.age
	def agesOfFemales = females.collect {
	    it.value.age
	}

	assert ageOfBob == 32
	assert agesOfFemales == [21,54]

	// but you could also use a key/pair value as the parameters of the closures
	def agesOfMales = people.findAll { id, person ->
	    person.gender == 'M'
	}.collect { id, person ->
	    person.age
	}
	assert agesOfMales == [32, 36]

	// `every` returns true if all entries match the predicate
	assert people.every { id, person ->
	    person.age > 18
	}

	// `any` returns true if any entry matches the predicate

	assert people.any { id, person ->
	    person.age == 54
	}

	// 分组
	assert ['a', 7, 'b', [2, 3]].groupBy {
	    it.class
	} == [(String)   : ['a', 'b'],
	      (Integer)  : [7],
	      (ArrayList): [[2, 3]]
	]

	assert [
	        [name: 'Clark', city: 'London'], [name: 'Sharma', city: 'London'],
	        [name: 'Maradona', city: 'LA'], [name: 'Zhang', city: 'HK'],
	        [name: 'Ali', city: 'HK'], [name: 'Liu', city: 'HK'],
	].groupBy { it.city } == [
	        London: [[name: 'Clark', city: 'London'],
	                 [name: 'Sharma', city: 'London']],
	        LA    : [[name: 'Maradona', city: 'LA']],
	        HK    : [[name: 'Zhang', city: 'HK'],
	                 [name: 'Ali', city: 'HK'],
	                 [name: 'Liu', city: 'HK']],
	]
```