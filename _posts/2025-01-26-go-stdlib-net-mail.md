---
layout: post
title: 微读 Go 标准库 - net/mail 解析邮件地址与消息格式
categories: [标准库]
tags: [net/mail, 邮件解析, RFC5322]
---

想要解析邮件地址或处理邮件消息？Go 的 net/mail 包提供了符合 RFC 5322 标准的邮件解析能力，让你轻松处理邮件相关的各种场景。

### 1. 解析邮件地址

最常见的需求就是解析邮件地址。net/mail 能智能识别各种格式：

```go
package main

import (
    "fmt"
    "net/mail"
)

func main() {
    // 解析带名称的邮件地址
    addr, _ := mail.ParseAddress("张三 <zhangsan@example.com>")
    fmt.Println(addr.Name)    // 张三
    fmt.Println(addr.Address) // zhangsan@example.com
    
    // 解析纯邮件地址
    addr2, _ := mail.ParseAddress("admin@example.com")
    fmt.Println(addr2.Address) // admin@example.com
}
```

---

### 2. 批量解析邮件地址列表

处理群发邮件？ParseAddressList 一次搞定多个地址：

```go
addresses := "Alice <alice@example.com>, Bob <bob@example.com>, charlie@example.com"
list, _ := mail.ParseAddressList(addresses)

for _, addr := range list {
    fmt.Printf("姓名: %s, 邮箱: %s\n", addr.Name, addr.Address)
}
// 输出:
// 姓名: Alice, 邮箱: alice@example.com
// 姓名: Bob, 邮箱: bob@example.com
// 姓名: , 邮箱: charlie@example.com
```

---

### 3. 读取完整邮件消息

net/mail 能解析完整的邮件消息，包括头部和正文：

```go
import (
    "io"
    "net/mail"
    "strings"
)

func parseEmail(emailContent string) {
    reader := strings.NewReader(emailContent)
    msg, _ := mail.ReadMessage(reader)
    
    // 获取邮件头部信息
    header := msg.Header
    fmt.Println("发件人:", header.Get("From"))
    fmt.Println("收件人:", header.Get("To"))
    fmt.Println("主题:", header.Get("Subject"))
    fmt.Println("日期:", header.Get("Date"))
    
    // 读取邮件正文
    body, _ := io.ReadAll(msg.Body)
    fmt.Println("正文:", string(body))
}
```

---

### 4. 解析邮件日期

邮件日期格式复杂？ParseDate 帮你转换成 Go 的 time.Time：

```go
// 解析 RFC 5322 格式的日期
dateStr := "Mon, 23 Jun 2024 09:30:00 +0800"
t, _ := mail.ParseDate(dateStr)
fmt.Println(t.Format("2006-01-02 15:04:05"))
// 输出: 2024-06-23 09:30:00

// 也能处理带时区的格式
dateStr2 := "23 Jun 2024 09:30:00 EST"
t2, _ := mail.ParseDate(dateStr2)
fmt.Println(t2.UTC())
```

---

### 5. 构建邮件地址字符串

Address 类型提供了 String() 方法，能正确格式化邮件地址：

```go
addr := &mail.Address{
    Name:    "技术支持",
    Address: "support@example.com",
}
fmt.Println(addr.String())
// 输出: "技术支持" <support@example.com>

// 特殊字符会自动转义
addr2 := &mail.Address{
    Name:    `John "The Boss" Doe`,
    Address: "john@example.com",
}
fmt.Println(addr2.String())
// 输出: "John \"The Boss\" Doe" <john@example.com>
```

---

### 6. 处理邮件头部编码

处理国际化邮件？net/mail 支持 RFC 2047 编码的头部：

```go
// 解析包含中文的邮件
emailStr := `From: =?UTF-8?B?5byg5LiJ?= <zhangsan@example.com>
To: lisi@example.com
Subject: =?UTF-8?B?5rWL6K+V6YKu5Lu2?=

邮件正文内容`

msg, _ := mail.ReadMessage(strings.NewReader(emailStr))
from, _ := msg.Header.AddressList("From")
fmt.Println(from[0].Name) // 能正确解析出中文名
```

---

### 7. 验证邮件地址格式

利用 ParseAddress 进行邮件格式验证：

```go
func isValidEmail(email string) bool {
    _, err := mail.ParseAddress(email)
    return err == nil
}

// 测试
fmt.Println(isValidEmail("user@example.com"))     // true
fmt.Println(isValidEmail("invalid.email"))        // false
fmt.Println(isValidEmail("user@"))                // false
```

---

### 8. 提取邮件附件信息

虽然 net/mail 不直接处理附件，但能获取相关头部信息：

```go
msg, _ := mail.ReadMessage(reader)
contentType := msg.Header.Get("Content-Type")

// 检查是否为 multipart 消息
if strings.Contains(contentType, "multipart") {
    // 获取边界标识
    boundary := extractBoundary(contentType)
    fmt.Println("这是一个包含附件的邮件")
    fmt.Println("边界标识:", boundary)
}

// 检查附件相关头部
disposition := msg.Header.Get("Content-Disposition")
if strings.Contains(disposition, "attachment") {
    fmt.Println("发现附件")
}
```

net/mail 包虽然简单，却是处理邮件相关任务的利器。它严格遵循 RFC 标准，让你的邮件处理代码更加健壮可靠。记住，它主要用于解析邮件格式，如果需要发送邮件，你需要配合 net/smtp 包使用。