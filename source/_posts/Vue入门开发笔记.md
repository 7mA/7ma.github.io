---
title: Vue入门开发笔记
permalink: Vue_Start_Log/
date: 2018-06-20 21:22:17
tags:
- JavaScript
- 前端
- Legacy
categories:
- 技术笔记
---

5月中旬到6月中旬这段时间，除了整毕设、准备工作以外，还有一件事就是带着俱乐部的学弟学妹们参加了IBM的一个聊天机器人的开发比赛。比赛中我负责Web端实现，第一次尝试使用Vue.js来写界面。这里做一点开发当中的笔记。

<!--more-->

---

## 1.开始

Vue（/vjuː/，音同view）是一套用于构建用户界面的渐进式框架。Vue的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。在本次开发过程中，我就是用Vue写的界面，然后内嵌到基于Express的Web应用当中。可以直接用`<script>`标签直接引用Vue.js文件，同时也可以用npm命令安装`vue-cli`工具来创建基于Vue的单页应用。本次开发我采用的是后者。

    npm install --global vue-cli

在安装完毕以后，就可以在自己的工作目录下创建Vue项目了。创建命令如下：

    vue init webpack your_project_name

项目名称`your_project_name`部分可替换。

输入该命令之后，会进入项目设置阶段。

`Project name (test)`可以设置项目名称，直接回车就是使用括号里原来指定的名称（test）。

`Project description (A Vue.js project)`可以设置项目描述，括号里为缺省选项，直接回车即可。

`Author (****)`可以设置作者，同样括号里为缺省选项，不指定的话回车即可。

```
    Runtime + Compiler: recommended for most users
    Runtime-only: about 6KB lighter min+gzip, but templates (or any Vue-specific
    HTML) are ONLY allowed in .vue files - render functions are required elsewhere
```

这个选项是选择环境是仅运行时还是运行时+编译器，大部分用户推荐第一个。

`Install vue-router? (Y/n)`是询问是否安装`vue-router`。如果项目中需要使用路由的话推荐安装。但是因为本次开发是单页应用，而且之前已经使用Express配置过路由，因此我选择了“n”。

`Use ESLint to lint your code? (Y/n)`是询问是否使用ESLint管理代码。根据需要选择即可。

`Set up unit tests (Y/n)`询问是否安装单元测试。

`Setup e2e tests with Nightwatch? (Y/n)`询问是否安装e2e测试。

```
	Should we run `npm install` for you after the project has been created? (reco
	mmended) (Use arrow keys)
	Yes, use NPM
	Yes, use Yarn
	No, I will handle that myself
```

这里是询问项目创建之后是否自动运行npm命令安装依赖包，推荐选择第一项。

---

## 2.开发与部署

在开发和部署过程中，我们用到最多的命令就是`npm run dev`和`npm run build`了。`npm run dev`的功能是在开发环境中测试页面，而`npm run build`的功能是部署当前工程生成build.js文件。在上面的图片中我们也能看到在to get started的指引下也有`npm run dev`命令。

这两个命令的具体原理可以从package.json文件中得知。package.json文件中有如下字段：

```json
    "scripts": {
    	"dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    	"start": "npm run dev",
    	"build": "node build/build.js"
    }
```

其中dev对应的是使用webpack-dev-server打开build/webpack.dev.conf.js，而build对应的是使用node打开build/build.js文件。

当用户执行`npm run dev`之后，默认设置下可以到 http://localhost:8080/ 查看页面效果。而当用户执行`npm run build`之后，用户可以在dist文件夹下看到新生成的build.js和build.js.map文件。之后只要将这两个文件复制到对应的服务器或者项目的文件夹下，就可以直接部署页面了。

---

## 3.http请求

在本次开发中，我用到了Vue的http请求，主要用来调用云端暴露出来的API并通过http来进行信息交换。要在Vue当中使用http请求，需要安装vue-resource。

    npm install vue-resource

之后需要在项目的main.js文件中添加`import VueResource from ‘vue-resource’;`，然后用`Vue.use(VueResource)`来启用插件。

Vue中的http请求函数如下所示。

```JavaScript
    //get方式
    this.$http.get('url',{
        param1:value1,  
        param2:value2  
    }).then(function(response){  
        // response.data中获取ResponseData实体
    },function(response){  
        // 发生错误
    });

    //post方式
    this.$http.post('url',{  
        param1:value1,  
        param2:value2  
    },{  
        emulateJSON:true  
    }).then(function(response){  
        // response.data中获取ResponseData实体
    },function(response){  
        // 发生错误
    });
```

---
**Reference Source:**

1. [Vue.js](https://cn.vuejs.org/ "https://cn.vuejs.org/")
2. [关于vue的npm run dev和npm run build](https://www.cnblogs.com/hl0203/p/7138600.html "https://www.cnblogs.com/hl0203/p/7138600.html")
3. [VUE HTTP调用方式总结](https://blog.csdn.net/qq_21033663/article/details/79141632 "https://blog.csdn.net/qq_21033663/article/details/79141632")
