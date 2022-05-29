# 例程：遍历文件层次结构的程序

目的：统计某一路径下的各种文件的数量，其功能类似 `ftw(3)` ，我们编写的 `myftw` 只是简单的遍历文件目录。下面是 `ftw(3)` 的简介和我们的历程。

* `ftw(3)` 

```c
FTW(3)                
NAME
       ftw, nftw - file tree walk

SYNOPSIS
       #include <ftw.h>
    
       int nftw(const char *dirpath,
               int (*fn) (const char *fpath, const struct stat *sb,
                          int typeflag, struct FTW *ftwbuf),
               int nopenfd, int flags);

       #include <ftw.h>

       int ftw(const char *dirpath,
               int (*fn) (const char *fpath, const struct stat *sb,
                          int typeflag),
               int nopenfd);

   Feature Test Macro Requirements for glibc (see feature_test_macros(7)):
       nftw(): _XOPEN_SOURCE >= 500

DESCRIPTION
       nftw()  walks  through  the  directory tree  that is located  under 
       the directory dirpath, andcalls fn() once for each entry in the tree.
       By default, directories are handled before the files and subdirectories
       they contain (preorder traversal). To avoid using up all of the calling
       process's  file descriptors, nopenfd specifies the  maximum  number  of 
       directories that nftw() will hold open simultaneously. When the  search
       depth exceeds this, nftw() will become slower  because directories have 
       to be closed and reopened. nftw() uses at most one file  descriptor for
       each level in the directory tree.

       For each entry found in the tree, nftw() calls fn() with four arguments: 
       fpath, sb, typeflag, and ftwbuf. fpath is the pathname of the entry, and 
       is  expressed  either as a  pathname relative  to the calling  process's 
       current working directory at the time of the  call to nftw(), if dirpath 
       was  expressed  as  a  relative pathname, or as an absolute pathname, if 
       dirpath was expressed as an absolute pathname.  sb is  a  pointer to the 
       stat structure returned by a  call to stat(2) for fpath.
```



我们的实现 `myftw` ：（代码后边会有说明）

* myftw.c

