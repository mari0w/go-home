---
layout: post
title: 微读 Go 标准库 - archive/tar 处理 tar 归档文件
categories: [标准库]
tags: [archive, tar, 文件压缩, 归档]
---

tar 是 Unix/Linux 系统中最常用的归档格式之一，Go 的 archive/tar 包提供了读写 tar 归档文件的能力。无论是备份数据、分发文件还是容器镜像构建，tar 格式都扮演着重要角色。本文将带你快速掌握 Go 中 tar 文件的处理方法。

---

### 1. tar 格式的基本概念

#### 工作原理

tar（Tape Archive）文件像磁带一样线性存储多个文件。每个文件由两部分组成：

1. **文件头（Header）**：512 字节的元信息
2. **文件内容**：实际数据，按 512 字节对齐

```go
import "archive/tar"

// tar.Header 的核心字段
type Header struct {
    Name     string    // 文件名
    Size     int64     // 文件大小
    Mode     int64     // 权限
    ModTime  time.Time // 修改时间
    Typeflag byte      // 文件类型
}
```

---

### 2. 创建 tar 归档

#### 基本创建步骤

```go
func createTar(output string, files []string) error {
    out, _ := os.Create(output)
    defer out.Close()
    
    tw := tar.NewWriter(out)
    defer tw.Close()
    
    for _, file := range files {
        addFile(tw, file)
    }
    return nil
}
```

---

### 3. 添加文件到归档

#### 写入单个文件

```go
func addFile(tw *tar.Writer, name string) error {
    file, _ := os.Open(name)
    defer file.Close()
    
    info, _ := file.Stat()
    header, _ := tar.FileInfoHeader(info, "")
    header.Name = name
    
    tw.WriteHeader(header)
    io.Copy(tw, file)
    return nil
}
```

---

### 4. 处理目录

#### 递归添加目录

```go
func addDir(tw *tar.Writer, dir string) error {
    return filepath.Walk(dir, func(path string, info os.FileInfo, err error) error {
        header, _ := tar.FileInfoHeader(info, "")
        header.Name = path
        tw.WriteHeader(header)
        
        if !info.IsDir() {
            file, _ := os.Open(path)
            io.Copy(tw, file)
            file.Close()
        }
        return nil
    })
}
```

---

### 5. 解压 tar 文件

#### 基本解压流程

```go
func extract(tarFile, dest string) error {
    file, _ := os.Open(tarFile)
    defer file.Close()
    
    tr := tar.NewReader(file)
    
    for {
        header, err := tr.Next()
        if err == io.EOF {
            break
        }
        
        target := filepath.Join(dest, header.Name)
        
        if header.Typeflag == tar.TypeDir {
            os.MkdirAll(target, 0755)
        } else {
            extractFile(tr, target)
        }
    }
    return nil
}
```

---

### 6. 安全解压

#### 防止路径穿越

```go
func extractFile(tr *tar.Reader, target string) error {
    // 防止 ../.. 攻击
    if strings.Contains(target, "..") {
        return fmt.Errorf("invalid path")
    }
    
    os.MkdirAll(filepath.Dir(target), 0755)
    out, _ := os.Create(target)
    defer out.Close()
    
    io.Copy(out, tr)
    return nil
}
```

---

### 7. 处理 tar.gz

#### 配合 gzip 压缩

```go
func createTarGz(output string, files []string) error {
    out, _ := os.Create(output)
    defer out.Close()
    
    gw := gzip.NewWriter(out)
    defer gw.Close()
    
    tw := tar.NewWriter(gw)
    defer tw.Close()
    
    // 添加文件...
    return nil
}
```

---

### 8. 流式处理

#### 处理大文件避免内存溢出

```go
func streamTar(src io.Reader) error {
    tr := tar.NewReader(src)
    
    for {
        header, err := tr.Next()
        if err == io.EOF {
            break
        }
        
        // tr 现在指向当前文件内容
        // 可以流式处理，不加载到内存
        processFile(header, tr)
    }
    return nil
}
```

---

### 9. 自定义文件属性

#### 设置权限和时间

```go
func customHeader(name string, data []byte) *tar.Header {
    return &tar.Header{
        Name:    name,
        Mode:    0644,
        Size:    int64(len(data)),
        ModTime: time.Now(),
        Typeflag: tar.TypeReg,
    }
}
```

---

### 10. 处理符号链接

#### 创建和还原链接

```go
// 添加符号链接
header := &tar.Header{
    Name:     "link",
    Linkname: "target",
    Typeflag: tar.TypeSymlink,
}
tw.WriteHeader(header)

// 解压时处理
if header.Typeflag == tar.TypeSymlink {
    os.Symlink(header.Linkname, target)
}
```

---

### 小结

archive/tar 包提供了完整的 tar 文件处理能力。记住核心要点：

1. **Writer/Reader 模式**：创建用 Writer，读取用 Reader
2. **流式处理**：适合大文件，避免内存占用
3. **安全检查**：解压时防止路径穿越
4. **压缩支持**：配合 gzip/bzip2 实现压缩
5. **元信息保留**：可保持文件权限、时间戳

tar 格式虽然古老但依然强大，在容器镜像、备份系统等场景中广泛应用。掌握 archive/tar 包，让你的 Go 程序轻松处理各种归档需求。