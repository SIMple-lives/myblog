---
title: shell 笔记
date: 2024-03-29
---

# shell

## 主函数

* 先创建一个Shell类型的类
* 然后屏蔽信号ctrl+c与ctrl+D
* 接着打印提示符，接收命令的输入
* 判断该次命令是否含有`exit` `clear` `!`等信息，如果有，执行相应操作
* 通过调用解析函数解析本次输入的命令

## 屏蔽信号

* ctrl  +  C

  ```c++
  signal(SIGINT, SIG_IGN);//将该信号的处理方式设置为忽略
  ```

* ctrl  +  D

  ```c++
  	struct termios term;
      tcgetattr(STDIN_FILENO, &term);
      term.c_cc[VEOF] = _POSIX_VDISABLE;
      tcsetattr(STDIN_FILENO, TCSANOW, &term);
  ```

  

> 首先定义一个termios类型的结构体，用来存储终端相关的属性信息
>
> 然后通过调用tcgetattr函数将获取终端的相关属性，并将这些属性保存到 **term **结构体中，第一个参数表示标准输入，是一个预定的文件描述符常量
>
> 将结构体中的c_cc数组成员的VEOF设置为禁用. c_cc数组是包含终端特殊字符的数组，VEOF是其中的一个索引，第二个参数是一个特殊的宏，用于禁用该特殊字符
>
> 最后，`tcsetattr(STDIN_FILENO, TCSANOW, &term);`调用`tcsetattr()`函数，它用于设置终端的属性。`STDIN_FILENO`表示标准输入文件描述符，`TCSANOW`表示立即生效的选项，`&term`是要设置的终端属性结构体。

### tcgetattr

```c++
int tcgetattr(int fd, struct termios *termios_p);
```

参数说明：

- `fd` 是一个打开的终端设备文件描述符。

- `termios_p` 是一个指向 `termios` 结构体的指针，用于存储获取

  到的终端属性。

`tcgetattr` 函数返回一个整数值，表示函数调用的成功与否。如果函数成功执行，返回值为0；如果出现错误，返回值为-1，并设置相应的错误码。在调用该函数时，你需要检查返回值以确保函数调用是否成功。

###  tcsetattr

```c++
int tcsetattr(int fd, int optional_actions, const struct termios *termios_p);
```

参数说明：

- `fd` 是一个打开的终端设备文件描述符。
- `optional_actions` 是一个表示操作选项的整数值，用于指定何时应用属性的更改。
- `termios_p` 是一个指向 `termios` 结构体的指针，其中包含了要应用的终端属性。

`optional_actions` 参数用于指定何时应用属性的更改，它可以取以下三个值之一：

- `TCSANOW`：立即更改属性。
- `TCSADRAIN`：在所有输出都被传输后更改属性。
- `TCSAFLUSH`：在所有输出都被传输后更改属性，并丢弃所有未读的输入。

## print

**getlogin**

```c++
char* getlogin(void)
//返回一个指向当前登陆用户的用户名的字符串指针
```

**strcspn**

```c++
size_t strscpn(const char *str1,const char *str2);
//str1：要搜索的字符串
//str2：要查找的字符集合
//计算字符串中连续不包含指定字符集合的最大前缀长度。
```

**strftime**

```c++
size_t strftime(char *str, size_t maxsize, const char *format, const struct tm *timeptr);
```

参数说明：

- `str` 是一个指向字符数组的指针，用于存储格式化后的时间字符串。
- `maxsize` 是 `str` 所指向的字符数组的最大容量。
- `format` 是一个格式字符串，用于定义输出的时间格式。
- `timeptr` 是一个指向 `struct tm` 结构体的指针，其中包含要格式化的时间和日期。

```c++
chrono::system_clock::now();
//是c++<chrono>头文件中的一个函数调用，用于获取当前的系统时间点
chrono::system_clock::to_time_t
//是一个函数模板，将time_point对象转换为time_t的类型
```

## mycd

**getenv**

```c++
char* getenv(const char* name);
```

参数说明:

* name是一个指向以null结尾的字符串，表示要获取的环境变量的名称.

根据指定的环境变量的名称，返回对应的环境变量的值。找到则返回一个null结尾的字符串，表示该环境变量的值.否则返回null.

```c++
const char* path = s[1].c_str();
//将string对象转换为c风格的字符串.
```

> 需要注意的是c_str()返回的指针指向的仍为string内部存储的字符数组，因此需要确保在使用path指针时，string对象仍然有效

**get_current_dir_name**

是一个标准的POSIX函数，用于获取当前工作目录的路径

由于get_current_dir_name( )分配了动态内存，因此使用完毕后需要调用free( )。

