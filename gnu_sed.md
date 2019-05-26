# <a name="gnu-sed"></a>GNU sed

**目录**

* [简单查找替换](#简单查找替换)
    * [编辑标准输入](#编辑标准输入)
    * [编辑文件输入](#编辑文件输入)
* [文件内编辑](#文件内编辑)
    * [备份](#备份)
    * [不用备份](#不用备份)
    * [多文件](#多文件)
    * [备份前缀](#备份前缀)
    * [把备份放到文件夹中](#把备份放到文件夹中)
* [按行过滤的选项](#按行过滤的选项)
    * [打印命令](#打印命令)
    * [删除命令](#删除命令)
    * [退出命令](#退出命令)
    * [地址取反正则表达式](#地址取反正则表达式)
    * [绑定多个表达式](#绑定多个表达式)
    * [按行号过滤](#按行号过滤)
    * [只打印行号](#只打印行号)
    * [地址范围](#地址范围)
    * [相对地址](#相对地址)
* [在正则中使用不同的分隔符](#在正则中使用不同的分隔符)
* [正则表达式](#正则表达式)
    * [行锚点](#行锚点)
    * [单词锚点](#单词锚点)
    * [匹配元字符](#匹配元字符)
    * [逻辑或](#逻辑或)
    * [点元字符](#点元字符)
    * [多倍性](#多倍性)
    * [字符相关](#字符相关)
    * [转移字符](#转移字符)
    * [分组](#分组)
    * [逆参照](#逆参照)
    * [大小写转换](#大小写转换)
* [替换命令修改选项](#替换命令修改选项)
    * [g选项](#g选项)
    * [替换指定的次数](#替换指定的次数)
    * [iI选项-忽略大小写](#iI选项-忽略大小写)
    * [p选项-输出](#p选项-输出)
    * [w选项-输出到文件](#w选项-输出到文件)
    * [e选项-替换编辑行](#e选项-替换编辑行)
    * [m-选项](#m-选项)
* [脚本替换](#脚本替换)
    * [变量替换](#变量替换)
    * [命令替换](#命令替换)
* [z和s命令](#z和s命令)
* [修改命令](#修改命令)
* [插入命令](#插入命令)
* [追加命令](#追加命令)
* [添加文件内容](#添加文件内容)
    * [r命令插入整个文件](#r命令插入整个文件)
    * [r命令逐行插入](#r命令逐行插入)
* [n and N commands](#n-and-n-commands)
* [Control structures](#control-structures)
    * [if then else](#if-then-else)
    * [replacing in specific column](#replacing-in-specific-column)
    * [overlapping substitutions](#overlapping-substitutions)
* [Lines between two REGEXPs](#lines-between-two-regexps)
    * [Include or Exclude matching REGEXPs](#include-or-exclude-matching-regexps)
    * [First or Last block](#first-or-last-block)
    * [Broken blocks](#broken-blocks)
* [sed scripts](#sed-scripts)
* [Gotchas and Tips](#gotchas-and-tips)
* [Further Reading](#further-reading)

<br>

```bash
$ sed --version | head -n1
sed (GNU sed) 4.2.2

$ man sed
SED(1)                           User Commands                          SED(1)

NAME
       sed - stream editor for filtering and transforming text

SYNOPSIS
       sed [OPTION]... {script-only-if-no-other-script} [input-file]...

DESCRIPTION
       Sed  is a stream editor.  A stream editor is used to perform basic text
       transformations on an input stream (a file or input from  a  pipeline).
       While  in  some  ways similar to an editor which permits scripted edits
       (such as ed), sed works by making only one pass over the input(s),  and
       is consequently more efficient.  But it is sed's ability to filter text
       in a pipeline which particularly distinguishes it from other  types  of
       editors.
...
```

**注意:** [Multiline and manipulating pattern space](https://www.gnu.org/software/sed/manual/sed.html#Multiline-techniques) with h,x,D,G,H,P etc is not covered in this chapter and examples/information is based on ASCII encoded text input only

<br>

## <a name="简单查找替换"></a>简单查找替换

详细的**substitute**替换命令例子在以后的章节中提供, 语法是：

```
s/REGEXP/REPLACEMENT/FLAGS
```

`/`字符通常当作分割符使用. See also [在正则中使用不同的分隔符](#在正则中使用不同的分隔符)

<br>

#### <a name="编辑标准输入"></a>编辑标准输入

```bash
$ # 要被编辑的命令
$ seq 10 | paste -sd,
1,2,3,4,5,6,7,8,9,10

$ # 只把第一个 ',' 换成 ' : '
$ seq 10 | paste -sd, | sed 's/,/ : /'
1 : 2,3,4,5,6,7,8,9,10

$ # 只把所有的 ',' 换成 ' : '
$ seq 10 | paste -sd, | sed 's/,/ : /g'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
```

**注意:** 作为一个好的实践, 所有使用带引号的参数都会妨碍脚本翻译. 见 [脚本替换](#脚本替换) 章节使用双引号

<br>

#### <a name="编辑文件输入"></a>编辑文件输入

* 换行符是默认的行分割标准
* 见 [正则表达式](#正则表达式) 节确定搜索的词汇, 例如
    * 单词的边界区分 'hi', 'this', 'his', 'history', 等
    * 查找多个单词, 指定一系列字符,等

```bash
$ cat greeting.txt
Hi there
Have a nice day

$ # 每行第一个 'e' 改为 'E'
$ sed 's/e/E/' greeting.txt
Hi thEre
HavE a nice day

$ # change first 'nice day' in each line to 'safe journey'
$ # 每行第一个 'enice day' 改为 'safe journey'
$ sed 's/nice day/safe journey/' greeting.txt
Hi there
Have a safe journey

$ # 所有 'e' 改为 'E'保存到另外一个文件
$ sed 's/e/E/g' greeting.txt > out.txt
$ cat out.txt
Hi thErE
HavE a nicE day
```

<br>

## <a name="文件内编辑"></a>文件内编辑

* 之前的章节, `sed` 的输出直接显示或保存在一个文件
* 会写文件的改变,使用 `-i` 选项

**注意**:

* 到`man sed` 有如何使用`-i`选项. 根据`sed` 的不同实现不同.这列使用`sed (GNU sed) 4.2.2`
* 见[unix.stackexchange - working with symlinks](https://unix.stackexchange.com/questions/348693/sed-update-etc-grub-conf-in-spite-this-link-file)

<br>

#### <a name="备份"></a>备份

* 档有扩展名,原来的文件根据给的扩展名备份

```bash
$ # '.bkp'是提供的扩展名
$ sed -i.bkp 's/Hi/Hello/' greeting.txt
$ # sed 备份原来的greeting.txt到 'greeting.txt.bkp'
$ cat greeting.txt
Hello there
Have a nice day

$ cat greeting.txt.bkp
Hi there
Have a nice day
```

<br>

#### <a name="不用备份"></a>不用备份

* 这样使用请注意,修改无法取消

```bash
$ sed -i 's/nice day/safe journey/' greeting.txt

$ # note, 'Hi' was already changed to 'Hello' in previous example
$ cat greeting.txt
Hello there
Have a safe journey
```

<br>

#### <a name="多文件"></a>多文件

* 每个文件当作单独文件对待,修改也分布写回各自文件

```bash
$ cat f1
I ate 3 apples
$ cat f2
I bought two bananas and 3 mangoes

$ # -i 可以使用 不用备份
$ sed -i 's/3/three/' f1 f2
$ cat f1
I ate three apples
$ cat f2
I bought two bananas and three mangoes
```

<br>

#### <a name="备份前缀"></a>备份前缀

* `-i`选项后面的 `*` 参数被展开成输入的文件名
* 这样,可以为备份添加前后缀

```bash
$ cat var.txt
foo
bar
baz

$ sed -i'bkp.*' 's/foo/hello/' var.txt
$ cat var.txt
hello
bar
baz

$ cat bkp.var.txt
foo
bar
baz
```

<br>

#### <a name="把备份放到文件夹中"></a>把备份放到文件夹中

* `*` 也运行指定一个目录放备份

```bash
$ mkdir bkp_dir
$ sed -i'bkp_dir/*' 's/bar/hi/' var.txt
$ cat var.txt
hello
hi
baz

$ cat bkp_dir/var.txt
hello
bar
baz

$ # extensions can be added as well
$ # bkp_dir/*.bkp for suffix
$ # bkp_dir/bkp.* for prefix
$ # bkp_dir/bkp.*.2017 for both and so on
```

<br>

## <a name="按行过滤的选项"></a>按行过滤的选项

* 默认`sed` 作用于整个文件. 根据文件查找的结果,行号或两个模式匹配中间的行,只修改某些指定的行很常见
* 这个过滤器像`grep`, `head` and `tail` 等命令,但是有更多的功能
    * TODO:: ? Use `sed` for inplace editing, the filtered lines to be transformed etc. Not as substitute for those commands

<br>

#### <a name="打印命令"></a>打印命令

* 通常打印命令和 `-n` 选项连用
* `sed` 默认打印输入文件被修改后的最后结果
    * 打印这里指在终端显示,重定向到文件
* 一起用`-n`和`p`, 只显示被影响的那些行
* 以下例子使用`/REGEXP/`正则表达式地址

```bash
$ cat poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # 所有包含'are'的行
$ # 同: grep 'are' poem.txt
$ sed -n '/are/p' poem.txt
Roses are red,
Violets are blue,
And so are you.

$ # all lines containing the string 'so are'
$ # 所有包含'so are'的行
$ # 同: grep 'so are' poem.txt
$ sed -n '/so are/p' poem.txt
And so are you.
```

* 一起使用打印和替换

```bash
$ # 只打印有替换的行
$ sed -n 's/are/ARE/p' poem.txt
Roses ARE red,
Violets ARE blue,
And so ARE you.

$ # 如果行中包含'are', 执行替换操作
$ # 如果替换成功就打印
$ sed -n '/are/ s/so/SO/p' poem.txt
And SO are you.
```

* 重复输入文件的每一行

```bash
$ # note, -n is not used and no filtering applied
$ seq 3 | sed 'p'
1
1
2
2
3
3
```

<br>

#### <a name="删除命令"></a>删除命令

* `sed` 默认打印输入文件的修改结果
* 用`d` 命令, 删除的行不被显示

```bash
$ # 同: grep -v 'are' poem.txt
$ sed '/are/d' poem.txt
Sugar is sweet,

$ # 同: seq 5 | grep -v '3'
$ seq 5 | sed '/3/d'
1
2
4
5
```

* `I` 选项运行用大小写敏感的方式过滤行
* 见[正则表达式](#正则表达式)

```bash
$ # /rose/I 不考虑大小写匹配字符串'rose'
$ sed '/rose/Id' poem.txt
Violets are blue,
Sugar is sweet,
And so are you.
```

<br>

#### <a name="退出命令"></a>退出命令

* 不处理其它的输入退出 `sed`

```bash
$ # 同: seq 23 45 | head -n5
$ # 记住没有使用-n 是默认打印
$ # 这里, 5 是行号
$ seq 23 45 | sed '5q'
23
24
25
26
27
```

* `Q` 同`q` 少打印正好匹配的那一行

```bash
$ seq 23 45 | sed '5Q'
23
24
25
26

$ # 从文件开始打印到匹配的那一行(但不包括该行)
$ sed '/is/Q' poem.txt
Roses are red,
Violets are blue,
```

* 用`tac`反转文件所有行

```bash
$ # 从文件的后面开始第一次出现数字'7'的行
$ seq 50 | tac | sed '/7/q' | tac
47
48
49
50

$ # 从文件的后面开始第一次出现数字'7'的行(但不包括该行)
$ seq 50 | tac | sed '/7/Q' | tac
48
49
50
```

**注意**

* TODO: 这样使用 退出命令 在多文件内编辑不生效
* See also [unix.stackexchange - applying changes to 多文件](https://unix.stackexchange.com/questions/309514/sed-apply-changes-in-多文件)

<br>

#### <a name="地址取反正则表达式"></a>地址取反正则表达式

* 用`!` 去指定地址的其它地址

```bash
$ # 同: sed -n '/so are/p' poem.txt
$ sed '/so are/!d' poem.txt
And so are you.

$ # 同: sed '/are/d' poem.txt
$ sed -n '/are/!p' poem.txt
Sugar is sweet,
```

<br>

#### <a name="绑定多个表达式"></a>绑定多个表达式

* 见[sed manual - Multiple commands syntax](https://www.gnu.org/software/sed/manual/sed.html#Multiple-commands-syntax) for more details
* 见[sed scripts](#sed-scripts) section for an alternate way

```bash
$ # 每个命令当作-e的一个参数
$ sed -n -e '/blue/p' -e '/you/p' poem.txt
Violets are blue,
And so are you.

$ # 用;分开
$ # 不是所有的命令可以这样做
$ sed -n '/blue/p; /you/p' poem.txt
Violets are blue,
And so are you.

$ # 每个命令占一行
$ # 取决于脚本是否运行多行命令
$ sed -n '
/blue/p
/you/p
' poem.txt
Violets are blue,
And so are you.
```

* 用 `{}` 对命令分组进行 AND 运算

```bash
$ # 同: grep 'are' poem.txt | grep 'And'
$ # /REGEXP/ {} 之间的空格是可选的
$ sed -n '/are/ {/And/p}' poem.txt
And so are you.

$ # 同: grep 'are' poem.txt | grep -v 'so'
$ sed -n '/are/ {/so/!p}' poem.txt
Roses are red,
Violets are blue,

$ # 同: grep -v 'red' poem.txt | grep -v 'blue'
$ sed -n '/red/!{/blue/!p}' poem.txt
Sugar is sweet,
And so are you.
$ # 有多种方法完成,用感觉简单的即可
$ # sed -e '/red/d' -e '/blue/d' poem.txt
$ # grep -v -e 'red' -e 'blue' poem.txt
```

* 不同方法做同样的事情. 见[逻辑或](#逻辑或) and [Control structures](#control-structures)

```bash
$ # 多个命令因为匹配了多次会造成重复
$ sed -n '/blue/p; /t/p' poem.txt
Violets are blue,
Violets are blue,
Sugar is sweet,
$ # 这种情况可以使用表达式
$ sed -nE '/blue|t/p;' poem.txt
Violets are blue,
Sugar is sweet,

$ sed -nE '/red|blue/!p' poem.txt
Sugar is sweet,
And so are you.

$ sed -n '/so/b; /are/p' poem.txt
Roses are red,
Violets are blue,
```

<br>

#### <a name="按行号过滤"></a>按行号过滤

* 可以指定要处理的具体行
* 特例, `$` 表示文件的最后一行
* 见[sed manual - Multiple commands syntax](https://www.gnu.org/software/sed/manual/sed.html#Multiple-commands-syntax)

```bash
$ # 这里, 2 代表打印第2行, 同/REGEXP/p
$ # 同: head -n2 poem.txt | tail -n1
$ sed -n '2p' poem.txt
Violets are blue,

$ # 打印第2和4行
$ sed -n '2p; 4p' poem.txt
Violets are blue,
And so are you.

$ # 同: tail -n1 poem.txt
$ sed -n '$p' poem.txt
And so are you.

$ # 除了第3行删除其它
$ sed '3!d' poem.txt
Sugar is sweet,

$ # 只在第2行替换
$ sed '2 s/are/ARE/' poem.txt
Roses are red,
Violets ARE blue,
Sugar is sweet,
And so are you.
```

* 大型文件, 同时使用`p`和`q`为了快速退出
* `q`使用了后 `sed`将立即退出

```bash
$ # 不加q 循环结束才退出
$ seq 3542 4623452 | sed -n '2452{p;q}'
5993

$ seq 3542 4623452 | sed -n '250p; 2452{p;q}'
3791
5993

$ # 这个例子打印出花费的时间
$ time seq 3542 4623452 | sed -n '2452{p;q}' > /dev/null

real    0m0.003s
user    0m0.000s
sys     0m0.000s
$ time seq 3542 4623452 | sed -n '2452p' > /dev/null

real    0m0.334s
user    0m0.396s
sys     0m0.024s
```

* 用  `q`命令模仿`head`

```bash
$ # 同: seq 23 45 | head -n5
$ seq 23 45 | sed '5q'
23
24
25
26
27
```

<br>

#### <a name="只打印行号"></a>只打印行号

```bash
$ # 匹配的行号和行内容
$ grep -n 'blue' poem.txt
2:Violets are blue,

$ # 只给匹配行行号
$ sed -n '/blue/=' poem.txt
2

$ sed -n '/are/=' poem.txt
1
2
4
```

* 如果需要,匹配行也可以打印但是回变成一个新行

```bash
$ sed -n '/blue/{=;p}' poem.txt
2
Violets are blue,

$ # or
$ sed -n '/blue/{p;=}' poem.txt
Violets are blue,
2
```

<br>

#### <a name="地址范围"></a>地址范围

* `sed` 运行指定地址范围

```bash
$ cat addr_range.txt
Hello World

Good day
How are you

Just do-it
Believe it

Today is sunny
Not a bit funny
No doubt you like it too

Much ado about nothing
He he he
```

* 用*正则表达式*定义地址范围
* 其它例子见 For other cases like getting lines without the line matching start and/or end, unbalanced start/end, when end *正则表达式* doesn't match, etc see [Lines between two REGEXPs](#lines-between-two-regexps) section

```bash
$ sed -n '/is/,/like/p' addr_range.txt
Today is sunny
Not a bit funny
No doubt you like it too

$ sed -n '/just/I,/believe/Ip' addr_range.txt
Just do-it
Believe it

$ # 第2个表达式总数在第一个地址匹配后才检查
$ sed -n '/No/,/No/p' addr_range.txt
Not a bit funny
No doubt you like it too

$ # 所有匹配的范围都会被打印
$ sed -n '/you/,/do/p' addr_range.txt
How are you

Just do-it
No doubt you like it too

Much ado about nothing
```

* 用起止行号定义范围

```bash
$ # 打印3到7行
$ sed -n '3,7p' addr_range.txt
Good day
How are you

Just do-it
Believe it

$ # 从13行打印到最后一行
$ sed -n '13,$p' addr_range.txt
Much ado about nothing
He he he

$ # 从2删除到13行
$ sed '2,13d' addr_range.txt
Hello World
He he he
```

* 混合使用行号和*正则表达式*定义范围

```bash
$ sed -n '3,/do/p' addr_range.txt
Good day
How are you

Just do-it

$ sed -n '/Today/,$p' addr_range.txt
Today is sunny
Not a bit funny
No doubt you like it too

Much ado about nothing
He he he
```

* 地址范围取反, 只需要在地址范围后加`!` 

```bash
$ # 同: seq 10 | sed '3,7d'
$ seq 10 | sed -n '3,7!p'
1
2
8
9
10

$ # 同: sed '/Today/,$d' addr_range.txt
$ sed -n '/Today/,$!p' addr_range.txt
Hello World

Good day
How are you

Just do-it
Believe it

```

<br>

#### <a name="相对地址"></a>相对地址

* 在数字前用`+`表示相对地址
* Similar to using `grep -A<num> --no-group-separator 'REGEXP'` but `grep` merges adjacent groups while `sed` does not

```bash
$ # 行匹配 'is' 所在行和后面2行
$ sed -n '/is/,+2p' addr_range.txt
Today is sunny
Not a bit funny
No doubt you like it too

$ # 所有匹配的范围都会被过滤出来
$ sed -n '/do/,+2p' addr_range.txt
Just do-it
Believe it

No doubt you like it too

Much ado about nothing
```

* 第一个地址也可以是数组
* 在使用[脚本替换](#脚本替换)很有用

```bash
$ sed -n '3,+4p' addr_range.txt
Good day
How are you

Just do-it
Believe it
```

* 另一个相对格式地址是`i~j`,代表了i+j, i+2j, i+3j, 等行
    * `1~2` 意味着1, 3, 5, 7, ... 行 (即 奇数行)
    * `5~3`  5, 8, 11, ... 行 

```bash
$ # 匹配奇数行
$ # 偶数,用2~2
$ seq 10 | sed -n '1~2p'
1
3
5
7
9

$ # 匹配线性数行: 2, 2+2*2, 2+3*2, etc
$ seq 10 | sed -n '2~4p'
2
6
10
```

* 如果 `~j` 在`,` 后,意思就完全不一样了
* 在匹配第一个地址后,到最近的`j`的倍数就是结束地址

```bash
$ # 2是开始地址
$ # 最接近4的倍数的地址是4
$ seq 10 | sed -n '2,~4p'
2
3
4
$ #  最接近4的倍数的地址是8
$ seq 10 | sed -n '5,~4p'
5
6
7
8

$ # 匹配`Just`的是第6行, 所以第10行结束
$ sed -n '/Just/,~5p' addr_range.txt
Just do-it
Believe it

Today is sunny
Not a bit funny
```

<br>

## <a name="在正则中使用不同的分隔符"></a>在正则中使用不同的分隔符

* `/` 习惯上用于*正则表达式*的分隔符
    * 见[a bit of history on why / is commonly used as delimiter](https://www.reddit.com/r/commandline/comments/3lhgwh/why_did_people_standardize_on_using_forward/cvgie7j/)
* 其它的字符也可以用于正则表达式的分隔符
* 有助于避免/重用`\`

```bash
$ # 不修改的做法
$ echo '/home/learnbyexample/reports' | sed 's/\/home\/learnbyexample\//~\//'
~/reports

$ # 使用其它分隔符
$ echo '/home/learnbyexample/reports' | sed 's#/home/learnbyexample/#~/#'
~/reports
```

* *正则表达式*用于地址匹配语法不完全相同`\<char>REGEXP<char>`

```bash
$ printf '/foo/bar/1\n/foo/baz/1\n'
/foo/bar/1
/foo/baz/1

$ printf '/foo/bar/1\n/foo/baz/1\n' | sed -n '\;/foo/bar/;p'
/foo/bar/1
```

<br>

## <a name="正则表达式"></a>正则表达式

* 默认, `sed` 把 *REGEXP正则表达式* 当作 BRE (Basic Regular Expression基本正则表达式)
* `-E` 选项激活了 ERE (Extended Regular Expression扩展正则表达式) 在GNU sed's中只是在元字符使用上有区别, 功能上没有区别
    * 最早的GNU sed只有`-r`选项,激活ERE,`man sed` 没有提到`-E`
    * 其它的`sed`版本,使用`-E` 和`grep` 使用`-E`. 所以`-r`在这里不用
    * 见[sed manual - BRE-vs-ERE](https://www.gnu.org/software/sed/manual/sed.html#BRE-vs-ERE)
* 详见[sed manual - 正则表达式](https://www.gnu.org/software/sed/manual/sed.html#sed-正则表达式) 

<br>

#### <a name="行锚点"></a>行锚点

* 通常, 从行的开头或者行的结束对查找很有用
* 例如, `C`语言整形变量声明用可选空格开始,关键字`int`,空格然后是变量名
    * 可以避免在行内匹配注释
* 同样, 从后面匹配可以得到变量名

不使用行锚点

```bash
$ cat anchors.txt
cat and dog
too many cats around here
to concatenate, use the cmd cat
catapults laid waste to the village
just scat and quit bothering me
that is quite a fabricated tale
try the grape variety muscat

$ # 没有锚点,替换命令将替换所有发行的字符串
$ sed 's/cat/XXX/g' anchors.txt
XXX and dog
too many XXXs around here
to conXXXenate, use the cmd XXX
XXXapults laid waste to the village
just sXXX and quit bothering me
that is quite a fabriXXXed tale
try the grape variety musXXX
```

* 元字符 `^` 强迫 *正则表达式* 匹配行首

```bash
$ # 过滤所有'cat' 开始的行
$ sed -n '/^cat/p' anchors.txt
cat and dog
catapults laid waste to the village

$ # 只替换在行首的
$ # g选项这里是不需要的
$ sed 's/^cat/XXX/' anchors.txt
XXX and dog
too many cats around here
to concatenate, use the cmd cat
XXXapults laid waste to the village
just scat and quit bothering me
that is quite a fabricated tale
try the grape variety muscat

$ # 在行首添加
$ echo 'Have a good day' | sed 's/^/Hi! /'
Hi! Have a good day
```

* 元字符 `$` 强迫 *正则表达式* 匹配行尾

```bash
$ # 匹配以'cat'结尾的行
$ sed -n '/cat$/p' anchors.txt
to concatenate, use the cmd cat
try the grape variety muscat

$ # 只替换行尾出现的
$ sed 's/cat$/YYY/' anchors.txt
cat and dog
too many cats around here
to concatenate, use the cmd YYY
catapults laid waste to the village
just scat and quit bothering me
that is quite a fabricated tale
try the grape variety musYYY

$ # 在行尾添加
$ echo 'Have a good day' | sed 's/$/. Cya later/'
Have a good day. Cya later
```

<br>

#### <a name="单词锚点"></a>单词锚点

* A **word** character is any alphabet (irrespective of case) or any digit or the underscore character
* The word anchors help in matching or not matching boundaries of a word
    * 例如, to distinguish between `par`, `spar` and `apparent`
* `\b` matches word boundary
    * `\` is meta character and certain combinations like `\b` and `\B` have special meaning

```bash
$ # 以'cat'结尾的词
$ sed -n 's/cat\b/XXX/p' anchors.txt
XXX and dog
to concatenate, use the cmd XXX
just sXXX and quit bothering me
try the grape variety musXXX

$ # 以'cat'开头的词
$ sed -n 's/\bcat/YYY/p' anchors.txt
YYY and dog
too many YYYs around here
to concatenate, use the cmd YYY
YYYapults laid waste to the village

$ # 只匹配这个单词
$ sed -n 's/\bcat\b/ZZZ/p' anchors.txt
ZZZ and dog
to concatenate, use the cmd ZZZ

$ # 单词有字母,数组和下划线组成
$ echo 'foo, foo_bar and foo1' | sed 's/\bfoo\b/baz/g'
baz, foo_bar and foo1
```

* `\B`和`\b`相反, 就是它不匹配单词的边界

```bash
$ # 当 'cat'前后是字符时才替换
$ sed -n 's/\Bcat\B/QQQ/p' anchors.txt
to conQQQenate, use the cmd cat
that is quite a fabriQQQed tale

$ # 'cat'不是单词的开始才替换
$ sed -n 's/\Bcat/RRR/p' anchors.txt
to conRRRenate, use the cmd cat
just sRRR and quit bothering me
that is quite a fabriRRRed tale
try the grape variety musRRR

$ # 'cat'不是单词的结束才替换
$ sed -n 's/cat\B/SSS/p' anchors.txt
too many SSSs around here
to conSSSenate, use the cmd cat
SSSapults laid waste to the village
that is quite a fabriSSSed tale
```

* 以下可以代替`\b`使用
    * `\<` 单词头部
    * `\>` 单词尾部

```bash
$ # 同: sed 's/\bcat\b/X/g'
$ echo 'concatenate cat scat cater' | sed 's/\<cat\>/X/g'
concatenate X scat cater

$ # 在单词的首尾添加,就是替换\b
$ echo 'hi foo_baz 3b' | sed 's/\b/:/g'
:hi: :foo_baz: :3b:

$ # 只在单词的首部加
$ echo 'hi foo_baz 3b' | sed 's/\</:/g'
:hi :foo_baz :3b

$ # 只在单词的尾部加
$ echo 'hi foo_baz 3b' | sed 's/\>/:/g'
hi: foo_baz: 3b:
```

<br>

#### <a name="匹配元字符"></a>匹配元字符

* `^`, `$`, `\` 等在*正则表达式*中有特殊的意义, 匹配他们时要使用`\`

```bash
$ # '^' 匹配行首
$ echo '(a+b)^2 = a^2 + b^2 + 2ab' | sed 's/^/**/g'
**(a+b)^2 = a^2 + b^2 + 2ab

$ # '\` 在'^'前,匹配'^'字符
$ echo '(a+b)^2 = a^2 + b^2 + 2ab' | sed 's/\^/**/g'
(a+b)**2 = a**2 + b**2 + 2ab

$ # 匹配'\' 用'\\'
$ echo 'foo\bar' | sed 's/\\/ /'
foo bar

$ echo 'pa$$' | sed 's/$/s/g'
pa$$s
$ echo 'pa$$' | sed 's/\$/s/g'
pass

$ # '^' 只在表达式的首部有特殊意义
$ # 同样, '$' 只在表达式尾部有特殊意义
$ echo '(a+b)^2 = a^2 + b^2 + 2ab' | sed 's/a^2/A^2/g'
(a+b)^2 = A^2 + b^2 + 2ab
```

* 特殊的字符`&` and `\` 在*REPLACEMENT*替换节有特殊意义. 也要用`\`
* 分隔符当然要转义
* 见[逆参照](#逆参照) 部分`&`在*REPLACEMENT*替换部分的用法

```bash
$ # & 代表要匹配的整个字符串
$ echo 'foo and bar' | sed 's/and/"&"/'
foo "and" bar
$ echo 'foo and bar' | sed 's/and/"\&"/'
foo "&" bar

$ # 需要的时候使用不同的分隔符
$ echo 'a b' | sed 's/ /\//'
a/b
$ echo 'a b' | sed 's# #/#'
a/b

$ # 用\\ 代表 \
$ echo '/foo/bar/baz' | sed 's#/#\\#g'
\foo\bar\baz
```

<br>

#### <a name="逻辑或"></a>逻辑或

* 几个 *正则表达式* 可以用 `|` 连接表示逻辑 或
    * 语法是 `\|` 基本正则表达式 and `|` for 扩展正则表达式
* `|` 的两边都是一个完整的正则表达式
* 逻辑或的两边是怎么处理的超过了这里的范围
    * 见[this](https://www.正则表达式.info/alternation.html) for more info on this topic.

```bash
$ # BRE
$ sed -n '/red\|blue/p' poem.txt
Roses are red,
Violets are blue,

$ # ERE
$ sed -nE '/red|blue/p' poem.txt
Roses are red,
Violets are blue,

$ # 匹配以'cat'开头或结尾的行
$ sed -nE '/^cat|cat$/p' anchors.txt
cat and dog
to concatenate, use the cmd cat
catapults laid waste to the village
try the grape variety muscat

$ # g选项代表多次匹配
$ echo 'foo and temp and baz' | sed -E 's/foo|temp|baz/XYZ/'
XYZ and temp and baz
$ echo 'foo and temp and baz' | sed -E 's/foo|temp|baz/XYZ/g'
XYZ and XYZ and XYZ
```

<br>

#### <a name="点元字符"></a>点元字符

* `.` 匹配任何一个字符,包括新行

```bash
$ # 替换所有'c'开头 't'结尾的3个字符
$ echo 'coat cut fit c#t' | sed 's/c.t/XYZ/g'
coat XYZ fit XYZ

$ #  替换所有'c'开头 't'结尾的4个字符
$ echo 'coat cut fit c#t' | sed 's/c..t/ABCD/g'
ABCD cut fit c#t

$ # 空格,制表符等也可以用'.'匹配
$ echo 'coat cut fit c#t' | sed 's/t.f/IJK/g'
coat cuIJKit c#t
```

<br>

#### <a name="多倍性"></a>多倍性

`sed`的所有的多倍性都是贪婪的, 即: 当有多个满足的匹配时,最长的匹配胜出,优先权从左到右

* `?` 匹配 0 或 1 次
* 对于基本正则表达式, 用 `\?`

```bash
$ printf 'late\npale\nfactor\nrare\nact\n'
late
pale
factor
rare
act

$ # 同 using: 代表c出现了0次或1次 sed -nE '/at|act/p'
$ printf 'late\npale\nfactor\nrare\nact\n' | sed -nE '/ac?t/p'
late
factor
act

$ # 贪婪匹配在一些情况下很好用
$ # 问题 : 如果 '<'前没有 '\' 就被替换为'\<'
$ echo 'blah \< foo bar < blah baz <'
blah \< foo bar < blah baz <
$ # '\<' 替换成'\\<'不正确
$ echo 'blah \< foo bar < blah baz <' | sed -E 's/</\\</g'
blah \\< foo bar \< blah baz \<
$ # 使用'\\?<' , '\<' 和 '<' 都被替换成了'\<'
$ echo 'blah \< foo bar < blah baz <' | sed -E 's/\\?</\\</g'
blah \< foo bar \< blah baz \<
```

* `*`  匹配 0 或 多 次

```bash
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n'
abc
ac
adc
abbc
bbb
bc
abbbbbc

$ # match 'a' and 'c' with any number of 'b' in between
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -n '/ab*c/p'
abc
ac
abbc
abbbbbc

$ # 从头删除到'te'的位置,这里匹配了最长的
$ echo 'that is quite a fabricated tale' | sed 's/.*te//'
d tale
$ # 从头删除到'te '的位置,这里匹配了最长的
$ echo 'that is quite a fabricated tale' | sed 's/.*te //'
a fabricated tale
$ # 从f开头产出到行尾
$ echo 'that is quite a fabricated tale' | sed 's/f.*//'
that is quite a 
```

* `+`  匹配 1 或 多 次
* 对于基本正则表达式, use `\+`

```bash
$ # 匹配'a' 到'c' 中间最少有1个'b'
$ # BRE
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -n '/ab\+c/p'
abc
abbc
abbbbbc

$ # ERE
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab+c/p'
abc
abbc
abbbbbc
```

* 更精确的控制匹配的次数,用 `{}`

```bash
$ # 正好 5 次
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{5}c/p'
abbbbbc

$ # 1 到 3 次, 包括 1 和 3
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{1,3}c/p'
abc
abbc

$ # 最多 2 次, 包括 0 次
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{,2}c/p'
abc
ac
abbc

$ # 最少 2 次
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{2,}c/p'
abbc
abbbbbc

$ # BRE
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -n '/ab\{2,\}c/p'
abbc
abbbbbc
```

<br>

#### <a name="字符相关"></a>字符相关

* `.` 可以匹配任何字符
* 字符相关的匹配可以匹配 `[]` 中的所有字符

```bash
$ # 同: sed -nE '/lane|late/p'
$ printf 'late\nlane\nfate\nfete\n' | sed -n '/la[nt]e/p'
late
lane

$ printf 'late\nlane\nfate\nfete\n' | sed -n '/[fl]a[nt]e/p'
late
lane
fate

$ # 多倍性也可以用于其它字符
$ # 匹配所有数字组成的行,至少包括一个数字
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[0123456789]+$/p'
123
42
$ # 匹配所有数字组成的行,至少包括3个数字
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[0123456789]{3,}$/p'
123
```

字符范围

* 如果每个字符都要被单独指定匹配会很麻烦
* 所以,有个简单的办法,使用`-` 表示一个范围(用升序指定)
* 见[ascii codes table](https://ascii.cl/) for reference
    * Note that behavior of range will depend on locale settings
    * [arch wiki - locale](https://wiki.archlinux.org/index.php/locale)
    * [Linux: Define Locale and Language Settings](https://www.shellhacks.com/linux-define-locale-language-settings/)

```bash
$ # 匹配所有数字组成的行,至少包括一个数字
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[0-9]+$/p'
123
42

$ # filter lines made up entirely of lower case alphabets, at least one
$ # 匹配所有小写字母组成的行,至少包括一个
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[a-z]+$/p'
foo

$ # 匹配所有小写字母和数字组成的行,至少包括一个
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[a-z0-9]+$/p'
cat5
foo
123
42
```

* 数字的范围,并不总是适合的,对于算术运算用`awk` 或`perl`
* 见[Matching Numeric Ranges with a Regular Expression](https://www.正则表达式.info/numericranges.html)

```bash
$ # 10 29 之间的数字
$ printf '23\n154\n12\n26\n98234\n' | sed -n '/^[12][0-9]$/p'
23
12
26

$ # numbers >= 100
$ printf '23\n154\n12\n26\n98234\n' | sed -nE '/^[0-9]{3,}$/p'
154
98234

$ # numbers >= 100 处理有0开头的情况
$ printf '0501\n035\n154\n12\n26\n98234\n' | sed -nE '/^0*[1-9][0-9]{2,}$/p'
0501
154
98234
```

字符类取反

* 元字符在`[]`内外意义完全不一样
* 例如, `^` 在 `[]` 中的第一个的意思是匹配不是该字符的其它字符

```bash
$ # 删除=号前面的0或多个字符
$ echo 'foo=bar; baz=123' | sed 's/^[^=]*//'
=bar; baz=123

$ # 删除最后一个=号后面的0或多个字符
$ echo 'foo=bar; baz=123' | sed 's/[^=]*$//'
foo=bar; baz=

$ # 同: sed -n '/[aeiou]/!p'
$ printf 'tryst\nglyph\npity\nwhy\n' | sed -n '/^[^aeiou]*$/p'
tryst
glyph
why
```

在`[]`內匹配元字符

* `^`, `]`, `-`, 等作为列表的部分需要特别注意
* 当然, 序列 `[.` or `=]` 在 `[]` 中有特别的意思
    * 见[sed manual - Character-Classes-and-Bracket-Expressions](https://www.gnu.org/software/sed/manual/sed.html#Character-Classes-and-Bracket-Expressions) for complete list

```bash
$ # 匹配 - 要放在[] 的第一个或最后一个
$ printf 'Foo-bar\nabc-456\n42\nCo-operate\n' | sed -nE '/^[a-z-]+$/Ip'
Foo-bar
Co-operate

$ # 匹配 ] 要在[]中的第一个
$ printf 'int foo\nint a[5]\nfoo=bar\n' | sed -n '/[]=]/p'
int a[5]
foo=bar

$ # 用 [ 匹配 [
$ # [][] 将匹配 [ 和 ]
$ printf 'int foo\nint a[5]\nfoo=bar\n' | sed -n '/[[]/p'
int a[5]

$ # 匹配^不能放在首字母
$ printf 'c=a^b\nd=f*h+e\nz=x-y\n' | sed -n '/[*^]/p'
c=a^b
d=f*h+e
```

命名字符分类

* 同C locale 和ASCII character encoding分类
    * See [ascii codes table](https://ascii.cl/) for reference
* See [sed manual - Character Classes and Bracket Expressions](https://www.gnu.org/software/sed/manual/sed.html#Character-Classes-and-Bracket-Expressions) for more details

| 字符相关 | 描述 |
| ------------- | ----------- |
| `[:digit:]` | 同 `[0-9]` |
| `[:lower:]` | 同 `[a-z]` |
| `[:upper:]` | 同 `[A-Z]` |
| `[:alpha:]` | 同 `[a-zA-Z]` |
| `[:alnum:]` | 同 `[0-9a-zA-Z]` |
| `[:xdigit:]` | 同 `[0-9a-fA-F]` |
| `[:cntrl:]` | 控制字符 - first 32 ASCII characters and 127th (DEL) |
| `[:punct:]` | 标点符号|
| `[:graph:]` | `[:alnum:]` and `[:punct:]` |
| `[:print:]` | `[:alnum:]`, `[:punct:]` and space |
| `[:blank:]` | Space and tab characters |
| `[:space:]` | white-space characters: tab, newline, vertical tab, form feed, carriage return and space |

```bash
$ # 只包括16进制字符的所有行
$ printf '128\n34\nfe32\nfoo1\nbar\n' | sed -nE '/^[[:xdigit:]]+$/p'
128
34
fe32

$ # 包括最少一个非16进制字符的行
$ printf '128\n34\nfe32\nfoo1\nbar\n' | sed -n '/[^[:xdigit:]]/p'
foo1
bar

$ # 同: sed -nE '/^[a-z-]+$/Ip'
$ printf 'Foo-bar\nabc-456\n42\nCo-operate\n' | sed -nE '/^[[:alpha:]-]+$/p'
Foo-bar
Co-operate

$ # 移除所有标点符号
$ sed 's/[[:punct:]]//g' poem.txt
Roses are red
Violets are blue
Sugar is sweet
And so are you
```

反斜杠字符类

*  同C locale 和ASCII character encoding分类
    * See [ascii codes table](https://ascii.cl/) for reference
* See [sed manual - regular expression extensions](https://www.gnu.org/software/sed/manual/sed.html#regexp-extensions) for more details

| 字符相关 | 描述 |
| ------------- | ----------- |
| `\w` | 同 `[0-9a-zA-Z_]` or `[[:alnum:]_]` |
| `\W` | 同 `[^0-9a-zA-Z_]` or `[^[:alnum:]_]` |
| `\s` | 同 `[[:space:]]` |
| `\S` | 同 `[^[:space:]]` |

```bash
$ # 只包含数字,大小写字母和下划线
$ printf '123\na=b+c\ncmp_str\nFoo_bar\n' | sed -nE '/^\w+$/p'
123
cmp_str
Foo_bar

$ # 反斜杠字符类不能用于[]中,不像perl
$ # \w would simply match w
$ echo 'w=y-x+9*3' | sed 's/[\w=]//g'
y-x+9*3
$ echo 'w=y-x+9*3' | perl -pe 's/[\w=]//g'
-+*
```

<br>

#### <a name="转移字符"></a>转移字符

* 某些字符像tab, carriage return, newline, 等有转义字符代表他们
    * 可以用于`[]` 内
* Any ASCII character can be also represented using their decimal or octal or hexadecimal value
    * See [ascii codes table](https://ascii.cl/) for reference
* See [sed manual - Escapes](https://www.gnu.org/software/sed/manual/sed.html#Escapes) for more details

```bash
$ # tab的例子
$ printf 'foo\tbar\tbaz\n'
foo     bar     baz
$ printf 'foo\tbar\tbaz\n' | sed 's/\t/ /g'
foo bar baz
$ echo 'a b c' | sed 's/ /\t/g'
a       b       c

$ # 在字符类中使用转义
$ printf 'a\tb\vc\n'
a       b
         c
$ printf 'a\tb\vc\n' | cat -vT
a^Ib^Kc
$ printf 'a\tb\vc\n' | sed 's/[\t\v]/ /g'
a b c

$ # 用16进制代表单引号
$ # 等于十进制的'\d039' 和八进制的'\o047'
$ echo "foo: '34'"
foo: '34'
$ echo "foo: '34'" | sed 's/\x27/"/g'
foo: "34"
$ echo 'foo: "34"' | sed 's/"/\x27/g'
foo: '34'
```

<br>

#### <a name="分组"></a>分组

* 字符允许根据选择的多个字符匹配,可加上出现的数量 
* 分组一个作用是模拟整个表达式,不需要逐个列出
* `()` 用于分组
    * 基础正则表达式用`\(\)`
* 类似于数学中的`ab + ac = a(b+c)`, 正则表达式这样写 `a(b|c) = ab|ac`

```bash
$ # 4个字母的中间包含'on' 或'no' 单词
$ printf 'known\nmood\nknow\npony\ninns\n' | sed -nE '/\b[a-z](on|no)[a-z]\b/p'
know
pony
$ # TODO:: common mistake to use character class, will match 'oo' and 'nn' as well
$ printf 'known\nmood\nknow\npony\ninns\n' | sed -nE '/\b[a-z][on]{2}[a-z]\b/p'
mood
know
pony
inns

$ # 数量的例子 ? 0或1
$ printf 'handed\nhand\nhandy\nhands\nhandle\n' | sed -nE '/^hand([sy]|le)?$/p'
hand
handy
hands
handle

$ # 移除开头两列用:分隔的
$ echo 'foo:123:bar:baz' | sed -E 's/^([^:]+:){2}//'
bar:baz

$ # 可以嵌套
$ printf 'spade\nscore\nscare\nspare\nsphere\n' | sed -nE '/^s([cp](he|a)[rd])e$/p'
spade
scare
spare
sphere
```

<br>

#### <a name="逆参照"></a>逆参照

 在`()`中间匹配的字符串可以用于后引用.按括号分组
* `\1` 代表第一个匹配的分组, `\2` 代表第二个..
    * 次序是左起
    * 可用于*正则表达式*和*REPLACEMENT*替换节
* `&` 或 `\0` 代表要被替换的字符
* 注意是参照被匹配的字符串,不是参照正则表达式
    * 例如: `([0-9][a-f])` 匹配 `3b`, 逆参照应该是 `3b`不是其它的表达式
* `\` 和 `&` 在*REPLACEMENT*替换部分是特殊字符, 用`\\` 和`\&`代表

```bash
$ # 匹配连续的两个字母的行
$ printf 'eel\nflee\nall\npat\nilk\nseen\n' | sed -nE '/([a-z])\1/p'
eel
flee
all
seen

$ # 减少 \\ 为  \ 如果只有一个 \ 删除
$ echo '\[\] and \\w and \[a-zA-Z0-9\_\]' | sed -E 's/(\\?)\\/\1/g'
[] and \w and [a-zA-Z0-9_]

$ # 移除2或多个空格分隔的重复单词
$ # word boundaries prevent false matches like 'the theatre' 'sand and stone' etc
$ echo 'a a a walking for for a cause' | sed -E 's/\b(\w+)( \1)+\b/\1/g'
a walking for a cause

$ # 只对第3列用双引号包围 // TODO:
$ # 注意嵌套在分组和数字在替换中的使用
$ echo 'foo:123:bar:baz' | sed -E 's/^(([^:]+:){2})([^:]+)/\1"\3"/'
foo:123:"bar":baz

$ # 把第一列添加到行尾
$ echo 'foo:123:bar:baz' | sed -E 's/^([^:]+).*/& \1/'
foo:123:bar:baz foo

$ # 整行用双引号包围
$ echo 'hello world' | sed 's/.*/"&"/'
"hello world"
$ # 在行首或行尾添加
$ echo 'hello world' | sed 's/.*/Hi. &. Have a nice day/'
Hi. hello world. Have a nice day
```

<br>

#### <a name="大小写转换"></a>大小写转换

* 只在*替换*部分有效, 不像`perl`可以在*正则表达式*有效
* 详见[sed manual - The s Command](https://www.gnu.org/software/sed/manual/sed.html#The-_0022s_0022-Command) for more details and corner cases

```bash
$ # 大写所有字符,遇到\L 或者 \E 停止
$ echo 'HeLlO WoRLD' | sed 's/.*/\U&/'
HELLO WORLD

$ # 小写所有字符,遇到\U 或者 \E 停止
$ echo 'HeLlO WoRLD' | sed 's/.*/\L&/'
hello world

$ # 只大写下一个字符
$ echo 'foo bar' | sed 's/\w*/\u&/g'
Foo Bar
$ echo 'foo_bar next_line' | sed -E 's/_([a-z])/\u\1/g'
fooBar nextLine

$ # 只小写下一个字符
$ echo 'FOO BAR' | sed 's/\w*/\l&/g'
fOO bAR
$ echo 'fooBar nextLine Baz' | sed -E 's/([a-z])([A-Z])/\1_\l\2/g'
foo_bar next_line Baz

$ # 大写第一个字符,输入是大小写混合的
$ echo 'HeLlO WoRLD' | sed 's/.*/\L&/; s/\w*/\u&/g'
Hello World
$ # sed 's/.*/\L\u&/' 也可以,但是不确定是不是已定义的行为
$ echo 'HeLlO WoRLD' | sed 's/.*/\L&/; s/./\u&/'
Hello world

$ # \E 会停止\U 或 \L开始的转换
$ echo 'foo_bar next_line baz' | sed -E 's/([a-z]+)(_[a-z]+)/\U\1\E\2/g'
FOO_bar NEXT_line baz
```

<br>

## <a name="替换命令修改选项"></a>替换命令修改选项

`s` 命令的语法:

```
s/REGEXP/REPLACEMENT/FLAGS
```

* 修改选项 `g`, `p` and `I` 已经了解过. 为了完整性,讨论其它的 替换命令修改选项
* 详见[sed manual - The s Command](https://www.gnu.org/software/sed/manual/sed.html#The-_0022s_0022-Command) 

<br>

#### <a name="g选项"></a>g选项

默认,替换命令只匹配第一个. `g` 替换所有

```bash
$ # 只体会第一个 : 为 -  
$ echo 'foo:123:bar:baz' | sed 's/:/-/'
foo-123:bar:baz

$ # 替换所有的 : 为 -
$ echo 'foo:123:bar:baz' | sed 's/:/-/g'
foo-123-bar-baz
```

<br>

#### <a name="替换指定的次数"></a>替换指定的次数

* 可以指定替换的次数

```bash
$ # 替换第一次出现
$ echo 'foo:123:bar:baz' | sed 's/:/-/'
foo-123:bar:baz
$ # [^:] 匹配不为:的所有字符1个到多个,替换第2次出现这样的情况
$ echo 'foo:123:bar:baz' | sed -E 's/[^:]+/XYZ/'
XYZ:123:bar:baz

$ # 在第二次出现时替换
$ echo 'foo:123:bar:baz' | sed 's/:/-/2'
foo:123-bar:baz
$ # [^:] 匹配不为:的所有字符1个到多个,替换第2次出现这样的情况
$ echo 'foo:123:bar:baz' | sed -E 's/[^:]+/XYZ/2'
foo:XYZ:bar:baz

$ #  在第3次出现时替换
$ echo 'foo:123:bar:baz' | sed 's/:/-/3'
foo:123:bar-baz
$ # [^:] 匹配不为:的所有字符1个到多个,替换第3次出现这样的情况
$ echo 'foo:123:bar:baz' | sed -E 's/[^:]+/XYZ/3'
foo:123:XYZ:baz

$ # 基于输入的多倍性的选择
$ # 不为:的0-n个字符第2次出现,第一个:前算第一次出现
$ echo ':123:bar:baz' | sed 's/[^:]*/XYZ/2'
:XYZ:bar:baz
$ echo ':123:bar:baz' | sed -E 's/[^:]+/XYZ/2'
:123:XYZ:baz
```

* 从行尾匹配第几次的替换未知如何做
* 确保多倍性贪婪原则

```bash
$ # 替换最后一次匹配
$ # \1 第一个匹配的引用
$ # \1 匹配 foo:123:bar  整个表达式匹配为 foo:123:bar:  
$ # 也可以用 sed -E 's/:([^:]*)$/-\1/'
$ echo 'foo:123:bar:baz' | sed -E 's/(.*):/\1-/'
foo:123:bar-baz
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):/\1-/'
456:foo:123:bar:789-baz
$ echo 'foo and bar and baz land good' | sed -E 's/(.*)and/\1XYZ/'
foo and bar and baz lXYZ good
$ # 可以使用单词边界
$ echo 'foo and bar and baz land good' | sed -E 's/(.*)\band\b/\1XYZ/'
foo and bar XYZ baz land good

$ # 替换倒数第2个:
$ echo 'foo:123:bar:baz' | sed -E 's/(.*):(.*:)/\1-\2/'
foo:123-bar:baz
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):(.*:)/\1-\2/'
456:foo:123:bar-789:baz

$ # 替换倒数第3个:
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):((.*:){2})/\1-\2/'
456:foo:123-bar:789:baz
$ # 替换倒数第4个:
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):((.*:){3})/\1-\2/'
456:foo-123:bar:789:baz
```

* 替换所有的但是处理前*N*次出现和 `g`一起用

```bash
$ # 用-替换所有的:除了前两个
$ echo '456:foo:123:bar:789:baz' | sed -E 's/:/-/3g'
456:foo:123-bar-789-baz

$ # 用-替换所有的:除了前3个
$ echo '456:foo:123:bar:789:baz' | sed -E 's/:/-/4g'
456:foo:123:bar-789-baz
```

* 替换*N*次出现

```bash
$ # 把前两个: 为-
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/; s/:/-/'
456-foo-123:bar:789:baz

$ # 用-替换第2和3次出现的: 
$ # note the changes in number to be used for subsequent replacement
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/2; s/:/-/2'
456:foo-123-bar:789:baz

$ # 更好的方法使用降序
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/3; s/:/-/2'
456:foo-123-bar:789:baz
$ # 用-替换第2,3,5次出现
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/5; s/:/-/3; s/:/-/2'
456:foo-123-bar:789-baz
```

<br>

#### <a name="iI选项-忽略大小写"></a>iI选项-忽略大小写

* `i` 或者`I` 可以用户大小写敏感的替换
* TODO:  Since only `I` can be used for address filtering (for ex: `sed '/rose/Id' poem.txt`), use `I` for substitute command as well for consistency

```bash
$ echo 'hello Hello HELLO HeLlO' | sed 's/hello/hi/g'
hi Hello HELLO HeLlO

$ echo 'hello Hello HELLO HeLlO' | sed 's/hello/hi/Ig'
hi hi hi hi
```

<br>

#### <a name="p选项-输出"></a>p选项-输出

* 通常和`-n` 一起用只输出修改的行

```bash
$ # 没有替换就不输出
$ echo 'hi there. have a nice day' | sed -n 's/xyz/XYZ/p'
$ # 有替换只输出修改过的行
$ echo 'hi there. have a nice day' | sed -n 's/\bh/H/pg'
Hi there. Have a nice day

$ # 只输出包括'are'的行
$ sed -n 's/are/ARE/p' poem.txt
Roses ARE red,
Violets ARE blue,
And so ARE you.

$ # 包含'are'和'so'的行,替换'so'为'SO'
$ sed -n '/are/ s/so/SO/p' poem.txt
And SO are you.
```

<br>

#### <a name="w选项-输出到文件"></a>w选项-输出到文件

* 允许把修改输出到指定文件

```bash
$ # 在w 和 文件名之间的空格是可选的
$ # 同: sed -n 's/3/three/p' > 3.txt
$ seq 20 | sed -n 's/3/three/w 3.txt'
$ cat 3.txt
three
1three

$ # 当写到文件时也要输出到屏幕 不要用-n
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(:[^:]*){2}$//w col.txt'
456:foo:123:bar
$ cat col.txt
456:foo:123:bar
```

* 多个输出文件,用`-e`指定每个文件

```bash
$ seq 20 | sed -n -e 's/5/five/w 5.txt' -e 's/7/seven/w 7.txt'
$ cat 5.txt
five
1five
$ cat 7.txt
seven
1seven
```

* 有两个预定义的文件名
    * `/dev/stdout` to write to **stdout**
    * `/dev/stderr` to write to **stderr**

```bash
$ # 在编辑文件的时候同时输出到终端
$ sed -i 's/three/3/w /dev/stdout' 3.txt
3
13
$ cat 3.txt
3
13
```

<br>

#### <a name="e选项-替换编辑行"></a>e选项-替换编辑行

* 替换的时候不输出原来的行

```bash
$ # 替换行
$ printf 'Date:\nreplace this line\n'
Date:
replace this line
$ printf 'Date:\nreplace this line\n' | sed 's/^replace.*/date/e'
Date:
Thu May 25 10:19:46 IST 2017

$ # 使用 p选项-输出 和 e-选项, 顺序很重要
$ printf 'Date:\nreplace this line\n' | sed -n 's/^replace.*/date/ep'
Thu May 25 10:19:46 IST 2017
$ printf 'Date:\nreplace this line\n' | sed -n 's/^replace.*/date/pe'
date

$ # 用e选项改后的结果当作shell命令执行
$ echo 'xyz 5' | sed 's/xyz/seq/e'
1
2
3
4
5
```

<br>

#### <a name="m-选项"></a>m-选项

* `m` 或`M` 都可以使用
* 我们已经见识过基于行的操作,换行符用于区分不同的行
* 有不用的方法(见[sed manual - How sed Works](https://www.gnu.org/software/sed/manual/sed.html#Execution-Cycle)) 在模式空间中多于一行,`m`选项可以运用
* 见[unix.stackexchange - usage of multi-line选项-替换编辑行](https://unix.stackexchange.com/questions/298670/simple-significant-usage-of-m-multi-line-address-suffix) for more examples

在看`m`选项前,看一下两行的模式空间的例子

```bash
$ # 在模式空间匹配'blue'所在行和下一行
$ sed -n '/blue/{N;p}' poem.txt
Violets are blue,
Sugar is sweet,

$ # 运用替换,记住.匹配了新行
$ sed -n '/blue/{N;s/are.*is//p}' poem.txt
Violets  sweet,
```

* 当`m`选项使用,影响`^`, `$` 和 `.` 的元字符的行为

```bash
$ # 没有用m选项, ^ 只匹配整个模式空间的开头
$ sed -n '/blue/{N;s/^/:: /pg}' poem.txt
:: Violets are blue,
Sugar is sweet,
$ # 用m选项, ^ 匹配模式空间中的每一行的开头
$ sed -n '/blue/{N;s/^/:: /pgm}' poem.txt
:: Violets are blue,
:: Sugar is sweet,

$ # 同样运用于$
$ sed -n '/blue/{N;s/$/ ::/pg}' poem.txt
Violets are blue,
Sugar is sweet, ::
$ sed -n '/blue/{N;s/$/ ::/pgm}' poem.txt
Violets are blue, ::
Sugar is sweet, ::

$ # 用m选项, . 不匹配换行符
$ sed -n '/blue/{N;s/are.*//p}' poem.txt
Violets 
$ sed -n '/blue/{N;s/are.*//pm}' poem.txt
Violets 
Sugar is sweet,
```

<br>

## <a name="脚本替换"></a>脚本替换

* `bash`的例子和其它的 shell不同
* See also [stackoverflow - Difference between single and double quotes in Bash](https://stackoverflow.com/questions/6697753/difference-between-single-and-double-quotes-in-bash)
* For robust substitutions taking care of meta characters in *正则表达式* and *REPLACEMENT* sections, see
    * [unix.stackexchange - How to ensure that string interpolated into sed substitution escapes all metachars](https://unix.stackexchange.com/questions/129059/how-to-ensure-that-string-interpolated-into-sed-substitution-escapes-all-metac)
    * [unix.stackexchange - What characters do I need to escape when using sed in a sh script?](https://unix.stackexchange.com/questions/32907/what-characters-do-i-need-to-escape-when-using-sed-in-a-sh-script)
    * [stackoverflow - Is it possible to escape regex metacharacters reliably with sed](https://stackoverflow.com/questions/29613304/is-it-possible-to-escape-regex-metacharacters-reliably-with-sed)

<br>

#### <a name="变量替换"></a>变量替换

* 简单的例子是用双引号

```bash
$ word='are'
$ sed -n "/$word/p" poem.txt
Roses are red,
Violets are blue,
And so are you.

$ replace='ARE'
$ sed "s/$word/$replace/g" poem.txt
Roses ARE red,
Violets ARE blue,
Sugar is sweet,
And so ARE you.

$ # 需要使用合适的分隔符
$ echo 'home path is:' | sed "s/$/ $HOME/"
sed: -e expression #1, char 7: unknown option to `s'
$ echo 'home path is:' | sed "s|$| $HOME|"
home path is: /home/learnbyexample
```

* 如果命令中有`\`, backtick, `!` 等, 只用双引号引用变量

```bash
$ # if history expansion is enabled, ! is special
$ word='are'
$ sed "/$word/!d" poem.txt
sed "/$word/date +%A" poem.txt
sed: -e expression #1, char 7: extra characters after command

$ # so double quote only the variable
$ # the command is concatenation of '/' and "$word" and '/!d'
$ sed '/'"$word"'/!d' poem.txt
Roses are red,
Violets are blue,
And so are you.
```

<br>

#### <a name="命令替换"></a>命令替换

* 比`e`选项更加灵活

```bash
$ echo 'today is date' | sed 's/date/'"$(date +%A)"'/'
today is Tuesday

$ # 合适的使用分隔符
$ echo 'current working dir is: ' | sed 's/$/'"$(pwd)"'/'
sed: -e expression #1, char 6: unknown option to `s'
$ echo 'current working dir is: ' | sed 's|$|'"$(pwd)"'|'
current working dir is: /home/learnbyexample/command_line_text_processing

$ # 这样是不能替换成多行
$ echo 'foo' | sed 's/foo/'"$(seq 5)"'/'
sed: -e expression #1, char 7: unterminated `s' command
```

<br>

## <a name="z和s命令"></a>z和s命令

* 已经看过`-n`, `-e`, `-i` and `-E`
* 这里介绍`-z` and `-s`选项
* 见[sed manual - Command line options](https://www.gnu.org/software/sed/manual/sed.html#Command_002dLine-Options) for other options and more details

`-z` 将使`sed`用ASCII NUL字符分割新行而不是用换行符

```bash
$ # 用\0 null分隔行
$ # for ex: output of grep -Z, find -print0, etc
$ printf 'teal\0red\nblue\n\0green\n' | sed -nz '/red/p' | cat -A
red$
blue$
^@

$ # 整个文件当作一行字符串处理
$ # 如果当前行用c开头,在前一行加;
$ printf 'cat\ndog\ncoat\ncut\nmat\n' | sed -z 's/\nc/;&/g'
cat
dog;
coat;
cut
mat
```

`-s`选项使`sed`把多个输入文件当作单独对待,而不是当作一串输入. 如果用了`-i`, `-s`默认使用

```bash
$ # 没有 -s, 只有一个第一行
$ # F 命令打印当前文件的名字
$ sed '1F' f1 f2
f1
I ate three apples
I bought two bananas and three mangoes

$ # 用 -s, 每个文件有自己的地址
$ sed -s '1F' f1 f2
f1
I ate three apples
f2
I bought two bananas and three mangoes
```

<br>

<br>

## <a name="修改命令"></a>修改命令

修改命令 `c` 删除当前行用给定的字符串代替

**注意** 字符串不能包含换行符,有的话使用转义字符

```bash
$ # c和替换成字符的空格可以忽略
$ seq 3 | sed '2c foo bar'
1
foo bar
3

$ # 所有地址范围都被替换
$ seq 8 | sed '3,7cfoo bar'
1
2
foo bar
8

$ # 可以在替换字符串中使用转义字符
$ sed '/red/,/is/chello\nhi there' poem.txt
hello
hi there
And so are you.
```

* 命令将作用于所有匹配的地址

```bash
$ seq 5 | sed '/[24]/cfoo'
1
foo
3
foo
5
```

* `\`紧跟`c`有特殊意义, see [sed manual - other commands](https://www.gnu.org/software/sed/manual/sed.html#Other-Commands) for details
* If escape sequence is needed at beginning of replacement string, use an additional `\`

```bash
$ # \ 帮助添加前导的空格
$ seq 3 | sed '2c  a'
1
a
3
$ seq 3 | sed '2c\ a'
1
 a
3

$ seq 3 | sed '2c\tgood day'
1
tgood day
3
$ seq 3 | sed '2c\\tgood day'
1
        good day
3
```

* `;` 出现不能区分字符串和命令结尾, 多个命令`-e`

```bash
$ sed -e '/are/cHi;s/is/IS/' poem.txt
Hi;s/is/IS/
Hi;s/is/IS/
Sugar is sweet,
Hi;s/is/IS/

$ sed -e '/are/cHi' -e 's/is/IS/' poem.txt
Hi
Hi
Sugar IS sweet,
Hi
```

* 用脚本替换

```bash
$ text='good day'
$ seq 3 | sed '2c'"$text"
1
good day
3

$ text='good day\nfoo bar'
$ seq 3 | sed '2c'"$text"
1
good day
foo bar
3

$ seq 3 | sed '2c'"$(date +%A)"
1
Thursday
3

$ # 多行输出的命令会报错
$ seq 3 | sed '2c'"$(seq 2)"
sed: -e expression #1, char 5: missing command
```

<br>

## <a name="插入命令"></a>插入命令

插入命令运行在指定地址的一行前加字符串

**注意** 字符串不能包含换行符,有的话使用转义字符

```bash
$ # i和字符串中间的空格被忽略
$ # 同: sed '2s/^/hello\n/'
$ seq 3 | sed '2i hello'
1
hello
2
3

$ # 使用转移字符
$ seq 3 | sed '2ihello\nhi'
1
hello
hi
2
3
```

* 命令将作用与所有匹配的地址命令将作用于所有匹配的地址

```bash
# [24] 表示一行中只要有2或4都匹配成功
$ seq 5 | sed '/[24]/ifoo'
1
foo
2
3
foo
4
5
```

* `\` 紧跟在`i`是特殊的,单独一个`\`代表换行 see [sed manual - other commands](https://www.gnu.org/software/sed/manual/sed.html#Other-Commands) for details
* If escape sequence is needed at beginning of replacement string, use an additional `\`

```bash
$ seq 3 | sed '2i  foo'
1
foo
2
3
$ seq 3 | sed '2i\ foo'
1
 foo
2
3

$ seq 3 | sed '2i\tbar'
1
tbar
2
3
$ seq 3 | sed '2i\\tbar'
1
        bar
2
3
```

* `;` 不能用于区分是字符还是命令的结束,多个命令用`-e`

```bash
$ sed -e '/is/ifoobar;s/are/ARE/' poem.txt
Roses are red,
Violets are blue,
foobar;s/are/ARE/
Sugar is sweet,
And so are you.

$ sed -e '/is/ifoobar' -e 's/are/ARE/' poem.txt
Roses ARE red,
Violets ARE blue,
foobar
Sugar is sweet,
And so ARE you.
```

* 使用脚本替换

```bash
$ text='good day'
$ seq 3 | sed '2i'"$text"
1
good day
2
3

$ text='good day\nfoo bar'
$ seq 3 | sed '2i'"$text"
1
good day
foo bar
2
3

**Note** 用$() 中间运行命令,得到字符串接到插入的内容中
$ seq 3 | sed '2iToday is '"$(date +%A)"
1
Today is Thursday
2
3

$ # multiline command output will lead to error
$ seq 3 | sed '2i'"$(seq 2)"
sed: -e expression #1, char 5: missing command
```

<br>

## <a name="追加命令"></a>追加命令

追加命令 在指定地址的行后面增加字符串

**注意** 字符串不能包含换行符,有的话使用转义字符

```bash
$ # white-space between a and string is ignored
$ # 同: sed '2s/$/\nhello/'
$ seq 3 | sed '2a hello'
1
2
hello
3

$ # 可以使用转移字符
$ seq 3 | sed '2ahello\nhi'
1
2
hello
hi
3
```

* 命令将作用于所有匹配的地址

```bash
$ seq 5 | sed '/[24]/afoo'
1
2
foo
3
4
foo
5
```

* `\`在`a`后是特殊的, see [sed manual - other commands](https://www.gnu.org/software/sed/manual/sed.html#Other-Commands) for details
* 如果在字符串的前面需要转义字符,需要多加一个`\`

```bash
$ seq 3 | sed '2a  foo'
1
2
foo
3
$ seq 3 | sed '2a\ foo'
1
2
 foo
3

$ seq 3 | sed '2a\tbar'
1
2
tbar
3
$ seq 3 | sed '2a\\tbar'
1
2
        bar
3
```

* `;` 不能用于区分是字符还是命令的结束,多个命令用`-e`

```bash
$ sed -e '/is/afoobar;s/are/ARE/' poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
foobar;s/are/ARE/
And so are you.

$ sed -e '/is/afoobar' -e 's/are/ARE/' poem.txt
Roses ARE red,
Violets ARE blue,
Sugar is sweet,
foobar
And so ARE you.
```

* 使用脚本替换

```bash
$ text='good day'
$ seq 3 | sed '2a'"$text"
1
2
good day
3

$ text='good day\nfoo bar'
$ seq 3 | sed '2a'"$text"
1
2
good day
foo bar
3

$ seq 3 | sed '2aToday is '"$(date +%A)"
1
2
Today is Thursday
3

$ # 多行输出的命令会导致错误
$ seq 3 | sed '2a'"$(seq 2)"
sed: -e expression #1, char 5: missing command
```

* 见 [stackoverflow - add newline character if last line of input doesn't have one](https://stackoverflow.com/questions/41343062/what-does-this-mean-in-linux-sed-a-a-txt)

<br>

## <a name="添加文件内容"></a>添加文件内容

<br>

#### <a name="r命令插入整个文件"></a>r命令插入整个文件

* `r` 命令运行按给出的地址插入文件的内容
* 这是一个有效的方法加入多行内容
*特殊的文件名`/dev/stdin`允许从键盘输入
* 第一,把一个文件插入到另一个特别的文件中

```bash
$ cat 5.txt
five
1five

$ cat poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # r命令和文件名中间的空格是可选的
$ sed '2r 5.txt' poem.txt
Roses are red,
Violets are blue,
five
1five
Sugar is sweet,
And so are you.

$ # 内容不能在第一行前添加
$ sed '0r 5.txt' poem.txt
sed: -e expression #1, char 2: invalid usage of line address 0
$ # but that is trivial to solve: cat 5.txt poem.txt
```

* 命令将作用于所有匹配的地址

```bash
$ seq 5 | sed '/[24]/r 5.txt'
1
2
five
1five
3
4
five
1five
5
```

* adding content of variable as it is without any interpretation
* also shows example for using `/dev/stdin`

```bash
$ text='Good day\nfoo bar baz\n'
$ # 用'a'命令的时候转义字符如\n 被解释
$ sed '/is/a'"$text" poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
Good day
foo bar baz

And so are you.

$ # \ 是另一个字符,不会被r命令当做特殊字符
$ echo "$text" | sed '/is/r /dev/stdin' poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
Good day\nfoo bar baz\n
And so are you.
```

* 添加多行命令输出也很简单

```bash
$ seq 3 | sed '/is/r /dev/stdin' poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
1
2
3
And so are you.
```

* 用文件内容替换一行或一个范围
* 见[unix.stackexchange - various ways to replace line M in file1 with line N in file2](https://unix.stackexchange.com/a/396450)

```bash
$ # 替换范围
$ # 顺序很重要,先用'r'插入再用'd'删除
$ sed -e '/is/r 5.txt' -e '1,/is/d' poem.txt
five
1five
And so are you.

$ # 替换一行
$ seq 3 | sed -e '3r /dev/stdin' -e '3d' poem.txt
Roses are red,
Violets are blue,
1
2
3
And so are you.

$ # 也可以使用{} 分组避免重复地址
$ seq 3 | sed -e '/blue/{r /dev/stdin' -e 'd}' poem.txt
Roses are red,
1
2
3
Sugar is sweet,
And so are you.
```

<br>

#### <a name="R命令逐行插入"></a>R命令逐行插入

* 为所有匹配的地址添加一行
* Special name `/dev/stdin` allows to read from **stdin** instead of file input

```bash
$ # R 和 文件名之间的空格是可选的
$ seq 3 | sed '/are/R /dev/stdin' poem.txt
Roses are red,
1
Violets are blue,
2
Sugar is sweet,
And so are you.
3
$ # TODO:: to replace matching line
$ seq 3 | sed -e '/are/{R /dev/stdin' -e 'd}' poem.txt
1
2
Sugar is sweet,
3

$ sed '2,3R 5.txt' poem.txt
Roses are red,
Violets are blue,
five
Sugar is sweet,
1five
And so are you.
```

* number of lines from file to be read different from number of matching address lines

```bash
$ # file has more lines than matching address
$ # 2 lines in 5.txt but only 1 line matching 'is'
$ sed '/is/R 5.txt' poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
five
And so are you.

$ # lines matching address is more than file to be read
$ # 3 lines matching 'are' but only 2 lines from stdin
$ seq 2 | sed '/are/R /dev/stdin' poem.txt
Roses are red,
1
Violets are blue,
2
Sugar is sweet,
And so are you.
```

<br>

## <a name="n-and-n-commands"></a>n and N commands

* 这两个命令会取得匹配的下一行(newline or NUL character separated, depending on options)

Quoting from [sed manual - common commands](https://www.gnu.org/software/sed/manual/sed.html#Common-Commands) for `n` command

>If auto-print is not disabled, print the pattern space, then, regardless, replace the pattern space with the next line of input. If there is no more input then sed exits without processing any more commands.

```bash
$ # 如果当前行包含'blue', 只替换下一行的'e'变成'E'
$ sed '/blue/{n;s/e/E/g}' poem.txt
Roses are red,
Violets are blue,
Sugar is swEEt,
And so are you.

$ # 用-n选项描述更好
$ sed -n '/blue/{n;s/e/E/pg}' poem.txt
Sugar is swEEt,

$ # 如果当前行包含'blue', 只替换下下一行的'e'变成'E'
$ sed -n '/blue/{n;n;s/e/E/pg}' poem.txt
And so arE you.
```

Quoting from [sed manual - other commands](https://www.gnu.org/software/sed/manual/sed.html#Other-Commands) for `N` command

>Add a newline to the pattern space, then append the next line of input to the pattern space. If there is no more input then sed exits without processing any more commands

>When -z is used, a zero byte (the ascii ‘NUL’ character) is added between the lines (instead of a new line)

* See also [stackoverflow - apply substitution every 4 lines but excluding the 4th line](https://stackoverflow.com/questions/40229578/how-to-insert-a-line-feed-into-a-sed-line-concatenation)

```bash
$ # 如果当前行包含'blue', 替换两行的'e'变成'E'
$ sed '/blue/{N;s/e/E/g}' poem.txt
Roses are red,
ViolEts arE bluE,
Sugar is swEEt,
And so are you.

$ # 用-n选项描述更好
$ sed -n '/blue/{N;s/e/E/pg}' poem.txt
ViolEts arE bluE,
Sugar is swEEt,

$ sed -n '/blue/{N;N;s/e/E/pg}' poem.txt
ViolEts arE bluE,
Sugar is swEEt,
And so arE you.
```

* Combination

```bash
$ # n will fetch next line, current line is out of pattern space
$ # N will then add another line
$ sed -n '/blue/{n;N;s/e/E/pg}' poem.txt
Sugar is swEEt,
And so arE you.
```

* not necessary to qualify with an address

```bash
$ seq 6 | sed 'n;cXYZ'
1
XYZ
3
XYZ
5
XYZ

$ seq 6 | sed 'N;s/\n/ /'
1 2
3 4
5 6
```

<br>

## <a name="control-structures"></a>Control structures

* Using `:label` one can mark a command location to branch to conditionally or unconditionally
* See [sed manual - Commands for sed gurus](https://www.gnu.org/software/sed/manual/sed.html#Programming-Commands) for more details

<br>

#### <a name="if-then-else"></a>if then else

* Simple if-then-else can be simulated using `b` command
* `b` command will unconditionally branch to specified label
* Without label, `b` will skip rest of commands and start next cycle
* See [unix.stackexchange - processing only lines between REGEXPs](https://unix.stackexchange.com/questions/292819/remove-commented-lines-except-one-comment-using-sed) for interesting use case

```bash
$ # 取相反数 changing -ve to +ve and vice versa
$ cat nums.txt
42
-2
10101
-3.14
-75
$ # 同: perl -pe '/^-/ ? s/// : s/^/-/'
$ # 空正则表达式的部分会使用之前的正则
$ sed '/^-/{s///;b}; s/^/-/' nums.txt
-42
2
-10101
3.14
75

$ # 同: perl -pe '/are/ ? s/e/*/g : s/e/#/g'
$ # 包含'are'的行'e' 替换成 '*' 没有的替换成'#'
$ sed '/are/{s/e/*/g;b}; s/e/#/g' poem.txt
Ros*s ar* r*d,
Viol*ts ar* blu*,
Sugar is sw##t,
And so ar* you.
```

<br>

#### <a name="replacing-in-specific-column"></a>replacing in specific column

* `t` command will branch to specified label on successful substitution
* Without label, `t` will skip rest of commands and start next cycle
* More examples
    * [stackoverflow - replace data after last delimiter](https://stackoverflow.com/questions/39907133/replace-data-after-last-delimiter-of-every-line-using-sed-or-awk/39908523#39908523)
    * [stackoverflow - replace multiple occurrences in specific column](https://stackoverflow.com/questions/42886531/replace-mutliple-occurances-in-delimited-columns/42886919#42886919)

```bash
$ # replace space with underscore only in 3rd column
$ # ^(([^|]+\|){2} captures first two columns
$ # [^|]* zero or more non-column separator characters
$ # 只要一直有匹配,同一条命令在同一行会重复匹配
$ echo 'foo bar|a b c|1 2 3|xyz abc' | sed -E ':a s/^(([^|]+\|){2}[^|]*) /\1_/; ta'
foo bar|a b c|1_2_3|xyz abc

$ # use awk/perl for simpler syntax
$ # for ex: awk 'BEGIN{FS=OFS="|"} {gsub(/ /,"_",$3); print}'
```

* `b` and `t`的区别

```bash
$ # whether or not 'R' is found on lines containing 'are', branch will happen
$ sed '/are/{s/R/*/g;b}; s/e/#/g' poem.txt
*oses are red,
Violets are blue,
Sugar is sw##t,
And so are you.

$ # branch only if line contains 'are' and substitution of 'R' succeeds
$ sed '/are/{s/R/*/g;t}; s/e/#/g' poem.txt
*oses are red,
Viol#ts ar# blu#,
Sugar is sw##t,
And so ar# you.
```

<br>

#### <a name="overlapping-substitutions"></a>overlapping substitutions

* `t` command looping with label comes in handy for overlapping substitutions as well
* Note that in general this method will work recursively, see [stackoverflow - substitute recursively](https://stackoverflow.com/questions/9983646/sed-substitute-recursively) for example

```bash
$ # consider the problem of replacing empty columns with something
$ # case1: no consecutive empty columns - no problem
$ echo 'foo::bar::baz' | sed 's/::/:0:/g'
foo:0:bar:0:baz
$ # case2: consecutive empty columns are present - problematic
$ echo 'foo:::bar::baz' | sed 's/::/:0:/g'
foo:0::bar:0:baz

$ # t command looping will handle both cases
$ echo 'foo::bar::baz' | sed ':a s/::/:0:/; ta'
foo:0:bar:0:baz
$ echo 'foo:::bar::baz' | sed ':a s/::/:0:/; ta'
foo:0:0:bar:0:baz
```

<br>

## <a name="lines-between-two-regexps"></a>Lines between two REGEXPs

* Simple cases were seen in [地址范围](#地址范围) section
* This section will deal with more cases and some corner cases

<br>

#### <a name="include-or-exclude-matching-regexps"></a>Include or Exclude matching REGEXPs

Consider the sample input file, for simplicity the two REGEXPs are **BEGIN** and **END** strings instead of regular expressions

```bash
$ cat range.txt
foo
BEGIN
1234
6789
END
bar
BEGIN
a
b
c
END
baz
```

First, lines between the two *正则表达式*s are to be printed

* Case 1: both starting and ending *正则表达式* part of output

```bash
$ sed -n '/BEGIN/,/END/p' range.txt
BEGIN
1234
6789
END
BEGIN
a
b
c
END
```

* Case 2: both starting and ending *正则表达式* not part of ouput

```bash
$ # remember that empty REGEXP section will reuse previously matched REGEXP
$ sed -n '/BEGIN/,/END/{//!p}' range.txt
1234
6789
a
b
c
```

* Case 3: only starting *正则表达式* part of output

```bash
$ sed -n '/BEGIN/,/END/{/END/!p}' range.txt
BEGIN
1234
6789
BEGIN
a
b
c
```

* Case 4: only ending *正则表达式* part of output

```bash
$ sed -n '/BEGIN/,/END/{/BEGIN/!p}' range.txt
1234
6789
END
a
b
c
END
```

Second, lines between the two *正则表达式*s are to be deleted

* Case 5: both starting and ending *正则表达式* not part of output

```bash
$ sed '/BEGIN/,/END/d' range.txt
foo
bar
baz
```

* Case 6: both starting and ending *正则表达式* part of output

```bash
$ # remember that empty REGEXP section will reuse previously matched REGEXP
$ sed '/BEGIN/,/END/{//!d}' range.txt
foo
BEGIN
END
bar
BEGIN
END
baz
```

* Case 7: only starting *正则表达式* part of output

```bash
$ sed '/BEGIN/,/END/{/BEGIN/!d}' range.txt
foo
BEGIN
bar
BEGIN
baz
```

* Case 8: only ending *正则表达式* part of output

```bash
$ sed '/BEGIN/,/END/{/END/!d}' range.txt
foo
END
bar
END
baz
```

<br>

#### <a name="first-or-last-block"></a>First or Last block

* Getting first block is very simple by using `q` command

```bash
$ sed -n '/BEGIN/,/END/{p;/END/q}' range.txt
BEGIN
1234
6789
END

$ # use other tricks discussed in previous section as needed
$ sed -n '/BEGIN/,/END/{//!p;/END/q}' range.txt
1234
6789
```

* To get last block, reverse the input linewise, the order of *正则表达式*s and finally reverse again

```bash
$ tac range.txt | sed -n '/END/,/BEGIN/{p;/BEGIN/q}' | tac
BEGIN
a
b
c
END

$ # use other tricks discussed in previous section as needed
$ tac range.txt | sed -n '/END/,/BEGIN/{//!p;/BEGIN/q}' | tac
a
b
c
```

* To get a specific block, say 3rd one, `awk` or `perl` would be a better choice
    * See [Specific blocks](./gnu_awk.md#specific-blocks) for `awk` examples

<br>

#### <a name="broken-blocks"></a>Broken blocks

* If there are blocks with ending *正则表达式* but without corresponding starting *正则表达式*, `sed -n '/BEGIN/,/END/p'` will suffice
* Consider the modified input file where final starting *正则表达式* doesn't have corresponding ending

```bash
$ cat broken_range.txt
foo
BEGIN
1234
6789
END
bar
BEGIN
a
b
c
baz
```

* All lines till end of file gets printed with simple use of `sed -n '/BEGIN/,/END/p'`
* The file reversing trick comes in handy here as well
* But if both kinds of broken blocks are present, further processing will be required. Better to use `awk` or `perl` in such cases
    * See [Broken blocks](./gnu_awk.md#broken-blocks) for `awk` examples

```bash
$ sed -n '/BEGIN/,/END/p' broken_range.txt
BEGIN
1234
6789
END
BEGIN
a
b
c
baz

$ tac broken_range.txt | sed -n '/END/,/BEGIN/p' | tac
BEGIN
1234
6789
END
```

* If there are multiple starting *正则表达式* but single ending *正则表达式*, the reversing trick comes handy again

```bash
$ cat uneven_range.txt
foo
BEGIN
1234
BEGIN
42
6789
END
bar
BEGIN
a
BEGIN
b
BEGIN
c
BEGIN
d
BEGIN
e
END
baz

$ tac uneven_range.txt | sed -n '/END/,/BEGIN/p' | tac
BEGIN
42
6789
END
BEGIN
e
END
```

<br>

## <a name="sed-scripts"></a>sed scripts

* `sed` commands can be placed in a file and called using `-f` option or directly executed using [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))
* See [sed manual - Some Sample Scripts](https://www.gnu.org/software/sed/manual/sed.html#Examples) for more examples
* See [sed manual - Often-Used Commands](https://www.gnu.org/software/sed/manual/sed.html#Common-Commands) for more details on using comments

```bash
$ cat script.sed
# each line is a command
/is/cfoo bar
/you/r 3.txt
/you/d
# single quotes can be used freely
s/are/'are'/g

$ sed -f script.sed poem.txt
Roses 'are' red,
Violets 'are' blue,
foo bar
3
13

$ # command line options are specified as usual
$ sed -nf script.sed poem.txt
foo bar
3
13
```

* command line options can be specified along with shebang as well as added at time of invocation
* See also [stackoverflow - usage of options along with shebang depends on lot of factors](https://stackoverflow.com/questions/4303128/how-to-use-multiple-arguments-with-a-shebang-i-e)

```bash
$ type sed
sed is /bin/sed

$ cat executable.sed
#!/bin/sed -f
/is/cfoo bar
/you/r 3.txt
/you/d
s/are/'are'/g

$ chmod +x executable.sed

$ ./executable.sed poem.txt
Roses 'are' red,
Violets 'are' blue,
foo bar
3
13

$ ./executable.sed -n poem.txt
foo bar
3
13
```

<br>

## <a name="gotchas-and-tips"></a>Gotchas and Tips

* dos style line endings

```bash
$ # no issue with unix style line ending
$ printf 'foo bar\n123 789\n' | sed -E 's/\w+$/xyz/'
foo xyz
123 xyz

$ # dos style line ending causes trouble
$ printf 'foo bar\r\n123 789\r\n' | sed -E 's/\w+$/xyz/'
foo bar
123 789

$ # can be corrected by adding \r as well to match
$ # if needed, add \r in replacement section as well
$ printf 'foo bar\r\n123 789\r\n' | sed -E 's/\w+\r$/xyz/'
foo xyz
123 xyz
```

* changing dos to unix style line ending and vice versa

```bash
$ # bash functions
$ unix2dos() { sed -i 's/$/\r/' "$@" ; }
$ dos2unix() { sed -i 's/\r$//' "$@" ; }

$ cat -A 5.txt
five$
1five$

$ unix2dos 5.txt
$ cat -A 5.txt
five^M$
1five^M$

$ dos2unix 5.txt
$ cat -A 5.txt
five$
1five$
```

* variable/命令替换
* See also [stackoverflow - Is it possible to escape regex metacharacters reliably with sed](https://stackoverflow.com/questions/29613304/is-it-possible-to-escape-regex-metacharacters-reliably-with-sed)

```bash
$ # variables don't get expanded within single quotes
$ printf 'user\nhome\n' | sed '/user/ s/$/: $USER/'
user: $USER
home
$ printf 'user\nhome\n' | sed '/user/ s/$/: '"$USER"'/'
user: learnbyexample
home

$ # variable being substituted cannot have the delimiter character
$ printf 'user\nhome\n' | sed '/home/ s/$/: '"$HOME"'/'
sed: -e expression #1, char 15: unknown option to `s'
$ printf 'user\nhome\n' | sed '/home/ s#$#: '"$HOME"'#'
user
home: /home/learnbyexample

$ # use r command for robust insertion from file/command-output
$ sed '1a'"$(seq 2)" 5.txt
sed: -e expression #1, char 5: missing command
$ seq 2 | sed '1r /dev/stdin' 5.txt
five
1
2
1five
```

* common regular expression mistakes #1 - greediness

```bash
$ s='foo and bar and baz land good'
$ echo "$s" | sed 's/foo.*ba/123 789/'
123 789z land good

$ # use a more restrictive version
$ echo "$s" | sed -E 's/foo \w+ ba/123 789/'
123 789r and baz land good

$ # or use a tool with non-greedy feature available
$ echo "$s" | perl -pe 's/foo.*?ba/123 789/'
123 789r and baz land good

$ # for single characters, use negated character class
$ echo 'foo=123,baz=789,xyz=42' | sed 's/foo=.*,//'
xyz=42
$ echo 'foo=123,baz=789,xyz=42' | sed 's/foo=[^,]*,//'
baz=789,xyz=42
```

* common regular expression mistakes #2 - BRE vs ERE syntax

```bash
$ # + needs to be escaped with BRE or enable ERE
$ echo 'like 42 and 37' | sed 's/[0-9]+/xxx/g'
like 42 and 37
$ echo 'like 42 and 37' | sed -E 's/[0-9]+/xxx/g'
like xxx and xxx

$ # or escaping when not required
$ echo 'get {} and let' | sed 's/\{\}/[]/'
sed: -e expression #1, char 10: Invalid preceding regular expression
$ echo 'get {} and let' | sed 's/{}/[]/'
get [] and let
```

* common regular expression mistakes #3 - using PCRE syntax/features
    * especially by trying out solution on online sites like [regex101](https://regex101.com/) and expecting it to work with `sed` as well

```bash
$ # \d is not available as backslash character class, will match 'd' instead
$ echo 'like 42 and 37' | sed -E 's/\d+/xxx/g'
like 42 anxxx 37
$ echo 'like 42 and 37' | sed -E 's/[0-9]+/xxx/g'
like xxx and xxx

$ # features like lookarounds/non-greedy/etc not available
$ echo 'foo,baz,,xyz,,,123' | sed -E 's/,\K(?=,)/NaN/g'
sed: -e expression #1, char 16: Invalid preceding regular expression
$ echo 'foo,baz,,xyz,,,123' | perl -pe 's/,\K(?=,)/NaN/g'
foo,baz,NaN,xyz,NaN,NaN,123
```

* common regular expression mistakes #4 - end of line white-space

```bash
$ printf 'foo bar \n123 789\t\n' | sed -E 's/\w+$/xyz/'
foo bar 
123 789 

$ printf 'foo bar \n123 789\t\n' | sed -E 's/\w+\s*$/xyz/'
foo xyz
123 xyz
```

* and many more... see also
    * [unix.stackexchange - Why does my regular expression work in X but not in Y?](https://unix.stackexchange.com/questions/119905/why-does-my-regular-expression-work-in-x-but-not-in-y)
    * [stackoverflow - Greedy vs. Reluctant vs. Possessive Quantifiers](https://stackoverflow.com/questions/5319840/greedy-vs-reluctant-vs-possessive-Quantifiers)
    * [stackoverflow - How to replace everything between but only until the first occurrence of the end string?](https://stackoverflow.com/questions/45168607/how-to-replace-everything-between-but-only-until-the-first-occurrence-of-the-end)
    * [stackoverflow - How to match a specified pattern with multiple possibilities](https://stackoverflow.com/questions/43650926/how-to-match-a-specified-pattern-with-multiple-possibilities)
    * [stackoverflow - mixing different regex syntax](https://stackoverflow.com/questions/45389684/cant-comment-a-line-in-my-cnf/45389833#45389833)
    * [sed manual - BRE-vs-ERE](https://www.gnu.org/software/sed/manual/sed.html#BRE-vs-ERE)

* Speed boost for ASCII encoded input

```bash
$ time sed -nE '/^([a-d][r-z]){3}$/p' /usr/share/dict/words
avatar
awards
cravat

real    0m0.058s
$ time LC_ALL=C sed -nE '/^([a-d][r-z]){3}$/p' /usr/share/dict/words
avatar
awards
cravat

real    0m0.038s

$ time sed -nE '/^([a-z]..)\1$/p' /usr/share/dict/words > /dev/null

real    0m0.111s
$ time LC_ALL=C sed -nE '/^([a-z]..)\1$/p' /usr/share/dict/words > /dev/null

real    0m0.073s
```

<br>

## <a name="further-reading"></a>Further Reading

* Manual and related
    * `man sed` and `info sed` for more details, known issues/limitations as well as options/commands not covered in this tutorial
    * [GNU sed manual](https://www.gnu.org/software/sed/manual/sed.html) has even more detailed information and examples
    * [sed FAQ](http://sed.sourceforge.net/sedfaq.html), but last modified '10 March 2003'
    * [stackoverflow - BSD/macOS Sed vs GNU Sed vs the POSIX Sed specification](https://stackoverflow.com/questions/24275070/sed-not-giving-me-correct-substitute-operation-for-newline-with-mac-difference/24276470#24276470)
    * [unix.stackexchange - Differences between sed on Mac OSX and other standard sed](https://unix.stackexchange.com/questions/13711/differences-between-sed-on-mac-osx-and-other-standard-sed)
* Tutorials and Q&A
    * [sed basics](https://code.snipcademy.com/tutorials/shell-scripting/sed/introduction)
    * [sed detailed tutorial](http://www.grymoire.com/Unix/Sed.html) - has details on differences between various `sed` versions as well
    * [sed one-liners explained](http://www.catonmat.net/series/sed-one-liners-explained)
    * [cheat sheet](http://www.catonmat.net/download/sed.stream.editor.cheat.sheet.txt)
    * [unix.stackexchange - common search and replace examples](https://unix.stackexchange.com/questions/112023/how-can-i-replace-a-string-in-a-files)
    * [sed Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/sed?sort=votes&pageSize=15)
    * [sed Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/sed?sort=votes&pageSize=15)
* Selected examples - portable solutions, commands not covered in this tutorial, same problem solved using different tools, etc
    * [unix.stackexchange - replace multiline string](https://unix.stackexchange.com/questions/26284/how-can-i-use-sed-to-replace-a-multi-line-string)
    * [stackoverflow - deleting empty lines with optional white spaces](https://stackoverflow.com/questions/16414410/delete-empty-lines-using-sed)
    * [unix.stackexchange - print only line above the matching line](https://unix.stackexchange.com/questions/264489/find-each-line-matching-a-pattern-but-print-only-the-line-above-it)
    * [stackoverflow - How to select lines between two patterns?](https://stackoverflow.com/questions/38972736/how-to-select-lines-between-two-patterns)
    * [stackoverflow - get lines between two patterns only if there is third pattern between them](https://stackoverflow.com/questions/39960075/bash-how-to-get-lines-between-patterns-only-if-there-is-pattern2-between-them)
        * [unix.stackexchange - similar example](https://unix.stackexchange.com/questions/228699/sed-print-lines-matched-by-a-pattern-range-if-one-line-matches-a-condition)
* Learn 正则表达式 (has information on flavors other than BRE/ERE too)
    * [正则表达式 Tutorial](https://www.正则表达式.info/tutorial.html)
    * [regexcrossword](https://regexcrossword.com/)
    * [stackoverflow - What does this regex mean?](https://stackoverflow.com/questions/22937618/reference-what-does-this-regex-mean)
* Related tools
    * [sedsed - Debugger, indenter and HTMLizer for sed scripts](https://github.com/aureliojargas/sedsed)
    * [xo - composes regular expression match groups](https://github.com/ezekg/xo)
* [unix.stackexchange - When to use grep, sed, awk, perl, etc](https://unix.stackexchange.com/questions/303044/when-to-use-grep-less-awk-sed)
