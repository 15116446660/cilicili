# api 接口文档

## 0. 通用说明

0.1 所有返回值均为 JSON 格式

0.2 接口信息说明：

{序号} {接口名}

*{接口地址}*

{接口用途}

```
POST/GET => { // 请求类型
    // 请求需要携带的参数
}
------------------------------------------------------
=> success { // 被认定为成功的返回值
    // 返回值包含的参数及其说明
}
=> failure { // 被认定为失败的返回值
    // 返回值包含的参数及其说明
}
```

0.3 通用失败返回值
```
=> failure {
    "status": "failure"
    , "msg": "参数错误"

    // 请检查请求需要携带的参数是否合法或有遗漏
}
```

0.4 Token 鉴权失败返回值
```
=> failure {
    "msg": {错误信息}

    // 所有可能的错误说明：

    // "找不到该 token： {请求中携带的用户身份令牌} ，无法授权"
    //          1. 使用了伪造的用户身份令牌
    //          2. 用户在别处登录，旧的用户身份令牌全部失效不存在

    // "该 token： {请求中携带的用户身份令牌} 已失效，无法授权"
    //          该用户身份令牌已经超过时效，需要重新登录

    // "该 token： {请求中携带的用户身份令牌} 已超过最大授权次数，无法授权"
    //          该用户身份令牌已经超过其最大允许鉴权次数，需要重新登录

    // "该 token： {请求中携带的用户身份令牌} 无法用于 ussage: {用途}，无法授权"
    //          使用了错误的令牌，如将用户身份令牌用于上传文件等
}
```

## 1. 登录、权限相关

1.1 登录

*{API_SERVER}/login*

根据用户名和密码得到用户身份令牌
```
POST => {
    "username": "{用户名}"
    , "password": "{密码}"
}
------------------------------------------------------
=> success {
    "status": "success"
    , "token": "{用户身份令牌}"

    // 一旦登录成功，则之前所有令牌均失效
    // 默认用户令牌可授权 1000 次，有效期 1 个月
}
=> failure {
    "status: "failure"
    , "msg": "用户名或密码错误"
}
```

1.2 注销

*{API_SERVER}/logout*

使某个用户身份令牌失效
```
POST => {
    "token": "{用户身份令牌}"
}
------------------------------------------------------
=> success {
    "status": "success"

    // 一旦注销成功，则该令牌失效
}
```

1.3 验证登录状态

*{API_SERVER}/now*

检查用户身份令牌是否合法有效
```
POST => {
    "token": "{用户身份令牌}"
}
------------------------------------------------------
=> success {
    "status": true // 表示该用户身份令牌合法有效
    , "user": {
        "id": {用户 id}
        ,"username": "{用户名}"
        ,"identity": "{用户身份}"
    }
}
=> failure {
    "status": false // 表示该用户身份令牌非法或已失效
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "用户不存在":
    //          用户被管理员删除，该用户不能再登录

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```

1.4 申请权限

*{API_SERVER}/apply*

申请用户权限令牌
```
POST => {
    "token": "{用户身份令牌}"
    , "ussage": "{权限}"

    // 所有允许的权限：
    // "upload_file": 上传文件，如视频、视频封面图等
    // "modify_user_settings": 修改用户信息，如用户名、密码等
    // "update_video_info": 修改视频信息，如视频标题、视频描述、视频封面图等

    // 以上所有权限默认有 1 天有效期，最多 1 次鉴权操作
}
------------------------------------------------------
=> success {
    "status": "success"
    , "token": "{用户权限令牌}"
}
=> failure {
    "status": "failure" // 表示该用户身份令牌非法或已失效
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "用户不存在":
    //          用户被管理员删除，该用户不能再请求权限

    // "非法获取权限":
    //          请求的权限不在所有允许的权限列表之中

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```

## 2. 用户信息相关

2.1 根据用户 id 查找某个用户

*{API_SERVER}/user/findId*

