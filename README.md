# 项目：我的垃圾
## 简介
一个专门设计用来卖垃圾的网站，顺便满足大家捡垃圾的欲望。设计成c2c可以惠及大家一起*扔垃圾捡垃圾*。因为大家技术栈差别大，于是很愉快地设计成了**前后端分离**的模式。

## 功能畅想
- 正常空间：普普通通的上架自己的有点价值垃圾，比如植物学书籍、谭浩强的C语言、考研书籍、前女朋友送的装饰品、舒服的躺椅、羽毛球拍。要钱的，送货或自提。
- 垃圾堆：用来丢真正的垃圾，比如写了一半的笔记本、坏了的内存条、收集的瓶盖、成功学书籍。区别是可以0元捡走。
- 废墟拍卖场：超有价值的垃圾，比如用了两年没坏的洗衣机，换下的iPhone，没穿过的足球鞋。
- 服务区：出售各种服务，比如打印机、租自行车、陪吃饭、教写代码、代做海报
- 黑暗的角落: 只有好孩子才能看到的东西

## 盈利模式
- 普通用户免费
- 虽然不大好，但是垃圾堆那个区可以设计成虚拟货币（瓶盖）兑换垃圾。瓶盖登陆送。既防止了个人独占整个垃圾堆，也创造了盈利模式。
- 螺丝的可以卖明信片、考试资料(自己打印)，前期的营业额大家都看到了，都快收回打印机成本了
- 声望刷满了以后可以向wiki一样募捐一波
- 卖大会员？？？
- 组内贡献代码超过5%的都送管理员一个(无责任+随意ban人)

## 模块设计
- 后端  
分离式，只需要提供web api，不需要提供html。  
只描述API形式，语言框架内部逻辑大家随意。  
大模块暂时为**用户、商品、通用**  
设计要求，独立模块开发时开发环境只需配置自己模块及通用模块。  
数据之间的共享尽量利用redis，比如session_id。
- 前端  
分离式，需要的外部组件记得编写bower或npm安装。  
由于存在大量ajax及页面渲染的需要，最好用上Angular或者Vue什么的。  
做的丑点没关系，但要丑的**有个性**。  

### 后端API
#### 用户
包含登录注册权限管理  
##### 数据库
- 通用(redis)
  - Sessions-hset: account-session_id对
  - Captcha-hset: cookie-right_answer对
- user(关系型)
  - account-CharField: 64byte, Unique, 只限(纯数字及字母)或邮箱格式，用于唯一用户标记
  - name-CharField: 64byte，只限数字字母中文日文，用于各种profile中替代账户显示以保证安全
  - groupID-ForienKey: group外键
  - phone-CharField: 需符合手机号格式
  - email-CharField: 需符合邮箱格式
  - Address-CharField: 最好精确到寝室
- group(关系型)
  - id-IntegerField: 主键自增
  - permissionID-ForienKey:permission外键
  - name-CharField: 群名, 如管理员、中心、(暗网)什么的,不同组浏览商品、价格可不同
  - invite_seed-IntegerField: 128byte,邀请码生成种子
- permission(关系型)
  - id-IntegerField: 主键自增
  - action-IntegerField: 4byte，每个byte对应一种行为，按顺序为增删改查，每个byte的大小表示权限等级。如0x0148，表示不可增，1级删权限，4级改权限，8级查权限。只有user或所属group的permission>物品permission时，操作才被允许。
# 留坑  

##### web API
- /user
  - GET: 个人主页.html
    - 权限：登录-任意人主页
- /user/profile
  - GET: 个人信息.json
    - param: {account, session_id}
    - 权限：
      - 登录+普通用户-任意人信息  
      - 登录+管理员-任意人信息
  - POST:
    - param: {全部个人信息,account, session_id}
    - 权限：
      - 登录+普通用户-允许修改个人信息
      - 登录+管理员-允许修改任意人信息
- /user/login
  - GET: 登录界面.html
  - POST:
    - 参数：{account, sha256(password+验证码), 验证码}
- /user/register
  - GET: 注册界面.html
# 留坑

#### 通用
- /captcha
  - GET: 验证码图片的data以及相应cookie.json
    - 内部行为：
      - 1.生成图片,base64编码后返回(无硬盘IO)
      - 2.存储cookie及正确验证码对至内存型数据库(如redis)，设置超时(180s)
  - POST: 未定义行为
