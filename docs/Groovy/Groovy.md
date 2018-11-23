# Groovy 

## groovy 基础语法

- **变量类型**

```
# 强类型定义
int x = 10
println x.class
```

结果：class java.lang.Integer

```
double y = 3.14
println y.class
```

结果：class java.lang.Double

**groovy 中没有基本类型变量，所有类型都是对象类型**

- **变量的定义**

```

# def 弱类型定义
def x = 11
println x.class

def y = 3.1415
println y.class

def name = 'Android'
println name.class
```

输出结果：

```
class java.lang.Integer
class java.math.BigDecimal
class java.lang.String
```

---

- **字符串 GString**

单引号字符串不能扩展

```
def name = 'a single string'
println name.class

# 转义符
def str = ' \'a\' string '
println str.class

# 可以换行 不用 + 号 拼接
def thupleName = '''three signle string'''
println thupleName
```

输出结果：

```
class java.lang.String
class java.lang.String
three signle string
```

---

双引号字符串可扩展

```
def name = "java"
def sayHello = "Hello: ${name}"

println sayHello
println sayHello.class
```

```
# ${} 中可以是任意表达式
def sum = "The sum of 2 and 3 equals ${2 + 3}"

println sum
```

输出结果：

```
Hello: java
class org.codehaus.groovy.runtime.GStringImpl

The sum of 2 and 3 equals 5
```

---

字符串的方法

```
def str = "groovy"

# 填充字符串,以str为中心，两边都填充
str.center(8,a)

# 以str为中心，向左边都填充
str.padLeft(8,a)

# 比较字符串大小
str1 > str2

# 获取字符串中的字符
str[1]

# 获取字符串中下标为 0 到 1 的字符
str[1..3]

# 字符串中减去参数中相同的部分
str.minus(str2)

# 字符串倒序
str.reverse()

# 字符串首字母大写
str.capitalize()

# 判断字符串是否是数字类型的字符串
str.inNumber()

# 转换成基本类型数据
str.toInteger()
```

---

- 逻辑控制

对范围的 for 循环

```
def sum = 0
for(i in 0..9){
	sum += i
}

```

对 List 的循环

```
def sum = 0

for(i in [1,2,3,4,5,6,7]){
	sum += i
}
```

对 Map 进行循环

```
for(i in ['lili': 1 , 'luck': 2 , 'xiaoming': 3]){
	sum += i
}
```

## groovy 闭包

- 定义，调用

```
// 定义一个闭包
def clouser = { println 'hello groovy!' }

// 调用闭包 方式一
clouser.call()
// 调用闭包 方式二
clouser()
```

- 闭包参数

```
// { 参数 -> 闭包体 }
def clouser = { String name -> println 'hello ${name}' }

// 调用闭包 方式一
clouser.call('java!')
// 调用闭包 方式二
clouser('go!')

// 多个参数
def clouser = { String name, int age -> println 'hello ${name}, My age is ${age}' }

def name = "python"
clouser(name,5)

// 不声明参数（隐式参数,默认 it 接收）
def clouser = { -> println 'hello ${it}' }

clouser("java!")
```

- 闭包一定会有返回值

```
// 接收闭包返回值
def clouser = { -> return 'hello ${it}' }

def result = clouser("java!")
```

---

- 闭包用于普通类型结合使用


```
int x = fab(10)
println x

// 求指定数的 阶乘
int fab(int number) {
	int result = 1
	1.upto(number , { num -> result *= num })
	return result
}

// 求阶乘
int fab2(int number) {
	int result = 1
	number.downto(1) {
		num -> result *= num
	}
	return result
}

// 从 1 加到 100
int x = cal(101)

int cal(int number) {
	int result = 0
	number.times {
		num -> result += num
	}
	return result
}
```

---

- 闭包结合字符串的使用

```
// each 的遍历
String str = 'the 2 and 3 is 5'
str.each {
    // String temp -> print temp.multiply(2)
    String temp -> print temp
}

// find 来查找符合条件的第一个
str.find {
	String s -> s.isNumber()
}

// findAll 查找所有符合的条件
def list = str.findAll { String s -> s.isNumber() }
println list.toListString()

// any 只要满足一个条件返回 true
def result = str.any { String s -> s.isNumber() }

// every 所有的都满足条件返回 true
def result = str.every { String s -> s.isNumber() }

// collect 将字符串每个元素都加到 List 中返回
def list = str.collect { it.toUpperCase() }
```

---

- 闭包三个关键变量
	- this
	- owner
	- delegate

```
def scriptClouser = {
	// this 代表闭包定义处的类
	println "scriptClouser this: " + this
	// owner 代表闭包定义处的类或者对象
	println "scriptClouser owner: " + owner
	// delegate 代表任意对象，默认值：与 owner 一致 
	println "scriptClouser delegate: " + delegate
}

// 输出：
scriptClouser this: HelloGroovy@cad498c
scriptClouser owner: HelloGroovy@cad498c
scriptClouser delegate: HelloGroovy@cad498c
```


- 闭包的委托策略

```
class Student {
	String name
	def pretty = { "My name is ${name}" }
	
	String toString() {
		pretty.call()
	}
}

class Teacher {
	String name
}

def stu = new Student(name: 'Sarash')
def tea = new Teacher(name: 'Qndroid')

println stu.toString()

// 输出：
My name is Sarash
```

