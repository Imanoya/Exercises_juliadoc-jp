<!-- Start -->

# Interacting With Julia

> # ジュリアとの交流

<!-- End -->
<!-- Start -->
Julia comes with a full-featured interactive command-line REPL (read-eval-print loop) built into the `julia` executable. 
> Juliaには、 `julia`実行可能ファイルに組み込まれたフル機能のインタラクティブなコマンドラインREPL（read-eval-printループ）が付属しています。
<!-- End -->
<!-- Start -->
In addition to allowing quick and easy evaluation of Julia statements, it has a searchable history, tab-completion, many helpful keybindings, and dedicated help and shell modes. 
> Juliaステートメントの迅速かつ簡単な評価を可能にするだけでなく、検索可能な履歴、タブ補完、多くの有益なキーバインディング、専用のヘルプとシェルモードがあります。
<!-- End -->
<!-- Start -->
The REPL can be started by simply calling `julia` with no arguments or double-clicking on the executable:
> REPLは単に引数なしで `julia`を呼び出すか、実行可能ファイルをダブルクリックするだけで起動できます：
<!-- End -->

```
$ julia
               _
   _       _ _(_)_     |  A fresh approach to technical computing
  (_)     | (_) (_)    |  Documentation: https://docs.julialang.org
   _ _   _| |_  __ _   |  Type "?help" for help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 0.6.0-dev.2493 (2017-01-31 18:53 UTC)
 _/ |\__'_|_|_|\__'_|  |  Commit c99e12c* (0 days old master)
|__/                   |  x86_64-linux-gnu

julia>
```

<!-- Start -->
To exit the interactive session, type `^D` -- the control key together with the `d` key on a blank line -- or type `quit()` followed by the return or enter key. 
> インタラクティブセッションを終了するには、 `^ D ' - 空白行に` d`キーとともにコントロールキーを入力するか、 `quit（）`と続けてリターンキーまたはEnterキーを押します。
<!-- End -->
<!-- Start -->
The REPL greets you with a banner and a `julia>` prompt.
> REPLはバナーと `julia>`プロンプトであなたを迎えます。
<!-- End -->

<!-- Start -->

## The different prompt modes

<!-- End -->

## The Julian mode

> ## juliaモード

<!-- End -->
<!-- Start -->
The REPL has four main modes of operation. 
> REPLには4つの主な操作モードがあります。
<!-- End -->
<!-- Start -->
The first and most common is the Julian prompt. 
> 最も一般的なものはジュリアンプロンプトです。
<!-- End -->
<!-- Start -->
It is the default mode of operation; each new line initially starts with `julia>`. 
> これはデフォルトの動作モードです。 新しい行はそれぞれ最初に `julia>`で始まります。
<!-- End -->
<!-- Start -->
It is here that you can enter Julia expressions. 
> ジュリア式を入力することができます。
<!-- End -->
<!-- Start -->
Hitting return or enter after a complete expression has been entered will evaluate the entry and show the result of the last expression.
> 完全な式が入力された後、returnまたはenterを押すと、そのエントリが評価され、最後の式の結果が表示されます。
<!-- End -->

```jldoctest
julia> string(1 + 2)
"3"
```

<!-- Start -->
There are a number useful features unique to interactive work. 
> インタラクティブな仕事に特有の多くの便利な機能があります。
<!-- End -->
<!-- Start -->
In addition to showing the result, the REPL also binds the result to the variable `ans`. 
> 結果を表示することに加えて、REPLは結果を変数ansにバインドします。
<!-- End -->
<!-- Start -->
A trailing semicolon on the line can be used as a flag to suppress showing the result.
> 行の最後のセミコロンは、結果の表示を抑制するフラグとして使用できます。
<!-- End -->

```jldoctest
julia> string(3 * 4);

