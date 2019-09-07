# Chapter 4: Stopping in Code

无论您是在技术堆栈中使用Swift，Objective-C，C ++，C还是完全不同的语言，您都需要学习如何创建断点。 可以轻松地在Xcode中单击侧面板以使用GUI创建断点，但LLDB控制台可以让您更好地控制断点。

在本章中，您将学习所有关于断点以及如何使用LLDB创建断点的知识。

## 信号
在本章中，您将看到我提供的项目; 它被称为信号，你可以在本章的资源包中找到它。



[图片]



使用Xcode打开Signals项目。信号是一个基本的主要细节项目，主题是美式橄榄球应用程序，显示一些相当神经命名的进攻性打法。

在内部，该项目监视几个Unix信号，并在信号程序接收它们时显示它们。

Unix信号是进程间通信的基本形式。例如，其中一个信号SIGSTOP可用于保存状态并暂停执行进程，而其对应的SIGCONT则被发送到程序以恢复执行。调试器可以使用这两个信号暂停并继续执行程序。

这在几个方面是一个有趣的应用程序，因为它不仅探讨了Unix信号处理，而且还强调了当控制进程（LLDB）处理Unix信号传递到受控进程时会发生什么。默认情况下，LLDB具有用于处理不同信号的自定义操作。在连接LLDB时，某些信号不会传递到受控过程。

为了显示信号，您可以从应用程序中提升信号，或者从其他应用程序（如终端）发送信号。

此外，还有一个UISwitch可以切换信号处理。当切换开关时，它调用C函数sigprocmask来禁用或启用信号处理程序。

最后，Signal应用程序有一个Timeout bar按钮，可以从应用程序中提升SIGSTOP信号，基本上“冻结”程序。但是，如果LLDB附加到Signals程序（默认情况下，当您构建并运行Xcode时），调用SIGSTOP将允许您在Xcode中使用LLDB检查执行状态。

选择您的iOS模拟器，同时确保iOS模拟器的版本至少为iOS 12.0或更高版本。构建并运行应用程序。项目运行后，导航到Xcode控制台并暂停调试器。



[图片]



恢复Xcode并密切关注模拟器。只要调试器停止然后恢复执行，就会在UITableView中添加一个新行。这是通过信号监视SIGSTOP Unix信号事件并在数据模型发生时向数据模型添加行来实现的。当一个进程停止时，任何新的信号都不会立即被处理，因为该程序有点停止。

### Xcode断点
在您通过LLDB控制台学习酷炫闪亮的断点之前，值得介绍一下您可以通过Xcode实现的目标。

符号断点是Xcode的一个很好的调试功能。它们允许您在应用程序中的某个符号上设置断点。符号的示例是 -  [NSObject init]，它引用NSObject实例的init方法。

Xcode中符号断点的巧妙之处在于，一旦输入符号断点，下次程序启动时就不必再次输入符号断点。

您现在将尝试使用符号断点来显示正在创建的NSObject的所有实例。

如果应用程序当前正在运行，请将其杀死。接下来，切换到Breakpoint Navigator。在左下角，单击加号按钮以选择Symbolic Breakpoint ...选项。



[图片]



将出现一个弹出窗口。 在弹出窗口的符号部分中： -  [NSObject init]。 在“操作”下，选择“添加操作”，然后从下拉列表中选择“调试器命令”。 接下来，在下面的框中输入po [$ arg1 class]。

最后，选择评估操作后自动继续。 您的弹出窗口应类似于以下内容：



[图片]



构建并运行应用程序。 Xcode将在通过控制台运行Signals程序时转储它初始化的类的所有名称...在查看时，它是非常多的。



[图片]



你在这里做的是设置一个每次触发的断点 - 调用[NSObject init]。 当断点触发时，命令在LLDB中运行，程序的执行自动继续。

> 注意：您将在第11章“汇编，寄存器和调用约定”中学习如何正确使用和操作寄存器，但是现在，只需知道$ arg1与$ rdi寄存器同义，并且可以被认为是持有 调用init时的类的实例。

检查完所有转出的类名后，通过右键单击断点导航器中的断点并选择“删除断点”来删除符号断点。

除了符号断点，Xcode还支持几种类型的错误断点。 其中之一是异常断点。 有时候，你的程序出了问题而且只是简单地崩溃了。 当发生这种情况时，您对此的第一反应应该是启用异常断点，每次抛出异常时都会触发该异常断点。 Xcode将向您显示违规行，这有助于追捕导致崩溃的罪魁祸首。

