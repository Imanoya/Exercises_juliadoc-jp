<!-- Start -->

# [Modules](@id modules)

> # [Modules]

<!-- End -->
<!-- Start -->
Modules in Julia are separate variable workspaces, i.e. they introduce a new global scope.
> Juliaのモジュールは別々の変数ワークスペースです。つまり、新しいグローバルスコープを導入します。
<!-- End -->
<!-- Start -->
They are delimited syntactically, inside `module Name ... end`.
> それらは構文的に `module name ... end`の中で区切られています。
<!-- End -->
<!-- Start -->
Modules allow you to create top-level definitions (aka global variables) without worrying about name conflicts when your code is used together with somebody else's.
> モジュールは、コードが他の誰かと一緒に使用されるときに、名前の競合を心配することなく、トップレベルの定義（別名グローバル変数）を作成することを可能にします。
<!-- End -->
<!-- Start -->
Within a module, you can control which names from other modules are visible (via importing), and specify which of your names are intended to be public (via exporting).
> モジュール内では、他のモジュールのどの名前が（インポートによって）表示されるかを制御したり、名前を公開することを（エクスポートを介して）指定することができます。
<!-- End -->

<!-- Start -->
The following example demonstrates the major features of modules.
> 次の例は、モジュールの主な機能を示しています。
<!-- End -->
<!-- Start -->
It is not meant to be run, but is shown for illustrative purposes:
> これは実行されることを意味するのではなく、説明の目的で示されています。
<!-- End -->

```julia
module MyModule
using Lib

using BigLib: thing1, thing2

import Base.show

importall OtherLib

export MyType, foo

struct MyType
    x
end

bar(x) = 2x
foo(a::MyType) = bar(a.x) + 1

show(io::IO, a::MyType) = print(io, "MyType $(a.x)")
end
```

<!-- Start -->
Note that the style is not to indent the body of the module, since that would typically lead to whole files being indented.
> スタイルは、モジュールの本体をインデントしないことに注意してください。これは、通常、ファイル全体がインデントされるためです。
<!-- End -->

<!-- Start -->
This module defines a type `MyType`, and two functions.
> このモジュールは `MyType`型と2つの関数を定義します。
<!-- End -->
<!-- Start -->
Function `foo` and type `MyType` are exported, and so will be available for importing into other modules.
> `foo`関数と` MyType`関数がエクスポートされるので、他のモジュールにインポートすることもできます。
<!-- End -->
<!-- Start -->
Function `bar` is private to `MyModule`.
> 関数 `bar`は` MyModule`に対してprivateです。
<!-- End -->

<!-- Start -->
The statement `using Lib` means that a module called `Lib` will be available for resolving names as needed.
> `Libを使う` という文は、必要に応じて `Lib` という名前のモジュールを利用して名前を解決できることを意味します。
<!-- End -->
<!-- Start -->
When a global variable is encountered that has no definition in the current module, the system will search for it among variables exported by `Lib` and import it if it is found there.
> 現在のモジュールに定義されていないグローバル変数が見つかった場合、システムは `Lib 'によってエクスポートされた変数の中から変数を探し、そこに見つかったらインポートします。
<!-- End -->
<!-- Start -->
This means that all uses of that global within the current module will resolve to the definition of that variable in `Lib`.
> これは、現在のモジュール内のそのグローバルのすべての使用が `Lib` 内のその変数の定義に解決されることを意味します。
<!-- End -->

<!-- Start -->
The statement `using BigLib: thing1, thing2` is a syntactic shortcut for `using BigLib.thing1, BigLib.thing2`.
> `BigLib:thing1,thing2` を使う文は、 `BigLib.thing1,BigLib.thing2` を使うための構文上のショートカットです。
<!-- End -->

<!-- Start -->
The `import` keyword supports all the same syntax as `using`, but only operates on a single name at a time.
> `import`キーワードは、` using`と同じ構文をサポートしていますが、一度に一つの名前でしか動作しません。
<!-- End -->
<!-- Start -->
It does not add modules to be searched the way `using` does.
> `using`のように検索するモジュールを追加しません。
<!-- End -->
<!-- Start -->
`import` also differs from `using` in that functions must be imported using `import` to be extended with new methods.
> `import`は、新しいメソッドで拡張するために` import`を使って関数をインポートしなければならない点で `using`とは異なります。
<!-- End -->

<!-- Start -->
In `MyModule` above we wanted to add a method to the standard `show` function, so we had to write `import Base.show`.
> 上記の `MyModule`では、標準の` show`関数にメソッドを追加したいので、 `import Base.show`を書く必要がありました。
<!-- End -->
<!-- Start -->
Functions whose names are only visible via `using` cannot be extended.
> 名前が `using 'を通してしか見えない関数は、拡張することができません。
<!-- End -->

<!-- Start -->
The keyword `importall` explicitly imports all names exported by the specified module, as if `import` were individually used on all of them.
> `importall`というキーワードは、指定されたモジュールによってエクスポートされたすべての名前を明示的にインポートします。
<!-- End -->

