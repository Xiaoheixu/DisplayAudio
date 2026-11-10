# DisplayAudio

[English](#english) · [简体中文](#简体中文)

## English

DisplayAudio is an x86_64 Lilu plug-in that adds native macOS volume and mute controls for compatible HDMI and DisplayPort audio outputs. When the display is selected as the sound output, macOS can adjust its playback level through the system volume slider, mute control, and volume keys.

### How it works

1. DisplayAudio matches the loaded AppleGFXHDA and IOAudioFamily binaries against known driver profiles. It checks the AppleGFXHDA audio call site before installing the audio hook.
2. For an eligible HDMI or DisplayPort endpoint with a mixable linear PCM stream, the plug-in registers volume and, where available, mute controls with IOAudioFamily. It leaves endpoints with native volume controls or hardware volume support to macOS.
3. After AppleGFXHDA processes the samples, DisplayAudio attenuates or zeros PCM samples in the driver's intermediate buffer for the selected display. The controls adjust the audio stream sent by the Mac. They do not change the monitor's hardware volume setting.

DisplayAudio requires Lilu **1.7.3 or later**. The profiles below are specific to the listed driver binaries. A system update can change those binaries, even within the same macOS release.

| macOS family | Darwin | Reference build | Audio | Boot argument |
| --- | --- | --- | --- | --- |
| Tahoe | 25.6 | `25G332` | Mixable LPCM, up to 8 channels | No DisplayAudio argument required |
| Sequoia | 24.6 | `24H232` | Mixable stereo LPCM | `-dacompat` |
| Sonoma | 23.6 | `23J631` | Mixable stereo LPCM | `-dacompat` |

### Boot arguments

Add these to OpenCore's `boot-args` under `NVRAM → Add → 7C436110-AB2A-4BBB-A880-FE41995C9F82` and reboot. The table covers the arguments specific to DisplayAudio. Numeric arguments use decimal integers and fall back to their defaults if the supplied values are invalid.

| Argument | Default | Effect |
| --- | --- | --- |
| `-daoff` | Off | Disables DisplayAudio for the current boot. |
| `-dacompat` | Off | Allows matching Sonoma and Sequoia driver profiles to load. Driver checks still apply. |
| `-dacheck` | Off | Checks the AppleGFXHDA binary, required symbols, and PCM call site. In this mode DisplayAudio does not install the audio hook or add controls. |
| `-dadebug` | Off | Logs matching PCM call counts at each poll while the observation window is still open and the target has not been reached. |
| `daout=VALUE` | `auto` | Labels the declared outlet in observation logs. Accepts `auto`, `speaker`, `headphone`, `line`, or `jack`. This label has no effect on routing or device selection. |
| `dadelay=N` | `0` | Waits 0–300 seconds after the hook is installed before starting PCM observation. |
| `dawindow=N` | `180` | Sets the PCM observation window to 5–600 seconds. |
| `dapoll=N` | `5` | Sets the observation status polling interval to 1–30 seconds. |
| `dahits=N` | `1` | Sets the target to 1–10,000 matching PCM calls. |
| `dach=N` | `2` | Matches PCM calls whose input and output channel counts both equal N, from 1 to 8. |
| `daframes=N` | `1` | Matches PCM calls with at least N audio frames, from 1 to 65,536. |

The observation settings affect diagnostic reporting. Volume and mute continue to work after the observation window closes.

### Credits

- vit9696, Acidanthera, and the Lilu contributors for the plug-in API and kernel patching. [Lilu](https://github.com/acidanthera/Lilu)
- The MacKernelSDK contributors for the kernel headers. [MacKernelSDK](https://github.com/acidanthera/MacKernelSDK)
- The OpenCore contributors for boot-time kext loading and configuration. [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- Apple for macOS and its audio driver stack.

## 简体中文

DisplayAudio 是一款面向 x86_64 的 Lilu 插件，为兼容的 HDMI 和 DisplayPort 音频输出提供 macOS 原生音量和静音控制。将显示器选为声音输出后，可以通过系统音量滑块、静音控件和音量键调节播放音量。

### 工作原理

1. DisplayAudio 将已加载的 AppleGFXHDA 和 IOAudioFamily 与已知驱动配置匹配，并在安装音频钩子前校验 AppleGFXHDA 的音频调用点。
2. 对于使用可混音线性 PCM 音频流且满足条件的 HDMI 或 DisplayPort 输出，插件通过 IOAudioFamily 注册音量控件，并在可用时注册静音控件。已有原生音量控件或支持硬件音量的输出由 macOS 管理。
3. AppleGFXHDA 完成音频样本处理后，DisplayAudio 在驱动的中间缓冲区中对当前显示器的 PCM 样本进行衰减或清零。控件调节的是 Mac 发送的音频流，显示器的硬件音量设置不受影响。

DisplayAudio 需要 **Lilu 1.7.3 或更高版本**。下表中的驱动配置只匹配指定的驱动二进制文件。即使 macOS 大版本相同，系统更新也可能更换驱动文件。

| macOS 系列 | Darwin | 对应构建 | 音频格式 | 启动参数 |
| --- | --- | --- | --- | --- |
| Tahoe | 25.6 | `25G332` | 可混音 LPCM，最多 8 声道 | 无需 DisplayAudio 专用参数 |
| Sequoia | 24.6 | `24H232` | 可混音双声道 LPCM | `-dacompat` |
| Sonoma | 23.6 | `23J631` | 可混音双声道 LPCM | `-dacompat` |

### 启动参数

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

### 致谢

- 感谢 vit9696、Acidanthera 和 Lilu 贡献者提供插件 API 和内核补丁机制。[Lilu](https://github.com/acidanthera/Lilu)
- 感谢 MacKernelSDK 贡献者提供内核头文件。[MacKernelSDK](https://github.com/acidanthera/MacKernelSDK)
- 感谢 OpenCore 贡献者提供启动时加载和配置 kext 的支持。[OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- 感谢 Apple 提供 macOS 及其音频驱动体系。