最后，还有Swift错误断点，它通过在swift_willThrow方法上创建断点来随时停止Swift抛出错误。 如果您正在处理任何容易出错的API，这是一个很好的选择，因为它可以让您快速诊断情况，而不会对代码的正确性做出错误的假设。

## LLDB断点语法
现在您已经使用了Xcode的IDE调试功能的速成课程，现在是时候学习如何通过LLDB控制台创建断点了。 为了创建有用的断点，您需要学习如何查询您要查找的内容。

image命令是一个很好的工具，可以帮助内省对设置断点至关重要的细节。

您将在本书中使用两种配置来搜索代码。 第一个是以下内容：

```vim
(lldb) image lookup -n "-[UIViewController viewDidLoad]"
```

此命令转储 -  [UIViewController viewDidLoad]函数的实现地址（此方法在框架的二进制文件中的位置的偏移地址）。 -n参数告诉LLDB查找符号或函数名称。 输出类似于以下内容：

```vim
1 match found in /Applications/Xcode.app/Contents/Developer/Platforms/
iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/
iOS.simruntime/Contents/Resources/RuntimeRoot/System/Library/
PrivateFrameworks/UIKitCore.framework/UIKitCore:
        Address: UIKitCore[0x0000000000ac813c] (UIKitCore.__TEXT.__text +
11295452)
        Summary: UIKitCore`-[UIViewController viewDidLoad]
```

由于 -  [UIViewController viewDidLoad]方法所在的位置，您可以知道这是iOS 12。 在以前的iOS版本中，这个方法位于UIKit，但现在已经转移到UIKitCore，可能是由于macOS / iOS统一过程，非正式地称为Marzipan。

另一个有用的类似命令是这样的：

```vim
(lldb) image lookup -rn test
```

这会对单词“test”进行区分大小写的正则表达式查找。 如果在任何函数中，在任何函数中，在当前可执行文件中加载的任何模块（即UIKit，Foundation，Core Data等）中找到小写单词“test”（不会从发布版本中删除...更多 在稍后），此命令将吐出结果。

> 注意：如果需要完全匹配（如果查询包含空格，则在引号周围使用引号）并使用-rn参数进行正则表达式搜索时，请使用-n参数。 -n only命令有助于确定与断点匹配的确切参数，尤其是在处理Swift时，而-rn参数选项在本书中会受到很大的青睐，因为智能正则表达式可以消除相当多的输入 - 就像你' 很快就会发现。

### Objective-C属性
学习如何查询加载的代码对于学习如何在该代码上创建断点至关重要。 当Objective-C和Swift由编译器创建时，它们都具有特定的属性签名，这在查找代码时会产生不同的查询策略。

例如，在Signals项目中声明了以下Objective-C类：

```swift
@interface TestClass : NSObject
@property (nonatomic, strong) NSString *name;
@end
```

编译器将为属性名称的setter和getter生成代码。 getter将如下所示：

```vim
-[TestClass name]
```

...虽然setter看起来像这样：

```vim
-[TestClass setName:]
```

构建并运行应用程序，然后暂停调试器。 接下来，通过在LLDB中键入以下内容来验证这些方法是否存在：

```vim
(lldb) image lookup -n "-[TestClass name]"
```

在控制台输出中，您将获得类似于以下内容的内容：

```vim
1 match found in /Users/derekselander/Library/Developer/Xcode/
DerivedData/Signals-atqcdyprrotlrvdanihoufkwzyqh/Build/Products/Debug-
iphonesimulator/Signals.app/Signals:
        Address: Signals[0x0000000100002150] (Signals.__TEXT.__text + 0)
        Summary: Signals`-[TestClass name] at TestClass.h:28
```

LLDB将转储有关可执行文件中包含的函数的信息。 输出可能看起来很可怕，但这里有一些好的花絮。

> 注意：当查询匹配大量代码时，图像查找命令可以产生大量输出，这些输出对于眼睛来说非常困难。 在第26章“SB示例，改进的查找”中，您将构建一个更清晰的替代LLDB的图像查找命令，以避免您的眼睛看到太多的输出。

控制台输出告诉你LLDB能够找到这个函数是在Signals可执行文件中实现的，在__text段的__TEXT段中的偏移量为0x0000000100002150是准确的（如果没有任何意义，不要担心， 你将在本书的后面学到所有这些。 LLDB还能够告诉该方法是在TestClass.h的第28行声明的。

