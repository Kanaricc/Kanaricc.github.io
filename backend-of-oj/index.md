# OJ的后端

内容有复制和参考。

这篇文章主要用于记录在探索评测系统Reef期间我所学的东西,以便之后查阅.

Reef预计主要支持远程评测

加一点本地评测.....

<!--more-->

结果主要的东西都特么是本地评测的.

## 总体架构
Reef预计将采用seccomp作为第一道安全关卡,使用多线程检测程序耗时/内存等信息.

在外层使用docker封装并再次限制资源,接入队列以能够方便的横向扩展.

## seccomp
seccomp为linux系统上才有的安全技术,因此必须使用linux.在安装必要的安装包后

```bash
$ sudo apt install libseccomp2 libseccomp-dev seccomp
```

即可使用.

当然,在windows下的Jobs似乎也可以利用,但是我不太懂,微软文档写得也奇怪,而且还得是服务器版本的windows才能用.

### 基本使用
seccomp需要由程序主动加载.其使用方法基本为下

```cpp
//g++ -g test.c -o o -lseccomp
#include <unistd.h>
#include <seccomp.h>
#include <linux/seccomp.h>

int main(void){
	//初始化筛选器
	scmp_filter_ctx ctx;
	ctx = seccomp_init(SCMP_ACT_ALLOW);//flag指明默认通过
	//添加拦截,并指明一旦拦截就将程序kill掉.
	seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execve), 0);
	//将规则加载注入.
	seccomp_load(ctx);

	char * filename = "/bin/sh";
	char * argv[] = {"/bin/sh",NULL};
	char * envp[] = {NULL};
	write(1,"i will give you a shell\n",24);
	//程序将会在此行崩溃.
	syscall(59,filename,argv,envp);//execve
	return 0;
}
```

seccomp_init是初始化的过滤状态,这里用的是SCMP_ACT_ALLOW,表示默认允许所有的syscacll.如果初始化状态为SCMP_ACT_KILL,则表示默认不允许所有的syscall.

```cpp
/**
 * Kill the process
 */
#define SCMP_ACT_KILL		0x00000000U
/**
 * Throw a SIGSYS signal
 */
#define SCMP_ACT_TRAP		0x00030000U
/**
 * Return the specified error code
 */
#define SCMP_ACT_ERRNO(x)	(0x00050000U | ((x) & 0x0000ffffU))
/**
 * Notify a tracing process with the specified value
 */
#define SCMP_ACT_TRACE(x)	(0x7ff00000U | ((x) & 0x0000ffffU))
/**
 * Allow the syscall to be executed after the action has been logged
 */
#define SCMP_ACT_LOG		0x7ffc0000U
/**
 * Allow the syscall to be executed
 */
#define SCMP_ACT_ALLOW		0x7fff0000U
```

规则添加
```cpp
/**
 * Add a new rule to the filter
 * @param ctx the filter context
 * @param action the filter action
 * @param syscall the syscall number
 * @param arg_cnt the number of argument filters in the argument filter chain
 * @param ... scmp_arg_cmp structs (use of SCMP_ARG_CMP() recommended)
 *
 * This function adds a series of new argument/value checks to the seccomp
 * filter for the given syscall; multiple argument/value checks can be
 * specified and they will be chained together (AND&#039;d together) in the filter.
 * If the specified rule needs to be adjusted due to architecture specifics it
 * will be adjusted without notification.  Returns zero on success, negative
 * values on failure.
 *
 */
int seccomp_rule_add(scmp_filter_ctx ctx,
		     uint32_t action, int syscall, unsigned int arg_cnt, ...);
```

seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execve), 0);,arg_cnt为0,表示直接限制execve,不管他什么参数.

如果arg_cnt不为0,那arg_cnt表示后面限制的参数的个数,也就是只有调用execve,且参数满足要求时,才会拦截syscall.

```
seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(write),1,SCMP_A2(SCMP_CMP_EQ,0x10));//第2(从0)个参数等于0x10
```