**setenv**

```c++
int setenv(const char* name, const char* value, int overwrite);
```

参数说明：

- `name` 是一个指向以 null 结尾的字符串，表示要设置或新建的环境变量的名称。
- `value` 是一个指向以 null 结尾的字符串，表示要设置的环境变量的值。
- `overwrite` 是一个整数值，指示在环境变量已经存在时是否进行覆盖。如果 `overwrite` 为非零值，则会覆盖现有环境变量；如果 `overwrite` 为零，则不进行覆盖。

## parse

`istringstream`是C++标准库中的一个类，他是用于从字符串中进行输入操作的输入流类.

**getline()**

```c++
std::istream& getline(std::istream& is, std::string& str, char delim);
```

参数说明：

- `is` 是一个输入流对象，表示要从中读取文本的输入流，通常是 `std::cin`（标准输入）或文件流对象。
- `str` 是一个字符串对象的引用，表示读取的文本将存储在其中。
- `delim` 是一个字符（默认为换行符 `\n`），表示行的结束符。

### pipe

"pipe"（管道）是一种在进程间进行通信的机制。它允许一个进程的输出直接成为另一个进程的输入，从而实现进程间的数据传输。

1. 匿名管道：
   - 匿名管道是最常见的管道类型。它是一种单向通信管道，只能在具有父子关系的进程之间使用。父进程创建管道后，可以通过文件描述符进行读取或写入。子进程继承了父进程的文件描述符，从而可以使用相反的操作进行读取或写入。
   - 匿名管道是一种半双工通信方式，即数据只能在一个方向上流动。如果需要双向通信，需要创建两个匿名管道。
   - 使用匿名管道的典型流程是：父进程创建管道，然后调用 `fork()` 创建子进程。父进程关闭不需要的管道端口，子进程关闭另一个不需要的端口。然后父进程可以向管道写入数据，子进程可以从管道读取数据。
2. 命名管道：
   - 命名管道也称为FIFO（First-In-First-Out），它提供了一种无关进程关系的进程间通信方式。命名管道在文件系统中有一个与之相关联的路径名，多个进程可以通过这个路径名来进行通信。
   - 命名管道是一种半双工通信方式，类似于匿名管道，只能在一个方向上流动数据。
   - 命名管道可以使用 `mkfifo` 函数创建，也可以使用命令行工具 `mkfifo` 创建。

管道的使用可以通过系统调用函数来实现，例如 POSIX 标准中的 `pipe()` 函数。在使用管道时，重要的是要了解管道的读取和写入端口以及数据的流动方向，以确保正确的通信。

```c++
int pipe(int pipefd[2]);
```

> `pipe()` 函数接受一个整型数组 `pipefd[2]`，用于存储管道的文件描述符。`pipefd[0]` 表示管道的读取端口，`pipefd[1]` 表示管道的写入端口。该函数返回值为 0 表示成功，-1 表示失败。
>
> 以下是 `pipe()` 函数的详细解释：
>
> - `pipefd` 参数是一个包含两个元素的整型数组，用于存储管道的文件描述符。`pipefd[0]` 是管道的读取端口，用于从管道中读取数据；`pipefd[1]` 是管道的写入端口，用于向管道中写入数据。
> - `pipe()` 函数创建一个匿名管道，并将管道的读取端口和写入端口的文件描述符填充到 `pipefd` 数组中。
> - 匿名管道是一种半双工通信方式，数据只能在一个方向上流动。如果需要双向通信，需要创建两个匿名管道。
> - 管道是内核中的一个缓冲区，用于在相关进程之间传输数据。数据写入管道后，可以从管道的读取端口读取。
> - 管道的大小是有限的，一旦管道的缓冲区满了，进程写入数据时可能会被阻塞，直到有足够的空间来容纳数据。同样，如果管道为空，从管道读取数据时可能会被阻塞，直到有数据可读。
> - 如果一个进程关闭了管道的读取端口，而另一个进程关闭了管道的写入端口，这两个进程之间的管道通信将结束。

### dup2

```c++
#include <unistd.h>

int dup2(int oldfd, int newfd);
```

> - `oldfd` 参数是需要复制的文件描述符。它可以是任何有效的文件描述符，包括标准输入（0）、标准输出（1）和标准错误（2）。
> - `newfd` 参数是要复制到的新文件描述符。如果 `newfd` 已经打开，则会先关闭它，然后将 `oldfd` 复制到 `newfd`。
> - 复制后，`newfd` 将和 `oldfd` 指向相同的文件表项，它们共享同一个文件偏移量和文件状态标志。
> - 如果 `newfd` 等于 `oldfd`，则 `dup2()` 函数不进行任何操作，直接返回 `newfd`。
> - `dup2()` 函数的一个常见用途是重定向标准输入、标准输出和标准错误。例如，可以将标准输出重定向到一个文件，或者将标准错误重定向到一个套接字。