您也可以检查setter，如下所示：

```vim
(lldb) image lookup -n "-[TestClass setName:]"
```

您将获得类似于上一个命令的输出，这次显示实现地址和setter的name声明。

### Objective-C属性和点符号
入门级Objective-C（或仅限Swift）开发人员经常误导的是属性的Objective-C点表示法语法。

Objective-C点表示法是一个有点争议的编译器功能，允许属性使用速记getter或setter。

考虑以下：

```swift
TestClass *a = [[TestClass alloc] init];
// Both equivalent for setters
[a setName:@"hello, world"];
a.name = @"hello, world";

// Both equivalent for getters
NSString *b;
b = [a name]; // b = @"hello, world"
b = a.name;   // b = @"hello, world"
```

在上面的示例中， -  [TestClass setName：]方法被调用两次，即使使用点表示法也是如此。 对于getter来说也是如此， -  [TestClass name]。 重要的是要知道您是在处理Objective-C代码并尝试使用点表示法在setter和getter属性上创建断点。

### Swift属性
Swift中的属性语法有很大不同。 看一下SwiftTestClass.swift中的代码，其中包含以下内容：

```swift
class SwiftTestClass: NSObject {
  var name: String!
}
```

确保信号项目在LLDB中运行并暂停。 通过在调试窗口中键入Command + K来清除LLDB控制台，以便重新开始。

在LLDB控制台中，键入以下内容：

```vim
(lldb) image lookup -rn Signals.SwiftTestClass.name.setter
```

您将获得类似于以下的输出：

```vim
1 match found in /Users/derekselander/Library/Developer/Xcode/
DerivedData/Signals-atqcdyprrotlrvdanihoufkwzyqh/Build/Products/Debug-
iphonesimulator/Signals.app/Signals:
        Address: Signals[0x000000010000bd50] (Signals.__TEXT.__text +
39936)
        Summary: Signals`Signals.SwiftTestClass.name.setter :
Swift.Optional<Swift.String> at SwiftTestClass.swift:28
```

在输出中的单词Summary之后搜索信息。 这里有几个有趣的事情需要注意。

你看到功能名称有多长了！？ 整个事情需要输入一个有效的Swift断点！ 如果要在此setter上设置断点，则必须键入以下内容：

```vim
(lldb) b Signals.SwiftTestClass.name.setter :
Swift.Optional<Swift.String>
```

使用正则表达式是输入这种怪物的有吸引力的替代方案。

除了您生成的Swift函数名称的长度之外，还要注意Swift属性是如何形成的。 包含属性名称的函数签名在属性后面紧跟着setter一词。 也许同样的约定也适用于getter方法？

使用以下正则表达式查询同时搜索name属性的SwiftTestClass setter和getter：

```vim
(lldb) image lookup -rn Signals.SwiftTestClass.name
```

这使用正则表达式查询来转储包含短语Signals.SwiftTestClass.name的所有内容。

由于这是一个正则表达式，因此句点（。）被计算为通配符，通配符又匹配实际函数签名中的句点。

您将获得相当多的输出，但每次在控制台输出中看到“摘要”一词时都要磨练。 你会发现输出匹配getter，（Signals.SwiftTestClass.name.getter）setter，（Signals.SwiftTestClass.name.setter），以及两个包含materializeForSet的方法，Swift构造函数的辅助方法。

Swift属性的函数名称有一个模式：

ModuleName.Classname.PropertyName（吸气| setter方法）。

转换方法，查找模式和缩小搜索范围的能力是在您的代码中创建智能断点时发现Swift / Objective-C语言内部的一种很好的方法。

## 最后......创建断点
现在您知道如何查询代码中函数和方法的存在，是时候开始在它们上创建断点了。

如果您已经运行Signals应用程序，请停止并重新启动应用程序，然后按暂停按钮以停止应用程序并启动LLDB控制台。

有几种不同的方法可以创建断点。 最基本的方法是只输入字母b，后跟断点名称。 这在Objective-C和C中相当容易，因为名称简短且易于键入（例如 -  [NSObject init]或 -  [UIView setAlpha：]）。 输入C ++和Swift非常棘手，因为编译器会将您的方法转换为具有相当长名称的符号。

由于UIKit主要是Objective-C（至少在撰写本文时！），使用b参数创建一个断点，如下所示：

```vim
(lldb) b -[UIViewController viewDidLoad]
```

您将看到以下输出：

```vim
Breakpoint 1: where = UIKitCore`-[UIViewController viewDidLoad], address = 0x0000000114a4a13c
```

