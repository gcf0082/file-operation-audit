# 文件读取漏洞场景

## 跨目录读取
未净化用户输入导致目录遍历
```java
// 漏洞：用户输入直接拼接路径
String filepath = request.getParameter("path");
File file = new File(baseDir, filepath);
// 可能是 ../../../etc/passwd
// 修复：路径规范化后校验是否在允许目录内
```

## 越权读取
未校验用户对文件的访问权限
```java
// 漏洞：只校验文件存在，未校验权限
File file = new File(baseDir, filename);
if (file.exists()) {
    return readFile(file);  // 任意用户可读任意文件
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