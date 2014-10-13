##Python学习笔记之简明Python教程快速学习（二）

#####From JiaYing.Cheng

---
---
###7. 模块

```
 #!/usr/bin/python
 # Filename: using_sys.py

import sys

print 'The command line arguments are:'
for i in sys.argv:
    print i

print '\n\nThe PYTHONPATH is', sys.path, '\n'
```

首先，我们利用import语句 输入 sys模块。基本上，这句语句告诉Python，我们想要使用这个模块。sys模块包含了与Python解释器和它的环境有关的函数。

当Python执行import sys语句的时候，它在sys.path变量中所列目录中寻找sys.py模块。如果找到了这个文件，这个模块的主块中的语句将被运行，然后这个模块将能够被你 使用 。注意，初始化过程仅在我们 第一次 输入模块的时候进行。另外，“sys”是“system”的缩写。

sys模块中的argv变量通过使用点号指明——sys.argv——这种方法的一个优势是这个名称不会与任何在你的程序中使用的argv变量冲突。另外，它也清晰地表明了这个名称是sys模块的一部分。

sys.argv变量是一个字符串的 列表 （列表会在后面的章节详细解释）。特别地，sys.argv包含了 命令行参数 的列表，即使用命令行传递给你的程序的参数。

如果你使用IDE编写运行这些程序，请在菜单里寻找一个指定程序的命令行参数的方法。

这里，当我们执行python using_sys.py we are arguments的时候，我们使用python命令运行using_sys.py模块，后面跟着的内容被作为参数传递给程序。Python为我们把它存储在sys.argv变量中。

记住，脚本的名称总是sys.argv列表的第一个参数。所以，在这里，'using_sys.py'是sys.argv[0]、'we'是sys.argv[1]、'are'是sys.argv[2]以及'arguments'是sys.argv[3]。注意，Python从0开始计数，而非从1开始。

sys.path包含输入模块的目录名列表。我们可以观察到sys.path的第一个字符串是空的——这个空的字符串表示当前目录也是sys.path的一部分，这与PYTHONPATH环境变量是相同的。这意味着你可以直接输入位于当前目录的模块。否则，你得把你的模块放在sys.path所列的目录之一。

######字节编译的.pyc文件

输入一个模块相对来说是一个比较费时的事情，所以Python做了一些技巧，以便使输入模块更加快一些。一种方法是创建 字节编译的文件 ，这些文件以.pyc作为扩展名。字节编译的文件与Python变换程序的中间状态有关。当你在下次从别的程序输入这个模块的时候，.pyc文件是十分有用的——它会快得多，因为一部分输入模块所需的处理已经完成了。另外，这些字节编译的文件也是与平台无关的。

######from..import语句

如果你想要直接输入argv变量到你的程序中（避免在每次使用它时打sys.），那么你可以使用from sys import argv语句。如果你想要输入所有sys模块使用的名字，那么你可以使用from sys import *语句。这对于所有模块都适用。一般说来，应该避免使用from..import而使用import语句，因为这样可以使你的程序更加易读，也可以避免名称的冲突。


######模块的__name__

每个模块都有一个名称，在模块中可以通过语句来找出模块的名称。这在一个场合特别有用——就如前面所提到的，当一个模块被第一次输入的时候，这个模块的主块将被运行。假如我们只想在程序本身被使用的时候运行主块，而在它被别的模块输入的时候不运行主块，我们该怎么做呢？这可以通过模块的\__name__属性完成。

```
 #!/usr/bin/python
 # Filename: using_name.py

if __name__ == '__main__':
    print 'This program is being run by itself'
else:
    print 'I am being imported from another module'
```

```
$ python using_name.py
This program is being run by itself

$ python
>>> import using_name
I am being imported from another module
>>>
```

每个Python模块都有它的 \__name__ ，如果它是'\__main__'，这说明这个模块被用户单独运行，我们可以进行相应的恰当操作。

######制造你自己的模块

```
 #!/usr/bin/python
 # Filename: mymodule.py

def sayhi():
    print 'Hi, this is mymodule speaking.'

version = '0.1'

 # End of mymodule.py
```

注意我们使用了相同的点号来使用模块的成员。Python很好地重用了相同的记号来，使我们这些Python程序员不需要不断地学习新的方法。

from..import

下面是一个使用from..import语法的版本。

```
 #!/usr/bin/python
 # Filename: mymodule_demo2.py

from mymodule import sayhi, version
 # Alternative:
 # from mymodule import *

sayhi()
print 'Version', version
```

######dir()函数

你可以使用内建的dir函数来列出模块定义的标识符。标识符有函数、类和变量。

