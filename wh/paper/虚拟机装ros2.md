### Ros2的学习笔记-虚拟机装ros2

20260117

- [x] 解决rviz结点启动成功但是rviz不渲染成功的bug

  ```bash
  wanhao@DESKTOP-2E56G3B:~$ ros2 launch urdf_tutorial display.launch.py 
  [ERROR] [rviz2-3]: process has died [pid 530, exit code -6, cmd 
  [ERROR] [joint_state_publisher_gui-2]: process has died [pid 528, exit code -6, cmd '/opt/ros/humble/lib/joint_state_publisher_gui/joint_state_publisher_gui --ros-args'].
  ```

> WSL 对 QT+xcb 的支持是**先天缺陷**：
>
> - xcb 是 QT 连接 X Server 的核心协议，WSL 的 Linux 内核缺少对 xcb 的完整实现；
> - 即便修复了所有依赖，也可能因 WSL 的 “虚拟化网络 / 图形层” 限制，无法建立稳定的渲染连接；
> - 你已经尝试了补依赖、配 DISPLAY、启软件渲染等所有常规方案，仍无法解决，说明 WSL 环境已达兼容上限。
>
> ### 最终、最高效的解决方案（放弃 WSL）
>
> **安装 Ubuntu 22.04 虚拟机（VMware/VirtualBox）**，这是 ROS 2 图形工具的 “官方标配环境”：
>
> 1. 虚拟机内的 Ubuntu 是原生 Linux，QT/xcb/OpenGL 渲染无任何兼容问题；
> 2. 安装`ros-humble-desktop-full`后，`rviz2`/`gazebo`一键启动，无需配置任何 DISPLAY/X Server；
> 3. 虚拟机开启 3D 加速后，图形渲染流畅度远高于 WSL。
>
> ### 总结
>
> 1. 本次报错的核心是**WSL 无法连接 Windows 的 VcXsrv，且 QT 的 xcb 插件无法初始化**；
> 2. 最后尝试手动指定 Windows IP + 补全 xcb 依赖，若仍失败，不要再浪费时间；
> 3. 虚拟机是运行 ROS 2 图形工具的唯一稳定方案。
>
> **Ubuntu 22.04 + ROS 2 Humble 虚拟机安装的极简教程**

现在用 VMware Workstation 16 Pro 创建 Ubuntu 22.04 虚拟机（适配 ROS 2 Humble），按以下步骤操作：

### 第一步：准备 Ubuntu 镜像

先下载 Ubuntu 22.04 LTS 镜像（ROS 2 Humble 的推荐系统）：

