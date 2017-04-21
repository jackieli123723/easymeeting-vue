# easymeeting-vue

> The fore-end project of EasyMeeting
## 现况
根据[关于vue项目多页面的配置](http://www.jianshu.com/p/acbff04b4096)实行了webpack的魔改以实现多页面。现在各个页面放在`src/pages`下，webpack会自动检测各js并生成目录。  

注意：
- `pages/`下同一文件夹的vue和js文件名不建议相同
- `pages/`下的组件样式不建议用`scoped`属性
## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

## 登录机制
> 参考了淘宝的思路

本地保存的信息有三个级别： 
 
|存储位置|特性|
|:---:|:---:|  
| vue对象中的`data`|各页面内部存取，用于数据绑定|
|`SessionStorage`|会话存储，跨页面，伴随会话失效|
| `LocalStorage`|本地存储，长期|

用户初次次登录时，获取到的用户登录信息后，依次将信息存储到这三级中去：

|存储位置|信息|
|:---:|:---:|  
| vue对象中的`data`|`user`里的所有信息|
|`SessionStorage`|`user`内的所有信息|
| `LocalStorage`|仅存储`user.token`和`user.username`|

当用户新打开一个网页时，按以下顺序检查和恢复数据：  
1. 检查`SessionStorage`，若存在`user`则将其存到内存
2. 若无，则检查`LocalStorage`是否有`user`
    1. 无：首页则header显示登录按钮；个人页则跳转到登录页
    2. 有：将`LocalStorage`存储到内存；  
      若是个人页，则通过`/api/userinfo`提交token，获取完整信息并存到`SessionStorage`和内存；如果`token`已失效，则清除信息并跳转到登录页。  
      
_注：对于header，无需登录就能看见用户名，即仅需要localstorage的user信息就可以显示。但进入个人中心则需要登录_  

注销时应依次清除这三级保存的信息。

## 接口
### userExist 用户邮箱查重
```html
/api/userexist?email=xxxxxxxxxxxxxxxx
```
- type: GET  
- return:
  - `true` 该邮箱已注册
  - `false`邮箱未注册
  
### signup 用户注册
```
/api/signup
```
- type: POST
- data: 
  - email: 登录邮箱
  - username: 用户昵称
  - password: 密码
  - gender: 1为男性, 0为女性
  - description: 自我介绍
- return:
```json
{
  "status": 0,
  "description": "注册成功。"
}
```
- status
  - 0：注册成功
  - 1：邮箱已注册
  - 2：某项必填信息缺失
  
### login 用户登录
```
/api/login?email=xxxxxxxxxxxxxx&password=xxxxxxxxxxxxxxxxxx
```
- type: GET  
- return:
```json
{
  "status": 0,
  "user": {
    "username": "王大锤",
    "email": "fewjiofa@rrjgeor.cn",
    "gender": true,
    "avatar": "dfaeiwofe.png",
    "description": "dfwahefneklmd",
    "token": "faweirhoiewnkdksl"
 }
}
```
- status
  - 0: 登录成功，并返回user信息
  - 1：密码错误
  - 2：账号不存在
- user
  - username：用户昵称
  - email：用户邮箱
  - gender：用户性别（true为男，false为女）
  - avatar：头像图片地址
  - description：用户自我介绍
  - token：登录状态密钥，浏览器本地保存
  
### userinfo 获取用户信息
```
  /api/userinfo?token=xxxxxxxxxxxxxxxxxxxxxxx
```
- type: GET  
- return:
```json
{
  "status": 0,
  "user": {
      "username": "王大锤",
      "email": "fewjiofa@rrjgeor.cn",
      "gender": true,
      "avatar": "dfaeiwofe.png",
      "description": "dfwahefneklmd",
      "token": "faweirhoiewnkdksl"
   }
}
```
- status
  - 0：token有效，返回用户信息
  - 1：token已失效，需要重新登录
  - 2：token是假的
  
### logout 注销登录
```
  /api/logout?token=xxxxxxxxxxxxxxxxxxxxxx
```
- type: GET  
- return:  
`true`: 注销完成
