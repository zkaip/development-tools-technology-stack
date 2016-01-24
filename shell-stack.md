http://www.cnblogs.com/mchina/category/391868.html Shell相关知识

### 版本控制
- Git
- SVN

### 安全
- gpg // 文件签名
- ssh // ssh秘钥登录

### 文本操作
- 文本查看
  - nl 显示行号
  - more
  - cat
  - head
  - tail
- 文本剪切 cut
- 文本计算 wc (计算行数 字数 字符数)
- 行排序 sort
- 删除重复行 uniq
- 流编辑器 sed
- 逐行扫描文件 awk

### 文件操作
- 压缩解压 tar
- 文件拆分 split
- 文件查找 find
- 列出 ls
- 进入 cd

### 系统监控
- 进程监控 ps
- 改变进程优先级 nice renice
- 列出当前系统打开文件的工具 lsof
- 进程查找/杀掉命令 pgrep/pkill
- 实时监测命令 watch
- 查看当前系统内存使用状况 free
- CPU的实时监控工具 mpstat
- 虚拟内存的实时监控工具 vmstat
- 设备IO负载的实时监控工具 iostat
- 当前运行进程的实时监控工具 pidstat
- 报告磁盘空间使用状况 df
- 评估磁盘的使用状况 du

### 模式匹配
- 通配符
- 正则表达式

### 特殊文件
- `/dev/null` **需要命令的退出状态,而非输出**
- `/dev/tty` **读取人工输入**

### 命令行
- set/unset
- declare
- grep
- echo
- printf
- readonly
