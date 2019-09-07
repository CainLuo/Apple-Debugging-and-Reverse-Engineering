# Chapter 3: Attaching with LLDB

现在您已经了解了两个最基本的命令，帮助和apropos，现在是时候研究LLDB如何将自己附加到流程上了。 您将学习使用各种选项将LLDB附加到进程的所有不同方法，以及附加到进程后幕后发生的情况。

LLDB“附加”的短语实际上有点误导。 名为debugserver的程序（在Xcode.app/Contents/SharedFrameworks/LLDB.framework/ Resources /中找到）负责附加到目标进程。

如果它是远程进程，例如在远程设备上运行的iOS，watchOS或tvOS应用程序，则会在该远程设备上启动远程调试服务器。 LLDB的工作是启动，连接和协调调试服务器，以处理调试应用程序时的所有交互。

## 附加到现有流程
正如您在第1章中已经看到的那样，您可以附加到这样的过程：

```vim
lldb -n Xcode
```

但是，还有其他方法可以做同样的事情。 您可以通过提供正在运行的程序的进程标识符或PID来附加到Xcode。

> 注意：只需提醒一下，如果您未在macOS计算机上禁用SIP，则无法将LLDB附加到Apple应用程序。 如果在应用程序中没有反调试技术，从Mojave版本10.14.1开始，仍然可以连接到第三方应用程序（甚至可以从App Store！）。

打开Xcode，然后打开一个新的终端会话，最后运行以下命令：

```vim
pgrep -x Xcode
```

这将输出Xcode进程的PID。

接下来，运行以下命令，将89944替换为上面命令的输出数字：

```vim
lldb -p 89944
```

这告诉LLDB使用给定的PID附加到进程。 在这种情况下，这是您正在运行的Xcode进程。

## 附加到未来的过程
上一个命令仅解决正在运行的进程。 如果Xcode未运行，或者已经连接到调试器，则先前的命令将失败。 如果你还不知道PID，怎么能抓住即将发布的进程呢？

您可以使用-w参数执行此操作，这会导致LLDB等待，直到进程启动时PID或可执行文件名与使用-p或-n参数提供的条件匹配。

例如，在终端窗口中按Ctrl + D终止现有的LLDB会话，然后键入以下内容：

```vim
lldb -n Finder -w
```

这将告诉LLDB在下次启动时附加到名为Finder的进程。 接下来，打开一个新的终端选项卡，然后输入以下内容：

```vim
pkill Finder
```

这将终止Finder进程并强制它重新启动。 当它被杀死时，macOS会自动重启Finder。 切换回您的第一个终端选项卡，您会注意到LLDB现已将其自身附加到新创建的Finder进程。

附加到进程的另一种方法是指定可执行文件的路径，并在方便时手动启动进程：

```vim
lldb -f /System/Library/CoreServices/Finder.app/Contents/MacOS/Finder
```

这会将Finder设置为要启动的可执行文件。 准备好开始调试会话后，只需在LLDB会话中键入以下内容：

```vim
(lldb) process launch
```

> 注意：一个有趣的副作用是当手动启动进程时，stderr输出（即Swift的打印，Objective-C的NSLog，C的printf和公司）会自动发送到终端窗口。 其他LLDB附加配置不会自动执行此操作。

## 启动时的选项
流程启动命令附带一套值得进一步探索的选项。 如果您很好奇并希望查看流程启动的可用选项的完整列表，只需键入help process launch即可。

关闭以前的LLDB会话，打开一个新的终端窗口并键入以下内容：

```vim
lldb -f /bin/ls
```

这告诉LLDB使用/ bin / ls（文件列表命令）作为目标可执行文件。

> 注意：如果省略-f选项，LLDB将自动推断第一个参数是要启动和调试的可执行文件。 在调试终端可执行文件时，我经常会输入lldb $（即ls）（或等价物），然后将其转换为lldb / bin / ls。

您将看到以下输出：

```vim
(lldb) target create "/bin/ls"
Current executable set to '/bin/ls' (x86_64).
```

由于ls是一个快速程序（它启动，完成它的工作，然后退出），你将使用不同的参数多次运行该程序，以探索每个程序的作用。

从LLDB启动没有参数的ls。 输入以下内容：

```vim
(lldb) process launch
```

您将看到以下输出：

```vim
Process 7681 launched: '/bin/ls' (x86_64)
... # Omitted directory listing output
Process 7681 exited with status = 0 (0x00000000)
```

ls进程将在您启动的目录中启动。您可以通过使用-w选项告知LLDB在何处启动来更改当前工作目录。 输入以下内容：

```vim
(lldb) process launch -w /Applications
```

这将从/Applications目录中启动ls。 这相当于以下内容：

```vim
$ cd /Applications
$ ls
```

还有另一种方法可以做到这一点。 而不是告诉LLDB更改目录然后运行程序，您可以直接将参数传递给程序。

请尝试以下方法：

```vim
(lldb) process launch -- /Applications
```

这与前一个命令具有相同的效果，但这次它执行以下操作：

```vim
$ ls /Applications
```

同样，这会吐出所有macOS程序，但是您指定了一个参数而不是更改起始目录。 将桌面目录指定为启动参数怎么样？ 试试这个：

```vim
(lldb) process launch -- ~/Desktop
```

你会看到以下内容：

```vim
Process 8103 launched: '/bin/ls' (x86_64)
ls: ~/Desktop: No such file or directory
Process 8103 exited with status = 1 (0x00000001)
```

