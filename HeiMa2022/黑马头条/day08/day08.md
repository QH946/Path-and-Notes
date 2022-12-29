## Day08 平台管理

## 1)项目搭建

### 1.1)后端项目

![image-20220726084958201](assets\image-20220726084958201.png)

①：导入 "heima-leadnews-admin-gateway" 到 "heima-leadnews-gateway" 下

②：检查nacos配置文件 leadnews-admin-gateway.yml 输入以下内容

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        add-to-simple-url-handler-mapping: true
        corsConfigurations:
          '[/**]':
            allowedHeaders: "*"
            allowedOrigins: "*"
            allowedMethods:
              - GET
              - POST
              - DELETE
              - PUT
              - OPTION
      routes:
        # 平台管理
        - id: admin
          uri: lb://leadnews-admin
          predicates:
            - Path=/admin/**
          filters:
            - StripPrefix= 1
        - id: wemedia
          uri: lb://leadnews-wemedia
          predicates:
            - Path=/wemedia/**
          filters:
            - StripPrefix= 1
```

③：导入 "heima-leadnews-admin" 到 "heima-leadnews-service" 下

④：检查nacos配置文件 leadnews-admin.yml 输入以下内容

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.200.130:3306/leadnews_admin?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: tommaster8848
# 设置Mapper接口所对应的XML文件位置，如果你在Mapper接口中有自定义方法，需要进行该配置
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  # 设置别名包扫描路径，通过该属性可以给包中的类注册别名
  type-aliases-package: com.heima.model.admin.pojos
```

⑤：导入 leadnews_admin.sql

⑥：启动 "heima-leadnews-admin-gateway" 和 "heima-leadnews-admin"

### 1.2)前端项目

①：启动nginx

②：浏览器访问 http://localhost:8803/

③：点击登录测试

![image-20220719172037009](assets\image-20220719172037009.png)



## 2)需求开发

### 2.1)频道管理

在wemedia项目中开发频道管理相关接口。

#### 2.1)新增

- 前台输入内容进行频道的保存
- 频道名词不能重复

![image-20210725225940936](平台管理-需求说明.assets\image-20210725225940936.png)

##### 2.1.1)接口分析

|          | **说明**             |
| -------- | -------------------- |
| 接口路径 | /api/v1/channel/save |
| 请求方式 | POST                 |
| 参数     | WmChannel            |
| 响应结果 | ResponseResult       |


请求参数 AdChannelDto


```javascript
{
	"createdTime": "",        // 创建时间
	"description": "",        // 频道描述
	"name": "",               // 频道名称
	"ord": 0,                 // 排序方式
	"status": true            // 是否启动
}
```

响应结果

```javascript
{
	"code": 0,
	"data": {},
	"errorMessage": "",
	"host": ""
}
```

##### 2.1.2)代码实现

![image-20220725235953786](assets\image-20220725235953786.png)

①：WmChannelService 添加方法

```java
/**
  * 保存
  * @param wmChannel
  * @return
  */
ResponseResult channelSave(WmChannel channel);
```

②：WmChannelServiceImpl 添加方法

```java
@Override
public ResponseResult channelSave(WmChannel channel) {
    if (channel == null || channel.getOrd() < 0) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }

    String name = channel.getName();
    if (StringUtils.isEmpty(name)) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE, "频道名不能为空");
    }

    LambdaQueryWrapper<WmChannel> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(WmChannel::getName, name);
    Integer count = wmChannelMapper.selectCount(wrapper);
    if (count > 0) {
        return ResponseResult.errorResult(AppHttpCodeEnum.DATA_EXIST, "频道不能重名");
    }

    channel.setCreatedTime(new Date());
    boolean saveResult = this.save(channel);

    return saveResult ? ResponseResult.okResult(AppHttpCodeEnum.SUCCESS) : ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR);
}
```

③：控制器 WmchannelController 中添加方法

```java
@PostMapping("/save")
public ResponseResult save(@RequestBody WmChannel wmChannel) {
    return wmChannelService.channelSave(wmChannel);
}
```



#### 2.2)查询列表

![image-20210725230126215](平台管理-需求说明.assets\image-20210725230126215.png)

- 查询需要按照创建时间倒序查询
- 按照频道名称模糊查询
- 可以按照状态进行精确查找（1：启用   true，0：禁用   false）（该功能前端未实现）
- 分页查询

##### 2.2.1)接口分析

|          | **说明**             |
| -------- | -------------------- |
| 接口路径 | /api/v1/channel/list |
| 请求方式 | POST                 |
| 参数     | ChannelDto           |
| 响应结果 | PageResponseResult   |

参数

```json
{
	"name": "",         // 频道名（模糊查询）
	"page": 0,
	"size": 0
}
```

响应结果

```json
{
	"host": null,
	"code": 200,
	"errorMessage": null,
	"data": [
		{
			"id": 9,
			"name": "小黑马",
			"description": "哈哈哈",
			"isDefault": null,
			"status": true,
			"ord": 1,
			"createdTime": "2022-07-26T02:58:16.000+00:00"
		}
	],
	"currentPage": 1,
	"size": 10,
	"total": 1
}
```



##### 2.2.2)代码实现

①：ChannelDto

```java
package com.heima.model.wemedia.dtos;

