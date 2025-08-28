---
layout: post
title: 微读Go标准库 - net/url包的高级特性与实用技巧
categories: [标准库]
tags: [net/url, URL解析, 安全, 查询参数]
description: "深入探索Go net/url包的高级特性：密码自动脱敏、智能路径拼接、IPv6地址处理等实用技巧，让URL处理更安全优雅。"
permalink: /posts/2025/08/go-net-url-advanced-features-guide/
---

你可能每天都在用 `url.Parse()`，但 net/url 包里还藏着很多不为人知的宝藏功能。密码自动脱敏、智能路径拼接、IPv6地址处理...这些高级特性能让你的URL处理代码更安全、更优雅。

---

### 1. 不只是 Parse 那么简单

大多数人只用 `url.Parse()`，但实际上还有更严格的 `ParseRequestURI()`：

```go
// Parse 比较宽松
u1, _ := url.Parse("//example.com/path")

// ParseRequestURI 更严格，适用于 HTTP 请求
u2, _ := url.ParseRequestURI("https://example.com/path")
```

`ParseRequestURI` 会拒绝相对 URL，确保安全性。

---

### 2. 密码自动脱敏神器

```go
u, _ := url.Parse("https://user:secret@example.com/path")
fmt.Println(u.String())    // https://user:secret@example.com/path
fmt.Println(u.Redacted())  // https://user:xxxxx@example.com/path
```

日志记录时再也不用担心密码泄露了。

---

### 3. 智能路径拼接

别再用字符串拼接 URL 了：

```go
u, _ := url.Parse("https://api.com/v1")
// 自动处理 /，清理 ./ 和 ../
newURL := u.JoinPath("users", "../admin", "./list")
// 结果: https://api.com/v1/admin/list
```

自动处理路径清理，避免目录遍历攻击。

---

### 4. 查询参数的高级玩法

```go
values := url.Values{}
values.Add("tag", "golang")
values.Add("tag", "tutorial")  // 支持多值
values.Set("page", "1")        // 覆盖设置

// 批量操作
params := url.Values{
    "filters": []string{"new", "hot"},
    "limit":   []string{"10"},
}
```

`Add` vs `Set` 的区别很重要。

---

### 5. IPv6 地址处理

```go
u, _ := url.Parse("http://[::1]:8080/path")
fmt.Println(u.Hostname()) // ::1 (去掉方括号)
fmt.Println(u.Port())     // 8080
```

自动处理 IPv6 的方括号语法。

---

### 6. 相对 URL 智能解析

```go
base, _ := url.Parse("https://example.com/dir/")
rel, _ := url.Parse("../other/file.html")
resolved := base.ResolveReference(rel)
// 结果: https://example.com/other/file.html
```

比手动字符串处理可靠多了。

---

### 7. 转义的艺术

```go
// 路径转义 - 不转义 /
path := url.PathEscape("hello world/file name.txt")
// hello%20world/file%20name.txt

// 查询转义 - 转义所有特殊字符
query := url.QueryEscape("key=value&other")
// key%3Dvalue%26other
```

不同场景用不同的转义方法。

---

### 8. 获取原始编码

```go
u, _ := url.Parse("https://example.com/hello%20world")
fmt.Println(u.Path)        // hello world (解码后)
fmt.Println(u.EscapedPath()) // hello%20world (保留编码)
```

有时需要保持原始 URL 编码不变。

---

### 9. 安全的查询解析

```go
// 直接解析可能有问题
params, err := url.ParseQuery("malformed%")
if err != nil {
    // 处理解析错误
}

// 更安全的方式
u, _ := url.Parse("http://example.com?key=value")
values := u.Query() // 自动处理错误
```

始终检查解析错误，避免恶意输入。

这些特性让 URL 处理更安全、更可靠。别再用字符串拼接了！