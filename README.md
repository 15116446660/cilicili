# cilicili | 简单视频分享

## 1. 介绍

cilicili 是一个支持用户上传视频、观看视频、搜索视频、发送评论、发送弹幕的视频分享网站。

## 2. 架构

项目将前后端分离，并把功能拆分成 4 个模块，支持分布式部署。

### 2.1 后台 API - apiServer

提供除了上传的所有服务的支持，功能包括用户注册、登录、修改密码，视频信息上传，视频评论，视频弹幕。

### 2.2 简单文件上传服务 - simpleUploader

提供基于用户身份验证的文件上传功能。

### 2.3 前端页面 - web_server

全部的前端页面。

### 2.4 文件存储获取服务 - storage_server

用于获取上传的文件。

## 3. 环境

### 3.1 后台

MySQL: 5.7.18-log - Source distribution

JDK: OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)

Maven: Apache Maven 3.0.5 (Red Hat 3.0.5-17)

Node: v6.14.0

OS: centos-release-7-4.1708.el7.centos.x86_64

### 3.2 前端

Vue.js: v2.5.13

jQuery: v3.2.1

Bootstrap: v3.3.7

Bootstrap-select: v1.12.4

ECMAScript: ECMAScript 6

Browser: Chrome, 66.0.3359.139（正式版本） （64 位）

OS: Windows 10, 1803 (内部版本 17134.1)

## 4. 安装 & 运行

### 4.0 注意事项

1. 所有服务均可分别在不同的服务器上部署，除了

    **simpleUploader 必须和 storage_server 放在同一台服务器上**

2. storage_server 和 web_server 也可使用 Nginx 部署

### 4.1 数据库

在 MySQL 中新建 cilicili 数据库，字符集为 utf8mb4_general_ci ，并导入 cilicili.sql

### 4.2 apiServer

修改 apiServer/src/main/resources/_db.properties 中的信息，并将文件名改为 db.properties

```
db.driverClassName=com.mysql.cj.jdbc.Driver
db.fullURL=jdbc:mysql://{数据库服务器ip}:{数据库端口号}/{数据库名}?characterEncoding=utf8&useSSL=false&allowMultiQueries=true&serverTimezone=Asia/Shanghai
db.username={数据库用户名}
db.password={数据库密码}
```

修改 apiServer/pom.xml 129 行处的端口号
```
<plugin>
  <groupId>org.eclipse.jetty</groupId>
  <artifactId>jetty-maven-plugin</artifactId>
  <version>9.4.8.v20171121</version>
  <configuration>
    <httpConnector>
      <port>{端口号}</port>
    </httpConnector>
  </configuration>
</plugin>
```

在 apiServer/ 下执行:
```
rm -rf target
mvn install
mvn jetty:run
```

### 4.3 simpleUploader

修改 simpleUploader/src/main/resources/_db.properties 中的信息，并将文件名改为 db.properties

```
db.driverClassName=com.mysql.cj.jdbc.Driver
db.fullURL=jdbc:mysql://{数据库url}:{数据库端口号}/{数据库名}?characterEncoding=utf8&useSSL=false&allowMultiQueries=true&serverTimezone=Asia/Shanghai
db.username={数据库用户名}
db.password={数据库密码}
```

修改 simpleUploader/pom.xml 129 行处的端口号
```
<plugin>
  <groupId>org.eclipse.jetty</groupId>
  <artifactId>jetty-maven-plugin</artifactId>
  <version>9.4.8.v20171121</version>
  <configuration>
    <httpConnector>
      <port>{端口号}</port>
    </httpConnector>
  </configuration>
</plugin>
```

修改 simpleUploader/src/main/java/cilicili/jz2/utils/BaseUtil.java 里的信息
```
package cilicili.jz2.utils;

public class BaseUtil {
	public static final String STORAGE_DIR = "{storage_server/storage 的绝对路径}";
	public static final long MAX_FILE_SIZE = {最大文件大小，单位：字节}L;
}
```