import com.heima.model.common.dtos.PageRequestDto;
import lombok.Data;

@Data
public class ChannelDto extends PageRequestDto {
    /**
     * 频道名称
     */
    private String name;
}
```

②：WmChannelService

```java
/**
  * 查询
  * @param dto
  * @return
  */
PageResponseResult channelList2(ChannelDto dto);
```

③：WmChannelServiceImpl 添加方法

```java
@Override
public PageResponseResult channelList2(ChannelDto dto) {
    if (dto == null) {
        return new PageResponseResult(0, 0, 0);
    }

    Integer page = dto.getPage();
    Integer size = dto.getSize();

    page = page == null ? 1 : page;
    size = size == null ? 10 : size;

    Page<WmChannel> pageInfo = new Page<>(page, size);

    String name = dto.getName();
    LambdaQueryWrapper<WmChannel> wrapper = new LambdaQueryWrapper<>();
    wrapper.like(StringUtils.isNotBlank(name), WmChannel::getName, name);
    wrapper.orderByDesc(WmChannel::getCreatedTime);

    Page<WmChannel> pageResult = wmChannelMapper.selectPage(pageInfo, wrapper);

    Long total = pageResult.getTotal();
    PageResponseResult pageResponseResult = new PageResponseResult(page, size, total.intValue());
    pageResponseResult.setData(pageResult.getRecords());

    return pageResponseResult;
}
```

④：控制器 WmchannelController 中添加方法

```java
@PostMapping("/list")
public PageResponseResult list(@RequestBody ChannelDto dto) {
    return wmChannelService.channelList2(dto);
}
```

#### 2.3)修改

![image-20210725230556844](平台管理-需求说明.assets\image-20210725230556844.png)

- 点击编辑后可以修改频道
- 如果频道被引用则不能禁用

##### 2.3.1)接口分析

|          | **说明**               |
| -------- | ---------------------- |
| 接口路径 | /api/v1/channel/update |
| 请求方式 | POST                   |
| 参数     | WmChannel              |
| 响应结果 | ResponseResult         |

参数

```json
{        
	"description": "",            // 介绍
	"id": 0,                      // 频道id
	"name": "",                   // 频道名
	"ord": 0,                     // APP排序
	"status": true                // 是否启用
}
```

响应结果

```json
{
	"code": 0,
	"data": {},
	"errorMessage": "",
	"host": ""
}
```

##### 2.3.2)代码实现

![image-20220726000311467](assets\image-20220726000311467.png)

①：WmChannelService 添加方法

```java
/**
  * 修改
  * @param wmChannel
  * @return
  */
ResponseResult channelUpdate(WmChannel wmChannel);
```

②：WmChannelServiceImpl 添加方法

```java
@Override
public ResponseResult channelUpdate(WmChannel wmChannel) {
    if (wmChannel == null || wmChannel.getOrd() < 0) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }

    LambdaQueryWrapper<WmNews> newsWrapper = new LambdaQueryWrapper<>();
    newsWrapper.eq(WmNews::getChannelId, wmChannel.getId());
    newsWrapper.eq(WmNews::getStatus, 9);
    Integer newsCount = wmNewsMapper.selectCount(newsWrapper);
    if (newsCount > 0) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE, "频道关联了新闻，无法修改");
    }

    boolean updateResult = this.updateById(wmChannel);
    return updateResult ? ResponseResult.okResult(AppHttpCodeEnum.SUCCESS) : ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR);
}
```

③：控制器 WmchannelController 中添加方法

```java
@PostMapping("/update")
public ResponseResult update(@RequestBody WmChannel wmChannel) {
    return wmChannelService.channelUpdate(wmChannel);
}
```



#### 2.4)删除

只有禁用的频道才能删除

##### 2.4.1)接口分析

|          | **说明**                 |
| -------- | ------------------------ |
| 接口路径 | /api/v1/channel/del/{id} |
| 请求方式 | GET                      |
| 参数     | 无                       |
| 响应结果 | ResponseResult           |

##### 2.4.2)代码实现

![image-20220726000517738](assets\image-20220726000517738.png)

①：WmChannelService 添加方法

```java
/**
  * 删除
  * @param id
  * @return
  */
