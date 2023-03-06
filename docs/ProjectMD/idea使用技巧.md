

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

## 6.自定义快捷键模板

- File-->Settings-->Editor-->Live Templates -->

- 点击+号，新建一个我们自定义的快捷键组--> 选中我们创建的组，再点击加号创建快捷键模板

- Abbreviation:快捷键

- Template text:快捷键模板

- Description: 快捷键描述

- 编辑完后Define 选择快捷键模板对应的语言

- apply完成

- 例1快速try-catch-finally

  ```java
  try {
  
  }catch (Exception e){
      e.printStackTrace();
  }finally {
  
  }
  ```

- 例2 计算耗时

  ```java
  long startTime = System.currentTimeMillis();
  long endTime = System.currentTimeMillis();
  System.out.println("------costTime:" + (endTime - startTime) + "毫秒");
  ```

- 例3 使用lambda表达式形式创建新线程

  ```java
    new Thread(() -> {

    }, "t1").start();
  ```

- 例4 暂停几秒

    ```java
    try{TimeUnit.SECONDS.sleep($end$);}catch (InterruptedException e){e.printStackTrace();}
    ```

## 7.大小写切换

```java
ctrl+shift+u
```

## 8.全局搜索 and 局部搜索

> ctrl + shift + f 全局
>
> ctrl + f 局部

9.全局替换 and 局部替换

> ctrl + shift + r 全局
>
> ctrl + r 局部

