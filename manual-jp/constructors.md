
<!-- Start -->

 [Cnstructors](@id man-constructors)

# [コンストラクタConstructors](@id man-constructors)

<!-- End -->
<!-- Start -->
Contructors [^1] are functions that create new objects -- specifically, instances of [Composite Types](@ref).
> コンストラクタ[^1]は新しいオブジェクト、具体的には[Composite Types](@ref)のインスタンスを作成する関数です。
<!-- End -->
<!-- Start -->
In ulia, type objects also serve as constructor functions: they create new instances of themselves when applied to an argument tuple as a function. 
> Juliaでは、型オブジェクトはコンストラクタ関数としても機能します。引数タプルを関数として適用すると、型オブジェクトは新しいインスタンスを作成します。
<!-- End -->
<!-- Start -->
Thi much was already mentioned briefly when composite types were introduced.
> これは、複合型が導入されたときに既に簡単に触れられています。
<!-- End -->
<!-- Start -->
Forexample:
> 例えば：
<!-- End -->

```jldoctest footype
julia> struct Foo
           bar
           baz
       end

julia> foo = Foo(1, 2)
Foo(1, 2)

julia> foo.bar
1

julia> foo.baz
2
```

<!-- Start -->
For many types, forming new objects by binding their field values together is all that is ever needed to create instances. 
> 多くのタイプでは、フィールド値をバインドして新しいオブジェクトを形成することは、インスタンスを作成するために必要なものです。
<!-- End -->
<!-- Start -->
Thee are, however, cases where more functionality is required when creating composite objects. 
> しかし、複合オブジェクトを作成するときにはより多くの機能が必要になる場合があります。
<!-- End -->
<!-- Start -->
Somtimes invariants must be enforced, either by checking arguments or by transforming them. 
> 場合によっては、引数を調べたり変換したりすることによって不変量を強制する必要があります。
<!-- End -->
<!-- Start -->
[Recursive data structures](https://en.wikipedia.org/wiki/Recursion_%28computer_science%29#Recursive_data_structures_.28structural_recursion.29), especially those that may be self-referential, often cannot be constructed cleanly without first being created in an incomplete state and then altered programmatically to be made whole, as a separate step from object creation. 
> [再帰的データ構造](https://en.wikipedia.org/wiki/Recursion_%28computer_science%29#Recursive_data_structures_.28structural_recursion.29)、特に自己参照型のものは、最初に作成されることなくきれいに構築することはできません オブジェクトの作成とは別のステップとして、完全に作成されるようにプログラムで変更されています。
<!-- End -->
<!-- Start -->
Somtimes, it's just convenient to be able to construct objects with fewer or different types of parameters than they have fields. 
> 場合によっては、フィールドの数よりも少ないまたは異なるタイプのパラメーターを持つオブジェクトを構築できるのが便利な場合もあります。 
<!-- End -->
<!-- Start -->
Jula's system for object construction addresses all of these cases and more.
> Juliaのオブジェクト構築システムは、これらすべての事例に対応しています。
<!-- End -->

[^1]:
<!-- Start -->
  Nomnclature: while the term "constructor" generally refers to the entire function which constructs objects of a type, it is common to abuse terminology slightly and refer to specific constructor methods as "constructors". 
  > 用語：「コンストラクタ」という用語は一般に、あるタイプのオブジェクトを構成する関数全体を指しますが、用語を少し乱用し、特定のコンストラクタメソッドを「コンストラクタ」と呼ぶのが一般的です。
<!-- End -->
<!-- Start -->
  Insuch situations, it is generally clear from context that the term is used to mean "constructor method" rather than "constructor function", especially as it is often used in the sense of singling out a particular method of the constructor from all of the others.
  > このような状況では、文脈から、この用語は「コンストラクタ関数」ではなく「コンストラクタメソッド」を意味するために使用されることが一般的には明らかです。 特に、コンストラクタの特定のメソッドを他のすべてのメソッドから取り出すという意味でよく使用されます。
<!-- End -->

<!-- Start -->

## Oter Constructor Methods

> ## 外部コンストラクタメソッド

<!-- End -->
<!-- Start -->
A constructor is just like any other function in Julia in that its overall behavior is defined by the combined behavior of its methods. 
> コンストラクタはJuliaの他の関数と同様に、メソッドの動作を組み合わせ全体的な振る舞いが定義されています。
<!-- Start -->
Accordingly, you can add functionality to a constructor by simply defining new methods. 
> したがって、簡単な新しいメソッドを定義するだけで、コンストラクタに機能を追加できます。
<!-- End -->
<!-- Start -->
For example, let's say you want to add a constructor method for `Foo` objects that takes only one argument and uses the given value for both the `bar` and `baz` fields. 
> たとえば`Foo`オブジェクトへ、引数が1つしかなく`bar`フィールドと`baz`フィールドの両方に指定された値を使う、コンストラクタメソッドを追加してみましょう。
<!-- End -->
<!-- Start -->
This is simple:
> これは簡単です：
<!-- End -->

```jldoctest footype
julia> Foo(x) = Foo(x,x)
Foo

julia> Foo(1)
Foo(1, 1)
```

<!-- Start -->
You could also add a zero-argument `Foo` constructor method that supplies default values for both of the `bar` and `baz` fields:
> `bar`フィールドと` baz`フィールドの両方のデフォルト値を提供する、引数のない `Foo`コンストラクタメソッドを追加することもできます：
<!-- End -->

```jldoctest footype
julia> Foo() = Foo(0)
Foo

julia> Foo()
Foo(0, 0)
```

<!-- Start -->
Here the zero-argument constructor method calls the single-argument constructor method, which in turn calls the automatically provided two-argument constructor method. 
> ここでは、引数のないコンストラクターメソッドは、単一引数のコンストラクターメソッドを呼び出し、次に自動的に提供される2つの引数のコンストラクターメソッドを呼び出します。
<!-- End -->
<!-- Start -->
For reasons that will become clear very shortly, additional constructor methods declared as normal methods like this are called *outer* constructor methods. 
> すぐに明らかになるため、通常のメソッドとして宣言されているこのような追加のコンストラクタメソッドは、*outer*コンストラクタメソッドと呼ばれます。
<!-- End -->
<!-- Start -->
Outer constructor methods can only ever create a new instance by calling another constructor method, such as the automatically provided default ones.
> 外部コンストラクタメソッドは、自動的に提供されるデフォルトのインスタンスなど、別のコンストラクタメソッドを呼び出すことで、新しいインスタンスを作成することができます。
<!-- End -->

<!-- Start -->

## inner Constructor Methods

> ## 内部コンストラクタメソッド

<!-- End -->
<!-- Start -->
While outer constructor methods succeed in addressing the problem of providing additional convenience methods for constructing objects, they fail to address the other two use cases mentioned in the introduction of this chapter: enforcing invariants, and allowing construction of self-referential objects. 
> 外側のコンストラクタ・メソッドは、オブジェクトを構築するための追加の便利なメソッドを提供するという問題に対処するのに成功しますが、この章の導入で言及された他の2つのユース・ケース、すなわちインバリアントの実施および自己参照オブジェクトの構築を可能にしません。
<!-- End -->
<!-- Start -->
Forthese problems, one needs *inner* constructor methods.
> これらの問題のためには、*内部(inner)*コンストラクタメソッドが必要です。
<!-- End -->
<!-- Start -->
An nner constructor method is much like an outer constructor method, with two differences:
> 内側のコンストラクタメソッドは外部コンストラクタメソッドによく似ていますが、2つの違いがあります。
<!-- End -->

<!-- Start -->
1. t is declared inside the block of a type declaration, rather than outside of it like normal methods.
> 1. 型宣言のブロックの内部ではなく、通常のメソッドのように宣言されます。
<!-- End -->
<!-- Start -->
2. t has access to a special locally existent function called `new` that creates objects of the block's type.
> 2. ブロックタイプのオブジェクトを作成する `new`というローカルに存在する特殊な関数にアクセスできます。
<!-- End -->

<!-- Start -->
Forexample, suppose one wants to declare a type that holds a pair of real numbers, subject to the constraint that the first number is not greater than the second one.
> たとえば、第1の数が第2の数よりも大きくないという制約に従って、実数の対を保持する型を宣言したいとします。 
<!-- End -->
<!-- Start -->
One could declare it like this:
> これを次のように宣言することができます：
<!-- End -->

```jldoctest pairtype
julia> struct OrderedPair
           x::Real
           y::Real
           OrderedPair(x,y) = x > y ? error("out of order") : new(x,y)
       end

```

<!-- Start -->
Now `OrderedPair` objects can only be constructed such that `x <= y`:
> 今、`OrderedPair` オブジェクトは `x <= y` のようにしか構築できません：
<!-- End -->

```jldoctest pairtype
julia> OrderedPair(1, 2)
OrderedPair(1, 2)

julia> OrderedPair(2,1)
ERROR: out of order
Stacktrace:
 [1] OrderedPair(::Int64, ::Int64) at ./none:4
```

<!-- Start -->
If the type were declared `mutable`, you could reach in and directly change the field values to violate this invariant, but messing around with an object's internals uninvited is considered poor form.
> 型が `mutable`と宣言されていれば、この不変量に違反するようにフィールド値を直接変更することができますが、呼び出されていないオブジェクトを混乱させることは悪い形です。
<!-- End -->
<!-- Start -->
You (or someone else) can also provide additional outer constructor methods at any later point, but once a type is declared, there is no way to add more inner constructor methods. 
> あなた(または他の人)は、任意の後の時点で追加の外部コンストラクターメソッドを提供することもできますが、型が宣言されると、内部コンストラクターメソッドを追加する方法はありません。
<!-- End -->
<!-- Start -->
Since outer constructor methods can only create objects by calling other constructor methods, ultimately, some inner constructor must be called to create an object. 
> 外側のコンストラクタメソッドは他のコンストラクタメソッドを呼び出すことによってのみオブジェクトを作成できるため、最終的には、内部コンストラクタを呼び出してオブジェクトを作成する必要があります。
<!-- End -->
<!-- Start -->
This guarantees that all objects of the declared type must come into existence by a call to one of the inner constructor methods provided with the type, thereby giving some degree of enforcement of a type's invariants.
> これにより、宣言された型のすべてのオブジェクトが、その型に付属する内部コンストラクタメソッドの1つを呼び出すことによって存在しなければならないことが保証され、それによって型の不変量のある程度の強制が与えられます。
<!-- End -->

<!-- Start -->
If any inner constructor method is defined, no default constructor method is provided: it is presumed that you have supplied yourself with all the inner constructors you need. 
> 内部のコンストラクタメソッドが定義されている場合、デフォルトのコンストラクタメソッドは提供されません。必要とするすべての内部コンストラクタを自分自身に提供したと推定されます。
<!-- End -->
<!-- Start -->
The default constructor is equivalent to writing your own inner constructor method that takes all of the object's fields as parameters (constrained to be of the correct type, if the corresponding field has a type), and passes them to `new`, returning the resulting object:
> デフォルトのコンストラクタは、オブジェクトのフィールドのすべてをパラメータ(対応するフィールドに型がある場合は正しい型に制約されている)として取る独自の内部コンストラクタメソッドを記述し、それらを `new`に渡して結果を返します オブジェクト：
<!-- End -->

```jldoctest
julia> struct Foo
           bar
           baz
           Foo(bar,baz) = new(bar,baz)
       end

```

<!-- Start -->
This declaration has the same effect as the earlier definition of the `Foo` type without an explicit inner constructor method. 
> この宣言は、明示的な内部コンストラクタメソッドを持たない `Foo`型の以前の定義と同じ効果を持ちます。
<!-- End -->
<!-- Start -->
The following two types are equivalent -- one with a default constructor, the other with an explicit constructor:
> 次の2つの型は同等です：1つはデフォルトのコンストラクタを使用し、もう1つは明示的なコンストラクタを使用します。
<!-- End -->

```jldoctest
julia> struct T1
           x::Int64
       end

julia> struct T2
           x::Int64
           T2(x) = new(x)
       end

julia> T1(1)
T1(1)

julia> T2(1)
T2(1)

julia> T1(1.0)
T1(1)

julia> T2(1.0)
T2(1)
```

<!-- Start -->
It is considered good form to provide as few inner constructor methods as possible: only those taking all arguments explicitly and enforcing essential error checking and transformation. 
> できるだけ多くの内部コンストラクタメソッドを提供することは、すべての引数を明示的に取って、本質的なエラーチェックと変換を強制するものにしか適していないと考えられます。
<!-- End -->
<!-- Start -->
Additional convenience constructor methods, supplying default values or auxiliary transformations, should be provided as outer constructors that call the inner constructors to do the heavy lifting. 
> 重い持ち上げを行うために内部コンストラクタを呼び出す外部コンストラクタとして、デフォルト値や補助的な変換を提供する追加の便利なコンストラクタメソッドを用意する必要があります。
<!-- End -->
<!-- Start -->
This separation is typically quite natural.
> この分離は、典型的には非常に自然なものです。
<!-- End -->
<!-- Start -->

## Icomplete Initialization

> ## 不完全な初期化

<!-- Start -->
Thefinal problem which has still not been addressed is construction of self-referential objects, or more generally, recursive data structures. 
> それでも対処されていない最終的な問題は、自己参照オブジェクト、またはより一般的には再帰的なデータ構造の構築です。
<!-- End -->
<!-- Start -->
Sine the fundamental difficulty may not be immediately obvious, let us briefly explain it. 
> 根本的な難しさはすぐには分かりませんので、簡単に説明しましょう。
<!-- End -->
<!-- Start -->
Conider the following recursive type declaration:
> 次の再帰型宣言を考えてみましょう。
<!-- End -->

```jldoctest selfrefer
julia> mutable struct SelfReferential
           obj::SelfReferential
       end

```

<!-- Start -->
Thi type may appear innocuous enough, until one considers how to construct an instance of it.
> このタイプは、そのインスタンスを構築する方法を検討するまで、無害に見えるかもしれません。
<!-- End -->
<!-- Start -->
If a `is an instance of` SelfReferential`, then a second instance can be created by the call:
> `a` が `SelfReferential` の *インスタンスである場合*、呼び出しによって2番目のインスタンスを作成することができます：
<!-- End -->

```julia-repl
julia> b = SelfReferential(a)
```

<!-- Start -->
Buthow does one construct the first instance when no instance exists to provide as a valid value for its `obj` field? 
> しかし、 `obj` フィールドの有効な値としてインスタンスが存在しない場合、最初のインスタンスをどのように構築しますか？
<!-- End -->
<!-- Start -->
The only solution is to allow creating an incompletely initialized instance of `SelfReferential` with an unassigned `obj` field, and using that incomplete instance as a valid value for the `obj` field of another instance, such as, for example, itself.
> 唯一の解決策は、割り当てられていない `obj` フィールドを持つ `SelfReferential` のインスタンスを不完全に初期化し、そのインスタンスなどを別のインスタンスの `obj` フィールドの有効な値として使用することです。
<!-- End -->

<!-- Start -->
To llow for the creation of incompletely initialized objects, Julia allows the `new` function to be called with fewer than the number of fields that the type has, returning an object with the unspecified fields uninitialized. 
> 不完全に初期化されたオブジェクトの作成を可能にするために、Juliaは型が持つフィールドの数より少ない数で `new` 関数を呼び出して、未指定のフィールドが初期化されていないオブジェクトを返します。
<!-- End -->
<!-- Start -->
Theinner constructor method can then use the incomplete object, finishing its initialization before returning it. 
> 内部のコンストラクタメソッドは、不完全オブジェクトを使用して、初期化を完了してから戻します。
<!-- End -->
<!-- Start -->
Her, for example, we take another crack at defining the `SelfReferential` type, with a zero-argument inner constructor returning instances having `obj` fields pointing to themselves:
> ここでは、例えば、 `SelfReferential` 型を定義する際に別の亀裂を犯します。引数ゼロの内部コンストラクタは、 `obj` フィールドを持つインスタンスを返します。
<!-- End -->

```jldoctest selfrefer2
julia> mutable struct SelfReferential
           obj::SelfReferential
           SelfReferential() = (x = new(); x.obj = x)
       end

```

<!-- Start -->
We an verify that this constructor works and constructs objects that are, in fact, self-referential:
> このコンストラクタが動作し、自己参照型のオブジェクトを構築することを実際に確かめてみましょう。
<!-- End -->

```jldoctest selfrefer2
julia> x = SelfReferential();

julia> x === x
true

julia> x === x.obj
true

julia> x === x.obj.obj
true
```

<!-- Start -->
Altough it is generally a good idea to return a fully initialized object from an inner constructor, incompletely initialized objects can be returned:
> 内部的なコンストラクタから完全に初期化されたオブジェクトを返すことは一般的には良い考えですが、不完全に初期化されたオブジェクトを返すことができます：
<!-- End -->

```jldoctest incomplete
julia> mutable struct Incomplete
           xx
           Incomplete() = new()
       end

julia> z = Incomplete();
```

<!-- Start -->
Whie you are allowed to create objects with uninitialized fields, any access to an uninitialized reference is an immediate error:
> 初期化されていないフィールドを持つオブジェクトを作成することは許可されていますが、初期化されていない参照へのアクセスはのエラーとなります。
<!-- End -->

```jldoctest incomplete
julia> z.xx
ERROR: UndefRefError: access to undefined reference
```

<!-- Start -->
This avoids the need to continually check for `null` values. 
> これにより、ヌル値を継続的にチェックする必要がなくなります。
<!-- End -->
<!-- Start -->
Howver, not all object fields are references. 
> ただし、すべてのオブジェクトフィールドが参照であるとは限りません。
<!-- End -->
<!-- Start -->
Julia considers some types to be "plain data", meaning all of their data is self-contained and does not reference other objects. 
> Juliaは、いくつかのタイプを`プレーンデータ`とみなします。つまり、すべてのデータが自己完結型で他のオブジェクトを参照していないことを意味します。
<!-- End -->
<!-- Start -->
The plain data types consist of primitive types (e.g. `Int`) and immutable structs of other plain data types. 
> プレーンデータ型は、プリミティブ型(例えば、Int)と,他のプレーンデータ型の不変構造体とからなります。
<!-- End -->
<!-- Start -->
Theinitial contents of a plain data type is undefined:
> プレーンデータ型の初期値は未定義です：
<!-- End -->

```julia-repl
julia> struct HasPlain
           n::Int
           HasPlain() = new()
       end

julia> HasPlain()
HasPlain(438103441441)
```

<!-- Start -->
Arrys of plain data types exhibit the same behavior.
> プレーンなデータ型の配列は同じ動作を示します。
<!-- End -->

<!-- Start -->
Youcan pass incomplete objects to other functions from inner constructors to delegate their completion:
> 不完全なオブジェクトを他の関数に渡して、内部のコンストラクタへ完了を委譲することができます。
<!-- End -->

```jldoctest
julia> mutable struct Lazy
           xx
           Lazy(v) = complete_me(new(), v)
       end
```

<!-- Start -->
As with incomplete objects returned from constructors, if `complete_me` or any of its callees try to access the `xx` field of the `Lazy` object before it has been initialized, an error will be thrown immediately.
コンストラクタから返された不完全なオブジェクトの場合と同様に、 `complete_me`やその呼び出し先のいずれかが初期化される前に` Lazy`オブジェクトの `xx`フィールドにアクセスしようとすると、すぐにエラーがスローされます。
<!-- End -->
<!-- Start -->

## Prametric Constructors

> ## パラメトリックコンストラクタ

<!-- End -->
<!-- Start -->
Parametric types add a few wrinkles to the constructor story. 
> パラメトリック型は、コンストラクタのストーリーにちょっとした新機能を付け加えます。
<!-- End -->
<!-- Start -->
Recall from [Parametric Types](@ref) that, by default, instances of parametric composite types can be constructed either with explicitly given type parameters or with type parameters implied by the types of the arguments given to the constructor.
> [パラメトリック型](@ref)を思い出してください。デフォルトで、インスタンスのパラメトリックコンポジットの型が構築されるのは、明示的に与えられたパラメーターの型、もしくは、コンストラクタに与えられた引数の型から暗黙的に指定されたパラメータの型です。
<!-- End -->
<!-- Start -->
Here are some examples:
> ここではいくつかの例を示します。
<!-- End -->

```jldoctest parametric
julia> struct Point{T<:Real}
           x::T
           y::T
       end

julia> Point(1,2) ## implicit T ##
Point{Int64}(1, 2)

julia> Point(1.0,2.5) ## implicit T ##
Point{Float64}(1.0, 2.5)

julia> Point(1,2.5) ## implicit T ##
ERROR: MethodError: no method matching Point(::Int64, ::Float64)
Closest candidates are:
  Point(::T<:Real, !Matched::T<:Real) where T<:Real at none:2

julia> Point{Int64}(1, 2) ## explicit T ##
Point{Int64}(1, 2)

julia> Point{Int64}(1.0,2.5) ## explicit T ##
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int64}, ::Float64) at ./float.jl:680
 [2] Point{Int64}(::Float64, ::Float64) at ./none:2