当你为dir()提供一个模块名的时候，它返回模块定义的名称列表。如果不提供参数，它返回当前模块中定义的名称列表。

为了观察dir的作用，我们定义一个新的变量a并且给它赋一个值，然后检验dir，我们观察到在列表中增加了以上相同的值。我们使用del语句删除当前模块中的变量/属性，这个变化再一次反映在dir的输出中。

关于del的一点注释——这个语句在运行后被用来 删除 一个变量/名称。在这个例子中，del a，你将无法再使用变量a——它就好像从来没有存在过一样。

###8. 数据结构
在Python中有三种内建的数据结构——列表、元组和字典。


######列表

list是处理一组有序项目的数据结构，即你可以在一个列表中存储一个 序列 的项目。

列表中的项目应该包括在方括号中，这样Python就知道你是在指明一个列表。一旦你创建了一个列表，你可以添加、删除或是搜索列表中的项目。由于你可以增加或删除项目，我们说列表是 可变的 数据类型，即这种类型是可以被改变的。

***对象与类的快速入门***

尽管我一直推迟讨论对象和类，但是现在对它们做一点解释可以使你更好的理解列表。我们会在相应的章节详细探索这个主题。

列表是使用对象和类的一个例子。当你使用变量i并给它赋值的时候，比如赋整数5，你可以认为你创建了一个类（类型）int的对象（实例）i。事实上，你可以看一下help(int)以更好地理解这一点。

类也有方法，即仅仅为类而定义地函数。仅仅在你有一个该类的对象的时候，你才可以使用这些功能。例如，Python为list类提供了append方法，这个方法让你在列表尾添加一个项目。例如mylist.append('an item')列表mylist中增加那个字符串。注意，使用点号来使用对象的方法。

一个类也有域，它是仅仅为类而定义的变量。仅仅在你有一个该类的对象的时候，你才可以使用这些变量/名称。类也通过点号使用，例如mylist.field。

```
 #!/usr/bin/python
 # Filename: using_list.py

 # This is my shopping list
shoplist = ['apple', 'mango', 'carrot', 'banana']

print 'I have', len(shoplist),'items to purchase.'

print 'These items are:', # Notice the comma at end of the line
for item in shoplist:
    print item,

print '\nI also have to buy rice.'
shoplist.append('rice')
print 'My shopping list is now', shoplist

print 'I will sort my list now'
shoplist.sort()
print 'Sorted shopping list is', shoplist

print 'The first item I will buy is', shoplist[0]
olditem = shoplist[0]
del shoplist[0]
print 'I bought the', olditem
print 'My shopping list is now', shoplist
```

**注意，我们在print语句的结尾使用了一个 逗号 来消除每个print语句自动打印的换行符。这样做有点难看，不过确实简单有效。**

接下来，我们使用append方法在列表中添加了一个项目，就如前面已经讨论过的一样。然后我们通过打印列表的内容来检验这个项目是否确实被添加进列表了。打印列表只需简单地把列表传递给print语句，我们可以得到一个整洁的输出。

再接下来，我们使用列表的sort方法来对列表排序。需要理解的是，这个方法影响列表本身，而不是返回一个修改后的列表——这与字符串工作的方法不同。**这就是我们所说的列表是 可变的 而字符串是 不可变的 。**

######元组

元组和列表十分类似，只不过元组和字符串一样是 不可变的 即你不能修改元组。元组通过圆括号中用逗号分割的项目定义。元组通常用在使语句或用户定义的函数能够安全地采用一组值的时候，即被使用的元组的值不会改变。

```
 #!/usr/bin/python
 # Filename: using_tuple.py

zoo = ('wolf', 'elephant', 'penguin')
print 'Number of animals in the zoo is', len(zoo)

new_zoo = ('monkey', 'dolphin', zoo)
print 'Number of animals in the new zoo is', len(new_zoo)
print 'All animals in new zoo are', new_zoo
print 'Animals brought from old zoo are', new_zoo[2]
print 'Last animal brought from old zoo is', new_zoo[2][2]
```

我们看到len函数可以用来获取元组的长度。这也表明元组也是一个序列。

注意元组之内的元组不会失去它的身份。

我们可以通过一对方括号来指明某个项目的位置从而来访问元组中的项目，就像我们对列表的用法一样。这被称作 索引 运算符。我们使用new_zoo[2]来访问new_zoo中的第三个项目。我们使用new_zoo[2][2]来访问new_zoo元组的第三个项目的第三个项目。

