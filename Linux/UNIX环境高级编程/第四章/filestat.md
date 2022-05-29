# 例程：获取文件属性



代码：

```c
/**
 * @file file_stat.c
 * @author wangs7
 * @brief Get file properties by lstat.
 * @version 0.1
 * @date 2021-04-16
 * 
 * @copyright Copyright (c) 2021
 * 
 */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <time.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/sysmacros.h>


int main (int argc, char *argv[]) {

        int i;
        struct stat buf;
        /*  <sys/stat.h>
         *  struct stat {
         *          dev_t     st_dev;         // ID of device containing file 
         *          ino_t     st_ino;         // Inode number 
         *          mode_t    st_mode;        // File type and mode 
         *          nlink_t   st_nlink;       // Number of hard links 
         *          uid_t     st_uid;         // User ID of owner 
         *          gid_t     st_gid;         // Group ID of owner 
         *          dev_t     st_rdev;        // Device ID (if special file) 
         *          off_t     st_size;        // Total size, in bytes 
         *          blksize_t st_blksize;     // Block size for filesystem I/O 
         *          blkcnt_t  st_blocks;      // Number of 512B blocks allocated 
         *
         *          /  Since Linux 2.6, the kernel supports nanosecond
         *             precision for the following timestamp fields.
         *             For the details before Linux 2.6, see NOTES. /
         *
         *          struct timespec st_atim;  // Time of last access 
         *          struct timespec st_mtim;  // Time of last modification 
         *          struct timespec st_ctim;  // Time of last status change 
         *
         *      #define st_atime st_atim.tv_sec      // Backward compatibility 
         *      #define st_mtime st_mtim.tv_sec
         *      #define st_ctime st_ctim.tv_sec
         *      };
        **/

        char *ptr;

        /* Input check. */
        if (argc < 1) {
        printf("Parameter usage error...\n");
        exit(1);
        }

        for (i = 1; i < argc; i++) {
                /* Print file name. */
                printf("%s: ", argv[i]);

                /* Get stat of file */
                if (lstat(argv[i], &buf) < 0) {
                        printf("lstat error!\n");
                        continue;
                }

                /* Print dev number. */
                printf("ID of containing device:  [%lx,%lx]\n",
                        (long) major(buf.st_dev), (long) minor(buf.st_dev));

                /* Print file type. */
                printf("File type:                ");
                /* Check file type. */
                switch (buf.st_mode & S_IFMT) {  //S_IFMT is File type mask
                        case S_IFBLK:  printf("block device\n");            break;
                        case S_IFCHR:  printf("character device\n");        break;
                        case S_IFDIR:  printf("directory\n");               break;
                        case S_IFIFO:  printf("FIFO/pipe\n");               break;
                        case S_IFLNK:  printf("symlink\n");                 break;
                        case S_IFREG:  printf("regular file\n");            break;
                        case S_IFSOCK: printf("socket\n");                  break;
                        default:       printf("unknown?\n");                break;
                }

                /* Print inode number. */
                printf("I-node number:            %ld\n", (long) buf.st_ino);

                /* Print mode mask. */
                printf("Mode:                     %lo (octal)\n",
                        (unsigned long) buf.st_mode);
                
                printf("Link count:               %ld\n", (long) buf.st_nlink);
                printf("Ownership:                UID=%ld   GID=%ld\n",
                        (long) buf.st_uid, (long) buf.st_gid);

                printf("Preferred I/O block size: %ld bytes\n",
                        (long) buf.st_blksize);
                printf("File size:                %lld bytes\n",
                        (long long) buf.st_size);
                printf("Blocks allocated:         %lld\n",
                        (long long) buf.st_blocks);
                /* Time information. */
                printf("Last status change:       %s", ctime(&buf.st_ctime));
                printf("Last file access:         %s", ctime(&buf.st_atime));
                printf("Last file modification:   %s\n", ctime(&buf.st_mtime));

        }

        exit(0);
}
```



结果：

```
linux@ubuntu:~/UNIX_learning/Unit_4$ make file_stat
cc     file_stat.c   -o file_stat
linux@ubuntu:~/UNIX_learning/Unit_4$ ./file_stat file_stat.c
file_stat.c: ID of containing device:  [8,1]
File type:                regular file
I-node number:            11279115
Mode:                     100664 (octal)
Link count:               1
Ownership:                UID=1000   GID=1000
Preferred I/O block size: 4096 bytes
File size:                2190 bytes
Blocks allocated:         8
Last status change:       Fri Apr 16 01:48:15 2021
Last file access:         Fri Apr 16 01:48:17 2021
Last file modification:   Fri Apr 16 01:48:15 2021
```

