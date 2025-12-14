---
title: Rocky Linux VPS 一般安全配置
date: 2025-12-12 22:12:45
categories:
  - [VPS 配置]
tags:
  - Rocky Linux
  - VPS
---

### 添加一般用户

```bash
adduser your_username
passwd your_username # 设置密码
usermod -aG wheel your_username # 添加 sudo 权限
exit
```

测试新用户，能否正常使用：

```bash
ssh your_username@your_server_IP
sudo dnf upgrade
sudo dnf install vim
```

### ssh 设置

#### 禁用 root 用户远程登录

cd 到 ssh 的设置目录，检查当前的 PermitRootLogin 设置。

```bash
cd /etc/ssh/
sudo grep -r PermitRootLogin ./
```

将查询到的所有未注释的 PermitRootLogin 设置改为 `PermitRootLogin no`，一般是 `sshd_config` 和 `./sshd_config.d/01-permitrootlogin.conf` 这两个文件。

#### 更改 ssh 端口

cd 到 ssh 的设置目录，检查当前的 Port 设置。

```bash
cd /etc/ssh/
sudo grep -r Port ./
```

更改为一个不常用的端口号（1024-65535 之间）。

#### 重载 ssh 设置

```bash
sudo systemctl restart sshd
```

#### 确认更改设置生效

1. 尝试使用默认端口远程登录：

```bash
ssh root@your_server_IP
ssh your_username@your_server_IP
```

出现 `Connection closed by your_server_IP port 22` 报错则说明 ssh 端口设置成功。

2. 尝试远程登录到 root 用户，使用更改后的端口：

```bash
ssh root@your_server_IP -p your_ssh_port
```

出现 `Permission denied, please try again.` 报错则说明设置`禁用 root 登录`成功，否则需要确认 `PermitRootLogin` 是否全部修改正确，并且重载了 ssh 服务。

3. 最后远程登录到一般用户，使用更改后的端口：

```bash
ssh your_username@your_server_IP -p your_ssh_port
```

确认登录成功。

#### ssh 使用密钥登录

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" # 生成密钥
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p your_ssh_port your_username@your_server_IP # 选择其中一个密钥（如果有多个的话）并添加到VPS

ssh -i ~/.ssh/id_ed25519.pub -p your_ssh_port your_username@your_server_IP # 使用密钥登录 ssh
```

最后禁用密码登录：

```bash
cd /etc/ssh/
sudo grep -r PasswordAuthentication ./
sudo vim /etc/ssh/sshd_config
```

添加 `PasswordAuthentication no`。

```bash
sudo systemctl restart sshd
```

测试：

```bash
ssh your_username@your_server_IP -p your_ssh_port # 失败，提示：Permission denied
ssh -i ~/.ssh/id_ed25519.pub -p your_ssh_port your_username@your_server_IP # 成功
```

### 防火墙设置

Rocky Linux 使用 `firewall` 管理防火墙，与一般的 `ufw` 操作不同。

启用 `firewall` 服务并查看当前规则：

```bash
sudo systemctl enable firewalld --now
sudo firewall-cmd --list-all
```

放行 ssh 更改后的端口，禁用原来的端口：

```bash
sudo firewall-cmd --permanent --add-port=your_ssh_port/tcp
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --reload
```

顺便启用/禁用一些其他的服务，并设置默认丢弃请求：

```bash
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --remove-service=cockpit
sudo firewall-cmd --permanent --remove-service=dhcpv6-client
sudo firewall-cmd --permanent --set-target=DROP
sudo firewall-cmd --reload
```
