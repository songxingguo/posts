---
title: 从零开始搭建在线个人简历
description: 一个简单的且易于维护的在线简历。
urlname: wfzgkl5rmicnnog4
date: 2023-02-07 05:15:12 +0000
tags:
  - 编程/简历
categories:
  - 前端
---
## 目录

## 写在前面

做这个在线简历是出于两点考虑。

- word 文档式的简历没有办法展示到网上，不能体现我作为一个前端开发工程师的专长。
- 之前也写过一个在线简历，但是现在不好维护了。主要原因是之前使用 Webpack 打包，对于一个简历的小项目来说比较重。然后之前的图床是使用七牛云进行存储的，但现在七牛云所有域名都需要备案，我之前的域名地址也被莫名其妙（😡）的被删除了，已经无力去维护。如果后面有时间和精力再进行升级优化吧。

基于上面的两点考虑，我决定做一个简单的且易于维护的在线简历。这次打包采用了开箱即用的[parceljs](https://parceljs.org/)，对于在线简历这个小项目正好合适。而且这次简历依然是使用了模版引擎[ejs](https://ejs.bootcss.com/)，让数据和模版分离，通过 json 文件来配置简历，在设计思想上这个参考了[Json-Resume](https://jsonresume.org/)，在样式上参考了[SHOYUF](https://shoyuf.top/resume)的简历。

### 传送门

- 如果不想从零开始配置就可以直接下载`[example](https://github.com/sarscode/parcel-transformer-ejs)`运行。
- 该项目的[开源地址](https://github.com/songxingguo/resume)在 github 上，分支为`feature-ejs`。
- 如果想将简历发布到线上可以参考[使用 Github Action 部署静态网站](https://www.yuque.com/songxingguo/devhints/tfub27hk86lsdrpb)。

## 从零开始搭建一个 MVP 项目

### 安装和配置`parcel`

1. 创建一个 resume 文件夹
2. 使用 `npm init`初始化` package.json`，全部都选择默认选项。
3. 安装`parcel`

```bash
npm install --save-dev parcel
```

4. 在`package.json`中添加任务脚本

```json
"scripts": {
    "dev": "parcel",
    "build": "parcel build"
  }
```

5. 新建 src 目录，并在目录下新建`index.ejs`文件，并在`index.ejs`中添加内容。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hello parcel！</title>
  </head>
  <body>
    <h1>Hello parcel!</h1>
  </body>
</html>
```

然后在配置`package.json`中配置 parcel 的入口文件。

```json
 "source": "src/index.ejs",
```

### 安装和配置`parcel-transformer-ejs`

6. 安装`parcel-transformer-ejs`

```bash
npm i -D parcel-transformer-ejs
```

[parcel-transformer-ejs](https://www.npmjs.com/package/parcel-transformer-ejs) 是一个 ejs 转换器，将 ejs 转换成 html，这个库唯一的缺点就是修改了 json 配置文件之后，页面并没有更新，对开发效率还是影响蛮大的。

> 现在的解决办法是，修改了`.ejsrc`再立即修改一下`index.ejs`就会更新页面了。

7. 在根目录下新建`.parcelrc`，配置[parcel](https://parceljs.org/)转换器。

```json
{
  "extends": ["@parcel/config-default"],
  "transformers": {
    "*.ejs": ["parcel-transformer-ejs"]
  }
}
```

8. 在 src 目录下新建`.ejsrc`作为**简历配置**文件，json 格式可以参考[jsonresume](https://jsonresume.org/schema/)。

```json
{
  "basics": {
    "name": "John Doe",
    "label": "Programmer",
    "image": "",
    "email": "john@gmail.com",
    "phone": "(912) 555-4321",
    "url": "https://johndoe.com",
    "summary": "A summary of John Doe…",
    "location": {
      "address": "2712 Broadway St",
      "postalCode": "CA 94115",
      "city": "San Francisco",
      "countryCode": "US",
      "region": "California"
    },
    "profiles": [
      {
        "network": "Twitter",
        "username": "john",
        "url": "https://twitter.com/john"
      }
    ]
  },
  "work": [
    {
      "name": "Company",
      "position": "President",
      "url": "https://company.com",
      "startDate": "2013-01-01",
      "endDate": "2014-01-01",
      "summary": "Description…",
      "highlights": ["Started the company"]
    }
  ],
  "volunteer": [
    {
      "organization": "Organization",
      "position": "Volunteer",
      "url": "https://organization.com/",
      "startDate": "2012-01-01",
      "endDate": "2013-01-01",
      "summary": "Description…",
      "highlights": ["Awarded 'Volunteer of the Month'"]
    }
  ],
  "education": [
    {
      "institution": "University",
      "url": "https://institution.com/",
      "area": "Software Development",
      "studyType": "Bachelor",
      "startDate": "2011-01-01",
      "endDate": "2013-01-01",
      "score": "4.0",
      "courses": ["DB1101 - Basic SQL"]
    }
  ],
  "awards": [
    {
      "title": "Award",
      "date": "2014-11-01",
      "awarder": "Company",
      "summary": "There is no spoon."
    }
  ],
  "certificates": [
    {
      "name": "Certificate",
      "date": "2021-11-07",
      "issuer": "Company",
      "url": "https://certificate.com"
    }
  ],
  "publications": [
    {
      "name": "Publication",
      "publisher": "Company",
      "releaseDate": "2014-10-01",
      "url": "https://publication.com",
      "summary": "Description…"
    }
  ],
  "skills": [
    {
      "name": "Web Development",
      "level": "Master",
      "keywords": ["HTML", "CSS", "JavaScript"]
    }
  ],
  "languages": [
    {
      "language": "English",
      "fluency": "Native speaker"
    }
  ],
  "interests": [
    {
      "name": "Wildlife",
      "keywords": ["Ferrets", "Unicorns"]
    }
  ],
  "references": [
    {
      "name": "Jane Doe",
      "reference": "Reference…"
    }
  ],
  "projects": [
    {
      "name": "Project",
      "description": "Description…",
      "highlights": ["Won award at AIHacks 2016"],
      "keywords": ["HTML"],
      "startDate": "2019-01-01",
      "endDate": "2021-01-01",
      "url": "https://project.com/",
      "roles": ["Team Lead"],
      "entity": "Entity",
      "type": "application"
    }
  ]
}
```

9. 修改一下`index.ejs`，添加一些[ejs 语法](https://www.yuque.com/songxingguo/devhints/xnf7wu5wxoucphtg)。

```bash
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Hello EJS！</title>
</head>
<body>
  <h2><%= basics.name %></h2>
</body>
</html>
```

最后，你就能运行它了：

```bash
# 以开发模式运行
npm run dev

# 以生成模式运行
npm run build
```

下面是运行效果。

![从零开始搭建在线个人简历 20240606111858534](https://image.songxingguo.com/obsidian/20240714/%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%90%AD%E5%BB%BA%E5%9C%A8%E7%BA%BF%E4%B8%AA%E4%BA%BA%E7%AE%80%E5%8E%86-20240606111858534.webp)

## 一些小优化

### 加粗文字或者添加链接

1. 在文本中添加标签。

```json
{
  "basics": {
    "summary": "A <b>summary</b> of <a href='https://www.songxingguo.com/'>John Doe</a>…",
  },
```

2. 输出是使用`<%-`可以不转义 html。

```javascript
<%- basics.summary %>
```

3. 效果如下。

![从零开始搭建在线个人简历 20240606111909602](https://image.songxingguo.com/obsidian/20240714/%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%90%AD%E5%BB%BA%E5%9C%A8%E7%BA%BF%E4%B8%AA%E4%BA%BA%E7%AE%80%E5%8E%86-20240606111909602.webp)
