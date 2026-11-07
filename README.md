<div align="center">

  <img src="./assets/banner.svg" width="100%" alt="DualBoot & Steam & Antigravity Guide"/>

  <br><br>

  <p align="center">
    <img src="https://img.shields.io/badge/Guide-DualBoot_%26_Steam-1793d1?style=for-the-badge&logo=arch-linux&logoColor=white" />
    <img src="https://img.shields.io/badge/Antigravity-中文汉化补丁-FF69B4?style=for-the-badge&logo=google&logoColor=white" />
    <img src="https://img.shields.io/badge/Steam-NTFS跨盘避坑-1A9FFF?style=for-the-badge&logo=steam&logoColor=white" />
    <img src="https://img.shields.io/badge/Tested-Arch_%26_Nyarch-008080?style=for-the-badge&logo=gnubash&logoColor=white" />
    <img src="https://img.shields.io/badge/Author-shenchuchu91--hub-ffb6c1?style=for-the-badge" />
  </p>

  <p align="center">
    <i>🌸 一部真真切切从小白踩坑走过来的实战血泪实录：把踩过的每一个坑填平，让后来者一路通途 🌸</i>
  </p>

</div>

---

## 📑 避坑导航 (Index)

1. [🌐 反重力 (Google Antigravity) 简体中文汉化全攻略](#1-反重力-google-antigravity-简体中文汉化全攻略)
2. [🪟 Windows 转 Arch 双系统必踩大坑与完美化解](#2-windows-转-arch-双系统必踩大坑与完美化解)
   - [时间相差 8 小时 (RTC vs UTC)](#坑点-1双系统时间错乱相差-8-小时)
   - [Windows 快速启动导致 Linux 硬盘只读](#坑点-2windows-快速启动导致-linux-分区被锁只读)
   - [GRUB 引导找不到 Windows 启动项](#坑点-3grub-启动菜单检测不到-windows)
   - [EFI 分区 (ESP) 空间不足与分区避坑](#坑点-4efi-系统分区-esp-容量爆满)
3. [🎮 Steam 游戏在 Linux / 双系统下的极限避坑](#3-steam-游戏在-linux--双系统下的极限避坑)
   - [跨盘读取 Windows NTFS 分区的 Steam 游戏库 (Proton 秒退全解)](#坑点-1直接玩-windows-ntfs-盘里的-steam-游戏-proton-秒退)
   - [Steam 界面与游戏中文方块豆腐块 (Tofu)](#坑点-2steam-中文全是方块豆腐块)
   - [游戏手柄 / 控制器无法识别](#坑点-3手柄-xbox--ps5-插上无法识别)
   - [着色器缓存 (Shader Pre-caching) 塞满硬盘](#坑点-4着色器预缓存疯狂吃磁盘)
4. [🛠️ 细小但致命的日常隐藏暗坑](#4-细小但致命的日常隐藏暗坑)
   - [Fcitx5 输入法在 Electron / Steam 中无法打字](#暗坑-1fcitx5-输入法在-steam--electron--antigravity-打不出中文)
   - [双显卡笔记本（Intel + NVIDIA）偷跑核显](#暗坑-2游戏本双显卡默认使用核显偷跑游戏)
   - [GRUB 重新生成配置文件时的语言环境报错](#暗坑-3grub-mkconfig-因中文环境报错中断)

---

## 1. 🌐 反重力 (Google Antigravity) 简体中文汉化全攻略

作为 Google 专为下一代多智能体协作成型打造的 AI 开发平台，**Google Antigravity (AGY)** 在 Linux 环境下安装后默认通常是全英文界面。很多国内同好找不到中文化选项，其实它底层基于现代前端架构，中文化非常简单丝滑：

### 方法一：软件内极速一键切换（推荐）
1. 启动 Antigravity 软件；
2. 按下快捷键 <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> 调出全局命令面板；
3. 输入并点击：`Configure Display Language`（配置显示语言）；
4. 在下拉列表中选择 **`zh-cn`（中文简体）**；
   > 💡 *如果列表内只有 English，点击下面的 `Install Additional Languages...`，在左侧扩展商店搜索并安装 `Chinese (Simplified) Language Pack`（中文简体语言包）即可。*
5. 软件提示 `Restart`（重启软件），点击重启，即可获得纯正熟悉的中文简体界面！

### 方法二：全局启动配置文件与快捷方式固化
如果你希望每次从应用菜单或终端启动都强制使用简体中文，可以在桌面快捷方式或环境参数中指定：

```bash
# 启动时注入中文语言环境变量
antigravity --lang=zh-CN
```

或者检查用户的桌面文件 `/usr/share/applications/antigravity.desktop`，将其中的 `Exec=antigravity %U` 修改为：
```ini
Exec=env LANG=zh_CN.UTF-8 antigravity --lang=zh-CN %U
```

---

## 2. 🪟 Windows 转 Arch 双系统必踩大坑与完美化解

### 坑点 1：双系统时间错乱，相差 8 小时！
- **血泪现象**：在 Linux 里时间是准的，一重启进 Windows 时间直接慢了 8 小时；在 Windows 调准后，进 Linux 又快了 8 小时！
- **技术原因**：Windows 默认将主板硬件 RTC 时钟当作**本地时间**，而 Linux 默认将主板时钟当作**格林威治世界标准时间 (UTC)**。
- **永久解决（在 Linux 终端执行这一行即可）**：
  ```bash
  timedatectl set-local-rtc 1 --adjust-system-clock
  ```
  *(告诉 Linux 将硬件时钟保持为本地时间，双系统时间从此彻底同步！)*

---

### 坑点 2：Windows 快速启动导致 Linux 分区被锁、只读！
- **血泪现象**：进入 Linux 想往 Windows 盘（D 盘/游戏盘）存文件，提示 `Read-only file system`（只读），甚至无法挂载！
- **技术原因**：Windows 8/10/11 的“快速启动”本质上是一种**混合休眠**。关机时它并没有真正断电卸载硬盘，而是给 NTFS 驱动打上了锁定标记以加速下次开机。
- **永久解决**：
  1. 重启进入 Windows，打开「控制面板」 -> 「硬件和声音」 -> 「电源选项」；
  2. 点击左侧「选择电源按钮的功能」 -> 点击「更改当前不可用的设置」；
  3. **取消勾选「启用快速启动 (推荐)」**，点击保存修改；
  4. 顺手以管理员身份打开 CMD 运行：`powercfg -h off`（彻底关闭休眠）。

---

### 坑点 3：GRUB 启动菜单检测不到 Windows！
- **血泪现象**：安装好 Arch / Nyarch 之后，开机直接进 Linux，原来装好的 Windows 启动项不见了！
- **技术原因**：新版 GRUB 出于安全策略，默认禁用了外部系统探测器 `os-prober`。
- **永久解决**：
  ```bash
  # 1. 安装系统探测与 NTFS 驱动工具
  sudo pacman -S os-prober ntfs-3g

  # 2. 编辑 GRUB 配置文件
  sudo nano /etc/default/grub
  # 找到这行并确保去掉注释设置为 false：
  # GRUB_DISABLE_OS_PROBER=false

  # 3. 重新生成引导菜单 (关键命令)
  LANGUAGE=en_US.UTF-8 LANG=en_US.UTF-8 sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```
  *(生成时终端会打印：`Found Windows Boot Manager on /dev/nvme0n1p1@/EFI/Microsoft/Boot/bootmgfw.efi`，重启就能看到双系统选择菜单了！)*

---

### 坑点 4：EFI 系统分区 (ESP) 容量爆满
- **血泪现象**：安装内核或者更新 `linux` / `linux-zen` 时，提示硬盘空间不足，更新失败。
- **技术原因**：很多原厂 Windows 笔记本自带的 ESP 分区只有可怜的 100MB，装一个 Windows 引导就占了一大半，根本塞不下 Linux 的 Initramfs 内核镜像。
- **解决方案**：在给 Linux 分区时，在固态硬盘上**单独划分一个 512MB 到 1GB 的独立 ESP 分区（FAT32 格式）**挂载到 `/boot`，不要跟 Windows 挤那区区 100MB！

---

## 3. 🎮 Steam 游戏在 Linux / 双系统下的极限避坑

### 坑点 1：直接玩 Windows NTFS 盘里的 Steam 游戏，Proton 秒退！
- **血泪现象**：在 Linux Steam 里把 Windows 下已经下好的上百 G 游戏盘（NTFS）添加为游戏库，点击“开始”，转两圈直接秒退！
- **技术原因**：Proton 运行 Windows 游戏时，必须在游戏的 `compatdata` 目录下创建大量虚拟注册表与 Linux 特有的符号链接（Symlink）。而默认挂载的 NTFS 分区不支持这些权限操作，导致 Proton 直接崩溃。
- **终极解决方案**：
  #### 方案 A：正确的 NTFS 挂载参数（写进 `/etc/fstab`）
  ```fstab
  UUID=你的NTFS盘UUID  /mnt/Games  ntfs-3g  uid=1000,gid=1000,fmask=0022,dmask=0022,windows_names,nofail  0  0
  ```
  #### 方案 B：最稳妥的软链接大法（推荐）
  将游戏本体留在 Windows 盘，但把 Proton 运行环境定向到 Linux 原生 `ext4` 分区：
  ```bash
  # 创建 Linux 本地的 compatdata 目录
  mkdir -p ~/.local/share/Steam/steamapps/compatdata
  # 链接目标游戏的兼容层，彻底解决秒退！
  ```

---

### 坑点 2：Steam 中文全是方块豆腐块！
- **血泪现象**：Steam 打开后所有中文字全变成了空心小方块 `□□□`。
- **技术原因**：缺少支持 CJK（中日韩）的开源字体。
- **解决命令**：
  ```bash
  sudo pacman -S noto-fonts-cjk ttf-liberation wqy-microhei
  ```
  重启 Steam，字体瞬间清秀完整！

---

### 坑点 3：手柄 (Xbox / PS5) 插上无法识别
- **解决命令**：安装专门针对 Steam 的 Udev 控制器规则包：
  ```bash
  sudo pacman -S steam-devices
  sudo udevadm control --reload-rules && sudo udevadm trigger
  ```

---

### 坑点 4：着色器预缓存疯狂吃磁盘
- **解决办法**：Steam 会在后台预下载几百 MB 到几 GB 的 Vulkan 着色器。进入 Steam -> 「设置」 -> 「着色器预缓存」，可以按需关闭或定期清理，释放固态硬盘宝贵空间。

---

## 4. 🛠️ 细小但致命的日常隐藏暗坑

### 暗坑 1：Fcitx5 输入法在 Steam / Electron / Antigravity 打不出中文
- **解决办法**：在 `~/.config/environment.d/im.conf` 或 `/etc/environment` 中加入：
  ```ini
  GTK_IM_MODULE=fcitx
  QT_IM_MODULE=fcitx
  XMODIFIERS=@im=fcitx
  SDL_IM_MODULE=fcitx
  ```
  针对 Electron / Chrome 类应用，启动时可追加参数：`--enable-wayland-ime`。

---

### 暗坑 2：游戏本双显卡默认使用核显偷跑游戏
- **解决办法**：游戏本（例如带有 RTX 5060 的机器）在运行 Steam 游戏或外部 exe 时，必须显式声明 Prime 变量。
- 直接使用我们自研的工具：[`shenchuchu91-hub/moe-game-launcher`](https://github.com/shenchuchu91-hub/moe-game-launcher)，一条命令 `moe 游戏.exe` 自动锁定独显渲染！

---

### 暗坑 3：`grub-mkconfig` 因中文环境报错中断
- **血泪经验**：在某些中文终端编码下直接跑 `sudo grub-mkconfig`，可能会由于字符集转换失败而异常中断。
- **优雅写法**（这也是我在 Fish 配置里设置的别名）：
  ```bash
  LANGUAGE=en_US.UTF-8 LANG=en_US.UTF-8 sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```

---

<div align="center">

🌸 **特别致谢** 🌸  
感谢所有在 Linux 双系统与游戏道路上先行探索的极客前辈与社区同好！  
希望这本避坑实录能为你节省几十个小时的查报错时间。

<sub>Made with 💖 by <b>shenchuchu91-hub</b> · 献给所有在 Arch 上探索的拓荒者</sub>

</div>
