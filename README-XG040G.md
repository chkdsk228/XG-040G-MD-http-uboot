# Nokia XG-040G-MD HTTP U-Boot

基于 [naoki66/XG2010G-http-uboot](https://github.com/naoki66/XG2010G-http-uboot) 修改，增加 Nokia XG-040G-MD 支持。

## ✨ 新增功能

### 支持的设备

- ✅ **Nokia XG-040G-MD** (贝尔 XG-040G-MD)
- ✅ **Nokia XG-040G-MD-UBI** (UBI 分区版本)
- ✅ **Nokia XG-040G-TF** (变体型号)

### 硬件特性

| 组件 | 规格 |
|------|------|
| SoC | Airoha AN7581 (4× Cortex-A53 @ 1.8GHz) |
| 内存 | 1GB DDR4 |
| 闪存 | 512MB SPI-NAND |
| PON | EN7572 XGSPON |
| 网口 | 2× 10G + 1× 2.5G + 1× 1G |
| USB | 2× USB 3.0 ⚡ |
| LED | 6× GPIO LED (电源/WAN/USB) |

### 与 XG2010G 的差异

**主要改进**:
- ✅ **启用 USB 支持** — XG2010G 禁用了 USB，XG-040G-MD 完整支持
- ✅ **USB PHY 驱动** — `CONFIG_PHY_AIROHA_EN7581_USB=y`
- ✅ **USB LED 指示** — 2 个 USB 口各有独立 LED
- ✅ **自定义提示符** — `XG-040G-MD>` 区分设备型号

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/chkdsk228/XG-040G-MD-http-uboot.git
cd XG-040G-MD-http-uboot
git checkout xg-040g-md-support
```

### 2. 安装依赖

**macOS**:
```bash
brew install gcc-arm-embedded dtc bison flex
```

**Ubuntu/Debian**:
```bash
sudo -S -p '' apt install gcc-arm-none-eabi device-tree-compiler bison flex
```

### 3. 编译

```bash
make an7581_nokia-xg-040g-md_defconfig
make -j$(nproc)
```

输出: `u-boot.bin`

### 4. 刷写（HTTP Recovery 推荐）

1. 设备启动时按住 Reset 按钮
2. 访问 http://192.168.1.1
3. 上传固件（不是 U-Boot！）

⚠️ **直接刷写 U-Boot 有变砖风险，需要串口救砖！**

详细说明见 [BUILD_INSTRUCTIONS.md](BUILD_INSTRUCTIONS.md)

## 📁 文件清单

### Device Tree 文件

```
dts/upstream/src/arm64/airoha/
├── an7581-nokia_xg-040g-md.dts           # 原厂分区
├── an7581-nokia_xg-040g-md-ubi.dts       # UBI 分区（推荐）
├── an7581-nokia_xg-040g-md-common.dtsi   # 公共定义（GPIO/USB/LED）
├── an758x-nokia_xg-040g-common.dtsi      # 基础定义
├── an758x-nokia_xg-040g-stock-parts.dtsi # 原厂分区表
└── an758x-nokia_xg-040g-ubi-parts.dtsi   # UBI 分区表
```

### Board 文件

```
board/airoha/an7581/
├── an7581_rfb.c                # 板级初始化（已添加 XG-040G-MD 支持）
├── nokia-xg-040g-md.env        # 环境变量配置
├── xg2010g.env                 # XG2010G 环境变量（对比参考）
└── Makefile
```

### 配置文件

```
configs/an7581_nokia-xg-040g-md_defconfig
```

关键配置:
- `CONFIG_DEFAULT_DEVICE_TREE="airoha/an7581-nokia_xg-040g-md-ubi"`
- `CONFIG_USB=y`
- `CONFIG_USB_XHCI_HCD=y`
- `CONFIG_PHY_AIROHA_EN7581_USB=y`
- `CONFIG_SYS_PROMPT="XG-040G-MD> "`

## 🔧 技术细节

### Compatible 字符串

```c
static bool xg040g_is_compatible(void)
{
    return of_machine_is_compatible("nokia,xg-040g-md") ||
           of_machine_is_compatible("nokia,xg-040g-md-ubi") ||
           of_machine_is_compatible("nokia,xg-040g-tf");
}
```

### 环境变量

```bash
bootcmd=run boot_ubi || http_recovery
boot_ubi=ubi part ubi && run boot_production
boot_production=ubi read ${loadaddr} fit && bootm ${loadaddr}#${bootconf}
```

- ✅ UBI 分区启动
- ✅ HTTP Recovery 备用
- ✅ FIT image 格式

### USB 配置

Device Tree 中启用了 USB:

```dts
&usb0 {
    vusb33-supply = <&reg_3p3v>;
    status = "okay";
    usb_port1: port@1 { };
};

&usb1 {
    phys = <&usb1_phy PHY_TYPE_USB2>;
    vusb33-supply = <&reg_3p3v>;
    status = "okay";
};
```

## 🧪 测试状态

⚠️ **实验性** — 等待实机测试

测试清单见 [BUILD_INSTRUCTIONS.md](BUILD_INSTRUCTIONS.md#-测试清单)

## 🤝 贡献

### 感谢

- **naoki66** — XG2010G U-Boot 和 ImmortalWrt 支持
- **YYH2913** — 原始 http-uboot 项目
- **OpenWrt 社区** — AN7581 平台支持

### 如何贡献

1. Fork 本项目
2. 创建功能分支 (`git checkout -b feature/xxx`)
3. 提交修改 (`git commit -m 'Add xxx'`)
4. 推送分支 (`git push origin feature/xxx`)
5. 提 Pull Request

### 测试反馈

如果你有 XG-040G-MD 设备并成功刷写，欢迎：
- 提交测试报告（Issue）
- 完善文档
- 提供硬件照片/串口日志

## 📚 参考资源

- **上游项目**: [naoki66/XG2010G-http-uboot](https://github.com/naoki66/XG2010G-http-uboot)
- **DTS 来源**: [naoki66/ImmortalWrt-for-Gemtek-XG2010G](https://github.com/naoki66/ImmortalWrt-for-Gemtek-XG2010G)
- **原始项目**: [YYH2913/http-uboot](https://github.com/YYH2913/http-uboot)

## ⚖️ 许可证

与上游项目相同（GPL-2.0）

## ⚠️ 免责声明

- 刷写 U-Boot 有变砖风险
- 请确保有串口工具用于救砖
- 操作前务必备份原厂 U-Boot
- 作者不对设备损坏负责

---

**项目状态**: ⚠️ 实验性 | **最后更新**: 2026-08-21
