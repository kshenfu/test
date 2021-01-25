[TOC]

# 1. meson教程



官网：https://mesonbuild.com/

源码安装的话，是执行 setup.py 。

可以使用 Pip3 进行安装：

pip3 install meson

meson 又依赖 ninja ，

pip3 install ninja

要求的最低版本是 ninja 1.7+ 和 python3.5+ 。

在 centos7 上都是满足的。

注意：通过 pip3 安装的程序的可执行文件会放在 python 的bin目录下，我这里 python3是安装在 /usr/local/python3/ ，所以meson 可执行文件就在 ：

/usr/local/python3/bin/meson

将这个路径添加到 ~/.bashrc 下即可。



## 1.1 一个简单的例子

在hello目录下创建一个main.c 文件如下：

```c
#include<stdio.h>

int main(int argc, char **argv) {
  printf("Hello there.\n");
  return 0;
}

```

然后在同一目录创建一个 meson 的编译描述文件，名为 meson.build。

内容如下：

```meson
project('tutorial', 'c')
executable('demo', 'main.c')
```

这就是所有了，它不像其他的编译工具，需要将依赖的头文件添加到 source 文件列表中。

project() 第一个参数是项目名，第二个参数是使用的语言。注意：对于 C++ 项目，第二个参数应该是 cpp 。

这样就可以进行编译了。

首先我们需要进入源码目录，执行如下命令来初始化这个构建（hello目录下）。

```
meson builddir
```

这条命令执行完成后，会在 hello 目录下生成一个 builddir 目录，然后进入这个builddir 目录，执行命令：

```
ninja
```

进行编译。如果版本在 0.55 以上，可以通过命令：

```
meson compile
```

编译。编译完成后，会在这个编译目录下生成可执行文件。

```
meson install -C builddir
```

默认会安装到 /usr/local。



## 1.2 meson.build 语法





### 1.2.1 编译选项

全局的编译选项：

```
[built-in options]
c_std = 'c99'
```

对于子项目的编译选项：

```
[zlib:built-in options]
default_library = 'static'
werror = false
```



### 1.2.2 生成源文件

这也是它的比较大的优势之一，可以支持python 来生成源文件。



### 1.2.3 依赖

```
zdep = dependency('zlib', version : '>=1.2.8')
exe = executable('zlibprog', 'prog.c', dependencies : zdep)
```

version 用于指定版本。version这是一个可选的关键字。

如果有多个依赖，可以像如下这样：

```
executable('manydeps', 'file.c', dependencies : [dep1, dep2, dep3, dep4])
```

如果这是一个可选的依赖，

```
opt_dep = dependency('somedep', required : false)
if opt_dep.found()
  # Do something.
else
  # Do something else.
endif
```



### 1.2.4 meson sample

所有的 meson.build 都是以 project() 命令为开始，它指定项目的名称以及它使用的语言。比如：

```
project('simple', 'c')
executable('myexe', 'source.c')
```

下一行指定了编译的目标，目标名称为 myexe，源文件为 source.c ，注意：这里必须都使用单引号。当然也支持使用变量来定义源文件。比如：

```
project('simple', 'c')
src = 'source.c'
executable('myexe', src)
```

有些目标需要多个源文件编译而成，可以用如下方式：

```
project('simple', 'c')
src = ['source1.c', 'source2.c', 'source3.c']
executable('myexe', src)
```

上面的代码也可以写成如下这样：

```
project('simple', 'c')
src = ['source1.c', 'source2.c', 'source3.c']
executable('myexe', sources : src)
```



executable 命令实际上返回一个 executable 对象，他可以传递给其他的函数，就像这样：

```
project('simple', 'c')
src = ['source1.c', 'source2.c', 'source3.c']
exe = executable('myexe', src)
test('simple test', exe)
```

这里有一个单元测试叫做 'simple test'，并且使用这个编译生成的目标。当这个测试被 meson test 命令运行起来，这个编译的可执行程序myexe 会运行。如果返回0，这个测试通过，如果返回非0值表示出错了，meson 将会报告错误给用户。



### 1.2.5 语法

语言的主要构造块是变量、数字、布尔值、字符串、数组、函数调用、方法调用、if语句和include。

通常一个meson语句只需要一行。不可能像C.那样在一行中包含多个语句。函数和方法调用的参数列表可以拆分为多行。meson会自动探测到这种情况并做正确的事情。

另外，在 0.50 之后的版本，你可以在多行的某尾添加 '\' 作为连接符。



**变量**

变量不需要预定义，使用的时候直接进行赋值即可。一个变量可以包含任意类型的值。

```
var1 = 'hello'
var2 = 102
```



注意，meson 中的类型都是值类型，不会有引用类型。比如：

