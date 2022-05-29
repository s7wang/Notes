# IO操作



## 1 标准IO介绍

I/O: stdio	sysio  （优先使用标准IO）

IO是一切程序实现的基础。

* stdio

> fopen();  fclose();  fgetc();  fputc(); fgets(); fputs(); fread(); fwrite();
>
> 
>
> scanf();  printf();
>
> 
>
> fseek();  ftell();  rewind();
>
> 
>
> fflush();



### 1.1 fopen 和 fclose

* fopen();

```c
FILE *fopen(const char *path, const char *mode);

Upon successful completion. return a FILE pointer. Otherwise, NULL is returned  and errno is set to indicate the error.
 -----------------------------------    
|	/usr/include/asm-generic/       |
|    	error-basse.h	errno.h     |
 -----------------------------------    
r	Open text file for reading. The stream is positioned at the beginning of
    the file.
    
r+	Open for reading and writing. The stream is positioned at the beginning
    of the file.
    
w	Truncate file to zero length or create text file for writing. The stream 
    is positioned at the beginning of the file.
    
w+  Open for reading and writing.  The file is created if it does not exist,
	otherwise it is truncated. The stream is positioned at the beginning of 
    the file.
   
a   Open for appending (writing at end of file). The file is created if it 
    does not exist. The stream is positioned at the end of the file.
    
a+  Open for reading and appending (writing at end of file). The file is
    created if it does not exist. The initial file position for reading is 
    at the beginning of the file, but output is always appended to the end 
    of the file.
```



* perror()  打印错误原因。

```c
	void perror(const char *s);

	#include <errno.h>

	const char *sys_errlist[];
	int sys_nerr;
	int errno;

示例
=======================================================================
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main(){

    FILE *fp;

    fp = fopen("tmp", "r");

    if(fp == NULL){
        // fprintf(stderr, "fopen fail!\nerrno = %d\n", errno);
        perror("fopen()");
        exit(-1);
    }

    puts("OK");
    return 0;
}

结果：
-----------------------------------------------------------------------
fopen(): No such file or directory
```

* strerror()  

```c
	#include <string.h>
	
	char *strerror(int ernum);
示例
========================================================================
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
int main(){

    FILE *fp;

    fp = fopen("tmp", "r");

    if(fp == NULL){
        
        fprintf(stderr, "fopen():%s\n", strerror(errno));        
        exit(-1);
    }

    puts("OK");
    return 0;
}

结果
-----------------------------------------------------------------------
fopen():No such file or directory
```

* fclose();

```c
	int fclose(FILE *stream);

	The fclose() function flushes the stream pointed to by stream (writing 
    any buffered output data using fflush(3)) and closes the underlying file 
    descriptor.

	The behaviour of fclose() is undefined if the stream parameter is an 
    illegal  pointer, or is a descriptor already passed to a previous 
    invocation of fclose().
        
    Upon successful completion, 0 is returned.  Otherwise, EOF is returned 
    and errno is set to indicate the error. In either case, any further 
    access (including another call to fclose()) to the  stream  results in  
    undefined behavior.

```

**==打开的文件流一定要关闭！！！==**





### 1.2 fgetc 和 fputc

* fgetc();

```c
	getc 被定义成宏；
	fgetc 被定义成函数；
    从标准输入流/成功打开的文件流中读入
        成功返回 int	失败返回 EOF
```



* fputc();

```c
	同上；
```



* 简单的copy函数

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

int main(int argc, char *argv[]){

    FILE *fps, *fpd;
    int ch = EOF;

    if (argc < 3){
        fprintf(stderr, "Usage:%s <src_file> <dest_file>\n", argv[0]);
        exit(-1);
    }

    fps = fopen(argv[1], "r");
    if (fps == NULL){
        perror("fopen()");
        exit(-1);
    }

    fpd = fopen(argv[2], "w");
    if (fpd == NULL){
        fclose(fps);
        perror("fopen()");
        exit(-1);
    }

    while (1){
        ch = fgetc(fps);
        if (ch == EOF){
            break;
        }
        fputc(ch, fpd);
    }

    fclose(fps);
    fclose(fpd);

    return 0;
}

