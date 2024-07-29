---
created: 2024-07-28 21:13
modified: 2024-07-28 21:13
Type: Entity
tags:
  - 编程/前端
  - Apps/Obsidian
publish: true
date: 2024-07-28 21:13
title: 想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版）
description: 四年前当时我才参加工作，还保持着写博客的习惯，为了写博客还专门学习了Markdown语法，现在想想这个举动真的是很值，因为现在我在Obsidian中记笔记都会经常使用。记得当时写博客还很苦逼，每次都需要启动Hexo的服务，并且需要进入后台管理才能写。这样确实不是很方便，并且当时的我对于知识管理一无所知，把博客当成知识库管理来使用，大量的读书笔记都放到上面，但实在是操作起来太麻烦，渐渐的也就放弃了。后来，经过阮一峰的推荐，语雀进入了我的视野，当时觉得语雀就是天使，解决了我记笔记的最大痛点，因此也顺理成章将大量的笔记记录在语雀上，  之前还写了一篇[如何将语雀文章发布到Hexo博客](https://blog.songxingguo.com/posts/tech/yuque-hexo/)，希望将语雀当成我的博客后台，渐渐的我也发现了语雀的不足，毕竟语雀是线上知识库当数据量过大时它的瓶颈就出现了，加载缓慢，显然不适合快速的记笔记和思考。后来又了解到了Obsidian ，它数据本身就存储在本地的，一切都由自己掌控，因此能快速响应，这次总算是满足了管理海量知识的需求了。
categories:
  - 前端
---
## 目录
## 写在前面

四年前当时我才参加工作，还保持着写博客的习惯，为了写博客还专门学习了Markdown语法，现在想想这个举动真的是很值，因为现在我在Obsidian中记笔记都会经常使用。记得当时写博客还很苦逼，每次都需要启动Hexo的服务，并且需要进入后台管理才能写。这样确实不是很方便，并且当时的我对于知识管理一无所知，把博客当成知识库管理来使用，大量的读书笔记都放到上面，但实在是操作起来太麻烦，渐渐的也就放弃了。后来，经过阮一峰的推荐，语雀进入了我的视野，当时觉得语雀就是天使，解决了我记笔记的最大痛点，因此也顺理成章将大量的笔记记录在语雀上，  之前还写了一篇[如何将语雀文章发布到Hexo博客](https://blog.songxingguo.com/posts/tech/yuque-hexo/)，希望将语雀当成我的博客后台，渐渐的我也发现了语雀的不足，毕竟语雀是线上知识库当数据量过大时它的瓶颈就出现了，加载缓慢，显然不适合快速的记笔记和思考。后来又了解到了Obsidian ，它数据本身就存储在本地的，一切都由自己掌控，因此能快速响应，这次总算是满足了管理海量知识的需求了。

我一直相信输出是为了更好输入，在最开始我就希望博客文章能同时发布到多个平台，下面是我当时的畅想，但几经尝试，都以失败告终，直到遇到Obsidian，我的想法终于可以实现了，Obsidian支持自己开发插件，也就意味我自己可以实现一款插件，将文章发布到多个平台，并且Obsidian 本身就是一个浏览器环境，使用JS进行开发，对于一个资深前端来说，那还不是手到擒来。

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240728212623051](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240728212623051.jpeg)

现在只是实现了[obsidian-note-publish](https://github.com/songxingguo/obsidian-note-publish) 将文章发布到语雀上，后面还会支持掘金、公众号、个人博客等更多平台，真正实现一键发布到多个平台。

> 点击链接查看[原文](https://blog.songxingguo.com/posts/tech/obsidian-to-yuque/) ， 订阅 [SSR](https://blog.songxingguo.com/atom.xml) 获得最新动态。

## 安装插件

推荐使用[BRAT插件](https://github.com/TfTHacker/obsidian42-brat)安装。打开BRAT的设置界面，点击**Add Beta plugin**，然后输入本插件地址`https://github.com/songxingguo/obsidian-note-publish`，点击**Add Plugin**就可以安装本插件了。

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240728230636899](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240728230636899.jpeg)

## 配置插件

### 基础配置

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240729222256196](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240729222256196.webp)


- `Attachment location`：附件存放的位置，我附件默认都是存放在当前文件夹的 `attachmens` 中，这样将附件分开放置更方便管理附件，避免了将附件放到同一个文件夹下导致体积过大的问题。

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240729090149268](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240729090149268.webp)

- `Use image name as Alt Text` ：将图片文件中的 `-` 替换为 `_`。
- `Update original document` ：当图片上传之后是否需要更新图片本地的路径。
- `Delete local attachments` ：图片上传之后是否需要删除本地的图片。
- `Ignore note properties` ：在对笔记进行处理时是否需要去除顶部的元信息。


>  `Update original document` 和 `Delete local attachments` 一定要同时开启，否则删除了本地文件，但又没有更新线上地址会导致图片显示不出来。

### 语雀配置

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240729085646431](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240729085646431.webp)