```c
/**
 * @file myftw.c
 * @author wangs7
 * @brief Traversing file hierarchy
 * @version 0.1
 * @date 2021-04-20
 * 
 * @copyright Copyright (c) 2021
 * 
 */

/**
 *  NAME
 *      ftw, nftw - file tree walk
 *
 *  SYNOPSIS
 *      #include <ftw.h>
 *
 *      int nftw(const char *dirpath,
 *              int (*fn) (const char *fpath, const struct stat *sb,
 *                          int typeflag, struct FTW *ftwbuf),
 *              int nopenfd, int flags);
 *      #include <ftw.h>
 * 
 *      int ftw(const char *dirpath,
 *              int (*fn) (const char *fpath, const struct stat *sb,
 *                         int typeflag),
 *              int nopenfd);
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>
#include <limits.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include "path_alloc.h"

#define FTW_F   1 /* file other than directory */
#define FTW_D   2 /* directory */
#define FTW_DNR 3 /* directory that can't be read */
#define FTW_NS  4 /* file that we can't stat */

typedef int Myfunc(const char * , const struct stat * , int );

/* function declaration */
static Myfunc myfunc;

static int myftw(const char * , Myfunc * );
static int dopath(Myfunc * );

static char *fullpath; /* contains full pathname for every file */
static size_t pathlen;

static long nreg, ndir, nblk, nchr, nfifo, nslink, nsock, ntot; /* counts the number of various documents */

int main (int argc, char *argv[]) {

    int ret;
    if (argc != 2) {
        fprintf(stderr, "usage: myftw <starting-pathname>\n");
        exit(1);
    }
    ret = myftw(argv[1], myfunc); /* does it all */
    ntot = nreg + ndir + nblk + nchr + nfifo + nslink + nsock;
    if (ntot == 0) {
        ntot = 1; /* avoid divide by 0; print 0 for all counts */
    }
    printf("regular files   =   %7ld, %5.2f %%\n", nreg, nreg * 100.0 / ntot);
    printf("directories     =   %7ld, %5.2f %%\n", ndir, ndir * 100.0 / ntot);
    printf("block special   =   %7ld, %5.2f %%\n", nblk, nblk * 100.0 / ntot);
    printf("char special    =   %7ld, %5.2f %%\n", nchr, nchr * 100.0 / ntot);
    printf("FIFOs           =   %7ld, %5.2f %%\n", nfifo, nfifo * 100.0 / ntot);
    printf("symbolic links  =   %7ld, %5.2f %%\n", nslink, nslink * 100.0 / ntot);
    printf("sockets         =   %7ld, %5.2f %%\n", nsock, nsock * 100.0 / ntot);

    return 0;
}

static int myftw(const char *pathname, Myfunc *func ) {
    fullpath = path_alloc(&pathlen); /* malloc PATH_MAX + 1 bytes */
                                     /* ({Flgure 2.16}) */   
    if (pathlen <= strlen(pathname)) {
        pathlen = strlen(pathname) * 2;
        if ((fullpath = realloc(fullpath, pathlen)) == NULL) {
            fprintf(stderr, "realloc failed.\n");
        }
    }
    strncpy(fullpath, pathname, pathlen);
    return dopath(func);
}

static int dopath(Myfunc *func ) {
    int ret, n;
    struct stat statbuf;
    struct dirent *dirp;
    DIR *dp;
    if (lstat(fullpath, &statbuf) < 0) /* stat error */
        return(func(fullpath, &statbuf, FTW_NS));
    if (S_ISDIR(statbuf.st_mode) == 0) /* not a directory */
        return(func(fullpath, &statbuf, FTW_F));

    /**
     * @brief 
     * It's a directory. First call func() for the directory.
     * then process each filename in the directory.
     */
    if ((ret = func(fullpath, &statbuf, FTW_D)) != 0){
        return ret;
    }
    n = strlen(fullpath);
    if (n + NAME_MAX + 2 > pathlen) { /* expand path buffer */
        pathlen *= 2;
        if ((fullpath = realloc(fullpath, pathlen)) == NULL) {
            fprintf(stderr, "realloc failed!\n");
        }
    }
    fullpath[n++] = '/';
    fullpath[n] = 0;

    if ((dp = opendir(fullpath)) == NULL) { /* can't read directory */
        return (func(fullpath, &statbuf, FTW_DNR));
    }
    
    while ((dirp = readdir(dp)) != NULL) {
        
        if (strcmp(dirp->d_name, ".") == 0 ||
            strcmp(dirp->d_name, "..") == 0) {
                continue; /* ignore dot and dot-dot */
        }
        strcpy(&fullpath[n], dirp->d_name); /* append name after "/" */
        if ((ret = dopath(func)) != 0) {  /* recursive */
            break; /* time to leave */
        }
    }

    fullpath[n-1] = 0; /* erase everything from slash oneard */
    if (closedir(dp) < 0) {
        fprintf(stderr, "can't close directory %s\n", fullpath);
    }
    
    return ret;
}

static int myfunc (const char *pathname, const struct stat *statptr, int type) {

    switch (type) {
        case FTW_F:
            switch (statptr->st_mode & S_IFMT) {
                case S_IFREG:   nreg++;     break;
                case S_IFBLK:   nblk++;     break;
                case S_IFCHR:   nchr++;     break;
                case S_IFIFO:   nfifo++;    break;
                case S_IFLNK:   nslink++;   break;
                case S_IFSOCK:  nsock++;    break;
                case S_IFDIR:   /* directories should have type = FTW_D */
                    fprintf(stderr, "for S_IFDIR for %s\n", pathname);
            }
            break;
        case FTW_D:
            ndir++; 
            break;
        case FTW_DNR:
            fprintf(stderr, "can't read directory %s\n", pathname);
            break;
        case FTW_NS:
            fprintf(stderr, "stat error for %s\n", pathname);
            break;
        default:
            fprintf(stderr, "unknow type %d for pathname %s\n", type, pathname);


    }

    return 0;
}
```

分析：

