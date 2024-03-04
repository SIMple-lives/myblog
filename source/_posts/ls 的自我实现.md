---
title: ls的自我实现
---

# myls

> `ls` 是一个常用的命令行工具，用于列出指定目录中的文件和子目录。`ls` 的基本语法是:

```
ls [选项] [文件或目录]
```

以下是一些常用的 `ls` 命令选项：

1. **-l**：以长格式显示文件信息，包括文件类型、权限、所有者、组、大小、修改时间等。

   ```
   ls -l
   ```

2. **-a**：显示所有文件，包括隐藏文件（以点开头的文件）。

   ```
   ls -a
   ```

3. **-r**：与 `-l` 逆序输出文件信息。

   ```
   ls -r
   ```

4. **-R**：递归显示子目录中的文件。

   ```
   ls -R
   ```

5. **-t**：按修改时间排序，最新修改的文件显示在前面。

   ```
   ls -lt
   ```

6. **-S**：按文件大小排序，最大的文件显示在前面。

   ```
   ls -lS
   ```

7. **-i**：显示文件的 inode 号码。

   ```
   ls -i
   ```

## 思路

1.-i -a -R -r -i -s -l为命令行参数，我们首先需要解析命令行参数。

2.我们需要对以上参数进行了解，并进行构思一个框架以便后续对代码的维护与可读性。

3.我觉得可以将以上参数分为三类:1.-a 为确定需要显示的文件多少，例如-a包含 . 隐藏文件。2.-s -t 排序参数，如果含有该参数，则就需要对文件显示进行排序。3.-i -l 为输出参数，即在输出的时候进行判断是否需要输出对应的信息.

4.ls 还可以指定文件进行查看，我们还需要对不同的文件或目录进行操作.

## 相关函数

* **getopt**,**getcwd**

* **opendir**,**readdir**,**closedir**

* **snprintf**

* **stat**,**lstat**

  

## 主函数

```c
enum order {
    a, s, t, r, I, l, R
};//枚举命令行参数。提高代码的可读性.

int main(int argc, char *argv[])
{
    int find = 0;//用来接收getopt的返回值
    int orders[8] = {0};//存储对应参数的信息
    for (int i = 1; i < argc; i++)
    {
        if (argv[i][0] == '-')
        {
            while ((find = getopt(argc, argv, "aIlrstR")) != -1)//解析命令行参数
            {
                switch (find)//将对应的参数信息残赋值为1.
                {
                case 'a':
                    orders[a] = 1;
                    break;
                case 'r':
                    orders[r] = 1;
                    break;
                case 's':
                    orders[s] = 1;
                    break;
                case 't':
                    orders[t] = 1;
                    break;
                case 'l':
                    orders[l] = 1;
                    break;
                case 'R':
                    orders[R] = 1;
                    break;
                case 'I':
                    orders[I] = 1;
                    break;
                default:
                    fprintf(stderr, "Invalid option\n");
                    exit(EXIT_FAILURE);
                }
            }
        }
    }

    int countfile = 0;
    for (int i = 1; i < argc; i++)//获取文件的数量.
    {
        if (argv[i][0] != '-')
        {
            countfile++;
        }
    }

    if (countfile == 0)
    {
        char current_path[MAX_PATH];//存储当前的工作路径
        if ((getcwd(current_path, MAX_PATH)) != NULL)
        {
            if (orders[R] == 0)
            {
                do_open(current_path, orders);
            }
            else
            {
                do_open_R(current_path, orders);
            }
        }
        else
        {
            perror("getcwd");
            exit(EXIT_FAILURE);
        }
    }

    for (int i = 1; i < argc; i++)
    {
        if (argv[i][0] != '-')
        {
            if (countfile > 1)
            {
                printf("%s\n", argv[i]);
            }
            if (orders[R] == 0)
            {
                do_open(argv[i], orders);
            }
            else
            {
                do_open_R(argv[i],orders);
            }
        }
    }
}
```

了解了ls的基本语法与功能后，我们就需要进行实现.

那么就首先需要解析命令行参数.获取我们本次操作需要进行的工作.

### getopt

该函数专门用来解析命令行参数.

```c
int getopt(int argc, char * const argv[], const char *optstring);
```

**参数说明**

* argc：通常由 main 函数直接传入，表示参数的数量

* argv：通常也由 main 函数直接传入，表示参数的字符串变量数组