## executeCommand

**open**

- `command[i + 1].c_str()`：`command`是一个字符串向量，`command[i + 1]`表示该向量中的第`i + 1`个元素。`.c_str()`将该元素转换为C风格的字符串，以便与`open()`函数的参数类型匹配。这个参数是要打开的文件的路径。
- `O_WRONLY`：是`open()`函数的一个打开模式，表示以只写方式打开文件。这意味着你可以向文件写入数据，但不能读取文件的内容。
- `O_CREAT`：是`open()`函数的一个打开模式，表示如果文件不存在，则创建该文件。如果文件已经存在，`open()`函数仍然会成功打开文件。
- `O_TRUNC`：是`open()`函数的一个打开模式，表示如果文件已经存在且以只写方式打开，那么在打开文件时将截断文件的长度为0。这意味着打开文件后，文件中已经存在的内容将被清空。
- `0644`：表示文件的权限。在这里，`0644`表示文件所有者具有读和写的权限，而其他用户只有读的权限。它是一个八进制数，其中第一个数字`0`表示这是一个八进制数，后面的三个数字分别表示文件所有者、文件所属组和其他用户的权限。

# 代码

## shepp.hpp

```
#include <iostream>
using namespace std;
#include <string>
#include <vector>
#include <sstream>
#include <chrono>
#include <unistd.h>
#include <csignal>
#include <algorithm>
#include <termios.h>
#include <cstdlib>
#include <wait.h>
#include <fstream>
#include <sys/stat.h>
#include <fcntl.h>

class Shell
{
public:
    void print();
    void setlinebuf();
    string retlinebuf();
    void setsignal();
    bool isempty();
    void addHistory(const string& line);
    void setEOF();
    void printHistory();
    void searchAndReplay(const string &keyword);
    void mycd(vector<string> &s);
    void executeCommand(const vector<string> &command);
    void parse(string &s);
private:
    string linebuf;
    vector<string> history;
};
```

## func.cpp

