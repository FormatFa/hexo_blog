---
title: 第一个Vue项目笔记
tags:
categories:
- 前端
---





- 创建的是单页应用，只有一个页面

- 使用的是vue2 , vue cli3 ,vue cli3创建应用的结构和cli2的不一样，网上的不一样

- nodejs v10.16.3

  node 环境

- vscode

  ide

- vuejs

  https://cn.vuejs.org/v2/guide/#

- vue-cli 3

  用于创建vue项目

  https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create

- element-ui

  https://element.eleme.cn

  

### 创建项目

使用vue-cli 3 脚手架来创建项目,element ui做前端布局



命令 vue create 项目名称

结构

```
.
├── babel.config.js
├── dist
├── node_modules
├── package.json
├── package-lock.json
├── public			资源目录
├── README
├── README.md
├── src			  js 和 vue 文件 源码
└── vue.config.js 配置文件，默认没有，要修改配置时添加
```



### 运行项目

创建完后的是个nodejs样的， element-ui这些这些js,css都是通过npm安装进去的，在需要的地方导入他

npm run serve 启动一个服务器，调试项目

npm run build 编译生成 dist文件夹





### 部署到Git Page上时获取不到js,css等资源

默认的资源路径(publicPath)为/,上传到git上时是直接上传编译出来的整个dist文件夹,完整的访问url为

https://formatfa.github.io/Vue-Caculator/dist/ ,为了能够访问到需要修改`vue.config.js`,修改

```js
module.exports={
    // 直接上传dist目录的，
    publicPath:"/Vue-Caculator/dist/"
}
```

