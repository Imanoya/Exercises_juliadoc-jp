<!-- Start -->

# Noteworthy Differences from other Languages

> # 他の言語との相違点

<!-- End -->
<!-- Start -->

## Noteworthy differences from MATLAB

> ## MATLABとの相違点

<!-- End -->
<!-- Start -->
Although MATLAB users may find Julia's syntax familiar, Julia is not a MATLAB clone.
> MATLABのユーザは、Juliaの構文をよく知っているかもしれませんが、JuliaはMATLABのクローンではありません。
<!-- End -->
<!-- Start -->
There are major syntactic and functional differences.
> 構文的および機能的に大きな違いがあります。
<!-- End -->
<!-- Start -->
The following are some noteworthy differences that may trip up Julia users accustomed to MATLAB:-->
> 以下は、MATLABに慣れているJuliaユーザーがつまずきやすいので注意するべき違いです。
<!-- End -->

<!-- Start -->
  * Julia arrays are indexed with square brackets, `A[i,j]`.
  * ジュリア配列は角括弧A [i、j]で索引付けされます。
<!-- End -->
* Julia arrays are assigned by reference. After `A=B`, changing elements of `B` will modify `A` as well.
* > ジュリアアレイは参照により割り当てられる。 `A = B`の後、` B`の要素を変更すると `A`も変更されます。
<!-- End -->
<!-- Start -->
* Julia values are passed and assigned by reference. If a function modifies an array, the changes will be visible in the caller.
* > ジュリア値は、渡され、参照によって割り当てられます。関数が配列を変更すると、変更が呼び出し側に表示されます。
<!-- End -->
<!-- Start -->
* Julia does not automatically grow arrays in an assignment statement. Whereas in MATLAB `a(4) = 3.2` can create the array `a = [0 0 0 3.2]` and `a(5) = 7` can grow it into `a = [0 0 0 3.2 7]`, the corresponding Julia statement `a[5] = 7` throws an error if the length of `a` is less than 5 or if this statement is the first use of the identifier `a`. 
* > Juliaは代入文で配列を自動的に拡張しません。 MATLAB `a（4）= 3.2`では配列` a = [0 0 0 3.2] `を作成でき、` a（5）= 7`では `a = [0 0 0 3.2 7]`に展開できますが、対応するJuliaステートメント `a [5] = 7`は、` a`の長さが5未満の場合、またはこのステートメントが識別子 `a`の最初の使用である場合にエラーをスローします。
<!-- End -->
<!-- Start -->
* Julia has [`push!()`](@ref) and [`append!()`](@ref), which grow `Vector`s much more efficiently than MATLAB's `a(end+1) = val`.
* > JuliaはMATLABの `a（end + 1）= val`よりはるかに効率的に` Vector`を成長させる[`push！（）`]（@ref）と[`append！（）`]（@ref）を持っています。
<!-- End -->
<!-- Start -->
* The imaginary unit `sqrt(-1)` is represented in Julia as [`im`](@ref), not `i` or `j` as in MATLAB.
* > 虚数単位 `sqrt（-1）`はJuliaでMATLABのように `i`または` j`ではなく[`im`]（@ref）として表されます。
<!-- End -->
<!-- Start -->
* In Julia, literal numbers without a decimal point (such as `42`) create integers instead of floating point numbers. 
* > Juliaでは、小数点のないリテラル数字（ `42`など）は浮動小数点数の代わりに整数を作成します。
<!-- End -->
<!-- Start -->
* Arbitrarily large integer literals are supported. As a result, some operations such as `2^-1` will throw a domain error as the result is not an integer (see [the FAQ entry on domain errors](@ref faq-domain-errors) for details).
* > 任意の大きな整数リテラルがサポートされています。その結果、 `2 ^ -1`のようないくつかの操作は、結果が整数ではないのでドメインエラーをスローします（詳細については[ドメインエラーに関するFAQエントリ]（@ref faq-domain-errors）参照）。
<!-- End -->
<!-- Start -->
* In Julia, multiple values are returned and assigned as tuples, e.g. `(a, b) = (1, 2)` or `a, b = 1, 2`.
* > Juliaでは、複数の値が返され、タプルとして割り当てられます。 `（a、b）=（1,2）`または `a、b = 1,2 'である。
<!-- End -->
<!-- Start -->
* MATLAB's `nargout`, which is often used in MATLAB to do optional work based on the number of returned values, does not exist in Julia. 
* > MATLABの `nargout`は、返された値の数に基づいてオプションの作業を行うためにMATLABでよく使用されますが、Juliaでは存在しません。
<!-- End -->
<!-- Start -->
* Instead, users can use optional and keyword arguments to achieve similar capabilities.
* > 代わりに、ユーザーはオプション引数とキーワード引数を使用して同様の機能を実現できます。
<!-- End -->
<!-- Start -->
* Julia has true one-dimensional arrays. 
* > Juliaは真の1次元配列を持っています。
<!-- End -->
<!-- Start -->
* Column vectors are of size `N`, not `Nx1`. 
* > 列ベクトルは「Nx1」ではなく「N」サイズである。
<!-- End -->
<!-- Start -->
* For example, [`rand(N)`](@ref) makes a 1-dimensional array.
* > たとえば、[`rand（N）`]（@ ref）は1次元配列を作成します。
<!-- End -->
<!-- Start -->
*  In Julia, `[x,y,z]` will always construct a 3-element array containing `x`, `y` and `z`.
* > Juliaでは、 `[x、y、z]`は常に `x`、` y`、 `z`を含む3要素配列を作成します。
<!-- End -->
<!-- Start -->
  - To concatenate in the first ("vertical") dimension use either [`vcat(x,y,z)`](@ref) or separate with       semicolons (`[x; y; z]`).
  > - 最初の（ "垂直"）次元で連結するには、[`vcat（x、y、z）`]（@ ref）かセミコロン（ `[x; y; z]`）で区切ります。
<!-- End -->
<!-- Start -->
  - To concatenate in the second ("horizontal") dimension use either [`hcat(x,y,z)`](@ref) or separate with spaces (`[x y z]`).
  > - 第2（ "水平"）次元で連結するには、[hcat（x、y、z）]（@ ref）かスペースで区切る（ `[x y z]`）のいずれかを使用します。
<!-- End -->
<!-- Start -->
  - To construct block matrices (concatenating in the first two dimensions), use either [`hvcat()`](@ref) or combine spaces and semicolons (`[a b; c d]`).
  > - ブロック行列（最初の2つの次元を連結）を構築するには、[`hvcat（）`]（@ ref）か空白とセミコロン（ `[a b; c d]`）を組み合わせます。
<!-- End -->
<!-- Start -->
*  In Julia, `a:b` and `a:b:c` construct `Range` objects. To construct a full vector like in MATLAB, use [`collect(a:b)`](@ref). 
* > Juliaでは `a：b`と` a：b：c`は `Range`オブジェクトを構成します。 MATLABのように完全なベクトルを構築するには、[`collect（a：b）`]（@ ref）を使います。
<!-- End -->
<!-- Start -->
* Generally, there is no need to call `collect` though. `Range` will act like a normal array in most cases but is more efficient because it lazily computes its values.
* > 一般的には、「収集」を呼び出す必要はありません。 `Range`はほとんどの場合通常の配列のように動作しますが、遅延を遅く計算するので効率的です。
<!-- End -->
<!-- Start -->
*  This pattern of creating specialized objects instead of full arrays is used frequently, and is also seen in functions such as [`linspace`](@ref), or with iterators such as `enumerate`, and `zip`. 
* > 完全配列の代わりに特殊なオブジェクトを作成するこのパターンは頻繁に使用され、[`linspace`]（@ ref）や` enumerate`や `zip`のようなイテレータでも見られます。
<!-- End -->
<!-- Start -->
*  The special objects can mostly be used as if they were normal arrays.
* > 特別なオブジェクトは、通常の配列であるかのように使用することができます。
<!-- End -->
<!-- Start -->
*  Functions in Julia return values from their last expression or the `return` keyword instead of listing the names of variables to return in the function definition (see [The return Keyword](@ref) for details).
* > Juliaの関数は、関数定義で返される変数の名前をリストする代わりに、最後の式または `return`キーワードから値を返します（詳細は[The return Keyword]（@ ref）を参照してください）。
<!-- End -->
<!-- Start -->
*  A Julia script may contain any number of functions, and all definitions will be externally visible when the file is loaded. 
* > Juliaスクリプトには任意の数の関数を含めることができ、ファイルがロードされるとすべての定義が外部に表示されます。
<!-- End -->
<!-- Start -->
*  Function definitions can be loaded from files outside the current working directory.
* > 関数定義は、現在の作業ディレクトリ外のファイルからロードすることができます。
<!-- End -->
<!-- Start -->
*  In Julia, reductions such as [`sum()`](@ref), [`prod()`](@ref), and [`max()`](@ref) are performed over every element of an array when called with a single argument, as in `sum(A)`, even if `A` has more than one dimension.
* > Juliaでは、[`sum（）`]（@ ref）、[`prod（）`]（@ ref）、[`max（）`]（@ ref）などの縮小は配列の各要素に対して実行されます`A`に複数の次元がある場合でも、` sum（A） `のように単一の引数で呼び出されたとき。
<!-- End -->
<!-- Start -->
*  In Julia, functions such as [`sort()`](@ref) that operate column-wise by default (`sort(A)` is equivalent to `sort(A,1)`) do not have special behavior for `1xN` arrays; the argument is returned unmodified since it still performs `sort(A,1)`. 
* > Juliaでは、デフォルトで `sort（A）`と並行して動作する[`sort（）`]（@ref）のような関数は、 `sort（A、1）`と同じですが、 1×N 'アレイ;引数はまだ `sort（A、1）`を実行するので変更されずに返されます。
<!-- End -->
<!-- Start -->
*  To sort a `1xN` matrix like a vector, use `sort(A,2)`.
* >  `1xN`行列をベクトルのようにソートするには、` sort（A、2） `を使います。
<!-- End -->
<!-- Start -->
*  In Julia, parentheses must be used to call a function with zero arguments, like in [`tic()`](@ref) and [`toc()`](@ref).
* > Juliaでは、[`tic（）`]（@ ref）や[`toc（）`]（@ ref）のように、引数がゼロの関数を呼び出すために括弧を使用する必要があります。
<!-- End -->
<!-- Start -->
*  Julia discourages the used of semicolons to end statements. 
* >  ジュリアは、ステートメントを終了するためにセミコロンの使用を嫌う。
<!-- End -->
<!-- Start -->
*  The results of statements are not automatically printed (except at the interactive prompt), and lines of code do not need to end with semicolons. 
* >  ステートメントの結果は自動的には表示されません（対話式のプロンプトを除いて）。コード行はセミコロンで終わる必要はありません。
<!-- End -->
<!-- Start -->
*  [`println()`](@ref) or [`@printf()`](@ref) can be used to print specific output.
* >   [`println（）`]（@ref）または[`@printf（）`]（@ref）は、特定の出力を出力するために使用できます。
<!-- End -->
<!-- Start -->
*  In Julia, if `A` and `B` are arrays, logical comparison operations like `A == B` do not return an array of booleans. 
* >  Juliaでは、 `A`と` B`が配列の場合、 `A == B`のような論理比較演算はブール値の配列を返しません。
<!-- End -->
<!-- Start -->
*  Instead, use `A .== B`, and similarly for the other boolean operators like [`<`](@ref), [`>`](@ref) and `=`.
* >  代わりに、`A .== B`を使い、` `<` `（@ ref）、` `>`]（@ ref）、 `=`のような他のブール演算子でも同様に使います。
<!-- End -->
<!-- Start -->
*  In Julia, the operators [`&`](@ref), [`|`](@ref), and [`⊻`](@ref xor) ([`xor`](@ref)) perform the bitwise operations equivalent to `and`, `or`, and `xor` respectively in MATLAB, and have precedence similar to Python's bitwise operators (unlike C). 
* >  Juliaでは、演算子[`＆`]（@ ref）、[`|`]（@ ref）、[`⊻`]（@ref xor）（[` xor`]（@ ref））は、 MATLABではそれぞれ `and`、`または `xor`に相当する演算を持ち、Pythonのビット演算子に似た優先順位を持っています（Cとは異なります）。
<!-- End -->
<!-- Start -->
*  They can operate on scalars or element-wise across arrays and can be used to combine logical arrays, but note the difference in order of operations:
* >  それらはスカラーまたは要素間で配列を操作することができ、論理配列を結合するのに使用できますが、操作の順序の違いに注意してください。
<!-- End -->
<!-- Start -->
*  parentheses may be required (e.g., to select elements of `A` equal to 1 or 2 use `(A .== 1) | (A .== 2)`).
* >  （例えば、1または2に等しい「A」の要素を選択するためには、（A。== 1）|（A。== 2）を使用することができる）。
<!-- End -->
<!-- Start -->
*  In Julia, the elements of a collection can be passed as arguments to a function using the splat operator `...`, as in `xs=[1,2]; f(xs...)`.
* >  Juliaでは、コレクションの要素は `xs = [1,2];のようにsplat演算子` ... `を使って関数に引数として渡すことができます。 f（xs ...） `である。
<!-- End -->
<!-- Start -->
*  Julia's [`svd()`](@ref) returns singular values as a vector instead of as a dense diagonal matrix.
* >  Juliaの[`svd（）`]（@ ref）は、密な対角行列ではなく、特異値をベクトルとして返します。
<!-- End -->
<!-- Start -->
*  In Julia, `...` is not used to continue lines of code. Instead, incomplete expressions automatically continue onto the next line.
* >  Juliaでは、 `...`はコード行を続けるのに使われません。代わりに、不完全な式が自動的に次の行に続きます。
<!-- End -->
<!-- Start -->
*  In both Julia and MATLAB, the variable `ans` is set to the value of the last expression issued in an interactive session. 
* >  JuliaとMATLABの両方で、変数ansは対話セッションで発行された最後の式の値に設定されます。
<!-- End -->
<!-- Start -->
*  In Julia, unlike MATLAB, `ans` is not set when Julia code is run in non-interactive mode.
* >  ジュリアでは、MATLABとは異なり、ジュリアコードが非インタラクティブモードで実行されているとき、 `ans`は設定されません。
<!-- End -->
<!-- Start -->
*  Julia's `type`s do not support dynamically adding fields at runtime, unlike MATLAB's `class`es. Instead, use a [`Dict`](@ref).
* >  Juliaの `type`は、MATLABの` class`とは異なり、実行時にフィールドを動的に追加することをサポートしていません。代わりに、[`Dict`]（@ ref）を使用してください。
<!-- End -->
<!-- Start -->
*  In Julia each module has its own global scope/namespace, whereas in MATLAB there is just one global scope.
* > Juliaでは、各モジュールには独自のグローバルスコープ/名前空間がありますが、MATLABではグローバルスコープは1つだけです。
<!-- End -->
<!-- Start -->
*  In MATLAB, an idiomatic way to remove unwanted values is to use logical indexing, like in the expression `x(x>3)` or in the statement `x(x>3) = []` to modify `x` in-place. 
* >  MATLABでは、望ましくない値を取り除く慣習的な方法は、 `x（x> 3）`や `x（x> 3）= []`という式のような論理的なインデックスを使用して `場所。
<!-- End -->
<!-- Start -->
*  In contrast, Julia provides the higher order functions [`filter()`](@ref) and [`filter!()`](@ref), allowing users to write `filter(z->z>3, x)` and `filter!(z->z>3, x)` as alternatives to the corresponding transliterations `x[x.>3]` and `x = x[x.>3]`. Using [`filter!()`](@ref) reduces the use of temporary arrays.
* >  これとは対照的にJuliaは高次関数[`filter（）`]（@ref）と[`filter！（）`]（@ref）を提供し、 `filter（z-> z> 3、対応する変換子 `x [x。> 3]`と `x = x [x。> 3]`の代わりに `filter！（z-> z> 3、x）`を使用します。 [`filter！（）`]（@ ref）を使うと、一時配列の使用を減らすことができます。
<!-- End -->
<!-- Start -->
*  The analogue of extracting (or "dereferencing") all elements of a cell array, e.g. in `vertcat(A{:})` in MATLAB, is written using the splat operator in Julia, e.g. as `vcat(A...)`.
* > セル配列のすべての要素を抽出（または「間接参照」）するアナログ。 MATLABの `vertcat（A {：}）`ではJuliaのsplat演算子を使って書かれています。 `vcat（A ...）`と同じです。
<!-- End -->
<!-- Start -->

## Noteworthy differences from R

> ## Rとの特筆すべき違い

<!-- End -->
<!-- Start -->
One of Julia's goals is to provide an effective language for data analysis and statistical programming.
> Juliaの目標の1つは、データ分析と統計プログラミングのための効果的な言語を提供することです。
<!-- End -->
<!-- Start -->
For users coming to Julia from R, these are some noteworthy differences:-->
> JuliaへRから来るユーザーにとり、次のようないくつかの違いがあります。
<!-- End -->
* Julia's single quotes enclose characters, not strings.
* > Juliaの一重引用符は文字列ではなく文字を囲みます。
<!-- End -->
<!-- Start -->
* Julia can create substrings by indexing into strings. In R, strings must be converted into character vectors before creating substrings.
* > Juliaは文字列にインデックスを付けて部分文字列を作成できます。 Rでは、部分文字列を作成する前に文字列を文字ベクトルに変換する必要があります。
<!-- End -->
<!-- Start -->
*  In Julia, like Python but unlike R, strings can be created with triple quotes `""" ... """`. 
* > Juliaでは、Pythonと同様ですがRとは違って、文字列は三重引用符 "" "..." ""で作成できます。
<!-- End -->
<!-- Start -->
* This syntax is convenient for constructing strings that contain line breaks.
* > この構文は、改行を含む文字列を作成するのに便利です。
<!-- End -->
<!-- Start -->
*  In Julia, varargs are specified using the splat operator `...`, which always follows the name of a specific variable, unlike R, for which `...` can occur in isolation.
* > Juliaでは、varargsはスプラット演算子 `...`を使って指定されます。これはRとは異なり、常に特定の変数の名前に従います。 `...`は単独で発生します。
<!-- End -->
<!-- Start -->
*  In Julia, modulus is `mod(a, b)`, not `a %% b`. `%` in Julia is the remainder operator.
* > Juliaでは、modulusは `a %% b`ではなく`mod（a、b）`です。 Juliaの `％`は剰余演算子です。
<!-- End -->
<!-- Start -->
* In Julia, not all data structures support logical indexing. 
* > Juliaでは、すべてのデータ構造が論理インデックス作成をサポートしているわけではありません。
<!-- End -->
<!-- Start -->
* Furthermore, logical indexing in Julia is supported only with vectors of length equal to the object being indexed. 
* > さらに、Juliaでの論理的な索引付けは、索引付けされるオブジェクトと等しい長さのベクトルでのみサポートされます。
<!-- End -->
<!-- Start -->
* For example:
* > 例えば：
<!-- End -->
<!-- Start -->

  * In R, `c(1, 2, 3, 4)[c(TRUE, FALSE)]` is equivalent to `c(1, 3)`.
  * > Rでは、 `c（1,2,3,4）[c（TRUE、FALSE）]`は `c（1,3）`と等価です。
<!-- End -->
<!-- Start -->
  * In R, `c(1, 2, 3, 4)[c(TRUE, FALSE, TRUE, FALSE)]` is equivalent to `c(1, 3)`.
  * Rでは `c（1,2,3,4）[c（TRUE、FALSE、TRUE、FALSE）]`は `c（1,3）`に相当します。
<!-- End -->
<!-- Start -->
  * In Julia, `[1, 2, 3, 4][[true, false]]` throws a [`BoundsError`](@ref).
  * Juliaでは、[1,2,3,4] [[true、false]] `[BoundsError`]（@ ref）をスローします。
<!-- End -->
<!-- Start -->
  * In Julia, `[1, 2, 3, 4][[true, false, true, false]]` produces `[1, 3]`.
  * ジュリアでは、[1,2,3,4] [[真、偽、真、偽]]は `[1,3]`を生成します。
<!-- End -->
<!-- Start -->
* Like many languages, Julia does not always allow operations on vectors of different lengths, unlike R where the vectors only need to share a common index range.  
* > 多くの言語と同様、Juliaはベクトルが共通の索引範囲を共有する必要があるRとは異なり、長さの異なるベクトルの操作を常に許可するとは限りません。
<!-- End -->
<!-- Start -->
* For example, `c(1, 2, 3, 4) + c(1, 2)` is valid R but the equivalent `[1, 2, 3, 4] + [1, 2]` will throw an error in Julia.
* > 例えば、 `c（1,2,3,4）+ c（1,2）`は有効なRですが、 `[1,2,3,4] + [1,2]`と同等のものは、ジュリア。
<!-- End -->
<!-- Start -->
* Julia's [`map()`](@ref) takes the function first, then its arguments, unlike `lapply(<structure>, function, ...)` in R. 
* > Juliaの[`map（）`]（@ ref）は、Rの `lapply（<structure>、function、...）`とは異なり、関数を最初にとり、引数をとります。
<!-- End -->
<!-- Start -->
* Similarly Julia's equivalent of `apply(X, MARGIN, FUN, ...)` in R is [`mapslices()`](@ref) where the function is the first argument.
* >  同様に、Juliaの `apply（X、MARGIN、FUN、...） 'は、関数が最初の引数である[mapslices（）`]（@ ref）です。
<!-- End -->
<!-- Start -->
* Multivariate apply in R, e.g. `mapply(choose, 11:13, 1:3)`, can be written as `broadcast(binomial, 11:13, 1:3)` in Julia. 
* > 多変量はRに適用されます（例： `mapply（choose、11:13、1：3）`は、Juliaに `broadcast（binomial、11:13、1：3）`と書くことができます。
<!-- End -->
<!-- Start -->
* Equivalently Julia offers a shorter dot syntax for vectorizing functions `binomial.(11:13, 1:3)`.
* > 等価的にJuliaは関数binimial（11：13,1：3）をベクトル化するために短いドット構文を提供しています。
<!-- End -->
<!-- Start -->
* Julia uses `end` to denote the end of conditional blocks, like `if`, loop blocks, like `while`/ `for`, and functions. 
* > Juliaは `if`のような条件付きブロックの終わりを表すために` end`を使い、 `for` /` for`のようなループブロックと関数を使用します。
<!-- End -->
<!-- Start -->
* In lieu of the one-line `if ( cond ) statement`, Julia allows statements of the form `if cond; statement; end`, `cond && statement` and `!cond || statement`. 
* > 1行の `if（cond）文`の代わりに、Juliaは `if cond;という形式の文を許可します。ステートメント; `cond`と` statement`と `！cond ||ステートメント `。
<!-- End -->
<!-- Start -->
* Assignment statements in the latter two syntaxes must be explicitly wrapped in parentheses, e.g. `cond && (x = value)`.
* > 後者の2つの構文の代入文は、明示的にかっこで囲む必要があります。 `cond &&（x = value）`です。
<!-- End -->
<!-- Start -->
* In Julia, `<-`, `<<-` and `->` are not assignment operators.
* > Juliaでは、 `< - `、 `<< - `と ` - >`は代入演算子ではありません。
<!-- End -->
<!-- Start -->
* Julia's `->` creates an anonymous function, like Python.
* > Juliaの ` - >`は、Pythonのような無名関数を作ります。
<!-- End -->
<!-- Start -->
* Julia constructs vectors using brackets. Julia's `[1, 2, 3]` is the equivalent of R's `c(1, 2, 3)`.
* > Juliaは括弧を使ってベクトルを構築します。 Juliaの `[1,2,3]`は、Rの `c（1,2,3） 'に相当します。
<!-- End -->
<!-- Start -->
* Julia's [`*`](@ref) operator can perform matrix multiplication, unlike in R. 
* > Juliaの[`*`]（@ ref）演算子は、Rとは異なり、行列乗算を実行できます。
<!-- End -->
<!-- Start -->
* If `A` and `B` are matrices, then `A * B` denotes a matrix multiplication in Julia, equivalent to R's `A %*% B`.
* > AとBが行列の場合、A * BはJuliaの行列乗算であり、RのA％*％Bと等価である。
<!-- End -->
<!-- Start -->
* In R, this same notation would perform an element-wise (Hadamard) product. 
* > Rでは、この同じ表記法が要素単位（アダマール）の製品を実行します。
<!-- End -->
<!-- Start -->
* To get the element-wise multiplication operation, you need to write `A .* B` in Julia.
* > 要素単位の乗算演算を行うには、Juliaに `A。* B`と書く必要があります。
<!-- End -->
<!-- Start -->
* Julia performs matrix transposition using the `.'` operator and conjugated transposition using the `'` operator. 
* > Juliaは `。 '演算子と` ``演算子を使った共役転置を使って行列の転置を行います。
<!-- End -->
<!-- Start -->
* Julia's `A.'` is therefore equivalent to R's `t(A)`.
* > Juliaの `A 'はRの` t（A） `と等価です。
<!-- End -->
<!-- Start -->
* Julia does not require parentheses when writing `if` statements or `for`/`while` loops: use `for i in [1, 2, 3]` instead of `for (i in c(1, 2, 3))` and `if i == 1` instead of `if (i == 1)`.
* > Juliaは `if`文や` for` / `while`ループを書くときに括弧を必要としません。for（i in c（1,2,3））の代わりに` for 1 in [1,2,3] `と` if if i == 1 'の代わりに `if i == 1'を返します。
<!-- End -->
<!-- Start -->
* Julia does not treat the numbers `0` and `1` as Booleans. You cannot write `if (1)` in Julia, because `if` statements accept only booleans. 
* > ジュリアは数字「0」と「1」をブーリアンとして扱いません。 Juliaで `if（1）`を書くことはできません。なぜなら、 `if`文はブール値しか受け入れないからです。
<!-- End -->
<!-- Start -->
* Instead, you can write `if true`, `if Bool(1)`, or `if 1==1`.
* > 代わりに、if true、if Bool（1）、またはif 1 == 1と書くことができます。
<!-- End -->
<!-- Start -->
* Julia does not provide `nrow` and `ncol`. Instead, use `size(M, 1)` for `nrow(M)` and `size(M, 2)` for `ncol(M)`.
* > Juliaは `nrow`と` ncol`を提供しません。代わりに `ncol（M）`に `nrow（M）`と `size（M、2）`に `size（M、1）`を使います。
<!-- End -->
<!-- Start -->
* Julia is careful to distinguish scalars, vectors and matrices.  In R, `1` and `c(1)` are the same. 
* > Juliaは、スカラー、ベクトル、行列を区別するように注意しています。 Rでは、 `1`と` c（1） `は同じです。
<!-- End -->
<!-- Start -->
* In Julia, they can not be used interchangeably. 
* > ジュリアでは、それらは交換可能に使用することはできません。
<!-- End -->
<!-- Start -->
* One potentially confusing result of this is that `x' * y` for vectors `x` and `y` is a 1-element vector, not a scalar. 
* > これの潜在的に混乱している結果の1つは、ベクトルxとyの `x '* y`はスカラではなく1要素のベクトルであるということです。
<!-- End -->
<!-- Start -->
* To get a scalar, use [`dot(x, y)`](@ref).
* > スカラーを取得するには、[`dot（x、y）`]（@ ref）を使います。
<!-- End -->
<!-- Start -->
*  Julia's [`diag()`](@ref) and [`diagm()`](@ref) are not like R's.
* > Juliaの[`diag（）`]（@ ref）と[`diagm（）`]（@ ref）はRのようなものではありません。
<!-- End -->
<!-- Start -->
* Julia cannot assign to the results of function calls on the left hand side of an assignment operation: you cannot write `diag(M) = ones(n)`.
* > Juliaは代入演算の左辺で関数呼び出しの結果に代入することはできません：あなたは `diag（M）= ones（n）`を書くことはできません。
<!-- End -->
<!-- Start -->
* Julia discourages populating the main namespace with functions. 
* > Juliaはメインネームスペースに関数を取り込むことを嫌う。
<!-- End -->
<!-- Start -->
Most statistical functionality for Julia is found in [packages](http://pkg.julialang.org/) under the [JuliaStats organization](https://github.com/JuliaStats). 
* > Juliaの統計機能は、[JuliaStats organization]（https://github.com/JuliaStats）の[packages]（http://pkg.julialang.org/）にあります。
<!-- End -->
<!-- Start -->
  For example:
* > 例えば：
<!-- End -->
<!-- Start -->
  * Functions pertaining to probability distributions are provided by the [Distributions package](https://github.com/JuliaStats/Distributions.jl).
  * > 確率分布に関する関数は、[Distribution packages]（https://github.com/JuliaStats/Distributions.jl）によって提供されています。
<!-- End -->
<!-- Start -->
  * The [DataFrames package](https://github.com/JuliaStats/DataFrames.jl) provides data frames.
  * > [DataFramesパッケージ]（https://github.com/JuliaStats/DataFrames.jl）はデータフレームを提供します。
<!-- End -->
<!-- Start -->
  * Generalized linear models are provided by the [GLM package](https://github.com/JuliaStats/GLM.jl).
  * > 一般化線形モデルは[GLMパッケージ]（https://github.com/JuliaStats/GLM.jl）によって提供されています。
<!-- End -->
<!-- Start -->
*  Julia provides tuples and real hash tables, but not R-style lists. When returning multiple items, you should typically use a tuple: instead of `list(a = 1, b = 2)`, use `(1, 2)`.
* >  Juliaはタプルと実際のハッシュテーブルを提供しますが、Rスタイルリストは提供しません。複数の項目を返すときは、通常、 `list（a = 1、b = 2）`の代わりに `（1,2）`を使うタプルを使うべきです。
<!-- End -->
<!-- Start -->
*  Julia encourages users to write their own types, which are easier to use than S3 or S4 objects in R. 
* >  Juliaは、ユーザが独自のタイプを書くことを奨励しています。これは、RのS3またはS4オブジェクトよりも使いやすいです。
<!-- End -->
<!-- Start -->
  Julia's multiple dispatch system means that `table(x::TypeA)` and `table(x::TypeB)` act like R's `table.TypeA(x)` and `table.TypeB(x)`.
* >  Juliaの複数のディスパッチシステムは、 `table（x :: TypeA）`と `table（x :: TypeB）`がRの `table.TypeA（x）`や `table.TypeB（x）`のように動作することを意味します。
<!-- End -->
<!-- Start -->
*  In Julia, values are passed and assigned by reference. If a function modifies an array, the changes will be visible in the caller. 
* >  Juliaでは、値が渡され、参照によって割り当てられます。関数が配列を変更すると、変更が呼び出し側に表示されます。
<!-- End -->
<!-- Start -->
  This is very different from R and allows new functions to operate on large data structures much more efficiently.
* >  これはRとは非常に異なり、大規模なデータ構造で新しい関数をはるかに効率的に操作できるようにします。
<!-- End -->
<!-- Start -->
*  In Julia, vectors and matrices are concatenated using [`hcat()`](@ref), [`vcat()`](@ref) and [`hvcat()`](@ref), not `c`, `rbind` and `cbind` like in R.
* >  Juliaでは、ベクトルと行列は、 `` hcat（） ``（@ ref）、 `` vcat（） ``（@ ref）と `` hvcat（） `（@ ref） Rのように `rbind`と` cbind`です。
<!-- End -->
<!-- Start -->
*  In Julia, a range like `a:b` is not shorthand for a vector like in R, but is a specialized `Range` that is used for iteration without high memory overhead. 
* >  Juliaでは `a：b`のような範囲はRのようなベクトルの省略形ではありませんが、高いメモリオーバーヘッドなしに反復に使用される特殊な` Range`です。
<!-- End -->
<!-- Start -->
  To convert a range into a vector, use [`collect(a:b)`](@ref).
* >  範囲をベクトルに変換するには、[`collect（a：b）`]（@ ref）を使います。
<!-- End -->
<!-- Start -->
*  Julia's [`max()`](@ref) and [`min()`](@ref) are the equivalent of `pmax` and `pmin` respectively in R, but both arguments need to have the same dimensions.  
* >  Juliaの[`max（）`]（@ ref）と[`min（）`]（@ ref）はそれぞれRの `pmax`と` pmin`に相当しますが、両方の引数が同じ次元を持つ必要があります。
<!-- End -->
<!-- Start -->
  While [`maximum()`](@ref) and [`minimum()`](@ref) replace `max` and `min` in R, there are important differences.
* >  Rで `max`と` min`を `[maximum（）`]（@ref）と[`minimum（）`]（@ref）で置き換えますが、重要な違いがあります。
<!-- End -->
<!-- Start -->
*  Julia's [`sum()`](@ref), [`prod()`](@ref), [`maximum()`](@ref), and [`minimum()`](@ref) are different from their counterparts in R. 
* >  Juliaの[`sum（）`]（@ ref）、[`prod（）`]（@ ref）、[`maximum（）`]（@ ref）、[`minimum（）`]（@ ref）はRの対応物とは異なる。
<!-- End -->
<!-- Start -->
They all accept one or two arguments. The first argument is an iterable collection such as an array.  
* >  それらはすべて1つまたは2つの引数を受け入れる。最初の引数は、配列などの反復可能なコレクションです。
<!-- End -->
<!-- Start -->
* If there is a second argument, then this argument indicates the dimensions, over which the operation is carried out.  
* >  2番目の引数がある場合、この引数は操作が実行される次元を示します。
<!-- End -->
<!-- Start -->
* For instance, let `A=[[1 2],[3 4]]` in Julia and `B=rbind(c(1,2),c(3,4))` be the same matrix in R.  
* >  例えば、Juliaでは `A = [[1 2]、[3 4]]`とし、Rでは `B = rbind（c（1,2）、c（3,4））`を同じ行列とします。
<!-- End -->
<!-- Start -->
* Then `sum(A)` gives the same result as `sum(B)`, but `sum(A, 1)` is a row vector containing the sum over each column and `sum(A, 2)` is a column vector containing the sum over each row.  
* >  `sum（A、1）`は各列の合計を含む行ベクトルであり、 `sum（A、2）`は列である各行の合計を含むベクトル。
<!-- End -->
<!-- Start -->
* This contrasts to the behavior of R, where `sum(B,1)=11` and `sum(B,2)=12`.  
* >  これは、 `sum（B、1）= 11`と` sum（B、2）= 12`のRの動作とは対照的です。
<!-- End -->
<!-- Start -->
* If the second argument is a vector, then it specifies all the dimensions over which the sum is performed, e.g., `sum(A,[1,2])=10`.  
* >  2番目の引数がベクトルの場合、sum（A、[1,2]）= 10など、合計が実行されるすべての次元を指定します。
<!-- End -->
<!-- Start -->
* It should be noted that there is no error checking regarding the second argument.
* >  2番目の引数についてはエラーチェックが行われないことに注意してください。
<!-- End -->
<!-- Start -->
*  Julia has several functions that can mutate their arguments. For example, it has both [`sort()`](@ref) and [`sort!()`](@ref).
* >  Juliaには引数を変更できるいくつかの関数があります。たとえば、[`sort（）`]（@ ref）と[`sort！（）`]（@ ref）の両方があります。
<!-- End -->
<!-- Start -->
*  In R, performance requires vectorization. 
* >  Rでは、パフォーマンスにはベクトル化が必要です。
<!-- End -->
<!-- Start -->
* In Julia, almost the opposite is true: the best performing code is often achieved by using devectorized loops.
* >  Juliaでは、ほぼ逆のことが言えます。最高のパフォーマンスを発揮するコードは、多くの場合、デベクトル化されたループを使用して実現されます。
<!-- End -->
<!-- Start -->
*  Julia is eagerly evaluated and does not support R-style lazy evaluation. 
* >  Juliaは熱心に評価され、Rスタイルの遅延評価をサポートしていません。
<!-- End -->
<!-- Start -->
* For most users, this means that there are very few unquoted expressions or column names.
* >  ほとんどのユーザーにとって、引用符で囲まれていない式や列名はごくわずかしかありません。
<!-- End -->
<!-- Start -->
*  Julia does not support the `NULL` type.
* >  Juliaは `NULL`型をサポートしていません。
<!-- End -->
<!-- Start -->
*  Julia lacks the equivalent of R's `assign` or `get`.
* >  JuliaはRの `assign`や` get`に相当するものがありません。
<!-- End -->
<!-- Start -->
*  In Julia, `return` does not require parentheses.
* >  ジュリアでは、 `return`はカッコを必要としません。
<!-- End -->
<!-- Start -->
* In R, an idiomatic way to remove unwanted values is to use logical indexing, like in the expression `x[x>3]` or in the statement `x = x[x>3]` to modify `x` in-place. 
* >  Rでは、望ましくない値を取り除く慣用的な方法は、 `x [x> 3]`や `x = x [x> 3]`のような論理的なインデックスを使って、 。
<!-- End -->
<!-- Start -->
In contrast, Julia provides the higher order functions [`filter()`](@ref) and [`filter!()`](@ref), allowing users to write `filter(z->z>3, x)` and `filter!(z->z>3, x)` as alternatives to the corresponding transliterations `x[x.>3]` and `x = x[x.>3]`. Using [`filter!()`](@ref) reduces the use of temporary arrays.
* > これとは対照的にJuliaは高次関数[`filter（）`]（@ref）と[`filter！（）`]（@ref）を提供し、 `filter（z-> z> 3、対応する変換子 `x [x。> 3]`と `x = x [x。> 3]`の代わりに `filter！（z-> z> 3、x）`を使用します。 [`filter！（）`]（@ ref）を使うと、一時配列の使用を減らすことができます。
<!-- End -->
<!-- Start -->

<!-- Start -->

## Noteworthy differences from Python

> ## Pythonとの相違点

<!-- End -->
<!-- Start -->
* Julia requires `end` to end a block. Unlike Python, Julia has no `pass` keyword.
* > Juliaはブロックを終了するために `end`を必要とします。 Pythonとは異なり、Juliaには `pass`キーワードがありません。
<!-- End -->
<!-- Start -->
* In Julia, indexing of arrays, strings, etc. is 1-based not 0-based.
* > Juliaでは、配列、文​​字列などの索引付けは、0ベースではなく1ベースです。
<!-- End -->
<!-- Start -->
* Julia's slice indexing includes the last element, unlike in Python. 
* > Juliaのスライスインデックスには、Pythonとは異なり、最後の要素が含まれています。
<!-- End -->
<!-- Start -->
* `a[2:3]` in Julia is `a[1:3]` in Python.
* > Juliaの `a [2：3]`はPythonの `a [1：3]`です。
<!-- End -->
<!-- Start -->
* Julia does not support negative indexes. 
* > Juliaは負のインデックスをサポートしていません。
<!-- End -->
<!-- Start -->
* In particular, the last element of a list or array is indexed with `end` in Julia, not `-1` as in Python.
* > 特に、リストや配列の最後の要素は、Pythonのように `Julia`では` end`でインデックス付けされ、 `-1`ではインデックス付けされません。
<!-- End -->
<!-- Start -->
* Julia's `for`, `if`, `while`, etc. 
* > ジュリアは「for」、「if」、「while」など
<!-- End -->
<!-- Start -->
* blocks are terminated by the `end` keyword. 
* > ブロックは `end`キーワードで終了します。
<!-- End -->
<!-- Start -->
* Indentation level is not significant as it is in Python.
* > インデントレベルはPythonのように重要ではありません。
<!-- End -->
<!-- Start -->
* Julia has no line continuation syntax: if, at the end of a line, the input so far is a complete expression, it is considered done; otherwise the input continues. 
* > Juliaには行継続構文はありません。行の終わりで、これまでの入力が完全な式であれば、完了とみなされます。それ以外の場合は入力が継続されます。
<!-- End -->
<!-- Start -->
* One way to force an expression to continue is to wrap it in parentheses.
* > 式を強制的に続ける1つの方法は、それをカッコで囲むことです。
<!-- End -->
<!-- Start -->
* Julia arrays are column major (Fortran ordered) whereas NumPy arrays are row major (C-ordered) by default.
* > Julia配列は列メジャー（Fortran順）ですが、NumPy配列はデフォルトで行メジャー（C順序）です。
<!-- End -->
<!-- Start -->
* To get optimal performance when looping over arrays, the order of the loops should be reversed in Julia relative to NumPy (see relevant section of [Performance Tips](@ref man-performance-tips)).
* > 配列をループするときに最適なパフォーマンスを得るには、JuliaでNumPyを基準にしてループの順序を逆にする必要があります（[パフォーマンスのヒント]の関連セクションを参照）（@ ref man-performance-tips））。
<!-- End -->
<!-- Start -->
* Julia's updating operators (e.g. `+=`, `-=`, ...) are *not in-place* whereas NumPy's are. 
* > Juliaの更新演算子（例： `+ =`、 ` - =`、...）は、NumPyの代わりに*インプレース*ではありません。
<!-- End -->
<!-- Start -->
* This means `A = ones(4); B = A; B += 3` doesn't change values in `A`, it rather rebinds the name `B` to the result of the right-hand side `B = B + 3`, which is a new array. 
* > これは、A = ones（4）を意味します。 B = A; 「B + = 3」は「A」の値を変更しないで、新しい配列である「B = B + 3」という右側の結果に名前「B」を再バインドする。
<!-- End -->
<!-- Start -->
* For in-place operation, use `B .+= 3` (see also [dot operators](@ref man-dot-operators)), explicit loops, or `InplaceOps.jl`.
* > インプレース操作では、 `B + + = 3`（[ドット演算子]（@refマンドット演算子）も参照）、明示的なループ、または` InplaceOps.jl`を使用してください。
<!-- End -->
<!-- Start -->
* Julia evaluates default values of function arguments every time the method is invoked, unlike in Python where the default values are evaluated only once when the function is defined. 
* > Juliaは、関数が定義されたときにデフォルト値が一度だけ評価されるPythonとは異なり、メソッドが呼び出されるたびに関数引数のデフォルト値を評価します。
<!-- End -->
<!-- Start -->
* For example, the function `f(x=rand()) = x` returns a new random number every time it is invoked without argument.
* > たとえば、関数 `f（x = rand（））= x`は引数なしで呼び出されるたびに新しい乱数を返します。
<!-- End -->
<!-- Start -->
* On the other hand, the function `g(x=[1,2]) = push!(x,3)` returns `[1,2,3]` every time it is called as `g()`.
* > 一方、 `g（x = [1,2]）= push！（x、3）`は `g（）`と呼ばれるたびに `[1,2,3]`を返します。
<!-- End -->
<!-- Start -->
* In Julia `%` is the remainder operator, whereas in Python it is the modulus.
* > Juliaでは `％`は剰余演算子ですが、Pythonではこれがモジュラスです。
<!-- End -->
<!-- Start -->
<!-- Start -->

## Noteworthy differences from C/C++

> ## C / C ++との相違点

<!-- End -->
* Julia arrays are indexed with square brackets, and can have more than one dimension `A[i,j]`. 
* > ジュリア配列は角括弧で索引付けされ、複数の次元「A [i、j]」を持つことができます。
<!-- End -->
<!-- Start -->
* This syntax is not just syntactic sugar for a reference to a pointer or address as in C/C++. See the Julia documentation for the syntax for array construction (it has changed between versions).
* > この構文は、C / C ++のようにポインタやアドレスへの参照のための文法的な砂糖だけではありません。アレイ構築の構文については、Juliaのマニュアルを参照してください（バージョン間で変更されています）。
<!-- End -->
<!-- Start -->
* In Julia, indexing of arrays, strings, etc. is 1-based not 0-based.
* > Juliaでは、配列、文​​字列などの索引付けは、0ベースではなく1ベースです。
<!-- End -->
<!-- Start -->
* Julia arrays are assigned by reference. After `A=B`, changing elements of `B` will modify `A` as well. 
* > ジュリアアレイは参照により割り当てられる。 `A = B`の後、` B`の要素を変更すると `A`も変更されます。
<!-- End -->
<!-- Start -->
* Updating operators like `+=` do not operate in-place, they are equivalent to `A = A + B` which rebinds the left-hand side to the result of the right-hand side expression.
* > `+ =`のような演算子の更新は、インプレースでは動作しません。これは、左辺を右辺式の結果に再バインドする `A = A + B`と等価です。
<!-- End -->
<!-- Start -->
* Julia arrays are column major (Fortran ordered) whereas C/C++ arrays are row major ordered by default. 
* > Julia配列は列メジャー（Fortranオーダー）ですが、C / C ++配列はデフォルトで順序付けされた行メジャーです。
<!-- End -->
<!-- Start -->
* To get optimal performance when looping over arrays, the order of the loops should be reversed in Julia relative to C/C++ (see relevant section of [Performance Tips](@ref man-performance-tips)).
* > 配列をループするときに最適なパフォーマンスを得るには、C / C ++と比較してJuliaでループの順序を逆にする必要があります（[パフォーマンスのヒント]の関連するセクションを参照）（@ ref man-performance-tips））。
<!-- End -->
<!-- Start -->
* Julia values are passed and assigned by reference. If a function modifies an array, the changes will be visible in the caller.
* > ジュリア値は、渡され、参照によって割り当てられます。関数が配列を変更すると、変更が呼び出し側に表示されます。
<!-- End -->
<!-- Start -->
* In Julia, whitespace is significant, unlike C/C++, so care must be taken when adding/removing whitespace from a Julia program.
* > Juliaでは、C / C ++とは異なり、空白が重要なので、Juliaプログラムから空白を追加/削除するときには注意が必要です。
<!-- End -->
<!-- Start -->
* In Julia, literal numbers without a decimal point (such as `42`) create signed integers, of type `Int`, but literals too large to fit in the machine word size will automatically be promoted to a larger size type, such as `Int64` (if `Int` is `Int32`), `Int128`, or the arbitrarily large `BigInt` type. 
* > Juliaでは、小数点を持たないリテラル数字（ `42`など）は、` Int`型の符号付き整数を作成しますが、機械語サイズに収まらないリテラルは自動的に大きなサイズの型に昇格されますInt64`（ `Int`が` Int32`の場合）、 `Int128`、または任意に大きな` BigInt`型です。
<!-- End -->
<!-- Start -->
* There are no numeric literal suffixes, such as `L`, `LL`, `U`, `UL`, `ULL` to indicate unsigned and/or signed vs. unsigned. 
* > 符号なしおよび/または符号付き対符号なしを示す「L」、「LL」、「U」、「UL」、「ULL」などの数値リテラルサフィックスはありません。
<!-- End -->
<!-- Start -->
* Decimal literals are always signed, and hexadecimal literals (which start with `0x` like C/C++), are unsigned. 
* > 小数のリテラルは常に署名され、16進リテラル（C / C ++のように `0x 'で始まる）は符号なしです。
<!-- End -->
<!-- Start -->
* Hexadecimal literals also, unlike C/C++/Java and unlike decimal literals in Julia, have a type based on the *length* of the literal, including leading 0s. 
* > 16進リテラルは、C / C ++ / Javaとは異なり、Juliaの10進リテラルとは異なり、先頭の0を含むリテラルの* length *に基づく型を持ちます。
<!-- End -->
<!-- Start -->
* For example, `0x0` and `0x00` have type [`UInt8`](@ref), `0x000` and `0x0000` have type [`UInt16`](@ref), then literals with 5 to 8 hex digits have type `UInt32`, 9 to 16 hex digits type `UInt64` and 17 to 32 hex digits type `UInt128`. 
* > 例えば、 `0x0`と` 0x00`はタイプ[UInt8`]（@ref）、 `0x000`と` 0x0000`は[`UInt16`]（@ref）型ですが、5〜8桁のリテラルは`UInt32`型、9〜16桁の数字型` UInt64`、17〜32桁の型 `UInt128`です。
<!-- End -->
<!-- Start -->
* This needs to be taken into account when defining hexadecimal masks, for example `~0xf == 0xf0` is very different from `~0x000f == 0xfff0`. 
* > これは、16進マスクを定義するときに考慮する必要があります。たとえば、 `〜0xf == 0xf0`は`〜0x000f == 0xfff0`と大きく異なります。
<!-- End -->
<!-- Start -->
* 64 bit `Float64` and 32 bit [`Float32`](@ref) bit literals are expressed as `1.0` and `1.0f0` respectively. Floating point literals are rounded (and not promoted to the `BigFloat` type) if they can not be exactly represented.
* > 64ビットの「Float64」と32ビットの「 `Float32`」（@ ref）ビットリテラルは、それぞれ1.0と1.0f0と表されます。浮動小数点型のリテラルは、正確に表現できない場合は丸められます（ `BigFloat`型に昇格しません）。
<!-- End -->
<!-- Start -->
* Floating point literals are closer in behavior to C/C++. Octal (prefixed with `0o`) and binary (prefixed with `0b`) literals are also treated as unsigned.
* > 浮動小数点数リテラルは、C / C ++に近い動作をします。 Octal（接頭辞 `0o`）とバイナリ（接頭辞` 0b`）のリテラルもunsignedとして扱われます。
<!-- End -->
<!-- Start -->
* String literals can be delimited with either `"`  or `"""`, `"""` delimited literals can contain `"` characters without quoting it like `"\""` String literals can have values of other variables or expressions interpolated into them, indicated by `$variablename` or `$(expression)`, which evaluates the variable name or the expression in the context of the function.
* > 文字列リテラルは `` `や` `" ``、 `` "" ""で区切ることができます。区切りリテラルは `" \ ""のように引用符をつけない文字を含むことができます文字列リテラルは他の変数や式`$ variablename`または` $（expression） `で示される変数に補間され、関数の文脈で変数名や式を評価します。
<!-- End -->
<!-- Start -->
* `//` indicates a [`Rational`](@ref) number, and not a single-line comment (which is `#` in Julia)
* > `//`は、 `` Rational``（@ ref）番号を示し、一行コメントではありません（Juliaでは `＃`）
<!-- End -->
<!-- Start -->
* `#=` indicates the start of a multiline comment, and `=#` ends it.
* > `＃=`は複数行のコメントの開始を示し、 `=＃`はそれを終了します。
<!-- End -->
<!-- Start -->
* Functions in Julia return values from their last expression(s) or the `return` keyword.  
* > Juliaの関数は、最後の式または `return`キーワードから値を返します。
<!-- End -->
<!-- Start -->
* Multiple values can be returned from functions and assigned as tuples, e.g. `(a, b) = myfunction()` or `a, b = myfunction()`, instead of having to pass pointers to values as one would have to do in C/C++ (i.e. `a = myfunction(&b)`.
* > 関数から複数の値を返し、タプルとして割り当てることができます。 `（a、b）= myfunction（）`または C / C ++（すなわち、 `a = myfunction（＆b）`）で行わなければならないように、値へのポインタを渡す必要はなく、 `a、b = myfunction（）`
<!-- End -->
<!-- Start -->
* Julia does not require the use of semicolons to end statements. 
* > ジュリアは、ステートメントを終了するためにセミコロンを使用する必要はありません。
<!-- End -->
<!-- Start -->
* The results of expressions are not automatically printed (except at the interactive prompt, i.e. the REPL), and lines of code do not need to end with semicolons. 
* > 式の結果は自動的には印刷されません（対話式のプロンプト、すなわちREPLを除く）、コード行はセミコロンで終わる必要はありません。
<!-- End -->
<!-- Start -->
* [`println()`](@ref) or [`@printf()`](@ref) can be used to print specific output. 
* > [`println（）`]（@ref）または[`@printf（）`]（@ref）は、特定の出力を出力するために使用できます。
<!-- End -->
<!-- Start -->
* In the REPL, `;` can be used to suppress output. `;` also has a different meaning within `[ ]`, something to watch out for. 
* > REPLでは `;`を使用して出力を抑制することができます。 `;`は `[]`の中でも、別の意味を持ちます。
<!-- End -->
<!-- Start -->
* `;` can be used to separate expressions on a single line, but are not strictly necessary in many cases, and are more an aid to readability.
* > `;`は1行で式を区切るために使うことができますが、多くの場合厳密には必要ではなく、読みやすくするためにも役立ちます。
<!-- End -->
<!-- Start -->
* In Julia, the operator [`⊻`](@ref xor) ([`xor`](@ref)) performs the bitwise XOR operation, i.e. [`^`](@ref) in C/C++.  
* > Juliaでは、演算子[`⊻`]（@ref xor）（[`xor`]（@ ref））はC / C ++でビット単位のXOR演算、つまり` `^`]（@ ref）を実行します。
<!-- End -->
<!-- Start -->
* Also, the bitwise operators do not have the same precedence as C/++, so parenthesis may be required.
* > また、ビット演算子はC / ++と同じ優先順位を持たないため、かっこが必要な場合があります。
<!-- End -->
<!-- Start -->
* Julia's [`^`](@ref) is exponentiation (pow), not bitwise XOR as in C/C++ (use [`⊻`](@ref xor), or [`xor`](@ref), in Julia) * Julia has two right-shift operators, `>>` and `>>>`.  `>>>` performs an arithmetic shift, `>>` always performs a logical shift, unlike C/C++, where the meaning of `>>` depends on the type of the value being shifted.
* > Juliaの[`^`]（@ ref）は、C / C ++のようなビット単位のXORではなく、累乗（pow）です（Juliaの[`⊻`]（@ ref xor）、[` xor`] ）* Juliaには、2つの右シフト演算子、 `>>`と `>>>`があります。 `>>`は算術シフトを実行し、`> `はC / C ++とは異なり、常に論理シフトを実行します。ここで、`>>`の意味はシフトされる値の型に依存します。
<!-- End -->
<!-- Start -->
* Julia's `->` creates an anonymous function, it does not access a member via a pointer.
* > Juliaの ` - >`は無名関数を作成し、ポインタを介してメンバにアクセスしません。
<!-- End -->
<!-- Start -->
* Julia does not require parentheses when writing `if` statements or `for`/`while` loops: use `for i in [1, 2, 3]` instead of `for (int i=1; i <= 3; i++)` and `if i == 1` instead of `if (i == 1)`.
* > Juliaは `if`文や` for` / `while`ループを書くときには括弧を必要としません。for（int i = 1; i <= 3; i ++）の代わりに` for 1 in、 ） `と` if if i == 1 'の代わりに `if i == 1'を返します。
<!-- End -->
<!-- Start -->
* Julia does not treat the numbers `0` and `1` as Booleans. 
* > ジュリアは数字「0」と「1」をブーリアンとして扱いません。
<!-- End -->
<!-- Start -->
* You cannot write `if (1)` in Julia, because `if` statements accept only booleans. 
* > Juliaで `if（1）`を書くことはできません。なぜなら、 `if`文はブール値しか受け入れないからです。
<!-- End -->
<!-- Start -->
* Instead, you can write `if true`, `if Bool(1)`, or `if 1==1`.
* > 代わりに、if true、if Bool（1）、またはif 1 == 1と書くことができます。
<!-- End -->
<!-- Start -->
* Julia uses `end` to denote the end of conditional blocks, like `if`, loop blocks, like `while`/ `for`, and functions. 
* > Juliaは `if`のような条件付きブロックの終わりを表すために` end`を使い、 `for` /` for`のようなループブロックと関数を使用します。
<!-- End -->
<!-- Start -->
* In lieu of the one-line `if ( cond ) statement`, Julia allows statements of the form `if cond; statement; end`, `cond && statement` and `!cond || statement`. 
* > 1行の `if（cond）文`の代わりに、Juliaは `if cond;という形式の文を許可します。ステートメント; `cond`と` statement`と `！cond ||ステートメント `。
<!-- End -->
<!-- Start -->
* Assignment statements in the latter two syntaxes must be explicitly wrapped in parentheses, e.g. `cond && (x = value)`, because of the operator precedence.
* > 後者の2つの構文の代入文は、明示的にかっこで囲む必要があります。 `cond &&（x = value）`、演算子の優先順位のためです。
<!-- End -->
<!-- Start -->
* Julia has no line continuation syntax: if, at the end of a line, the input so far is a complete expression, it is considered done; otherwise the input continues. 
* > Juliaには行継続構文はありません。行の終わりで、これまでの入力が完全な式であれば、完了とみなされます。それ以外の場合は入力が継続されます。
<!-- End -->
<!-- Start -->
* One way to force an expression to continue is to wrap it in parentheses.
* > 式を強制的に続ける1つの方法は、それをカッコで囲むことです。
<!-- End -->
<!-- Start -->
* Julia macros operate on parsed expressions, rather than the text of the program, which allows them to perform sophisticated transformations of Julia code. 
* > Juliaマクロは、プログラムのテキストではなく、解析された式で動作し、ジュリアコードの洗練された変換を実行できます。
<!-- End -->
<!-- Start -->
* Macro names start with the `@` character, and have both a function-like syntax, `@mymacro(arg1, arg2, arg3)`, and a statement-like syntax, `@mymacro arg1 arg2 arg3`. 
* > マクロ名は `@`文字で始まり、関数のような構文@mymacro（arg1、arg2、arg3）と文のような構文@mymacro arg1 arg2 arg3の両方を持ちます。
<!-- End -->
<!-- Start -->
* The forms are interchangable; the function-like form is particularly useful if the macro appears within another expression, and is often clearest.
* > フォームは交換可能です。 関数形式のフォームは、マクロが別の式の中に現れ、しばしば最も明瞭な場合に特に便利です。
<!-- End -->
<!-- Start -->
* The statement-like form is often used to annotate blocks, as in the parallel `for` construct: `@parallel for i in 1:n; #= body =#; end`.
* > 文のような形式は、並列のfor構文のようにブロックに注釈を付けるために使われることがよくあります： `@parallel for i in 1：n; ＃= body =＃; 終わり。
<!-- End -->
<!-- Start -->
* Where the end of the macro construct may be unclear, use the function-like form.
* > マクロ構造の終わりが不明な場合は、関数のような形式を使用してください。
<!-- End -->
<!-- Start -->
* Julia now has an enumeration type, expressed using the macro `@enum(name, value1, value2, ...)` For example: `@enum(Fruit, banana=1, apple, pear)`
* > Juliaはマクロ `@enum（name、value1、value2、...）`を使って表現される列挙型を持つようになりました。例えば、 `@enum（Fruit、banana = 1、apple、pear）`
<!-- End -->
<!-- Start -->
* By convention, functions that modify their arguments have a `!` at the end of the name, for example `push!`.
* >  慣例により、引数を変更する関数には、名前の最後に `！`が付いています（例えば `push！`）。
<!-- End -->
<!-- Start -->
* In C++, by default, you have static dispatch, i.e. you need to annotate a function as virtual, in order to have dynamic dispatch. 
* > C ++では、デフォルトで静的ディスパッチがあります。つまり、動的ディスパッチを行うには、関数を仮想として注釈を付ける必要があります。
<!-- End -->
<!-- Start -->
* On the other hand, in Julia every method is "virtual" (although it's more general than that since methods are dispatched on every argument type, not only `this`, using the most-specific-declaration rule).
* > 一方、Juliaでは、すべてのメソッドは「仮想」です（ただし、メソッドは特定の宣言ルールを使用して、すべての引数型でディスパッチされるので、より一般的です）。
<!-- End -->
<!-- Start -->