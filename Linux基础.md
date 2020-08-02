# Linux基础

## shell定义

shell是一个命令解释器，把应用程序的输入命令信息解释给操作系统，将操作系统指令处理后的结果解释给应用程序。



## 通配符：*、？

- *：匹配任意多个字符
- ？：匹配任意一个字符



## 目录相关操作

### cd 切换路径

cd ..	返回上一级

cd path	进入到path路径



### ls 查看文件

ls	查看文件基本信息

ls -l	查看文件详细信息

ls -a	显示隐藏的文件或目录

ls -R	递归显示子目录的内容

ls -lrt	按时间排序，显示详细信息



### mkdir 创建目录

mkdir dir	创建dir目录

mkdir -p dir1/dir2	递归创建目录

### rmdir 删除目录

rmdir dir	若dir为空，则删除dir；否则，删除出错

rmdir dir1/dir2	递归删除dir2、dir1，但是要保证最底层目录为空，否则删除失败



### pwd 打印当前工作路径



### which 查看命令所在路径



## 文件相关操作

### touch 创建或更新文件

touch filename	如果filename不存在，创建filename文件；否则更新filename的访问时间（并不会打开文件）



### rm 删除文件

rm filename	删除filename；如果filename不存在，会报错

rm -r dir	递归删除目录，即使目录不为空（比rmdir好用）

rm -f filename/rm -rf dir	强制删除



### cp 拷贝文件

cp (dir1)/file1 (dir2)/file2	拷贝 (dir1中的) file1到 (dir2中的) file2中；如果file2不存在，创建file2后拷贝



### cat 查看文件内容

cat filename



### more 分屏查看文件

more filename	分屏查看filename，空格显示下一屏，回车显示下一部分内容

### less 分屏查看文件

less filename	分屏查看filename，包含more的所有功能，同时支持向上和向下翻动



### head 显示文件头n行

head filename	显示文件头10行

head -n filename	显示文件头n行



### tail 显示文件末尾n行

用法和head一致



## 统计信息（了解）

### tree 树形显示目录结构

tree dir



### wc 默认显示：行，单词树，文件大小

wc * dir	统计整个目录中每个文件或目录的行数，单词数，文件大小



### du 递归显示目录中所有内容

du	目录大小用字节数显示

du -h	目录大小用kb、Mb这些单位表示

du -h --max-depth n	递归显示只到n级子目录，n+1级的内容不显示



### df 显示磁盘使用情况

df	磁盘大小用字符数显示

df -h	磁盘大小用k、Mb这些单位显示

df -h --block-size k	磁盘大小指定用k作为单位显示



## 文件属性、用户和用户组

### 文件属性

使用ls -l查看文件时，开头的 “ -rwxrwxrw- ”表示文件的各种属性

##### 第1位：文件类型

- -：普通文件(包括硬连接文件)
- d：目录文件
- l：符号连接(软连接文件)
- b：块设备
- c：字符设备
- p：管道设备pipe
- s：本地套接字（网络编程）

##### 第2-4位：所属用户的权限

##### 第5-7位：所属用户所在的组中，其他用户的权限

##### 第8-10位：非所属用户的组中的用户的权限

- r：可读
- w：可写
- x：可执行

每3位代表一种用户的权限，可以用8进制表示

- r：0100
- w：0010
- x：0001

例：rw-的8进制表示为0110

也可以将8进制数转化为10进制数，rw-的10进制表示为6

所以“ -rwxrwxrw- ”可以用0776表示



### whoami 显示当前用户名



### ln/link 创建文件连接

#### 硬连接

ln file1 file2	创建一个新文件file2，file1与file2内容相同；用ls -l查看时，file1和file2的硬连接数+1；若删除其中一个文件，硬连接数-1，但剩下的文件不会受到影响



#### 软连接

ln -s file1 file2	创建一个新文件，file1与file2指向相同内容；但是硬连接数不会增加；删除file1后，file2无法继续使用



### unlink 解除连接

unlink linkfilename	linkfilename是一个硬/软连接创建出来的文件，unlink会将其删除；如果是硬连接，硬连接数-1



### chmod 改变用户文件权限

chmod [u|g|o|a] [+|-] [r|w|x]

- u:所属用户

- g：所属组

- o：其他用户

- a：所有用户

  例：chmod u-x	表示取消所属用户的可执行权限

  例：chmod u+x	表示增加所属用户的可执行权限

### chown 改变文件、目录所属用户

chown user : group dir/filename	将dir目录或者filename文件的所属用户更改为group组中的user用户



### chgro 改变文件、目录所属组

chgro group dir/filename



## 查找与检索

### find 按文件名、文件类型查找

find [dir] -name "filename"	按名字查找，在dir中查找包含filename的文件，通常配合通配符使用

find [dir] -type t	按类型查找，在dir中查找类型为t的文件

- f：普通文件

- d：目录文件

- l：符号连接

- b：块设备

- c：字符设备

- p：管道设备pipe

- s：本地套接字（网络编程）

  find [dir] -size [+|-]n	按大小查找，+代表大于，-代表小于，不写为等于（不精确）

find [dir] -maxdepth [+|-]n   按路径大小查找

#### -exec、-ok

使用exec、-ok可以对find的结果进行操作

find [dir] -name "*.c" -exec/-ok ls -l {} /; 	查看dir中的.c文件的详细信息

