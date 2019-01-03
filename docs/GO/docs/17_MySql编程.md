# MySql 编程

## 下载 Mysql 驱动库

执行命令：

```
go get github.com/go-sql-driver/mysql
```

## 链接 mysql

```
database, err := sqlx.Open("mysql", "root:root@tcp(127.0.0.1:3306)/test")
```

## insert 操作

```
r, err := Db.Exec("insert into person(username, sex, email)values(?, ?, ?)", "stu001", "man", "stu01@qq.com")
```

## Select 操作

```
var person []Person

err := Db.Select(&person, "select user_id, username, sex, email from person where user_id=?", 1)
```

## update 操作

```
_, err := Db.Exec("update person set username=? where user_id=?", "stu0001", 1)
```

## Delete 操作

```
_, err := Db.Exec("delete from person where user_id=?", 1)
```

## 示例代码

添加：

```
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type Person struct {
	UserId   int    `db:"user_id"`
	Username string `db:"username"`
	Sex      string `db:"sex"`
	Email    string `db:"email"`
}

type Place struct {
	Country string `db:"country"`
	City    string `db:"city"`
	TelCode int    `db:"telcode"`
}

var Db *sqlx.DB

func init() {
	database, err := sqlx.Open("mysql", "root:root@tcp(127.0.0.1:3306)/test")
	if err != nil {
		fmt.Println("open mysql failed,", err)
		return
	}
	Db = database
}

func main() {
	r, err := Db.Exec("insert into person(username, sex, email)values(?, ?, ?)", "stu001", "man", "stu01@qq.com")
	if err != nil {
		fmt.Println("exec failed, ", err)
		return
	}
	id, err := r.LastInsertId()
	if err != nil {
		fmt.Println("exec failed, ", err)
		return
	}

	fmt.Println("insert succ:", id)
}
```
