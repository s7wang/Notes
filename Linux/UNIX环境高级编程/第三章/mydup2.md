# 习题 3.2

编写一个与 3.12 节中 dup2 功能相同的函数，要求不调用 fcntl 函数，并且要有出错处理。



解题：dup2() 用来复制参数 oldfd 所指的文件描述符, 并将它拷贝至参数 newfd 后一块返回. 若参数 newfd为一已打开的文件描述符, 则 newfd 所指的文件会先被关闭. dup2() 所复制的文件描述符, 与原来的文件描述符共享各种文件状态, 详情可参考 dup().

所以，在不使用 fcntl 的情况下， 我们只能通过不断获取/占用较小的可用文件描述符，使得可用文件描述与目标文件描述相等，来实现文件描述符的复制。在这个过程中考虑到我们不断获取的最小文件描述符不一定是从 0 开始的（基本一定不是从 0 开始，通常 0、1、2被默认打开），也不一定是连续的，所以需要使用一个长度为能打开最大的文件数量的整型数值来记录我们已经打开但并不使用的文件描述符，在实现复制目标文件描述符到指定的 newfd 后，依次将这个数组中记录的文件描述符关闭。



代码：

```c
/**
 * @file mydup2.c
 * @author wangs7
 * @brief Write a function with the same function as dup2 in section 3.12, 
 *        which requires no call to fcntl function and error handling.
 * @version 0.1
 * @date 2021-04-15
 * 
 * @copyright Copyright (c) 2021
 * 
 */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <limits.h>
#include <sys/resource.h>

/* Guess file open max number */
#define OPEN_MAX_GUESS  256

/* Get file open max number */
long open_max(void);
/* My dup2 function */
int mydup2(int oldfd, int newfd);

/* Test function main */
int main (int argc, char *argv[]) {

    int fd = 0;
    const int newfd = 10;

    char s[32] = "This is stdout.\n";

    /* mydup2 test */
    fd = mydup2(1, newfd); //Redirect stdout to newfd (10)

    /* result check */
    if (newfd == fd) {
        /* fd is same as stdout. */
        write(fd, "mydup2 ok!\n", 12);
        fflush(NULL);
    }
    else {
        /* print "mydup2 fail!\n" */
        printf("mydup2 fail!\n");
    }
    /* print s on terminal. */
    write(fd, s, 32);
        

    exit(0);
}

long open_max(void) {
    long openmax;
    struct rlimit rl;

    if ( ( openmax = sysconf(_SC_OPEN_MAX)) < 0 ||
    openmax == LONG_MAX ) {
        if (getrlimit(RLIMIT_NOFILE, &rl) <0) 
            printf("can't get file limit.\n");
        if (rl.rlim_max == RLIM_INFINITY)
            openmax = OPEN_MAX_GUESS;
        else
            openmax = rl.rlim_max;
    }

    return openmax;

}

/* my dup2 function */
int mydup2(int oldfd, int newfd) {
    
    const long OPEN_MAX = open_max();
    int fd;  //result fd

    /* count and record opened fds*/
    int fds[OPEN_MAX];
    int count;

    /* Input check */
    if (oldfd < 0 || newfd < 0) {
        fprintf(stderr, "Parameter input error!\n");
        return -1;
    }

    /* If oldfd == newfd */
    if (oldfd == newfd) {
        return newfd; //do nothing and return newfd.
    }

    /* if oldfd != newfd*/
    close(newfd); //first close newfd anyway.
    count = 0;
    while ( count < OPEN_MAX ) {
        fds[count] = dup(oldfd);
        if (fds[count] == newfd) {
            fd = fds[count];
            break;
        }
        count++;
    }
    
    /* close other files not need but opened */
    for (int i = 0; i < count; i++) {
        close(fds[i]);
    }

    /* return fd */
    return fd;

}

```



结果：

```
linux@ubuntu:~/UNIX_learning/Unit_3$ make mydup2
cc     mydup2.c   -o mydup2
linux@ubuntu:~/UNIX_learning/Unit_3$ ./mydup2 
mydup2 ok!
This is stdout.
```

