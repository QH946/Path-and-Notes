1,第五阶段   web项目的开发:如何分析，设计，编码，测试。

2，crm项目。编程思想和编程习惯，关键。
  课堂上，听懂，跟上思路；
  课下，做项目。

3,crm的技术架构：
  视图层(View)：展示数据，跟用户交互。
                html,css,js,jquery,bootstrap(ext|easyUI),jsp
  控制层(Controller)：控制业务处理流程(接收请求,接收参数,封装参数;根据不同的请求调用业务层处理业务;根据处理结果，返回响应信息)
		(servlet,)springMVC(,webwork,struts1,struts2)
  业务层(Service)：处理业务逻辑(处理业务的步骤以及操作的原子性)
		 JAVASE(工作流:activiti|JBPM)
                 1,添加学生
		 2,记录操作日志
  持久层(Dao/Mapper)：操作数据库.
                (jdbc,)mybatis(,hibernate,ibatis)

		tbl_table----------pojo
  整合层：维护类资源,维护数据库资源
         spring(IOC,AOP)(,ejb,corba)

4,教学方式：26半天
  做web项目开发.

5,教学目的：
  1)对软件公司和软件开发有一定的了解
  2)了解CRM项目的核心业务
  3)能够独立完成CRM项目核心业务的开发
  4)对前期所学技术进行回顾,熟练,加深和扩展
  5)掌握互联网基础课：linux,redis,git

6,软件公司的组织结构：
  研发部(程序员,美工,DBA),测试部,产品部,实施部,运维部,市场部

7,软件开发的生命周期：
  1)招标：
    投标:----------标书
    
    甲方：
    乙方：
  2)可行性分析:---------可行性分析报告
    技术,经济
  3)需求分析:-----------需求文档
    产品经理,需求调研
    项目原型:容易确定需求,开发项目时作为jsp网页.
  4)分析与设计：
    

作业：复习前期所学技术：
  servlet,jsp(el,jstl)
  springMVC

  jquery1,软件开发的生命周期：
  1)招标：
    投标:----------标书
    
    甲方：
    乙方：
  2)可行性分析:---------可行性分析报告
    技术,经济
  3)需求分析:-----------需求文档
    产品经理,需求调研
    项目原型:容易确定需求,开发项目时作为jsp网页.
  4)分析与设计：
    架构设计：----------架构文档
             物理架构设计：
	         应用服务器:tomcat(apache),weblogic(bea-->oracle),websphere(ibm),jboss(redhat),resin(MS)
		            web  javaee:13种协议
			                servlet,jsp,xml,jdbc
					mq ....
		 数据库服务器：mysql,oracle,DB2,sqlserver,达梦
	     逻辑架构设计：代码分层.
	         视图层-->控制层-->业务层-->持久层-->数据库
	     技术选型：java,.net
    项目设计：---------项目设计文档
             物理模型设计：哪些表，哪些字段，字段的类型和长度，以及表和表之间的关系。
	                   powerdesigner-----xxxx.pdm
	     逻辑模型设计：哪些类，哪些属性和方法，方法的参数和返回值，以及类和类之间关系。
	                   rational rose-----.pdl
	     界面设计：企业级应用 朴素 -----项目原型
	               互联网应用 炫酷
             算法设计：------算法设计文档
  5)搭建开发环境：-----------技术架构文档
    创建项目,添加jar包,添加配置文件,添加静态页面,添加公共类以及其它资源;能够正常启动运行。
  6)编码实现：-------注释
  7)测试：-----------测试用例
  8)试运行：---------使用手册
  9)上线：-----------实施文档
  10)运维：----------运维手册

  11)文档编纂：

