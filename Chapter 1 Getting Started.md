# Chapter 1: Getting Started

在本章中，您将熟悉LLDB并研究内省和调试程序的过程。 你会先想象一下你甚至没有写过的程序--Xcode！

您将使用LLDB旋转一下调试会话，并发现您可以对您绝对零源代码的程序所做的惊人更改。 第一章非常倾向于过度学习，因此很多概念和深入研究某些LLDB功能将被保存用于以后的章节。

让我们开始吧。

## 绕过Rootless
在开始使用LLDB之前，您需要了解Apple推出的阻止恶意软件的功能。 不幸的是，此功能还会阻碍您使用LLDB和DTrace等其他工具进行内省和调试的尝试。 不要害怕，因为苹果公司提供了一种方法来解决这个问题 - 对于那些知道自己在做什么的人来说。 而且你将成为这些知道自己在做什么的人之一！

阻止内省和调试尝试的功能是系统完整性保护，也称为Rootless。 该系统限制程序可以执行的操作 - 即使它们具有root访问权限 - 也可以阻止恶意软件深入系统内部。

虽然Rootless在安全方面是一个重大飞跃，但它引入了一些烦恼，因为它使程序更难调试。 具体来说，它可以防止其他进程将调试器附加到程序Apple标志。

由于本书不仅涉及调试您自己的应用程序，而且涉及您对任何您感兴趣的应用程序，因此在您了解调试时删除此功能非常重要，这样您就可以检查您选择的任何应用程序。

如果您当前启用了Rootless，则无法附加到Apple的大多数程序。

例如，尝试将LLDB附加到Finder应用程序。

打开终端窗口并查找Finder进程，如下所示：

```vim
lldb -n Finder
```

您会注意到以下错误：

```text
error: attach failed: cannot attach to process due to System Integrity Protection
```

> 注意：有很多方法可以附加到进程，以及LLDB成功附加时的特定配置。 要了解有关附加到流程的更多信息，请查看第3章“使用LLDB附加”。

## 禁用Rootless

> 注意：遵循本书的更安全的方法是使用VMWare或VirtualBox创建专用虚拟机，并按照下面详述的步骤在该VM上禁用Rootless。 下载和设置macOS VM大约需要一个小时，具体取决于计算机的硬件（以及网速！）。 从mac获取最新的安装虚拟机说明，因为macOS版本和VM软件将有不同的安装步骤。 如果您选择在没有VM的情况下禁用计算机上的Rootless，那么在完成该特定章节后重新启用Rootless将是理想的选择。 幸运的是，本书中只有少数章节要求Rootless被禁用！

要禁用Rootless，请执行以下步骤：

1. 重新启动macOS计算机。
2. 当屏幕变为空白时，按住Command + R直到出现Apple启动徽标。 这将使您的计算机进入恢复模式。
3. 现在，从顶部找到Utilities菜单，然后选择Terminal。

4. 打开终端窗口，键入：

```vim
csrutil disable && reboot
```

5. 如果csrutil disable命令成功，则计算机将在Rootless禁用时重新启动。

通过键入以下内容，您可以通过在计算机启动后查询其在终端中的状态来验证是否已成功禁用Rootless：

```vim
csrutil status
```

你应该看到以下内容：

```text
System Integrity Protection status: disabled.
```

现在SIP已禁用，请执行您之前尝试过的相同“附加到Finder”LLDB命令。

```vim
lldb -n Finder
```

LLDB现在应该将其自身附加到当前的Finder流程。 成功附加的输出应如下所示：



[图片]



验证成功附加后，通过终止终端窗口或在LLDB控制台中键入退出并确认来分离LLDB。

## 将LLDB附加到Xcode
现在你已经禁用了Rootless，你可以将LLDB附加到macOS机器上的任何进程（可能会有一些障碍，例如ptrace系统调用，但我们稍后会讨论）。 您首先要查看您在日常开发中经常使用的应用程序：Xcode！ 在继续之前，请确保在计算机上安装了最新版本的Xcode 10。

