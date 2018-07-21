---
layout: post
title:  "gobox中mysql操作"
---

今天来说下使用gobox中mysql操作相关

## 说明

本包的driver部分使用了go-sql-driver：https://github.com/go-sql-driver/mysql

### 用法示例

示例表结构为：

```
| demo  | CREATE TABLE `demo` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `add_time` timestamp NOT NULL DEFAULT current_timestamp(),
  `edit_time` timestamp NOT NULL DEFAULT current_timestamp(),
  `name` varchar(20) COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  `status` tinyint(4) unsigned NOT NULL DEFAULT 0,
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`),
  KEY `status` (`status`),
  KEY `add_time` (`add_time`),
  KEY `edit_time` (`edit_time`)
) ENGINE=InnoDB AUTO_INCREMENT=478 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
```

```
package main

import (
	"github.com/goinbox/mysql"

	"database/sql"
	"strconv"
	"fmt"
)

type tableDemoRowItem struct {
	Id       int64
	AddTime  string
	EditTime string
	Name     string
	Status   int
}

var client *mysql.Client

func main() {
	config := mysql.NewConfig("root", "123", "127.0.0.1", "3306", "gobox-demo")
	client, _ = mysql.NewClient(config, nil)

	client.Exec("DELETE FROM demo")

	fmt.Println("testClientExec：")
	testClientExec()
	fmt.Println("")

	fmt.Println("testClientQuery：")
	testClientQuery()
	fmt.Println("")

	fmt.Println("testClientQueryRow：")
	testClientQueryRow()
	fmt.Println("")

	fmt.Println("testClientTrans：")
	testClientTrans()
	fmt.Println("")
}

func testClientExec() {
	result, err := client.Exec("INSERT INTO demo (name) VALUES (?),(?)", "a", "b")
	if err != nil {
		fmt.Println("exec error: " + err.Error())
	} else {
		li, err := result.LastInsertId()
		if err != nil {
			fmt.Println("lastInsertId error: " + err.Error())
		} else {
			fmt.Println("lastInsertId: " + strconv.FormatInt(li, 10))
		}

		rf, err := result.RowsAffected()
		if err != nil {
			fmt.Println("rowsAffected error: " + err.Error())
		} else {
			fmt.Println("rowsAffected: " + strconv.FormatInt(rf, 10))
		}
	}
}

func testClientQuery() {
	rows, err := client.Query("SELECT * FROM demo WHERE name IN (?,?)", "a", "b")
	if err != nil {
		fmt.Println("query error: " + err.Error())
	} else {
		for rows.Next() {
			item := new(tableDemoRowItem)
			err = rows.Scan(&item.Id, &item.AddTime, &item.EditTime, &item.Name, &item.Status)
			if err != nil {
				fmt.Println("rows scan error: " + err.Error())
			} else {
				fmt.Println(item)
			}
		}
	}
}

func testClientQueryRow() {
	row := client.QueryRow("SELECT * FROM demo WHERE name = ?", "a")
	item := new(tableDemoRowItem)
	err := row.Scan(&item.Id, &item.AddTime, &item.EditTime, &item.Name, &item.Status)
	if err != nil {
		if err == sql.ErrNoRows {
			fmt.Println("no rows: " + err.Error())
		} else {
			fmt.Println("row scan error: " + err.Error())
		}
	} else {
		fmt.Println(item)
	}
}

func testClientTrans() {
	client.Begin()

	row := client.QueryRow("SELECT * FROM demo WHERE name = ?", "a")
	item := new(tableDemoRowItem)
	err := row.Scan(&item.Id, &item.AddTime, &item.EditTime, &item.Name, &item.Status)
	if err != nil {
		fmt.Println("row scan error: " + err.Error())
	} else {
		fmt.Println(item)
	}

	client.Commit()

	err = client.Rollback()
	fmt.Println(err)
}
```
程序输出：

```
testClientExec：
lastInsertId: 474
rowsAffected: 2

testClientQuery：
&{474 2018-07-21 09:53:43 2018-07-21 09:53:43 a 0}
&{475 2018-07-21 09:53:43 2018-07-21 09:53:43 b 0}

testClientQueryRow：
&{474 2018-07-21 09:53:43 2018-07-21 09:53:43 a 0}

testClientTrans：
&{474 2018-07-21 09:53:43 2018-07-21 09:53:43 a 0}
Not in trans

ligang@vm-xubuntu ~/tmp/go $ go run main.go 
testClientExec：
lastInsertId: 476
rowsAffected: 2

testClientQuery：
&{476 2018-07-21 09:53:52 2018-07-21 09:53:52 a 0}
&{477 2018-07-21 09:53:52 2018-07-21 09:53:52 b 0}

testClientQueryRow：
&{476 2018-07-21 09:53:52 2018-07-21 09:53:52 a 0}

testClientTrans：
&{476 2018-07-21 09:53:52 2018-07-21 09:53:52 a 0}
Not in trans
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
