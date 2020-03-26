# Golang的反射reflect深入理解

在计算机科学领域，反射是指一类应用，它们能够自描述和自控制。也就是说，这类应用通过采用某种机制来实现对自己行为的描述（self-representation）和监测（examination），并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。

每种语言的反射模型都不同，并且有些语言根本不支持反射。Golang语言实现了反射，反射机制就是在运行时动态的调用对象的方法和属性，官方自带的reflect包就是反射相关的，只要包含这个包就可以使用。

多插一句，Golang的gRPC也是通过反射实现的。



```go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordID      int `db:"size:255"`
	customerID int `db:"type:int(10); not null; default 0"`
}

type employee struct {
	name    string
	id      int
	address string
	salary  int
	country string
}

func createQuery(q interface{}) string {
	if reflect.TypeOf(q).Kind() != reflect.Struct {
		panic("unsupported type")
	}
	// insert into table (name, age) values ('test', 15)
	// 获取反射结构体的名称

	// 获取结构体的具体的值
	t := reflect.TypeOf(q)
	v := reflect.ValueOf(q)
	query := fmt.Sprintf("insert into `%s` (", t.Name())
	for i := 0; i < t.NumField(); i++ {
		fieldName := t.Field(i).Name
		if i == 0 {
			query = fmt.Sprintf("%s`%s`", query, string(fieldName))
		} else {
			query = fmt.Sprintf("%s, `%s`", query, string(fieldName))
		}
	}
	query = fmt.Sprintf("%s) values (", query)

	// 遍历结构体中的字段
	for i := 0; i < v.NumField(); i++ {
		switch v.Field(i).Kind() {
		// 结构体中每个字段的具体的类型
		case reflect.String:
			if i == 0 {
				query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
			} else {
				query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
			}
		case reflect.Int:
			if i == 0 {
				query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
			} else {
				query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
			}
		default:
			fmt.Println("Unsupported type")
			return ""
		}
	}
	query = fmt.Sprintf("%s)", query)
	fmt.Println(query)
	return ""
}

func main() {
	o := order{
		ordID:      1234,
		customerID: 567,
	}
	createQuery(o)
	// e := employee{
	// 	name:    "Naveen",
	// 	id:      565,
	// 	address: "Science Park Road, Singapore",
	// 	salary:  90000,
	// 	country: "Singapore",
	// }
	// createQuery(e)
}

```

### 参考：

- https://juejin.im/post/5a75a4fb5188257a82110544

