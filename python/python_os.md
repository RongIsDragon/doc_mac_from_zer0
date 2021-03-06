# 操作系统接口 - `os`模块

此模块可以让我们“可移植”的使用平台相关函数。如果你只是想要读写文件，可以使用`__builtins__['open']`函数；如果你想要操作文件系统路径，可以使用`os.path`模块；如果你想要在命令行模式下读取所有指定文件的内容，可以使用`fileinput`模块；如果你想要创建临时文件及文件夹，可以使用`tempfile`模块；如果需要文件操作的高级函数，可以使用`shutil`模块。

留意此模块中函数的共性：

* Python设计这个边平台相关内建模块的初衷，是为了让同一个功能在任何支持的平台上，拥有相同的接口。比如，`os.stat(path)`函数在所有支持的平台上都以相同的形式返回*path*的状态（这个返回状态恰巧来自POSIX系统）；
* `os`模块也支持一些平台特有的属性，不过，使用这些平台特有属性可能会影响到代码的可移植性；
* 模块中接受路径或文件名做参数的函数都支持`str`和`bytes`两种类型。如果该函数返回路径或文件名，返回值的类型会与参数类型相同；
* 诸如“支持：Unix”的标记表示在Unix上更常见，而并不是说仅Unix提供支持；
* 如果不特意指明，“支持：Unix”同样使用于Unix内核的Mac OS X；

注意：模块中的函数在遇到：不合法或不可访问的路径或文件名，参数正确但系统不支持时会抛出`OSError`。

### exception os.error

内建异常`OSError`的别名。

### os.name

导入此模块时基于的操作系统名称，现在注册的名称有'posix', 'nt', 'mac', 'ce', 'java'。

`sys.platform`的结果更具体，`os.uname()`的结果还包含操作系统版本等信息。

要得到更详细的系统信息，可以使用`platform`模块。

## 文件名、命令行参数、环境变量

Python中使用`str`表示文件名、命令行参数、环境变量。对于模型操作环境，有必要将这些字符串转为`bytes`再传给操作系统。Python使用文件系统默认编码做这种转换（见`sys.getfilesystemencoding()`）。

Python3.1后：在一些环境中，使用文件系统编码可能会产生错误。所以Python使用了`surrogateescape`控制编码异常，也就是把不可被解码的`bytes`在解码时记为Unicode字符“U+DCxx”，于是这种字符在编码时会被重新译为原始`bytes`。

文件系统默认编码至少要做到能够正确解码每个数值小于128的`bytes`，否则API可能会抛出`UnicodeErrors`异常。

## 进程参数

下列函数和属性提供了当前进程及用户的信息与相应操作。

### os.ctermid()

返回控制进程的终端所关联的文件。

```
>>> os.ctermid()
'/dev/tty'
```

支持：Unix。

### os.environ

用`str`储存当前环境变量的映射。如`environ['HOME']`返回当前的用户目录（部分系统支持），相当于C语言中的`getenv("HOME")`。

这个映射在`os`模块被第一次导入的时候就生成了（如，当Python作为`site.py`的一部分启动的时候）。在此之后被修改的环境变量不会体现在`os.environ`中，除非直接修改`os.environ`本身。

如果平台支持`putenv()`，此映射可能既可以用来查询环境变量，也可以用来修改环境变量。`putenv()`会在修改映射时被自动调用。

在Unix上，映射的键与值会使用`sys.getfilesystemencoding()`和`surrogateescape`控制编码异常。如果希望使用不同的编码，可以访问`environb`属性。

注意：直接调用`putenv()`并不会改变`os.environ`，所以还是修改`os.environ`比较好。

注意：在某些环境中（包括FreeBSD和Mac OS X），直接修改`environ`属性可能会导致内存泄露。详情参见操作系统的`putenv`命令或函数。

如果平台不支持`putenv()`，一份被修改的环境变量映射可能会被进程创建函数传给子进程，使得子进程可以使用修改了的环境变量。

如果平台支持`unsetenv()`，就可以通过删除此映射中的项来释放环境变量。当`os.environ`中的项被删除时（包括`pop()`或`clear()`等方法）会自动调用`unsetenv()`。

### os.environb

`os.environ`的`bytes`版。`environ`与`environb`是同步的，修改其中一个会导致另一个也被修改。

`environb`只有在`supports_bytes_environ`为真的时候可用。

#### os.chdir(path)
#### os.fchdir(fd)
#### os.getcwd()

这三个函数在“文件和路径”中详细描述。

### os.fsencode(filename)

使用文件系统编码及`surrogateescape`异常控制来编码*filename*，在Windows上使用`strict`异常控制。若*filename*为`bytes`则直接返回这个字节串。

### os.fsdecode(filename)

使用文件系统编码及`surrogateescape`异常控制来解码*filename*，在Windows上使用`strict`异常控制。若*filename*为`str`则直接返回这个字符串。

### os.getenv(key, default=None)

查询环境变量，如果存在就返回*key*，不存在就返回*default*。*key*、*default*及返回值都是`str`。

在Unix上，映射的键与值会使用`sys.getfilesystemencoding()`和`surrogateescape`控制编码异常。如果希望使用不同的编码，可以调用`os.getenvb()`。

支持：多数Unix，Windows。

### os.getenvb(key, default=None)

查询环境变量，如果存在就返回*key*，不存在就返回*default*。*key*、*default*及返回值都是`bytes`。

支持：多数Unix。

### os.get_exec_path(env=None)

返回可执行对象的搜索路径列表，类似于shell在执行一个命令时的动作。如果指定*env*，则应该是能够查找到PATH的环境变量。*env*默认为空时，会使用`os.environ`。

```
>>> os.get_exec_path()
['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin', '/opt/X11/bin']
```

### os.getegid()

返回对当前进程有效的组ID，与当前进程中被执行文件的设置ID位有关。

支持：Unix。

### os.geteuid()

返回对当前进程有效的用户ID。

支持：Unix。

### os.getgid()

返回当前进程所属的组ID。

支持：Unix。

### os.getgrouplist(user, group)

返回*user*所属的组ID列表。如果*group*不在列表中，则会被加入。*group*通常来自*user*密码记录中的组ID域。

支持：Unix。

### os.getgroups()

返回当前进程关联的附加组ID。

支持：Unix。

注意：`getgroups()`在Mac OS X中的行为与在其他Unix中的行为有所不同。如果Python解释器编译部署在10.5及更早的版本，`getgroups()`返回与当前用户进程相关的的有效组ID列表，这时的列表的长度会被一个系统定义的整数限制，通常是16，而且列表可能会适当的权限下被`setgroups()`修改。如果是大于10.5的版本，`getgroups()`返回与当前进程有效用户ID相关的用户所在用户组的许可列表。组许可列表可以在进程生命周期的任何时间被修改，`setgroups()`不会修改列表，而且列表大小不再限制在16上。可以通过`sysconfig.get_config_var()`查询部署值`MACOSX_DEPLOYMENT_TARGET`。

### os.getlogin()

返回进程控制终端的登录用户名。通常在需要使用到环境变量`LOGNAME`或`USERNAME`时调用，以检查登录用户名，或使用`pwd.getpwuid(os.getuid())[0]`得到当前有效的用户登录名。

支持：Unix，Windows。

### os.getpgid(pid)

返回ID为*pid*的进程所属进程组的ID。如果*pid*为0，则返回当前进程所属进程组的ID。

支持：Unix。

### os.getpgrp()

返回当前进程所属的进程组ID。

支持：Unix。

### os.getpid()

放回当前进程ID。

支持：Unix，Windows。

### os.getppid()

返回父进程ID。在Unix上，当父进程退出时，返回值为当前进程的一个初始化进程的ID；而在Windows上仍然返回其父进程原有的ID，不过此时这个ID可能意见被其他进程占用了。

支持：Unix，Windows。

### os.getpriority(which, who)

获得程序的调度优先级。*which*的值应是`PRIO_PROCESS`, `PRIO_PGRP`, `PRIO_USER`中的一个，*who*是相对于*which*（`PRIO_PROCESS`的进程标识符，`PRIO_PGRP`的进程组标识符，`PRIO_USER`的用户ID）的。将*who*置为0表示调用的进程，调用的进程所属的进程组，或调用进程的用户ID。

支持：Unix。

* os.PRIO_PROCESS
* os.PRIO_PGRP
* os.PRIO_USER

为`getpriority()`和`setpriority()`提供参数。

支持：Unix。

### os.getresuid()

以元组形式返回当前进程的实际组ID（rgid）、有效组ID（egid）和设置组ID（sgid）。

支持：Unix。

### os.getuid()

返回当前进程的用户ID。

支持：Unix。

### os.initgroups(username, gid)

根据用户*username*在的所有组，以及*gid*指定的组，调用系统的initgroups()函数以初始化组访问列表。

### os.putenv(key, value)

将名为*key*的环境变量值置为*value*。此修改会影响使用`os.system()`, `os.popen()`, `os.fork()`, `os.execv()`发起的子进程。

支持：多数Unix，Windows。

注意：在某些环境中（包括FreeBSD和Mac OS X），直接修改`environ`属性可能会导致内存泄露。详情参见操作系统的`putenv`命令或函数。

如果操作系统支持`putenv()`，对`os.environ`的赋值会调用`putenv()`去修改对应的环境变量。但是，直接使用`putenv()`修改环境变量不会更新`os.environ`。所以，更推荐使用`os.environ`给环境变量赋值。

### os.setegid(egid)

设置当前进程的有效组ID。

支持：Unix。

### os.seteuid(euid)

设置当前进程的有效用户ID。

支持：Unix。

### os.setgid(gid)

设置当前进程组ID。

支持：Unix。

### os.setgroups(groups)

将当前进程关联的附加组置为*groups*。*groups*必须是一个序列，其中的每个元素都是组ID的整数。通常只有超级用户可以使用此函数。

支持：Unix。

注意：对于Mac OS X，*groups*的长度可能会受到系统最大有效组ID的限制，通常为16。详情参见`getgroups()`的说明文档。

### os.setpgrp()

根据系统的支持（如果支持的话），调用系统的`setpgrp()`或`setpgrp(0, 0)`函数。语法参见`man`命令。

支持：Unix。

### os.setpgid(pid, pgrp)

调用系统的`setpgid()`函数，ID为`pid`的进程所属进程组的ID置为*pgrp*。语法参见`man`命令。

支持：Unix。

### os.setpriority(which, who, priority)

设置程序的调度优先级。*which*的值应是`PRIO_PROCESS`, `PRIO_PGRP`, `PRIO_USER`中的一个，*who*是相对于*which*（`PRIO_PROCESS`的进程标识符，`PRIO_PGRP`的进程组标识符，`PRIO_USER`的用户ID）的。将*who*置为0表示调用的进程，调用的进程所属的进程组，或调用进程的用户ID。*priority*在-20到19取值，默认为0，值越小越容易调度。

支持：Unix。

### os.setregid(rgid, egid)

设置当前进程的实际组ID和有效组ID。

支持：Unix。

### os.setresgid(rgid, egid, sgid)

设置当前进程的实际组ID、有效组ID和设置组ID。

支持：Unix。

### os.setresuid(ruid, euid, suid)

设置当前进程的实际用户ID、有效用户ID和设置用户ID。

支持：Unix。

### os.setreuid(ruid, euid)

设置当前进程的实际用户ID和有效用户ID。

支持：Unix。

### os.getsid(pid)

调用系统的`getsid()`。语法参见`man`命令。

支持：Unix。

### os.setsid()

调用系统的`setsid()`。语法参见`man`命令。

