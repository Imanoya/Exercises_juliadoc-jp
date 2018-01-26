<!-- Start -->

# [Introduction](@id man-introduction)

<!-- End -->
<!-- Start -->
Scientific computing has traditionally required the highest performance, yet domain experts have largely moved to slower dynamic languages for daily work.
> 科学コンピューティングでは伝統的に最高のパフォーマンスが求められてきました、それにも関わらずこの分野の専門家は、 殆どの日常作業を遅い動的言語へ移行しています。
<!-- End -->
<!-- Start -->
We believe there are many good reasons to prefer dynamic languages for these applications, and we do not expect their use to diminish.
> 沢山の好ましい理由から動的言語が自然科学のアプリケーションに選択されていると信じていますし、 今後もその使用が減少するようには思えません。
<!-- End -->
<!-- Start -->
Fortunately, modern language design and compiler techniques make it possible to mostly eliminate the performance trade-off and provide a single environment productive enough for prototyping and efficient enough for deploying performance-intensive applications.
> 幸運なことに、現代の言語設計とコンパイラ技術は、パフォーマンスのトレードオフをほとんど排除し、プロトタイプ化に十分な生産性を持ち、パフォーマンスを重視するアプリケーションを展開するのに十分効率的な単一の環境を提供することができます。
<!-- End -->
<!-- Start -->
The Julia programming language fills this role: it is a flexible dynamic language, appropriate for scientific and numerical computing, with performance comparable to traditional statically-typed languages.
> Juliaプログラミング言語は、この役割を果たします。これは、科学的および数値計算に適した柔軟な動的言語であり、従来の静的型付き言語に匹敵するパフォーマンスを備えています。
<!-- End -->

<!-- Start -->
Because Julia's compiler is different from the interpreters used for languages like Python or R, you may find that Julia's performance is unintuitive at first.
> JuliaのコンパイラはPythonやRのような言語に使われているインタプリタとは異なるため、Juliaのパフォーマンスは最初は直感的ではありません。
<!-- End -->
<!-- Start -->
If you find that something is slow, we highly recommend reading through the [Performance Tips](@ref man-performance-tips) section before trying anything else. -->
> 遅いものが見つかった場合は、[パフォーマンスヒント]（@ ref man-performance-tips）セクションを読んでから、他のものを試してみることを強くお勧めします。
<!-- End -->
<!-- Start -->
Once you understand how Julia works, it's easy to write code that's nearly as fast as C.
> Juliaの仕組みを理解すれば、Cとほぼ同じ速さのコードを書くのは簡単です。
<!-- End -->

