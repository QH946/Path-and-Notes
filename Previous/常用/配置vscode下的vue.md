## 一、下载node.js

1)打开node.js的官网下载地址：http://nodejs.cn/download/

2)配置环境变量,把node.js安装路径配置到环境变量Path中,使用node.js安装包安装时一般会自动添加环境变量。

首先在node.js的安装路径中新建两个文件夹，分别作为缓存文件夹和全局安装文件夹（==node_cache,node_global==）

在cmd中输入以下指令，设置缓存和全局安装文件夹为上述新建文件夹。请将D:\Program Files\nodejs替换为你自己的node.js的安装路径。

==npm config set prefix "D:\Program Files\nodejs\node_global"==
==npm config set cache "D:\Program Files\nodejs\node_cache"==

打开“环境变量”,path变量中新增

==D:\Program Files\nodejs\node_global==
==D:\Program Files\nodejs\node_global\node_modules== 

3)配置npm

**经过测试，cnpm安装的组件可能存在问题，在此不建议使用cnpm！！！**
  由于网络原因，国内访问npm的速度很慢，可以通过设置国内淘宝镜像来访问npm。
  打开cmd，执行以下指令。

设置官方库：npm config set registry  [http://registry.npmmirror.com](http://registry.npmmirror.com/)

4)查看node.js是否安装成功：打开cmd，输入node -v 和 npm -v 如果显示版本信息，则说明安装成功。

## 二、安装配置脚手架

1)提前建文件夹，打开[VScode](https://so.csdn.net/so/search?q=VScode&spm=1001.2101.3001.7020)的终端，cd切换到提前建好的文件夹下

2)npm install -g @vue/cli

3)webpack打包==（可省略）==

若webpack版本>4+，需要先安装webpack-cli

npm install -save-dev webpack-cli

npm install -save-dev webpack

安装完成后通过webpack -v查看版本即表示安装成功



## 三、创建vue项目

1)vue init webpack first_vue

2)cd 到刚才项目目录(cd first vue)

3)执行npm cache clean --force 清除缓存,执行npm install 重新初始化依赖

## 四、启动项目

1）打开项目里面的package.json，在vscode终端执行start中的命令npm run dev，启动成功后会出现访问地址。

2）根据提示，访问[http://localhost:8080](http://localhost:8080/)