支持：Unix。

### os.setuid(uid)

设置当前进程的用户ID。

支持：Unix。

### os.strerror(code)

返回错误代码为`code`的错误信息。当提供未知错误代码时，平台的`strerror()`返回`NULL`，抛出`ValueError`异常。

支持：Unix，Windows。

### os.supports_bytes_environ

如果原生系统环境为`bytes`类型则返回`True`。（如在Windows上为`False`）

### os.umask(mask)

设置当前umask码，并返回原来的umask码。

支持：Unix，Windows。

### os.uname()

返回操作系统的标识信息。返回值有五个属性：

* `sysname` - 操作系统名；
* `nodename` - 网络节点名；
* `release` - 系统发布版本号；
* `version` - 操作系统版本；
* `machine` - 硬件标识；

为了保证向后兼容性，返回值是一个可迭代的、行为类似元组的、元素按顺序分别是`sysname`, `nodename`, `release`, `version`, `machine`的对象。

一些系统会截断`nodename`，返回8个字符或首个词组。推荐使用`socket.gethostname()`或`socket.gethostbyaddr(socket.gethostname())`获得主机名。

支持：多数Unix。

### os.unsetenv(key)

释放（删除）名为*key*的环境变量，这会影响到使用`os.system()`, `os.popen()`, `fork()`, `execv()`发起的子进程。

当系统支持`unsetenv()`时，删除`os.environ`中的项会自动调用`unsetenv()`以删除相应的环境变量。但是，直接调用`unsetenv()`删除环境变量不会更新`os.environ`。所以，更推荐删除`os.environ`中的项。

支持：多数Unix，Windows。

## [创建文件对象](id:file-object-creation)

此函数会创建新的文件对象（见`os.open()`，用以打开文件描述符）。

### os.fdopen(fd, *args, **kwargs)

返回一个与文件描述符*fd*关联的打开的文件对象。这是内建函数`open()`的别名，他们接受相同的参数，唯一的区别是`fdopen()`的第一个参数必须是整数。

## 文件描述符操作

以下的函数通过文件描述符操作I/O流。

文件描述符，是一个较小的整数，关联着当前进程中打开的文件。标准输入（stdin）的文件描述符通常为0，标准输出（stdout）为1，标准错误（stderr）为2，进程接下来打开的文件将获得3、4、5等文件描述符。“文件描述符”这个名称可能会误导用户，因为在Unix中，sockets和pipes也是通过文件描述符引用的。

`io.IOBase.fileno()`方法可以用来在需要的时候获取文件描述符。值得注意的是，如果绕过文件对象的方法，直接使用文件描述符，可能会忽略掉内部缓冲数据。

### os.close(fd)

关闭文件描述符*fd*。

支持：Unix，Windows。

注意：此函数用于支持底层I/O，通常用来关闭诸如`os.open()`或`pipe()`等函数返回的文件描述符。如果要关闭类似内建函数`open()`或`os.popen()`和`os.fdopen()`等函数打开的文件对象时，请使用文件对象自己的`close()`方法。

### os.closerange(fd_low, fd_high)

关闭介于*fd_low*（包括）与*fd_high*（不包括）之间的所有文件描述符，并忽略异常，相当于（当然，比下面的代码快的多）：

```
for fd in range(fd_low, fd_high):
    try:
        os.close(fd)
    except OSError:
        pass
```

支持：Unix，Windows。

### os.device_encoding(fd)

如果*fd*关联的设备连接到一个终端，则返回一个描述设备编码的字符串，否则返回`None`。

### [os.dup(fd)](id:os.dup)

返回文件描述符*fd*的副本，新的文件描述符是不可继承的。

在Windows上，如果复制标准流（stdin:0, stdout:1, stderr:2），新的文件描述符是可继承的。

支持：Unix，Windows。

### os.dup2(fd, fd2, inheritable=True)

将文件描述符*fd*复制到*fd2*，并会根据需要关闭后者。*fd2*默认是可继承的，除非将`inheritable`置为`False`。

支持：Unix，Windows。

### os.fchmod(fd, mode)

将给定文件*fd*的模式设置为*mode*。可用的*mode*参见`os.chmod()`的说明。对于Python3.3，相当于`os.chmod(fd, mode)`。

支持：Unix。

### os.fchown(fd, uid, gid)

将文件*fd*的属主码置为*uid*，属组码置为*gid*。若不需要改变文件某个id属性，置为-1即可。参见`os.chown()`。对于Python3.3，相当于`os.chown(fd, uid, gid)`。

支持：Unix。

### os.fdatasync(fd)

强制将与文件描述符*fd*关联的文件写入磁盘，但并不强制更新文件的元数据。

支持：Unix。

注意：MacOS不支持此函数。

### os.fpathconf(fd, name)

返回与被打开文件相关的的系统配置信息。用*name*参数指定取回的配置值，参数应为系统定义值，可能是字符串形式，这些系统定义值可以在某些标准（POSIX.1, Unix 95, Unix 98等）里找到。部分平台也支持额外的这类配置值。这些操作系统相关的配置值在`pathconf_names`目录中给出。如果有配置值并不在目录的映射中，则可以给传给*name*一个整型。

如果*name*是字符串，且未定义在系统中，则会抛出`ValueError`；如果*name*不被操作系统支持，即使定义在`pathconf_names`中，也会抛出`OSError`，并附带一个`errno.EINVAL`的错误代码。

对于Python3.3，相当于`os.pathconf(fd, name)`。

支持：Unix。

### os.fstat(fd)

返回文件描述符*fd*的状态，类似`os.stat()`。对于Python3.3，相当于`os.stat(fd)`。

支持：Unix，Windows。

### os.fstatvfs(fd)

返回包含与文件描述符*fd*相关联的文件的文件系统信息，类似`os.statvfs()`。对于Python3.3，相当于`os.statvfs(fd)`。

支持：Unix。

### [os.fsync(fd)](id:os.fsync)

强制将与文件描述符*fd*关联的文件写入磁盘。在Unix上，会调用系统的`fsync()`，在Windows上则是`_commit()`。

如果你操作的是一个有数据在缓冲区的文件对象*f*，首先应使用`f.flush()`，而后再使用`os.fsync(f.fileno())`，这样可以保证与文件对象*f*关联的缓冲区数据都被写入磁盘。

支持：Unix，Windows。

### os.ftruncate(fd, length)

截断文件描述符*fd*关联的文件，使其不超过*length*个字节。对于Python3.3，相当于`os.truncate(fd, length`。

支持：Unix。

### os.isatty(fd)

如果文件描述符*fd*关联的文件在类tty设备中打开，返回`True`，否则返回`False`。

### os.lockf(fd, cmd, len)

在一个打开的文件描述符*fd*上添加、测试或删除一个POSIX锁。*cmd*使用`F_LOCK`, `F_TLOCK`, `F_ULOCK`, `F_TEST`中的一个指定操作类型，*len*指定锁定的部位。

支持：Unix。

* os.F_LOCK
* os.F_TLOCK
* os.F_ULOCK
* os.F_TEST

指定`lockf()`的动作。

### os.lseek(fd, pos, how)

将文件描述符*fd*的游标移动至*pos*，移动的动作设置：`SEEK_SET`或0，表示相对于文件开头移动；`SEEK_CUR`或1，表示相对于当前游标的位置移动；`SEEK_END`或2，表示相对于文件结尾移动。动作结束后以字节形式返回当前游标相对于文件开头的位置。

支持：Unix，Windows。

* os.SEEK_SET
* os.SEEK_CUR
* os.SEEK_END

函数`lseek()`的参数，其值分别为0, 1, 2。

支持：Unix，Windows。

某些操作系统还支持额外的类似`os.SEEK_HOLE`和`os.SEEK_DATA`的操作。

### os.open(file, flags, mode=0o777, *, dir_fd=None)

以*mode*方式打开文件*file*，并根据*flags*设置标识。计算*mode*时，会先使用当前掩码遮罩输出。并返回新打开文件的文件描述符。新的文件描述符是不可继承的。

关于*flags*与*mode*的描述，参见C运行时文档。`os`模块中定义了标识常量（如`O_RDONLY`, `O_WRONLY`等）。特别的，在Windows上，如果需要以二进制方式打开文件，应使用`O_BINARY`。

通过设置*dir_fd*可以使此函数支持“相对路径文件描述符”。

支持：Unix，Windows。

注意：此函数用于支持底层I/O，正常情况下推荐使用返回值为文件对象的内建函数`open()`，文件对象拥有`read()`, `write()`等更方便的方法。如果需要用文件对象来封装得到的文件描述符，可以使用`os.fdopen()`函数。

以下常量为`os.open()`的*flags*参数选项，可以通过按位并或`|`操作符组合使用多个选项。其中某些并不在所有平台上支持。选项的使用详情参见`open(2)`的`man`文档（Unix），或MSDN（Windows）。

* os.O_RDONLY
* os.O_WRONLY
* os.O_RDWR
* os.O_APPEND
* os.O_CREAT
* os.O_EXCL
* os.O_TRUNC

以上是Unix和Windows支持的。

* os.O_DSYNC
* os.O_RSYNC
* os.O_SYNC
* os.O_NDELAY
* os.O_NONBLOCK
* os.O_NOCTTY
* os.O_SHLOCK
* os.O_EXLOCK
* os.O_CLOEXEC

以上是仅Unix支持的。

* os.O_BINARY
* os.O_NOINHERIT
* os.O_SHORT_LIVED
* os.O_TEMPORARY
* os.O_RANDOM
* os.O_SEQUENTIAL
* os.O_TEXT

以上是仅Windows支持的。

* os.O_ASYNC
* os.O_DIRECT
* os.O_DIRECTORY
* os.O_NOFOLLOW
* os.O_NOATIME
* os.O_PATH
* os.O_TMPFILE

以上是GNU附加的，如果未在C语言库中被定义，则不支持。

### os.openpty()

打开新的伪终端对，并返回一对文件描述符`(master, slave)`，分别代表pty和tty。这些文件描述符是不可继承的。如果需要可移植性强（轻量级）的实现，可以使用`pty`模块。

支持：多数Unix。

### os.pipe()

新建管道，返回一对文件描述符`(r, w)`，分别用于读和写。这些文件描述符是不可继承的。

支持：Unix，Windows。

### os.pipe2(flags)

以原子方式设置为新建的管道设置*flags*。多个*flags*选项（`O_NONBLOCK`, `O_CLOEXEC`）间可以使用并操作组合。返回一对文件描述符`(r, w)`，分别用于读和写。这些文件描述符是不可继承的。

### os.posix_fallocate(fd, offset, len)

用于确保磁盘上有足够的空间，以容纳*fd*指定的文件中从*offset*开始长度为*len*字节的内容。

支持：Unix。

### os.posix_fadvise(fd, offset, len, advice)

通过指定的模式*advice*访问文件*fd*中从*offset*开始后*len*个字节的数据，模式*advice*是`POSIX_FADV_NORMAL`, `POSIX_FADV_SEQUENTIAL`, `POSIX_FADV_RANDOM`, `POSIX_FADV_NOREUSE`, `POSIX_FADV_WILLNEED`, `POSIX_FADV_DONTNEED`中的一种。这样可以允许内核做相应的优化。

支持：Unix。

* os.POSIX_FADV_NORMAL
* os.POSIX_FADV_SEQUENTIAL
* os.POSIX_FADV_RANDOM
* os.POSIX_FADV_NOREUSE
* os.POSIX_FADV_WILLNEED
* os.POSIX_FADV_DONTNEED