ResponseResult channelDelete(Integer id);
```

②：WmChannelServiceImpl 添加方法

```java
@Override
public ResponseResult channelDelete(Integer id) {
    if (id == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }

    WmChannel wmChannel = wmChannelMapper.selectById(id);
    if (wmChannel == null) {
        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }

    // 判断频道状态
    Boolean status = wmChannel.getStatus();
    if (status) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID, "频道为启动状态，不能删除");
    }

    // 判断频道关联
    LambdaQueryWrapper<WmNews> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(WmNews::getChannelId, wmChannel.getId());
    wrapper.eq(WmNews::getStatus, 9);
    Integer newsCount = wmNewsMapper.selectCount(wrapper);
    if (newsCount > 0) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE, "频道关联了新闻，不能删除");
    }

    boolean removeResult = this.removeById(id);
    return removeResult ? ResponseResult.okResult(AppHttpCodeEnum.SUCCESS) : ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR);
}
```

③：控制器 WmchannelController 中添加方法

```java
@GetMapping("/del/{id}")
public ResponseResult del(@PathVariable Integer id) {
    return wmChannelService.channelDelete(id);
}
```



### 3)敏感词管理

#### 3.1)新增

- 弹出的输入框，输入敏感词可直接保存
- 已存在的敏感词则不能保存

![image-20210725230741203](平台管理-需求说明.assets\image-20210725230741203.png)

##### 3.1.1)接口分析

|          | **说明**               |
| -------- | ---------------------- |
| 接口路径 | /api/v1/sensitive/save |
| 请求方式 | POST                   |
| 参数     | WmSensitive            |
| 响应结果 | ResponseResult         |

参数

```json
{
	"createdTime": "",
	"id": 0,
	"sensitives": ""
}
```

##### 3.1.2)代码实现

①：WmSensitiveService

```java
/**
     * 新增
     * @param wmSensitive
     * @return
     */
ResponseResult insert(WmSensitive wmSensitive);
```

②：WmSensitiveServiceImpl

```java
@Override
public ResponseResult insert(WmSensitive wmSensitive) {
    //1.检查参数
    if(null == wmSensitive){
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
    }

    //已存在的敏感词，不能保存
    WmSensitive sensitive = getOne(Wrappers.<WmSensitive>lambdaQuery().eq(WmSensitive::getSensitives, wmSensitive.getSensitives()));
    if(sensitive != null){
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID, "敏感词已存在");
    }

    //2.保存
    wmSensitive.setCreatedTime(new Date());
    save(wmSensitive);
    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
}
```

③：WmSensitiveController

```java
@PostMapping("/save")
public ResponseResult insert(@RequestBody WmSensitive wmSensitive){
    return wmSensitiveService.insert(wmSensitive);
}
```

#### 3.2)查询列表

- 查询需要按照创建时间倒序查询
- 按照敏感词名称模糊查询
- 分页查询

![image-20210725230914763](平台管理-需求说明.assets\image-20210725230914763.png)

##### 3.2.1)接口分析

|          | **说明**               |
| -------- | ---------------------- |
| 接口路径 | /api/v1/sensitive/list |
| 请求方式 | POST                   |
| 参数     | SensitiveDto           |
| 响应结果 | PageResponseResult     |

参数

```json
{
	"name": "",
	"page": 0,
	"size": 0
}
```

响应结果

```json
{
    "host":null,
    "code":200,
    "errorMessage":null,
    "data":[
        {
            "id":"3109",
            "sensitives":"无抵押贷款",
            "createdTime":1621739441000
        },
        {
            "id":"3110",
            "sensitives":"广告代理",
            "createdTime":1621739459000
        },
        {
            "id":"3111",
            "sensitives":"代开发票",
            "createdTime":1621739478000
        },
        {
            "id":"3112",
            "sensitives":"蚁力神",
            "createdTime":1621739499000
        },
        {
            "id":"3113",
            "sensitives":"售肾",
            "createdTime":1621739528000
        }
    ],
    "currentPage":1,
    "size":10,
    "total":16
}
```

##### 3.2.1)代码实现

①：SensitiveDto

```java
package com.heima.model.wemedia.dtos;

import com.heima.model.common.dtos.PageRequestDto;
import lombok.Data;

@Data
public class SensitiveDto extends PageRequestDto {

    /**
     * 敏感词名称
     */
    private String name;
}
```

②：WmSensitiveService

```java
/**
  * 查询
  * @param dto
  * @return
  */
ResponseResult list(SensitiveDto dto);
```

③：WmSensitiveServiceImpl

```java
@Override
public ResponseResult list(SensitiveDto dto) {
    //1.检查参数
    if(dto == null){
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
    }
    //检查分页
    dto.checkParam();

    //2.模糊查询 + 分页
    IPage page = new Page(dto.getPage(),dto.getSize());
    LambdaQueryWrapper<WmSensitive> lambdaQueryWrapper = new LambdaQueryWrapper<>();
    if(StringUtils.isNotBlank(dto.getName())){
        lambdaQueryWrapper.like(WmSensitive::getSensitives,dto.getName());
    }
    page = page(page,lambdaQueryWrapper);

    //3.结果返回
    ResponseResult responseResult = new PageResponseResult(dto.getPage(),dto.getSize(),(int)page.getTotal());
    responseResult.setData(page.getRecords());
    return responseResult;
}
```

④：WmSensitiveController

```java
@PostMapping("/list")
public ResponseResult list(@RequestBody SensitiveDto dto){
    return wmSensitiveService.list(dto);
}
```

#### 3.3)修改

![image-20210725230942502](平台管理-需求说明.assets\image-20210725230942502.png)

##### 3.3.1)接口分析

|          | **说明**                 |
| -------- | ------------------------ |
| 接口路径 | /api/v1/sensitive/update |
| 请求方式 | POST                     |
| 参数     | WmSensitive              |
| 响应结果 | ResponseResult           |

参数

```json
{
	"createdTime": "",
	"id": 0,
	"sensitives": ""
}
```

##### 3.3.2)代码实现

①：WmSensitiveService

```java
/**
  * 修改
  * @param wmSensitive
  * @return
  */
