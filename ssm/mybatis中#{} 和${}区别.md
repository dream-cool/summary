#mybatis中  #{} 和${}区别



#### mapperXML 中select标签如下

![../image/mybatis中#{} 和${}区别/1.png]()



####经过mybatis解析后生成的sqlSource如下

![../image/mybatis中#{} 和${}区别/2.png]()



#### 在SQL执行的过程中，将通过MappedStatement.getBoundSql来过去对应SQL的BoundSql对象

![../image/mybatis中#{} 和${}区别/3.png]()



####rootSqlNode.apply(context)目的时替换${} 

![../image/mybatis中#{} 和${}区别/4.png]()



####sqlSourceParser.parse目的时替换#{}标签

![../image/mybatis中#{} 和${}区别/5.png]()



####最终替换对应标签后BoundSql对象中的sql内容为

![../image/mybatis中#{} 和${}区别/6.png]()



#### 结论

${}只会将传过来的参数对象值替换对应的参数标签，而#{}会将对应参数的标签替换成预编译语句中的占位符?,再最后再将对应值替代占位符，防止SQL注入 

