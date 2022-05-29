# 例程：修改文件时间信息

```
NAME
       utimensat, futimens - change file timestamps with nanosecond precision

SYNOPSIS
       #include <fcntl.h> /* Definition of AT_* constants */
       #include <sys/stat.h>

       int utimensat(int dirfd, const char *pathname,
                     const struct timespec times[2], int flags);

       int futimens(int fd, const struct timespec times[2]);
```



例：下面程序使用带 `O_TRUNC` 选项的 `open` 函数将文件长度截断为 0，但并不更改其访问时间和修改时间。为了做到这一点，首先用 `stat` 函数得到这些时间，然后截断文件，最后再用 `futimens` 函数重置这两个时间。

```c
/**
 * @file futimens.c
 * @author wangs7
 * @brief change file timestamps with nanosecond precision
 * @version 0.1
 * @date 2021-04-20
 * 
 * @copyright Copyright (c) 2021
 * 
 */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>

int main (int argc, char *argv[]) {
    int i, fd;
    struct stat statbuf;
    struct timespec times[2];
    /**
     *  struct timespec {
     *      time_t tv_sec;        // seconds
     *      long   tv_nsec;       // nanoseconds
     *  };
     */

    if (argc < 2) {
        fprintf(stderr, "Parameter usage error!\n");
        exit(1);
    }

    for (i = 1; i < argc; i++) {
        if (stat(argv[i], &statbuf) < 0) { /* fetch current times */
            fprintf(stderr, "%s: stat error.\n", argv[i]);
            continue;
        }
        if ((fd = open(argv[i], O_RDWR | O_TRUNC)) < 0) { /* truncate */
            fprintf(stderr, "%s: open error.\n", argv[i]);
            continue;
        }
        times[0].tv_sec = statbuf.st_atime;
        times[1].tv_sec = statbuf.st_mtime;

        if (futimens(fd, times) < 0) {
            fprintf(stderr, "%s: futimens error.\n", argv[i]);
        }

        close(fd);
    }

    return 0;
}
```

测试：

```
linux@ubuntu:~/UNIX_learning$ ls -l test1 test2		查看长度和最后修改时间
-rw-rw-r-- 1 linux linux 12 Apr 20 03:10 test1
-rw-rw-r-- 1 linux linux 22 Apr 20 03:11 test2

linux@ubuntu:~/UNIX_learning$ ls -lu test1 test2	查看最后访问时间
-rw-rw-r-- 1 linux linux 12 Apr 20 03:09 test1
-rw-rw-r-- 1 linux linux 22 Apr 20 03:09 test2

linux@ubuntu:~/UNIX_learning$ ./futimens test1 test2	运行测试程序
linux@ubuntu:~/UNIX_learning$ date					打印当天日期
Tue Apr 20 03:11:40 PDT 2021

linux@ubuntu:~/UNIX_learning$ ls -l test1 test2		检查结果
-rw-rw-r-- 1 linux linux 0 Apr 20 03:10 test1
-rw-rw-r-- 1 linux linux 0 Apr 20 03:11 test2

linux@ubuntu:~/UNIX_learning$ ls -lu test1 test2	检查最后访问时间
-rw-rw-r-- 1 linux linux 0 Apr 20 03:09 test1
-rw-rw-r-- 1 linux linux 0 Apr 20 03:09 test2

linux@ubuntu:~/UNIX_learning$ ls -lc test1 test2	检查状态更改时间
-rw-rw-r-- 1 linux linux 0 Apr 20 03:11 test1
-rw-rw-r-- 1 linux linux 0 Apr 20 03:11 test2
```

最后修改时间和最后访问时间未变。但是，状态更改时间则更改为程序运行时间。