ResponseResult update(WmSensitive wmSensitive);
```

②：WmSensitiveServiceImpl

```java
@Override
public ResponseResult update(WmSensitive wmSensitive) {
    //1.检查参数
    if(null == wmSensitive || wmSensitive.getId() == null){
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
    }
    //2.修改
    updateById(wmSensitive);
    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
}
```

③：WmSensitiveController

```java
@PostMapping("/update")
public ResponseResult update(@RequestBody WmSensitive wmSensitive){
    return wmSensitiveService.update(wmSensitive);
}
```

#### 3.4)删除

直接删除即可

##### 3.4.1)接口分析

|          | **说明**                   |
| -------- | -------------------------- |
| 接口路径 | /api/v1/sensitive/del/{id} |
| 请求方式 | DELETE                     |
| 参数     | 无                         |
| 响应结果 | ResponseResult             |

##### 3.4.2)代码实现

①：WmSensitiveService

```java
/**
  * 删除
  * @param id
  * @return
  */
ResponseResult delete(Integer id);
```

②：WmSensitiveServiceImpl

```java
@Override
public ResponseResult delete(Integer id) {
    //1.检查参数
    if(id == null){
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
    }

    //2.查询敏感词
    WmSensitive wmSensitive = getById(id);
    if(wmSensitive == null){
        return ResponseResult.errorResult(AppHttpCodeEnum.DATA_NOT_EXIST);
    }

    //3.删除
    removeById(id);
    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
}
```

③：WmSensitiveController

```java
@DeleteMapping("/del/{id}")
public ResponseResult delete(@PathVariable("id") Integer id){
    return wmSensitiveService.delete(id);
}
```

### 4)用户认证审核

在app端的个人中心用户可以实名认证，需要材料为：姓名、身份证号、身份证正面照、身份证反面照、手持照片、活体照片（通过**微笑、眨眼、张嘴、摇头、点头**等组合动作，确保操作的为真实活体人脸。），当用户提交审核后就到了后端让运营管理人员进行审核

- 平台运营端查看用户认证信息，进行审核
- 用户通过审核后需要开通自媒体账号（该账号的用户名和密码与app一致）

**注意：**

- 在 heima-leadnews-user 服务中心开发以下接口

- 用户认证信息存储在 leadnews_user 库的 ap_user_realname 表中

- 确保网关配置文件 leadnews-admin-gateway.yml 中有以下配置

  ```json
  - id: user
            uri: lb://leadnews-user
            predicates:
              - Path=/user/**
            filters:
              - StripPrefix= 1
  ```

  

#### 4.1)分页查询认证列表

- 可根据审核状态条件查询
- 需要分页查询

![image-20210725231452092](平台管理-需求说明.assets\image-20210725231452092.png)

##### 4.1.1)接口分析

|          | **说明**           |
| -------- | ------------------ |
| 接口路径 | /api/v1/auth/list  |
| 请求方式 | POST               |
| 参数     | AuthDto            |
| 响应结果 | PageResponseResult |

参数

```json
{
	"page": 0,             // 页码
	"size": 0,             // 每页显示条数
	"status": 0            // 状态
}
```

##### 4.1.2)代码实现

①：新建 AuthDto

```java
package com.heima.model.user.dtos;

import com.heima.model.common.dtos.PageRequestDto;
import lombok.Data;

@Data
public class AuthDto  extends PageRequestDto {

    /**
     * 状态
     */
    private Short status;

    private Integer id;

    //驳回的信息
    private String msg;
}
```

②：新建 ApUserRealnameService 并添加方法

```java
package com.heima.user.service;

import com.heima.model.common.dtos.PageResponseResult;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.user.dtos.AuthDto;

public interface ApUserRealNameService {

    /**
     * 查询认证列表
     *
     * @param dto dto
     * @return PageResponseResult
     */
    PageResponseResult list(AuthDto dto);

}
```

③：新建 ApUserRealnameServiceImpl 并添加方法

```java
package com.heima.user.service;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.heima.apis.wemedia.IWmUserClient;
import com.heima.model.common.dtos.PageResponseResult;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.user.dtos.AuthDto;
import com.heima.model.user.pojos.ApUser;
import com.heima.model.user.pojos.ApUserRealname;
import com.heima.user.mapper.ApUserMapper;
import com.heima.user.mapper.ApUserRealNameMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.HashMap;
import java.util.Map;


/**
 * @author itheima
 * @since 2022-07-26
 */
@Service
@Slf4j
public class ApUserRealNameServiceImpl implements ApUserRealNameService {

    @Autowired
    private ApUserRealNameMapper apUserRealNameMapper;

    @Autowired
    private ApUserMapper apUserMapper;

