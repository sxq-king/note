## 前言

需要环境有 git node.js GitHub 

## 修改npm下载源

```.java
npm config set registry https://registry.npm.taobao.org
```

## 安装Hexo依赖

```.java
npm install -g hexo-cli
```

安装完成可使用以下命令验证是否完成

```.java
hexo -v
```

## 创建GitHub仓库

选择 `New repository`，创建一个 `<用户名>.github.io` 的仓库。

***注意***

> - 仓库的格式必须为：<用户名>.github.io (否则无法正常访问)

## 将本地Git连接到GitHub

[Git基本操作](ProjectMD/Git基本操作.md)

连接完成可以使用以下命令测试一波

```.java
ssh -T git@github.com
```

## hexo 项目

```hexo init 项目名```

**项目结构**

【node_modules】：依赖包

【scaffolds】：生成文章的一些模板

【source】：用来存放你的文章

【themes】：主题

【.npmignore】：发布时忽略的文件（可忽略）

【_config.landscape.yml】：主题的配置文件

【config.yml】：博客的配置文件

【package.json】：项目名称、描述、版本、运行和开发等信息

## 项目启动

命令

```.java
hexo server 或 hexo s
```

## 将静态页面改在到GitHub Pages

1. 安装 hexo-deployer-git

   ```.java
   npm install hexo-deployer-git --save
   ```

2. 修改_config.yml文件

**注意坑点**

最好复制，别手敲，很容易如下错误

```.java
You have to configure the deployment settings in _config.yml first!

Example:
  deploy:
    type: git
    repo: <repository url>
    branch: [branch]
    message: [message]

    extend_dirs: [extend directory]

For more help, you can check the docs: http://hexo.io/docs/deployment.html
INFO  Deploy done: git

```



```.java
deploy:
  type: git
  repo: git@github.com:sxq-king/sxq-king.github.io.git
  branch: master
```



> 在 项目名 目录下的_config.yml，就是整个 Hexo 框架的配置文件了。可以在里面修改大部分的配置。详细可参考官方的[配置描述](https://hexo.io/zh-cn/docs/configuration)。

>  修改最后一行的配置，将 repository 修改为你自己的 github 项目地址即可，还有分支要改为 `main` 代表主分支（注意缩进）。

3. 修改配置，运行以下命令将代码部署到GitHub

```.java
hexo clean && hexo generate && hexo deploy  // Git BASH终端
hexo clean; hexo generate; hexo deploy  // VSCODE终端
```

**命令详解**

- hexo clean：删除之前生成的文件，若未生成过静态文件，可忽略此命令。
- hexo generate：生成静态文章，可以用 `hexo g` 缩写
- hexo deploy：部署文章，可以用 `hexo d` 缩写