使用演示：
--------------------------------------------------------------------
./mycopy /etc/services /tmp/out    
```

* 文件字符统计

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>


int main (int argc, char *argv[]){

    FILE *fp;
    int count = 0;
    if (argc < 2){
        fprintf(stderr, "Usage:%s <src_file>\n", argv[0]);
        exit(-1);
    }

    fp = fopen(argv[1], "r");
    if (fp == NULL){
        perror("fopen()");
        exit(-1);
    }

    while (fgetc(fp) != EOF){
        count++;
    }
    printf("count = %d\n", count);
    fclose(fp);


    return 0;
}


使用演示：
----------------------------------------------------------------------
./fgetc mycopy.c

结果：
------------------------
count = 714
```



### 1.3 fgets 和 fputs

* fgets()

```c
	gets()函数不安全，不检查缓冲区是否溢出，不要使用gets()。可以用fgets()代替。
    
	char *fgets(char *s, int size, FILE *stream);

	reads in at most one less than size characters from stream and stores 
    them into the buffer pointed to by s. Reading stops after an EOF or a 
    newline. If a newline is read, it is stored into the buffer. A termi‐
    nating null byte ('\0') is stored after the last character in the buffer.

    fgets(buf, SIZE, stream);  //文件末尾有一个换行符 '\n'
	1.读入一个SIZE-1长度的字符串
    	最后一个字符为'\0' 有效字符为SIZE-1个   
    2.读入字符串长度小于SIZE
        最后为'\n''\0'
```



* fputs()

```c
	char *fputs(const char *s, FILE *stream);
```



* copy改进

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

#define BUF_SIZE    1024

int main(int argc, char *argv[]){

    FILE *fps, *fpd;
    char buf[BUF_SIZE];

    if (argc < 3){
        fprintf(stderr, "Usage:%s <src_file> <dest_file>\n", argv[0]);
        exit(-1);
    }

    fps = fopen(argv[1], "r");
    if (fps == NULL){
        perror("fopen()");
        exit(-1);
    }

    fpd = fopen(argv[2], "w");
    if (fpd == NULL){
        fclose(fps);
        perror("fopen()");
        exit(-1);
    }

    while (fgets(buf, BUF_SIZE, fps) != NULL){
        
        fputs(buf, fpd);
    }

    fclose(fps);
    fclose(fpd);

    return 0;
}
```



### 1.4 fread 和 fwrite

* fread()

```c
	size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
	size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);

	PTR_SIZE == size * nmemb;

	On success, fread() and fwrite() return the number of items read or 
    written. This number equals the number of bytes transferred only when 
    size is 1. If an error occurs, or the end of the  file  is  reached, 
	the return value is a short item count (or zero).
        
    返回值为成功读取对象的数量，成功至少为1个对象，不足一个对象失败返回0。
```

* copy改进

```c
    while ((n = fread(buf, 1, BUF_SIZE, fps)) > 0){
		fwrite(buf, 1, n, fpd);
    }

```





### 1.5 printf 和 scanf



* printf()

```c
       #include <stdio.h>

       int printf(const char *format, ...);
       int fprintf(FILE *stream, const char *format, ...);
       int dprintf(int fd, const char *format, ...);
       int sprintf(char *str, const char *format, ...);
       int snprintf(char *str, size_t size, const char *format, ...);

       int atoi(const char *nptr);
       long atol(const char *nptr);
       long long atoll(const char *nptr);


       #include <stdarg.h>

       int vprintf(const char *format, va_list ap);
       int vfprintf(FILE *stream, const char *format, va_list ap);
       int vdprintf(int fd, const char *format, va_list ap);
       int vsprintf(char *str, const char *format, va_list ap);
       int vsnprintf(char *str, size_t size, const char *format, va_list ap);
```



* scanf()

```c
       #include <stdio.h>

       int scanf(const char *format, ...);
       int fscanf(FILE *stream, const char *format, ...);
       int sscanf(const char *str, const char *format, ...);

       #include <stdarg.h>

       int vscanf(const char *format, va_list ap);
       int vsscanf(const char *str, const char *format, va_list ap);
       int vfscanf(FILE *stream, const char *format, va_list ap);
```



### 1.6 fseek、ftell 和 rewind

```c
       #include <stdio.h>

       int fseek(FILE *stream, long offset, int whence); //设置指针位置
       long ftell(FILE *stream); //返回当前指针位置
       void rewind(FILE *stream); //重定向到文件开头