julia> ans
"12"
```

<!-- Start -->
In Julia mode, the REPL supports something called *prompt pasting*. 
> ジュリアモードでは、REPLは*プロンプト貼り付け*をサポートしています。
<!-- End -->
<!-- Start -->
This activates when pasting text that starts with `julia> ` into the REPL. 
> これは、 `julia>`で始まるテキストをREPLに貼り付けるときにアクティブになります。
<!-- End -->
<!-- Start -->
In that case, only expressions starting with `julia> ` are parsed, others are removed. 
> その場合、 `julia>`で始まる式だけが解析され、他のものは取り除かれます。
<!-- End -->
<!-- Start -->
This makes it is possible to paste a chunk of code that has been copied from a REPL session without having to scrub away prompts and outputs. 
> これにより、プロンプトと出力をスクラブすることなく、REPLセッションからコピーされたコードの塊を貼り付けることができます。
<!-- End -->
<!-- Start -->
This feature is enabled by default but can be disabled or enabled at will with `Base.REPL.enable_promptpaste(::Bool)`. 
> この機能はデフォルトで有効になっていますが、 `Base.REPL.enable_promptpaste（:: Bool）`で無効にしたり有効にすることができます。
<!-- End -->
<!-- Start -->
If it is enabled, you can try it out by pasting the code block above this paragraph straight into the REPL. 
> 有効になっている場合は、この段落の上にあるコードブロックをREPLに直接貼り付けることで試すことができます。
<!-- End -->
<!-- Start -->
This feature does not work on the standard Windows command prompt due to its limitation at detecting when a paste occurs.
> この機能は、標準のWindowsコマンドプロンプトでは動作しません。これは、ペーストが発生したときの検出に限界があるためです。
<!-- End -->
<!-- Start -->

## Help mode

<!-- End -->
<!-- Start -->
When the cursor is at the beginning of the line, the prompt can be changed to a help mode by typing `?`. 
> カーソルが行の先頭にある場合、プロンプトは `？`をタイプすることによってヘルプモードに変更することができます。
<!-- End -->
<!-- Start -->
Julia will attempt to print help or documentation for anything entered in help mode:
> ジュリアは、ヘルプモードで入力されたもののヘルプやドキュメントを印刷しようとします：
<!-- End -->

```julia-repl
julia> ? # upon typing ?, the prompt changes (in place) to: help?>

help?> string
search: string String stringmime Cstring Cwstring RevString readstring randstring bytestring SubString

  string(xs...)

  Create a string from any values using the print function.
```

<!-- Start -->
Macros, types and variables can also be queried:
> マクロ、型、および変数も照会できます。
<!-- End -->

```
help?> @time
  @time

  A macro to execute an expression, printing the time it took to execute, the number of allocations, and the total number of bytes its execution caused to be allocated, before returning the value of the expression.

  See also @timev, @timed, @elapsed, and @allocated.

help?> AbstractString
search: AbstractString AbstractSparseMatrix AbstractSparseVector AbstractSet

  No documentation found.

  Summary:

  abstract AbstractString <: Any

  Subtypes:

  Base.Test.GenericString
  DirectIndexString
  String
```

<!-- Start -->
Help mode can be exited by pressing backspace at the beginning of the line.
> ヘルプモードは、行頭でバックスペースを押すことで終了できます。
<!-- End -->

<!-- Start -->

### [Shell mode](@id man-shell-mode)

> ### [シェルモード]（@id マンシェルモード）

<!-- End -->
<!-- Start -->
Just as help mode is useful for quick access to documentation, another common task is to use the system shell to execute system commands. 
> ヘルプモードはドキュメントへの迅速なアクセスに便利ですが、システムシェルを使用してシステムコマンドを実行することも一般的な作業です。
<!-- End -->
<!-- Start -->
Just as `?` entered help mode when at the beginning of the line, a semicolon (`;`) will enter the shell mode. And it can be exited by pressing backspace at the beginning of the line.
> `?` が行の先頭にあるときにヘルプモードに入ったのと同様に、セミコロン（`;`） がシェルモードに入ります。 また、行頭でバックスペースを押すと終了することができます。
<!-- End -->

```julia-repl
julia> ; # upon typing ;, the prompt changes (in place) to: shell>

