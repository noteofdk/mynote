---
title: "Hexo 构建静态博客"
date: 2016-07-26 09:56:01
categories: [util]
tags: hexo
img: https://pbs.twimg.com/profile_images/476729162707644418/mQZOTo9f_200x200.png
---

Hexo 是个快速、简单且功能强大的 Node.js 博客框架。

> A fast, simple & powerful blog framework, powered by Node.js.

<!-- more -->

今天偶然看到一个名为[杰风居](http://jayfeng.com/)的博客，喜欢那种风格，就意外发现了 Hexo。进而找到[淡忘~浅思](http://www.ido321.com/)这篇博文[《简洁轻便的博客平台: Hexo详解》](http://www.ido321.com/1650.html)。后来又发现了这篇[hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)。兴奋之至，特此纪录。

# Hexo
## [官方文档](https://hexo.io/zh-cn/docs/)

## 安装
由于 Hexo 是基于 Node，安装前要先安装 Node。
安装 Hexo 要用全局安装：加 `-g` 参数

``` shell
npm install -g hexo
```

- `hexo help` : 查看帮助

## 使用
安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```
$ hexo init <folder>
$ cd <folder>
$ npm install
```

新建完成后，指定文件夹的目录如下：

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

### _config.yml
网站的 配置 信息，可以在此配置大部分的参数。

### scaffolds
模版 文件夹。当新建文章时，Hexo 会根据 scaffold 来建立文件。

### source
资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。

### themes
主题 文件夹。Hexo 会根据主题来生成静态页面。


```shell
hexo server # the same as: hexo s
```
这时，4000 端口打开。

## 配置

### 目录和文件
![](leanote://file/getImage?fileId=5795fb392ac5d002bb000000)

1. `scaffolds`：模板文件夹，新建文章时，Hexo 会根据 scaffold 来建立文件。Hexo 有三种默认布局：post、page 和 draft，它们分别对应不同的路径。新建文件的默认布局是post，可以在配置文件中更改布局。用draft布局生成的文件会被保存到 source/_drafts 文件夹。 
2. `source`：资源文件夹是存放用户资源的地方。 
3. `source/_post`：文件箱。（低版本的hexo还会存在一个_draft，这是草稿箱）除 _posts 文件夹之外，开头命名为 _ (下划线)的文件/ 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去 
4. `themes`：主题 文件夹。Hexo 会根据主题来生成静态页面。 
5. `themes/landscape`：默认的皮肤文件夹 
6. `_config.yml`：全局的配置文件，每次更改要重启服务。

低版本的Hexo还会生成scripts文件夹，里面用于保存扩展Hexo的脚本文件。

### 全局配置
可以在_config.yml 中修改：

```yml
# Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/
# Site 站点配置
title: Hexo-demo  #网站标题
subtitle: hexo is simple and easy to study   #网站副标题
description: this is hexo-demo #网栈描述
author: pomy #你的名字
language:  zh-CN #网站使用的语言
timezone: Asia/Shanghai  #网站时区
# URL #可以不用配置
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com #网址，搜索时会在搜索引擎中显示
root: /  #网站根目录
permalink: :year/:month/:day/:title/ #永久链接格式
permalink_defaults: #永久链接中各部分的默认值
# Directory 目录配置
source_dir: source #资源文件夹，这个文件夹用来存放内容
public_dir: public #公共文件夹，这个文件夹用于存放生成的站点文件
tag_dir: tags #标签文件夹
archive_dir: archives #归档文件夹
category_dir: categories #分类文件夹
code_dir: downloads/code #Include code 文件夹
i18n_dir: :lang   #国际化文件夹
skip_render:      #跳过指定文件的渲染，您可使用 glob 来配置路径
# Writing 写作配置
new_post_name: :title.md # 新文章的文件名称
default_layout: post   #默认布局
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0 #把文件名称转换为 (1) 小写或 (2) 大写
render_drafts: false #显示草稿
post_asset_folder: false #是否启动资源文件夹
relative_link: false #把链接改为与根目录的相对位址
future: true  
highlight:  #代码块的设置
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:
# Category & Tag 分类 & 标签
default_category: uncategorized #默认分类
category_map:   #分类别名   
tag_map:        #标签别名
# Date / Time format  时间和日期
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss
# Pagination 分页
## Set per_page to 0 to disable pagination
per_page: 10 #每页显示的文章量 (0 = 关闭分页功能)
pagination_dir: page #分页目录
# Extensions 扩展
## Plugins: http://hexo.io/plugins/ 插件
## Themes: http://hexo.io/themes/  主题
theme: landscape #当前主题名称
# Deployment #部署到github
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type:
```

一般主题下有一个 languages 文件夹，用于对应 language 配置项。比如在 ejs 中有：

```
<%= __('tags') %>
```
language 的配置项是 zh-CN，则会在languages 文件夹下找到 zh-CN.yml 文件中对应的项来解释。

修改全局配置时，注意缩进，同时注意冒号后面要有一个空格。

### 主题配置
主题的配置文件在/themes/主题文件夹/_config.yml，一般包括导航配置(menu)，内容配置(content)，评论插件，图片效果(fancybox)和边栏(sidebar)。

Hexo提高了大量的主题，可以在全局配置文件中更改主题：

```
# Extensions 扩展
## Plugins: http://hexo.io/plugins/ 插件
## Themes: http://hexo.io/themes/  主题
theme: 你的主题名称
```

主题的文件目录必须在 themes 目录下。[Hexo主题更换教程](https://github.com/dwqs/nx)

### 修改局部页面

页面展现的全部逻辑都在每个主题中控制，源代码在hexo\themes\你使用的主题\中，以modernist主题为例：

```
.
├── languages          #多语言
|   ├── default.yml    #默认语言
|   └── zh-CN.yml      #中文语言
├── layout             #布局，根目录下的*.ejs文件是对主页，分页，存档等的控制
|   ├── _partial       #局部的布局，此目录下的*.ejs是对头尾等局部的控制
|   └── _widget        #小挂件的布局，页面下方小挂件的控制
├── source             #源码
|   ├── css            #css源码 
|   |   ├── _base      #*.styl基础css
|   |   ├── _partial   #*.styl局部css
|   |   ├── fonts      #字体
|   |   ├── images     #图片
|   |   └── style.styl #*.styl引入需要的css源码
|   ├── fancybox       #fancybox效果源码
|   └── js             #javascript源代码
├── _config.yml        #主题配置文件
└── README.md          #用GitHub的都知道
```

如果你需要修改头部，直接修改hexo\themes\modernist\layout\_partial\header.ejs，比如头上加个搜索框：

```
<div>
<form class="search" action="//google.com/search" method="get" accept-charset="utf-8">
 <input type="search" name="q" id="search" autocomplete="off" autocorrect="off" autocapitalize="off" maxlength="20" placeholder="Search" />
 <input type="hidden" name="q" value="site:<%- config.url.replace(/^https?:\/\//, '') %>">
</form>
</div>
```
将如上代码加入即可，您需要修改css以便这个搜索框比较美观。

再如，你要修改页脚版权信息，直接编辑hexo\themes\modernist\layout\_partial\footer.ejs。同理，你需要修改css，直接去修改对应位置的styl文件。

### 评论框

静态博客要使用第三方评论系统，hexo默认集成的是Disqus，因为你懂的，所以国内的话还是建议用多说。
直接用你的微博/豆瓣/人人/百度/开心网帐号登录多说，做一下基本设置。如果使用modernist主题，在 modernist_config.yml 中配置 duoshuo_shortname 为多说的基本设置->域名中的shortname即可。你也可以在多说后台自定义一下多说评论框的格式，比如评论框的位置，对于css设置，可以参考这里，我是在HeroicYang的基础上修改的。

如果你是有的其他第三方评论系统，将通用代码粘贴到 `hexo\themes\modernist\layout\_partial\comment.ejs` 里面，如下：

```
<% if (config.disqus_shortname && page.comments){ %>
<section id="comment">
  #你的通用代码
<% } %>
```

## 基本使用
### 写文章 
通过new命令新建一篇文章：
执行new命令，生成指定名称的文章至`hexo\source\_posts\postName.md`。

```
hexo new [layout] "postName" #新建文章
```

```
$ hexo new [layout] <title>
//same as
hexo n
```

其中layout是可选参数，默认值为post。

如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，需用引号括起来。

Hexo提供的layout在scaffolds目录下，也可以在此目录下自建layout文件。新建的文件则会保存到source/_post目录下。

发表的文章会全部显示，如果文章很长，就只要显示文章的摘要就行了。在需要显示摘要的地方添加如下代码即可：

```
以上是摘要
<!--more-->
以下是余下全文
```

### 部署 
在部署之前，需要通过命令把所有的文章都做静态化处理，就是生成对应的html, javascript, css，使得所有的文章都是由静态文件组成的：

```
hexo generate
//same as
hexo g
```

在本地目录下，会生成一个public的目录，里面包括了所有静态化的文件。

生成静态文件之后，如果要发布到github，还需要配置deploy指令。在全局的配置文件中找到deploy：

```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/dongkui0712/dongkui0712.github.io.git
  branch: hexo
```

然后还要安装hexo-deployer-git：

```
npm install hexo-deployer-git -S
```

最后利用hexo指令发布到github：

```
hexo d
//same as
hexo deploy
```

## 总结
Hexo常用命令：

```
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```


## 参考

- [Hexo 主题Light修改教程](http://www.jianshu.com/p/70343b7c2fd3)
- [hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)
- [《简洁轻便的博客平台: Hexo详解》](http://www.ido321.com/1650.html)
