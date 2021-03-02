# `jmap命令`

参数：

- **option：** 选项参数。
- **pid：** 需要打印配置信息的进程ID。
- **executable：** 产生核心dump的Java可执行文件。
- **core：** 需要打印配置信息的核心文件。
- **server-id** 可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器。
- **remote server IP or hostname** 远程调试服务器的IP地址或主机名。

option

- **no option：** 查看进程的内存映像信息,类似 Solaris pmap 命令。
- **heap：** 显示Java堆详细信息
- **histo[:live]：** 显示堆中对象的统计信息
- **clstats：**打印类加载器信息
- **finalizerinfo：** 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
- **dump:<dump-options>：**生成堆转储快照
- **F：** 强制,当-dump或者-histo参数没有响应下使用. 在这个模式下,live子参数无效.
- **help：**打印帮助信息
- **J<flag>：**指定传递给运行jmap的JVM的参数

jps

 jinfo -flags <PID> 打印内存参数