- 修改闭包委托策略
```
stu.pretty.delegate = tea
// 修改闭包委托策略
stu.pretty.resolveStrategy = Closure.DELEGATE_FIRST

println stu.toString()

// 输出：
My name is Qndroid
```

---

## groovy 的列表

- 初始化列表

```
// 空的 List
def list = []

// 有内容 List 
def list = [1, 2, 3, 4, 5]

// 定义数组 关键字 as
def array = [1, 2, 3] as int[]

// 强类型定义数组
int[] array2 = [1, 2, 3] 
```

- 列表的排序

```
def sortList = [6, -3, 9, 2, -7, 1, 5]

// 从小到大默认排序
Collections.sort(sortList)

// 自定义排序（按绝对值大小排序） 
Comparator mc = {
	a, b -> a == b ? 0 :
		Math.abs(a) < Math.abs(b) ? -1 : 1
}
Collections.sort(sortList, mc)

// groovy 中方法 sort()
sortList.sort()

// 自定义排序 
sortList.sort{
	a, b -> a == b ? 0 :
		Math.abs(a) < Math.abs(b) ? 1 : -1
}

// 对字符串排序
def stringList = ['abc', 'z', 'Hello', 'groovy', 'java']
StringList.sort{ it -> return it.size() }
```

- 列表的查找

```
def findList = [6, -3, 9, 2, -7, 1, 5]

int result = findList.find{ return it % 2 == 0 }

def result = findList.findAll{ return it % 2 != 0 }

def result = findList.any{ return it % 2 != 0 }

def result = findList.every { return it % 2 == 0 }

// 查找最小值
def result = findList.min()
findList.min{return Math.abs(it)}

// 查找最大值
def result = findList.max()
findList.max{return Math.abs(it)}

// 查找符合条件的总数
def num = findList.count{ return it % 2 == 0 }
```

---

## groovy 中的 map

```
// 定义 map
def colors = [red  : 'ff0000',
              green: '00ff00',
              blue : '0000ff']

// 索引方式
colors['red']
colors.red

// 添加元素
colors.yellow = 'ffff00'

// 添加 map
colors.complex = [a: 1, b: 2]

println(colors.toMapString())
```

- map 的遍历

```
def students = [
        1 : [number: '0001', name: 'Bob', score: 55, sex: 'male'],
        2 : [number: '0002', name: 'Johnny', score: 76, sex: 'male'],
        3 : [number: '0003', name: 'Claire', score: 86, sex: 'female'],
        4 : [number: '0004', name: 'Amy', score: 32, sex: 'male']
]

// 遍历
students.each{
	def student -> println 
		"the key is ${student.key}," 
	  + "the value is ${student.vale}"
}

// 带索引遍历
students.each{
	def student, int index -> println
		"index id ${index}," 
	  +	"the key is ${student.key}," 
	  + "the value is ${student.vale}"
}

students.each{
	key, value, index -> println
		"index id ${index}," 
	  +	"the key is ${student.key}," 
	  + "the value is ${student.vale}"
}
```

- Map 的查找

```
// find
def entry = students.find{def student ->
	return student.value.score >= 60
}

// findAll
def entrys = students.findAll{def student ->
	return student.value.score >= 60
}

// count
def count = students.count{def student ->
	return student.value.score >= 60 
		&& student.value.sex == 'male'
	
}

// 查找符合条件的所有学生的名字
def names = student.findAll{def student ->
	return student.value.score >= 60
}.collect{
	return it.value.name
}

println names.toListString()

// 将结果分组
def group = students.groupBy{def student ->
	return student.value.score >= 60 ? '及格' : '不及格'
}

println group.toMapString()
```

-- Map 排序

```
def sort = students.sort{def student1, def student2 ->
	Number score1 = student1.value.score
	Number score2 = student2.value.score
	return score1 == score2 ? 0 : score1 < srcore2 ? -1 : 1
}
println sort.toMapString()
```

## groovy 数据结构

```
// 定义范围 1~10
def range = 1..10

pringln range[0]
println range.contains(10)
// 获取范围起始值
println range.from
// 获取范围的终止值
println range.to

// 输出：
1
true
1
10
```

- 范围遍历

```
// 闭包方式
range.each{ println it }

// for 循环方式
for(i in range){ println i }
```

- 在 switch case 中应用

```
def result = getGrade(75)
println result

def getGrade(Number number){
	def result
	switch(number){
		case 0..<60:
			result = '不及格'
			break
		case 60..<70:
			result = '及格'
			break
		case 70..<80:
			result = '良好'
			break
	}
	
	return result 
}
```

## groovy 面向对象





## json 操作

- 对象转 json

```
def list = [new Persion(name: 'john', age: 26),
			new Persion(name: 'lily', age: 16)]

// 实体对象转换 json
JsonOutput.toJson(list)

// 输出格式化 json
def json = JsonOutput.toJson(list)
println JsonOutput.prettyPrint(json)
```

- json 转对象

```
def jsonSlpuer = new JsonSlurper()
jsonSlpuer.parse()
```

## xml 文件操作

- 解析 xml 数据

```
final String xml = ''' [xml格式的内容] '''

// 开始解析 xml 数据
def xmlSluper = new XmlSlurper()
def response = xmlSluper.parseText(xml)

println response.value.books[1].xxxxxxxxx
```

- 生成xml格式数据

```
// 生成 xml 格式数据

```