DESCRIPTION
       The fseek() function sets the file position indicator for the stream 
       pointed to by stream. The new position, measured in bytes, is obtained 
       by adding offset bytes to the position specified by whence. If whence 
       is set to SEEK_SET, SEEK_CUR, or SEEK_END, the offset is relative to 
       the start of the file, the current position indicator, or end-of-file, 
       respectively. A successful call to the fseek() function clears the  
       end-of-file indicator for the stream and undoes any effects of the 
       ungetc(3) function on the same stream.

       The  ftell()  function  obtains  the current value of the file 
       position indicator for the stream pointed to by stream.

       The rewind() function sets the file position indicator for the stream 
       pointed to by stream to the beginning of the file. It is equivalent 
       to:

              (void) fseek(stream, 0L, SEEK_SET)

       except that the error indicator for the stream is also cleared (see 
       clearerr(3)).
-----------------------------------------------------------------------------

       #include <stdio.h>

       int fseeko(FILE *stream, off_t offset, int whence);
       off_t ftello(FILE *stream);
	   
       #define _FILE_OFFSET_BITS	64
	   CFLAGS+=-D_FILE_OFFSET_BITS=64
```

* fseek

```c
int fseek(FILE *stream, long offset, int whence);

------------------------------------------------------------
    stream 流指针
    offset 偏移量
    whence 起始位置
    |SEEK_SET |  0	|文件开头|
    |SEEK_CUR |  1	|文件当前|
    |SEEK_END |  2	|文件结尾|


```



### 1.7 fflush

```c
 	   #include <stdio.h>

       int fflush(FILE *stream);
DESCRIPTION
       For  output  streams,  fflush()  forces a write of all user-space 
       buffered data for the given output or update stream via the stream's 
       underlying write function.

       For input streams associated with seekable files (e.g., disk files, 
       but not pipes or terminals), fflush() discards any buffered data that 
       has been fetched from the underlying file, but has not been consumed 
       by the application.
           
       The open status of the stream is unaffected.

       If the stream argument is NULL, fflush() flushes all open output 
       streams.

       For a nonlocking counterpart, see unlocked_stdio(3).

RETURN VALUE
       Upon successful completion 0 is returned.  Otherwise, EOF is returned 
       and errno is set to indicate the error.
```

* 示例

```c
#include <stdio.h>
#include <string.h>
 
int main()
{
 
   char buff[1024];
 
   memset( buff, '\0', sizeof( buff ));
 
   fprintf(stdout, "启用全缓冲\n");
   setvbuf(stdout, buff, _IOFBF, 1024);
 
   fprintf(stdout, "该输出将保存到 buff\n");
   fflush( stdout );
 
   fprintf(stdout, "这将在编程时出现\n");
   fprintf(stdout, "最后休眠五秒钟\n");
 
   sleep(5);
 
   return(0);
}
------------------------------------------------------------------------
让我们编译并运行上面的程序，这将产生以下结果。在这里，程序把缓冲输出保存到 buff，直到首次调用 fflush() 为止，然后开始缓冲输出，最后休眠 5 秒钟。它会在程序结束之前，发送剩余的输出到 STDOUT。
    ____引用 runoob.com
    
```



### 1.8 getline

```c
SYNOPSIS
       #include <stdio.h>

       ssize_t getline(char **lineptr, size_t *n, FILE *stream);
       ssize_t getdelim(char **lineptr, size_t *n, int delim, FILE *stream);

DESCRIPTION
       getline() reads an entire line from stream, storing the address of 
       the buffer containing the text into *lineptr.  The buffer is null-
       terminated and includes the newline character, if one was found.

       If *lineptr is set to NULL and *n is set 0 before the call, then 
       getline() will allocate a buffer for storing the line. This buffer 
       hould be freed by the user program even if getline() failed.

       Alternatively,  before  calling  getline(), *lineptr can contain a 
       pointer to a malloc(3)-allocated buffer *n bytes in size. If the 
       buffer is not large enough to hold the line,  getline() resizes it  
       with  realloc(3), updating *lineptr and *n as necessary.

       In either case, on a successful call, *lineptr and *n will be updated 
       to reflect the buffer address and allocated size respectively.


```

* 示例

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>


int main (int argc, char *argv[]){

    FILE *fp;
    char *linebuf;
    size_t linesize;
    int count = 0;
    if (argc < 2){
        fprintf(stderr, "Usage:%s <src_file>\n", argv[0]);
        exit(-1);
    }

    fp = fopen(argv[1], "r");
    if (fp == NULL){
        perror("fopen()");
        exit(-1);
    }
    ssize_t read;
    
    linebuf = NULL;
    linesize = 0;
    read = 0;
    
    while (1){
       if ((read = getline(&linebuf, &linesize, fp)) < 0){
           break;
       }
        printf("%d\n", strlen(linebuf));
    }
    
    // free(linebuf);
    // linebuf = NULL;
    fclose(fp);


    return 0;
}
----------------------------------------------------------------------
    统计文件每行字符数量
```



