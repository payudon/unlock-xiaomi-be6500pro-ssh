# 解锁小米路由器 BE6500 Pro SSH 并启用科学上网

## 功能说明
本 Skill 提供完整的小米路由器 BE6500 Pro 解锁 SSH 并安装 ShellCrash 实现科学上网的操作指南。

## 适用设备
- 小米路由器 BE6500 Pro

## 准备工作

### 所需工具
1. **Termius** - SSH/TELNET工具
   - 下载地址：https://termius.com/download
   
2. **小米路由器 BE6500 Pro 固件**
   - 版本：1.0.46
   - 主下载：https://dl.viimg.com/gaicas/miwifi-rom/miwifi_rd08_firmware_076b5_1.0.46.bin
   - iCloud备用：https://www.icloud.com/iclouddrive/0bc5qzSoVz4Ocn9DctUyr8bKg#miwifi%5Frd08%5Ffirmware%5F076b5%5F1.0.46

### 注意事项
⚠️ **解锁设备有风险，若无编程能力请严格按照教程步骤执行，切勿随意插拔设备，以免设备变砖。**

---

## 操作步骤

### 第一步：刷入稳定版固件

1. **确认固件版本**
   - 登录小米路由器后台
   - 依次点击：**常用设置** → **系统状态**
   - 检查系统版本是否为 **1.0.46**

2. **固件升级**
   - 若当前版本 **低于** 1.0.46：下载1.0.46版固件并手动升级
   - 若当前版本 **高于** 1.0.46：系统将提示无法降级，需使用官方"小米路由器修复工具"完成降级

---

### 第二步：获取 STOK 密钥

1. 登录小米路由器后台
2. 在浏览器地址栏中获取 **STOK** 密钥并记录
3. STOK 密钥为地址栏中蓝色部分数值
   - 示例：`http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/...`
   - 其中 `<STOK>` 就是需要记录的密钥

---

### 第三步：开启 SSH 端口

1. **连接路由器**（无线或有线均可）

2. **执行开启命令**
   - Windows用户：打开 **命令提示符**
   - MacOS用户：打开 **终端**
   
3. **执行以下命令**（将 `<STOK>` 替换为第二步获取的实际值）：
   ```bash
   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20set%20ssh_en%3D1%0A"
   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20commit%0A"
   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/arn_switch -d "open=1&model=1&level=%0Ased%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%22debug%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%0A"
   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/arn_switch -d "open=1&model=1&level=%0A%2Fetc%2Finit.d%2Fdropbear%20start%0A"
   ```

4. **验证结果**
   - 命令返回 `{"code": 0}` 表示 SSH 端口已成功开启

---

### 第四步：执行备份固化操作

#### 4.1 软固化 SSH 端口

