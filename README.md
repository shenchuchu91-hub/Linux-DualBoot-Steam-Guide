<div align="center">

  <img src="./assets/banner.svg" width="100%" alt="DualBoot & Steam & Antigravity Guide"/>

  <br><br>

  <p align="center">
    <img src="https://img.shields.io/badge/Guide-DualBoot_%26_Steam-1793d1?style=for-the-badge&logo=arch-linux&logoColor=white" />
    <img src="https://img.shields.io/badge/Antigravity-中文语言配置-FF69B4?style=for-the-badge&logo=google&logoColor=white" />
    <img src="https://img.shields.io/badge/Steam-NTFS跨盘挂载-1A9FFF?style=for-the-badge&logo=steam&logoColor=white" />
    <img src="https://img.shields.io/badge/Platform-Arch_%26_Nyarch-008080?style=for-the-badge&logo=gnubash&logoColor=white" />
    <img src="https://img.shields.io/badge/Author-shenchuchu91--hub-ffb6c1?style=for-the-badge" />
  </p>

  <p align="center">
    <i>面向 Windows + Arch Linux 双系统环境的配置指南、Steam 跨盘兼容与 Google Antigravity 语言设置实践</i>
  </p>

</div>

---

## 📑 目录索引 (Index)

1. [🌐 Google Antigravity (AGY) 简体中文界面配置](#1-google-antigravity-agy-简体中文界面配置)
2. [🪟 Windows + Arch 双系统常见问题排查与修复](#2-windows--arch-双系统常见问题排查与修复)
   - [时间相差 8 小时 (RTC 本地时钟与 UTC 同步)](#1-双系统时间偏差-8-小时)
   - [Windows 快速启动导致 NTFS 分区只读锁死](#2-windows-快速启动导致-ntfs-分区被锁定为只读)
   - [GRUB 引导菜单未检测到 Windows 启动项](#3-grub-启动菜单未检测到-windows)
   - [EFI 系统分区 (ESP) 容量规划建议](#4-efi-系统分区-esp-容量规划)
3. [🎮 Steam 游戏在 Linux / 双系统下的兼容配置](#3-steam-游戏在-linux--双系统下的兼容配置)
   - [跨盘读取 Windows NTFS 分区的 Steam 游戏库 (Proton 启动闪退排查)](#1-跨盘读取-windows-ntfs-分区的-steam-游戏库-proton-启动闪退排查)
   - [Steam 界面与游戏内中文字体方块乱码处理](#2-steam-界面与游戏中文字体方块乱码处理)
   - [游戏手柄 / 控制器 (Xbox / PS5) 无法识别处理](#3-游戏手柄-xbox--ps5-无法识别处理)
   - [着色器预缓存 (Shader Pre-caching) 磁盘空间管理](#4-着色器预缓存-shader-pre-caching-磁盘空间管理)
4. [🛠️ 系统日常细节问题与实用排查](#4-系统日常细节问题与实用排查)
   - [Fcitx5 输入法在 Electron / Steam 中无法打字](#1-fcitx5-输入法在-steam--electron--antigravity-中无法输入中文)
   - [双显卡笔记本混合调度与独显渲染设置](#2-双显卡笔记本混合调度与独显渲染设置)
   - [GRUB 生成配置文件时的字符集环境处理](#3-grub-mkconfig-因本地字符集报错中断)

---

## 1. 🌐 Google Antigravity (AGY) 简体中文界面配置

**Google Antigravity (AGY)** 在 Linux 环境下安装后默认通常为英文界面。中文化设置方式如下：

### 方法一：软件内命令面板切换（推荐）
1. 启动 Antigravity 软件；
2. 按下快捷键 <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> 打开命令面板；
3. 输入并选择：`Configure Display Language`；
4. 在下拉列表中选择 **`zh-cn`（中文简体）**；
   > 💡 *若列表内未包含中文，可点击下方的 `Install Additional Languages...`，在插件市场搜索安装 `Chinese (Simplified) Language Pack`。*
5. 软件提示重启，点击 `Restart` 完成切换。

### 方法二：全局启动参数与桌面快捷方式配置
通过环境参数启动：
```bash
# 启动时注入语言环境变量
antigravity --lang=zh-CN
```

或修改 `/usr/share/applications/antigravity.desktop` 中的 `Exec` 行：
```ini
Exec=env LANG=zh_CN.UTF-8 antigravity --lang=zh-CN %U
```

---

## 2. 🪟 Windows + Arch 双系统常见问题排查与修复

### 1. 双系统时间偏差 8 小时
- **问题表现**：在 Linux 与 Windows 之间切换时，系统时间始终相差 8 小时；
- **原因分析**：Windows 默认将主板硬件 RTC 时钟视作本地时间（Local Time），而 Linux 默认视作世界协调时间（UTC）；
- **解决方案**：在 Linux 终端中执行以下命令，统一以硬件本地时间同步：
  ```bash
  timedatectl set-local-rtc 1 --adjust-system-clock
  ```

---

### 2. Windows 快速启动导致 NTFS 分区被锁定为只读
- **问题表现**：在 Linux 下尝试向 Windows NTFS 分区写入数据时，报错 `Read-only file system`；
- **原因分析**：Windows 的“快速启动”采用了混合休眠机制，关机时未完全卸载文件系统并保留了锁定标记；
- **解决方案**：
  1. 进入 Windows「控制面板」 -> 「硬件和声音」 -> 「电源选项」；
  2. 点击「选择电源按钮的功能」 -> 点击「更改当前不可用的设置」；
  3. 取消勾选「启用快速启动 (推荐)」并保存；
  4. 以管理员身份运行终端并执行：`powercfg -h off`（关闭休眠文件）。

---

### 3. GRUB 启动菜单未检测到 Windows
- **问题表现**：安装 Linux 后，GRUB 引导菜单中没有 Windows 启动项；
- **原因分析**：GRUB 默认禁用了第三方系统探测工具 `os-prober`；
- **解决方案**：
  ```bash
  # 1. 安装系统探测与 NTFS 驱动工具
  sudo pacman -S os-prober ntfs-3g

  # 2. 编辑 GRUB 配置文件
  sudo nano /etc/default/grub
  # 确保取消注释或设置如下行：
  # GRUB_DISABLE_OS_PROBER=false

  # 3. 重新生成引导菜单
  LANGUAGE=en_US.UTF-8 LANG=en_US.UTF-8 sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```

---

### 4. EFI 系统分区 (ESP) 容量规划
- **问题表现**：更新 Linux 内核或生成 Initramfs 镜像时提示磁盘空间不足；
- **原因分析**：部分出厂预装 Windows 的机器自带 ESP 分区仅 100MB，不足以同时容纳多个 Linux 内核镜像；
- **规划建议**：在给 Linux 分区时，建议单独划出一个 512MB~1GB 的独立 ESP 分区（FAT32 格式）挂载至 `/boot`。

---

## 3. 🎮 Steam 游戏在 Linux / 双系统下的兼容配置

### 1. 跨盘读取 Windows NTFS 分区的 Steam 游戏库 (Proton 启动闪退排查)
- **问题表现**：直接将 Windows 下的 NTFS 目录作为 Steam 游戏库时，部分 Windows 游戏在 Proton 环境下启动即闪退；
- **原因分析**：Proton 依赖 Linux 符号链接与特定的文件权限机制来维护 `compatdata` 运行时前缀，而标准挂载下的 NTFS 分区不原生支持这些属性；
- **解决方案**：
  #### 方案 A：配置 `/etc/fstab` 挂载参数
  ```fstab
  UUID=你的NTFS分区UUID  /mnt/Games  ntfs-3g  uid=1000,gid=1000,fmask=0022,dmask=0022,windows_names,nofail  0  0
  ```
  #### 方案 B：重定向 compatdata 软链接（推荐）
  将游戏本体存放在 NTFS 分区，但将运行时兼容层链接至 Linux 原生的 `ext4` 或 `btrfs` 分区：
  ```bash
  mkdir -p ~/.local/share/Steam/steamapps/compatdata
  ```

---

### 2. Steam 界面与游戏中文字体方块乱码处理
- **问题表现**：Steam 界面或某些 Proton 游戏中的中文字符显示为空心方块 `□□□`；
- **原因分析**：系统缺乏支持 CJK（中日韩）的通用开源字体；
- **解决命令**：
  ```bash
  sudo pacman -S noto-fonts-cjk ttf-liberation wqy-microhei
  ```

---

### 3. 游戏手柄 (Xbox / PS5) 无法识别处理
- **解决命令**：安装并加载官方针对控制器设备的 udev 规则：
  ```bash
  sudo pacman -S steam-devices
  sudo udevadm control --reload-rules && sudo udevadm trigger
  ```

---

### 4. 着色器预缓存 (Shader Pre-caching) 磁盘空间管理
- **说明**：Steam 会为 Vulkan 游戏后台下载预编译着色器缓存。进入 Steam -> 「设置」 -> 「着色器预缓存」，可按需调整或清理已卸载游戏的残留缓存。

---

## 4. 🛠️ 系统日常细节问题与实用排查

### 1. Fcitx5 输入法在 Steam / Electron / Antigravity 中无法输入中文
- **解决方案**：在 `~/.config/environment.d/im.conf` 或 `/etc/environment` 中配置环境变量：
  ```ini
  GTK_IM_MODULE=fcitx
  QT_IM_MODULE=fcitx
  XMODIFIERS=@im=fcitx
  SDL_IM_MODULE=fcitx
  ```
  针对 Electron 应用，可增加 `--enable-wayland-ime` 启动参数。

---

### 2. 双显卡笔记本混合调度与独显渲染设置
- **说明**：搭载 NVIDIA 独显的混合显卡笔记本在运行 Steam 游戏时，若未显式指定渲染设备，可能默认调用核心显卡；
- **解决方案**：可通过注入 Prime 环境变量（或借助 [`moe-game-launcher`](https://github.com/shenchuchu91-hub/moe-game-launcher) 工具）实现独立显卡委托渲染。

---

### 3. `grub-mkconfig` 因本地字符集报错中断
- **实践经验**：在部分中文环境终端下直接执行 `sudo grub-mkconfig` 时，可能出现字符集编码转换中断。建议在命令前临时声明英文环境：
  ```bash
  LANGUAGE=en_US.UTF-8 LANG=en_US.UTF-8 sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```

---

<div align="center">

🌸 **版权与安全规范** 🌸  
本项目基于 [MIT License](LICENSE) 开源发布。  
涉及系统引导与磁盘分区调整存在操作风险，操作前请参阅 [全局免责声明与通用安全规范 (Disclaimer)](https://github.com/shenchuchu91-hub/disclaimer)。

<sub>Made with 💖 by <b>shenchuchu91-hub</b></sub>

</div>