2,CRM项目的核心业务：
  1)CRM项目的简介：Customer Relationship Management 客户关系管理系统
    企业级应用,传统应用;给销售或者贸易型公司使用,在市场,销售,服务等各个环节中维护客户关系，
    CRM项目的宗旨：增加新客户,留住老客户，把已有客户转化为忠诚客户。
  2)CRM是一类项目,我们的CRM是给一个大型的进出口贸易公司来使用的，做大宗商品的进出口贸易;商品是受管家管制的。
  3)CRM项目的核心业务：
    系统管理功能：不是直接处理业务数据，为了保证业务管理的功能正常安全运行而设计的功能。
                  用户登录,安全退出,登录验证等
		  给超级管理员，开发和运维人员使用。
    业务管理功能：处理业务数据
                  市场活动：市场部，设计市场活动营销活动
		  线索：销售部(初级销售),增加线索
		  客户和联系人：销售部(高级销售),有效地区分和跟踪客户和联系人.
                  交易：销售部(高级销售),更好地区分和统计交易的各个阶段。
		  售后回访：客服部,妥善安排售后回访。主动提醒。
                  统计图表：管理层,统计交易表中各个阶段数据量。

    作业：复习前期学技术(笔记,代码,视频)
          jstl视频.


                  1,crm的表结构：
  tbl_user   用户表

  tbl_dic_type   数据字典类型表
  tbl_dic_value  数据字典值

  tbl_activity   市场活动表
  tbl_activity_remark  市场活动备注表

  tbl_clue       线索表
  tbl_clue_remark   线索备注表

  tbl_clue_activity_relation  线索和市场活动的关联关系表

  tbl_customer   客户表
  tbl_customer_remark  客户备注表

  tbl_contacts   联系人表
  tbl_contacts_remark 联系人备注表

  tbl_contacts_activity_relation 联系人和市场活动的关联关系表

  tbl_tran       交易表
  tbl_tran_remark  交易备注表
  tbl_tran_history  交易历史表

  tbl_task   任务表

  1)主键字段：在数据库表中，如果有一组字段能够唯一确定一条记录，则可以把它们设计成表的主键字段。
              推荐使用一个字段做主键，而且推荐使用没有业务含义的字段做主键,比如：id等。

	      主键字段的类型和长度由主键值的生成方式来决定：
	               主键值的生成方式：
		       1)自增：借助数据库自身主键生成机制
		               数值型 长度由数据量来决定
	
			       运行效率低
			       开发效率高
		       2)assighed：程序员手动生成主键值,唯一非空,算法.
		               hi/low：数值型 长度由数据量决定
			       UUID：字符串 长度是32位
		       3)共享主键：由另一张表的类型和长度决定
		               tbl_person         tbl_card
			       id     name        id     name
			       1001   zs          1001    card1
			       1002   ls
		       4)联合主键：由多个字段的类型和长度决定
   2)外键字段：用来确定表和表之间的关系。
              1)一对多：一张表(A)中的一条记录可以对应另一张表(B)中的多条记录;
	                另一张表(B)中的一条记录只能对应一张表(A)中的一条记录。
			A(1)---------B(n)
			父表         子表
			tbl_student                    tbl_class
			id      name class_id          id     name
			1001    zs    111              111    class1
			1002    ls    111              222    class2
			1003    ww    222
			1004    zl    

			添加数据时,先添加父表记录，再添加子表记录;
			删除数据时,先删除子表记录，再删除父表记录;
			查询数据时,可能会进行关联查询：
			//查询所有姓张的学生的id,name和所在班级name
			select s.id,s.name,c.name as className
			from tbl_student s
			join tbl_class c on s.class_id=c.id//假如外键不可以为空
			where s.name like 'z%'
	
			内连接：查询所有符合条件的数据，并且要求结果在两张表中都有相对应的记录
			左外连接：查询左侧表中所有符合条件的数据，即使在右侧表中没有相对应的记录
			         
		        *如果外键不能为空，优先使用内连接；
			 如果外键可以为空，
			                   --假如只需要查询那些在另一张表中有相对应的记录，使用内连接
					   --假如需要查询左侧表中所有符合条件的记录，使用左外连接.
	    2)一对一：一张表(A)中的一条记录只能对应另一张表(B)中的一条记录;
	              另一张表(B)中的一条记录也只能对应一张表(A)中的一条记录。
			tbl_person         tbl_card
			id     name        id     name
			1001   zs          1001    card1
		       a)共享主键：(不推荐)
		         添加数据：先添加先产生的表，再后产生的表记录
			 删除数据：先删除后产生的表记录，再删除先产生的表记录
			 查询数据：无需进行连接查询
			           //查询zhangsan的驾照信息  1001
				   select *
				   from tbl_card
				   where id='1001'
	                   b)唯一外键：
		         tbl_person             tbl_card
			 id     name            id     name     person_id(唯一性约束)
			 1001   zs              111    card1    1001
			 1002   ls              222    card2    1002
			 1003   ww              333    card3    1003
			 *一对一就是一种特殊的一对多。
			 *操作跟一对多完全一样。
	     3)多对多：一张表(A)中的一条记录可以对应另一张表(B)中的多条记录;
	               另一张表(B)中的一条记录也可以对应一张表(A)中的多条记录。
		       tbl_student                    tbl_course
		       id     name                    id     name   
		       1001   zs                      111    java   
		       1002   ls                      222    mysql  
		               tbl_student_course_relation
			        student_id     course_id
				1001            111
				1001            222
				1002            111
				1002            222
		       添加数据时，先添加父表记录(tbl_student,tbl_course),再添加子表(tbl_student_course_relation)记录;
		       删除数据时，先删除子表记录(tbl_student_course_relation),再删除父表记录(tbl_student,tbl_course)
		       查询数据时，可能会进行关联查询：
		          //查询所有姓张的学生的id,name,和所选课程的name
			  select s.id,s.name,c.name as courseName
			  from tbl_student s
	                      join tbl_student_course_relation scr on s.id=scr.student_id
	                      join tbl_course c on scr.course_id=c.id
			  where s.name like 'z%'
  3)给关于日期和时间的字段：
     都按照字符串处理：
     char(10)   yyyy-MM-dd
     char(19)   yyyy-MM-dd HH:mm:ss

