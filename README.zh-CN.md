# DisplayAudio

[English](README.md)

DisplayAudio 是一款面向 x86_64 的 Lilu 插件，为兼容的 HDMI 和 DisplayPort 音频输出提供 macOS 原生音量和静音控制。将显示器选为声音输出后，可以通过系统音量滑块、静音控件和音量键调节播放音量。

## 工作原理

1. DisplayAudio 将已加载的 AppleGFXHDA 和 IOAudioFamily 与已知驱动配置匹配，并在安装音频钩子前校验 AppleGFXHDA 的音频调用点。
2. 对于使用可混音线性 PCM 音频流且满足条件的 HDMI 或 DisplayPort 输出，插件通过 IOAudioFamily 注册音量控件，并在可用时注册静音控件。已有原生音量控件或支持硬件音量的输出由 macOS 管理。
3. AppleGFXHDA 完成音频样本处理后，DisplayAudio 在驱动的中间缓冲区中对当前显示器的 PCM 样本进行衰减或清零。控件调节的是 Mac 发送的音频流，显示器的硬件音量设置不受影响。

DisplayAudio 需要 **Lilu 1.7.3 或更高版本**。下表中的驱动配置只匹配指定的驱动二进制文件。即使 macOS 大版本相同，系统更新也可能更换驱动文件。

| macOS 系列 | Darwin | 对应构建 | 音频格式 | 启动参数 |
| --- | --- | --- | --- | --- |
| Tahoe | 25.6 | `25G332` | 可混音 LPCM，最多 8 声道 | 无需 DisplayAudio 专用参数 |
| Sequoia | 24.6 | `24H232` | 可混音双声道 LPCM | `-dacompat` |
| Sonoma | 23.6 | `23J631` | 可混音双声道 LPCM | `-dacompat` |

## 启动参数

将参数加入 OpenCore 配置中 `NVRAM → Add → 7C436110-AB2A-4BBB-A880-FE41995C9F82` 下的 `boot-args`，然后重启。下表列出 DisplayAudio 专用参数。数值参数使用十进制整数，输入无效时使用默认值。

| 参数 | 默认值 | 作用 |
| --- | --- | --- |
| `-daoff` | 关闭 | 在本次启动中禁用 DisplayAudio。 |
| `-dacompat` | 关闭 | 允许加载匹配的 Sonoma 和 Sequoia 驱动配置。驱动校验仍然生效。 |
| `-dacheck` | 关闭 | 校验 AppleGFXHDA 二进制文件、所需符号和 PCM 调用点。此模式下不安装音频钩子，也不添加音量或静音控件。 |
| `-dadebug` | 关闭 | 在观察窗口尚未结束且未达到目标次数时，每次轮询记录匹配的 PCM 调用次数。 |
| `daout=VALUE` | `auto` | 为观察日志标注所声明的输出类型。可选 `auto`、`speaker`、`headphone`、`line`、`jack`。该标签不影响音频路由或设备选择。 |
| `dadelay=N` | `0` | 安装音频钩子后，等待 0–300 秒再开始观察 PCM 调用。 |
| `dawindow=N` | `180` | 将 PCM 观察窗口设为 5–600 秒。 |
| `dapoll=N` | `5` | 将观察状态的轮询间隔设为 1–30 秒。 |
| `dahits=N` | `1` | 将目标匹配次数设为 1–10,000 次 PCM 调用。 |
| `dach=N` | `2` | 仅统计输入和输出声道数均为 N 的 PCM 调用，范围为 1–8。 |
| `daframes=N` | `1` | 仅统计至少包含 N 个音频帧的 PCM 调用，范围为 1–65,536。 |

观察参数影响诊断日志。观察窗口结束后，音量和静音控制仍会继续工作。

## 致谢

- 感谢 vit9696、Acidanthera 和 Lilu 贡献者提供插件 API 和内核补丁机制。[Lilu](https://github.com/acidanthera/Lilu)
- 感谢 MacKernelSDK 贡献者提供内核头文件。[MacKernelSDK](https://github.com/acidanthera/MacKernelSDK)
- 感谢 OpenCore 贡献者提供启动时加载和配置 kext 的支持。[OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- 感谢 Apple 提供 macOS 及其音频驱动体系。