    @Override
    public PageResponseResult list(AuthDto dto) {
        // 1.入参校验
        if (dto == null) {
            return new PageResponseResult(0, 0, 0);
        }

        // 2.分页参数校验
        dto.checkParam();

        Integer page = dto.getPage();
        Integer size = dto.getSize();

        // 3.构建分页条件
        Page<ApUserRealname> pageInfo = new Page<>(page, size);

        // 4.构建查询条件
        LambdaQueryWrapper<ApUserRealname> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(dto.getStatus() != null, ApUserRealname::getStatus, dto.getStatus());

        // 5.执行查询
        Page<ApUserRealname> pageResult = apUserRealNameMapper.selectPage(pageInfo, wrapper);

        // 6.组装查询结果
        Long total = pageResult.getTotal();
        PageResponseResult pageResponseResult = new PageResponseResult(page, size, total.intValue());
        pageResponseResult.setData(pageResult.getRecords());
        return pageResponseResult;
    }
}
```

④：新建 ApUserRealnameController 并添加方法

```java
package com.heima.user.controller.v1;

import com.heima.model.common.dtos.PageResponseResult;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.user.dtos.AuthDto;
import com.heima.user.service.ApUserRealNameService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {

    @Autowired
    private ApUserRealNameService apUserRealNameService;

    @PostMapping("/list")
    public PageResponseResult list(@RequestBody AuthDto dto) {
        return apUserRealNameService.list(dto);
    }
}
```

#### 4.2)审核拒绝

![image-20220726152434351](assets\image-20220726152434351.png)

注意：审核通过和拒绝使用的是同一个方法，审核通过后需要创建自媒体账户。

##### 4.2.1)接口分析

|          | **说明**              |
| -------- | --------------------- |
| 接口路径 | /api/v1/auth/authFail |
| 请求方式 | POST                  |
| 参数     | AuthDto               |
| 响应结果 | ResponseResult        |

参数

```json
{
	"id": 0,                  // 审核记录的id
	"msg": ""                 // 驳回原因
}
```

##### 4.2.2)自媒体账户创建

①：添加创建自媒体账户代码

```java
package com.heima.wemedia.client;

import com.heima.apis.wemedia.IWmUserClient;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.wemedia.service.WmUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/wemedia/user")
public class WmUserClient implements IWmUserClient {

    @Autowired
    private WmUserService wmUserService;

    @PostMapping("/add")
    @Override
    public ResponseResult addUser(@RequestBody Map<String, String> map) {
        String name = map.get("name");
        return wmUserService.addUser(name);
    }
}
```

②：WmUserService中添加方法

```java
/**
  * 添加用户
  *
  * @param name 用户名
  * @return ResponseResult
  */
ResponseResult addUser(String name);
```

③：WmUserServiceImpl中添加方法

```java
@Override
public ResponseResult addUser(String name) {
    // 1. 检测入参
    if (StringUtils.isBlank(name)) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }

    // 2. 根据用户名去重
    LambdaQueryWrapper<WmUser> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(WmUser::getName, name);
    Integer userCount = wmUserMapper.selectCount(wrapper);
    if (userCount > 0) {
        return ResponseResult.errorResult(AppHttpCodeEnum.DATA_EXIST, "用户名已存在");
    }

    // 3. 创建用户
    WmUser wmUser = new WmUser();
    wmUser.setName(name);
    wmUser.setCreatedTime(new Date());
    int insertResult = wmUserMapper.insert(wmUser);

    // 4. 根据创建结果响应
    return insertResult == 1 ? ResponseResult.okResult(AppHttpCodeEnum.SUCCESS) : ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR);
}
```

④：heima-leadnews-feign-api模块中添加接口

```java
package com.heima.apis.wemedia;

import com.heima.model.common.dtos.ResponseResult;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.Map;

/**
 * @author itheima
 * @since 2022-07-17
 */
@FeignClient("leadnews-wemedia")
public interface IWmUserClient {

    @PostMapping("/wemedia/user/add")
    ResponseResult addUser(@RequestBody Map<String, String> param);

}
```

##### 4.2.3)审核接口实现

heima-leadnew-user服务中心实现以下逻辑。

①：在 ApUserRealnameService 中添加方法

```java
/**
  * 通过/不通过
  *
  * @param dto  审核信息
  * @param type 1通过 2不通过
  * @return
  */
ResponseResult auth(AuthDto dto, Integer type);
```

②：在 ApUserRealnameServiceImpl 中添加方法

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public ResponseResult auth(AuthDto dto, Integer type) {
    // 1.入参检测
    if (dto == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }

    // 2.提取id
    Integer id = dto.getId();

  	// 3.根据id查询认证记录
    ApUserRealname apUserRealname = apUserRealNameMapper.selectById(id);
    if (apUserRealname == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.DATA_NOT_EXIST);
    }

    // 4.更新认证审核状态
    apUserRealname.setStatus((short) (type == 1 ? 9 : 2));
    apUserRealname.setId(id);
    apUserRealname.setReason(dto.getMsg());
    apUserRealNameMapper.updateById(apUserRealname);

    // 5.通过
    if (type == 1) {
        // 5.1 通过认证记录获得对应的用户名
        String name = apUserRealname.getName();

        // 5.2 调用Feign接口完成媒体账号创建
        Map<String, String> map = new HashMap<>();
        map.put("name", name);
        ResponseResult responseResult = iWmUserClient.addUser(map);
        if (responseResult == null || responseResult.getCode() != 200) {
            throw new RuntimeException("自媒体用户创建失败");
        }

        // 5.3 更新APP用户的flag状态
        ApUser apUser = new ApUser();
        apUser.setId(apUserRealname.getUserId());
        apUser.setFlag((short) 1);
        int updateResult = apUserMapper.updateById(apUser);
        if (updateResult < 1) {
            throw new RuntimeException("APP用户状态更新失败");
        }
    }

    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
}
```

