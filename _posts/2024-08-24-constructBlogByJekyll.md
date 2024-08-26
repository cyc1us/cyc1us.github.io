---
layout: post
title: "Jekyll搭建个人博客网站"
date: 2024-08-23 
description: "Jekyll+GithubPages搭建个人博客网站"
tag: Blog
---   
## 写在前面

其实很早就有搭建个人博客网站的想法，首先我认为这是个很酷的事情，倒不是觉得有一个个人博客是个很酷的事情，只是觉得拥有一个网站是很酷的事情，这个念想在初次接触计算机的时候就萌生了。
后来接触到了软件开发，也慢慢的了解了网站背后的一些原理，那自己是不是也可以拥有一个自己的网站呢。后来在学习和工作的过程中我越发认为费曼学习法是一个很好的理解、巩固知识的方式。于是我也开始学着写一些博客，那这些博客仅仅是在文件夹里面放着我又觉得不美观也不方便，于是搭建个人博客网站的想法便油然而生了。
一开始我的想法其实是用SpringBoot+数据库+服务器部署的方式来实现，但这样其实还挺麻烦的，前端我也不会写。后来去了解后发现其实没有必要用这种方式，页面方面可以有很多框架的选择像Jekyll、Hexo之类的，并且有很多模板可以选择。而部署方面GitHub就提供了GithubPages这样一个服务。那么知道大体该怎么做之后，就可以开始了。

## Ruby

我选择的是Jekyll+GithubPages的形式来搭建博客网站。而Jekyll是Ruby开发的，所以需要先安装Ruby的环境。

Ruby是一种动态、开源的编程语言，注重简洁性和效率。它由日本人松本行弘（Yukihiro "Matz" Matsumoto）在20世纪90年代中期开发，旨在结合多种编程范式的优点，提供一种更人性化的编程体验。Ruby以其优雅的语法和强大的元编程能力而闻名，广泛应用于Web开发、脚本编写、数据分析等领域。

可以在https://www.ruby-lang.org下载安装，安装后还要配置好环境变量即可使用命令安装Jekyll。

## Jekyll

### 简介

Jekyll是一个简单的、可扩展的静态站点生成器，使用Ruby编写。它由GitHub的联合创始人Tom Preston-Werner创建，旨在为开发者和写作者提供一个易于使用的内容管理系统。Jekyll的主要特点包括：

1. **静态站点生成**：Jekyll将纯文本转换为静态网站和博客，无需数据库或复杂的安装过程。
2. **基于Markdown**：支持Markdown格式编写内容，使得内容创作更加简单高效。
3. **模板系统**：使用Liquid模板语言，可以轻松创建可重用的模板和布局。
4. **插件扩展**：通过Ruby插件，可以扩展Jekyll的功能，满足各种定制需求。
5. **GitHub Pages支持**：Jekyll与GitHub Pages无缝集成，可以直接将Jekyll生成的站点部署到GitHub上。

### 安装

打开命令行：

1. 安装Jekyll：``` gem install jekyll bundler ``` 

2. 随便到一个文件夹下，创建博客：``` jekyll new myBlog```,这里的myBlog可以自定义，创建后会在当前目录下新生成一些文件如下图：

   ![image-20240824135347037](/images/posts/Blog/constructBlogByJekyll/image-8.png)

在Gemfile这个文件中有一个source "https://rubygems.org"，是用于指定 Ruby 项目所使用的 gem 的来源。这个在后续下载插件可能会有点慢，可以把它替换为"https://gems.ruby-china.com",也可以使用下面两行命令：
~~~ ruby
gem sources --remove http://rubygems.org/
gem sources --add https://gems.ruby-china.com/
~~~

检查是否替换成功：``` gem sources -l```

![image-20240824142916942](/images/posts/Blog/constructBlogByJekyll/image-9.png)

3. 之后运行``` jekyll server```，如果报错的话，直接把Gemfile删了再运行，然后再次运行，命令行就会提示你可以进入
   http://127.0.0.1:4000/，在浏览器中打开：
   <img src="/images/posts/Blog/constructBlogByJekyll/image-1.png" alt="image-20240826193408452" style="zoom: 50%;" />

到此你就成功安装并创建了一个博客页面。

### 目录结构介绍

jekyll的目录结构大致如下：

~~~ 
.
├── _config.yml
├── _data
│   └── members.yml
├── _drafts
│   ├── draft1.md
│   └── draft2.html
├── _includes
│   ├── footer.html
│   └── header.html
├── _layouts
│   ├── default.html
│   ├── post.html
│   └── page.html
├── _posts
│   ├── 2022-01-01-title1.md
│   └── 2022-01-02-title2.md
├── _sass
│   ├── _base.scss
│   └── _layout.scss
├── _site
├── .jekyll-cache
├── .jekyll-metadata
├── assets
│   ├── css
│   │   └── main.scss
│   ├── images
│   └── js
│       └── script.js
├── index.html
└── about.md
~~~

部分文件的作用：

1. **_config.yml**: Jekyll 的全局配置文件。包含站点的主要设置，如标题、描述、URL、插件、变量等。

2. **_data**: 存放 YAML、JSON 或 CSV 格式的数据文件。这些数据可以在模板中使用。

3. **_drafts**: 存放未发布的草稿文章。这些文章不会在生成站点时包含，除非使用 `--drafts` 选项。

4. **_includes**: 存放可重用的模板片段，如页头、页脚、侧边栏等。可以在布局或页面中通过 `{% raw %}{% include file.html %}{% endraw %}` 引入。

5. **_layouts**: 存放布局模板文件。布局定义了页面的基本结构，可以通过 YAML 前言在页面或文章中指定。

