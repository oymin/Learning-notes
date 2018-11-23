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
