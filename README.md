# 建兴智能-无线传输-(QRCode 前端)
## 前言
为了防止黑客或其他人为导致重要数据被篡改或窃取，公司致力研究一款产品能与机房物理隔绝，还能数据交互。
于是这么一款读取二维码数据无线传输的产品就出来了。
该Program是显示端，主要作用于收到后端的图片数据, 显示二维码，供给识别端读取。

![AppVeyor](https://img.shields.io/static/v1?label=MoJeffrey&message=OCR-QR-Code-Show&color=<COLOR>)

## 软件要求
[![Node - < 9.8](https://img.shields.io/badge/Node-v21.1.0-2ea44f?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://www.python.org/)

## 程序工作流程
1. 连接后端Websocket
2. 连接后后端会分配一个生产代号
3. 当有request的时候后端生成的二维码图片路径会以Websocket传输
4. 轮巡显示所需的二维码
5. 等待后端通知删除二维码

## 开发环境部署
```shell
# 下载 NVM（NodeJs 版本控制器）
https://github.com/coreybutler/nvm-windows/releases

# 使用教程
https://nvm.uihtm.com/

# 安装NodeJs v21.1
nvm install 21.1.0
nvm use 21.1.0
node -v

# 安装依赖
npm i
npm run serve
```

### 访问
```shell
http://localhost:8080/?highQuantity=1&widthQuantity=1&qrcodeSize=300
```
| 参数名           | 必选 | Type   | 解释          |
|---------------|----|--------|-------------|  
| highQuantity  | ❌  | number | 高有多少个QRCode |  
| widthQuantity | ❌  | number | 宽有多少个QRCode |  
| qrcodeSize    | ❌  | number | QRCode 多少xp |  


## 文件結構
```
.
│   .gitignore
│   babel.config.js
│   jsconfig.json
│   README.md
│   package.json
│   vue.config.js
│
├─── public --静态文档
│
├───src --程序主代碼
│   │   main.js
│   │   config.js -- 配置文档
│   │   App.vue
│   ├─── components -- 主要修改里面的qrcodeShow 文件
└───└─── assets
```
