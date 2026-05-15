# 文件下载漏洞场景

## 跨目录下载
未净化用户输入导致目录遍历
```java
// 漏洞：用户输入直接拼接路径
String filename = request.getParameter("filename");
File file = new File(downloadDir, filename);
// 可能是 ../../../etc/passwd
// 修复：校验 filename，路径规范化后校验是否在允许目录内
```

## 泄露绝对路径
错误信息或日志泄露服务器路径
```java
// 漏洞：异常时泄露路径
catch (IOException e) {
    log.error("文件读取失败: " + file.getAbsolutePath());
    // 日志包含 /var/www/downloads/
}
// 修复：只记录通用信息，不记录真实路径
```

## 临时文件未删除
下载过程中产生的临时文件未清理
```java
// 漏洞：使用临时文件
File temp = new File(System.getProperty("java.io.tmpdir"), "download_" + id);
writeToFile(temp);
// 下载完成后未删除
// 修复：使用内存流或显式删除临时文件
```

## 敏感信息泄露
下载的文件内容包含敏感信息
```java
// 漏洞：用户可下载包含敏感数据的文件
File file = findFile(request.getParameter("id"));
// 文件可能是日志、配置备份、session等
// 修复：下载前检查文件权限、敏感标记
```