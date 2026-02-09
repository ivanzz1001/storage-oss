# HTTP 100 continue 

HTTP 100-continue是HTTP协议中的一个机制，允许客户端在发送请求主体之前，先确认服务器是否愿意接受该请求。

## 1. GoLang模式实现http 100-continue

### 1.1 client端

```golang
package main

import (
	"bytes"
	"fmt"
	"io"
	"net/http"
	"time"
)

func main() {
	// 创建一个带body的POST请求
	body := bytes.NewBufferString("hello,world")
	req, err := http.NewRequest("POST", "http://localhost:8082/upload", body)
	if err != nil {
		fmt.Printf("创建请求失败: %v\n", err)
		return
	}

	// 设置Expect: 100-continue头部
	req.Header.Set("Expect", "100-continue")
	req.Header.Set("Content-Type", "text/plain")

	// 创建HTTP客户端并设置超时时间
	client := &http.Client{
		Timeout: 30 * time.Second,
	}

	// 发送请求
	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("发送请求失败: %v\n", err)
		return
	}
	defer resp.Body.Close()

	// 读取响应
	responseBody, err := io.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("读取响应体失败: %v\n", err)
		return
	}

	fmt.Printf("状态码: %d\n", resp.StatusCode)
	fmt.Printf("响应体: %s\n", responseBody)
}
```

### 1.2 server端实现

```golang
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"time"
)

func uploadHandler(w http.ResponseWriter, r *http.Request) {
	// 检查是否有Expect: 100-continue头部
	if r.Header.Get("Expect") == "100-continue" {
		// 可以在这里进行一些预检查(比如认证、权限等)
		// 如果拒绝请求，可以返回4xx状态码
		// 这里我们接受请求，发送100 Continue响应
		
		// 显式地发送100 Continue响应
		w.WriteHeader(http.StatusContinue)
		
		// 注意：这里不能调用w.Write()，因为这会干扰后续的写入
		// 我们只是通知客户端继续发送body
		fmt.Println("已发送100 Continue响应")
	}
	
	// 读取请求体
	body, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "无法读取请求体", http.StatusBadRequest)
		return
	}
	
	// 模拟处理延迟
	time.Sleep(1 * time.Second)
	
	// 返回响应
	response := fmt.Sprintf("服务器收到%d字节的数据: %s", len(body), string(body))
	w.WriteHeader(http.StatusOK)
	w.Write([]byte(response))
	
	fmt.Printf("接收到请求体(%d字节)\n", len(body))
}

func main() {
	http.HandleFunc("/upload", uploadHandler)
	
	fmt.Println("服务器启动在 :8082")
	log.Fatal(http.ListenAndServe(":8082", nil))
}
```
