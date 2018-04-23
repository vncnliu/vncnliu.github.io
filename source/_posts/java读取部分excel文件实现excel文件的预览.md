---
title: java读取部分excel文件实现excel文件的预览
date: 2016-04-20 13:59:43
category:
- 程序猿的attack
tag:
- poi
---

一个报表导入的需求要求在保存导入数据前可以预览部分数据，本来直接用poi从文件得到workbook后直接取前十行返回预览。做完后发现在excel表格非常大的时候从流构建workbook要挺长时间，体验有点不好。要是能读取部分流展示就完美了。看了一下poi构建workbook的源码，自己写了个测试用例结果失败了......好在有google找到了这个[Excel Streaming Reader](https://github.com/monitorjbl/excel-streaming-reader)使用起来还是比较简单的
添加maven库
```xml
<dependencies>
  <dependency>
    <groupId>com.monitorjbl</groupId>
    <artifactId>xlsx-streamer</artifactId>
    <version>0.2.12</version>
  </dependency>
</dependencies>
```
使用示例
```java
import com.monitorjbl.xlsx.StreamingReader;

InputStream is = new FileInputStream(new File("/path/to/workbook.xlsx"));
StreamingReader reader = StreamingReader.builder()
        .rowCacheSize(100)    // number of rows to keep in memory (defaults to 10)
        .bufferSize(4096)     // buffer size to use when reading InputStream to file (defaults to 1024)
        .sheetIndex(0)        // index of sheet to use (defaults to 0)
        .read(is);            // InputStream or File for XLSX file (required)

for (Row r : reader) {
  for (Cell c : r) {
    System.out.println(c.getStringCellValue());
  }
}
}
```
比较遗憾的是这个不支持excel2003
以上是读取，当导出时文件过大导致OOM可以用
```java
//内存中只保留1000条记录
SXSSFWorkbook workbook = new SXSSFWorkbook(1000);
```