## 2 临时文件

问题：

1.如何不冲突地创建文件。

2.没有及时销毁临时文件。

* 函数

```c
tmpnam	tmpfile
    
    char *tmpnam(char *s); //返回一个可用的名字 可能被大断

       The tmpnam() function returns a pointer to a string that is a valid 
       filename, and such that a file with this name did not exist at some 
       point in time, so that naive programmers may think it a suitable name 
       for a temporary file.If the argument s is NULL, this name is generated 
       in an internal static buffer and may be over‐written by the next call 
       to tmpnam().  If s is not NULL, the name is copied to the character 
       array (of length at least L_tmpnam) pointed to by s and the value s is 
       returned in case of success.

       The  created  pathname has a directory prefix P_tmpdir. (Both L_tmpnam 
       and P_tmpdir are defined in <stdio.h>, just like the TMP_MAX mentioned 
       below.)
----------------------------------------------------------------------------- 
     FILE *tmpfile(void); //匿名文件 关闭文件时直接销毁 不会冲突 不会被打断

DESCRIPTION
       The  tmpfile() function opens a unique temporary file in binary 
       read/write (w+b) mode. The file will be automatically deleted when 
       it is closed or the program terminates.       
           
           
```



## 3 系统调用

### 3.1 文件描述符原理

* 文件描述符的概念

```c

文件标识 inode（整型数， 数组下标）
    			open
    file[inode]  ->  文件结构体 -> pos 位置指针
    							 count 打开计数器
    文件流会存在一个进程空间的数组中  标准输入输出错误 会占用当前可使用地最小地三个文件描述符。

```



* 文件IO操作

>open, close, read, write, lseek
>
>



* 文件IO与标准IO地区别



* IO效率问题

### 3.2 open 和 close

* open

```c
       #include <sys/types.h>
       #include <sys/stat.h>
       #include <fcntl.h>

       int open(const char *pathname, int flags);//变参函数
       int open(const char *pathname, int flags, mode_t mode);

       int creat(const char *pathname, mode_t mode);

       int openat(int dirfd, const char *pathname, int flags);
       int openat(int dirfd, const char *pathname, int flags, mode_t mode);

DESCRIPTION
       The  argument  flags  must  include  one  of the following access 
       modes: O_RDONLY, O_WRONLY, or O_RDWR.  These request opening the file 
       read-only, write-only, or read/write, respectively.

       In addition, zero or more file creation flags and file status flags 
       can be bitwise-or'd in flags. The file creation flags are O_CLOEXEC, 
       O_CREAT, O_DIRECTORY, O_EXCL, O_NOCTTY, O_NOFOLLOW, O_TMPFILE, and 
       O_TRUNC. The file status flags are all of the remaining flags listed 
       below. The distinction between these  two groups of flags is that the 
       file creation flags affect the semantics of the open operation itself, 
       while the file status flags affect the semantics of subsequent I/O 
       operations. The file status flags can be retrieved and (in some cases) 
       modified; see fcntl(2) for details.
           
flags 参数的实质就是位表，对应到标准IO的打开模式下如下：
------------------------------------------------------------------
       r  -> O_RDONLY
       r+ -> O_RDWR
       w  -> O_WRONLY|O_CREAT|O_TRUNC
       w+ -> O_RDWR|O_TRUNC|O_CREEAT
```



* close

```c
       #include <unistd.h>

       int close(int fd);

DESCRIPTION
       close() closes a file descriptor, so that it no longer refers to any 
       file and may be reused.  Any record locks (see fcntl(2)) held on the 
       file it was associated with, and owned by the process, are removed  
       (regardless of the file descriptor that was used to obtain the lock).

       If  fd  is  the  last  file  descriptor  referring  to the underlying 
       open file description (see open(2)), the resources associated with the 
       open file description are freed; if the file descriptor was the last   
       reference to a file which has been removed using unlink(2), the file 
       is deleted.
           
```



### 3.3 read 、write 和 lseek

基本原理用法与标准IO相同，是标准IO的支撑。

* read

