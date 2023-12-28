# 实验2 进程隐藏
1. 目标
   1. 进程隐藏
2. 实验步骤
   1. 首先使用strace命令追踪ps.发现ps的命令是通过readdir函数来访问/proc/pid（进程号id）的状态来显示所有的进程号。
    ![](./photo/屏幕截图%202023-11-29%20203128.png)
    2. 主要思想
       1. 首先进入内核源码在PCB增加一个hide字段，用来表示是否要隐藏进程（1表示隐藏，0表示不隐藏）。
       2. 在相关联的函数增加一个判断条件，若是hide字段为1就不做将信息复制到/proc目录下的操作，在fork函数添加一个hide初始化内容，之后添加两个系统调用，用来实现将进程隐藏和显示。
    3. 在include/linux/sched.h文件中，加入hide字段。这个字段是linux内核源码提供给用户自定义变量的地方。
    ![](photo/屏幕截图%202023-11-29%20210228.png)
    4. 修改fs/proc/base.c文件中的proc_pid_readdir() 函数。因为这个函数时根据系统内进程动态添加子进程号目录，我们要做的就是在这个循环里增加一个判断当hide字段为0，即不隐藏的状态下是才进行添加操作，否则不进行添加操作。
    ![](photo/屏幕截图%202023-11-29%20211112.png)
    5. 在kernel/fork.c中增加hide字段的初始化，将其初始化为0，表示不隐藏
    6. 在kernel/sys.c增加两个系统调用。在/unistd.h文件中修改系统调用号的总数。在/syscalls.h函数中声明刚才添加的两个系统调用函数。最后在系统调用表中syscall_64.tbl，添加系统调用号对应的表项。
    ![](photo/屏幕截图%202023-11-29%20213401.png)
    7. 重新编译内核。
 3. 实验结果展示
![](屏幕截图%202023-11-30%20000708.png)
通过使用系统调用和ps指令，发现进程被隐藏了，hide进程为运行的进程。但是After hide后就被隐藏，After unhide后就又显示。
1. 代码展示
```cpp
#include <stdio.h>
#include <sys/types.h>
#include <linux/unistd.h>
#define __NR_hide 436  
#define __NR_unhide 437  
int main()
{
        printf("Before hide:\n");
        system("ps");
        syscall(__NR_hide);//调用系统号为436的系统调用  
        printf("After hide:\n");
        system("ps");
        syscall(__NR_unhide);
        printf("After unhide:\n");
        system("ps");
        return 0;
}

```
5. 遇到问题以及解决
   1. 使用gconfig可视化图形化进行设置出现
    ```cpp
    /bin/sh: 1: bison: not found
    make[1]: *** [scripts/Makefile.host:17：scripts/kconfig/parser.tab.h] 错误 127
    make: *** [Makefile:601：gconfig] 错误 2
    ```
    解决使用按照flex
    2. make install时会出现提示我们创建的系统调用不正确。
    解决：使用专门的而系统调用函数头
    ![](photo/屏幕截图%202023-12-05%20143328.png)
