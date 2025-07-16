---
title: "[转]记录bash的操作"
description: "简单的Linux系统用户操作行为管理方案"
# category: 
# tags: [Linux]
---

转自：https://www.freeoa.net/osuport/sysec/simple-linux-sys-users-operate-manage-solution_1880.html


当需要登录服务器的用户增加时，就必须知道运维人员、开发人员或者黑客在机器上进行了什么样的操作，或如果机器上的重要文件被删除了，你是否很想知道是谁在什么时候删的？

本文将介绍这样的一种组合方案，来对用户登录后的行为进行跟踪和审计。

bash 本身就有记录命令的功能，即 history。可以执行 history 命令显示你之前执行了什么命令。但history有其固有的缺陷，比如默认有大小限制、可轻易被人篡改或清空或不记录。而history分散在各机器上既无法保证其完整性也不方便审计，所以我们需要将其统一收集起来。

# 方案基本思路
收集 history的思路比较简单，即将history写入文件并上传到服务端或者写入syslog，然后由 syslog实时的发到远程日志服务器，或将收集到的 history 进行后续处理则可以写入文件保存或者写入数据库供日后查询。而如何收集history，则可以从bash的现有功能或者源代码着手考虑。 主要有两种方法：

# 一、script方法
将用户执行的命令，以及命令所产生的结果都重定向到具体文件中。
将下面的命令添加到/etc/profile中： 

    exec /usr/bin/script -a -f -q /var/log/ops/`whoami`-`date +%Y%m%d%H%M`.log 

创建日志目录：mkdir /var/log/operation/ -p 

用户登录时便会自动记录用户的操作记录了，用户通过当前主机SSH到其他服务器上面去也可以记录用户在那台机器上的操作日志。

    /usr/bin/script –a  /var/log/operation/$USER.log 2>&1

有时在日志文件里不能一直看到命令行的输出，好像有丢失，不知何故。不推荐使用这种方法，'script'指令多年不再更新了，最新的手册页是2000年的。在用户登录后，它会开启另外一个进程(/usr/bin/script -a /var/log/operation/root.log 像这种形式的)，来监控用户的操作及其输出，当然从终端退出时要退出两次。

当操作输出量很大时(像导入mysql数据库)时，对应的记录日志文件会变的很大，对事后查找反而又变得不方便了。

    exec /usr/bin/script -a -f -q /var/log/ops/`whoami`-`date +%Y%m%d%H%M`.log

You can set the PS4 variable, which is evaluated for every command being executed just before the execution if trace is on:

    PS4='$(echo $(date) $(history 1) >> /tmp/trace.txt) TRACE: '

Then, enable trace:

    set -x

To stop tracing, just:

    set +x

# 二、设置bash 来记录的方法
## $PROMPT_COMMAND
If set, contains a command to execute before printing the prompt ($PS1). Set it to the name of a function that captures the output of history 1.

思路非常简单，就是bash程序将 history发到syslog，然后利用 syslog 集中存放并入库供方便查询。当然这里需要对系统日志系统进行相应的改造，具体操作方法见文章末尾的链接。

即利用PROMPT_COMMAND将当前执行的命令通过logger命令写入syslog，再加上当前用户名，去掉history 1 中显示的数字就完美了。我们在全局配置文件
/etc/bashrc里加上就可以记录所有用户了，具体方法如下：
 
使用'logger'指令记录在'/etc/(r)syslog.conf'配置文件里所定义的日志类型所指的日志文件中。

    logger -p local6.notice 'ssh of bash command.'

For BASH shells, edit the system-wide BASH runtime config file:

  vim /etc/bash.bashrc

在其末尾加入：

    Append to the end of that file:
    export PROMPT_COMMAND='RETRN_VAL=$?;logger -p local6.debug "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" ) [$RETRN_VAL]"'

我使用的方法：

    export PROMPT_COMMAND='{ Cmd=$(history 1 | { read x y; echo $y; }); logger -p local5.info "$HOSTNAME [HIST] : $SSH_CLIENT : $PWD : $Cmd"; }'