根据用户 id 得到某个用户的信息
```
GET => {
    "id": "{被查找的用户 id}"
    , "token": "{用户身份令牌}"
}
------------------------------------------------------
=> success {
    "status": "success"
    , "user": {
        "id": {被查找的用户 id}
        , "username": "{被查找的用户名}"
        , "identity": "{被查找的用户身份}"
    }
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "找不到该用户":
    //          找不到与请求的被查找的用户 id 一致的用户
}
```

2.2 根据用户名查找某个用户

*{API_SERVER}/user/findUsername*

根据用户 username 得到某个用户的信息
```
GET => {
    "username": "{被查找的用户名}"
    , "token": "{用户身份令牌}"
}
------------------------------------------------------
=> success {
    "status": "success"
    , "user": {
        "id": {被查找的用户 id}
        , "username": "{被查找的用户名}"
        , "identity": "{被查找的用户身份}"
    }
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "找不到该用户":
    //          找不到与请求的被查找的用户 id 一致的用户
}
```

2.3 添加用户信息（注册）

*{API_SERVER}/user/add*

注册新用户
```
POST => {
    "username": "{用户名}"
    , "password": "{密码}"
}
------------------------------------------------------
=> success {
    "status": "success"
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "用户名为空"
    // "用户名为空或超过20长度限制"
    // "密码为空"
    // "密码为空或超过20长度限制"
    // "{请求中携带的用户名} 已被注册"
}
```

2.4 修改用户信息

*{API_SERVER}/user/update*

修改用户名或密码
```
POST => {
    "username": "{新用户名}"
    , "password": "{新密码}"
    , "token": "{用户身份令牌}"
    , "apply": "{用户权限令牌}"

    // 在请求该接口前需先申请用途为 "modify_user_settings" 的用户权限令牌，参考 1.4
}
------------------------------------------------------
=> success {
    "status": "success"
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "用户名为空"
    // "用户名为空或超过20长度限制"
    // "密码为空"
    // "密码为空或超过20长度限制"
    // "{请求中携带的用户名} 已被注册"

    // "非本人操作，拒绝授权":
    //          请求中携带的用户身份令牌与请求中携带的用户权限令牌所属用户不同

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```

## 3. 视频信息相关

3.1 根据视频 id 查找某个视频

*{API_SERVER}/video/findId*

根据视频 id 得到某个视频的信息
```
GET => {
    "id": "{被查找的视频 id}"
}
------------------------------------------------------
=> success {
    "status": "success"
    , "video": {
        "uploadUsername": "{视频上传者用户名}"
        , "id": {视频 id}
        , "title": "{视频标题}"
        , "url": "{视频文件名，不包含存储服务器}"
        , "uploadTime": { // 上传时间
            "dateTime": { // 日期
                "date": {
                    "year": {年}
                    , "month": {月}
                    , "day": {日}
                }, "time": {
                    "hour": {时}
                    , "minute": {分}
                    , "second": {秒}
                    , "nano": {毫秒}
                }
            }, "offset": {
                "totalSeconds": {与 UTC 相隔的时差，单位：秒}
            }, "zone": {
                "id": "{时区}"
            }
        }, "countPlay": {播放量}
        , "countLike": {点赞量}
        , "picUrl": "{封面图文件名，不包含存储服务器地址}"
        , "description": "{视频描述}"
    }
}
=> failure {
    "status: "failure"
    , "msg": "找不到该视频"
}
```

3.2 添加视频信息（上传新视频）

*{API_SERVER}/video/add*

上传新视频
```
POST => {
    "token": "{用户身份令牌}"
    , "title": "{视频标题}"
    , "url": "{视频地址}"
    , "picUrl": "{封面图地址}"
    , "description": "{视频描述}"

    // 在请求该接口前需先将视频和视频封面图上传，参考 6.1
}
------------------------------------------------------
=> success {
    "status": "success"
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "视频标题为空"
    // "视频标题为空或超过50长度限制"
    // "视频地址为空"
    // "视频地址为空或超过100长度限制"
    // "视频封面地址为空"
    // "视频封面地址为空或超过100长度限制"

    // "用户不存在":
    //          用户被管理员删除，该用户不能再上传

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```

3.3 更新播放量

*{API_SERVER}/video/play*

