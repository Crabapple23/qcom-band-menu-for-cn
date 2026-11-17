
[简体中文](README.md) | [English](README-en.md)

# 高通 QMI 频段控制（Qualcomm QMI Band Control）

通过 Android 的 QRTR 接口发送 QMI 指令，实现网络制式（RAT）偏好设置、频段锁定（锁频）以及 5G-NR 模式切换等蜂窝网络强制控制功能。本项目同时提供命令行工具和 Android 应用两个版本。

# 功能特性

- 读取/修改 SIM 1 和 SIM 2 的网络强制配置文件
- 支持 GSM、WCDMA、LTE 和 NR 的网络制式（RAT）偏好选择与频段锁定
- 独立配置 NR-SA 和 NR-NSA 频段掩码（鸣谢：[h3nnes](https://github.com/h3nnes)）
- 支持选择 SA+NSA、仅 NSA（NSA-only）或仅 SA（SA-only）的 NR 运行模式
- 查询基带（Modem）上报的硬件支持频段列表
- 将频段锁定恢复为硬件默认设置
- 即时小区（Cell）与频点（ARFCN）锁定（仅限 App 版）
- 实时锁小区（Live cell lock）：实时列出基带上报的邻区信息，点击即可一键锁定（仅限 App 版）
- 实时网络（Live Network）页面：通过 QMI 直接从基带读取主服务小区、LTE 载波聚合（CA）、NR 小区、天线及邻区信息（仅限 App 版）
- 全局频段过滤开关：在此处隐藏的频段也会同步在邻区列表中隐藏（仅限 App 版）
- 基于 DIAG 模式的高级网络指标监测（实验性功能）
  - MIMO、Rank（秩）、MCS、调制方式、RB（资源块）数量、CSI-RS 端口等
  - 物理层（PHY）下行/上行吞吐量
  - 详细的 NRCA（5G 载波聚合）读数
  - 每个 LTE 和 NR 载波（PCell 主小区及 SCell 辅小区）各接收通道（RX chain）的天线测量数据
- 同时提供 Android 应用程序与命令行工具

<img height="600" alt="image" src="https://github.com/user-attachments/assets/56620800-fa04-4f80-a2db-914f86331a60" />
<img height="600" alt="image" src="https://github.com/user-attachments/assets/e31a72ee-fb34-4c1f-8686-a4baf68045c2" />
<img height="600" alt="image" src="https://github.com/user-attachments/assets/3a369f98-895e-4c53-94f1-f9e44227b51b" />

# 环境要求

- 搭载高通（Qualcomm）基带的设备
- 已获取 Root 权限
- App 版本需要 Android 11 及以上系统
- 如需使用 DIAG 模式：设备需具备厂商 DIAG 动态库（`/vendor/lib64/libdiag.so`）且正在运行 `diag-router` 服务
- 命令行版本需要终端环境（如 Termux 或 ADB Shell）

不同代际的高通基带、设备厂商定制、固件版本以及双卡实现方案之间可能存在兼容性差异。

# 参与贡献

欢迎提交贡献！在提交 Issue 或 Pull Request 之前，请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。

# 安装与使用

## Android 应用版

从本仓库的 **Releases** 页面下载 APK 安装包，安装并在弹窗提示时授予 Root 权限即可。

应用通过原生后端经由 QRTR 上的 QMI 协议直接与基带通信。

## 命令行版本

命令行版本可通过 Termux 或直接通过 ADB 启动。

### Termux 安装步骤

1. 下载编译好的 `qcom-band-menu` 二进制文件，并将其放入 Android 的 `Download` 目录下：

   ```text
   /sdcard/Download/qcom-band-menu

```

2. 确保 Termux 拥有存储访问权限。如有需要，请运行：
```bash
termux-setup-storage

```


3. 在普通的 Termux Shell 中运行以下命令完成安装并启动：
```bash
cp /sdcard/Download/qcom-band-menu "$PREFIX/bin/qcom-band-menu" && chmod 755 "$PREFIX/bin/qcom-band-menu" && printf '%s\n' '#!/data/data/com.termux/files/usr/bin/sh' 'exec su -c /data/data/com.termux/files/usr/bin/qcom-band-menu' > "$PREFIX/bin/band" && chmod 755 "$PREFIX/bin/band" && band

```



执行安装命令前请勿提前进入 Root Shell，启动脚本内部会自动调用 `su -c` 来运行原生二进制程序。

安装完成后，随时可以通过以下命令启动命令行界面：

```bash
band

```

启动器已安装至 Termux 现有的可执行文件路径中，因此无需重启 Termux 或修改 `.profile`。

# 锁小区（Cell Lock）

锁小区页面提供两种模式，可通过 SIM 卡标签页旁边的切换开关进行选择。

**实时模式（Live，默认）** 会通过 QMI 每秒读取一次基带当前上报的邻区信息，并以与“实时网络”页面相同的表格形式展示（PCI、频段、EARFCN、RSRP、RSRQ）。点击列表中的某一行即可锁定至该小区：

| 列表行类型 | 发送的指令 |
| --- | --- |
| LTE 邻区 | LTE 小区锁定（EARFCN + PCI） |
| NR 邻区（仅含频点信息，基带未上报 NR PCI） | NR-ARFCN 频点锁定（附带显示的 SCS 子载波间隔） |
| 已锁定的行 | 解除对应网络制式（RAT）的小区锁定 |

如果当前 NR 主服务小区携带了 PCI 信息，则会作为完整小区进行锁定（ARFCN + PCI + SCS + 频段）。顶部的启用/暂停开关可停止读取基带数据（列表画面保持静止以便点选小区，锁定操作依然有效）；**Bands（频段）** 筛选开关用于控制显示哪些频段——该过滤规则与“频段”页面共用，因此对所有页面全局生效。WCDMA 和 GSM 邻区虽会列出测量数据，但基带本身不支持对这两种制式进行锁小区（对应行也会给出相应提示）。

**手动模式（Manual）** 保留了原有的文本输入框，支持手动输入参数进行锁定（NR-ARFCN、PCI、多 PCI、gNB 白名单、LTE PCI、多小区及 PLMN）。

# 工作原理

`qcom-band-menu` 使用 `AF_QIPCRTR` 数据报套接字，通过 Android 的 **QRTR** 传输层发送 QMI 指令。

```text
Android App / Termux / ADB
            |
            v
      qcom-band-menu
            |
            | AF_QIPCRTR / SOCK_DGRAM
            v
       QRTR 控制服务
            |
            | 动态服务发现
            v
     高通 QMI 服务组件
            |
            +-- NAS (服务号 0x03)
            |     RAT 网络制式偏好
            |     GSM/WCDMA/LTE/NR 频段掩码
            |     SIM 卡订阅绑定
            |     NR-SA / NR-NSA 频段掩码
            |     NR 运行模式
            |
            +-- DMS (服务号 0x02)
                  硬件支持的频段能力查询

```

### 动态 QRTR 服务发现

QMI 服务端口是在运行时动态分配的，不同设备或基带固件版本之间可能各不相同。

原生后端没有硬编码 QRTR 端口，而是向 QRTR 控制端点发送查询请求，动态搜索以下服务：

```text
QMI NAS service: 3
QMI DMS service: 2
QMI version:     1
QMI instance:    0

```

获取到基带对应的节点（node）和端口（port）后，后端便会向该服务建立已连接的 `AF_QIPCRTR` 套接字。

### QMI 数据包格式

通过 QRTR 发送的 QMI 数据包采用以下头部结构：

```text
偏移量  字节数  字段说明
0x00    1       QMI 消息类型
0x01    2       事务 ID（Transaction ID，小端序）
0x03    2       消息 ID（Message ID，小端序）
0x05    2       TLV 载荷长度（小端序）
0x07    ...     TLV 载荷数据

```

本工具使用的消息类型：

```text
0x00  Request（请求）
0x02  Response（响应）
0x04  Indication（指示）

```

每条 QMI 响应通常都包含一个结果 TLV：

```text
TLV ID: 0x02
值内容:
  uint16 result
  uint16 error

```

指令执行成功时返回：

```text
result = 0x0000
error  = 0x0000

```

### 订阅绑定（SIM 卡选择）

在读取或修改强制配置之前，NAS 客户端会通过以下消息绑定到所选的 SIM 卡：

```text
QMI NAS 消息: 0x0045

```

在已测试的实现中，绑定值定义如下：

```text
0 = SIM 1
1 = SIM 2

```

绑定关系的作用域仅限当前套接字（Socket），因此绑定请求以及随后的所有读写操作都必须使用同一个 NAS 连接。

这一机制使得工具能够独立读取和修改每张 SIM 卡所保存的强制网络配置。

### 读取当前配置

当前 RAT 和频段偏好通过以下消息读取：

```text
QMI NAS 消息: 0x0034
Get System Selection Preference

```

响应中包含的关键 TLV 如下：

```text
0x11  RAT 网络制式偏好
0x12  GSM/WCDMA 频段位图（Bitmap）
0x15  LTE B1-B64 频段位图
0x23  扩展 LTE 频段位图
0x2C  NR-SA 频段位图
0x2D  NR-NSA 频段位图
0x2B  当前 NR 运行模式（当以 4 字节字段返回时）

```

NR 运行模式的编码定义如下：

```text
0 = SA + NSA
1 = 仅 NSA (NSA only)
2 = 仅 SA (SA only)

```

### 应用 RAT 与频段设置

强制控制指令通过以下消息发送：

```text
QMI NAS 消息: 0x0033
Set System Selection Preference

```

本工具采用精细化的独立更新策略，而非重写整个基带配置。

例如可单独下发：

```text
仅修改 RAT 制式
仅修改 LTE 频段
仅修改 NR-SA 频段
仅修改 NR-NSA 频段
仅修改 NR 运行模式

```

设置请求中的关键 TLV 包括：

```text
0x11  RAT 网络制式偏好
0x12  GSM/WCDMA 频段位图
0x15  LTE B1-B64 频段位图
0x17  变更持续时间（Change duration）
0x24  扩展 LTE 频段位图
0x2B  组合 NR 频段位图
0x2E  NR 运行模式
0x2F  NR-SA 频段位图
0x30  NR-NSA 频段位图

```

LTE 频段在位图中对应的比特位为 `band - 1`。

例如：

```text
B1  -> bit 0
B3  -> bit 2
B28 -> bit 27

```

NR 频段在 64 字节位图中遵循相同的映射规则：

```text
n1  -> bit 0
n41 -> bit 40
n78 -> bit 77

```

### 硬件频段检测

硬件支持的频段列表通过以下服务和消息查询：

```text
QMI DMS 服务: 2
QMI DMS 消息: 0x0045
Get Band Capabilities

```

响应内容包含：

```text
传统 GSM/WCDMA 频段能力位图
LTE 频段能力位图
扩展 LTE 频段数组
NR 频段数组

```

工具会将这些字段解析为具体的频段列表，并用于：

* 拦截并拒绝硬件不支持的频段选择
* 展示基带实际支持的 GSM、WCDMA、LTE 和 NR 频段
* 将频段掩码恢复为硬件支持的默认全开状态

### 指令校验机制

部分高通基带固件会以异步方式更新存储的强制状态。

因此，在设置指令返回成功后，后端会短暂等待并重新查询一次基带状态，以避免在指令刚被接受时立即显示过期的旧数值。

完整的指令执行流程如下：

```text
发现 QMI 服务
     |
打开 QRTR 套接字
     |
绑定目标 SIM 卡
     |
发送 QMI 设置指令
     |
检查结果 TLV 状态
     |
等待状态同步生效
     |
查询当前最新状态
     |
将更新后的数值返回给 CLI 或 Android 应用

```

# DIAG 网络指标

QMI 协议仅上报基带面向用户汇总的基础信息：主服务小区、信号强度以及载波频率列表。而单载波级别的详细底层参数——例如每个辅小区的 MCS、资源块（RB）、Rank、SINR 或物理层实时吞吐量——仅存在于基带的 **DIAG** 诊断日志中。

DIAG 模式通过 **Diag Consumer Interface（DCI，诊断消费者接口）** 读取这些日志，这与设备端原生诊断工具所使用的接口一致。独立的 Root 守护进程 `qbm-diagd` 会订阅一组固定的日志代码（Log Codes），在手机本地完成解码，并将按载波汇总的数据传递给应用程序。

```text
Android App (实时网络页面)
            |
            | 抽象套接字 "qbm_diagd"，每秒请求一次
            v
        qbm-diagd (Root 权限)
            |
            | dlopen 动态加载
            v
  /vendor/lib64/libdiag.so
            |
            | DCI 客户端
            v
        diag-router
            |
            v
    基带 (MPSS) DIAG 日志流

```

### 仅使用 DCI 接口（不抢占主日志会话）

守护进程仅注册为 DCI 客户端并添加自身的日志订阅，它绝不会接管主诊断日志会话、绝不修改主日志掩码，也绝不切换日志记录模式。

因此，在开启 DIAG 模式的同时，其他第三方 DIAG 工具和 USB 诊断调试依然可以正常工作。该守护进程对无线射频模块完全保持只读访问。

DCI 会话通过以下流程初始化：

```text
Diag_LSM_Init
diag_register_dci_client      外设目标: modem (MPSS)
diag_dci_set_version(1)       启用扩展头部: 区分每条日志所属的 SIM 卡
diag_log_stream_config        订阅下方的指定日志代码

```

在骁龙 865 / 870 等采用外挂 X55 基带的平台上，基带作为远程处理器（MDM）存在，而非集成在 SoC 内部的 MPSS 中。由于 `libdiag` 只有在客户端注册后才会报告哪个处理器的 MPSS 支持 DCI，因此守护进程会先在 SoC 侧注册，随后进行探测：若发现 SoC 侧无 MPSS 而外挂基带上存在 MPSS，则自动迁移至外挂基带建立客户端；若探测结果不明确且驱动报告存在远程设备，则会同时在两侧保留客户端，并优先解码最先传回日志的一侧。

### 订阅的日志代码（Log Codes）

LTE 相关日志：

```text
0xB173  PDSCH stat indication     MCS、调制方式、RB 数量、层数（Layers）、CRC、传输块大小（TB size）
0xB126  PDSCH demapper config     收发天线数（Tx/Rx）、Rank（秩）、传输模式（TM）
0xB139  PUSCH Tx report           上行传输块大小（TB size）
0xB184  CA metrics                PCell/SCell 的 EARFCN、PCI、频段、带宽及激活状态
0xB193  Serving cell measurement  单载波 RSRP、RSRQ、SINR，以及各接收通道（RX chain）独立测量值
0xB179  Intra-frequency measure   未激活 SCell 辅小区的 RSRP、RSRQ
0xB168  Random access MSG2        时间提前量（Timing Advance, TA）

```

NR 相关日志：

```text
0xB887  PDSCH status              MCS、调制方式、RB 数量、层数（Layers）、传输块大小（TB size）
0xB883  UL physical channel       上行传输块大小（TB size）
0xB97F  Measurements              SS-RSRP、SS-RSRQ，以及各接收通道（RX chain）的 RSRP
0xB8D8  SNR                       SS-SINR、CSI-SINR，以及各接收通道的独立信噪比
0xB8DD  SNR (X55 至 X65 基带)     SS-SINR、CSI-SINR，以及各接收通道的独立信噪比
0xB96A  Beam measurements (X65)   最强波束的 SNR
0xB8A7  CSF                       CQI（信道质量指示）、RI（秩指示）
0xB825  RRC configuration         载波列表、频段、带宽、子载波间隔（SCS）
0xB823  Serving cell              PCell 主小区 PCI、ARFCN、频段
0xB88A  RACH attempt              时间提前量（Timing Advance, TA）
0xB821  RRC OTA                   CSI-RS 端口配置

```

### 日志版本兼容处理

每条日志代码的二进制结构布局会随高通基带代际的演进发生变化。所有受支持的版本结构均定义在布局表（`qbm_layouts.h`）中，只有当数据包的版本号已知且长度完全匹配时才会进行解码。

对于未知版本的数据包，程序会直接跳过而不会强行猜测解析，以确保稳定性。

其中，`0xB821` 日志携带的 NR RRC 消息使用基于 3GPP TS 38.331 规范生成的 ASN.1 解码器进行解析（见“致谢”部分），用于读取每个小区配置的 CSI-RS 端口数。

# 致谢

特别感谢 [h3nnes](https://github.com/h3nnes) 开发了 Android 应用版本，并发现了独立控制 NR-SA 与 NR-NSA 频段锁定的 QMI 指令。

本项目也极大地受益于 [libqmi 项目](https://github.com/linux-mobile-broadband/libqmi) 提供的文档、源代码和 QMI 协议定义。

同时感谢由 Lev Walkin 创建（[原始仓库](https://github.com/vlm/asn1c)）并由 mouse07410 及其他贡献者维护的 [asn1c](https://github.com/mouse07410/asn1c) 项目。DIAG 守护进程中的 NR RRC 解码器正是借助该项目从 3GPP TS 38.331 ASN.1 定义文件生成而来的（[许可证文件](https://www.google.com/search?q=app/binary/ASN1C-LICENSE)）。

```
