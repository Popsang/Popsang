# 普通

rz：传文件到linux
sz：发文件到windows
mdkir
cp -r ：复制文件夹
ls -l ：文件信息
ps -ef ： 查看进程
kill -l PID :优雅删除进程 ； kill -pid： 可能会有子进程残留；kill -9 PID：强制删除进程
unzip filename.zip ：解压命令

tar -zxvf filename.tar.gz 解压

zip -r yasuo.zip dir1
sdiff -abls -w 168 ：对比文件（仅显示不同的内容，直观）
du -sh * ：查看文件夹大小
du -h --max-depth=1 看一层目录大小
ll -lh :查看文件大小
\> xx.log :清空日志

ps aux | grep auto-launch：查看进程pid

lsof -i [端口号] : 端口占用进程

nohup /kibana-6.3.2-linux-x86_64/bin/kibana >/dev/null 2>&1 & 启动命令



# vim

u:撤销





# >/dev/null 2>&1 含义

\>  ：代表重定向到哪里，例如：echo "123" > /home/123.txt
1  ：表示stdout标准输出，系统默认值是1，所以">/dev/null"等同于"1>/dev/null"
2  ：表示stderr标准错误
&  ：表示等同于的意思，2>&1，表示2的输出重定向等同于1



curl ifconfig.me : 本机ip