创建有效断点时，控制台将吐出有关该断点的一些信息。 在这种特殊情况下，断点创建为断点1，因为这是此特定调试会话中的第一个断点。 在创建更多断点时，此断点ID将递增。

恢复调试器。 一旦恢复执行，将显示一个新的SIGSTOP信号。 点击单元格以显示详细信息UIViewController。 当调用详细视图控制器的viewDidLoad时，程序应该暂停。

> 注意：与许多速记命令一样，b是另一个更长的LLDB命令的缩写。 使用b命令运行帮助以自己弄清楚实际命令，并了解b可以在引擎盖下做的所有很酷的技巧。

除了b命令之外，还有另一个更长的断点设置命令，它有一些可用的选项。 您将在接下来的几个部分中探索这些选项。 许多命令都源于断点集命令的各种选项。

### 正则表达式断点和范围
另一个非常强大的命令是正则表达式断点rbreak，它是断点集-r％1的缩写。 您可以使用智能正则表达式快速创建许多断点，以便随时随地停止。

回到上一个示例，使用了极长的Swift属性函数名称，而不是键入：

```vim
(lldb) b Signals.SwiftTestClass.name.setter :
Swift.Optional<Swift.String>
```

你可以简单地输入：

```vim
(lldb) rb SwiftTestClass.name.setter
```

rb命令将扩展到rbreak（假设您没有任何以“rb”开头的其他LLDB命令）。 这将在SwiftTestClass中的name的setter属性上创建一个断点

更简单一点，您可以简单地使用以下内容：

```vim
(lldb) rb name\.setter
```

这将在包含短语name.setter的任何内容上生成断点。 如果您知道项目中没有任何其他名为name的Swift属性，这将有效; 否则，您将为包含具有setter的“name”属性的每个类创建多个断点。

让我们了解这些正则表达式的复杂性。

在UIViewController的每个Objective-C实例方法上创建一个断点。 在LLDB会话中键入以下内容：

```vim
(lldb) rb '\-\[UIViewController\ '
```