**含有0个或1个项目的元组。**一个空的元组由一对空的圆括号组成，如myempty = ()。然而，**含有单个元素的元组就不那么简单了。你必须在第一个（唯一一个）项目后跟一个逗号，这样Python才能区分元组和表达式中一个带圆括号的对象。**即如果你想要的是一个包含项目2的元组的时候，你应该指明**singleton = (2 , )**。

**元组与打印语句**

元组最通常的用法是用在打印语句中，下面是一个例子：

```
 #!/usr/bin/python
 # Filename: print_tuple.py

age = 22
name = 'Swaroop'

print '%s is %d years old' % (name, age)
print 'Why is %s playing with that python?' % name
```

print语句可以使用跟着%符号的项目元组的字符串。这些字符串具备定制的功能。定制让输出满足某种特定的格式。定制可以是%s表示字符串或%d表示整数。元组必须按照相同的顺序来对应这些定制。

在第二个print语句中，我们使用了一个定制，后面跟着%符号后的单个项目——没有圆括号。这只在字符串中只有一个定制的时候有效。

######字典

字典类似于你通过联系人名字查找地址和联系人详细情况的地址簿，即，我们把键（名字）和值（详细情况）联系在一起。注意，键必须是唯一的，就像如果有两个人恰巧同名的话，你无法找到正确的信息。

注意，你只能使用不可变的对象（比如字符串）来作为字典的键，但是你可以不可变或可变的对象作为字典的值。基本说来就是，你应该只使用简单的对象作为键。

键值对在字典中以这样的方式标记：d = {key1 : value1, key2 : value2 }。注意它们的键/值对用冒号分割，而各个对用逗号分割，所有这些都包括在花括号中。

记住字典中的键/值对是没有顺序的。如果你想要一个特定的顺序，那么你应该在使用前自己对它们排序。

字典是dict类的实例/对象。

```
 #!/usr/bin/python
 # Filename: using_dict.py

 # 'ab' is short for 'a'ddress'b'ook

ab = {       'Swaroop'   : 'swaroopch@byteofpython.info',
             'Larry'     : 'larry@wall.org',
             'Matsumoto' : 'matz@ruby-lang.org',
             'Spammer'   : 'spammer@hotmail.com'
     }

print "Swaroop's address is %s" % ab['Swaroop']

 # Adding a key/value pair
ab['Guido'] = 'guido@python.org'

 # Deleting a key/value pair
del ab['Spammer']

print '\nThere are %d contacts in the address-book\n' % len(ab)
for name, address in ab.items():
    print 'Contact %s at %s' % (name, address)

if 'Guido' in ab: # OR ab.has_key('Guido')
    print "\nGuido's address is %s" % ab['Guido']

```

我们使用已经介绍过的标记创建了字典ab。然后我们使用在列表和元组章节中已经讨论过的索引操作符来指定键，从而使用键/值对。我们可以看到字典的语法同样十分简单。

我们可以使用索引操作符来寻址一个键并为它赋值，这样就增加了一个新的键/值对，就像在上面的例子中我们对Guido所做的一样。

我们可以使用我们的老朋友——del语句来删除键/值对。我们只需要指明字典和用索引操作符指明要删除的键，然后把它们传递给del语句就可以了。执行这个操作的时候，我们无需知道那个键所对应的值。

接下来，我们使用字典的items方法，来使用字典中的每个键/值对。这会返回一个元组的列表，其中每个元组都包含一对项目——键与对应的值。我们抓取这个对，然后分别赋给for..in循环中的变量name和address然后在for－块中打印这些值。

我们可以使用in操作符来检验一个键/值对是否存在，或者使用dict类的has_key方法。你可以使用help(dict)来查看dict类的完整方法列表。

关键字参数与字典。如果换一个角度看待你在函数中使用的关键字参数的话，你已经使用了字典了！只需想一下——你在函数定义的参数列表中使用的键/值对。当你在函数中使用变量的时候，它只不过是使用一个字典的键（这在编译器设计的术语中被称作 符号表 ）。

######参考

当你创建一个对象并给它赋一个变量的时候，这个变量仅仅 参考 那个对象，而不是表示这个对象本身！也就是说，变量名指向你计算机中存储那个对象的内存。这被称作名称到对象的绑定。

```
 #!/usr/bin/python
 # Filename: reference.py

print 'Simple Assignment'
shoplist = ['apple', 'mango', 'carrot', 'banana']
mylist = shoplist # mylist is just another name pointing to the same object!

del shoplist[0]

print 'shoplist is', shoplist
print 'mylist is', mylist
 # notice that both shoplist and mylist both print the same list without
 # the 'apple' confirming that they point to the same object

print 'Copy by making a full slice'
mylist = shoplist[:] # make a copy by doing a full slice
del mylist[0] # remove first item

print 'shoplist is', shoplist
print 'mylist is', mylist
 # notice that now the two lists are different
```