shell> echo hello
hello
```

<!-- Start -->

### Search modes

> ### 検索モード

<!-- End -->
<!-- Start -->
In all of the above modes, the executed lines get saved to a history file, which can be searched.
> 上記のすべてのモードでは、実行された行は、検索可能な履歴ファイルに保存されます。
<!-- End -->
<!-- Start -->
To initiate an incremental search through the previous history, type `^R` -- the control key together with the `r` key. 
> 以前の履歴を使ってインクリメンタル検索を開始するには、 `r`キー（` r`キーと共にコントロールキー）を入力します。
<!-- End -->
<!-- Start -->
The prompt will change to ```(reverse-i-search)`':```, and as you type the search query will appear in the quotes. 
> プロンプトは `` `（reverse-i-search）` `：` ``に変わり、あなたが入力すると検索クエリが引用符で表示されます。
<!-- End -->
<!-- Start -->
The most recent result that matches the query will dynamically update to the right of the colon as more is typed. To find an older result using the same query, simply type `^R` again.
> クエリに一致する最新の結果は、より多くの型が入力されると、コロンの右側に動的に更新されます。 同じクエリを使用して古い結果を見つけるには、もう一度 `^ R`と入力してください。
<!-- End -->

<!-- Start -->
Just as `^R` is a reverse search, `^S` is a forward search, with the prompt ```(i-search)`':```.
> `^R` が逆方向検索であるのと同様に `^S` は前方検索で、 ```(i-search)`': ``` となります。
<!-- End -->
<!-- Start -->
The two may be used in conjunction with each other to move through the previous or next matching results, respectively.
> これらの2つは、前回の一致結果または次の一致結果をそれぞれ移動するために、互いに組み合わせて使用されてもよい。
<!-- End -->

<!-- Start -->

## Key bindings

> ## キーバインディング

<!-- End -->
<!-- Start -->
The Julia REPL makes great use of key bindings. Several control-key bindings were already introduced above (`^D` to exit, `^R` and `^S` for searching), but there are many more. 
> Julia REPLはキーバインディングを最大限に活用しています。 上にはいくつかのコントロールキーバインディングが既に導入されています（ `^ D`を終了するには` ^ R`と `^ S`を検索します）。
<!-- End -->
<!-- Start -->
In addition to the control-key, there are also meta-key bindings. 
> コントロールキーに加えて、メタキーバインディングもあります。
<!-- End -->
<!-- Start -->
These vary more by platform, but most terminals default to using alt- or option- held down with a key to send the meta-key (or can be configured to do so).
> これらはプラットフォームによって異なりますが、ほとんどの端末はデフォルトでaltキーまたはoptionキーを押しながらメタキーを送信するためにキーを使用します（またはこれを行うように構成することもできます）。
<!-- End -->

| Keybinding          | Description                                                                      |
|:------------------- |:-------------------------------------------------------------------------------- |
| **Program control** |                                                                                  |
| `^D`                | Exit (when buffer is empty)                                                      |
| `^C`                | Interrupt or cancel                                                              |
| `^L`                | Clear console screen                                                             |
| Return/Enter, `^J`  | New line, executing if it is complete                                            |
| meta-Return/Enter   | Insert new line without executing it                                             |
| `?` or `;`          | Enter help or shell mode (when at start of a line)                               |
| `^R`, `^S`          | Incremental history search, described above                                      |
| **Cursor movement** |                                                                                  |
| Right arrow, `^F`   | Move right one character                                                         |
| Left arrow, `^B`    | Move left one character                                                          |
| Home, `^A`          | Move to beginning of line                                                        |
| End, `^E`           | Move to end of line                                                              |
| `^P`                | Change to the previous or next history entry                                     |
| `^N`                | Change to the next history entry                                                 |
| Up arrow            | Move up one line (or to the previous history entry)                              |
| Down arrow          | Move down one line (or to the next history entry)                                |
| Page-up             | Change to the previous history entry that matches the text before the cursor     |
| Page-down           | Change to the next history entry that matches the text before the cursor         |
| `meta-F`            | Move right one word                                                              |
| `meta-B`            | Move left one word                                                               |
| **Editing**         |                                                                                  |
| Backspace, `^H`     | Delete the previous character                                                    |
| Delete, `^D`        | Forward delete one character (when buffer has text)                              |
| meta-Backspace      | Delete the previous word                                                         |
| `meta-D`            | Forward delete the next word                                                     |
| `^W`                | Delete previous text up to the nearest whitespace                                |
| `^K`                | "Kill" to end of line, placing the text in a buffer                              |
| `^Y`                | "Yank" insert the text from the kill buffer                                      |
| `^T`                | Transpose the characters about the cursor                                        |
| `^Q`                | Write a number in REPL and press `^Q` to open editor at corresponding stackframe or method |


