##  spring boot开发小记

1.SSM整合springboot基本流程

1)新建SpringBoot环境

2）导入新的依赖，并使用maven-helper插件查看依赖是否冲突，若冲突则通过更换版本等方式解决

3）原来的resources下的配置文件统一整合成application配置文件（properties/yml）

4)删除web.xml及WEB-INF文件，重新配置视图解析器

5)static目录下导入与web下相同的静态资源文件

6）将mapper层的xml文件统一放在resources目录下新建的mapper包中

7）Mapper层接口添加@Mapper注解或在启动类Application配置包扫描@MapperScan

8)使用@RestController代替@Controller和@ResponseBody

9）部署与测试

==注：restcontroller必须全部方法返回响应报文==

2.配置文件戴上双引号引发的一系列最关键前端无法访问

因配置视图解析器过程懒得手写便去百度CV导致未注意=后面的前缀后缀内容加了双引号导致无法识别

引起两个晚上的调试已经自我怀疑，期间从查看资源路径到修改不同前缀，甚至到重构Mapper都未解决

只得不遗余力地请教各种大佬，最终于2022.08.22大佬使用腾讯会议演示放解决，且得到大佬的学习建议

~~spring.mvc.view.prefix=''/pages/''~~
~~spring.mvc.view.suffix=''.jsp''~~

3.maven冲突导致后端项目无法启动

此事连同配置文件一起警醒，在配置环境的时候尽量按部就班，或根据能跑的项目配置酌情进行CV，

在做项目前，一定确定依赖之间是否冲突

4.本地测试为方便github提交只弄一个主分支，一个测试分支