2,创建crm的数据库实例：
  把sql脚本导入数据库实例：

3,搭建开发环境：
  1)创建项目：crm-project
              设置JDK.
    创建工程：crm
              补全目录结构：
  2)添加jar包：添加依赖---参考课件.


    1,搭建开发环境：
  1)创建项目：crm-project
              设置JDK.
    创建工程：crm
              补全目录结构：
	      设置编码格式：UTF-8
  2)添加jar包：添加依赖---参考课件.
  3)添加配置文件：参考课件.
  4)添加静态页面资源：
    webapps
       |->stumgr
       |->crm
            |->.html,.css,.js,.img   test.jsp
            |->WEB-INF
	          |->web.xml
		  |->classes
		  |->lib
   *web应用根目录下的内容都是不安全的，外界可以通过url直接访问;
    所以，一般为了数据的安全，都会把页面放到WEB-INF下,因为WEB-INF目录下的资源是受保护的，外界不能直接访问。
    
   http://127.0.0.1:8080/crm/test.jsp

   webapps
       |->stumgr
       |->crm
            |->.css,.js,.img   
            |->WEB-INF
	          |->web.xml
		  |->classes
		  |->lib
		  |->pages  test.jsp
  5)把crm项目部署到tomcat上：
    http://127.0.0.1:8080/crm

2,首页：
  1)分析需求：
  2)分析与设计：
  3)编码实现:
  4)测试:

3,用户登录：
  1,同步请求和异步请求的区别：
  同步请求：浏览器窗口发出的请求,响应信息返回到浏览器窗口,所以会进行全局刷新。
  异步请求：ajax发出的请求,响应信息返回到ajax的回调函数,既可以进行全局刷新，也可以进行局部刷新。

  小结：如果需要进行全局刷新，推荐使用同步请求，当然也可以使用异步请求；
        如果需要进行局部刷新，只能使用异步请求；
	如果既可能进行全局刷新，也可能进行局部刷新，也是只能使用异步请求。

