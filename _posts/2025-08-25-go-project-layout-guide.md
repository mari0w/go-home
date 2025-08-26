---
layout: post
title: Go项目目录结构最佳实践指南
categories: [最佳实践]
tags: [Go, 项目结构, 工程化]
---

在Go项目开发中，良好的目录结构对项目的可维护性和团队协作至关重要。本文基于社区广泛认可的项目布局标准，介绍Go项目的目录组织最佳实践。

## 1. 核心应用目录 - /cmd

项目的主要应用程序入口点应放在/cmd目录下。每个应用程序都应该有自己的子目录：

```go
/cmd
  /myapp
    main.go     // package main
  /myworker
    main.go     // package main
```

每个子目录名应该与生成的可执行文件名一致。main.go文件应该尽量简洁，实际的业务逻辑应该导入自/internal或/pkg目录。

## 2. 私有代码目录 - /internal

私有应用代码和库代码放在/internal目录。Go编译器会强制执行这个目录的访问控制，其他项目无法导入internal目录下的包：

```go
/internal
  /app
    /myapp        // 应用私有代码
  /pkg
    /database     // 内部共享库
    /auth         // 认证模块
```

这是Go语言级别的保护机制，确保内部实现不会被外部项目依赖。

## 3. 公共库目录 - /pkg

可以被外部项目导入的库代码放在/pkg目录：

```go
/pkg
  /httputil       // HTTP工具库
  /stringutil     // 字符串处理工具
  /errors         // 错误处理包
```

使用/pkg目录时要谨慎，确保这里的代码确实适合对外公开，并且有良好的API设计和文档。

## 4. 依赖管理 - /vendor

使用vendor目录管理项目依赖（如果启用了vendor模式）：

```bash
# 启用vendor模式
go mod vendor

# 使用vendor构建
go build -mod=vendor
```

现代Go项目通常使用Go Modules，vendor目录变为可选。

## 5. API定义目录 - /api

API定义文件、协议文件放在/api目录：

```
/api
  /openapi
    swagger.yaml     # OpenAPI规范
  /proto
    user.proto       # Protocol Buffers定义
  /graphql
    schema.graphql   # GraphQL schema
```

## 6. 配置文件目录 - /configs

配置文件模板或默认配置：

```yaml
# /configs/config.yaml
server:
  host: localhost
  port: 8080
database:
  driver: postgres
  dsn: postgres://localhost/mydb
```

注意：实际的配置文件（包含敏感信息）不应该提交到版本控制。

## 7. 部署相关目录

部署和构建相关的文件分别组织在不同目录：

```
/build
  /ci              # CI配置文件
    .travis.yml
  /package         # 打包脚本
    Dockerfile

/deployments
  /docker-compose
    docker-compose.yml
  /kubernetes
    deployment.yaml
```

## 8. 脚本目录 - /scripts

各种构建、安装、分析等脚本：

```bash
/scripts
  build.sh         # 构建脚本
  install.sh       # 安装脚本
  test.sh          # 测试脚本
  lint.sh          # 代码检查
```

## 9. 测试目录 - /test

额外的外部测试应用和测试数据：

```go
/test
  /integration     // 集成测试
  /e2e            // 端到端测试
  /testdata       // 测试数据文件
  /mocks          // 模拟对象
```

单元测试应该与被测试的代码在同一个包中，使用_test.go后缀。

## 10. 其他常用目录

文档和工具相关目录：

```
/docs           # 项目文档
  design.md     # 设计文档
  api.md        # API文档

/tools          # 项目工具
  /gen          # 代码生成工具

/examples       # 示例代码
  simple.go     # 基础示例
  advanced.go   # 高级用法

/assets         # 静态资源
  /images       # 图片
  /templates    # 模板文件
```

## 11. 不推荐的做法

避免以下目录结构：

```
# 不要使用src目录
/src  ❌

# 避免在根目录放置过多Go文件
main.go  ❌ (除非是简单的单文件项目)
server.go ❌
handler.go ❌
```

Go项目不需要Java风格的src目录，直接在项目根目录组织代码即可。

## 12. 实际项目示例

一个典型的Web服务项目结构：

```
myproject/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handler/
│   │   └── user.go
│   ├── model/
│   │   └── user.go
│   └── service/
│       └── user.go
├── pkg/
│   └── logger/
│       └── logger.go
├── api/
│   └── openapi.yaml
├── configs/
│   └── config.yaml
├── deployments/
│   └── docker-compose.yaml
├── scripts/
│   └── build.sh
├── go.mod
├── go.sum
└── README.md
```

---

## 总结

良好的项目结构能够提高代码的可维护性、可测试性和团队协作效率。虽然这不是Go官方强制的标准，但已经被社区广泛采用。在实际项目中，应该根据项目规模和团队需求，选择合适的目录结构，避免过度设计。小型项目可以从简单结构开始，随着项目增长逐步完善目录组织。