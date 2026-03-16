# SSH 免密登录及快捷配置教程 (ed25519)

本教程基于目前 GitHub 官方推荐且已经广泛使用的更加安全和高效的 `ed25519` 算法。

什么是 **SSH 免密登录**？  
基于公钥密码学（Public-key cryptography），SSH 密钥认证通过非对称加密避免了在网络侧信道中传递密码哈希的风险。客户端持有私钥，服务端将对应公钥存放于 `~/.ssh/authorized_keys` 中。在 SSH 握手协议的用户认证阶段，服务器会向客户端发送一个随机质询（Challenge），客户端利用私钥对该质询计算数字签名（Signature）并发送回服务端，服务端通过已授权的公钥验证签名的合法性从而完成认证。这一质询-响应（Challenge-response）机制从根本上防御了密码暴力破解和中间人（MITM）攻击。

什么是 **`ed25519`**？  
`ed25519` 是一种基于 Twisted Edwards 曲线的 EdDSA（Edwards-curve Digital Signature Algorithm）签名方案，其底层依赖于 Curve25519。相比于基于大整数分解难题的传统 RSA，`ed25519` 具备以下显著技术优势：

1. **高安全性与抗侧信道攻击**：在仅需 256 位密钥长度下即可提供约 128 位的安全强度（等同于对等规模的 3072 位 RSA）。并且其算法实现天然移除了数据依赖的分支与内存访问模式，从而对缓存计时等侧信道攻击（Side-channel attacks）免疫。
2. **极简的存储与通信开销**：公钥仅为 32 字节，生成的等参签名仅有 64 字节，大幅压缩了认证过程中的载荷尺寸。
3. **性能优异**：得益于优化的曲线设计和恒定时间计算，其密钥生成及签名/验签的速度在所有主流架构上皆具备压倒性优势。

## 第一步：生成 ed25519 密钥对

相较于传统的 RSA，`ed25519` 具有更高的安全性和更短的密钥长度。在本地电脑的终端（Terminal 或 PowerShell）中运行以下命令：

```bash
ssh-keygen -t ed25519 -C "你的邮箱或备注名"
```

1. 运行后会提示你保存路径，默认即可（按 `Enter` 键）。
2. 接着会提示输入密码（passphrase），如果为了完全免密无需每次输入密码，可以直接按 `Enter` 跳过。
3. 生成成功后，你的 `~/.ssh/` （Windows 下为 `C:\Users\你的用户名\.ssh\`）目录下会生成两个文件：
   - `id_ed25519`：私钥（**打死也不能告诉别人**）
   - `id_ed25519.pub`：公钥（用于上传给服务器）

## 第二步：将公钥上传到服务器

你需要将刚才生成的 `id_ed25519.pub` 文件中的内容添加到服务器的 `~/.ssh/authorized_keys` 文件中。可以根据你的本地操作系统选择以下方式：

### 方式一：使用 `ssh-copy-id` 快捷命令（Mac / Linux / Git Bash 推荐）

在本地终端执行：

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@server_ip
```

_(期间会要求你输入一次服务器的密码，输入成功后公钥会自动添加)_

### 方式二：手动复制（Windows 或无快捷命令时适用）

1. 查看并复制本地公钥内容：
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   _(Windows 可以直接用记事本打开 `C:\Users\用户名\.ssh\id_ed25519.pub` 并复制内容)_
2. 用密码登录你的服务器：
   ```bash
   ssh username@server_ip
   ```
3. 在服务器上创建 `.ssh` 目录及 `authorized_keys` 文件，并赋予正确权限：
   ```bash
   mkdir -p ~/.ssh
   echo "你的公钥内容粘贴到这里" >> ~/.ssh/authorized_keys
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

此时，你已经可以通过 `ssh username@server_ip` 直接免密登录了。

## 第三步：配置 SSH Config 实现别名快捷登录

为了避免每次都要输入 `username@server_ip`，我们可以配置 SSH 客户端配置文件。

1. 在本地机器打开或创建 `~/.ssh/config` 文件（Windows 下为 `C:\Users\你的用户名\.ssh\config`）。
2. 在文件中添加以下内容：

```text
Host myserver                    # 这是你给服务器起的别名，连接时可以直接使用这个名字，譬如ip的尾号
    HostName 192.168.1.100       # 替换为你的服务器IP或域名
    User root                    # 替换为你的登录用户名
    Port 22                      # 替换为你的SSH端口号，默认是22
    IdentityFile ~/.ssh/id_ed25519 # 指定私钥路径（可选，如果你用的是指定的密钥文件，如.pem等则需要指定，否则默认会自动使用.ssh里的密钥，不用写这一行）
```

_(你可以配置多个 `Host`，方便管理多台服务器)_

## 第四步：测试快捷免密登录

配置完成后，以后你只需要在本地终端输入以下命令即可一键登录：

```bash
ssh myserver
```

并且如果你使用 VS Code 的 Remote-SSH 插件，它也会自动读取这份 `config` 文件，在左侧的列表中直接显示 `myserver`，点击即可一键免密连接到远程开发环境。