2,mybatis逆向工程：
  1)简介：根据表生成mapper层三部分代码：实体类，mapper接口，映射文件。
  2)使用mybatis逆向工程：
    a)创建工程:crm-mybatis-generator
    b)添加插件：
      <!--myBatis逆向工程插件-->
    <plugin>
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-maven-plugin</artifactId>
	<version>1.3.2</version>
	<configuration>
	    <verbose>true</verbose>
	    <overwrite>true</overwrite>
	</configuration>
    </plugin>
   c)添加配置文件：
     数据库连接信息
     代码保存的目录
     表的信息
   d)运行mybatis的逆向工程，根据指定表生成java代码，保存到指定的目录中。

3,使用jquery获取指定元素的指定属性的值：
  选择器.attr("属性名");//用来获取那些值不是true/false的属性的值.
  选择器.prop("属性名");//用来获取值是true/false的属性的值.例如：checked,selected,readonly,disabled等。



1，把控制层(controller)代码中处理好的数据传递到视图层(jsp),使用作用域传递：
   pageContext:用来在同一个页面的不同标签之间传递数据。
   request：在同一个请求过程中间传递数据。
   session: 同一个浏览器窗口的不同请求之间传递数据。
   application:所有用户共享的数据，并且长久频繁使用的数据。

   <c:aaaa>
   <c:bbbb>

2，jquery事件函数的用法：
   选择器.click(function(){//给指定的元素添加事件
      //js代码
   });

   选择器.click();//在指定的元素上模拟发生一次事件