打开一个新的终端窗口。 接下来，按⌘+ Shift + I编辑终端选项卡的标题。将出现一个新的弹出窗口。 将标题标题编辑为LLDB。



[图片]



接下来，确保Xcode没有运行，否则你最终会遇到多个正在运行的Xcode实例，这可能会引起混淆。

在终端中，键入以下内容：

```vim
lldb
```

这将启动LLDB。

按⌘+ T创建一个新的终端选项卡。使用⌘+ Shift + I再次编辑选项卡的标题，并将选项卡命名为Xcode stderr。 从调试器打印内容时，此终端选项卡将包含所有输出。

确保您在Xcode stderr终端选项卡上并键入以下内容：

```vim
tty
```

您应该看到类似于下面的内容：

```vim
/dev/ttys027
```

不要担心，如果你的不同; 如果不是，我会感到惊讶。 将此视为终端会话的地址。

为了说明您将使用Xcode stderr选项卡执行的操作，请创建另一个选项卡并在其中键入以下内容：

```vim
echo "hello debugger" 1>/dev/ttys027
```

请务必使用从tty命令获得的唯一路径替换终端路径。

现在切换回Xcode stderr选项卡。 应该弹出hello debugger这个词。 您将使用相同的技巧将Xcode的stderr输出传递给此选项卡。

最后，关闭第三个未命名的选项卡，然后导航回LLDB选项卡。

总结一下：您现在应该有两个终端选项卡：一个名为“LLDB”的选项卡，其中包含一个运行LLDB的实例，另一个选项卡名为“Xcode stderr”，其中包含您之前执行的tty命令。

从那里，在LLDB终端选项卡中输入以下内容：

```vim
(lldb) file /Applications/Xcode.app/Contents/MacOS/Xcode
```

这会将可执行目标设置为Xcode。

> 注意：如果您使用的是Xcode的预发布版本，那么Xcode的名称和路径可能会有所不同。
>
> 您可以通过启动Xcode并在终端中键入以下内容来检查当前运行的Xcode的路径：
>
> ps -ef`pgrep -x Xcode`
>
> 获得Xcode的路径后，请使用该新路径。

现在从LLDB启动Xcode进程，再次将/ dev / ttys027替换为Xcode stderr选项卡的tty地址：

```vim
(lldb) process launch -e /dev/ttys027 --
```

启动参数e指定stderr的位置。 常见的日志记录功能，例如Objective-C的NSLog或Swift的打印功能，输出到stderr  - 是的，不是标准输出！ 稍后您将打印自己的日志记录到stderr。

Xcode将在片刻之后推出。 切换到Xcode并单击File ▸ New ▸ Project....接下来，选择iOS ▸ Application ▸ Single View Application，然后单击下一步。 将产品命名为Hello Debugger。 确保选择Swift作为编程语言，并取消选择单元或UI测试的任何选项。 单击“下一步”并将项目保存到任何位置。



[图片]



您现在有了一个新的Xcode项目。 安排窗口，以便您可以看到终端和Xcode。

导航到Xcode并打开ViewController.swift。

> 注意：您可能会注意到Xcode stderr Terminal窗口上的一些输出; 这是由于Xcode的作者通过NSLog或其他stderr控制台打印功能记录的内容。

### 一个“迅速”变化的景观
Apple在自己的软件中采用Swift一直持谨慎态度 - 这是可以理解的。 无论一个人（似乎是宗教的）对斯威夫特的信仰，无论好坏，它仍然是一种不成熟的语言，以惊人的速度发展，突然发生变化。

然而，事情是苹果的变革。 Apple现在更积极地在他们自己的应用程序中采用Swift，例如iOS模拟器......甚至是Xcode！

最后，Xcode 10包含了近200个包含Swift的框架。

你如何自己验证这些信息？ 此信息是使用我在此处找到的帮助器LLDB脚本的组合获得的：https：//github.com/DerekSelander/ LLDB，对所有人都是免费的。 我强烈建议您克隆并安装此repo，因为我偶尔推出新的LLDB命令，使调试更加愉快。 安装说明位于repo的README中。 当这些LLDB脚本的情况变得非常容易时，我会在整本书中提到这个回购。