julia> Point{Float64}(1.0, 2.5) ## explicit T ##
Point{Float64}(1.0, 2.5)

julia> Point{Float64}(1,2) ## explicit T ##
Point{Float64}(1.0, 2.0)
```

<!-- Start -->
As you can see, for constructor calls with explicit type parameters, the arguments are converted to the implied field types: `Point{Int64}(1,2)` works, but `Point{Int64}(1.0,2.5)` raises an [`InexactError`](@ref) when converting `2.5` to [`Int64`](@ref). 
> 見て分かるように、コンストラクタを明示的な型パラメータで呼び出しすると、引数が暗黙的にフィールドの型へ変換されます: `Point{Int64}(1,2)`は動作しますが、`Point{Int64}(1.0,2.5)`では[`InexactError`](@ref) が`2.5`を[Int64](@ref) へ変換するときに発生します。
<!-- End -->

When the type is implied by the arguments to the constructor call, as in `Point(1,2)`, then the types of the arguments must agree -- otherwise the `T` cannot be determined -- but any pair of real arguments with matching type may be given to the generic `Point` constructor.
> 型が `Point(1,2)`のようにコンストラクタ呼び出しの引数によって暗黙に指定されている場合、引数の型は一致する必要があります。そうでなければ `T`は決定できませんが、マッチする型をジェネリックな`Point`コンストラクタに与えることができます。
<!-- End -->

<!-- Start -->
What's really going on here is that `Point`, `Point{Float64}` and `Point{Int64}` are all different constructor functions. 
> ここで実際に行われていることは、`Point`,`Point{Float64}`及び`Point{Int64}`がすべて異なるコンストラクタ関数であるということです。
<!-- End -->
#In fact, `Point{T}` is a distinct constructor function for each type `T`. 
> 実際、`Point{T}`は`T`型ごとに異なるコンストラクタ関数です。
<!-- End -->
<!-- Start -->
Without any explicitly provided inner constructors, the declaration of the composite type `Point{T<:Real}` automatically provides an inner constructor, `Point{T}`, for each possible type `T<:Real`, that behaves just like non-parametric default inner constructors do. 
> 明示的に提供された内部コンストラクタがなければ、コンポジット型 `Point {T <：Real}`の宣言は、内部的なコンストラクタである` Point {T} `を` T <：Real` ノンパラメトリックなデフォルトの内部コンストラクタはそうです。
<!-- End -->
<!-- Start -->
#It also provides a single general outer `Point` constructor that takes pairs of real arguments, which must be of the same type.
> また、同じタイプのものでなければならない実際の引数のペアをとる単一の一般的な外側の `Point`コンストラクタを提供します。
<!-- End -->
<!-- Start -->
This automatic provision of constructors is equivalent to the following explicit declaration:
> このコンストラクタの自動生成は、次の明示的宣言と同じです。
<!-- End -->

```jldoctest parametric2
julia> struct Point{T<:Real}
           x::T
           y::T
           Point{T}(x,y) where {T<:Real} = new(x,y)
       end

