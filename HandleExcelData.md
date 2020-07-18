



# 处理Excel表格数据

## 依赖

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.17</version>
</dependency>
```



## 遍历处理Excel表格数据

```java
		List<String> list = new ArrayList<>();
        //读取Excel文件
        XSSFWorkbook excel = new XSSFWorkbook(new FileInputStream(sourceFilePath));
        BufferedWriter bw = new BufferedWriter(new FileWriter(new File(targetFilePath)));
        //遍历Excel文件中对应的表格
        for (Sheet sheet : excel) {
            Iterator<Row> it = sheet.rowIterator();
            //遍历表格中的每一行
            while (it.hasNext()) {
                Row row = it.next();
                //遍历对应行的列
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < row.getPhysicalNumberOfCells(); i++) {
                    Cell cell = null;
                    //拼接每行的内容
                    if ((cell = row.getCell(i)) != null) {
                        sb.append(cell);
                        // TODO: 2020/1/2 针对Excel数据表格自定义逻辑操作
                        if (i < row.getPhysicalNumberOfCells()-1){
                            sb.append("#####");
                        }
                    }
                    //将操作好的数据写入新的文件中
                    bw.write(sb.toString());
                    bw.newLine();
                    bw.flush();
                }
            }
        }
```

## 处理的数据重新转换Excel表格

对于重新写入txt文件的数据可以通过idea的列选择赋值操作，逐一将列的数据重新复制到新的Excel表格中



## 至此，Excel数据便处理完毕