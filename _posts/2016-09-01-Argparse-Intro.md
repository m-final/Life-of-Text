---
layout: post
title:  "Argparse介绍"
date:   2016-09-01
categories: jekyll update
---

[__argparse__](https://docs.python.org/2/library/argparse.html) 是Python中一款常用的命令行（CLI）解析（parsing）模块，与C-style的Paser: [__getopt__](https://docs.python.org/2/library/getopt.html) 相比更加简洁易用，是 [__optparse__](https://docs.python.org/2/library/optparse.html) 的进化版。

基础用法的介绍可以参看Python HOWTO 里的[__Argparse Tutorial__](https://docs.python.org/2/howto/argparse.html)。本文对API中最重要的三个语句进行有选择的整理；

## 简单的例子

下面用一个最简单的例子来说明argparse的用法

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("s")
parser.add_argument("-f", "--flag")
args = parser.parse_args()
print args.echo
```
其中不以dash `-` 开头的为__positional参数__，一般必填。以dash `-` 开头的为__optional参数__，可以不指定。保存代码为_prog.py_，以上代码即包含了我们最常用的三个操作
1. 创建parser
2. 添加argument（包括positiona和optional）
3. 解析parser


下面是模拟我们在CLI中面对这个陌生文件最常用的处理情境：

1. 直接运行，没有指定positional参数值，报错

```shell
$ python /Users/m_fin/Desktop/prog.py 
usage: prog.py [-h] [-f FLAG] s
prog.py: error: too few arguments
```

2. 使用 `--help` 查看文件使用方法 


```shell
$ python /Users/m_fin/Desktop/prog.py --help
usage: prog.py [-h] [-f FLAG] s

positional arguments:
  s

optional arguments:
  -h, --help            show this help message and exit
  -f FLAG, --flag FLAG
```

3. 输入正确的参数值

```shell
$ python /Users/m_fin/Desktop/prog.py foo
Namespace(flag=None, s='foo')
```

## API 精华介绍 ##

> ### class argparse.__ArgumentParser__ _(prog=None, usage=None, description=None, epilog=None, parents=[], formatter_class=argparse.HelpFormatter, prefix_chars='-', fromfile_prefix_chars=None, argument_default=None, conflict_handler='error', add_help=True)_

___prog___: 在 `—help` 的 _usage_ 后展示的名称，默认为是`sys.argv[0]`，即py程序的名称；

___description___：用来在 `—help` 的usage后面添加任何描述性语句；

___epilog___：用来在`—help`结尾处添加任何描述性语句；
```python
>>> parser = argparse.ArgumentParser(
...     description='~~~~~This is a description~~~~~',
...     epilog="+++++This is a epilog+++++")
>>> parser.print_help()
usage: argparse.py [-h]

​~~~~~This is a description~~~~~

optional arguments:
 -h, --help  show this help message and exit

+++++This is a epilog+++++
```

___parent___：用来作为parser的继承关系，注意父parser必须先被初始化；

___prefix_chars___：用来指定参数前缀，默认为dash ```-```，可以添加额外前缀；

```python
>>> parser = argparse.ArgumentParser(prog='PROG', prefix_chars='-+')
>>> parser.add_argument('+f')
>>> parser.add_argument('++bar')
>>> parser.parse_args('+f X ++bar Y'.split())
Namespace(bar='Y', f='X')
```

___conflict_handler___：用来处理当add_argument()添加冲突时的动作，将默认的`'error'`改为`'resolve'`即当冲突时适用覆盖操作；

---

> ### ArgumentParser.__add_argument__ _(name or flags...\[, action] \[, nargs]\[, const]\[, default]\[, type]\[, choices]\[, required]\[, help]\[, metavar][, dest])_

___name or flags___： 参数名。以dash`-`开头则为optional argument，否则为positional argument（name 表示参数会有值，flags表示参数只需要被设置，不需要具体设置值；

___action___：添加时的额外动作：

- ___store___：default，仅仅储存动作，最简单情况；
- ___store_const___：储存为之后被`const`参数指定的值。即常常作为flag使用；
```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', action='store_const', const=42)
>>> parser.parse_args(['--foo'])
>>> Namespace(foo=42)
```
- ___store_true___/___store_false___：`store_const` 的特例，这里指仅仅储存为Boolean值；常用，比如`store_true`就默认参数值为`False`（可被default参数覆盖），只有当指定时才为`True`；`store_false`则相反；该参数仅为指示参数，没有参数值；
```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', action='store_true')
>>> parser.add_argument('--bar', action='store_false')
>>> parser.add_argument('--baz', action='store_false')
>>> parser.parse_args('--foo --bar'.split())
>>> Namespace(bar=False, baz=True, foo=True)
```

- ___append___：允许重复的参数出现，并且将相同参数的值合并为一个list；

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', action='append')
>>> parser.parse_args('--foo 1 --foo 2'.split())
Namespace(foo=['1', '2'])
```
- ___append_const___：append的特例，将const的值合并到指定位置，位置由dest参数指定；
- ___count___：统计参数的出现次数，一般用在类似于verbose的语境；
```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--verbose', '-v', action='count')
>>> parser.parse_args(['-vvv'])
Namespace(verbose=3)
```

___nargs___：用来接受多于一个参数值的情况，产生一个list：
- ___N___：指定某个参数接受值的个数，强制必须为N个如果该参数被设置。`default`也不可用
- ___?___：出现1次或0次；0次则使用`default`值，1次则使用`const`值（如果没被指定值）

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', nargs='?', const='c', default='d')
>>> parser.add_argument('bar', nargs='?', default='d')
>>> parser.parse_args(['XX', '--foo', 'YY'])
Namespace(bar='XX', foo='YY')
>>> parser.parse_args(['XX', '--foo'])
Namespace(bar='XX', foo='c')
>>> parser.parse_args([])
Namespace(bar='d', foo='d')
```

- __*__：出现N次也可以--即所有的参数值都会被归纳为一个list
- ___+___：同上，出现N次也可以，但是必须至少有一个参数值，否则报错。

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('foo', nargs='+')
>>> parser.parse_args(['a', 'b'])
Namespace(foo=['a', 'b'])
>>> parser.parse_args([])
usage: PROG [-h] foo [foo ...]
PROG: error: too few arguments
```

- ___argparse.REMAINDER___：之后所有的参数值都被当放入list中，往往用来供之后再处理。

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('--foo')
>>> parser.add_argument('command')
>>> parser.add_argument('args', nargs=argparse.REMAINDER)
>>> print parser.parse_args('--foo B cmd --arg1 XX ZZ'.split())
Namespace(args=['--arg1', 'XX', 'ZZ'], command='cmd', foo='B')
```


___const___：供`action`指定值使用；

___default___：当参数没有被指定时使用，如果指定了`type`会随后进行类型转换；

___type___：指定存储类型；可以使用与文件相关的值比如`argparse.FileType('w')`；

___choices___：类似于enumerate，限定参数值只能为其中的某个值。可以为任意一个支持`in`操作的container，比如`dict`和`set`；
```python
>>> parser = argparse.ArgumentParser(prog='doors.py')
>>> parser.add_argument('door', type=int, choices=range(1, 4))
>>> print(parser.parse_args(['3']))
Namespace(door=3)
>>> parser.parse_args(['4'])
usage: doors.py [-h] {1,2,3}
doors.py: error: argument door: invalid choice: 4 (choose from 1, 2, 3)
```

___required___：为`True`或`False`（default），使参数必须被指定（变为类似于positional的参数）；

___help___：指定在CLI下`--help`时显示的文字，可以额外使用一些`add_argument()`中的keywords，例如`%(default)s`和`%(type)s`；另外也可以设置为`argparse.SUPPRESS`不显示 ；

___metavar___：用来设置在`--help`时输出的变量值，positional则直接改变名称，optional则改变对应参数值显示的名称（default为参数名大写）；

___dest___：决定变量在`parse_args()`调用后被返回的名称（可以理解为存储变量名称），默认情况下positional参数则为变量名；optional参数则为长变量名（double dash`--`）之后的string，如果没有长变量名，则使用短变量名（single dash`-`）之后的string。String中的所有dash都会被转换为下划线`_`；

---

> ### ArgumentParser.parse_args(args=None, namespace=None)


默认情况下接受程序外部传来的arguments，在调试时可以传入string来模拟外部输入。类似于`parser.parse_args('--foo 1 -y 2'.split())`

缩写：
- 长变量可以使用等于`=`来连接名称和值`parser.parse_args(['--foo=FOO'])`
- 短变量可以直接在后面连接值`parser.parse_args(['-xX'])`
- 多个短变量可以相连使用，只要只有最后一个变量需要传入值
```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('-x', action='store_true')
>>> parser.add_argument('-y', action='store_true')
>>> parser.add_argument('-z')
>>> parser.parse_args(['-xyzZ'])
Namespace(x=True, y=True, z='Z')
```

- 变量名可以被缩写（无论`--`or `-`），只要其中没有ambiguous，类似于tab补全；

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('-bacon')
>>> parser.add_argument('-badger')
>>> parser.parse_args('-bac MMM'.split())
Namespace(bacon='MMM', badger=None)
>>> parser.parse_args('-bad WOOD'.split())
Namespace(bacon=None, badger='WOOD')
>>> parser.parse_args('-ba BA'.split())
usage: PROG [-h] [-bacon BACON] [-badger BADGER]
PROG: error: ambiguous option: -ba could match -badger, -bacon
```

Name space：
`Args = parse_args()`返回的对象类型，可以被`vars`变为`dict`形式；在`parse_args()`的keywords中也可以指定`namespace=c`，c为任意一个`class`，则将这些args作为attributes赋值给这个对象（useful）

### 其他utilities

> #### ArgumentParser.add_subparsers(\[title]\[, description]\[, prog]\[, parser_class]\[, action]\[, option_string]\[, dest]\[, help][, metavar])

提供parser的分层服务，即将parser作为一个参数，可以用来指定执行哪个parser。不同的parser可以对应不同的help文档和默认调用等一般parser可以具有的功能。

> #### ArgumentParser.add_argument_group(title=None, description=None)

提供将arguments在`--help`时分组展示的功能，用以DIY展示界面的布局。

> #### ArgumentParser.add_mutually_exclusive_group(required=False)

一个互斥组，组内最多只有一个参数可以被设置，___required___为`True`则保证必须有一个被设置。

> #### ArgumentParser.set_defaults(**kwargs)

可以在`add_argument()`之后对default值进行设置/覆盖.