6. **_posts**: 存放已发布的文章。文章文件名必须遵循 `YEAR-MONTH-DAY-title.MARKUP` 格式。

​	细心的话你会发现之前创建的项目中在_posts下有一个自动生成的md文件，而这个文章就可以在博客页面中访问到

7. **index.html**: 站点的首页文件。可以是一个 HTML 文件或一个包含 YAML 前言的 Markdown 文件。

8. **about.md**: 一个示例页面文件。可以是一个 Markdown 文件或 HTML 文件。

## Github Pages

GitHub Pages 是一个由 GitHub 提供的静态站点托管服务，允许用户直接从 GitHub 仓库中托管和发布网站。它非常适合用于个人、项目或组织的文档、博客和静态网站。GitHub Pages 支持使用 Jekyll 等静态站点生成器，并且可以免费使用。

### 主要特点

1. **免费托管**:GitHub Pages 提供免费的静态站点托管服务，无需额外费用。
2. **集成 GitHub**:直接从 GitHub 仓库中发布站点，方便版本控制和协作。
3. **支持自定义域名**:可以使用自定义域名，使你的站点看起来更专业。
4. **自动构建**:支持 Jekyll 等静态站点生成器，可以自动构建和部署站点。
5. **HTTPS 支持**:提供免费的 HTTPS 证书，确保站点安全。

### 使用步骤

首先你需要一个GitHub账号，然后创建一个同名仓库如下：

<img src="/images/posts/Blog/constructBlogByJekyll/image-2.png" style="zoom:50%;" />

这样你就可以在这个仓库中开始操作了。

##  搭建个人博客

### 模板选择

你可以选择自己去写博客的页面，当然也有很多现成的优秀模板供你选择：[Jekyll Themes – a curated directory](https://jekyllthemes.io/)

我这里使用的是潘柏信大佬的模板：[leopardpan/leopardpan.github.io: 个人博客，看效果进入](https://github.com/leopardpan/leopardpan.github.io)

![](/images/posts/Blog/constructBlogByJekyll/image-3.png)

可以看到这个模板是非常漂亮的，并且潘老师也写了详细的文档介绍该如何去搭建博客，笔者就是参考这些文档来搭建的这个博客，十分简便。你可以选择fork这个项目到你的同名仓库下，也可以直接fork我的[个人博客](https://github.com/cyc1us/cyc1us.github.io)。之后你访问你的 同名仓库.github.io，便可以出现博客页面。
当然目前来说这个页面还是它原本的样子，所以说你需要去修改一些文件从而把它变成你自己的博客。

### 修改配置

这里我使用的vscode，把你自己仓库下的项目clone下来。你可以选择在master分支上操作也可以单独创建一个分支。

<img src="/images/posts/Blog/constructBlogByJekyll/image-4.png" style="zoom:50%;" />

进去后把_posts文件夹下以及images下的一些博客需要用的的图片删掉。avatar.jpg以及background-cover.jpg是头像和背景，你可以换成你自己的。其它图片就是一些小图标，可以换也可以不用换。
上图中三个高亮的文件也删掉，之后打开终端在此目录下执行``` jekyll server```,这样你就可以在本地4000端口访问到博客界面了：

<img src="/images/posts/Blog/constructBlogByJekyll/image-5.png" style="zoom: 33%;" />

这里面还有一些个人信息的修改，你可以在_config.yml中把信息换成你的

<img src="/images/posts/Blog/constructBlogByJekyll/image-6.png" style="zoom:50%;" />

此外还有README、about、support这些文件中的信息你也可以换成你自己的。修改好后重新执行命令或者刷新就能看到你想要的样子了。如果后续你发现你发布的博客在主页无法显示，那你可以尝试在_config.yml中加上![](/images/posts/Blog/constructBlogByJekyll/image-7.png)

最终呈现效果如下：

<img src="/images/posts/Blog/constructBlogByJekyll/image-20240826202034362.png" alt="image-20240826202034362" style="zoom: 33%;" />

### 发布文章

接下来你只需要准备好你的文章，然后在文章的开头加上这一段文字：

~~~ 
---
layout: post

title: "XXX"

date: YYYY-MM-DD

description: "XXX"

tag: Blog

---
~~~

并且文章的名字必须为YYYY-MM-DD-title.md，这样Jekyll便会将你的文章转换成静态HTML文件，从而生成对应的页面：

![image-20240826203214348](/images/posts/Blog/constructBlogByJekyll/image-20240826203214348.png)

现在你在本地以及确定好了没有问题，那么现在你只需要把本地的文件push到GitHub上。以后你需要增加新文章时，再拉下来写好之后推上去即可。最终你就拥有了自己的一个博客网站。

## 改进想法

不得不说这样搭建网站确实很方便，并且能自定义的程度也很高，当然这要掌握一定的前端页面知识。不过也确确实实存在许多问题。

- 对于国内来说访问GitHub确实太慢了，而且会经常出现访问不到的情况，这大大的影响了阅读体验。
- 要完成一篇文章还是很麻烦，要自己去推送，一点也不自动化，
- 页面都是静态页面，没有数据库的支撑，导致其展现形式只限于少量的文字。

对于第一点来说，Gitee好像也有类似的部署静态页面的功能，所以之后我会尝试着把它迁到Gitee上去，或是把它部署到服务器上。
那第二三点呢，简而言之就是需要后端做支撑，需要完成博客的在线编辑发布之类的功能，但是我不知道这是否能和Jekyll生成的这种静态页面结合起来。如果不行的话后续可能还是会使用前后端那一套。

不管怎么说，这只是第一步。之后我会去掌握更多的知识来完善这个博客的功能。并且也会坚持写blog来充实其内容。