在日志文件中定义相关的条目(供'logger'指令调用)：
Set up logging for "local6" with a new file:

    sudo -e /etc/rsyslog.d/bash.conf

    And the contents...
    local6.*    /var/log/commands.log

重启日志服使其生效：
Restart rsyslog:

    sudo service rsyslog restart

在日志文件中记录如下：

    Apr 11 09:57:01 pde root: pde.freeoa.net [HIST] : 192.168.18.98 51917 22 : /var/log : cd /var/log/
    Apr 11 09:57:01 pde root: pde.freeoa.net [HIST] : 192.168.18.98 51917 22 : /var/log : ls
    Apr 11 09:57:08 pde root: pde.freeoa.net [HIST] : 192.168.18.98 51917 22 : /var/log : tail -100 messages
    Apr 11 10:17:03 pde root: pde.freeoa.net [HIST] : 192.168.18.98 51917 22 : /var/log : pwd

但是，这个方法非常容易被发现和被绕过。用`env|set`命令就可以看到`PROMPT_COMMAND`，而且现在的黑客都非常聪明，上来就执行`unset HISTORY HISTFILE HISTSAVE HISTZONE HISTORY HISTLOG PROMPT_COMMAND; export HISTFILE=/dev/null; export HISTSIZE=0; export HISTFILESIZE=0` 这样的命令防止执行的命令记录到history，那上面这个方法自然就失效了。

用户通过`unset` bash中的相关环境变量来逃避审计规则并不是没有方法阻止，我们依然可能通过较为高级权限设置来实现。

第一，更改相关记录文件的权限，使之只能追加，不能删除。这当然要借助于'chattr'指令来实现，注意只有'root'用户才能使用。

    # chattr +a .sh_history or chattr +a .bash_history

第二，设置shell内的变量，并使之在普通用户使用的指令为只读，尤其是与历史记录相关的变量；这需要借助于'typeset'指令来实现：
Use the shell’s typeset command with the -r option: this makes the specified variables read-only. This will make all history environment variables read-only. For example:

    export HISTCONTROL=
    export HISTFILE=$HOME/.bash_history
    export HISTFILESIZE=2000
    export HISTIGNORE=
    export HISTSIZE=1000
    export HISTTIMEFORMAT="%a %b %Y %T %z "
    
    typeset -r HISTCONTROL
    typeset -r HISTFILE
    typeset -r HISTFILESIZE
    typeset -r HISTIGNORE
    typeset -r HISTSIZE
    typeset -r HISTTIMEFORMAT

更多关于'HISTTIMEFORMAT'的相关信息，请参考文章尾部：“在bash的历史记录中显示命令执行过的日期及时间”段。

在bash中，可以通过'shopt'指令来修改其标准选项：

    shopt -s cmdhist
    shopt -s histappend

Setting cmdhist will put multiple line commands into a single history line, and setting histappend will make sure that the history file is added to, not overwritten as is usually done.

设置bash的'PROMPT_COMMAND'变量：

    PROMPT_COMMAND="history -a"
    typeset -r PROMPT_COMMAND

This is because bash actually writes the history in memory; the history file is only updated at the end of the shell session. This command will append the last command to the history file on disk.

也可以在bash的主配置文件中定义信号捕捉函数，来实现信息收集，Create a SIGDEBUG trap to send commands to syslog. 

但通过'history'方法当用户知道关于其使用方法后容易绕过，从而使上面的设置失效，下面我们说说改bash源代码的方法。可能有读者说改源代码这种事情好麻烦，其实bash4.1 就提供了这样的功能，你只需要启用它就可以了。如果不是这个版本则需要再进行相应修改，也不是非常难的事。
bash从4.1版本开始，支持将操作历史记录直接记录到系统日志文件(syslog)中，但这种特性需要在编译时开启。

# 三、修改bash源码以直接记录用户操作
这种方法需要修改bash源码来实现，如果不想用它来代替系统原有的bash，还要为用户指定新的bash shell位置，操作要复杂些；但不必在bash的配置文件设置了，用户也不能通过设置bash环境来逃过审计(前提是用户仅使用bash)。