* optstring：一个包含正确的参数选项字符串，用于参数的解析。例如 “abc:”，其中 -a，-b 就表示两

  个普通选项，-c 表示一个必须有参数的选项，因为它后面有一个冒号.

  

[详细用法]: https://blog.csdn.net/afei__/article/details/81261879?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170324193316800192231385%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&amp;request_id=170324193316800192231385&amp;biz_id=0&amp;utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-81261879-null-null.142^v96^pc_search_result_base9&amp;utm_term=getopt&amp;spm=1018.2226.3001.4187

这里命令行参数已经解析，那么我们接下来就要进行对要查看的文件或目录数量进行统计.

**如果数量大于2,就需要在进行操作前输出本次查看的文件或目录路径.**

### getcwd

```
#include<unistd.h>
char *getcwd(char *buf,size_t size);
```

介绍：
参数说明：getcwd()会将当前工作目录的绝对路径复制到参数[buffer](https://so.csdn.net/so/search?q=buffer&spm=1001.2101.3001.7020)所指的内存空间中,参数size为buf的空间大小。

如果未传入文件或目录，就需要对当前路径下的文件进行操作.所以需要用到该函数.

***

现在已经获取到需要执行的参数和文件，接下来就需要进行参数操作.

所有参数中最难的就是递归部分，所以我们单独为-R封装一个函数来进行递归操作.

## do_open

```c
struct fileinfo//由于-l需要输出文件的详细信息，所以需要结构体来存储信息
{
    unsigned int i_node;
    char permission[16];
    short owner;
    short group;
    off_t size;
    time_t c_ctime;
    time_t m_ctime;
    nlink_t link_num;
    char name[MAX_PATH];
    mode_t mode;
    blkcnt_t block;
};

void do_open(char *path, int orders[])
{
    int count = 0;
    struct fileinfo infos[MAX_FILE];//我们的-s -t -r参数需要对文件进行排序，那么我们就需要数组来存储文件相关信息。
    if (orders[a])
    {
        ls_a(path, infos, &count);//ls-用来打开文件以及存储文件名称
    }
    if (orders[a] == 0)
    {
        only_ls(path, infos, &count);
    }
    if (orders[t])
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_time);//对存储的信息进行相关的排序.t的优先级高于s
    }
    if ((orders[s] == 1) && (orders[t] == 0))
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_size);
    }
    if ((orders[s] == 0) && orders[t] == 0)
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_name);//按名称排序，使文字命名文件在前
    }
    show_orders(infos, count, orders);//最后进行输出.
}
```

既然获取到了参数与文件，所以我们的do_open就需要接受这两类参数.

我们在do_open中进行文件打开，排序参数，以及展示.

count参数用于计数文件个数，便于输出结果.

## do_open_R

```
void do_open_R(char *path, int orders[])
{
    int count = 0;
    struct fileinfo *infos = (struct fileinfo *)malloc(sizeof(struct fileinfo) * MAX_RFILE);
    if (infos == NULL)
    {
        perror("malloc");
        exit(EXIT_FAILURE);
    }
    if (orders[a])
    {
        ls_a(path, infos, &count);
    }
    if (orders[a] == 0)
    {
        only_ls(path, infos, &count);
    }
    if (orders[t])
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_time);
    }
    if ((orders[s] == 1) && (orders[t] == 0))
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_size);
    }
    if ((orders[s] == 0) && orders[t] == 0)
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_name);
    }
    show_orders(infos, count, orders);
    for (int i = 0; i < count; i++)
    {
        if (S_ISDIR(infos[i].mode) && strcmp(infos[i].name, ".") != 0 && strcmp(infos[i].name, "..") != 0)
        {
            char sub_path[MAX_PATH];
            snprintf(sub_path, sizeof(sub_path), "%s/%s", path, infos[i].name);
            struct stat flag;
            if (lstat(sub_path, &flag) == -1)
            {
                fprintf(stderr, "Permission denied or error reading: %s\n", sub_path);
                continue;
            }
            if(S_ISLNK(flag.st_mode))
            {
                continue;
            }
            do_open_R(sub_path,orders);
        }
    }
    free(infos);
}
```

**由于-R参数需要递归，所以采取堆上开辟空间进行存储**

这里多了一个判断，如果存储的路径对应的仍为目录，则需要继续进行简单的do_open操作。

所以S_ISDIR用来判断是否是目录，如果是则进行递归操作。

S_ISLNK用来判断文件是否为符号链接，如果不加该判断，会在对 / 目录下/dev/fd中的符号链接进行叠加路径而出现错误.

## ls_a

```c
void ls_a(char *path, struct fileinfo *infos, int *count)
{
    DIR *dir;//接收opendir的返回值.
    struct dirent *dirp;//readdir的返回值
    if ((dir = opendir(path)) == NULL)
    {
        printf("%s\n", path);
        return ;
    }
    *count = 0;
    struct stat st;
    while ((dirp = readdir(dir)) != NULL)
    {
        char current_path[MAX_PATH];
        snprintf(current_path, sizeof(current_path), "%s/%s", path, dirp->d_name);
        
        if ((stat(current_path, &st)) != -1)//存储文件的相关信息。
        {
            
            strncpy(infos[*count].name, dirp->d_name, sizeof(infos[*count].name) - 1);
            infos[*count].name[sizeof(infos[*count].name) - 1] = '\0';  // Null-terminate the string
            infos[*count].group = st.st_gid;
            infos[*count].owner = st.st_uid;
            infos[*count].mode = st.st_mode;
            infos[*count].c_ctime = st.st_ctime;
            infos[*count].size = st.st_size;
            infos[*count].m_ctime = st.st_mtime;
            infos[*count].i_node = st.st_ino;
            mode_to_letters(st.st_mode, infos[*count].permission);
            infos[*count].link_num = st.st_nlink;
            infos[*count].block=st.st_blocks;
        }
        else
        {
            perror("stat");
            continue;
        }
        (*count)++;
    }
    closedir(dir);//一定要记得关闭文件
}
```

由于需要将信息进行存储，所以ls_a需要接收结构体数组，count 用于记录文件个数.

### opendir

opendir()函数用于打开一个目录，并返回指向该目录的句柄，供后续操作使用。	

```c
#include <dirent.h>
DIR * opendir(const char * dirpath);
							Returns directory stream handlc,or NULL on error
```

### readdir

readdir()函数从一个目录流中读取连续的条目.

```c
#include <dirent.h>
struct dirent *readdir(DIR * dirp);
							Returns pointer to a statically allocated structure describing next directory entry,or NULL on end-of-directory or error
```

每调用 readdir()一次，就会从 dirp 所指代的目录流中读取下一目录条目，并返回一枚指针， 指向经静态分配而得的 dirent 类型结构，内含与该条目相关的如下信息： 每次调用 readdir()都会覆盖该结构。

### closedir

closedir()函数用于关闭处于打开状态的目录，同时释放它所使用的资源.(一定要记得关闭文件，否则就会造成内存泄漏).

```c
int closedir(DIR *dirp);
```

### stat (获取文件信息)

```c
#include <sys/types.h> 
#include <sys/stat.h> 
#include <unistd.h> 
 
int stat(const char *pathname, struct stat *buf); 
```

函数参数及返回值含义如下：

> * pathname：用于指定一个需要查看属性的文件路径。
> * buf：struct stat 类型指针，用于指向一个 struct stat 结构体变量。调用 stat 函数的时候需要传入一个 struct stat 变量的指针，获取到的文件属性信息就记录在 struct stat 结构体中。
> * 返回值：成功返回 0；失败返回-1，并设置 error。

#### struct stat

```c
struct stat 
{ 
 dev_t st_dev; /* 文件所在设备的 ID */ 
 ino_t st_ino; /* 文件对应 inode 节点编号 */ 
 mode_t st_mode; /* 文件对应的模式 */ 
 nlink_t st_nlink; /* 文件的链接数 */ 
 uid_t st_uid; /* 文件所有者的用户 ID */ 
 gid_t st_gid; /* 文件所有者的组 ID */ 
 dev_t st_rdev; /* 设备号（指针对设备文件） */ 
 off_t st_size; /* 文件大小（以字节为单位） */ 
 blksize_t st_blksize; /* 文件内容存储的块大小 */ 
 blkcnt_t st_blocks; /* 文件内容所占块数 */ 
 struct timespec st_atim; /* 文件最后被访问的时间 */ 
 struct timespec st_mtim; /* 文件内容最后被修改的时间 */ 
 struct timespec st_ctim; /* 文件状态最后被改变的时间 */
};
```

## snprintf

`snprintf` 是一个 C 语言标准库函数，用于格式化字符串并将结果存储到一个字符数组中

```c++
int snprintf(char *str, size_t size, const char *format, ...);	
```

- `tr`: 指向存储结果的字符数组的指针。
- `size`: 最大允许写入的字符数（包括字符串终止符）。
- `format`: 格式化字符串，包含了要被写入到字符串中的文本和格式说明符。
- `...`: 可变数量的参数，用于替换格式字符串中的格式说明符。

`snprintf` 的行为类似于 `printf`，但是它会限制输出字符的数量，以防止溢出缓冲区。如果成功，`snprintf` 返回写入字符的总数（不包括字符串终止符），如果输出被截断，它会返回实际尝试写入的字符数。如果发生错误，返回负值。

# 完整代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <pwd.h>
#include <grp.h>
#include <time.h>
#include <unistd.h>
#include <locale.h>
#include <ctype.h>
#include <sys/ioctl.h>
#include <stdbool.h>
#include <getopt.h>
#include <error.h>

#define MAX_PATH 1024
#define MAX_FILE 512
#define MAX_RFILE 60000

enum order {
    a, s, t, r, I, l, R
};

struct fileinfo
{
    unsigned int i_node;
    char permission[16];
    short owner;
    short group;
    off_t size;
    time_t c_ctime;
    time_t m_ctime;
    nlink_t link_num;
    char name[MAX_PATH];
    mode_t mode;
    blkcnt_t block;
};

void do_open(char *path, int orders[]);
void ls_a(char *path, struct fileinfo *infos, int *count);
void only_ls(char *path, struct fileinfo *infos, int *count);
void mode_to_letters(mode_t mode, char modestr[]);
int cmp_name(const void *a, const void *b);
int cmp_time(const void *a, const void *b);
int cmp_size(const void *a, const void *b);
void show_orders(struct fileinfo *infos, int count, int orders[]);
char *uid_to_name(uid_t uid);
char *gid_to_name(gid_t gid);
void do_open_R(char *path, int orders[]);
void print(struct fileinfo infos);

int main(int argc, char *argv[])
{
    int find = 0;
    int orders[8] = {0};
    for (int i = 1; i < argc; i++)
    {
        if (argv[i][0] == '-')
        {
            while ((find = getopt(argc, argv, "aIlrstR")) != -1)
            {
                switch (find)
                {
                case 'a':
                    orders[a] = 1;
                    break;
                case 'r':
                    orders[r] = 1;
                    break;
                case 's':
                    orders[s] = 1;
                    break;
                case 't':
                    orders[t] = 1;
                    break;
                case 'l':
                    orders[l] = 1;
                    break;
                case 'R':
                    orders[R] = 1;
                    break;
                case 'I':
                    orders[I] = 1;
                    break;
                default:
                    fprintf(stderr, "Invalid option\n");
                    exit(EXIT_FAILURE);
                }
            }
        }
    }

    int countfile = 0;
    for (int i = 1; i < argc; i++)
    {
        if (argv[i][0] != '-')
        {
            countfile++;
        }
    }

    if (countfile == 0)
    {
        char current_path[MAX_PATH];
        if ((getcwd(current_path, MAX_PATH)) != NULL)
        {
            if (orders[R] == 0)
            {
                do_open(current_path, orders);
            }
            else
            {
                do_open_R(current_path, orders);
            }
        }
        else
        {
            perror("getcwd");
            exit(EXIT_FAILURE);
        }
    }

    for (int i = 1; i < argc; i++)
    {
        if (argv[i][0] != '-')
        {
            if (countfile > 1)
            {
                printf("%s\n", argv[i]);
            }
            if (orders[R] == 0)
            {
                do_open(argv[i], orders);
            }
            else
            {
                do_open_R(argv[i],orders);
            }
        }
    }
}

void do_open(char *path, int orders[])
{
    int count = 0;
    struct fileinfo infos[MAX_FILE];
    if (orders[a])
    {
        ls_a(path, infos, &count);
    }
    if (orders[a] == 0)
    {
        only_ls(path, infos, &count);
    }
    if (orders[t])
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_time);
    }
    if ((orders[s] == 1) && (orders[t] == 0))
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_size);
    }
    if ((orders[s] == 0) && orders[t] == 0)
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_name);
    }
    show_orders(infos, count, orders);
}