③：ApUserRealnameController 添加方法

```java
@PostMapping("/authFail")
public ResponseResult authFail(@RequestBody AuthDto dto) {
    return apUserRealNameService.auth(dto, 2);
}
```

#### 4.3)审核通过

##### 4.3.1)接口分析

|          | **说明**              |
| -------- | --------------------- |
| 接口路径 | /api/v1/auth/authPass |
| 请求方式 | POST                  |
| 参数     | AuthDto               |
| 响应结果 | ResponseResult        |

参数

```json
{
	"id": 0
}
```

##### 4.3.2)代码实现

①：ApUserRealnameController 添加方法

```java
@PostMapping("/authPass")
public ResponseResult authPass(@RequestBody AuthDto dto) {
    return apUserRealNameService.auth(dto, 1);
}
```

### 5)文章人工审核

自媒体文章如果没有自动审核成功，而是到了人工审核（自媒体文章状态为3），需要在admin端人工处理文章的审核

- 平台管理员可以查看待人工审核的文章信息，如果存在违规内容则驳回（状态改为2，文章审核失败）
- 平台管理员可以查看待人工审核的文章信息，如果不存在违规，则需要创建app端的文章信息，并更新自媒体文章的状态

也可以通过点击**查看**按钮，查看文章详细信息，查看详情后可以根据内容判断是否需要通过审核

**注意：**

- 该模块接口在wemedia服务中开发

- 确保网关配置文件 leadnews-admin-gateway.yml 中有以下配置

  ```json
  - id: article
            uri: lb://leadnews-article
            predicates:
              - Path=/article/**
            filters:
              - StripPrefix= 1
  ```

  

#### 5.1)文章列表查询

- 分页查询自媒体文章
- 可以按照标题模糊查询
- 可以按照审核状态进行精确检索
- 文章查询按照创建时间倒序查询
- 注意：需要展示作者名称

![image-20210725233328453](平台管理-需求说明.assets\image-20210725233328453.png)

##### 5.1.1)接口分析

|          | **说明**             |
| -------- | -------------------- |
| 接口路径 | /api/v1/news/list_vo |
| 请求方式 | POST                 |
| 参数     | NewsAuthDto          |
| 响应结果 | ResponseResult       |

参数

```json
{
	"id": 0,
	"msg": "",
	"page": 0,
	"size": 0,
	"status": 0,
	"title": ""
}
```

响应结果

```json
{
    "host":"",
    "code":200,
    "errorMessage":null,
    "data":[
        {
            "id":"6225",
            "userId":1102,
            "title":"“真”项目课程对找工作有什么帮助？",
            "content":"[{\"type\":\"text\",\"value\":\"找工作，企业重点问的是项目经验，更是HR筛选的“第一门槛”，直接决定了你是否有机会进入面试环节。\\n\\n　　项目经验更是评定“个人能力/技能”真实性的“证据”，反映了求职者某个方面的实际动手能力、对某个领域或某种技能的掌握程度。\"},{\"type\":\"image\",\"value\":\"http://192.168.200.130:9000/leadnews/2021/4/20210418/7d0911a41a3745efa8509a87f234813c.jpg\"},{\"type\":\"text\",\"value\":\"很多经过培训期望快速上岗的程序员，靠着培训机构“辅导”顺利经过面试官对于“项目经验”的考核上岗后，在面对“有限时间”“复杂业务”“新项目需求”等多项标签加持的工作任务，却往往不知从何下手或开发进度极其缓慢。最终结果就是：熬不过试用期。\\n\\n　　从而也引发了企业对于“培训出身程序员”的“有色眼光”。你甚至也一度怀疑“IT培训班出来的人真的不行吗?”\"}]",
            "type":1,
            "channelId":1,
            "labels":"项目课程",
            "createdTime":1618762090000,
            "submitedTime":1618762090000,
            "status":9,
            "publishTime":1618762085000,
            "reason":"审核成功",
            "articleId":1383828014629179393,
            "images":"http://192.168.200.130:9000/leadnews/2021/04/26/ef3cbe458db249f7bd6fb4339e593e55.jpg",
            "enable":1,
            "authorName":"admin"
        },
        {
            "id":"6226",
            "userId":1102,
            "title":"学IT，为什么要学项目课程？",
            "content":"[{\"type\":\"text\",\"value\":\"在选择IT培训机构时，你应该有注意到，很多机构都将“项目课程”作为培训中的重点。那么，为什么要学习项目课程?为什么项目课程才是IT培训课程的核心?\\n\\n　　1\\n\\n　　在这个靠“技术经验说话”的IT行业里，假如你是一个计算机或IT相关专业毕业生，在没有实际项目开发经验的情况下，“找到第一份全职工作”可能是你职业生涯中遇到的最大挑战。\\n\\n　　为什么说找第一份工作很难?\\n\\n　　主要在于：实际企业中用到的软件开发知识和在学校所学的知识是完全不同的。假设你已经在学校和同学做过周期长达2-3个月的项目，但真正工作中的团队协作与你在学校中经历的协作也有很多不同。\"},{\"type\":\"image\",\"value\":\"http://192.168.200.130:9000/leadnews/2021/4/20210418/e8113ad756a64ea6808f91130a6cd934.jpg\"},{\"type\":\"text\",\"value\":\"在实际团队中，每一位成员彼此团结一致，为项目的交付而努力，这也意味着你必须要理解好在项目中负责的那部分任务，在规定时间交付还需确保你负责的功能，在所有环境中都能很好地发挥作用，而不仅仅是你的本地机器。\\n\\n　　这需要你对项目中的每一行代码严谨要求。学校练习的项目中，对bug的容忍度很大，而在实际工作中是绝对不能容忍的。项目中的任何一个环节都涉及公司利益，任何一个bug都可能影响公司的收入及形象。\"},{\"type\":\"image\",\"value\":\"http://192.168.200.130:9000/leadnews/2021/4/20210418/c7c3d36d25504cf6aecdcd5710261773.jpg\"}]",
            "type":3,
            "channelId":1,
            "labels":"项目课程",
            "createdTime":1618762438000,
            "submitedTime":1618762438000,
            "status":1,
            "publishTime":1618762248000,
            "reason":"审核成功",
            "articleId":1383827995813531650,
            "images":"http://192.168.200.130:9000/leadnews/2021/04/26/ec893175f18c4261af14df14b83cb25f.jpg",
            "enable":1,
            "authorName":"admin"
        }
    ],
    "currentPage":1,
    "size":10,
    "total":455
}
```

