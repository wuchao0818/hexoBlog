---
title: Hexo 简介
date: 2018-12-07 16:33:08
categories: 框架简介
---

##### Hexo是一个简单、快速、强大的基于 Github Pages 的博客发布工具，支持Markdown格式，有众多优秀插件和主题。 [官方文档：http://hexo.com](http://hexo.com)

#### 安装：

###### 全局安装依赖包

```
npm install -g hexo
```

###### 创建 `myBlog`文件夹，进入文件夹

```ja
cd /Desktop/myBLog
hexo init
```

###### 生成静态文件，启动服务(默认情况下，访问网址为  `http://localhost:4000/`)

```
$ hexo g
$ hexo s
```

#### 修改主题：

本 Blog 主题采用的为  [NexT](http://theme-next.iissnan.com/) 主题

- ###### 下载主题

  ```
  $ cd /Desktop/myBLog/hexo/
  $ git clone https://github.com/iissnan/hexo-theme-next themes/next
  ```

- ###### 启用主题

  在项目根目录文件夹下的 `.config.yml` 文件里修改 `theme: next`

> 备注：如果出现一些莫名其妙的问题，可以先执行 `hexo clean` 来清理一下 public 的内容，然后再来重新生成和发布。