修改 simpleUploader/src/main/resources/springmvcContext.xml 22 行处的最大文件大小信息，与 BaseUtil.java 文件里保持一致
```
  <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!--最大文件体积 单位字节 50M-->
    <property name="maxUploadSize" value="{最大文件大小，单位：字节}"/>
    <property name="defaultEncoding" value="UTF-8"/>
  </bean>
```

在 simpleUploader/ 下执行:
```
rm -rf target
mvn install
mvn jetty:run
```

### 4.4 storage_server

修改 storage_server/app.js 第 8 行处的端口号
```
const server = app.listen({端口号});
```

在 storage_server/ 下执行:
```
npm install
npm run app
```

### 4.5 web_server

修改 web_server/webroot/js/utils/ajaxBase.js 里的信息
```
......
const API_SERVER_HOSTNAME = '{apiServer 部署的服务器 ip}';
const API_SERVER_PORT = {apiServer 的端口号};
......
const STORAGE_SERVER_HOSTNAME = '{storage_server 部署的服务器 ip}';
const STORAGE_SERVER_PORT = {storage_server 的端口号};
......
const UPLOAD_SERVER_HOSTNAME = '{simpleUploader 部署的服务器 ip}';
const UPLOAD_SERVER_PORT = {simpleUploader 的端口号};
......
```

修改 web_server/app.js 第 8 行处的端口号
```
const server = app.listen({端口号});
```

在 web_server/ 下执行:
```
npm install
npm run app
```

### 4.6 访问

浏览器打开 
```
http://{web_server 部署的服务器 ip}:{web_server 的端口号}/
```

## 5. 基于 Token 的用户鉴权功能

### 5.1 介绍

将用户的登录、授权信息用数据库记录，使得服务支持分布式部署及实现单点登录功能。

### 5.2 Token 原型

```
private Integer id; // 唯一自增 id
private Integer userId; // 权限所属用户 id
private String token; // token 字符串，会保存在用户的浏览器 cookie 中
private ZonedDateTime applytime; // 申请时间
private ZonedDateTime expiretime; // 失效时间
private Integer countAuth; // 当前鉴权次数
private Integer maxCountAuth; // 最大允许鉴权次数
private String ussage; // 用途
```

### 5.3 TokenUtil 原型

