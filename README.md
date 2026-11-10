# DisplayAudio

[简体中文](README.zh-CN.md)

<a name="english"></a>

DisplayAudio is an x86_64 Lilu plug-in that adds native macOS volume and mute controls for compatible HDMI and DisplayPort audio outputs. When the display is selected as the sound output, macOS can adjust its playback level through the system volume slider, mute control, and volume keys.

## How it works

1. DisplayAudio matches the loaded AppleGFXHDA and IOAudioFamily binaries against known driver profiles. It checks the AppleGFXHDA audio call site before installing the audio hook.
2. For an eligible HDMI or DisplayPort endpoint with a mixable linear PCM stream, the plug-in registers volume and, where available, mute controls with IOAudioFamily. It leaves endpoints with native volume controls or hardware volume support to macOS.
3. After AppleGFXHDA processes the samples, DisplayAudio attenuates or zeros PCM samples in the driver's intermediate buffer for the selected display. The controls adjust the audio stream sent by the Mac. They do not change the monitor's hardware volume setting.

DisplayAudio requires Lilu **1.7.3 or later**. The profiles below are specific to the listed driver binaries. A system update can change those binaries, even within the same macOS release.

| macOS family | Darwin | Reference build | Audio | Boot argument |
| --- | --- | --- | --- | --- |
| Tahoe | 25.6 | `25G332` | Mixable LPCM, up to 8 channels | No DisplayAudio argument required |
| Sequoia | 24.6 | `24H232` | Mixable stereo LPCM | `-dacompat` |
| Sonoma | 23.6 | `23J631` | Mixable stereo LPCM | `-dacompat` |

## Boot arguments

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

## Credits

- vit9696, Acidanthera, and the Lilu contributors for the plug-in API and kernel patching. [Lilu](https://github.com/acidanthera/Lilu)
- The MacKernelSDK contributors for the kernel headers. [MacKernelSDK](https://github.com/acidanthera/MacKernelSDK)
- The OpenCore contributors for boot-time kext loading and configuration. [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- Apple for macOS and its audio driver stack.
