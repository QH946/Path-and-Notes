1.Service层负责所有CRUD，Controller层负责根据条件返回数据
(阿里规范文档)
2.企业级开发时分包：
setting代表系统管理功能（与用户相关）
workbench代表业务管理功能（CRUD）

index本身占一个controller
user占一个controller
workbench下的业务占一个controller
3.
携带数据使用请求转发
不携带数据使用重定向
4.idea正则表达式删除所有空行
^\s*\n
5.引用
 :很遗憾，你能找到的项目，别人都写过了……
我最近想写一个车辆故障管理系统，其中一个亮点就是把车子的故障码和bug进行多对多关联，然后进行分析。
当然你也可以写一个类似比如化验单的指标和疑似诊断的系统，帮助医生定位病症（比如xxx指标大于yyy时，zz%的患者是abc病）。
这种系统完全可以根据这个crm来魔改实现。这样的东西比较有价值，面试官更愿意听你讲
6.return retMap;
封装的map对象自动转为json对象
（Map、实体类、List、数组、基本数据类型）皆可转为json
7.先通过UML图（主要是时序图）分析整个业务功能的流程，再根据时序图敲码
写项目时，从后往前写（mapper->service->controller）

8.优化后的模糊查询(以后都要使用这种方式)
    <select id="getByNameGood" parameterType="string" resultType="users">
        select id,username,birthday,sex,address
        from users
        where username like concat('%',#{name},'%')
    </select>
9.
完成ajax请求访问服务器,返回学生集合时:
返回json格式的对象.自动将对象或集合转为json.使用的jackson工具进行转换
10.所有文件下载的请求只能发同步请求
11.各层分工
controller：接受啥？返回啥？
service:怎么查
dao：查谁
12.参数保持一致
1)批量删除
#mapper:
int deleteActivityByIds(String[] ids);
<delete id="deleteActivityByIds" parameterType="string">
        delete from tbl_activity where id in
        <foreach collection="array" item="id" separator="," open="(" close=")">
            #{id}
        </foreach>
    </delete>
-----------------------------------------------------
##service:
int deleteActivityByIds(String[] ids);
public int deleteActivityByIds(String[] ids)
-----------------------------------------------------
##controller:
public Object deleteActivityByIds(String[] id)

2）根据id选择
mapper:
List<Activity> selectActivitiesByIds(String[] ids);
<select id="selectActivitiesByIds" parameterType="string" resultMap="BaseResultMap">
-----------------------------------------------------
service:
List<Activity> queryActivitiesByIds(String[] ids);
public List<Activity> queryActivitiesByIds(String[] ids)
-----------------------------------------------------
controller:
public void exportActivitiesByIds(String[] id, HttpServletResponse response) throws IOException

《注：controller层方法中的形参要与客户端发来的参数保持一致即id》

3)复选框checkbox
每个按钮模块中：
var checkedIds = $("#tBody input[type='checkbox']:checked");
var后的变量使用相同的名称（名称自己设定）

13.
多表连接时
先分析好每个表字段之间的关系如外键，
写查询语句时，若字段可以为空则使用左连接(edit_by:修改者)
若字段不能为空则使用内连接，
外键字段需两表之间连接（on）后,才能在select后使用
elect c.id,c.fullname,dv1.value as appellation,u1.name as owner,c.company,c.job,c.email,c.phone,c.website,
           c.mphone,dv2.value as state,dv3.value as source,u2.name as create_by,c.create_time,u3.name as edit_by,
           c.edit_time,c.description,c.contact_summary,c.next_contact_time,c.address
    from tbl_clue c
           left join tbl_dic_value dv1 on c.appellation=dv1.id
           join tbl_user u1 on c.owner=u1.id
           left join tbl_dic_value dv2 on c.state=dv2.id
           left join tbl_dic_value dv3 on c.source=dv3.id
           join tbl_user u2 on c.create_by=u2.id
           left join tbl_user u3 on c.edit_by=u3.id
    where c.id=#{id}
    
14.
404一般为资源路径错误，仔细检查每层的路径（尤其是controller层）
500可能为调用循环或递归，可能堆栈空间不足
本人因在查看线索明细中调用了查看活动明细的方法，而活动mapper层未写查看活动明细的方法（仅在service层写方法便在controller层调用）
导致
ava.lang.StackOverflowError
	com.QH.crm.workbench.service.impl.ActivityServiceImpl.queryActivityForDetailByClueId(ActivityServiceImpl.java:74)
	com.QH.crm.workbench.service.impl.ActivityServiceImpl.queryActivityForDetailByClueId(ActivityServiceImpl.java:74)
	com.QH.crm.workbench.service.impl.ActivityServiceImpl.queryActivityForDetailByClueId(ActivityServiceImpl.java:74)
................................................................

15.命名方法时，根据操作直白含义驼峰命名，且为简洁直观（少敲代码），
尽量将单词统一为单数（Activity）
exportCheckedActivity（导出选中的活动）

16.Controller层方法定义类型
若跳转页面则定义成String
若CRUD则可定义成Object
方法内的参数由service、mapper层决定，在传实体类时，
有时注意补充缺少的参数
如在public Object saveCreateCustomer(Customer customer, HttpSession session)创建客户方法中，传入了一个实体类
但实体类中包含的参数个数与mapper层的参数个数不一致，需要手动补充
缺少的参数
// 补充参数
        customer.setCreateBy(user.getId());
        customer.setCreateTime(DateUtils.formatDateTime(new Date()));
        customer.setId(UUIDUtils.getUUID());
        ReturnObject returnObject = new ReturnObject();
        创建者的id、创建时间、创建的客户id
        这些参数在用户的详细信息中缺少，且在弹出的模态窗口中不存在
        
17.
1.e.printStackTrace()是打印异常栈信息
2.throw new RuntimeException(e)是吧异常包在一个运行时异常中抛出
第一句话感觉实际开发意义不大，很少有人会去看控制台打印。。
第二种是把异常继续抛出，要么由上层方法解决，要么终止程序进行，应用范围比较广。
try catch、循环、选择快捷键
选中要包裹代码 + Ctrl + Alt +t

18.
解决bug流程很简单
1.f12看请求是不是200（正常），正常的话再看一下响应的数据是不是正确的
2.看控制台有没有报错
3.根据报错显示的信息找到对应的代码修改即可
请求有很多重要参数要看

1.负载就是你传的参数（有可能少传或错传）请求有很多重要参数要看
2.请求的url

19.
模糊查询【重点】
由于模糊查询传入是一个字符串

模糊查询的三种方式

方式一：concat('%',#{值},'%')
使用concat函数，来拼接字符串

 select * from t_user where username like concat('%',#{username},'%');
1
方式二：'%' #{值} '%'【推荐使用这种方式】（注意两个''和#{间有空格）
select * from t_user where username like '%' #{username} '%';

方式三：
使用 bind 标签
public UserDto selectByLike(@Param("_name") String name, @Param("_note") String note);

<select id="selectByLike">
    <bind name="user_name" value="'%' + _name + '%'"/>
    <bind name="user_note" value="'%' + _note + '%'"/>
    SELECT * FROM t1 WHERE name LIKE #{user_name} AND note LIKE #{user_note}
</select>
如果报错如下，说明 MyBatis 版本过低，需要升级版本
Cause: org.xml.sax.SAXParseException;  必须声明元素类型 "bind"`

20.做项目前，先检查maven是否冲突，推荐maven helper,若爆红
则更换指定版本直至不红