```
public static final Integer DEFAULT_MAX_COUNT_AUTH = 1000; // 默认用途的最大允许鉴权次数
public static class TokenUssage { // token 用途
  public static final String DEFAULT = "default"; // 默认用途，用于登录
  public static final String UPLOAD_FILE = "upload_file"; // 上传文件
  public static final String MODIFY_USER_SETTINGS = "modify_user_settings"; // 修改用户信息
  public static final String UPDATE_VIDEO_INFO = "update_video_info"; // 修改视频信息
}

public static class TokenExpired extends Error {...} // token 已失效
public static class TokenNotFound extends Error {...} // 无效 token
public static class TokenOverAuthed extends Error {...} // token 超过最大允许鉴权次数
public static class TokenUssageNotMatched extends Error {...} // token 用途不匹配

/**
  用    途:  检查 token 是否合法
  传入参数:  token 字符串，用途（必须是 TokenUtil.TokenUssage.* 中的给定值）
  返 回 值:  Token
  运行过程：
            1. 去数据库中查找该 token，如果找不到则抛出 TokenNotFound
            2. 检查是否超过失效时间，如果是则抛出 TokenExpired
            3. 检查是否超过最大允许鉴权次数，如果是则抛出 TokenOverAuthed
            4. 检查用途是否匹配，如果不则抛出 TokenUssageNotMatched
            5. 以上均为不允许授权的情况，如果都通过则将该 Token.countAuth 加 1 后写回数据库，并将完整的 Token （含有用户 id） 返回
  调用参考：
            1. apiServer/src/main/java/cilicili/jz2/controller/impl/LoginControllerImpl.java : 84
               Token tokenCheck = TokenUtil.checkToken(token, TokenUtil.TokenUssage.DEFAULT); // 检查用户是否登录，获得该 token 的所属用户 id

            2. simpleUploader/src/main/java/cilicili/jz2/controller/impl/UploadControllerImpl.java : 29
               Token tokenCheck = TokenUtil.checkToken(token, TokenUtil.TokenUssage.UPLOAD_FILE); // 检查用于上传文件的 token 是否合法
*/
public static Token checkToken(String tokenString, String ussage) throws ... {...}

/**
  用    途:  创建新 token
  传入参数:  用户 id（在调用前保证 id 正确）, 用途（必须是 TokenUtil.TokenUssage.* 中的给定值），最大允许鉴权次数，有效持续时间
  返 回 值:  Token
  运行过程：
            1. 设置用户 id、用途、申请时间、最大允许鉴权次数、当前已允许鉴权次数信息
            2. 计算失效时间并设置
            3. 用以用户 id、系统时间、盐、默认串为种子的随机字符串，前后添加时间戳，设置为 token 并设置
            4. 写入数据库
            5. 若以上操作均无误，将该完整的 Token 返回
  调用参考：
            1. apiServer/src/main/java/cilicili/jz2/controller/impl/LoginControllerImpl.java : 41
               Token newToken = TokenUtil.createToken(user.getId(), TokenUtil.TokenUssage.DEFAULT, TokenUtil.DEFAULT_MAX_COUNT_AUTH, Period.of(0, 1, 0)); // 用户登陆，使用默认最大允许鉴权次数，有效期为 1 个月

            2. apiServer/src/main/java/cilicili/jz2/controller/impl/LoginControllerImpl.java : 91
               Token newToken = TokenUtil.createToken(user.getId(), ussage, 1, Period.of(0, 0, 1)); // 用于用户申请权限
*/
public static Token createToken(Integer userId, String tokenUssage, Integer maxCountAuth, Period validPeriod) {...}

/**
  用    途:  销毁一条 token
  传入参数:  token 字符串
  返 回 值:  无
  运行过程：
            1. 去数据库中查找该 token，如果找不到则抛出 TokenNotFound
            2. 将失效日期设为现在
            3. 将更新后的 token 写回数据库
  调用参考：
            1. apiServer/src/main/java/cilicili/jz2/controller/impl/LoginControllerImpl.java : 52
               TokenUtil.destoryToken(token); // 用户注销登录

            2. apiServer/src/main/java/cilicili/jz2/controller/impl/UserControllerImpl.java : 119
               TokenUtil.destoryToken(token); // 清除更新信息前的用户登陆

            3. apiServer/src/main/java/cilicili/jz2/controller/impl/UserControllerImpl.java : 120
               TokenUtil.destoryToken(apply); // 销毁用于用户修改信息的 token
*/
public static void destoryToken(String tokenString) throws TokenNotFound {...}

/**
  用    途:  销毁某用户的所有历史 tokens
  传入参数:  用户 id（在调用前保证 id 正确）
  返 回 值:  无
  运行过程：
            1. 删除数据库中 user_id 为给定值的所有记录
  调用参考：
            1. apiServer/src/main/java/cilicili/jz2/controller/impl/LoginControllerImpl.java : 40
               TokenUtil.destoryOldTokens(user.getId()); // 单点登录
*/
public static void destoryOldTokens(Integer userId) {...}
```


## 6. 后续开发

1. 功能

    1.1 用户角色权限功能，例如高级会员

    1.2 视频信息的修改、删除功能

    1.3 主页 Banner 的管理
    
    1.4 网站管理，包括全部的用户信息、视频信息、评论、弹幕管理

    1.5 基于视频播放量的热度榜

2. 架构

    2.1 前台异步处理请求，以防止因为等待处理请求造成的页面卡住

    2.2 接入 OAuth 登录授权

    2.3 数据库拆分，数据库分布式部署