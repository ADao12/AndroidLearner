##Python学习笔记之简明Python教程快速学习（四）

#####From JiaYing.Cheng

---
---
###11. 输入/输出

在很多时候，你会想要让你的程序与用户（可能是你自己）交互。你会从用户那里得到输入，然后打印一些结果。我们可以分别使用raw_input和print语句来完成这些功能。对于输出，你也可以使用多种多样的str（字符串）类。例如，你能够使用rjust方法来得到一个按一定宽度右对齐的字符串。利用help(str)获得更多详情。

另一个常用的输入/输出类型是处理文件。创建、读和写文件的能力是许多程序所必需的，我们将会在这章探索如何实现这些功能。

#######文件

你可以通过创建一个file类的对象来打开一个文件，分别使用file类的read、readline或write方法来恰当地读写文件。对文件的读写能力依赖于你在打开文件时指定的模式。最后，当你完成对文件的操作的时候，你调用close方法来告诉Python我们完成了对文件的使用。

```
 #!/usr/bin/python
 # Filename: using_file.py

poem = '''\
Programming is fun
When the work is done
if you wanna make your work also fun:
        use Python!
'''

f = file('poem.txt', 'w') # open for 'w'riting
f.write(poem) # write text to file
f.close() # close the file

f = file('poem.txt')
 # if no mode is specified, 'r'ead mode is assumed by default
while True:
    line = f.readline()
    if len(line) == 0: # Zero length indicates EOF
        break
    print line,
    # Notice comma to avoid automatic newline added by Python
f.close() # close the file
```
首先，我们通过指明我们希望打开的文件和模式来创建一个file类的实例。模式可以为读模式（'r'）、写模式（'w'）或追加模式（'a'）。事实上还有多得多的模式可以使用，你可以使用help(file)来了解它们的详情。

我们首先用写模式打开文件，然后使用file类的write方法来写文件，最后我们用close关闭这个文件。

接下来，我们再一次打开同一个文件来读文件。如果我们没有指定模式，读模式会作为默认的模式。在一个循环中，我们使用readline方法读文件的每一行。这个方法返回包括行末换行符的一个完整行。所以，当一个 空的 字符串被返回的时候，即表示文件末已经到达了，于是我们停止循环。

注意，因为从文件读到的内容已经以换行符结尾，所以我们在print语句上使用逗号来消除自动换行。最后，我们用close关闭这个文件。

现在，来看一下poem.txt文件的内容来验证程序确实工作正常了

######储存器

Python提供一个标准的模块，称为pickle。使用它你可以在一个文件中储存任何Python对象，之后你又可以把它完整无缺地取出来。这被称为 持久地 储存对象。

还有另一个模块称为cPickle，它的功能和pickle模块完全相同，只不过它是用C语言编写的，因此要快得多（比pickle快1000倍）。你可以使用它们中的任一个，而我们在这里将使用cPickle模块。记住，我们把这两个模块都简称为pickle模块。

```
 #!/usr/bin/python
 # Filename: pickling.py

import cPickle as p
 #import pickle as p

shoplistfile = 'shoplist.data'
 # the name of the file where we will store the object

shoplist = ['apple', 'mango', 'carrot']

 # Write to the file
f = file(shoplistfile, 'w')
p.dump(shoplist, f) # dump the object to a file
f.close()

del shoplist # remove the shoplist

 # Read back from the storage
f = file(shoplistfile)
storedlist = p.load(f)
print storedlist
```
首先，请注意我们使用了import..as语法。这是一种便利方法，以便于我们可以使用更短的模块名称。在这个例子中，它还让我们能够通过简单地改变一行就切换到另一个模块（cPickle或者pickle）！在程序的其余部分的时候，我们简单地把这个模块称为p。

为了在文件里储存一个对象，首先以写模式打开一个file对象，然后调用储存器模块的dump函数，把对象储存到打开的文件中。这个过程称为 储存 。

