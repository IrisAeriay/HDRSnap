# HDRSnap

Windows 上的 HDR 截图工具：在开启 HDR 的显示器上截图不再发灰发白，颜色和屏幕所见一致；还能同时保存一份真正的 HDR 副本。

## 特性

- **SDR PNG**：按系统当前 SDR 白点（`DISPLAYCONFIG_SDR_WHITE_LEVEL`）做色调映射，SDR 内容与 GDI 截图逐像素一致（实测平均误差约 1/255）
- **HDR 副本**：可选同时输出 AVIF（PQ / BT.2020 / 10-bit），支持 HDR 查看器回看高光
- **区域 / 全屏截图**：全局热键，默认 `Ctrl+Alt+S` / `Ctrl+Alt+D`
- **托盘常驻**：右键菜单截图、改保存位置、自定义热键、开关 HDR 副本
- **自动复制到剪贴板**，保存路径可一键更改
- **命令行模式**：`--shot` / `--status` / `--set-savedir`，便于脚本调用
- 无需管理员权限，不联网、不收集数据

## 环境要求

- Windows 10/11 x64（HDR 捕获需系统 HDR 已开启；已在 Windows 11 24H2 + RTX 3060 以上环境验证）
- .NET Framework 4.7.2+（Windows 10 1803+ 自带）
- **ffmpeg**（需包含 `ddagrab` 与 `libaom-av1`，建议 ffmpeg 6.1+）。放在程序同目录 `ffmpeg.exe`、加入 PATH，或首次启动时在弹窗中手动选择
- 编译只需 PowerShell（无需 .NET SDK）

## 使用

1. 双击 `HDRSnap.exe`（或 Release 里的 exe），托盘出现图标
2. 区域截图 `Ctrl+Alt+S`，拖出选区；全屏截图 `Ctrl+Alt+D`
3. 截图自动复制到剪贴板，保存到 `我的图片\HDR Snap`（可右键托盘图标 → `更改保存位置…` 修改）
4. 右键托盘图标 → `设置热键…`，点进输入框直接按下想要的组合键即可，立即生效；如果新热键被其他程序占用会提示，并自动保留原来的热键
5. 右键托盘图标 → `同时保存 HDR 副本 (.avif)` 可开关 HDR 副本

## 配置（config.ini，与 exe 同目录）

```ini
; ffmpeg 路径；留空 = 自动查找 exe 同目录 / PATH
ffmpeg=
; 保存目录；留空 = 我的图片\HDR Snap
saveDir=
hotkeyRegion=Ctrl+Alt+S
hotkeyFull=Ctrl+Alt+D
; 1 = 同时保存 AVIF(HDR)，0 = 只保存 SDR PNG
saveHdrAvif=1
; 高光压缩强度（0~1），越大越接近原 HDR 观感
toneMapKnee=0.15
```

## 命令行

```bat
HDRSnapCli.exe --status
HDRSnapCli.exe --shot --rect 0,0,2560,1440 --out D:\shot.png
HDRSnapCli.exe --shot --rect 300,300,600,400 --out D:\region.png --no-avif
HDRSnapCli.exe --set-savedir "D:\Screenshots"
```

## 工作原理

- 通过 ffmpeg `ddagrab` 以 `rgbaf16` 抓取 Desktop Duplication 的 scRGB 线性帧（1.0 = 80 nit）
- SDR 输出：`v / (SDR白点nit / 80)` 后按 sRGB 编码；超过 1 的高光按 `toneMapKnee` 压缩后截断
- HDR 输出：线性值 → nits → BT.2020 → PQ（ST 2084）16-bit，再由 libaom-av1 编码为 AVIF
- HDR 关闭或抓取失败时自动回退 GDI 截屏

## 编译

```powershell
powershell -ExecutionPolicy Bypass -File build.ps1 -Mode gui
powershell -ExecutionPolicy Bypass -File build.ps1 -Mode cli
```

产物为同目录的 `HDRSnap.exe`（托盘程序）与 `HDRSnapCli.exe`（命令行）。

## 已知限制

- 只捕获主显示器
- 全屏/区域热键可能被其他软件占用；可在托盘菜单 `设置热键…` 里改，或编辑 `config.ini`
- SDR 输出经过高光压缩，画面中极亮的 HDR 高光不会与 HDR 显示器上的亮度完全等同（但真 HDR 副本会保留）

## License

MIT