void do_open_R(char *path, int orders[])
{
    int count = 0;
    struct fileinfo *infos = (struct fileinfo *)malloc(sizeof(struct fileinfo) * MAX_RFILE);
    if (infos == NULL)
    {
        perror("malloc");
        exit(EXIT_FAILURE);
    }
    if (orders[a])
    {
        ls_a(path, infos, &count);
    }
    if (orders[a] == 0)
    {
        only_ls(path, infos, &count);
    }
    if (orders[t])
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_time);
    }
    if ((orders[s] == 1) && (orders[t] == 0))
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_size);
    }
    if ((orders[s] == 0) && orders[t] == 0)
    {
        qsort(infos, count, sizeof(struct fileinfo), cmp_name);
    }
    show_orders(infos, count, orders);
    for (int i = 0; i < count; i++)
    {
        if (S_ISDIR(infos[i].mode) && strcmp(infos[i].name, ".") != 0 && strcmp(infos[i].name, "..") != 0)
        {
            char sub_path[MAX_PATH];
            snprintf(sub_path, sizeof(sub_path), "%s/%s", path, infos[i].name);
            struct stat flag;
            if (lstat(sub_path, &flag) == -1)
            {
                fprintf(stderr, "Permission denied or error reading: %s\n", sub_path);
                continue;
            }
            if(S_ISLNK(flag.st_mode))
            {
                continue;
            }
            do_open_R(sub_path,orders);
        }
    }
    free(infos);
}


