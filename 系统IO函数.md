  系统IO函数
一、 概念
   文件描述符
   PCB
   c库函数的IO缓冲区

系统调用：
  LINux内核提供了一组用户进程与内核进行交互的接口对文件和设备进行访问控制，这些接口被称为系统调用。
系统调用对于应用程序最大的作用在于：
  以同意的形式：为应用程序提供了一组文件访问的抽象接口，因共用程序不需要关心文件的具体类型，也不用关心其内部实现细节。 

常用的系统调用IO函数有：
另外介绍：Linux系统中的man是系统内部分页手册,可用于查询函数在那一页

1、open
   open用于创建一个新文件或打开一个已有文件，返回一个非负的文件描述符fd。
   0、1、2为系统预定义的文件描述符，分别代表STDIN_FILENO、STDOUT_FILENO、STDERR_FILENO。
  
  flags参数一般在O_RDONLY、O_WRONLY和O_RDWR中选择指定一个，还可以根据需要或上以下常值：
    O_CREAT：若文件不存在则创建它，此时需要第三个参数mode，mode可设的值及含义如下图所示。
    O_APPEND：每次写时都追加到文件的尾端
    O_NONBLOCK：如果pathname对应的是FIFO、块特殊文件或字符特殊文件，则该命令使open操作及后续IO操作设定为非阻塞模式
    
   打开方式：
    必选项： flags
       O_RDONLY  只读
       O_WROMLY  只写
       O_RDWR    读写
    可选项：
       O_CREAT
       文件权限：本地存在一个掩码
          文件的实际权限
          给定的权限
          本地掩码
         &
          实际的文件权限
          比如：777
               111111111
               本地掩码取反再做按位操作
       O_TRUNC 
       O_EXCL
       O_APPEND
       
   #include<sys/types.h>
   #include<sys/stat.h>
   #include<fcntl.h>
   //成功返回文件描述符，失败返回-1
   int open(const char *pathname, int flags,... /* mode_t mode */);
   
   
       
       
 2、read
read用于从打开文件中读数据。

#include <unistd.h>

//成功返回读到的字节数；若读到文件尾则返回0；失败返回-1
ssize_t read(int fd, void *buf, size_t count);
read操作从文件的当前偏移量处开始，在成功返回之前，文件偏移量将增加实际读到的字节数。
有几种情况可能导致实际读到的字节数少于要求读的字节数：

读普通文件时，在读到要求字节数之前就到达了文件尾。例如，离文件尾还有30字节，要求读100字节，则read返回30，下次在调用read时会直接返回0
从网络读时，网络中的缓冲机制可能造成返回值少于要求读的字节数，解决办法在网络编程专题中再讲
    
 4、write

write用于向文件写入数据。

#include <unistd.h>

//成功返回写入的字节数，失败返回-1
ssize_t write(int fd, const void *buf, size_t count);
write的返回值通常与参数count相同，否则表示出错。
对于普通文件，write操作从文件的当前偏移量处开始
若指定了O_APPEND选项，则每次写之前先将文件偏移量设置到文件尾
成功写入之后，文件偏移量增加实际写的字节数。
    
 #include<stdio.h>
 #include<stdlib.h>
 #include<sys/stat.h>
 #include<sys/types.h>
 #include<unistd.h>
 #include<fcntl.h>
 
 int main()
 {
   //打开已经存在的文件
   int fd = open("test.sh",O_RDONLY);
   if(fd == -1)
   {
     perror("read file");
     exit(1);
   }
   //创建一个新的文件
   int fd1=open("myword",O_CREAT|O_WRONLY,0664);
   if(fd1 == -1)
   {
     perror("create file");
     exit(1);
   }
   //read file 
   char buf[2048]={0};
   int count = read(fd,buf,seizeof(buf));
   if(count ==-1)  //读取失败
   {
    perror("read");
    exit(1);
   }
   //读取成功
   //写入另一个文件
   while(count) //循环读取
   {
   //数据写入新文件
    int ret = write(fd1,buf,count);
    printf("write bytes: %d\n",count);
    count = read(fd,buf,seizeof(buf));
   }
   
   close (fd);
   close(fd1);
 }
 
      
 4、lseek
 lseek用于设置打开文件的偏移量。

#include <sys/types.h>
#include <unistd.h>

//成功返回新的文件偏移量，失败返回-1
off_t lseek(int fd, off_t offset, int whence);
对offset的解释取决于whence的值：

若whence == SEEK_SET，则将文件偏移量设为距文件开头offset个字节，此时offset必须为非负
若whence == SEEK_CUR，则将文件偏移量设为当前值 + offset，此时offset可正可负
若whence == SEEK_END，则将文件偏移量设为文件长度 + offset，此时offset可正可负
注意：

lseek仅将新的文件偏移量记录在内核中，它并不引起任何IO操作，因此它不是系统调用IO，但该偏移量会用于下一次read/write操作
管道、FIFO和套接字不支持设置文件偏移量，不能对其调用lseek
  
   #include<stdio.h>
 #include<stdlib.h>
 #include<sys/stat.h>
 #include<sys/types.h>
 #include<unistd.h>
 #include<fcntl.h>
 
 int main()
 {
   //打开已经存在的文件
   int fd = open("test.sh",O_RDONLY);
   if(fd == -1)
   {
     perror("read file");
     exit(1);
   }
   //将指针指向文件的末尾
   int ret =lseek(fd,0,SEEK_END);
   printf("return value %d \n",ret);
   //文件拓展
   ret =lseek(fd,2000,SEEK_END); //在文件第2000处拓展
   printf("return value  %d\n",ret);
   //还要进行写操作
   write(fd,"a",1);
   close(fd);
   return 0;
   
  }
  
 5、close