<!-- Start -->

### Customizing keybindings

> ### キーバインディングをカスタマイズする

<!-- End -->

<!-- Start -->
Julia's REPL keybindings may be fully customized to a user's preferences by passing a dictionary to `REPL.setup_interface()`. 
> JuliaのREPLキーバインドは、辞書を `REPL.setup_interface（）`に渡すことによって、ユーザーの好みに完全にカスタマイズできます。
<!-- End -->
<!-- Start -->
The keys of this dictionary may be characters or strings. 
> この辞書のキーは文字または文字列です。
<!-- End -->
<!-- Start -->
The key `'*'` refers to the default action. Control plus character `x` bindings are indicated with `"^x"`.
> キー `` * ''は、デフォルトのアクションを参照します。 コントロールと文字の `x`バインディングは、` `^ x" `で示されます。
<!-- End -->
<!-- Start -->
Meta plus `x` can be written `"\\Mx"`. The values of the custom keymap must be `nothing` (indicating that the input should be ignored) or functions that accept the signature `(PromptState, AbstractREPL, Char)`.
> メタプラス `x`は` `\\ Mx" `と書くことができます。 カスタムキーマップの値は `nothing`（入力を無視することを示す）か、`（PromptState、AbstractREPL、Char）という署名を受け入れる関数でなければなりません。
<!-- End -->
<!-- Start -->
The `REPL.setup_interface()` function must be called before the REPL is initialized, by registering the operation with `atreplinit()`. 
> `REPL.setup_interface（）`関数は、REPLが初期化される前に `atreplinit（）`で操作を登録することによって呼び出さなければなりません。
<!-- End -->
<!-- Start -->
For example, to bind the up and down arrow keys to move through history without prefix search, one could put the following code in `.juliarc.jl`:
> たとえば、上下の矢印キーをバインドして、プレフィックス検索を行わずに履歴を移動するには、次のコードを `.juliarc.jl`に入れます。
<!-- End -->

```julia
import Base: LineEdit, REPL

const mykeys = Dict{Any,Any}(
    # Up Arrow
    "\e[A" => (s,o...)->(LineEdit.edit_move_up(s) || LineEdit.history_prev(s, LineEdit.mode(s).hist)),
    # Down Arrow
    "\e[B" => (s,o...)->(LineEdit.edit_move_up(s) || LineEdit.history_next(s, LineEdit.mode(s).hist))
)

function customize_keys(repl)
    repl.interface = REPL.setup_interface(repl; extra_repl_keymap = mykeys)
end

atreplinit(customize_keys)
```

<!-- Start -->
Users should refer to `LineEdit.jl` to discover the available actions on key input.
> ユーザは `LineEdit.jl`を参照して、キー入力で利用可能なアクションを発見する必要があります。
<!-- End -->

<!-- Start -->

### Tab completion

### > タブ補完

<!-- End -->
<!-- Start -->
In both the Julian and help modes of the REPL, one can enter the first few characters of a function or type and then press the tab key to get a list all matches:
> REPLのジュリアンモードとヘルプモードの両方で、関数またはタイプの最初の数文字を入力してから、タブキーを押してすべてのマッチをリストにすることができます：
<!-- End -->

```julia-repl
julia> stri[TAB]
stride     strides     string      stringmime  strip

julia> Stri[TAB]
StridedArray    StridedMatrix    StridedVecOrMat  StridedVector    String
```