- `Token` : 语雀的开发[授权码](https://www.yuque.com/settings/tokens)， 当然这个只有超级会员才可以创建。
- `BookSlug`：知识库的路径，比如我的知识库`https://www.yuque.com/songxingguo/read` 的路径是 `songxingguo/read`。
- `Public Note`:  新建文档时，是否公开发布文档。

### 图床配置

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240729085836013](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240729085836013.webp)

最后是图床相关的配置，这个会比较麻烦一下，下面主要讲 AliYun OSS 的配置，因为对于国内用户来说，阿里云的对象存储可能更加实惠。具体如何创建存储桶，可以阅读这篇[Obsidian + 阿里云OSS 支持云同步](https://juejin.cn/post/7156922309733777422) 的文章。

- `Region`：对象存储的地域。
- `Access Key Id`： 可以进入[用户权限](https://ram.console.aliyun.com/users)界面进行获取。
- `Access Key Secret`： 同上。
- `Access Bucket Name`：存储桶名。
- `Target Path`：图片的保存目录，支持 `{year} {mon} {day} {random} {filename}` 等变量，比如使用 `obsidian/{year}{mon}{day}/{filename}`，上传的图片 `pic.jpg` 将保存在 `obsidian/20230608/pic.jpg`。
- `Custom Domain Name`: 自定义域名地址。

更多配置可以查看 [Obsidian 图片上传插件：Image Upload Toolkit](https://atbug.com/obsidian-plugin-image-upload-toolkit/)，这里也要感谢 [obsidian-image-upload-toolkit](https://github.com/addozhang/obsidian-image-upload-toolkit)， 本插件是在其基础上开发的。

最后，温馨提示，可以进入 [对象存储OSS资源包(包月)](https://common-buy.aliyun.com/?spm=5176.7933691.J_5253785160.2.31174c59Y8lFmm&commodityCode=ossbag#/buy") 购买40G的包年套餐只要9块钱，十分划算，作为图床完全足够。有了图床就可以极大的减小本地Obsidian的仓库大小，方便多端同步。

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240729223046706](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240729223046706.webp)
## 使用插件

### 上传图片

 点击`Ctrl + P` 或者 `Cmd + P` 唤起命令工具，输入 Note Publish 就可以看到 Upload Image 的命令，点击就可以将图片上传到图床中。

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240729125209877](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240729125209877.webp)
### 发布文章

点击左侧的按钮会打开预览界面，现在只要三个按钮分别是选择知识库目录，复制博客文章，以及发布到语雀。

发布到语雀主要做了以下几件事情：

- 将Callouts 语法的头部去掉，转换成普通的引用，后面将其转换为语雀的高亮块。
- 将文章中的双链的两对括号去掉。
- 删除文章的元信息。

![想了四年，一键发布文章到多平台终于在Obsidian中实现了（语雀版） 20240729125451707](https://image.songxingguo.com/obsidian/20240729/%E6%83%B3%E4%BA%86%E5%9B%9B%E5%B9%B4%EF%BC%8C%E4%B8%80%E9%94%AE%E5%8F%91%E5%B8%83%E6%96%87%E7%AB%A0%E5%88%B0%E5%A4%9A%E5%B9%B3%E5%8F%B0%E7%BB%88%E4%BA%8E%E5%9C%A8Obsidian%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%88%E8%AF%AD%E9%9B%80%E7%89%88%EF%BC%89-20240729125451707.webp)