修改的具体步骤如下：
主要是打开syslog宏定义调用函数，再配以其它需要记录的值就可以了。本文以目前最新的bash-4.2以例进行说明：
在其解压后的目录下有三个与日志历史记录相关的文件：config-top.h、config-bot.h、bashhist.c

对其进行简单的阅读后，只需修改'config-top.h'这个头文件，打开相应的注释即可。本例从104-108行：

    /* #define SYSLOG_HISTORY */
    #if defined (SYSLOG_HISTORY)
    #  define SYSLOG_FACILITY LOG_USER
    #  define SYSLOG_LEVEL LOG_INFO
    #endif

仅需要把104行`/* #define SYSLOG_HISTORY */`去掉注释`#define SYSLOG_HISTORY`，在将其编译安装到其它位置('/opt/ebash')。

然后修改用户的使用shell：

    usermod -s /opt/ebash/bin/bash hto
    more /etc/passwd|grep hto

可以检查一下用户'hto'的shell是不是修改成功了。然后用该用户登录，这样他的操作便会记录在日志文件中(debian下在'/var/log/messages'文件中)。记录如下：

    Apr 11 09:50:53 pde -bash: HISTORY: PID=12672 UID=1000 unset PROMPT_COMMAND
    Apr 11 09:50:55 pde -bash: HISTORY: PID=12672 UID=1000 w
    Apr 11 09:51:11 pde -bash: HISTORY: PID=12672 UID=1000 env
    Apr 11 09:53:33 pde -bash: HISTORY: PID=12672 UID=1000 date
    Apr 11 09:53:58 pde -bash: HISTORY: PID=12672 UID=1000 echo $BASH_VERSION

当然这种记录的字段及格式可能通过修改'bashhist.c'文件而改变。代码段如下：
    #if defined (SYSLOG_HISTORY)
    #define SYSLOG_MAXLEN 600
    
    void bash_syslog_history (line)
         const char *line;
    {
      char trunc[SYSLOG_MAXLEN];
    
      if (strlen(line) < SYSLOG_MAXLEN)
        syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY: PID=%d UID=%d %s", getpid(), current_user.uid, line);
      else
        {
          strncpy (trunc, line, SYSLOG_MAXLEN);
          trunc[SYSLOG_MAXLEN - 1] = '\0';
          syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY (TRUNCATED): PID=%d UID=%d %s", getpid(), current_user.uid, trunc);
        }
    }
    #endif

修改'syslog'函数的两种情况下的记录样式：

    syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY: PID=%d PPID=%d SID=%d User=%s Cmd=%s", getpid(), getppid(), getsid(getpid()),current_user.user_name, line);

    syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY (TRUNCATED): PID=%d PPID=%d SID=%d User=%s Cmd=%s", getpid(), getppid(), getsid(getpid()),current_user.user_name, trunc);

编译安装后，记录格式如下：

    Apr 11 11:26:05 pde -bash: HISTORY: PID=24849 PPID=24848 SID=24849 User=hto Cmd=unset PROMPT_COMMAND
    Apr 11 11:26:48 pde -bash: HISTORY: PID=24849 PPID=24848 SID=24849 User=hto Cmd=perl -v
    Apr 11 11:26:51 pde -bash: HISTORY: PID=24849 PPID=24848 SID=24849 User=hto Cmd=gcc -v
    Apr 11 11:26:57 pde -bash: HISTORY: PID=24849 PPID=24848 SID=24849 User=hto Cmd=ping www.freeoa.net

当用户'hto' su 成'root'后，root用户的默认shell不是改良后的bash shell时，便不记录操作日志了。只有将'root'用户的shell改为改良后的'bash'时，就可以记录后续的日志：

    Apr 11 11:31:51 pde -su: HISTORY: PID=24995 PPID=24994 SID=24849 User=root Cmd=w
    Apr 11 11:31:57 pde -su: HISTORY: PID=24995 PPID=24994 SID=24849 User=root Cmd=echo $BASH_VERSION
    Apr 11 11:32:19 pde -su: HISTORY: PID=24995 PPID=24994 SID=24849 User=root Cmd=ls /bin

