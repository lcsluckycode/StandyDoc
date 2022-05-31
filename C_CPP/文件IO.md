# 文件I/O

## 1. 系统调用open()

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int open(const char *name, int flags);
int open(const cahr *name, int flags, modt_t mode);
```

如果系统调用open()执行成功,会返回文件的描述符.  假如要打开的文件存在,则打开文件, 不存在, 则创建文件, 再打开文件.

flags参数时由一个或者多个标志位的按位或组合. 常用标记(`O_RDONLY | O_WRONLY | O_RDWR`)

新建文件的所有者是创建该文件的进程的拥有者,  新建文件的权限由mode参数指定。由常见的UNIX权限位集合（0644）[1：x，2：w，4：r]

creat()函数

## 2. read()

```c
#include<unistd.h>
ssize_t read(int fd, void* buf, size_t len);
```

每次调用read函数，都会从fd指向的文件的 **当前偏移位置** 开始读取len个字节到buf到所指向的内存。

执行成功的时候会返回**写入buf中的字节数**，出错的时候，返回 -1.

fd文件位置指针会向前移动，移动的长度由读取的字节数所决定。如果fd所指向的对象不支持seek操作（比如字符设备文件），则读操作总是从"当前"的位置开始。

**返回值**

- 调用返回值等于len。读取到的所有len字节都被存储再buf中。
- 调用返回值小于len，大于0。读取到的字节被存储到buf中。之中情况有很多原因，比如再读取过程中信号中断或读取中出错，可读的数据长度小于len，在读取len字节前到达EOF。再次执行read会把剩余的字节读到缓冲区或者给出错误信息
- 返回0。标识EOF，没有可读的数据。由于当前没有数据可读，调用堵塞。在非堵塞模式下，不会发生这种情况
- 返回 -1。并把 errno 设置成 EINTR。 标识在读取任何字节前接收到信号。可以重新执行调用
- 返回 -1，errno为EAGAIN。表示当前没有数据可读，并进入阻塞，请求稍后再重新执行
- 返回 -1，errno没有设置成上述两值。表示发生严重错误。

**读入所有字节**

```c
ssize_t ret;
while (len != 0 && (ret = read(fd, buf, len) != 0)) {
	if (ret = -1) {
		if (errno == EINTR) continue;
		perror("read");
		break;
	}
	len -= ret;
	buf += ret;
}
```

**非阻塞读**

需要open()调用指定参数 O_NONBLOCK 。没有数据可读，会返回 -1，并且设置 errno 为 EAGAIN。 所以非堵塞模式读文件的时候，需要检查 EAGAIN。

```c
char buff[BUFFSIZE];
ssize_t nr;

start:
nr = read(fd, buf, BUFFSIZE);
if (nr == -1) {
    if (errno == EINTR) {
        goto start;
    }
    if (errno == EAGAIN) {
        /* other deal */
    }
    else {
        /* errno */
    }
}
```

## 3. write()

```c
#include<unistd>