julia> Point(x::T, y::T) where {T<:Real} = Point{T}(x,y);
```

<!-- Start -->
Notice that each definition looks like the form of constructor call that it handles.
> 各定義は、それが処理するコンストラクタ呼び出しの形式のように見えることに注意してください。
<!-- End -->
<!-- Start -->
The call `Point{Int64}(1,2)` will invoke the definition `Point{T}(x,y)` inside the `type` block.
> `Point{Int64}(1,2)`の呼び出しは、`type`ブロック内で`Point{T}(x,y)`の定義を呼び出します。
<!-- End -->
The outer constructor declaration, on the other hand, defines a method for the general `Point` constructor which only applies to pairs of values of the same real type. 
> 一方、外部コンストラクタ宣言は、同じ実数型の値の組にのみ適用される一般的な `Point`コンストラクタのためのメソッドを定義します。
<!-- End -->
<!-- Start -->
This declaration makes constructor calls without explicit type parameters, like `Point(1,2)` and `Point(1.0,2.5)`, work. 
> この宣言は、 `Point(1,2)`や `Point(1.0,2.5)`のような明示的な型パラメータなしのコンストラクタ呼び出しを行います。 
<!-- End -->
<!-- Start -->
Since the method declaration restricts the arguments to being of the same type, calls like `Point(1,2.5)`, with arguments of different types, result in "no method" errors.
> メソッドの宣言では引数が同じ型に制限されているため、 `Point(1,2.5)`のような型の引数を使用すると、メソッドが存在しないというエラーが発生します。
<!-- End -->

<!-- Start -->
Suppose we wanted to make the constructor call `Point(1,2.5)` work by "promoting" the integer value `1` to the floating-point value `1.0`. 
> コンストラクタ呼び出し`Point(1,2.5)`が動くように,整数値`1`を浮動小数点値`1.0`に*昇格*するように作りたいとします。
<!-- Start -->
<!-- End -->
The simplest way to achieve this is to define the following additional outer constructor method:
> 最も簡単な方法でこれを実現するには、次のように追加の外部コンストラクターメソッドを定義することです。
<!-- End -->

```jldoctest parametric2
julia> Point(x::Int64, y::Float64) = Point(convert(Float64,x),y);
```

<!-- Start -->
This method uses the [`convert()`](@ref) function to explicitly convert `x` to [`Float64`](@ref) and then delegates construction to the general constructor for the case where both arguments are [`Float64`](@ref). 
> このメソッドは明示的に `x`を[` Float64`](@ref)に変換するために[`convert()`]](@ref)関数を使い、両方の引数が[`Float64 `](@ref)。
<!-- End -->
<!-- Start -->
With this method definition what was previously a [`MethodError`](@ref) now successfully creates a point of type `Point{Float64}`:
> このメソッド定義で、以前は[`MethodError`](@ref)は` Point {Float64} `型のポイントを正常に作成します：
<!-- End -->

```jldoctest parametric2
julia> Point(1,2.5)
Point{Float64}(1.0, 2.5)