<!-- Start -->
Once a variable is made visible via `using` or `import`, a module may not create its own variable with the same name.
> 変数が `using`や` import`を介して可視になると、モジュールは同じ名前の変数を作成しないかもしれません。
<!-- End -->
<!-- Start -->
Imported variables are read-only; assigning to a global variable always affects a variable owned by the current module, or else raises an error.
> インポートされた変数は読み取り専用です。 グローバル変数に割り当てることは、常に現在のモジュールが所有する変数に影響します。そうでない場合は、エラーが発生します。
<!-- End -->

<!-- Start -->

## Summary of module usage

<!-- End -->
<!-- Start -->
To load a module, two main keywords can be used: `using` and `import`.
> モジュールをロードするには、 `using`と` import`の2つの主要なキーワードを使用できます。
<!-- End -->
<!-- Start -->
To understand their differences, consider the following example:
> 相違点を理解するには、次の例を考えてください。
<!-- End -->

```julia
module MyModule

export x, y

x() = "x"
y() = "y"
p() = "p"

end
```

<!-- Start -->
In this module we export the `x` and `y` functions (with the keyword `export`), and also have the non-exported function `p`.
> このモジュールでは `x`と` y`関数（キーワード `export`）をエクスポートし、エクスポートされていない関数` p`も持っています。
<!-- End -->
<!-- Start -->
There are several different ways to load the Module and its inner functions into the current workspace:
> モジュールとその内部関数を現在のワークスペースにロードするには、いくつかの方法があります。
<!-- End -->

| Import Command                  | What is brought into scope                                                      | Available for method extension              |
|:------------------------------- |:------------------------------------------------------------------------------- |:------------------------------------------- |
| `using MyModule`                | All `export`ed names (`x` and `y`), `MyModule.x`, `MyModule.y` and `MyModule.p` | `MyModule.x`, `MyModule.y` and `MyModule.p` |
| `using MyModule.x, MyModule.p`  | `x` and `p`                                                                     |                                             |
| `using MyModule: x, p`          | `x` and `p`                                                                     |                                             |
| `import MyModule`               | `MyModule.x`, `MyModule.y` and `MyModule.p`                                     | `MyModule.x`, `MyModule.y` and `MyModule.p` |
| `import MyModule.x, MyModule.p` | `x` and `p`                                                                     | `x` and `p`                                 |
| `import MyModule: x, p`         | `x` and `p`                                                                     | `x` and `p`                                 |
| `importall MyModule`            | All `export`ed names (`x` and `y`)                                              | `x` and `y`                                 |

<!-- Start -->

### Modules and files

<!-- End -->
<!-- Start -->
Files and file names are mostly unrelated to modules; modules are associated only with module expressions.
> ファイルとファイル名は、モジュールとはほとんど関係ありません。 モジュールはモジュール式とのみ関連付けられます。
<!-- End -->
<!-- Start -->
One can have multiple files per module, and multiple modules per file:
> モジュールごとに複数のファイルを持ち、ファイルごとに複数のモジュールを持つことができます。
<!-- End -->

```julia
module Foo

include("file1.jl")
include("file2.jl")

end
```

<!-- Start -->
Including the same code in different modules provides mixin-like behavior.
> 異なるモジュールに同じコードを組み込むことで、mixinのような動作が実現します。
<!-- End -->
<!-- Start -->
One could use this to run the same code with different base definitions, for example testing code by running it with "safe" versions of some operators:
> これを使用して、異なる基本定義を持つ同じコードを実行することができます。たとえば、いくつかの演算子の「安全な」バージョンでコードを実行するなど、
<!-- End -->

```julia
module Normal
include("mycode.jl")
end

module Testing
include("safe_operators.jl")
include("mycode.jl")
end
```

<!-- Start -->

### Standard modules

<!-- End -->
<!-- Start -->
There are three important standard modules: Main, Core, and Base.
> 重要な標準モジュールには、Main、Core、Baseの3つがあります。
<!-- End -->

<!-- Start -->
Main is the top-level module, and Julia starts with Main set as the current module.  
> Mainはトップレベルのモジュールであり、JuliaはMainを現在のモジュールとして設定します。
<!-- End -->
<!-- Start -->
Variables defined at the prompt go in Main, and `whos()` lists variables in Main.
> プロンプトで定義された変数はMainにあり、 `whos（）`はMainの変数をリストします。
<!-- End -->

<!-- Start -->
Core contains all identifiers considered "built in" to the language, i.e. part of the core language and not libraries.
> コアには、言語に「組み込まれている」とみなされるすべての識別子、つまりライブラリではなくコア言語の一部が含まれます。
<!-- End -->
<!-- Start -->
Every module implicitly specifies `using Core`, since you can't do anything without those definitions.
> すべてのモジュールは暗黙的に `using Core`を指定しています。なぜならあなたはそれらの定義なしでは何もできないからです。
<!-- End -->