使某视频播放量 + 1
```
POST => {
    "id": "{被播放的视频 id}"
}
------------------------------------------------------
=> success {
    "status": "success"
    , "count_play": {最新播放量}
}
=> failure {
    "status: "failure"
    , "msg": "参数错误"
}
```

3.4 更新点赞量

*{API_SERVER}/video/like*

使某视频点赞量 + 1
```
POST => {
    "id": "{被播放的视频 id}"
}
------------------------------------------------------
=> success {
    "status": "success"
    , "count_like": {最新点赞量}
}
=> failure {
    "status: "failure"
    , "msg": "参数错误"
}
```

3.5 所有视频信息（主页）

*{API_SERVER}/video/show*

获得所有视频的信息，结果以最多 12 个视频为一组分页信息
```
POST => {
    "offset": "{页码}" // 从 1 开始
}
------------------------------------------------------
=> success {
    "page": { // 数据为 PageHelper 原始数据，包含了有关分页的全部信息
        "pageNum": {当前页码号}
        , "pageSize": {默认一页包含的信息数量}
        , "size": {当前返回信息中包含的信息数量}
        , "startRow": {返回值中第一个视频信息对应数据库中开始的序号}
        , "endRow": {返回值中最后一个视频信息对应数据库中结束的序号}
        , "total": {数据库中视频信息个数}
        , "pages": {全部页数}
        , "prePage": {上一页页码} // 若已经是第一页，则为 0
        , "nextPage": {下一页页码} // 若已经是最后一页，则为 0
        , "isFirstPage": {是否是第一页}
        , "isLastPage": {是否是最后一页}
        , "hasPreviousPage": {是否有上一页}
        , "hasNextPage": {是否有下一页}
        , "navigatePages": {导航页码数}
        , "navigatepageNums": [ // 所有导航页号
            {页码号}
            ......
        ]
        , "navigateFirstPage": {导航第 1 页页码号}
        , "navigateLastPage": {导航最后 1 页页码号}
        , "list": [ // 结果数组，个数就是上面的 size
            {
                "uploadUsername": "{视频上传者用户名}"
                , "id": {视频 id}
                , "title": "{视频标题}"
                , "url": "{视频文件名，不包含存储服务器}"
                , "countPlay": {播放量}
                , "countLike": {点赞量}
                , "picUrl": "{封面图文件名，不包含存储服务器地址}"
                , "description": "{视频描述}"
                , "uploadTime": { // 上传时间
                    "dateTime": { // 日期
                        "date": {
                            "year": {年}
                            , "month": {月}
                            , "day": {日}
                        }, "time": {
                            "hour": {时}
                            , "minute": {分}
                            , "second": {秒}
                            , "nano": {毫秒}
                        }
                    }, "offset": {
                        "totalSeconds": {与 UTC 相隔的时差，单位：秒}
                    }, "zone": {
                        "id": "{时区}"
                    }
                }
            }
            ......
        ]
    }
}
```

3.6 所有指定视频信息（搜索）

*{API_SERVER}/video/find*