上面的修改加入了进程及会话id信息，能记录是从其他用户'su'过去的一些信息。与老方案相比，它没有记录当前的工作目录，可能会对重现指令当时的情况有影响，当然加上它并不难，代码如下：

    syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY: PID=%d PPID=%d SID=%d User=%s Pwd=%s Cmd=%s", getpid(), getppid(), getsid(getpid()),current_user.user_name,get_current_dir_name(),line);

这里使用了'get_current_dir_name'函数，编译后重新安装。记录的日志格式如下：

    Apr 11 11:50:23 pde -bash: HISTORY: PID=25872 PPID=25871 SID=25872 User=hto Pwd=/home/hto Cmd=w
    Apr 11 11:50:25 pde -bash: HISTORY: PID=25872 PPID=25871 SID=25872 User=hto Pwd=/home/hto Cmd=date
    Apr 11 11:51:42 pde -bash: HISTORY: PID=25872 PPID=25871 SID=25872 User=hto Pwd=/home/hto Cmd=man scp
    Apr 11 11:54:48 pde -bash: HISTORY: PID=25872 PPID=25871 SID=25872 User=hto Pwd=/home/hto Cmd=su -
    Apr 11 11:54:59 pde -su: HISTORY: PID=25948 PPID=25947 SID=25872 User=root Pwd=/root Cmd=ls /sbin
    Apr 11 11:55:16 pde -su: HISTORY: PID=25948 PPID=25947 SID=25872 User=root Pwd=/root Cmd=cd /var/log
    Apr 11 11:55:20 pde -su: HISTORY: PID=25948 PPID=25947 SID=25872 User=root Pwd=/var/log Cmd=ls -lth

方案缺点：

1、该方案只能记录history类似的命令执行日志，无法记录通过程序或者脚本执行的命令；

2、该方案依然存在被用户绕过的风险，用户可以修改自己的默认shell，而linux默认提供了除bash 之外的其它shell，比如/bin/csh、/bin/zsh、/bin/ksh等，如果用户登录之后将自己的bash 改为/bin/csh或没加记录history到syslog的 bash 则无法记录，或者直接执行/bin/csh即可绕过，当然你可以将不用的shell 想办法去掉；

3、关于审计linux操作还有一些其它的技术实现，比如加载内核模块拦截系统调用或者读取系统pty接口等。

--------------------
Bash History: Display Date And Time For Each Command
在bash的历史记录中显示命令执行过的日期及时间

需要设置'HISTTIMEFORMAT'环境变量来实现：
If the HISTTIMEFORMAT is set, the time stamp information associated with each history entry is written to the history file.Defining the environment variable as follows:

    $ HISTTIMEFORMAT="%d/%m/%y %T "

OR

    $ echo 'export HISTTIMEFORMAT="%d/%m/%y %T "' >> ~/.bash_profile
    
Where,
    
    %d - Day
    %m - Month
    %y - Year
    %T - Time
    
    $ history

会有如下输出：
    
    987  11/03/10 04:31:36 w
    988  11/03/10 04:31:37 iostat
    989  11/03/10 04:31:37 top
    993  11/03/10 04:31:41 grep CPU /proc/cpuinfo
    994  11/03/10 04:31:45 vmstat 3 100
    ..

参考-References:

    man bash
    help history
    man 3 strftime

--------------
Don't save commands in bash history (only for current session)
如何在bash shell中不记录当前执行过的命令

通过对'HISTFILE'环境变量的操作来实现(这种设置仅对当前会话有效)

    unset HISTFILE

this will cause any commands that you have executed in the current shell session to not be written in your bash_history file upon logout.

    export HISTSIZE=0

Don't save commands in bash history (only for current session).

    HISTFILE=/dev/null

disable history for current shell session.
禁止记录当前会话

    history -c

Clear current session history (bash).
清空当前会话的历史记录

    ssh user@host "> ~/.bash_history"

Don't save commands in bash history (only for current session)
Only from a remote machine:
Only access to the server will be logged, but not the command.
The same way, you can run any command without loggin it to history.
ssh user@localhost will be registered in the history as well, and it's not usable.