void ls_a(char *path, struct fileinfo *infos, int *count)
{
    DIR *dir;
    struct dirent *dirp;
    if ((dir = opendir(path)) == NULL)
    {
        printf("%s\n", path);
        return ;
    }
    *count = 0;
    struct stat st;
    while ((dirp = readdir(dir)) != NULL)
    {
        char current_path[MAX_PATH];
        snprintf(current_path, sizeof(current_path), "%s/%s", path, dirp->d_name);
        
        if ((stat(current_path, &st)) != -1)
        {
            
            strncpy(infos[*count].name, dirp->d_name, sizeof(infos[*count].name) - 1);
            infos[*count].name[sizeof(infos[*count].name) - 1] = '\0';  // Null-terminate the string
            infos[*count].group = st.st_gid;
            infos[*count].owner = st.st_uid;
            infos[*count].mode = st.st_mode;
            infos[*count].c_ctime = st.st_ctime;
            infos[*count].size = st.st_size;
            infos[*count].m_ctime = st.st_mtime;
            infos[*count].i_node = st.st_ino;
            mode_to_letters(st.st_mode, infos[*count].permission);
            infos[*count].link_num = st.st_nlink;
            infos[*count].block=st.st_blocks;
        }
        else
        {
            perror("stat");
            continue;
        }
        (*count)++;
    }
    closedir(dir);//一定要记得关闭文件
}


