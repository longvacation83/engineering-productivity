## 代码格式化工具

### ClangFormat

> 主要参考以下内容：
> * <https://clang.llvm.org/docs/ClangFormat.html>

#### 介绍

ClangFormat基于LibFormat创建，包含一套工具集。这套工具集包括独立的代码格式化工具，以及主流代码编辑器/IDE（例如VSCode，VIM，CLON等）的配套集成工具。

其中clang-format工具位于clang/tools/clang-format中，可用于格式化 C/C++/Java/JavaScript/JSON/Objective-C/Protobuf/C# 等代码。下文重点介绍clang-format的使用。

> 在介绍clang-format使用前，重点介绍下clang-format的git集成：
> 
> git-clang-format可以让我们只针对本次修改的代码，实现格式化；避免因为格式化导致大量代码修改，无法进行code review。

#### 安装

以Ubuntu安装为例。Ubuntu20.04自带的clang-format版本较老，建议配置llvm源。

> 配置llvm源可以参考：<https://apt.llvm.org/>

对于Ubuntu 20.04 focal，添加如下内容到`/etc/apt/sources.list`：
```
deb http://apt.llvm.org/focal/ llvm-toolchain-focal main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal main
# 14
deb http://apt.llvm.org/focal/ llvm-toolchain-focal-14 main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal-14 main
# 15
deb http://apt.llvm.org/focal/ llvm-toolchain-focal-15 main
deb-src http://apt.llvm.org/focal/ llvm-toolchain-focal-15 main
```

安装clang-format（以16版本为例）：
```
sudo apt update
sudo apt search clang-format
sudo apt install clang-format-16
```

如果之前已经安装了clang-format，可以将clang-format软链接替换为clang-format-16

#### 选项

使用clang-format时，既可以使用其中某种预定义样式（如LLVM、Google、Chromium、Mozilla、WebKit、Microsoft等），也可以通过配置特定`样式选项`来创建自定义样式。

> 格式选项（及配置方式）可参考 <https://clang.llvm.org/docs/ClangFormatStyleOptions.html>

例如: 
```
// 以Google C++ Coding Style格式化hello.cc, 结果输出stdout
clang-format --style=LLVM hello.cc
// 以Google C++ Coding Style格式化hello.cc, 结果写入hello.cc
clang-format --style=Google -i hello.cc
// 格式化hello.cc的第1，10行
clang-format --lines=1:10 hello.cc
```

clang-format工具命令行参数可以通过运行`clang-format --help`查看

