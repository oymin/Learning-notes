## 练习题

---

#### 第1题

写一个程序，对于给定一个数字n，求出所有两两相加等于n的组合
比如： 对于n=5，所有组合如下所示：

0+5=5  
1+4=5  
2+3=5  
3+2=5  
4+1=5  
5+0=5  

> 答案：

```
package main

import "fmt"

func list(n int){
	for i:=0;i<=n;i++{
		fmt.Printf("%d+%d=%d\n",i,n-i,n)
	}
}

func main(){
	list(5)
}

```

---

#### 2. 冒泡排序 数组

```
func main() {

	// 设置种子，只需一次
	rand.Seed(time.Now().UnixNano()) // 已当前时间作为种子参数

	b := [10]int{}

	for i := 0; i < len(b); i++ {
		b[i] = rand.Intn(100) // 生成随机数赋值给数组
	}
	fmt.Println("b = ", b)

	n:=len(b)
	for i := 0; i < n-1; i++ {
		for j := 0; j < n-1-i; j++ {
			if b[j] > b[j+1] {
				b[j], b[j+1] = b[j+1], b[j]
			}
		}
	}
	fmt.Println("b = ", b)
}
```

---

#### 3. 写一个猜数字游戏

```
/**
 * 猜数字游戏 4位数
 */
func main() {

	var randNum int

	// 产生一个4位的随机数
	CreateNum(&randNum)

	randSlice := make([]int,4)

	// 保存这个4位数的每一位
	GetNum(randSlice,randNum)

	// 开始游戏
	OnGame(randSlice)

}

// 生成4位的随机数
func CreateNum(p *int) {
	// 设置种子
	rand.Seed(time.Now().UnixNano())

	var randNum int
	for {
		randNum = rand.Intn(10000)
		if randNum >= 1000 {
			break
		}
	}

	*p =randNum
}

func OnGame(ints []int) {
	var num int
	keySlice := make([]int,4)

	for {
		for {
			fmt.Printf("请输入一个4位数：")
			fmt.Scan(&num)
			if 999 < num && num < 10000 {
				break
			}
			fmt.Println("输入的数不符合要求！！！")
		}

		GetNum(keySlice,num)

		n := 0
		for i := 0; i < 4; i++ {
			if keySlice[i] > ints[i] {
				fmt.Printf("第%d位数大了\n",i+1)
			}else if keySlice[i] < ints[i] {
				fmt.Printf("第%d位数小了\n",i+1)
			}else {
				fmt.Printf("第%d位数猜对了\n",i+1)
				n++
			}
		}

		if n == 4 {
			fmt.Println("恭喜你，全部猜对了！")
			break // 结束游戏
		}
	}
}

func GetNum(ints []int, num int) {
	ints[0] = num / 1000 		//取千位
	ints[1] = num % 1000 / 100 //取千位
	ints[2] = num % 100/ 10 	//取千位
	ints[3] = num % 10 			//取千位
}
```