获得所有指定视频的信息，结果以最多 12 个视频为一组分页信息
```
POST => {
    "q": "{关键词}"
    , "offset": "{页码}" // 从 1 开始
}
------------------------------------------------------
=> success {
    "page": { // 数据为 PageHelper 原始数据，包含了有关分页的全部信息
        "pageNum": {当前页码号}
        , "pageSize": {默认一页包含的信息数量}
        , "size": {当前返回信息中包含的信息数量}
        , "startRow": {返回值中第一个视频信息对应数据库中开始的序号}
        , "endRow": {返回值中最后一个视频信息对应数据库中结束的序号}
        , "total": {数据库中视频信息个数}
        , "pages": {全部页数}
        , "prePage": {上一页页码} // 若已经是第一页，则为 0
        , "nextPage": {下一页页码} // 若已经是最后一页，则为 0
        , "isFirstPage": {是否是第一页}
        , "isLastPage": {是否是最后一页}
        , "hasPreviousPage": {是否有上一页}
        , "hasNextPage": {是否有下一页}
        , "navigatePages": {导航页码数}
        , "navigatepageNums": [ // 所有导航页号
            {页码号}
            ......
        ]
        , "navigateFirstPage": {导航第 1 页页码号}
        , "navigateLastPage": {导航最后 1 页页码号}
        , "list": [ // 结果数组，个数就是上面的 size
            {
                "uploadUsername": "{视频上传者用户名}"
                , "id": {视频 id}
                , "title": "{视频标题}"
                , "url": "{视频文件名，不包含存储服务器}"
                , "countPlay": {播放量}
                , "countLike": {点赞量}
                , "picUrl": "{封面图文件名，不包含存储服务器地址}"
                , "description": "{视频描述}"
                , "uploadTime": { // 上传时间
                    "dateTime": { // 日期
                        "date": {
                            "year": {年}
                            , "month": {月}
                            , "day": {日}
                        }, "time": {
                            "hour": {时}
                            , "minute": {分}
                            , "second": {秒}
                            , "nano": {毫秒}
                        }
                    }, "offset": {
                        "totalSeconds": {与 UTC 相隔的时差，单位：秒}
                    }, "zone": {
                        "id": "{时区}"
                    }
                }
            }
            ......
        ]
    }
}
```

## 4. 视频评论相关

4.1 获取所有评论

*{API_SERVER}/comment/findId*

获得所有某视频的评论，结果以最多 10 个评论为一组分页信息
```
GET => {
    "id": "{视频 id}"
    , "offset": "{页码}" // 从 1 开始
}
------------------------------------------------------
=> success {
    "page": { // 数据为 PageHelper 原始数据，包含了有关分页的全部信息
        "pageNum": {当前页码号}
        , "pageSize": {默认一页包含的信息数量}
        , "size": {当前返回信息中包含的信息数量}
        , "startRow": {返回值中第一个视频信息对应数据库中开始的序号}
        , "endRow": {返回值中最后一个视频信息对应数据库中结束的序号}
        , "total": {数据库中视频信息个数}
        , "pages": {全部页数}
        , "prePage": {上一页页码} // 若已经是第一页，则为 0
        , "nextPage": {下一页页码} // 若已经是最后一页，则为 0
        , "isFirstPage": {是否是第一页}
        , "isLastPage": {是否是最后一页}
        , "hasPreviousPage": {是否有上一页}
        , "hasNextPage": {是否有下一页}
        , "navigatePages": {导航页码数}
        , "navigatepageNums": [ // 所有导航页号
            {页码号}
            ......
        ]
        , "navigateFirstPage": {导航第 1 页页码号}
        , "navigateLastPage": {导航最后 1 页页码号}
        , "list": [ // 结果数组，个数就是上面的 size
            {
                "content": "{评论内容}"
                , "countLike": {点赞量}
                , "id": {评论 id}
                , "sendUsername": "{评论者用户名}"
                , "userId": {评论者用户 id}
                , "videoId": {视频 id}
                , "sendTime": { // 评论时间
                    "dateTime": { // 日期
                        "date": {
                            "year": {年}
                            , "month": {月}
                            , "day": {日}
                        }, "time": {
                            "hour": {时}
                            , "minute": {分}
                            , "second": {秒}
                            , "nano": {毫秒}
                        }
                    }, "offset": {
                        "totalSeconds": {与 UTC 相隔的时差，单位：秒}
                    }, "zone": {
                        "id": "{时区}"
                    }
                }
            }
            ......
        ]
    }
}
```

4.2 添加新评论信息

*{API_SERVER}/comment/add*

创建新评论
```
POST => {
    "token": "{用户身份令牌}"
    , "content": "{评论内容}"
    , "videoId": {视频 id}
}
------------------------------------------------------
=> success {
    "status": "success"
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "视频 id 为空"
    // "视频 id 不正确"
    // "评论内容为空"
    // "评论内容为空或超过250长度限制"

    // "未知错误":
    //          服务器错误，尝试重新提交

    // "用户不存在":
    //          用户被管理员删除，该用户不能再评论

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```

4.3 删除某条评论

*{API_SERVER}/comment/delete*

