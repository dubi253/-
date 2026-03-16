# 内网离线使用 VS Code 远程开发配置指南

在无互联网的环境下，配置 VS Code 远程开发需要我们手动完成配置。

---

## 第一部分：核心概念与背景知识

在动手之前，请先了解以下核心原则，这将极大帮助你理解后续步骤：

1. 版本必须一致：你必须先在 Windows 本地安装 VS Code，然后获取它的 Commit ID。部署在 Ubuntu 服务器上的 vscode-server 和 vscode-cli 必须与这个 Commit ID 完全一致。
2. 插件分离机制：在远程开发模式下，VS Code 插件分为本地和远程两部分：
   - 本地插件：安装在你的 Windows 电脑上（例如：UI 主题、SSH 连接工具等）。
   - 远程插件：安装在你的 Ubuntu 服务器上（例如：Python、C/C++、Jupyter 等运行环境所需插件）。

---

## 第二部分：文件准备与解压

受限于内网共享盘的规则，单个文件大小不能超过 20MB。所以你需要先解压从共享盘获取的分卷压缩包。

操作步骤：

1. 找到所需的压缩包分卷文件。
2. 双击后缀为 .001 的文件，这会自动调用 7-Zip 打开。
3. 点击“解压”。

解压成功后，你会得到以下文件素材：

- VSCodeUserSetup-x64-xxxx.exe （VS Code Windows 安装包）
- vscode-server.tar.gz （服务端运行核心）
- vscode-cli.tar.gz （服务端命令行工具）
- 一系列 .vsix 文件 （常用的离线插件安装包）

---

## 第三部分：Windows 本地配置

### 3.1 安装本地 VS Code

双击解压出来的安装包（例如：VSCodeUserSetup-x64-1.111.0.exe），按默认提示完成安装即可。

安装完成后，建议你在 VS Code 中打开本教程所在的 Markdown (`.md`) 文件，并按下快捷键 `Ctrl+Shift+V` 开启预览模式。渲染后的排版将极大方便你清晰地阅读后续的命令行脚本和步骤。

### 3.2 获取 Commit ID（最关键的一步）

打开 VS Code 后，按照以下路径获取版本号（默认为英文界面，如果是中文环境请参考括号内提示）：

1. 点击顶部菜单栏的 Help (帮助)。
2. 找到最下面的 About (关于) 并点击。
3. 在弹出的信息面板底部，点击 Copy (复制) 按钮，将所有信息复制到剪贴板。
4. 在 VS Code 中按 Ctrl+N 新建一个空白文件，按 Ctrl+V 粘贴内容。
5. 找到行首为 "Commit: " 的那一行，提取后面紧跟的 40位十六进制字符，这就是 Commit ID。

注意：请将这串 Commit ID 记录下来，我们在后续服务端的配置中会用到它。

### 3.3 安装 SSH 连接插件

为了让 Windows 上的 VS Code 连上服务器，需要先在本地安装必要的插件：

1. 点击左侧工具栏的“ Extensions (扩展)”视图图标。
2. 点击侧边栏右上角 ... 图标。
3. 在下拉菜单中选择 Install from VSIX... (从 VSIX 安装...)。
4. 找到解压出的插件文件夹，依次选中并安装以下插件：
   - Remote - SSH
   - Remote Explorer
   - Remote - SSH: Editing Configuration Files

---

## 第四部分：Ubuntu 服务器配置

注意：VS Code 在 1.82 版本之后，server 的文件结构发生了较大变化。本教程基于 1.82 及以上版本的新结构编写。

**在每一台你需要连接的 Ubuntu 服务器上，都需要进行如下部署：**

### 4.1 上传文件到服务器

在 Windows 下打开 cmd 或 PowerShell 终端，使用 scp 命令将解压好的服务端组件传至 Ubuntu 服务器的用户根目录（~）：

```bash
scp vscode-server.tar.gz username@server_ip:~
scp vscode-cli.tar.gz username@server_ip:~
```

### 4.2 解压与目录配置

在你熟悉的终端内，使用 SSH 登录到 Ubuntu 服务器。复制并执行以下命令进行部署。

**在执行前，将脚本第一行的内容替换为你真实的 Commit ID。**

