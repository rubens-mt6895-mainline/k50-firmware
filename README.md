# k50-firmware — Redmi K50（rubens）要用的厂商固件

从 Redmi K50 出厂系统里原样取出来的二进制：`/vendor/firmware`、`/lib/firmware` 下跟
触摸、无线、连接子系统、GPU 有关的那一小部分。没有源码，MediaTek 和 ST 也没给任何再分发许可。

放在这里只有一个目的：让 [k50-rootfs](https://github.com/rubens-mt6895-mainline/k50-rootfs)
的构建能出一个"刷完就能碰屏幕、能连 WiFi"的镜像。所以照 postmarketOS、LineageOS 这类项目的
老办法转载，不做任何权利主张，也不修改内容。

**别在这个仓库里加别的东西。** 文件名必须和驱动 `request_firmware()` 要的名字逐字一致，
多一个字符都加载不上。

## 里面是什么

按镜像里的落点摆放（`lib/firmware/…`）：

| 路径 | 是什么 | 谁要它 |
|---|---|---|
| `st_fts_L11a.ftb` | L11a 面板的 FTS 触摸固件 | `fts_touch_spi` |
| `mediatek/mt6895/WIFI_RAM_CODE_soc7_0_{1b,1b_t,1c}_1.bin` | 无线协处理器的 RAM code（三份对应不同 RF 配置） | `mt7921` |
| `mediatek/mt6895/soc7_0_ram_bt_{1,1_t,1a,1b,1b_t}_1_hdr.bin` | 蓝牙补丁（`1a` 那份 2.8 MB，其余 586 KB） | `btmtk` |
| `mediatek/mt6895/soc7_0_ram_mcu_*_hdr.bin` | 连接子系统 MCU 固件 | `mtk_conninfra` |
| `mediatek/mt6895/soc7_0_ram_wmmcu_*_hdr.bin` | WiFi MAC/PHY 固件 | `mt7921` |
| `mediatek/mt6895/{BT_FW,conninfra,wifi,fm_cust}.cfg` | 连接子系统的配置/校准 | 同上 |
| `mediatek/mt6895/{mt6630,soc}_fm_*` | FM 的 patch 与 coeff。FM 没在用，但不确定删掉会不会影响连接子系统初始化，先留着 | `mt6630` |
| `mali_csffw*.bin`、`mediatek/mt6895/mali_csffw*.bin` | Mali CSF 固件 | GPU 驱动 |

`SHA256SUMS` 是这堆文件的校验和。

## 怎么进镜像

k50-rootfs 的 workflow 会按 `config/config` 里的 `firmware_tarball` 下载本仓库某个 commit 的
归档，解到 `work/extra/lib/firmware/`，构建脚本再 `cp -a work/extra/. <rootfs>/`。不想要固件，
把 `firmware_tarball` 留空就行。

本地构建：

```bash
git clone --depth 1 https://github.com/rubens-mt6895-mainline/k50-firmware /tmp/fw
mkdir -p work/extra && cp -a /tmp/fw/lib work/extra/
sudo bash scripts/build-rootfs.sh
```

## 许可

没有许可。这里是别人编译好的二进制，不附带任何许可证，也不授权任何用途，只做原样转载。
介意就别用，或者自己从自己的设备上 dump 一份（怎么 dump 见 k50-rootfs 的 `docs/FLASH.md`）。