```
var1 = [1, 2, 3]
var2 = var1
var2 += [4]
# var2 is now [1, 2, 3, 4]
# var1 is still [1, 2, 3]
```



**数字**

meson 只支持整型的数字，基础的数学操作也是支持的。

```
x = 1 + 2
y = 3 * 4
d = 5 % 3 # Yields 2.
```



在 0.45 版本之后，支持了16进制。0.47版本之后支持了 8进制和二进制。字符串和数字可以相互转化。



**字符串**

字符串在meson 中通过 单引号定义。字符串中要包含单引号的话，如下：

```
single quote = 'contains a \' character'
```

字符串的拼接是通过 '+' 来实现：

```
str1 = 'abc'
str2 = 'xyz'
combined = str1 + '_' + str2 # combined is now abc_xyz
```

**字符串路径**

你可以使用 '/' 拼接2个字符串生成一个 文件路径。在任何平台都是使用 '/' 来拼接。

**字符串包含多行**

多行字符串，像python 一样，使用 '''  来包含，就可以了。

```
multiline_string = '''#include <foo.h>
int main (int argc, char ** argv) {
  return FOO_SUCCESS;
}'''
```

这是原始字符串，字符串里面的转义也不会生效。



**字符串格式化**

格式化的方式如下，通过一个 format() 函数来实现：

```
template = 'string: @0@, number: @1@, bool: @2@'
res = template.format('text', 1, true)
# res now has value 'string: text, number: 1, bool: true'
```

如你所见，@number@ 的数字就被替换为format 的第number个参数。



meson 字符串支持如下的方法：

**.strip()**、**.to_upper(), .to_lower()** 、**.to_int()**、**.contains(), .startswith(), .endswith()**、**.substring()**  等等







 **Arrays**

数组是通过中括号包起来的。数组可以包含任意数量的任何类型的对象。

```
my_array = [1, 2, 'string', some_obj]
```

可以通过下标访问数组的任意元素：

```
my_array = [1, 2, 'string', some_obj]
second_element = my_array[1]
last_element = my_array[-1]
```

记住所有对象在 meson 中是不变的，所以追加内容到数组中总是会创建一个新的数组，而不是去更新老的数组。



**字典**

字典用大括号分隔。字典包括任意个键值对。键要求是 string，但是value 可以是任意类型。

key 必须是唯一的。

字典是在 0.47 版本之后支持。

关于字典支持的方法可以参考： https://mesonbuild.com/Reference-manual.html#dictionary-object



**函数**

meson 提供了一系列可用的函数。比如最常用的：

```
executable('progname', 'prog.c')
```



**条件判断**



if 使用 跟其他的语言一样。

```
var1 = 1
var2 = 2
if var1 == var2 # Evaluates to false
  something_broke()
elif var3 == var2
  something_else_broke()
else
  everything_ok()
endif

opt = get_option('someoption')
if opt != 'foo'
  do_something()
endif
```



**逻辑操作**

与 and，或 or，非 not

```
if a and b
  # do something
endif
if c or d
  # do something
endif
if not e
  # do something
endif
if not (f or g)
  # do something
endif
```



**foreach 语法**

foreach 遍历数组:

```
progs = [['prog1', ['prog1.c', 'foo.c']],
         ['prog2', ['prog2.c', 'bar.c']]]

foreach p : progs
  exe = executable(p[0], p[1])
  test(p[0], exe)
endforeach

```

foreach 遍历字典：

```
components = {
  'foo': ['foo.c'],
  'bar': ['bar.c'],
  'baz': ['baz.c'],
}

# compute a configuration based on system dependencies, custom logic
conf = configuration_data()
conf.set('USE_FOO', 1)

# Determine the sources to compile
sources_to_compile = []
foreach name, sources : components
  if conf.get('USE_@0@'.format(name.to_upper()), 0) == 1
    sources_to_compile += sources
  endif
endforeach
```



自从 0.49 版本之后，可以在 foreach 中使用 break 和continue 来进行控制。

```
items = ['a', 'continue', 'b', 'break', 'c']
result = []
foreach i : items
  if i == 'continue'
    continue
  elif i == 'break'
    break
  endif
  result += i
endforeach
# result is ['a', 'b']
```

还支持三目运算符。

```
x = condition ? true_value : false_value
```



**Include**

大多数源码目录都有多个子目录要处理。这些可以由meson的subdir命令来处理。

他会切换到子目录，然后执行子目录下的 meson.build 的内容。







## 1.2 线程

test.c 如下：

```c
#include <math.h>
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

void *task(void *param)
{
	sleep(5);
	printf("hello\n");
}

int main()
{
	pthread_t pid;
	pthread_create(&pid, NULL, task, NULL);	
	pthread_join(pid, NULL);
	
	return 0;
}
```

meson.build 如下：

```
project('math', 'c')
#pthreaddep = dependency('pthread')
#executable('demo1', 'main.c', dependencies: pthreaddep)
thread_dep = dependency('threads')
executable('demo1', 'main.c', dependencies: thread_dep)
```

要链接 -lpthread ，在 meson 中只能通过 threads ，而不是 pthread。



## 1.3 编译目标

编译生成的目标可以有四种选项，分比为： executable，library（可以为 static 或者 shared，或者2者都有，在 编译的时候可以进行配置），static_library，shared_library 。

在类Unix 的版本中，shared library 是可以有多个版本共存的。

```
project('shared lib', 'c')
library('mylib', 'source.c', version : '1.2.3', soversion : '0')
```



编译了一个动态库之后，又可以将其链接到 可执行文件：

```
project('shared lib', 'c')
lib = library('mylib', 'source.c')
executable('program', 'prog.c', link_with : lib)
```



## 1.4 安装编译目标

可以通过

```
meson install
```

或者

```
ninja install
```

默认 meson 不会安装任何东西，文件需要被安装的话（也就是执行了 meson install 之后会被安装的），需要通过 install :true 来指定。比如：

```
project('install', 'c')
shared_library('mylib', 'libfile.c', install : true)
```

默认的话，被安装到 prefix 目录（可以通过选项指定，默认的话是在 /usr/local/），可以通过参数 install_dir 改变安装目录：

```
executable('prog', 'prog.c', install : true, install_dir : 'my/special/dir')
```

其他的install 命令如下：

```
install_headers('header.h', subdir : 'projname') # -> include/projname/header.h
install_man('foo.1') # -> share/man/man1/foo.1
install_data('datafile.dat', install_dir : get_option('datadir') / 'progname')
# -> share/progname/datafile.dat
```



### 1.4.1 自定义install 行为

比如有些软件在安装的时候需要设置环境变量啥的，该怎么办呢？

可以将所需要做的操作写入到一个脚本中，例如：

```
#!/bin/sh

mkdir "${DESTDIR}/${MESON_INSTALL_PREFIX}/mydir"
touch "${DESTDIR}/${MESON_INSTALL_PREFIX}/mydir/file.dat"
```

然后通过

```
meson.add_install_script('myscript.sh')
```

语句进行设置。当安装的时候，就会调用这个脚本。



## 1.5 配置



略



## 1.6 生成源码

有时源文件需要在传递给实际编译器之前进行预处理。例如，您可能需要构建一个IDL编译器，然后通过它运行一些文件来生成实际的源文件。在meson中，这是通过generator（）或custom_target（）完成的。



假设您有一个构建目标，它必须使用编译器生成的源代码来构建。编译器可以是编译目标：

```
mycomp = executable('mycompiler', 'compiler.c')
```



或者是在已经安装到系统中的外部命令，或者是源目录下的脚本。

```
mycomp = find_program('mycompiler')
```



自定义目标可以获取零个或多个输入文件，并使用它们生成一个或多个输出文件。使用自定义目标，可以在生成时运行此编译器以生成源：

```
gen_src = custom_target('gen-output',
                        input : ['somefile1.c', 'file2.c'],
                        output : ['out.c', 'out.h'],
                        command : [mycomp, '@INPUT@',
                                   '--c-out', '@OUTPUT0@',
                                   '--h-out', '@OUTPUT1@'])
```

@INPUT@ 会被翻译为 'somefile1.c' 'file2.c' 。就像 output 一样，你也可以您也可以通过索引分别引用每个输入文件。（如上面的 @OUTPUT1@ 会被翻译为 'out.h'）。

```
prog_python = import('python').find_installation('python3')

foo_c = custom_target(
    'foo.c',
    output : 'foo.c',
    input : 'my_gen.py',
    command : [prog_python, '@INPUT@', '--code', '@OUTPUT@'],
)

foo_h = custom_target(
    'foo.h',
    output : 'foo.h',
    input : 'my_gen.py',
    command : [prog_python, '@INPUT@', '--header', '@OUTPUT@'],
)

libfoo = static_library('foo', [foo_c, foo_h])

executable('myexe', ['main.c', foo_h], link_with : libfoo)
```



依赖于生成的头的每个目标都应该将该头添加到它的源中，如上面libfoo和myexe所示。因为表面上因为libfoo 已经依赖了 foo_h，那么 myexe 链接了 libfoo，为什么还要依赖 foo_h 呢？

这是因为meson无法仅仅因为libfoo依赖foo.h就知道myexe依赖foo.h，它可能是一个私有头。