```bash
# 1. 请替换成你真实的 Commit ID
COMMIT_ID="在这里填入你的40位CommitID"

# 2. 备份旧的 server（防止配置丢失）
tar -czf ~/vscode-server-backup-$(date +%Y%m%d%H%M%S).tar.gz ~/.vscode-server

# 注意：如果你已经按照1.82几以后版本的新目录结构配置过一次了，会发现 ~/.vscode-server/cli/servers/ 目录下已经有 Stable-<COMMIT_ID> 这样的文件夹了。你应当跳过第3步，以保留你的插件和个人配置。
# 3. 清理旧的 server 目录
rm -rf ~/.vscode-server

# 4. 创建 1.82+ 新版所需的目录结构
mkdir -p ~/.vscode-server/cli/servers/Stable-${COMMIT_ID}/server

# 5. 解压 vscode-server
tar -zxf ~/vscode-server.tar.gz -C ~/.vscode-server/cli/servers/Stable-${COMMIT_ID}/server --strip-components 1

# 6. 解压 vscode-cli (会产生一个名为 code 的文件)
tar -zxf ~/vscode-cli.tar.gz -C ~/

# 7. 移动 cli 到目标位置，加上 Commit ID 重命名
mv ~/code ~/.vscode-server/code-${COMMIT_ID}

# 8. (可选) 清理安装包
rm -f ~/vscode-server.tar.gz ~/vscode-cli.tar.gz
```

完成后，你的 ~/.vscode-server 应该符合如下层级：

```text
.vscode-server/
├── cli/
│   └── servers/
│       └── Stable-<COMMIT_ID>/
│           └── server/
└── code-<COMMIT_ID>
```

---

## 第五部分：首次连接远程 Ubuntu 服务器

1. 回到 Windows 电脑打开 VS Code。
2. 按下快捷键 Ctrl+Shift+P 呼出命令面板。
3. 搜索并选择：Remote-SSH: Connect to Host...
4. 输入你的登录信息，例如： ssh username@server_ip
5. 首次连接时，顶部会提示你是否信任该主机的 "指纹(fingerprint)"。
   （SSH 指纹是用来识别服务器真实身份的哈希值，能防止中间人攻击。如果确认 IP 无误，选择“是”或“Continue”即可。）
6. 按提示输入服务器账户密码。

连接成功后，VS Code 左下角会出现彩色（一般是绿色）的远程 SSH 标识。

---

## 第六部分：在远程窗口安装 Linux 插件（容易遗漏的关键步）

连接成功后，当前的 VS Code 窗口已经是远程工作环境了。为了让代码提示和调试生效，你必须在此远程窗口安装开发插件：

1. 点击左侧工具栏的“扩展”视图图标。
2. 点击右上角 ... -> Install from VSIX... 。
3. 弹出的文件选择框此时显示的是服务器目录。请点击右上角的 Show Local (显示本地) 按钮，切换到你的 Windows 本地文件目录。
4. 找到并选择适用于 linux-x64 或 all-platform 架构的 .vsix 插件文件。
5. VS Code 会自动将插件从 Windows 传输到 Ubuntu 并自行安装完毕。

---

## 第七部分：后续版本更新指南

若未来需要升级到 VS Code 新版本，流程如下：

1. 在公网设备下载新版安装包：
   官方下载地址： https://code.visualstudio.com/Download
2. 安装后查看新的 Commit ID。

3. 下载对应新 Commit ID 的离线包。将下方链接中的 {COMMIT_ID} 替换为你刚刚获取的 Commit ID 即可访问：

   Server 离线包下载链接:
   https://update.code.visualstudio.com/commit:{COMMIT_ID}/server-linux-x64/stable

   CLI 离线包下载链接:
   https://update.code.visualstudio.com/commit:{COMMIT_ID}/cli-linux-x64/stable

   注：如果服务器是 ARM64 架构，需将链接中的 linux-x64 改为 linux-arm64。

4. 下载完成后，将其分别重命名为 vscode-server.tar.gz 和 vscode-cli.tar.gz。

5. 通过内网传输工具传回无网环境，按本教程“第四部分”重新解压覆盖即可。

---

## 第八部分：进阶配置与扩展阅读

恭喜你已经跑通了核心的远程开发环境！为了让你的 VS Code 更好用、更高效，强烈建议继续阅读本项目中的其他专项配置指南：

- **[SSH 免密登录及快捷配置教程](./SSH_免密配置教程.md)**：彻底告别每次输入密码的烦恼，实现一键丝滑连入服务器。
- **[VS Code 自动保存配置指南](./VSCode_自动保存配置指南.md)**：防断电、防误关，将代码保存放心地交给自动化。
- **[VS Code 使用 Prettier 格式化指南](./VSCode_Prettier_格式化配置指南.md)**：统一并保持规范的代码风格。

**欢迎共建分享！**
工具的强大离不开大家的折腾与分享。如果你有其他优秀的 VS Code 插件推荐、独门调试配置或特定的工作流经验，非常欢迎你在本工作区下新建 `.md` 文档与大家分享，或者直接修改完善现有的教程。让我们一起打造内网开发环境使用手册！