##### 5.1.2)代码实现

核心SQL语句

```sql
SELECT wm_news.*, wm_user.name authorName
FROM wm_news
LEFT JOIN wm_user ON wm_news.user_id = wm_user.id
WHERE
	wm_news.title LIKE CONCAT('%标题%')
	AND wm_news.status = 9
ORDER BY wm_news.publish_time DESC
LIMIT 0, 10
```

①：NewsAuthDto

```java
package com.heima.model.wemedia.dtos;

import com.heima.model.common.dtos.PageRequestDto;
import lombok.Data;

@Data
public class NewsAuthDto extends PageRequestDto {

    /**
     * 文章标题
     */
    private String title;
    /**
     * 状态
     */
    private Short status;
    /**
     * 自媒体文章id
     */
    private Integer id;
    /**
     * 审核失败的原因
     */
    private String msg;
}
```

②：WmNewsService

```java
/**
  * 查询文章列表
  * @param dto
  * @return
  */
ResponseResult findList(NewsAuthDto dto);
```

③：WmNewsServiceImpl

```java
@Override
public ResponseResult findList(NewsAuthDto dto) {
    // 1.参数检查
    dto.checkParam();

    // 2.记录当前页
    int currentPage = dto.getPage();

    // 3.分页查询+count查询
    dto.setPage((dto.getPage() - 1) * dto.getSize());
    List<WmNewsVo> wmNewsVoList = wmNewsMapper.findListAndPage(dto);
    int count = wmNewsMapper.findListCount(dto);

    // 4.结果返回
    ResponseResult responseResult = new PageResponseResult(currentPage, dto.getSize(), count);
    responseResult.setData(wmNewsVoList);
    return responseResult;
}
```

④：WmNewsMapper.java中添加方法

```java
package com.heima.wemedia.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.heima.model.wemedia.dtos.NewsAuthDto;
import com.heima.model.wemedia.pojos.WmNews;
import com.heima.model.wemedia.vo.WmNewsVo;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

@Mapper
@Repository
public interface WmNewsMapper extends BaseMapper<WmNews> {
    /**
      * 查询列表
      * @param dto dto
      * @return List
      */
    List<WmNewsVo> findListAndPage(@Param("dto") NewsAuthDto dto);

    /**
      * 查询记录数
      * @param dto dto
      * @return Integer
      */
    Integer findListCount(@Param("dto") NewsAuthDto dto);
}
```

