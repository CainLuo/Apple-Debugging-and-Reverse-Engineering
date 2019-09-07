# Chapter 2: Help & Apropos

就像任何受人尊敬的开发人员工具一样，LLDB附带了大量的文档。 了解如何浏览本文档 - 包括一些更加模糊的命令标志 - 对于掌握LLDB至关重要。

## “帮助”命令
打开终端窗口并键入lldb。 将显示LLDB提示。 从那里，只需输入help命令：

```vim
(lldb) help
```

这将转储所有可用的命令，包括从〜/ .lldbinit加载的自定义命令 - 但稍后会详细介绍。



[图片]



LLDB可以使用很多命令。

但是，许多命令都有许多子命令，而子命令又可以有子命令，子命令也有自己的相关文档。 我告诉过你这是一份健康的文件！

以断点命令为例。 键入以下命令运行断点文档：

```vim
(lldb) help breakpoint
```

您将看到以下输出：

```vim
   Commands for operating on breakpoints (see 'help b' for shorthand.)
Syntax: breakpoint
The following subcommands are supported:
      clear   -- Delete or disable breakpoints matching the specified
source file and line.
      command -- Commands for adding, removing and listing LLDB commands
executed when a breakpoint is hit.
      delete  -- Delete the specified breakpoint(s).  If no breakpoints
are specified, delete them all.
      disable -- Disable the specified breakpoint(s) without deleting
them.  If none are specified, disable all breakpoints.
      enable  -- Enable the specified disabled breakpoint(s). If no
breakpoints are specified, enable all of them.
      list    -- List some or all breakpoints at configurable levels of
detail.
      modify  -- Modify the options on a breakpoint or set of breakpoints
in the executable.  If no breakpoint is specified,
                 acts on the last created breakpoint.  With the exception
of -e, -d and -i, passing an empty argument clears
                 the modification.
      name    -- Commands to manage name tags for breakpoints
      read    -- Read and set the breakpoints previously saved to a file
with "breakpoint write".
      set     -- Sets a breakpoint or set of breakpoints in the
executable.
      write   -- Write the breakpoints listed to a file that can be read
in with "breakpoint read".  If given no arguments,
                 writes all breakpoints.
For more help on any particular subcommand, type 'help <command>
<subcommand>'.
```

从那里，您可以看到几个受支持的子命令。 通过键入以下内容查找断点名称的文档：

```vim
(lldb) help breakpoint name
```

您将看到以下输出：

```vim
   Commands to manage name tags for breakpoints
Syntax: breakpoint name
The following subcommands are supported:
      add       -- Add a name to the breakpoints provided.
      configure -- Configure the options for the breakpoint name
provided.  If you provide a breakpoint ID, the options will be
                   copied from the breakpoint, otherwise only the options
specified will be set on the name.
      delete    -- Delete a name from the breakpoints provided.
      list      -- List either the names for a breakpoint or info about a
given name.  With no arguments, lists all names
For more help on any particular subcommand, type 'help <command>
<subcommand>'.
```

如果您此刻不了解断点名称，请不要担心 - 您将很快熟悉断点和所有后续命令。 现在，help命令是您能记住的最重要的命令。

## “apropos”命令
有时您不知道要搜索的命令的名称，但您知道可能指向正确方向的某个单词或短语。 apropos命令可以为您执行此操作; 这有点像使用搜索引擎在网络上找到一些东西。

apropos将针对LLDB文档对任何单词或字符串进行不区分大小写的搜索，并返回任何匹配的结果。 例如，尝试搜索与Swift有关的任何内容：

```vim
(lldb) apropos swift
```

您将看到以下输出：

```vim
The following commands may relate to 'swift':
  swift    -- A set of commands for operating on the Swift Language
Runtime.
  demangle -- Demangle a Swift mangled name
  refcount -- Inspect the reference count data for a Swift object
The following settings variables may relate to 'swift':
  target.swift-framework-search-paths -- List of directories to be searched when locating frameworks for Swift.
  target.swift-module-search-paths -- List of directories to be searched
when locating modules for Swift.
  target.use-all-compiler-flags -- Try to use compiler flags for all
modules when setting up the Swift expression parser, not just the main
executable.
  target.experimental.swift-create-module-contexts-in-parallel -- Create
the per-module Swift AST contexts in parallel.
```

这会抛弃可能与Swift一词相关的所有内容：首先是命令，然后是LLDB设置，它们可用于控制LLDB的运行方式。

您还可以使用apropos搜索特定句子。 例如，如果您正在搜索可以帮助引用计数的内容，则可以尝试以下操作：

```vim
(lldb) apropos "reference count"
The following commands may relate to 'reference count':
  refcount -- Inspect the reference count data for a Swift object
```

请注意“引用计数”字样的引号。 apropos只接受一个参数来搜索，因此引号是将输入视为单个参数所必需的。

这不是很整洁吗？ apropos是一个方便的查询工具。 它不像现代互联网搜索引擎那么复杂; 然而，随着一些游戏，你通常可以找到你想要的东西。

> 注意：在最新的XLD 10版本的LLDB（lldb-1000.11.37.1）中弹出的新错误在使用apropos时不会提供完整的命令树。 例如，上面显示的refcount命令实际上只能通过语言swift refcount使用。 apropos命令在以前的版本中正确地显示了这一点（并且可能在将来再次出现），但是现在你需要做一些挖掘以使用apropos获得确切的命令

## 然后去哪儿？
很容易忘记即将到来的LLDB命令的冲击，但试着将这两个命令，帮助和apropos提交到内心。 它们是查询命令信息的基础，您将在掌握调试时始终使用它们。