###  RViz2（ROS2 Humble）部署+配置流程总结

####  一、整体流程脉络 

>基于 **Windows 10/11 + WSL Ubuntu 22.04** 环境，完成了 ROS2 Humble 的安装（含 RViz2），并实现了 “点云可视化→无人机模型加载→目标搜索 + 路径规划可视化”，核心阶段为：
>
>1. WSL 环境搭建 → 2. ROS2 Humble 安装（含 RViz2） → 3. RViz2 基础配置（点云可视化） 

#### 二、全流程问题&解决方案（按阶段梳理）

 ##### 阶段1：WSL 环境搭建

+ Windows 端（管理员权限 PowerShell 操作）

  - 启用 WSL 和虚拟机功能（执行后必须重启电脑 ）

  ```powershell
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```

+ 安装 WSL2 内核更新包


+ 设置 WSL2 为默认版本

     ```powershell
     wsl --set-default-version 2
     ```


+ 应用商店安装 Ubuntu 22.04
   - 打开微软应用商店，搜索 **Ubuntu 22.04 LTS**
   - 点击 “获取”，等待安装完成后启动
+ Ubuntu 22.04 初始化（首次启动后）
	
	- 按提示设置**用户名**和**密码**
  
  - 执行系统更新（后续 ROS2 安装基础）
  
   ```
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y curl git  # 装基础工具
   ```
+ 验证安装（PowerShell 执行）

    ```powershell
    wsl -l -v
    ```
    **正常输出**（VERSION 为 2 即成功）：

    ```
    NAME            STATE           VERSION
    * Ubuntu-22.04    Running         2
    ```

    

    ###### 尝试的方案和失败原因
    
    | 尝试方案                                    | 操作核心                                                     | 失败原因 / 踩坑点                                            |
    | ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | 1. WSL2（Ubuntu22.04）装 ROS2 Humble 桌面版 | 加清华源→apt update→apt install ros-humble-desktop           | ① dpkg 数据库损坏，Python 依赖包（catkin-pkg/rospkg）处理失败；② 完整桌面版依赖过多，冲突累积导致安装终止 |
    | 2. WSL2 装 ROS2 极简版（仅 rviz2）          | apt remove ros-humble-*→apt install ros-humble-rviz2         | ① dpkg 残留未清理干净，仍报依赖错误；② 未启动 systemd（WSL2 默认禁用），后续 Docker 启动也受影响 |
    | 3.WSL2 装 Docker→拉取 ROS2 镜像             | curl [get.docker.com](https://get.docker.com)装 Docker→docker run 启动 rviz2 | ① WSL2 无 systemd，service/docker.service 找不到；② 用 systemctl 启动 Docker 报错，改用 dockerd 仍因网络问题拉取镜像超时 |
    | 4.WSL2 装 ROS1（rviz）                      | apt install rviz→roscore                                     | ① Ubuntu22.04 不支持 ROS1 Noetic，roscore 包找不到；② DISPLAY 配置错误，图形窗口无法映射 |
    | 5.Windows Docker Desktop→拉取 ROS2 镜像     | PowerShell 执行 docker run 命令                              | ① 阿里云镜像源连接超时（dial tcp 42.120.57.110:443 失败）；② 官方源下载速度极慢 / 链接失效 |
    | 6.尝试 Windows 原生装 ROS2                  | 下载官方 msi 安装包                                          | ① 下载链接失效（404）；② 有效链接下载速度极慢，耗时超一天    |
    
    > **绝对避开**：① WSL2 中装完整 ROS2 桌面版（依赖必冲突）；② Windows Docker 用第三方镜像源（易超时）；③ Ubuntu22.04 装 ROS1（版本不兼容）；
    >
    > **关键前提**：① WSL2 必须先修复 dpkg + 配置图形映射；② Docker 仅用官方源 + 离线镜像；③ 优先选 “WSL2+ROS2 极简版”（仅 rviz2 + 核心），而非完整桌面版。
    >
    > 最后，面临的是**Ubuntu apt 依赖链完全断裂**的问题 —— 核心 Python 包的依赖环无法修复，常规的`apt fix`/`pip安装`都绕不开，继续在这个损坏的环境里修只会浪费时间。
    >
    > **解决方案：重置 WSL Ubuntu 22.04 环境**（保留 WSL，只清空 Ubuntu 系统），回到干净的初始状态，再重新安装 ROS2
    
    ###### 重置 WSL Ubuntu 22.04（Windows PowerShell 中执行）

    1. 关闭所有 WSL 终端，打开**管理员权限的 PowerShell**；

    2. 执行以下命令重置 Ubuntu（会清空 Ubuntu 内的所有文件，仅保留系统本身）：
    
       ```powershell
       # 停止WSL
       wsl --shutdown
       # 重置Ubuntu-22.04到初始状态（核心：清空损坏的apt环境）
       wsl --unregister Ubuntu-22.04
       # 重新安装干净的Ubuntu-22.04
       wsl --install -d Ubuntu-22.04
       ```
    
    3. 安装完成后，重启 WSL，设置新的 Ubuntu 用户名 / 密码（记好，后续要用）。

##### 阶段2: ROS2 Humble 安装（含 RViz2）

​	进入新的 Ubuntu 22.04 终端，执行以下**干净的安装脚本**（无任何清理步骤，直接装）：

```bash
# ========== 第一步：换清华源（提速，避免下载慢） ==========
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
sudo sed -i 's/security.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
sudo apt update && sudo apt upgrade -y

# ========== 第二步：安装ROS2 Humble核心+rviz2（全新环境，无依赖冲突） ==========
sudo apt install -y curl gnupg2 lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/ros2/ubuntu jammy main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
# 仅装核心+rviz2，全新环境下无任何dpkg报错
sudo apt install -y ros-humble-ros-core ros-humble-rviz2

# ========== 第三步：启动rviz2（WSLg自动出窗口） ==========
source /opt/ros/humble/setup.bash
rviz2
```



#####  阶段3：RViz2 基础配置（点云可视化）

1. 启动点云发布（新开 WSL 终端）

   ```bash
   # 加载ROS2环境 + 发布点云（替换成你的PCD路径）
   source /opt/ros/humble/setup.bash
   ros2 run pcl_ros pcd_to_pointcloud --ros-args -p file_name:="/mnt/d/multi-UAVs-path-planner-main/Map/10X30m.pcd" -p publish_rate:=0.1
   ```

2. 启动 RViz2 加载配置（再开一个 WSL 终端）

    ```bash
    # 加载ROS2环境 + 加载保存的配置
    source /opt/ros/humble/setup.bash
    rviz2 -d ~/pcd_view.rviz
    ```