<!-- Start -->
The tab key can also be used to substitute LaTeX math symbols with their Unicode equivalents, and get a list of LaTeX matches as well:
> タブキーはまた、LaTeXの数学記号をそれらのUnicodeの同等物に置き換えて、LaTeXのマッチのリストを取得するためにも使用できます：
<!-- End -->

```julia-repl
julia> \pi[TAB]
julia> π
π = 3.1415926535897...

julia> e\_1[TAB] = [1,0]
julia> e₁ = [1,0]
2-element Array{Int64,1}:
 1
 0

julia> e\^1[TAB] = [1 0]
julia> e¹ = [1 0]
1×2 Array{Int64,2}:
 1  0

julia> \sqrt[TAB]2     # √ is equivalent to the sqrt() function
julia> √2
1.4142135623730951

julia> \hbar[TAB](h) = h / 2\pi[TAB]
julia> ħ(h) = h / 2π
ħ (generic function with 1 method)

julia> \h[TAB]
\hat              \hermitconjmatrix  \hkswarow          \hrectangle
\hatapprox        \hexagon           \hookleftarrow     \hrectangleblack
\hbar             \hexagonblack      \hookrightarrow    \hslash
\heartsuit        \hksearow          \house             \hspace

julia> α="\alpha[TAB]"   # LaTeX completion also works in strings
julia> α="α"
```

<!-- Start -->
A full list of tab-completions can be found in the [Unicode Input](@ref) section of the manual.
> タブ補完の完全なリストは、マニュアルの[Unicode入力]（@ ref）セクションにあります。
<!-- End -->

<!-- Start -->
Completion of paths works for strings and julia's shell mode:
> 文字列とjuliaのシェルモードでは、パスの補完が完了します：
<!-- End -->

```julia-repl
julia> path="/[TAB]"
.dockerenv  .juliabox/   boot/        etc/         lib/         media/       opt/         root/        sbin/        sys/         usr/
.dockerinit bin/         dev/         home/        lib64/       mnt/         proc/        run/         srv/         tmp/         var/
shell> /[TAB]
.dockerenv  .juliabox/   boot/        etc/         lib/         media/       opt/         root/        sbin/        sys/         usr/
.dockerinit bin/         dev/         home/        lib64/       mnt/         proc/        run/         srv/         tmp/         var/
```

<!-- Start -->
Tab completion can help with investigation of the available methods matching the input arguments:
> タブの補完は、入力引数と一致する利用可能なメソッドの調査に役立ちます。
<!-- End -->

```julia-repl
julia> max([TAB] # All methods are displayed, not shown here due to size of the list

julia> max([1, 2], [TAB] # All methods where `Vector{Int}` matches as first argument
max(x, y) in Base at operators.jl:215
max(a, b, c, xs...) in Base at operators.jl:281

julia> max([1, 2], max(1, 2), [TAB] # All methods matching the arguments.
max(x, y) in Base at operators.jl:215
max(a, b, c, xs...) in Base at operators.jl:281
```

<!-- Start -->
Keywords are also displayed in the suggested methods, see second line after `;` where `limit` and `keep` are keyword arguments:
> キーワードはまた提案された方法で表示されます。 `;`の後ろの2行目を参照してください。ここで、 `limit`と` keep`はキーワード引数です：
<!-- End -->

```julia-repl
julia> split("1 1 1", [TAB]
split(str::AbstractString) in Base at strings/util.jl:302
split(str::T, splitter; limit, keep) where T<:AbstractString in Base at strings/util.jl:277
```

<!-- Start -->
The completion of the methods uses type inference and can therefore see if the arguments match even if the arguments are output from functions. 
> メソッドの完成は型推論を使用するので、引数が関数から出力されても引数が一致するかどうかを見ることができます。
<!-- End -->
<!-- Start -->
The function needs to be type stable for the completion to be able to remove non-matching methods.
> この関数は、一致しないメソッドを削除できるようにするためには、型を安定させる必要があります。
<!-- End -->