删除某用户曾经发布过的某评论
```
POST => {
    "token": "{用户身份令牌}"
    , "id": "{评论 id}"
}
------------------------------------------------------
=> success {
    "status": "success"
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "没有该评论":
    //          请求中携带的评论 id 错误

    // "非本人操作，拒绝授权":
    //          被删除的评论不是请求中携带的用户身份令牌所属用户发送的

    // "未知错误":
    //          服务器错误，尝试重新提交

    // "用户不存在":
    //          用户被管理员删除，该用户不能再评论

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```

4.4 更新点赞量

*{API_SERVER}/comment/like*

使某评论点赞量 + 1
```
POST => {
    "id": "{被点赞的评论 id}"
}
------------------------------------------------------
=> success {
    "status": "success"
    , "count_like": {最新点赞量}
}
=> failure {
    "status: "failure"
    , "msg": "参数错误"
```

## 5. 弹幕相关

5.1 获取所有弹幕

*{API_SERVER}/barrage/findId*

获得所有某视频的评论，结果以最多 10 个评论为一组分页信息
```
GET => {
    "id": "{视频 id}"
}
------------------------------------------------------
=> success {
    "barrages": [
        {
            "color": "{弹幕颜色}"
            , "content": "{弹幕内容}"
            , "id": {弹幕 id}
            , "offtime": {弹幕发送偏移时间，单位：秒}
            , "position": {弹幕位置：0: 固定弹幕, 2: 浮动弹幕}
            , "userId": {发送者用户 id}
            , "videoId": {视频 id}
            , "sendTime": { // 发送时间
                "dateTime": { // 日期
                    "date": {
                        "year": {年}
                        , "month": {月}
                        , "day": {日}
                    }, "time": {
                        "hour": {时}
                        , "minute": {分}
                        , "second": {秒}
                        , "nano": {毫秒}
                    }
                }, "offset": {
                    "totalSeconds": {与 UTC 相隔的时差，单位：秒}
                }, "zone": {
                    "id": "{时区}"
                }
            }
        }
        ......
    ]
}
```

5.2 添加新弹幕信息

*{API_SERVER}/barrage/add*

创建新弹幕
```
POST => {
    "token": "{用户身份令牌}"
    , "content": "{弹幕内容}"
    , "videoId": {视频 id}
    , "color": "{可选：弹幕颜色，如 #ffffff，默认：#ffffff}"
    , "offtime": {可选：弹幕发送偏移时间，单位：秒，默认： 0}
    , "position": {可选：弹幕位置：0: 固定弹幕, 2: 浮动弹幕，默认：0}
}
------------------------------------------------------
=> success {
    "status": "success"
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "视频 id 为空"
    // "视频 id 不正确"
    // "弹幕内容为空"
    // "弹幕内容为空或超过250长度限制"

    // "未知错误":
    //          服务器错误，尝试重新提交

    // "用户不存在":
    //          用户被管理员删除，该用户不能再评论

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```

## 6. 上传文件相关

6.1 上传文件

*{UPLOAD_SERVER}/upload/{applyToken}*

上传文件
```
POST => {
    "file": {文件}
}
------------------------------------------------------
=> success {
    "status": "success"
	, "msg": "上传成功"
    , "url": "{文件在存储服务器中的文件名}"
}
=> failure {
    "status: "failure"
    , "msg": "{错误说明}"

    // 所有可能的错误说明：

    // "没有选择文件"
    // "文件体积超过上限"

    // "未登录或参数错误":
    //          未登录或服务器错误，检查登录状态或尝试重新提交

    // 全部 Token 鉴权失败返回值，参考 0.4
}
```
```
AJAX 调用样例：

HTML:
<form name="fileform" style="display: none">
    <input name="file" type="file">
</form>

JavaScript:
$.ajax({
        url: `${UPLOAD_SERVER}/upload/' + applyToken + '}`
        , data: new FormData(document.forms['fileform'])
        , type: 'POST'
        , encType: 'multipart/form-data'
        , processData: false
        , contentType: false
        , cache: false
        , success(response) {
          json = response;
        }
    });
```