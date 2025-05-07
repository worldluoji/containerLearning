# MacOS安装docker

以下是在 macOS 上通过 Homebrew 安装并启动 Docker 的详细步骤：

---

### **1. 安装 Homebrew（如未安装）**
若尚未安装 Homebrew，可通过以下命令安装：
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

### **2. 通过 Homebrew 安装 Docker Desktop**
Docker 在 macOS 上的官方支持形式是 **Docker Desktop**，需通过 Homebrew 的 `cask` 安装：
```bash
brew install --cask docker
```

---

### **3. 启动 Docker Desktop**
安装完成后，可以通过以下方式启动：
• **图形界面**：打开 `应用程序` 文件夹，双击 **Docker** 图标。
• **命令行**：
  ```bash
  open -a Docker
  ```

---

### **4. 初始化配置**
首次启动时：
1. 系统会提示权限申请，点击 **OK**。
2. 可能需要登录 Docker 账号（可选，可跳过）。
3. 等待 Docker 守护进程启动（状态栏出现鲸鱼图标🐳）。

---

### **5. 验证安装**
在终端运行以下命令，检查 Docker 是否正常工作：
```bash
docker version
docker run hello-world
```
若看到 `Hello from Docker!` 的欢迎信息，表示安装成功。

---

### **常见问题**
• **权限问题**：若提示权限不足，尝试将用户加入 `docker` 用户组（通常安装时已自动配置）。
• **Docker 未运行**：运行 `docker` 命令前，需确保 Docker Desktop 已启动。
• **性能优化**：可在 Docker Desktop 设置中调整资源分配（如 CPU、内存）。

---

在 macOS 上配置 Docker 国内镜像源（镜像加速器）可以显著提升拉取镜像的速度。以下是详细步骤：

---

## 配置 Docker 国内镜像源

**方法 1：通过 Docker Desktop 图形界面配置**
1. 打开 Docker Desktop
   • 点击菜单栏的 Docker 图标（鲸鱼图标），选择 Settings（偏好设置）。


2. 进入 Docker Engine 配置
   • 左侧导航栏选择 Docker Engine，右侧会显示 `daemon.json` 配置文件。


3. 编辑配置文件
   • 在 JSON 对象中添加或修改 `registry-mirrors` 字段，填入国内镜像源地址。例如：

     ```json
     {
       "registry-mirrors": [
         "https://<你的ID>.mirror.aliyuncs.com",  // 阿里云（需替换为你的专属地址）
         "https://mirror.ccs.tencentyun.com",     // 腾讯云
         "https://docker.mirrors.ustc.edu.cn",    // 中科大
         "https://hub-mirror.c.163.com"           // 网易云
       ],
       "experimental": false,
       "features": {
         "buildkit": true
       }
     }
     ```
   • 注意：多个镜像源用逗号分隔，确保 JSON 语法正确（如末尾无多余逗号）。


4. 保存并重启 Docker
   • 点击 Apply & Restart，等待 Docker 重启生效。


---

**方法 2：直接修改配置文件（适用于高级用户）**
1. 手动编辑 `daemon.json` 文件：
   ```bash
   # 打开配置文件（如果不存在会自动创建）
   vim ~/.docker/daemon.json
   ```
2. 输入上述 JSON 内容并保存。

3. 重启 Docker Desktop 服务。

---

**验证配置是否生效**
1. 打开终端，运行以下命令：
   ```bash
   docker info
   ```
2. 在输出中查找 `Registry Mirrors`，确认列出的镜像源地址。

---

**常见国内镜像源地址**
| 服务商      | 镜像地址                                      | 备注                                  |
|-------------|---------------------------------------------|---------------------------------------|
| 阿里云  | `https://<你的ID>.mirror.aliyuncs.com`      | 需[阿里云控制台](https://cr.console.aliyun.com)获取专属地址 |
| 腾讯云  | `https://mirror.ccs.tencentyun.com`         | 无需登录                              |
| 中科大  | `https://docker.mirrors.ustc.edu.cn`        | 无需登录                              |
| 网易云  | `https://hub-mirror.c.163.com`              | 无需登录                              |

---

**常见问题**
1. 配置后 Docker 无法启动  
   • 检查 `daemon.json` 的 JSON 格式是否正确（可使用 [JSONLint](https://jsonlint.com/) 验证）。

   • 删除 `daemon.json` 或恢复默认配置后重试。


2. 镜像拉取仍慢  
   • 尝试更换其他镜像源地址。

   • 检查网络是否正常（如 VPN 可能影响连接）。


3. 旧版 Docker Toolbox 用户  
   • 若使用 VirtualBox 管理的虚拟机，需登录虚拟机并修改 `/etc/docker/daemon.json`。


---

重新启动:
```
# 1. 终止所有 Docker 相关进程
pkill -9 -x Docker

# 2. 重新启动 Docker Desktop
open -a Docker
```