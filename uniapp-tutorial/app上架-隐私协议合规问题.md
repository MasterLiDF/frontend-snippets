# app上架-隐私协议合规问题

## 一、uni-app 隐私合规的核心自检目标（免费检测版）
1. __弹窗时机合规__：用户同意隐私前，绝对不调用敏感 API（getSystemInfo、getLocation、chooseImage）、不初始化任何 SDK（uni 统计、友盟、极光、地图）、不申请权限
2. __权限最小化__：manifest.json 只留实际用到的权限，每个权限必须写具体用途，删除所有冗余权限（READ_CONTACTS、SEND_SMS 等）
3. __SDK 全披露__：隐私政策必须列出所有第三方 SDK+uni-app 引擎本身的收集行为（设备 ID、OAID、IMEI、日志），不能漏
4. __弹窗合规__：必须用template模式（安卓）、有「拒绝并退出」、无默认勾选、不可跳过、阻塞启动流程

## 二、安卓 App（uni-app）免费自检全流程
### Step1：自检（查权限、SDK、配置，最快速）
1. 检查 manifest.json+androidPrivacy.json
打开 manifest.json → app-plus → permissions：删除所有不用的权限，保留的每个权限必须写 desc（用途），示例：
```
"CAMERA": {"desc": "用于拍摄/上传商品图片、头像"},
"WRITE_EXTERNAL_STORAGE": {"desc": "用于保存图片、缓存数据"},
"ACCESS_FINE_LOCATION": {"desc": "用于获取当前位置，展示附近门店"},
"READ_PHONE_STATE": {"desc": "用于设备标识，保障账号安全、崩溃统计"}
```
2. 检查根目录androidPrivacy.json：必须"prompt": "template"、"buttonRefuse": "拒绝并退出"、二次确认文案完整，禁止用 custom 模式（custom 会提前采集设备信息）
### Step2：免费工具扫描 APK（查隐藏权限、SDK）
1. Exodus Privacy（查 SDK / 权限）：https://exodus-privacy.eu.org/
 - 上传你的 uni-app 打包 APK（未加固、未签名）
 - 报告直接列出：所有第三方 SDK、每个 SDK 申请的权限、收集的数据（设备 ID、位置、网络）
 - 对照报告：检查有没有你没主动引入的 SDK（uni-app 自带 uni 统计、DCloud 崩溃 SDK，必须写进隐私政策）、有没有多余权限

### Step3：时序检查
同意前不调用敏感 API、不初始化 SDK、不申请权限

### Step4：隐私政策
 隐私政策：含 uni-app 引擎声明、全 SDK 清单、权限用途、可访问链接