void only_ls(char *path, struct fileinfo *infos, int *count)
{
    DIR *dir;
    struct dirent *dirp;
    if ((dir = opendir(path)) == NULL)
    {
        printf("%s\n", path);
        return ;
    }
    *count = 0;
    struct stat st;
    while ((dirp = readdir(dir)) != NULL)
    {
        if (dirp->d_name[0] != '.')
        {
            char current_path[MAX_PATH];
            snprintf(current_path, sizeof(current_path), "%s/%s", path, dirp->d_name);
            if ((stat(current_path, &st)) != -1)
            {
                snprintf(infos[*count].name, sizeof(infos[*count].name), "%s", dirp->d_name);
                infos[*count].group = st.st_gid;
                infos[*count].owner = st.st_uid;
                infos[*count].mode = st.st_mode;
                infos[*count].c_ctime = st.st_ctime;
                infos[*count].size = st.st_size;
                infos[*count].m_ctime = st.st_mtime;
                infos[*count].i_node = st.st_ino;
                mode_to_letters(st.st_mode, infos[*count].permission);
                infos[*count].link_num = st.st_nlink;
                infos[*count].block = st.st_blocks;
            }
            (*count)++;
        }
    }
    closedir(dir);//一定要记得及时释放
}

void mode_to_letters(mode_t mode, char modestr[]) {
    strcpy(modestr, "----------");
    if (S_ISDIR(mode)) modestr[0] = 'd';
    if (S_ISCHR(mode)) modestr[0] = 'c';
    if (S_ISBLK(mode)) modestr[0] = 'b';

    if (mode & S_IRUSR) modestr[1] = 'r';
    if (mode & S_IWUSR) modestr[2] = 'w';
    if (mode & S_IXUSR) modestr[3] = 'x';

    if (mode & S_IRGRP) modestr[4] = 'r';
    if (mode & S_IWGRP) modestr[5] = 'w';
    if (mode & S_IXGRP) modestr[6] = 'x';

    if (mode & S_IROTH) modestr[7] = 'r';
    if (mode & S_IWOTH) modestr[8] = 'w';
    if (mode & S_IXOTH) modestr[9] = 'x';
}

