# 文件删除漏洞场景

## 跨目录删除
未净化用户输入导致目录遍历删除
```java
// 漏洞：用户输入直接拼接路径
String filename = request.getParameter("filename");
File file = new File(baseDir, filename);
// 可能是 ../../../important.txt
// 修复：校验 filename，路径规范化后校验是否在允许目录内
```