获取此信息的可怕命令如下。 如果要执行此命令，则需要安装上述注释中提到的repo。

```swift
(lldb) sys echo "$(dclass -t swift)" | grep -v _ | grep "\." | cut -d. -f1 | uniq | wc -l
```

断开此命令，dclass -t swift命令是一个自定义LLDB命令，它将转储流程中已知的Swift类的所有类。 sys命令允许您执行类似于终端的命令，但$（）中的任何内容都将首先通过LLDB进行评估。 从那里，它是一个操纵dclass命令给出的所有Swift类的输出的问题。

Swift类命名通常具有ModuleName.ClassName形式，其中模块是实现该类的框架。该命令的其余部分执行以下操作：

* grep -v _：排除包含下划线的任何Swift名称，这是Swift标准库中类名的典型特征。
* grep“\。”：按类名称中包含句点的Swift类进行过滤。
* cut -d。 -f1：在句点之前隔离模块名称。
* uniq：然后获取模块的所有唯一值。
* wc -l：并获取它的计数。

这些自定义LLDB命令（dclass，sys）是使用Python和LLDB的Python模块（混淆地也称为lldb）构建的。 在学习构建自定义的高级LLDB脚本时，您将非常习惯于在本书的第IV部分中使用此Python模块。

### 点击查找课程
现在已经设置了Xcode并正确创建和定位了终端调试窗口，现在是时候在调试器的帮助下开始探索Xcode了。

在调试时，了解Cocoa SDK非常有用。 例如， -  [NSView hitTest：]是一个有用的Objective-C方法，它返回负责处理运行循环中事件的单击或手势的类。 首先在包含的NSView上触发此方法，并递归地钻入处理此触摸的最远的子视图。 您可以使用Cocoa SDK的这些知识来帮助确定您单击的视图的类。

在LLDB选项卡中，键入Ctrl + C以暂停调试器。 从那里，键入：

