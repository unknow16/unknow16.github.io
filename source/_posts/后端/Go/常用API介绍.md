---
title: 常用API介绍
toc: true
date: 2020-03-03 20:38:34
tags:
categories:
---



## 日期

```
package main

import (
	"fmt"
	"time"
)

func main()  {

	// 当前日期
	now := time.Now()
	fmt.Printf("now = %v \nnow type=%T\n", now, now)

	fmt.Printf("year=%v\n", now.Year())
	fmt.Printf("hour=%v\n", now.Hour())

	// 格式化日期
	dateStr := fmt.Sprintf("当前年月日 %d-%d-%d %d:%d:%d\n",
			now.Year(), now.Month(), now.Day(),
			now.Hour(), now.Minute(), now.Second())
	fmt.Printf("datestr=%s\n", dateStr)

	// 该字符串各数字都是固定的，可以任意组合成需要的格式，结果为当前时间的该种格式显示
	fmt.Printf(now.Format("2006-01-02 15:04:05"))
	fmt.Println()
	fmt.Printf(now.Format("2006-01-02"))
	fmt.Println()
	fmt.Printf(now.Format("15:04:05"))
	fmt.Println()

	fmt.Printf("unix秒数：%v\n", now.Unix())
	fmt.Printf("unix纳秒数： %v", now.UnixNano())
}

```



## 字符串处理

```
package main

import (
	"fmt"
	"strconv"
	"strings"
)

func main()  {
	// 字符串转整数
	n, _ := strconv.Atoi("12")
	fmt.Println("atoi : ", n)

	// 整数转字符串
	str := strconv.Itoa(1234)
	fmt.Println("itoa : ", str)

	// 字符串转[]byte
	var bytes = []byte("hello go")
	fmt.Printf("bytes=%v\n", bytes)

	// []bytes转字符串
	str1 := string([]byte{97,98,99})
	fmt.Printf("str=%v\n", str1)

	// 10进制转2
	str2 := strconv.FormatInt(123, 2)
	fmt.Println("formatInt2: ", str2)

	// 10进制转16
	str3 := strconv.FormatInt(123, 16)
	fmt.Println("formatInt16: ", str3)

	// 在字符串中是否存在子串
	is := strings.Contains("seafood", "foo")
	fmt.Println("strings.Contains: ", is)

	// 不区分大小写的字符串比较
	fmt.Println(strings.EqualFold("abc", "Abc"))

	// 区分大小写比较
	fmt.Println("abc" == "Abc")

	// 返回子串第一次出现的index值
	// LastIndex最后一次出现
	fmt.Println(strings.Index("LNT_abc", "abc"))

	str4 := strings.Replace("go go hello", "go", "go语言", -1)
	fmt.Println("Replace: " + str4)

	//fmt.Println(strconv.IntSize)
}

```







## 文件操作

- os.File封装所有文件操作，是一个结构体
- bufio.Reader和bufio.Writer包含缓冲的读写，都是一个结构体
- io/ioutil可以一次性将整个文件读到内存中，适合文件不大的情况

#### 读文件

```
package main

import (
	"fmt"
	"os"
)

func read_file(filename string) {
	// fin是一个*File
	fin, err := os.Open(filename)

	if err != nil {
		fmt.Println(err)
		return
	}
	defer fin.Close()

	buf := make([]byte, 1024)

	for {
		n, _ := fin.Read(buf)

		// 0表示到达EOF
		if n == 0 {
			break
		}

		os.Stdout.Write(buf)
	}

}

func main() {
	read_file("./aa.txt")

}
```


#### 写文件

```
package main

import (
	"fmt"
	"os"
)

func main() {
	myFile := "./aa.txt"
	fout, err := os.Create(myFile)

	if err != nil {
		fmt.Println(err)
		return
	}

	for i := 0; i < 10; i++ {
		outStr := fmt.Sprintf("%s:%d\n", "Hello", i)

		fout.WriteString(outStr)
		fout.Write([]byte("abcd\n"))
	}

	fout.Close()

}

```


#### 读时写

```
package main

import (
	"fmt"
	"io"
	"os"
)

func main() {
	fi, err := os.Open("./aa.txt")
	if err != nil {
		fmt.Println(err)
	}
	defer fi.Close()

	fo, err := os.Create("./aa_new.txt")
	if err != nil {
		fmt.Println(err)
	}
	defer fo.Close()

	buf := make([]byte, 1024)

	for {
		n, err := fi.Read(buf)
		if err != nil && err != io.EOF {
			fmt.Println(err)
		}
		if n == 0 {
			break
		}

		if n2, err := fo.Write(buf[:n]); err != nil {
			panic(err)
		} else if n2 != n {
			panic("error in writing")
		}
	}
}

```

#### 用bufio库

```
package main

import (
	"os"
	"bufio"
)

func main() {

	file, err := os.OpenFile("./test.txt", os.O_CREATE | os.O_WRONLY, 0666)
	if err != nil {
		panic(err)
	}

	defer file.Close()

	w := bufio.NewWriter(file)
	
	for i := 0; i < 5; i++ {
		w.WriteString("hello\n")
	}

	// 数据是先写到缓冲区的，此处刷新才到文件中
	w.Flush()
}
```