呃哦，那没用。 您需要shell来扩展参数中的波浪号。 试试这个：

```vim
(lldb) process launch -X true -- ~/Desktop
```

-X选项可扩展您提供的任何shell参数，例如代字号。 LLDB中有一个快捷方式：只需键入run即可。 要了解有关创建自己的命令快捷方式的更多信息，请查看第9章“保留和自定义命令”。

键入以下内容以查看运行的文档：

```vim
(lldb) help run
```

你会看到以下内容：

```vim
...
Command Options Usage:
  run [<run-args>]
'run' is an abbreviation for 'process launch -X true --'
```

看到？ 它是您刚刚运行的命令的缩写！ 通过键入以下命令来命令：

```vim
(lldb) run ~/Desktop
```

## 环境变量
对于终端程序，环境变量与程序的参数同样重要。 如果您要咨询该人1 ls，您将看到ls命令可以显示颜色输出，只要启用了颜色环境变量（CSICOLOR）并且您有“color pallete”环境变量LSCOLORS来告诉如何 显示某些文件类型。

使用LLDB中的目标，您可以使用任何环境变量组合启动和设置程序。

例如，要显示ls命令将启动的所有环境变量，请在LLDB中运行以下命令：

```vim
(lldb) env
```

这将显示目标的所有环境变量。 值得注意的是，在目标至少运行一次之前，LLDB不会显示环境变量。 如果没有看到任何输出，只需在执行env命令之前给lldb一个简单的运行。

您可以使用设置set | show | replace | clear | list target.env-vars命令在启动之前检查和扩充这些环境变量，但您也可以使用process launch命令中的-v选项在启动时指定它们！

是时候用可怕的红色显示/ Applications目录了！

```vim
(lldb) process launch -v LSCOLORS=Ab -v CLICOLOR=1  -- /Applications/
```



[图片]



哇！ 那只是灼伤眼睛吗？ 尝试使用以下不同的颜色：

```vim
(lldb) process launch -v LSCOLORS=Af -v CLICOLOR=1  -- /Applications/
```

这相当于您在没有LLDB的终端中执行以下操作：

```vim
LSCOLORS=Af CLICOLOR=1 ls /Applications/
```

许多终端命令将在命令的手册页中包含环境变量及其描述。 始终确保阅读有关您希望如何使用环境变量来扩充程序的信息。

此外，许多命令（和Apple框架！）具有“私有”环境变量，未在任何文档或手册页中讨论。 您将在本书后面看看如何从可执行文件中提取此信息。

## stdin，stderr和stout
如何将标准流更改为其他位置？ 您已经尝试使用-e标志将stderr更改为第1章中的其他终端选项卡，但stdout怎么样？

输入以下内容：

```vim
(lldb) process launch -o /tmp/ls_output.txt -- /Applications
```

-o选项告诉LLDB将stdout传递给给定文件。

您将看到以下输出：

```vim
Process 15194 launched: '/bin/ls' (x86_64)
Process 15194 exited with status = 0 (0x00000000)
```

请注意，没有直接来自ls的输出。

打开另一个终端选项卡并运行以下命令：

```vim
cat /tmp/ls_output.txt
```

正如预期的那样，它再次成为您的应用程序目录输出！

stdin也有一个选项-i。 首先，键入以下内容：

```vim
(lldb) target delete
```

这会删除ls作为目标。 接下来，键入：

```vim
(lldb) target create /usr/bin/wc
```

这将/ usr / bin / wc设置为新目标。 wc可用于计算给予stdin的输入中的字符，单词或行。

您已将LLDB会话的目标可执行文件从ls交换为wc。 现在你需要一些数据提供给wc。 打开新的终端选项卡，然后输入以下内容：

```vim
echo "hello world" > /tmp/wc_input.txt
```

您将使用此文件为wc提供一些输入。

切换回LLDB会话并输入以下内容：

```vim
(lldb) process launch -i /tmp/wc_input.txt
```

您将看到以下输出：

```vim
Process 24511 launched: '/usr/bin/wc' (x86_64)
       1       2      12
Process 24511 exited with status = 0 (0x00000000)
```

这在功能上等同于以下内容：

```vim
$ wc < /tmp/wc_input.txt
```

有时您不需要标准输入（标准输入）。 这对于诸如Xcode之类的GUI程序非常有用，但对ls和wc等终端命令没有帮助。

为了说明，运行没有参数的wc目标，如下所示：

```vim
(lldb) run
```

该程序将只是坐在那里挂起，因为它期望从stdin读取一些东西。

通过输入hello world给它一些输入，按Return键，然后按Control + D，这是传输字符的结束。 wc将解析输入并退出。 使用该文件作为输入时，您将看到与之前相同的输出。

现在，启动这样的过程：

```vim
(lldb) process launch -n
```

你会看到wc立即退出并输出以下内容：

```vim
Process 28849 launched: '/usr/bin/wc' (x86_64)
Process 28849 exited with status = 0 (0x00000000)
```

-n选项告诉LLDB不要创建标准输入; 因此wc没有数据可以使用并立即退出。

## 然后去哪儿？
还有一些更有趣的选项（您可以通过帮助命令找到），但这是您可以在自己的时间进行探索。

目前，尝试附加到GUI和非GUI程序。 如果没有源代码，你可能看起来很难理解，但是你会在接下来的部分中找到你对这些程序有多少信息和控制。