```c++
#include "shell.hpp"

void Shell::print()
{   cout << "----------------------------------------------------------------------------" << endl;
    cout << "           ___                                                     ___" << endl;
    cout << "  ______  /  /___       ______  _____  ___   ___       ____  _____/  /___" << endl;
    cout << " /  __  \\/  __   \\     /  ___ - ___  \\/  /  /  /     /_   / /  __/  __   \\" << endl;
    cout << "/  /_/  /  /  /  /    /  /  /  /  /  /  /__/  /       /  /_(__  )  /  /  /" << endl;
    cout << "\\______/__/  /__/    /__/  /__/  /__/\\____,  /       /____/____/__/  /__/" << endl;
    cout << "                                    /_______/" << endl;
    cout << "----------------------------------------------------------------------------" << endl;
    char cwd [1024];
    auto now = chrono::system_clock::now();//返回当前的时间
    time_t time = chrono::system_clock::to_time_t(now);//将time_t point转换为time_t类型
    tm* timeinfo = std::localtime(&time);//将time_t转换为本地时间年月日,并存储在timeinfo中
    char timeStr[9];
    strftime(timeStr, sizeof(timeStr), "%H:%M:%S", timeinfo);
    //std::strftime()将std::tm结构的时间信息格式化为字符串，并将结果存储在指定的字符数组中。这里使用"%H:%M:%S"作为格式字符串
    //，它表示小时、分钟和秒的格式。timeStr是目标字符数组，sizeof(timeStr)用于指定数组的大小，
    //timeinfo是包含时间信息的std::tm结构的指针。
    cout << "# future @ future-arch in " << getcwd(cwd,1024) << " [" << timeStr << "]" << endl;
    cout << "$ " ;
    fflush(stdout);
}

void Shell ::setlinebuf()
{
    getline(cin,this->linebuf);
}

string Shell::retlinebuf ()
{
    return this->linebuf;
}

void Shell:: setsignal()
{
    signal(SIGINT, SIG_IGN);//忽略ctrl+c的操作
}

bool Shell ::isempty() 
{
    if(this->linebuf.empty())
    {
        return true;
    }
    return false;
}

void Shell::addHistory(const string& line)
{
    for(vector<string>::iterator it = history.begin(); it != history.end();it++)
    {
        if((*it).compare(line) == 0)
        return ;
    }
    history.push_back(line);
}

void Shell ::setEOF()
{
    struct termios term;//存储终端的相关信息
    tcgetattr(STDIN_FILENO, &term);//获取相关信息，并存入结构体
    term.c_cc[VEOF] = _POSIX_VDISABLE;//禁止使用EOF
    tcsetattr(STDIN_FILENO, TCSANOW, &term);//将设置存入终端
}

void Shell::printHistory()
{
    for(vector<string>::iterator it = this->history.begin(); it != this->history.end();it++)
    {
        cout << *it << endl;
    }
    cout << "~~~~~~~~~~~~~~~~~~~~~~~~~~~"<< endl;
}

void Shell::searchAndReplay(const string& keyword)
{
    for (const string& line : history)
    {
        if (line.find(keyword) != string::npos)
        {
            cout << "Replaying: " << line << endl;
            linebuf = line;
            break;
        }
    }
}

void Shell::parse(string& s)
{
    vector<vector<string>> tokens;
    istringstream iss(s);//可方便的从字符串中读取数据
    string root;
    while (getline(iss, root, '|'))//将s以|分割开
    {
        istringstream root_iss(root);
        vector<string> root_tokens;
        string token;
        while (root_iss >> token)//将单个命令以空格分隔开
        {
            root_tokens.push_back(token);
        }

        tokens.push_back(root_tokens);
    }
    if (tokens.size() == 1)
    {
        vector<string>& command = tokens[0];
        if (command[0] == "cd")
        {
            mycd(command);
        }
        else
        {
            executeCommand(command);
        }
    }
    else if (tokens.size() > 1)
    {
        int numPipes = tokens.size() - 1;
        vector<int> pipefds(2 * numPipes, 0); // 用于存储管道的文件描述符
        for (int i = 0; i < numPipes; ++i)
        {
            if (pipe(pipefds.data() + i * 2) < 0)
            {
                perror("pipe()");
                exit(1);
            }
        }
        int commandIndex = 0;
        for (auto& command : tokens)
        {
            pid_t pid = fork();
            if (pid < 0)
            {
                perror("fork()");
                exit(1);
            }
            if (pid == 0)
            {
                // 子进程
                if (commandIndex != 0) // 不是第一个命令，读取前一个管道的输出
                {
                    if (dup2(pipefds[(commandIndex - 1) * 2], STDIN_FILENO) < 0)
                    {
                        perror("dup2()");
                        exit(1);
                    }
                }
                if (commandIndex != numPipes) // 不是最后一个命令，将输出写入下一个管道
                {
                    if (dup2(pipefds[commandIndex * 2 + 1], STDOUT_FILENO) < 0)
                    {
                        perror("dup2()");
                        exit(1);
                    }
                }
                // 关闭所有管道的文件描述符
                for (int i = 0; i < 2 * numPipes; ++i)
                {
                    close(pipefds[i]);
                }
                if (command[0] == "cd")
                {
                    mycd(command);
                }
                else
                {
                    executeCommand(command);
                }

                exit(0);
            }
            else
            {
                // 父进程
                ++commandIndex;
            }
        }
        // 关闭所有管道的文件描述符
        for (int i = 0; i < 2 * numPipes; ++i)
        {
            close(pipefds[i]);
        }

        // 等待所有子进程结束
        for (size_t i = 0; i < tokens.size(); ++i)
        {
            wait(NULL);
        }
    }
}

void Shell::mycd(vector<string>& s)
{
    if (s.size() > 2)
    {
        cout << "cd: 函数调用参数过多" << endl;
        return;
    }
    else if (s.size() == 1 || s[1].compare("~") == 0)
    {
        const char* homeDir = getenv("HOME");
        if (homeDir)
        {
            if (chdir(homeDir) == -1)
            {
                perror("chdir()");
            }
        }
        else
        {
            cout << "cd: 无法获取家目录路径" << endl;
            return;
        }
    }
    else if (s[1].compare("-") == 0)
    {
        const char* lastDir = getenv("OLDPWD");
        if (lastDir)
        {
            if (chdir(lastDir) == -1)
            {
                perror("chdir()");
            }
        }
        else
        {
            cout << "cd: 无法找到上一个路径" << endl;
        }
    }
    else
    {
        const char* path = s[1].c_str();
        if (chdir(path) == -1)
        {
            perror("chdir()");
        }
    }
    // 更新环境变量 OLDPWD
    char* currentDir = get_current_dir_name();//获取当前工作目录的路径
    if (currentDir)
    {
        if (setenv("OLDPWD", currentDir, 1) == -1)//设置环境变量
        {
            perror("setenv()");
        }
        free(currentDir);
    }
    else
    {
        perror("get_current_dir_name()");
    }
}

void Shell::executeCommand(const vector<string>& command)
{
    pid_t pid;
    fflush(NULL);
    pid = fork();
    if (pid < 0)
    {
        perror("fork()");
    }
    if (pid == 0)
    {
        vector<char*> args;
        vector<int> outputFds; // 存储输出文件描述符的容器

        for (size_t i = 0; i < command.size(); ++i)
        {
            if (command[i] == ">") // 输出重定向：覆盖写入文件
            {
                int fd = open(command[i + 1].c_str(), O_WRONLY | O_CREAT | O_TRUNC, 0644);
                if (fd == -1)
                {
                    perror("open()");
                    exit(1);
                }
                outputFds.push_back(fd); // 将新的文件描述符添加到容器中
                dup2(fd, STDOUT_FILENO);
                close(fd);
                i++;
            }
            else if (command[i] == ">>") // 输出重定向：追加写入文件
            {
                int fd = open(command[i + 1].c_str(), O_WRONLY | O_CREAT | O_APPEND, 0644);
                if (fd == -1)
                {
                    perror("open()");
                    exit(1);
                }
                outputFds.push_back(fd); // 将新的文件描述符添加到容器中
                dup2(fd, STDOUT_FILENO);
                close(fd);
                i++;
            }
            else if (command[i] == "<") // 输入重定向
            {
                int fd = open(command[i + 1].c_str(), O_RDONLY);
                if (fd == -1)
                {
                    perror("open()");
                    exit(1);
                }
                dup2(fd, STDIN_FILENO);
                close(fd);
                i++;
            }
            else if (command[i] == "<<") // Here文档
            {
                string delimiter = command[i + 1];
                string inputText;

                while (getline(cin, inputText))
                {
                    if (inputText == delimiter)
                        break;
                    inputText += "\n";
                    write(STDIN_FILENO, inputText.c_str(), inputText.size());
                }

                i++;
            }
            else
            {
                args.push_back(const_cast<char*>(command[i].c_str()));
            }
        }

        args.push_back(nullptr); // execvp 需要以 nullptr 结尾的参数数组
        execvp(args[0], args.data());

        // execvp 执行失败时才会执行到这里
        perror("execvp()");
        exit(1);
    }
    else
    {
        wait(NULL);
    }
}
```