close用于关闭一个已打开文件。

#include <unistd.h>

//成功返回0，失败返回-1
int close(int fd);
进程终止时，内核会自动关闭它所有的打开文件，应用程序经常利用这一点而不显式关闭文件。

6、stat函数 --软链接

函数原型:  

int stat(const char *file_name, struct stat *buf)// 通过文件名filename获取文件信息，并保存在buf所指的结构体stat中

int fstat(int filedes, struct stat *buf);//通过文件描述符获取文件对应的属性。

int lstat(const char *restrict pathname, struct stat *restrict buf);//连接文件描述命，获取文件属性。

返回值：成功返回0，失败返回-1，并且设置error的值

error错误代码:

ENOENT 参数file_name指定的文件不存在

ENOTDIR 路径中的目录存在但却非真正的目录

ELOOP 欲打开的文件有过多符号连接问题，上限为16符号连接

EFAULT 参数buf为无效指针，指向无法存在的内存空间

EACCESS 存取文件时被拒绝

ENOMEM 核心内存不足

ENAMETOOLONG   参数file_name的路径名称太长

给定一个pathname，stat函数返回一个与此命名文件有关的信息结构，fstat函数获得已在描述符filedes上打开的文件的有关信息。
lstat函数类似于stat，但是当命名的文件是一个符号连接时，lstat返回该符号连接的有关信息，而不是由该符号连接引用的文件的信息。

struct stat结构体成员：

struct stat {
dev_t st_dev; //文件的设备编号

ino_t st_ino; //节点

mode_t st_mode; //文件的类型和存取的权限2个btye：文件类型、特殊权限位、文件所有者、文件所有组、其他人

nlink_t st_nlink; //连到该文件的硬连接数目，刚建立的文件值为1

uid_t st_uid; //用户ID

gid_t st_gid; //组ID

dev_t st_rdev; //(设备类型)若此文件为设备文件，则为其设备编号

off_t st_size; //文件字节数(文件大小)

unsigned long st_blksize;   //块大小(文件系统的I/O 缓冲区大小)

unsigned long st_blocks; //块数

time_t st_atime; //最后一次访问时间

time_t st_mtime; //指最近修改文件内容的时间

time_t st_ctime; //指最近改动Inoed的时间

};

7、链接函数
  1）、link 函数
    作用：创建一个硬链接
    原型：int link(const char* oldpath, const char* newpath);
  2)、symlink 函数
    作用：创建一个软链接
  3）、readlink 函数
    作用：读取连接对应的文件名，不是读内容
  4）、UNlink函数
    作用：删除一个文件的目录项并减少它的链接数，若成功则返回0，否则返回-1，错误存在于error中
         如果想通过调用这个函数来成功删除文件，你就必须拥有这个文件的所属目录的写和执行权限
         有时候可以将一个临时文件删除掉
     使用：如果是硬链接，硬链接数减1，当减为0时，释放数据块和iNode
           如果是符号链接，删除符号链接
           如果是文件硬链接数为0，但有进程已打开该文件，并持有文件描述符，则等该进程关闭后该文件时，kernel才真正去删除该文件，是自动删除

8、rename 函数
   作用：文件重命名
   头文件：stdio.h
   函数原型：int rename(const char *oldpath, const char* newpath );

9、目录操作函数
   1）chdir 函数
      作用：修改当前进程的路径
      原型：int chdir(const char *path);
   2)getcwd 函数
      作用：获取当前进程工作目录
      函数原型：char *getcwd(char *buf,size_t size);
   3) mkdir 函数
      作用：创建目录
      注意：创建目录需要有执行权限，否则无法进入目录
      函数原型： int mkdir(const char *pathname, mode_t mode);
   4) rmdir函数
      作用：删除一个空目录
      函数原型： int mkdir(const char *pathname);
   5)opendir 函数
      作用：打开一个目录
      函数原型：DIR *opendir (const char*name);
      返回值：DIR结构指针，该结构是一个内部结构，保存所打开的目录信息，作用类似于FILE结构
      函数出错返回NULL
   6)readdir 函数
      作用：读目录
      函数原型：struct dirent *readdir(DIR *dirp)
      返回值：
            返回一条记录项
            struct dirent
            {
              ino_t d_ino;     //此目录进入点的iNode
              ff_t d_off;         //目录文件开头致辞目录进入点的位移
              signed short int d_reclen; //d_name 的长度 不包括NULL字符
              unsigned char d_type ;   //所指文件类型
              char d_name[256];      //文件名
            }
          其中 d_type
           {
             DT_DIR --目录
             DT_BLK --块设备
             DT_CHR --字符设备
             DT_LNK --软链接
             DT_FIFO --管道
             DT_REG  --普通文件
             DT_SOCK --套接字
             DT_UNKNOWN --未知
           }

10、文件描述符的复制
   dup()
   dup2() //可以进行文件指针重定向，例如：dup2(3,1); 表示 1代表的文件指向3代表的文件区
   再如：dup2(fd,STDOUT_FILENO);// STDOUT_FILENO 表示宏定义

11、fcntl 函数
  改变已经打开的文件的属性
  打开文件的时候：只读
     修改文件的： 







       
   
   
   