接下来，我们使用pickle模块的load函数的返回来取回对象。这个过程称为 取储存 。

###12. 异常
当你的程序中出现某些 异常的 状况的时候，异常就发生了。例如，当你想要读某个文件的时候，而那个文件不存在。或者在程序运行的时候，你不小心把它删除了。上述这些情况可以使用异常来处理。

假如你的程序中有一些无效的语句，会怎么样呢？Python会引发并告诉你那里有一个错误，从而处理这样的情况。

######try..except

```
 #!/usr/bin/python
 # Filename: try_except.py

import sys

try:
    s = raw_input('Enter something --> ')
except EOFError:
    print '\nWhy did you do an EOF on me?'
    sys.exit() # exit the program
except:
    print '\nSome error/exception occurred.'
    # here, we are not exiting the program

print 'Done'
```
我们把所有可能引发错误的语句放在try块中，然后在except从句/块中处理所有的错误和异常。except从句可以专门处理单一的错误或异常，或者一组包括在圆括号内的错误/异常。如果没有给出错误或异常的名称，它会处理 所有的 错误和异常。对于每个try从句，至少都有一个相关联的except从句。

如果某个错误或异常没有被处理，默认的Python处理器就会被调用。它会终止程序的运行，并且打印一个消息，我们已经看到了这样的处理。

你还可以让try..catch块关联上一个else从句。当没有异常发生的时候，else从句将被执行。

我们还可以得到异常对象，从而获取更多有个这个异常的信息。这会在下一个例子中说明。

######引发异常

你可以使用raise语句 引发 异常。你还得指明错误/异常的名称和伴随异常 触发的 异常对象。你可以引发的错误或异常应该分别是一个Error或Exception类的直接或间接导出类。

```
 #!/usr/bin/python
 # Filename: raising.py

class ShortInputException(Exception):
    '''A user-defined exception class.'''
    def __init__(self, length, atleast):
        Exception.__init__(self)
        self.length = length
        self.atleast = atleast

try:
    s = raw_input('Enter something --> ')
    if len(s) < 3:
        raise ShortInputException(len(s), 3)
    # Other work can continue as usual here
except EOFError:
    print '\nWhy did you do an EOF on me?'
except ShortInputException, x:
    print 'ShortInputException: The input was of length %d, \
          was expecting at least %d' % (x.length, x.atleast)
else:
    print 'No exception was raised.'
```

这里，我们创建了我们自己的异常类型，其实我们可以使用任何预定义的异常/错误。这个新的异常类型是ShortInputException类。它有两个域——length是给定输入的长度，atleast则是程序期望的最小长度。

在except从句中，我们提供了错误类和用来表示错误/异常对象的变量。这与函数调用中的形参和实参概念类似。在这个特别的except从句中，我们使用异常对象的length和atleast域来为用户打印一个恰当的消息。

######try..finally

假如你在读一个文件的时候，希望在无论异常发生与否的情况下都关闭文件，该怎么做呢？这可以使用finally块来完成。注意，在一个try块下，你可以同时使用except从句和finally块。如果你要同时使用它们的话，需要把一个嵌入另外一个。

```
 #!/usr/bin/python
 # Filename: finally.py

import time

try:
    f = file('poem.txt')
    while True: # our usual file-reading idiom
        line = f.readline()
        if len(line) == 0:
            break
        time.sleep(2)
        print line,
finally:
    f.close()
    print 'Cleaning up...closed the file'
```

我们进行通常的读文件工作，但是我有意在每打印一行之前用time.sleep方法暂停2秒钟。这样做的原因是让程序运行得慢一些（Python由于其本质通常运行得很快）。在程序运行的时候，按Ctrl-c中断/取消程序。

我们可以观察到KeyboardInterrupt异常被触发，程序退出。但是在程序退出之前，finally从句仍然被执行，把文件关闭


###13. Python标准库

