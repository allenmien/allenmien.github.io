---
title:      Hackintosh踩坑盘点
---
# Hackintosh 那些踩过的坑

## 常识

### kext

| 驱动程序                 | 详细信息                                                     | 备注 |
| ------------------------ | ------------------------------------------------------------ | ---- |
| FakeSMC.kext             | 安装hackintosh的核心程序，没有它就没法在你的电脑上面运行macOS | 必备 |
| Lilu.kext                | 内核扩展程序，离开它，下面的几个程序都无法正常运行           | 必备 |
| WhateverGreen.kext       | 显卡综合修复，整合了核显、AMD、NVIDIA的综合修复，包括 （单卡启动黑屏，唤醒黑屏 等等）(依赖于Lilu) | 必备 |
| RealtekRTL8xxx.kext      | Realtek 8xxx网卡驱动程序                                     | 必选 |
| NvidiaGraphicsFixup.kext | 修复N卡的卡顿问题                                            |      |
| VoodooPS2Controller.kext | Voodoo键盘/鼠标驱动程序                                      |      |
| VoodooHDA.kext           | 万能声卡驱动                                                 |      |



## 具体问题