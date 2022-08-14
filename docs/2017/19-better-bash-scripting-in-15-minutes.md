# 15分钟Bash进阶

## 说明
- [原文链接](http://robertmuth.blogspot.sg/2012/08/better-bash-scripting-in-15-minutes.html)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时: 1.5h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## 更安全的脚本
每个脚本中我都以下面的内从开始：
```
#!/bin/bash
set -o nounset
set -o errexit
```

这会处理两个常见的错误：
1. 引用未定义的变量（默认是""）
2. 忽略执行失败的命令

这两个设置是有对应快捷写法的（"-u"和"-e")，但是原始写法更佳易读。

如果你要忽略可能执行错误的命令，可以使用下面的写法：
```
if ! <possible failing command> ; then
    echo "failure ignored"
fi
```

需要注意的是，有些Linux命令可以使用一些选项来强制忽略错误，比如`rm -f`和`mkdir -p`。

还需要注意的是，在“errexit”模式下，虽然能有效捕捉错误，但不能捕捉全部错误。在特定情况下，有些失败的命令没办法检测。（更多信息可以参考[这篇文章](https://groups.google.com/forum/?fromgroups#!topic/gnu.bash.bug/-9ySnEo1WrQ)）

*一位读者还推荐另一个用法`set -o pipefail`*

## 函数
在Bash中你可以定义其它函数，它们和其它命令一样--你可以随意调用它们;这也会让你的脚本更具可读性。
```
ExtractBashComments() {
    egrep "^#"
}
cat myscript.sh | ExtractBashComments | wc
comments=$(ExtractBashComments < myscript.sh)
```

更多例子：
```
SumLines() {  # iterating over stdin - similar to awk
    local sum=0
    local line=””
    while read line ; do
        sum=$((${sum} + ${line}))
    done
    echo ${sum}
}
SumLines < data_one_number_per_line.txt
log() {  # classic logger
   local prefix="[$(date +%Y/%m/%d\ %H:%M:%S)]: "
   echo "${prefix} $@" >&2
}
log "INFO" "a message"
```

尝试把所有bash代码移植到函数中，只留下全局变量/常量，然后在main函数中统一调用它们。

## 变量注解
Bash允许一种有限制的变量注解形式，最终要的有：
- local(在函数内定义局部变量)
- readonly（只读变量）
```
# a useful idiom: DEFAULT_VAL can be overwritten
#       with an environment variable of the same name
readonly DEFAULT_VAL=${DEFAULT_VAL:-7}
myfunc() {
   # initialize a local variable with the global default
   local some_var=${DEFAULT_VAL}
   ...
}
```

这样，你就可以把以前不是只读的变量声明成只读变量：
```
x=5
x=6
readonly x
x=7   # failure
```

尽量把bash中的所有变量注解成`readonly`或者`local`。

## 用$() 代替 (`)
反单引号在一些字体中难以辨识，很容易和单引号混淆。

**$()允许内嵌，而且避免了转义的麻烦**
```
# both commands below print out: A-B-C-D
echo "A-`echo B-\`echo C-\\\`echo D\\\`\``"
echo "A-$(echo B-$(echo C-$(echo D)))"
```

## [[]]代替[]
`[[]]`能够避免文件扩展名异常，提供一些语法上的改进，还增加了一些新的特性：

|Operator（操作符）|Meaning（含义）|
|:------:|:------:|
| || | 逻辑或 |
| && | 逻辑与 |
| < | 字符比较（双中括号中不需要转义） |
| -lt | 数字比较 |
| = | 字符串比较 |
| == | 以globbing的方式比较字符串，见下文 （仅双中括号有效）|
| =~ | 正则方式比较字符串，见下文（仅双中括号有效） |
| -n | 字符串非空 |
| -z | 字符串为空 |
| -eq | 数字相等 |
| -ne | 数字不相等 |

单中括号：
```
[ "${name}" \> "a" -o ${name} \< "m" ]
```

双中括号：
```
 [[ "${name}" > "a" && "${name}" < "m"  ]]
```

## 正则和Globbing
以下几个例子能够体现出双中括号的强大能力：
```
t="abc123"
[[ "$t" == abc* ]]         # true (globbing)
[[ "$t" == "abc*" ]]       # false (literal matching)
[[ "$t" =~ [abc]+[123]+ ]] # true (regular expression)
[[ "$t" =~ "abc*" ]]       # false (literal matching)
```

注意，bash3.2以后正则表达式或Globbing表达式不能被引号包裹。如果表达式中含有空格，你可以存到变量中：
```
r="a b+"
[[ "a bbb" =~ $r ]]        # true
```

基于Globbing的字符串比较也可以用到case中：
```
case $t in
abc*)  <action> ;;
esac
```

## 字符串操作
bash有很多操作字符串的方法。

- Basics
    ```
    f="path1/path2/file.ext"
    len="${#f}" # = 20 (string length)
    # slicing: ${<var>:<start>} or ${<var>:<start>:<length>}
    slice1="${f:6}" # = "path2/file.ext"
    slice2="${f:6:5}" # = "path2"
    slice3="${f: -8}" # = "file.ext"(Note: space before "-")
    pos=6
    len=5
    slice4="${f:${pos}:${len}}" # = "path2"
    ```
- Substitution (with globbing)
    ```
    f="path1/path2/file.ext"
    single_subst="${f/path?/x}"   # = "x/path2/file.ext"
    global_subst="${f//path?/x}"  # = "x/x/file.ext"
    # string splitting
    readonly DIR_SEP="/"
    array=(${f//${DIR_SEP}/ })
    second_dir="${array[1]}"     # = path2
    ```
- Deletion at beginning/end (with globbing)
    ```
    f="path1/path2/file.ext"
    # deletion at string beginning extension="${f#*.}"  # = "ext"
    # greedy deletion at string beginning
    filename="${f##*/}"  # = "file.ext"
    # deletion at string end
    dirname="${f%/*}"    # = "path1/path2"
    # greedy deletion at end
    root="${f%%/*}"      # = "path1"
    ```

## 避免使用临时文件
一些命令使用文件名作为参数，所以管道就无法使用了。这时`<()`就派上用场了，它可以接受一个命令，然后把命令转换成可以作为文件名的东西：
```
# download and diff two webpages
diff <(wget -O - url1) <(wget -O - url2)
```

“here document”也很有用，它允许把任意行字符串传给标准输入。

下面的MARKER可以换成任何文本：
```
# DELIMITER is an arbitrary string
command  << MARKER
...
${var}
$(cmd)
...
MARKER
```

## 内置变量
- 变量说明
    ```
    $0   name of the script
    $n   positional parameters to script/function
    $$   PID of the script
    $!    PID of the last command executed (and run in the background)
    $?   exit status of the last command  (${PIPESTATUS} for pipelined commands)
    $#   number of parameters to script/function
    $@  all parameters to script/function (sees arguments as separate word)
    $*    all parameters to script/function (sees arguments as single word)
    ```
- 注意
    ```
    $*   is rarely the right choice.
    $@ handles empty parameter list and white-space within parameters correctly
    $@ should usually be quoted like so "$@"
    ```

## 调试
对脚本进行语法检查
```
bash  -n myscript.sh
```

跟踪脚本里每个命令的执行：
```
bash -v myscript.sh
```

跟踪脚本里每个命令的执行并附加扩充信息：
```
bash -x myscript.sh
```

你可以在脚本头部添加`set -o verbose`和`set -o xtrace`来永久指定`-x`和`-v`。

如果脚本运行在远程机器上这会很有效，用它来输出远程信息。

## 什么时候不该用脚本
- 你的脚本很长，不下于几百行
- 除了简单的数组外你还需要数据结构
- 出现复杂的转义问题
- 需要很多字符串操作
- 不太需要调用其它程序或者通过管道和其它程序交互
- 你比较在意性能

你需要考虑Python或者Ruby这样的脚本语言

## 参考
- [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html)
- [Bash Reference Manual](http://www.gnu.org/software/bash/manual/bashref.html)

## 译者说
本文介绍了Bash中很多好的编程习惯和经验，字符串操作和比较是容易忽视以及不易掌握的。
注意bash中的正则和globbing的区别。

另外，本文有很多国人翻译了，译者在翻译本文时有一些翻译参考了[Bash脚本15分钟进阶教程](http://www.vaikan.com/bash-scripting/)，这篇翻译质量很高，我个人学习和借鉴了很多。