```
package main

import (
	"bufio"
	"io"
	"os"
)

func main() {
	fi, err := os.Open("./aa.txt")
	if err != nil {
		panic(err)
	}
	defer fi.Close()
	r := bufio.NewReader(fi)

	fo, err := os.Create("./aa1.txt")
	if err != nil {
		panic(err)
	}
	defer fo.Close()
	w := bufio.NewWriter(fo)

	buf := make([]byte, 1024)

	for {
		n, err := r.Read(buf)
		if err != nil && err != io.EOF {
			panic(err)
		}
		if n == 0 {
			break
		}

		if n2, err := w.Write(buf[:n]); err != nil {
			panic(err)
		} else if n2 != n {
			panic("error in writing")
		}
	}

	if err = w.Flush(); err != nil {
		panic(err)
	}
}

```

#### 用ioutil库

```
package main

import (
	"io/ioutil"
)

func main() {
	b, err := ioutil.ReadFile("./aa.txt")
	if err != nil {
		panic(err)
	}

	err = ioutil.WriteFile("./aa2.txt", b, 0644)
	if err != nil {
		panic(err)
	}
}

```

#### 判断文件是否存在

```
func PathExists(path string)(boo, error) {
    _, err := os.Stat(path)
    if err == nil { // 获取path的信息成功，说明路径存在
        return true, nil
    }
    if os.IsNotExist(err) {
        return false, nil
    }
    return false, err
    
}
```







## JSON操作

#### 编码

```
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type Moive struct {
	Title  string
	Year   int  `json:"released"`        // Tag元信息，编码时重命名key
	Color  bool `json:"color,omitempty"` // omitempty表示结构体成员为空或零值时不生成该JSON对象
	Actors []string
}

var movies = []Moive{
	{Title: "战狼2", Year: 2017, Color: true, Actors: []string{"吴京", "吴刚"}},
	{Title: "老炮儿", Year: 2015, Color: true, Actors: []string{"冯小刚", "吴亦凡", "许晴"}},
	{Title: "Bullitt", Year: 1968, Color: false, Actors: []string{"Steve McQueen", "Jacqueline Bi sset"}},
}

func main() {
	// json紧凑格式
	
	//json_str, err := json.Marshal(movies)

	// 格式化的json,后两参数表示每一行输出的前缀和每一个层级的缩进
	json_str, err := json.MarshalIndent(movies, "", "	")
	if err != nil {
		log.Fatal(err)
		return
	}

	fmt.Printf("%s\n", json_str)
}
```

#### 解码

```
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type Moive struct {
	Title  string
	Year   int  `json:"released"`        // Tag元信息，编码时重命名key
	Color  bool `json:"color,omitempty"` // omitempty表示结构体成员为空或零值时不生成该JSON对象
	Actors []string
}

var movies = []Moive{
	{Title: "战狼2", Year: 2017, Color: true, Actors: []string{"吴京", "吴刚"}},
	{Title: "老炮儿", Year: 2015, Color: true, Actors: []string{"冯小刚", "吴亦凡", "许晴"}},
	{Title: "Bullitt", Year: 1968, Color: false, Actors: []string{"Steve McQueen", "Jacqueline Bi sset"}},
}

func main() {
	// json紧凑格式
	json_str, err := json.Marshal(movies)

	// 格式化的json,后两参数表示每一行输出的前缀和每一个层级的缩进
	// json_str, err := json.MarshalIndent(movies, "", "	")
	if err != nil {
		log.Fatal(err)
		return
	}

	fmt.Printf("%s\n", json_str)

	fmt.Println("-----------------")

	var titles []struct {
		Title string
	}
	if err := json.Unmarshal(json_str, &titles); err != nil {
		log.Fatal(err)
		return
	}
	fmt.Printf("Titles:  %+v\n", titles)

	/* 注意 json 编解码 只能处理结构体成员变量是大写的字段,小写的直接忽略
	如果在编码时, 想让json字符串的字段小写,那么需要再结构体中加Tag
	如果再解码时, json有小写的字段名,那么在定义结构体接收的时候,成员名要大写,然后也要加Tag */
	var my_movies []struct {
		Title  string
		Year   int `json:"released"`
		Actors []string
	}
	if err := json.Unmarshal(json_str, &my_movies); err != nil {
		log.Fatal(err)
		return
	}

	fmt.Printf("movies:  %+v\n", movies)
}

```
## 解析命令行参数
#### os.Args

```
var Args []string // 切片
```

存储所有命令行参数，第0个是程序名

#### flag包

os.Args对解析cmd> main.exe -f c:/aa.txt -p 200 -u root 这样的命令行参数不太方便

```
package main

import (
	"flag"
	"fmt"
)

func main() {

	// 用于接受命令行参数
	var host string
	var port int

	flag.StringVar(&host, "h", "localhost", "主机名，默认localhost")
	flag.IntVar(&port, "p", 8080, "端口号，默认8080")

	// 解析
	flag.Parse()
	fmt.Printf("host: %v, port: %v\n", host, port)

}
```

## 单元测试

- 测试用例文件必须以_test.go结尾
- 测试用例函数必须以Test开头，且形参必须是t *testing.T

```
go test // 如果正确无日志,错误会有
go test -v // 正确与否，都会有错误日志
go test -v cal_test.go cal.go // 测试单个文件，要带上被测试的原文件
go test -v -test.run TestAddUser // 测试单个方法
```

- 出错时，可用t.Fatalf来格式化输出错误

- t.Logf方法可以输出日志






## 参考资料
> - []()
> - []()