<!-- Start -->
Julia features optional typing, multiple dispatch, and good performance, achieved using type inference and [just-in-time (JIT) compilation](https://en.wikipedia.org/wiki/Just-in-time_compilation), implemented using [LLVM](https://en.wikipedia.org/wiki/Low_Level_Virtual_Machine). 
> Juliaは、タイプ推論と[ジャストインタイム（JIT）コンパイル]（https://en.wikipedia.org/wiki/Just-in-time_compilation）を使用して実現される、オプションの入力、マルチディスパッチ、および優れたパフォーマンスを備えています。 [LLVM]（https://en.wikipedia.org/wiki/Low_Level_Virtual_Machine）。
<!-- End -->
<!-- Start -->
It is multi-paradigm, combining features of imperative, functional, and object-oriented programming. 
> 命令型、機能型、オブジェクト指向プログラミングの機能を組み合わせたマルチパラダイムです。
<!-- End -->
<!-- Start -->
Julia provides ease and expressiveness for high-level numerical computing, in the same way as languages such as R, MATLAB, and Python, but also supports general programming. 
> Juliaは、R、MATLAB、Pythonなどの言語と同じように、高度な数値計算のための容易さと表現力を提供しますが、一般的なプログラミングもサポートしています。
<!-- End -->
<!-- Start -->
To achieve this, Julia builds upon the lineage of mathematical programming languages, but also borrows much from popular dynamic languages, including [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)), [Perl](https://en.wikipedia.org/wiki/Perl_(programming_language)), [Python](https://en.wikipedia.org/wiki/Python_(programming_language)), [Lua](https://en.wikipedia.org/wiki/Lua_(programming_language)), and [Ruby](https://en.wikipedia.org/wiki/Ruby_(programming_language)).
> これを実現するために、Juliaは数学的プログラミング言語の系譜に基づいていますが、[Lisp]（https://en.wikipedia.org/wiki/Lisp_（programming_language））、[Perl]（ （Python）（https://en.wikipedia.org/wiki/Python_（プログラミング言語））、[Lua]（https：// ja。 wikipedia.org/wiki/Lua_(programming_language））、[Ruby]（https://en.wikipedia.org/wiki/Ruby_(programming_language））を参照してください。
<!-- End -->

<!-- Start -->
The most significant departures of Julia from typical dynamic languages are:
> ジュリアの典型的な動的言語からの最も重要な出発点は次のとおりです。
<!-- End -->
<!-- Start -->
   * The core language imposes very little; the standard library is written in Julia itself, including primitive operations like integer arithmetic
   * > 中核言語は非常に小さなものです。 標準ライブラリは、整数演算のような基本的な演算を含むJulia自体で書かれています
<!-- End -->
<!-- Start -->
   * A rich language of types for constructing and describing objects, that can also optionally be used to make type declarations
   * > オブジェクトの作成と記述のための型の豊富な言語。型宣言の作成にオプションで使用することもできます。
<!-- End -->
<!-- Start -->
   * The ability to define function behavior across many combinations of argument types via [multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch)
   * > [複数ディスパッチ]（https://en.wikipedia.org/wiki/Multiple_dispatch）を介して、引数タイプの多くの組み合わせにわたる関数の動作を定義する機能。
<!-- End -->
<!-- Start -->
   * Automatic generation of efficient, specialized code for different argument types
   * > 異なる引数型の効率的で特殊なコードの自動生成
<!-- End -->
<!-- Start -->
   *  Good performance, approaching that of statically-compiled languages like C
   * > Cのような静的にコンパイルされた言語に近い性能
<!-- End -->

<!-- Start -->
Although one sometimes speaks of dynamic languages as being "typeless", they are definitely not: every object, whether primitive or user-defined, has a type.
> 動的言語は「タイプなし」と言われることもありますが、基本的であれユーザ定義であれ、すべてのオブジェクトに型があります。
<!-- End -->
<!-- Start -->
The lack of type declarations in most dynamic languages, however, means that one cannot instruct the compiler about the types of values, and often cannot explicitly talk about types at all.
> しかし、ほとんどの動的言語における型宣言の欠如は、コンパイラに値の型について指示することができず、しばしば型について明示的に話すことができないことを意味します。
<!-- End -->
<!-- Start -->
In static languages, on the other hand, while one can -- and usually must -- annotate types for the compiler, types exist only at compile time and cannot be manipulated or expressed at run time.
> 一方、静的言語ではコンパイラの型に注釈を付けることができますが、通常は型に注釈を付ける必要がありますが、型はコンパイル時にのみ存在し、実行時には操作または表現できません。
<!-- End -->
<!-- Start -->
In Julia, types are themselves run-time objects, and can also be used to convey information to the compiler.
> Juliaでは、型はそれ自身が実行時オブジェクトであり、情報をコンパイラに伝えるためにも使用できます。
<!-- End -->

<!-- Start -->
While the casual programmer need not explicitly use types or multiple dispatch, they are the core unifying features of Julia: functions are defined on different combinations of argument types, and applied by dispatching to the most specific matching definition. 
> カジュアルなプログラマは、型や多重ディスパッチを明示的に使用する必要はありませんが、Juliaのコア統合機能です。関数はさまざまな引数型の組み合わせで定義され、最も具体的なマッチング定義にディスパッチすることによって適用されます。
<!-- End -->
<!-- Start -->
This model is a good fit for mathematical programming, where it is unnatural for the first argument to "own" an operation as in traditional object-oriented dispatch. 
> このモデルは、従来のオブジェクト指向のディスパッチのように操作を「所有」する第一引数が不自然である数学的プログラミングに適しています。
<!-- End -->
<!-- Start -->
Operators are just functions with special notation -- to extend addition to new user-defined data types, you define new methods for the `+` function.
> 演算子は特別な表記法を持つ関数に過ぎません。新しいユーザー定義データ型への追加を行うには、 `+`関数の新しいメソッドを定義します。
<!-- End -->
<!-- Start -->
Existing code then seamlessly applies to the new data types.
> 既存のコードは、新しいデータ型にシームレスに適用されます。
<!-- End -->

<!-- Start -->
Partly because of run-time type inference (augmented by optional type annotations), and partly because of a strong focus on performance from the inception of the project, Julia's computational efficiency exceeds that of other dynamic languages, and even rivals that of statically-compiled languages. 
> 部分的には実行時型推論（オプション型アノテーションによって補強されている）が原因で、またプロジェクトの開始からのパフォーマンスに重点を置いていることもあって、Juliaの計算効率は他の動的言語の効率を上回り、静的にコンパイルされた言語。
<!-- End -->
<!-- Start -->
For large scale numerical problems, speed always has been, continues to be, and probably always will be crucial: the amount of data being processed has easily kept pace with Moore's Law over the past decades.
> 大規模な数値問題については、スピードは常にあり続けてきており、おそらく常に重要になります。処理されるデータの量は過去数十年間のムーアの法則に容易に追いついています。
<!-- End -->

<!-- Start -->
Julia aims to create an unprecedented combination of ease-of-use, power, and efficiency in a single language. 
> Juliaは、使い易さ、消費電力、効率性の前例のない組み合わせを単一言語で作成することを目指しています。
<!-- End -->
<!-- Start -->
In addition to the above, some advantages of Julia over comparable systems include:
> 上記に加えて、比較可能なシステムに比べてジュリアのいくつかの利点は以下のとおりです。
<!-- End -->

<!-- Start -->
  * Free and open source ([MIT licensed](https://github.com/JuliaLang/julia/blob/master/LICENSE.md))
  * > 無料かつオープンソース
<!-- End -->
<!-- Start -->
  * User-defined types are as fast and compact as built-ins
  * No need to vectorize code for performance; devectorized code is fast
  * Designed for parallelism and distributed computation
  * Lightweight "green" threading ([coroutines](https://en.wikipedia.org/wiki/Coroutine))
  * Unobtrusive yet powerful type system
  * Elegant and extensible conversions and promotions for numeric and other types
  * Efficient support for [Unicode](https://en.wikipedia.org/wiki/Unicode), including but not limited
  * to [UTF-8](https://en.wikipedia.org/wiki/UTF-8)
  *>  C言語で作成された関数を直接呼び出せます。(ラッパーや特別なAPIを必要としませんl)
  * Call C functions directly (no wrappers or special APIs needed)
  * 強力でシェルに似た機能を持つプロセス制御
  * Powerful shell-like capabilities for managing other processes
  * Lispに似たマクロとメタプログミング機能
  * Lisp-like macros and other metaprogramming facilities
