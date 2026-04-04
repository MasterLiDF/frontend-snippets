# uni-app的热更新/整包更新

### uni-app 自定义服务器热更新，核心是生成 wgt 资源包、自建版本检测接口、客户端下载安装 wgt、重启生效，完全脱离官方云服务，下面是完整可落地的实现方案（仅 App 端有效，H5 / 小程序不适用）。

## 一、核心原理
### uni-app App 热更新本质是：替换应用内置的 JS/CSS/HTML/ 静态资源（wgt 包），不修改原生层（so / 二进制）；用 plus.runtime.install 安装 wgt、plus.runtime.restart 重启生效；仅支持资源层更新，原生插件 / 权限 / 包名 / 图标变更必须整包上架。

## 二、准备工作
### 1. 生成 wgt 热更包（HBuilderX）
1. 打开项目 → 编辑 manifest.json → 修改 version 版本号（必须比当前线上 App 版本高，如 1.0.0 → 1.0.1）
2. 菜单：发行 → 原生 App - 制作移动 App 资源升级包
3. 等待打包完成，控制台会输出 .wgt 文件路径（如 unpackage/release/__UNI__XXXX.wgt）
4. 注意：wgt 包只能升级同 appid、同平台（Android/iOS 分开打包）

### 2. 服务器端准备
1. 上传 wgt 包,获取可直接下载的 URL（如 https://your-domain.com/update/__UNI__XXXX_1.0.1.wgt）
2. 开发版本检测接口（核心）
接口地址示例：https://your-domain.com/api/app/check-update
3. 接口逻辑：根据 currentVersion 和 latestVersion 做版本比较（建议用语义化版本比较，支持 1.0.0 < 1.0.1 < 1.1.0）

## 三、客户端实现（核心代码）
### 1. 入口：App.vue onLaunch 触发检查（仅 App 端）
```
<script>
export default {
  onLaunch: function() {
    // #ifdef APP-PLUS
    this.checkHotUpdate() // 启动时检查热更新
    // #endif
  },
  methods: {
    // 检查热更新
    async checkHotUpdate() {
      try {
        // 1. 获取当前App信息（版本、appid、平台）
        const widgetInfo = await this.getAppInfo()
        const currentVersion = widgetInfo.version
        const platform = plus.os.name.toLowerCase() // android/ios
        const appid = widgetInfo.appid

        // 2. 请求自建接口，获取更新信息
        const { data } = await uni.request({
          url: 'https://your-domain.com/api/app/check-update',
          method: 'POST',
          data: {
            currentVersion,
            platform,
            appid
          }
        })

        if (data.code !== 0 || !data.data.hasUpdate) return

        const updateInfo = data.data
        // 3. 提示用户更新（强制/非强制）
        await this.showUpdateModal(updateInfo)
      } catch (err) {
        console.error('热更新检查失败', err)
      }
    },

    // 获取App基础信息（Promise封装）
    getAppInfo() {
      return new Promise((resolve, reject) => {
        plus.runtime.getProperty(plus.runtime.appid, (info) => {
          resolve(info)
        })
      })
    },

    // 显示更新弹窗
    async showUpdateModal(updateInfo) {
      const { isForce, updateDesc, latestVersion } = updateInfo
      const modalOpts = {
        title: `发现新版本 v${latestVersion}`,
        content: updateDesc,
        showCancel: !isForce, // 强制更新隐藏取消按钮
        confirmText: '立即更新',
        cancelText: '稍后再说'
      }

      uni.showModal(modalOpts, async (res) => {
        if (res.confirm) {
          // 确认更新：下载+安装
          await this.downloadAndInstallWgt(updateInfo.updateUrl)
          // 安装成功，重启App
          plus.runtime.restart()
        } else if (isForce) {
          // 强制更新，取消则退出App
          plus.runtime.quit()
        }
      })
    },

    // 下载wgt并安装（核心）
    downloadAndInstallWgt(url) {
      return new Promise((resolve, reject) => {
        uni.showLoading({ title: '下载更新中...' })
        // 1. 下载wgt包
        uni.downloadFile({
          url,
          success: (downloadRes) => {
            uni.hideLoading()
            if (downloadRes.statusCode !== 200) {
              reject(new Error('下载失败，状态码：' + downloadRes.statusCode))
              return
            }
            const wgtPath = downloadRes.tempFilePath
            // 2. 安装wgt包（plus.runtime.install 仅App可用）
            plus.runtime.install(
              wgtPath,
              { force: true }, // 强制覆盖安装
              () => {
                uni.showToast({ title: '更新成功，即将重启', icon: 'success' })
                setTimeout(resolve, 1500)
              },
              (err) => {
                uni.showToast({ title: '安装失败：' + err.message, icon: 'none' })
                reject(err)
              }
            )
          },
          fail: (err) => {
            uni.hideLoading()
            uni.showToast({ title: '下载失败：' + err.errMsg, icon: 'none' })
            reject(err)
          }
        })
      })
    }
  }
}
</script>
```

## 四、关键配置与权限（必配）
### 1. manifest.json 权限开启（App 模块配置）
1. 勾选：App 资源在线升级（热更新）（5 + 模块）
2. 网络权限：确保开启 INTERNET 权限（Android）、NSAppTransportSecurity（iOS，允许 HTTP/HTTPS 下载）
### 2. iOS 审核注意：
1. App Store 禁止用热更修改核心功能、支付、登录逻辑，仅允许修复 bug、优化 UI
2. 建议：热更包仅用于非核心更新，核心变更走整包上架
### 3. Android 注意：
1. 允许应用内安装未知来源（Android 8+ 需动态申请权限）
2. 配置 AndroidManifest.xml 添加安装权限

## 五、完整流程总结
1. 开发完成 → 修改 manifest.json 版本号 → 生成 wgt 包
2. 上传 wgt 到自建服务器 → 更新版本检测接口的最新版本信息
3. 用户启动 App → App.vue onLaunch 调用 checkHotUpdate
4. 客户端获取当前版本 → 请求自建接口 → 对比版本
5. 有更新 → 弹窗提示 → 下载 wgt → 安装 → 重启生效

## 六、常见问题与优化
1. 版本号比较错误：用语义化版本比较（如 semver 库），避免字符串直接比较（1.10.0 > 1.2.0）
2. 下载失败 / 安装失败：检查 wgt 下载 URL 是否可访问、跨域、服务器带宽；wgt 必须和当前 App 同 appid、同平台
3. 强制更新体验：强制更新时隐藏取消按钮，取消则退出 App，避免用户跳过
4. 断点续传：大 wgt 包可加断点续传（uni.downloadFile 支持）、下载进度显示
5. 回滚机制：服务器可配置版本白名单，旧版本强制整包更新