```cpp
/**
 * Specify an argument comparison struct for use in declaring rules
 * @param arg the argument number, starting at 0
 * @param op the comparison operator, e.g. SCMP_CMP_*
 * @param datum_a dependent on comparison
 * @param datum_b dependent on comparison, optional
 */
#define SCMP_CMP(...)		((struct scmp_arg_cmp){__VA_ARGS__})

/**
 * Specify an argument comparison struct for argument 0
 */
#define SCMP_A0(...)		SCMP_CMP(0, __VA_ARGS__)

/**
 * Specify an argument comparison struct for argument 1
 */
#define SCMP_A1(...)		SCMP_CMP(1, __VA_ARGS__)

/**
 * Specify an argument comparison struct for argument 2
 */
#define SCMP_A2(...)		SCMP_CMP(2, __VA_ARGS__)

/**
 * Specify an argument comparison struct for argument 3
 */
#define SCMP_A3(...)		SCMP_CMP(3, __VA_ARGS__)

/**
 * Specify an argument comparison struct for argument 4
 */
#define SCMP_A4(...)		SCMP_CMP(4, __VA_ARGS__)

/**
 * Specify an argument comparison struct for argument 5
 */
#define SCMP_A5(...)		SCMP_CMP(5, __VA_ARGS__)



/**
 * Comparison operators
 */
enum scmp_compare {
	_SCMP_CMP_MIN = 0,
	SCMP_CMP_NE = 1,		/**< not equal */
	SCMP_CMP_LT = 2,		/**< less than */
	SCMP_CMP_LE = 3,		/**< less than or equal */
	SCMP_CMP_EQ = 4,		/**< equal */
	SCMP_CMP_GE = 5,		/**< greater than or equal */
	SCMP_CMP_GT = 6,		/**< greater than */
	SCMP_CMP_MASKED_EQ = 7,		/**< masked equality */
	_SCMP_CMP_MAX,
};

/**
 * Argument datum
 */
typedef uint64_t scmp_datum_t;

/**
 * Argument / Value comparison definition
 */
struct scmp_arg_cmp {
	unsigned int arg;	/**< argument number, starting at 0 */
	enum scmp_compare op;	/**< the comparison op, e.g. SCMP_CMP_* */
	scmp_datum_t datum_a;
	scmp_datum_t datum_b;
};
```

ctx的内容可以使用函数dump出来,之后可以直接使用prctl命令相关直接载入,方便使用?

### seccomp调试

使用如下命令导出所有可能的命令
```bash
file=syscall-names.h
echo "static const char *syscall_names[] = {" > $file
echo "#include <sys/syscall.h>" | cpp -dM | grep &#039;^#define __NR_&#039; | LC_ALL=C sed -r -n -e &#039;s/^\#define[ \t]+__NR_([a-z0-9_]+)[ \t]+([0-9]+)(.*)/ [\2] = "\1",/p&#039; >> $file
echo "};" >> $file
```