`posix_fadvise()`中的*advice*参数所能够使用的选项，用以指定可能的文件访问模式。

### os.pread(fd, buffersize, offset)

从文件描述符*fd*的*offset*处开始，读取*buffersize*个字节，此函数不会改变该文件游标的偏移量。

支持：Unix。

### os.pwrite(fd, string, offset)

向文件描述符*fd*的*offset*处写入*string*，此函数不会改变该文件描述符游标的偏移量。

### os.read(fd, n)

从文件描述符*fd*中读取最多*n*字节的内容，以字节串形式返回读取的内容。如果遇到*fd*指定的文件末端，则返回空的`bytes`对象。

支持：Unix，Windows。

注意：此函数用于支持底层I/O，通常用来读取诸如`os.open()`或`pipe()`等函数返回的文件描述符。如果要读取类似内建函数`open()`或`os.popen()`和`os.fdopen()`等函数打开的文件对象或`sys.stdin`时，请使用文件对象自己的`read()`或`readline()`方法。

### os.sendfile(out, in, offset, nbytes)
### os.sendfile(out, in, offset, nbytes, headers=None, trailers=None, flags=0)

从文件描述符*in*中读取*nbytes*字节，写入文件描述符*out*的*offset*处，返回成功写入的字节数。当遇到EOF时，返回0。

第一个函数适用于定义了`sendfile()`命令或函数的所以平台。

在Linux上，如果*offset*被置为`None`，则从*in*的当前游标读取内容后，*in*的游标位置会被更新。

第二个函数常用于Mac OS X和FreeBSD，其中的*headers*和*trailers*是任意序列，分别在*in*的数据写入前和写入后被更新。此函数与上一个函数返回值相同。

在Mac OS X和FreeBSD上，将*nbytes*置为0，表示持续发送*in*中的内容，直到*in*读取结束。

所有平台都支持将socket作为文件描述符*out*的参数，一些平台也支持诸如普通文件、管道等其他类型。

* os.SF_NODISKIO
* os.SF_MNOWAIT
* os.SF_SYNC

`sendfile()`函数的*flag*选项，如果平台支持的话。

### os.readv(fd, buffers)

将件描述符*fd*的内容读入至一系列可变类字节对象（bytes-like object）*buffers*中。`readv()`将数据装入一个buffer直至装满，而后转向*buffers*中的下一个buffer继续写入数据。`readv()`返回读取的总字节数（可能会比*buffers*中所有对象能够容纳的总字节数小）。

支持：Unix。

### os.tcgetpgrp(fd)

返回*fd*（一个由`os.open()`返回的文件描述符）上打开的终端相关联的进程组ID。

支持：Unix。

### os.tcsetpgrp(fd, pg)

将*fd*（一个由`os.open()`返回的文件描述符）上打开的终端相关联的进程组ID置为*pg*。

支持：Unix。

### os.ttyname(fd)

返回与文件描述符*fd*相关联的终端设备名称。如果*fd*没有与终端相关联，则抛出异常。

支持：Unix。

### os.write(fd, str)

将*str*中的字节串写入文件描述符*fd*。返回实际写入的字节数。

支持：Unix，Windows。

注意：此函数用于支持底层I/O，通常用来写入诸如`os.open()`或`pipe()`等函数返回的文件描述符。如果要写入类似内建函数`open()`或`os.popen()`和`os.fdopen()`等函数打开的文件对象或`sys.stdin`时，请使用文件对象自己的`write()`方法。

### os.writev(fd, buffers)

将*buffers*中的内容写入文件描述符*fd*。*buffers*必须是类字节对象的序列，`writev()`将每个*buffers*中的对象内容都写入*fd*，并返回写入的总字节数。

支持：Unix。

### 查询终端大小

#### os.get_terminal_size(fd=STDOUT_FILENO)

以类似`(columns, lines)`的形式返回终端窗口大小，返回值为`os.terminal_size`类型。

可选参数*fd*指定需要使用的文件描述符，默认为`STDOUT_FILENO`或标准输出。

通常使用`shutil.get_terminal_size()`这个高级函数来获取终端窗口大小，`os.get_terminal_size`是一个底层的实现。

支持：Unix，Windows。

#### class os.terminal_size

tuple的子类，以`(columns, lines)`的形式存放终端大小。

* columns为终端窗口的宽度，可以容纳多少字节；
* lines为终端窗口的高度，可以容纳多少字节；

### 文件描述符的继承

文件描述符都有“可继承”标识，用以表明该描述符是否可以被子进程继承。在Python3.4中，由Python创建的文件描述符默认是不可继承的。

在Unix上，子进程在处理新程序时会关闭不可继承的文件描述符，其余的文件描述符是都是可继承的。

在Windows上，子进程会关闭不可继承的句柄和除了标准流（分别代表stdin, stdout, stderr的文件描述符0, 1, 2总是可继承的）以外的文件描述符。如果使用`spawn*`系列函数，所有可继承句柄和可继承文件描述符都可以被继承。如果使用`subprocess`模块，除了标准流以外的文件描述符都会被关闭，而只有当`close_fds`置为`False`时，可继承句柄才会被继承。

#### os.get_inheritable(fd)

返回指定文件描述符*fd*的可继承标识（布尔值）。

#### os.set_inheritable(fd, inheritable)

将指定文件描述符*fd*的可继承标识置为*inheritable*。

#### os.get_handle_inheritable(handle)

返回指定句柄*handle*的可继承标识（布尔值）。

支持：Windows。

#### os.set_handle_inheritable(handle, inheritable)

将指定句柄*handle*的可继承标识置为*inheritable*。

支持：Windows。

## 文件与目录

对于一些Unix平台，模块中的很多函数都有以下一种或几种特性：