```vim
(lldb) b -[NSView hitTest:]
Breakpoint 1: where = AppKit`-[NSView hitTest:], address =
0x000000010338277b
```

这是你未来很多人的第一个断点。 您将在第4章“在代码中停止”中了解如何创建，修改和删除断点的详细信息，但现在只需知道您已创建了一个断点 -  [NSView hitTest：]。

由于调试器，Xcode现在暂停了。 恢复程序：

```vim
(lldb) continue
```

单击Xcode窗口中的任意位置（或者在某些情况下，甚至将光标移动到Xcode上也会这样做）; Xcode将立即暂停，LLDB将指示断点已被击中。



[图片]



hotTest：断点已经解雇了。 您可以通过检查RDI CPU寄存器来检查哪个视图被命中。 在LLDB中打印出来：

```vim
(lldb) po $rdi
```

此命令指示LLDB在存储在RDI程序集寄存器中的内存地址处打印出对象的内容。

> 注意：想知道为什么命令是po？ po代表打印对象。 还有p，它只是打印RDI的内容。 po通常更有用，因为它提供了NSObject（或Swift的SwiftObject）描述或debugDescription方法（如果可用）。

如果您想将调试提升到一个新的水平，汇编是一项重要的技能。 它可以让您深入了解Apple的代码 - 即使您没有任何源代码可供阅读。 它将使您更好地了解Swift编译器团队如何使用Swift在Objective-C中跳出，并且它将使您更好地了解Apple设备上的一切是如何工作的。

您将在第11章“汇编注册调用约定”中了解有关寄存器和汇编的更多信息。

现在，只需知道上面的LLDB命令中的$ rdi寄存器包含调用hitTest：方法的子类NSView的实例。

> 注意：根据您单击的位置以及您正在使用的Xcode版本，输出将产生不同的结果。 它可以提供一个特定于Xcode的私有类，或者它可以为您提供属于Cocoa的公共类。

在LLDB中，键入以下内容以恢复该程序：

```vim
(lldb) continue
```

而不是继续，Xcode可能会击中hitTest的另一个断点：并暂停执行。 这是因为hitTest：方法以递归方式为所单击的父视图中包含的所有子视图调用此方法。 您可以检查此断点的内容，但由于组成Xcode的视图太多，这很快就会变得乏味。

### 自动化hitTest：
点击视图，停止，关闭RDI寄存器然后继续的过程可能会很快累。 如果您创建了一个断点来自动执行所有这些操作，该怎么办？

有几种方法可以实现这一点，但也许最干净的方法是声明一个具有你想要的所有特征的新断点。 那不是很整洁吗？！：]

使用以下命令删除上一个断点：

```vim
(lldb) breakpoint delete
```

LLDB会询问您是否确定要删除所有断点，按Enter或按“Y”然后输入进行确认。

现在，使用以下命令创建一个新断点：

```vim
(lldb) breakpoint set -n  "-[NSView hitTest:]" -C "po $rdi" -G1
```

这个命令的要点是在 -  [NSView hitTest：]上创建一个断点，让它执行“po $ rdi”命令，然后在执行命令后自动继续。 您将在后面的章节中了解有关这些选项的更多信息。

使用continue命令继续执行：

```vim
(lldb) continue
```

现在，单击Xcode中的任意位置并检查终端控制台中的输出。 您将看到许多NSView被调用以查看是否应该单击鼠标！

#### 过滤重要内容的断点
由于有很多NSView组成Xcode，你需要一种方法来过滤掉一些噪音，并且只在NSView上停止与你正在寻找的相关。这是一个调试常用方法的示例，您可以在其中找到一个独特的案例，帮助查明您真正想要的内容。

从Xcode 10开始，负责在Xcode IDE中直观显示代码的类是属于IDESourceEditor模块的私有Swift类，名为IDESourceEditorView。此类充当可视协调器，将所有代码传递给其他私有类，以帮助编译和创建应用程序。

假设您只想在单击IDESourceEditorView的实例时中断。您可以使用断点条件修改现有断点以仅在IDESourceEditorView单击上停止。

如果您仍然设置了 -  [NSView hitTest：]断点，并且它是LLDB会话中唯一的活动断点，则可以使用以下LLDB命令修改该断点：

```vim
(lldb) breakpoint modify -c '(BOOL)[NSStringFromClass((id)[$rdi class]) containsString:@"IDESourceEditorView"]' -G0
```

此命令修改调试会话中的所有现有断点，并创建一个每次都会评估的条件 -  [NSView hitTest：]触发。 如果条件的计算结果为true，则执行将在调试器中暂停。 此条件检查NSView的实例是否为IDESourceEditorView类型。 最后-G0表示修改断点，以便在执行操作后不自动恢复执行。

修改上面的断点后，单击Xcode中的代码区域。 LLDB应该在hitTest上停止： 打印出调用此方法的类的实例：

```vim
(lldb) po $rdi
```

您的输出应类似于以下内容：

```vim
IDESourceEditorView: Frame: (0.0, 0.0, 1109.0, 462.0), Bounds: (0.0, 0.0, 1109.0, 462.0) contentViewOffset: 0.0
```

这是打印出对象的描述。 你会注意到这里没有指针引用，因为Swift隐藏了指针引用。 如果需要指针引用，有几种方法可以解决这个问题。 最简单的方法是使用打印格式。 在LLDB中键入以下内容：

```vim
(lldb) p/x $rdi
```

您将获得类似于以下内容的内容：

```vim
(unsigned long) $3 = 0x0000000110a42600
```

由于RDI指向一个有效的Objective-C NSObject子类（用Swift编写），你也可以通过poing这个地址而不是寄存器来获得相同的信息。

在LLDB中键入以下内容，同时确保使用您自己的地址替换地址：

```vim
(lldb) po 0x0000000110a42600
```

您将获得与之前相同的输出。

您可能怀疑RDI寄存器指向的此引用实际上指向显示您的代码的NSView。 您可以通过在LLDB中键入以下内容轻松验证是否属实：

```vim
(lldb) po [$rdi setHidden:!(BOOL)[$rdi isHidden]]; [CATransaction flush]
```

> 注意：输入一个很长的命令，对吧？ 在第10章：“正则表达式命令”中，您将学习如何构建方便的快捷方式，这样您就不必输入这些长LLDB命令。 如果您选择安装前面提到的LLDB存储库，则可以使用上述操作的便捷命令tv命令或“切换视图”

如果RDI指向正确的引用，您的代码编辑器视图将消失！



[图片]



[图片]



您只需反复按Enter即可打开和关闭此视图。 LLDB将自动执行上一个命令。

由于这是NSView的子类，因此NSView的所有方法都适用。 例如，string命令可以通过LLDB查询源代码的内容。 输入以下内容：

```vim
(lldb) po [$rdi string]
```

这将转储源代码编辑器的内容。 整齐！

永远记住，您在开发周期中拥有的任何API都可以在LLDB中使用。 如果你足够疯狂，你可以通过执行LLDB命令创建一个完整的应用程序！

当您厌倦在此实例上使用NSView API时，请复制RDI引用的地址（将其复制到剪贴板或将其添加到stickies应用程序）。 你会在一秒钟内再次引用它。

或者，您是否注意到`p / x $ rdi`命令中十六进制值之前的输出？ 在我的输出中，我得到3美元，这意味着您可以使用$ 3作为您刚捕获的指针值的参考。 当RDI寄存器指向其他内容并且您仍希望稍后引用此NSView时，这非常有用。

### Swift vs Objective-C调试
等等 - 我们在Swift类上使用Objective-C ?! 你打赌！ 你会发现Swift类主要是封面下的所有Objective-C（但是关于Swift结构也是如此）。 您将通过使用Swift通过LLDB修改控制台的源代码来确认这一点！

首先，在Swift调试上下文中导入以下模块：

```vim
(lldb) ex -l swift -- import Foundation
(lldb) ex -l swift -- import AppKit
```

ex命令（表达式的简称）允许您评估代码，并且是您的p / po LLDB命令的基础。 -l swift告诉LLDB将您的命令解释为Swift代码。 您刚刚导入了标头，通过Swift调用这两个模块中的相应方法。 在接下来的两个命令中你需要这些。

输入以下内容，将0x0110a42600替换为最近复制到剪贴板的NSView子类的内存地址：

```vim
(lldb) ex -l swift -o -- unsafeBitCast(0x0110a42600, to: NSView.self)
```

此命令打印出IDESourceEditorView实例 - 但这次使用Swift！

现在，通过LLDB向源代码添加一些文本：

```vim
(lldb) ex -l swift -o -- unsafeBitCast(0x0110a42600, to:
NSView.self).insertText("Yay! Swift!")
```

根据光标在Xcode控制台中的位置，您将看到新字符串“Yay！Swift！” 添加到您的源代码中。

当突然停止调试器或在Objective-C代码上停止调试器时，LLDB将默认在调试时使用Objective-C上下文。 这意味着您执行的po将使用Objective-C语法，除非您强制LLDB使用与上面不同的语言。 可以改变这一点，但本书更喜欢使用Objective-C，因为Swift REPL对于错误检查来说可能是残酷的，执行命令的编译时间很慢，通常会有更多错误，并且阻止您执行Swift LLDB的方法 上下文不知道。

所有这一切最终都会消失，但我们必须耐心等待。 Swift ABI必须首先稳定下来。 只有这样，Swift工具才能真正成为坚如磐石的工具。

## 然后去哪儿？
这是一个广度优先，旋风式的介绍，使用LLDB并附加到您没有任何源代码来帮助您的过程。 本章重点介绍了很多细节，但目标是让您直接进入调试/逆向工程流程。

对于某些人来说，这第一章可能会有点吓人，但我们会从这里开始减速并详细描述方法。 还有很多章节可以帮助您了解详细信息！

继续阅读以了解第1部分其余部分的基本知识。快乐的调试！