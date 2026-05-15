# 文件上传漏洞场景

## 类型绕过
检测客户端/服务器端类型校验不严
```java
// 漏洞：仅检查 Content-Type
String type = request.getContentType();
if (type.equals("image/jpeg")) {
    // 直接保存，未校验扩展名
}
// 修复：校验扩展名+Content-Type+文件头
```

## 跨目录上传
未净化用户输入导致目录遍历
```java
// 漏洞：用户输入直接拼接路径
String filename = request.getParameter("filename");
File file = new File(uploadDir, filename);  // 可能是 ../../../etc/passwd
// 修复：校验 filename 不含 .. ，使用真实路径规范化
Path resolved = basePath.resolve(filename).normalize();
if (!resolved.startsWith(basePath)) throw new SecurityException();
```

## 跨目录删除（上传场景）
上传后自动删除原文件时发生目录遍历
```java
// 漏洞：删除旧文件时未校验路径
String filename = request.getParameter("filename");
deleteFile(uploadDir + filename);  // 可能是 ../../../important.txt
```

## 压缩包跨目录
解压时未校验内部路径导致目录遍历
```java
// 漏洞：解压时未处理 ../ 路径
ZipEntry entry = zipFile.getNextEntry();
while (entry != null) {
    File outFile = new File(outputDir, entry.getName());
    // 未校验 outFile 是否在 outputDir 外
    extract(entry, outFile);
}
```
修复：解压前校验路径

## 高压缩比 DoS
解压时无限制展开导致资源耗尽
```java
// 漏洞：未限制解压比例
while ((entry = zipFile.getNextEntry()) != null) {
    // 递归解压或无限展开
}
// 修复：限制解压大小、层级、数量
```

## 超大文件 DoS
未限制文件大小导致磁盘耗尽
```java
// 漏洞：直接写入
InputStream is = request.getInputStream();
FileOutputStream fos = new FileOutputStream(file);
is.transferTo(fos);  // 无大小限制
// 修复：限制最大 size
```

## 各类文件 DoS
特定文件格式导致服务器资源耗尽（XML实体注入、渐进式图片等）
```java
// 漏洞：解析用户上传的 XML/JSON/图片
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
Document doc = dbf.newDocumentBuilder().parse(file);
// 修复：限制解析时间、内存、实体数量
```

## 临时文件未删除
上传后临时文件未清理
```java
// 漏洞：使用临时文件但未删除
File temp = File.createTempFile("upload_", ".tmp");
temp.deleteOnExit();  // JVM 退出才删
// 修复：显式删除或使用内存流
```

## 泄露绝对路径
错误信息包含服务器路径
```java
// 漏洞：异常信息泄露路径
catch (Exception e) {
    response.getWriter().write("上传失败: " + e.getMessage());
    // e.getMessage() 可能包含 /var/www/uploads/
}
// 修复：只返回通用错误信息
```

## 文件完整性校验
上传文件可能被篡改
```java
// 漏洞：未校验完整性
FileInputStream fis = new FileInputStream(uploadedFile);
// 直接使用，未校验是否被篡改
// 修复：计算并校验 hash 或使用数字签名
```

## 压缩包软链接
压缩包内含软链接指向敏感文件
```java
// 漏洞：解压时未检测软链接
ZipEntry entry = zipFile.getNextEntry();
if (entry.isSymbolicLink()) {
    // 直接创建软链接，可能指向 /etc/passwd
}
// 修复：解压前检查文件类型，拒绝软链接
```