* [指定文件描述符](id:path-fd)：对于很多函数，*path*参数不仅可以接受一个字符串作为路径名，而且可以接受一个文件描述符。若是文件描述符，则函数会继续操作描述符所关联的文件。（对于POSIX系统，Python会调用该函数的`f...`版本以支持对文件描述符的操作。）
    
    可以通过[`os.supports_fd`](#os.supports_fd)标识来判断当前平台是否支持将文件描述符当做*path*参数。如果不支持时强制使用文件描述符，则会抛出`NotImplementedError`异常。
    
    如果函数有*dir_fd*和*follow_symlinks*参数，且当*path*参数为文件描述符时，指定前面的两个参数是错误的。
    
* [目录描述符的相对路径](id:dir-fd)：若*dir_fd*不为`None`，则应指定为某个目录关联的文件描述符，其他的参数都应指定为该目录的相对路径。若提供绝对路径，则*dir_fd*会被忽略。（对于POSIX系统，Python会调用该函数的`...at`和`f...at`版本以支持对目录描述符的操作。）
    
    可以通过`os.supports_dir_fd`标识来判断当前平台是否支持将文件描述符当做*dir_fd*参数。如果不支持时强制使用目录描述符，则会抛出`NotImplementedError`异常。
    
* [不跟踪符号链接](id:follow-symlinks)（软链接）：若*follow_symlinks*为`False`，且路径最后一个元素所指的是一个符号链接，函数会直接操作该符号链接，而不是符号链接所指的对象。（对于POSIX系统，Python会调用该函数的`l...`版本以支持对符号链接的操作。）
    
    可以通过`os.supports_follow_symlinks`标识来判断当前平台是否支持符号链接操作。如果不支持时强行操作符号链接，则会抛出`NotImplementedError`异常。

### os.access(path, mode, *, dir_fd=None, effective_ids=False, follow_symlinks=True)

使用实际uid/gid测试是否可以访问*path*。应该了解的是，大多数操作使用有效uid/gid，所以在使用suid/sgid的环境中同样可以用于测试调用者是否对*path*有足够的访问权限。将*mode*置为`F_OK`用以测试*path*是否存在，也可以通过并操作结合`R_OK`, `W_OK`, `X_OK`选项测试访问权限。如果允许访问则返回`True`，否则返回`False`。更多细节参见`access(2)`的`man`手册。

此函数支持设置[目录描述符的相对路径](#dir-fd)和[不跟踪符号链接](#follow-symlinks)。

如果`effective_ids`置为`True`，`access()`将使用有效uid/gid代替实际uid/gid进行访问权限测试。可以通过`os.supports_effective_ids`标识来判断当前平台是否支持有效ID。如果不支持时强行设置，则会抛出`NotImplementedError`异常。

支持：Unix，Windows。

注意：在使用`open()`或其他操作前调用`access()`测试用户是否具有访问权限，会造成安全漏洞。因为用户可能会利用**检查权限**与**打开文件**的间隙操作文件。推荐使用EAFP(Easier to ask for forgiveness than permission)技术解决这个问题：

可能被利用的代码：

```
if os.access("myfile", os.R_OK):
    with open("myfile") as fp:
        return fp.read()
return "some default data"
```

改写为EAFP模式：

```
try:
    fp = open("myfile")
except PermissionError:
    return "some default data"
else:
    with fp:
        return fp.read()
```

注意：即使通过了`access()`测试，I/O操作仍可能失败。这种情况更可能发生于网际间的文件系统，因为这种系统可能具有比POSIX的权限位更复杂的访问控制机制。

* os.F_OK
* os.R_OK
* os.W_OK
* os.X_OK

`access()`函数的*mode*选项，分别用于测试存在、可读、可写、可执行。

### os.chdir(path)

改变当前工作目录。

此函数支持[指定文件描述符](#path-fd)。描述符必须指向一个打开的目录，而不是文件。

支持：Unix，Windows。

### os.chflags(path, flags, *, follow_symlinks=True)

将*path*的标识置为*flags*。*flags*可以是下列选项（定义在`stat`模块中）的任意组合（按位或）：

* stat.UF_NODUMP
* stat.UF_IMMUTABLE
* stat.UF_APPEND
* stat.UF_OPAQUE
* stat.UF_NOUNLINK
* stat.UF_COMPRESSED
* stat.UF_HIDDEN
* stat.SF_ARCHIVED
* stat.SF_IMMUTABLE
* stat.SF_APPEND
* stat.SF_NOUNLINK
* stat.SF_SNAPSHOT

此函数支持[不跟踪符号链接](#follow-symlinks)。

### [os.chmod(path, mode, *, dir_fd=None, follow_symlinks=True)](id:os.chmod)

将*path*的模式置为*mode*。*mode*可以说下列选项（定义在`stat`模块中）的任意组合（按位或）：

* stat.S_ISUID
* stat.S_ISGID
* stat.S_ENFMT
* stat.S_ISVTX
* stat.S_IREAD
* stat.S_IWRITE
* stat.S_IEXEC
* stat.S_IRWXU
* stat.S_IRUSR
* stat.S_IWUSR
* stat.S_IXUSR
* stat.S_IRWXG
* stat.S_IRGRP
* stat.S_IWGRP
* stat.S_IXGRP
* stat.S_IRWXO
* stat.S_IROTH
* stat.S_IWOTH
* stat.S_IXOTH

此函数支持[指定文件描述符](#path-fd)、[目录描述符的相对路径](#dir-fd)、[不跟踪符号链接](#follow-symlinks)。

注意：尽管Windows也有支持`chmod()`函数，但只允许用户修改文件是否只读的属性（通过`stat.S_IWRITE`、`stat.S_IREAD`选项或选项的对应数值）。其他权限位会被忽略。

### os.chown(path, uid, gid, *, dir_fd=None, follow_symlinks=True)

将*path*的属主、属组ID分别置为*uid*和*gid*。如果其中的某个ID不需要修改，只需将该ID置为-1.

此函数支持[指定文件描述符](#path-fd)、[目录描述符的相对路径](#dir-fd)、[不跟踪符号链接](#follow-symlinks)。

参考高级函数`shutil.chown()`，除了接受ID，还接受属主/属组名称作为参数。

### os.chroot(path)

将当前进程的根目录置为*path*。

支持：Unix。

### os.fchdir(fd)

将当前工作目录置为文件描述符*fd*所指的目录。描述符必须指向一个打开的目录，而不是文件。在Python3.3中，相当于`os.chdir(fd)`。

支持：Unix。

### os.getcwd()

以字符串形式返回当前工作目录的路径。

支持：Unix，Windows。

### os.getcwdb()

以字节串形式返回当前工作目录的路径。

支持：Unix，Windows。

### os.lchflags(path, flags)

将*path*的标识置为*flags*，用法类似`chflags()`，只是不跟踪符号链接。在Python3.3中，相当于`os.chflags(path, flags, follow_symlinks=False)`。

支持：Unix。

### os.lchmod(path, mode)

将*path*的模式置为*mode*。如果路径所指为符号链接，则直接作用于符号链接，而不是其指向的目标。*mode*选项参见[`chmod()`](#os.chmod)。在Python3.3上，相当于`os.chmod(path, mode, follow_symlinks=False)`。

支持：Unix。

### os.lchown(path, uid, gid)

将*path*的属主、属组ID分别置为*uid*和*gid*。此函数不会跟踪符号链接。在Python3.3上，相当于`os.chown(path, uid, gid, follow_symlinks=False)`。

支持：Unix。

### os.link(src, dst, *, src_dir_fd=None, dst_dir_fd=None, follow_symlinks=True)

创建一个指向*src*，名为*dst*的硬链接。

此函数提供参数*src_dir_fd*和*dst_dir_fd*，以支持[目录描述符的相对路径](#dir-fd)和[不跟踪符号链接](#follow-symlinks)。

支持：Unix，Windows。

### [os.listdir(path='.')](id:os.listdir)

以列表形式返回*path*中所有对象的名字。名称没有排序，且不包含特殊路径`.`和`..`。

*path*可以是`str`或`bytes`。如果参数为*bytes*，则返回值列表中的元素也为`bytes`类型；如果参数为*str*，则返回值列表中的元素也为`str`类型。

此函数支持[指定文件描述符](#path-fd)，且文件描述符必须指向目录。

### os.lstat(path, *, dir_fd=None)

与系统的`lstat()`起同样的作用。用法类似[`stat()`](#os.stat)，但是不跟踪符号链接。在不支持符号链接的平台上，此函数等同于`stat()`。在Python3.3上，相当于`os.stat(path, dir_fd=dir_fd, follow_symlinks=False)`。

此函数支持[目录描述符的相对路径](#dir-fd)。

### os.mkdir(path, mode=0o777, *, dir_fd=None)

以*mode*模式创建*path*路径。

在一些系统上，*mode*会被忽略。当支持*mode*时，会先使用当前掩码遮罩输出。如果路径以及存在，则抛出`OSError`。

此函数支持[目录描述符的相对路径](#dir-fd)。

此函数也支持创建临时目录，见`tempfile`模块的`tempfile.mkdtemp()`函数。

支持：Unix，Windows。

### os.makedirs(name, mode=0o777, exist_ok=False)

此函数会递归的创建路径。类似`mkdir()`，在创建叶节点目录的过程中会自动创建所有叶节点所需的上级目录。

*mode*默认使用`0o777`（八进制）。在一些系统上，*mode*会被忽略。当支持*mode*时，会先使用当前掩码遮罩输出。

当*exist_ok*置为`False`（默认）时，如果目标目录存在则会抛出`OSError`。当*exist_ok*置为`True`时，如果已存在目标目录的*mode*与指定*mode*不一致时，也会抛出`OSError`。如果目录创建失败，同样会抛出`OSError`。

注意：如果目标目录的某级中出现`pardir`（如在Unix上为`..`），可能会产生与预期不相符的结果。

函数可以正确支持UNC路径。

### os.mkfifo(path, mode=0o666, *, dir_fd=None)

使用模式*mode*创建一个名为*path*的FIFO（命名管道），使用当前掩码遮罩。

函数同样支持[目录描述符的相对路径](#dir-fd)。

FIFO是一种可以被当做普通文件访问的管道。FIFO在删除（如使用[`os.unlink()`](#os.unlink)）前会一直存在。FIFO通常用来作“server”型进程与“client”型进程之间的数据交互：server进程打开FIFO读取数据，client打开FIFO写入数据。注意到`mkfifo()`并不负责打开FIFO，它只是创建而已。

支持：Unix。

### os.mknod(filename, mode=0o600, device=0, *, dir_fd=None)

创建一个名为*filename*的文件系统节点（如文件、设备或命名管道）。*mode*参数指定权限位和节点类型（`stat.S_IFREG`, `stat.S_IFCHR`, `stat.S_IFBLK`, `stat.S_IFIFO`的按位或组合，这些常量定义在`stat`模块中）。选择`stat.S_IFCHR`和`stat.S_IFBLK`时，使用参数*device*指定新建的设备文件（可能使用[`os.makedev()`](#os.makedev)函数）；否则会忽略该参数。

函数同样支持[目录描述符的相对路径](#dir-fd)。

### os.major(device)

从原始的设备号中抽取主要设备号（device major number，通常是`stat`模块中的`st_dev`和`st_rdev`）。

### os.minor(device)

从原始的设备号中抽取具体设备号（device minor number，通常是`stat`模块中的`st_dev`和`st_rdev`）。

### [os.makedev(major, minor)](id:os.makedev)

使用*major*和*minor*构成原始设备序号。

### os.pathconf(path, name)

返回与指定文件相关的系统配置信息。*name*指定返回的配置键，参数应为系统定义值，可能是字符串形式，这些系统定义值可以在某些标准（POSIX.1, Unix 95, Unix 98等）里找到。部分平台也支持额外的这类配置值。这些操作系统相关的配置值在`pathconf_names`目录中给出。如果有配置值并不在目录的映射中，则可以给传给*name*一个整型。

如果*name*是字符串，且未定义在系统中，则会抛出`ValueError`；如果*name*不被操作系统支持，即使定义在`pathconf_names`中，也会抛出`OSError`，并附带一个`errno.EINVAL`的错误代码。

此函数支持[指定文件描述符](#path-fd)。

支持：Unix。

### os.readlink(path, *, dir_fd=None)

以字符串形式返回符号链接指向的路径，返回结果可能是绝对路径或相对路径。如果是相对路径，则可以使用`os.path.join(os.path.dirname(path), result)`将其转为绝对路径。

如果*path*以字符串形式给出，则返回值也是字符串，如果中途出错则会抛出`UnicodeDecodeError`。如果*path*以字节形式给出，则返回值也是字节形式。

此函数支持[指定文件描述符](#path-fd)。

### [os.remove(path, *, dir_fd=None)](id:os.remove)

删除*path*。如果*path*是路径则抛出`OSError`。要删除目录请使用`rmdir()`。

此函数支持[目录描述符的相对路径](#dir-fd)。

在Windows上，删除正在使用的文件会抛出异常；在Unix上，目录入口会被删除，但分配给文件的存储空间是无效的，直到该文件不再使用。

此函数与[`unlink()`](#os.unlink))等价。

支持：Unix，Windows。

### os.removedirs(name)

递归的删除目录。类似[`rmdir()`](id:os.rmdir)，只是在叶目录删除成功时，函数会继续尝试删除*path*中出现的父路径直到抛出异常（会被忽略，因为这通常表示父路径不为空）。例如，` os.removedirs('foo/bar/baz')`会先尝试删除`'foo/bar/baz'`，接下来是`'foo/bar'`，最后是`'foo'`（如果这几个父目录为空的情况下才可以删除）。在不能成功删除叶节点时抛出`OSError`异常。

### [os.rename(src, dst, *, src_dir_fd=None, dst_dir_fd=None)](id:os.rename)

将文件或目录名*src*该为*dst*。如果*dst*是目录，则会抛出异常`OSError`。在Unix上，如果*dst*是一个存在的文件，在权限足够的情况下文件名会被静默的替换。在一些Unix系统上，如果*src*与*dst*在不同的文件系统上时，可能会导致操作失败。如果操作成功，重命名操作将会是一个原子操作（POSIX系统的必须条件）。在Windows上，如果*dst*已存在，即使是文件，也会抛出`OSError`。

此函数提供参数*src_dir_fd*和*dst_dir_fd*，以支持[目录描述符的相对路径](#dir-fd)。

如果需要跨系统重新目的路径，使用[`replace()`](#os.replace)。若在一个文件系统内，可以使用此函数来移动文件。

支持：Unix，Windows。

### [os.renames(old, new)](id:os.renames)

此函数用于递归的重命名。类似[`rename()`](#os.rename)，只是在新建目的路径的操作中会尝试创建所有需要的中间级目录。在重命名后，目录关联到的路径最右端的节点将使用[`removedirs()`](#os.removedirs)删除。

注意：如果没有删除叶节点的权限，会导致操作失败。

### [os.replace(src, dst, *, src_dir_fd=None, dst_dir_fd=None)](id:os.replace)

将*src*重命名为*dst*。如果*dst*是目录，则抛出`OSError`。如果*dst*是一个存在的文件，在权限足够的情况下文件名会被静默的替换。如果*src*与*dst*在不同的文件系统上时，可能会导致操作失败。如果操作成功，重命名操作将会是一个原子操作（POSIX系统的必须条件）。

此函数提供参数*src_dir_fd*和*dst_dir_fd*，以支持[目录描述符的相对路径](#dir-fd)。

支持：Unix，Windows。

### [os.rmdir(path, *, dir_fd=None)](id:os.rmdir)

删除目录*path*。只有在*path*为空时才可以成功，否则会抛出`OSError`。如果要删除整个目录树，可以使用`shutil.rmtree()`。

此函数支持[目录描述符的相对路径](#dir-fd)。

支持：Unix，Windows。

### [os.stat(path, *, dir_fd=None, follow_symlinks=True)](id:os.stat)

行为等同于在给定目录*path*上调用系统的`stat()`。*path*可以是字符串，也可以是文件描述符。（此函数默认跟踪符号链接，要获得符号链接的状态，可以`follow_symlinks=False`，也可以使用`lstat()`。）

返回值为一个大致与系统`stat()`返回值对应的对象：

* `st_mode` - 保护位；
* `inode` - 索引节；
* `st_dev` - 设备；
* `st_nlink` - 硬链接数；
* `st_uid` - 属主ID；
* `st_gid` - 属组ID；
* `st_size` - 文件总大小，以字节为单位；
* `st_atime` - 最近一次访问时间，以秒为单位；
* `st_mtime` - 最近一次内容修改，以秒为单位；
* `st_ctime` - 平台相关属性；在Unix上，为最近一次文件元数据修改，在Windows上，为文件创建时间；以秒为单位；
* `st_atime_ns` - 最近一次访问时间，以纳秒为单位；
* `st_mtime_ns` - 最近一次内容修改，以纳秒为单位；
* `st_ctime_ns` - 平台相关属性；在Unix上，为最近一次文件元数据修改，在Windows上，为文件创建时间；以纳秒为单位；

在某些Unix系统（如Linux）上，提供下列额外属性：

* `st_blocks` - 文件总块数，以512字节为一块；
* `st_blksize` - 高效文件系统I/O的文件系统块大小
* `st_rdev` - 索引节点设备的类型；
* `st_flags` - 文件的用户定义标识；

在另一些Unix系统（如FreeBSD）上，提供下列额外属性：

* `st_gen` - 文件生成号；
* `st_birthtime` - 文件生成时间；

在另一些Unix系统（如FreeBSD）上，提供下列额外属性：

* `st_rsize`
* `st_creator`
* `st_type`

注意：`st_atime`, `st_mtime`, `st_ctime`属性的时间精确度是跟操作系统、文件系统相关的。例如在Windows上的文件系统FAT和FAT32，`st_mtime`属性精确到2秒，而`st_atime`精确到1天，详情需要参考操作系统文档。类似的，尽管`st_atime_ns`, `st_mtime_ns`, `st_ctime_ns`的单位总是纳秒，但很多系统不一定支持到纳秒级的精确度。在那些支持纳秒级精确度的操作系统上，存放`st_atime`, `st_mtime`, `st_ctime `的浮点数存不下如此大的数字，于是也会有轻微的近似。如果需要用到很精确的时间戳，应该始终使用`st_atime_ns, st_mtime_ns`, `st_ctime_ns`。

为了保证向后兼容性，[`stat()`](#os.stat)总是返回由至少10个元素组成的元组，这10个元素是`stat`结构体最重要（也是通用的，可移植的）的10个成员，按顺序分别为`st_mode`, `st_ino`, `st_dev`, `st_nlink`, `st_uid`, `st_gid`, `st_size`, `st_atime`, `st_mtime`, `st_ctime`。额外的成员可能继续追加在元组的尾部。

此函数支持[指定文件描述符](#path-fd)、[不跟踪符号链接](#follow-symlinks)。

`stat`标准模块定义了用于从`stat`结构体中解析出可读信息的函数及常量。（在Windows上，一些成员会被预设为虚值(dummy value)）。

例如：

```
>>> import os
>>> statinfo = os.stat('somefile.txt')
>>> statinfo
posix.stat_result(st_mode=33188, st_ino=7876932, st_dev=234881026,
st_nlink=1, st_uid=501, st_gid=501, st_size=264, st_atime=1297230295,
st_mtime=1297230027, st_ctime=1297230027)
>>> statinfo.st_size
264
```

支持：Unix，Windows。

### [os.statvfs(path)](id:os.statvfs)

行为等同于在给定的目录*path*上调用系统的`statvfs()`。返回值是一个用来描述文件系统的`statvfs`结构体。其成员为别为：`f_bsize`, `f_frsize`, `f_blocks`, `f_bfree`, `f_bavail`, `f_files`, `f_ffree`, `f_favail`, `f_flag`, `f_namemax`。

`f_flag`属性位标识有两个模块级常量：如果设置`ST_RDONLY`，则文件系统挂载为只读；如果设置`ST_NOSUID`，setuid/setgid的语法会被禁用或不支持。

额外的模块级常数用于基于GNU/glibc的系统，分别为：

* `ST_NODEV` - 不允许访问设备类文件；
* `ST_NOEXEC` - 不允许执行程序；
* `ST_SYNCHRONOUS` - 写同步；
* `ST_MANDLOCK` - 强制锁FS；
* `ST_WRITE` - 写文件、目录、符号链接；
* `ST_APPEND` - 只允许追加文件；
* `ST_IMMUTABLE` - 不可变文件；
* `ST_NOATIME` - 不更新文件的最近访问时间；
* `ST_NODIRATIME` - 不更新路径的最近访问时间；
* `ST_RELATIME` - 将atime更新为mtime/ctime的相对值；

此函数支持[指定文件描述符](#path-fd)。

支持：Unix。

### [os.supports_dir_fd](id:os.supports_dir_fd)

一个`set`对象，包括了`os`模块中所有支持*dir_fd*参数有效的函数。不同的平台上支持此参数的函数有所不同，在一个平台上支持，在另一个平台上可能就不支持了。为了保证一致性，所有提供*dir_fd*参数的函数都允许指定该参数，但是如果功能上不支持，会抛出异常。

为了测试特定的函数是否允许使用*dir_fd*参数，可以在`supports_dir_fd`集合上使用`in`操作。下面测试了[`os.stat()`](#os.stat)函数的`dir_fd`参数是否在当前被平台下有效：

```
os.stat in os.supports_dir_fd
```

目前，*dir_fd*参数只在Unix上有效，在Windows上都无效。

### [os.supports_effective_ids](id:os.supports_effective_ids)

一个`set`对象，包括了`os`模块中所有支持*effective_ids*参数有效的函数。如果当前平台支持，则该集合中将会包括[`os.access()`](#os.access)，否则集合为空。

下面的例子测试了[`os.access()`](#os.access)函数的`effective_ids`参数是否在当前被平台下有效：

```
os.access in os.supports_effective_ids
```

目前，*effective_ids*参数只在Unix上有效，在Windows上都无效。

### [os.supports_fd](id:os.supports_fd)

一个`set`对象，包括了所有支持文件描述符形式的*path*参数的函数。不同的平台上支持此参数的函数有所不同，在一个平台上支持，在另一个平台上可能就不支持了。为了保证一致性，所有提供*fd*参数的函数都允许指定该参数，但是如果功能上不支持，会抛出异常。

为了测试特定的函数是否允许使用*supports_fd*参数，可以在`supports_fd`集合上使用`in`操作。下面测试了[`os.chdir()`](#os.chdir)函数是否接受文件描述符做参数：

```
os.chdir in os.supports_fd
```

### [os.supports_follow_symlinks](id:os.supports_follow_symlinks)

一个`set`对象，包括了`os`模块中所有支持*follow_symlinks*参数有效的函数。不同的平台上支持此参数的函数有所不同，在一个平台上支持，在另一个平台上可能就不支持了。为了保证一致性，所有提供*follow_symlinks*参数的函数都允许指定该参数，但是如果功能上不支持，会抛出异常。

为了测试特定的函数是否允许使用*supports_fd*参数，可以在`supports_follow_symlinks`集合上使用`in`操作。下面测试了[`os.stat()`](#os.stat)函数是否接受文件描述符做参数：

```
os.stat in os.supports_follow_symlinks
```

### [os.symlink(source, link_name, target_is_directory=False, *, dir_fd=None)](id:os.symlink)

新建一个名为*link_name*的符号链接，指向*source*。

在Windows上，符号链接既可以表示文件，也可以表示目录，该链接不会随着目标改变。如果目标存在，则创建链接时就会匹配其类型。另外，在*target_is_directory*为`True`时，会创建一个目录链接，为`False`则是一个文件链接（默认）。

Windows 6.0 (Vista)引进了符号链接，在以前的Windows版本中使用`symlink()`会抛出`NotImplementedError`。

此函数支持[目录描述符的相对路径](#dir-fd)。

注意：在Windows上，要成功创建一个符号链接则必须有*SeCreateSymbolicLinkPrivilege*权限。普通用户一般不具备此权限，只有那些能够升级为管理员的用户才可以使用此权限。所以，如果想要成功创建符号链接，要么得到该权限，要么使用管理员运行程序。

权限不够时调用此函数会抛出`OSError`。

支持：Unix，Windows。

### [os.sync()](id:os.sync)

强制将所有内容写入磁盘。

支持：Unix。

### [os.truncate(path, length)](id:os.truncate)

截断*path*指定的文件，使其最多只有*length*个字节大小。

此函数支持[指定文件描述符](#path-fd)。

支持：Unix。

### os.unlink(path, *, dir_fd=None)

删除*path*指定的文件。此函数与[`remove()`](#os.remove)等价。`unlink`这个名字来自Unix的习惯。详情参见[`remove()`](#os.remove)函数。

支持：Unix，Windows。

### [os.utime(path, times=None, *, ns=None, dir_fd=None, follow_symlinks=True)](id:os.utime)

设置*path*所指定文件的访问时间与修改时间。

`utime()`有两个可选参数*times*和*ns*，它们对*path*参数所指文件的作用如下：

* 如果*ns*不为`None`，则参数必须是一个有两个元素的元组，以`(atime_ns, mtime_ns)`形式给出，并且每个成员都必须是整型，单位为纳秒；
* 如果*times*不为`None`，则参数必须是一个有两个元素的元组，以`(atime, mtime)`形式给出，成员可以是整型也可以是浮点型，单位为纳秒；
* 如果*times*和*ns*都是`None`，相当于指定`ns=(atime_ns, mtime_ns)`，且两个参数都是当前时间；

不能同时为*times*和*ns*指定元组作为参数。

能否将目录传给*path*参数取决于操作系统是否支持将目录当做文件（Windows就不支持）。应该注意的是，设置的精确时间可能并不会被接下来调用的[`stat()`](#os.stat)函数返回，这取决于操作系统中访问时间与修改时间的精确度，详见[`stat()`](#os.stat)。保存精确时间的最好方法，是传给`utime()`的*ns*参数`os.stat()`返回对象的*st_atime_ns*和*st_mtime_ns*属性。

此函数支持[指定文件描述符](#path-fd)、[目录描述符的相对路径](#dir-fd)、[不跟踪符号链接](#follow-symlinks)。

支持：Unix，Windows。

### [os.walk(top, topdown=True, onerror=None, followlinks=False)](id:os.walk)

返回一个生成器，自上而下或自下而上的遍历指定目录。根节点下的每一个目录（包括根节点）都会返回一个型为`(dirpath, dirnames, filenames)`的元组。

*dirpath*是目录的路径，字符串形式；*dirnames*是*dirpath*下的子目录名称列表（不包括`.`和`..`）；*filenames*是非目录节点的名称列表，不包含完整路径，只有文件名。如果要得到文件的完整路径，使用`os.path.join(dirpath, name)`。

如果*topdown*为真或缺省，则根目录信息会在其子目录信息之前输出（自上而下的遍历）；如果*topdown*置为`False`，则根目录信息会在其所有子目录信息输出之后才输出（自下而上的遍历）。

如果*topdown*为真，函数会就地修改*dirnames*列表（可能是使用`del`或切片赋值），使得函数只需要继续遍历*dirnames*中剩下的子目录即可；此行为会简化遍历，指定遍历顺序，甚至可以在再次递归调用`walk()`之前通知函数“调用者新建或重命名了哪些目录”。将*topdown*置为`False`会降低效率，因为自下而上遍历时，*dirnames*中的目录名称会在*dirpath*之前给出。

默认的，此函数调用[`listdir()`](#os.listdir)时的异常会被忽略。如果要指定可选参数*onerror*，则必须指定一个函数，该函数必须接收一个`OSError`的实例作为唯一参数。通常该函数用以记录异常，使得遍历可以继续，或是直接抛出异常，以终止遍历。如果出现异常，异常的实例会具备`filename`属性，用以存放发生异常的文件名。

默认的，此函数不会跟踪符号链接去解析另一个目录。不过，在系统支持的情况下，将*followlinks*置为`True`会使得函数可以解析符号链接。

注意：*followlinks*为`True`时，如果符号链接指向此目录的某个父目录，则会导致无限递归，因为此函数不会跟踪已经访问过的目录。

注意：如果指定的是相对路径，则在下一次`walk()`调用前不要改变当前工作目录。因为`walk()`不会改变当前目录，并会假设调用者也不会改变此目录。

下面的例子可以统计每个目录下非目录文件的总大小，并且在遍历时跳过那些名为“CVS”的目录：

```
import os
from os.path import join, getsize
for root, dirs, files in os.walk('python/Lib/email'):
    print(root, "consumes", end=" ")
    print(sum(getsize(join(root, name)) for name in files), end=" ")
    print("bytes in", len(files), "non-directory files")
    if 'CVS' in dirs:
        dirs.remove('CVS')  # don't visit CVS directories
```

下面的例子中，自下而上的遍历成为了关键，因为`rmdir()`在目录为空前不能删除该目录：

```
# Delete everything reachable from the directory named in "top",
# assuming there are no symbolic links.
# CAUTION:  This is dangerous!  For example, if top == '/', it
# could delete all your disk files.
import os
for root, dirs, files in os.walk(top, topdown=False):
    for name in files:
        os.remove(os.path.join(root, name))
    for name in dirs:
        os.rmdir(os.path.join(root, name))
```

### [os.fwalk(top='.', topdown=True, onerror=None, *, follow_symlinks=False, dir_fd=None)](id:os.fwalk)

函数行为与[`walk()`](#os.walk)相同，只不过在便利时返回的元组是`(dirpath, dirnames, filenames, dirfd)`形式的，而且函数支持*dir_fd*参数。

*dirpath*, *dirnames*和*filenames*与[`walk()`](#os.walk)中的一致，而*dirfd*是与*dirpath*关联的文件描述符。

此函数同样支持[目录描述符的相对路径](#dir-fd)、[不跟踪符号链接](#follow-symlinks)。尽管如此，此函数*follow_symlinks*参数默认为`False`。

注意：此函数会返回文件描述符，而这些文件描述符只有在下一次迭代前可以访问，所以，如果不想文件描述符失效的话，可以使用[`dup()`](#os.dup())函数复制要用的文件描述符。

下面的例子可以统计每个目录下非目录文件的总大小，并且在遍历时跳过那些名为“CVS”的目录：

```
import os
for root, dirs, files, rootfd in os.fwalk('python/Lib/email'):
    print(root, "consumes", end="")
    print(sum([os.stat(name, dir_fd=rootfd).st_size for name in files]),
          end="")
    print("bytes in", len(files), "non-directory files")
    if 'CVS' in dirs:
        dirs.remove('CVS')  # don't visit CVS directories
```

下面的例子中，自下而上的遍历成为了关键，因为`rmdir()`在目录为空前不能删除该目录：

```
# Delete everything reachable from the directory named in "top",
# assuming there are no symbolic links.
# CAUTION:  This is dangerous!  For example, if top == '/', it
# could delete all your disk files.
import os
for root, dirs, files, rootfd in os.fwalk(top, topdown=False):
    for name in files:
        os.unlink(name, dir_fd=rootfd)
    for name in dirs:
        os.rmdir(name, dir_fd=rootfd)
```

支持：Unix。

### Linux额外特性

以下函数只在Linux中提供：

### [os.getxattr(path, attribute, *, follow_symlinks=True)](id:os.getxattr)

返回*path*的文件系统扩展属性*attribute*，*attribute*可以是`str`或`bytes`，如果是`str`，则使用文件系统编码。

此函数支持[指定文件描述符](#path-fd)、[不跟踪符号链接](#follow-symlinks)。

### [os.listxattr(path=None, *, follow_symlinks=True)](id:os.listxattr)

返回*path*的文件系统扩展属性列表。列表中的属性以字符串形式给出，使用文件系统编码。如果*path*为`None`，则默认为当前目录。

此函数支持[指定文件描述符](#path-fd)、[不跟踪符号链接](#follow-symlinks)。

### [os.removexattr(path, attribute, *, follow_symlinks=True)](id:os.removexattr)

删除*path*的文件系统扩展属性*attribute*，*attribute*可以是`str`或`bytes`，如果是`str`，则使用文件系统编码。

此函数支持[指定文件描述符](#path-fd)、[不跟踪符号链接](#follow-symlinks)。

### [os.setxattr(path, attribute, value, flags=0, *, follow_symlinks=True)](id:os.setxattr)

将*path*的文件系统扩展属性*attribute*置为*value*。*attribute*必须是一个不为NUL的`str`或`bytes`，如果是`str`，则使用文件系统编码。*flags*可以是`XATTR_REPLACE`或`XATTR_CREATE`。如果设置`XATTR_REPLACE`时，指定的属性不存在，则抛出`EEXISTS`。如果设置为`XATTR_CREATE`时，指定的属性已经存在，则该属性不会被重新创建，函数抛出`ENODATA`。

此函数支持[指定文件描述符](#path-fd)、[不跟踪符号链接](#follow-symlinks)。

注意：在内核版本低于2.6.39的某些系统中，可能会因为bug导致标识不起作用。

### os.XATTR_SIZE_MAX

扩展属性大小的上限，当前在Linux上为64 KiB。

### os.XATTR_CREATE

`setxattr()`的可用标识，告诉函数必须创建新属性。

### os.XATTR_REPLACE

`setxattr()`的可用标识，告诉函数必须替换已有属性。

## 进程管理

以下函数可能用于创建或管理进程。

[`exec*`](#os.execl)系列函数接受一个参数列表，并将其传入加载于当前进程的新程序。在每个函数中，参数列表中的第一个元素被当做该程序的名称传入，而后面的参数则被当做正常的、如同用户在命令行中输入的参数。对于C程序员，第一个元素相当于传给`main()`函数的`argv[0]`参数。示例：`os.execv('/bin/echo', ['foo', 'bar'])`只会输出`bar`，从输出来看`foo`是被忽略的。

### [os.abort()](id:os.abort)

给当前进程一个`SIGABRT`信号。在Unix上默认会产生一次内核dump；而在Windows上则会立刻终止进程并返回结束码`3`。请注意，这个函数不会调用Python信号管理函数`signal.signal()`来注册`SIGABRT`信号。

支持：Unix，Windows。

### [os.execl(path, arg0, arg1, ...)](id:osexecl)
### [os.execle(path, arg0, arg1, ..., env)](id:os.execle)
### [os.execlp(file, arg0, arg1, ...)](id:os.execlp)
### [os.execlpe(file, arg0, arg1, ..., env)](id:os.execlpe)
### [os.execv(path, args)](id:os.execv)
### [os.execve(path, args, env)](id:os.execve)
### [os.execvp(file, args)](id:os.execvp)
### [os.execvpe(file, args, env)](id:os.execvpe)

这些函数都会执行新的程序，并替换掉当前进程，且不返回。在Unix上，新的可执行对象会直接加载在当前进程中，并于其调用者的进程ID相同。中途的错误会以`OSError`形式抛出。

当前的进程会被立刻替换。打开的文件对象和文件描述符并的缓冲区数据并不会因此被自动写入磁盘，所以，如果这些对象的缓冲区有数据，应当使用`sys.stdout.flush()`或[`os.fsync()`](#os.fsync)将其写入磁盘后，再调用[`exec*`](#os.execl)函数。

“l”与“v”版本的[`exec*`](#os.execl)区别在于如何传递命令行参数。如果写代码时参数的个数是已知的，推荐使用“l”版本，因为这些独立的参数将以[`exec*`](#os.execl)的附加参数形式传入。当参数的个数可变时，推荐使用“v”版，此时的参数*args*将以列表或元组的形式传入。不论哪种形式，传入参数的第一个元素应当是该程序的名称，但这不是强制的。

“p”版的函数（`execlp()`, `execlpe()`, `execvp()`, `execvpe()`）会使用环境变量中的`PATH`来查找强音运行的程序*file*。当环境被替换时（使用[`exec*e`](#os.execle)函数，见下一段），新的环境会被当做`PATH`的来源。而`execl()`, `execle()`, `execv()`, `execve()`则不会使用`PATH`查找可执行程序；*path*必须是正确的绝对或相对路径。

“e”版的函数（execle(), execlpe(), execve(), execvpe()）中的*env*参数必须是一个映射，用于定义新进程运行时的环境变量（会代替当前进程的运行时环境）。由`execl()`, `execlp()`, `execv()`, `execvp()`函数唤起的新进程会继承当前进程的运行环境。

对于`execve()`，在部分平台上，*path*也可以指定为文件描述符，可以使用[`os.supports_fd`](#os.supports_fd)来检查此特性是否被当前平台支持。在不支持时强行使用文件描述符会抛出`NotImplementedError`。

支持：Unix，Windows。

### [os._exit(n)](id:os._exit)

使用结束码`n`退出进程。此函数会直接结束进程，而不进行诸如“将缓存数据写入磁盘”之类的收尾工作。

支持：Unix，Windows。

注意：通常使用`sys.exit(n)`结束进程，`_exit()`一般只用于结束[`fork()`](#os.fork)产生的子进程。

下面列出可以用在`_exit()`中的结束码，函数并不强制使用这些结束码。它们通常用于Python编写的系统程序，如邮件服务器的外部递送程序。

注意，这些结束码中的一部分可能不被Unix支持。

### [os.EX_OK](id:os.EX_OK)

表示正常退出的结束码。

支持：Unix。

### [os.EX_USAGE](id:os.EX_USAGE)

表示命令使用有误的结束码，比如给错了参数个数。

支持：Unix。

### [os.EX_DATAERR](id:os.EX_DATAERR)

表示输入数据有误的结束码。

支持：Unix。

### [os.EX_NOINPUT](id:os.EX_NOINPUT)

表示输入文件不存在或不可读的结束码。

支持：Unix。

### [os.EX_NOUSER](id:os.EX_NOUSER)

表示指定用户不存在的结束码。

支持：Unix。

### [os.EX_NOHOST](id:os.EX_NOHOST)

表示指定主机不存在的结束码。

支持：Unix。

### [os.EX_UNAVAILABLE](id:os.EX_UNAVAILABLE)

表示指定服务不存在的结束码。

支持：Unix。

### [os.EX_SOFTWARE](id:os.EX_SOFTWARE)

表示侦测到软件内部错误的结束码。

支持：Unix。

### [os.EX_OSERR](id:os.EX_OSERR)

表示侦测到操作系统错误的结束码，比如在不能fork进程或无法创建管道时。

支持：Unix。

### [os.EX_OSFILE](id:os.EX_OSFILE)

表示某些系统文件缺失、无法打开、或其他错误的结束码。

支持：Unix。

### [os.EX_CANTCREAT](id:os.EX_CANTCREAT)

表示用户指定的输出文件不能被创建的结束码。

支持：Unix。

### [os.EX_IOERR](id:os.EX_IOERR)

表示在I/O操作时发生错误的结束码。

支持：Unix。

### [os.EX_TEMPFAIL](id:os.EX_TEMPFAIL)

表示发生暂时性异常的结束码，这有时并不代表发生了错误，比如“无法建立网络连接等待重试”。

支持：Unix。

### [os.EX_PROTOCOL](id:os.EX_PROTOCOL)

表示协议交换时发生的不合法、无效、不识别等情况的结束码。

支持：Unix。

### [os.EX_NOPERM](id:os.EX_NOPERM)

表示执行操作（指的不是文件系统错误）时权限不够的结束码。

支持：Unix。

### [os.EX_CONFIG](id:os.EX_CONFIG)

表示发生了配置错误的结束码。

支持：Unix。

### [os.EX_NOTFOUND](id:os.EX_NOTFOUND)

表示发生了类似“某对象未找到”错误的结束码。

支持：Unix。

### [os.fork()](id:os.fork)

Fork一个子进程，给子进程返回`0`，给父进程返回子进程ID。如果中途发生错误则抛出`OSError`。

应该留意，包括FreeBSD <= 6.3和Cygwin在内的部分平台在fork线程时存在问题。

警告：对于使用SSL模块fork进程的程序，请先阅读`ssl`模块的文档。

### [os.forkpty()](id:os.forkpty)

Fork一个子进程，新建一个伪终端作为子进程的控制终端。返回一对`(pid, fd)`，其中的*pid*在子进程中返回`0`，在父进程中返回子进程ID；而*fd*是伪终端的文件描述符。如果需要更强的可移植性，请使用`pty`模块。如果中途发生错误则抛出`OSError`。

支持：多数Unix。

### [os.kill(pid, sig)](id:os.kill)

将信号*sig*送往进程*pid*。主机平台支持的信号常量定义在[`signal`](https://docs.python.org/3/library/signal.html#module-signal)模块中。

对于Windows：[`signal.CTRL_C_EVENT`](https://docs.python.org/3/library/signal.html#signal.CTRL_C_EVENT)和[`signal.CTRL_BREAK_EVENT`](https://docs.python.org/3/library/signal.html#signal.CTRL_BREAK_EVENT)信号只能送往共享终端窗口的命令提示符进程，比如一些子进程。如果给*sig*赋予其他任何值，都会使得进程被TerminateProcess API无条件终止，而结束码将为*sig*。Windows版的`kill()`会额外终止进程句柄。

参见：[signal.pthread_kill()](https://docs.python.org/3/library/signal.html#signal.pthread_kill)

### [os.killpg(pgid, sig)](id:os.killpg)

将信号*sig*送往进程组*pgid*。

支持：Unix。

### [os.nice(increment)](id:os.nice)

给进程增加大小为*increment*的nice值（优先级值NI），返回新的nice值。

支持：Unix。

### [os.plock(op)](id:os.plock)

将程序段锁入内存，*op*值（定义在`<sys/lock.h>`中）用于指定锁定的片段。

支持：Unix。

### [os.popen(...)](id:os.popen)

新起子进程，返回用以通信的打开的管道。此类函数的详细描述在[创建文件对象](#file-object-creation)一节。

### [os.spawnl(mode, path, ...)](id:os.spawnl)
### [os.spawnle(mode, path, ..., env)](id:os.spawnle)
### [os.spawnlp(mode, file, ...)](id:os.spawnlp)
### [os.spawnlpe(mode, file, ..., env)](id:os.spawnlpe)
### [os.spawnv(mode, path, args)](id:os.spawnv)
### [os.spawnve(mode, path, args, env)](id:os.spawnve)
### [os.spawnvp(mode, file, args)](id:os.spawnvp)
### [os.spawnvpe(mode, file, args, env)](id:os.spawnvpe)

在新进程中执行*path*指定的程序。

注：[`subprocess`](https://docs.python.org/3/library/subprocess.html#module-subprocess)模块提供了功能更加强大的用于新建子进程并取回其执行结果的工具。所以更推荐使用`subprocess`模块，留意[使用subprocess模块代替旧版函数](https://docs.python.org/3/library/subprocess.html#subprocess-replacements)一节。

如果*mode*设为[`P_NOWAIT`](#os.P_NOWAIT)，则此函数返回新进程的ID；如果*mode*设为[`P_WAIT`](#os.P_WAIT)，在进程正常结束时返回其结束码，在指定终止信号*signal*时返回`-signal`。在Windows上，进程ID就是进程句柄，所以可以用于[`waitpid()`](#os.waitpid)函数。

“l”和“v”版本的[`spawn*`](#os.spawnl)函数的区别在于如何传递命令行参数。如果写代码时参数的个数是已知的，推荐使用“l”版本，因为这些独立的参数将以[`exec*`](#os.execl)的附加参数形式传入。当参数的个数可变时，推荐使用“v”版，此时的参数*args*将以列表或元组的形式传入。不论哪种形式，传入参数的第一个元素应当是该程序的名称。

“p”版的函数（`spawnlp()`, `spawnlpe()`, `spawnvp()`, `spawnvpe()`）会使用环境变量中的`PATH`来查找强音运行的程序*file*。当环境被替换时（使用[`spawn*e`](#os.spawnle)函数，见下一段），新的环境会被当做`PATH`的来源。而`spawnl()`, `spawnle()`, `spawnv()`, `spawnve()`则不会使用`PATH`查找可执行程序；*path*必须是正确的绝对或相对路径。

“e”版的函数（`spawnle()`, `spawnlpe()`, `spawnve()`, `spawnvpe()`）中的*env*参数必须是一个映射，用于定义新进程运行时的环境变量（会代替当前进程的运行时环境）。由`spawnl()`, `spawnlp()`, `spawnv()`, `spawnvp()`函数唤起的新进程会继承当前进程的运行环境。值得注意的是，*env*中的键-值对均应为字符串，无效的键或值会导致函数执行失败，而后返回结束码`127`。

等价使用`spawnlp()`和`spawnvpe()`的示例：

```
import os
os.spawnlp(os.P_WAIT, 'cp', 'cp', 'index.html', '/dev/null')

L = ['cp', 'index.html', '/dev/null']
os.spawnvpe(os.P_WAIT, 'cp', L, os.environ)
```

支持：Unix，Windows。`spawnlp()`, `spawnlpe()`, `spawnvp()`, `spawnvpe()`不在Windows上支持；`spawnle()`, `spawnve()`在Windows上不是线程安全的；推荐使用`subprocess`模块。

* os.P_NOWAIT
* os.P_NOWAITO

[`spawn*`](#os.spawnl)函数中*mode*参数的可用选项。如果指定其中的任意一个选项，`spawn*()`函数将在新建进程后立刻返回新进程的ID。

支持：Unix，Windows。

* os.P_WAIT

[`spawn*`](#os.spawnl)函数中*mode*参数的可用选项。如果*mode*设置为此选项，则函数会等到新进程执行完毕后，在进程正常结束时返回新进程的结束码，或在进程被信号*signal*终止时返回`-signal`值。

支持：Unix，Windows。

* os.P_DETACH
* os.P_OVERLAY

[`spawn*`](#os.spawnl)函数中*mode*参数的可用选项。这两个选项的可移植性比上面已经列出的几个选项略低。`P_DETACH`类似于`P_NOWAIT`，但是新进程会从调用进程的控制台中分离。如果使用`P_OVERLAY`，则当前进程会被新建进程替换，函数不返回。

支持：Windows。

### [os.startfile(path[, operation])](id:os.startfile)

使用关联的程序打开文件。

如果*operation*未指定或指定为`'open'`，则此函数行为类似于在Windows资源管理器中双击该文件，或在命令行中将文件名作为参数传给**start**命令：使用文件扩展名关联的应用程序（如果指定的话）打开。

当*operation*设置为其他值时，该值必须是一个“命令动词”，用以指定该对文件做何种操作。微软给出的常见动词有`'print'`、`'edit'`（用于文件）、`explore`、`find`（用于目录）。

`startfile()`在关联的应用程序启动后立刻返回。此函数没有等待应用程序执行结束再返回的选项，而且无法取回应用程序结束状态码。*path*参数为当前的相对路径。如果想要使用绝对路径，请确保路径的第一个字符不是斜线（`'/'`）；若指定为基本的Win32函数`ShellExecute()`则不会执行。可以使用[`os.path.normpath()`](https://docs.python.org/3/library/os.path.html#os.path.normpath)确保路径被编码为正确的Win32格式。

支持：Windows。

### [os.system(command)](id:os.system)

在子shell中执行命令（字符串形式给出）。此函数通过调用标准C函数`system()`实现，从而也有同样的限制。例如`sys.stdin`中的变动不会反映在已经执行完毕的命令的环境中。如果*command*有输出，则会送往解释器的标准输出中。

在Unix上，函数返回值是`wait()`函数使用的进程结束码。注意到POSIX并不向C函数`system()`的返回值指定含义，所以此Python函数的返回值是平台相关的。

在Windows上，执行*command*后，返回值通过系统命令行返回。系统命令行由Windows环境变量`COMSPEC`给出：通常为**cmd.exe**，由**cmd**返回命令执行完毕的结束码；对于使用非原生命令行的系统，请查看该命令行程序的帮助文档。

[`subprocess`](https://docs.python.org/3/library/subprocess.html#module-subprocess)模块提供了功能更加强大的用于新建子进程并取回其执行结果的工具。所以更推荐使用`subprocess`模块，留意[使用subprocess模块代替旧版函数](https://docs.python.org/3/library/subprocess.html#subprocess-replacements)一节的帮助文档。

支持：Unix，Windows。

### [os.times()](id:os.times)

返回当前进程全局时间。返回值有以下五个属性：

* user - 用户时间；
* system - 系统时间；
* children_user - 所有子进程的用户时间；
* children_system - 所有子进程的系统时间；
* elapsed - 从过去一个时间节点开始算起，逝去的总时间；

为了向后兼容性，该结果对象行为类似一个拥有5个元素的元组，顺序为`user`, `system`, `children_user`, `children_system`, `elapsed`。

详情参见Unix上*time(2)*的用户手册，或Windows上的相应API文档。在Windows上，只有`user`和`system`可知，其他的属性皆为0。

支持：Unix，Windows。

### [os.wait()](id:os.wait)

等待子进程结束，而后返回一个包括了进程pid和进程结束状态码的元组：一个16位的数字，其低8位为终止该进程的信号值*signal*，其高8位为进程结束状态码（信号值为0时），如果产生了内核dump则低8位的最高位会被设置。

支持：Unix。

### [os.waitid(idtype, id, options)](id:os.waitid)

等待一个或多个子进程结束。参数*idtype*可以是`P_PID`, `P_PGID`, `P_ALL`。*id*为等待的进程pid。参数*options*为`WEXITED`, `WSTOPPED`, `WCONTINUED`选项的组合（或操作），也可能使用额外的选项`WNOHANG`, `WNOWAIT`。返回值为`siginfo_t`结构体对象，包括`si_pid`, `si_uid`, `si_signo`, `si_status`, `si_code`；返回值也可能为`None`，如果指定了`WNOHANG`选项且没有子进程处于等待状态。

支持：Unix。

* os.P_PID
* os.P_PGID
* os.P_ALL

以上为`waitid()`函数的*idtype*参数选项。此参数影响到*id*如何被解析。

支持：Unix。

* os.WEXITED
* os.WSTOPPED
* os.WNOWAIT

以上为`waitid()`函数的*option*参数选项。此参数指定等待信号的类型。

支持：Unix。

* os.CLD_EXITED
* os.CLD_DUMPED
* os.CLD_TRAPPED
* os.CLD_CONTINUED

以上为`waitid()`函数返回值中`si_code`属性可能使用的值。

支持：Unix。

### [os.waitpid(pid, options)](id:os.waitpid)

此函数的行为在Windows上与在Unix上略有不同。

在Unix上，等待给定子进程进程*pid*结束，返回一个包括子进程ID和结束状态码（由`wait()`函数定义）的元组。指定*options*参数会改变函数行为，该值通常为`0`。

如果*pid*不为`0`，则`waitpid()`请求给定进程的状态。如果*pid*为`0`，则请求当前进程所在进程组下任意子进程状态。如果*pid*为`-1`，则请求当前进程的任意子进程状态。如果*pid*小于`-1`，则请求进程组`-pid`（即*pid*的绝对值）中任意进程的状态。

当系统调用返回-1时，抛出`OSError`，并附带错误码。

在Windows上，等待进程句柄*pid*指定进程结束，返回一个包括*pid*和结束状态码（较原来的状态码左移8位，为了使得函数跨平台使用更加方便）。在Windows上指定小于等于0的*pid*值没有特殊意义，而且会使函数抛出异常；*options*参数不起作用；*pid*可以指向任意已知ID的进程，并不要求必须为子进程。使用带有`P_NOWAIT`选项的[`spawn*`](#os.spawnl)函数即可返回恰当的进程句柄。

### [os.wait3(options)](id:os.wait3)

与[`waitpid()`](#os.waitpid)类似，只不过不需要进程ID这个参数，且返回值变为包含了子进程ID、结束状态码以及资源利用信息的元组。资源利用信息由[`resource`](https://docs.python.org/3/library/resource.html#module-resource).[`getrusage()`](https://docs.python.org/3/library/resource.html#resource.getrusage)给出，*options*选项与[`waitpid()`](#os.waitpid)、[`wait4()`](#os.wait4)相同。

支持：Unix。

### [os.wait4(options)](id:os.wait4)

与[`waitpid()`](#os.waitpid)类似，只不过返回值变为包含了子进程ID、结束状态码以及资源利用信息的元组。资源利用信息由[`resource`](https://docs.python.org/3/library/resource.html#module-resource).[`getrusage()`](https://docs.python.org/3/library/resource.html#resource.getrusage)给出。此函数参数与[`waitpid()`](#os.waitpid)完全相同。

支持：Unix。

* os.WNOHANG

`waitpid()`的*options*参数选项，如果子进程没有提供状态信息则立刻返回`(0, 0)`。

支持：Unix。

* os.WCONTINUED

`waitpid()`的*options*参数选项，使得子进程自上一次状态返回后从一个业务控制中断后恢复时，返回状态信息。

支持：Unix。

* os.WUNTRACED

`waitpid()`的*options*参数选项，如果子进程被中止，而中止时的状态没有被返回，则返回该状态。

支持：Unix。

下面的函数需要使用`system()`, `wait()`, `waitpid()`函数返回的进程状态码作为参数。可能用于确定进程的配置。

### [os.WCOREDUMP(status)](id:os.WCOREDUMP)

如果进程产生了内核dump，则返回`True`，否则返回`False`

支持：Unix。

### [os.WIFCONTINUED(status)](id:os.WIFCONTINUED)

如果进程从一个业务控制中断中恢复，则返回`True`，否则返回`False`

支持：Unix。

### [os.WIFSTOPPED(status)](id:os.WIFSTOPPED)

如果进程中止，则返回`True`，否则返回`False`

支持：Unix。

### [os.WIFSIGNALED(status)](id:os.WIFSIGNALED)

如果进程因为一个信号而终止，则返回`True`，否则返回`False`

支持：Unix。

### [os.WIFEXITED(status)](id:os.WIFEXITED)

如果进程因为系统调用*exit(2)*而终止，则返回`True`，否则返回`False`

支持：Unix。

### [os.WEXITSTATUS(status)](id:os.WEXITSTATUS)

如果`WIFEXITED(status)`为真，则返回系统调用*exit(2)*的整形参数值，其他情况下的返回值均无意义。

支持：Unix。

### [os.WSTOPSIG(status)](id:os.WSTOPSIG)

返回导致进程中止的信号。

支持：Unix。

### [os.WTERMSIG(status)](id:os.WTERMSIG)

返回导致进程退出的信号。

支持：Unix。

## 进程调度接口

以下函数可以控制操作为进程系统调度CPU时间的行为。只在部分Unix系统上有效，更多详情请查看Unix用户手册（`man`）。

如果系统支持，则在模块中可以找到下列调度策略相关选项：

### [os.SCHED_OTHER](id:os.SCHED_OTHER)

默认调度策略。

### [os.SCHED_BATCH](id:os.SCHED_BATCH)

对于密集调度CPU进程的调度策略，剩余计算能力用以保证交互性。

### [os.SCHED_IDLE](id:os.SCHED_IDLE)

低优先级后台进程的调度策略。

### [os.SCHED_SPORADIC](id:os.SCHED_SPORADIC)

服务进程的调度策略。

### [os.SCHED_FIFO](id:os.SCHED_FIFO)

先进先出调度策略。

### [os.SCHED_RR](id:os.SCHED_RR)

轮询调度策略。

### [os.SCHED_RESET_ON_FORK](id:os.SCHED_RESET_ON_FORK)

此选项可以与其他选项做或操作（`|`）共同使用。当带有此标识的进程fork子进程时，子进程的子进程的调度策略及进程优先级会被重置（不继承）。

### [class os.sched_param(sched_priority)](id:os.sched_param)

该类定义了`sched_setparam()`, `sched_setscheduler()`, `sched_getparam()`函数中使用的可调谐调度参数。该参数是可变的。

目前只有一个可选项：

#### [sched_priority](id:os.sched_param.sched_priority)

调度策略的优先级。

### [os.sched_get_priority_min(policy)](id:os.sched_get_priority_min)

返回策略*policy*的最小优先级。*policy*参数为上述调度策略常量中的一种。

### [os.sched_get_priority_max(policy)](id:os.sched_get_priority_max)

返回策略*policy*的最小优先级。*policy*参数为上述调度策略常量中的一种。

### [os.sched_setscheduler(pid, policy, param)](id:os.sched_setscheduler)

为进程*pid*设置调度策略。*pid*为`0`时表示调用者所在进程。*policy*参数为上述调度策略常量中的一种。*param*参数是一个[`sched_param`](#os.sched_param)实例。

### [os.sched_getscheduler(pid)](id:os.sched_getscheduler)

返回进程*pid*的调度策略。*pid*为`0`时表示调用者所在进程。*param*参数是一个[`sched_param`](#os.sched_param)实例。

### [os.sched_setparam(pid, param)](id:os.sched_setparam)

为进程*pid*设置调度参数。*pid*为`0`时表示调用者所在进程。*param*参数是一个[`sched_param`](#os.sched_param)实例。

### [os.sched_getparam(pid)](id:os.sched_getparam)

以[`sched_param`](#os.sched_param)实例的形式返回*pid*的调度参数。*pid*为`0`时表示调用者所在进程。

### [os.sched_rr_get_interval(pid)](id:os.sched_rr_get_interval)

返回进程*pid*美妙的轮询量。*pid*为`0`时表示调用者所在进程。

### [os.sched_yield()](id:os.sched_yield)

主动放弃CPU占用。

### [os.sched_setaffinity(pid, mask)](id:os.sched_setaffinity)

将进程*pid*的调用限制在指定核心上，*pid*为`0`时表示调用者所在进程。*mask*是一个整型容器的迭代器，用以表示将进程限制在哪些核心上。

### [os.sched_getaffinity(pid)](id:os.sched_getaffinity)

查看进程*pid*被限制在了哪些核心上。

## [其他系统信息](id:miscellaneous-system-information)

### [os.confstr(name)](id:os.confstr)

以字符串形式返回系统设置值。*name*指定返回的设置值的名称，它可能是以字符串形式给出的系统设置值的名称，这些名称定义在一些标准中（POSIX, Unix 95, Unix 98, and others）。一些平台也可能定义额外的名称。主机平台支持的所有名称都会定义在`confstr_names`字典中，如果要找的设置不再该字典中，则也可以给*name*传入一个整型。

如果*name*指定的系统设置不存在，则返回`None`。

如果*name*是字符串，且该值未定义，则抛出`ValueError`。如果系统不支持*name*指定的设置值，即使该值的名称定义在`confstr_names`中，也会抛出`OSError`，并给出[`errno.EINVAL`](https://docs.python.org/3/library/errno.html#errno.EINVAL)作为错误代码。

支持：Unix。

### [os.confstr_names](id:os.confstr_names)

一个字典，存储了从`confstr()`函数使用的设置名称，到一个系统支持的代表该设置的整型值的映射。可以用于定义系统支持的设置名称的集合。

支持：Unix。

### [os.cpu_count()](id:os.cpu_count)

返回系统中的CPU个数。

### [os.getloadavg()](id:os.getloadavg)

返回系统队列中最近1、5、15分钟内的平均进程数，如果无法获得该平均数，则抛出`OSError`。

支持：Unix。

### [os.sysconf(name)](id:os.sysconf)

返回系统配置值所对应的整数。如果*name*指定的名称未定义，则返回`-1`。用于`confstr()`函数的*name*参数的值同样适用于此函数的*name*参数。已知名称的映射定义在`sysconf_names`字典中。

支持：Unix。

### [os.sysconf_names](id:os.sysconf_names)

一个字典，存储了从`sysconf()`函数使用的设置名称，到一个系统支持的代表该设置的整型值的映射。可以用于定义系统支持的设置名称的集合。

支持：Unix。

下面的模块级变量用于路径操作，支持所有平台。

注：高级路径操作函数定义在[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块中。

### [os.curdir](id:os.curdir)

操作系统用来表示当前目录的字符串常量。在Windows和POSIX上就是`'.'`。[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块同样提供此常量。

### [os.pardir](id:os.pardir)

操作系统用来表示上级目录的字符串常量。在Windows和POSIX上就是`'..'`。[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块同样提供此常量。

### [os.sep](id:os.sep)

操作系统用来分隔路径名各部分的字符串常量。在POSIX上是`'/'`，在Windows上是`'\\'`。仅知道这个变量并不能对路径进行方便的解析或合并，推荐使用[`os.path.split()`](https://docs.python.org/3/library/os.path.html#os.path.split)和[`os.path.join()`](https://docs.python.org/3/library/os.path.html#os.path.join)函数。[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块同样提供此常量。

### [os.altsep](id:os.altsep)

操作系统用来分隔路径各部分的另一种可选的字符串常量，在只提供一种分隔符的平台上返回`None`。在Windows上是`'/'`。[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块同样提供此常量。

### [os.extsep](id:os.extsep)

用于分隔文件名与扩展名的字符串常量，如对于名为`os.py`文件的文件，分隔符就是`'.'`。[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块同样提供此常量。

### [os.pathsep](id:os.pathsep)

操作系统用来分隔各搜索路径（如在`PATH`中定义的路径）的字符串常量，比如在POSIX上是`':'`，在Windows上是`';'`。[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块同样提供此常量。

### [os.defpath](id:os.defpath)

如果系统不提供`PATH`环境变量，则此常量就是[`exec*p*`](#os.execl)和[`spawn*p*`](#os.spawnl)函数的默认搜索路径。

### [os.linesep](id:os.linesep)

操作系统的行分隔符（或结束符），这可能是一个字符，如在POSIX上是`'\n'`；也可能是多个字符，如在Windows上是`'\r\n'`。在文本模式下写文件（`mode='wt'`）时请勿使用*os.linespe*作为行结束符，在所有平台上都应该使用`'\n'`。

### [os.devnull](id:os.devnull)

系统中空设备的路径，如在POSIX上是`'/dev/null'`，在Windows上是`'nul'`。[`os.path`](https://docs.python.org/3/library/os.path.html#module-os.path)模块同样提供此常量。

* os.RTLD_LAZY
* os.RTLD_NOW
* os.RTLD_GLOBAL
* os.RTLD_LOCAL
* os.RTLD_NODELETE
* os.RTLD_NOLOAD
* os.RTLD_DEEPBIND

`setdlopenflags()`和`getdlopenflags()`函数的标识选项。这些标识的解释参见Unix上*dlopen(3)*函数的用户手册（`man`）。

## 其它函数

### [os.urandom(n)](id:os.urandom)

返回*n*个随机生成的字节，常用于加密。

函数返回的随机字节由平台指定的随机源给出。虽然返回的值应该“足够随机”（足够不可预测），可以用于加密程序，但此函数的随机程度依赖于平台实现。在类Unix平台上，会查询`/dev/urandom`；而在Windows上，则会调用`CryptGenRandom()`。如果函数找不到平台的随机源，则抛出`NotImplementedError`。

要在当前平台上使用更加易用的随机函数，可以使用[`random.SystemRandom`](https://docs.python.org/3/library/random.html#random.SystemRandom)。