```c
SYNOPSIS
       #include <unistd.h>

       ssize_t read(int fd, void *buf, size_t count);

DESCRIPTION
       read() attempts to read up to count bytes from file descriptor fd into 
       the buffer starting at buf.

       On  files that support seeking, the read operation commences at the 
       file offset, and the file offset is incremented by the number of bytes 
       read. If the file offset is at or past the end of file, no bytes are 
       read, and read() returns zero.

       If count is zero, read() may detect the errors described below. In 
       the absence of any errors, or if read() does not check for errors, 
       a read() with a count of 0 returns zero and has no other effects.

       According to POSIX.1, if count is greater than SSIZE_MAX, the result 
       is implementation-defined; see NOTES for the upper limit on Linux.
```

* write

```c
SYNOPSIS
       #include <unistd.h>

       ssize_t write(int fd, const void *buf, size_t count);

DESCRIPTION
       write()  writes up to count bytes from the buffer starting at buf to 
       the file referred to by the file descriptor fd.

       The number of bytes written may be less than count if, for example, 
       there is insufficient space on the  underlying  physical  medium, or 
       the RLIMIT_FSIZE resource limit is encountered (see setrlimit(2)), or 
       the call was interrupted by a signal handler after having written less 
       than count bytes. (See also pipe(7).)

       For a seekable file (i.e., one to which lseek(2) may be applied, for 
       example, a regular  file)  writing  takes place  at the file offset, 
       and the file offset is incremented by the number of bytes actually 
       written. If the file was open(2)ed with O_APPEND, the file offset is 
       first set to the end of the  file  before  writing. The adjustment of 
       the file offset and the write operation are performed as an atomic 
       step.

       POSIX  requires that a read(2) that can be proved to occur after a 
       write() has returned will return the new data. Note that not all 
       filesystems are POSIX conforming.

       According to POSIX.1, if count is greater than SSIZE_MAX, the result 
       is implementation-defined; see NOTES for the upper limit on Linux.
```



* lseek

```c
SYNOPSIS
       #include <sys/types.h>
       #include <unistd.h>

       off_t lseek(int fd, off_t offset, int whence);

DESCRIPTION
       lseek() repositions the file offset of the open file description 
       associated with the file descriptor fd to the argument offset 
       according to the directive whence as follows:

       SEEK_SET
              The file offset is set to offset bytes.

       SEEK_CUR
              The file offset is set to its current location plus offset 
              bytes.

       SEEK_END
              The file offset is set to the size of the file plus offset 
              bytes.

       lseek() allows the file offset to be set beyond the end of the file 
       (but this does not change the size of  the file). If data is later 
       written at this point, subsequent reads of the data in the gap (a 
       "hole") return null bytes ('\0') until data is actually written into 
       the gap.
```



文件IO没有缓存区相应速度快，标准IO有缓冲区吞吐量大。

标准IO与文件IO不可混用。

```shell
time 命令
指令运行的时间。
```



## 4 文件共享

* 一个程序打开两次同一个文件。
* 两个进程打开同一个文件。
* 两个进程打开同一个文件。。

---------------------------------------------



* truncate

```c
SYNOPSIS
       #include <unistd.h>
       #include <sys/types.h>

       int truncate(const char *path, off_t length);
       int ftruncate(int fd, off_t length);
DESCRIPTION
       The  truncate() and ftruncate() functions cause the regular file named 
       by path or referenced by fd to be truncated to a size of precisely 
       length bytes.

       If the file previously was larger than this size, the extra data is 
       lost.  If the file previously was shorter, it is extended, and the 
       extended part reads as null bytes ('\0').

       The file offset is not changed.

       If  the size changed, then the st_ctime and st_mtime fields 
       (respectively, time of last status change and time of last 
       modification; see inode(7)) for the file are updated, and the set-
       user-ID and set-group-ID mode bits may be cleared.

       With ftruncate(), the file must be open for writing; with truncate(), 
       the file must be writable.

```



例题：写程序删除文件的第10行。





### 4.1 原子操作

概念：不可分割的操作。

作用：解决竞争和冲突。

*  dup 和 dup2