- 下载地址：[Ubuntu 22.04.3 LTS](https://releases.ubuntu.com/22.04/)
- 选择 “64-bit PC (AMD64) desktop image”，下载 ISO 文件。

### 第二步：创建新虚拟机

1. 点击界面中 “创建新的虚拟机”，选择**典型（推荐）**→ 下一步；

2. 选择 “安装程序光盘映像文件（ISO）”，点击 “浏览” 选择刚才下载的 Ubuntu 22.04 ISO → 下一步；

3. 填写 “全名”“用户名”“密码”（记住密码）→ 下一步；

4. 输入 “虚拟机名称”（比如 “ROS2-Ubuntu22”），选择虚拟机存储位置（建议非 C 盘）→ 下一步；

5. 配置磁盘大小：至少**50GB**（ROS 2+Gazebo 会占较多空间），选择 “将虚拟磁盘存储为单个文件”→ 下一步；

6. 点击 “自定义硬件”，调整配置（关键）：

   - **内存**：至少 4GB（推荐 8GB）；
   - **处理器**：至少 2 核（推荐 4 核）；
   - **显示**：勾选 “加速 3D 图形”，显存设置为 1GB；
   - 其他保持默认 → 关闭 “自定义硬件”→ 完成。

   

### 第三步：安装 Ubuntu 系统

1. 启动虚拟机，等待 Ubuntu 安装界面加载；

   ![image-20260117154405030](C:\Users\wanhao\AppData\Roaming\Typora\typora-user-images\image-20260117154405030.png)

   ![image-20260117154852883](C:\Users\wanhao\AppData\Roaming\Typora\typora-user-images\image-20260117154852883.png)

   #### a. 彻底关闭所有 Windows 虚拟化功能

   - 以**管理员身份**打开 PowerShell。

   - 执行以下命令来关闭所有冲突的虚拟化功能：

     ```powershell
     # 关闭 Hyper-V
     dism.exe /Online /Disable-Feature:Microsoft-Hyper-V /All
     # 关闭 Windows 虚拟机监控程序平台
     dism.exe /Online /Disable-Feature:HypervisorPlatform
     # 关闭虚拟机平台
     dism.exe /Online /Disable-Feature:VirtualMachinePlatform
     ```

     

   - 执行后，按提示**重启电脑**。

   #### b. 验证并关闭 “核心隔离”（关键步骤）

   - 打开 Windows 设置 → 隐私和安全性 → Windows 安全中心 → 设备安全性 → 核心隔离详细信息。
   - 关闭 “内存完整性” 开关（此功能会强制启用 Hyper-V，导致冲突）。
   - 再次**重启电脑**。

   #### c. 检查 VMware 设置

   - 关闭虚拟机，进入 “虚拟机设置” → “处理器”。

   - 确保 “虚拟化引擎” 下的 “虚拟化 Intel VT-x/EPT 或 AMD-V/RVI (V)” 已勾选。

   - 保存设置后，重新启动虚拟机。

     

2. 选择 “Install Ubuntu”→ 语言选 “中文（简体）”→ 继续；

3. 键盘布局选 “汉语”→ 继续；

4. 安装类型选 “正常安装”，勾选 “下载更新”（可选，建议勾）→ 继续；

5. 磁盘分区选 “清除整个磁盘并安装 Ubuntu”（虚拟机磁盘，放心选）→ 现在安装；

6. 选择时区（比如 “上海”）→ 继续；

7. 输入之前设置的用户名和密码→ 继续；

8. 等待安装完成（约 10-20 分钟），安装完成后点击 “现在重启”。

### 第四步：安装 ROS 2 Humble（一键命令）

重启后进入 Ubuntu 桌面，打开终端（快捷键`Ctrl+Alt+T`），执行以下命令：

```bash
# 1. 创建名为install_ros2_humble.sh的脚本文件
nano install_ros2_humble.sh
```

在脚本文件中粘贴。保存并退出 nano 编辑器

```bash
#!/bin/bash
set -e

# ====================== 1. 配置系统编码 ======================
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# ====================== 2. 添加ROS 2软件源 ======================
sudo apt update && sudo apt install -y curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# ====================== 3. 安装ROS 2 Humble完整版 ======================
sudo apt update && sudo apt install -y ros-humble-desktop-full

# ====================== 4. 配置环境变量（自动加载） ======================
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc

# ====================== 5. 安装开发依赖工具 ======================
sudo apt install -y python3-colcon-common-extensions python3-rosdep python3-vcstool
# 避免rosdep init重复执行报错
if [ ! -f /etc/ros/rosdep/sources.list.d/20-default.list ]; then
    sudo rosdep init
else
    echo "rosdep 已初始化，跳过"
fi
rosdep update

# ====================== 6. 安装URDF示例包 ======================
sudo apt install -y ros-humble-urdf-tutorial

# ====================== 安装完成提示 ======================
echo "====================================="
echo "✅ ROS 2 Humble 安装完成！"
echo "👉 测试命令1：rviz2（启动可视化工具）"
echo "👉 测试命令2：gazebo（启动仿真环境）"
echo "👉 测试命令3：ros2 run demo_nodes_cpp talker（运行测试节点）"
echo "====================================="
```

输入完脚本内容后，按以下快捷键操作：

1. 按 `Ctrl + O`（字母 O，不是数字 0）→ 按回车确认保存文件名；
2. 按 `Ctrl + X` 退出 nano 编辑器，回到终端。

问题1：

> 报错 `Temporary failure resolving 'cn.archive.ubuntu.com'` 说明你的 Ubuntu 虚拟机**DNS 解析失败**
>
> ###### 1. 正确修改 DNS（适配 Ubuntu 22.04）
>
> 1. 打开终端，编辑 Netplan 网络配置文件：
>
>    ```bash
>    sudo nano /etc/netplan/01-network-manager-all.yaml
>    ```
>
> 2. 把文件内容改成下面这样（如果有其他内容，替换掉）：
>
>    ```yaml
>    network:
>      version: 2
>      renderer: NetworkManager
>      ethernets:
>        ens33:
>          dhcp4: true
>          nameservers:
>            addresses: [114.114.114.114, 8.8.8.8]
>    ```
>
>    > 注：`ens33` 是默认的网卡名，如果你的虚拟机里不是这个名字，可以用 `ip addr` 命令查看真实的网卡名。
>
> 3. 保存并退出（`Ctrl+O` → 回车 → `Ctrl+X`）。
>
> 4. 应用新的网络配置：
>
>    ```bash
>    sudo netplan apply
>    ```
>
> ------
>
> ###### 2. 验证 DNS 是否生效
>
> 执行这条命令，测试能否正常解析域名：
>
> ```bash
> ping cn.archive.ubuntu.com
> ```
>
> 如果能收到响应，说明 DNS 已经修复成功。
>
> ------
>
> ###### 3. 继续完成后续步骤
>
> DNS 正常后，你就可以执行：
>
> ```bash
> sudo apt update
> sudo apt install -y open-vm-tools open-vm-tools-desktop
> sudo reboot
> ```

问题2

> `ping: Temporary failure in name resolution` 错误，说明你的虚拟机不仅**DNS 解析失败**，很可能连物理网络都没有连通。我们换一个更彻底的方案，直接在 VMware 层面强制修复网络：
>
> ------
>
> ######  第一步：在 VMware 里重置虚拟网络
>
> 1. 关闭你的 Ubuntu 虚拟机。
> 2. 在 VMware 主界面，点击顶部菜单栏的 **“编辑” → “虚拟网络编辑器”**。
> 3. 点击 **“更改设置”**（需要管理员权限）。
> 4. 选中 **“VMnet8 (NAT 模式)”**，然后点击 **“恢复默认设置”**。
> 5. 等待恢复完成后，点击 **“确定”** 保存。
>
> ------
>
> ######  第二步：重新配置虚拟机网络适配器
>
> 1. 右键你的 Ubuntu 虚拟机 → **设置** → **网络适配器**。
> 2. 确保连接方式是 **“NAT”**，并且勾选了 **“启动时连接”**。
> 3. 点击 **“确定”**，然后启动虚拟机。
>
> ------
>
> ######  第三步：在 Ubuntu 里强制获取 IP
>
> 启动虚拟机后，打开终端，执行：
>
> ```bash
> sudo dhclient -r
> sudo dhclient
> ```
>
> 这两条命令会先释放旧的 IP，再重新从 VMware 的 DHCP 服务器获取新的 IP 地址。
>
> ------
>
> ######  第四步：测试网络
>
> 执行以下命令，看能否正常 ping 通外部地址：
>
> ```bash
> ping 114.114.114.114
> ```
>
> 如果能收到响应，说明物理网络已经通了。再测试 DNS 解析：
>
> ```bash
> ping baidu.com
> ```
>
> 如果也能通，说明 DNS 也正常了。

问题3

> `curl: (56) OpenSSL SSL_read: Connection reset by peer` 错误，是**网络连接不稳定 / ROS 官方源访问超时**导致的，换国内 ROS 镜像源就能解决：
>
> ######  第一步：先停止当前脚本，替换 ROS 官方源为国内源
>
> 1. 按 `Ctrl+C` 终止正在运行的脚本；
>
> 2. 打开终端，执行以下命令替换 ROS 源（用清华镜像，速度快且稳定）：
>
> 
>   ```bash
>       # 备份原有ROS源
>   sudo cp /etc/apt/sources.list.d/ros2.list /etc/apt/sources.list.d/ros2.list.bak
> 
>   # 替换为清华ROS 2镜像源
>       echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
>
>       # 更新软件源缓存
>   sudo apt update
>   ```
>
>    ######  第二步：重新运行 ROS 2 安装脚本
>
> ```bash
> ./install_ros2_humble.sh
> ```
>    
> 这次会从清华镜像源下载，不会再出现 SSL 连接重置的问题。
> 
>    ######  补充说明：
> 
> - 报错原因：ROS 官方源在国外，国内访问容易超时 / 断连，替换成清华 / 阿里云等国内镜像源是解决这类问题的核心方案；
> 
>- 如果仍提示 “密钥错误”，执行以下命令重新导入 ROS 密钥：
>    
>
>   ```bash
>  sudo curl -sSL https://mirrors.tuna.tsinghua.edu.cn/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
>   sudo apt update
>   ```
> 
>------
> 
>###### 总结
> 
>1. 核心问题是**ROS 官方源访问不稳定**，替换为清华镜像源即可解决；
> 2. 先备份原有源→替换为国内源→更新缓存→重新运行脚本；
>3. 国内镜像源下载速度更快，也能避免 SSL 连接类报错。