3,记住密码：
   访问：login.jsp---->后台：.html：如果上次记住密码，自动填上账号和密码;否则，不填。
                             如何判断上次是否记住密码？`
			     --上次登录成功，判断是否需要记住密码：如果需要记住密码，则往浏览器写cookie；否则，删除cookie。
			                     而且cookie的值必须是该用户的loginAct和loginPwd
			     --下次登录时，判断该用户有没有cookie：如果有，则自动填写账号和密码；否则，不写。
			                     而且填写的是cookie的值.
                  ----->浏览器显示
  获取cookie：
  1,使用java代码获取cookie：
    Cookie[] cs=request.getCookies();
    for(Cookie c:cs){
	if(c.getName().equals("loginAct")){
	    String loginAct=c.getValue();
	}else if(c.getName().equals("loginPwd")){
	    String loginPwd=c.getValue();
	}
    }
  2,使用EL表达式获取cookie：
    ${cookie.loginAct.value}
    ${cookie.loginPwd.value}

 1，登录验证：

   1)过滤器：
     a)implements Filter{
         --init
	 --doFilter
	 --destroy
       }
     b)配置过滤器:web.xml
   2)拦截器：
     a)提供拦截器类：implements HandlerInterceptor{
			  --pre
			  --post
			  --after
                     }
     b)配置拦截器：springmvc.xml
    
2，页面切割技术：
  1)<frameset>和<frame>：
    <frameset>：用来切割页面.
                <frameset cols="20%,60%,20%" rows="10%,80%,10%">
    <frame>：显示页面.
                <frame src="url">

		<frameset cols="20%,60%,20%">
			<frame src="url1" name="f1">
			<frame src="url2" name="f2">
			<frame src="url3" name="f3">
		</frameset>
	
	         每一个<frame>标签就是一个独立的浏览器窗口。
	
	     <a href="url" target="f3">test</a>
  2)<div>和<iframe>：
    <div>：切割页面。
            <div style="height:10%;width=20%">
   <iframe>:显示页面。
            <div style="height:10%;width=20%">
		<iframe href="url">
	    </div>
3,创建市场活动：

  模态窗口：模拟的窗口,本质上是<div>，通过设置z-index大小来实现的;
            初始时，z-index初始参数是<0，所以不显示；
	    需要显示时，z-index值设置成>0即可。

	    bootstrap来控制z-index的大小。
  控制模态窗口的显示与隐藏：
            1)方式一：通过标签的属性data-toggle="modal" data-target="模态窗口的id"
	    2)方式二：通过js函数控制：
	              选择器(选中div).modal("show");//显示选中的模态窗口
		      选择器(选中div).modal("hide");//关闭选中的模态窗口
            3)方式三：通过标签的属性data-dismiss=""
		      点击添加了data-dismiss=""属性的标签，自动关闭该标签所在的模态窗口。
  模态窗口的意义：
            window.open("url","_blank");
	    模态窗口本质上就是原来页面中的一个<div>，只有一个页面;所有的操作都是在同一个页面中完成。



       1，js日历：

   一类问题：
       1)实现起来比较复杂。
       2)跟业务无关。

   日历插件：bootstrap-datetimepicker
       前端插件使用步骤：
           1)引入开发包：.js,.css
	      下载开发包，拷贝到项目webapp目录下
	      把开发包引入到jsp文件中：<link><script>
	   2)创建容器：<input type="text"><div>
	   3)当容器加载完成之后，对容器调用工具函数.
       1,前端插件的使用步骤：
  1)引入开发包：.css,.js
    下载开发包,
    引入jsp页面,先引入被依赖技术的开发包。
  2)创建容器：<input type="text">
              <div>
  3)当容器加载完成之后，对容器调用工具函数:

2，在指定的标签中显示jsp页面片段：
   选择器.html(jsp页面片段的字符串);//覆盖显示
   选择器.append(jsp页面片段的字符串);//追加显示
   选择器.after(jsp页面片段的字符串);
   选择器.before(jsp页面片段的字符串);
   选择器.text(jsp页面片段的字符串);1，函数：如果一段用来完成特定功能的代码到处出现，可以封装成函数。
   函数的参数：在编写函数的过程中，如果有一个或者多个数据无法确定，可以把这些数据定义成函数的参数(形参)，将来由函数的调用者来传递参数的具体的值(实参)。

2，分页查询插件：bs_pagination

   前端插件的使用步骤：
       1,引入开发包：
       2,创建容器:<div>
       3,当容器加载完成之后，对容器调用工具函数:

3，块元素和行元素：

4,js的系统函数：
   1,eval()：作业
   2,parseInt()获取小数的整数部分

5,演示分页查询市场活动的过程：
  1,queryActivityByConditionForPage(1,10)
       |->把pageNo,pageSize和查询条件一起发送到后台，查询数据
       |->data
            |->activityList：遍历list，显示列表
	    |->totalRows:调用工具函数，显示翻页信息
  2,当用户切换页号或者每页显示条数时：pageNo,pageSize
       |->翻页信息会自动变化
       |->手动刷新列表：
              |->把pageNo,pageSize和查询条件一起发送到后台，查询数据
	      |->data
	           |->activityList：遍历list，显示列表
	           |->totalRows:调用工具函数，显示翻页信息
6，删除市场活动：


   1,在页面中给元素添加事件语法：
  1)使用元素的事件属性：onxxxx="f()"
  2)使用jquery对象：选择器.xxxx(function(){
                        //js代码
			//this
                    });
		    *只能给固有元素添加事件
		     固有元素：当调用事件函数给元素添加事件时，如果元素已经生成，则这些元素叫做固有元素；
		     动态生成的元素：当调用事件函数给元素添加事件时，如果元素还没有生成，后来生成的元素叫做动态生成的元素。
  3)使用jquery的on函数：父选择器.on("事件类型",子选择器,function(){
                              //js代码
			      //this
                        });

                        父元素:必须是固有元素,可以直接父元素,也可以是间接父元素.
    		       原则固有父元素范围越小越好.
    		事件类型：跟事件属性和事件函数一一对应。
    		子选择器：目标元素,跟父选择器构成一个父子选择器
    		*不但能给固有元素添加事件，还能够给动态生成的元素添加事件。

2,js中截取字符串：
  str.substr(startIndex,length);//从下标为startIndex的字符开始截取，截取length个字符
  str.substring(startIndex,endIndex)//从下标为startIndex的字符开始截取，截取到下标是endIndex的字符
  var str="beijing";
  str.substr(2,3);//iji
  str.substring(2,3);//ij
3,ajax向后台发送请求时，可以通过data提交参数,data的数据格式有三种格式：
  1)data:{
       k1:v1,
       k2:v2,
       ....
    }
    *劣势：只能向后台提交一个参数名对应一个参数值的数据，
           不能向后台提交一个参数名对应多个参数值的数据。
	   只能向后台提交字符串数据
     优势：操作简单
  2)data:k1=v1&k2:v2&....
    *优势：不但能够向后台提交一个参数名对应一个参数值的数据，
           还能向后台提交一个参数名对应多个参数值的数据。
     劣势：操作麻烦
           只能向后台提交字符串数据
  3)data:FormData对象
     优势：不但能提交字符串数据，
            还能提交二进制数据
     劣势：操作更复杂
  1，封装参数：
   1)如果做查询条件，或者参数之间不是属于一个实体类对象，封装成map
   2)如果做写数据，并且参数本来就是属于一个实体类对象，封装成实体类对象.

2，使用jquery获取或者设置指定元素的value属性值：
   获取：选择器.val();
   设置：选择器.val(属性值);

3,导出市场活动：
   1)给"批量导出"按钮添加单击事件，发送导出请求
   2)查询所有的市场活动
   3)创建一个excel文件，并且把市场活动写到excel文件中
   4)把生成的excel文件输出到浏览器(文件下载)

   技术准备：
       1)使用java生成excel文件：iText,apache-poi

         关于办公文档插件使用的基本思想：把办公文档的所有元素封装成普通的Java类，
                                     程序员通过操作这些类达到操作办公文档目的。
     文件---------HSSFWorkbook
     页-----------HSSFSheet
     行-----------HSSFRow
         列-----------HSSFCell
     样式---------HSSFCellStyle
    
     使用apache-poi生成excel：
         a)添加依赖：
                <dependency>
    	      <groupId>org.apache.poi</groupId>
    	      <artifactId>poi</artifactId>
    	      <version>3.15</version>
    	    </dependency>
         b)使用封装类生成excel文件：
           
       2)文件下载：
         filedownloadtest.jsp
     ActivityController
         |->fileDownload()
    
     *所有文件下载的请求只能发送同步请求。1,导入市场活动：

  1)把用户计算机上的excel文件上传到服务器(文件上传)
  2)使用java解析excel文件，获取excel文件中的数据
  3)把解析出来的数据添加数据库中
  4)返回响应信息

  技术准备：
      1)文件上传：
        fileuploadtest.jsp
	ActivityController
	        |->fileUpload()
      2)使用java解析excel文件：iText,apache-poi
        
	关于办公文档插件使用的基本思想：把办公文档的所有元素封装成普通的Java类，
	                                程序员通过操作这些类达到操作办公文档目的。
	 文件---------HSSFWorkbook
	 页-----------HSSFSheet
	 行-----------HSSFRow
	     列-----------HSSFCell
	    1,文件上传：上传的文件跟用户约定好的。

2,js截取字符串：
  str.substr(startIndex,length)
  str.substr(startIndex)//从下标为startIndex的字符开始截取，截取到字符串的最后
  str.substring(startIndex,endIndex)

3,查看市场活动明细：
  tbl_activity                   tbl_activity_remark
  id    name                     id      note_content    activity_id
  1001  act1                     111     remark1         1001
  1002  act2                     222     remark2         1001
                                 333     remark3         1002
  //查询act1下所有的备注
  select *
  from tbl_activity_remark
  where activity_id=10011，使用标签保存数据，以便在需要的时候能够获取到这些数据:

   给标签添加属性：
       如果是表单组件标签，优先使用value属性，只有value不方便使用时，使用自定义属性;
       如果不是表单组件标签，不推荐使用value，推荐使用自定义属性。

   获取属性值时：
       如果获取表单组件标签的value属性值：dom对象.value
                                          jquery对象.val()
       如果自定义的属性，不管是什么标签，只能用：jquery对象.attr("属性名");

       <a style="text-decoration: none; cursor: pointer;" onclick="">ii测试03_2</a>
    
       window.location.href='workbench/activity/detailActivity.do?id=74f60cc6f64c452fbf02c1aaa3b7e2fb1,添加市场活动备注：

  jsp的运行原理：
  xxx.jsp：1)、tocmat中运行：
              把xxx.jsp翻译成一个servlet,
	      运行servlet,运行的结果是一个html网页
	      把html网页输出到浏览器
	   2)、html网页在浏览器上运行：
	      先从上到下加载html网页到浏览器，在加载过程中，运行前端代码
	      当页面都加载完成，再执行入口函数.

2,把页面片段显示在动态显示在页面中：
  选择器.html(htmlStr)：覆盖显示在标签的内部
  选择器.text(htmlStr)：覆盖显示在标签的内部
  选择器.append(htmlStr)：追加显示在指定标签的内部的后边
   <div id="myDiv">
      aaaaaaaaa
      bbbbbbbbb
   </div>
   var htmlStr="<p>ccccccccc</p>";
    $("#myDiv").append(htmlStr);
    <div id="myDiv">
      aaaaaaaaa
      bbbbbbbbb
      <p>ccccccccc</p>
   </div>
  选择器.after(htmlStr)：追加显示在指定标签的外部的后边
   <div id="myDiv">
      aaaaaaaaa
      bbbbbbbbb
   </div>
   var htmlStr="<p>ccccccccc</p>";
    $("#myDiv").after(htmlStr);
    <div id="myDiv">
      aaaaaaaaa
      bbbbbbbbb
   </div>
   <p>ccccccccc</p>
   选择器.before(htmlStr)：追加显示在指定标签的外部的前边
   <div id="myDiv">
      aaaaaaaaa
      bbbbbbbbb
   </div>
   var htmlStr="<p>ccccccccc</p>";
    $("#myDiv").before(htmlStr);
    <p>ccccccccc</p>
    <div id="myDiv">
      aaaaaaaaa
      bbbbbbbbb
   </div>

3,给元素扩展属性：html页面是可扩展的标记语言，可以给指定的标签任意扩展属性，只要属性名符合标识符的命名规则即可。
  两个目的：
      1)使用标签保存数据：
        如果是表单组件标签，优先使用value属性，只有value不方便使用时，使用自定义属性;
        如果不是表单组件标签，不推荐使用value，推荐使用自定义属性。
      2)定位标签：
        优先考虑id属性,其次考虑name属性，只有id和name属性都不方便使用时，才考虑使用自定义属性。
1，线索：初级销售 
         数据量最大，每一条数据最详细：tbl_clue

	 有购买意向的线索，转换到高级销售阶段;
	 没有购买意向的线索，删除掉.
	
	     创建线索，查看线索明细，线索关联市场活动，解除线索和市场活动的关联关系,线索转换

2,创建线索：

   tbl_dic_type code   存储下拉列表的类型的，每一个下拉列表在tbl_dic_type对应一条记录,主键值都是各自的编码,有含义的字段做主键，在程序如果需要用到这些主键值，可以直接使用。

   tbl_dic_value    type_code  存储每一个下拉列表中的选项值，通过type_code区分选项值属于哪一个下拉列表

   queryDicValueByTypeCode("type")1，查看线索明细：

  tbl_clue
  tbl_clue_remark
  tbl_activity

  tbl_clue                  tbl_acvtivity
  id     fullname           id        name
  1001   clue1              111       act1
  1002   clue2              222       act2

        tbl_clue_activity_relation
    clue_id             activity_id
        1001                111
        1001                222
    1002                111
    1002                222
  //查询clue1关联的市场活动信息  1001
  select id,name
  from tbl_acvtivity a
  join tbl_clue_activity_relation car on a.id=car.activity_id
  where car.clue_id=1001

2,线索关联市场活动：
  根据activityName查询所有符合条件的市场活动
  根据clueId查询已经关联过的市场活动1，线索转换：
   线索是给初级销售人员使用；如果线索没有购买意向，则删除线索，如果线索有购买意向，则把该线索信息转换到客户和联系人表中，把该线索删除。

   数据转换：
       把该线索中有关公司的信息转换到客户表中
       把该线索中有关个人的信息转换到联系人表中
       把该线索下所有备注信息转换到客户备注表中一份
       把该线索下所有备注信息转换到联系人备注表中一份
       把该线索和市场活动的关联关系转换联系人和市场活动的关联关系表中
       如果需要创建交易，则往交易表中添加一条记录
       如果需要创建交易，则还需要把该线索下所有备注转换到交易备注表中一份
       删除该线索下所有的备注
       删除该线索和市场活动的关联关系
       删除该线索

       以上所有操作必须在同一个事务中完成,在同一个service方法中完成。



1,交易：
  创建交易,查看交易明细1，可能性的可配置：用户提供配置文件，配置每一个阶段对应一个可能性；
                   当用户选择阶段时，根据阶段获取可能性，显示到输入框。

   1)提供配置文件：由用户提供,保存在后台服务器上。
     配置文件：
         a)xxxx.properties配置文件：key1=value1
	                            key2=value2
				    .....

				    适合配置简单数据，几乎没有冗余数据，效率高
				    解析相对简单：Properties，BundleResource
	 b)xxx.xml配置文件：标签语言.
	   <studentList>
		   <student email="zs@163.com">
			<id>1001</id>
			<name>zs</name>
			<age>20</age>
		   </student>
		   <student email="ls@163.com">
			<id>1002</id>
			<name>ls</name>
			<age>20</age>
		   </student>
	   </studentList>
	                            适合配置复杂数据，产生冗余数据，效率低
				    解析相对复杂：dom4j,jdom
	  配置可能性：possibility.properties
	              阶段的名称做key，可能性做value

   2)用户每次选择阶段，向后台发送请求。
   3)后台提供controller，接收请求，根据选择的阶段，解析配置文件，获取对应的可能性。
   4)把可能性返回前台，显示在输入框。

2，客户名称自动补全：

   1)给输入框添加键盘弹起事件
   2)后台接收到请求，根据名称模糊查询，返回到客户端
   3)把查询到的数据显示在输入框下边
     用户选择一个客户，实现自动补全

   自动补全插件：bs_typeahead
        1)引入开发包：.css,.js
	2)创建容器：<div> <input type="text">
	3)当容器加载完成之后，对容器调用工具函数：
	  
     

   1，查看交易明细：

2,java中的实体类是为了操作数据库表，所以，实体类要和数据库中的表相对应，实体类中的属性要和表中的字段相对应，属性的类型要和表字段的类型相对应。

  java中的实体类不是只为了操作表，还有可能进行数据传输，所以，java中的实体类在数据库中不一定有表相对应，实体类中的属性在数据库表中也不一定有字段相对应；
                                  但是，数据库一张表在java中一定有实体类相对应，数据库表中一个字段在实体类中一定有属性相对应。
				  java--------->table(不一定)
				      <---------(一定)
3,分析交易阶段的图标：
  图标的数量：跟交易总的阶段数量一致
              每一个阶段对应显示一个图标
  图标的种类：三类
  图标的颜色：绿色和黑色
  图标的顺序：跟阶段的顺序一致
  图标数量的变化：阶段的数量可能变化，图标的数量也可能变化
  图标的实现：
            <span class="glyphicon glyphicon-ok-circle" data-content="" style="color: #90F790;">
	    -----------

  显示交易阶段的图标：
      按照顺序查询交易所有的阶段：stageList
      遍历stageList,显示每一个阶段对应图标，图标上显示的阶段的名称从遍历出的阶段中获取。

4,统计图表：以更专业、更形象的形式展示系统中的数据。

  销售漏斗图：展示商品销售数据、销售业绩
              展示交易表中的数据,统计交易表中各个阶段的数量

  1,统计图表：以更专业、更形象的形式展示系统中的数据。

  销售漏斗图：展示商品销售数据、销售业绩
              展示交易表中的数据,统计交易表中各个阶段的数量

  报表插件:jfreechart,iReport,锐浪,echarts
           
  echarts的使用：
         1)引入开发包：echarts.min.js
	 2)创建容器：<div id="main" style="width: 600px;height:400px;"></div>
	 3)当容器加载完成之后，对容器调用工具函数：