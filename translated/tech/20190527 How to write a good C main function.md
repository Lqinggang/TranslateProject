[#]: collector: (lujun9972)
[#]: translator: (MjSeven)
[#]: reviewer: ( )
[#]: publisher: ( )
[#]: url: ( )
[#]: subject: (How to write a good C main function)
[#]: via: (https://opensource.com/article/19/5/how-write-good-c-main-function)
[#]: author: (Erik O'Shaughnessy https://opensource.com/users/jnyjny)

如何写好 C main 函数
======
学习如何构造一个 C 文件并编写一个 C main 函数来处理命令行参数，像 champ 一样。
(to 校正：champ 是一个命令行程序吗？但我并没有找到，这里按一个程序来解释了)
![Hand drawing out the word "code"][1]

我知道，现在孩子们用 Python 和 JavaScript 编写他们疯狂的“应用程序”。但是不要这么快就否定 C 语言-- 它能够提供很多东西，并且简洁。如果你需要速度，用 C 语言编写可能就是你的答案。如果你正在寻找工作保障或者学习如何捕获[空指针解引用][2]，C 语言也可能是你的答案！在本文中，我将解释如何构造一个 C 文件并编写一个 C main 函数来处理像 champ 这样的命令行参数。

**我**：一个顽固的 Unix 系统程序员。
**你**：一个有编辑器，C 编译器，并有时间打发的人。

_让我们开工吧。_

### 一个无聊但正确的 C 程序

![Parody O'Reilly book cover, "Hating Other People's Code"][3]

一个 C 程序以 **main()** 函数开头，通常保存在名为 **main.c** 的文件中。

```
/* main.c */
int main(int argc, char *argv[]) {

}
```

这个程序会 _编译_ 但不 _执行_ 任何操作。

```
$ gcc main.c
$ ./a.out -o foo -vv
$
```

正确但无聊。

### main 函数是唯一的。

**main()** 函数是程序开始执行时执行的第一个函数，但不是第一个执行的函数。_第一个_ 函数是 **_start()**，它通常由 C 运行库提供，在编译程序时自动链接。此细节高度依赖于操作系统和编译器工具链，所以我假装没有提到它。

**main()** 函数有两个参数，通常称为 **argc** 和 **argv**，并返回一个有符号整数。大多数 Unix 环境都希望程序在成功时返回 **0**（零），失败时返回 **-1**（负一）。

参数 | 名称 | 描述
---|---|---
argc | 参数个数 | 参数向量的个数
argv | 参数向量 | 字符指针数组

参数向量 **argv** 是调用程序的命令行的标记化表示形式。在上面的例子中，**argv** 将是以下字符串的列表：


```
`argv = [ "/path/to/a.out", "-o", "foo", "-vv" ];`
```

参数向量保证在第一个索引中始终至少有一个字符串 **argv[0]**，这是执行程序的完整路径。

### main.c 文件的剖析

当我从头开始编写 **main.c** 时，它的结构通常如下：

```
/* main.c */
/* 0 copyright/licensing */
/* 1 includes */
/* 2 defines */
/* 3 external declarations */
/* 4 typedefs */
/* 5 全局变量声明 */
/* 6 函数原型 */

int main(int argc, char *argv[]) {
/* 7 命令行解析 */
}

/* 8 函数声明 */
```

下面我将讨论这些编号的各个部分，除了编号为 0 的那部分。如果你必须把版权或许可文本放在源代码中，那就放在那里。

另一件我不想谈论的事情是注释。

```
"Comments lie."
\- A cynical but smart and good looking programmer.
```

使用有意义的函数名和变量名而不是注释。

为了迎合程序员固有的惰性，一旦添加了注释，维护负荷就会增加一倍。如果更改或重构代码，则需要更新或扩展注释。随着时间的推移，代码会发生变化，与注释所描述的内容完全不同。

如果你必须写注释，不要写关于代码正在做 _什么_，相反，写下 _为什么_ 代码需要这样写。写一些你想在五年后读到的注释，那时你已经将这段代码忘得一干二净。世界的命运取决于你。_不要有压力。_

#### 1\. Includes

我添加到 **main.c** 文件的第一个东西是 include 文件，它们为程序提供大量标准 C 标准库函数和变量。C 标准库做了很多事情。浏览 **/usr/include** 中的头文件，了解它们可以为你做些什么。

**#include** 字符串是 [C 预处理程序][4]（cpp）指令，它会将引用的文件完整地包含在当前文件中。C 中的头文件通常以 **.h** 扩展名命名，且不应包含任何可执行代码。它只有宏、定义、typedef、外部变量和函数原型。字符串 **<header.h>** 告诉 cpp 在系统定义的头文件路径中查找名为 **header.h** 的文件，通常在 **/usr/include** 目录中。


```
/* main.c */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <libgen.h>
#include <errno.h>
#include <string.h>
#include <getopt.h>
#include <sys/types.h>
```

以下内容是我默认会包含的最小全局 include 集合：

#include 文件 | 提供的东西
---|---
stdio | 提供 FILE, stdin, stdout, stderr 和 fprint() 函数系列
stdlib | 提供 malloc(), calloc() 和 realloc()
unistd | 提供 EXIT_FAILURE, EXIT_SUCCESS
libgen | 提供 basename() 函数
errno | 定义外部 errno 变量及其可以接受的所有值
string | 提供 memcpy(), memset() 和 strlen() 函数系列
getopt | 提供 外部 optarg, opterr, optind 和 getopt() 函数
sys/types | 类型定义快捷方式，如 uint32_t 和 uint64_t

#### 2\. Defines

```
/* main.c */
<...>

#define OPTSTR "vi⭕f:h"
#define USAGE_FMT "%s [-v] [-f hexflag] [-i inputfile] [-o outputfile] [-h]"
#define ERR_FOPEN_INPUT "fopen(input, r)"
#define ERR_FOPEN_OUTPUT "fopen(output, w)"
#define ERR_DO_THE_NEEDFUL "do_the_needful blew up"
#define DEFAULT_PROGNAME "george"
```

这在现在没有多大意义，但 **OPTSTR** 定义是我说明程序将推荐的命令行切换。参考 [**getopt(3)**][5] man 页面，了解 **OPTSTR** 将如何影响 **getopt()** 的行为。

**USAGE_FMT** 定义了一个 **printf()** 风格形式的格式字符串，在 **usage()** 函数中被引用。

我还喜欢将字符串常量放在文件的这一部分作为 **#defines**。如果需要，收集它们可以更容易地修复拼写、重用消息和国际化消息。

最后，在命名 **#define** 时使用全部大写字母，以区别变量和函数名。如果需要，可以将单词放在一起或使用下划线分隔，只要确保它们都是大写的就行。

#### 3\. 外部声明


```
/* main.c */
<...>

extern int errno;
extern char *optarg;
extern int opterr, optind;
```

**extern** 声明将该名称带入当前编译单元的命名空间（也称为 "file"），并允许程序访问该变量。这里我们引入了三个整数变量和一个字符指针的定义。**getopt()** 函数使用 **opt** 前缀变量，C 标准库使用 **errno** 作为带外通信通道来传达函数可能的失败原因。

#### 4\. Typedefs


```
/* main.c */
<...>

typedef struct {
int verbose;
uint32_t flags;
FILE *input;
FILE *output;
} options_t;
```

在外部声明之后，我喜欢为结构、联合和枚举声明 **typedefs**。命名 **typedef** 本身就是一种传统行为。我非常喜欢 **_t** 后缀来表示该名称是一种类型。在这个例子中，我将 **options_t** 声明为一个包含 4 个成员的 **struct**。C 是一种与空格无关的编程语言，因此我使用空格将字段名排列在同一列中。我只是喜欢它的样子。对于指针声明，我在名称前面加上星号，以明确它是一个指针。

#### 5\. 全局变量声明


```
/* main.c */
<...>

int dumb_global_variable = -11;
```

全局变量是一个坏主意，你永远不应该使用它们。但如果你必须使用全局变量，请在这里声明并确保给它们一个默认值。说真的，_不要使用全局变量_。

#### 6\. 函数原型


```
/* main.c */
<...>

void usage(char *progname, int opt);
int do_the_needful(options_t *options);
```

在编写函数时，将它们添加到 **main()** 函数之后而不是之前，这里放函数原型。早期的 C 编译器使用单遍策略，这意味着你在程序中使用的每个符号（变量或函数名称）必须在使用之前声明。现代编译器几乎都是多遍编译器，它们在生成代码之前构建一个完整的符号表，因此并不严格要求使用函数原型。但是，有时你无法选择代码中使用的编译器，所以请编写函数原型并继续。

当然，我总是包含一个 **usage()** 函数，当 **main()** 函数不理解你从命令行传入的内容时，它会调用这个函数。

#### 7\. 命令行解析


```
/* main.c */
<...>

int main(int argc, char *argv[]) {
int opt;
options_t options = { 0, 0x0, stdin, stdout };

opterr = 0;

while ((opt = getopt(argc, argv, OPTSTR)) != EOF)
switch(opt) {
case 'i':
if (!(options.input = [fopen][6](optarg, "r")) ){
[perror][7](ERR_FOPEN_INPUT);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}
break;

case 'o':
if (!(options.output = [fopen][6](optarg, "w")) ){
[perror][7](ERR_FOPEN_OUTPUT);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}
break;

case 'f':
options.flags = (uint32_t )[strtoul][9](optarg, NULL, 16);
break;

case 'v':
options.verbose += 1;
break;

case 'h':
default:
usage(basename(argv[0]), opt);
/* NOTREACHED */
break;
}

if (do_the_needful(&options) != EXIT_SUCCESS) {
[perror][7](ERR_DO_THE_NEEDFUL);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}

return EXIT_SUCCESS;
}
```

好吧，代码有点多。**main()** 函数的目的是收集用户提供的参数，执行最小的输入验证，然后将收集的参数传递给使用它们的函数。这个示例声明使用默认值初始化的 **options** 变量，并解析命令行，根据需要更新 **options**。

**main()** 函数的核心是一个 **while** 循环，它使用 **getopt()** 来遍历 **argv**，寻找命令行选项及其参数（如果有的话）。文件前面的 **OPTSTR** **#define** 是驱动 **getopt()** 行为的模板。**opt** 变量接受 **getopt()** 找到的任何命令行选项的字符值，程序对检测命令行选项的响应发生在 **switch** 语句中。

现在你注意到了可能会问，为什么 **opt** 被声明为 32 位 **int**，但是预期是 8 位 **char**？事实上 **getopt()** 返回一个 **int**，当它到达 **argv** 末尾时取负值，我会使用 **EOF**（_文件末尾_ 标记）匹配。**char** 是有符号的，但我喜欢将变量匹配到它们的函数返回值。

当检测到一个已知的命令行选项时，会发生特定的行为。有些选项有一个参数，在 **OPTSTR** 中指定了一个以冒号结尾的参数。当一个选项有一个参数时，**argv** 中的下一个字符串可以通过外部定义的变量 **optarg** 提供给程序。我使用 **optarg** 打开文件进行读写，或者将命令行参数从字符串转换为整数值。

这里有几个关于风格的要点：

  * 将 **opterr** 初始化为 0，禁止 **getopt** 发出 **?**。
  * 在 **main()** 的中间使用 **exit(EXIT_FAILURE);** 或 **exit(EXIT_SUCCESS);**。
  * **/* NOTREACHED */** 是我喜欢的一个 lint 指令。
  * 在函数末尾使用 **return EXIT_SUCCESS;** 返回一个 int 类型。
  * 显示强制转换隐式类型。

这个程序的命令行签名经过编译如下所示：
```
$ ./a.out -h
a.out [-v] [-f hexflag] [-i inputfile] [-o outputfile] [-h]
```

事实上，**usage()** 在编译后就会向 **stderr** 发出这样的命令。

#### 8\. 函数声明

```
/* main.c */
<...>

void usage(char *progname, int opt) {
[fprintf][10](stderr, USAGE_FMT, progname?progname:DEFAULT_PROGNAME);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}

int do_the_needful(options_t *options) {

if (!options) {
errno = EINVAL;
return EXIT_FAILURE;
}

if (!options->input || !options->output) {
errno = ENOENT;
return EXIT_FAILURE;
}

/* XXX do needful stuff */

return EXIT_SUCCESS;
}
```

最后，我编写的函数不是样板函数。在本例中，函数 **do_the_needful()** 接受指向 **options_t** 结构的指针。我验证 **options** 指针不为 **NULL**，然后继续验证 **input** 和 **output** 结构成员。如果其中一个测试失败，返回 **EXIT_FAILURE**，并且通过将外部全局变量 **errno** 设置为常规错误代码，我向调用者发出一个原因的信号。调用者可以使用便捷函数 **perror()** 来根据 **errno** 的值发出人类可读的错误消息。

函数几乎总是以某种方式验证它们的输入。如果完全验证代价很大，那么尝试执行一次并将验证后的数据视为不可变。**usage()** 函数使用 **fprintf()** 调用中的条件赋值验证 **progname** 参数。**usage()** 函数无论如何都要退出，所以我不需要设置 **errno** 或为使用正确的程序名大吵一场。

在这里，我要避免的最大错误是取消引用 **NULL** 指针。这将导致操作系统向我的进程发送一个名为 **SYSSEGV** 的特殊信号，导致不可避免的死亡。用户希望看到的是由 **SYSSEGV** 引起的崩溃。为了发出更好的错误消息并优雅地关闭程序，捕获 **NULL** 指针要好得多。 

有些人抱怨在函数体中有多个 **return** 语句，他们争论“控制流的连续性”和其他东西。老实说，如果函数中间出现错误，那么这个时候是返回错误条件的好时机。写一大堆嵌套的 **if** 语句只有一个 return 绝不是一个“好主意。”™

最后，如果您编写的函数接受四个或更多参数，请考虑将它们绑定到一个结构中，并传递一个指向该结构的指针。这使得函数签名更简单，更容易记住，并且在以后调用时不会出错。它还使调用函数速度稍微快一些，因为需要复制到函数堆栈中的东西更少。在实践中，只有在函数被调用数百万或数十亿次时，才会考虑这个问题。如果认为这没有意义，那么不要担心。

### 等等，你说没有注释 !?!!

在 **do_the_needful()** 函数中，我写了一种特殊类型的注释，它被设计为占位符而不是记录代码：


```
`/* XXX do needful stuff */`
```

当你在该区域时，有时你不想停下来编写一些特别复杂的代码，你会之后再写，而不是现在。那就是我留给自己一点面包屑的地方。我插入一个带有 **XXX** 前缀的注释和一个描述需要做什么的简短注释。之后，当我有更多时间的时候，我会在源代码中寻找 **XXX**。使用什么并不重要，只要确保它不太可能在另一个上下文中以函数名或变量形式显示在代码库中。

### 把它们放在一起

好吧，当你编译这个程序后，它 _仍_ 仍几乎没有任何作用。但是现在你有了一个坚实的骨架来构建你自己的命令行解析 C 程序。

```
/* main.c - the complete listing */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <libgen.h>
#include <errno.h>
#include <string.h>
#include <getopt.h>

#define OPTSTR "vi⭕f:h"
#define USAGE_FMT "%s [-v] [-f hexflag] [-i inputfile] [-o outputfile] [-h]"
#define ERR_FOPEN_INPUT "fopen(input, r)"
#define ERR_FOPEN_OUTPUT "fopen(output, w)"
#define ERR_DO_THE_NEEDFUL "do_the_needful blew up"
#define DEFAULT_PROGNAME "george"

extern int errno;
extern char *optarg;
extern int opterr, optind;

typedef struct {
int verbose;
uint32_t flags;
FILE *input;
FILE *output;
} options_t;

int dumb_global_variable = -11;

void usage(char *progname, int opt);
int do_the_needful(options_t *options);

int main(int argc, char *argv[]) {
int opt;
options_t options = { 0, 0x0, stdin, stdout };

opterr = 0;

while ((opt = getopt(argc, argv, OPTSTR)) != EOF)
switch(opt) {
case 'i':
if (!(options.input = [fopen][6](optarg, "r")) ){
[perror][7](ERR_FOPEN_INPUT);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}
break;

case 'o':
if (!(options.output = [fopen][6](optarg, "w")) ){
[perror][7](ERR_FOPEN_OUTPUT);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}
break;

case 'f':
options.flags = (uint32_t )[strtoul][9](optarg, NULL, 16);
break;

case 'v':
options.verbose += 1;
break;

case 'h':
default:
usage(basename(argv[0]), opt);
/* NOTREACHED */
break;
}

if (do_the_needful(&options) != EXIT_SUCCESS) {
[perror][7](ERR_DO_THE_NEEDFUL);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}

return EXIT_SUCCESS;
}

void usage(char *progname, int opt) {
[fprintf][10](stderr, USAGE_FMT, progname?progname:DEFAULT_PROGNAME);
[exit][8](EXIT_FAILURE);
/* NOTREACHED */
}

int do_the_needful(options_t *options) {

if (!options) {
errno = EINVAL;
return EXIT_FAILURE;
}

if (!options->input || !options->output) {
errno = ENOENT;
return EXIT_FAILURE;
}

/* XXX do needful stuff */

return EXIT_SUCCESS;
}
```

现在，你已经准备好编写更易于维护的 C 语言。如果你有任何问题或反馈，请在评论中分享。

--------------------------------------------------------------------------------

via: https://opensource.com/article/19/5/how-write-good-c-main-function 
                     
作者：[Erik O'Shaughnessy][a]
选题：[lujun9972][b]
译者：[MjSeven](https://github.com/MjSeven)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]: https://opensource.com/users/jnyjny
[b]: https://github.com/lujun9972
[1]: https://opensource.com/sites/default/files/styles/image-full-size/public/lead-images/code_hand_draw.png?itok=dpAf--Db (Hand drawing out the word "code")
[2]: https://www.owasp.org/index.php/Null_Dereference
[3]: https://opensource.com/sites/default/files/uploads/hatingotherpeoplescode-big.png (Parody O'Reilly book cover, "Hating Other People's Code")
[4]: https://en.wikipedia.org/wiki/C_preprocessor
[5]: https://linux.die.net/man/3/getopt
[6]: http://www.opengroup.org/onlinepubs/009695399/functions/fopen.html
[7]: http://www.opengroup.org/onlinepubs/009695399/functions/perror.html
[8]: http://www.opengroup.org/onlinepubs/009695399/functions/exit.html
[9]: http://www.opengroup.org/onlinepubs/009695399/functions/strtoul.html
[10]: http://www.opengroup.org/onlinepubs/009695399/functions/fprintf.html