```
$ clang-format --help
OVERVIEW: A tool to format C/C++/Java/JavaScript/JSON/Objective-C/Protobuf/C# code.

If no arguments are specified, it formats the code from standard input
and writes the result to the standard output.
If <file>s are given, it reformats the files. If -i is specified
together with <file>s, the files are edited in-place. Otherwise, the
result is written to the standard output.

USAGE: clang-format [options] [@<file>] [<file> ...]

OPTIONS:

Clang-format options:

  --Werror                       - If set, changes formatting warnings to errors
  --Wno-error=<value>            - If set don't error out on the specified warning type.
    =unknown                     -   If set, unknown format options are only warned about.
                                     This can be used to enable formatting, even if the
                                     configuration contains unknown (newer) options.
                                     Use with caution, as this might lead to dramatically
                                     differing format depending on an option being
                                     supported or not.
  --assume-filename=<string>     - Set filename used to determine the language and to find
                                   .clang-format file.
                                   Only used when reading from stdin.
                                   If this is not passed, the .clang-format file is searched
                                   relative to the current working directory when reading stdin.
                                   Unrecognized filenames are treated as C++.
                                   supported:
                                     CSharp: .cs
                                     Java: .java
                                     JavaScript: .mjs .js .ts
                                     Json: .json
                                     Objective-C: .m .mm
                                     Proto: .proto .protodevel
                                     TableGen: .td
                                     TextProto: .textpb .pb.txt .textproto .asciipb
                                     Verilog: .sv .svh .v .vh
  --cursor=<uint>                - The position of the cursor when invoking
                                   clang-format from an editor integration
  --dry-run                      - If set, do not actually make the formatting changes
  --dump-config                  - Dump configuration options to stdout and exit.
                                   Can be used with -style option.
  --fallback-style=<string>      - The name of the predefined style used as a
                                   fallback in case clang-format is invoked with
                                   -style=file, but can not find the .clang-format
                                   file to use. Defaults to 'LLVM'.
                                   Use -fallback-style=none to skip formatting.
  --ferror-limit=<uint>          - Set the maximum number of clang-format errors to emit
                                   before stopping (0 = no limit).
                                   Used only with --dry-run or -n
  --files=<filename>             - A file containing a list of files to process, one
                                   per line.
  -i                             - Inplace edit <file>s, if specified.
  --length=<uint>                - Format a range of this length (in bytes).
                                   Multiple ranges can be formatted by specifying
                                   several -offset and -length pairs.
                                   When only a single -offset is specified without
                                   -length, clang-format will format up to the end
                                   of the file.
                                   Can only be used with one input file.
  --lines=<string>               - <start line>:<end line> - format a range of
                                   lines (both 1-based).
                                   Multiple ranges can be formatted by specifying
                                   several -lines arguments.
                                   Can't be used with -offset and -length.
                                   Can only be used with one input file.
  -n                             - Alias for --dry-run
  --offset=<uint>                - Format a range starting at this byte offset.
                                   Multiple ranges can be formatted by specifying
                                   several -offset and -length pairs.
                                   Can only be used with one input file.
  --output-replacements-xml      - Output replacements as XML.
  --qualifier-alignment=<string> - If set, overrides the qualifier alignment style
                                   determined by the QualifierAlignment style flag
  --sort-includes                - If set, overrides the include sorting behavior
                                   determined by the SortIncludes style flag
  --style=<string>               - Set coding style. <string> can be:
                                   1. A preset: LLVM, GNU, Google, Chromium, Microsoft,
                                      Mozilla, WebKit.
                                   2. 'file' to load style configuration from a
                                      .clang-format file in one of the parent directories
                                      of the source file (for stdin, see --assume-filename).
                                      If no .clang-format file is found, falls back to
                                      --fallback-style.
                                      --style=file is the default.
                                   3. 'file:<format_file_path>' to explicitly specify
                                      the configuration file.
                                   4. "{key: value, ...}" to set specific parameters, e.g.:
                                      --style="{BasedOnStyle: llvm, IndentWidth: 8}"
  --verbose                      - If set, shows the list of processed files

Generic Options:

  --help                         - Display available options (--help-hidden for more)
  --help-list                    - Display list of available options (--help-list-hidden for more)
  --version                      - Display the version of this program
```

#### 自定义选项

clang-format支持两种方式提供自定义样式选项：直接在-style=命令行选项中指定样式配置；或使用-style=file，并将样式配置放在项目（根）目录下的`.clang-format`（or `_clang-format`）文件中。
创建`.clang-format`最简单的方式是基于内置的规则修改自己的规则，导出内置的规则命令示例如下：
```
clang-format --style=llvm -dump-config > .clang-format
```

当创建了自己的.clang-format文件后，可以使用如下方式格式化代码：
```
clang-format --style=file -i hello.cc
```

#### .clang-format配置示例