大多数解释已经在程序的注释中了。你需要记住的只是如果你想要复制一个列表或者类似的序列或者其他复杂的对象（不是如整数那样的简单 对象 ），那么你必须使用切片操作符来取得拷贝。如果你只是想要使用另一个变量名，两个名称都 参考 同一个对象，那么如果你不小心的话，可能会引来各种麻烦。


### 9. 解决问题——编写一个Python脚本

问题

我提出的问题是： 我想要一个可以为我的所有重要文件创建备份的程序。

尽管这是一个简单的问题，但是问题本身并没有给我们足够的信息来解决它。进一步的分析是必需的。例如，我们如何确定该备份哪些文件？备份保存在哪里？我们怎么样存储备份？

在恰当地分析了这个问题之后，我们开始设计我们的程序。我们列了一张表，表示我们的程序应该如何工作。对于这个问题，我已经创建了下面这个列表以说明 我 如何让它工作。如果是你设计的话，你可能不会这样来解决问题——每个人都有其做事的方法，这很正常。

1. 需要备份的文件和目录由一个列表指定。

2. 备份应该保存在主备份目录中。

3. 文件备份成一个zip文件。

4. zip存档的名称是当前的日期和时间。

5. 我们使用标准的zip命令，它通常默认地随Linux/Unix发行版提供。Windows用户可以使用Info-Zip程序。注意你可以使用任何地存档命令，只要它有命令行界面就可以了，那样的话我们可以从我们的脚本中传递参数给它。

```
 #!/usr/bin/python
 # Filename: backup_ver4.py

import os
import time
 # 1. The files and directories to be backed up are specified in a list.
source = ['/home/swaroop/byte', '/home/swaroop/bin']
 # If you are using Windows, use source = [r'C:\Documents', r'D:\Work'] or something like that

 # 2. The backup must be stored in a main backup directory
target_dir = '/mnt/e/backup/' # Remember to change this to what you will be using

 # 3. The files are backed up into a zip file.
 # 4. The current day is the name of the subdirectory in the main directory
today = target_dir + time.strftime('%Y%m%d')
 # The current time is the name of the zip archive
now = time.strftime('%H%M%S')

 # Take a comment from the user to create the name of the zip file
comment = raw_input('Enter a comment --> ')
if len(comment) == 0: # check if a comment was entered
    target = today + os.sep + now + '.zip'
else:
    target = today + os.sep + now + '_' + \
        comment.replace(' ', '_') + '.zip'
    # Notice the backslash!

 # Create the subdirectory if it isn't already there
if not os.path.exists(today):
    os.mkdir(today) # make directory
    print 'Successfully created directory', today

 # 5. We use the zip command (in Unix/Linux) to put the files in a zip archive
zip_command = "zip -qr '%s' %s" % (target, ' '.join(source))

 # Run the backup
if os.system(zip_command) == 0:
    print 'Successful backup to', target
else:
    print 'Backup FAILED'
```

######进一步优化
它仍然有进一步改进的空间。比如，你可以在程序中包含 交互 程度——你可以用-v选项来使你的程序更具交互性。

另一个可能的改进是使文件和目录能够通过命令行直接传递给脚本。我们可以通过sys.argv列表来获取它们，然后我们可以使用list类提供的extend方法把它们加到source列表中去。

我还希望有的一个优化是使用tar命令替代zip命令。这样做的一个优势是在你结合使用tar和gzip命令的时候，备份会更快更小。如果你想要在Windows中使用这些归档，WinZip也能方便地处理这些.tar.gz文件。tar命令在大多数Linux/Unix系统中都是默认可用的。Windows用户也可以下载安装它。

命令字符串现在将称为：

```
tar = 'tar -cvzf %s %s -X /home/swaroop/excludes.txt' % (target, ' '.join(srcdir))
```
选项解释如下：

-c表示创建一个归档。

-v表示交互，即命令更具交互性。

-z表示使用gzip滤波器。

-f表示强迫创建归档，即如果已经有一个同名文件，它会被替换。

-X表示含在指定文件名列表中的文件会被排除在备份之外。例如，你可以在文件中指定*~，从而不让备份包括所有以~结尾的文件。

***重要***

最理想的创建这些归档的方法是分别使用zipfile和tarfile。它们是Python标准库的一部分，可以供你使用。***使用这些库就避免了使用os.system这个不推荐使用的函数，它容易引发严重的错误。***
然而，我在本节中使用os.system的方法来创建备份，这纯粹是为了教学的需要。这样的话，例子就可以简单到让每个人都能够理解，同时也已经足够用了。