丑陋的反斜杠是转义字符，表示您希望文字字符在正则表达式搜索中。 因此，此查询会在包含字符串的每个方法上中断 -  [UIViewController后跟一个空格。

但是等等...... Objective-C类别怎么样？ 它们采用（ -  | +）[ClassName（categoryName）方法]的形式。 您还必须重写正则表达式以包含类别。

在LLDB会话中键入以下内容，并在提示时键入y以确认：

```vim
(lldb) breakpoint delete
```

此命令删除您设置的所有断点。

接下来，键入以下内容：

```vim
(lldb) rb '\-\[UIViewController(\(\w+\))?\ '
```

在断点中的UIViewController之后，这提供了一个带有一个或多个字母数字字符后跟空格的可选括号。

使用正则表达式断点可以使用单个表达式捕获各种断点。

您可以使用-f选项将断点的范围限制为某个文件。 例如，您可以键入以下内容：

```vim
(lldb) rb . -f DetailViewController.swift
```

如果您正在调试DetailViewController.swift，这将非常有用。 它将在此文件中的所有属性getter / setter，块/闭包，扩展/类别和函数/方法上设置断点。 -f称为范围限制。

如果你是完全疯狂并且是痛苦的粉丝（医生称之为自虐？），你可以省略范围限制并简单地这样做：

```vim
(lldb) rb .
```

这将在所有事情上创造一个断点...是的，一切！ 这将在Signals项目中的所有代码上创建断点，UIKit中的所有代码以及Foundation，所有事件运行循环代码都被解雇（希望）60赫兹 - 一切。 因此，如果执行此操作，期望在调试器中输入keep。

还有其他方法可以限制搜索范围。 您可以使用-s选项限制到单个库：

```vim
(lldb) rb . -s Commons
```

这将在Commons库中的所有内容上设置断点，这是一个包含在Signals项目中的动态库。

这不仅限于您的代码; 您可以使用相同的策略在UIKitCore中的每个函数上创建断点，如下所示：

```vim
(lldb) rb . -s UIKitCore
```

即使这仍然有点疯狂。 有很多方法 - 在iOS 12.0中大约有86,760个UIKitCore方法。 如何才能停止UIKitCore的第一个方法，你只需要继续？ -o选项为此提供了解决方案。 它创造了所谓的“一次性”断点。 当这些断点命中时，断点将被删除。 所以它只会打一次。

要查看此操作，请在LLDB会话中键入以下内容：

```vim
(lldb) breakpoint delete
(lldb) rb . -s UIKitCore -o 1
```

> 注意：在计算机执行此命令时请耐心等待，因为LLDB必须创建大量断点。 还要确保你使用的是模拟器，否则你会等很长时间！

接下来，继续调试器，然后单击表视图中的单元格。 调试器在此操作调用的第一个UIKitCore方法上停止。 最后，继续调试器，断点将不再触发。

### 其他很酷的断点选项
-L选项允许您按源语言进行过滤。 因此，如果您只想在Signals应用程序的Commons模块中使用Swift代码，则可以执行以下操作：

```vim
(lldb) breakpoint set -L swift -r . -s Commons
```

这将在Commons模块中的每个Swift方法上设置断点。

如果你想在Swift周围寻找一些有趣的东西，如果让它完全忘记你的应用程序在哪里，该怎么办？ 您可以使用源正则表达式断点来帮助计算感兴趣的位置！ 像这样：

```vim
(lldb) breakpoint set -A -p "if let"
```

这将在包含if的每个源代码位置创建一个断点。 当然，由于-p采用正则表达式断点来处理复杂的表达式，因此您可以获得更多的花哨。 -A选项表示搜索项目已知的所有源文件。

如果要将上述断点查询过滤为仅MasterViewController.swift和DetailViewController.swift，则可以执行以下操作：

```vim
(lldb) breakpoint set -p "if let" -f MasterViewController.swift -f DetailViewController.swift
```

注意-A如何消失，以及每个-f如何指定文件名。 我很懒，所以我通常默认使用-A给我所有文件并从那里钻取。

最后，您还可以按特定模块进行过滤。 如果你想为Signals可执行文件中的任何东西创建“if let”的断点（忽略其他框架，如Commons），你可以这样做：

```vim
(lldb) breakpoint set -p "if let" -s Signals -A
```

这将获取所有源文件（-A），但只过滤那些属于Signals可执行文件的源文件（使用-s Signals选项）。

还有一个很酷的断点选项示例？ 好的，你跟我谈过了。 每当viewDidLoad被命中时，您将创建一个打印UIViewController的断点，但是您将通过LLDB控制台而不是符号断点窗口来执行此操作。 然后，您将此断点导出到一个文件，这样您就可以通过使用断点读取和断点写入命令来显示您对同事的酷感！

首先，删除所有断点：

```vim
(lldb) breakpoint delete
```

现在创建以下（复杂！）断点：

```vim
(lldb) breakpoint set -n "-[UIViewController viewDidLoad]" -C "po $arg1" -G1
```

确保使用capitol -C，因为LLDB的-c执行不同的选项！

这表示在 -  [UIViewController viewDidLoad]上创建一个断点，然后执行（C）ommand“po $ arg1”，它打印出UIViewController的实例。 从那里，-G1选项告诉断点在执行命令后自动继续。

通过点击其中一个包含Unix信号的UITableViewCells，触发viewDidLoad，验证控制台是否显示预期信息。

现在，你怎么把这个发送给同事？ 在LLDB中，键入以下内容：

```vim
(lldb) breakpoint write -f /tmp/br.json
```

这会将会话中的所有断点写入/tmp/br.json文件。 您可以通过断点ID指定单个断点或断点列表，但这可以通过您自己的时间通过帮助文档确定。

您可以使用platform shell命令在终端或通过LLDB验证断点数据，以便突破使用终端。

使用cat Terminal命令显示断点数据。

```vim
(lldb) platform shell cat /tmp/br.json
```

这意味着您可以将此文件发送给您的同事并让她通过断点读取命令打开它。

要模拟此，请再次删除所有断点。

```vim
(lldb) breakpoint delete
```

您现在将拥有一个没有断点的干净调试会话。

现在，重新导入自定义断点命令：

```vim
(lldb) breakpoint read -f /tmp/br.json
```

再一次，如果您要触发UIViewController的viewDidLoad方法，由于您的自定义断点逻辑，该实例将被打印出来！ 使用这些命令，您可以轻松发送和接收LLDB断点命令，以帮助复制难以捕获的bug！

### 修改和删除断点
现在您已经基本了解了如何创建这些断点，您可能想知道如何更改它们。 如果您找到了您感兴趣的对象并想要删除断点，或暂时禁用它，该怎么办？ 如果您需要修改断点以在下次触发时执行特定操作，该怎么办？

首先，您需要了解如何唯一地标识断点或一组断点。

构建并运行应用程序以获得干净的LLDB会话。 接下来，暂停调试器并在LLDB会话中键入以下内容：

```vim
(lldb) b main
```

输出将类似于以下内容：

```vim
Breakpoint 1: 70 locations.
```

这将创建一个包含70个位置的断点，与各个模块中的“main”功能相匹配。

在这种情况下，断点ID为1，因为它是您在此会话中创建的第一个断点。 要查看有关此断点的详细信息，可以使用breakpoint list子命令。 输入以下内容：

```vim
(lldb) breakpoint list 1
```

输出看起来类似于下面的截断输出：

```vim
1: name = 'main', locations = 70, resolved = 70, hit count = 0
  1.1: where = Signals`main at AppDelegate.swift, address =
0x00000001098b1520, resolved, hit count = 0
  1.2: where = Foundation`-[NSThread main], address = 0x0000000109bfa9e3,
resolved, hit count = 0
  1.3: where = Foundation`-[NSBlockOperation main], address =
0x0000000109c077d6, resolved, hit count = 0
  1.4: where = Foundation`-[NSFilesystemItemRemoveOperation main],
address = 0x0000000109c40e99, resolved, hit count = 0
  1.5: where = Foundation`-[NSFilesystemItemMoveOperation main], address
= 0x0000000109c419ee, resolved, hit count = 0
  1.6: where = Foundation`-[NSInvocationOperation main], address =
0x0000000109c6aee4, resolved, hit count = 0
  1.7: where = Foundation`-[NSDirectoryTraversalOperation main], address
= 0x0000000109caefa6, resolved, hit count = 0
  1.8: where = Foundation`-[NSOperation main], address =
0x0000000109cfd5e3, resolved, hit count = 0
  1.9: where = Foundation`-
[_NSFileAccessAsynchronousProcessAssertionOperation main], address =
0x0000000109d55ca9, resolved, hit count = 0
  1.10: where = UIKit`-[_UIFocusFastScrollingTest main], address =
0x000000010b216598, resolved, hit count = 0
  1.11: where = UIKit`-[UIStatusBarServerThread main], address =
0x000000010b651e97, resolved, hit count = 0
  1.12: where = UIKit`-[_UIDocumentActivityDownloadOperation main],
address = 0x000000010b74f718, resolved, hit count = 0
```

这显示了该断点的详细信息，包括包含“main”一词的所有位置。

更简洁的方法是键入以下内容：

```vim
(lldb) breakpoint list 1 -b
```

这将为您提供在视觉上更容易的输出。 如果你有一个封装了很多断点的断点ID，那么这个简短的标志就是一个很好的解决方案。

如果要查询LLDB会话中的所有断点，只需省略ID，如下所示：

```vim
(lldb) breakpoint list
```

您还可以指定多个断点ID和范围：

```vim
(lldb) breakpoint list 1 3
(lldb) breakpoint list 1-3
```

使用断点删除删除所有断点有点笨拙。 您可以简单地使用breakpoint list命令中使用的相同ID模式来删除集合。

您可以通过指定ID来删除单个断点，如下所示：

```vim
(lldb) breakpoint delete 1
```

但是，“main”的断点有70个位置（可能更多或更少，具体取决于iOS版本）。 您也可以删除单个位置，如下所示：

```vim
(lldb) breakpoint delete 1.1
```

这将删除断点1的第一个子断点，这会导致只删除一个主函数断点，同时保持其余的主断点处于活动状态。

## 然后去哪儿？
你在本章中已经介绍了很多内容。断点是一个很大的主题，掌握快速找到感兴趣的项目的艺术对于成为调试专家至关重要。您还开始使用正则表达式探索函数搜索。现在是了解正则表达式语法的好时机，因为在本书的其余部分中您将使用许多正则表达式。

查看https://docs.python.org/2/library/re.html以了解（或重新学习）正则表达式。尝试找出如何进行不区分大小写的断点查询。

您刚刚开始发现编译器如何在Objective-C和Swift中生成函数。尝试找出停止Objective-C块或Swift闭包的语法。完成后，尝试设计一个仅在Signals项目的Commons框架内的Objective-C块上停止的断点。这些是您将来需要构建更复杂断点的正则表达式技能。