<!-- Start -->
Base is the standard library (the contents of base/).
> Baseは標準ライブラリ（base /の内容）です。
<!-- End -->
<!-- Start -->
All modules implicitly contain `using Base`, since this is needed in the vast majority of cases.
> すべてのモジュールは暗黙のうちに `using Base` を含んでいます。なぜならこれは大部分のケースで必要とされるからです。
<!-- End -->
<!-- Start -->

### Default top-level definitions and bare modules

<!-- End -->
<!-- Start -->
In addition to `using Base`, modules also automatically contain a definition of the `eval` function, which evaluates expressions within the context of that module.
> `Using Base`に加えて、モジュールは自動的に` eval`関数の定義を含んでいます。この関数は、そのモジュールのコンテキスト内で式を評価します。
<!-- End -->

<!-- Start -->
If these default definitions are not wanted, modules can be defined using the keyword `baremodule` instead (note: `Core` is still imported, as per above).
> これらのデフォルト定義が望ましくない場合は、代わりに `baremodule`というキーワードを使用してモジュールを定義することができます（上記のように` Core`は引き続きインポートされます）。 
<!-- End -->
<!-- Start -->
In terms of `baremodule`, a standard `module` looks like this:
> `baremodule` に関しては、標準の `module` は次のようになります：
<!-- End -->

```
baremodule Mod

using Base

eval(x) = Core.eval(Mod, x)
eval(m,x) = Core.eval(m, x)

...

end
```

<!-- Start -->

### Relative and absolute module paths

<!-- End -->
<!-- Start -->
Given the statement `using Foo`, the system looks for `Foo` within `Main`.
> `Fooを使う 'という文があると、システムは` Main`内で `Foo`を探します。
<!-- End -->
<!-- Start -->
If the module does not exist, the system attempts to `require("Foo")`, which typically results in loading code from an installed package.
> モジュールが存在しない場合、システムは `require（" Foo "）`を試行します。通常はインストールされたパッケージからコードをロードします。
<!-- End -->

<!-- Start -->
However, some modules contain submodules, which means you sometimes need to access a module that is not directly available in `Main`.
> しかし、モジュールの中にはサブモジュールが含まれているものもあります。つまり、 `Main`で直接利用できないモジュールにアクセスする必要があることがあります。
<!-- End -->
<!-- Start -->
There are two ways to do this. The first is to use an absolute path, for example `using Base.Sort`.
> これを行うには2つの方法があります。 第1の方法は絶対パスを使用することです。たとえば `using Base.Sort`です。
<!-- End -->
<!-- Start -->
The second is to use a relative path, which makes it easier to import submodules of the current module or any of its enclosing modules:
> 2つ目は相対パスを使用することです。これにより、現在のモジュールまたはそのモジュールを含むモジュールのサブモジュールを簡単にインポートできます。
<!-- End -->

```
module Parent

module Utils
...
end

using .Utils

