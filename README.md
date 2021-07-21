# wx-jssdk-uni
uni-app的微信JSSDK调用Demo

### 配置
#### 后端服务：  
server目录下为 koa 框架写的简易后端服务  
- 需要修改 config.js ，替换自己的 appid 及 secrect
- 默认端口为 4000

依赖安装：  
```
npm install
```

启动服务：  
```
node app.js
```

#### uni前端：  
uni-app 项目默认使用 8082 端口

本地调试：  
- 公众号后台添加 js 安全域名
- 运行项目后需要搭建内网穿透服务将端口映射至公网安全域名

