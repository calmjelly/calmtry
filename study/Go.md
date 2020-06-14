1、添加输入参数

```
go run Test.go arg=param
```

2、go goroutine 示例

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	go compute(10)
	go compute(10)
	var input string
	fmt.Scanln(&input)

}

func compute(x int) {
	for i := 0; i < x; i++ {
		time.Sleep(time.Second)
		fmt.Println(i)
	}
}

```

3、一个简单的web服务器示例

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w,"%s %s %s\n",r.Method,r.URL,r.Proto)
	for k,v:=range r.Header{
		fmt.Fprintf(w,"Header[%q]=%q\n",k,v)
	}
	fmt.Fprintf(w,"Host=%q\n",r.Host)
	fmt.Fprintf(w,"RemoteAddr=%q\n",r.RemoteAddr)
	if err := r.ParseForm();err!=nil {
		log.Print(err)
	}
	for k,v:=range r.Form{
		fmt.Fprintf(w,"Form[%q]=%q\n",k,v)
	}

}

func main() {
	http.HandleFunc("/", handler)       // 设置访问的路由
	err := http.ListenAndServe(":8080", nil) // 设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServer Failed:", err)
	}
}

```

