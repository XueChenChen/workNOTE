
# 需求--NOTE
## 列表
1. [登录和注册](### 登录和注册[手机验证码])

2. [主页](### 主页)
  1. [主页内容](#### 1. 主页内容)
  1. [跑马灯](#### 2. 跑马灯)
  1. [下注](#### 3.  下注)
  1. [购买](#### 4. 购买)
3. [记录](### 记录)
  1. [购买记录](#### 1. 购买记录)
  1. [开奖记录](#### 2. 开奖记录)
4. [好友](### 好友)
5. [个人中心](### 个人中心)


### 登录和注册[手机验证码]
- reponseBody-data<br>
![login-response](./assert/login.png)
- needData<br>
![needData-response](./assert/userinfo.png)

### 主页
#### 1. 主页内容
- responseBody-data<br>
![buymain-response](./assert/buymain.png)
- needData<br>
latestWinningHistory补齐
#### 2. 跑马灯
- 尚缺少。
#### 3.  下注
- responseBody-data<br>
![](./assert/betpage.png)
- needData<br>
需不需要加以下数据。一切以后台为准<br>
![needData](./assert/betdetail.png)
#### 4. 购买
- responseBody-data<br>
![](./assert/paybet.png)
- needData<br>
开奖时间<br>
![](./assert/loeetyTime.png)

### 记录

#### 1. 购买记录
**1. responseOne(数据内容)**

- responseBody-data<br>
请求bug<br>
![](./assert/buylog.png)<br>
数据<br>
![](./assert/buylogDetail.png)
- needData<br>
![](./assert/buylogDetailtwo.png)

**2. responseTwo(取消关注)**
- 尚缺少

#### 2. 开奖记录
- responseBody-data<br>
![](./assert/historylog.png)
- needData<br>
 暂无

### 好友
- responseBody-data<br>
 ![](./assert/friend.png)
- needData<br>
![](./assert/friendNew.png)

### 个人中心
- 尚无(上传（需fromData）)<br>
![](./assert/UserInfo.png)
- 尚无（支付二维码列表）<br>
![](./assert/payQRCode.png)