1. 调用逻辑：除去主函数函数和包含的头文件，但看我们主要实现函数： `myftw()` `dopath()` 和 `myfunc()` ，其中 `myftw()` 是我们的实现目标，`myfunc()` 是我们调用 `myftw()` 是所需要的参数（一个文件类型的判断和统计，其结果存放在再文件开头声明的静态变量里）。`myftw()` 内主要实现的是文件目录的拷贝，然后调用 `dopath()` 实现 **递归** 的文件目录遍历。
2. `int myftw(const char *pathname, Myfunc *func )` ：返回值为 int 其实这里的返回值总是 0 ，我在后边会说明。参数有两个，第一个字符串常量，为我们要遍历的路径名；第二个为统计函数的入口地址。在 `myftw` 中，首先将 路径拷贝到 `fullpathname` 中， `fullpathname` 是一个全局变量，其过程中调用了一个函数 `path_alloc(&pathlen)` 目的是给字符串指针分配空间，并将空间大小赋值给 `pathlen` ，具体实现在 path_alloc.h 会在文章结尾给出。最后调用 `dopath(func)` 并返回该函数的返回值。
3. `int dopath(Myfunc *func )` ：返回值为 int ，为了简化程序，虽然该函数中存在错误提示但是不做错误处理，故返回值总是 0，这也导致了 `myftw` 的返回值总是 0。参数为返回值为 int（MyFunc）类型的函数指针，这里的 `*func` 其实就是下面的函数 `myfunc`，作为统计数量的函数。首先尝试获取路径的属性，如果获取失败，返回 `func(fullpath, &statbuf, FTW_NS)` ，这里的 `FTW_DNR` 为标志位，用于告诉 `func` 该文件无法打开；如果该路径不是目录文件，那么返回 `func(fullpath, &statbuf, FTW_F)` ，这里的标志位 `FTW_F` 将告诉 `func` 该路径是一个文件，具体是什么文件将由 `func` 判断；除上述情况，该路径必定为目录文件，我们首先调用 `func(fullpath, &statbuf, FTW_D)` 告诉  `func` 该路径是一个目录文件，先进行记录，然后判断路径字符串的空间是否充足并进行扩容，随后打开目录并对该目录的内容进行读取，对目录的每一项**递归**调用 `dopath` ；最后关闭该目录。以此实现对目录的遍历。
4. `int myfunc (const char *pathname, const struct stat *statptr, int type)` ：返回值同样为 int 。第一个参数为文件路径，第二个为文件属性结构体的指针，第三个参数为 标志位，用于告诉 `func` 对该文件该采取什么行为。这个函数只是一个简单分类统计，比较简单不做过多说明。

* 运行测试

```
linux@ubuntu:~/UNIX_learning/Unit_4$ ./myftw /home/linux/UNIX_learning
regular files   =        15, 71.43 %
directories     =         6, 28.57 %
block special   =         0,  0.00 %
char special    =         0,  0.00 %
FIFOs           =         0,  0.00 %
symbolic links  =         0,  0.00 %
sockets         =         0,  0.00 %
linux@ubuntu:~/UNIX_learning/Unit_4$ 
linux@ubuntu:~/UNIX_learning/Unit_4$ sudo ./myftw /sys
[sudo] password for linux: 
regular files   =     72800, 78.73 %
directories     =     14084, 15.23 %
block special   =         0,  0.00 %
char special    =         1,  0.00 %
FIFOs           =         0,  0.00 %
symbolic links  =      5581,  6.04 %
sockets         =         0,  0.00 %
linux@ubuntu:~/UNIX_learning/Unit_4$ 
```

结果基本正确，当然如果想打印目录结构也是可以的。



* 附件 path_alloc.h

```c
#ifndef PATH_ALLOC_H__
#define PATH_ALLOC_H__

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h> 
#include <limits.h> 
#ifdef   PATH_MAX 
static int   pathmax = PATH_MAX; 
#else 
static int   pathmax = 0; 
#endif 
#define SUSV3 200112L 
static long posix_version = 0; 
/* If PATH_MAX is indeterminate, no guarantee this is adequate */ 
#define PATH_MAX_GUESS   1024 
char * 
path_alloc(size_t *sizep) /* also return allocated size, if nonnull */ 
{ 
       char *ptr; 
       size_t size; 
       if (posix_version == 0) 
              posix_version = sysconf(_SC_VERSION); 
       if (pathmax == 0) {     /* first time through */ 
              errno = 0; 
              if ((pathmax = pathconf("/", _PC_PATH_MAX)) < 0) { 
              if (errno == 0) 
              pathmax = PATH_MAX_GUESS; /* it's indeterminate */ 
              else 
              fprintf(stderr, "pathconf error for _PC_PATH_MAX"); 
              } else { 
              pathmax++;    /* add one since it's relative to root */ 
              } 
       } 
       if (posix_version < SUSV3) 
              size = pathmax + 1; 
       else 
              size = pathmax; 
       if ((ptr = (char *)malloc(size)) == NULL) 
              fprintf(stderr, "malloc error for pathname"); 
       if (sizep != NULL) 
              *sizep = size; 
       return(ptr); 
} 

#endif
```





