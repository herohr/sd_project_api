# API文档

## 通用HTTP状态码解释

**405** 使用的HTTP方法不正确

**500** 服务器出错，联系我

**401** 未鉴权，检查表单是否有authorization项



## 用户

### 注册POST /users

**POST表单**

| 表单项      | 数据类型           | 示例                                 |
| -------- | -------------- | ---------------------------------- |
| username | string(64)     | "test"                             |
| password | string(32)-MD5 | "34819d7beeabb9260a5c854bc85b3e44" |
| email    | string(64)     | "test@test.email"                  |

**响应**

| 响应码  | 解释     | json内容                                   |
| ---- | ------ | ---------------------------------------- |
| 200  | 注册成功   | {}                                       |
| 400  | 某项输入太长 | {<br />"reason": "...",<br />"tags": ["username", ...]} |
| 409  | 用户名已存在 | {<br />"reason":"..."}                   |

### 登陆、鉴权GET /user/login

**POST表单**

| 表单项      | 数据类型           | 示例                                 |
| -------- | -------------- | ---------------------------------- |
| username | string(64)     | "test"                             |
| password | string(32)-MD5 | "34819d7beeabb9260a5c854bc85b3e44' |

**响应**

| 响应码  | 解释           | json内容                                   |
| ---- | ------------ | ---------------------------------------- |
| 200  | 登陆成功，返回身份识别码 | {"authorization":"26be104b-1544-3a1d-bbda-76d5b32be521"} |
| 400  | 用户名或密码错误     | {"reason":"..."}                         |

### 初始个人信息POST /users/info

**POST表单**

| 表单项           | 数据类型       | 示例                                     |
| ------------- | ---------- | -------------------------------------- |
| authorization | string256  | “26be104b-1544-3a1d-bbda-76d5b32be521” |
| nickname      | string(64) | "petty"                                |
| sex           | string(8)  | "male" / "female"                      |
| age           | integer    | 18                                     |
| college       | string(64) | "陕西师范大学"                               |

**响应**

| 响应码  | 解释     | json内容                                   |
| ---- | ------ | ---------------------------------------- |
| 200  | 初始化成功  | {}                                       |
| 400  | 信息已经存在 | {"reason":"The user's info already exist"} |

### 更改个人信息PUT /users/info

**PUT表单**

| 表单项（可选项）      | 数据类型       | 示例                                     |
| ------------- | ---------- | -------------------------------------- |
| authorization | string(32) | “26be104b-1544-3a1d-bbda-76d5b32be521” |
| sex           | string     | "male" / "female"                      |
| college       | string(64) | "Xidian University"                    |
| 同上...         | ...        | ...                                    |

**响应**

| 响应码  | 解释                  | json内容                               |
| ---- | ------------------- | ------------------------------------ |
| 200  | 修改成功，返回被修改的字段名      | {"changed":["age", "sex"......]}     |
| 400  | 修改失败，某些项不合法，返回不合法的项 | {"not_allowed":["age", "sex"]......} |

### 获取个人信息 GET /users/info

**GET表单**

| 表单项                  | 数据类型       | 示例                                     |
| -------------------- | ---------- | -------------------------------------- |
| id(可选，不选则为当前登陆用户的id) | int        | 3                                      |
| authorization        | string(32) | “26be104b-1544-3a1d-bbda-76d5b32be521” |

**响应**

| 响应码  | 解释    | json内容                                   |
| ---- | ----- | ---------------------------------------- |
| 200  | 获取成功  | {'sex': 'm', 'user_id': 3, 'nickname': 'jack', 'college': 'ShanShiDa', 'age': 18} |
| 404  | ID不存在 | {"reason": ""User's info does not exist"} |



## 图片储存相关

图片储存与阿里云OSS上，客户端必须请求服务端才能获取阿里云OSS的图片的地址。

图片大小最好限制在30M以内。

### 获取上传图片的地址 POST /imags

#### POST表单

服务端响应的"form_item"为给阿里云上传文件必须的表单项，一定要都填写，然后再在后面加上上传的文件。

| 表单项           | 数据类型                               | 示例     |
| ------------- | ---------------------------------- | ------ |
| img_format    | string(必须为jpeg, jpg, bmp, png四种格式) | "jpeg" |
| authorization | ...                                | ...    |

**响应**

| 响应码  | 解释            | json内容                                   |
| ---- | ------------- | ---------------------------------------- |
| 200  | 获取成功          | {'image_id': 16, 'method': 'post', 'form_item': {'Signature': ..., 'success_action_status': 200, 'key': '16.jpg', 'policy': ..., 'OSSAccessKeyId': ...}, 'max_size': 1073741824, 'host': 'sd-project-test.oss-cn-shenzhen.aliyuncs.com'} |
| 403  | 未指定img_format | {"reason": "img_format is required"}     |
| 403  | 格式不符          | {"[fmt] is not allowed"}                 |

#### GET表单



#### PUT表单

在POST之后并且上传给阿里云，阿里云返回状态码之后，客户端一定要将阿里云返回的状态码和刚刚上传的图片image_id发送给服务端，服务端以此标记阿里云上OSS的资源被上传成功。

| 表单项         | 数据类型 | 示例              |
| ----------- | ---- | --------------- |
| image_id    | int  | 18              |
| status_code | int  | 200/203/400/403 |

##### 响应

| 响应码  | 解释                     | json内容                                   |
| ---- | ---------------------- | ---------------------------------------- |
| 200  | 服务器记录了客户端的上传成功         | {}                                       |
| 203  | 服务器记录了客户端的上传失败         | {}                                       |
| 403  | 图片不属于用户，检查image_id是否正确 | {"reason": "Image's uploader is not you" |
| 400  | 未给出status_code         | {"reason": "status_code is required"}    |
| 400  | 未给出image_id            | {"reason": "form_id is required"}        |
| 404  | image_id不正确            | {"reason": "Image id: [id] not found"}   |

#### GET表单

GET方法用户获取阿里云上的图片，可以由image_id、user_id、image_id_list获取图片的URL，其中user_id和image_id_list返回的都是一个[{"signed_url": …, "image_id": …}, …]数组，当用image_id_list查询时，返回的数组序和客户端提交的image_id_list序对应，当有image_id不存在的时候，该位置为null。

| 表单项               | 数据类型                         | 示例              |
| ----------------- | ---------------------------- | --------------- |
| image_id(可选)      | int                          | 9               |
| user_id(可选)       | int                          | 33              |
| image_id_list(可选) | 以分号";"分隔的字符串，每一项都是一个image_id | 23;44;22;11;13; |



##### 响应

| 响应码  | 解释                | json内容                                   |
| ---- | ----------------- | ---------------------------------------- |
| 200  | 获取成功              | {"signed_url": "http://......", "expire_time": } |
| 200  | 获取成功              | {"image_url_list":[{{"signed_url": "http://......", "expires": 6000, "params":None, "headers":None}}] |
| 404  | image_id对应的图片不存在  | {"reason": "image_id not found"}         |
| 403  | 没有权限访问该user_id    | {"reason": "You have no access to get the images"} |
| 400  | image_id_list解析错误 | {"reason": "The param image_id_list is illegal"} |
| 400  | 没有给出任何查询的参数       | {reason": "Must give one argument"       |