ssize_t write(int fd, const void *buf, size_t count);
```

每次调用read函数，都会从fd指向的文件的 **当前偏移位置** 开始写入count个字节到文件中。不支持seek的文件（如字符设备）总是从起始位置开始写。

调用成功时，会返回写入的长度。失败返回 -1 并设置 errno 值。返回 0 表示写入了 0 个字节。

一般很少出现部分写的情况

**Append模式**

open()参数设置为 O_APPEND 打开文件描述符号。从文件末尾开始写。

假如两个进程都想从文件的末尾开始写数据。如果不采用append模式，当第一个进程写完后，第二个进程还是会在之前的文件**末尾**的位置进行写入。append模式可以理解为 write 的原子操作。

非阻塞写，与非阻塞读类似，失败都会返回 -1 ，errno为 EAGAIN

当write返回的时候，内核以及把数据从提供的缓存区拷贝到了**内核缓冲区**，但是**不保证数据已经写到了目的地**。

当要对一份刚写道缓冲区的数据进行读操作的时候，内核会直接在缓存区里面读，而不会在磁盘里面读。

## 4. 同步I/O

### 4.1 fsync() , fdatasync()

为了确保数据写入磁盘，最简单的方式是调用 fsync()

```c
#include <unistd.h>
int fsync(int fd);
```

文件描述符必须是以写方式打开。

fdatasync()

```c
#include<unistd.h>
int fdatasync(int fd);
```

与fsync()类似，主要区别是fdatasync只会写入数据和以后要访问文件所需要的元数据。

返回值

`EBADF`文件不是以写方式打开

`EINVAL`文件描述符指向的对象不支持同步

`EIO`出现底层IO错误

### 4.2 sync()

对磁盘所有的缓存区进行同步

```c
#include<unistd.h>
void sync(void);
```

### 4.3 特殊标志位

- O_SYNC

  该文件所有的I/O操作需要同步

  ```c
  int fd;
  fd = open(file, O_WRONLY | O_SYNC);
  ```

  执行完写操作后，隐式执行fsync

- O_DSYNC

  每次读写操作后，只同普通数据，不同步元数据

- O_RSYNC

  指定读请求和写请求之间的同步。必须与上面两个标志位一起用

- O_DIRECT

  直接I/O，I/O操作会忽略页缓存机制，直接对用户空间缓冲区或者设备进行操作

## 5. 关闭文件close()

```c
#include<unistd.h>
int close(int fd);
```

取消当前进程的文件描述符和文件之间的映射

## 6. lseek()

调整当前文件描述符的位置指针的位置

```c
#include<unistd.h>
#include<sys/type.h>

off_t lseek(int fd, off_t pos, int origin);
```

origin

- SEEK_CUR

  位置 = 当前位置 + pos偏移值

- SEEK_END

  位置 = 当前文件长度 + pos偏移值

- SEEK_SET

  位置 = pos偏移值

## 7. 定位读写pread()、pwrite()

相当于read + lseek(SEEK_SET) 、write + lseek(SEEK_SET)

```c
#define _XOPEN_SOURCE 500
#include <unistd.h>

ssize_t pread(int fd, void *buf, size_t count, off_t pos);

ssize_t pwrite(int fd, void *buf, size_t count, off_t pos);
```

# 标准IO

## 1. 打开文件流

```c
#include <stdio.h>

FILE * fopen(const char *path, const char * mode);
```

模式主要是（r、r+、w、w+、a、a+）+表示可读写，r+指针指向开始，w+和a+指向末尾

**通过文件描述符打开流**

```c
#include <stdio.h>

FILE * fdopen(int fd, const char *mode);
```

## 2. 关闭流

```c
#include <stdio.h>
int fclose(FILE *stream);

int fcloseall(void); //关闭所有流
```

## 3. 从流读取数据

```c
#include <stdio.h>
// 每次读取一个字节
int fgetc(FILE *stream);

// 把字符放回流中
int ungetc(int c, FILE *stream);

// 每次读一行
char* fgets (char* str, int size, FILE *stream);

// 读取二进制文件
size_t fread(void *buf, size_t size, size_t nr, FILE *stream);
```



# I/O多路复用（重点）

## 1. select()

条件触发，只有满足了某个条件才会触发.

```c
#include<sys/select.h>

int select(int n, //等于所有集合中文件描述符的最大值 + 1， 最多只支持 1024 
           fd_set *readfds, //是否有数据可读（某个读操作是否可以无阻塞完成）
           fd_set *writefds, //某个写操作是否可以无阻塞完成
           fd_set *exceptfds, //是否发生异常
           struct timeval *timeout //NULL表示阻塞， 结构体内参数都设为 0 表示立即返回
          );

FE_CLR(int fd, fd_set *set);
FD_ISSET(int fd, fd_set *set);
FD_SET(int fd, fd_set *set);
FD_ZERO(fd_set *set);

