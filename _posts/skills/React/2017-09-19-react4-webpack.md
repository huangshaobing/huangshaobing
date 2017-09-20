---
layout: post
title: "React Webpack工程搭建"
date: 2017-09-19 18:30:26 +0800 
categories: React
tag: React
---
* content
{:toc}

最近不忙，试了试搭建前端React工程，用Webpack打包，遇到的问题做个记录。

<!-- more -->

## 1. 开发工具准备
Macbook 笔记本开发，用习惯了Sublime Text，因此就用这个工具了，版本是3.0。
首先要安装插件，使得jsx，scss文件语法有个颜色的直观体现，总比白色的代码好的多，😄
（1）安装sass插件 —— Command + Shift + P 选择Package Control：Install Package 搜索sass点击安装即可；<br />
（2）安装babel-sublime —— 同上安装过程，搜索babel安装；<br />
（3）安装jsFormat<br />
好了一个简单的开发工具准备完毕。

## 2. 软件环境
首先要安装nodejs，自行安装吧

## 3. 简单的基础工程搭建
（1）新建工程目录：clover-node<br/>
（2）执行npm init，创建package.json文件，按照提示输入内容一路回车～～<br/>
（3）创建工程目录与文件，最终的目录如下<br/>
	+ clover-node<br/>
	  + dist					------------- 打包发布工程存放路径<br/>
	  + node_modules			------------- 第三方安装包<br/>
	  + src                     ------------- 源码<br/>
	  	+ component             ------------- 公共组件<br/>
	  	  + Menu<br/>
	  	  + Navigator<br/>
	  	  ......<br/>
	  	+ config                ------------- 配置文件<br/>
	  	+ entry                 ------------- 入口文件<br/>
	  	+ page                  ------------- 页面<br/>
	  	+ assets                ------------- 资源文件<br/>
	  	  + image<br/>
	  	  + scss<br/>
	  -	webpack.config.js       ------------- webpack配置<br/>
	  - server.js               ------------- node启动server文件<br/>
	  - index.html              ------------- 首页模版（开发）<br/>
	  - template.html           ------------- 模版文件<br/>
	  - package.json<br/>

#### Package.json
	第三方软件包，可以直接写入package.json文件后执行npm install，或者命令行npm install --save-dev babel-core css-loader 安装最新版
```javascript
{
  "name": "clover-node",
  "version": "1.0.0",
  "description": "",
  "main": "webpack.config.js",
  "scripts": {
    "prestart": "npm install",
    "start": "export NODE_ENV=test&& node server",
    "build": "export NODE_ENV=production&& webpack -p",
    "test": "export NODE_ENV=dev&& node server"
  },
  "author": "Huang Shaobing",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-es2015": "^6.24.1",
    "babel-preset-react": "^6.24.1",
    "component-ajax": "0.0.2",
    "css-loader": "^0.28.7",
    "extract-text-webpack-plugin": "^3.0.0",
    "file-loader": "^0.11.2",
    "html-loader": "^0.5.1",
    "html-webpack-plugin": "^2.30.1",
    "moment": "^2.18.1",
    "node-sass": "^4.5.3",
    "react": "^15.6.1",
    "react-cookie": "^2.1.1",
    "react-dom": "^15.6.1",
    "react-helmet": "^5.2.0",
    "react-router-dom": "^4.2.2",
    "sass-loader": "^6.0.6",
    "style-loader": "^0.18.2",
    "url-loader": "^0.5.9",
    "webpack": "^3.5.6",
    "webpack-dev-server": "^2.8.1"
  }
}	
```

#### webpack.config.js文件内容配置

