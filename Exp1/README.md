## 实验2：进程隐藏

### 1. 实验目标
- **进程隐藏**：实现在Linux系统中隐藏特定进程的功能。

### 2. 实验步骤
1. **分析现有命令**：使用`strace`追踪`ps`命令，发现其通过`readdir`函数访问`/proc/pid`获取进程信息。
   <img style="height: 300px" src ="../photo/屏幕截图%202023-11-29%20203128.png">    
3. **内核源码修改**：
   - 在`PCB`（Process Control Block）中增加`hide`字段，用于标识进程是否隐藏（1为隐藏，0为不隐藏）。
   - 修改相关函数，添加判断条件，基于`hide`字段决定是否复制信息到`/proc`目录。
   - 在`fork`函数中初始化`hide`字段。
   - 在include/linux/sched.h文件中，加入hide字段。这个字段是linux内核源码提供给用户自定义变量的地方。
      <img style="height: 300px" src ="../photo/屏幕截图%202023-11-29%20210228.png">
   - 修改fs/proc/base.c文件中的proc_pid_readdir() 函数。因为这个函数时根据系统内进程动态添加子进程号目录，我们要做的就是在这个循环里增加一个判断当hide字段为0，即不隐藏的状态下是才进行添加操作，否则不进行添加操作。
        <img style="height: 300px" src ="../photo/屏幕截图%202023-11-29%20211112.png">

4. **系统调用添加**：
   - 在`kernel/fork.c`中初始化`hide`字段为0（不隐藏）。
   - 在`kernel/sys.c`中增加两个系统调用，分别用于隐藏和显示进程。
   - 修改`/unistd.h`和`/syscalls.h`，添加系统调用号和声明。
   - 在`syscall_64.tbl`系统调用表中添加新的系统调用项。
       <img  src ="../photo/屏幕截图%202023-11-29%20213401.png"> 

### 3. 实验结果
- 实现了通过系统调用隐藏和显示进程的功能。使用`ps`命令验证，隐藏的进程不显示，而未隐藏的进程正常显示。
   <img style="height: 300px" src ="../photo/屏幕截图%202023-11-30%20000708.png"> 
### 4. 代码示例
```cpp
// 示例代码：使用系统调用隐藏和显示进程
#include <stdio.h>
#include <sys/types.h>
#include <linux/unistd.h>
#define __NR_hide 436  
#define __NR_unhide 437  

int main() {
    printf("Before hide:\n");
    system("ps");
    syscall(__NR_hide);  // 调用系统号为436的系统调用隐藏进程
    printf("After hide:\n");
    system("ps");
    syscall(__NR_unhide);  // 调用系统号为437的系统调用显示进程
    printf("After unhide:\n");
    system("ps");
    return 0;
}
