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

通过以上步骤，你可以在 macOS 上顺利使用 Docker 运行容器。若需开机自启，可在 Docker Desktop 设置中启用相关选项。