⑤：新增WmNewsMapper.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.heima.wemedia.mapper.WmNewsMapper">

    <select id="findListAndPage" resultType="com.heima.model.wemedia.vo.WmNewsVo">
      SELECT wm_news.*, wm_user.name authorName
      FROM wm_news
      LEFT JOIN wm_user ON wm_news.user_id = wm_user.id
      <where>
          wm_news.title LIKE CONCAT('%', #{dto.title} ,'%')
          <if test="dto.status != null">
              AND wm_news.status = #{dto.status}
          </if>
      </where>
      ORDER BY wm_news.publish_time DESC
      LIMIT #{dto.page}, #{dto.size}
    </select>

    <select id="findListCount" resultType="java.lang.Integer">
        SELECT COUNT(*) FROM wm_news
        <where>
            wm_news.title LIKE CONCAT('%', #{dto.title} ,'%')
            <if test="dto.status != null">
                AND wm_news.status = #{dto.status}
            </if>
        </where>
    </select>

</mapper>
```

⑥：WmNewsController中添加方法

```java
@PostMapping("/list_vo")
public ResponseResult findList(@RequestBody NewsAuthDto dto){
    return wmNewsService.findList(dto);
}
```

#### 5.2)查询文章详情

![image-20210725233521821](平台管理-需求说明.assets\image-20210725233521821.png)

- 可以查看文章详细内容
- 注意：需要展示作者名称

##### 5.2.1)接口分析

|          | **说明**             |
| -------- | -------------------- |
| 接口路径 | /api/v1/news/list_vo |
| 请求方式 | GET                  |
| 参数     | 无                   |
| 响应结果 | ResponseResult       |

##### 5.2.2)代码实现

- 入参判空
- wmNews表根据id查询 -> wmNews完整信息
- wmNews.getUserId() -> wmUser.selectById() -> wmUser信息
- 创建一个返回值结构，把wmNews + 用户名称合并起来

①：WmNewsService

```java
/**
     * 查询文章详情
     * @param id
     * @return
     */
ResponseResult findWmNewsVo(Integer id);
```

②：WmNewsServiceImpl中添加方法

```java
@Override
public ResponseResult findWmNewsVo(Integer id) {
    // 1.检查参数
    if(id == null){
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
    }
    // 2.查询文章信息
    WmNews wmNews = getById(id);
    if(wmNews == null){
        return ResponseResult.errorResult(AppHttpCodeEnum.DATA_NOT_EXIST);
    }

    // 3.查询用户信息
    WmUser wmUser = wmUserMapper.selectById(wmNews.getUserId());

    // 4.封装vo返回
    WmNewsVo vo = new WmNewsVo();
    // 5.属性拷贝
    BeanUtils.copyProperties(wmNews,vo);
    if(wmUser != null){
        vo.setAuthorName(wmUser.getName());
    }

    return return ResponseResult.okResult(vo);
}
```

③：WmNewsController中添加方法

```java
@GetMapping("/one_vo/{id}")
public ResponseResult findWmNewsVo(@PathVariable("id") Integer id){
    return wmNewsService.findWmNewsVo(id);
}
```

#### 5.3)新闻人工审核 - 拒绝

![image-20210726013458078](平台管理-需求说明.assets\image-20210726013458078.png)

拒绝以后需要给出原因，并修改文章的状态为2

##### 5.3.1)接口分析

|          | **说明**               |
| -------- | ---------------------- |
| 接口路径 | /api/v1/news/auth_fail |
| 请求方式 | POST                   |
| 参数     | NewsAuthDto            |
| 响应结果 | ResponseResult         |

参数

```json
{
	"id": 0,              // 新闻id
	"msg": ""             // 拒绝原因
}
```

##### 5.3.2)代码实现

![image-20220726152805116](assets\image-20220726152805116.png)

①：WmNewsService中添加方法

```java
/**
  * 文章审核，修改状态
  * @param status  2  审核失败  4 审核成功
  * @param dto
  * @return
  */
ResponseResult passOrFail(NewsAuthDto dto, Integer type);
```

②：WmNewsServiceImpl中添加方法

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public ResponseResult passOrFail(NewsAuthDto dto, Integer type) {
    if (dto == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }

    Integer id = dto.getId();
    String msg = dto.getMsg();

    WmNews wmNews = wmNewsMapper.selectById(id);
    if (wmNews == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.DATA_NOT_EXIST, "新闻不存在");
    }

    int status = type == 1 ? 9 : 2;
    wmNews.setStatus((short) status);
    wmNews.setReason(type == 1 ? "通过" : msg);
    int updateResult = wmNewsMapper.updateById(wmNews);
    if (updateResult < 1) {
        return ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR, "新闻审核失败");
    }

    if (type == 1) {
        ArticleDto articleDto = autoScanService.wmNews2ArticleDto(wmNews);
        ResponseResult responseResult = iArticleClient.saveArticle(articleDto);
        if (responseResult == null || responseResult.getCode() != 200) {
            throw new RuntimeException("文章保存失败");
        }
    }

    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
}
```

③：WmNewsController中添加方法

```java
@PostMapping("/auth_fail")
public ResponseResult authFail(@RequestBody NewsAuthDto newsAuthDto) {
    return wmNewsService.passOrFail(newsAuthDto, 2);
}
```

#### 5.4)新闻人工审核 - 通过

![image-20210726013527654](平台管理-需求说明.assets\image-20210726013527654.png)

需要创建app端的文章信息，并更新自媒体文章的状态

##### 5.4.1)接口分析

|          | **说明**               |
| -------- | ---------------------- |
| 接口路径 | /api/v1/news/auth_pass |
| 请求方式 | POST                   |
| 参数     | NewsAuthDto            |
| 响应结果 | ResponseResult         |

参数

```json
{
	"id": 0,
	"msg": ""
}
```

##### 5.4.2)代码实现

①：WmNewsController中添加方法

```java
@PostMapping("/auth_pass")
public ResponseResult authPass(@RequestBody NewsAuthDto newsAuthDto) {
    return wmNewsService.passOrFail(newsAuthDto, 1);
}
```