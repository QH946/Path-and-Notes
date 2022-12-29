1.补全变量

==Map<String, Object> map = userMapper.selectMapById(id);==

userMapper.selectMapById(id)后alt+enter补全变量

2.下载mybatisX插件后快速生成crud

![image-20220816112940201](./assets/image-20220816112940201-16606205994202-1670815431138-8.png)

![image-20220816113043644](./assets/image-20220816113043644-16606206466244-1670815433177-10.png)

![image-20220816113101079](./assets/image-20220816113101079-16606206628886-1670815434577-12.png)

在接口类中：

如写入insert

![image-20220816113146505](./assets/image-20220816113146505-1670815435938-14.png)

选择插件自带的方法

alt+enter

![image-20220816113213254](assets/image-20220816113213254-16606207362728.png)

![image-20220816113237358](./assets/image-20220816113237358-1670815436970-16.png)

自动生成xml文件

![image-20220816113255138](./assets/image-20220816113255138-1670815438004-18.png)

根据年龄和性别修改

![image-20220816113606191](./assets/image-20220816113606191-1670815440741-22.png)

![image-20220722165316257](./assets/image-20220722165316257-1670815439318-20.png)

![image-20220722165423054](./assets/image-20220722165423054-1670815442605-24.png)

![image-20220722165431710](./assets/image-20220722165431710-1670815443575-26.png)

![image-20220722165622216](./assets/image-20220722165622216-1670815444648-28.png)

![image-20220722165639958](./assets/image-20220722165639958-1670815445679-30.png)

3.替换所有相同变量

先ctrl+f

![image-20220722182221613](./assets/image-20220722182221613-1670815448583-32.png)

ctrl+r

![image-20220722182252182](./assets/image-20220722182252182-1670815449811-34.png)

replace all

  /Ctrl+Shift+Alt+J/shift+F6(选中相同变量)

--------



1.1 ctrl + …
常用的快捷键如下：
功能	快捷键
删除当前行	ctrl + Y
复制当前行	ctrl + D（Duplicate 复制）
选中整个单词，连续按可以扩大选中范围	ctrl + W
快速替换字符串，可以替换单个/全部选中字符串	ctrl + R （replace 替换）
在当前文件中查找	ctrl + F（Find 查找）
快速搜索，一般用来快速搜索源码	ctrl + N
查看一个类的层级关系	ctrl + H （Hierarhcy 层级）
快速定位源码，将光标放在一个方法上输入 ctrl + B , 可以去到方法的源码	ctrl + B
添加注释和取消注释，【第一次是添加注释，第二次是取消注释】	ctrl + /
添加/取消多行注释，【第一次是添加注释，第二次是取消注释】	ctrl + shift + /
重写基类的方法	ctrl + O（Override 重写）
1.2 alt + …
常用的快捷键如下：
功能	快捷键
补全代码	alt + /
快速提示完成，在代码可能存在语法问题时，IDEA会提示使用该快捷键可以快速自动修正	alt + enter
快速生成构造器，可以生成含有任意形参的构造器	alt + insert
快速显示类结构，可以显示类中包含的所有属性和方法	alt + 7
1.3 ctrl + alt + …
常用的快捷键如下：
功能	快捷键
快速格式化代码	ctrl + alt + L
自动缩进行	ctrl + alt + I
将选中的代码使用if、while、try/catch等包装	ctrl + alt + T
1.4 ctrl + shift + …
常用的快捷键如下：
功能	快捷键
去除相关的包装代码	ctrl + shift + Delete
将光标所在的代码块向上/下整体移动	ctrl + shift + 向上/下箭头
快速运行当前的程序	ctrl + shift + F10
添加/取消多行注释，【第一次是添加注释，第二次是取消注释】	ctrl + shift + /
1.5 其它
常用的快捷键如下：
功能	快捷键
自动分配变量名，在新建对象时在后面加 .var





> idea快捷建新类
> 先alt+1选中左边项目再上下键选择软件包最后alt+insert

> 快捷键：shift+enter换下一行
> ctrl+d复制上一行

> 使用smart tomcat可以简化部署，避免自带使用自带tomcat时，若project settings中artifacts中的web application archive&exploded没有导入，或web application exploded
> 中的lib未导入导致访问jsp报404（若使用自带插件则需要删除原有web application archive&exploded重新导入，先导入exploded再导入archive）







> 统一配置所有项目的maven:
> File->New Projects Setup->Setting for New Projects->搜索maven，修改maven home 
> path为自带的，users setting file中选择maven文件中的conf->setting.xml/C盘.m2下的
> setting.xml,local repostory选择提前创建好的文件夹
>
> 配置mapper文件时如UserMapper时，把变量粘贴到本页面的注释中，能cv则不手写



快捷键：shift+enter换下一行
ctrl+d复制上一行
