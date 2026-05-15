# 跨目录漏洞场景

所有跨目录攻击共享同一根因：用户输入未净化直接拼接路径，导致 `../` 逃逸。

## 跨目录上传
```java
// 漏洞
String filename = request.getParameter("filename");
File file = new File(uploadDir, filename);
// 修复：路径规范化后校验
Path resolved = basePath.resolve(filename).normalize();
if (!resolved.startsWith(basePath)) throw new SecurityException();
```

## 跨目录删除（上传场景）
上传后自动删除原文件时发生目录遍历
```java
// 漏洞
deleteFile(uploadDir + request.getParameter("filename"));
```

## 压缩包跨目录（Zip Slip）
解压时未校验条目路径导致覆盖任意文件
```java
// 漏洞
ZipEntry entry = zipFile.getNextEntry();
File outFile = new File(outputDir, entry.getName());
// 修复：解压前规范化并校验路径
```

## 跨目录下载
```java
// 漏洞
File file = new File(downloadDir, request.getParameter("filename"));
```

## 跨目录读取
```java
// 漏洞
File file = new File(baseDir, request.getParameter("path"));
```

## 跨目录删除
```java
// 漏洞
File file = new File(baseDir, request.getParameter("filename"));
```

## 修复模式（通用）
```java
Path resolved = baseDir.resolve(userInput).normalize();
if (!resolved.startsWith(baseDir)) {
    throw new SecurityException("非法路径");
}
```