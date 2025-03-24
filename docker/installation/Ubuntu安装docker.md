# Ubuntu安装docker

以下是在 **Ubuntu** 系统上安装 Docker 的详细步骤（适用于 Ubuntu 20.04/22.04 或更高版本）：

---

### **1. 卸载旧版本（如有）**
如果之前安装过旧版 Docker，先清理残留：
```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

---

### **2. 安装依赖工具**
更新软件源并安装必要工具：
```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```

---

### **3. 添加 Docker 官方 GPG 密钥和仓库**
#### **添加 GPG 密钥**
```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

#### **添加 APT 仓库**
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

### **4. 安装 Docker 引擎**
更新仓库并安装 Docker：
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

---

### **5. 启动 Docker 服务**
```bash
sudo systemctl start docker
sudo systemctl enable docker  # 设置开机自启
```

---

### **6. 验证安装**
运行测试容器：
```bash
sudo docker run hello-world
```
若看到 `Hello from Docker!` 表示安装成功。

---

### **7. 配置非 root 用户权限（可选）**
默认需 `sudo` 运行 Docker 命令。若希望直接使用，将用户加入 `docker` 组：
```bash
sudo usermod -aG docker $USER
```
**生效方式**：注销并重新登录，或重启系统。

---

### **常见问题**
#### **权限问题**
• 若提示 `Permission denied`，确保用户已加入 `docker` 组。
• 若仍失败，尝试重启 Docker 服务：
  ```bash
  sudo systemctl restart docker
  ```

#### **镜像加速**
国内用户可配置镜像加速器（如阿里云、腾讯云）：
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://your-mirror-id.mirror.aliyuncs.com"]
}
EOF
sudo systemctl restart docker
```

---

### **总结**
通过以上步骤，你已在 Ubuntu 上成功安装 Docker。现在可以运行容器（如 Nginx、Redis）：
```bash
docker run -d -p 80:80 nginx
```