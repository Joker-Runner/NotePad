# Vue入门
## 一、环境搭建
1. 安装Nodejs
```shell
# brew下载nodejs，网速慢的和可以去网站下载安装
brew install nodejs 
# 安装淘宝镜像cnpm，使用cnpm安装包
npm install -g cnpm --registry=https://registry.npm.taobao.org
# 替换npm的源，直接使用npm即可
npm config set registry https://registry.npm.taobao.org
```
2. 安装依赖
```shell
# 安装Vue脚手架
npm install vue-cli -g
# 安装webpack打包工具
npm install webpack -g
```
3. 开始Vue
```shell
# 创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
# 进入项目
cd my-project
# 安装依赖
npm install
# 运行dev版本
npm run dev

# 后续添加路由
npm install vue-router --save -dev
# 安装element-ui支持
npm i element-ui -S
```