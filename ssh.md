### **文档：Windows 平台 SSH 远程开发环境配置与排错 SOP**

#### **1. 目标与环境**

  * **目标**: 在一台Windows主机（服务端）上配置OpenSSH服务，允许另一台Windows主机（客户端）通过互联网进行安全的、基于密钥的无密码远程连接，以实现远程开发。
  * **服务端环境**:
      * **操作系统**: Windows 11
      * **内网IP**: `192.168.1.103` (通过DHCP保留实现静态分配)
      * **账户名**: `yan02`
  * **客户端环境**:
      * **操作系统**: Windows 11
      * **账户名**: `ptlan`
  * **网络环境**:
      * **路由器**: TP-Link (管理地址 `http://192.168.1.1`)
      * **公网IP**: `124.168.121.187` (示例,可通过[该网站](https://whatismyipaddress.com/)查询公网ip)
  * **核心工具**:
      * `OpenSSH for Windows`
      * `Visual Studio Code (Remote Development 扩展)`
      * `Windows PowerShell`

-----

#### **2. 配置流程与排错记录**

##### **步骤 2.1: 服务端 - 配置静态内网IP (DHCP保留)**

  * **操作**: 为服务端电脑分配一个固定的局域网IP地址，防止因IP地址变化导致端口转发失效。
  * **工具**: 浏览器，访问路由器管理后台。
  * **路径**: `Network` \> `LAN Settings` \> `Address Reservation`。
  * **配置**: 基于服务端网卡的MAC地址（`90:65:84:...`），添加一条地址保留规则，将其IP地址固定为 `192.168.1.103`。  
[![固定ip.png](https://youke1.picui.cn/s1/2025/08/09/689625cb4af8b.png)](https://youke1.picui.cn/s1/2025/08/09/689625cb4af8b.png)

##### **步骤 2.2: 服务端 - 安装与启动 OpenSSH Server**

  * **操作**: 安装并启动SSH服务。
  * **工具**: `Windows 设置`，`Windows PowerShell (管理员)`。
  * **流程**:
    1.  通过 `设置 > 应用 > 可选功能` 安装 `OpenSSH 服务器`。
    2.  在管理员PowerShell中执行以下命令以启动并自启服务：
        ```powershell
        Start-Service sshd
        Set-Service -Name sshd -StartupType 'Automatic'
        ```
    3.  执行以下命令，创建防火墙入站规则以允许TCP `22` 端口的通信：
        ```powershell
        New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
        ```

##### **步骤 2.3: 路由器 - 配置端口转发**

  * **操作**: 将公网的特定端口映射至服务端的SSH端口。
  * **工具**: 浏览器，访问路由器管理后台。
  * **路径**: `NAT Forwarding` \> `Virtual Servers`。
  * **配置**: 添加新规则，具体参数如下：
      * **Service Type**: `VSCodeSSH` (自定义名称)
      * **External Port**: `22222`
      * **Internal IP**: `192.168.1.103`
      * **Internal Port**: `22`
      * **Protocol**: `TCP`
      * **Status**: `Enabled`  
[![内网穿透.png](https://youke1.picui.cn/s1/2025/08/09/689625cacc2ab.png)](https://youke1.picui.cn/s1/2025/08/09/689625cacc2ab.png)

##### **步骤 2.4: 客户端 - 首次连接与密码认证问题**

  * **操作**: 客户端使用VS Code或`ssh`命令尝试连接。
  * **命令**: `ssh yan02@124.168.121.187 -p 22222`
  * **现象**: 连接时被要求输入密码，但服务端账户（`yan02`）未设置密码，仅使用PIN登录。
  * **原因分析**:
    1.  SSH协议认证基于账户**密码(Password)**，而非设备**PIN码**。
    2.  OpenSSH Server默认配置（`sshd_config`）禁止空密码账户登录。
  * **解决方案**: 在服务端，通过 `设置 > 账户 > 登录选项` 为`yan02`账户添加一个强密码。

##### **步骤 2.5: 客户端/服务端 - 配置SSH密钥对**

  * **操作**: 在客户端生成密钥对。

      * **工具 (客户端)**: `PowerShell`。
      * **命令**: `ssh-keygen -t rsa -b 4096` (使用默认选项)。

  * **操作**: 将公钥部署至服务端。

      * **流程**:
        1.  在客户端，复制 `C:\Users\ptlan\.ssh\id_rsa.pub` 文件内容。
        2.  在服务端，创建 `C:\Users\yan02\.ssh\authorized_keys` 文件，并将公钥内容粘贴进去。

  * **操作**: 在服务端禁用密码认证。

      * **工具 (服务端)**: `Notepad++ (管理员)`。
      * **文件**: `C:\ProgramData\ssh\sshd_config`。
      * **修改**: 将 `PasswordAuthentication yes` 修改为 `PasswordAuthentication no`（并移除行首`#`）。

##### **步骤 2.6: 排错回合一 - `Permission Denied`**

  * **现象**: 完成密钥配置并重启`sshd`服务后，连接失败，报错 `Permission denied (publickey, keyboard-interactive)`。
  * **诊断**: 使用客户端`ssh -v`命令，日志显示公钥被服务端拒绝。问题定位在服务端。  
[![-v.png](https://youke1.picui.cn/s1/2025/08/09/68962752bbb66.png)](https://youke1.picui.cn/s1/2025/08/09/68962752bbb66.png)

##### **步骤 2.7: 排错回合二 - `authorized_keys` 文件格式与 `sshd_config` 配置**


  * **问题**: `sshd_config` 文件末尾的 `Match Group administrators` 配置块，将管理员的密钥文件路径重定向至 `C:\ProgramData\ssh\administrators_authorized_keys`。  
[![match group.png](https://youke1.picui.cn/s1/2025/08/09/6896271581a69.png)](https://youke1.picui.cn/s1/2025/08/09/6896271581a69.png)

      * **解决方案**: 将该配置块的两行用 `#` 注释掉，使其失效。

##### **步骤 2.8: 排错回合三 - 网络连通性验证与日志缺失**

  * **现象**: 修复上述问题后，连接依然失败。同时，在`sshd_config`中设置`LogLevel DEBUG3`后，服务端的`事件查看器 > OpenSSH > Operational` 日志为空。  
[![空日志.png](https://youke1.picui.cn/s1/2025/08/09/689626ac8e83c.png)](https://youke1.picui.cn/s1/2025/08/09/689626ac8e83c.png)
  * **诊断**: 使用[在线端口检查工具](https://www.portcheckers.com/)测试公网IP的`22222`端口，结果显示**Open (开启)**。  
[![unnamed.png](https://youke1.picui.cn/s1/2025/08/09/68962675a7d52.png)](https://youke1.picui.cn/s1/2025/08/09/68962675a7d52.png)
  * **结论**: TCP网络路径通畅。问题是`sshd`服务在收到连接后、记入日志前即发生错误，推断为Windows文件系统ACL权限问题。

##### **步骤 2.9: 最终修复 - Windows 文件系统 ACL 权限**

  * **原因分析**: `sshd`服务以`SYSTEM`账户身份运行，但`SYSTEM`账户无权访问属于`yan02`用户目录下的`authorized_keys`文件。
  * **解决方案**: 在服务端，以管理员身份运行PowerShell，执行以下脚本以精确重置文件权限。
      * **命令**:
        ```powershell
        $authorizedKeysFile = "C:\Users\yan02\.ssh\authorized_keys"
        $acl = Get-Acl $authorizedKeysFile
        $acl.SetOwner([System.Security.Principal.NTAccount]"yan02")
        $acl.Access | ForEach-Object { $acl.RemoveAccessRule($_) } | Out-Null
        $acl.SetAccessRuleProtection($true, $false)
        $acl.AddAccessRule((New-Object System.Security.AccessControl.FileSystemAccessRule("yan02", "FullControl", "Allow")))
        $acl.AddAccessRule((New-Object System.Security.AccessControl.FileSystemAccessRule("SYSTEM", "FullControl", "Allow")))
        $acl.AddAccessRule((New-Object System.Security.AccessControl.FileSystemAccessRule("Administrators", "FullControl", "Allow")))
        Set-Acl $authorizedKeysFile $acl
        ```
  * **操作**: 重启 `sshd` 服务。
      * **命令**: `Restart-Service sshd`

##### **步骤 2.10: 最终验证**

  * **操作**: 客户端再次尝试连接。
  * **结果**: **连接成功**。实现了无需密码的密钥认证登录。

-----

#### **3. 核心问题总结**

本次配置过程中的关键障碍点为Windows环境下OpenSSH服务的权限问题。最终成功是基于以下关键修复：

1.  **停用`sshd_config`中针对`administrators`组的`AuthorizedKeysFile`路径覆盖规则。**
2.  **修正`authorized_keys`文件的ACL权限**，确保运行`sshd`服务的`SYSTEM`账户，拥有对用户目录下`authorized_keys`文件的完全控制权限。