## shell.cpp

```c++
#include "func.cpp"

int main()
{
    int num = 1;
    Shell mysh;
    mysh.setsignal();   
    mysh.setEOF();
    while(num)
    {
        mysh.print();
        mysh.setlinebuf();
        string t = mysh.retlinebuf();
        if(t.empty())
        {
            cout << "No input" << endl;
        }
        else if(t.rfind("exit")!=string::npos)
        {
            break;
        }
        else if(t.find("!")==0)
        {
            string keyword = t.substr(1); //获取去除感叹号的关键字
            mysh.searchAndReplay(keyword);
        }
        else if(t.find("clear")==0)
        {
            mysh.addHistory(t);
            // cout << "\033[2J\033[1;1H";//通过转义序列清除终端，但只能清楚当次的命令
            system("clear");
            num++;
        }
        else
        {
            mysh.addHistory(t);
            num++;
        }
        mysh.parse(t);
        if(num%3==0)
        {
            cout << "~~~~~~~~~~~~~~~~~~~~~~~~~~~"<< endl;
            cout << " History Order :" << endl;
            mysh.printHistory();
        }
    }
    return 0;
}
```

**后续问题**： 

1. 在输入命令全为`空格`的时候，会直接推出程序，造成程序崩溃.

   1. 是在`parse`中解析命令的时候，由于全为空格，所以会将一个**空字符串**token放入命令数组tokens,导致tokens.size()不为0造成exec函数的执行出现错误

   2. 所以在开始解析命令的时候加上全为`空格`的判断.

      ```c++
      if(std::all_of(s.begin(),s.end(),::isspace))
      {
          return {};
      }
      ```

   3. 在全为空格的时候直接返回，这样就可以返会空的二维数组.

2. 缺乏对乱码命令的提示:例如输入"djsakldjalkjsduj"，等命令没有类似`zsh : command not found : jdklasjdlak`的提示，而是由于exec函数的无法执行出现的提示.
3. 对于主题打印的提示，可以将`Oh my zsh`的打印只放在初始的时候，而不是每一次都打印.
4. 可以考虑对命令的存储进行改进，运用其他的结构存储，而不是二维数组.
5. 增加类与类之间的联系，而不是简单的创造一个类，这样的话就只是相当于类似头文件的作用，并没有过多的用到一些c++的特性之类的东西.
