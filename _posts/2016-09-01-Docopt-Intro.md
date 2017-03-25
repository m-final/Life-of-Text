---
layout: post
title:  "Docopt介绍"
date:   2016-09-01
categories: jekyll update
---

[Docopt](docopt.org)是一款简洁化书写CLI parser的工具包，可以以书写doc文档的方式决定对参数的parsing。在Github上有超过4k的star。但是它可能要求你对Python自带的parser比较熟悉，学习之前可以先了解argparse，以及其对应的在CLI时`—help`时的输出格式。

```python
"""Naval Fate.

Usage:
  naval_fate.py ship new <name>...
  naval_fate.py ship <name> move <x> <y> [--speed=<kn>]
  naval_fate.py ship shoot <x> <y>
  naval_fate.py mine (set|remove) <x> <y> [--moored | --drifting]
  naval_fate.py (-h | --help)
  naval_fate.py --version
  naval_fate.py PATH
Arguments:
  Path destination path

Options:
  -h --help     Show this screen.
  --version     Show version.
  --speed=<kn>  Speed in knots [default: 10].
  --moored      Moored (anchored) mine.
  --drifting    Drifting mine.

"""
from docopt import docopt


if __name__ == '__main__':
    arguments = docopt(__doc__, version='Naval Fate 2.0')
    print(arguments)
```



以上就是一个最普遍的使用情境。将需要的参数以doc的形式写在Python文件的开始，这也是Python Community建议的形式，形式与`—help`的输出一致，之后docopt会自动解析，再返回一个与argparse的namespace近似的对象.

在main函数中，使用docopt对上面书写的\__doc__进行解析即得到需要的parser结构。

在doc中，__Usage__是必选项，作为docopt的识别标志。其中

- 方括号`[]`代表optional参数
- `()`为positional参数，`<>`为参数值
- `|`为mutual exclusive group
- `…`代表多个参数列表
- `[—]`double dash代表长名称
- `[-]`single dash代表短名称

### 因为其书写简单，且与help输出保持一致，容易理解，所以受到热捧，具体详细用法可以参看首行超链的官方文档

尝试心得：
1. 当输入不符合规范时，会将`--help`的输出打印出来；
2. options(arguments)的参数，与Usage的参数，没有对应关系，只要在任一中出现的argument都可以被识别。但是写在除Usage之外可以增加可读性。
3. default的类型默认为string。暂时没有找到像argparse的设置默认值为boolean的方式，一般采用设置optional的类似flag的解决办法，不设则默认为`False`。
4. Doc中除了规定的语法规则之外(argument以dash起等)，可以自由编辑，比如增加类似于注释`#`的内容。

