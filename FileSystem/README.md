# 模拟文件系统
### 实验四
1. 实验内容
   1. 模拟文件系统
2. 实验步骤
   1. 定义数据结构
      1. 文件
      ```cpp
      struct file
      {

      string name;//文件名

      vector<string> content;//文件内容

      user owner;//文件所有者

      };
      ``` 
      2. 目录
      ```cpp
      struct dir {
      string name;//目录名

      dir* pre;//父目录

      map<string, file*> files;//所包含的文件

      map<string, dir*> next;//子目录

      };
      ```  
      3. 使用map的数据结构进行文件的存储和子目录的存储。每一次遍历到文件是通过遍历map来获得文件或者子目录。
   2. 各功能函数
      1. cd
         1. 类似于实验3shell实现中的cd指令。但这个实验我将cd的合理性判断分开新实现了一个函数以实现模块化
      2. ls
         1. 当路径解析正确时遍历输出文件和目录信息。输出就是对于map的操作
         ```cpp
            for (auto it = curdir->files.begin(); it != curdir->files.end(); it++) {

            cout << it->first << setw(25 - it->first.length()) << "<FILE>" << "  owner:" << it->second->owner.name << endl;

         }
         for (auto it = curdir->next.begin(); it != curdir->next.end(); it++) {

            cout << it->first << setw(26 - it->first.length()) << "<DIR>  " << endl;

         }
         ```
      3. mkdir
         1. 先判断再当前目录下有没有重名的目录，如果又就不创建新的，否则创建一个新的目录对象，并放入map中
         ```cpp
         dir *tmp = new dir();
         tmp->name = name;

         tmp->pre = curdir;

         curdir->next[name] = tmp;
         ``` 
      4. touch
         1. 创建一个新的文件，判断有没有非法字符或者重名现象。*值得注意的是，文件要给出文件的创建者，在后续文件读取操作时，需要判断文件的所有者才可以进行操作*
         ```cpp
         file *tmp = new file();
         tmp->name = name;

         tmp->owner = curuser;

         tmp->content.push_back("");

         curdir->files[name] = tmp;
         ``` 
      5. gedit
         1. 进行文件的读写操作
         ```cpp
         //是否存在目标文件  
         if (curdir->files.find(name) == curdir->files.end()) {}  
         //目标文件是否为当前用户所拥有  
         else if (curdir->files[name]->owner.name != curuser.name) {}  
         else {  
            ofstream out("tmp.dat");  
            //将文件当前内容输入临时文件  
            for (int i = 0; i < curdir->files[name]->content.size(); i++) {}  
            out.close();  
            //用gedit打开临时文件  
            system("gedit tmp.dat");  
            //读取临时文件中的内容，存入文件  
            ifstream in("tmp.dat");  
            while (getline(in, t)) {  
                  //读取临时文件内容  
            }  
         }  
         ```   
         2. 为提高用户读写文件的体验，在此引入了一个临时文件机制：将存储在内存中的文件暂时存储到临时文件中，然后用gedit打开这个临时文件。在用户编辑完成后，再将临时文件中的内容转存到内存中。
      6. rm
         1. file|dir，其中file与dir代表需要删除的文件或目录，目录（或文件）均可用绝对路径或者相对路径表示
         2. 删目录应该时递归的删除.因本系统采用树形的目录结构，因此在删除目录时需要用到递归
            1.递归的过程
            ```cpp
            //递归删除目录  
            void deletedir(dir *cur) {  
               //先删文件  
               for (auto it = cur->files.begin(); it != cur->files.end(); it++) {  
                  delete(it->second);  
               }  
               cur->files.clear();  
               //再删目录，要嵌套删除  
               for (auto it = cur->next.begin(); it != cur->next.end(); it++) {  
                  deletedir(it->second);  
               }  
               cur->next.clear();  
               delete(cur);  
            }  
            ```
      7. cp
         1. 复制一个文件或目录到指定路径下。
         2. 跟删除类似，在复制目录时需要用到递归复制
         3. 分为 -d 复制目录 -f复制文件 -cd复制目录，但不在原路径下保留原目录 -cf 复制文件，但不在原路径下保留原文件
         ```cpp
         //递归复制目录  
         dir* cpDir(dir *tmp) {  
            dir *goal = new dir(*tmp);  
            //清除原来的内容  
            goal->next.clear();  
            goal->files.clear();  
            //把文件重建  
            for (auto it = tmp->files.begin(); it != tmp->files.end(); it++) {  
               file *f = new file(*(it->second));  
               goal->files[it->first] = f;  
            }  
            //重建目录  
            for (auto it = tmp->next.begin(); it != tmp->next.end(); it++) {  
               dir *d = cpDir(it->second);  
               d->pre = goal;  
               goal->next[it->first] = d;  
            }  
            return goal;  
         } 
         ```
      8. rename
         1. 更改指定文件或目录的名字。要判断是否重名和非法字符。
         2. 重名判断就是遍历该目录下的所有文件或者子目录看有没有重名的。
         3. 非法字符通过函数```judgeName``` 
3. 实验结果展示
   ![](photo/file1.png) 
   ![](photo/file2.png)