1. **SSH 登录信息**
   - 账号：`root`
   - 密码：通过 [秘钥计算器](https://www.oxygen7.cn/miwifi/) 获取
   - 备用链接：https://miwifi.dev/ssh

2. **连接 SSH**
   ```bash
   ssh root@192.168.31.1
   ```

3. **执行软固化命令**：
   ```bash
   nvram set ssh_en=1
   nvram set telnet_en=1
   nvram set uart_en=1
   nvram set boot_wait=on
   nvram commit
   sed -i 's/channel=.*/channel="debug"/g' /etc/init.d/dropbear
   /etc/init.d/dropbear restart
   echo -e 'admin\nadmin' | passwd root
   ```

4. **结果**
   - SSH 登录用户名修改为：`root`
   - 密码修改为：`admin`

5. **添加自动开启 SSH 脚本**
   ```bash
   mkdir /data/auto_ssh && cd /data/auto_ssh
   curl -O https://cdn.jsdelivr.net/gh/lemoeo/AX6S@main/auto_ssh.sh
   chmod +x auto_ssh.sh
   ./auto_ssh.sh install
   ```
   
   ⚠️ **注意**：若上述链接无法访问，可尝试手机热点或其他网络环境。

#### 4.2 固化 SSH 端口

1. **第一次重启**（执行后设备自动重启）
   ```bash
   zz=$(dd if=/dev/zero bs=1 count=2 2>/dev/null) ; printf '\xA5\x5A%c%c' $zz $zz | mtd write - crash
   reboot
   ```

2. **第二次重启**（待设备重启后重新 SSH 进入，执行后设备自动重启）
   ```bash
   nvram set ssh_en=1
   nvram set telnet_en=1
   nvram set uart_en=1
   nvram set boot_wait=on
   nvram commit
   bdata set ssh_en=1
   bdata set telnet_en=1
   bdata set uart_en=1
   bdata set boot_wait=on
   bdata commit
   reboot
   ```

3. **第三次重启**（待设备重启后重新 SSH 进入，执行后设备自动重启）
   ```bash
   mtd erase crash
   reboot
   ```

4. **固化完成**

---

### 第五步：升级固件并重新开启 SSH 端口

1. **升级固件**
   - 完成固化操作后，可将小米路由器固件升级至最新版本

2. **重新开启 SSH**（若升级后 SSH 无法访问）
   - 使用 Termius 进行 TELNET 管理
   - 登录信息：
     - 用户名：`root`
     - 密码：`admin`
   
3. **TELNET 登录后执行**：
   ```bash
   sed -i '/flg_ssh=`nvram get ssh_en`/{:loop; N; /\n.*channel=`\/sbin\/uci get \/usr\/share\/xiaoqiang\/xiaoqiang_version.version.CHANNEL`\n.*return 0\n.*fi/!b loop; d}' /etc/init.d/dropbear
   /etc/init.d/dropbear restart
   echo -e 'admin\nadmin' | passwd root
   ```

---

### 第六步：安装 ShellCrash 面板

#### 6.1 执行安装命令

1. **SSH 进入路由器系统**

2. **执行安装命令**（选择一个源）：
   
   **主源（推荐）**：
   ```bash
   sh -c "$(curl -kfsSl https://cdn.jsdelivr.net/gh/juewuy/ShellClash@master/install.sh)" && source /etc/profile &> /dev/null
   ```
   
   **备用源**（若主源无法访问）：
   
   - fastgit.org：
     ```bash
     export url='https://raw.fastgit.org/juewuy/ShellClash/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
     ```
   
   - GitHub：
     ```bash
     export url='https://raw.githubusercontent.com/juewuy/ShellClash/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
     ```
   
   - jsDelivr CDN：
     ```bash
     export url='https://fastly.jsdelivr.net/gh/juewuy/ShellClash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
     ```
   
   - 作者私人源：
     ```bash
     export url='https://shellclash.ga' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
     ```

#### 6.2 安装配置

1. **选择版本**
   - 输入 `1` 选择 **公测版（推荐）**

2. **选择安装目录**
   - 输入 `1` 安装到 `/data` 目录（推荐）

3. **确认安装**
   - 输入 `1` 确认安装

#### 6.3 配置 ShellCrash

1. **启动管理程序**
   ```bash
   crash
   ```

2. **选择使用环境**
   - 输入 `1` 选择 **路由设备配置局域网透明代理**（全屋科学上网）
   - 输入 `2` 选择 **Linux设备仅配置本机代理**（仅当前设备）

3. **小闪存模式**（若提示空间不足）
   - 输入 `1` 开启（设备无法扩展闪存）
   - 输入 `0` 不开启（设备可扩展闪存，如AX9000）

4. **自动任务配置**
   - 输入 `1` 启用推荐配置

5. **软固化功能**
   - 输入 `0` 或 `1`（根据需求选择）

#### 6.4 导入配置文件

1. **配置文件管理**
   - 输入 `2` 选择 **在线获取完整配置文件**

2. **输入订阅链接**
   - 右键粘贴服务商提供的配置文件链接
   - 若订阅链接为 SS/SSR/VMESS 格式，需先转换：https://acl4ssr.netlify.app/

3. **启动服务**
   - 输入 `1` 立即启动服务

#### 6.5 安装 Dashboard 本地面板

1. **进入更新/卸载菜单**
   - 在 crash 主界面输入 `9`

2. **安装本地面板**
   - 输入 `4` 选择安装本地 Dashboard 面板

3. **选择面板类型**
   - 输入 `1` 安装 **Yacd面板**（推荐，支持自动测速）

4. **访问管理面板**
   - 地址：http://192.168.31.1:9999/ui

---

## 常见问题

### Q1: auto_ssh.sh 脚本下载失败
**A**: 国内网络环境可能屏蔽了该地址，尝试：
- 使用手机热点
- 在上一级网络挂载已有梯子的路由器
- 手动创建 auto_ssh.sh 脚本

### Q2: SSH 登录提示 Permission denied
**A**: 
- 确认使用 [秘钥计算器](https://www.oxygen7.cn/miwifi/) 正确计算密码
- 备用链接：https://miwifi.dev/ssh
- 若已执行软固化，密码已改为 `admin`

### Q3: 执行 mtd erase crash 后无法上网
**A**: 
- 等待设备完全重启
- 若长时间无响应，尝试断电重启
- 若仍无法恢复，可能需要使用官方修复工具

### Q4: 安装 ShellCrash 后 crash 命令 not found
**A**: 
- 执行：`source /etc/profile`
- 若仍无效，重新安装 ShellCrash

### Q5: 本地面板访问显示 404
**A**: 
- 确认 ShellCrash 服务已启动
- 检查端口是否正确（默认9999）
- 重新安装本地面板

### Q6: 秘钥计算器网站无法访问
**A**: 
- 主用：https://www.oxygen7.cn/miwifi/
- 备用：https://miwifi.dev/ssh
- 本地计算：使用 SN 号通过 MD5 算法本地计算（参考：https://blog.csdn.net/bthuntergg/article/details/136956094）

---

## 进阶教程

### Mesh 组网全屋覆盖
若已实现单台路由器的科学上网，可通过 Mesh 组网实现全屋网络覆盖：
- 参考教程：https://www.gaicas.com/mesh.html

---

## 相关资源

- **秘钥计算器（主）**：https://www.oxygen7.cn/miwifi/
- **秘钥计算器（备）**：https://miwifi.dev/ssh
- **ShellCrash 官方 TG 群**：https://t.me/ShellClash
- **订阅转换工具**：https://acl4ssr.netlify.app/
- **ShellCrash 官方安装文档**：https://github.com/juewuy/ShellClash/blob/master/README_CN.md

---

## 技术原理

### SSH 开启原理
通过小米路由器的 Web API 接口（`/api/misystem/arn_switch`）执行命令，修改 NVRAM 配置并启动 Dropbear SSH 服务。

### 固化原理
1. **软固化**：修改 `/etc/init.d/dropbear` 文件，添加自启动脚本
2. **硬固化**：修改 crash 分区和 bdata 配置，使 SSH 设置在固件升级后依然保留

### ShellCrash 原理
在路由器上运行 Clash 核心，通过透明代理（Transparent Proxy）实现局域网内所有设备的科学上网。

---

## 安全提示

⚠️ **重要**：
1. 操作前备份重要数据
2. 确保电源稳定，避免操作中断电
3. 记录每步操作，便于故障时回溯
4. 不要随意尝试未经验证的命令
5. 固件升级可能丢失 SSH，需重新开启

---

## 更新日志

- **2024-01-01**：初始版本
- **2026-06-27**：修复秘钥计算器链接（原链接失效）
- 适用于小米路由器 BE6500 Pro
- 固件版本：1.0.46
- ShellCrash 版本：1.9.0 release

---

**版权声明**：本教程整理自 [草东日记](https://www.gaicas.com/xiaomi-be6500-pro.html)，遵循 CC BY-NC-SA 4.0 许可协议。