julia> typeof(ans)
Point{Float64}
```

<!-- Start -->
However, other similar calls still don't work:
> しかし、他の同様の呼び出しはまだ動作しません：
<!-- End -->

```jldoctest parametric2
julia> Point(1.5,2)
ERROR: MethodError: no method matching Point(::Float64, ::Int64)
Closest candidates are:
  Point(::T<:Real, !Matched::T<:Real) where T<:Real at none:1
```

<!-- Start -->
For a more general way to make all such calls work sensibly, see [Conversion and Promotion](@ref conversion-and-promotion).
> このような呼び出しを分かりやすくするためのより一般的な方法については、[変換とプロモーション](@ref conversion-and-promotion)を参照してください。
<!-- End -->
<!-- Start -->
At the risk of spoiling the suspense, we can reveal here that all it takes is the following outer method definition to make all calls to the general `Point` constructor work as one would expect:
> サスペンスを台無しにする危険性がある場合、ここでは、次のアウターが必要です 一般的な `Point`コンストラクタへのすべての呼び出しを、期待通りに動作させるためのメソッド定義です：
<!-- End -->

```jldoctest parametric2
julia> Point(x::Real, y::Real) = Point(promote(x,y)...);
```

<!-- Start -->
The `promote` function converts all its arguments to a common type -- in this case [`Float64`](@ref).
> `promote`関数はすべての引数を共通の型に変換します。この場合、` `Float64``(@ref)です。
<!-- End -->
<!-- Start -->
With this method definition, the `Point` constructor promotes its arguments the same way that numeric operators like [`+`](@ref) do, and works for all kinds of real numbers:
> このメソッド定義では、 `Point`コンストラクタは、[`+`](@ref)のような数値演算子と同じように引数を宣言し、あらゆる種類の実数に対して機能します：
<!-- End -->

```jldoctest parametric2
julia> Point(1.5,2)
Point{Float64}(1.5, 2.0)