/* timeval struct */
#include<sys/time.h>

struct timeval {
    long tv_sec;
    long tv_usec;
};
```

示例

阻塞等待stdin的输入，超时设置为 5 s

```c
#include<stdio.h>
#include<sys/time.h>
#include<sys/types.h>
#include<unistd.h>

#define TIMEOUT 5
#define BUF_LEN 1024

int main(void) {
    struct timeval tv;
    fd_set readfds;
    int ret;
    FD_ZERO(&readfds);
    FD_SET(STDIN_FILENO, &readfds);
    
    tv.tv_sec = TIMEOUT;
    tv.tv_usec = 0;
    
    ret = select (STDIN_FILENO + 1,
                 &readfds,
                 NULL,
                 NULL,
                 &tv);
    
    if (ret == -1) {
        perror("select");
        return 1;
    } else if (!ret) {
        printf("%d seconds elapsed.\n", TIMEOUT);
        return 0;
    }
    
    if (FD_ISSET(STDIN_FILENO, &readfds)) {
        char buf[BUF_LEN + 1];
        int len;
        len = read(STDIN_FILENO, buf, BUF_LEN);
        if (len == -1) {
            perror("read");
            return 1;
        }
        
        if (len) {
            buf[len] = '\0';
            printf("read: %s\n", buf);
        }
        return 0;
    }
    fprintf(stderr, "Rhis should not happend!\n");
    return 1;
}
```

## 2. poll()

每一个pollfd都表示对某一个fd的操作，可以支持多个fd同时操作，但需要为每一个fd声明一个pollfd文件

timeout 表示毫秒(ms)

```c
#include<poll.h>

int poll(struct pollfd *fds, nfds_t nfds, int timeout);

struct pollfd {
    int fd;  //文件描述符
    short events; //声明触发的条件（so many）
    short revents; //内核调用poll后，对事件结果处理的返回值
};
```

示例：

检测stdin和stdout是否发生阻塞

```c
#include<stdio.h>
#include<unistd.h>
#include<poll.h>

#define TIMEOUT 5

int main(void) {
    struct pollfd fds[2];
    int ret;
    
    fds[0].fd = STDIN_FILENO;
    fds[0].events = POLLIN;
    
    fds[1].fd = STDOUT_FILENO;
    fds[1].events = POLLOUT;
    
    ret = poll(fds, 2, TIMEOUT * 1000);
    if (ret == -1) {
        perror("poll");
        return 1;
    }
    
    if (!ret) {
        printf("%d seconds elapsed.\n", TIMEOUT);
        return 0;
    }
    
    if (fds[0].revents & POLLIN) {
        printf ("stdin is readable\n");
    }
    if (fds[1].revents & POLLOUT) {
        printf ("stdout is writable\n");
    }
    return 0;
}
/* ./poll */
// stdout is writable

/* ./poll < test.txt */
// stdin is readable
// stdout is writable
```

**select 和 poll 的区别**

- poll 不需要用户计算最大文件描述符的值 + 1作为参数传递
- poll 对于更大的文件描述符效率更高
- select 的文件描述符集合是静态的。
- select 返回会重新创建文件描述符集，因此每次调用必须重新初始化
- select 调用的timeout在返回时是未定义的。
- select 可以执行更好，有些unix系统不支持 poll
- select 提供超高精度（us）， poll（ms）

# others

## 1. read()执行过程

- 从C库获取系统调用的定义，在编译的时候转化为 trap 语句
- 一旦进程从用户空间进入内核，交给系统调用 handler 处理， 然后交给 read() 系统调用
- 内核确定给定的文件描述符所对应的对象类型，并调用相应的 read 函数
- 该函数继续后续操作——比如从文件系统中读取数据，并把数据返回给用户空间的 read 调用，该调用返回系统调用 handler， 它将数据拷贝到用户空间，最后 read 系统调用返回， 程序继续执行。

