

# idea使用技巧(2022.2)

## 1.  设置自定义注释模板(创建类时自动加上)

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

## 2. Debug技巧，step into 和 force step info

- step into 是进入自己写的代码的里面，比如调用xxx方法。servcie.xxx。step into就可以正常进入到xxx方法。单如果调用的方法是原生的，这时step into 就和step over（跳转到下一行）一样。
- force step into 相当于强化版的step into 当你调用某个原生的方法时，使用force step into 可以进入源码中。download之后慢慢看。

## 3. 设置idea通过滚轮缩放放大code

> file-->settings-->Editor-->General-->Mouse Control--> change font size  with Ctrl +Mouse Wheel in:

## 4. idea设置打开时选则项目

> File-->Settings-->Appearance&Behavior-->System Settings-->

> 在Project中有一个Repen projects on startup选项。

> 取消勾选即可.



## 5. 切换项目时询问

> File-->Settings-->Appearance&Behavior-->System Settings-->

> 勾选 Comfirm before exiting the IDE

> Open project in new window 打开新窗口
>
> Open project in the same window 打开在当前窗口