```javascript
var path = require("path");
var webpack = require('webpack');
var ExtractTextPlugin = require("extract-text-webpack-plugin");
/* 是否为开发模式 */
var env_if_dev = process.env.NODE_ENV == "test" || process.env.NODE_ENV == "dev";
var filename = env_if_dev ? "[name]" : "[chunkhash:8].[name]";
/* CSS Plugins */
var extractCSS = new ExtractTextPlugin('' + filename + '.css');
/* html Plugins */
var HtmlWebpackPlugin = require('html-webpack-plugin');
/* html 模版 */
var htmlPlugin = new HtmlWebpackPlugin({
	title: "首页",
	filename: "../index.html",
	template: "template.html"
});
var modulesDirectories = ["web_modules", "node_modules", "bower_components", "src/config", "src/scss", "src"];

var config = {
	entry: {
		app: ["./src/entry/app.jsx"],
		vendor: ["react", "react-dom", 'react-router']
	},
	output: {
		path: path.resolve(__dirname, "dist/build"),
		publicPath: "/build/",
		filename: filename + ".js"
	},
	resolve: {
		modules: modulesDirectories,
		extensions: ['.js', '.jsx', 'css']
	},
	module: {
		rules: [{
			test: /\.(js|jsx)$/,
			exclude: /(node_modules|bower_components)/,
			loader: 'babel-loader',
			query: {
				presets: ['es2015', 'react']
			}
		}, {
			test: /\.(eot|woff|ttf|svg)/,
			loader: 'file-loader?name=[name].[ext]'
		}, {
			test: /\.scss$/,
			use: ExtractTextPlugin.extract({
				fallback: 'style-loader',
				//resolve-url-loader may be chained before sass-loader if necessary
				use: ['css-loader', 'sass-loader']
			})
		}, {
			test: /\.css$/,
			use: ExtractTextPlugin.extract({
				fallback: 'style-loader',
				//resolve-url-loader may be chained before sass-loader if necessary
				use: ['css-loader']
			})
		}, {
			test: /\.html$/,
			loader: "html-loader"
		}, {
			test: /\.png$/,
			loader: "file-loader?name=[hash:8].[name].[ext]"
		}]
	},
	plugins: [
		new webpack.DefinePlugin({
			"process.env": {
				NODE_ENV: JSON.stringify("production")
			}
		}),
		//ignoreFiles
		new webpack.optimize.CommonsChunkPlugin({
			name: 'vendor',
			filename: 'base.js'
		}),
		extractCSS,
		htmlPlugin
	]
};
if (process.env.NODE_ENV == "test" || process.env.NODE_ENV == "dev") {
	config.devtool = "source-map";
	config.output.publicPath = "/";
}
module.exports = config
``` 
#### server.js 内容 
```javascript
webpack = require('webpack');
WebpackDevServer = require('webpack-dev-server');
var config = require("./webpack.config.js");
if (process.env.NODE_ENV == "dev") {
	config.entry.app.unshift("webpack-dev-server/client?http://localhost:9090/");
} else {
	config.entry.app.unshift("webpack-dev-server/client?http://192.168.1.100:8880/");
}
var compiler = webpack(config);
var server = new WebpackDevServer(compiler, {

});
console.log("-----------" + process.env.NODE_ENV);
if (process.env.NODE_ENV == "dev") {
	server.listen(9090);
} else {
	server.listen(8880, '127.0.0.1');
}
```

#### src/entry/app.jsx 配置

```javascript
import React, { Component } from 'react';
import { render } from 'react-dom'
import { Route,IndexRoute, Link, IndexLink, hashHistory,HashRouter } from 'react-router-dom'

import Styles from './_App.scss';
import Home from '../page/Home/Home';
import List from '../page/List/List';
console.log("-------");
render((
	<HashRouter history={hashHistory}>
	<div>
    	<Route exact path="/" component={Home}/>
    	<Route path="/list" component={List}/>
    </div>
	</HashRouter>
), document.getElementById('app'))	

```
这里用的react-router-dom 4.0 以上的版本，和以前的还是有些差别，直接食用Router会报错。要用HashRouter或者Browserouter，官方推荐Browserouter，但是这个现在有些浏览器还不支持，因此用了HashRouter。