<!-- Start -->
Tab completion can also help completing fields:
> タブの補完は、フィールドの補完にも役立ちます。
<!-- End -->

```julia-repl
julia> Pkg.a[TAB]
add       available
```

<!-- Start -->
Fields for output from functions can also be completed:
> 関数からの出力のためのフィールドも完成することができます：
<!-- End -->

```julia-repl
julia> split("","")[1].[TAB]
endof  offset  string
```

<!-- Start -->
The completion of fields for output from functions uses type inference, and it can only suggest fields if the function is type stable.
> 関数からの出力のためのフィールドの完成は型推論を使用し、関数が型安定である場合にのみフィールドを提案することができます。
<!-- End -->
<!-- Start -->

### Customizing Colors

> ### カラーのカスタマイズ

<!-- End -->
<!-- Start -->
The colors used by Julia and the REPL can be customized, as well. 
> JuliaとREPLが使用する色はカスタマイズすることもできます。
<!-- End -->
<!-- Start -->
To change the color of the Julia prompt you can add something like the following to your `.juliarc.jl` file, which is to be placed inside your home directory:
> Juliaプロンプトの色を変更するには、あなたのホームディレクトリの中に置かれる `.juliarc.jl`ファイルに以下のようなものを追加することができます：
<!-- End -->

```julia
function customize_colors(repl)
    repl.prompt_color = Base.text_colors[:cyan]
end

atreplinit(customize_colors)
```

<!-- Start -->
The available color keys can be seen by typing `Base.text_colors` in the help mode of the REPL.
> 使用可能なカラーキーは、REPLのヘルプモードで `Base.text_colors`とタイプすることで見ることができます。
<!-- End -->
<!-- Start -->
In addition, the integers 0 to 255 can be used as color keys for terminals with 256 color support.
> さらに、整数0〜255は、256色のサポートを持つ端末のカラーキーとして使用できます。
<!-- End -->

<!-- Start -->
You can also change the colors for the help and shell prompts and input and answer text by setting the appropriate field of `repl` in the `customize_colors` function above (respectively, `help_color`, `shell_color`, `input_color`, and `answer_color`). 
> 上記の `customize_colors`関数（` help_color`、 `shell_color`、` input_color`、および `help_color`）に` repl`の適切なフィールドを設定することによって、ヘルプとシェルのプロンプトと入力と回答のテキストの色を変更することもできます。 answer_color`）。
<!-- End -->
<!-- Start -->
For the latter two, be sure that the `envcolors` field is also set to false.
> 後者の2つについては、 `envcolors`フィールドもfalseに設定されていることを確認してください。
<!-- End -->

<!-- Start -->
It is also possible to apply boldface formatting by using `Base.text_colors[:bold]` as a color. For instance, to print answers in boldface font, one can use the following as a `.juliarc.jl`:
> `Base.text_colors [：bold]`を色として使って太字の書式を適用することも可能です。 例えば、太字のフォントで回答を印刷するには、以下を `.juliarc.jl`として使うことができます：
<!-- End -->

```julia
function customize_colors(repl)
    repl.envcolors = false
    repl.answer_color = Base.text_colors[:bold]
end

atreplinit(customize_colors)
```

<!-- Start -->
You can also customize the color used to render warning and informational messages by setting the appropriate environment variables. 
> 適切な環境変数を設定することによって、警告および情報メッセージの表示に使用する色をカスタマイズすることもできます。
<!-- End -->
<!-- Start -->
For instance, to render error, warning, and informational messages respectively in magenta, yellow, and cyan you can add the following to your `.juliarc.jl` file:
> たとえば、エラーメッセージ、警告メッセージ、および情報メッセージをそれぞれマゼンタ、イエロー、シアンで表示するには、 `.juliarc.jl`ファイルに次のものを追加します：
<!-- End -->

```julia
ENV["JULIA_ERROR_COLOR"] = :magenta
ENV["JULIA_WARN_COLOR"] = :yellow
ENV["JULIA_INFO_COLOR"] = :cyan
```
