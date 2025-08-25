---
layout: post
title: Go语言encoding/json包实用技巧总结
categories: [Go标准库]
tags: [Go, JSON, 编程技巧]
---

Go语言的encoding/json包是处理JSON数据的标准库，除了基础的Marshal和Unmarshal功能外，还提供了许多实用的特性。本文整理了10个常用技巧，帮助开发者更高效地处理JSON数据。

---

## 1. 基础序列化 - 结构体与JSON互转

最常用的JSON操作，但很多人不知道的细节：

```go
type User struct {
    Name  string `json:"name"`           // 字段映射
    Email string `json:"email,omitempty"` // 空值忽略
    Age   int    `json:"-"`              // 忽略字段
}

// 序列化
user := User{Name: "张三", Email: "test@example.com"}
data, _ := json.Marshal(user)

// 反序列化
var newUser User
json.Unmarshal(data, &newUser)
```

---

## 2. 动态JSON处理 - 不定结构也能搞定

遇到不确定的JSON结构？用interface{}和类型断言：

```go
// 处理任意JSON
var result map[string]interface{}
json.Unmarshal(jsonData, &result)

// 类型断言获取值
if name, ok := result["name"].(string); ok {
    fmt.Println("Name:", name)
}

// 嵌套结构
if items, ok := result["items"].([]interface{}); ok {
    for _, item := range items {
        // 处理每个item
    }
}
```

---

## 3. 自定义序列化 - 控制输出格式

需要特殊的JSON格式？实现Marshaler接口：

```go
type Timestamp time.Time

func (t Timestamp) MarshalJSON() ([]byte, error) {
    stamp := fmt.Sprintf("\"%s\"", time.Time(t).Format("2006-01-02 15:04:05"))
    return []byte(stamp), nil
}

func (t *Timestamp) UnmarshalJSON(data []byte) error {
    str := string(data)
    str = str[1 : len(str)-1] // 去除引号
    parsed, err := time.Parse("2006-01-02 15:04:05", str)
    *t = Timestamp(parsed)
    return err
}
```

---

## 4. 流式处理 - 大文件不怕内存爆

处理大型JSON文件，一次性加载会爆内存？用Decoder流式处理：

```go
// 读取大文件
file, _ := os.Open("large.json")
decoder := json.NewDecoder(file)

// 逐个解析
for decoder.More() {
    var item DataItem
    if err := decoder.Decode(&item); err != nil {
        break
    }
    // 处理单个item，不会全部加载到内存
    processItem(item)
}
```

---

## 5. JSON Number - 精度不丢失

处理大数字或需要精确控制数值类型？用json.Number：

```go
type Config struct {
    ID    json.Number `json:"id"`
    Price json.Number `json:"price"`
}

// 使用
var config Config
decoder := json.NewDecoder(strings.NewReader(jsonStr))
decoder.UseNumber() // 启用Number类型
decoder.Decode(&config)

// 按需转换
id, _ := config.ID.Int64()
price, _ := config.Price.Float64()
```

---

## 6. 嵌入式结构 - 代码复用神器

避免重复定义相同字段，用嵌入式结构：

```go
type BaseResponse struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
}

type UserResponse struct {
    BaseResponse // 嵌入
    Data User    `json:"data"`
}

// JSON输出会平铺所有字段
// {"code":200,"message":"success","data":{...}}
```

---

## 7. RawMessage - 延迟解析

不确定某个字段的具体类型？先存起来后面再解析：

```go
type Message struct {
    Type string          `json:"type"`
    Data json.RawMessage `json:"data"` // 原始JSON
}

// 根据type决定如何解析data
var msg Message
json.Unmarshal(jsonData, &msg)

switch msg.Type {
case "user":
    var user User
    json.Unmarshal(msg.Data, &user)
case "order":
    var order Order
    json.Unmarshal(msg.Data, &order)
}
```

---

## 8. 美化输出 - 调试必备

需要人类可读的JSON输出？用MarshalIndent：

```go
// 普通输出
data, _ := json.Marshal(user)
// {"name":"张三","age":25}

// 美化输出
prettyData, _ := json.MarshalIndent(user, "", "  ")
// {
//   "name": "张三",
//   "age": 25
// }
```

---

## 9. 验证JSON - 避免panic

处理不可信的JSON数据？先验证再解析：

```go
// 检查JSON是否有效
if json.Valid(jsonData) {
    var result Data
    json.Unmarshal(jsonData, &result)
} else {
    // 处理无效JSON
    return errors.New("invalid JSON")
}

// 严格模式解析
decoder := json.NewDecoder(reader)
decoder.DisallowUnknownFields() // 遇到未知字段报错
err := decoder.Decode(&result)
```

---

## 10. 性能优化 - 提速技巧

处理大量JSON时的性能优化技巧：

```go
// 复用Encoder/Decoder
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func FastMarshal(v interface{}) []byte {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buf)
    buf.Reset()
    
    encoder := json.NewEncoder(buf)
    encoder.Encode(v)
    return buf.Bytes()
}

// 预分配切片容量
var items []Item
json.Unmarshal(data, &items)
// 如果知道大概数量，预分配更高效
items = make([]Item, 0, 1000)
```

---

## 总结

以上介绍了encoding/json包的10个实用技巧，涵盖了从基础使用到高级特性的各个方面。合理运用这些功能可以让JSON处理代码更加简洁、高效和健壮。在实际开发中，根据具体场景选择合适的方法，能够有效提升开发效率和代码质量。