...
end
```

<!-- Start -->
Here module `Parent` contains a submodule `Utils`, and code in `Parent` wants the contents of `Utils` to be visible.
> ここでモジュール `Parent`はサブモジュール` Utils`を含み、コード `Parent`は` Utils`の内容を可視化します。
<!-- End -->
<!-- Start -->
This is done by starting the `using` path with a period.
> これは、ピリオドで `using`パスを開始することによって行われます。
<!-- End -->
<!-- Start -->
Adding more leading periods moves up additional levels in the module hierarchy.
> 先頭の期間を追加すると、モジュール階層の追加レベルが上がります。
<!-- End -->
<!-- Start -->
For example `using ..Utils` would look for `Utils` in `Parent`'s enclosing module rather than in `Parent` itself.
> 例えば、 `using ..Utils` は、 `Parent` 自身ではなく `Parent` の囲みモジュールで `Utils` を探すでしょう。
<!-- End -->

<!-- Start -->
Note that relative-import qualifiers are only valid in `using` and `import` statements.
> 相対インポート修飾子は `using`ステートメントと` import`ステートメントでのみ有効であることに注意してください。
<!-- End -->
<!-- Start -->

### Module file paths

<!-- End -->
<!-- Start -->
The global variable [`LOAD_PATH`](@ref) contains the directories Julia searches for modules when calling `require`.
> グローバル変数[`LOAD_PATH`](@ref) には、 `require` を呼び出すときにジュリアがモジュールを検索するディレクトリが含まれています。
<!-- End -->
<!-- Start -->
It can be extended using [`push!`](@ref):
> [`push！`](@ref) を使って拡張することができます：
<!-- End -->

```julia
push!(LOAD_PATH, "/Path/To/My/Module/")
```

<!-- Start -->
Putting this statement in the file `~/.juliarc.jl` will extend [`LOAD_PATH`](@ref) on every Julia startup.
> この文を `~/.juliarc.jl` ファイルに入れると、ジュリアの起動時に [`LOAD_PATH`](@ref)が拡張されます。
<!-- End -->
<!-- Start -->
Alternatively, the module load path can be extended by defining the environment variable `JULIA_LOAD_PATH`.
> あるいは、環境変数 `JULIA_LOAD_PATH` を定義してモジュールロードパスを拡張することもできます。
<!-- End -->
<!-- Start -->

### Namespace miscellanea

<!-- End -->
<!-- Start -->
If a name is qualified (e.g. `Base.sin`), then it can be accessed even if it is not exported.
> 名前が修飾されている場合（例： `Base.sin`）、エクスポートされていなくてもアクセスできます。
<!-- End -->
<!-- Start -->
This is often useful when debugging.
> これは、デバッグ時に便利です。
<!-- End -->
<!-- Start -->
It can also have methods added to it by using the qualified name as the function name.
> また、修飾名を関数名として使用してメソッドを追加することもできます。
<!-- End -->
<!-- Start -->
However, due to syntactic ambiguities that arise, if you wish to add methods to a function in a different module whose name contains only symbols, such as an operator, `Base.+` for example, you must use `Base.:+` to refer to it.
> しかし、構文上のあいまいさが原因で、名前にシンボルのみが含まれている別のモジュール、たとえば、演算子 'Base。+'などの関数にメソッドを追加する場合は、 `Base：+` それを参照してください。
<!-- End -->
<!-- Start -->
If the operator is more than one character in length you must surround it in brackets, such as: `Base.:(==)`.
> 演算子が複数の文字である場合は、 `Base :( ==）`のように括弧で囲む必要があります。
<!-- End -->

<!-- Start -->
Macro names are written with `@` in import and export statements, e.g. `import Mod.@mac`.
> マクロ名は、インポートとエクスポートの文で `@` と書かれています。 `Mod.@mac` をインポートします。
<!-- End -->
<!-- Start -->
Macros in other modules can be invoked as `Mod.@mac` or `@Mod.mac`.
> 他のモジュールのマクロは、 `Mod.@mac` や `@Mod.mac` として呼び出すことができます。
<!-- End -->

<!-- Start -->
The syntax `M.x = y` does not work to assign a global in another module; global assignment is always module-local.
> `M.x = y` という構文は、別のモジュールにグローバルを割り当てるのには使えません。 グローバル割り当ては常にモジュールローカルです。
<!-- End -->

<!-- Start -->
A variable can be "reserved" for the current module without assigning to it by declaring it as `global x` at the top level.
> 変数は、トップレベルで `global x`として宣言することによって、現在のモジュールに対して"予約 "されていることはありません。
<!-- End -->
<!-- Start -->
This can be used to prevent name conflicts for globals initialized after load time.
> これは、読み込み後に初期化されたグローバルの名前の競合を防ぐために使用できます。
<!-- End -->
<!-- Start -->

### Module initialization and precompilation

<!-- End -->
<!-- Start -->
Large modules can take several seconds to load because executing all of the statements in a module often involves compiling a large amount of code.
> モジュール内のすべてのステートメントを実行すると、大量のコードをコンパイルすることが多いため、大きなモジュールには数秒かかることがあります。
<!-- End -->
<!-- Start -->
Julia provides the ability to create precompiled versions of modules to reduce this time.
> ジュリアは、この時間を短縮するために、コンパイルされたバージョンのモジュールを作成する機能を提供します。
<!-- End -->

<!-- Start -->
To create an incremental precompiled module file, add `__precompile__()` at the top of your module file (before the `module` starts).
> 増分プリコンパイルされたモジュールファイルを作成するには、モジュールファイルの先頭に（ `module` が起動する前に ）`__precompile__()` を追加します。
<!-- End -->
<!-- Start -->
This will cause it to be automatically compiled the first time it is imported.
> これにより、最初にインポートされるときに自動的にコンパイルされます。
<!-- End -->
<!-- Start -->
Alternatively, you can manually call `Base.compilecache(modulename)`.
> あるいは、 `Base.compilecache(modulename)`を手動で呼び出すこともできます。
<!-- End -->
<!-- Start -->
The resulting cache files will be stored in `Base.LOAD_CACHE_PATH[1]`.
> 結果キャッシュファイルは `Base.LOAD_CACHE_PATH [1]` に格納されます。
<!-- End -->
<!-- Start -->
Subsequently, the module is automatically recompiled upon `import` whenever any of its dependencies change; dependencies are modules it imports, the Julia build, files it includes, or explicit dependencies declared by `include_dependency(path)` in the module file(s).
> その後、依存関係のいずれかが変わるたびにモジュールは自動的に `import` で再コンパイルされます。 依存関係は、インポートするモジュール、Juliaビルド、それに含まれるファイル、またはモジュールファイルに `include_dependency(path)` で宣言された明示的な依存関係です。
<!-- End -->

<!-- Start -->
For file dependencies, a change is determined by examining whether the modification time (mtime) of each file loaded by `include` or added explicitly by `include_dependency` is unchanged, or equal to the modification time truncated to the nearest second (to accommodate systems that can't copy mtime with sub-second accuracy).
> ファイルの依存性については、変更は、 `include`によってロードされたファイルまたは` include_dependency`によって明示的に追加された各ファイルの変更時間（mtime）が変更されていないか、または最も近い秒に切り捨てられた変更時間 秒未満の精度でmtimeをコピーすることはできません）。
<!-- End -->
<!-- Start -->
It also takes into account whether the path to the file chosen by the search logic in `require` matches the path that had created the precompile file.
> 検索ロジックによって `require`で選択されたファイルへのパスがプリコンパイル・ファイルを作成したパスと一致するかどうかも考慮されます。
<!-- End -->

<!-- Start -->
It also takes into account the set of dependencies already loaded into the current process and won't recompile those modules, even if their files change or disappear, in order to avoid creating incompatibilities between the running system and the precompile cache.
> また、現在のプロセスにすでにロードされている依存関係のセットを考慮に入れ、実行中のシステムとプリコンパイル・キャッシュとの間の非互換性を避けるために、ファイルが変更または消滅してもそれらのモジュールを再コンパイルしません。
<!-- End -->
<!-- Start -->
If you want to have changes to the source reflected in the running system, you should call `reload("Module")` on the module you changed, and any module that depended on it in which you want to see the change reflected.
> 実行中のシステムに反映されたソースへの変更を加えたい場合は、変更したモジュール上で `reload（" Module "）を呼び出す必要があります。
<!-- End -->

<!-- Start -->
Precompiling a module also recursively precompiles any modules that are imported therein.
> また、モジュールをプリコンパイルすると、そこにインポートされたモジュールが再帰的にプリコンパイルされます。
<!-- End -->
<!-- Start -->
If you know that it is *not* safe to precompile your module (for the reasons described below), you should put `__precompile__(false)` in the module file to cause `Base.compilecache` to throw an error (and thereby prevent the module from being imported by any other precompiled module).
> モジュールをあらかじめコンパイルするのが安全でないことが分かっているならば、モジュールファイルに `__precompile __（false）`を入れて、 `Base.compilecache`がエラーを投げ出すようにする必要があります。 モジュールは他のプリコンパイルされたモジュールによってインポートされません）。
<!-- End -->

<!-- Start -->
`__precompile__()` should *not* be used in a module unless all of its dependencies are also using `__precompile__()`.
> `__precompile__()`は、その依存関係のすべてが `__precompile __()`も使っていない限り、モジュールで使われてはいけません。
<!-- End -->
<!-- Start -->
Failure to do so can result in a runtime error when loading the module.
> そうしないと、モジュールをロードするときにランタイムエラーが発生する可能性があります。
<!-- End -->

<!-- Start -->
In order to make your module work with precompilation, however, you may need to change your module to explicitly separate any initialization steps that must occur at *runtime* from steps that can occur at *compile time*.
> ただし、モジュールをプリコンパイルで動作させるには、* runtime *で発生する必要がある初期化ステップを*コンパイル時に発生する可能性のあるステップから明示的に分離するようにモジュールを変更する必要があります。
<!-- End -->
<!-- Start -->
For this purpose, Julia allows you to define an `__init__()` function in your module that executes any initialization steps that must occur at runtime.
> この目的のために、Juliaでは、実行時に発生する必要がある初期化ステップを実行するモジュール内の `__init __()` 関数を定義することができます。
<!-- End -->
<!-- Start -->
This function will not be called during compilation (`--output-*` or `__precompile__()`).
> この関数は、コンパイル時には呼び出されません（ `--output-*` または `__precompile __()`）。
You may, of course, call it manually if necessary, but the default is to assume this function deals with computing state for the local machine, which does not need to be – or even should not be – captured in the compiled image.
> もちろん、必要に応じて手動で呼び出すこともできますが、デフォルトでは、コンパイルされたイメージにキャプチャする必要はありませんし、必要でもないローカルマシンの計算状態を扱うと仮定することがデフォルトです。
It will be called after the module is loaded into a process, including if it is being loaded into an incremental compile (`--output-incremental=yes`), but not if it is being loaded into a full-compilation process.
> インクリメンタルコンパイル (`--output-incremental=yes`) にロードされている場合を含め、モジュールがプロセスにロードされた後で呼び出されますが、フルコンパイルプロセスにロードされている場合は含まれません。
<!-- End -->

<!-- Start -->
In particular, if you define a `function __init__()` in a module, then Julia will call `__init__()` immediately *after* the module is loaded (e.g., by `import`, `using`, or `require`) at runtime for the *first* time (i.e., `__init__` is only called once, and only after all statements in the module have been executed).
> 特に、モジュール内で `function __init __()`を定義した場合、モジュールは実行時に *最初に* ロードされ( `import`, `using`, または `require` ) た *後に即座に* `__init __()`を呼び出します。 （つまり、 `__init__`は一度だけ呼び出され、モジュール内のすべての文が実行された後にのみ呼び出されます）。
<!-- End -->
<!-- Start -->
Because it is called after the module is fully imported, any submodules or other imported modules have their `__init__` functions called *before* the `__init__` of the enclosing module.
> モジュールが完全にインポートされた後に呼び出されるため、サブモジュールやその他のインポートされたモジュールは、そのモジュールの `__init__` の前に呼び出される `__init__` 関数を持っています。
<!-- End -->

<!-- Start -->
Two typical uses of `__init__` are calling runtime initialization functions of external C libraries and initializing global constants that involve pointers returned by external libraries.
> `__init__` の2つの典型的な使い方は、外部Cライブラリのランタイム初期化関数を呼び出し、外部ライブラリから返されるポインタを含むグローバル定数を初期化することです。
<!-- End -->
<!-- Start -->
For example, suppose that we are calling a C library `libfoo` that requires us to call a `foo_init()` initialization function at runtime.
> 例えば、実行時に `foo_init()` 初期化関数を呼び出さなければならないCライブラリ `libfoo` を呼び出しているとします。
<!-- End -->
<!-- Start -->
Suppose that we also want to define a global constant `foo_data_ptr` that holds the return value of a `void *foo_data()` function defined by `libfoo` -- this constant must be initialized at runtime (not at compile time) because the pointer address will change from run to run.
> `libfoo` で定義された `void * foo_data()` 関数の戻り値を保持するグローバル定数 `foo_data_ptr` を定義したいとします。この定数は実行時（コンパイル時ではなく）で初期化する必要があります。 ポインタアドレスは実行ごとに変化します。
<!-- End -->
<!-- Start -->
You could accomplish this by defining the following `__init__` function in your module:
> モジュールで次の `__init__` 関数を定義することでこれを達成できます：
<!-- End -->

```julia
const foo_data_ptr = Ref{Ptr{Void}}(0)
function __init__()
    ccall((:foo_init, :libfoo), Void, ())
    foo_data_ptr[] = ccall((:foo_data, :libfoo), Ptr{Void}, ())
end
```

<!-- Start -->
Notice that it is perfectly possible to define a global inside a function like `__init__`; this is one of the advantages of using a dynamic language.
> `__init__`のような関数内でグローバルを定義することは完全に可能であることに注意してください。 これは動的言語を使用する利点の1つです。
<!-- End -->
<!-- Start -->
But by making it a constant at global scope, we can ensure that the type is known to the compiler and allow it to generate better optimized code.
> しかし、これをグローバルスコープで一定にすることで、型がコンパイラに認識され、より最適化されたコードを生成できるようにすることができます。
<!-- End -->
<!-- Start -->
Obviously, any other globals in your module that depends on `foo_data_ptr` would also have to be initialized in `__init__`.
> 明らかに、 `foo_data_ptr` に依存するモジュール内の他のグローバル変数も `__init__` で初期化する必要があります。
<!-- End -->

<!-- Start -->
Constants involving most Julia objects that are not produced by `ccall` do not need to be placed in `__init__`: their definitions can be precompiled and loaded from the cached module image.
> `ccall` によって生成されないほとんどのJuliaオブジェクトを含む定数は、`__init__` に置かれる必要はありません。それらの定義は、プリコンパイルされ、キャッシュされたモジュールイメージからロードされます。
<!-- End -->
<!-- Start -->
This includes complicated heap-allocated objects like arrays.
> これには、配列のような複雑なヒープ割り当てオブジェクトが含まれます。
<!-- End -->
<!-- Start -->
However, any routine that returns a raw pointer value must be called at runtime for precompilation to work (Ptr objects will turn into null pointers unless they are hidden inside an isbits object).
> しかし、未処理のポインタ値を返すルーチンは、実行時にプリコンパイルを実行するために呼び出さなければなりません（Ptrオブジェクトはisbitsオブジェクト内に隠されていない限り、NULLポインタになります）。
<!-- End -->
<!-- Start -->
This includes the return values of the Julia functions `cfunction` and `pointer`.
> これには、Julia関数 `cfunction` と `pointer` の戻り値が含まれます。
<!-- End -->

<!-- Start -->
Dictionary and set types, or in general anything that depends on the output of a `hash(key)` method, are a trickier case.
> ディクショナリとセットの型、または一般的には `hash(key)`メソッドの出力に依存するものは、より扱いにくいケースです。
<!-- End -->
<!-- Start -->
In the common case where the keys are numbers, strings, symbols, ranges, `Expr`, or compositions of these types (via arrays, tuples, sets, pairs, etc.) they are safe to precompile.
> キーが数字、文字列、シンボル、範囲、 `Expr` 、またはこれらの型のコンポジション（配列、タプル、セット、ペアなど）である一般的なケースでは、それらはプリコンパイルするのが安全です。
<!-- End -->
<!-- Start -->
However, for a few other key types, such as `Function` or `DataType` and generic user-defined types where you haven't defined a `hash` method, the fallback `hash` method depends on the memory address of the object (via its `object_id`) and hence may change from run to run.
> しかし、 `Function` や `DataType` や `hash` メソッドを定義していない一般的なユーザ定義型のような他のいくつかのキー型では、フォールバック `hash` メソッドはオブジェクトのメモリアドレスに依存します （その `object_id` を介して）、実行ごとに変更することができます。
<!-- End -->
<!-- Start -->
If you have one of these key types, or if you aren't sure, to be safe you can initialize this dictionary from within your `__init__` function.
> これらのキータイプのいずれかを持っているか、またはわからない場合は、 `__init__` 関数の中からこの辞書を初期化することができます。
<!-- End -->
<!-- Start -->
Alternatively, you can use the `ObjectIdDict` dictionary type, which is specially handled by precompilation so that it is safe to initialize at compile-time.
> あるいは、 `ObjectIdDict` ディクショナリ型を使用することもできます。この型は、コンパイル時に初期化するのが安全なように、プリコンパイルによって特別に処理されます。
<!-- End -->

<!-- Start -->
When using precompilation, it is important to keep a clear sense of the distinction between the compilation phase and the execution phase.
> プリコンパイルを使用する場合は、コンパイル・フェーズと実行フェーズの区別を明確にすることが重要です。
<!-- End -->
<!-- Start -->
In this mode, it will often be much more clearly apparent that Julia is a compiler which allows execution of arbitrary Julia code, not a standalone interpreter that also generates compiled code.
> このモードでは、Juliaがコンパイルされたコードを生成するスタンドアロンインタプリタではなく、任意のJuliaコードを実行できるコンパイラであることがはっきりしていることがよくあります。
<!-- End -->

<!-- Start -->
Other known potential failure scenarios include:
> その他の既知の潜在的な障害シナリオには、
<!-- End -->

<!-- Start -->
1. Global counters (for example, for attempting to uniquely identify objects) Consider the following code snippet:
> 1.グローバルカウンタ（たとえば、オブジェクトを一意に識別しようとする場合）次のコードスニペットを考えてみましょう。
<!-- End -->

   ```julia
   mutable struct UniquedById
       myid::Int
       let counter = 0
           UniquedById() = new(counter += 1)
       end
   end
   ```

<!-- Start -->
   while the intent of this code was to give every instance a unique id, the counter value is recorded at the end of compilation.
   > このコードの目的はすべてのインスタンスに一意のIDを与えることでしたが、コンパイルの最後にカウンタ値が記録されます。
<!-- End -->
<!-- Start -->
   All subsequent usages of this incrementally compiled module will start from that same counter value.
   > このインクリメンタルにコンパイルされたモジュールのその後の使用はすべて、同じカウンタ値から開始されます。
<!-- End -->

<!-- Start -->
   Note that `object_id` (which works by hashing the memory pointer) has similar issues (see notes on `Dict` usage below).
   > `object_id`（メモリポインタをハッシュすることで動作する）にも同様の問題があることに注意してください（下記の` Dict`の使用上の注意を参照してください）。
<!-- End -->

<!-- Start -->
   One alternative is to use a macro to capture [`@__MODULE__`](@ref) and store it alone with the current `counter` value, however, it may be better to redesign the code to not depend on this global state.
   > もう1つの方法は、マクロを使って[`@__MODULE__`](@ref) をキャプチャし、それを現在の `counter` 値で保存することです。しかし、このグローバル状態に依存しないようにコードを再設計する方が良いかもしれません。
<!-- End -->
<!-- Start -->
2. Associative collections (such as `Dict` and `Set`) need to be re-hashed in `__init__`.
   (In the future, a mechanism may be provided to register an initializer function.)
>2. 連想集合（ `Dict`や` Set`など）を `__init__`に再ハッシュする必要があります。
>  （将来、初期化関数を登録するためのメカニズムが提供されるかもしれません。）
<!-- End -->
<!-- Start -->
3. Depending on compile-time side-effects persisting through load-time.
> 3.ロード時まで持続するコンパイル時の副作用に依存します。
<!-- End -->
<!-- Start -->
   Example include: modifying arrays or other variables in other Julia modules; maintaining handles to open files or devices;
   > 例としては、他のJuliaモジュールの配列や他の変数を変更する; ファイルやデバイスを開くためのハンドルを維持する。
<!-- End -->
<!-- Start -->
   storing pointers to other system resources (including memory);
   > 他のシステムリソース（メモリを含む）へのポインタを格納する。
<!-- End -->
<!-- Start -->
4. Creating accidental "copies" of global state from another module, by referencing it directly instead of via its lookup path. For example, (in global scope):
> 4.他のモジュールからのグローバル状態の偶発的な「コピー」を、ルックアップパス経由ではなく直接参照することによって作成する。 たとえば、（グローバルスコープ内で）
<!-- End -->

   ```julia
   #mystdout = Base.STDOUT #= will not work correctly, since this will copy Base.STDOUT into this module =#
   # instead use accessor functions:
   getstdout() = Base.STDOUT #= best option =#
   # or move the assignment into the runtime:
   __init__() = global mystdout = Base.STDOUT #= also works =#
   ```

<!-- Start -->
Several additional restrictions are placed on the operations that can be done while precompiling code to help the user avoid other wrong-behavior situations:
> ユーザーが他の誤った動作状況を回避するためにコードをプリコンパイルしている間に実行できる操作には、いくつかの追加の制限があります。
<!-- End -->

<!-- Start -->
1. Calling [`eval`](@ref) to cause a side-effect in another module.
   This will also cause a warning to be emitted when the incremental precompile flag is set.
> 1. 別のモジュールで副作用を引き起こすために[`eval`](@ref) を呼び出す。
   > また、インクリメンタル・プリコンパイル・フラグが設定されているときに警告が出されます。
<!-- End -->
<!-- Start -->
2. `global const` statements from local scope after `__init__()` has been started (see issue #12010 for plans to add an error for this)
> 2. `__init __()` の後のローカルスコープからの `global const` 文が開始されました（これにエラーを追加する計画については、issue＃12010を参照してください）
<!-- End -->
<!-- Start -->
3. Replacing a module (or calling [`workspace()`](@ref)) is a runtime error while doing an incremental precompile.
> 3.モジュールを置き換える（または [`workspace()`](@ref)を呼び出す）ことは、インクリメンタルなプリコンパイルを実行する際の実行時エラーです。
<!-- End -->

<!-- Start -->
A few other points to be aware of:
> 注意すべき他のいくつかの点は次のとおりです。
<!-- End -->

<!-- Start -->
1. No code reload / cache invalidation is performed after changes are made to the source files themselves,
> 1. ソースファイル自体が変更された後、コードの再ロード/キャッシュ無効化は実行されません。
   (including by [`Pkg.update`](@ref)), and no cleanup is done after [`Pkg.rm`](@ref)
   > [`Pkg.update`](@ref) を含む、[`Pkg.rm`](@ref) の後にはクリーンアップは行われません)
<!-- End -->
<!-- Start -->
2. The memory sharing behavior of a reshaped array is disregarded by precompilation (each view gets
   its own copy)
> 2.再構成された配列のメモリ共有動作は、プリコンパイルによって無視されます（各ビューは それ自身のコピー）
<!-- End -->
<!-- Start -->
3. Expecting the filesystem to be unchanged between compile-time and runtime e.g. [`@__FILE__`](@ref)/`source_path()` to find resources at runtime, or the BinDeps `@checked_lib` macro.
> 3. コンパイル時と実行時との間でファイルシステムが変更されないことを期待する。 実行時にリソースを見つけるには [`@__FILE__`](@ref)/`source_path()`, `BinDeps` `@checked_lib` マクロを使います。
<!-- End -->
<!-- Start -->
   Sometimes this is unavoidable.
   > 時にはこれは避けられないこともあります。
<!-- End -->
<!-- Start -->
   However, when possible, it can be good practice to copy resources into the module at compile-time so they won't need to be found at runtime.
   > しかし、可能であれば、コンパイル時にリソースをモジュールにコピーして実行時にリソースを見つける必要はありません。
<!-- End -->
<!-- Start -->
4. `WeakRef` objects and finalizers are not currently handled properly by the serializer (this will be fixed in an upcoming release).
> 4. `WeakRef` オブジェクトとファイナライザは現在、シリアライザによって適切に処理されていません（これは、今後リリースされる予定です）。
<!-- End -->
<!-- Start -->
5. It is usually best to avoid capturing references to instances of internal metadata objects such as `Method`, `MethodInstance`, `MethodTable`, `TypeMapLevel`, `TypeMapEntry` and fields of those objects, as this can confuse the serializer and may not lead to the outcome you desire.
> 5.通常、MethodInstance、MethodInstance、MethodTable、TypeMapLevel、TypeMapEntry、およびこれらのオブジェクトのフィールドなどの内部メタデータオブジェクトのインスタンスへの参照をキャプチャすることを避けることが最善です。シリアライザと あなたが望む結果につながりかねます。
<!-- End -->
<!-- Start -->
   It is not necessarily an error to do this, but you simply need to be prepared that the system will try to copy some of these and to create a single unique instance of others.
   > これを行うことは必ずしも間違いではありませんが、システムがこれらのいくつかをコピーし、他の一意のインスタンスを作成するように準備する必要があります。
<!-- End -->

<!-- Start -->
It is sometimes helpful during module development to turn off incremental precompilation.
> 増分プリコンパイルをオフにすることは、モジュール開発中に役立つことがあります。
<!-- End -->
<!-- Start -->
The command line flag `--compilecache={yes|no}` enables you to toggle module precompilation on and off.
> コマンドラインフラグ `--compilecache = {yes | no}`を使うと、モジュールのプリコンパイルのオンとオフを切り替えることができます。
<!-- End -->
When Julia is started with `--compilecache=no` the serialized modules in the compile cache are ignored when loading modules and module dependencies.
> Juliaが `--compilecache = no`で起動されると、モジュールとモジュールの依存関係を読み込む際に、コンパイルキャッシュ内の直列化モジュールは無視されます。
<!-- End -->
`Base.compilecache()` can still be called manually and it will respect `__precompile__()` directives for the module.
> `Base.compilecache（）`は手動で呼び出すことができ、モジュールの `__precompile __（）`ディレクティブを尊重します。
<!-- End -->
The state of this command line flag is passed to [`Pkg.build()`](@ref) to disable automatic precompilation triggering when installing, updating, and explicitly building packages.
> このコマンドラインフラグの状態は、パッケージをインストール、更新、明示的にビルドする際の自動プリコンパイルトリガを無効にするために[`Pkg.build（）`]（@ ref）に渡されます。
<!-- End -->