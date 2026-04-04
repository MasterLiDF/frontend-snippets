# uni-app推送消息通知

### 架构：服务器 → 调用 UniPush2.0 服务端API → 推送到用户手机
不需要任何云函数、不需要 uniCloud，纯后端接口调用。

## 一、核心前提：开启 UniPush2.0 模块（必须）
### 1. 开发者中心开通
1. 登录DCloud 开发者中心 → 我的应用 → 选择你的 App → 左侧「UniPush」→ 开通 UniPush2.0
2. 记录AppID、AppKey、AppSecret（后续配置用）
3. 安卓端：分别开通华为、小米、OPPO、VIVO、荣耀厂商推送通道（按官方指引提交应用信息、签名、包名，获取厂商密钥），否则离线推送在对应品牌手机失效

### 2. manifest.json 配置（HBuilderX）
1. 打开 manifest.json → 「App 模块配置」→ 勾选「Push (uniPush)」→ 选择 UniPush2.0
2. 填写 UniPush2.0 的 AppID、AppKey、AppSecret
3. 「App 权限配置」→ Android：勾选通知权限（android.permission.ACCESS_NOTIFICATION_POLICY、android.permission.POST_NOTIFICATIONS，Android13 + 必须）
4. 包名、签名证书必须与厂商后台一致，否则推送收不到

### 3. 云打包 / 自定义基座
必须用云打包 / 自定义基座真机测试，HBuilderX 基座不支持推送；打包时勾选「UniPush」模块

## 二、远程推送（服务端→客户端，后台 / 离线都能收到）
### 1. 客户端注册与监听（核心）
```
// #ifdef APP-PLUS
// 监听推送注册结果（获取CID，设备唯一标识，服务端推送用）
uni.onPushRegister((res) => {
  if (res.success) {
    const cid = res.cid;
    console.log('推送注册成功，CID:', cid);
    // 把CID发给后端，绑定用户ID
    uni.request({
      url: 'https://your-api.com/bind-cid',
      method: 'POST',
      data: { userId: 'user123', cid: cid }
    });
  } else {
    console.error('推送注册失败:', res.errMsg);
  }
});

// 监听推送消息（在线/离线点击）
uni.onPushMessage((res) => {
  console.log('推送消息:', res);
  if (res.type === 'receive') {
    // 在线收到消息，手动创建通知（UniPush在线默认不弹通知）
    uni.createPushMessage({
      title: res.data.title,
      content: res.data.content,
      payload: res.data.payload,
      channelId: 'msg_channel'
    });
  }
  if (res.type === 'click') {
    // 点击通知，处理跳转
    uni.navigateTo({ url: `/pages/detail/detail?id=${res.data.payload.id}` });
  }
});

// 初始化推送（App启动时调用）
uni.registerPush({ provider: 'unipush' });
// #endif
```
### 2. 服务端推送
调用 UniPush2.0 官方 HTTP 接口 发推送官方文档

## 三、关键细节（接近原生体验）
### 1. Android 通知渠道（8.0 + 必须）
#### 必须调用setPushChannel创建渠道，否则通知不显示；渠道可在系统设置中单独管理声音、震动、横幅
#### 渠道 ID 一旦创建不可修改，只能新增

### 2. 离线推送（应用关闭也能收到）
#### 必须开通厂商通道（华为 / 小米 / OPPO/VIVO），仅靠个推通道会被系统杀进程，离线收不到
#### 厂商后台配置：包名、签名 SHA256、应用名称，必须与打包一致
### 3. 权限处理（Android13+）
#### 必须动态请求POST_NOTIFICATIONS权限，用uni.requestPushPermission
#### 引导用户去系统设置开启通知：
```
uni.openAppAuthorizeSetting({
  appAuthorizeSetting: 'notification'
});
```


