#### 实验三
1. 实验内容
   1. 实现自己的shell内容
2. 实验思路
   1. 读取用户输入
      1. 为了能够实现按Tab补全的功能，我们使用readline函数进行用户的读入。
      2. 最后将读入的字符串结尾修改'\n' 为 '\0'
      3. 使用strtok函数按照空格分割命令和执行内容
   2. 若执行ls命令, 则在ls命令后携带一个--color=auto选项。因为在执行ls命令时，如果我们只实现ls的话会报错，因为必须要跟着颜色指令
   3. ll命令和ls命令差不多
   4. 内建指令cd
      1. 如果cd后面的内容为空的话，默认补全".";
      2. 通过系统调用执行$ret =chdir(new_path);$.并且通过返回值ret判断是否存在访问的路径
   5. 实现export内建指令
      1. 我们接收的命令, 都在command_S 字符串中, 此字符串每次循环都会被清除
      2. 所以不能直接将 command_argV[1] putenv到环境变量中, 因为指向的同一块地址
      3. 所以 需要先拷贝一份
   6. 需要执行时创建子进程, 并进程替换
      1. 使用execvp，传入命令和需要执行的内容、
      2. 父进程要等待子进程，父进程回收子进程。
   7. 如何实现按上下键切换历史指令
      1. 使用readline的函数add_history,将每一次输入的命令通过add_history函数保存下来。
   8. 注册了一个SIGINT信号[ctrl+c]的处理函数，就是说进程接收到ctrl+c调用signalhandle的信号处理函数处理该信号
   9. 显示历史命令行的history命令如果用户只输入“history”，打印所有命令，将数组的元素一一打印；如果用户输入“history加数字”，我们就打印所需要的指令。最后，当然避免不了错误，就直接打印error即可。
3.  实验结果展示
![](photo/屏幕截图%202023-12-05%20140359.png)
![](photo/屏幕截图%202023-12-05%20140458.png)
![](photo/屏幕截图%202023-12-05%20140522.png)
    