使用如下代码导出一段代码所需要的权限.遵循最小权限原则,试验代码运行所需要的最少权限.
```cpp
#define __USE_GNU 1
#define _GNU_SOURCE 1
#include <signal.h>
#include <sys/prctl.h>
#include <linux/types.h>
#include <linux/filter.h>
#include <linux/seccomp.h>
#include <seccomp.h>
#include <unistd.h>
#include <string.h>
#include <stddef.h>
#include "syscall-names.h"
#if defined(__i386__)
#define REG_RESULT	REG_EAX
#define REG_SYSCALL	REG_EAX
#define REG_ARG0	REG_EBX
#define REG_ARG1	REG_ECX
#define REG_ARG2	REG_EDX
#define REG_ARG3	REG_ESI
#define REG_ARG4	REG_EDI
#define REG_ARG5	REG_EBP
#elif defined(__x86_64__)
#define REG_RESULT	REG_RAX
#define REG_SYSCALL	REG_RAX
#define REG_ARG0	REG_RDI
#define REG_ARG1	REG_RSI
#define REG_ARG2	REG_RDX
#define REG_ARG3	REG_R10
#define REG_ARG4	REG_R8
#define REG_ARG5	REG_R9
#endif
#ifndef SYS_SECCOMP
#define SYS_SECCOMP 1
#endif

const char *const msg="system call invalid: ";

static void write_uint(char *buf, unsigned int val)
{
    int width = 0;
    unsigned int tens;
    if (val == 0) {
        strcpy(buf, "0");
        return;
    }
    for (tens = val; tens; tens /= 10)
        ++ width;
    buf[width] = &#039;\0&#039;;
    for (tens = val; tens; tens /= 10)
        buf[--width] = (char) (&#039;0&#039; + (tens % 10));
}
static void helper(int nr, siginfo_t *info, void *void_context) {
    char buf[255];
    ucontext_t *ctx = (ucontext_t *)(void_context);
    unsigned int syscall;
    if (info->si_code != SYS_SECCOMP)
        return;
    if (!ctx)
        return;
    syscall = (unsigned int) ctx->uc_mcontext.gregs[REG_SYSCALL];
    strcpy(buf, msg);
    if (syscall < sizeof(syscall_names)) {
        strcat(buf, syscall_names[syscall]);
        strcat(buf, "(");
    }
    write_uint(buf + strlen(buf), syscall);
    if (syscall < sizeof(syscall_names))
        strcat(buf, ")");
    strcat(buf, "\n");
    write(STDOUT_FILENO, buf, strlen(buf));
    _exit(1);
}
static int install_helper() {
    struct sigaction act;
    sigset_t mask;
    memset(&act, 0, sizeof(act));
    sigemptyset(&mask);
    sigaddset(&mask, SIGSYS);
    act.sa_sigaction = &helper;
    act.sa_flags = SA_SIGINFO;
    if (sigaction(SIGSYS, &act, NULL) < 0) {
        perror("sigaction");
        return -1;
    }
    if (sigprocmask(SIG_UNBLOCK, &mask, NULL)) {
        perror("sigprocmask");
        return -1;
    }
    return 0;
}

#include <stdio.h>
int main(){
	if(install_helper()){
		printf("install helper failed");
		return 1;
	}

	scmp_filter_ctx ctx = NULL;
	ctx = seccomp_init(SCMP_ACT_ALLOW);

	seccomp_rule_add(ctx, SCMP_ACT_TRAP, SCMP_SYS(execve), 0);
	seccomp_load(ctx);
	seccomp_release(ctx);
	fprintf(stdout, "something to stdout\n");
	char * filename = "/bin/sh";
	char * argv[] = {"/bin/sh",NULL};
	char * envp[] = {NULL};
	write(1,"i will give you a shell\n",24);
	syscall(59,filename,argv,envp);//execve

	return 0;
}
```

### 注入程序
刚刚提到了seccomp必须由程序主动加载,因此需要有一个办法将代码注入到用户的代码中.

> 等待完成

### 其他语言
这玩意貌似至少能方便的用在c系语言,java和python上.

## Docker
准备使用Docker重构整个OJ

### 多阶段构建
使用Docker,将OJ分解为多个阶段来构建.

1. 前端构建
2. 后端构建
3. 运行环境构建与代码整合
4. nginx构建

#### 命令
```Dockerfile
#...

# 使用php作为基础,指明该构建阶段为codeisland
FROM php:alpine as codeisland
# 在容器打包阶段安装数据库驱动
RUN docker-php-ext-install pdo pdo_pgsql

ARG PATH=/app/laravel

# 从其他阶段复制代码到本阶段
COPY --from=DELETED /app/ ${PATH}

# 执行其他初始化命令
# [DELETED]

# 指明容器的工作路径
WORKDIR ${PATH}

#...
```

### 容器协调
使用Docker-compose来协调各个容器的关系.

* 数据库容器
* redis容器
* 网站后端容器
* 评测器容器
* 评测代理容器
* nginx容器

限制评测器容器的资源消耗的例子

```yml
deploy:
  resources:
    limits:
      cpus: &#039;0.50&#039;
      memory: 1024M
```

### 网络
...学校网关的登录状态根本没法维持,卡死.