Python标准库是随Python附带安装的，它包含大量极其有用的模块。熟悉Python标准库是十分重要的，因为如果你熟悉这些库中的模块，那么你的大多数问题都可以简单快捷地使用它们来解决。

我们已经研究了一些这个库中的常用模块。你可以在Python附带安装的文档的“库参考”一节中了解Python标准库中所有模块的完整内容。

######sys模块
sys模块包含系统对应的功能。我们已经学习了sys.argv列表，它包含命令行参数。

```
 #!/usr/bin/python
 # Filename: cat.py

import sys

def readfile(filename):
    '''Print a file to the standard output.'''
    f = file(filename)
    while True:
        line = f.readline()
        if len(line) == 0:
            break
        print line, # notice comma
    f.close()

 # Script starts from here
if len(sys.argv) < 2:
    print 'No action specified.'
    sys.exit()

if sys.argv[1].startswith('--'):
    option = sys.argv[1][2:]
    # fetch sys.argv[1] but without the first two characters
    if option == 'version':
        print 'Version 1.2'
    elif option == 'help':
        print '''\
This program prints files to the standard output.
Any number of files can be specified.
Options include:
  --version : Prints the version number
  --help    : Display this help'''
    else:
        print 'Unknown option.'
    sys.exit()
else:
    for filename in sys.argv[1:]:
        readfile(filename)
```

这个程序用来模范Linux/Unix用户熟悉的cat命令。你只需要指明某些文本文件的名字，这个程序会把它们打印输出。

在Python程序运行的时候，即不是在交互模式下，在sys.argv列表中总是至少有一个项目。它就是当前运行的程序名称，作为sys.argv[0]（由于Python从0开始计数）。其他的命令行参数在这个项目之后。

为了使这个程序对用户更加友好，我们提供了一些用户可以指定的选项来了解更多程序的内容。我们使用第一个参数来检验我们的程序是否被指定了选项。如果使用了--version选项，程序的版本号将被打印出来。类似地，如果指定了--help选项，我们提供一些关于程序的解释。我们使用sys.exit函数退出正在运行的程序。和以往一样，你可以看一下help(sys.exit)来了解更多详情。

如果没有指定任何选项，而是为程序提供文件名的话，它就简单地打印出每个文件地每一行，按照命令行中的顺序一个文件接着一个文件地打印。

顺便说一下，名称cat是 concatenate 的缩写，它基本上表明了程序的功能——它可以在输出打印一个文件或者把两个或两个以上文件连接/级连在一起打印。

######os模块

这个模块包含普遍的操作系统功能。如果你希望你的程序能够与平台无关的话，这个模块是尤为重要的。即它允许一个程序在编写后不需要任何改动，也不会发生任何问题，就可以在Linux和Windows下运行。一个例子就是使用os.sep可以取代操作系统特定的路径分割符。

下面列出了一些在os模块中比较有用的部分。它们中的大多数都简单明了。

- os.name字符串指示你正在使用的平台。比如对于Windows，它是'nt'，而对于Linux/Unix用户，它是'posix'。

- os.getcwd()函数得到当前工作目录，即当前Python脚本工作的目录路径。

- os.getenv()和os.putenv()函数分别用来读取和设置环境变量。

- os.listdir()返回指定目录下的所有文件和目录名。

- os.remove()函数用来删除一个文件。

- os.system()函数用来运行shell命令。

- os.linesep字符串给出当前平台使用的行终止符。例如，Windows使用'\r\n'，Linux使用'\n'而Mac使用'\r'。

- os.path.split()函数返回一个路径的目录名和文件名。

```
>>> os.path.split('/home/swaroop/byte/code/poem.txt')
('/home/swaroop/byte/code', 'poem.txt')
```
- os.path.isfile()和os.path.isdir()函数分别检验给出的路径是一个文件还是目录。类似地，os.path.existe()函数用来检验给出的路径是否真地存在。

你可以利用Python标准文档去探索更多有关这些函数和变量的详细知识。你也可以使用help(sys)等等。