```c
SYNOPSIS
       #include <unistd.h>

       int dup(int oldfd);//重定向 非原子
       int dup2(int oldfd, int newfd);//重定向 原子

       #define _GNU_SOURCE             /* See feature_test_macros(7) */
       #include <fcntl.h>              /* Obtain O_* constant definitions */
       #include <unistd.h>

       int dup3(int oldfd, int newfd, int flags);

DESCRIPTION
       The  dup()  system  call  creates  a  copy of the file descriptor 
       oldfd, using the lowest-numbered unused file descriptor for the new 
       descriptor.

       After a successful return, the old and new file descriptors may be 
       used interchangeably. They refer to the same open file description 
       (see open(2)) and thus share file offset and file status flags; for 
       example, if the file offset is modified by using lseek(2) on one of 
       the file descriptors, the offset is also changed  for the other.

       The  two file descriptors do not share file descriptor flags (the 
       close-on-exec flag).  The close-on-exec flag (FD_CLOEXEC; see 
       fcntl(2)) for the duplicate descriptor is off.
   dup2()
       The dup2() system call performs the same task as dup(), but instead 
       of using the lowest-numbered unused file descriptor, it uses the file 
       descriptor number specified in newfd.  If the file descriptor newfd 
       was previously open, it is silently closed before being reused.

       The steps of closing and reusing the file descriptor newfd  are  
       performed  atomically. This is important, because  trying to 
       implement equivalent functionality using close(2) and dup() would be 
       subject to race conditions, whereby newfd might be reused between the 
       two steps. Such reuse could happen because the main program is 
       interrupted by a signal handler that allocates a file descriptor, or 
       because a parallel thread allocates a file descriptor.

       Note the following points:

       *  If oldfd is not a valid file descriptor, then the call fails, and 
          newfd is not closed.

       *  If oldfd is a valid file descriptor, and newfd has the same value 
          as oldfd, then dup2() does  nothing,  and returns newfd.
```

文件重定向示例：

```c
/**
 * 文件重定向 dup dup2 
 * 
 * 将标准输出（控制台）重定向到 out 文件
 */

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define FNAME   "out"

int main(){

#if 0    
    int fd;
    close(1);
    fd = open(FNAME, O_WRONLY|O_CREAT|O_TRUNC, 0600);
    if (fd < 0){
        perror("open()");
        exit(-1);
    }
#endif
//----------------一下虚线内容同条件注释内容-----------------------
    int fd;
    fd = open(FNAME, O_WRONLY|O_CREAT|O_TRUNC, 0600);
    if (fd < 0){
        perror("open()");
        exit(-1);
    }
    /* 以下注释内容非原子 可以被打断 */
    //close(1);
    //dup(fd);
    //close(fd);

    /* 原子操作 */
    dup2(fd, 1);
    if (fd != 1){
        close(fd);
    }
//---------------------------------------------------------------
    /*******************************************************/    
    puts("hello!");

    return 0;
}
```

### 4.2 同步 sync 和 syncfs

* sync  syncfs

同步数据，刷内核buf，接触挂载

同步文件只刷数据不刷亚数据。

```c
SYNOPSIS
       #include <unistd.h>

       void sync(void);
       int syncfs(int fd);

   Feature Test Macro Requirements for glibc (see feature_test_macros(7)):

       sync():
           _XOPEN_SOURCE >= 500
               || /* Since glibc 2.19: */ _DEFAULT_SOURCE
               || /* Glibc versions <= 2.19: */ _BSD_SOURCE

       syncfs():
           _GNU_SOURCE

DESCRIPTION
       sync() causes all pending  modifications to filesystem metadata and 
       cached file data to be written to the underlying filesystems.

       syncfs() is like sync(), but synchronizes just the filesystem 
       containing file referred to by the open file descriptor fd.

RETURN VALUE
       syncfs() returns 0 on success; on error, it returns -1 and sets errno 
       to indicate the error.
```



* fcntl 

```c
SYNOPSIS
       #include <unistd.h>
       #include <fcntl.h>
	   /* 文件描述符的基础函数 */
       int fcntl(int fd, int cmd, ... /* arg */ );
```

*  ioctl

```c
//设备相关内容
SYNOPSIS
       #include <sys/ioctl.h>

       int ioctl(int fd, unsigned long request, ...);

DESCRIPTION
       The  ioctl()  system  call manipulates the underlying device 
       parameters of special files. In particular, many operating
       characteristics of character special files  (e.g., terminals) may 
       be controlled with ioctl() requests. The argument fd must be an 
       open file descriptor.

       The  second  argument is a device-dependent request code. The third 
       argument is an untyped pointer to memory.
       It is traditionally char *argp (from the days before void * was valid 
       C), and will be so named for this discussion.

       An ioctl() request has encoded in it whether the argument is an in 
       parameter or out parameter, and the size of the argument argp in 
       bytes. Macros and defines used in specifying an ioctl() request are 
       located in the file <sys/ioctl.h>.
```