int cmp_name(const void *a, const void *b)
{
    const char *name_a = ((struct fileinfo *)a)->name;
    const char *name_b = ((struct fileinfo *)b)->name;
    setlocale(LC_COLLATE, "");
    return strcoll(name_a, name_b);
}

int cmp_size(const void *a, const void *b)
{
    off_t size_a = ((struct fileinfo *)a)->size;
    off_t size_b = ((struct fileinfo *)b)->size;
    return size_a > size_b ? (size_a < size_b ? -1 : 0) : 1;
}

int cmp_time(const void *a, const void *b)
{
    time_t time_a = ((struct fileinfo *)a)->m_ctime;
    time_t time_b = ((struct fileinfo *)b)->m_ctime;
    return time_a > time_b ? (time_a < time_b ? -1 : 0) : 1;
}

void show_orders(struct fileinfo *infos, int count, int orders[])
{
    long long total = 0;
    if(orders[l])
    {
        for(int i = 0; i < count; i++)
        {
            total+=infos[i].block/2;
        }
        printf("总计 : %-11ld\n", total);
    }
    if (orders[r])
    {
        for (int i = count - 1; i >= 0; i--)
        {
            if (orders[I])
            {
                printf("%-8d", infos[i].i_node);
            }
            if(orders[s])
            {
                printf("%-8d", infos[i].block/2);
            }
            if (orders[l])
            {
                printf("%s ", infos[i].permission);
                printf("%4d ", (int)infos[i].link_num);
                printf("%-8s ", uid_to_name(infos[i].owner));
                printf("%-8s ", gid_to_name(infos[i].group));
                printf("%8ld ", infos[i].size);
                printf("%.12s ", ctime(&infos[i].c_ctime) + 4);
            }
            print(infos[i]);
        }
    }
    else
    {
        for (int i = 0; i < count; i++)
        {
            if (orders[I])
            {
                printf("%-8d", infos[i].i_node);
            }
            if(orders[s])
            {
                printf("%-8d", infos[i].block/2);
            }
            if (orders[l])
            {
                printf("%s ", infos[i].permission);
                printf("%4d ", (int)infos[i].link_num);
                printf("%-8s ", uid_to_name(infos[i].owner));
                printf("%-8s ", gid_to_name(infos[i].group));
                printf("%8ld ", infos[i].size);
                printf("%.12s ", ctime(&infos[i].c_ctime) + 4);
            }
            print(infos[i]);
        }
    }
}

char *uid_to_name(uid_t uid)
{
    struct passwd *pw_ptr;
    pw_ptr = getpwuid(uid);
    if (pw_ptr == NULL)
    {
        static char numstr[10];
        snprintf(numstr, sizeof(numstr), "%d", uid);
        return numstr;
    }
    else
    {
        return pw_ptr->pw_name;
    }
}

char *gid_to_name(gid_t gid)
{
    struct group *getgrgid(), *grp_ptr;
    static char numstr[10];
    if ((grp_ptr = getgrgid(gid)) == NULL)
    {
        sprintf(numstr, "%d", gid);
        return numstr;
    }
    else
    {
        return grp_ptr->gr_name;
    }
}

void print(struct fileinfo infos)
{
    if (S_ISREG(infos.mode))
    {
        // Regular file
        if (strstr(infos.name, ".c") != NULL)
        {
            // .c file, print in a different color
            printf("\033[40;32m %s\033[0m\n", infos.name);
        }
        else if (strstr(infos.name, ".out") != NULL)
        {
            // .out file, print in another color
            printf("\033[40;33m %s\033[0m\n", infos.name);
        }
        else
        {
            // Other regular file
            printf("%s\n", infos.name);
        }
    }
    else if (S_ISDIR(infos.mode))
    {
        // Directory
        printf("\033[40;34m %s\033[0m\n", infos.name);
    }
    else
    {
        // Other file types
        printf("%s\n", infos.name);
    }
}


```

