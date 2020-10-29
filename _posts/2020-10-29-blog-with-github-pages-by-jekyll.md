---
title: "使用 Jekyll 与 github pages 搭建个人博客"
layout: post
author: "jlshix"
header-style: text
tags:
  - jekyll
---

之前的个人博客是使用 `Hexo` 搭建的, 使用 `NexT` 主题.

`Hexo` 作为静态站点生成工具, 需要将编写的文章转换为一个个的静态页面, 然后将转换结果
提交至 github 上以 `yourname.github.io` 命名的 repo 上, 访问
`http://yourname.github.io` 即可.

这样的不方便之处在于文章原文若要进行版本管理还要提交到另外一个分支上.

前段时间才发现 github pages 早就支持了 `Jekyll` 作为静态模板转换工具.
只要将项目文件提交至 github, github 就会自动生成静态页面并提供服务.


## Jekyll 是什么

[Jekyll 究竟是什么?](https://jekyllcn.com/docs/home/#Jekyll-%E7%A9%B6%E7%AB%9F%E6%98%AF%E4%BB%80%E4%B9%88)

> Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，
> 通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，
> 你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，
> 也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

作为静态网站, 无数据库支持, 所有的文章都是静态页面. 如果想加入评论功能(前提是有人看文章-_-),
可以借助第三方服务, 如多说和 disqus.


## 安装

### 安装 Ruby

和 `Hexo` 使用 js 编写不同, `Jekyll` 是使用 Ruby 编写的. 所以在安装前需要先在本机安装 Ruby.

关于 Ruby 的安装, 在[此页面](https://www.ruby-lang.org/en/documentation/installation/)
有详细的说明. 开头就说了 `You may already have Ruby installed on your computer. `
所以可以先在终端中输入 `ruby -v` 检查一下.

Mac 或 Linux 用户可按平台执行相应命令, windows 用户可使用
[RubyInstaller](https://rubyinstaller.org/)

安装完成后就可以使用 `ruby` 和 `gem` 命令了, 类似于 `python` 和 `pip`.


### 安装 Jekyll

打开 [Jekyll官网](https://jekyllrb.com/), 如无法访问可访问[中文翻译网页](https://jekyllcn.com/).

在醒目的位置标出了安装的命令, 执行即可:

```shell
gem install jekyll bundler

jekyll new my-awesome-site

cd my-awesome-site

tree
# 目录结构如下
# .
# ├── 404.html
# ├── Gemfile
# ├── Gemfile.lock
# ├── _config.yml
# ├── _posts
# │   └── 2020-10-29-welcome-to-jekyll.markdown
# ├── about.markdown
# └── index.markdown

bundle install

bundle exec jekyll serve
# 此时打开浏览器 http://localhost:4000 即可预览网页.
```

## 配置与博客编写

网站的配置文件存放于 `_config.yml`, 如果不熟悉 YAML 语法, 可以参见:

- [Learn X in Y minutes where X=yaml](https://learnxinyminutes.com/docs/zh-cn/yaml-cn/)
- [YAML语言教程-阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/07/yaml.html)

配置项一目了然, 按需修改即可, 可参见官方指南.

编写博客可以使用 `markdown`, 但需要包含头信息才能被 jekyll 作为博文解析, 如:

```markdown
---
title: "使用 Jekyll 与 github pages 搭建个人博客"
layout: post
author: "jlshix"
header-style: text
tags:
  - Jekyll
---

正文...
```

头信息参见[官方文档](https://jekyllrb.com/docs/front-matter/), 实际上根据主题的不同
是可变的. 所定义的变量会在主题的实现中引用, 并在渲染时体现.


## 发布到 github pages

1. 如果之前未在 github 建立以 `yourname.github.io` 为名称的 repo, 新建一个.

2. 首次推送需在文件夹中执行以下命令与远程库关联:

```shell
git init
git remote add origin git@github.com/yourname/yourname.github.io.git
```

3. 之后撰写文章, 执行 `bundle exec jekyll serve` 预览站点.

4. 重复 `3` 直至满意文章.

5. 提交并推送至远程库.

```shell
git add .
git commit -m "your commit message"
git push origin master
```


## 参考

- 更多配置选项 [configuration](https://jekyllrb.com/docs/configuration/)

- 我使用的主题 [Hux Theme](https://github.com/Huxpro/huxpro.github.io) 及其
[文档](https://github.com/Huxpro/huxpro.github.io/blob/master/_doc/README.zh.md)

- [中文翻译网站](http://jekyllcn.com/)

- [Github Pages 使用入门](https://docs.github.com/cn/free-pro-team@latest/github/working-with-github-pages/getting-started-with-github-pages)

- [Liquid template language - Shopify](https://shopify.github.io/liquid/)
