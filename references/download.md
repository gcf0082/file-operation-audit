# 文件下载漏洞场景

## 泄露绝对路径
错误信息或日志泄露服务器路径
```java
// 漏洞：异常时泄露路径
catch (IOException e) {
    log.error("文件读取失败: " + file.getAbsolutePath());
}
// 修复：只记录通用信息
```

## 敏感信息泄露
下载的文件内容包含敏感信息
```java
// 漏洞：用户可下载包含敏感数据的文件
File file = findFile(request.getParameter("id"));
// 文件可能是日志、配置备份、session等
// 修复：下载前检查文件权限、敏感标记
```