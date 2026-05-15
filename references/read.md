# 文件读取漏洞场景

## 越权读取
未校验用户对文件的访问权限
```java
// 漏洞：只校验文件存在，未校验权限
File file = new File(baseDir, filename);
if (file.exists()) {
    return readFile(file);
}
// 修复：校验用户是否有权访问该文件
```

## 泄露绝对路径
错误信息或响应头泄露路径
```java
// 漏洞：文件不存在时暴露路径
if (!file.exists()) {
    response.sendError(404, "File not found: " + file.getAbsolutePath());
}
// 修复：使用通用错误信息，不暴露路径
```