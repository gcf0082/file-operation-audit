# 临时文件清理

上传或下载过程中生成的临时文件必须及时清理，否则会耗尽磁盘空间或泄露数据。

## 检查点

- 使用 `File.createTempFile()` 后是否在 finally/try-with-resources 中删除
- 文件流使用后是否关闭并清理临时文件
- `deleteOnExit()` 只在 JVM 退出时删除，不适合长时间运行的服务

```java
// 漏洞
File temp = File.createTempFile("upload_", ".tmp");
// 使用 temp 文件...
// 缺少 cleanup

// 修复
File temp = File.createTempFile("upload_", ".tmp");
try {
    // 使用 temp 文件...
} finally {
    temp.delete();
}
```