**格式注意：ls -l可以换成其他命令，命令与{}之间有一个空格，{}中间不能有空格，{}与/之间有一个空格**



#### xargs

find [dir] -name "*.c" | xargs ls -l	作用和上面一致

**find的结果与xargs之间的符号是与（|）**



#### -exec、-ok与xargs的区别

前者一次性处理find出来的所有项，xargs分块处理



### grep 按文件内容查找

grep -r "context" [dir]	递归查找dir目录下含有context内容的文件

grep -n "context" [dir]	显示context在文件中的行号

#### 与find配合使用，用法和xargs一致

|grep "context"	筛选出find结果中包含context内容的文件

|grep -v "context"	筛选出find结果中不包含context内容的文件



## 压缩包管理

### zip

zip -r dir.zip dir	将dir目录下的所有文件压缩到dir.zip中，dir.zip放到dir目录下

**-r表示递归子目录**

unzip dir.zip	将dir.zip解压到当前目录下



### gzip与gunzip

tar zcvf  dir.tar.gz dir	打包

- z：生成.gz格式的压缩包，如果没有，则生成.tar
- c/x：压缩/解压
- v：文件信息
- f：指定压缩包的名字

tar zxcf dir.tar.gz    解压.gz文件



### rar

rar a -r newdir dir	打包

rar x newdir.rar	解压



## 其他常用命令

### man 帮助手册

### umask 文件权限补码

### echo 输出变量或者字符串



## vim

### 命令模式

##### 光标移动（可直接用方向键）

- H：左
- L：右
- J：上
- K：下
- 0(数字0)：行首
- $：行尾
- gg：文件的开头
- G：文件的末尾
- nG：到第n行

##### 删除操作

- x：删除光标所在字符
- X：删除光标前一个字符
- dw：删除单词（从光标开始）
- d0(数字0)：删除行首到光标前的内容
- d$ or D：删除行尾到光标的内容
- dd：删除光标所在行（剪切）
- ndd：删除从光标所在行开始的n行

##### 撤销操作

- u：撤销操作
- ctrl + r：反撤销

##### 复制粘贴

- yy：复制一行
- nyy：复制n行
- dd：剪切一行
- p：粘贴到光标所在位置的下一行
- P(cap)：粘贴到光标所在位置的上一行
- r：替换一个字符（按r后，再输入一个字符，就可以替换当前光标所在位置的字符为输入的字符）

##### 可视模式

1. 按v进入可视模式
2. 移动光标选中内容
3. 按y复制选中内容
4. 移动光标按p粘贴

##### 查找

1. 按/进入查找模式
2. 输入查找内容，回车进行查找
3. 按n跳到下一个查找结果，N跳到上一个查找结果
4. 也可以用？进行查找，与/一致，不同的是n是向上跳，N向下跳

##### 格式调整

- gg=G：文件整体调整格式
- <<：左移一个tab

- .>>：右移一个tab
- n>>：从当前行开始的n行，左移一个tab
- n<<：从当前行开始的n行，右移一个tab



### 编辑模式

命令模式 >> 编辑模式

- i：在光标前插入
- a：光标后插入
- I(大写)：行首插入
- A：行尾插入
- o：在下一行插入
- O：在上一行插入
- s：删除当前字母，进入插入模式
- S：删除当前行，进入插入模式



编辑模式 >> 命令模式

Esc：从编辑模式变换成命令模式



### 末行模式

命令模式 >> 末行模式 输入 :

- :w：保存
- :q：在未修改文件的前提下，退出
- :wq：保存退出
- :q!：强制退出，不保存
- :wq!：强制保存退出
- :x：保存退出
- :s/content1/content2：将当前行中第一个content1替换为content2
- :s/content1/content2/g：将当前行中所有content1替换为content2
- :%s/content1/content2：将所有行中的第一个content1替换为content2
- :%s/content1/content2/g：将所有content1替换为content2



### 末行模式下分屏

- :sp filename：竖直方向分屏
- :vsp filename：水平方向分屏
- :q：退出一个文件
- :qall：退出全部文件
- wqall：全部保存退出



## 信号

### 信号四要素

- 编号
- 名称
- 事件
- 默认处理动作
  - 终止
  - 忽略
  - 终止并产生core文件
  - 暂停/继续进程

### 信号特点

- 简单
- 不能携带大量信息
- 在特定条件下产生

### 信号处理方式

- 执行默认动作

- 忽略

- 捕捉

  **9号、19号信号不能捕捉，不能忽略，不能阻塞**

### 信号的阻塞和未决

实际执行信号的处理动作称为信号递达（Delivery），信号从产生到递达之间的状态，称为信号未决（Pending）。进程可以选择阻塞（Block）某个信号，SIGKILL 和 SIGSTOP 不能被阻塞。被阻塞的信号产生时将保持在未决状态，直到进程解除对此信号的阻塞，才执行递达的动作。注意，阻塞和忽略是不同的，只要信号被阻塞就不会递达，而忽略是在递达之后可选的一种处理动作。



### man 7 signal

查看所有信号及其含义



## 文件描述符扩展

查看文件描述符最大值

```shell
cat /proc/sys/fs/file-max
```

修改

```shell
/etc/security/limits.conf

#在末尾插入,2048是一个自定义的值
*	soft	nofile	2048
*	hard	nofile	2048

#soft的值可以通过命令修改，但是不能超过配置文件中的值(2048)
ulimit -n 1024
```

