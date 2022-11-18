

# idea使用技巧

1. 设置自定义注释模板(创建类时自动加上)

> file --> setting --> Editor --> File and Code Templates --> 找到想加注释模板的文件类型 

例Java中的 class 文件

```java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")

/**
 * @author ${Your Name}
 * @date ${DATE} ${TIME}
 * @version 1.0
 */
 
public class ${NAME} {
}
```

2. 