```
---
Language:        Cpp
BasedOnStyle:  Google
# 访问说明符(public、private等)的偏移
AccessModifierOffset: -2
# 开括号(开圆括号、开尖括号、开方括号)后的对齐: Align, DontAlign, AlwaysBreak(总是在开括号后换行)
AlignAfterOpenBracket: AlwaysBreak
# 数据对齐方式，clang-format 13支持
# AlignArrayOfStructures: Right
# 对齐连续宏定义
AlignConsecutiveMacros: false
# 连续赋值时，对齐所有等号
AlignConsecutiveAssignments: false
# 连续声明时，对齐所有声明的变量名
AlignConsecutiveDeclarations: false
# 逃脱换行(使用反斜杠换行)的反斜杠对齐方式
AlignEscapedNewlines: Left
# 水平对齐二元和三元表达式的操作数
AlignOperands: true
# 对齐连续的尾随的注释
AlignTrailingComments: true
# 允许所有参数在下一行
AllowAllArgumentsOnNextLine: true
# 允许所有构造函数初始值设定项在下一行
AllowAllConstructorInitializersOnNextLine: true
# 允许函数声明的所有参数在放在下一行
AllowAllParametersOfDeclarationOnNextLine: true
# 允许短的块放在同一行
AllowShortBlocksOnASingleLine: Never
# 允许短的case标签放在同一行
AllowShortCaseLabelsOnASingleLine: false
# 允许短的函数放在同一行: false, InlineOnly(定义在类中), Empty(空函数), Inline(定义在类中，空函数), All
AllowShortFunctionsOnASingleLine: false
# 允许短的Lambda表达式语句保持在同一行
AllowShortLambdasOnASingleLine: false
# 允许短的if语句保持在同一行
AllowShortIfStatementsOnASingleLine: WithoutElse
# 允许短的循环保持在同一行
AllowShortLoopsOnASingleLine: false
# 总是在定义返回类型后换行None, All, TopLevel(顶级函数，不包括在类中的函数),
AlwaysBreakAfterDefinitionReturnType: None
# 总是在return之后换行
AlwaysBreakAfterReturnType: None
# 总是在多行string字面量前换行
AlwaysBreakBeforeMultilineStrings: false
# 总是在template声明后换行
AlwaysBreakTemplateDeclarations: Yes
# false表示函数实参要么都在同一行，要么都各自一行
BinPackArguments: true
# false表示所有形参要么都在同一行，要么都各自一行
BinPackParameters: true
# bit定义（:）空格方式
BitFieldColonSpacing: After
# 大括号换行
# 如果“BreakBeforeBraces”设置为“BS_Custom”, 使用这个指定如何处理每个单独的括号的情况。否则，这是被忽略的。
BraceWrapping:
  # case标签后面
  AfterCaseLabel:  false
  # class定义后面
  AfterClass:      false
  # 控制语句(if/for/while/switch/..)后面换行。
  AfterControlStatement: false
  # enum定义后面
  AfterEnum:       false
  # 函数定义后面
  AfterFunction:   true
  # 命名空间定义后面
  AfterNamespace:  false
  # ObjC定义后面
  AfterObjCDeclaration: false
  # struct定义后面
  AfterStruct:     false
  # union定义后面
  AfterUnion:      false
  # Extern定义之后
  AfterExternBlock: false
  # catch之前
  BeforeCatch:     false
  # else之前
  BeforeElse:      false
  # 缩进大括号
  IndentBraces:    false
  # 分离空函数
  SplitEmptyFunction: true
  # 分离空语句
  SplitEmptyRecord: true
  # 分离空命名空间
  SplitEmptyNamespace: true
# 使二进制操作符换行的方法。
# 可能的值有：
#   BOS_None (在配置中： None) 在操作符后换行。
#   BOS_NonAssignment (在配置中： NonAssignment) 在操作符没有被指定前换行。
#   BOS_All (在配置中： All) 在操作符前换行。
BreakBeforeBinaryOperators: None
# 用于大括号换行样式。
# 可能的值有：
#   BS_Attach (在配置中： Attach) 总是将大括号与上下文连在一起。
#   BS_Linux (在配置中： Linux) 像Attach一样, 但是在一个方法、命名空间或一个类定义的大括号之前换行
#   BS_Mozilla (在配置中： Mozilla) 像Attach一样, 但是在一个枚举、方法或记录定义前换行。
#   BS_Stroustrup (在配置中： Stroustrup) 像Attach一样,但是在方法定义、catch、和else前换行
#   BS_Allman (在配置中： Allman) 总是在大括号之前换行。
#   BS_GNU (在配置中： GNU) 总是在括号前中断，并添加一个额外的级别的缩进到控件语句的括号中，而不是类、函数或其他定义的括号中。
#   BS_WebKit (在配置中： WebKit) 像Attach一样, 但是在方法前换行。
#   BS_Custom (在配置中： Custom) 在“BraceWrapping”里配置每一个单独的大括号。
BreakBeforeBraces: Attach
# 在继承符合（:）前换行
BreakBeforeInheritanceComma: false
# 继承列表样式
BreakInheritanceList: AfterColon
# 在三元运算符前换行
BreakBeforeTernaryOperators: false
# 在构造函数的初始化列表的逗号前换行
BreakConstructorInitializersBeforeComma: false
# 在构造函数的初始化列表的冒号前换行
BreakConstructorInitializers: AfterColon
# 在java字段的注释后换行
BreakAfterJavaFieldAnnotations: false
# 允许打破当格式化字符串。
BreakStringLiterals: false
# 每行字符的限制，0表示没有限制
ColumnLimit:     100
# 描述具有特殊意义的注释的正则表达式，它不应该被分割为多行或以其它方式改变
CommentPragmas:  '^ IWYU pragma:'
# 在新行上声明每个命名空间
CompactNamespaces: false
# false 构造函数的初始化列表要么都在同一行，要么都各自一行
ConstructorInitializerAllOnOneLineOrOnePerLine: false
# 构造函数的初始化列表的缩进宽度
ConstructorInitializerIndentWidth: 2
# 延续的行的缩进宽度
ContinuationIndentWidth: 4
# 去除C++11的列表初始化的大括号{后和}前的空格
Cpp11BracedListStyle: true
# 继承最常用的换行方式
DeriveLineEnding: true
# 继承最常用的指针和引用的对齐方式
DerivePointerAlignment: false
# 关闭格式化
DisableFormat:   false
# 在访问限定符前换行
EmptyLineBeforeAccessModifier: LogicalBlock
# 自动检测函数的调用和定义是否被格式为每行一个参数(Experimental)
ExperimentalAutoDetectBinPacking: false
# 是否自动添加命名空间结束注释
FixNamespaceComments: true
# 需要被解读为foreach循环而不是函数调用的宏
ForEachMacros:
  - foreach
  - Q_FOREACH
  - BOOST_FOREACH
IncludeBlocks:   Regroup
# 对#include进行排序，匹配了某正则表达式的#include拥有对应的优先级，匹配不到的则默认优先级为INT_MAX(优先级越小排序越靠前)，
# 可以定义负数优先级从而保证某些#include永远在最前面
IncludeCategories:
  - Regex:           '^<ext/.*\.h>'
    Priority:        2
    SortPriority:    0
  - Regex:           '^<.*\.h>'
    Priority:        1
    SortPriority:    0
  - Regex:           '^<.*'
    Priority:        2
    SortPriority:    0
  - Regex:           '.*'
    Priority:        3
    SortPriority:    0
IncludeIsMainRegex: '([-_](test|unittest))?$'
IncludeIsMainSourceRegex: ''
# 缩进case标签
IndentCaseLabels: true
# 缩进Extern C
IndentExternBlock: true
# 缩进goto标签
IndentGotoLabels: false
# 多层宏定义镶嵌时，缩进方式
IndentPPDirectives: None
# 缩进宽度
IndentWidth:     2
# 函数返回类型换行时，缩进函数声明或函数定义的函数名
IndentWrappedFunctionNames: false
# 保留JavaScript字符串引号
JavaScriptQuotes: Leave
# 包装 JavaScript 导入/导出语句
JavaScriptWrapImports: true
# 保留在块开始处的空行
KeepEmptyLinesAtTheStartOfBlocks: true
# 开始一个块的宏的正则表达式
MacroBlockBegin: ''
# 结束一个块的宏的正则表达式
MacroBlockEnd:   ''
# 连续空行的最大数量
MaxEmptyLinesToKeep: 1
# 命名空间的缩进: None, Inner(缩进嵌套的命名空间中的内容), All
NamespaceIndentation: None
# ObjC协议根据 ColumnLimit 长度换行 实现内部缩进宽度
ObjCBinPackProtocolList: Never
# 使用ObjC块时缩进宽度
ObjCBlockIndentWidth: 2
# 在ObjC的@property后添加一个空格
ObjCSpaceAfterProperty: false
# 在ObjC的protocol列表前添加一个空格
ObjCSpaceBeforeProtocolList: true
PenaltyBreakAssignment: 2
# 在call(后对函数调用换行的penalty
PenaltyBreakBeforeFirstCallParameter: 1
# 在一个注释中引入换行的penalty
PenaltyBreakComment: 300
# 第一次在<<前换行的penalty
PenaltyBreakFirstLessLess: 120
# 在一个字符串字面量中引入换行的penalty
PenaltyBreakString: 1000
PenaltyBreakTemplateDeclaration: 10
# 对于每个在行字符数限制之外的字符的penalty
PenaltyExcessCharacter: 1000000
# 将函数的返回类型放到它自己的行的penalty
PenaltyReturnTypeOnItsOwnLine: 200
# 指针和引用的对齐: Left, Right, Middle
PointerAlignment: Right
RawStringFormats:
  - Language:        Cpp
    Delimiters:
      - cc
      - CC
      - cpp
      - Cpp
      - CPP
      - 'c++'
      - 'C++'
    CanonicalDelimiter: ''
    BasedOnStyle:    google
  - Language:        TextProto
    Delimiters:
      - pb
      - PB
      - proto
      - PROTO
    EnclosingFunctions:
      - EqualsProto
      - EquivToProto
      - PARSE_PARTIAL_TEXT_PROTO
      - PARSE_TEST_PROTO
      - PARSE_TEXT_PROTO
      - ParseTextOrDie
      - ParseTextProtoOrDie
    CanonicalDelimiter: ''
    BasedOnStyle:    google
# 允许重新排版注释
ReflowComments:  true
# 拆分每一块的定义 clang-format14
# SeparateDefinitionBlocks: Leave
# 允许排序#include
SortIncludes: true
# 允许排序using
SortUsingDeclarations: true
# 在C风格类型转换后添加空格
SpaceAfterCStyleCast: false
# 在!后添加空格
SpaceAfterLogicalNot: false
# 在Template关键字后添加空格
SpaceAfterTemplateKeyword: false
# 在赋值运算符之前添加空格
SpaceBeforeAssignmentOperators: true
# 在C++11大括号列表之前添加空格
SpaceBeforeCpp11BracedList: false
# 在构造函数初始化器冒号之前添加空格
SpaceBeforeCtorInitializerColon: true
# 在继承冒号前添加空格
SpaceBeforeInheritanceColon: true
# 开圆括号之前添加一个空格: Never, ControlStatements, Always
SpaceBeforeParens: ControlStatements
# 在基于范围的for循环冒号之前添加空格
SpaceBeforeRangeBasedForLoopColon: true
# {}中间不添加空格
SpaceInEmptyBlock: false
# 在空的圆括号中添加空格
SpaceInEmptyParentheses: false
# 在尾随的评论前添加的空格数(只适用于//)
SpacesBeforeTrailingComments: 2
# 在尖括号的<后和>前添加空格
SpacesInAngles:  false
# 在if/for/switch/while条件周围插入空格
SpacesInConditionalStatement: false
# 在容器(ObjC和JavaScript的数组和字典等)字面量中添加空格
SpacesInContainerLiterals: false
# 在C风格类型转换的括号中添加空格
SpacesInCStyleCastParentheses: false
# 在圆括号的(后和)前添加空格
SpacesInParentheses: false
# 在方括号的[后和]前添加空格，lamda表达式和未指明大小的数组的声明不受影响
SpacesInSquareBrackets: false
# 在[前添加空格
SpaceBeforeSquareBrackets: false
# 标准
Standard:        Auto
# 应该被解释为完整语句的宏定义
StatementMacros:
  - Q_UNUSED
  - QT_REQUIRE_VERSION
# tab宽度
TabWidth:        2
# 使用\r\n换行(false: \n)
UseCRLF:         false
# 使用tab字符：ForIndentation——仅将制表符用于缩进
UseTab:          Never
...
```

