> **Fork 说明**：本仓库 fork 自 [Reukix/Reloc](https://github.com/Reukix/Reloc)，原作者为 [Reukix](https://github.com/Reukix)，著作权归原作者所有。本 fork 仅用于个人学习与同步上游，遵循原项目的 AGPL-3.0 许可证。
>
> <div align="center">
  <img src="docs/icon.png" alt="Reloc Icon" width="96" height="96" />
  <h1>Reloc</h1>
  <p><strong>面向 iOS 的外设级定位模拟工具</strong></p>
  <p>
    基于 Swift、Objective-C 与 Rust FFI 构建 · 通过本机回环实现 iAP2 / MFi-v3 / NMEA 外设定位注入
  </p>
  <p>
    <a href="#系统兼容性"><img src="https://img.shields.io/badge/Platform-iOS%2017.0%2B-0A84FF?style=flat-square" height="20" /></a>
    <a href="#从源码构建"><img src="https://img.shields.io/badge/Stack-Swift%20%7C%20ObjC%20%7C%20Rust-F05138?style=flat-square" height="20" /></a>
    <a href="#免责声明与许可证"><img src="https://img.shields.io/badge/License-AGPL--3.0-red?style=flat-square" height="20" /></a>
  </p>
</div>

---

## 简介

**Reloc** 是一款运行在 iPhone 本机的外设级定位模拟应用。应用通过外部代理提供的本地回环（`10.7.0.1`）连接系统的 `com.apple.carkit.service` 服务，完成 Remote Pairing / Lockdown 会话协商、iAP2 链路同步、MFi-v3 鉴权与 NMEA-0183（`$GPGGA` / `$GPRMC`）报文流式下发，使系统以标准外设 GPS 身份接收实时坐标。

---

## 核心功能

| 模块 | 能力说明 |
| :--- | :--- |
| **定格定位** | 地图点击选点、关键词搜索、手动输入 WGS-84 坐标、多级反地理编码（`CLGeocoder` → `Nominatim` → `BigDataCloud` 兜底不卡顿）、运行中浏览并热更新定位 |
| **多源地图与纠偏** | 默认使用 **Apple 地图（MapKit 原生）** 并支持中国大陆 GCJ-02 ↔ WGS-84 自动双向纠偏；同时内置免 API Key 的 **全球街道图（Esri） / OpenStreetMap / 全球卫星图** 纯原生 WGS-84 瓦片地图（完全脱离 `MKMapView`，无高德字样）；可选 ±0.6 m 自然微漂移 |
| **极速触发与优雅恢复** | iAP2 鉴权后延迟唤醒 `CoreLocation` + 自动脉冲订阅 `0xFFFA` + 初始高速锁定突发，首页实时显示外设生效状态；系统缓存顽固时提供 **一键刷新定位服务（PiP 悬浮 5 秒倒计时）**；停止定位时下发 NMEA 失效报文（`quality=0` / `V`）与 iAP2 `RST` 复位包并自动唤醒本机真实 GPS |
| **后台保活** | 导航级后台定位保活 + 非零次声音频双保险；支持开关系统「定位服务」刷新缓存后自动重建后台断言，以及断线自动重连 |
| **本机配对与导入** | 兼容 `.rppairing` / `.mobiledevicepairing` / `.mobilepair` / `.plist`；原生副本文件选择器避免置灰不可选，支持「用 Reloc 打开」直接导入；iOS 27+ 支持本机一键配对与 **画中画（PiP）悬浮指引窗** 实时展示 6 位配对码 |
| **数据与实测** | 收藏夹（支持重命名 / 删除）、最近 30 条使用记录、异常中断一键恢复；内置 `CoreLocation` 实测面板（`isSimulatedBySoftware` / `isProducedByAccessory`）与协议日志筛选 |

---

## 系统兼容性

| 设备 / 系统版本 | 日常脱离电脑运行 | 首次获取配对文件方式 | 说明 |
| :--- | :---: | :--- | :--- |
| **iPhone · iOS 27.0+** | 支持 | **本机一键配对**（免电脑）或导入已有文件 | 通过 `_remotepairing-pairable-host._tcp.` + 画中画悬浮窗完成配对 |
| **iPhone · iOS 17.0 – 26.x** | 支持 | **需电脑通过 USB 首次导出**，导入后长期脱离电脑 | iOS 27 之前系统限制陌生主机无线首次配对 |
| **iPad（iPadOS）** | **不支持** | — | iPadOS 不支持 CarPlay 且不提供 `com.apple.carkit.service` 外设通道，启动时将自动提示不可用 |

> Reloc 仅支持 **iPhone**；应用自身不创建 VPN，回环流量（`10.7.0.1`）由外部代理工具提供。

---

## 快速开始与获取配对文件

### 1. 启用本机回环
在外部代理工具中启用指向 `10.7.0.1` 的本机回环映射。

### 2. 获取并导入配对文件

#### 方式 A：iOS 27+ 本机直接配对（免电脑，推荐）
1. 打开 Reloc **「设备连接」** 页，点击 **「一键配对这台 iPhone」**，画中画悬浮指引窗会自动弹出。
2. 前往系统 **「设置 → 隐私与安全性 → 开发者模式」**，滑到底部点击 **「与 Reloc 配对」**。
3. 对照悬浮窗中实时显示的 6 位配对码完成输入，配对记录将自动保存。

#### 方式 B：iOS 17 – 26 通过电脑首次导出配对文件
使用 USB 数据线连接 iPhone 并点击 **「信任此电脑」**，任选以下一种工具导出：

- **使用 `idevice_pair`（推荐，跨平台）**：
  连接设备后直接生成并导出 `.mobiledevicepairing` / `.plist` 配对文件。
- **使用 `pymobiledevice3`（命令行）**：
  ```bash
  pip3 install -U pymobiledevice3

  # 通过 USB 完成首次配对并开启 Wi-Fi 连接
  pymobiledevice3 lockdown pair
  pymobiledevice3 lockdown wifi-connections on

  # 导出系统已保存的配对记录（将 <UDID> 替换为设备 UDID）
  # macOS:
  sudo cp /var/db/lockdown/<UDID>.plist ~/Desktop/pairing.mobiledevicepairing
  # Windows:
  # 复制 C:\ProgramData\Apple\Lockdown\<UDID>.plist
  ```

- **导入到 Reloc**：
  将 `.rppairing` / `.mobiledevicepairing` / `.mobilepair` / `.plist` 传至 iPhone 后，任选其一即可完成导入：
  1. 在 Reloc **「设备连接」** 页点击 **「导入已有配对文件」** 选择文件；
  2. 在「文件 / 隔空投送 / 微信」中直接点击该文件并选择 **「用 Reloc 打开」**；
  3. 或放入 **「文件 → 我的 iPhone → Reloc」** 目录，启动 App 时会自动导入最新配对记录。

### 3. 验证并开始使用
在 **「设备连接」** 页点击 **「检测设备连接」**，状态变为 **「设备回环已就绪」** 后即可切回 **「定位」** 页开始定格定位。

> **后台保活提示**：建议在系统 **「设置 → 隐私与安全性 → 定位服务 → Reloc」** 中将定位权限设为 **「始终」** 并保持「精确位置」开启，避免切入微信等第三方应用时被系统挂起导致定位恢复。

---

## 从源码构建

### 环境要求
- macOS + Xcode 15 及以上
- [XcodeGen](https://github.com/yonaskolb/XcodeGen)（`brew install xcodegen`）
- （可选）如需重新编译 `RelocDeviceBridge.xcframework`：Rust toolchain（`aarch64-apple-ios` / `aarch64-apple-ios-sim`）

### 准备本地 MFi 凭据
请自行准备合法的 MFi-v3 凭据文件并放入本地目录（切勿将私有凭据提交至公开仓库）：
- `Reloc/Resources/MFi/certificate.p7b`
- `Reloc/Resources/MFi/identity.pk8`

### 生成工程与打包未签名 IPA
```bash
# 1. 生成 Xcode 工程
xcodegen generate

# 2. 编译未签名真机 Release 产物
xcodebuild build \
  -project Reloc.xcodeproj \
  -scheme Reloc \
  -configuration Release \
  -sdk iphoneos \
  -derivedDataPath build/DerivedData \
  CODE_SIGNING_ALLOWED=NO

# 3. 打包为未签名 Reloc.ipa（供自行签名安装）
mkdir -p build/Payload
cp -R build/DerivedData/Build/Products/Release-iphoneos/Reloc.app build/Payload/
(cd build && zip -qry Reloc.ipa Payload)
```

---

## 常见问题

<details>
<summary><strong>1. 本机配对或连接验证提示无法发布 / 发现本地服务？</strong></summary>

请前往系统 **「设置 → 隐私与安全性 → 本地网络（繁體：區域網路）」** 确认已允许 Reloc 访问本地网络，并确保外部代理已正确启用 `10.7.0.1` 回环。
</details>

<details>
<summary><strong>2. 已开始定位或停止定位后，系统位置未立刻切换 / 恢复？</strong></summary>

- **开始定位后**：Reloc 会在 iAP2 鉴权完成后自动脉冲唤醒 `0xFFFA` 并高速推送 NMEA，首页会实时显示是否已锁定为外设源（`isProducedByAccessory = true`）。若强室内 Wi-Fi / 室外 GNSS 下系统仍沿用缓存，直接点击卡片上的 **「一键刷新定位服务（浮窗指引）」**，按画中画浮窗提示关闭「定位服务」5 秒后重新打开即可 100% 强制生效。
- **停止定位后**：Reloc 会自动发送 NMEA 失效报文与 iAP2 `RST` 链路复位包，并自动唤醒本机真实 GPS 恢复定位；若室内弱信号下系统仍缓存上次位置，点击卡片上的 **「一键刷新定位服务（恢复真实位置）」** 即可立刻恢复。
</details>

<details>
<summary><strong>3. 为什么默认 Apple 地图在中国大陆会显示「高德地图」？</strong></summary>

受中国大陆测绘合规政策限制，iOS 原生 `MapKit` 在中国大陆环境下会自动切换为高德地图底图与角标。如需完全无高德字样的纯国际 WGS-84 地图，点击地图右侧图层按钮切换为 **「全球街道图 / OpenStreetMap / 全球卫星图」** 即可（全部免 API Key，且自动禁用 GCJ-02 偏移）。
</details>

<details>
<summary><strong>4. 为什么 iPad 无法使用 Reloc？</strong></summary>

Reloc 通过系统 CarKit 外设通道（`com.apple.carkit.service`）注入外设级 GPS。Apple 仅在 iPhone 上提供 CarPlay / CarKit 服务，iPadOS 不包含该服务，因此 Reloc 在 iPad 上启动时会直接提示设备不可用。
</details>

---

## 免责声明与许可证

> **免责声明**：本项目仅用于 iOS 外设定位通信协议研究、应用开发调试与自动化测试。使用者须严格遵守所在地区法律法规及目标服务的使用条款，因滥用本软件产生的一切后果由使用者自行承担。

本项目依据 **GNU Affero General Public License v3.0 (`AGPL-3.0-only`)** 开源发布，完整条款请参阅 [LICENSE](./LICENSE)。任何基于本项目的修改版本在分发或通过网络提供交互服务时，均须在相同 AGPL-3.0 协议下公开完整对应源代码。
