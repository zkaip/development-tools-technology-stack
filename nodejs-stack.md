###应用
- hexo Blog框架 -g
- cleaver -g 用markdown制作Keynote
- [构建npm包](http://blog.fens.me/nodejs-npm-package/) | [搭建私有Npm服务器](http://blog.fens.me/nodejs-cnpm-npm/)
- nodemailer 发送邮件
- noteppt 制作ppt

#### 编辑器
- [markdown](https://github.com/evilstreak/markdown-js)
- marked -g markdown文本解析
- simditor JS文本编辑器

#### 安全
- var md5Value = require('utility').md5(value) //生成MD5
- crypto //salt MD5 对称加密解密 非对称加密解密
- [xss](https://github.com/leizongmin/js-xss)

#### 文件
- reap 文件回收
- del 删除
- mkdirp 建立文件夹
- ncp 复制文件

#### image
- layzr.js 图片延时加载
- [placehold.it](http://placehold.it/) 图片占位
- [fontawesome](http://fontawesome.io/) 

#### shell
- commanderjs 编辑命令行命令应用
- browserify -g nodeJS模块跑在浏览器中
- tty.js 浏览器运行终端

#### timer
- later 计划任务
- [moment](http://momentjs.com) 时间格式化

#### 可视化
- highcharts
- fastclick 消除点击事件延迟时间

#### auth
- cookie: cookie-parser / express
- session: express / express-session / connect-redis
- oauth: passport
- etagSession
- sessionCookie
- signedCookie
- token: jsonwebtoken

### build
#### assets
- uglify-js -g 合并压缩混淆JS代码
- decorator
 - core-decorators // @autobind @readonly @override @deprecate(别名@deprecated) @suppressWarnings
 - traits-decorator // @traits

#### modules
- webpack
- browserify
- requirejs

#### 兼容性
- babel -g // ES6=>ES5 $ babel-node
- Traceur

####版本管理
- nvm **node版本管理**
- bower **js版本管理**
- npm / cnpm **nodePackage包管理**

####应用管理
- nodemon -g `nodemon app.js` 自动重启(简单)
- forever -g
- pm2 -g 推荐

###爬虫
- superagent 抓取网页
- cheerio 分析网页
- supertest test环境抓取网页

###并发
- eventproxy 控制并发逻辑(事件发布订阅模式)
- async 控制并发量(异步流程控制)
- q | bluebird (Promise方案)
- generator (ES6原生Generator,koa代表) co模块
- thunkify

###Database
- mysql 
 - Sequelize  //MySQL ORM
- mongodb
 - [mmb](https://github.com/moajs/mmb) // mongodb backup
 - mongoose //mongodb ORM
 - `mongoexport -d xxx-db -c activities -o activities.csv` 命令行备份

###测试
- Makefile写脚本辅助
- package.json:`"test": "mocha-phantomjs index.html"`

####后端测试
- mocha 测试框架 -g `mocha`
- should 断言库
- istanbul 测试代码覆盖率工具 -g `istanbul cover _mocha`

####前端浏览器测试
- mocha -g `mocha init` 自动生成简单的测试原型
- chaijs 全栈的断言库(浏览器引入)
- mocha-phantomjs -g 模拟浏览器环境,命令行进行浏览器测试原型 `mocha-phantomjs index.html`

####配合兼容connect的web框架的集成测试
- supertest
- mocha
- should

####函数效率
benchmark  可以使用 http://jsperf.com/ 分享 benchmark

###框架
- express (express-cli -g) //大而全框架
- koa //ES6原生Generator,yeild
- vue // MVVM(VM),绑定数据
- react //虚拟DOM view,函数式编程
- reactNative 
- angular1.x //双向绑定
- angular2.x //组件化
- Electron //封装nodejs程序为桌面应用
- meteor //统一开发环境,Server Client都可以操作数据库,响应式操作

###线上部署 线上测试
- heroku (paas)
- [travis-ci](https://travis-ci.org/) 持续集成测试,测试不同node版本兼容性(线上源为 github )
- [coveralls](https://coveralls.io/) 测试行覆盖率

### 负载均衡
- retry //失败重试

### 信息传输
- dnode //rpc
- socketio //socket传输

### log
- log4js