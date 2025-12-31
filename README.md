# nodejs-
nodejs开发中工具类代码
运行效果：
服务启动时，控制台输出备份 + 清理的配置信息，并立即执行一次清理
每天 00:00 自动清理 7 天前的备份文件，控制台输出删除日志
每天 12:30/22:30 正常执行备份，不影响清理功能
过期备份文件自动清理功能（保留最近 7 天备份），并通过定时任务定期执行（推荐每天凌晨执行，不与备份任务冲突）.

关键说明与注意事项
1. mysqldump 路径问题
若执行时报 mysqldump: command not found（Linux/Mac）或 'mysqldump' 不是内部或外部命令（Windows），说明 mysqldump 未配置到系统环境变量中。
解决方案：在工具类中指定 mysqldump 绝对路径：
Windows 示例：C:\\Program Files\\MySQL\\MySQL Server 8.0\\bin\\mysqldump.exe
Linux/Mac 示例：/usr/bin/mysqldump
2. MySQL8.0 权限问题
确保 MySQL 账号（如 root）拥有备份权限，若权限不足，可通过 MySQL 命令授权：
sql
GRANT SELECT, LOCK TABLES, SHOW VIEW ON *.* TO 'root'@'localhost';
FLUSH PRIVILEGES;
3. 备份文件过大问题
若数据库较大，可在备份后自动压缩（推荐 gzip），修改 backupMysql 方法中的 mysqldump 命令：
bash
运行
# Linux/Mac 压缩命令
mysqldump [参数] | gzip > ${backupFilePath}.gz
# Windows 需安装 gzip 工具（如 Git Bash 自带），或使用其他压缩方式
4. 定时任务验证
服务启动后，可先通过访问 http://localhost:3000/api/mysql/backup 手动触发备份，验证是否能生成 .sql 备份文件。
若手动备份成功，说明配置无误，等待 12:30 或 22:30 即可自动触发。
5. 备份文件清理
长期运行会生成大量备份文件，可在工具类中新增「过期文件清理逻辑」（如保留 7 天备份），结合定时任务定期执行。
五、运行效果
启动 Express 服务：node app.js
控制台输出初始化日志，说明定时任务已配置成功
到指定时间（12:30/22:30），控制台会输出备份日志，同时在 ./backup/mysql 目录下生成对应的 .sql 备份文件
可通过 http://localhost:3000/api/mysql/backup 手动触发备份，快速验证功能
总结
核心逻辑：mysqldump（备份） + node-schedule（定时） + 工具类封装（解耦）
关键步骤：配置 MySQL 信息 → 封装备份方法 → 创建定时规则 → 集成 Express 启动
避坑点：mysqldump 环境变量配置、MySQL 权限、备份目录创建
