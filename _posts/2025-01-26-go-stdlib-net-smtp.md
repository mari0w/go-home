---
layout: post
title: 微读 Go 标准库 - net/smtp 发送邮件的简单协议
categories: [标准库]
tags: [net/smtp, 邮件发送, SMTP, 认证]
---

需要用 Go 发送邮件？net/smtp 包实现了 RFC 5321 标准的简单邮件传输协议，让你轻松连接 SMTP 服务器发送邮件。

### 1. 一键发送邮件

最简单的方式，使用 SendMail 函数一步到位：

```go
package main

import (
    "net/smtp"
)

func main() {
    // SMTP 服务器配置
    auth := smtp.PlainAuth("", "your@gmail.com", "password", "smtp.gmail.com")
    
    // 邮件内容
    msg := []byte("Subject: 测试邮件\r\n" +
                  "\r\n" +
                  "这是一封测试邮件。\r\n")
    
    // 发送邮件
    err := smtp.SendMail("smtp.gmail.com:587",
                        auth,
                        "your@gmail.com",
                        []string{"recipient@example.com"},
                        msg)
    if err != nil {
        panic(err)
    }
}
```

---

### 2. PLAIN 认证

最常用的认证方式，适合大多数邮件服务商：

```go
// 创建 PLAIN 认证
auth := smtp.PlainAuth(
    "",                    // identity，通常为空
    "username@domain.com", // 用户名
    "password",           // 密码
    "smtp.domain.com",    // 主机名
)

// 常见邮件服务商配置
// Gmail
gmailAuth := smtp.PlainAuth("", "user@gmail.com", "app-password", "smtp.gmail.com")

// 163邮箱
auth163 := smtp.PlainAuth("", "user@163.com", "password", "smtp.163.com")

// QQ邮箱
qqAuth := smtp.PlainAuth("", "user@qq.com", "auth-code", "smtp.qq.com")
```

---

### 3. 使用 Client 进行精细控制

需要更多控制权？使用 Client 类型分步操作：

```go
import (
    "crypto/tls"
    "net/smtp"
)

func sendWithClient() error {
    // 连接到SMTP服务器
    client, err := smtp.Dial("smtp.gmail.com:587")
    if err != nil {
        return err
    }
    defer client.Close()
    
    // 启动TLS加密
    tlsConfig := &tls.Config{ServerName: "smtp.gmail.com"}
    if err = client.StartTLS(tlsConfig); err != nil {
        return err
    }
    
    // 认证
    auth := smtp.PlainAuth("", "user@gmail.com", "password", "smtp.gmail.com")
    if err = client.Auth(auth); err != nil {
        return err
    }
    
    // 设置发件人
    if err = client.Mail("sender@gmail.com"); err != nil {
        return err
    }
    
    // 添加收件人
    if err = client.Rcpt("recipient@example.com"); err != nil {
        return err
    }
    
    // 发送邮件内容
    w, err := client.Data()
    if err != nil {
        return err
    }
    
    _, err = w.Write([]byte("Subject: 测试\r\n\r\n邮件内容"))
    if err != nil {
        return err
    }
    
    return w.Close()
}
```

---

### 4. 构建标准邮件格式

SMTP 要求特定的邮件格式，包含完整的头部信息：

```go
func buildEmail(from, to, subject, body string) []byte {
    msg := fmt.Sprintf(
        "From: %s\r\n" +
        "To: %s\r\n" +
        "Subject: %s\r\n" +
        "MIME-Version: 1.0\r\n" +
        "Content-Type: text/plain; charset=UTF-8\r\n" +
        "\r\n" +
        "%s",
        from, to, subject, body,
    )
    return []byte(msg)
}

// 使用示例
emailContent := buildEmail(
    "发件人 <sender@example.com>",
    "收件人 <recipient@example.com>",
    "邮件主题",
    "邮件正文内容",
)
```

---

### 5. HTML 邮件发送

发送富文本邮件，设置正确的 Content-Type：

```go
func sendHTMLEmail(auth smtp.Auth, from string, to []string, subject, htmlBody string) error {
    msg := fmt.Sprintf(
        "From: %s\r\n" +
        "To: %s\r\n" +
        "Subject: %s\r\n" +
        "MIME-Version: 1.0\r\n" +
        "Content-Type: text/html; charset=UTF-8\r\n" +
        "\r\n" +
        "%s",
        from,
        strings.Join(to, ","),
        subject,
        htmlBody,
    )
    
    return smtp.SendMail("smtp.example.com:587", auth, from, to, []byte(msg))
}

// 使用示例
htmlContent := `
<html>
<body>
    <h1>欢迎使用Go发送邮件</h1>
    <p>这是一封<strong>HTML格式</strong>的邮件。</p>
</body>
</html>
`

sendHTMLEmail(auth, "sender@example.com", []string{"recipient@example.com"}, 
              "HTML邮件", htmlContent)
```

---

### 6. 群发邮件处理

发送给多个收件人，注意错误处理：

```go
func sendBulkEmail(auth smtp.Auth, from string, recipients []string, subject, body string) {
    msg := buildEmail(from, "", subject, body)
    
    for _, to := range recipients {
        err := smtp.SendMail("smtp.example.com:587", auth, from, []string{to}, msg)
        if err != nil {
            fmt.Printf("发送给 %s 失败: %v\n", to, err)
            continue
        }
        fmt.Printf("成功发送给 %s\n", to)
    }
}

// 批量发送
recipients := []string{
    "user1@example.com",
    "user2@example.com", 
    "user3@example.com",
}
sendBulkEmail(auth, "sender@example.com", recipients, "通知", "重要通知内容")
```

---

### 7. 错误处理和重试机制

SMTP 连接可能不稳定，加入重试逻辑：

```go
func sendWithRetry(auth smtp.Auth, addr, from string, to []string, msg []byte, maxRetries int) error {
    var lastErr error
    
    for i := 0; i < maxRetries; i++ {
        err := smtp.SendMail(addr, auth, from, to, msg)
        if err == nil {
            return nil
        }
        
        lastErr = err
        fmt.Printf("第%d次发送失败: %v\n", i+1, err)
        
        // 等待后重试
        time.Sleep(time.Second * time.Duration(i+1))
    }
    
    return fmt.Errorf("邮件发送失败，已重试%d次: %v", maxRetries, lastErr)
}
```

---

### 8. 邮件发送状态检查

检查 SMTP 服务器连接状态：

```go
func checkSMTPConnection(addr string) error {
    client, err := smtp.Dial(addr)
    if err != nil {
        return fmt.Errorf("无法连接SMTP服务器: %v", err)
    }
    defer client.Close()
    
    // 检查服务器是否支持扩展功能
    ok, err := client.Extension("AUTH")
    if err != nil {
        return fmt.Errorf("检查认证支持失败: %v", err)
    }
    
    if !ok {
        return fmt.Errorf("服务器不支持认证")
    }
    
    fmt.Println("SMTP服务器连接正常")
    return nil
}

// 使用示例
checkSMTPConnection("smtp.gmail.com:587")
```

net/smtp 是 Go 发送邮件的基础工具，虽然功能相对简单，但足以应对大多数邮件发送需求。如需处理复杂的 MIME 格式或附件，建议配合第三方库使用。