julia> Point(1,1//2)
Point{Rational{Int64}}(1//1, 1//2)

julia> Point(1.0,1//2)
Point{Float64}(1.0, 0.5)
```

<!-- Start -->
Thus, while the implicit type parameter constructors provided by default in Julia are fairly strict, it is possible to make them behave in a more relaxed but sensible manner quite easily. 
> したがって、デフォルトでJuliaで提供されている暗黙的な型パラメータコンストラクタはかなり厳密ですが、それらをよりリラックスしていながらも感覚的に簡単に動作させることは可能です。
<!-- End -->
<!-- Start -->
Moreover, since constructors can leverage all of the power of the type system, methods, and multiple dispatch, defining sophisticated behavior is typically quite simple.
> さらに、コンストラクタは、型システム、メソッド、および複数のディスパッチのすべての機能を活用できるため、洗練された動作を定義するのは通常非常に簡単です。
<!-- End -->

<!-- Start -->

## Case Study: Rational

## ケーススタディ：Rational

<!-- End -->
<!-- Start -->
Perhaps the best way to tie all these pieces together is to present a real world example of a parametric composite type and its constructor methods. 
> おそらく、これらのすべての要素を結びつける最良の方法は、パラメトリックコンポジット型の実例とそのコンストラクタメソッドを提示することです。
<!-- End -->
<!-- Start -->
To that end, here is the (slightly modified) beginning of [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl), which implements Julia's [Rational Numbers](@ref):
> そのために、ここにJuliaの[Rational Numbers(Rational Numbers)]を実装する[`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl) ](@ref)：
<!-- End -->

```jldoctest rational
julia> struct OurRational{T<:Integer} <: Real
           num::T
           den::T
           function OurRational{T}(num::T, den::T) where T<:Integer
               if num == 0 && den == 0
                    error("invalid rational: 0//0")
               end
               g = gcd(den, num)
               num = div(num, g)
               den = div(den, g)
               new(num, den)
           end
       end

julia> OurRational(n::T, d::T) where {T<:Integer} = OurRational{T}(n,d)
OurRational

julia> OurRational(n::Integer, d::Integer) = OurRational(promote(n,d)...)
OurRational

julia> OurRational(n::Integer) = OurRational(n,one(n))
OurRational

julia> //(n::Integer, d::Integer) = OurRational(n,d)
// (generic function with 1 method)

julia> //(x::OurRational, y::Integer) = x.num // (x.den*y)
// (generic function with 2 methods)

julia> //(x::Integer, y::OurRational) = (x*y.den) // y.num
// (generic function with 3 methods)

julia> //(x::Complex, y::Real) = complex(real(x)//y, imag(x)//y)
// (generic function with 4 methods)

julia> //(x::Real, y::Complex) = x*y'//real(y*y')
// (generic function with 5 methods)

julia> function //(x::Complex, y::Complex)
           xy = x*y'
           yy = real(y*y')
           complex(real(xy)//yy, imag(xy)//yy)
       end
// (generic function with 6 methods)
```

<!-- Start -->
The first line -- `struct OurRational{T<:Integer} <: Real` -- declares that `OurRational` takes one type parameter of an integer type, and is itself a real type. 
> 最初の行 `struct OurRational {T<:Integer} <:Real`は`OurRational`が1つの整数型パラメータをとり、それ自体が実数型であることを宣言します。
<!-- End -->
<!-- Start -->
The field declarations `num::T` and `den::T` indicate that the data held in a `OurRational{T}` object are a pair of integers of type `T`, one representing the rational value's numerator and the other representing its denominator.
> フィールド宣言 `num::T`と`den::T`は、`OurRational {T}`オブジェクトに保持されているデータが有理値の分子を表す`T`型の整数のペアであり、その分母を表します。
<!-- End -->

<!-- Start -->
Now things get interesting. `OurRational` has a single inner constructor method which checks that both of `num` and `den` aren't zero and ensures that every rational is constructed in "lowest terms" with a non-negative denominator. 
> 今は物事が面白いです。 `OurRational`は`num`と `den`の両方がゼロでないことをチェックし、すべての有理数が非負の分母を持つ「最低項」で構成されることを保証する単一の内部コンストラクタメソッドを持っています。
<!-- End -->
<!-- Start -->
This is accomplished by dividing the given numerator and denominator values by their greatest common divisor, computed using the `gcd` function. 
> これは、与えられた分子と分母の値を、 `gcd`関数を使って計算された最大公約数で除算することによって達成されます。
<!-- End -->
<!-- Start -->
Since `gcd` returns the greatest common divisor of its arguments with sign matching the first argument (`den` here), after this division the new value of `den` is guaranteed to be non-negative. 
> `gcd`は引数の最大公約数を最初の引数(ここでは`den`)にマッチさせて返すので、この除算の後では `den`の新しい値は非負であることが保証されます。
<!-- End -->
<!-- Start -->
Because this is the only inner constructor for `OurRational`, we can be certain that `OurRational` objects are always constructed in this normalized form.
> これは `OurRational`のための唯一の内部コンストラクタであるため、`OurRational`オブジェクトは常にこの正規化された形式で構築されていると確信できます。
<!-- End -->

<!-- Start -->
`OurRational` also provides several outer constructor methods for convenience. 
> `OurRational`はまた、いくつかの外部コンストラクターメソッドを提供しています。
<!-- End -->
<!-- Start -->
The first is the "standard" general constructor that infers the type parameter `T` from the type of the numerator and denominator when they have the same type. 
> 1つ目は、型パラメータ `T`を分子と分母の型から推論する*標準*汎用コンストラクタです。
<!-- End -->
<!-- Start -->
The second applies when the given numerator and denominator values have different types: it promotes them to a common type and then delegates construction to the outer constructor for arguments of matching type. 
> 二つ目は、与えられた分子と分母の値が異なる場合に適用されます。共通の型にそれらを昇格させ、次に外部のコンストラクタに構造を委譲して、一致する型の引数を与えます。
<!-- End -->
<!-- Start -->
The third outer constructor turns integer values into rationals by supplying a value of `1` as the denominator.
> 第3の外部コンストラクタは、分母として `1`の値を供給することによって整数値を有理数に変える。
<!-- End -->

<!-- Start -->
Following the outer constructor definitions, we have a number of methods for the [`//`](@ref) operator, which provides a syntax for writing rationals. 
> 外側のコンストラクタの定義に続いて、[`//`](@ref)演算子のための多数のメソッドがあります。これは、有理数を書くための構文を提供します。
<!-- End -->
<!-- Start -->
Before these definitions, [`//`](@ref) is a completely undefined operator with only syntax and no meaning. 
> これらの定義の前に、[`//`](@ref)は完全に定義されていない演算子であり、構文だけで意味はありません。
<!-- End -->
<!-- Start -->
Afterwards, it behaves just as described in [Rational Numbers](@ref) -- its entire behavior is defined in these few lines.
> その後、[Rational Numbers](@ref)に記述されているように動作します。その全体の動作は、これらの数行で定義されています。
<!-- End -->
<!-- Start -->
The first and most basic definition just makes `a//b` construct a `OurRational` by applying the `OurRational` constructor to `a` and `b` when they are integers. 
> 最初の最も基本的な定義では、 `a//b`は整数であるときに`OurRational`コンストラクタを`a`と `b`に適用することによって `OurRational`を構成します。
<!-- End -->
<!-- Start -->
When one of the operands of [`//`](@ref) is already a rational number, we construct a new rational for the resulting ratio slightly differently; this behavior is actually identical to division of a rational with an integer.
> [`//`](@ref)のオペランドの1つがすでに有理数である場合、結果の比率に対して少し異なる方法で新しい有理数を構築します。この振る舞いは実際には有理数と整数の除算と同じです。
<!-- End -->
<!-- Start -->
Finally, applying [`//`](@ref) to complex integral values creates an instance of `Complex{OurRational}` -- a complex number whose real and imaginary parts are rationals:
> 最後に、複素積分値に[`//`](@ref)を適用すると、実数部と虚数部が有理数である複素数である `Complex {OurRational}`のインスタンスが生成されます。
<!-- End -->

```jldoctest rational
julia> ans = (1 + 2im)//(1 - 2im);

julia> typeof(ans)
Complex{OurRational{Int64}}

julia> ans <: Complex{OurRational}
false
```

<!-- Start -->
Thus, although the [`//`](@ref) operator usually returns an instance of `OurRational`, if either of its arguments are complex integers, it will return an instance of `Complex{OurRational}` instead.
> したがって、[`//`](@ref)演算子は通常、`OurRational`のインスタンスを返しますが、いずれかの引数が複素数の場合、代わりに `Complex {OurRational}`のインスタンスを返します。
<!-- End -->
<!-- Start -->
The interested reader should consider perusing the rest of [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl): it is short, self-contained, and implements an entire basic Julia type.
> 興味のある読者は、残りの[`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)を熟読することを検討するべきです：それは短く、自己完結型であり、実装しています基本的なジュリアタイプ全体。
<!-- End -->
<!-- Start -->

## [Constructors and Conversion](@id constructors-and-conversion)

> ## [コンストラクタと変換](@id コンストラクタと変換)

<!-- End -->
<!-- Start -->
Constructors `T(args...)` in Julia are implemented like other callable objects: methods are added to their types. 
> Juliaでコンストラクタ`T(args ...)`は、他の呼び出し可能なオブジェクトのように実装されます。メソッドは型に追加されます。
<!-- End -->
<!-- Start -->
The type of a type is `Type`, so all constructor methods are stored in the method table for the `Type` type. 
> タイプの型は `Type`なので、すべてのコンストラクタメソッドは`Type`型のメソッドテーブルに格納されます。
<!-- End -->
<!-- Start -->
This means that you can declare more flexible constructors, e.g. constructors for abstract types, by explicitly defining methods for the appropriate types.
> つまり、より柔軟なコンストラクタを宣言することができます。適切な型のメソッドを明示的に定義することによって、抽象型のコンストラクタを作成します。
<!-- End -->

<!-- Start -->
However, in some cases you could consider adding methods to `Base.convert` *instead* of defining a constructor, because Julia falls back to calling [`convert()`](@ref) if no matching constructor is found. 
> 場合によっては、メソッドを`Base.convert`に追加することが、コンストラクタを定義する*代わりに*考えられます。なぜなら、Juliaはコンストラクタが見つからなければ[convert()`](@ref)を呼び出すためです。
<!-- End -->
<!-- Start -->
For example, if no constructor `T(args...) = ...` exists `Base.convert(::Type{T}, args...) = ...` is called.
> たとえば、 `T(args...)= ...`コンストラクタが存在しない場合、`Base.convert(::Type {T},args...)= ...`が呼び出されます。
<!-- End -->

<!-- Start -->
`convert` is used extensively throughout Julia whenever one type needs to be converted to another (e.g. in assignment, [`ccall`](@ref), etcetera), and should generally only be defined (or successful) if the conversion is lossless. 
> `convert`は、あるタイプを別のタイプに変換する必要があるとき(例えば、代入、[`ccall`](@ref) など)、Julia全体で広く使われていて、変換がロスレスであれば一般的に定義されていて(成功)します。
<!-- Start -->
For example, `convert(Int, 3.0)` produces `3`, but `convert(Int, 3.2)` throws an `InexactError`.
> たとえば、 `convert(Int,3.0)` は `3`を生成しますが、`convert(Int,3.2)` は `InexactError` をスローします。
<!-- End -->
<!-- Start -->
If you want to define a constructor for a lossless conversion from one type to another, you should probably define a `convert` method instead.
> ある型から別の型への可逆変換のためのコンストラクタを定義したい場合は、代わりに `convert` メソッドを定義するべきでしょう。
<!-- End -->

<!-- Start -->
On the other hand, if your constructor does not represent a lossless conversion, or doesn't represent "conversion" at all, it is better to leave it as a constructor rather than a `convert` method.
> 一方、あなたのコンストラクタがロスレスな変換を表していない場合や、 *変換*を全く表現していない場合は、 `convert` メソッドではなくコンストラクタとして残す方が良いでしょう。
<!-- End -->
<!-- Start -->
For example, the `Array{Int}()` constructor creates a zero-dimensional `Array` of the type `Int`, but is not really a "conversion" from `Int` to an `Array`.
> たとえば、`Array {Int}()` コンストラクタは、`Int` 型のゼロ次元の `Array` を作成し、実際には `Int` から `Array` への *変換* ではありません。
<!-- End -->
<!-- Start -->

## Outer-only constructors

> ## 外部専用のコンストラクタ

<!-- End -->
<!-- Start -->
As we have seen, a typical parametric type has inner constructors that are called when type parameters are known; e.g. they apply to `Point{Int}` but not to `Point`. 
> これまで見てきたように、典型的なパラメトリック型には、型パラメータがわかっているときに呼び出される内部コンストラクタがあります。例えば`Point{Int}`には適用されますが、`Point`には適用されません。
<!-- End -->
<!-- Start -->
Optionally, outer constructors that determine type parameters automatically can be added, for example constructing a `Point{Int}` from the call `Point(1,2)`. 
> オプションで、タイプパラメータを自動的に決定する外部コンストラクタを追加することができます。たとえば、`Point(1,2)`コールから `Point{Int}`を構築することができます。
<!-- End -->
<!-- Start -->
Outer constructors call inner constructors to do the core work of making an instance. 
> 外部のコンストラクタは内部のコンストラクタを呼び出して、インスタンスを作成する中核的な作業を行います。
<!-- End -->
<!-- Start -->
However, in some cases one would rather not provide inner constructors, so that specific type parameters cannot be requested manually.
> しかし、場合によっては内部コンストラクタを提供しないので、特定の型パラメータを手動で要求することはできません。
<!-- End -->

<!-- Start -->
For example, say we define a type that stores a vector along with an accurate representation of its sum:
> 例えば、ベクトルの和を正確に表現したベクトルを格納する型を定義するとします。
<!-- End -->

```jldoctest
julia> struct SummedArray{T<:Number,S<:Number}
           data::Vector{T}
           sum::S
       end

julia> SummedArray(Int32[1; 2; 3], Int32(6))
SummedArray{Int32,Int32}(Int32[1, 2, 3], 6)
```

<!-- Start -->
The problem is that we want `S` to be a larger type than `T`, so that we can sum many elements with less information loss. 
> 問題は`S`を`T`よりも大きな型にして、情報損失を抑え多くの要素を合計できるようにすることです。
<!-- End -->
<!-- Start -->
For example, when `T` is [`Int32`](@ref), we would like `S` to be [`Int64`](@ref). 
> たとえば、`T`が[`Int32`](@ref) の場合、`S`は[`Int64`](@ref)にしたいと思います。
<!-- End -->
<!-- Start -->
Therefore we want to avoid an interface that allows the user to construct instances of the type `SummedArray{Int32,Int32}`. 
> したがって、ユーザが `SummedArray {Int32,Int32}`型のインスタンスを構築できるようにするインタフェースを避けたいとします。
<!-- End -->
<!-- Start -->
One way to do this is to provide a constructor only for `SummedArray`, but inside the `type` definition block to suppress generation of default constructors:
> これを行う1つの方法は、コンストラクタを`SummedArray`に対してのみ提供することで、`type`定義ブロック内でデフォルトのコンストラクタの生成を抑止します：
<!-- End -->

```jldoctest
julia> struct SummedArray{T<:Number,S<:Number}
           data::Vector{T}
           sum::S
           function SummedArray(a::Vector{T}) where T
               S = widen(T)
               new{T,S}(a, sum(S, a))
           end
       end

julia> SummedArray(Int32[1; 2; 3], Int32(6))
ERROR: MethodError: no method matching SummedArray(::Array{Int32,1}, ::Int32)
Closest candidates are:
  SummedArray(::Array{T,1}) where T at none:5
```

<!-- Start -->
This constructor will be invoked by the syntax `SummedArray(a)`.
> このコンストラクタは、 `SummedArray(a)`という構文で呼び出されます。
<!-- End -->
<!-- Start -->
The syntax `new{T,S}` allows specifying parameters for the type to be constructed, i.e. this call will return a `SummedArray{T,S}`.
> `new{T,S}`構文は、構築される型のパラメータを指定することを可能にします。つまり、この呼び出しは `SummedArray{T,S}`を返します。
<!-- End -->
<!-- Start -->
`new{T,S}` can be used in any constructor definition, but for convenience the parameters to `new{}` are automatically derived from the type being constructed when possible.
> `new{T,S}`は任意のコンストラクタ定義で使うことができますが、便宜上、`new{}`のパラメータは可能なときに自動的に生成される型から派生します。
<!-- End -->
