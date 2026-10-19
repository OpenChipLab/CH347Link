<div align="center">

# CH347Link for Keil µVision

**让 WCH CH347 直接作为 Arm® Keil® MDK / µVision® 的 SWD / JTAG 调试器使用。**

[![License](https://img.shields.io/badge/license-BSD--2--Clause-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows-blue.svg)](#环境要求)

</div>

---

## 简介

CH347Link 基于开源项目 [elaphureLink](https://github.com/windowsair/elaphureLink) 修改。CH347Link 将 AGDI、RDDI、CMSIS-DAP 解释器和 CH347 传输层整合到同一个 32 位 `CH347Link.dll` 中，使 Keil µVision 可以把 CH347 设备直接作为 SWD / JTAG 调试器使用，无需运行额外的上位机或 Proxy 进程。

```text
Keil µVision (32 位)
   │ 加载 TDRV 驱动
   ▼
CH347Link.dll        AGDI + RDDI + CMSIS-DAP 解释器 + CH347 传输层
   │ 加载厂商接口
   ▼
CH347DLL.DLL         WCH 官方 CH347 厂商接口库
   │ USB
   ▼
CH347 设备  ──SWD / JTAG──►  目标 MCU (Cortex-M)
```

> 图形化安装：将 `CH347Link.dll`、`CH347DLL.DLL` 和 `KeilCh347Installer.exe` 放在同一目录，以管理员身份运行安装器，选择包含 `TOOLS.INI` 的 Keil IDE根目录，安装完成后重启 keil IDE。

## 主要特性

- **单驱动 DLL**：AGDI、RDDI、CMSIS-DAP 解释器与 CH347 传输层位于 `CH347Link.dll` 内，与 keil IDE 同进程运行。
- **SWD / JTAG**：可在 Keil 的 Debug Settings 中切换端口。
- **调试规范**：ADIv5；支持 ARMv6-M、ARMv7-M、ARMv8-M（继承自上游）。
- **完整调试能力**：断点、单步、寄存器 / 内存 / 外设查看，以及基于 Keil 标准 Flash 算法流程的下载和擦除。
- **系统复位**：AutoDetect 会先探测 SRST 是否真正产生硬件复位；有效时使用硬件复位，未接 nSRST 时回退到 `SYSRESETREQ` 软件复位。
- **可调时钟**：SWD 支持 100 kHz – 5 MHz；JTAG 最高可达 60 MHz。
- **多设备区分**：插入多个 CH347 时，可通过设备 SN 区分。
- **图形化安装**：自动定位 Keil、备份并修改 `TOOLS.INI`，随后复制驱动文件。

## 本目录文件

| 文件 | 架构 | 说明 |
|---|---|---|
| `CH347Link.dll` | Win32 / 32 位 | CH347Link 主驱动，供 32 位 µVision 加载。**必需。** |
| `CH347DLL.DLL` | Win32 / 32 位 | WCH 官方 CH347 厂商接口库。**必需。** 版权归 WCH 所有，不适用本项目的 BSD 许可证。 |
| `KeilCh347Installer.exe` | x64 / 64 位 | 图形化安装器。可选；也可以按本文档手动安装。 |
| `README.md` | - | 本说明。 |
| `LICENSE` | - | 项目许可证、第三方组件说明及 CMSIS-DAP EULA。 |

！请勿重命名两个 DLL。安装器要求上述三个运行文件位于同一目录。

## 环境要求

- **操作系统**：Windows 7 或更高版本。
- **图形化安装器**：需要 64 位 Windows 和 .NET Framework 4.x；在 32 位系统上请使用手动安装方式。
- **IDE**：Arm Keil MDK / µVision 5。µVision 为 32 位程序，`CH347Link.dll` 为配套的 Win32 构建；上游已在 v5.12 – v5.40 的多个版本中验证 AGDI 框架。
- **硬件**：具备 JTAG 能力接口的 WCH CH347 设备，例如 CH347T 或 CH347F。
- **WCH USB 驱动**：如果插入设备后系统未识别，请先安装 [WCH 官方 CH347 驱动包](https://www.wch.cn/downloads/CH341PAR_EXE.html)，并在设备管理器中确认设备已经正常枚举。

> 关闭 µVision 后再执行安装、升级或卸载操作。

## CH347Link驱动安装

### 方式一：图形化安装器（推荐）

1. 将 `CH347Link.dll`、`CH347DLL.DLL` 和 `KeilCh347Installer.exe` 放在同一目录。
2. 关闭 µVision。如果 Keil 位于 `C:\Keil_v5`、`C:\Program Files` 等受保护目录，请右键以管理员身份运行 `KeilCh347Installer.exe`。
3. 确认安装器自动检测到的 Keil 根目录，或手动输入包含 `TOOLS.INI` 的目录，然后点击“安装”。
4. 安装器会执行以下操作：
   - 将 `TOOLS.INI` 备份为 `TOOLS.INI.bak_时间戳`；
   - 在 `[ARMADS]` 中注册驱动（已注册时不会重复添加）：

     ```ini
     TDRVn=BIN\CH347Link.dll ("WCH / CH347 JTAG_SWD")
     ```

   - 将新增的 `TDRVn` 加入相应内核的 `CPUDLL` 列表；
   - 将 `CH347Link.dll` 和 `CH347DLL.DLL` 复制到 `<Keil>\ARM\BIN\`。
5. 重启 µVision。

### 方式二：手动安装

1. 关闭 µVision，将 `CH347Link.dll` 和 `CH347DLL.DLL` 复制到 Keil 的 `ARM\BIN` 目录，例如：

   ```text
   C:\Keil_v5\ARM\BIN\
   ```

2. 备份 Keil 根目录下的 `TOOLS.INI`，然后在 `[ARMADS]` 小节中新增一行 `TDRV`。编号应使用该小节中尚未占用的编号，下例使用 `TDRV8`：

   ```ini
   [ARMADS]
   TDRV0=BIN\UL2ARM.DLL ("ULINK2/ME ARM Debugger")
   TDRV1=BIN\CMSIS_AGDI.dll ("CMSIS-DAP Debugger")
   TDRV8=BIN\CH347Link.dll ("WCH / CH347 JTAG_SWD")
   ```

3. 将同一个 `TDRV` 编号加入所使用内核对应的 `CPUDLL` 列表。例如：

   ```ini
   CPUDLL1=SARMCM3.DLL(TDRV0,TDRV1,TDRV2,TDRV3,TDRV4,TDRV5,TDRV6,TDRV7,TDRV8)
   CPUDLL3=SARMV8M.DLL(TDRV2,TDRV4,TDRV6,TDRV7,TDRV8)
   ```

   > 如果没有把该编号同步到对应的 `CPUDLL`，Keil 的调试器下拉框中不会显示 CH347。

4. 保存 `TOOLS.INI`，重启 µVision。

## 使用

### 硬件连接

| 调试端口 | 连接信号 |
|---|---|
| SWD | `SWDIO`、`SWCLK`、`GND`，以及可选的 `nSRST` |
| JTAG | `TMS`、`TCK`、`TDI`、`TDO`、`GND`，以及可选的 `nSRST` |

确认 CH347 与目标板共地，并给目标板正常供电。

> Autodetect 会先发送一次 SRST 探测脉冲。若 nSRST 已连接并检测到硬件复位，将继续使用硬件复位；若未连接或未检测到复位，则自动回退到 `SYSRESETREQ`。显式选择 HW RESET、SYSRESETREQ 或 VECTRESET 时，按所选方式执行。

### Keil 配置

1. 打开工程，进入 `Options for Target -> Debug`。
2. 在 **Use:** 下拉框中选择 **“WCH / CH347 JTAG_SWD”**，然后点击 **Settings**。
3. 在驱动设置中配置：
   - **Port**：选择 `SWD` 或 `JTAG`；
   - **Max Clock**：选择目标可稳定工作的时钟。新板、长线或飞线环境建议从不超过 1 MHz 开始。
4. 在 `Flash Download` 页确认 RAM / Flash 算法与目标芯片一致。
5. 执行 Build + Download，或进入调试会话进行断点、单步、寄存器和内存查看。

连接正常时，驱动设置窗口会显示检测到的 CH347 设备和目标 CoreSight DAP 信息。

## 卸载

1. 关闭 µVision。
2. 使用 `TOOLS.INI.bak_*` 备份恢复原配置；也可以手动删除引用 `CH347Link.dll` 的 `TDRV` 项，并从相应 `CPUDLL` 列表中移除该 `TDRV` 编号。
3. 删除 `<Keil>\ARM\BIN\` 下的 `CH347Link.dll`。`CH347DLL.DLL` 如仍被其他工具使用，可以保留。
4. 重启 µVision。

## 常见问题

| 现象 | 处理方法 |
|---|---|
| 下拉框中找不到 “WCH / CH347 JTAG_SWD” | 检查 `CH347Link.dll` 是否位于 `<Keil>\ARM\BIN\`，`TOOLS.INI` 是否已注册该 `TDRV` 并同步到相应 `CPUDLL`，然后彻底重启 µVision。 |
| 提示连接失败或找不到设备 | 检查 USB 连接并在设备管理器中确认 CH347 已枚举；必要时安装 WCH 官方 CH347 驱动；关闭可能占用设备的其他软件。 |
| 连接不稳定或 Flash 校验失败 | 降低 Max Clock，缩短调试线，确认目标板与 CH347 共地，并检查目标供电是否稳定。 |
| 多个 CH347 无法分辨 | 使用 USB Tree View / USBView 等工具读取设备 SN，再按 SN 选择对应设备。 |
| 安装器无法写入 Keil 目录 | 以管理员身份重新运行安装器，或按“手动安装”步骤复制文件并修改 `TOOLS.INI`。 |
| `KeilCh347Installer.exe` 无法启动 | 确认系统为 64 位 Windows，并安装 .NET Framework 4.x；也可改用手动安装。 |

## 许可证与第三方组件

CH347Link 基于 [elaphureLink](https://github.com/windowsair/elaphureLink)（BSD-2-Clause，Copyright © 2022 windowsair）修改而来，修改部分继续按 **BSD-2-Clause** 分发。

- `CH347DLL.DLL` 是南京沁恒微电子（WCH）的专有厂商接口库，版权归 WCH 所有，不适用 BSD-2-Clause，按其自身许可条款分发。建议同时从 [WCH 官方网站](https://www.wch-ic.com/) 获取最新驱动包。
- 本项目采用 CMSIS-DAP 相关标准；使用时还需遵守 Arm CMSIS-DAP EULA。
- Arm、Keil、µVision 是 Arm Limited（或其子公司）在美国和/或其他地区的注册商标。
- 完整许可证文本、免责声明、CMSIS-DAP EULA 和第三方组件列表见 [LICENSE](LICENSE)。

本项目与 WCH、Arm 均无隶属或官方背书关系。

## 致谢

- [windowsair/elaphureLink](https://github.com/windowsair/elaphureLink)：本项目的 AGDI / RDDI / CMSIS-DAP 基础；
- [WCH 南京沁恒微电子](https://www.wch-ic.com/)：CH347 芯片、驱动和 `CH347DLL` 接口库。

---

## English

**CH347Link** turns a JTAG-capable **WCH CH347** device into an **SWD / JTAG debugger for Arm Keil MDK / µVision**. It is a fork of [elaphureLink](https://github.com/windowsair/elaphureLink) and combines AGDI, RDDI, the CMSIS-DAP interpreter, and the CH347 transport into one 32-bit `CH347Link.dll`.

### Quick start

1. Put `CH347Link.dll`, `CH347DLL.DLL`, and `KeilCh347Installer.exe` in the same folder.
2. Close µVision. Run `KeilCh347Installer.exe` as administrator.
3. Select the Keil root directory containing `TOOLS.INI` and click **Install**.
4. Restart µVision, open `Options for Target -> Debug`, and select **“WCH / CH347 JTAG_SWD”**.
5. Open **Settings**, select SWD or JTAG, and choose a suitable Max Clock. Start at or below 1 MHz for a new or unstable connection.

The driver is a 32-bit DLL for µVision. The GUI installer is 64-bit and requires .NET Framework 4.x. If the CH347 is not enumerated by Windows, install the [official WCH CH347 driver package](https://www.wch.cn/downloads/CH341PAR_EXE.html) first.

`CH347DLL.DLL` is WCH proprietary software and is not covered by the BSD license. The project is distributed under BSD-2-Clause; see [LICENSE](LICENSE) for the complete terms, third-party notices, and CMSIS-DAP EULA.