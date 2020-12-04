---
title: web
toc: true
date: 2020-03-05 12:19:21
tags:
categories:
---



## 案例：简单的web服务器

```
package main

import (
	"fmt"
	"log"
	"net/http"
	_ "strings"
)

func sayHi(w http.ResponseWriter, r *http.Request) {
	r.ParseForm()
	fmt.Println(r.Form)

	for k, v := range r.Form {
		fmt.Println("key", k)
		fmt.Println("val", v)
	}

	fmt.Fprintf(w, "Hello This is simple web")
}

func main() {
	http.HandleFunc("/", sayHi)

	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```

## 模版案例1

```
package main

import (
	"html/template"
	"os"
)

type Person struct {
	Username string
	Email    string
}

func main() {
	t := template.New("temp_name")
	t.Parse("hello {{.Username}}, {{.Email}}\n {{.}}")
	p := Person{Username: "fuyi", Email: "dingidng@qq.com"}

	t.Execute(os.Stdout, p)
}
```

## 模版案例2

```
package main

import (
	"html/template"
	"os"
)

type Friend struct {
	Fname string
}

type Person struct {
	Username string
	Friends  []*Friend
	Emails   []string
}

func main() {
	t := template.New("temp_name")
	t.Parse(`hello {{.Username}}!
				{{range .Emails}}
					an email {{.}}
				{{end}}
				
				{{with .Friends}}
					{{range .}}
						my friends name is {{.Fname}}
					{{end}}
				{{end}}
			`)

	f1 := Friend{Fname: "Boo"}
	f2 := Friend{Fname: "App"}
	p := Person{Username: "abced",
		Emails:  []string{"aaa@qq.com", "bbb@qq.com"},
		Friends: []*Friend{&f1, &f2}}

	t.Execute(os.Stdout, p)
}
```





## 参考资料
> - []()
> - []()
